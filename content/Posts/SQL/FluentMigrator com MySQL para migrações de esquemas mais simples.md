---
title: FluentMigrator com MySQL para migrações de esquemas mais simples
date: 2024-05-10
tags:
  - MYSQL
  - FluentMigrator
  - Migrations
  - DotNET
  - CSharp
---
Vamos explorar como criar migrações mais limpas e organizadas usando `FluentMigrator` e o banco de dados `MySQL`.

### Pré-requisitos:

- MySQL

Começaremos com as configurações mais básicas. Não vou detalhar cada elemento ou como funciona, pois estou assumindo que você já possui algum conhecimento em WebApi com ASP.NET. O projeto que estarei usando segue o padrão arquitetural Clean Architecture, com os projetos separados em camadas. No entanto, sinta-se à vontade para adaptar conforme sua preferência, a implementação será a mesma.

### Configurando a ConnectionString

No arquivo appsettings.json, adicione:

```csharp
"ConnectionStrings": {
"NomeDatabase": "sistemaEstoque"
}
```

Aqui, estamos definindo o nome do banco de dados.

Em seguida, crie um arquivo chamado appsettings.Development.json e adicione a ConnectionString:

```csharp
"ConnectionStrings": {
"Conexao": "Server=localhost;Uid=root;Pwd=1q2w3e4r@#$;"
},
```

Certifique-se de que a senha corresponde à do seu MySQL.

Agora, vamos criar um método de extensão para obter o nome do banco de dados, a conexão e a conexão completa. Este método reduz a duplicidade de código, permitindo a reutilização do método.

Adicione o pacote para ter acesso à interface IConfiguration:

```
dotnet add package Microsoft.Extensions.Configuration
```

No projeto Domain, em Extension/, adicione:

```csharp
public static class RepositorioExtension
    {
        public static string GetNomeDatabase(this IConfiguration configuration)
        {
            var nomeDatabase = configuration.GetConnectionString("NomeDatabase");

            return nomeDatabase;
        }

        public static string GetConexao(this IConfiguration configuration)
        {
            var conexao = configuration.GetConnectionString("Conexao");

            return conexao;
        }

        public static string GetConexaoCompleta(this IConfiguration configuration)
        {
            var nomeDatabase = configuration.GetNomeDatabase();
            var conexao = configuration.GetConexao();

            return $"{conexao}Database={nomeDatabase}";
        }
    }
```

Com essas configurações básicas concluídas, vamos agora para o projeto Infrastructure, onde toda a configuração do banco de dados será realizada.

### Verificação do Schema

Após configurar a ConnectionString vamos verificar se o schema do database já existe no MySQL. Caso não exista, será criado com o nome configurado no arquivo appsettings.json.

Para realizar essa verificação, precisaremos de dois pacotes:

```
dotnet add package Dapper;
dotnet add package MySqlConnector;
```

Crie uma pasta chamada Migrations e, dentro dela, crie a classe Database:

```csharp
public static class Database
{
    public static void CriarDatabase(string conexaoComBancoDeDados, string nomeDatabase)
    {
        
    }
}
```

Essa classe será estática e conterá um método que recebe como parâmetros a string de conexão e o nome do database.

Vamos estabelecer uma conexão com o banco de dados e criar um parâmetro dinâmico que recebe o nome do database para ser passado para a consulta:

```csharp
public static class Database
{
    public static void CriarDatabase(string conexaoComBancoDeDados, string nomeDatabase)
    {
        using var minhaConexao = new MySqlConnection(conexaoComBancoDeDados);

        var parametros = new DynamicParameters();
        parametros.Add("nome", nomeDatabase);
    }
}
```

Por fim, adicionamos a consulta SQL, que realiza um `SELECT` para verificar no MySQL se existe um schema com o nome passado pelo parâmetro:

```csharp
public static class Database
{
    public static void CriarDatabase(string conexaoComBancoDeDados, string nomeDatabase)
    {
        using var minhaConexao = new MySqlConnection(conexaoComBancoDeDados);

        var parametros = new DynamicParameters();
        parametros.Add("nome", nomeDatabase);

        var registros = minhaConexao.Query("SELECT * FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = @nome", parametros);

        if (!registros.Any())
            minhaConexao.Execute($"CREATE DATABASE {nomeDatabase}");
    }
}
```

Se não houver registros, o database será criado com o nome passado por parâmetro.

### Invocando o Método

Para que o método funcione, é necessário invocá-lo na classe principal do projeto, onde os serviços estão contidos.

No arquivo Program.cs, crie uma função para chamar o método e passar os parâmetros:

```csharp
app.Run();

void AtualizarBaseDeDados()
{
    var conexao = builder.Configuration.GetConexao();

    var nomeDatabase = builder.Configuration.GetNomeDatabase();

    Database.CriarDatabase(conexao, nomeDatabase);
}
```

Agora, chame a função:

```csharp
AtualizarBaseDeDados();

app.Run();

void AtualizarBaseDeDados()
{
    var conexao = builder.Configuration.GetConexao();

    var nomeDatabase = builder.Configuration.GetNomeDatabase();

    Database.CriarDatabase(conexao, nomeDatabase);
}
```

Ao rodar o projeto poderá verificar no MySQL que o schema foi gerado.

[[FluentMigrator com MySQL para migrações de esquemas mais simples Pt.2]]
