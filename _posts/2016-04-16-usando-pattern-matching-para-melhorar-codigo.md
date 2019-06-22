---
layout: post
title: Elixir - Usando Pattern Matching para melhorar seu código
tags: [elixir, functional-programming]
date: 2016-04-16
background: '/img/pattern.jpg'
---

Pattern Matching é o tipo de recurso que se leva um tempo para aprender e perceber o seu real valor. Felizmente temos [vários](http://elixir-lang.org/getting-started/pattern-matching.html) [materiais](https://elixirschool.com/pt/lessons/basics/pattern-matching/) e [artigos](http://philipsampaio.com.br/blog/2015/01/08/10-exemplos-de-pattern-matching-em-elixir) que nos ajudam nesta tarefa. Se você não tem ideia do que é Pattern Matching recomendo ler os links antes de continuar.

Neste artigo vou mostrar uma forma de usar Pattern Matching que me fez perceber como este recurso é poderoso e elegante. Vamos começar com um exemplo de código simples.

Quem nunca escreveu um código assim ?

{% highlight ruby %}
cliente = Cliente.new(id: 1000, cartao_fidelidade: true, email: 'cliente@pizzaria.com')
desconto_pedido = desconto(produto: pizza, cliente: cliente)

# Aplica 5% de desconto no valor do produto caso o cliente possua o cartão fidelidade.
# Caso não possua, envia email de marketing para oferecer o cartão fidelidade.
# Retorna o valor calculado do desconto
def desconto(produto:, cliente:)
    if cliente.cartao_fidelidade
        desconto = calcular_valor_desconto(produto, 5)
    else
        desconto = 0
        if cliente.email
            enviar_marketing_fidelidade(cliente.email)
        end
    end
    desconto
end
{% endhighlight %}

À primeira vista é um código inofensivo, mas este projeto irá crescer e o excesso de condicionais pode se tornar um problema na manutenção e evolução da base de código. Imagine que no futuro o sistema precise calcular desconto no primeiro pedido do cliente, ou que além de email deve enviar SMS, ou ainda aplicar desconto progressivo no desconto... acho que ficou claro onde mora o perigo.

Você já sabe que Pattern Matching, como o próprio nome diz, serve para casar padrões. Então como podemos reescrever nosso método `desconto` para que fique mais legível e compreensível ? Pattern Matching!

{% highlight elixir %}
cliente = %{id: 1000, cartao_fidelidade: true, email: 'cliente@pizzaria.com'}
pedido = desconto(pizza, cliente)

def desconto(produto, %{cartao_fidelidade: false, email: nil}) do
    0
end

def desconto(produto, %{cartao_fidelidade: false, email: email_cliente}) do
    enviar_marketing_fidelidade(email_cliente)
    0
end

def desconto(produto, %{cartao_fidelidade: true}) do
    calcular_valor_desconto(produto, 5)
end
{% endhighlight %}

O [map](http://elixir-lang.org/getting-started/keywords-and-maps.html#maps) `cliente` possui as chaves `cartao_fidelidade` e `email`, que serão usadas para casar com os parâmetros de cada função:

{% highlight elixir %}
# Qual das funções casa/combina com este cliente ?
cliente = %{cartao_fidelidade: false, email: nil}
{% endhighlight %}

A grande diferença entre as duas versões é que usando Pattern Matching seu código se torna explícito adotando lógicas isoladas para cada condição, este é um efeito colateral típico de linguagens funcionais.

No primeiro caso a complexidade é maior pois temos que lidar com o estado dos atributos do objeto `cliente`, porém o que importa para o processamento da lógica são os dados, e eles devem ser explícitos. 
