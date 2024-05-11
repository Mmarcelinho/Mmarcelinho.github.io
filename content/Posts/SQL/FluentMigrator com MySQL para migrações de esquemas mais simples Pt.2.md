---
title: FluentMigrator com MySQL para migrações de esquemas mais simples Pt.2
date: 2024-05-10
tags:
  - MYSQL
  - FluentMigrator
  - Migrations
  - DotNET
  - CSharp
---
Já que configuramos a conexão e geramos o database, vamos prosseguir com a configuração das migrações.

### Implementando MigrationRunner

Adicione os pacotes que vão ser utilizados:

```
dotnet add package FluentMigrator
dotnet add package FluentMigrator.Runner
```

Primeiro, vamos criar um método de extensão para a migração do database. Em Migrations/, adicione a classe `MigrateExtension`:

```csharp
public static class MigrateExtension
{
    public static void MigrateBancoDeDados(this IApplicationBuilder app)
     {
        // Cria um escopo para o serviço de migração
        using var scope = app.ApplicationServices.CreateScope();

        // Obtém o serviço de migração
        var runner = scope.ServiceProvider.GetRequiredService<IMigrationRunner>();
        
        // Lista as migrações disponíveis
        runner.ListMigrations();

        // Executa todas as migrações disponíveis
        runner.MigrateUp();
    }
}

```

Este método irá executar todas as migrações disponíveis que ainda não foram aplicadas ao banco de dados.


### Configurando o serviço

Agora, vamos configurar o FluentMigrator. Para isso, crie o seguinte método na classe Bootstrapper:

```csharp
private static void AdicionarFluentMigrator(IServiceCollection services, IConfiguration configuration)
{
    services.AddFluentMigratorCore().ConfigureRunner(c => c.AddMySql8().WithGlobalConnectionString(configuration.GetConexaoCompleta()).ScanIn(Assembly.Load("SistemaDeEstoque.Infrastructure")).For.All());
}
```

O código acima:

- Adiciona o FluentMigratorCore ao serviço de injeção de dependência
- Configura o FluentMigrator para usar o MySQL 8
- Usa a string de conexão completa 
- Faz a varredura no assembly “SistemaDeEstoque.Infrastructure” para todas as migrações
- Aplica todas as migrações encontradas

A classe Bootstrapper é utilizada para adicionar os serviços de cada projeto. Com ela podemos organizar os serviços por projeto e diminuir a quantidade de linhas de código da classe Main

Primeiro, adicione o pacote com o seguinte comando:

```csharp
dotnet add package Microsoft.AspNetCore.Http.Abstractions;
```

Dentro da classe, crie um método que extende da interface `IServiceCollection` e chame a função `AdicionarFluentMigrator` passando os parâmetros:

```csharp
public static class Bootstrapper
{
    public static void AdicionarInfrastructure(this IServiceCollection services, IConfiguration configuration)
    {
        AdicionarFluentMigrator(services, configuration);
    }
}
```

Esses métodos irão adicionar o FluentMigrator ao nosso projeto e configurar o runner para usar o MySQL.


## Main

Na classe `Program`, adicione:

```csharp
builder.Services.AdicionarInfrastructure(builder.Configuration);
```

E na função `AtualizarBaseDeDados()`, chame o método de extensão `MigrateBancoDeDados()`:

```csharp
void AtualizarBaseDeDados()
{
    var conexao = builder.Configuration.GetConexao();

    var nomeDatabase = builder.Configuration.GetNomeDatabase();

    Database.CriarDatabase(conexao, nomeDatabase);

    // Executa as migrações do banco de dados
    app.MigrateBancoDeDados();
}
```

[[FluentMigrator com MySQL para migrações de esquemas mais simples Pt.3]]