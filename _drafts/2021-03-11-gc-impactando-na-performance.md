---
layout: post
title:  ".NET - Melhore o desempenho da sua aplicação aliviando a pressão do Garbage Collection"
tags: [peformance, .NET, garbage-collection]
categories: [Dicas de peformance, .NET]
pin: false
date: 2021-03-11 02:25:00 -0300
---

Neste artigo eu explico como obtive um desempenho significativo reduzindo a alocação de mémoria na área dinâmica (mais conhecida como heap) evitando a carga de acionamento excessivo sobre o GC e aliviando sua pressão de execução.

Mas antes de prosseguir com a leitura meu caro leitor, preciso deixar claro que este artigo é apenas para demonstração de como eu obtive um resultado bacana evitando o acionamento do GC e que o código melhorado não significa que ele seja 100% ruím, até por que existem outros meios de resolver este problema. Resumindo, quando o assunto é peformance não necessariamente o problema estará no código, outros pontos precisam ser elevados em consideração como por exemplo: IO, Rede,

``` cs


// Após medição e ajustes para ganho de desempenho.

Worker Cycle GC:
-----------------
 Gen0: 0
 Gen1: 0
 Gen2: 0
Time spent on execution: 4493 ms
Allocated memory: 27 MB
```


## Resultado obtido após ajustes.

``` cs
  decimal x = 10m;
```
