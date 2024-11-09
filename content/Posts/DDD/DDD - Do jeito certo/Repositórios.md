---
title: Repositórios | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Repository
  - Pattern
  - DotNET
---
**Conceito de Repositório no DDD**:
- Um repositório é visto como uma coleção em memória que armazena todos os objetos de um determinado tipo. Ele permite que se adicione, remova e recupere instâncias dessa coleção de forma simples, ocultando dos desenvolvedores o fato de que esses dados estão sendo persistidos em um banco de dados ou outro meio persistente.

**Relacionamento entre Repositórios e Agregados**:
- Segundo Eric Evans, há uma relação quase sempre de 1 para 1 entre agregados e repositórios. Isso significa que a quantidade de repositórios geralmente corresponde à quantidade de agregados no modelo de domínio.

**Exemplo de Código**:
```csharp
// Interface de Repositório em C#
public interface IRepository<T> where T : IAggregateRoot
{
    // Adiciona uma entidade ao repositório
    void Add(T entity);

    // Atualiza uma entidade no repositório
    void Update(T entity);

    // Remove uma entidade do repositório
    void Remove(T entity);

    // Obtém uma entidade pelo ID
    T GetById(Guid id);

    // Obtém todos os itens do repositório
    IEnumerable<T> GetAll();
}
```
- **IAggregateRoot**: Essa interface de marcação indica quais são as raízes de agregado no modelo de domínio. Todas as raízes de agregado são entidades, e uma entidade é identificada principalmente pelo seu ID.
- **Métodos Básicos**: Um repositório geralmente inclui métodos para adicionar, atualizar, remover e recuperar entidades por ID.

**Consultas e Agregados**:
- Se há a necessidade de buscar objetos mais leves (com menos propriedades), o repositório pode retornar instâncias dessas representações mais simples, mas é importante não confundir essa abordagem com a persistência de dados.
- Se consultas complexas são comuns e exigem objetos mais leves, é recomendado considerar a utilização de um *Data Access Object* (DAO) ou uma outra abordagem específica para consultas.

**Limite Transacional**:
- Um agregado define um limite transacional. Isso significa que todas as entidades dentro de um agregado devem ser persistidas em uma única transação.
- Se houver a necessidade de transações que envolvem múltiplos agregados, é recomendada a utilização de um padrão como o *Unit of Work*.

**Implementação e Separação**:
- A interface do repositório faz parte do modelo de domínio, enquanto a implementação concreta pode ser colocada em outro pacote ou *namespace*. Isso permite a flexibilidade de trocar o mecanismo de persistência sem alterar o domínio.
- **Exemplo Prático**: Mesmo que as recomendações gerais apontem para a separação da implementação concreta do repositório, na prática, muitos sistemas utilizam um único mecanismo de persistência. Nesses casos, pode ser mais eficiente manter tudo junto e separar somente quando necessário.

**Complexidade e Design**:
- O objetivo do DDD é combater a complexidade no coração do software, não adicioná-la. Quando repositórios são mapeados para tabelas do banco de dados em vez de agregados, ocorre uma falha conceitual, resultando em repositórios anêmicos que, na verdade, são *Data Access Objects*.

[Repositórios](https://youtu.be/FeqpZ9w-CRA?si=SchQfXra3JX3wP30)

[[Que idioma utilizar no código que expressa o modelo de domínio?]]
