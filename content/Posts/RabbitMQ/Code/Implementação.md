# Implementação de Mensageria com RabbitMQ no VacationSystem

Este documento descreve a implementação de mensageria assíncrona usando RabbitMQ no VacationSystem, seguindo boas práticas e respeitando a arquitetura existente do projeto.

## 1. Estrutura de Arquivos

```
src/VacationSystem.Application/
├── Infrastructure/
│   └── Services/
│       └── Messaging/
│           ├── IRabbitMQService.cs      // Interface do serviço RabbitMQ
│           ├── RabbitMQService.cs       // Implementação do serviço RabbitMQ
│           └── MessagingConstants.cs    // Constantes para exchanges, filas e routing keys
├── Domain/
│   └── Vacation/
│       └── Events/
│           ├── VacationRequestCreatedEvent.cs   // Evento de solicitação criada
│           └── VacationRequestReviewedEvent.cs  // Evento de solicitação revisada
└── Features/
    └── VacationRequests/
        ├── EventHandlers/
        │   ├── VacationRequestCreatedEventHandler.cs  // Handler para publicar mensagens
        │   └── VacationRequestReviewedEventHandler.cs // Handler para publicar mensagens 
        └── Services/
            └── VacationNotificationService.cs  // Serviço para consumir mensagens
```

## 2. Dependências

Adicione a dependência do RabbitMQ ao arquivo de projeto:

```xml
<ItemGroup>
  <!-- Outras dependências... -->
  <PackageReference Include="RabbitMQ.Client" Version="6.8.1" />
</ItemGroup>
```

## 3. Interface do Serviço RabbitMQ

```csharp
namespace VacationSystem.Application.Infrastructure.Services.Messaging;

public interface IRabbitMQService
{
    void PublishMessage<T>(string exchange, string routingKey, T message) where T : class;
    void Subscribe<T>(string queue, string exchange, string routingKey, Action<T> onMessage) where T : class;
}
```

## 4. Implementação do Serviço RabbitMQ

```csharp
using System.Text;
using System.Text.Json;
using Microsoft.Extensions.Logging;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

namespace VacationSystem.Application.Infrastructure.Services.Messaging;

public class RabbitMQService : IRabbitMQService, IDisposable
{
    private readonly IConnection _connection;
    private readonly IModel _channel;
    private readonly ILogger<RabbitMQService> _logger;

    public RabbitMQService(string hostName, int port, string userName, string password, ILogger<RabbitMQService> logger)
    {
        _logger = logger;
        
        try
        {
            var factory = new ConnectionFactory
            {
                HostName = hostName,
                Port = port,
                UserName = userName,
                Password = password
            };
            
            _connection = factory.CreateConnection();
            _channel = _connection.CreateModel();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to establish connection with RabbitMQ");
            throw;
        }
    }

    public void PublishMessage<T>(string exchange, string routingKey, T message) where T : class
    {
        try
        {
            _channel.ExchangeDeclare(exchange, ExchangeType.Direct, durable: true);
            
            var json = JsonSerializer.Serialize(message);
            var body = Encoding.UTF8.GetBytes(json);
            
            _channel.BasicPublish(
                exchange: exchange,
                routingKey: routingKey,
                basicProperties: null,
                body: body);
            
            _logger.LogInformation("Message published to {Exchange} with routing key {RoutingKey}", exchange, routingKey);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to publish message to {Exchange} with routing key {RoutingKey}", exchange, routingKey);
            throw;
        }
    }

    public void Subscribe<T>(string queue, string exchange, string routingKey, Action<T> onMessage) where T : class
    {
        try
        {
            _channel.ExchangeDeclare(exchange, ExchangeType.Direct, durable: true);
            _channel.QueueDeclare(queue, durable: true, exclusive: false, autoDelete: false);
            _channel.QueueBind(queue, exchange, routingKey);
            
            var consumer = new EventingBasicConsumer(_channel);
            
            consumer.Received += (_, ea) =>
            {
                try
                {
                    var body = ea.Body.ToArray();
                    var message = Encoding.UTF8.GetString(body);
                    var deserializedMessage = JsonSerializer.Deserialize<T>(message);
                    
                    if (deserializedMessage != null)
                    {
                        onMessage(deserializedMessage);
                        _channel.BasicAck(ea.DeliveryTag, multiple: false);
                    }
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Error processing message from queue {Queue}", queue);
                    _channel.BasicNack(ea.DeliveryTag, multiple: false, requeue: true);
                }
            };
            
            _channel.BasicConsume(queue, autoAck: false, consumer);
            
            _logger.LogInformation("Subscribed to {Queue} bound to {Exchange} with routing key {RoutingKey}", 
                queue, exchange, routingKey);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to subscribe to {Queue} bound to {Exchange} with routing key {RoutingKey}", 
                queue, exchange, routingKey);
            throw;
        }
    }

    public void Dispose()
    {
        _channel?.Dispose();
        _connection?.Dispose();
    }
}
```

