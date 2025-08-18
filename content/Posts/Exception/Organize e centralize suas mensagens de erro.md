---
title: Organize e centralize suas mensagens de erro
date: 2024-05-20
tags:
  - "#CSharp"
  - DotNET
  - Exceptions
  - ErrorMessages
  - CleanCode
---
Criar mensagens de erro e exceções padronizadas ajuda a manter um código mais organizado e facilita a manutenção. Neste post, vou demonstrar como criar essas mensagens e como reutilizá-las em todo o código, evitando a duplicação de strings e centralizando as mensagens de erro em um único local.

#### Onde Colocar as Mensagens de Erro

Em meus projetos, geralmente crio esses modelos no projeto `Exceptions`, onde configuro o tratamento de exceções.

#### Criando o modelo de mensagens de erro

1. **Criação da Classe:**
   Primeiro, crie uma classe com o nome que representa a entidade para a qual você deseja criar essas mensagens:

   ```csharp
   namespace SistemaDeEstoque.Exceptions.ErrorMessages;

   public static class UsuarioModelMensagensDeErro
   { }
   ```

   Esta classe precisa ser estática, pois não queremos criar uma instância dela em todos os lugares onde necessitamos utilizar essas mensagens.

2. **Definição das Mensagens:**
   Agora, vamos criar as mensagens de erro:

   ```csharp
   public static string EMAIL_JA_REGISTRADO = "O e-mail informado já está registrado na base de dados.";
   ```

   - **Estática:** A variável precisa ser estática, indicando que pertence à classe e não a uma instância da classe.
   - **Tipo String:** A variável deve ser do tipo string.
   - **Nomeação:** Nomeie as variáveis em maiúsculas e com underscores, seguindo a convenção para constantes.

#### Código completo do modelo

```csharp
namespace SistemaDeEstoque.Exceptions.ErrorMessages;

public static class UsuarioModelMensagensDeErro
{
    public static string EMAIL_JA_REGISTRADO = "O e-mail informado já está registrado na base de dados.";
    public static string EMAIL_USUARIO_EMBRANCO = "O e-mail do usuário deve ser informado.";
    public static string EMAIL_USUARIO_INVALIDO = "O e-mail do usuário é inválido.";
    public static string LOGIN_INVALIDO = "O e-mail e/ou senha estão incorretos.";
    public static string NOME_USUARIO_EMBRANCO = "O nome do usuário deve ser informado.";
    public static string SENHA_ATUAL_INVALIDA = "Senha atual é inválida.";
    public static string SENHA_USUARIO_EMBRANCO = "A senha do usuário deve ser informada.";
    public static string SENHA_USUARIO_MINIMO_SEIS_CARACTERES = "A senha deve conter no mínimo 6 caracteres.";
    public static string TELEFONE_USUARIO_EMBRANCO = "O telefone do usuário deve ser informado.";
    public static string TELEFONE_USUARIO_INVALIDO = "O telefone do usuário deve estar no formato XX X XXXX-XXXX.";
    public static string TOKEN_EXPIRADO = "Faça login novamente no App.";
    public static string ADMINISTRADOR_NAO_ENCONTRADO = "Administrador não encontrado.";
}
```

#### Exemplos de utilização

1. **Validação:**

   ```csharp
   private async Task Validar(RequisicaoRegistrarAdminJson requisicao)
   {
       var validator = new RegistrarAdminValidator();
       var resultado = validator.Validate(requisicao);

       var existeAdminComEmail = await _adminReadOnlyRepositorio.ExisteAdminComEmail(requisicao.Email);
       if (existeAdminComEmail)
           resultado.Errors.Add(new FluentValidation.Results.ValidationFailure("email", UsuarioModelMensagensDeErro.EMAIL_JA_REGISTRADO));

       if (!resultado.IsValid)
       {
           var mensagensDeErro = resultado.Errors.Select(error => error.ErrorMessage).ToList();
           throw new ErrosDeValidacaoException(mensagensDeErro);
       }
   }
   ```

   Observe como a classe estática e a variável são chamadas diretamente, facilitando a compreensão do código. Caso queira alterar a mensagem de erro, não precisa se preocupar em modificar em todos os lugares onde a variável é chamada, pois ela está centralizada na classe estática.

2. **Validação de Senha:**

   ```csharp
   public SenhaValidator()
   {
       RuleFor(senha => senha).NotEmpty().WithMessage(UsuarioModelMensagensDeErro.SENHA_USUARIO_EMBRANCO);
       When(senha => !string.IsNullOrWhiteSpace(senha), () => 
       {
           RuleFor(senha => senha.Length)
               .GreaterThanOrEqualTo(6)
               .WithMessage(UsuarioModelMensagensDeErro.SENHA_USUARIO_MINIMO_SEIS_CARACTERES);
       });
   }
   ```

Essas mensagens podem ser utilizadas em qualquer lugar que espera uma string como parâmetro, mantendo a reutilização de código e a consistência nas mensagens de erro.

### Conclusão

Centralizar e padronizar suas mensagens de erro não só melhora a organização do código, mas também facilita a manutenção e a compreensão do sistema. Utilize classes estáticas para armazenar essas mensagens e reutilize-as em todo o seu projeto para uma abordagem mais eficiente e limpa.