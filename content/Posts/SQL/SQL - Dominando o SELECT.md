---
title: SQL - Dominando o SELECT
date: 2024-04-02
tags:
  - SQL
  - MYSQL
  - Comandos
  - Comandos-SQL
---
### SELECT

O comando `SELECT` é utilizado para visualizar dados na tabela, tem uma variedade de opções para filtrar esses dados conforme a necessidade.

## Código

#### SELECT AS

```sql
SELECT descricao AS produtos FROM tb_produto;
```

Nesse primeiro comando estamos selecionando só a `descricao` de cada produto utilizando `AS` para criar uma coluna `produtos` para apresentar esses dados.

- Resultado

![[select.1.png]]

#### SELECT  CONCAT

```sql
SELECT concat('Produto: ', descricao) FROM tb_produto;
```

Com esse comando estamos fazendo a junção da string `'Produto: ` com a `descricao` da tabela produto

- Resultado

![[select.2.png]]

#### SELECT DISTINCT

```sql
SELECT distinct endereco FROM tb_fornecedor;
```

O comando `distinct` é usado para remover duplicatas dos resultados de uma consulta. Útil para obter uma lista de valores únicos para uma determinada coluna.

#### SELECT com operações aritméticas

```sql
SELECT valor+taxa from tb_produto;
```

No SQL é possível realizar operações aritméticas básicas diretamente nas suas consultas sem a necessidade de modificar a estrutura da tabela ou dos dados para suportar essas operações. Elas são realizadas "em tempo de execução" pela consulta SQL, o que significa que os dados originais permanecem inalterados.

- Resultado

![[select.3.png]]

#### SELECT LIMIT

```sql
SELECT * FROM tb_produto LIMIT 2;
```

O `LIMIT` cria uma limitação de resultados retornando só a quantidade que é específicada.

- Resultado

![[select.4.png]]

#### SELECT FORMAT

```sql
SELECT FORMAT((((1000+1000)*2)/valor),2) FROM tb_produto;
```

Esta função é utilizada para formatar o número do resultado da operaçãoo aritmética. O primeiro argumento é o número a ser formatado, e o segundo argumento (`2`) indica o número de casas decimais.

Resumindo, essa consulta calcula um valor (4000 dividido pelo `valor` de cada produto) e o formata para ter duas casas decimais, retornando este valor formatado para cada linha da tabela.

- Resultado

![[select.5.png]]

#### SELECT WHERE

```sql
SELECT fornecedor, endereco FROM tb_fornecedor WHERE fornecedor = 'Elegância Acessórios';
```

O comando `WHERE` cria um filtro de acordo com determinada condição, nesse exemplo acima, será retornado o `fornecedor` e o `endereco`  onde `fornecedor = 'Elegância Acessórios'` 

- Resultado

![[select.6.png]]


- Selecionando `descricao` e `valor` de produto onde o `valor` é igual `1200.00`.

```sql
SELECT descricao,valor FROM tb_produto WHERE VALOR=1200.00;
```

- Selecionando `descricao` do produto onde o `id` é igual a 5.

```sql
SELECT descricao from tb_produto WHERE id=5;
```

-  Selecionando `descricao` de produtos que possuem `valor` menor que `1000`.

```sql
SELECT descricao FROM tb_produto WHERE valor<1000;
```

- Selecionando `descricao` de produtos que possuem `valor` entre `10` e `3000`.

```sql
SELECT descricao from tb_produto WHERE valor between 10 and 3000;
```

-  Selecionando `descricao` e `valor`de produtos que correspondem a qualquer um dos valores listados.

```sql
SELECT descricao,valor FROM tb_produto WHERE valor in(50,1200);
```

-  Selecionando produtos que possuem a `descricao` que correspondem a qualquer um dos valores listados.

```sql
SELECT descricao FROM tb_produto WHERE descricao in ('Notebook Gamer','Ração para Cães');
```

-  Selecionando `descricao` de produto que o valor seja nulo.

```sql
SELECT descricao FROM tb_produto WHERE valor is null;
```

- Selecionando `descricao` de produtos que começam com a letra N.

```sql
SELECT descricao FROM tb_produto WHERE descricao like 'N%';
```

- Selecionando `descricao` de produtos que terminam com 'es'.

```sql
SELECT descricao FROM tb_produto WHERE descricao like '%es';
```

- Selecionando descrição de produtos que **começam com qualquer caractere seguido por 'a'**, e podem ter qualquer número de caracteres depois do 'a'.

```sql
SELECT descricao FROM tb_produto WHERE descricao like '_a%';
```

## Conclusão

Os variados exemplos discutidos acima não só ilustram a versatilidade e potência do SQL na manipulação e consulta de dados em bancos de dados relacionais, mas também destacam a importância crucial de dominar essa linguagem.

[[SQL - Dominando o SELECT pt.2]]