## 5. Constantes para Mensageria

```csharp
namespace VacationSystem.Application.Infrastructure.Services.Messaging;

public static class MessagingConstants
{
    // Exchanges
    public const string VacationExchange = "vacation_exchange";
    
    // Routing Keys
    public const string VacationRequestCreatedRoutingKey = "vacation.request.created";
    public const string VacationRequestApprovedRoutingKey = "vacation.request.approved";
    public const string VacationRequestDeniedRoutingKey = "vacation.request.denied";
    
    // Queues
    public const string VacationRequestCreatedQueue = "vacation_request_created_queue";
    public const string VacationRequestApprovedQueue = "vacation_request_approved_queue";
    public const string VacationRequestDeniedQueue = "vacation_request_denied_queue";
}
```

## 6. Eventos de Domínio

### Evento de Solicitação de Férias Criada

```csharp
using VacationSystem.Application.Common.Abstractions;

namespace VacationSystem.Application.Domain.Vacation.Events;

public class VacationRequestCreatedEvent : IDomainEvent
{
    public VacationRequestCreatedEvent(
        Guid vacationRequestId, 
        DateTime requestDate, 
        DateTime startDate, 
        DateTime endDate, 
        int days, 
        Guid employeeId)
    {
        VacationRequestId = vacationRequestId;
        RequestDate = requestDate;
        StartDate = startDate;
        EndDate = endDate;
        Days = days;
        EmployeeId = employeeId;
    }

    public Guid VacationRequestId { get; }
    public DateTime RequestDate { get; }
    public DateTime StartDate { get; }
    public DateTime EndDate { get; }
    public int Days { get; }
    public Guid EmployeeId { get; }
}
```

### Evento de Solicitação de Férias Revisada

```csharp
using VacationSystem.Application.Common.Abstractions;
using VacationSystem.Application.Domain.Vacation.Enums;

namespace VacationSystem.Application.Domain.Vacation.Events;

public class VacationRequestReviewedEvent : IDomainEvent
{
    public VacationRequestReviewedEvent(
        Guid vacationRequestId, 
        EStatus status, 
        Guid adminId)
    {
        VacationRequestId = vacationRequestId;
        Status = status;
        AdminId = adminId;
    }

    public Guid VacationRequestId { get; }
    public EStatus Status { get; }
    public Guid AdminId { get; }
}
```

## 7. Manipuladores de Eventos (Event Handlers)

### Handler para Solicitação de Férias Criada

```csharp
using MediatR;
using Microsoft.Extensions.Logging;
using VacationSystem.Application.Domain.Vacation.Events;
using VacationSystem.Application.Infrastructure.Services.Messaging;

namespace VacationSystem.Application.Features.VacationRequests.EventHandlers;

public class VacationRequestCreatedEventHandler : INotificationHandler<VacationRequestCreatedEvent>
{
    private readonly IRabbitMQService _rabbitMQService;
    private readonly ILogger<VacationRequestCreatedEventHandler> _logger;

    public VacationRequestCreatedEventHandler(
        IRabbitMQService rabbitMQService,
        ILogger<VacationRequestCreatedEventHandler> logger)
    {
        _rabbitMQService = rabbitMQService;
        _logger = logger;
    }

    public Task Handle(VacationRequestCreatedEvent notification, CancellationToken cancellationToken)
    {
        try
        {
            _logger.LogInformation("Publicando evento de solicitação de férias criada: {VacationRequestId}", 
                notification.VacationRequestId);
            
            _rabbitMQService.PublishMessage(
                MessagingConstants.VacationExchange,
                MessagingConstants.VacationRequestCreatedRoutingKey,
                notification);
                
            return Task.CompletedTask;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Erro ao publicar evento de solicitação de férias criada: {VacationRequestId}", 
                notification.VacationRequestId);
            throw;
        }
    }
}
```

