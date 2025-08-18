---
title: Serviços de Domínio | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Services
  - Domain
  - DotNET
---
### Serviços de Domínio (Domain Services) em DDD

**Definição e Papel dos Serviços de Domínio**
- Serviços de domínio são fundamentais quando a lógica de negócio não se encaixa adequadamente dentro de uma única entidade ou objeto de valor.
- Eles são essenciais em operações que envolvem múltiplos agregados ou quando é necessário coordenar ações além dos limites transacionais de um agregado único.

**Indicação de Problemas na Modelagem**
- A presença excessiva de serviços de domínio pode sugerir um modelo de entidades e objetos de valor fraco ou anêmico.
- Isso pode indicar a necessidade de revisar a modelagem do domínio para garantir que as responsabilidades estejam bem distribuídas entre as entidades e os objetos de valor.

**Exemplo Prático: Reajuste Global de Preços**
- Imagine um sistema de e-commerce onde é preciso aplicar um reajuste global em todos os produtos de uma categoria específica.
- Como cada produto é um agregado independente, a operação se torna complexa e não pode ser realizada dentro de um único agregado.
- Um serviço de domínio poderia coordenar essa operação, garantindo que o reajuste seja aplicado de forma atômica e consistente em todos os produtos, respeitando as regras de negócio.

**Diferença entre Serviços de Domínio e Serviços de Aplicação**
- **Serviços de Aplicação**: Orquestram a interação entre a interface do usuário e o domínio, mas não contêm lógica de negócio.
- **Serviços de Domínio**: Encapsulam e centralizam a lógica de negócio complexa, especialmente quando envolve múltiplos agregados ou não se encaixa em uma única entidade.

**Aproximação da Infraestrutura**
- Serviços de domínio podem se aproximar da infraestrutura para otimizar a performance.
- Por exemplo, ao invés de carregar todos os produtos na memória para aplicar o reajuste de preços, um serviço de domínio pode executar a operação diretamente no banco de dados.
- Essa abordagem garante eficiência e consistência, mesmo que a implementação seja separada em uma camada de infraestrutura.

**Características dos Serviços de Domínio**
- **Stateless**: Serviços de domínio não mantêm estado relevante para o domínio, o que facilita a reutilização e manutenção.
- **Ideal para Operações Complexas**: Eles são adequados para operações que requerem computação intensiva, como a consolidação de vendas de uma categoria de produtos em um período específico.
- Essas operações geralmente envolvem múltiplos agregados e podem se beneficiar de implementações otimizadas na infraestrutura.

**Foco na Complexidade do Domínio**
- No DDD, o foco é na complexidade do domínio, não na pureza técnica da solução.
- Misturar infraestrutura na implementação de serviços de domínio pode ser aceitável se melhorar a eficiência e a performance.
- Essa flexibilidade permite que o modelo de domínio seja poderoso e eficaz, mesmo que a implementação técnica não seja completamente "pura" ou isolada.

**Resumo da Diferença entre Serviços de Domínio e de Aplicação**
- Se um serviço apenas interage com repositórios e entidades sem envolver lógica de negócio complexa, ele é considerado um serviço de aplicação.
- Serviços de domínio, por outro lado, encapsulam e centralizam a lógica de negócio, assegurando que as regras do domínio sejam aplicadas corretamente e consistentemente em todo o sistema.

[Serviços de Domínio](https://youtu.be/bpETAJ_8y6Y?si=wKxm6qSKrsOMmWT_)

[[Factories]]

