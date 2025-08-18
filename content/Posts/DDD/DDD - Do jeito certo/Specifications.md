---
title: Specifications | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Specifications
  - DotNET
---
### Padrão Specification

**Definição:**  
O padrão Specification visa mover regras de negócio que podem mudar ao longo do tempo para fora da entidade ou objeto de valor, colocando-as em uma classe específica para essa regra.

**Exemplo:**  
No contexto de uma lei de aposentadoria, cria-se uma classe para cada edição da lei, com um método booleano que recebe a entidade (por exemplo, funcionário) e retorna verdadeiro se ela atende à regra ou falso se não atende.
### Vantagens do Padrão Specification

- **Facilidade de Adaptação:**  
  Com uma nova edição de uma lei, basta criar uma nova implementação da especificação, sem precisar alterar a existente.

- **Testabilidade:**  
  Especificações antigas continuam válidas, e novas podem ser testadas separadamente.

- **Verificação Cruzada:**  
  Permite comparar resultados de diferentes versões de uma regra, verificando, por exemplo, se um funcionário elegível para aposentadoria sob uma lei ainda é elegível sob outra.

- **Expressividade:**  
  Traz para o modelo de domínio expressões da linguagem ubíqua, facilitando a comunicação com especialistas de domínio.
### Abordagem com Interfaces Abstratas

**Abstração de Specification:**  
Utiliza uma interface genérica com um método booleano que valida se uma instância atende à especificação.

**Combinação de Especificações:**  
Possibilita a criação de implementações genéricas combináveis, usando operadores como AND, OR, NOT.
### Injeção de Dependências

Com uma nova especificação, basta alterar o tipo no contêiner de injeção de dependências para atualizar todo o sistema.

### Uso Conjunto com Factor

**Factory para Escolha de Especificações:**  
Utilizada em cenários onde múltiplas especificações são vigentes simultaneamente, a Factory toma decisões sobre qual especificação deve ser usada com base em parâmetros.

### Outros Usos do Padrão Specification

Além de regras de negócio que mudam ao longo do tempo, o padrão Specification pode ser aplicado em algoritmos de recomendação, seleção de produtos, entre outros. Ele é especialmente eficaz em normas e especificações que mudam periodicamente em grandes empresas.

[Specifications](https://youtu.be/dCdOG_RZpxU?si=-HbSX7Tgs1qz5Xql)

[[Repositórios]]