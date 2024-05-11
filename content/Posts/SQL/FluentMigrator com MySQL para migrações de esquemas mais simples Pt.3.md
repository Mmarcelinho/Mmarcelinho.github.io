---
title: FluentMigrator com MySQL para migrações de esquemas mais simples Pt.3
date: 2024-05-10
tags:
  - MYSQL
  - FluentMigrator
  - Migrations
  - DotNET
  - CSharp
---
Por fim, vamos criar a primeira versão da migração com a tabela de exemplo e suas colunas correspondentes.

### Versão Base

Para evitar a duplicação de código, vamos criar uma versão base.

No diretório `Migrations/`, crie a classe `VersaoBase`:

```csharp
public class VersaoBase
{
    // Este é um método estático chamado InserirColunasPadrao. 
    // Ele recebe um parâmetro do tipo ICreateTableWithColumnOrSchemaOrDescriptionSyntax chamado 'tabela'.
    public static ICreateTableColumnOptionOrWithColumnSyntax InserirColunasPadrao(ICreateTableWithColumnOrSchemaOrDescriptionSyntax tabela)
    {
        // Aqui, estamos adicionando colunas à tabela passada como parâmetro.
        // A coluna "Id" é do tipo Int64, é a chave primária e é autoincrementada.
        // A coluna "DataCriacao" é do tipo DateTime e não pode ser nula.
        return tabela
        .WithColumn("Id").AsInt64().PrimaryKey().Identity()
        .WithColumn("DataCriacao").AsDateTime().NotNullable();
    }
}
```

Ao criar novas versões, utilizamos o método acima, passando o tipo que recebe e o nome da tabela a ser criada.

### Enum

Também vamos criar um `Enum` com os números das versões para passar via atributo, juntamente com a descrição.

No diretório `Migrations/`, adicione:

```csharp
public enum NumeroVersoes
{
    CriarTabelaAdmin = 1
}
```

### Primeira Versão

No diretório `Migrations/Versoes`, crie a classe `Versao0000001`.

Observação: Para padronizar as versões utilize o nome "Versao" + `000000` + Número da versão

A classe deve herdar de `Migration` e sobrescrever seus métodos:

```csharp
public class Versao0000001 : Migration
{
    public override void Down() { }

    public override void Up() { }
}
```

Vamos criar a tabela utilizando a versão base e passando o  `Create.Table( )`  com o nome da tabela como parâmetros:

```csharp
public class Versao0000001 : Migration
{
    public override void Down() { }

    public override void Up()
    {
        var tabela = VersaoBase.InserirColunasPadrao(Create.Table("Admins"));
    }
}
```

Agora, vamos adicionar as respectivas colunas:

```csharp
public class Versao0000001 : Migration
{
    public override void Down() { }

    public override void Up()
    {
        var tabela = VersaoBase.InserirColunasPadrao(Create.Table("Admins"));

        tabela
            .WithColumn("Nome").AsString(100).NotNullable()
            .WithColumn("Email").AsString().NotNullable()
            .WithColumn("Senha").AsString(2000).NotNullable()
            .WithColumn("Telefone").AsString(14).NotNullable()
            .WithColumn("Administrador").AsBoolean().WithDefaultValue(true).NotNullable();
    }
}
```

Lembre-se de passar o atributo com o número da versão e a descrição:

```csharp
[Migration((long)NumeroVersoes.CriarTabelaAdmin, "Cria tabela admin")]
public class Versao0000001 : Migration
{
    public override void Down() { }

    public override void Up()
    {
        var tabela = VersaoBase.InserirColunasPadrao(Create.Table("Admins"));

        tabela
            .WithColumn("Nome").AsString(100).NotNullable()
            .WithColumn("Email").AsString().NotNullable()
            .WithColumn("Senha").AsString(2000).NotNullable()
            .WithColumn("Telefone").AsString(14).NotNullable()
            .WithColumn("Administrador").AsBoolean().WithDefaultValue(true).NotNullable();
    }
}
```

Pronto! Ao executar o projeto, as migrações serão geradas no banco de dados. Sempre que o projeto for iniciado, será verificado se existem migrações pendentes.

## Projeto de exemplo

- [SistemaDeEstoque](https://github.com/Mmarcelinho/sistema_de_estoque)

