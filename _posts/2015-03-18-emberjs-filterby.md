---
layout: post
title: Ember.js - Filter By
tags: [ember, javascript]
date: 2015-03-18
background: '/img/ember.jpg'
---

O Ember provê muitos métodos úteis para manipulação de coleção dados (Arrays), você pode ver a lista na documentação do [Enumerable](http://emberjs.com/api/classes/Ember.Enumerable.html).

Neste post vou explicar o funcionamento do método [filterBy](http://emberjs.com/api/classes/Ember.Enumerable.html#method_filterBy), que é utilizado para filtrar um array pela chave.

Imagine que você tenha que filtrar todos os alunos que tiraram nota 10, com `filterBy` isso é simples:

{% highlight js %}
var alunos = [
  {id: 1, nome: "João", nota: 9},
  {id: 2, nome: "José", nota: 10},
  {id: 3, nome: "Maria", nota: 10},
  {id: 4, nome: "Pedro", nota: 5}
];

// filterBy(key, value)
var bonsAlunos = alunos.filterBy('nota', 10);

// Resultado
/*
[
  {
    "id": 2,
    "nome": "José",
    "nota": 10
  },
  {
    "id": 3,
    "nome": "Maria",
    "nota": 10
  }
]
*/
{% endhighlight %}

Internamente o `filterBy` vai fazer um loop em cada item do array e verificar se a `key` 'nota' é igual ao `value` 10. Para cada item que a condição for verdadeira, ele vai retornar no array `bonsAlunos`.

Abaixo deixo um exemplo prático de como utilizar o filterBy para criar um tela com filtro:

<a class="jsbin-embed" href="http://emberjs.jsbin.com/goboca/4/embed?html,js,output">JS Bin</a>
