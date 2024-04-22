---
title: Iniciando em SQL - SELECT, UPDATE, DELETE E ALTER
date: 2024-04-02
tags:
  - SQL
  - MYSQL
  - Comandos
  - Comandos-SQL
---
Tabelas criadas, agora podemos fazer operações como inserir, atualizar e alterar dados.

## Código
### Insert

O comando `INSERT` é utilizado para adicionar novas linhas a uma tabela. Ao executar esse comando, especificamos a tabela, a coluna onde os valores serão inseridos e, os próprios valores. Segue o exemplo:

```sql
INSERT INTO tb_categoria(categoria) VALUES ('Acessórios');
INSERT INTO tb_categoria(categoria) VALUES ('Beleza');
INSERT INTO tb_categoria(categoria) VALUES ('Informática');
INSERT INTO tb_categoria(categoria) VALUES ('Jardinagem');
INSERT INTO tb_categoria(categoria) VALUES ('Pet Shop');
```

Neste exemplo, selecionamos a tabela `tb_categoria` para inserção, especificamos a coluna `categoria`, e passamos os valores desejados utilizando o comando `VALUES`. Cada valor é inserido em uma nova linha da tabela, na coluna especificada.

#### `INSERT` na tabela Cidade

```sql
#INSERT TABELA CIDADE
INSERT INTO tb_cidade(cidade, uf) VALUES ('Manaus', '13');
INSERT INTO tb_cidade(cidade, uf) VALUES ('Natal', '24');
INSERT INTO tb_cidade(cidade, uf) VALUES ('Goiânia', '52');
INSERT INTO tb_cidade(cidade, uf) VALUES ('Campo Grande', '50');
INSERT INTO tb_cidade(cidade, uf) VALUES ('Belém', '15');
```

Na tabela `tb_cidade`, passamos duas colunas (`cidade` e `uf`) e inserimos valores correspondentes para cada uma delas.

#### `INSERT` na tabela Fornecedor

```sql
INSERT INTO tb_fornecedor(fornecedor, endereco, bairro, cep, contato, cnpj, id_cidade) VALUES ('Elegância Acessórios', 'Av. Brasil', 'Comércio', '69050030', '92988556677', '11122233344455', 1);

INSERT INTO tb_fornecedor(fornecedor, endereco, bairro, cep, contato, cnpj, id_cidade) VALUES ('Beleza Pura', 'Rua das Flores', 'Petrópolis', '59020240', '84988556677', '55566677788899', 2);

INSERT INTO tb_fornecedor(fornecedor, endereco, bairro, cep, contato, cnpj, id_cidade) VALUES ('TechInfo', 'Av. Goiás', 'Setor Central', '74005310', '62988556677', '99988877766655', 3);

INSERT INTO tb_fornecedor(fornecedor, endereco, bairro, cep, contato, cnpj, id_cidade) VALUES ('Green Garden', 'Rua Ceará', 'Monte Castelo', '79010200', '67988556677', '44433322211100', 4);

INSERT INTO tb_fornecedor(fornecedor, endereco, bairro, cep, contato, cnpj, id_cidade) VALUES ('Pet Feliz', 'Av. Nazaré', 'Nazaré', '66035120', '91988556677', '22233344455566', 5);

```

Na tabela `tb_fornecedor`, incluímos o `id_cidade` como uma chave estrangeira que estabelece a relação entre fornecedor e cidade. É importante garantir que o valor de `id_cidade` inserido exista na tabela `tb_cidade`, caso contrário, um erro será gerado devido à violação da integridade referencial.

#### `INSERT` na tabela Produto

```sql
INSERT INTO tb_produto(descricao, qtdmin, id_fornecedor, id_categoria) VALUES ('Óculos de Sol', '150', 1, 1);
INSERT INTO tb_produto(descricao, qtdmin, id_fornecedor, id_categoria) VALUES ('Kit de Maquiagem', '200', 2, 2);
INSERT INTO tb_produto(descricao, qtdmin, id_fornecedor, id_categoria) VALUES ('Notebook Gamer', '50', 3, 3);
INSERT INTO tb_produto(descricao, qtdmin, id_fornecedor, id_categoria) VALUES ('Conjunto de Ferramentas para Jardim', '100', 4, 4);
INSERT INTO tb_produto(descricao, qtdmin, id_fornecedor, id_categoria) VALUES ('Ração para Cães', '300', 5, 5);
```

Na tabela `tb_produto` incluímos o `id_fornecedor` e `id_categoria` para vincular cada produto a seu fornecedor e categoria

#### `INSERT` na tabela Loja

