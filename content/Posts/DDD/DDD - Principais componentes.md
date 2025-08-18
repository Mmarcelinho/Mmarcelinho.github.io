---
title: DDD - Principais componentes
date: 2024-11-08
tags:
  - DDD
  - Components
  - DotNET
---
No DDD (Domain-Driven Design), a estruturação correta dos elementos do domínio é essencial para manter a integridade do sistema e garantir que as regras de negócio sejam respeitadas. Abaixo, discutimos a importância de Objetos de Valor, Entidades, Agregados, Serviços de Domínio e Repositórios.

#### Exemplo de Implementação: Modelagem do Pedido

Considere o seguinte exemplo de uma classe `Pedido`:

```csharp
namespace Ecommerce.Core.Domain.Models;

public class Pedido
{
    public int Id { get; set; }

    public int Ordem { get; set; }

    public string Descricao { get; set; }

    public DateTime DataEntrega { get; set; }
}
```

Esta classe, no estado atual, é considerada **anêmica**. Uma classe anêmica contém apenas propriedades, sem comportamentos ou validações, o que pode se tornar perigoso à medida que o sistema cresce. Essa modelagem simplista pode comprometer a integridade do domínio, especialmente quando surgem necessidades de definir regras imutáveis ou validar dados críticos.

### Problemas com o Modelo Anêmico

- **Violação do Domínio**: Um domínio pode ser "ferido" quando informações importantes do sistema são armazenadas de maneira incorreta ou inválida no banco de dados. Isso ocorre quando as regras de negócio não são aplicadas corretamente nas entidades.
  
- **Falta de Regras Imutáveis**: É vital que certas regras de negócio sejam imutáveis e aplicadas diretamente no "core" do sistema, garantindo que a integridade seja mantida.

### Passo a Passo para uma Modelagem Saudável

Para evitar esses problemas, é necessário repensar a modelagem da classe `Pedido` e introduzir conceitos como Agregados e Objetos de Valor, que encapsulam regras de negócio e mantêm a consistência do sistema.

#### **Definindo o Agregado**

Um agregado é um conjunto de entidades e objetos de valor que são tratados como uma única unidade de consistência. O agregado geralmente tem uma entidade raiz, que é o único ponto de acesso para modificar o agregado. Neste caso, podemos tratar `Pedido` como um agregado.

### Estrutura do Domínio: Agregados, Entidades, Objetos de Valor e Serviços de Domínio

Ao organizar os elementos do domínio, como Agregados, Entidades, Objetos de Valor e Serviços de Domínio, é importante manter uma separação clara entre eles. No exemplo a seguir, mostramos como estruturar corretamente essas componentes.

#### Organização do Projeto

- `Domain/Aggregates`
- `Domain/Entities`
- `Domain/ValueObjects`
- `Domain/Services`

### O Que é um Objeto de Valor?

Um **Objeto de Valor** é um conceito importante em DDD. Ele representa algo que não possui identidade própria, mas cujo valor é determinado pelo conteúdo que carrega. Ao modelar um objeto, podemos perceber que ele não faz sentido por si só, ou seja, ele não possui uma identidade única. O que realmente importa em um Objeto de Valor são suas propriedades internas.

Por exemplo, considere um documento como uma CNH ou CPF. O valor do documento está em seu conteúdo, como o número de registro, e não no próprio documento como uma entidade distinta. Ao comparar dois documentos, a igualdade é baseada em suas propriedades internas, como o número de documento, e não na identidade do objeto. Isso é diferente de uma entidade, que possui uma identidade própria e única. Por exemplo, duas pessoas têm identidades diferentes, e são as identidades que distinguem uma da outra, independentemente das propriedades internas.

### Implementação das Entidades

#### Entidade `Pedido`

Primeiro, vamos criar uma entidade chamada `Pedido`. Esta classe será responsável por representar um pedido no sistema.

```csharp
public sealed class Pedido 
{
    public int Id { get; set; }
    
    public int Ordem { get; set; }

    public string Descricao { get; set; }
}
```

#### Criando a Entidade `Item`

Agora, vamos criar uma entidade chamada `Item`. Dois itens com descrições iguais podem não ser os mesmos, pois cada um pode ter características diferentes, como preços distintos ou promoções específicas. Assim, `Item` é uma entidade com identidade única.

