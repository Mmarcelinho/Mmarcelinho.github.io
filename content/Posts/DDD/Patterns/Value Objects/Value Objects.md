---
title: Value Objects
date: 2024-11-08
tags:
  - DDD
  - ValueObjects
  - DotNET
---
#### 1. **Contexto de Design Orientado a Domínios (DDD)**

Em DDD (Domain-Driven Design), um *Value Object* (Objeto de Valor) é um objeto que não possui identidade própria. Ele é definido exclusivamente pelos seus atributos. Diferente de uma entidade, um *Value Object* é considerado igual a outro se todos os seus atributos forem iguais. Esses objetos são imutáveis, ou seja, uma vez criados, seus estados não podem ser alterados.

#### 2. **Construção do Objeto de Valor**

A classe `ValueObject` é uma classe base abstrata que fornece uma implementação padrão para objetos de valor no domínio.

- **GetAtomicValues:** Este método abstrato deve ser implementado por subclasses para retornar uma coleção dos valores atômicos que compõem o objeto de valor. Essa implementação é essencial para a correta comparação de igualdade entre dois objetos de valor, permitindo a comparação sequencial dos valores internos.

#### 3. **Sobrescrita dos Métodos de Igualdade**

A comparação de objetos de valor baseia-se na igualdade dos seus valores internos, em vez de uma identidade única. Portanto, os métodos `Equals`, `GetHashCode`, e `ValuesAreEqual` são implementados para garantir que dois objetos de valor sejam considerados iguais se todos os seus valores atômicos forem iguais.

- **Método `Equals(ValueObject? other)`:**
    - Este método compara o objeto atual com outro objeto de valor, verificando se os valores atômicos de ambos são iguais. Se forem, os objetos são considerados iguais.

- **Método `Equals(object obj)`:**
    - Este método compara o objeto atual com um objeto genérico. Ele verifica se o objeto é um `ValueObject` e utiliza `ValuesAreEqual` para comparar os valores atômicos.

- **Método `GetHashCode`:**
    - Este método utiliza `GetAtomicValues` para combinar os valores atômicos em um código hash único, representando o objeto de valor. A combinação dos valores usando `HashCode.Combine` ajuda a garantir uma distribuição uniforme dos códigos hash, reduzindo colisões.

- **Método `ValuesAreEqual`:**
    - Este método privado compara os valores atômicos de dois objetos de valor, verificando se todos os valores são iguais, na mesma ordem. Ele usa `SequenceEqual`, que compara duas coleções sequencialmente.

#### 4. **Exemplo de Implementação: `FirstName`**

A classe `FirstName` é um exemplo concreto de um *Value Object*. Ela encapsula o valor de um primeiro nome, garantindo que ele nunca seja nulo, vazio, ou ultrapasse um comprimento máximo.

- **Imutabilidade:** A propriedade `Value` é somente leitura (`get`), tornando o objeto imutável após sua criação.
- **Validação:** O método `Create` realiza validações para garantir que o valor de `FirstName` seja válido antes de criar a instância.

```csharp
public class FirstName : ValueObject
{
    public const int MaxLength = 50;

    private FirstName(string value)
    {
        if(value.Length > MaxLength)
            throw new ArgumentException();

        Value = value;
    }
    public string Value { get; }

    public static Result<FirstName> Create(string firstName)
    {
        if(string.IsNullOrWhiteSpace(firstName))
            return Result.Failure<FirstName>(new Error("FirstName.Empty", "First name is empty."));

        if(firstName.Length > MaxLength)
            return Result.Failure<FirstName>(new Error("FirstName.TooLong", "First name is too long."));

        return new FirstName(firstName);
    }

    public override IEnumerable<object> GetAtomicValues()
    {
       yield return Value;
    }
}
```

### 5. **Comparação com Records**

A introdução dos *records* no C# oferece uma maneira mais simples e direta de implementar objetos de valor. Um *record* é um tipo de referência que fornece implementações automáticas para igualdade de valor, clonagem e mutabilidade controlada.

#### **Vantagens dos Records:**

- **Menos Código:** Usar *records* elimina a necessidade de sobrescrever manualmente os métodos `Equals` e `GetHashCode`, reduzindo significativamente o código boilerplate.
  
- **Imutabilidade:** *Records* suportam a imutabilidade nativamente, tornando-os ideais para objetos de valor.

- **Sintaxe Expressiva:** A sintaxe de *records* é mais concisa e legível, permitindo a definição de objetos de valor com menos esforço.

#### **Exemplo de Implementação com Records:**

```csharp
public record FirstName(string Value)
{
    public const int MaxLength = 50;

    public static Result<FirstName> Create(string firstName)
    {
        if(string.IsNullOrWhiteSpace(firstName))
            return Result.Failure<FirstName>(new Error("FirstName.Empty", "First name is empty."));

        if(firstName.Length > MaxLength)
            return Result.Failure<FirstName>(new Error("FirstName.TooLong", "First name is too long."));

        return new FirstName(firstName);
    }
}
```

#### **Considerações Finais:**

- **Uso de `ValueObject`:** A implementação manual com a classe `ValueObject` permite maior controle sobre a lógica de comparação e personalização. Ela é útil em cenários onde a lógica de igualdade precisa ser ajustada ou estendida.

- **Uso de Records:** *Records* são mais diretos e simples para a maioria dos casos de uso de objetos de valor, especialmente quando se busca concisão e clareza. Para a maioria dos cenários em que a imutabilidade e a comparação de valores são as principais preocupações, *records* são a escolha ideal.

Em suma, ambos os métodos têm seus méritos, e a escolha entre eles pode depender do nível de controle necessário e da preferência por uma implementação mais explícita ou mais concisa.