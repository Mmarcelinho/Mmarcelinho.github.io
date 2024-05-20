---
title: Como Verificar a Existência de um Item no Banco de Dados com Dapper
date: 2024-05-20
tags:
  - CSharp
  - DotNET
  - SQL
  - MYSQL
  - Dapper
---
### Como Verificar a Existência de um Item no Banco de Dados com Dapper

Neste post, vou mostrar como verificar se um item existe no banco de dados utilizando Dapper, uma biblioteca leve para mapeamento de objetos e bancos de dados em .NET.

O Dapper adiciona métodos à interface `IDbConnection` que permitem executar consultas SQL e mapear os resultados para objetos no seu código. No entanto, ele não possui um método equivalente ao `AnyAsync()` do Entity Framework Core, que verifica se um elemento existe na sequência e retorna `true` se existir e `false` caso contrário. Vou apresentar uma solução para essa verificação no Dapper.

Como exemplo, irei demonstrar a implementação de um método de repositório para a entidade Admin:

```csharp
public interface IAdminReadOnlyRepositorio
{
    Task<bool> ExisteAdminComEmail(string email);
}
```

### Criando a Query

Em seguida, vamos criar uma query que conta quantos registros possuem o email fornecido:

```csharp
public static QueryModel RecuperarAdminExistentePorEmailQuery(string email)
{
    string tabela = Map.ContextMapping.RecuperarTabelaAdmin();
    string query = @$"SELECT COUNT(*) FROM {tabela} WHERE Email = @Email";

    var parameters = new
    {
        Email = email
    };

    return new QueryModel(query, parameters);
}
```

Essa query executa um `SELECT COUNT(*)` na tabela especificada e verifica quantos registros possuem o email fornecido como parâmetro.

### Implementando o Método

Agora, vamos implementar o método que utiliza essa query. Primeiro, chamamos o método estático que cria a query e passamos o parâmetro `email`:

```csharp
public async Task<bool> ExisteAdminComEmail(string email)
{
    var query = AdminQueries.RecuperarAdminExistentePorEmailQuery(email);
}
```

### Executando a Query

Para executar a query, utilizamos o método `ExecuteScalarAsync()`, que retorna o valor da contagem:

```csharp
public async Task<bool> ExisteAdminComEmail(string email)
{
    var query = AdminQueries.RecuperarAdminExistentePorEmailQuery(email);
    var count = await _connection.ExecuteScalarAsync<int>(query.Query, query.Parameters);
}
```

O método `ExecuteScalarAsync<int>()` realiza a consulta e converte o resultado para `int`, armazenando-o na variável `count`.

### Verificando o Resultado

Finalmente, retornamos `true` se `count` for maior que 0, indicando que o email existe na tabela:

```csharp
public async Task<bool> ExisteAdminComEmail(string email)
{
    var query = AdminQueries.RecuperarAdminExistentePorEmailQuery(email);
    var count = await _connection.ExecuteScalarAsync<int>(query.Query, query.Parameters);
    return count > 0;
}
```

Se o email existir, a query retornará um valor maior que 0, e o método retornará `true`. Caso contrário, retornará `false`.

### Conclusão

Embora o Dapper não tenha um método embutido para verificar a existência de um item como o `AnyAsync()` do Entity Framework Core, é possível alcançar o mesmo resultado utilizando uma combinação de query SQL e o método `ExecuteScalarAsync()`. Essa abordagem permite realizar verificações eficientes e mantém o código limpo e de fácil manutenção.


### Dúvidas

#### 1. **O que é o método `ExecuteScalarAsync<int>` e como ele funciona?**

O método `ExecuteScalarAsync<T>` em Dapper é utilizado para executar uma consulta SQL que retorna um único valor. Ele executa a consulta no banco de dados e retorna o valor da primeira coluna da primeira linha do conjunto de resultados, convertido para o tipo especificado (`T`). No caso de `ExecuteScalarAsync<int>`, ele retorna um valor inteiro.

**Funcionamento do `ExecuteScalarAsync<T>`:**
- **Execução da Consulta**: A consulta SQL fornecida é executada no banco de dados.
- **Retorno do Valor**: O valor da primeira coluna da primeira linha do conjunto de resultados é retornado.
- **Conversão do Tipo**: O valor retornado é convertido para o tipo especificado (`int` neste caso).

#### 2. **O que acontece se existirem vários registros com o mesmo email?**

No contexto da consulta `SELECT COUNT(*) FROM Admins WHERE Email = @Email`, o método `COUNT(*)` retorna o número de registros que correspondem ao critério `WHERE Email = @Email`. Não importa quantos registros correspondam ao critério, o `COUNT(*)` sempre retornará um único valor inteiro que representa a quantidade de registros.

Se existirem múltiplos registros com o mesmo email, a função `COUNT(*)` retornará um número maior que 1. A condicional `return count > 0;` continua funcionando corretamente, pois ela só verifica se há pelo menos um registro que corresponde ao critério.

#### 3. **Como a condicional `return count > 0;` funciona e é correta?**

A condicional `return count > 0;` verifica se o valor de `count` é maior que zero. Se for, significa que existe pelo menos um registro no banco de dados com o email fornecido, e o método retorna `true`. Caso contrário, se `count` for igual a zero, significa que não há registros correspondentes, e o método retorna `false`.

