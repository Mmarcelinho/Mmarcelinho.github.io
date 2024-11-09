---
title: Middleware de validação no pipeline de requisições em ASP.NET Core
date: 2024-05-20
tags:
  - CSharp
  - DotNET
  - Validations
  - Pipeline
  - FluentValidation
  - MediatR
---
O processo de validação e o pipeline de requisições são essenciais para assegurar a integridade e a consistência das operações em uma aplicação ASP.NET Core. A seguir, vamos explorar como o pipeline funciona e como a utilização do behavior do MediatR pode aprimorar a lógica de validação.

## Funcionamento do pipeline de requisições

O pipeline de requisições é responsável por:
- Receber as requisições HTTP.
- Processar essas requisições.
- Gerar e retornar as respostas correspondentes.

### Componentes do pipeline

- **Middlewares**: São os componentes que compõem o pipeline. Cada middleware tem a responsabilidade de processar a requisição de maneira específica.
- **Operações dos Middlewares**: Incluem autenticação, roteamento, tratamento de erros, entre outras funções.
- **Execução em Cadeia**: Os middlewares são executados em uma ordem definida, onde cada um pode modificar a requisição ou a resposta antes de passá-la para o próximo middleware.

## Abordagem Tradicional vs. Abordagem com Behavior do MediatR

### Abordagem Tradicional

Na abordagem tradicional, a lógica de validação é realizada diretamente no comando, após este ser recebido pela aplicação. Embora funcional, essa abordagem pode tornar o código menos organizado e mais difícil de manter.

### Abordagem com Behavior do MediatR

Ao utilizar o behavior do MediatR, a lógica de validação é inserida dentro do pipeline. Isso garante que a validação ocorra antes do comando chegar à aplicação, proporcionando um código mais limpo e organizado.

![[Modelo de pipeline com validação.png]]

## Etapas para implementação:

### Inclusão do pacote FluentValidation

Para incluir o pacote FluentValidation no projeto Application, utilize os seguintes comandos:

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
```

### Implementação da classe `ValidationsBehavior`

A seguir, implementaremos a classe `ValidationsBehavior`, que atuará como middleware de validação para as requisições.

#### Construtor

O construtor da classe `ValidationsBehavior` recebe uma coleção de validadores (`IValidator<TRequisicao>`) e os armazena em `_validators`.

```csharp
public class ValidationsBehavior<TRequisicao, TResposta> : IPipelineBehavior<TRequisicao, TResposta> 
    where TRequisicao : IRequest<TResposta>
{
    private readonly IEnumerable<IValidator<TRequisicao>> _validators;

    public ValidationsBehavior(IEnumerable<IValidator<TRequisicao>> validators)
    {
        _validators = validators;
    }
```

#### Método `Handle`

O método `Handle` é responsável por realizar a validação. Ele verifica se há validadores disponíveis, cria um contexto de validação, executa a validação de todos os validadores de forma assíncrona, coleta todas as falhas de validação e, se houver falhas, lança uma exceção. Caso contrário, chama o próximo middleware no pipeline (`next()`).

```csharp
    public async Task<TResposta> Handle(TRequisicao requisicao, RequestHandlerDelegate<TResposta> next, CancellationToken cancellationToken)
    {
        if (_validators.Any())
        {
            var context = new ValidationContext<TRequisicao>(requisicao);
            var validationResults = await Task.WhenAll(_validators.Select(v =>
                v.ValidateAsync(context, cancellationToken)));

            var failures = validationResults
                .SelectMany(r => r.Errors)
                .Where(f => f != null)
                .ToList();

            if (failures.Count != 0)
                throw new ValidationException(failures);
        }
        return await next();
    }
}
```

- **Verificação de Validadores**: O método verifica se há validadores disponíveis.
- **Contexto de Validação**: Cria um contexto de validação (`ValidationContext<TRequisicao>`).
- **Execução da Validação**: Executa a validação de todos os validadores de forma assíncrona.
- **Coleta de Falhas**: Coleta todas as falhas de validação.
- **Exceção de Validação**: Se houver falhas, lança uma exceção (`ValidationException`).
- **Próximo Middleware**: Caso contrário, chama o próximo middleware no pipeline (`next()`).

### Registro do `ValidationsBehavior` no serviço

Para que o `ValidationsBehavior` funcione corretamente, precisamos registrá-lo no serviço do ASP.NET Core. Abaixo está um exemplo de como fazer isso:

#### Método `AdicionarMediatR`

Este método configura o MediatR, registrando o `ValidationsBehavior` como um comportamento aberto para todas as requisições e respostas, e adiciona todos os validadores do assembly especificado.

```csharp
private static void AdicionarMediatR(IServiceCollection services)
{
    var myHandlers = AppDomain.CurrentDomain.Load("CQRS.Estoque.Application");

    services.AddMediatR(config =>
    {
        config.RegisterServicesFromAssemblies(myHandlers);
        config.AddOpenBehavior(typeof(ValidationsBehavior<,>));
    });

    services.AddValidatorsFromAssembly(Assembly.Load("CQRS.Estoque.Application"));
}
```

#### Método `AdicionarApplication`

Este método chama `AdicionarMediatR` para o serviço.

```csharp
public static void AdicionarApplication(this IServiceCollection services, IConfiguration configuration)
{
    AdicionarMediatR(services);
}
```

Por fim, chamamos o método `AdicionarApplication` no `Program.cs`  para garantir que todas as configurações sejam aplicadas:

```csharp
builder.Services.AdicionarApplication(builder.Configuration);
```

Essa abordagem com MediatR proporciona um pipeline organizado e garante que todas as validações sejam realizadas antes que o comando seja processado pela aplicação, aumentando a robustez e a manutenibilidade do código.

## Projeto

- [CQRS](https://github.com/Mmarcelinho/cqrs_estoque_api)