---
layout: post
title: "Códigos AMPscript com editor externo"
date: 2020-2-10 21:14:01 -0200
categories: ['AMPscript']
author: "Tony Arthur"
description: "Escreva seus códigos AMPscript, SSJS, HTML e CSS de sua landing page em um editor externo e 
seja mais produtivo."
---

<div>
      <p>E aí dev ou usuário do Markerting cloud!</p> 
      
      <p>Você já teve que escrever muitos códigos AMPscript, SSJS, HTML e CSS em uma landing page, e já perdeu um bom tempo salvando e publicando as alterações?</p>

      <p> Sabemos que isto consome bastante nosso tempo de trabalho, tornando nossa produtividade baixa. <br/>
      Então estou te escrevendo este post, para te dá uma ajudinha e melhorar este tempo. Vamos utilizar um editor online e gratuito <a href="https://codesandbox.io/" target="_blank"><strong>(codesandbox.io)</strong></a> ou pode ser qualquer outro e fazer uso das funções HTTPGET e TreatAsContent do AMPscript na lading page.</p>

      <h2>Criando e configurando uma conta no codesandbox.</h2>
      <p>
        Crie sua conta no codesandbox com sua conta de login do Github, se você não tem uma conta no Github, crie uma é super fácil e prático. 
        </p>
        <p>
            Galera após isso crie um projeto selecionando o tipo <em>Static</em>. 
        </p>
        <p>
            Feito isso o passo mais importante é ir até a aba <em>Deployment</em> e realizar um deploy do projeto, seguindo os passos do netlify. 
        </p>
      <div class="text-al-center">
            <img src="/assets/img/posts/2020-01-22-codigo-ampscript-com-editor-externo/Codesandbox-create-account.JPG" title="Editor externo Codesandbox" alt="Editor externo CodeSandbox">
      </div>
      <p>
            Processo pronto, copie o link de acesso externo deste projeto, geralmente ele estará no lado direito, algo como <em><a target="_blank" href="https://vt5hpe.csb.app/">https://vt5hpe.csb.app/</a></em>.
      </p>
      <br/>
      <h2>Configurando a landing page</h2>
      <p>
            No aplicativo CloudPages, crie sua coleção em seguida a landing page.
      </p>
      <p>
            Arraste uma atividade de HTML para sua página e em seguida inclua as funções HTTPGet e TreatAsContent, as funções ficarão da seguinte maneira. 
      </p>
      <p>
           <strong>TreatAsContent(HTTPGet('https://vt5hpe.csb.app/'))</strong>
      </p>
      <p>
            No HTTPGet passamos nossa URL externa do codesandbox, o resultado é atribuido na função TreatAsContent, que gera um bloco de conteúdo em nossa página.
      </p>
      <p>
            Feito isso publique sua página, e agora é só codar através do codesandbox e apenas dá um refresh na sua landingpage com o link que foi gerado na publicação. Pronto, nada mais de fazer qualquer alterção e publicar só para ver uma pequena alteração, fique a vontade para escrever seus códigos AMPscript, SSJS,HTML e CSS.
      </p>
      <p>
            Isto facilita demais o desenvolvimento na nuvem do Marketing Cloud!
      </p>
      <p>
            E aí, curtiu a dica? fica ligado por aqui pois irei trazer bastante conteúdo sobre essa nuvem da Salesforce, espero poder ajudar a todos de alguma maneira e contribuir com a comunidade. 
      </p>
      <p>
            See you guys!
      </p>
</div>
  