```csharp
public sealed class Item 
{
    public int Id { get; set; }

    public string Descricao { get; set; }

    public string Nome { get; set; }
}
```

### Classe Base para Entidades

Para garantir que todas as entidades compartilhem propriedades comuns, como a data de criação, criamos uma classe base chamada `EntityBase`. Isso permite que todas as entidades herdem propriedades compartilhadas de uma única classe.

```csharp
public abstract class EntityBase
{
    public DateTime DataCriacao { get; set; } = DateTime.UtcNow;
}
```

### Herdando de `EntityBase`

Agora, vamos fazer com que a entidade `Item` herde de `EntityBase`. Isso assegura que toda vez que um `Item` for criado, ele tenha sua data de criação registrada automaticamente.

```csharp
public sealed class Item : EntityBase
{
    public int Id { get; set; }

    public string Descricao { get; set; }

    public string Nome { get; set; }
}
```

### Resultado

Com essa estrutura, toda vez que criarmos um item, ele possuirá automaticamente uma data de criação no momento em que é instanciado. Isso garante que todas as entidades dentro do sistema mantenham consistência em relação às suas propriedades compartilhadas, como a data de criação.

### Estrutura do Domínio: Implementação de Entidades e Objetos de Valor

Agora, vamos estender nossa estrutura de domínio com a criação da classe `Pessoa` e de um `Value Object` para encapsular nomes e documentos.

#### Entidade `Pessoa`

Primeiro, vamos criar a entidade `Pessoa` que representa uma pessoa no sistema:

```csharp
public sealed class Pessoa : EntityBase
{
    public int Id { get; set; }

    public string Nome { get; set; }

    public string Sobrenome { get; set; }

    public string NumeroDocumento { get; set; }

    public ETipoPessoa TipoPessoa { get; set; }
}
```

#### Enumeração para Tipo de Pessoa

Definimos um enum para representar os tipos de pessoas, seja física ou jurídica:

```csharp
public enum ETipoPessoa
{
    Fisica,
    Juridica
}
```

### Identificando a Obsessão por Tipos Primitivos

Na classe `Pessoa`, há uma obsessão por tipos primitivos. Quando notamos essa obsessão, começamos a identificar oportunidades para utilizar Objetos de Valor (Value Objects) que encapsulam comportamentos e regras de negócio.

#### Criando o Value Object `Nome`

Vamos criar um `Value Object` chamado `Nome` que encapsula o primeiro nome e o sobrenome:

```csharp
public sealed record Nome
{
    public string PrimeiroNome { get; init; }

    public string Sobrenome { get; init; }
}
```

O `record` facilita a comparação de valores, pois ao comparar dois records, ele compara seus valores internos. Usamos `init` para que os valores sejam imutáveis após a criação.

### Atualizando a Entidade `Pessoa` para Usar o Value Object `Nome`

Agora, vamos atualizar a entidade `Pessoa` para usar o `Value Object` `Nome`:

```csharp
public sealed class Pessoa : EntityBase
{
    public int Id { get; set; }

    public Nome Nome { get; set; }

    public string NumeroDocumento { get; set; }

    public ETipoPessoa TipoPessoa { get; set; }
}
```

### Criando o Value Object `Documento`

De forma semelhante, vamos criar um `Value Object` para encapsular o número do documento e o tipo de pessoa:

```csharp
public sealed record Documento
{
    public string NumeroDocumento { get; set; }

    public ETipoPessoa TipoPessoa { get; set; }
}
```

### Atualizando a Entidade `Pessoa` para Usar o Value Object `Documento`

Atualizamos novamente a entidade `Pessoa` para usar o `Value Object` `Documento`:

```csharp
public sealed class Pessoa : EntityBase
{
    public int Id { get; set; }

    public Nome Nome { get; set; }

    public Documento Documento { get; set; }
}
```

### Métodos de Instanciação e Lógica de Negócio

Para encapsular a lógica de criação e validação dos documentos, implementamos métodos específicos:

#### Criando Método para Recuperar Tipo de Pessoa

