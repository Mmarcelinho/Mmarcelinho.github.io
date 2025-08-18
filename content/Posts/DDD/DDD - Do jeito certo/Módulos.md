---
title: Módulos | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Modules
  - DotNET
---
### Organização do Código em Grandes Sistemas

- **Problema Comum**: Em sistemas grandes, é difícil organizar o código de forma que seja fácil encontrar entidades, repositórios ou serviços de domínio.
- **Solução em DDD**: A organização é feita por meio de **módulos**, que são blocos de código coesos, com pouca ou nenhuma dependência externa. Referências entre módulos podem ocorrer, mas **referências cíclicas** devem ser evitadas.

### Módulos: Estrutura Hierárquica no Código

- A estrutura de módulos segue uma **hierarquia** para facilitar a organização e navegação.
- Exemplo: Talheres em uma cozinha organizados em compartimentos dentro de gavetas. Assim como na cozinha, o código é organizado de forma hierárquica com módulos e submódulos.

### Implementação de Módulos no Código

- Em Java, utiliza-se **pacotes**. Em .NET, usa-se **namespaces**.
- A hierarquia é refletida na separação de **nomes por pontos**, indicando diferentes níveis de especialização e separação lógica do código.
  
### Estrutura Hierárquica dos Módulos

1. **Primeiro Nível**: Deve conter o **nome da empresa** ou **produto**.
   - **Cuidado com produtos**: Se o nome do produto mudar, pode haver problemas no código.
2. **Segundo Nível**: Contextos delimitados devem ser claros e separados.
   - Um **contexto delimitado** deve ter poucas ou nenhuma referência a outros contextos.
   - Cada contexto possui um **especialista de domínio** que utiliza uma **linguagem onipresente** específica do contexto.
3. **Terceiro Nível**: Deve representar a **natureza do código**:
   - `Domain` para o modelo de domínio.
   - `Infrastructure` para infraestrutura.
   - `UI` ou `Application` para interface de usuário ou aplicação.

### Lei de Conway

- A estrutura da empresa é refletida na estrutura do código, e vice-versa. Ao separar módulos por **contextos delimitados**, a organização também tende a ser replicada na divisão de **equipes**.

### Separação de Outros Tipos de Código

- Além do código do modelo de domínio, há código de interface e infraestrutura que também precisa ser organizado.
- A recomendação é seguir a hierarquia: empresa/produto → contextos delimitados → natureza do código.

### Separação de Serviços e Modelo

- Pode ser útil criar uma subdivisão para **serviços** separada do restante do modelo de domínio.
- **Cuidado**: Evite exageros, como criar módulos separados para **repositórios** ou **fábricas**, pois isso cria dependências fortes entre os módulos, o que não é desejável.

### Organizando o Modelo de Domínio

- No nível mais baixo da hierarquia, pode-se separar o código por **agregados**.
- Exemplo: O agregado `Pedido` teria uma estrutura própria contendo o `Pedido` (entidade raiz), itens do pedido, repositório de pedidos, etc. Essa separação facilita a navegação.

### Resumo das Recomendações

- A hierarquia recomendada é:
  1. Nome da empresa/produto.
  2. Contexto delimitado.
  3. Natureza do código (domínio, infraestrutura, etc.).
  4. Separação de serviços e modelo de domínio (se necessário).
  5. Separação de agregados dentro do modelo de domínio.
  
- **Complexidade**: Deve ser balanceada. Organize os módulos de forma eficiente, mas sem adicionar complexidade desnecessária.

[Módulos](https://youtu.be/TVdAyLhR1uw?si=FMt2UDl_lI18TWQF)
