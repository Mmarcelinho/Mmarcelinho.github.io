---
title: Aggregate Root Design
date: 2024-11-08
tags:
  - DDD
  - Aggregates
  - Design
  - DotNET
---
![[aggregate.1.png]]
### 1. **Contexto de Design Orientado a Domínios (DDD)**

Em DDD (Domain-Driven Design), um *Aggregate* (Agregado) é um padrão que ajuda a manter a integridade dos dados e a consistência dentro de um modelo de domínio. Um agregado é composto por uma raiz de agregado e uma coleção de entidades e objetos de valor que estão relacionadas entre si. A raiz de agregado é a única entrada para manipulação e acesso aos dados dentro do agregado, garantindo que todas as regras de consistência e integridade sejam aplicadas.

### 2. **Código: Manipulação de Convite em um Evento**

#### **Código Original**

No código original, a manipulação do convite é feita diretamente por meio de consultas ao banco de dados, buscando o convite específico sem considerar o agregado. Veja o trecho original:

```csharp
var invitation = await _invitationRepository.GetByIdAsync(request.InvitationId, cancellationToken);
```

Aqui, o convite é acessado diretamente pela sua identidade, o que pode violar a integridade do modelo de domínio, especialmente se a lógica de manipulação de convites estiver encapsulada no agregado do evento (`Gathering`).

#### **Mudança no Código**

A mudança realizada implica que agora o convite é acessado através da raiz do agregado, que é o `Gathering`. Veja a alteração no código:

```csharp
var gathering = await _gatheringRepository.GetByIdWithCreatorAsync(request.GatheringId, cancellationToken);
var invitation = gathering.Invitation.FirstOrDefault(i => i.Id == request.InvitationId);
```

Essa abordagem segue o padrão de design de agregados, onde a raiz do agregado (`Gathering`) é responsável por gerenciar todas as entidades e objetos de valor relacionados, como `Invitation` e `Attendee`.

### 3. **Benefícios da Mudança**

- **Consistência do Modelo de Domínio:** Ao acessar o convite através da raiz do agregado, garantimos que todas as regras de negócios e a integridade dos dados são mantidas de acordo com as regras definidas na classe `Gathering`.
  
- **Encapsulamento:** A lógica de aceitação do convite agora está encapsulada na classe `Gathering`, o que ajuda a manter o código limpo e a centralizar a lógica de manipulação de convites e participantes.

- **Simplificação do Código:** A mudança reduz a necessidade de consultar diretamente o repositório de convites e integra a manipulação diretamente na lógica do agregado, facilitando a manutenção e a compreensão do código.

### 4. **Exemplo de Implementação: Classe `Gathering`**

Aqui está uma visão geral da classe `Gathering`, que é a raiz do agregado:

```csharp
public sealed class Gathering : AggregateRoot
{
    private readonly List<Invitation> _invitations = new();
    private readonly List<Attendee> _attendees = new();

    public Member Creator { get; private set; }
    public GatheringType Type { get; private set; }
    public string Name { get; private set; }
    public DateTime ScheduledAtUtc { get; private set; }
    public string? Location { get; private set; }
    public int? MaximumNumberOfAttendees { get; private set; }
    public DateTime? InvitationsExpireAtUtc { get; private set; }
    public int NumberOfAttendees { get; private set; }

    public IReadOnlyCollection<Attendee> Attendees => _attendees;
    public IReadOnlyCollection<Invitation> Invitations => _invitations;

    public Attendee? AcceptInvitation(Invitation invitation)
    {
        // Lógica para aceitar um convite e adicionar um participante
    }
}
```

### 5. **Conclusão**

A mudança para acessar o convite através do agregado `Gathering` é uma prática recomendada em DDD. Ela assegura que todas as operações e regras de negócios associadas ao convite e aos participantes sejam gerenciadas de forma consistente e centralizada, refletindo melhor a lógica do domínio e mantendo a integridade dos dados.
