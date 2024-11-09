---
title: Validation
date: 2024-11-08
tags:
  - DDD
  - Validations
  - DotNET
---
O código apresentado utiliza diferentes estratégias para validar regras de negócio e garantir a consistência do domínio.

#### 1. **Exceções de Domínio (`DomainException`)**

As exceções de domínio são utilizadas para capturar erros que ocorrem quando as regras de negócio são violadas. Essas exceções são específicas do domínio da aplicação e derivam de uma classe base `DomainException`, que herda de `Exception`.

##### **Exemplo de Código:**

```csharp
public abstract class DomainException : Exception
{
    protected DomainException(string message)
        : base(message)
    {
    }
}

public sealed class GatheringInvitationsValidBeforeInHoursIsNullDomainException : DomainException
{
    public GatheringInvitationsValidBeforeInHoursIsNullDomainException(string message) 
        : base(message)
    {
    }
}

public class GatheringMaximumNumberOfAttendeesIsNullDomainException : DomainException
{
    public GatheringMaximumNumberOfAttendeesIsNullDomainException(string message) 
        : base(message)
    {
    }
}

private void CalculateGatheringTypeDetails(
        int? maximumNumberOfAttendees,
        int? invitationsValidBeforeInHours)
{
    switch (Type)
    {
        case GatheringType.WithFixedNumberOfAttendees:
            if (maximumNumberOfAttendees is null)
            {
                throw new GatheringMaximumNumberOfAttendeesIsNullDomainException(
                    $"{nameof(maximumNumberOfAttendees)} can't be null.");
            }

            MaximumNumberOfAttendees = maximumNumberOfAttendees;
            break;

        case GatheringType.WithExpirationForInvitations:
            if (invitationsValidBeforeInHours is null)
            {
                throw new GatheringInvitationsValidBeforeInHoursIsNullDomainException(
                    $"{nameof(invitationsValidBeforeInHours)} can't be null.");
            }

            InvitationsExpireAtUtc =
                ScheduledAtUtc.AddHours(-invitationsValidBeforeInHours.Value);
            break;

        default:
            throw new ArgumentOutOfRangeException(nameof(GatheringType));
    }
}
```

##### **Justificativa:**

Essas exceções são utilizadas para garantir que as regras de negócio críticas sejam cumpridas. Por exemplo, se o tipo de reunião (`GatheringType`) exige um número máximo de participantes (`MaximumNumberOfAttendees`) ou uma validade para os convites (`invitationsValidBeforeInHours`), essas exceções garantem que essas informações estejam presentes. Caso contrário, uma exceção específica é lançada, interrompendo o fluxo de execução e sinalizando um erro grave.

#### 2. **Objeto de Erro (`Error`) e Resultados (`Result`)**

Outra abordagem utilizada no código é a de retornar objetos que encapsulam o resultado de operações, indicando se foram bem-sucedidas ou se falharam, e, em caso de falha, detalhando o motivo do erro. Essa abordagem utiliza as classes `Error` e `Result`.

##### **Exemplo de Código:**

```csharp
namespace Gatherly.Domain.Shared;

public class Error : IEquatable<Error>
{
    public static readonly Error None = new(string.Empty, string.Empty);
    public static readonly Error NullValue = new("Error.NullValue", "The specified result value is null.");

    public Error(string code, string message)
    {
        Code = code;
        Message = message;
    }

    public string Code { get; }

    public string Message { get; }

    public static implicit operator string(Error error) => error.Code;

    public static bool operator ==(Error? a, Error? b)
    {
        if (a is null && b is null)
        {
            return true;
        }

        if (a is null || b is null)
        {
            return false;
        }

        return a.Equals(b);
    }

    public static bool operator !=(Error? a, Error? b) => !(a == b);

    public virtual bool Equals(Error? other)
    {
        if (other is null)
        {
            return false;
        }

        return Code == other.Code && Message == other.Message;
    }

    public override bool Equals(object? obj) => obj is Error error && Equals(error);

    public override int GetHashCode() => HashCode.Combine(Code, Message);

    public override string ToString() => Code;
}
```

