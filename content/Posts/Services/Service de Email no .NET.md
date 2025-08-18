---
title: Implementando o Envio de E-mails com MailKit em um Projeto .NET
date: 2024-11-30
tags:
  - CSharp
  - DotNET
  - "#Services"
---
Enviar e-mails é uma funcionalidade essencial em muitas aplicações, seja para confirmar registros, redefinir senhas ou enviar notificações. Neste post, vamos aprender como implementar o **MailKit** como um serviço de e-mail em um projeto .NET.

---

## 1. Instalando o MailKit

Como primeiro passo, iremos adicionar o pacote do MailKit ao nosso projeto:

```bash
dotnet add package MailKit
```

---

## 2. Configurando o `appsettings.json`

Em seguida, vamos modificar o `appsettings.json` para adicionar as configurações de e-mail que serão utilizadas pelo MailKit:

```json
{
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "SenderName": "Seu Nome",
    "SenderEmail": "seu-email@dominio.com",
    "UserName": "seu-email@dominio.com",
    "Password": "sua-senha"
  }
}
```

> **Nota:** Neste exemplo, estamos usando o **Gmail** como provedor, mas você pode utilizar qualquer outro provedor de e-mail de sua preferência.

---

## 3. Criando a Classe `EmailSettings`

Precisamos de uma classe para receber essas configurações:

```csharp
public class EmailSettings
{
    public string SmtpServer { get; set; } = string.Empty;
    public int Port { get; set; }
    public string SenderName { get; set; } = string.Empty;
    public string SenderEmail { get; set; } = string.Empty;
    public string UserName { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

---

## 4. Definindo a Interface `IEmailService`

Agora, vamos começar a construção do serviço. Primeiramente, precisamos de uma interface com um método `SendEmailAsync`:

```csharp
public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body, bool isHtml = true);
}
```

---

## 5. Implementando o Serviço de E-mail

Com a interface criada, vamos implementar o serviço:

```csharp
using MailKit.Net.Smtp;
using MimeKit;
using Microsoft.Extensions.Options;

public class EmailService : IEmailService
{
    private readonly EmailSettings _emailSettings;

    // Injeta as configurações tipadas via IOptions
    public EmailService(IOptions<EmailSettings> emailSettings)
    {
        _emailSettings = emailSettings.Value;
    }

    public async Task SendEmailAsync(string to, string subject, string body, bool isHtml = true)
    {
        // Cria a mensagem de e-mail
        var email = new MimeMessage();
        email.From.Add(new MailboxAddress(_emailSettings.SenderName, _emailSettings.SenderEmail));
        email.To.Add(new MailboxAddress(to, to));
        email.Subject = subject;

        // Define o corpo do e-mail como HTML ou texto simples
        if (isHtml)
            email.Body = new TextPart(MimeKit.Text.TextFormat.Html) { Text = body };
        else
            email.Body = new TextPart(MimeKit.Text.TextFormat.Text) { Text = body };

        // Envia o e-mail via SMTP
        using var smtp = new SmtpClient();
        try
        {
            await smtp.ConnectAsync(_emailSettings.SmtpServer, _emailSettings.Port, MailKit.Security.SecureSocketOptions.StartTls);
            await smtp.AuthenticateAsync(_emailSettings.UserName, _emailSettings.Password);
            await smtp.SendAsync(email);
        }
        catch (Exception ex)
        {
            // Trate o erro conforme necessário (log, rethrow, etc.)
            throw new InvalidOperationException("Erro ao enviar e-mail.", ex);
        }
        finally
        {
            await smtp.DisconnectAsync(true);
        }
    }
}
```

---

## 6. Registrando o Serviço no Container de Injeção de Dependência

Por fim, precisamos adicionar o serviço ao container de injeção de dependência:

```csharp
// Mapeia as configurações do EmailSettings
builder.Services.Configure<EmailSettings>(builder.Configuration.GetSection("EmailSettings"));

// Registra o EmailService
builder.Services.AddTransient<IEmailService, EmailService>();
```

---

## 7. Utilizando o Serviço de E-mail

Agora, você pode injetar o `IEmailService` em qualquer lugar da sua aplicação. Por exemplo, em um controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmailController : ControllerBase
{
    private readonly IEmailService _emailService;

    // Injeta o serviço de e-mail via construtor
    public EmailController(IEmailService emailService)
    {
        _emailService = emailService;
    }

    [HttpPost("send")]
    public async Task<IActionResult> SendEmail([FromBody] EmailRequest request)
    {
        await _emailService.SendEmailAsync(request.To, request.Subject, request.Body, request.IsHtml);
        return Ok(new { Message = "E-mail enviado com sucesso!" });
    }
}

public record EmailRequest(string To, string Subject, string Body, bool IsHtml);
```

Com essa abordagem, você conseguirá enviar e-mails via .NET.

---

## 8. Configurando a Senha de Aplicativo no Gmail

> **Importante:** Por motivos de segurança, a senha padrão da sua conta do Gmail não funcionará diretamente para autenticação SMTP. É necessário criar uma **senha de aplicativo** e atribuí-la no `appsettings.json`. Abaixo está um breve tutorial de como criar essa senha.

### Ative a Verificação em Duas Etapas

Antes de gerar uma senha de aplicativo, você precisa ativar a **verificação em duas etapas** na sua conta Google:

1. Acesse sua conta em [Minhas contas do Google](https://myaccount.google.com/).
2. No menu lateral, clique em **Segurança**.
3. Em **Como fazer login no Google**, clique em **Verificação em duas etapas**.
4. Siga as instruções para configurar a verificação em duas etapas.

### Gere uma Senha de Aplicativo

1. Após ativar a verificação em duas etapas, volte à página de **Segurança**.
2. Localize a seção **Senhas de app** (abaixo de **Como fazer login no Google**) e clique nela.
3. No menu **Selecionar o aplicativo e o dispositivo**:
    - Em **Aplicativo**, selecione **Outro (nome personalizado)**.
    - Insira um nome descritivo, como `SMTP-MailKit`, e clique em **Gerar**.
4. Copie a senha gerada pelo Google.

### Atualize o `appsettings.json` com a Senha de Aplicativo

Substitua o valor de `Password` no seu `appsettings.json` pela senha de aplicativo gerada:

```json
{
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "SenderName": "Seu Nome",
    "SenderEmail": "seu-email@dominio.com",
    "UserName": "seu-email@dominio.com",
    "Password": "sua-senha-de-aplicativo"
  }
}
```

---

## 9. Conclusão

Com essas etapas, você configurou com sucesso o **MailKit** como um serviço de e-mail em sua aplicação .NET, seguindo as melhores práticas de desenvolvimento. Lembre-se sempre de proteger suas credenciais e seguir as políticas de segurança dos provedores de e-mail.

Agora você pode enviar e-mails de forma eficiente e segura em sua aplicação .NET!
