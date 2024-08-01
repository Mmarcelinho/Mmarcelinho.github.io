---
title: "Testando Handlers do MediatR: Abordagem de Testes Unitários"
date: 2024-08-01
tags:
  - CSharp
  - DotNET
  - Tests
  - Xunit
  - MediatR
---
Ao adotar o padrão CQRS (Command Query Responsibility Segregation), é comum utilizar a biblioteca MediatR para facilitar a separação das responsabilidades e manter o código desacoplado e escalável. Embora o padrão CQRS traga benefícios significativos, ele também aumenta a complexidade do sistema. Por isso, o uso de ferramentas como o MediatR pode ser bastante vantajoso. No entanto, por ser uma biblioteca externa, surgem dúvidas sobre como abordar a criação de testes unitários de maneira eficaz e se existe uma abordagem padrão para isso.

Neste post, vou mostrar como implementei e criei testes unitários para os `Handlers` do MediatR, que são responsáveis por lidar com os casos de uso do projeto. 
## Utilitários para Testes

Antes de mais nada, precisamos configurar um projeto dedicado para criar utilitários de teste. Esses utilitários são essenciais para a criação de Mocks e para evitar a repetição de código em todos os testes, além de centralizar as modificações em um único lugar, facilitando a manutenção do código.

Para este projeto, utilizaremos duas bibliotecas fundamentais:

- **Bogus**: uma biblioteca .NET que gera dados fictícios realistas de maneira fácil e controlada. Ela é ideal para testes que exigem objetos populados com dados consistentes, como nomes de usuários, endereços ou e-mails, sem depender de dados reais ou de uma base de dados.

- **Moq**: uma das bibliotecas de mocking mais populares para .NET. Com o Moq, é possível criar objetos simulados (mocks) que imitam o comportamento de dependências externas, como serviços ou repositórios. Isso facilita o teste de unidades de código sem a necessidade de interagir com implementações reais.

A seguir, apresento os mocks que serão utilizados no exemplo de teste:

### Mock do Repositório de Escrita

```csharp
public class ClienteWriteOnlyRepositorioBuilder
{
    public static IClienteWriteOnlyRepositorio Instancia()
    {
        var mock = new Mock<IClienteWriteOnlyRepositorio>();
        return mock.Object;
    }
}
```

Esta classe cria e retorna um mock de `IClienteWriteOnlyRepositorio` usando o Moq. O método `Instancia` cria um objeto simulado do repositório de escrita, permitindo a realização de testes sem depender de uma implementação real.

### Mock do Repositório de Leitura

```csharp
public class ClienteReadOnlyRepositorioBuilder
{
    private readonly Mock<IClienteReadOnlyRepositorio> _repositorio;

    public ClienteReadOnlyRepositorioBuilder() => _repositorio = new Mock<IClienteReadOnlyRepositorio>();

    public ClienteReadOnlyRepositorioBuilder RecuperarTodos(List<Cliente> clientes)
    {
        _repositorio.Setup(repositorio => repositorio.RecuperarTodos()).ReturnsAsync(clientes);
        return this;
    }

    public ClienteReadOnlyRepositorioBuilder RecuperarPorId(Cliente cliente)
    {
        _repositorio.Setup(repositorio => repositorio.RecuperarPorId(cliente.Id)).ReturnsAsync(cliente);
        return this;
    }

    public ClienteReadOnlyRepositorioBuilder RecuperarClienteExistente(string nomeEmpresa)
    {
        _repositorio.Setup(repositorio => repositorio.ExisteClienteComEmpresa(nomeEmpresa)).ReturnsAsync(true);
        return this;
    }

    public IClienteReadOnlyRepositorio Instancia() => _repositorio.Object;
}
```

Esta classe cria e configura um mock de `IClienteReadOnlyRepositorio`. O `ClienteReadOnlyRepositorioBuilder` inicializa o mock e define comportamentos esperados para métodos como `RecuperarTodos`, `RecuperarPorId`, e `RecuperarClienteExistente`. Esses métodos configuram o mock para retornar dados simulados conforme necessário nos testes.

### Mock da Unidade de Trabalho (UnitOfWork)

```csharp
public static class UnidadeDeTrabalhoBuilder
{
    public static IUnidadeDeTrabalho Instancia()
    {
        var mock = new Mock<IUnidadeDeTrabalho>();
        return mock.Object;
    }
}
```

Esta classe cria e retorna um mock de `IUnidadeDeTrabalho` utilizando o Moq. O método `Instancia` fornece um objeto simulado da unidade de trabalho, permitindo testar o comportamento sem uma implementação real.

### Mock do `RegistrarClienteCommand`

```csharp
public class RegistrarClienteCommandBuilder
{
    public static RegistrarClienteCommand Instancia()
    {
        var faker = new Faker();

        return new RegistrarClienteCommand(
            new RequisicaoClienteJson(
                faker.Company.CompanyName(),
                (SistemaCliente.Communication.Enums.Porte)faker.Random.Int(0, 2)
            )
        );
    }
}
```

