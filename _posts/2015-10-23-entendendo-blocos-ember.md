---
layout: post
title: Entendendo blocos do Ember - Handlebars
tags: [ember, javascript]
date: 2015-10-23
background: '/img/ember.jpg'
---

Componentes e helpers utilizados nas views do Ember podem receber blocos, como é o caso do `if`:

{% highlight html %}
{% raw %}
{{#if cliente.ativo}}
  <span class="ativo">Cliente {{cliente.nome}} está ativo</span>
{{else}}
  <button {{action "ativar"}}>
{{/if}}
{% endraw %}
{% endhighlight %}

O que delimita o início e fim de um blocos são os operadores `#` e `/` prefixados ao nome do componente.
Há casos de componentes que operam tanto com blocos como na versão sem blocos, como é o caso do `link-to`:

{% highlight html %}
{% raw %}
{{#link-to 'clientes'}}
  Lista de Clientes
{{/link-to}}

{{link-to 'Lista de Clientes' 'clientes'}}
{% endraw %}
{% endhighlight %}

Os dois formatos renderizam o seguinte HTML:

{% highlight html %}
<a href="/clientes">Lista de Clientes</a>
{% endhighlight %}

**Como blocos funcionam nos componentes ?**

Vamos fazer um pequeno componente para exemplificar:

{% highlight bash %}
ember generate component mostrar-cliente
{% endhighlight %}

Este comando gera os dois arquivos básicos para o funcionamento de um componente:

* O arquivo `.js` onde você programa o comportament
* O arquivo `.hbs` que é o template do componente
 
Vamos focar no segundo. Um novo template de componente possui apenas a instrução `yield`:

{% highlight html %}
<!-- components/mostrar-cliente/template.hbs -->
{% raw %}
{{yield}}
{% endraw %}
{% endhighlight %}

Esta instrução diz que seu componente pode receber um bloco, e que todo código do bloco será inserido neste trecho do `yield`. Confuso ? Exemplo:

{% highlight html %}
<!-- components/mostrar-cliente/template.hbs -->
{% raw %}
<h3>Minha lista de clientes:</h3>

{{yield}}

<ul>
  <li>João</li>
  <li>Maria</li>
</ul>
{% endraw %}
{% endhighlight %}

{% highlight html %}
<!-- clientes/template.hbs -->
{% raw %}
{{#mostrar-cliente}}
<p>Ordenado por nome</p>
{{/mostrar-cliente}}
{% endraw %}
{% endhighlight %}

Resulta no seguinte HTML:

{% highlight html %}
<h3>Minha lista de clientes:</h3>
<p>Ordenado por nome</p>
<ul>
  <li>João</li>
  <li>Maria</li>
</ul>
{% endhighlight %}

Repare que o `mostra-cliente` foi chamado com `#` e `/`, ou seja, foi passado um bloco ao componente, que foi renderizado no `yield`.

Mas nem sempre seu componente precisa aceitar blocos, portanto é seguro remover o `yield` e assim você pode chamar seu componente pela forma simplificada sem passar um bloco:

{% highlight html %}
{% raw %}
{{mostrar-cliente}}
{% endraw %}
{% endhighlight %}
