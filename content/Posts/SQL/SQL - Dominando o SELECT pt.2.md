---
title: SQL - Dominando o SELECT Pt.2
date: 2024-04-02
tags:
  - SQL
  - MYSQL
  - Comandos
  - Comandos-SQL
---
## OPERADORES LÓGICOS

Os operadores lógicos no SQL são ferramentas essenciais para a construção de consultas mais complexas e precisas, permitindo a manipulação e o filtro de dados de acordo com condições específicas.

### Código

#### SELECT com AND

```sql
SELECT descricao FROM tb_produto WHERE id=4 AND valor>1000;
```

Esta consulta seleciona a descrição de produtos cujo ID é igual a 4 e cujo valor é maior que 1000, combinando duas condições com o operador lógico `AND`.

- Resultado

![[select 2.1.png]]


#### SELECT sem condições específicas

```sql
SELECT descricao, qtdmin, id FROM tb_produto;
```

Esta consulta retorna a descrição, quantidade mínima e ID de todos os produtos, sem aplicar qualquer filtro.

- Resultado

![[select 2.2.png]]

#### SELECT com OR

```sql
SELECT descricao, valor FROM tb_produto WHERE valor<1000 OR valor=1200;
```

Seleciona a descrição e o valor de produtos cujo valor é menor que 1000 ou igual a 1200, usando o operador lógico `OR` para aplicar um filtro composto.

- Resultado

![[select 2.3.png]]


#### SELECT com NOT

```sql
SELECT descricao FROM tb_produto WHERE NOT(valor>2000 AND id=3);
```

Seleciona a descrição de produtos que não têm valor maior que 2000 e ID igual a 3, utilizando o operador `NOT` para inverter a condição especificada.

- Resultado

![[select 2.4.png]]


#### Ordenando resultados ASC

```sql
SELECT descricao FROM tb_produto ORDER BY descricao ASC;
```

Ordena os produtos pela descrição em ordem ascendente.

- Resultado

![[select 2.5.png]]

#### Ordenando resultados DESC

```sql
SELECT descricao FROM tb_produto ORDER BY descricao DESC;
```

Ordena os produtos pela descrição em ordem descendente.

- Resultado

![[select 2.6.png]]


#### SELECT com ORDER BY

```sql
SELECT descricao FROM tb_produto WHERE valor >= 1000 ORDER BY descricao;
```

Seleciona e ordena a descrição de produtos com valor igual ou superior a 1000, sem especificar a direção da ordenação (padrão ASC).

- Resultado

![[select 2.8.png]]

#### Funções de agregação

- AVG
  ```sql
  SELECT AVG(valor) FROM tb_produto;
  ```
  
  Calcula a média dos valores dos produtos.

- Resultado

![[select 2.9.png]]


- COUNT
  ```sql
  SELECT COUNT(*) FROM tb_produto;
  ```
  
  Conta o total de produtos.

- Resultado

![[select 2.10.png]]


- COUNT com GROUP BY e HAVING
  ```sql
  SELECT COUNT(*) FROM tb_produto GROUP BY id_categoria HAVING AVG(valor)>=50;
  ```
  
  Conta os produtos agrupados por categoria, apenas onde a média do valor é maior ou igual a 50.

- Resultado

![[select 2.11.png]]


- SUM
  ```sql
  SELECT sum(valor) FROM tb_produto;
  ```
  
  Soma o valor de todos os produtos.

- Resultado

![[select 2.12.png]]


#### Transformação de texto

- UPPER
  ```sql
  SELECT UPPER(descricao) FROM tb_produto;
  ```
  
  Transforma a descrição de todos os produtos em letras maiúsculas.

- Resultado

![[select 2.13.png]]


- LOWER
  ```sql
  SELECT LOWER(descricao) FROM tb_produto;
  ```
  
  Transforma a descrição de todos os produtos em letras minúsculas.

- Resultado

![[select 2.14.png]]


- SUBSTR
  ```sql
  SELECT substr(descricao,2,length(descricao)) FROM tb_produto;
  ```
  
  Extrai parte da descrição de cada produto, começando do segundo caractere até o final.

- Resultado

![[select 2.15.png]]


- RPAD
  ```sql
  SELECT RPAD(descricao, length(descricao)+6, "$") FROM tb_produto;
  ```
  
  Adiciona o símbolo `$` à direita da descrição de cada produto, até atingir o comprimento especificado.

- Resultado

![[select 2.16.png]]


- LPAD
  ```sql
  SELECT LPAD(descricao, 9, "$") FROM tb_produto;
  ```
  
  Adiciona o símbolo `$` à esquerda da descrição de cada produto, até atingir o comprimento especificado.

- Resultado

![[select 2.17.png]]


#### Valores máximos e mínimos

- MAX
  ```sql
  SELECT MAX(valor) FROM tb_produto;
  ```
  
  Seleciona o maior valor dentre todos os produtos.

- Resultado

![[select 2.18.png]]


- MIN
  ```sql
  SELECT MIN(valor) FROM tb_produto;
  ```
  
  Seleciona o menor valor dentre todos os produtos.

- Resultado

![[select 2.19.png]]


### Conclusão

O uso de operadores lógicos e funções de agregação no SQL possibilita consultas detalhadas e específicas, permitindo aos desenvolvedores manipular e analisar dados de formas complexas e eficientes.

[[JOIN, UNION, Views, PROCEDURE, e TRIGGER]]



