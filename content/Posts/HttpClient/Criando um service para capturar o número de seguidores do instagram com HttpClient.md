---
title: Criando um service para capturar o número de seguidores do instagram com HttpClient
date: 2024-09-11
tags:
  - CSharp
  - DotNET
  - HttpClient
  - Scraper
---
Neste tutorial, vamos aprender a criar um `Service` que utiliza `HttpClient` para capturar a contagem de seguidores de um perfil público do Instagram. 

### 1. Criando a Interface

Primeiro, criamos a interface do `Service`. Isso permite que o `Service` seja adicionado ao container de injeção de dependência.

```csharp
public interface IInstagramScraper
{
    Task<string> GetFollowerCountAsync(string instagramHandle);
}
```

Essa interface define um método `GetFollowerCountAsync`, que recebe o `instagramHandle` (nome de usuário do Instagram) e retorna uma `string` contendo a quantidade de seguidores.

### 2. Implementando o `Service`

Seguindo o padrão da arquitetura limpa, implementamos serviços externos na camada de infraestrutura. Primeiro, criamos a classe `InstagramScraper` que implementa a interface `IInstagramScraper`.

```csharp
public class InstagramScraper : IInstagramScraper
{
    private readonly HttpClient _httpClient;
    
    public InstagramScraper(HttpClient httpClient)
    {
        _httpClient = httpClient;
        ConfigureHttpClient();
    }
}
```

#### Injeção de Dependência do `HttpClient`

A classe `InstagramScraper` tem uma variável `readonly` do tipo `HttpClient` para gerenciar as requisições. Recebemos a instância do `HttpClient` no construtor, facilitando a injeção de dependência.

### 3. Configurando o `HttpClient`

Configuramos o `HttpClient` para enviar cabeçalhos específicos que ajudam a imitar um navegador, garantindo que o Instagram não bloqueie a requisição.

```csharp
private void ConfigureHttpClient()
{
    _httpClient.DefaultRequestHeaders.UserAgent.ParseAdd("Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3");
    _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("text/html"));
    _httpClient.DefaultRequestHeaders.AcceptLanguage.ParseAdd("en-US,en;q=0.5");
}
```

Essa configuração adiciona cabeçalhos `User-Agent`, `Accept` e `Accept-Language`, simulando um navegador comum para evitar bloqueios do site.

### 4. Implementando o Método de Scraping

Agora, implementamos o método `GetFollowerCountAsync` na classe `InstagramScraper`. Esse método é responsável por acessar a página do perfil, buscar o HTML e extrair a contagem de seguidores.

#### Passo 1: Construindo a URL do Perfil do Instagram

```csharp
public async Task<string> GetFollowerCountAsync(string instagramHandle)
{
    var url = $"https://www.instagram.com/{instagramHandle}/";
```

Usamos interpolação de strings para construir a URL do perfil a partir do `instagramHandle` fornecido.

#### Passo 2: Fazendo a Requisição HTTP

```csharp
    var response = await _httpClient.GetStringAsync(url);
```

Usando o `HttpClient`, enviamos uma requisição HTTP para a página do perfil e aguardamos o HTML da resposta. Esse método é assíncrono (`await`) para não bloquear a execução enquanto a resposta é carregada.

#### Passo 3: Verificando a Resposta

```csharp
    if (string.IsNullOrEmpty(response))
        throw new Exception("The page response is empty or null.");
```

Caso a `response` seja nula ou vazia, lançamos uma exceção indicando que a página não foi carregada corretamente. Isso ajuda a evitar erros ao tentar processar um HTML inexistente.

#### Passo 4: Capturando a Contagem de Seguidores com Expressão Regular

Para extrair a contagem de seguidores, usamos uma expressão regular que busca no HTML o valor esperado no meta tag `og:description`.

```csharp
    var followersRegex = new Regex(@"<meta property=""og:description"" content=""(\d+(?:[,.]\d+)?[KM]?) Followers");
    var match = followersRegex.Match(response);
```

A expressão regular captura um padrão específico, permitindo números formatados como `5K`, `10.5M` ou `1000`. O grupo de captura `(\d+(?:[,.]\d+)?[KM]?)` lida com vírgulas, pontos e as letras `K` e `M` para representar milhares e milhões.

#### Passo 5: Verificando se a Contagem foi Encontrada

```csharp
    if (!match.Success)
        throw new Exception("Unable to find the follower count.");
```

Se não houver correspondência (`match.Success` é `false`), lançamos uma exceção informando que a contagem de seguidores não foi encontrada. Esse tratamento de erro é importante, pois o HTML pode variar, e a estrutura da página do Instagram pode mudar.

#### Passo 6: Retornando a Contagem de Seguidores

```csharp
    return match.Groups[1].Value;
}
```

Finalmente, retornamos o valor do grupo de captura `Groups[1].Value`, que contém a contagem de seguidores no formato capturado do HTML.

### 5. Adicionando o `Service` ao Container de Dependência

Para tornar o `Service` disponível por injeção de dependência, adicionamos a configuração ao container de serviços.

```csharp
private static void AddScraper(IServiceCollection services)
    => services.AddHttpClient<IInstagramScraper, InstagramScraper>();
```

No método de configuração de serviços:

```csharp
public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration configuration)
{
    AddScraper(services);
    
    // Outros métodos...
    
    return services;
}
```

### 6. Exemplo de Uso do `Service`

Aqui está um exemplo de como o `Service` pode ser utilizado em uma `Handler` para processar uma consulta que retorna informações sobre influenciadores:

```csharp
public record GetInfluencersQuery : IRequest<IEnumerable<ResponseInfluencerProfileJson>>;

public class GetInfluencersQueryHandler(
    IInfluencerReadOnlyRepository repository,
    IInstagramScraper instagramScraper) : IRequestHandler<GetInfluencersQuery, IEnumerable<ResponseInfluencerProfileJson>>
{
    public async Task<IEnumerable<ResponseInfluencerProfileJson>> Handle(GetInfluencersQuery request, CancellationToken cancellationToken)
    {
        var influencers = await repository.GetInfluencers();

        var responseTasks = influencers.Select(async influencer =>
        {
            try
            {
                var instagram = await instagramScraper.GetFollowerCountAsync(influencer.Instagram);
                return new ResponseInfluencerProfileJson(
                    influencer.Username.Value,
                    influencer.Instagram,
                    instagram);
            }
            catch (Exception ex)
            {
                return new ResponseInfluencerProfileJson(
                    influencer.Username.Value,
                    influencer.Instagram,
                    ex.Message);
            }
        });

        var responses = await Task.WhenAll(responseTasks);
        return responses;
    }
}
```

Esse exemplo demonstra como o `InstagramScraper` pode ser usado em um `Handler` para obter informações de seguidores e compilar um perfil de resposta, com tratamento de exceção para eventuais falhas na captura de dados.