```csharp
private ETipoPessoa GetTipoPessoa()
{
    if (this.NumeroDocumento.Length == 11)
        return ETipoPessoa.Fisica;

    return ETipoPessoa.Juridica;
}
```

#### Definindo a Propriedade `TipoPessoa`

```csharp
public ETipoPessoa TipoPessoa => GetTipoPessoa();
```

#### Construtor Privado para `Documento`

Criamos um construtor privado que define as regras de criação:

```csharp
private Documento(string numeroDocumento)
{
    NumeroDocumento = numeroDocumento;
    this.TipoPessoa = GetTipoPessoa();
}
```

#### Método `Create` para `Documento`

O método `Create` é utilizado para instanciar o `Documento`:

```csharp
public static Documento Create(string numeroDocumento) 
    => new Documento(numeroDocumento);
```

### Usando o Pattern Criacional em `Nome`

De forma semelhante, implementamos um método criacional para `Nome`:

```csharp
public record Nome
{
    private Nome(string primeiroNome, string sobrenome)
    {
        PrimeiroNome = primeiroNome;
        Sobrenome = sobrenome;
    }

    public string PrimeiroNome { get; init; }

    public string Sobrenome { get; init; }

    public static Nome Create(string primeiroNome, string sobrenome) 
        => new Nome(primeiroNome, sobrenome);
}
```

Com essa estrutura, garantimos que a criação e manipulação de objetos no domínio sejam feitas de maneira consistente e segura, aplicando corretamente as regras de negócio e evitando a obsessão por tipos primitivos.

### Encapsulamento e Criação de Entidades e Agregados no Domínio

Para garantir a integridade das entidades no domínio, é essencial encapsular suas propriedades, restringindo o acesso e a modificação direta. Vamos explorar como aplicar esses conceitos nas entidades `Pessoa`, `Pedido` e `Item`, além de configurar agregados para organizar as relações entre elas.

#### Privando Propriedades das Entidades

Para evitar que as propriedades sejam alteradas de forma indevida, todas as propriedades de nossas entidades serão privadas. Assim, elas só poderão ser alteradas através de métodos específicos ou no momento da criação da entidade:

```csharp
public sealed class Pessoa : EntityBase
{
    public Nome Nome { get; private set; }

    public Documento Documento { get; private set; }
}
```

```csharp
public sealed class Pedido : EntityBase
{
    public int Ordem { get; private set; }

    public string Descricao { get; private set; }
}
```

#### Refatorando a Base de Entidade

Podemos mover a propriedade `Id` para uma classe base, fazendo com que todas as entidades herdem essa característica. O identificador será um `Guid`, gerado automaticamente na criação da entidade:

```csharp
public abstract class EntityBase
{
    public Guid Id { get; } = Guid.NewGuid();

    public DateTime DataCriacao { get; set; } = DateTime.UtcNow;
}
```

#### Construtor Privado para Entidades

A criação de uma entidade deve passar por um construtor privado que encapsula a lógica de criação, garantindo que todas as regras sejam seguidas:

```csharp
private Pessoa(string nome, string sobrenome, string numeroDocumento)
{
    Nome = Nome.Create(nome, sobrenome);
    Documento = Documento.Create(numeroDocumento);
}
```

Aqui, os métodos `Create` para `Nome` e `Documento` são utilizados para garantir que os objetos de valor sejam criados corretamente com base nas regras de negócio.

#### Método de Fábrica para Criar Instâncias

Para instanciar uma entidade de forma controlada, utilizamos um método estático de fábrica:

```csharp
public static Pessoa Create(string nome, string sobrenome, string numeroDocumento)
    => new Pessoa(nome, sobrenome, numeroDocumento);
```

O mesmo conceito é aplicado em `Pedido` e `Item`:

```csharp
public sealed class Pedido : EntityBase
{
    private Pedido(int ordem, string descricao)
    {
        Ordem = ordem;
        Descricao = descricao;
    }

    public static Pedido Create(int ordem, string descricao)
        => new Pedido(ordem, descricao);
}
```

```csharp
public sealed class Item : EntityBase
{
    private Item(string descricao, string nome)
    {
        Descricao = descricao;
        Nome = nome;
    }

    public static Item Create(string descricao, string nome)
        => new Item(descricao, nome);
}
```

### Criando um Agregado

