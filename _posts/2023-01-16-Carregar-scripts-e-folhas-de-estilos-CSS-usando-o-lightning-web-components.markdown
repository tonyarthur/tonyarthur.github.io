---
layout: post
title: "Carregar scripts e folhas de estilos CSS usando o lightning web components"
date: 2023-1-16 01:37:01 -0200
categories: ['lwc']
author: "Tony Arthur"
description: "Fazer uso de bibliotecas de terceiros ou qualquer outro recurso estático."
---
<div>
      <p>Fala pessoal, tudo bem ?</p>
      <p>Estou retornando aqui criando conteúdos voltado ao desenvolvimento de Salesforce em geral, e hoje vamos abordar o carregamento de scripts e folhas de estilo (style/css) usando o lwc.</p>
      <p>Bem primeiro vamos baixar um arquivo chamado PDF-lib, tu  podes fazer isto através deste <a href="https://pdf-lib.js.org/" target="_blank"><span>link</span></a>, 
      e também criar nosso arquivo <span style="font-style:italic">style.css</span> para que possamos importar no lwc.</p>
      <p>Vou criar um arquivo CSS que contém apenas uma classe chamada <span style="font-style:italic">blue-text</span> que possui a propriedade <span style="font-style:italic">color</span> e o valor blue, simples assim.</p>
      {% highlight css %}
      .blue-text {
            color: blue;
      }
      {% endhighlight %}
      <p>Bom, com todos estes arquivos preparados, iremos fazer a importação dos mesmos para os recursos estáticos na organização da Salesforce.</p>
      <p>Para isso tu deves acessar o ícone de engrenagem, em seguida clicar em Setup, na caixa de busca rápida digite por Static Resources ou ( Recursos estáticos).</p>
      <p>Clique em new (novo), dê um nome para o seu recurso estático, em meu caso será MyStyle e vou carregar o meu arquivo CSS direto do meu computador. </p>
    
      <img src="/assets/img/posts/2023-01-16-Carregar-scripts-e-folhas-de-estilos-CSS-usando-o-lightning-web-components/inserindo-arquivo-estatico.png" title="Editor externo Codesandbox" alt="Editor externo CodeSandbox">
    
      <p>Faço o mesmo processo para o Script de PDF-lib baixado anteriormente com o nome PDFLib</p>
      <p>Agora com os arquivos carregados na organização iremos criar um arquivo em lwc chamado <strong>loadScriptAndStyle</strong></p>
      <p>Faremos a importação do módulo <strong>platformResourceLoader</strong>, com ele é possível usar os métodos <strong>loadScript</strong> e <strong>loadStyle</strong> para carregar nossos recursos. </p>
      <p>import { loadScript, loadStyle } from 'lightning/platformResourceLoader’</p>
      <p>Logo abaixo faremos a nossa importação dos recursos estáticos.</p>
      <p>import PDFLib from '@salesforce/resourceUrl/PDFLib';</p>
      <p>import MyStyle from '@salesforce/resourceUrl/MyStyle';</p>
      <p>Para que nossos recursos estejam prontos no momento da inicialização do arquivo vou fazer uso do <strong>connectedCallback</strong> que é um Lifecycle Hooks ( clico de vida ), 
      para saber mais tu podes acessar a documentação clicando <a href="https://developer.salesforce.com/docs/component-library/documentation/en/lwc/reference_lifecycle_hooks" target="_blank" ><span>aqui</span></a>.
      </p>
      {% highlight javascript %}
            import { LightningElement } from 'lwc';
            import { loadScript, loadStyle } from 'lightning/platformResourceLoader';
            import pdfLib from '@salesforce/resourceUrl/PDFLib';
            import myStyle from '@salesforce/resourceUrl/MyStyle';
            export default class LoadScriptAndStyle extends LightningElement {
                  connectedCallback(){
                        Promise.all([
                              loadScript(this, pdfLib ),
                              loadStyle(this, myStyle ),
                        ]).then(() => {
                              console.log(`Arquivos carregados com sucesso!`);
                        })
                  }
            }
      {% endhighlight %}
      <p>Os métodos <strong>loadScript</strong> e <strong>loadStyle</strong> retornam uma promise, por isso estou fazendo uso da Promise.</p>
      <p>Há outra maneira também de chamar uma promise. É tornar o <strong>connectedCallback</strong> em async e escrever da seguinte forma.</p>
      {% highlight javascript%}
            async connectedCallback(){
                  await loadScript(this, pdfLib );
                  await loadStyle(this, myStyle );
                  console.log(`Arquivos carregados com sucesso!`);
            }
      {% endhighlight %}
      <p>Após tudo ocorrer corretamente um <strong>console.log</strong> é lançado e então é possível visualizar a o texto <span style="font-style:italic">Arquivos carregados com sucesso</span>!</p>
      <p>Em seguida incluir um texto em uma tag H1 diretamente no arquivo HTML e chamei o classe <strong>blue-text</strong> definida em nosso CSS MyStyle.</p>
      {% highlight html%}
            <template>
                  <div>
                        <h1 class="blue-text">Meu texto azul </h1>
                  </div>
            </template>
      {% endhighlight %}
      <p>O resultado será um texto na cor azul.</p>
      <br>
      <p>Muitas vezes precisamos fazer uso de bibliotecas externas para auxiliar o novo desenvolvimento ou criar um arquivo CSS para que seja de uso global dentro da organização e entre outras coisas.</p>
      <p>É isso, agradeço por tu ter passado por aqui, espero que esse conteúdo tenha te ajudado de alguma forma.</p>
</div>
  