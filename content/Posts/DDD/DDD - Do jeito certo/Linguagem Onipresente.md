---
title: Linguagem Onipresente | DDD do Jeito Certo
date: 2024-11-08
tags:
  - DDD
  - DotNET
---
## Comunicação Eficaz entre Especialistas de Domínio e Desenvolvedores

Em projetos de software, é essencial que os desenvolvedores compreendam profundamente o negócio que estão tentando automatizar. Isso não pode ser alcançado através de uma abordagem onde "um fala e outro anota". A prática de simplesmente ouvir o especialista de domínio descrever os processos enquanto a equipe técnica apenas toma notas, ou o contrário, onde os técnicos tentam adaptar o sistema às necessidades do negócio sem uma troca real de ideias, não é eficaz.

### A Importância da Linguagem Onipresente

Para que essa comunicação seja eficaz, é necessário que ambas as partes - os especialistas de domínio e os desenvolvedores - falem a mesma língua. Este conceito é conhecido como "linguagem onipresente" no contexto de Domain-Driven Design (DDD). A linguagem onipresente surge da colaboração contínua entre a equipe técnica e os especialistas de domínio, onde todos utilizam os mesmos termos e conceitos para descrever o domínio e as operações envolvidas. Isso garante que todos compreendam as mesmas coisas da mesma maneira, reduzindo o risco de mal-entendidos e erros de implementação.

### Evitando a Anemia de Domínio

Um problema comum em sistemas desenvolvidos sem essa colaboração é a chamada "anemia de domínio". Isso ocorre quando as classes do sistema são projetadas apenas para refletir dados, sem capturar a complexidade e as regras do domínio. Um indicativo claro de uma classe anêmica é a ausência de testes significativos para ela. Classes anêmicas geralmente não possuem comportamentos complexos para serem testados, o que sugere que a lógica de negócio não está sendo adequadamente representada no código.

### Um Exemplo Prático

Considere o exemplo de uma classe de funcionário em um sistema de recursos humanos. Se a classe tiver apenas propriedades simples, como "nome" e "salário", e métodos como "alterar salário" sem qualquer lógica adicional, ela provavelmente é anêmica. A lógica de negócios relacionada ao aumento de salário, por exemplo, pode ser muito mais complexa do que simplesmente alterar um valor. Pode envolver regras diferentes dependendo se o aumento é devido a uma promoção, um acordo coletivo, ou outros fatores.

Neste caso, a colaboração com um especialista de domínio revelaria nuances como "movimentação salarial", que pode ser compulsória (como em um dissídio) ou espontânea (como em uma promoção). Cada tipo de movimentação tem suas próprias regras e critérios, que devem ser refletidos no código e testados adequadamente.

### Conclusão

Trabalhar próximo aos especialistas de domínio permite que os desenvolvedores criem classes que encapsulam verdadeiramente a lógica do domínio, evitando a anemia de domínio e garantindo que o sistema realmente atenda às necessidades do negócio. A linguagem onipresente é o alicerce dessa colaboração, garantindo que todos os envolvidos no projeto compartilhem a mesma visão e compreensão do domínio.

[Linguagem Onipresente](https://youtu.be/HnvmpyUAITs?si=FUZdSWuncclVfO0z)

[[Subdomínios e contextos delimitados]]