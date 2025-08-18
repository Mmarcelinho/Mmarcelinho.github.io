---
title: Eventos de Domínio | DDD do Jeito Certo
date: 2024-11-08
tags:
  - DDD
  - Events
  - Domain
  - DotNET
---
## O que são Eventos de Domínio?

- **Definição:**
  - Um evento de domínio registra algo significativo que ocorreu dentro de um subdomínio específico.
  - No contexto técnico, refere-se a uma mudança de estado no servidor.

## Função dos Eventos de Domínio

- **Comunicação entre Contextos:**
  - Permitem que um contexto delimitado informe outro sobre mudanças importantes.
  - São fundamentais para garantir que informações cruciais sejam compartilhadas entre diferentes partes do sistema.

## Identificação de Eventos de Domínio

- **Motivos para Mudança:**
  - Para identificar eventos de domínio, é necessário compreender os motivos das mudanças de estado das entidades.
  - Esses motivos devem ser expressos no código por meio de métodos específicos, evitando o uso direto de propriedades `get` e `set`.
  - **Exemplo:** Se uma entidade `Funcionário` tem um método que aumenta o salário, o evento correspondente seria "Salário Aumentado".

## Anatomia de um Evento de Domínio

- **Classe Simples:**
  - A classe que representa o evento deve ter um nome que descreva claramente o que aconteceu, sempre no passado.
  - **Propriedades Importantes:**
    - Data/hora do evento.
    - Detalhes relevantes, como valores anteriores e novos, para comparações.

## Encaminhamento e Persistência de Eventos

- **Conversão para Mensagem:**
  - Após a ocorrência de um evento, cria-se uma instância da classe de evento e a envia para um sistema de mensageria.
  - Em arquiteturas hexagonais, o evento é adaptado ao mecanismo de mensageria usado.
- **Persistência de Eventos:**
  - Os eventos podem ser armazenados em um banco de dados, formando um histórico das mudanças de estado das entidades, um conceito conhecido como event sourcing.

## Consumo de Eventos

- **Separação de Contextos:**
  - O contexto que produz um evento geralmente não deve ser o mesmo que o consome, para evitar complexidade desnecessária.
  - Eventos são particularmente úteis para comunicação entre diferentes contextos delimitados ou integração com outras partes do sistema.

## Aplicações dos Eventos de Domínio

- **Integração de Aplicações:**
  - Eventos de domínio são poderosos para integrar diferentes sistemas, permitindo consistência eventual.
- **Event Sourcing:**
  - Utilizados para persistência histórica, armazenando todas as mudanças de estado no sistema.

## Arquiteturas Baseadas em Eventos

- Sistemas que adotam arquiteturas orientadas a eventos (Event-Driven Architectures) dependem fortemente de eventos de domínio para seu funcionamento.

[Eventos de Domínio](https://youtu.be/_By3QRBMHSo?si=NRI6n9c746SIjvJW)

[[Specifications]]

