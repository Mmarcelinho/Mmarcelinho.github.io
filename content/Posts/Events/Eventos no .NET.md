---
title: Eventos no .NET
date: 2024-03-30
tags:
  - CSharp
  - DotNET
  - Events
---

Os Eventos  no  .NET são baseados no modelo de representante. O modelo de representante segue o padrão de design do observador, que permite a um assinante se registrar em um provedor e receber notificações dele. 

Um evento é uma mensagem enviada por um objeto para sinalizar a ocorrência de uma ação ou mudança de estado, como por exemplo, se você tem uma classe que representa uma tarefa, você pode ter um evento que é disparado toda vez que uma tarefa é concluída.

Será abordado abaixo como os Eventos funcionam em código para esclarecer melhor.

## Código

-  **Declarando um evento**:

Para declarar um evento, é utilizado a palavra-chave `event`, seguida por um tipo de delegate que define a assinatura dos métodos que podem responder ao evento. `EventHandler<TEventArgs>` é um delegate comum usado para eventos que podem passar informações adicionais aos ouvintes.

```csharp
public event EventHandler<MudancaTemperaturaEventArgs> EventoDeMudancaDeTemperatura;
```

- **Classe para os argumentos do evento**

Se o evento precisa passar informações extras, você cria uma classe que herda de `EventArgs`. Essa classe define o que será passado como informação para aos "ouvintes" do evento.

```csharp
public class MudancaTemperaturaEventArgs : EventArgs
    {
        public double TemperaturaAnterior { get; }

        public double TemperaturaAtual { get; }

        public MudancaTemperaturaEventArgs(double temperaturaAnterior, double temperaturaAtual)
        {
            TemperaturaAnterior = temperaturaAnterior;
            TemperaturaAtual = temperaturaAtual;
        }
    }
```

- **Notificando os assinantes do evento**

Para notificar os assinantes, é definido um método `protected virutal` que dispara o evento utilizando o `Invoke` 

```csharp
protected virtual void AoMudarTemperatura(MudancaTemperaturaEventArgs e)
        {
            EventoDeMudancaDeTemperatura?.Invoke(this, e);
        }
```

É utilizado o operador `?` para garantir que o evento só será invocado se houver pelo menos um assinante inscrito, para evitar exceções de referência nula. E após essa verificação o evento é chamado com `this` (referência ao objeto atual) e os argumentos do evento.

- **Gerando o evento**

O evento pode ser gerado a partir de qualquer método da classe que queira implementá-lo, criando uma instância da classe de argumentos do evento e chamando o método que notifica os assinantes.

```csharp
public void AtualizarTemperatura(double novaTemperatura)
        {
            if(Math.Abs(novaTemperatura - _temperaturaAtual) >= 0.1)
            {
                MudancaTemperaturaEventArgs args = new(_temperaturaAtual, novaTemperatura);

                _temperaturaAtual = novaTemperatura;
                AoMudarTemperatura(args);
            }
        }
```

Nesse código é verificado se houve uma mudança de temperatura maior que >= 0.1 comparando a temperatura antiga com a atual. Assim é criado a instância da classe de argumentos do evento que recebe os parâmetros e é chamado o método que notifica os assinantes que recebe essa instancia.

- **Ouvindo o evento**

As classes que desejam ser "notificadas" sobre a mudança de estado da classe onde o evento é declarado, podem se inscrever para ele e definir um método que corresponda à assinatura do delegado do evento. Este método será chamado sempre que o evento for acionado.

```csharp
public void InscreverNoEventoDeMudancaDeTemperatura(Termometro termometro)
        {
            termometro.EventoDeMudancaDeTemperatura += AlertarMudancaDeTemperatura;
        }

        private void AlertarMudancaDeTemperatura(object sender, MudancaTemperaturaEventArgs e)
        {
            Console.WriteLine($"Alerta! Temperatura mudou de {e.TemperaturaAnterior} para {e.TemperaturaAtual} graus.");
        }
```

- **Testando o código**

```csharp
public class Program
{
    static void Main()
    {
        Termometro termometro = new(25.0);

        SistemaDeAlerta sistemaDeAlerta = new();

        sistemaDeAlerta.InscreverNoEventoDeMudancaDeTemperatura(termometro);

        Console.WriteLine("Atualizando temperaturas:");
        termometro.AtualizarTemperatura(25.05);
        termometro.AtualizarTemperatura(25.2);
        termometro.AtualizarTemperatura(25.1);
        termometro.AtualizarTemperatura(24.0);
    }
}
```

