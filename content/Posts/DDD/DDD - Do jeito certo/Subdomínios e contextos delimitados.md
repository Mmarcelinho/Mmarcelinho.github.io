---
title: Subdomínios e contextos delimitados | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - BoundedContext
  - DotNET
---
### Introdução ao DDD

**Objetivo do DDD:**  
O Domain-Driven Design (DDD) não é uma ferramenta para resolver a complexidade técnica ou problemas de legado de software. Ele também não garante sistemas escaláveis, seguros ou mais fáceis de gerenciar. O verdadeiro objetivo do DDD é lidar com a complexidade do **domínio** (o negócio), ou seja, a complexidade essencial do que o sistema está tentando resolver.
### **Linguagem Onipresente**

**Definição:**  
A **linguagem onipresente** é a linguagem comum usada por especialistas do domínio (profissionais que entendem profundamente o negócio) e a equipe técnica. O objetivo é garantir que todos falem a mesma língua, utilizando termos que têm o mesmo significado tanto no negócio quanto no sistema. Isso ajuda a trazer as operações do negócio para dentro do sistema de forma clara e consistente.
### Subdomínios e Contextos Delimitados

**Subdomínios:**  
Dentro de um domínio complexo, existem diferentes áreas chamadas de **subdomínios**. Cada subdomínio é uma parte do domínio principal e pode ser visto como uma área específica do negócio. No contexto de DDD, subdomínios pertencem ao **espaço do problema** – a parte do processo onde você identifica o que precisa ser resolvido.

**Contextos Delimitados:**  
Os **contextos delimitados** são a resposta do espaço da solução para os subdomínios do espaço do problema. Dentro do software, os contextos delimitados representam diferentes partes do domínio, garantindo que cada subdomínio seja tratado de forma adequada e isolada.

![[ddd.4.png]]

**Exemplo Explicativo:**  
Imagine que você esteja desenvolvendo um software para um hospital. Aqui, o domínio é o hospital como um todo. Dentro desse domínio, há vários subdomínios, como:

- **Atendimento Médico**
- **Administração Financeira**
- **Gestão de Pacientes**

Cada um desses subdomínios representa uma parte distinta do hospital. O Atendimento Médico é tratado por especialistas da saúde, a Administração Financeira por contadores, e assim por diante. No espaço da solução, esses subdomínios serão representados por diferentes contextos delimitados, cada um com sua própria lógica e linguagem específica.
### Tipos de Subdomínios

**Core Domain (Domínio Principal):**  
Essa é a parte central do negócio que diferencia a empresa de outras. No caso do hospital, o Core Domain poderia ser a área de Atendimento Médico, que é o que faz o hospital existir.

**Supporting Domain (Domínio de Suporte):**  
São os subdomínios que suportam o Core Domain, como o setor de Gestão de Pacientes, que ajuda o Atendimento Médico a funcionar.

**Generic Domain (Domínio Genérico):**  
Esses são subdomínios que executam operações importantes, mas que não estão diretamente ligadas ao Core Domain. Por exemplo, a Administração Financeira do hospital pode ser considerada um Domínio Genérico, pois é algo necessário, mas não é o que diferencia o hospital de outras organizações.

![[ddd.5.png]]
### Aplicações Práticas

**Visão Estratégica vs. Tática:**  
Ao tratar do espaço do problema e da solução, você está agindo de forma estratégica (entendendo o negócio, identificando subdomínios). Quando você cria um contexto delimitado no software para representar esses subdomínios, você está atuando de forma tática, implementando as soluções no código.

**Customização de Softwares:**  
Geralmente, as partes do sistema que lidam com o Core Domain precisam de mais customização, pois representam o que torna o negócio único. Já os subdomínios genéricos podem ser atendidos por softwares prontos (de prateleira), pois os processos são comuns a várias organizações.
### Conclusão

Compreender o DDD significa entender a separação entre o espaço do problema (identificação de subdomínios) e o espaço da solução (implementação desses subdomínios no software). O domínio principal precisa ser tratado com especial atenção, utilizando contextos delimitados para garantir que cada parte do negócio seja representada de maneira clara e eficiente no sistema.

[Subdomínios e contextos delimitados](https://youtu.be/9hlRHZ4Pfyo?si=-8AP_5RE5y6OxmTL)

[[Mapeamento de Contexto em DDD]]