### Handler para Solicitação de Férias Revisada

```csharp
using MediatR;
using Microsoft.Extensions.Logging;
using VacationSystem.Application.Domain.Vacation.Enums;
using VacationSystem.Application.Domain.Vacation.Events;
using VacationSystem.Application.Infrastructure.Services.Messaging;

namespace VacationSystem.Application.Features.VacationRequests.EventHandlers;

public class VacationRequestReviewedEventHandler : INotificationHandler<VacationRequestReviewedEvent>
{
    private readonly IRabbitMQService _rabbitMQService;
    private readonly ILogger<VacationRequestReviewedEventHandler> _logger;

    public VacationRequestReviewedEventHandler(
        IRabbitMQService rabbitMQService,
        ILogger<VacationRequestReviewedEventHandler> logger)
    {
        _rabbitMQService = rabbitMQService;
        _logger = logger;
    }

    public Task Handle(VacationRequestReviewedEvent notification, CancellationToken cancellationToken)
    {
        try
        {
            var routingKey = notification.Status == EStatus.Approved
                ? MessagingConstants.VacationRequestApprovedRoutingKey
                : MessagingConstants.VacationRequestDeniedRoutingKey;

            _logger.LogInformation(
                "Publicando evento de solicitação de férias {Status}: {VacationRequestId}",
                notification.Status,
                notification.VacationRequestId);
                
            _rabbitMQService.PublishMessage(
                MessagingConstants.VacationExchange,
                routingKey,
                notification);
                
            return Task.CompletedTask;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, 
                "Erro ao publicar evento de solicitação de férias {Status}: {VacationRequestId}",
                notification.Status,
                notification.VacationRequestId);
            throw;
        }
    }
}
```

## 8. Serviço de Notificação (Consumidor)

```csharp
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using VacationSystem.Application.Domain.Vacation.Events;
using VacationSystem.Application.Infrastructure.Services.Messaging;

namespace VacationSystem.Application.Features.VacationRequests.Services;

public class VacationNotificationService : BackgroundService
{
    private readonly IRabbitMQService _rabbitMQService;
    private readonly ILogger<VacationNotificationService> _logger;

    public VacationNotificationService(
        IRabbitMQService rabbitMQService,
        ILogger<VacationNotificationService> logger)
    {
        _rabbitMQService = rabbitMQService;
        _logger = logger;
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        SetupSubscriptions();
        return Task.CompletedTask;
    }

    private void SetupSubscriptions()
    {
        // Inscrição para solicitações de férias criadas
        _rabbitMQService.Subscribe<VacationRequestCreatedEvent>(
            MessagingConstants.VacationRequestCreatedQueue,
            MessagingConstants.VacationExchange, 
            MessagingConstants.VacationRequestCreatedRoutingKey,
            HandleVacationRequestCreated);

        // Inscrição para solicitações de férias aprovadas
        _rabbitMQService.Subscribe<VacationRequestReviewedEvent>(
            MessagingConstants.VacationRequestApprovedQueue,
            MessagingConstants.VacationExchange,
            MessagingConstants.VacationRequestApprovedRoutingKey,
            HandleVacationRequestApproved);

        // Inscrição para solicitações de férias negadas
        _rabbitMQService.Subscribe<VacationRequestReviewedEvent>(
            MessagingConstants.VacationRequestDeniedQueue,
            MessagingConstants.VacationExchange,
            MessagingConstants.VacationRequestDeniedRoutingKey,
            HandleVacationRequestDenied);
            
        _logger.LogInformation("Todas as assinaturas para notificações de férias foram configuradas");
    }

    private void HandleVacationRequestCreated(VacationRequestCreatedEvent @event)
    {
        // Em um cenário real, aqui seria implementado o envio de e-mail para notificar o admin
        // sobre uma nova solicitação de férias
        _logger.LogInformation(
            "Nova solicitação de férias criada: {VacationRequestId} - Funcionário: {EmployeeId} - " +
            "Período: {StartDate:dd/MM/yyyy} a {EndDate:dd/MM/yyyy} ({Days} dias)",
            @event.VacationRequestId, 
            @event.EmployeeId,
            @event.StartDate,
            @event.EndDate,
            @event.Days);
    }

    private void HandleVacationRequestApproved(VacationRequestReviewedEvent @event)
    {
        // Em um cenário real, aqui seria implementado o envio de e-mail para notificar o funcionário
        // que sua solicitação de férias foi aprovada
        _logger.LogInformation(
            "Solicitação de férias aprovada: {VacationRequestId} - Aprovada pelo Admin: {AdminId}",
            @event.VacationRequestId,
            @event.AdminId);
    }

    private void HandleVacationRequestDenied(VacationRequestReviewedEvent @event)
    {
        // Em um cenário real, aqui seria implementado o envio de e-mail para notificar o funcionário
        // que sua solicitação de férias foi negada
        _logger.LogInformation(
            "Solicitação de férias negada: {VacationRequestId} - Negada pelo Admin: {AdminId}",
            @event.VacationRequestId,
            @event.AdminId);
    }
}
```