1. A temperatura inicial é definido como 25 graus
2. O sistema de alerta é inscrito para "ouvir" as mudanças de temperaturas
3. É simulado a mudança de temperatura

- **Saída**

```csharp
Alerta! Temperatura mudou de 25 para 25.2 graus.
Alerta! Temperatura mudou de 25.2 para 24 graus.
```

Após esse exemplo de código implementando os eventos, abaixo seram citados brevemente algumas boa práticas e considerações que são importantes saber para que tenha uma compreensão mais completa de como trabalhar com eventos em C#.

- **Desinscrição de Eventos**

Desinscrever-se de eventos é crucial para evitar vazamentos de memória, especialmente em cenários onde o assinante tem um ciclo de vida mais curto que o publicador. Aqui um exemplo de como se desinscrever de um evento:

```csharp
static void Main()
    {
        Termometro termometro = new(25.0);

        SistemaDeAlerta sistemaDeAlerta = new();

        //Inscrição no evento
	sistemaDeAlerta.InscreverNoEventoDeMudancaDeTemperatura(termometro);

        Console.WriteLine("Atualizando temperaturas:");
        termometro.AtualizarTemperatura(25.05);
        termometro.AtualizarTemperatura(25.2);
        termometro.AtualizarTemperatura(25.1);
        termometro.AtualizarTemperatura(24.0);

        //Desinscição no evento
sistemaDeAlerta.DesinscreverNoEventoDeMudancaDeTemperatura(termometro);
    }
```

### 2. Uso de `EventHandler` vs. Delegates personalizados

`EventHandler` e `EventHandler<TEventArgs>` são suficientes para a maioria dos eventos. No entanto, se precisar de uma assinatura de evento que não se encaixe no padrão `(object sender, TEventArgs e)`, um delegate personalizado pode ser necessário.

Exemplo com `EventHandler`:

```csharp
public event EventHandler<MudancaTemperaturaEventArgs> EventoDeMudancaDeTemperatura;
```

Exemplo usando um delegate personalizado:

```csharp
public delegate void MudancaTemperaturaDelegate(object sender, double temperaturaNova, double temperaturaAntiga);

public event MudancaTemperaturaDelegate EventoDeMudancaDeTemperaturaPersonalizado;
```

-  **Thread Safety**

Eventos padrão em C# não são thread-safe. Aqui está um exemplo para garantir a thread safety ao invocar um evento:

```csharp
private readonly object lockObject = new object();

protected virtual void AoMudarTemperatura(MudancaTemperaturaEventArgs e)
{
    EventHandler<MudancaTemperaturaEventArgs> handler;
    lock (lockObject)
    {
        handler = EventoDeMudancaDeTemperatura;
    }
    
    handler?.Invoke(this, e);
}
```

## **Boas práticas**

- **Evitar operações de longa duração nos manipuladores de eventos**
 
Isso pode bloquear o thread que dispara o evento, causando lentidão na aplicação.

```csharp
private void AlertarMudancaDeTemperatura(object sender, MudancaTemperaturaEventArgs e)
{
    // Realizar operações rápidas ou delegar tarefas longas a outro thread
    Console.WriteLine($"Temperatura mudou de {e.TemperaturaAnterior} para {e.TemperaturaAtual} graus.");
}
```

- **Documentação de eventos**
 
É importante documentar o que cada evento representa e quando é disparado, para facilitar o uso correto pelos desenvolvedores.

```csharp
/// <summary>
/// Evento disparado quando a temperatura muda em mais de 0.1 graus.
/// </summary>
public event EventHandler<MudancaTemperaturaEventArgs> EventoDeMudancaDeTemperatura;
```


## Projeto

- [Events](https://github.com/Mmarcelinho/events_csharp)

### Referência

- [Eventos, Delegados, Cadeia de Delegados por exemplo C#](https://www.codeproject.com/Articles/1077216/Csharp-Lectures-Lecture-Events-Delegates-Delegat)
- [Master C# Events Like a Senior Developer: Real-World Examples in .NET!](https://youtu.be/9H7PU-cy0Sw?si=VaBC9FAw-fuTPe4U)
- [Microsoft - Manipular e gerar eventos](https://learn.microsoft.com/pt-br/dotnet/standard/events/)
- [Events e delegates | C# Avançado | Notificações, Func, Action, Predicate e Anônimos](https://www.youtube.com/watch?v=SuW2GwO17qA)
