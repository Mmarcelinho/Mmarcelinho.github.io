---
title: Atacando a Complexidade | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - DotNET
---
![[ddd.3.png]]

O Domain-Driven Design (DDD) é uma abordagem que nasceu em 2003 com o objetivo de enfrentar a complexidade no desenvolvimento de software, concentrando-se naquilo que realmente importa: a complexidade do domínio. Esta complexidade é inerente ao problema de negócio que o software visa resolver e é o que motiva a contratação de desenvolvedores para criar uma solução.

Ao abordar a complexidade no desenvolvimento de software, é importante diferenciar três tipos principais:

1. **Complexidade da Solução Técnica:** Refere-se às decisões técnicas que tomamos ao escolher tecnologias e arquiteturas para desenvolver software. Exemplos incluem optar por uma arquitetura de micro serviços ou monolítica, escolher frameworks como Angular ou React, ou decidir entre uma forte separação entre front-end e back-end. Essas escolhas geram complexidades que são chamadas de acidentais porque são resultado das nossas decisões.

2. **Complexidade do Legado:** Esta complexidade surge da necessidade de manter e evoluir sistemas antigos. Embora também seja considerada uma complexidade acidental, diferentemente da complexidade técnica, ela não é tão fácil de evitar. Manter suporte a sistemas legados é uma escolha consciente, muitas vezes necessária devido a dívidas técnicas acumuladas ao longo do tempo.

3. **Complexidade do Domínio:** Esta é a única complexidade essencial, pois está diretamente relacionada ao problema de negócio que estamos tentando resolver. A complexidade do domínio trata dos processos, regras e desafios específicos do negócio. É, em última análise, o motivo pelo qual alguém decide investir no desenvolvimento de software.

O objetivo central do DDD é abordar a complexidade do domínio de maneira eficaz, deixando as complexidades acidentais (técnicas e de legado) em segundo plano. No entanto, na prática, os profissionais de tecnologia frequentemente se concentram mais nas complexidades técnicas e de legado porque são mais confortáveis em lidar com esses aspectos. Isso pode levar à implementação de soluções que, embora tecnicamente sofisticadas, não agregam valor real ao negócio porque não refletem adequadamente a complexidade do domínio.

Para implementar o DDD de maneira eficaz, é crucial que a comunicação entre os especialistas de domínio (aqueles que entendem profundamente o negócio) e a equipe técnica seja clara e eficaz. Essa comunicação é facilitada pelo desenvolvimento de uma **linguagem onipresente**, um vocabulário comum que todos entendem e que pode ser diretamente refletido no código.

Infelizmente, é comum que essa parte vital do DDD seja ignorada. Muitas equipes afirmam usar DDD, mas acabam implementando entidades e serviços anêmicos — classes que não contêm a lógica de negócio necessária para refletir o domínio de forma significativa. Isso é um sinal de que o DDD não está sendo aplicado corretamente.

O DDD é mais eficaz em cenários onde o domínio é complexo. Em situações onde o problema de negócio é simples, o DDD pode não agregar tanto valor, e outras abordagens podem ser mais adequadas. 

Portanto, implementar DDD do jeito certo significa focar na complexidade do domínio, entendendo profundamente o problema de negócio e construindo soluções que reflitam essa complexidade no código. É sobre atacar o cerne do problema e evitar o aumento desnecessário de complexidade técnica ou de legado. A partir daí, o uso de padrões de DDD, como entidades, objetos de valor, repositórios e agregados, deve servir para expressar de forma clara e eficaz a lógica de domínio, sempre alinhada com o entendimento compartilhado entre a equipe técnica e os especialistas de negócio.

Essa abordagem não só facilita a criação de software que realmente atende às necessidades do negócio, mas também promove uma colaboração mais produtiva entre todas as partes envolvidas, garantindo que o foco permaneça naquilo que é essencial.

[Atacando a Complexidade](https://youtu.be/2X9Q97u4tUg?si=8wDb172-MenI3bQz)

[[Linguagem Onipresente]]
