---
title: Tratamento de exceções globais com Problem Details
date: 2024-06-19
tags:
  - CSharp
  - DotNET
  - Exceptions
  - ErrorMessages
  - ProblemDetails
---
Após utilizar `ValidationBehavior` para realizar a validação no pipeline de requisições, precisei ajustar minhas exceções personalizadas para tratar as mensagens de erro. Encontrei uma solução que trata as exceções globalmente de forma padronizada utilizando Problem Details através da interface `IExceptionHandler`, que utiliza o middleware `UseExceptionHandler` integrado ao .NET.

[[Middleware de validação no pipeline de requisições em ASP.NET Core]]

Primeiramente, fiz alterações nas exceções personalizadas:

```csharp
namespace SistemaDeEstoque.Exceptions.ExceptionsBase
{
    public abstract class SistemaDeEstoqueException : SystemException
    {
        protected SistemaDeEstoqueException(string mensagem) : base(mensagem) { }

        public abstract int StatusCode { get; }

        public abstract List<string> RecuperarErros();
    }
}
```

A classe base agora é abstrata.

```csharp
namespace SistemaDeEstoque.Exceptions.ExceptionsBase
{
    public class LoginInvalidoException : SistemaDeEstoqueException
    {
        public LoginInvalidoException() : base(UsuarioModelMensagensDeErro.LOGIN_INVALIDO) { }

        public override int StatusCode => (int)HttpStatusCode.Unauthorized;

        public override List<string> RecuperarErros()
        {
            return new List<string> { Message };
        }
    }
}
```

As classes derivadas sobrescrevem seus métodos.

Para adicionar o tratamento de exceções global, o processo é simples.

Na camada de API, criei uma pasta chamada Exceptions e dentro dela a classe `GlobalExceptionHandler`:

### Definição da Classe

```csharp
public class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
```
Esta linha define a classe `GlobalExceptionHandler`, que implementa a interface `IExceptionHandler`. Ela também utiliza o `ILogger` para registrar logs.

### Método `TryHandleAsync`

```csharp
public async ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
{
```

Este método é assíncrono e tenta lidar com exceções que ocorrem durante o processamento das requisições HTTP. Ele recebe o contexto HTTP (`HttpContext`), a exceção (`Exception`) e um token de cancelamento (`CancellationToken`).

### Criação de `ProblemDetails`
```csharp
ProblemDetails problemDetails = new()
{
    Instance = httpContext.Request.Path
};
```

Aqui, um objeto `ProblemDetails` é instanciado. Este objeto é utilizado para fornecer uma resposta de erro detalhada e padronizada. A propriedade `Instance` é definida com o caminho da requisição atual.

### Tratamento de Exceções de Validação

```csharp
if (exception is FluentValidation.ValidationException fluentException)
{
    problemDetails.Title = "Um ou mais erros de validação ocorreram.";
    problemDetails.Type = "https://tools.ietf.org/html/rfc7231#section-6.5.1";
    httpContext.Response.StatusCode = StatusCodes.Status400BadRequest;
    var validationErrors = new List<string>();
    foreach (var error in fluentException.Errors)
    {
        validationErrors.Add(error.ErrorMessage);
    }
    problemDetails.Extensions.Add("errors", validationErrors);
}
```

Este bloco de código verifica se a exceção é do tipo `FluentValidation.ValidationException`. Se for, ele configura o título e o tipo da resposta de erro. Além disso, define o status HTTP para 400 (Bad Request) e adiciona os erros de validação à extensão `errors` de `ProblemDetails`.

### Tratamento de Outras Exceções

```csharp
else
{
    problemDetails.Title = exception.Message;
    httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
}
```

Se a exceção não for de validação, este bloco configura o título da resposta de erro com a mensagem da exceção e define o status HTTP para 500 (Internal Server Error).

### Registro do Log e Envio da Resposta

```csharp
logger.LogError("{ProblemDetailsTitle}", problemDetails.Title);

problemDetails.Status = httpContext.Response.StatusCode;
await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken).ConfigureAwait(false);
return true;
```

Nesta parte final, a mensagem de erro é registrada no log. A propriedade `Status` de `ProblemDetails` é configurada com o status da resposta HTTP. A resposta JSON é escrita e enviada ao cliente, completando o tratamento da exceção.

Este método retorna `true` indicando que a exceção foi tratada com sucesso.

No arquivo `Program.cs`, adicionei o seguinte código:

```csharp
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

app.UseExceptionHandler();
```

O código acima garante que a implementação seja registrada no contêiner de serviço do aplicativo junto com `ProblemDetails`.

Essa abordagem padroniza o tratamento de exceções em toda a aplicação, tornando o processo de depuração e manutenção mais eficiente e consistente.

### Testando

- Requisição

![[req-invalido-exception.png]]

- Resposta 

![[res-exception.png]]

Você deve ter percebido que na saída do console aparecem logs adicionais que vêm diretamente do middleware integrado. Para remover esses logs, basta inserir um trecho de código no `appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.AspNetCore.Diagnostics.ExceptionHandlerMiddleware": "None" <-
    }
  },
  "AllowedHosts": "*"
}
```

## Projeto

- [Sistema de Estoque](https://github.com/Mmarcelinho/sistema_de_estoque)

## Referências

- [Tratamento de exceções globais no ASP.NET Core - IExceptionHandler no .NET 8](https://codewithmukesh.com/blog/global-exception-handling-in-aspnet-core/)
