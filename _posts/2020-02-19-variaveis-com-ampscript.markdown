---
layout: post
title: "Como criar variáveis com AMPscript"
date: 2020-2-19 22:12:01 -0200
categories: ['AMPscript']
author: "Tony Arthur"
description: "Definição de variáveis, sintaxe e atribuição de valores!"
---
<div>
      <p>
            E aí galera, estou criando uma série de conteúdos para dá os primeiros passos com a linguagem de script do salesforce Marketing cloud, então fica ligado aqui pelo blog!
      </p>
      <p>
            Irei começar com a declaração de variáveis. Sabemos que para guardar valores é necessário o uso de variáveis, isso acontece em qualquer linguagem de programação, aqui, irei passar como é a sintaxe para a criação de variáveis no AMPscript, atribuir valores e imprimir em tela. 
      </p>
      <h2>Variáveis</h2>
      <p>
            A declaração é realizada com o uso da palavra chave <strong>VAR</strong>, seguido do nome da variável com o sinal de <strong>@</strong>  no início. Segue um exemplo:
      </p>
      <div>
            <script src="https://gist.github.com/tonyarthur/5c2c5d5fb0d35e8122c80e56a6da3ca3.js"></script>
      </div>
      <p>
            A declaração de muitas variáveis pode ser realizada em uma única linha, separando as mesmas por vírgula. <br/>
            É o que ocorre na <strong>linha 2</strong>, onde <strong>@nome</strong> e <strong>@sobrenome</strong> são declaradas.
      </p>
      <h2>Atribuição de valores da variável</h2>
      <p>
            Ainda aproveitando o exemplo acima, as linhas 3 e 4 estão incluindo valores para as variáveis, isto é possível através da palavra chave <strong>SET</strong>.
      </p>
      <h2>Imprimindo valores de variáveis</h2>
      <p>
            Existem diferentes formas de se imprimir o valor de uma variável, porém, neste exemplo iremos utilizar a mais comum! <br/>
            Observe as linhas 7 e 8 do exemplo anterior.
      </p>
      <p>
            A impressão é realizada através da função <strong>V</strong> , esta é responsável por imprimir o valor de qualquer variável e é utilizada fora do bloco de conteúdo, a sua utilização é da seguinte forma: 
            <strong>%%=v(@suaVariavel)=%%</strong>

      </p>
      <details>
            <summary><em>Obs.: O valor compilado pode ser utilizado mesclando tags de html ou atributos, no exemplo é utilizado a tag SPAN.</em></summary>
      </details>
      <p>
            E aí, o que achou? espero poder contribuir de alguma forma com o seu aprendizado em AMPscript, mesmo que seja ainda básico.
            Comenta aqui em baixo ou fique a vontade para entrar em contato comigo através das redes sociais. 
      </p>
</div>
  