Esta classe cria uma instância de `RegistrarClienteCommand` usando a biblioteca `Bogus` para gerar dados fictícios. O `Faker` cria um nome de empresa e um valor de porte aleatório, que são usados para construir um objeto `RequisicaoClienteJson`. Este objeto é então utilizado para criar um `RegistrarClienteCommand`, facilitando a simulação de `Commands` em testes.

O caso de uso que será testado é o `Registrar` da entidade `Cliente`. Como mencionado anteriormente, criei um mock de `RegistrarClienteCommand`, que é um record utilizado pelo `Handle` do caso de uso `Registrar`.

```csharp
public record RegistrarClienteCommand(RequisicaoClienteJson requisicaoCliente) : IRequest<RespostaClienteJson>;

public class RegistrarClienteCommandHandler : IRequestHandler<RegistrarClienteCommand, RespostaClienteJson>
{}

// Implementação ...
```

Com a configuração dos mocks e a definição do caso de uso, podemos agora criar os testes. Para isso, você deve criar um projeto de teste utilizando `Xunit` e adicionar o pacote `FluentAssertions` para facilitar as asserções em seus testes. 

Não irei detalhar a configuração do projeto de teste e a adição de pacotes aqui para manter a brevidade. Certifique-se de adicionar as referências aos projetos necessários para que os testes possam acessar e verificar o comportamento do código.
### RegistrarClienteCommandTest

Primeiro, precisamos de um método que crie um caso de uso fictício e retorne o `Handler` de `RegistrarClienteCommand`.

```csharp
private static RegistrarClienteCommandHandler CriarUseCase(string? nomeEmpresa = null)
{
    var repositorioWrite = ClienteWriteOnlyRepositorioBuilder.Instancia();
    var repositorioRead = new ClienteReadOnlyRepositorioBuilder();
    var unidadeDeTrabalho = UnidadeDeTrabalhoBuilder.Instancia();

    if (!string.IsNullOrWhiteSpace(nomeEmpresa))
        repositorioRead.RecuperarClienteExistente(nomeEmpresa);

    return new RegistrarClienteCommandHandler(repositorioWrite, repositorioRead.Instancia(), unidadeDeTrabalho);
}
```

O método `CriarUseCase` configura o ambiente para o teste, criando instâncias dos mocks necessários. Ele configura o `repositorioRead` para simular a existência de um cliente, se um `nomeEmpresa` for fornecido. Em seguida, retorna uma instância do `RegistrarClienteCommandHandler` configurado com esses mocks. Esse método ajuda a simplificar a criação do handler para diferentes cenários de teste.

### Testes

#### Teste de Sucesso

```csharp
[Fact]
public async Task Sucesso()
{
    var command = RegistrarClienteCommandBuilder.Instancia();

    var useCase = CriarUseCase();

    var resultado = await useCase.Handle(command, default);

    resultado.Should().NotBeNull();
    resultado.NomeEmpresa.Should().Be(command.requisicaoCliente.NomeEmpresa);
    resultado.Porte.Should().Be(command.requisicaoCliente.Porte);
}
```

Neste teste, um `Command` é criado usando `RegistrarClienteCommandBuilder.Instancia()`, e o caso de uso é configurado com o método `CriarUseCase`. O `Handle` do handler é chamado com o `Command`, e o resultado é verificado usando FluentAssertions. Asserções são feitas para garantir que o resultado não seja nulo e que os valores retornados correspondam aos valores fornecidos no `Command`.

#### Teste de Falha

```csharp
[Fact]
public async Task ClienteExistente_DeveRetornarErro()
{
    var command = RegistrarClienteCommandBuilder.Instancia();

    var useCase = CriarUseCase(command.requisicaoCliente.NomeEmpresa);

    Func<Task> acao = async () => await useCase.Handle(command, default);

    var resultado = await acao.Should().ThrowAsync<Exception>();

    resultado.Where(ex => ex.Message.Contains(ClienteMensagensDeErro.CLIENTE_JA_REGISTRADO));
}
```

Neste teste, um `Command` é criado e o caso de uso é configurado para simular a existência de um cliente com o nome fornecido. O teste verifica se o `Handle` do handler lança uma exceção quando o cliente já está registrado. A exceção é verificada com FluentAssertions para garantir que a mensagem de erro contenha a mensagem esperada.
### Conclusão

Neste post, vimos como testar handlers do MediatR usando o padrão CQRS. Exploramos a configuração de mocks, além da criação de testes unitários com `Xunit` e `FluentAssertions`. Essas práticas garantem que seus handlers funcionem corretamente e ajudam a manter seu código limpo e confiável.

## Projeto

- [SistemaCliente](https://github.com/Mmarcelinho/sistema_cliente)
