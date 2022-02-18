---
title: "Como eu criei este site usando Hugo e Github Pages"
date: 2022-02-17T21:02:16-03:00
authors: ["porfaxyz"]
categories:
  - porfaxyz
tags:
  - hugo
  - github
  - github-pages
slug: como-eu-criei-este-site-usando-hugo-e-github-pages
---

## O que é o Hugo
[Hugo](https://gohugo.io) é um gerador de site estático escrito em [Go](https://go.dev) que pode ser hospedado em qualquer lugar inclusive no [Github Pages](https://pages.github.com/) que é a hospedagem deste site.

## Instalação do Hugo
A instalação do Hugo é bem simples no meu caso usei o Homebrew.

```bash
brew install hugo
```

Uma vez instalado podemos executar o comando ***hugo version*** para verificar se está tudo ok.

```bash
hugo version
```

Para detalhes sobre outras formas de instalar, consulte a seção [instalação](https://gohugo.io/getting-started/installing/) na documentação do Hugo

## Criando o site
Agora podemos criar o nosso site e para isso devemos executar o comando abaixo

```bash
hugo new site porfa.xyz
```

Esse comando cria o diretório conforme o parâmetro informado, que no meu caso foi ***porfa.xyz***.
 A saída do comando exibe algumas instruções de como adicionar um tema e criar o seu primeiro conteúdo.

Nesse estágio já temos o nosso site em branco sem nenhum conteúdo e sem um tema, mas vamos resolver isso.

## Adicionar um tema
O tema que eu escolhi para o meu site pessoal foi o [Minimo](https://minimo.netlify.app/), um tema minimalista e simples, a ideia é dar foco ao conteúdo e não criar distração.

Adicionar um tema é bem simples, podemos clonar ou adicionar como submódulo do repositório do tema escolhido na pasta ***themes*** do nosso projeto.

Nesse momento precisamos inicializar um repositório git executando o comando abaixo.

```bash
git init
```

Como o repositório inicializado vamos adicionar um submódulo.

```bash
git submodule add https://github.com/porfaxyz/minimo themes/minimo
```

Para testar o tema e validar que está tudo certo, vamos usar o site de exemplo disponibilizado pelo próprio tema, para isso precisamos sincronizar o conteúdo da pasta ***themes/minimo/exampleSite*** na raiz do projeto.

```bash
rsync -a  themes/minimo/exampleSite/ ./
```

Agora podemos executar o projeto e validar se está tudo certo.

```bash
hugo server
```

Podemos abrir o browser e acessar *http://localhost:1313* e se tudo deu certo você verá uma página igual a imagem abaixo.

## Configuração do tema
Chegou a hora de dar a nossa cara para o site e para isso vamos editar o arquivo de configuração config.toml

Abaixo deixei apenas as chaves que eu alterei no arquivo de configuração e omiti as demais chaves. 

```ini
baseURL = "http://porfa.xyz"
title = "Porfa"

defaultContentLanguage = "pt-br"
 
disqusShortname = "porfaxyz"
 
[params.info]
description = "404, Bio not found"
 
[params.copyright]
prefix = ""
holder = "Porfa -"
startYear = "2022"
suffix = "Powered by Github Pages, Hugo and Minimo."

[params.social]
email = "porfaxyz@gmail.com"
github = "porfaxyz"
gitlab = "porfaxyz"
instagram = "porfaxyz"
twitter = "porfaxyz"
youtube = "UCBrKT_aELMXWbWfUcgw63dg"
 
[params.comments]
enable = true

[params.settings]
accentColor = "#65c8fe"
 
[languages]
[languages.pt-br]
lang = "pt-BR"
languageName = "Portuguese Brazilian"
weight = 1
```

## Publicando no Github Pages
Chegou a hora de publicar o site usando o Github Pages, um recurso do Github que nos permite publicar um site estático.

Para essa tarefa iremos usar outro recurso bacana o [Github Actions](https://docs.github.com/actions), um serviço de CI que nos permite automatizar o processo de integração e publicação do nosso site.

Vamos criar o diretório onde vai ficar o arquivo de configuração do nosso pipeline.

```bash
mkdir -p .github/workflows 
```

Agora podemos criar o arquivo do pipeline com o conteúdo abaixo.

 ```yaml
name: github pages
on:
 push:
   branches:
     - main
 pull_request:

jobs:
 deploy:
   runs-on: ubuntu-20.04
   steps:
     - uses: actions/checkout@v2
       with:
         submodules: true
         fetch-depth: 0

     - name: Setup Hugo
       uses: peaceiris/actions-hugo@v2
       with:
         hugo-version: '0.92.1'
        
     - name: Build
       run: hugo --minify

     - name: Deploy
       uses: peaceiris/actions-gh-pages@v3
       if: github.ref == 'refs/heads/main'
       with:
         github_token: ${{ secrets.GH_TOKEN }}
         publish_dir: ./public
         cname: www.porfa.xyz #Só deixe essa chave se você quiser ter um dominio customizado
```
O pipeline possui quatro passos:

1. baixa o conteúdo do repositório para que os próximos passos realizem o seu trabalho
1. instala o hugo com a versão informada.
1. executar o build do hugo, esse passo gera todo o conteúdo do site já minificado
1. criar uma branch com o nome gh-pages e fazer o commit do conteúdo gerado na pasta ./public. 
## Configurando o Dominio

