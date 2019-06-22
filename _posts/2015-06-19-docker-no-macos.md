---
layout: post
title: Como rodar Docker no MacOS
tags: [docker, macos]
date: 2015-06-19
background: '/img/containers.jpg'
---

[Docker](https://www.docker.com/) é uma ferramenta para rodar aplicações em ambientes isolados, nos chamados Containers. É desta forma que empresas como Google, Twitter e muitas outras disponibilizam suas aplicações para milhões de usuários. E você também pode se beneficiar desta arquitetura.

O único sistema operacional suportado atualmente é o Linux, porém há formas de rodar o Docker no Mac com praticamente a mesma facilidade e performance, como veremos neste artigo.

**TL/DR**

{% highlight bash %}

# Instale os gerenciadores brew, cask e também o git, caso ainda não tenha:
ruby -e "$(curl -fsSL
https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew install caskroom/cask/brew-cask

brew install git

#Instale o VirtualBox
brew cask install virtualbox

# Instale o Vagrant
brew cask install vagrant

# Instale o Docker
brew install docker

# Clone o projeto com o sistema pronto
git clone https://github.com/leandrocp/coreos-vagrant.git

cd coreos-vagrant

# Inicialize o Vagrant
vagrant up

# Aponte o host do docker
# Dica: coloque este comando no arquivo ~/.bashrc
export DOCKER_HOST=127.0.0.1:2375

# Docker rodando!
docker info

docker run -p 8080:80 nginx
{% endhighlight %}

Acesse http://localhost:8080/ para provar que o Docker está ativo e você acabou de executar um servidor web.

**Mas como funciona ?**

A solução é simples: virtualizar um linux que possua docker, apontar o docker local para este ambiente, compartilhar arquivos e finalmente usar o Docker.

O projeto [boot2docker](http://boot2docker.io/) facilita todo esse processo, porém a performance dos arquivos compartilhados é sofrível. Por isso usamos o [CoreOS](https://coreos.com/), um sistema enxuto criado com o objetivo de rodar containers.

Você ainda pode customizar os parâmetros do CoreOS, para isso edite o arquivo [config.rb](https://github.com/leandrocp/coreos-vagrant/blob/master/config.rb) que está todo explicado com comentários.
