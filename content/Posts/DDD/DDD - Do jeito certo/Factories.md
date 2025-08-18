---
title: Factories | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - Factories
  - DotNET
---
## Factories - O que são e quando utilizar? 

- **Simplificação na criação de agregados**: A instanciação de agregados deve ser uma operação simples. O uso de construtores é comum em linguagens como C# e Java, mas há cenários onde o construtor não é ideal, especialmente quando a lógica de validação de negócio é complexa. É aqui que as Factories entram em cena, facilitando a criação de objetos de maneira clara.

- **Uso de Factories**: As Factories oferecem uma interface para criar objetos sem expor os detalhes internos de implementação, permitindo maior flexibilidade e clareza.

---

### Padrões Relacionados:

- **Gangue dos Quatro (GoF)**: No livro clássico de 1994 sobre padrões de projeto, já havia preocupação em simplificar a criação de objetos. Três padrões específicos são destacados:
  1. **Abstract Factory**
  2. **Factory Method**
  3. **Builder**

- **Eric Evans (DDD)**: Evans trouxe essas ideias para o DDD, utilizando padrões Factory para encapsular a criação de objetos complexos, garantindo que regras de negócio sejam respeitadas.

---

### Cenários Comuns para o Uso de Factories

1. **Fluxos de inicialização distintos (Múltiplos parâmetros)**:
   - Quando um tipo pode ser inicializado com diferentes conjuntos de parâmetros, os construtores tradicionais podem gerar sobrecargas difíceis de gerenciar. Um Factory Method pode organizar esses fluxos de forma clara.
   
   **Exemplo**:
   - Imagine a classe `Timestamp` que pode ser criada com horas, minutos, segundos, etc. No lugar de várias sobrecargas de construtores, você pode usar métodos estáticos como `FromHours`, `FromMinutes`, e `FromSeconds`, que tornam o código mais legível:

```csharp
var timestampFromHours = Timestamp.FromHours(5);
    var timestampFromMinutes = Timestamp.FromMinutes(300); // Mais claro que apenas 'new Timestamp(300)'
```
 
   - Esse tipo de implementação facilita a compreensão do código e evita ambiguidade que poderia ocorrer com construtores sobrecarregados.

---

2. **Hierarquias de tipos (Abstract Factory)**:
   - Quando diferentes parâmetros levam à criação de diferentes tipos concretos, o padrão **Abstract Factory** se torna útil.
   
   **Exemplo**:
   - Suponha que você está implementando um sistema de regras de aposentadoria, onde as leis variam de acordo com a data de contratação do funcionário. Você pode ter várias classes concretas que implementam diferentes regras de aposentadoria, e uma Abstract Factory decide qual classe concreta usar com base nas datas:

     ```csharp
     public interface IAposentadoria
     {
         bool PodeAposentar(Funcionario funcionario);
     }

     public class AposentadoriaLeiAntiga : IAposentadoria
     {
         public bool PodeAposentar(Funcionario funcionario)
         {
             // Lógica para regras de aposentadoria antiga
         }
     }

     public class AposentadoriaLeiNova : IAposentadoria
     {
         public bool PodeAposentar(Funcionario funcionario)
         {
             // Lógica para regras de aposentadoria nova
         }
     }

     public class AposentadoriaFactory
     {
         public IAposentadoria CriarAposentadoria(Funcionario funcionario)
         {
             if (funcionario.DataContratacao < new DateTime(1995, 01, 01))
                 return new AposentadoriaLeiAntiga();
             else
                 return new AposentadoriaLeiNova();
         }
     }
     ```

   - Aqui, a `AposentadoriaFactory` escolhe a classe concreta com base na data de contratação, garantindo que o funcionário siga as regras adequadas.

---

3. **Lógica Complexa de Inicialização (Builder)**:
   - Quando a criação de um objeto envolve lógica complexa, a Factory ajuda a encapsular essa complexidade, possibilitando que o código cliente não precise lidar com todos os detalhes.
   
   **Exemplo**:
   - Suponha que você está desenvolvendo um sistema de empréstimos, e o valor do empréstimo precisa ser distribuído em várias parcelas. Em vez de dividir o valor igualmente, pode haver regras de arredondamento onde o valor quebrado vai para a última parcela. Essa lógica pode ser complexa e fica melhor encapsulada em um Builder:

     ```csharp
     public class Parcela
     {
         public decimal Valor { get; private set; }
         public Parcela(decimal valor)
         {
             Valor = valor;
         }
     }

     public class Emprestimo
     {
         public List<Parcela> Parcelas { get; private set; }

         private Emprestimo(List<Parcela> parcelas)
         {
             Parcelas = parcelas;
         }

         public static class EmprestimoBuilder
         {
             public static Emprestimo Criar(decimal valorTotal, int numeroParcelas)
             {
                 var parcelas = new List<Parcela>();
                 decimal valorParcela = Math.Floor(valorTotal / numeroParcelas);
                 decimal valorUltimaParcela = valorParcela + (valorTotal % numeroParcelas);

                 for (int i = 0; i < numeroParcelas - 1; i++)
                 {
                     parcelas.Add(new Parcela(valorParcela));
                 }
                 parcelas.Add(new Parcela(valorUltimaParcela));

                 return new Emprestimo(parcelas);
             }
         }
     }
     ```

   - A lógica para distribuir o valor e aplicar o arredondamento fica encapsulada no Builder, facilitando a modificação no futuro, caso as regras de negócio mudem.

---

### Vantagens das Factories:

- **Flexibilidade**: Quando a lógica de criação de um objeto muda, apenas a Factory precisa ser alterada, sem necessidade de mexer no código cliente.
- **Testabilidade**: Ao encapsular a lógica de criação em uma Factory, fica mais fácil testar a criação de objetos em diferentes cenários, simulando diferentes regras de negócio.
- **Desacoplamento**: Factories reduzem o acoplamento entre o código que cria o objeto e o objeto em si, mantendo a criação organizada e permitindo que o restante do sistema não precise conhecer os detalhes da implementação.

---

### Considerações Finais:

- Eric Evans considera as Factories um padrão secundário, mas elas desempenham um papel importante em DDD quando há regras de negócio complexas envolvidas na criação de objetos.
- **Factories** podem ser fundamentais para deixar o código mais claro e expressivo, além de reduzir o acoplamento e tornar a lógica de criação de objetos mais flexível e testável.

[Factories](https://youtu.be/vGhtcdP5gsI?si=YBM97wuqrZzXXXsN)

[[Módulos]]
