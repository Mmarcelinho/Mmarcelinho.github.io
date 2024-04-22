---
title: Iniciando em SQL
date: 2024-04-02
tags:
  - SQL
  - MYSQL
  - Comandos
  - Comandos-SQL
---
Olá! Hoje, abordarei o SQL, demonstrando sua aplicação prática com um exemplo real. Atualmente, é essencial para um desenvolvedor compreender os fundamentos do SQL. Mais do que apenas entender os comandos, é crucial saber como eles se aplicam em cenários do dia a dia de um banco de dados. Portanto, explorarei desde comandos essenciais até conceitos mais avançados, necessários para gerenciar um banco de dados de maneira eficiente.


> [!Requisitos]
> - MySQL
> - MYSQL Workbench

Não é obrigátorio instalar o Workbench. Eu utilizo o Azure Data Studio com uma imagem do MYSQL rodando em um container docker, Fica ai a sugestão.

#### Arquivos utilizados para criar os posts

[Estoque-SQL](https://github.com/Mmarcelinho/estoque-arquivo-sql)

Vamos praticar! 
## Código

Primeiro começaremos com a criação do banco de dados

```sql
CREATE DATABASE ESTOQUE;
```

Este comando cria um novo banco de dados chamado `ESTOQUE`

Agora precisamos selecionar o banco `ESTOQUE`  como o banco de dados ativo para os próximos comandos SQL

```sql
USE ESTOQUE;
```

Com isso já podemos inserir as tabelas específicas no banco de dados 

 Vamos criar a primeira tabela

```sql
CREATE TABLE tb_categoria(
id int primary key not null auto_increment,
categoria varchar(50) not null
);
```

Nesta tabela:

- Criamos a tabela com o nome `tb_categoria` 
- Indicamos que `id` será a chave primaria da tabela
- O `id` é não nulo e se auto incrementa a cada objeto
-  O nome da `categoria` é do tipo varchar(50) e não nulo

Agora criaremos as próximas tabelas que são parecidas com a primeira

#### Tabela Cidade

```sql
CREATE TABLE tb_cidade(
id int primary key not null auto_increment,
cidade varchar(50) not null,
uf char(2) not null
);
```

#### Tabela Fornecedor

```sql
CREATE TABLE tb_fornecedor(
id int primary key not null auto_increment,
fornecedor varchar(50) not null,
endereco varchar(50) not null,
num int,
bairro varchar(50) not null,
cep char(9) not null,
contato varchar(50) not null,
cnpj char(18) not null,
insc varchar(50),

-- Chave estrangeira
id_cidade int,
foreign key(id_cidade) references tb_cidade(id)

);
```

Opa, parece que tem um conceito novo na tabela. Temos na tabela `Fornecedor` um `id_cidade` e o comando `foreign key(id_cidade) references tb_cidade(id)` mas o que isso quer dizer?

O `FOREIGN KEY` mais conhecido como chave estrangeira, são chaves que estabelecem uma relação de dependência entre as tabelas. Por exemplo `id_cidade` na tabela `tb_fornecedor` é uma chave estrangeira que referencia o `id` na tabela `tb_cidade`, indicando  qual cidade o fornecedor pertence.

Esse é um relacionamento de um para muitos, onde, uma cidade possui muitos fornecedores.

- Diagrama de exemplo:

![[Diagrama.png]]


#### Tabela Produto

```sql
CREATE TABLE tb_produto(
id int primary key not null auto_increment,
valor FLOAT (8,2),
taxa FLOAT (8,2),
descricao varchar(50) not null,
peso double,
controlado bool,
qtdmin int,

-- Chave estrangeira
id_fornecedor int,
id_categoria int,
foreign key(id_fornecedor) references tb_fornecedor(id),
foreign key(id_categoria) references tb_categoria(id)
);
```

A tabela produto possui duas chaves estrangeiras
- Um fornecedor possui muitos produtos
- Uma categoria possui muitos produtos

#### Tabela Loja

```sql
CREATE TABLE tb_loja(
id int primary key not null auto_increment,
nome varchar(50) not null,
endereco varchar(50) not null,
num int,
bairro varchar(50) not null,
tel char(14) not null,
insc varchar(50) not null,
cnpj char(18) not null,

-- Chave estrangeira
id_cidade int,
foreign key(id_cidade) references tb_cidade(id)
);
```

A tabela loja possui uma chave estrangeira
- Uma cidade possui muitas lojas

#### Tabela Transportadora

```sql
CREATE TABLE tb_transportadora(
id int primary key not null auto_increment,
transportadora varchar(50) not null,
endereco varchar(50) not null,
num int,
bairro varchar(50) not null,
cep char(14) not null,
cnpj char(18) not null,
insc varchar(50) not null,
contato varchar(50) not null,
tel char(14) not null,

-- Chave estrangeira
id_cidade int,
foreign key(id_cidade) references tb_cidade(id)
);
```

A tabela transportadora possui uma chave estrangeira
- Uma cidade possui muitas transportadoras

#### Tabela Entrada

```sql
CREATE TABLE tb_entrada(
id int primary key not null auto_increment,
dataped date not null,
dataentr date not null,
total double,
frete double,
numnf int,
imposto double,

-- Chave estrangeira
id_transportadora int,
FOREIGN KEY(id_transportadora) references tb_transportadora(id)
);
```

A tabela entrada possui uma chave enstrangeira
- Uma transportadora possui muitas entradas

#### Tabela Item Entrada

```sql
CREATE TABLE tb_itementrada(
id int primary key not null auto_increment,
lote varchar(50) not null,
qtde int not null,
valor double,

-- Chave estrangeira
id_entrada int,
id_produto int,
FOREIGN KEY(id_entrada) references tb_entrada(id),
FOREIGN KEY(id_produto) references tb_produto(id)
);
```

A tabela item entrada possui duas chaves entrangeiras
- Uma entrada possui muitos item entrada
- Um produto possui muitos item entrada

#### Tabela Saida

```sql
CREATE TABLE tb_saida(
id int primary key not null auto_increment,
total double,
frete double,
imposto double,

-- Chave estrangeira
id_loja int,
id_transportadora int,
FOREIGN KEY(id_transportadora) references tb_transportadora(id),
FOREIGN KEY(id_loja) references tb_loja(id)
);
```

A tabela saída possui duas chaves estrangeiras
- Uma transportadora possui muitas saídas
- Uma loja possui muitas saídas

#### Tabela Item Saida

```sql
CREATE TABLE tb_itemsaida(
id int primary key not null auto_increment,
lote varchar(50) not null,
qtde int not null,
valor double,

-- Chave estrangeira
id_saida int,
id_produto int,
FOREIGN KEY(id_produto) references tb_produto(id),
FOREIGN KEY(id_saida) references tb_saida(id)
);
```

Na tabela item saída possui duas chaves estrangeiras 
- Um produto possui muitas saídas
-  Uma saída possui muitos item saída


Essas são as tabelas do banco de dados `ESTOQUE`. Algumas delas estão relacionadas entre si por meio de chaves estrangeiras, sendo uma representação das relações do mundo real no banco de dados.

Abaixo será explicado alguns conceitos importantes

## Tirando dúvidas

O que é `PRIMARY KEY`?
- Cada tabela tem um campo `id` que é definido como chave primária, o que significa que cada registro nessa tabela pode ser unicamente identificado por esse identificador `id`.

Para que serve o `not null`?
- O `not null` é utilizado para tornar o campo assinado obrigátorio.

E o `auto_increment`? 
- Serve para incrementar automaticamente um id único para cada registro, começa pelo valor 1 e vai incrementando de um em um para cada novo registro.

Como funciona o `VARCHAR(50)`?
- Quando um campo é definido como `VARCHAR` está dizendo que esse campo pode armazenar strings com até 50 caracteres.

Quando usar `VARCHAR` e `CHAR`?
- `VARCHAR` pode ser utilizado para dados de comprimento variável e `CHAR` para dados de comprimento fixo. Por exemplo o CEP que possui o comprimento fixo de 8 números.

## Conclusão

Este é um guia básico de SQL na prática. Explore mais fontes para aprofundar seu conhecimento, já que alguns tópicos não foram cobertos aqui. No próximo post, abordarei operações como inserir, atualizar, deletar e modificar dados nas tabelas.

[[Iniciando em SQL - SELECT, UPDATE, DELETE E ALTER]]

