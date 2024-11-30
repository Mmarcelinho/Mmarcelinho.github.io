---
title: Protegendo e mantendo suas senhas seguras com BCrypt
date: 2024-06-29
tags:
  - CSharp
  - DotNET
  - Security
  - Password
  - Criptografia
---

No post de hoje, aprenderemos como proteger e manter suas senhas seguras utilizando a biblioteca `BCrypt`. Quando temos senhas salvas em um banco de dados, é essencial criptografá-las, pois são dados sensíveis que, em caso de vazamento, podem ser comprometidos.

## O que é BCrypt?

BCrypt é uma biblioteca de hashing de senhas para aplicações .NET. Ela utiliza o algoritmo bcrypt, projetado para ser seguro contra ataques de força bruta, dificultando a adivinhação de senhas pelos atacantes. A biblioteca oferece funções para gerar hashes de senhas e verificar se uma senha corresponde a um determinado hash, sendo amplamente utilizada para armazenar senhas de maneira segura em bancos de dados.

### Interface 

Primeiro, precisamos definir uma interface com os métodos que serão implementados no serviço de criptografia de senhas:

```csharp
public interface IEncriptadorDeSenha
{
    string Encriptar(string senha);
    bool Verificar(string senha, string senhaEncriptografada);
}
```

A interface requer dois métodos:

- **Encriptar:** Recebe a senha como uma string e a criptografa antes de ser salva no banco de dados.
- **Verificar:** Método booleano que verifica se a senha fornecida corresponde à senha criptografada armazenada no banco de dados.

Antes de implementar esses métodos, vamos adicionar a biblioteca necessária ao nosso projeto:

```csharp
dotnet add package BCrypt.Net-Next
```

Agora, vamos à implementação.

## Implementação do Encriptador de Senha

Primeiro, implementamos a interface:

```csharp
internal class EncriptadorDeSenha : IEncriptadorDeSenha
```

### Método Encriptar

Vamos começar com o método `Encriptar`, que é bastante simples:

```csharp
public string Encriptar(string senha)
{
    string senhaEncriptografada = BCrypt.Net.BCrypt.HashPassword(senha);
    return senhaEncriptografada;
}
```

Este método recebe a senha como parâmetro, criptografa utilizando o método `HashPassword()` da biblioteca `BCrypt`, e retorna a senha criptografada para ser salva no banco de dados.

### Método Verificar

Agora, vamos implementar o método de verificação, que também é essencial:

```csharp
public bool Verificar(string senha, string senhaEncriptografada) => BCrypt.Net.BCrypt.Verify(senha, senhaEncriptografada);
```

Este método recebe a senha digitada pelo usuário e a senha criptografada armazenada no banco de dados. Utilizando o método `Verify` da biblioteca `BCrypt`, ele compara as duas senhas e retorna `True` se forem iguais, e `False` se não forem.

### Adicionando o Serviço

Para utilizar o encriptador, precisamos adicionar o serviço no container de injeção de dependência:

```csharp
services.AddScoped<IEncriptadorDeSenha, EncriptadorDeSenha>();
```

Podemos também criar um alias para encurtar as chamadas da biblioteca:

```csharp
using BC = BCrypt.Net.BCrypt;
```

### Código Final

Aqui está o código completo da implementação:

```csharp
internal class EncriptadorDeSenha : IEncriptadorDeSenha
{
    public string Encriptar(string senha)
    {
        string senhaEncriptografada = BC.HashPassword(senha);
        return senhaEncriptografada;
    }

    public bool Verificar(string senha, string senhaEncriptografada) => BC.Verify(senha, senhaEncriptografada);
}
```

Agora você está pronto para proteger e manter suas senhas seguras utilizando `BCrypt`!

## Exemplos Práticos

Vamos ver exemplos práticos de como utilizar o serviço de encriptação de senha com BCrypt.

### Exemplo 1: Criando um Usuário

Neste exemplo, mostraremos como encriptamos a senha do `Usuário`, utilizando o serviço de encriptação de senha.

```csharp
private readonly IEncriptadorDeSenha _encriptadorDeSenha;

public RegistrarAdminCommandHandler(IEncriptadorDeSenha encriptadorDeSenha)
{
    _encriptadorDeSenha = encriptadorDeSenha;
}

public void RegistrarAdmin(RegistrarAdminRequest request)
{
    var admin = new Admin
    {
        Nome = request.Nome,
        Email = request.Email,
        Senha = _encriptadorDeSenha.Encriptar(request.Senha),
        Telefone = request.Telefone,
        IdentificadorUsuario = Guid.NewGuid(),
        Role = Roles.ADMIN
    };

    // Persistir o usuário no banco de dados
}
```

Neste trecho de código, utilizamos o método `Encriptar` do serviço `IEncriptadorDeSenha` para criptografar a senha antes de armazená-la no banco de dados.

### Exemplo de Senha Encriptografada

Após executar `Encriptar`, a senha é transformada em um hash criptografado, como o exemplo a seguir:

```
$2a$12$h8xn8aE9nEr/EZNCuJme1.tE0MdlB8F5W6Wc4J7c3B36H6zIxFbZq
```

Este hash é composto por:

- `$2a$`: Identificador do algoritmo bcrypt.
- `12$`: Fator de custo, indicando quantas iterações de hashing foram realizadas (2^12 no caso).
- `h8xn8aE9nEr/EZNCuJme1.`: O salt gerado aleatoriamente.
- `tE0MdlB8F5W6Wc4J7c3B36H6zIxFbZq`: O hash resultante da senha.

### Exemplo 2: Verificando a Senha no Login

No exemplo a seguir, mostramos como verificar a senha durante o processo de login do `Usuário`:

```csharp
private readonly IEncriptadorDeSenha _encriptadorDeSenha;

public LoginAdminCommandHandler(IEncriptadorDeSenha encriptadorDeSenha)
{
    _encriptadorDeSenha = encriptadorDeSenha;
}

public async Task<RespostaLoginAdminJson> Handle(LoginAdminCommand request, CancellationToken cancellationToken)
{
    var admin = await _adminReadOnlyRepositorio.RecuperarPorEmail(request.Email);

    if (admin == null)
        throw new LoginInvalidoException();

    var senhaCorreta = _encriptadorDeSenha.Verificar(request.Senha, admin.Senha);

    if (!senhaCorreta)
        throw new LoginInvalidoException();

    return new RespostaLoginAdminJson(
        admin.Nome,
        _geradorTokenAcesso.Gerar(admin)
    );
}
```

Neste código, utilizamos o método `Verificar` do serviço `IEncriptadorDeSenha` para comparar a senha fornecida durante o login com a senha criptografada armazenada no banco de dados. Se as senhas coincidirem, o login é autorizado; caso contrário, uma exceção de login inválido é lançada.

### Considerações Finais

Ao utilizar BCrypt para criptografar senhas, garantimos um nível adicional de segurança ao armazenar informações sensíveis como senhas de usuários em bancos de dados.

