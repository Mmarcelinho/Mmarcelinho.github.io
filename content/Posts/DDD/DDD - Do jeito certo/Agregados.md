---
title: Agregados | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Aggregates
  - DotNET
---
## Conceito de Agregado

Um agregado é um conceito chave no DDD, representando um grupo de entidades que são tratadas como uma unidade única de consistência e transação. Entidades e objetos de valor dentro de um agregado são fortemente relacionados.

## Critérios para Determinar Entidades e Objetos de Valor

- **Entidades** são elementos que possuem identidade própria e cujos dados podem mudar ao longo do tempo. Por exemplo, um **cliente** ou um **pedido** são entidades, pois suas informações podem ser alteradas, e eles são identificados de maneira única através de um ID.
- **Objetos de Valor** são elementos que não possuem identidade própria e são definidos pelas suas propriedades. Um **endereço** seria um exemplo de objeto de valor, pois a comparação é feita com base em todas as suas propriedades, e não por um ID único.

![[ddd.18.png]]


## Relacionamentos em Agregados

- **Composição:** É uma relação forte onde o ciclo de vida do componente é dependente do todo. Por exemplo, um **pedido** é composto por vários **itens de pedido**, e essa relação é de composição. Ambos, o pedido e os itens de pedido, devem ser tratados na mesma transação.

![[ddd.19.png]]


- **Referência:** Relação mais fraca, onde o componente pode existir independentemente do todo. Por exemplo, um **pedido** pode ter uma referência para um **cliente**, mas o cliente não faz parte do pedido em si. Assim, não é necessário manter uma consistência transacional entre o cliente e o pedido.

![[ddd.20.png]]


**Raiz do Agregado:**  
A raiz do agregado é a entidade que serve como ponto de entrada para o agregado. Por exemplo, no agregado **Pedido**, a entidade raiz seria o **pedido** em si, e todos os elementos dentro do agregado, como os itens de pedido, são acessados através dele.

![[ddd.21.png]]


## Persistência e Modelagem de Agregados

- Ao pensar em persistência e modelagem, especialmente em bancos de dados NoSQL, os agregados geralmente correspondem a coleções. Por exemplo, um banco de dados poderia ter uma coleção de **pedidos** e uma coleção de **clientes**, onde os documentos de pedido contêm dados dos pedidos, itens de pedido e o endereço de entrega, e uma referência ao cliente que fez o pedido.
  
## Dicas para Modelagem de Agregados

- **Limites Transacionais:** Agregados devem ser modelados de acordo com seus limites transacionais. Cada agregado deve ser pequeno o suficiente para ser facilmente gerenciável, evitando blocos muito grandes que possam dificultar o transporte e a manipulação.
- **Referências entre Agregados:** Quando houver referência entre agregados diferentes, é importante usar IDs ao invés de propriedades de navegação para manter o isolamento e facilitar a separação conceitual.
- **Consistência Eventual:** Em casos onde a consistência entre agregados precisa ser mantida, considere a aplicação de consistência eventual, onde a sincronização entre dados de agregados diferentes ocorre de maneira assíncrona e gradual.

Esses conceitos são fundamentais para entender como modelar e organizar agregados de forma eficaz no DDD, garantindo a integridade e a consistência do domínio ao longo das operações.

[Agregados](https://youtu.be/1AEOcQWQR2o?si=ioJgJaHWwixutpKJ)

[[Eventos de Domínio]]
