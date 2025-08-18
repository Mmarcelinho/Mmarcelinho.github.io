---
title: Entidades e Objetos de Valor | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Entities
  - ValueObjects
  - DotNET
---
## Entidades (Entities)

- **Entidades:** Representam objetos do mundo real ou conceitos do domínio que possuem uma identidade distinta e são identificados por um **ID único** (Identity).
  - **Comparação por ID:** Duas instâncias de uma entidade são consideradas a mesma entidade se possuem o mesmo ID, mesmo que outras propriedades sejam diferentes.
  - **Mutabilidade:** Entidades geralmente são mutáveis, ou seja, suas propriedades podem mudar ao longo do tempo. Por exemplo, um funcionário pode mudar de endereço, mas continua sendo o mesmo funcionário.
  - **Persistência:** Em bancos de dados relacionais, as entidades geralmente se mapeiam para tabelas. Contudo, nem sempre essa correspondência é direta ou necessária.

- **Exemplo:** Se um funcionário tem duas instâncias com IDs diferentes, mesmo que todas as propriedades sejam iguais, são considerados funcionários diferentes.

## Objetos de Valor (Value Objects)

- **Objetos de Valor:** São descrições ou partes de uma entidade e não têm identidade própria. Eles são comparados com base em todas as suas propriedades.
  - **Imutabilidade:** Geralmente, objetos de valor são imutáveis. Se alguma propriedade muda, um novo objeto de valor é criado. Isso ocorre porque mudar qualquer propriedade cria uma nova instância, diferentemente das entidades.
  - **Comparação por Propriedades:** A comparação de objetos de valor é feita propriedade por propriedade, não por um ID.
  - **Exemplo:** O endereço de um funcionário é um objeto de valor. Se o número da casa muda, o endereço passa a ser um novo objeto de valor.

- **Implementação em C#:**
  - **Structs:** Em C#, objetos de valor são frequentemente implementados como structs, pois a comparação de structs é feita propriedade por propriedade, simplificando a implementação.
  - **Classes:** Se implementados como classes, deve-se definir explicitamente o método de comparação, comparando todas as propriedades.

## Entidades e Objetos de Valor no Contexto do Bounded Context

- **Entidades como Cola entre Bounded Contexts:** Entidades podem ser representadas de forma diferente em diferentes bounded contexts, mas compartilham o mesmo ID, que serve como uma "cola" que conecta essas representações diferentes.
  - **Exemplo:** Um cliente pode ter diferentes representações no contexto de vendas e no contexto de suporte, mas o ID do cliente será o mesmo em ambos os contextos.

![[ddd.17.png]]

## Separação entre Modelos de Domínio e Persistência

- **Problemas com Modelagem Relacional:** A estrutura das bases de dados relacionais pode levar a representações de objetos de valor como entidades, adicionando IDs a objetos que não deveriam tê-los, o que pode causar confusão entre o modelo de domínio e o modelo de persistência.
  - **Importância da Separação:** Manter uma clara separação entre o modelo de domínio (que reflete a lógica de negócio) e o modelo de persistência (que lida com a forma de armazenamento) é crucial para evitar confusões e garantir a integridade do design.

## Considerações Finais

- **Adotar DDD:** A adoção correta de DDD envolve entender profundamente o domínio e aplicar esses conceitos para resolver a complexidade inerente ao negócio, mantendo o design do software alinhado com as necessidades do domínio.

[Entidades e Objeetos de Valor](https://youtu.be/6Pjod34OCnE?si=ip9MY_UCn3zEQ9ur)

[[Agregados]]

