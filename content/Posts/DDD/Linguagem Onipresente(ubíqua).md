---
title: Linguagem Onipresente(ubíqua)
date: 2024-03-24
tags:
  - DDD
  - BoundedContext
---
**O que é DDD?**

- Domain-Driven Design (DDD) é um conjunto de ideias e princípios para modelagem de software.
- Ele possui uma forte influência nas regras de negócio.
- Foi idealizado para manter um padrão linguístico entre as equipes de domínio e técnica.
- DDD não é uma arquitetura.
- Não oferece uma solução pronta (receita de bolo) ou uma solução universal (bala de prata).
- Foi idealizado por Eric Evans em 2003.

### Linguagem Onipresente

- Define uma linguagem comum entre os integrantes das equipes de negócio e técnica.
- Nas fronteiras que os delimitam, essa linguagem é absoluta.
- Essas fronteiras são chamadas de contextos delimitados.

### Exemplo de Linguagem Onipresente

- O que define a fronteira entre as linguagens onipresentes são os contextos delimitados.
- Não existe nada absoluto; as linguagens onipresentes não devem ser tratadas como algo fixo ou imutável.
- Se o negócio evoluir e houver a necessidade de mudar essa linguagem, uma reunião deve ser realizada imediatamente para discutir essa mudança.
- A linguagem onipresente é definida em conjunto por gerentes, especialistas de domínio e equipe técnica. Nunca se deve criar uma linguagem onipresente sem o consenso dessas partes, pois o acordo comum é o início do sucesso na implementação do DDD.

![[ddd.1.png]]

A `PESSOA` seria a linguagem onipresente; é o nome dado a um tipo de entidade ou objeto de valor. No entanto, no mesmo sistema, pode haver outro objeto também chamado `PESSOA`, o que pode ser problemático.

Mas, quando delimitamos os contextos, faz mais sentido ter essas duas entidades com o mesmo nome.

![[ddd.2.png]]

Quando temos essas duas entidades dentro de um contexto delimitado, como o `contexto de funcionários` e o `contexto de clientes`, é possível ter a `Pessoa` em ambos os contextos. O contexto vai definir o que essa `Pessoa` representa no sistema.
### Exemplo de Projeto

**Estrutura do Domínio:**

`MusicCorp.Core.Domain`

- **/Contexts**
  - **Contexts/Album**
  - **Contexts/Artistas**
  - **Contexts/Clientes**
  - **Contexts/Fornecedores**
  - **Contexts/Pedidos**

Os contextos são organizados dessa forma por dois motivos principais. Primeiro, se for necessário dividir o sistema em serviços menores no futuro, os contextos já estarão separados, facilitando a extração de cada um como uma aplicação independente. Essa separação foi planejada desde a estrutura inicial do domínio.

**Estrutura de Álbum:**

- **/Album**
  - **/Album/Entities**
  - **/Album/ValueObjects**

O contexto de `Album` contém entidades e objetos de valor, representando um domínio real. Tudo dentro deste contexto é construído com base nessa abstração.

---

### Exemplo de Implementação

**Classe Base para Álbum:**

A classe `AlbumBase` é uma classe abstrata que define as propriedades principais de um álbum, permitindo o compartilhamento de informações entre as entidades concretas (filhas) que a herdam.

```csharp
public abstract class AlbumBase
{
    protected AlbumBase(string nomePrincipal, AlbumDetalhes detalhes, AlbumHistoria historia, DateTime dataEntrada)
    {
        NomePrincipal = nomePrincipal;
        Detalhes = detalhes;
        Historia = historia;
        DataEntrada = dataEntrada;
    }

    public string NomePrincipal { get; private set; }

    public AlbumDetalhes Detalhes { get; private set; }

    public AlbumHistoria Historia { get; private set; }

    public DateTime DataEntrada { get; private set; }
}
```

Utilizar classes base é uma boa prática, pois permite o compartilhamento de funcionalidades e propriedades comuns entre as subclasses.

**Evite a Obsessão por Tipos Primitivos:**

Outro ponto importante é evitar a obsessão por tipos primitivos. Em vez de usar tipos como `int` ou `string` diretamente, é recomendável criar objetos de valor. Objetos de valor são tipos que encapsulam dados, mas não possuem identidade própria.

**Exemplo de Objeto de Valor:**

```csharp
public record AlbumDetalhes
{
    public AlbumDetalhes(string nomeOriginal, string descricao, string origem)
    {
        NomeOriginal = nomeOriginal;
        Descricao = descricao;
        Origem = origem;
    }

    public string NomeOriginal { get; init; }

    public string Descricao { get; init; }

    public string Origem { get; init; }
}
```

Neste exemplo, `AlbumDetalhes` é um objeto de valor que encapsula detalhes do álbum, evitando o uso direto de tipos primitivos como `string`. 
