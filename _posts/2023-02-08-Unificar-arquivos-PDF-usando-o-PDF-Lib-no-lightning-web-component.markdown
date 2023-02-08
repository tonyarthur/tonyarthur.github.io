---
layout: post
title: "Unificar arquivos PDF usando o PDF Lib no lightning web component"
date: 2023-2-08 14:37:01 -0200
categories: ['lwc']
author: "Tony Arthur"
description: "Unificar arquivos em PDF fazendo uso de uma biblioteca externa no lightning web component"
---
Fala pessoal, tudo bem ?
Estou por aqui de novo, focado em escrever mais sobre todo o ecosistema de desenvolvimento em Salesforce.

Dessa vez, irei falar como podemos unificar alguns arquivos PDF usando uma biblioteca externa no lightning web component.

Pois bem, devo considerar que tu já sabes carregar um arquivo externo no lightning web component, mas caso tu não saibas ainda, tu podes ir para o post que eu falo sobre [Carregar scripts e folhas de estilos CSS](Carregar-scripts-e-folhas-de-estilos-CSS-usando-o-lightning-web-components){:target="_blank"}.

**1. Criando a classe controller no apex para obter os arquivos em base64 e reber depois estes arquivos unificados e salvar os mesmo no contentVersion.**

{% highlight java %}
public without sharing class MergePDFFileController {
    private static final String PDF = 'pdf';

    @AuraEnabled
    public static List<String> getAllFilesByAccount(Id accountId){
        List<ContentDocumentLink> contentDocumentLinks = getContentDocumentLinkByLinkedEntityId(accountId);
        List<Id> contentDocumentId = uniqueContentDocumentId(contentDocumentLinks);
        List<ContentVersion> contentVersions = getContentVersionByContentDocumentIds(contentDocumentId);

        List<String> files = new List<String>();

        for(ContentVersion contentVersion : contentVersions){
            if(contentVersion.FileExtension.toLowerCase() == PDF){
                String base64File = EncodingUtil.base64Encode(contentVersion.VersionData);
                files.add(base64File);
            }
        }

        return files;
    }
}
{% endhighlight %}

Aqui devemos nos atentar ao método **getAllFilesByAccount**,ele é responsável por trazer todos os arquivos do tipo PDF que está vinculado ao id de uma conta que é passado como parâmetro. 

Extraímos o Blob do ContentVersion no campo VersionData e códificamos usando o método **base64Encode** e então adicionamos em uma lista de string para retornar para o front-end.

> "Se tu queres saber mais sobre o os métodos base64Encode e base64Decode, tu podes acessar a documentação oficial da Salesforce aqui."

**2. Criando o componente mergePDFExample**

{% highlight javascript %}
import { LightningElement, api } from 'lwc'
import { loadScript } from "lightning/platformResourceLoader"
import { ShowToastEvent } from 'lightning/platformShowToastEvent'

import pdflib from "@salesforce/resourceUrl/pdfLib"

import getAllFilesByAccount from '@salesforce/apex/MergePDFFileController.getAllFilesByAccount'

export default class MergePDFExample extends LightningElement {

    @api recordId

    _pdfLibInitialized
    
    renderedCallback() {
        loadScript(this, pdflib).then(() => {
          this._pdfLibInitialized = true
        })
    }

