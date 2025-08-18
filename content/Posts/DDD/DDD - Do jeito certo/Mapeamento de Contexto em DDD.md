---
title: Mapeamento de Contexto | DDD do Jeito Certo
date: 2024-11-08
tags:
  - DDD
  - BoundedContext
  - DotNET
---
O mapeamento de contexto é uma prática crucial para compreender como diferentes partes de um domínio e seus respectivos subdomínios se relacionam no espaço da solução. Isso ajuda a definir as fronteiras entre os contextos delimitados (bounded contexts) e a planejar a arquitetura do software, bem como a organização dos times.

![[ddd.7.png]]

**Entendimento do Domínio e Subdomínios**

Quando se trabalha com um domínio complexo, como o de uma rede varejista que opera tanto online quanto fisicamente, é comum dividir o domínio em subdomínios. No caso de uma rede varejista, os subdomínios poderiam incluir: loja online, loja física e centro de distribuição.

![[ddd.8.png]]

Cada um desses subdomínios corresponde a um contexto delimitado na solução, e é possível que cada um desses contextos tenha um time dedicado para tratá-lo.

![[ddd.9.png]]

**Comunicação e Relacionamento entre Contextos**

A comunicação entre diferentes contextos delimitados é essencial, especialmente quando se lida com conceitos como multicanalidade, onde loja online e loja física precisam trabalhar juntas. Essa comunicação pode gerar acoplamentos, o que aumenta a complexidade e o custo de manutenção do sistema. No entanto, é necessário para garantir que os contextos estejam alinhados em termos de funcionalidades e experiência do usuário.

**Posicionamento Conformista e Camada de Anticorrupção**

Quando um contexto assume uma posição conformista em relação a um fornecedor, ele precisa adaptar-se às mudanças externas, o que pode gerar riscos significativos. Se um serviço de terceiros, como um sistema de cobrança, altera seu modelo de domínio, o impacto pode ser profundo no contexto cliente.

![[ddd.15.png]]

Para mitigar esses riscos, adota-se a **camada de anticorrupção (anti-corruption layer)**. Essa camada atua como um intermediário que traduz e simplifica as interações com o modelo de domínio externo. Assim, quando o serviço externo sofre alterações, apenas a camada de anticorrupção precisa ser ajustada, isolando o restante do sistema de mudanças drásticas. Isso é especialmente útil em contextos onde múltiplos fornecedores podem ser utilizados, cada um com suas próprias representações e tecnologias.

![[ddd.16.png]]

**Estratégias de Relacionamento entre Contextos**

Existem diferentes formas de relacionamento entre contextos delimitados:

- **Parceria**: Contextos trabalham em conjunto, mas com autonomia. Um exemplo seria a loja online e a loja física que precisam alinhar estratégias, mas cada uma mantém suas próprias responsabilidades.

  ![[ddd.11.png]]

- **Cooperação**: Contextos compartilham partes de seus modelos de domínio, como no caso de um "Shared Kernel", onde parte do código é reutilizada por diferentes contextos.

  ![[ddd.13.png]]

- **Cliente-Fornecedor**: Um contexto atua como fornecedor, enquanto o outro consome seus serviços, como no caso do centro de distribuição que fornece informações de estoque para as lojas.

  ![[ddd.14.png]]

**Impacto na Arquitetura e Organização**

A forma como esses contextos se relacionam influencia diretamente a arquitetura do software e a estrutura organizacional da empresa. Times que trabalham em contextos delimitados separados precisam de mecanismos claros de comunicação e gestão para evitar problemas de acoplamento e garantir que o software evolua de forma coesa.

**Considerações Finais**

O mapeamento de contexto é uma ferramenta poderosa no DDD, ajudando a definir limites claros entre os contextos, planejar as interações entre eles e, finalmente, construir uma solução que reflita fielmente o domínio que está sendo modelado. Ao mapear contextos, é importante considerar as relações entre eles e como isso impacta tanto a arquitetura do sistema quanto a organização dos times envolvidos no projeto.

![[ddd.6.png]]

[Mapeamento de Contexto](https://youtu.be/yhlaNZ7c494?si=5VBmzK1bhfkmx24x)

[[Entidades e Objetos de Valor]]