## 9. Modificação da Entidade VacationRequest

Adicione a geração de eventos na entidade `VacationRequest`:

```csharp
public static VacationRequest Create(DateTime startDate, int days, Employee.Employee employee)
{
    ValidateHoliday(employee);
    var request = new VacationRequest(
        requestDate: DateTime.Today,
        startDate: startDate,
        endDate: startDate.AddDays(days),
        days: days,
        status: EStatus.Pending,
        employeeId: employee.Id,
        employee: employee,
        adminId: null,
        admin: null);
    
    // Adicionar o evento de domínio
    request.AddDomainEvent(new VacationRequestCreatedEvent(
        request.Id,
        request.RequestDate,
        request.StartDate,
        request.EndDate,
        request.Days,
        request.EmployeeId));
        
    return request;
}

public void ReviewVacationRequest(Admin.Admin admin, EStatus status)
{
    ValidateStatus();

    if (status == EStatus.Approved)
        Approve(admin);
    else
        Reject(admin);
        
    // Adicionar o evento de domínio
    AddDomainEvent(new VacationRequestReviewedEvent(
        Id,
        Status,
        admin.Id));
}
```

## 10. Configuração de Serviços (DI)

Adicione o seguinte método à classe `DependencyInjection`:

```csharp
private static void AddMessaging(IServiceCollection services, IConfiguration configuration)
{
    // Configuração do RabbitMQ
    var rabbitMQConfig = configuration.GetSection("RabbitMQ");
    var hostName = rabbitMQConfig["HostName"] ?? "localhost";
    var port = int.Parse(rabbitMQConfig["Port"] ?? "5672");
    var userName = rabbitMQConfig["UserName"] ?? "guest";
    var password = rabbitMQConfig["Password"] ?? "guest";
    
    // Registrando o serviço RabbitMQ como singleton para manter a conexão viva
    services.AddSingleton<IRabbitMQService>(provider => 
        new RabbitMQService(
            hostName, 
            port, 
            userName, 
            password, 
            provider.GetRequiredService<ILogger<RabbitMQService>>()));
            
    // Registrando o serviço de notificação de férias
    services.AddHostedService<VacationNotificationService>();
}
```

E chame-o a partir de `AddInfrastructure`:

```csharp
public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration configuration)
{
    // Código existente...
    
    AddPasswordEncrpter(services);
    AddTokens(services, configuration);
    AddLoggedUser(services);
    AddMessaging(services, configuration); // Adicione esta linha
    
    return services;
}
```

## 11. Configuração no appsettings.json

Adicione as configurações do RabbitMQ ao arquivo `appsettings.json`:

```json
{
  "RabbitMQ": {
    "HostName": "localhost",
    "Port": 5672,
    "UserName": "guest",
    "Password": "guest"
  }
}
```

## Benefícios da Implementação

1. **Desacoplamento**: Produtores e consumidores de mensagens estão desacoplados.
2. **Escalabilidade**: Operações assíncronas permitem melhor escalabilidade.
3. **Resiliência**: Mensagens persistidas proporcionam maior resiliência contra falhas.
4. **Manutenibilidade**: Código modular e fácil de manter.
5. **Flexibilidade**: Fácil adição de novos consumidores para diversos casos de uso.

## Considerações de Projeto

1. A implementação segue o padrão de eventos de domínio existente no projeto.
2. A injeção de dependência é usada para fornecer o serviço RabbitMQ onde necessário.
3. O serviço RabbitMQ é registrado como singleton para manter uma única conexão viva.
4. O serviço de notificação é implementado como um BackgroundService para consumir mensagens em segundo plano.

Esta implementação pode ser expandida para outros casos de uso no sistema, seguindo o mesmo padrão.
