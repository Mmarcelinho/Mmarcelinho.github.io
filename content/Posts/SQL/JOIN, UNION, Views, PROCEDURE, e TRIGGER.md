---
title: JOIN, UNION, Views, PROCEDURE, e TRIGGER
date: 2024-04-02
tags:
  - SQL
  - MYSQL
  - Comandos
  - Comandos-SQL
---
Nesse post iremos abordar diferentes recursos do SQL que permitem a manipulação avançada de dados em um banco de dados relacional.

## Código

### JOIN

#### INNER JOIN

```sql
SELECT tb_fornecedor.fornecedor, tb_cidade.cidade FROM tb_fornecedor INNER JOIN tb_cidade ON tb_fornecedor.id_cidade = tb_cidade.id;
```

Esta consulta utiliza `INNER JOIN` para combinar linhas de `tb_fornecedor` com linhas de `tb_cidade` onde os IDs das cidades coincidem, retornando fornecedor e cidade correspondentes.

- Resultado

![[join.1.png]]


#### LEFT JOIN

```sql
SELECT tb_fornecedor.fornecedor, tb_cidade.cidade FROM tb_fornecedor LEFT JOIN tb_cidade ON tb_fornecedor.id_cidade = tb_cidade.id;
```

Utiliza um `LEFT JOIN` para selecionar todos os registros da tabela à esquerda (`tb_fornecedor`) juntamente com os registros correspondentes da tabela à direita (`tb_cidade`). Se não houver correspondência, o resultado para a tabela à direita será NULL.

- Resultado

![[join.2.png]]

#### RIGHT JOIN

```sql
SELECT tb_produto.descricao, tb_categoria.categoria FROM tb_produto RIGHT JOIN tb_categoria ON tb_produto.id_categoria = tb_categoria.id;
```

Executa um `RIGHT JOIN`, escolhendo todos os registros da tabela à direita (`tb_categoria`) e os registros correspondentes da tabela à esquerda (`tb_produto`). Se não houver correspondência, o resultado para a tabela à esquerda será NULL.

- Resultado

![[join.3.png]]


### Union

```sql
SELECT id_cidade FROM tb_loja
UNION
SELECT id_cidade FROM tb_fornecedor
ORDER BY id_cidade;
```

Combina os resultados de duas consultas sem duplicatas, graças ao operador `UNION`, e os ordena pelo `id_cidade`.

- Resultado

![[union.1.png]]

#### UNION ALL
```sql
SELECT id_cidade FROM tb_loja
UNION ALL
SELECT id_cidade FROM tb_fornecedor
ORDER BY id_cidade;
```

Utiliza `UNION ALL` para combinar os resultados de duas consultas, incluindo duplicatas.

- Resultado

![[union.2.png]]

#### Simulando FULL OUTER JOIN com UNION

```sql
SELECT * FROM tb_fornecedor
LEFT JOIN tb_cidade ON tb_fornecedor.id = tb_cidade.id
UNION ALL
SELECT * FROM tb_fornecedor
RIGHT JOIN tb_cidade ON tb_fornecedor.id = tb_cidade.id
WHERE tb_cidade.id IS NULL;
```

Combina `LEFT JOIN` e `RIGHT JOIN` com `UNION ALL` para simular um `FULL OUTER JOIN`, unindo dados de duas tabelas independentemente de correspondências diretas entre elas.

- Resultado

![[outer-join com union.png]]

### Criação de Views

```sql
CREATE VIEW viewCidade AS
SELECT tb_fornecedor.fornecedor AS Fornecedor, tb_cidade.cidade AS Cidade
FROM tb_fornecedor
INNER JOIN tb_cidade ON tb_fornecedor.id = tb_cidade.id;

SELECT Fornecedor, Cidade FROM viewCidade ORDER BY Fornecedor;
```

Define uma `View` chamada `viewCidade` para simplificar e organizar a consulta de dados relacionados entre fornecedores e cidades, e em seguida realiza uma seleção a partir dessa view.

- Resultado

![[views.png]]

### PROCEDURE

```sql
CREATE PROCEDURE verValor(varProduto SMALLINT)
BEGIN
    SELECT CONCAT('O valor:', valor) FROM tb_produto WHERE id = varProduto;
END;

CALL verValor(2);
```

Cria uma `PROCEDURE` chamada `verValor` que, quando chamada, retorna o valor de um produto baseado em seu ID.

- Resultado

![[procedure.png]]


### TRIGGER

```sql
ALTER TABLE tb_produto ADD Preco_desconto DECIMAL(5,2);

CREATE TRIGGER tr_desconto BEFORE INSERT ON tb_produto
FOR EACH ROW
SET NEW.Preco_desconto = (NEW.valor * 0.20);

INSERT INTO tb_produto(descricao, valor, qtdmin, taxa) VALUES ('Monitor', '900.00', '30', '20');
```

Adiciona uma coluna `Preco_desconto` à tabela `tb_produto` e define um `TRIGGER` chamado `tr_desconto`, que automaticamente calcula um desconto antes de cada inserção de novos registros na tabela.

- Resultado

![[trigger.png]]

## Conclusão

A utilização de JOIN, UNION, Views, Procedures e Triggers no SQL oferece uma gama robusta de ferramentas, permitindo a realização de consultas complexas, a manipulação eficiente de dados e a automatização de tarefas.