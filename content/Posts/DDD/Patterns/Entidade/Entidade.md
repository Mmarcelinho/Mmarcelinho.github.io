---
title: Entidade
date: 2024-11-08
tags:
  - DDD
  - Entities
  - DotNET
---
#### 1. **Contexto de Design Orientado a Domínios (DDD)**

No contexto de DDD (Domain-Driven Design), uma *Entity* é um objeto que possui uma identidade distinta, que persiste ao longo do tempo e entre diferentes instâncias. A identidade é o que diferencia uma *Entity* de outra, mesmo que outras propriedades sejam idênticas. Essa identidade é frequentemente representada por um identificador único (neste caso, um `Guid`).

#### 2. **Construção da Entidade**

A classe `Entity` é abstrata porque serve como uma base para todas as entidades do domínio, garantindo que todas as subclasses compartilhem um padrão comum para identificar e comparar entidades.

- **Id:** O identificador `Id` é gerado automaticamente no construtor com `Guid.NewGuid()`, garantindo que cada instância de uma entidade tenha um identificador único desde o momento de sua criação. A propriedade é `private init` para evitar que ela seja modificada após a criação, assegurando a imutabilidade da identidade da entidade.

#### 3. **Sobrescrita dos Métodos de Igualdade**

No contexto de DDD, a igualdade de entidades não é baseada na comparação de suas propriedades, mas sim em seus identificadores únicos. Portanto, os métodos `Equals`, `GetHashCode`, e os operadores `==` e `!=` são sobrescritos para garantir que duas instâncias de `Entity` sejam consideradas iguais se e somente se seus `Id`s forem iguais.

- **Método `Equals`:** 
    - O método `Equals(Entity? other)` verifica se a entidade passada como parâmetro é do mesmo tipo (`GetType()`) e, em seguida, compara os identificadores (`Id`). Isso garante que duas entidades são iguais apenas se são do mesmo tipo e têm o mesmo identificador.
    - O método `Equals(object obj)` realiza uma verificação semelhante, mas lida com o tipo `object`, garantindo que o comportamento de igualdade esteja consistente quando objetos são comparados de forma genérica.

- **Método `GetHashCode`:**
    - O método `GetHashCode()` é sobrescrito para fornecer um código hash baseado no `Id` da entidade. Multiplicar o hash do `Id` por um número primo (como `41`) ajuda a distribuir uniformemente os hashes, reduzindo colisões em estruturas de dados que usam hashing, como dicionários e conjuntos.

- **Operadores `==` e `!=`:**
    - Os operadores são sobrecarregados para usar o método `Equals` nas comparações, proporcionando uma sintaxe mais intuitiva ao comparar entidades.

### Conclusão

A entidade base é construída dessa forma para garantir que a identidade das entidades seja imutável e única, e para que a comparação de entidades seja baseada em sua identidade única (`Id`). A sobrescrita dos métodos de igualdade é fundamental para assegurar que o comportamento de comparação de entidades seja consistente e correto dentro de um sistema orientado a domínios, onde a identidade é central.

# Entidade em Design Orientado a Domínios (DDD)

## 1. O Que é uma Entidade?

No Design Orientado a Domínios (DDD), uma *Entidade* é um objeto que possui uma identidade única que se mantém ao longo do tempo, mesmo que suas características possam mudar. Essa identidade é frequentemente representada por um identificador exclusivo, como um `Guid`.

## 2. Como Construir uma Entidade

A classe `Entity` é definida como **abstrata** para servir como base para todas as entidades no seu domínio. Isso garante que todas as subclasses sigam um padrão comum na identificação e comparação de entidades.

- **Identificador (`Id`):** 
  - O `Id` é criado automaticamente quando a entidade é instanciada, utilizando `Guid.NewGuid()`. Isso assegura que cada entidade tenha um identificador único desde sua criação.
  - A propriedade do `Id` é marcada como `private init`, o que significa que ela não pode ser alterada após sua criação, garantindo assim a imutabilidade da identidade da entidade.

## 3. Comparando Ent