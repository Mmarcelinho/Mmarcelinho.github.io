---
title: ViaCep - Recuperando endereço por cep
date: 2024-06-25
tags:
  - CSharp
  - DotNET
  - HttpClient
---
O ViaCep é um WebService gratuito para consulta de CEP do Brasil, e é bastante útil pois retorna o endereço sempre de forma padronizada.

### Exemplo de Retorno

```json
{
    "cep": "01001-000",
    "logradouro": "Praça da Sé",
    "complemento": "lado ímpar",
    "bairro": "Sé",
    "localidade": "São Paulo",
    "uf": "SP",
    "ibge": "3550308",
    "gia": "1004",
    "ddd": "11",
    "siafi": "7107"
}
```

Neste post, irei demonstrar como implementei o consumo dessa API em meu sistema de cadastro para preencher automaticamente o endereço a partir do CEP.

## Interface do Serviço

Primeiro, é necessário criar a interface do serviço.

```csharp
public interface IViaCep
{
    bool ValidarCep(string cep);
    Task<EnderecoJson> RecuperarEndereco(string cep);
}
```

Esta interface define dois métodos que compõem esse serviço:

- **ValidarCep**: Importante para validar o CEP recebido da requisição.
- **RecuperarEndereco**: É uma `Task` que recebe uma string `cep` e retorna um objeto `EnderecoJson`. Este método possui toda a lógica de consumo da API.

### Definição do Objeto `EnderecoJson`

O objeto `EnderecoJson` é um record utilizado para mapear o retorno da consulta da API.

```csharp
public record EnderecoJson(
    [property: JsonPropertyName("cep")] string Cep,
    [property: JsonPropertyName("logradouro")] string Logradouro,
    [property: JsonPropertyName("complemento")] string Complemento,
    [property: JsonPropertyName("bairro")] string Bairro,
    [property: JsonPropertyName("localidade")] string Localidade,
    [property: JsonPropertyName("uf")] string Uf,
    [property: JsonPropertyName("ibge")] string Ibge,
    [property: JsonPropertyName("gia")] string Gia,
    [property: JsonPropertyName("ddd")] string Ddd,
    [property: JsonPropertyName("siafi")] string Siafi);
```

O atributo `JsonPropertyName` é usado para especificar o nome da propriedade JSON que deve ser mapeada para o campo ou propriedade correspondente na classe ou record.

## Implementação do Serviço

Ao criar a classe, precisamos implementar a interface `IViaCep`.

```csharp
public class ViaCep : IViaCep
{
    public bool ValidarCep(string cep) => !string.IsNullOrEmpty(cep) && cep.Length == 8;

    public async Task<EnderecoJson> RecuperarEndereco(string cep)
    {
        if (!ValidarCep(cep))
            throw new Exception("CEP inválido.");

        using var client = new HttpClient();
        var resposta = await client.GetAsync($"https://viacep.com.br/ws/{cep}/json/");
        resposta.EnsureSuccessStatusCode();

        var content = await resposta.Content.ReadAsStringAsync();
        var resultado = JsonSerializer.Deserialize<EnderecoJson>(content);

        return resultado;
    }
}
```

### Detalhes da Implementação

#### Método `ValidarCep`

Este método verifica se o CEP não é nulo e possui exatamente 8 caracteres.

```csharp
public bool ValidarCep(string cep) => !string.IsNullOrEmpty(cep) && cep.Length == 8;
```

O método retorna `true` se as duas condições forem atendidas: a string não for nula e seu tamanho for igual a 8.

#### Método `RecuperarEndereco`

Dentro do método `RecuperarEndereco`, validamos o CEP antes de prosseguir com a requisição.

```csharp
if (!ValidarCep(cep))
    throw new Exception("CEP inválido.");
```

Agora criamos uma instância de `HttpClient`:

```csharp
using var client = new HttpClient();
```

A classe `HttpClient` é usada para enviar solicitações HTTP e receber respostas de recursos web. Utilizamos `using` para garantir que o objeto `HttpClient` seja corretamente descartado (liberado da memória) assim que ele não for mais necessário.

Fazemos a requisição:

```csharp
var resposta = await client.GetAsync($"https://viacep.com.br/ws/{cep}/json/");
```

Perceba que a string `cep`, recebida via parâmetro, está interpolada na URL da requisição. Utilizamos `GetAsync`, que envia uma requisição HTTP GET para o URI especificado.

Tratamos a resposta:

```csharp
var content = await resposta.Content.ReadAsStringAsync();
```

Lemos o conteúdo da resposta HTTP como uma string.

Com o conteúdo lido, transformamos o JSON em objeto:

```csharp
var resultado = JsonSerializer.Deserialize<EnderecoJson>(content);
```

Utilizamos a classe `JsonSerializer` para desserializar a string JSON em um objeto do tipo `EnderecoJson`.

Por fim, retornamos esse objeto:

```csharp
return resultado;
```

Com esta implementação, você pode consumir o serviço ViaCep em seu sistema de cadastro para preencher automaticamente o endereço a partir do CEP fornecido. 

## Exemplo de Uso

Primeiro, temos que adicionar o serviço no container de injeção de dependência.

```csharp
services.AddScoped<IViaCep, ViaCep>();
```

Em seguida, injetamos o serviço no caso de uso:

```csharp
private readonly IViaCep _viaCep;

public RegistrarPessoaUseCase(IViaCep viaCep)
{
    _viaCep = viaCep;
    // Código adicional...
}
```

Agora o serviço está pronto para ser utilizado. Veja um exemplo de uso dentro de um método que preenche endereços automaticamente a partir de uma requisição:

```csharp
private async Task<List<Domicilio>> CepServices(RequisicaoPessoaJson requisicao)
{
    List<Domicilio> domicilios = new List<Domicilio>();

    foreach (var domicilio in requisicao.Domicilios)
    {
        var resultado = await _viaCep.RecuperarEndereco(domicilio.Endereco.Cep);

        Endereco endereco = new(
            domicilio.Endereco.Cep,
            resultado.Logradouro,
            domicilio.Endereco.Numero,
            resultado.Bairro,
            domicilio.Endereco.Complemento,
            domicilio.Endereco.PontoReferencia,
            resultado.Uf,
            resultado.Localidade,
            resultado.Ibge);

        Domicilio novoDomicilio = new()
        {
            Tipo = (DomicilioTipo)domicilio.Tipo,
            Endereco = endereco
        };

        domicilios.Add(novoDomicilio);
    }
    return domicilios;
}
```

Com essa abordagem, você garante que o endereço será preenchido automaticamente, melhorando a eficiência e precisão no cadastro de endereços em seu sistema.