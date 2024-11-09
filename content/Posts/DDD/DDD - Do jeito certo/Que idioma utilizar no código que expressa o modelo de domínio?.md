---
title: Que idioma utilizar no código que expressa o modelo de domínio? | DDD do jeito certo
date: 2024-11-08
tags:
  - DDD
  - DotNET
---
### Considerações sobre o Idioma no Código de Domínio

1. **Linguagem Ubíqua vs. Idioma:**
   - A linguagem ubíqua não está vinculada a um idioma específico, mas sim a conceitos. O idioma utilizado para representar esses conceitos é uma escolha de representação, e não uma característica intrínseca da linguagem ubíqua.
   - Assim como diferentes tecnologias podem expor um mesmo recurso de formas distintas (JSON, XML), a linguagem ubíqua pode ser expressa em diferentes idiomas sem perder a essência dos conceitos.

2. **Fluência e Complexidade:**
   - É essencial que a equipe de desenvolvimento tenha fluência suficiente no idioma escolhido para expressar a linguagem ubíqua. A falta de proficiência pode levar a erros de tradução e a complexidades desnecessárias.
   - A escolha de um idioma diferente daquele falado pelo especialista de domínio ou pelo time de desenvolvimento pode adicionar custos e complexidade, tornando-se difícil de justificar se não houver um motivo claro.

3. **Casos Especiais:**
   - Em situações onde o especialista de domínio fala um idioma completamente desconhecido pelo time de desenvolvimento, é preciso encontrar uma solução viável. Por exemplo, pode-se escolher um idioma intermediário (como o inglês) que seja compreendido por ambas as partes, mesmo que não seja a língua nativa de ninguém.
   - A tradução literal de termos que não possuem equivalentes diretos pode gerar aberrações no código. É recomendável utilizar o termo no idioma original nesses casos.

4. **Custos e Benefícios:**
   - Optar por um idioma diferente do falado pela equipe ou pelos especialistas de domínio deve ser uma decisão ponderada. Se a equipe não tem fluência suficiente no segundo idioma, os custos de comunicação e entendimento podem superar os benefícios.
   - A escolha de escrever código em inglês é comum em equipes internacionalizadas, mas deve-se avaliar se essa decisão realmente agrega valor ou se apenas adiciona dificuldades.

5. **Pragmatismo:**
   - A decisão sobre o idioma no código deve ser pragmática. Se o time de desenvolvimento e os especialistas de domínio têm maior fluência em um determinado idioma, esse deve ser o escolhido, desde que facilite a comunicação e a expressão dos conceitos de forma clara e eficaz.
   - A utilização de ferramentas como a Wikipédia para traduzir termos específicos é sugerida, ao invés de confiar em traduções automáticas, que podem não capturar o significado correto.

### Conclusão

- **Escolha Consciente:** A decisão de qual idioma utilizar no código deve ser baseada na fluência da equipe e na clareza na expressão dos conceitos. Adicionar um idioma que não seja o mais natural para a equipe ou para os especialistas de domínio pode ser uma escolha desnecessária que só aumenta o custo do projeto.
- **Flexibilidade:** Não há problema técnico em optar por um idioma diferente, desde que a equipe esteja ciente dos custos envolvidos e preparada para lidar com as implicações dessa escolha.

[Que idioma utilizar no código que expressa o modelo de domínio?](https://youtu.be/uh_iLnsjKyg?si=AHD8ogS8-qkSM3vb)

[[Serviços de Domínio]]