```sql
INSERT INTO tb_loja(nome, endereco, num, bairro, tel, insc, cnpj, id_cidade) VALUES ('Acessórios Modernos', 'Av. Eduardo Ribeiro', 120, 'Centro', '(92) 9985-6677', 'AM345678901', '77.888.999/0001-66', 1);
INSERT INTO tb_loja(nome, endereco, num, bairro, tel, insc, cnpj, id_cidade) VALUES ('Beleza em Casa', 'Rua Potengi', 580, 'Petrópolis', '(84) 9985-6677', 'RN123456789', '66.777.888/0001-55', 2);
INSERT INTO tb_loja(nome, endereco, num, bairro, tel, insc, cnpj, id_cidade) VALUES ('Info Tech Loja', 'Av. Anhanguera', 1080, 'Setor Central', '(62) 9985-6677', 'GO987654321', '55.666.777/0001-44', 3);
INSERT INTO tb_loja(nome, endereco, num, bairro, tel, insc, cnpj, id_cidade) VALUES ('Jardim & Cia', 'Rua 14 de Julho', 450, 'Centro', '(67) 9985-6677', 'MS876543219', '44.555.666/0001-33', 4);
INSERT INTO tb_loja(nome, endereco, num, bairro, tel, insc, cnpj, id_cidade) VALUES ('Pet Shop Amigo Fiel', 'Av. Almirante Barroso', 220, 'Marco', '(91) 9985-6677', 'PA765432198', '33.444.555/0001-22', 5);
```

Na tabela `tb_loja`, também incluímos o `id_cidade` para vincular cada loja à sua respectiva cidade.

**Visualizando os Dados com `SELECT`**

Para ver os dados inseridos nas tabelas, utilizamos o comando `SELECT`:

```sql
select * from tb_categoria;
select * from tb_cidade;
select * from tb_fornecedor;
select * from tb_produto;
select * from tb_loja;
```

O comando `SELECT *` nos permite visualizar todos os registros de uma tabela específica.

### DELETE

O comando `DELETE` é utilizado para remover uma ou mais linhas de uma tabela, com base em uma condição especificada. 

#### Removendo linhas da tabela

```sql
DELETE FROM tb_produto WHERE descricao = 'Óculos de Grau';
```

Este comando removerá todas as linhas da tabela `tb_produto` que tenham a descrição igual a 'Óculos de Grau'. É importante utilizar a condição `WHERE` para não fazer o famoso DELETE sem `WHERE` 🤣.

- Antes

![[select.7.png]]

- Depois

![[select.8.png]]


### UPDATE

O comando `UPDATE` é utilizado para atualizar as linhas de uma tabela. Tem a mesma abordagem do `INSERT` com o diferencial de que passamos o `id` do campo que queremos modificar. Segue o exemplo:

```sql
#UPDATE
UPDATE tb_produto set descricao = 'Óculos de Grau' where id=1;
```

#### Antes:
![[update.1.png]]

#### Depois

![[update.2.png]]


#### Inserindo valores e taxa dos produtos

```sql
#UPDATE VALOR
UPDATE tb_produto SET valor = '50.00' where id=1;
UPDATE tb_produto SET valor = '30.00' where id=2;
UPDATE tb_produto SET valor = '60.00' where id=3;
UPDATE tb_produto SET valor = '2500.00' where id=4;
UPDATE tb_produto SET valor = '1200.00' where id=5;

UPDATE tb_produto SET taxa = '10.00' where id=1;
UPDATE tb_produto SET taxa = '15.00' where id=2;
UPDATE tb_produto SET taxa = '20.00' where id=3;
UPDATE tb_produto SET taxa = '200.00' where id=4;
UPDATE tb_produto SET taxa = '120.00' where id=5;
```

#### Valores atualizados

![[update.3.png]]

### ALTER  TABLE

O comando `ALTER TABLE` é utilizado para alterar uma tabela, desde renomear tabelas à adicionar um novo campo. Segue o exemplo:

```sql
ALTER TABLE tb_produto RENAME tb_produtos;
```

No comando acima, alteramos o nome da tabela `tb_produto` para `tb_produtos` 

Após alterar o nome, ao tentar utilizar a tabela com o nome antigo apresenta um erro
![[alter table.1.png]]

#### Adicionando um novo campo

```sql
ALTER TABLE tb_produto add qtdAtual int;
```

Com esse comando é adicionado um campo `qtdAtual` do tipo `int` na tabela `tb_produto` 

## Conclusão

Dominar as operações de INSERT, UPDATE e ALTER TABLE é crucial para gerenciar e manter bancos de dados relacionais eficientemente, recomendo a prática contínua dessas operações.

[[SQL - Dominando o SELECT]]