Para organizar e dar sentido às entidades, criamos agregados. Um agregado reúne uma entidade raiz e outras entidades ou objetos de valor que estão logicamente relacionados.

#### Definindo o Agregado `OrdemPedido`

Neste exemplo, criamos o agregado `OrdemPedido` para encapsular a relação entre `Pessoa` (cliente) e `Pedido`:

```csharp
public class OrdemPedido
{
    public Pessoa Cliente { get; set; }

    public Pedido Pedido { get; set; }
}
```

### Transformando `Pedido` em um Agregado

O `Pedido` precisa de uma lista de `Item` para representar os produtos comprados. Como os itens são parte integrante do pedido, `Pedido` torna-se um agregado. Para associar o pedido a um cliente, adicionamos a propriedade `ClienteId`:

```csharp
public sealed class Pedido : EntityBase
{
    private Collection<Item> _itens = new();

    private Pedido(int ordem, string descricao, Guid clienteId)
    {
        Ordem = ordem;
        Descricao = descricao;
        ClienteId = clienteId;
    }

    public Guid ClienteId { get; private set; }

    public int Ordem { get; private set; }

    public string Descricao { get; private set; }

    public IReadOnlyCollection<Item> Itens => _itens;

    public static Pedido Create(int ordem, string descricao, Guid clienteId)
        => new Pedido(ordem, descricao, clienteId);

    public void AddItem(Item item)
        => _itens.Add(item);
}
```

#### Protegendo a Lista de Itens

A coleção `_itens` é privada e só pode ser modificada através de métodos controlados como `AddItem`. Para expor a lista de itens de forma segura, utilizamos uma `IReadOnlyCollection`:

```csharp
public IReadOnlyCollection<Item> Itens => _itens;
```

Essa abordagem garante que as entidades e agregados sejam criados e manipulados de acordo com as regras de negócio definidas, protegendo a integridade dos dados e promovendo um design orientado ao domínio (DDD) eficaz.

### Construtor Privado e Método de Criação em `OrdemPedido`

Para garantir que a instância de `OrdemPedido` seja criada de maneira controlada e consistente, utilizamos um construtor privado e um método estático de fábrica, `Create`. Isso assegura que o objeto seja inicializado corretamente com suas dependências essenciais, `Pessoa` e `Pedido`.

```csharp
public class OrdemPedido : EntityBase
{
    private OrdemPedido(Pessoa cliente, Pedido pedido)
    {
        Cliente = cliente;
        Pedido = pedido;
    }

    public Pessoa Cliente { get; set; }

    public Pedido Pedido { get; set; }

    public static OrdemPedido Create(Pessoa cliente, Pedido pedido)
        => new OrdemPedido(cliente, pedido);
}
```

### Definição das Interfaces de Repositório

Agora que as entidades e agregados estão definidos, podemos criar as interfaces de repositório que serão responsáveis pela persistência e recuperação de dados. Essas interfaces fornecem métodos para interagir com as entidades de forma abstrata, facilitando a implementação e substituição das classes concretas de repositório.

#### Interface `IItemRepository`

Esta interface define os métodos necessários para gerenciar itens no sistema, como listar todos os itens, buscar itens por ID de pedido, recuperar um item específico, inserir e deletar itens.

```csharp
public interface IItemRepository
{
    IEnumerable<Item> GetAllItens();

    IEnumerable<Item> GetItensByPedidoId(Guid pedidoId);

    Item GetItemById(Guid id);

    void InsertItem(Item item);

    void DeleteItemById(Guid id);
}
```

#### Interface `IPessoaRepository`

Similarmente, a interface `IPessoaRepository` define os métodos para gerenciar instâncias de `Pessoa`, incluindo a listagem de todas as pessoas, recuperação por ID, inserção e deleção de pessoas.

```csharp
public interface IPessoaRepository
{
    IEnumerable<Pessoa> GetAllItens();

    Pessoa GetPessoaById(Guid id);

    void InsertPessoa(Pessoa item);

    void DeletePessoaById(Guid id);
}
```

Com essas interfaces, temos uma base sólida para implementar os repositórios que irão interagir com a camada de persistência de dados, mantendo a separação de responsabilidades e facilitando a manutenção e teste do código.