```csharp
namespace Gatherly.Domain.Shared;

public class Result
{
    protected internal Result(bool isSuccess, Error error)
    {
        if (isSuccess && error != Error.None)
        {
            throw new InvalidOperationException();
        }

        if (!isSuccess && error == Error.None)
        {
            throw new InvalidOperationException();
        }

        IsSuccess = isSuccess;
        Error = error;
    }

    public bool IsSuccess { get; }

    public bool IsFailure => !IsSuccess;

    public Error Error { get; }

    public static Result Success() => new(true, Error.None);

    public static Result<TValue> Success<TValue>(TValue value) => new(value, true, Error.None);

    public static Result Failure(Error error) => new(false, error);

    public static Result<TValue> Failure<TValue>(Error error) => new(default, false, error);

    public static Result<TValue> Create<TValue>(TValue? value) => value is not null ? Success(value) : Failure<TValue>(Error.NullValue);
}
```

```csharp
namespace Gatherly.Domain.Shared;

public class Result<TValue> : Result
{
    private readonly TValue? _value;

    protected internal Result(TValue? value, bool isSuccess, Error error)
        : base(isSuccess, error) =>
        _value = value;

    public TValue Value => IsSuccess
        ? _value!
        : throw new InvalidOperationException("The value of a failure result can not be accessed.");

    public static implicit operator Result<TValue>(TValue? value) => Create(value);
}
```

##### **Exemplo de Uso:**

```csharp
public Result<Invitation> SendInvitation(Member member)
{
    if (Creator.Id == member.Id)
    {
        return Result.Failure<Invitation>(DomainErrors.Gathering.InvitingCreator);
    }

    if (ScheduledAtUtc < DateTime.UtcNow)
    {
        return Result.Failure<Invitation>(DomainErrors.Gathering.AlreadyPassed);
    }

    var invitation = new Invitation(Guid.NewGuid(), member, this);

    _invitations.Add(invitation);

    return Result.Success(invitation);
}
```

##### **Justificativa:**

Essa abordagem é útil para operações que podem falhar por vários motivos. Em vez de lançar uma exceção imediatamente, um objeto `Result` é retornado, encapsulando o resultado da operação e qualquer erro que tenha ocorrido. Isso permite ao código chamador decidir como lidar com a falha, seja exibindo uma mensagem de erro, registrando a falha, ou tentando uma alternativa.

#### 3. **Comparação das Abordagens**

- **Exceções de Domínio:**
  - **Prós:** 
    - Diretas e fáceis de usar para capturar erros críticos.
    - Interrompem o fluxo de execução imediatamente, forçando o tratamento do erro.
  - **Contras:** 
    - Podem ser consideradas "caras" em termos de desempenho, especialmente em operações onde falhas são comuns e esperadas.
    - Menos flexíveis para cenários onde múltiplas condições podem falhar.

- **Objeto de Erro e Resultados:**
  - **Prós:** 
    - Flexíveis e oferecem controle detalhado sobre o fluxo de execução após uma falha.
    - Permitem feedback mais detalhado e reutilizável sobre erros.
    - Podem ser usados em operações onde falhas são esperadas e parte do fluxo normal (e.g., validação de entrada).
  - **Contras:** 
    - Requerem mais código boilerplate para implementação e uso.
    - Podem ser menos óbvios para iniciantes, exigindo familiaridade com o padrão.

#### 4. **Conclusão**

A escolha entre utilizar exceções de domínio ou objetos de erro e resultados depende do contexto específico da aplicação e dos requisitos de tratamento de erro. As exceções de domínio são ideais para capturar erros graves e que violam regras fundamentais do negócio, enquanto os objetos de erro e resultados oferecem uma abordagem mais refinada e controlada para cenários onde o erro é parte do fluxo normal da aplicação. Ambas as estratégias podem coexistir dentro do mesmo projeto, atendendo a diferentes necessidades de validação e tratamento de erro.