    async create(){
        const contentVersionsData = await getAllFilesByAccount({accountId: this.recordId})

        const mergePDF = await PDFLib.PDFDocument.create()

        for(let file of contentVersionsData){
            const existingPDFbytes = Uint8Array.from(atob(file), (c) => c.charCodeAt(0))
            
            const pdf = await PDFLib.PDFDocument.load(existingPDFbytes)
            const copiedPages = await mergePDF.copyPages(pdf, pdf.getPageIndices())

            copiedPages.forEach((page) => mergePDF.addPage(page))
        }

        const mergedPdfFile = await mergePDF.save()
        this.prepareToUploadFile(mergedPdfFile)
}
{% endhighlight %}

No componente iremos importar nosso método **getAllFilesByAccount** da classe apex, assim como a biblioteca pdflib que irá nos auxiliar na unifiação dos arquivos.

Criei a função **create()** que será chamada no html através de um botão. 

Antes de tudo iremos recuperar todos os arquivos codificados em **getAllFilesByAccount({accountId: this.recordId})**.

Após isso, iremos criar uma instancia da biblioteca em **PDFLib.PDFDocument.create()**

A biblioteca precisa que os arquivos estejam em um array na base de 8bits. É o que fazemos na liha **Uint8Array.from(atob(file), (c) => c.charCodeAt(0))**

Em seguida usamos a função load do PDFlib caso queira saber mais sobre a biblioteca você pode acessar a [documentação](https://pdf-lib.js.org/){:target="_blank"} oficial e ir até [Embed PDF Pages](https://pdf-lib.js.org/#embed-pdf-pages){:target="_blank"}

Fazemos uma copia da página usando a função **copyPages** e em seguida adicionamos essa página usando a função **addPage**. 

Ao final do loop for, chamamos a função **save().**

**3.Preparando o arquivo para carregar no Salesforce.**
{% highlight javascript %}
async prepareToUploadFile(pdfFiles){
        
        let pdfBlob = new Blob([pdfFiles], { type: "application/pdf" })
        
         if(pdfBlob.size > MAX_FILE_SIZE) {
            this.dispatchEvent(
                new ShowToastEvent({
                    title: 'Tamanho do arquivo excedido',
                    message: ` O tamanho do arquivo não pode exceder ${MAX_FILE_SIZE} bytes. Tamanho do arquivo unificado: ${pdfBlob.size}`,
                    variant: 'error',
                })
            )
            return
        } 
        
        const base64 = await this.readUploadedFileAndReturnBase64(pdfBlob)
        this.fileUpload(pdfBlob, base64)
    
}
{% endhighlight %}

A função **prepareToUploadFile()** recebe o arquivo **Serializado em bytes (um Uint8Array)** que retorna da função **save()**.

 Transformamos esses arquivos no tipo blob na linha 

 **let pdfBlob = new Blob([pdfFiles], { type: "application/pdf" })**

 A constante **MAX_FILE_SIZE** determina o tamanho máximo que este arquivo poderá conter, passamos o valor **4718592** em bytes, isso dá um pouco mais que **4MB**. 
O Salesforce tem um tamanho para uso de String, mais para frente irei falar sobre este assunto, enquanto isso, vamos continuar com a leitura do código. Se esse tamanho for atingido mostrare-mos uma mensagem do tipo toast ao usuário, caso contrário seguiremos com o código. 
A funçao **readUploadedFileAndReturnBase64** recebe esse nosso novo tipo de variável blob.

{% highlight javascript %}
readUploadedFileAndReturnBase64(pdfBlob){
        let binary = new String()
        const temporaryFileReader = new FileReader()

        return new Promise((resolve, reject) => {
            temporaryFileReader.onerror = () => {
            temporaryFileReader.abort()
                reject(new DOMException("Problema ao analisar o arquivo de entrada"))
            }

            temporaryFileReader.onload = () => {
                let fileContents = temporaryFileReader.result
                let base64Mark = 'base64,'
                let startOfIndex = fileContents.indexOf(base64Mark) + base64Mark.length
                binary = fileContents.substring(startOfIndex)
                resolve(binary)
            }

            temporaryFileReader.readAsDataURL(pdfBlob)
      })
}
{% endhighlight %}

Nele, básicamente fazemos uso do objeto [FileReader](https://developer.mozilla.org/pt-BR/docs/Web/API/FileReader){:target="_blank"} disponível em browsers que permite trabalhar com conteúdos de arquivos. 
Utilizamos a propriedade [onload](https://developer.mozilla.org/pt-BR/docs/Web/API/FileReader/load_event){:target="_blank"}, para manipular o aquivo, quando esse arquivo está pronto, nós pegamos o seu conteúdo e estraímos apenas a base64 dele, retornando este na função. 
{% highlight javascript %}
fileUpload(file, fileContents){
      let startPosition = 0
      let endPosition = Math.min(fileContents.length, startPosition + CHUNK_SIZE)

      this.fileUploadInChunks(file, fileContents, startPosition, endPosition, null)
}
{% endhighlight %}

Criamos uma nova função denominada **fileUpload** na qual recebe nosso blob e o conteúdo em base64. 
Essa função é responsável por passar o ínicio e o fim do indice da nossa base64. 
A Constante **CHUNK_SIZE** diz, qual será o tamanho do pedaço deste aquivo será inserido no Salesforce por vez. 

{% highlight javascript %}
async fileUploadInChunks(file, fileContents, startPosition, endPosition, contentVersionId){
      let chunk = fileContents.substring(startPosition, endPosition)        

      try {
      const result = await fileUploadInChunks({
            parentId: this.recordId,
            fileName: file.name,
            base64Data: encodeURIComponent(chunk),
            fileId: contentVersionId
      })

      contentVersionId = result
      startPosition = endPosition
      endPosition = Math.min(fileContents.length, startPosition + CHUNK_SIZE)

      if (startPosition < endPosition) {
            this.fileUploadInChunks(file, fileContents, startPosition, endPosition, contentVersionId)

      }else {
            this.dispatchEvent(
                  new ShowToastEvent({
                  title: "Sucesso!!",
                  message: "O Arquivo foi unificado com sucesso",
                  variant: "success"
                  })
            )
      }
      } catch (error) {
      console.error("Erro: ", error)
      }
}
{% endhighlight %}

Por fim, a função **fileUploadInChunks** é a responsável chamar a classe controller e passar em pedaços do arquivo até o fim. 
A linha abaixo é que de fato chama a controller, passando os parametros necessários 

{% highlight javascript %}
const result = await fileUploadInChunks({
                parentId: this.recordId,
                fileName: file.name,
                base64Data: encodeURIComponent(chunk),
                fileId: contentVersionId
            })
{% endhighlight %}

Neste momento é feito a inserção do primeiro pedaço (chunk) e com isso, é retornado um Id de um registro do ContentVersion como resposta. 
Esse resultado passamos na variável **contentVersionId**, em seguida calculamos novamente o novo posicionamento dos indices
**startPosition = endPosition**
**endPosition = Math.min(fileContents.length, startPosition + CHUNK_SIZE)**
E então verificamos se ainda tem mais pedaços a serem inseridos, e então chamamos a mesma função **this.fileUploadInChunks** novamente até finalizar o processo. 

4. **Inserindo e atualizando em pedaços (chunk) os arquivos na classe controller do apex.**

{% highlight java %}
      @AuraEnabled                            
      public static Id fileUploadInChunks(Id parentId, String fileName, String base64Data, Id fileId){
      
            base64Data = EncodingUtil.urlDecode(base64Data, 'UTF-8');
            

            if ( String.isBlank(fileId) ) {
            fileId = saveFile(parentId, fileName, base64Data );
            } else {
            appendToFile(fileId, base64Data);
            }

            return Id.valueOf(fileId) ;
    }

    @AuraEnabled
    public static Id saveFile(Id recordId, String fileName, String base64Data )  { 
        
            if(!ContentVersion.sObjectType.getDescribe().isCreateable() ||
                  !ContentDocumentLink.sObjectType.getDescribe().isCreateable()){
                  throw new AuraHandledException('Você não tem permissão para criar um arquivo');
            }

            contentVersion contentVersion = insertContentVersion(recordId, fileName,  base64Data);
            contentVersion = getContentVersionById(contentVersion.Id);
            
            ContentDocumentLink cl = insertContentDocumentLink(contentVersion.ContentDocumentId, recordId );

            return contentVersion.Id;

      }

    @AuraEnabled 
    public static void appendToFile(Id contentVersionId, String base64Data) {
        
            try {
            ContentVersion contentVersionResult = getContentVersionById(contentVersionId);
            
            String existingBody = EncodingUtil.base64Encode(contentVersionResult.VersionData);
            contentVersionResult.VersionData = EncodingUtil.base64Decode(existingBody + base64Data);

            if(!ContentVersion.sObjectType.getDescribe().isUpdateable()){
                  throw new AuraHandledException('Você não tem permissão para atualizar o arquivo');
            }   
            
            update contentVersionResult;
            } catch (Exception err) {
            throw new AuraHandledException(err.getMessage());
            } 
    
      }
{% endhighlight %}

O método **fileUploadInChunks** é o que sempre será chamado pelo front-end, nele verificamos se o **Id** do contentVersion (fileId) já existe. 
Se não existir chamamos o método saveFile e fazemos a inserção do contentVersion no banco e retornamos o Id criado. 
Como o front-end passa em pedaços(chunk) o arquivo, na próxima chamada o Id existirá, e então chamamos o método **appendToFile**, ele é responsável por recuperar no banco o pedaço (chunk) que inserimos e então somar mais o outro pedaço que estamos recebendo do front-end e  atualizamos este registro no banco novamente.
> Lembra que disse que comentária sobre o tamanho do arquivo ? pois bem, na linha                        String existingBody = EncodingUtil.base64Encode(contentVersionResult.VersionData);         Recuperamos o pedaço já salvo do arquivo que está salvo no tipo Blob no campo VersionData do objeto ContentVersion e codificamos ele usando base64Encode, este retorna uma string e salvamos na variável **existingBody** a String possui um tamanho máximo de 6000000, caso isso seja atingido o Salesforce lançará o erro: **[String length exceeds maximum: 6000000]** por está razão é necessário limitar o tamanho do arquivo lá no front-end, pois um arquivo decodificado gera uma string muito grande.

O processo se repete até finalizar . 

Abaixo está todo o código do classe controller do apex chamada **MergePDFFileController**
{% highlight java %}
public without sharing class MergePDFFileController {
    private static final String PDF = 'pdf';

    @AuraEnabled
    public static List<String> getAllFilesByAccount(Id accountId){
        List<ContentDocumentLink> contentDocumentLinks = MergePDFFileController.getContentDocumentLinkByLinkedEntityId(accountId);
        List<Id> contentDocumentId = MergePDFFileController.uniqueContentDocumentId(contentDocumentLinks);
        List<ContentVersion> contentVersions = MergePDFFileController.getContentVersionByContentDocumentIds(contentDocumentId);

        List<String> files = new List<String>();

        for(ContentVersion contentVersion : contentVersions){
            if(contentVersion.FileExtension.toLowerCase() == PDF){
                String base64File = EncodingUtil.base64Encode(contentVersion.VersionData);
                files.add(base64File);
            }
        }

        return files;
    }

    @AuraEnabled                            
    public static Id fileUploadInChunks(Id parentId, String fileName, String base64Data, Id fileId){
      
        base64Data = EncodingUtil.urlDecode(base64Data, 'UTF-8');
        

        if ( String.isBlank(fileId) ) {
            fileId = saveFile(parentId, fileName, base64Data );
        } else {
            appendToFile(fileId, base64Data);
        }

        return Id.valueOf(fileId) ;
    }

    @AuraEnabled
    public static Id saveFile(Id recordId, String fileName, String base64Data )  { 
        
        if(!ContentVersion.sObjectType.getDescribe().isCreateable() ||
        !ContentDocumentLink.sObjectType.getDescribe().isCreateable()){
            throw new AuraHandledException('Você não tem permissão para criar um arquivo');
        }

        contentVersion contentVersion = insertContentVersion(recordId, fileName,  base64Data);
        contentVersion = getContentVersionById(contentVersion.Id);
        
        ContentDocumentLink cl = insertContentDocumentLink(contentVersion.ContentDocumentId, recordId );

        return contentVersion.Id;

    }

    @AuraEnabled 
    public static void appendToFile(Id contentVersionId, String base64Data) {
        
        try {
            ContentVersion contentVersionResult = getContentVersionById(contentVersionId);
            
            String existingBody = EncodingUtil.base64Encode(contentVersionResult.VersionData);
            contentVersionResult.VersionData = EncodingUtil.base64Decode(existingBody + base64Data);

            if(!ContentVersion.sObjectType.getDescribe().isUpdateable()){
                throw new AuraHandledException('Você não tem permissão para atualizar o arquivo');
            }   
            
            update contentVersionResult;
        } catch (Exception err) {
            throw new AuraHandledException(err.getMessage());
        } 
    
    }

    private static ContentDocumentLink insertContentDocumentLink(Id contentDocumentId, Id linkedEntityId){
        ContentDocumentLink contentDocumentLink = new ContentDocumentLink();
        contentDocumentLink.ContentDocumentId = contentDocumentId;
        contentDocumentLink.LinkedEntityId = linkedEntityId; 
        contentDocumentLink.ShareType = 'V';
        contentDocumentLink.Visibility = 'AllUsers';
        insert contentDocumentLink;

        return contentDocumentLink;
    }

    private static ContentVersion insertContentVersion(Id recordId, String fileName, String base64Data ){
        ContentVersion contentToInsert = new ContentVersion(); 
        contentToInsert.Title = fileName; 
        contentToInsert.VersionData = EncodingUtil.base64Decode(base64Data);
        contentToInsert.PathOnClient = 'CustomPDF.pdf' ;
        contentToInsert.IsMajorVersion = false;
        contentToInsert.Title = 'ConvertedtoPDF';
        insert contentToInsert; 

        return contentToInsert;
    }

    private static ContentVersion getContentVersionById(Id contentVersionId){
        ContentVersion contentVersion = new ContentVersion();
        contentVersion = [SELECT Id, ContentDocumentId, VersionData from ContentVersion WHERE Id =: contentVersionId WITH SECURITY_ENFORCED];
        
        return contentVersion;
    }

    private static List<ContentDocumentLink> getContentDocumentLinkByLinkedEntityId(Id recordId){
        List<ContentDocumentLink> contentDocumentlinks = new List<ContentDocumentLink>();
        contentDocumentlinks = [SELECT Id, ContentDocumentId FROM ContentDocumentLink WHERE LinkedEntityId =: recordId WITH SECURITY_ENFORCED];
       
        return contentDocumentlinks;
    }

    private static List<Id> uniqueContentDocumentId(List<ContentDocumentLink> contentDocumentLinks){
        Set<Id> uniqueId = new Set<Id>();
        
        for(ContentDocumentLink contentDocumentLink :contentDocumentLinks){
            uniqueId.add(contentDocumentLink.ContentDocumentId);
        }

        return new List<Id>(uniqueId);
    }

    private static List<ContentVersion> getContentVersionByContentDocumentIds(List<Id> contentDocumentId){
        List<ContentVersion> contentVersions = new List<ContentVersion>();

        contentVersions = [SELECT Id, VersionData, FileExtension FROM ContentVersion WHERE ContentDocumentId =: contentDocumentId WITH SECURITY_ENFORCED];
        
        return contentVersions;
    }
    
}
{% endhighlight %}

E também abaixo o código completo do componente chamado **MergePDFExample**
{% highlight java %}
import { LightningElement, api } from 'lwc'
import { loadScript } from "lightning/platformResourceLoader"
import { ShowToastEvent } from 'lightning/platformShowToastEvent'

import pdflib from "@salesforce/resourceUrl/pdfLib"

import getAllFilesByAccount from '@salesforce/apex/MergePDFFileController.getAllFilesByAccount'
import fileUploadInChunks from '@salesforce/apex/MergePDFFileController.fileUploadInChunks'

const MAX_FILE_SIZE = 4718592
const CHUNK_SIZE = 750000

export default class MergePDFExample extends LightningElement {

    @api recordId

    _pdfLibInitialized
    
    renderedCallback() {
        loadScript(this, pdflib).then(() => {
          this._pdfLibInitialized = true
        })
    }

    async create(){
        const contentVersionsData = await getAllFilesByAccount({accountId: this.recordId})

        const mergePDF = await PDFLib.PDFDocument.create()

        for(let file of contentVersionsData){
            const existingPDFbytes = Uint8Array.from(atob(file), (c) => c.charCodeAt(0))
            
            const pdf = await PDFLib.PDFDocument.load(existingPDFbytes)
            const copiedPages = await mergePDF.copyPages(pdf, pdf.getPageIndices())

            copiedPages.forEach((page) => mergePDF.addPage(page))
        }

        const mergedPdfFile = await mergePDF.save()
        this.prepareToUploadFile(mergedPdfFile)
    }

    async prepareToUploadFile(pdfFiles){
        
        let pdfBlob = new Blob([pdfFiles], { type: "application/pdf" })
        
         if(pdfBlob.size > MAX_FILE_SIZE) {
            this.dispatchEvent(
                new ShowToastEvent({
                    title: 'Tamanho do arquivo excedido',
                    message: ` O tamanho do arquivo não pode exceder ${MAX_FILE_SIZE} bytes. Tamanho do arquivo unificado: ${pdfBlob.size}`,
                    variant: 'error',
                })
            )
            return
        } 
        
        const base64 = await this.readUploadedFileAndReturnBase64(pdfBlob)
        this.fileUpload(pdfBlob, base64)
    
    }

    readUploadedFileAndReturnBase64(pdfBlob){
        let binary = new String()
        const temporaryFileReader = new FileReader()

        return new Promise((resolve, reject) => {
            temporaryFileReader.onerror = () => {
            temporaryFileReader.abort()
                reject(new DOMException("Problema ao analisar o arquivo de entrada"))
            }

            temporaryFileReader.onload = () => {
                let fileContents = temporaryFileReader.result
                let base64Mark = 'base64,'
                let startOfIndex = fileContents.indexOf(base64Mark) + base64Mark.length
                binary = fileContents.substring(startOfIndex)
                resolve(binary)
            }

            temporaryFileReader.readAsDataURL(pdfBlob)
        })
    }

    fileUpload(file, fileContents){
        let startPosition = 0
        let endPosition = Math.min(fileContents.length, startPosition + CHUNK_SIZE)

        this.fileUploadInChunks(file, fileContents, startPosition, endPosition, null)
    }


    async fileUploadInChunks(file, fileContents, startPosition, endPosition, contentVersionId){
        let chunk = fileContents.substring(startPosition, endPosition)      

        try {
            const result = await fileUploadInChunks({
                parentId: this.recordId,
                fileName: file.name,
                base64Data: encodeURIComponent(chunk),
                fileId: contentVersionId
            })

            contentVersionId = result
            startPosition = endPosition
            endPosition = Math.min(fileContents.length, startPosition + CHUNK_SIZE)

            if (startPosition < endPosition) {
                this.fileUploadInChunks(file, fileContents, startPosition, endPosition, contentVersionId)

            }else {
                this.dispatchEvent(
                    new ShowToastEvent({
                      title: "Sucesso!!",
                      message: "O Arquivo foi unificado com sucesso",
                      variant: "success"
                    })
                  )
            }
        } catch (error) {
            console.error("Erro: ", error)
        }
    }
}
{% endhighlight %}

Eu sei que esse post ficou um pouco extenso, mas era necessário explicar algumas coisas minimas que está ocorrendo no código. 
Mas e ai me conta? tu já implementou algo parecido com isto ? ou está passando por isso agora e espero que este conteúdo realmente te ajude. 
Eu passei por alguns dias tentando fazer isso dá muito certo, e claro pesquisei muito pela internet e esse conteúdo tem referências do post do **[Kapil Batra](https://www.linkedin.com/pulse/create-custom-pdf-lightning-web-component-salesforce-kapil-batra/){:target="_blank"}.**

Essa é uma solução que tem um limite da criação do arquivo, porém, há outra solução para aumentar esse limite trabalhando com a API de inserção do ContentVersion. Me conta aqui se tu queres que eu escreva sobre isso também em um outro post. 

Mais uma vez, muito obrigado por passar por aqui, e até o próximo post.