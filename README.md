# Cookbook Fluig

## 1 - eventos de formulários

### 1.1 - afterSaveNew: após salvar um novo registro, recebe o form

```js
function afterSaveNew(form) {
    log.info("Colaborador de abertura: " + form.getValue("RNC_colab_abertura"));
}
```
    

### 1.2 - displayFields: após a exibição, reebe o form e o customHTML

```js
function displayFields(form, customHTML) {
    if ( form.getFormMode() == “MOD” ) {
        form.setValue('RNC_colab_abertura', new java.lang.Integer(1));
    }
}
```

Para desabilitar campos usar setShowDisabledFields
```js
form.setShowDisabledFields(true);
```

Para desabilitar o botão de excluir do form filho, usar setHideDeleteButton
```js
form.setHideDeleteButton(false);
```

Para ocultar o os botões Imprimir e Imprimir em nova Janela, deve-se utilizar o método setHidePrintLink.
```js
form.setHidePrintLink(true);
```

Para esconder campos usar o setVisible ou setVisibleById

```js
function displayFields(form, customHTML) {
    form.setVisible("campoA", false);
    form.setVisibleById("linha___1", false);
}
```

Com o customHTML é possivel injetar html ou scripts direto na página

```js
if(form.getFormMode() != "VIEW")  {
    customHTML.append("<script>");
    customHTML.append("function MostraEscondeBtn_zoom()");
    customHTML.append("{");
    customHTML.append("document.getElementById(\'zoomUsuario\').className = \'show\';");
    customHTML.append("document.getElementById(\'zoomModulo\').className = \'show\';");
    customHTML.append("}");
    customHTML.append("</script>");
}
```

### 1.3 - enableFields: qndo os elementos são habilitados, recebe o form

```js
form.setEnabled("nome-do-campo",true/false)
```

```js
function enableFields(form) {
    if ( form.getFormMode() != 'ADD' ){
        form.setEnabled("rnc_area",false);
        form.setEnabled("rnc_tipo_ocorrencia",false);
    }
}
```

por preferência use o readonly para que um campo não seja alterado, pois ao usar o disabled, o campo não será salvo

para proteger um campo de alteração ( via chrome dev tools) informe true no segundo parametro do setEnabled

```js
function enableFields(form) {
    if ( form.getFormMode() != 'ADD' ){
        form.setEnabled("rnc_cod_ocorrencia",false, true);
    }
}
```

existe também um metódo que protege todos os campos escondidos

```js
function enableFields(form) {
    if ( form.getFormMode() != 'ADD' ){
        form.setEnhancedSecurityHiddenInputs(true);
        form.setEnabled("rnc_cod_ocorrencia",false);
    }
}
```

### 1.4 - inputFields: disparado no momento que os dados do form são      passados para o backend, recebe o form

evento geralmente utilizado para salvar informações em campos ocultos      ou para tratar dados antes de serem salvos

por exemplo o chrome salva campos de data no formato americano **(yyyy-mm-dd)**, já outros navegados salvam no formato brasileiro **(dd/mm/yyyy)**. Abaixo um exemplo de código que padroniza o formato      de data salvo

```js
function inputFields(form) {

  var regEx = /^\d{4}-\d{2}-\d{2}$/;

  if (form.getValue("dt_solicitacao").match(regEx)) {
    var split = form.getValue("dt_solicitacao").split('-');
    form.setValue("dt_solicitacao", split[2] + '-' + split[1] + '-' + split[0]);
  }

}
```

### 1.4 - seEnable: evento não chamado automaticamente, mas pode ser      invocado de outros metodos

```js
function setEnable() {
  log.info(“Teste de chamada de função”);
}
```

```js
function displayFields(form, customHTML) {
  setEnable();
}
```

### 1.5 - validadeForm: evento chamado antes da gravação dos dados,      recebe o form

para retornar erros após a validação fazer um throw
quebras de linha usar \n

```js
function validateForm(form) {
   if (form.getValue('RNC_colab_abertura') == null){
     throw "O colaborador de abertura não foi informado";
   }
}
```

### 1.6 - afterProcessing: ultimo evento, recebe o form

nesse evento os dados já foram salvos, pode ser usado para verificar as      informações

```js
function afterProcessing(form){
}
```

### 1.7 - beforeProcessing: primeiro evento a ser chamado, recebe o form

### 1.8 - beforeMovementOptions: é chamado ao chamar o botão      movimentar, antes de mosstrar as opções, recebe o número da atividade      atual

```js
<script>
  var beforeMovementOptions = function(numState) {
    console.log("_________________ beforeMovementOptions");
    console.log("numState: " + numState);
    console.log("valor campo: " + document.form.c7_total.value);
    if (document.form.c7_total.value == '') {
      throw ("Erro: nenhum valor selecionado!");
    }
    return true;
  }
</script>
```

### 1.9 - beforeSendValidade: é chamado antes de ser movimentado, após já      ter selecionado a atividade destino, recebe numero da atividade, e o      número da proxima

```js
<script>
var beforeSendValidate = function(numState,nextState){
    console.log("-beforeSendValidate-");
    console.log("numState: " + numState);
    console.log("nextState: " + nextState);
    throw("Erro Xyz");
}
</script>
```

```js
<script>
var beforeSendValidate = function(numState,nextState){
  console.log("-beforeSendValidate-");
  console.log("numState: " + numState);
  console.log("nextState: " + nextState);
  var isOk = confirm("Deseja realmente enviar o processo ?");
  return isOk;
 }
</script>
```

### 1.10 - Extras do FORM

#### 1.10.1 - getFormMode: retorna o modo atual do form

```js
form.getFormMode()
```

* ADD: inclusão
* MOD: edição
* VIEW: visualização
* NONE: qndo não há comunicação com o form, como na validação

#### 1.10.2 - mascarás

```html
<input name="cep" type="text" mask="00000-000">
```

> O fluig mobile não suporta o atributo mask.

|Código|Descrição|
|:---:|---|
|0| Somente Números|
|9| Somente Números mais opcional|
|\#| Somente números mais recursivo|
|A| Números ou letras|
|S| Somente letras entre A-Z e a-z|

#### 1.10.3 - combobox

```html
<select name="RNC_volume" id="RNC_volume" dataset="nome-dataset" datasetkey="chave" datasetvalue="valor" addBlankLine=”false”></select>
```
#### 10.10.4 - zoom

> precisa estar usando o style guide

```js
<input
    type="zoom"
    id = "c7_total"
    name="c7_total"
    data-zoom="{
        'displayKey':'colleagueName',
        'datasetId':'colleague',
        'maximumSelectionLength':'2',
        'placeholder':'Escolha o usuário',
        'fields':[
            {
               'field':'colleagueId',
               'label':'ID'
            },{
              'field':'colleagueName',
              'label':'Nome',
              'standard':'true'
            },{
              'field':'login',
              'label':'Login'
            }
        ]
     }"
/>
```


## 2 -  Eventos Pai Filho

### 2.1 - getChildrenFromTable: evento que retorna os campos de um pai filho, passando o nome da table

### 2.2 - getChildrenIndexes: retorna os índices de uma tabela filha, passando o nome da table

```js
function validateForm(form){
  var indexes = form.getChildrenIndexes("tabledetailname");
  var total = 0;
  for (var i = 0; i < indexes.length; i++) {
    var fieldValue = parseInt(form.getValue("valor___" + indexes[i]));
    if (isNaN(fieldValue)){
        fieldValue = 0;
    }
    total = total + fieldValue;
    log.info(total);
  }
  log.info(total);
  if (total < 100) {
    throw "Valor Total da requisição não pode ser inferior a 100";
  }
}
```

> onde
* type: o atributo type para este componente obrigatoriamente é 'zoom'
* name: nome do campo
* data-zoom: parâmetros do zoom em formato json onde:
  * maximumSelectionLength: limite de registros selecionáveis, caso não seja informado, o valor padrão é 1.
  * resultLimit: número máximo de resultados que serão listados na busca, o valor padrão é 300.
  * placeholder: texto de placeholder, que irá aparecer no zoom. Pode ser utilizado para instrução.
    * displayKey: coluna filtrável e de exibição após selecionado o registro
    * filterValues: atributo do dataset e valor para serem filtrados. Devem ser colocados em pares, separados por vírgula (,) onde o primeiro valor é o nome do campo e o segundo refere-se ao valor do campo.
  * datasetId ou cardDatasetId: opte por uma das opções:
    * datasetId:  é o nome do dataset (Built-in, CardIndex ou Customized).
    * cardDatasetId: é o numero de outro formulário para consulta.
    * fields: Estrutura do filtro
      * field: atributo do dataset que será utilizado.
      * label: descrição da coluna.
      * standard: a coluna que será utilizada como ordenação padrão e valor do registro selecionado.

> exemplo de zoom no [git](https://git.fluig.com/projects/SAMPLES/repos/projetos/browse/form-smart-zoom)

### 2.3 - enableFields: é possivel usar o enableFields para os filhos, precisa o indice da linha

```js
function enableFields(form){
  var indexes = form.getChildrenIndexes("ingredientes");
  for (var i = 0; i < indexes.length; i++) {
    form.setEnabled("quantidade___" + indexes[i], false);
    form.setEnabled("unidade___" + indexes[i], false);
    form.setEnabled("produto___" + indexes[i], false);
  }
}
```

## 3 - Eventos de Processos

### 3.1 - beforeStateEntry: evento para validar movimentação do processo, recebe a proxima atividade

### 3.2 beforeTaskCreate: acontece antes que o usuário receba uma tarefa, recebe a matricula do usuário

### 3.3 afterTaskCreate: acontece depois do usuário receber a tarefa, recebe a matricula do usuário

### 3.4 afterStateSEntry: ocorre após entrar na atividade, recebe a atividade

> Este não retorna erros para a tela

### 3.5 beforeSendData: ultimo evento, é possivel integrar com analytics, podendo enviar dados do processo

### 3.6 validateAvailableStates: ocorre após montada a lista de tarefas disponíveis para o usuário

> é possivel modificar a lista

```js
function validateAvailableStates(iCurrentState, stateList) {
  // Código: 1 - Descrição: Atividade inicial
  // Código: 2 - Descrição: Atividade ordem 3
  // Código: 3 - Descrição: Atividade ordem 2
  // Código: 4 - Descrição: Atividade ordem 1
    
  // stateList atual: [2,3,4]

  var stateArray = new Array();
    
  if (iCurrentState == 1) {
    stateList.clear();
    stateArray.push(4,3,2);
  }
    
  stateArray.forEach(function(code) {
    stateList.add(new java.lang.Integer(code));
  });
    
  // stateList reordenado: [4,3,2]
  return stateList;
}
```

### 3.7 beforeTaskSave: ocorre antes de salvar as informações do usuário

### 3.8 afterProcessCreate: após criação do processo

### 3.9 beforeTaskComplete: ocorre antes que o usuário complete uma tarefa, mas os dados de próxima tarefa e usuário destino ja salvas

### 3.10 afterTaskComplete: ocorre após o usuário completar uma tarefa, porém as informações de proxima tarefa e usuário destino já foi salva

### 3.11 beforeStateLeave: antes de sair da atividade

### 3.12 afterStateLeave: depois de sair da atividade

### 3.13 checkComplementsPermission: é possivel ver se o usuário pode ou não adicionar complementos, mesmo se ja tiver permissão

> evento não é executado na abertura da solicitação, e não impede o responsável de adicionar anexos e observações

> esse evento não permite dar permissões extras, apenas uma validação mais especifica

```js
function checkComplementsPermission() {
    var user = getValue("WKUser");
    var company = getValue("WKCompany");
    var group = "Auditoras";
    var Id = DatasetFactory.createConstraint('colleagueGroupPK.colleagueId',
            user, user, ConstraintType.MUST);
    var group = DatasetFactory.createConstraint('colleagueGroupPK.groupId',
            group, group, ConstraintType.MUST);
    var company = DatasetFactory.createConstraint('colleagueGroupPK.companyId',
            company, company, ConstraintType.MUST);
    var colleagueGroup = DatasetFactory.getDataset('colleagueGroup', null,
            new Array(Id, group, company), null);
    if (colleagueGroup != null && colleagueGroup.getRowsCount() == 1) {
        return true;
    } else {
        return false
    }
  
}
```

### 3.14 subProcessCreated: qndo um sub-processo é criado

### 3.15 afterProcessFinish: qndo um processo é finalizado

### 3.16 beforeCancelProcess: antes do cancelamento da solicitação

### 3.17 afterCancelProcess: após o cancelamento,

> não disparece ações aqui, pois já foi cancelado o processo

## hAPI - api de workflow

> Em todos os eventos do processo é possível obter informações da API de Workflow. Cada evento possui acesso ao handle da API de workflow pela variável global hAPI. Os seguintes métodos estão disponíveis através da hAPI:
---
|método|Especificação
|---|---
getCardValue("nomeCampo")|Permite acessar o valor de um campo do formulário do processo, onde: nomeCampo = nome do campo do formulário. 

* nomeCampo: nome do campo do formulário. 
```js
var campoCheckbox = hAPI.getCardValue("campoCheckbox") == "on" ? true : false;
```
---
|método|Especificação
|---|---
setCardValue("nomeCampo", "valor")|Permite definir o valor de um campo do formulário do processo, onde:nomeCampo: nome do campo do formulário; valor: valor a ser definido para o campo do formulário.

* nomeCampo: nome do campo do formulário;
* valor: valor a ser definido para o campo do formulário.
---
|método|Especificação
|---|---
setAutomaticDecision(numAtiv, listaColab, "obs")| **A propriedade automaticTasks esta depreciada não havendo mais suporte a partir da atualização 1.5.9 do fluig.** É recomendada a utilização da atividade de [Serviço](http://tdn.totvs.com/pages/releaseview.action?pageId=237397494) ou Gateway Exclusivo.
---
|método|Especificação
|---|---
getActiveStates()|Retorna uma lista das atividades ativas do processo.
---
|método|Especificação
|---|---
getActualThread(numEmpresa, numProcesso, numAtiv)|Retorna a thread da atividade que está ativa, lembrando que em caso de atividades paralelas, retorna 0, 1, 2 e assim sucessivamente.

* numEmpresa: número da empresa;
* numProcesso: número da solicitação;
* numAtiv: número da atividade.
> Exemplo de uso para esta função:

```js
function afterTaskCreate(colleagueId) {
  
    var nrProxAtividade = getValue("WKNextState");
    if (nrProxAtividade == "5"){ //atividade entre paralelas
  
        var data = new Date();
        var numEmpresa = getValue("WKCompany");
      
        //seta o dia, mês (Janeiro é 0) e ano
        data.setDate(20);
        data.setMonth(10);
        data.setFullYear(2010);
         
        // Recupera o numero da solicitação
        var numProcesso = getValue("WKNumProces");
      
        // Seta o prazo para as 14:00
        hAPI.setDueDate(numProcesso, hAPI.getActualThread(numEmpresa, numProcesso, nrProxAtividade), colleagueId, data, 50400);
    }
}
```
---
|método|Especificação
|---|---
setDueDate(numProcesso, numThread, "userId", dataConclusao, tempoSeg)|Permite alterar o prazo de conclusão para uma determinada atividade do processo

* numProcesso: número da solicitação;
* numThread: número da thread (normalmente 0, quando não se utiliza atividades paralelas);
* userId: o usuário responsável pela tarefa;
* dataConclusao: a nova data de conclusão;
* tempoSeg: tempo que representa a nova hora de conclusão, calculado em segundos após a meia-noite.


```js
function afterTaskCreate(colleagueId) {
    var atividade = getValue('WKCurrentState');
    // Atividade de sequência 5 é a da tarefa criada e que vou alterar o prazo de conclusão
    if (atividade == 5) {
        // Recuperando a data informada no campo do formulário
        var prazoFormulario = hAPI.getCardValue('prazoConclusao');
        if (prazoFormulario != undefined && prazoFormulario != '') {
            var numeroDaSolicitacao = getValue('WKNumProces');
            var threadDaSolicitacao = 0; // Normalmente 0, quando não for atividade paralela
            var responsavelPelaTarefa = colleagueId;
             
            /* Nesse caso o formato da data salva pelo formulário no exemplo é DD/MM/AAAA, mas isso pode variar de acordo com a formatação utilizada, 
               mudando assim as posições das informações dentro do array */
             
            /* Extrai os dados da data do formulário para um array, para posteriormente transformar em data do Javascript */
            var arrayPrazoConclusao = prazoFormulario.split("/");
            var dia = arrayPrazoConclusao[0]; // Posição 0 do array é o dia
            var mes = arrayPrazoConclusao[1] - 1; // Posição 1 do array é o mês (Subtraímos 1 porque na data do Javascript o mês vai de 0 a 11)
            var ano = arrayPrazoConclusao[2]; // Posição 2 do array é o ano
             
            var horaDoPrazo = (24*60*60) - 1; /* A hora é em milisegundos, e esse cálculo tem resultado de 23:59:59, ou seja, 
            o prazo de conclusão vai ser até o último segundo do dia informado no formulário */
             
            // Cria a data no Javascript
            var dataDoPrazo = new Date();
            dataDoPrazo.setDate(dia);
            dataDoPrazo.setMonth(mes);
            dataDoPrazo.setFullYear(ano);
             
            // Altera o prazo de conclusão
            hAPI.setDueDate(numeroDaSolicitacao, threadDaSolicitacao, responsavelPelaTarefa, dataDoPrazo, horaDoPrazo);
        }
    }  
}
```
---

|método|Especificação
|---|---
transferTask(transferUsers, "obs", int numThread)|Transfere uma tarefa de um usuário para outro(s) usuário(s).

* transferUsers: lista (do tipo String) de usuários;
* obs: a observação;
* numThread: sequência da thread, em caso de atividades paralelas.

---

|método|Especificação
|---|---
transferTask(transferUsers, "obs")|Transfere uma tarefa de um usuário para outro(s) usuário(s). Este método não pode ser usado em processos com atividades paralelas:

* transferUsers: lista (do tipo String) de usuários;
* obs: a observação.

---
|método|Especificação
|---|---
startProcess(processId, ativDest, listaColab, "obs", completarTarefa, valoresForm, modoGestor)|Inicia uma solicitação workflow, onde:

* processId: código do processo;
* ativDest: código da atividade de destino;
* listaColab: lista (do tipo String) de usuários;
* obs: texto da observação;
* completarTarefa: indica se deve completar a tarefa (true) ou apenas salvar (false);
* valoresForm: um Mapa com os valores do formulário do * processo;
modoGestor: indica que o usuário iniciará a solicitação como gestor (true) ou que o usuário iniciará a solicitação apenas como solicitante (false).

> Retorna um mapa com informações da solicitação criada. Entre elas, o iProcess que é o número da solicitação criada.

> Exemplo de inicialização de uma solicitação pelo método hAPI.startProcess enviando a atividade para um papel:

```js
function beforeStateEntry(sequenceId){
     
    if (sequenceId == 5) {
        //A tarefa destino tem o mecanismo de atribuição para um papel, cujo o código é papelUser
        var users = new java.util.ArrayList();
        users.add("Pool:Role:papelUser");
         
        var formData = new java.util.HashMap();
  
        formData.put("Nome_do_Campo1", "Valor do Campo 1");
        formData.put("Nome_do_Campo2", "Valor do Campo 2");
                         
        hAPI.startProcess("pool", 4, users, "Solicitação inicializada pela função hAPI", true, formData, false);
    }
     
}
```

---
|método|Especificação
|---|---
setColleagueReplacement(userId)|Seta um usuário substituto, onde:

* userId: código do usuário substituto.

--- 
|método|Especificação
|---|---
setTaskComments("userId", numProcesso,  numThread, "obs")|Define uma observação para uma determinada tarefa do processo, onde:
* userId: usuário responsável pela tarefa;
* numProcesso: número da solicitação de processo;
* numThread: é o número da thread (normalmente 0, quando não se utiliza atividades paralelas);
* obs: a observação.

>**É recomendado utilizar em eventos do tipo 'after', pois o comentário será criado no histórico da solicitação, então é necessário que já exista uma movimentação do processo para atribuir este comentário.**

--- 
|método|Especificação
|---|---
getCardData(numProcesso)|Retorna um Mapa com todos os campos e valores do formulário da solicitação.

* numProcesso: número da solicitação de processo.

O retorno do mapa seria:

campo|valor
---|---
|numNota | 99999
|codItem___1 | 91
|desItem___1 | Caneta
|qtdItem___1 | 100
|codItem___2 | 92
|desItem___2 | Lápis
|qtdItem___2 | 200
|codItem___3 | 93
|desItem___3 | Borracha
|qtdItem___3 | 150

--- 
|método|Especificação
|---|---
getAdvancedProperty("propriedade")|Retorna o valor da propriedade avançada de um processo.

* propriedade: nome da propriedade avançada. 

--- 
|método|Especificação
|---|---
calculateDeadLineHours(data, segundos, prazo, periodId)|Calcula um prazo a partir de uma data com base no expediente e feriados cadastrados no produto passando o prazo em horas:

* data: data inicial (tipo Date);
* segundos: quantidade de segundos após a meia noite;
* prazo: prazo que será aplicado em horas (tipo int);
* periodId: código de expediente.  
* Retorno: Array de Objeto, onde a primeira posição do array é a data e a segunda a hora.

> Exemplo:

```js
function afterTaskCreate(colleagueId) {
    var data = new Date();
  
    //Calcula o prazo
    var obj = hAPI.calculateDeadLineHours(data, 50000, 2, "Default");
    var dt = obj[0];
    var segundos = obj[1];
  
    //Recupera o numero da solicitação
    var processo = getValue("WKNumProces");
  
    //Altera o prazo do processo
    hAPI.setDueDate(processo,0,colleagueId, dt, segundos);
}
```

--- 
|método|Especificação
|---|---
calculateDeadLineTime(data, segundos, prazo, periodId)|Calcula um prazo a partir de uma data com base no expediente e feriados cadastrados no produto passando o prazo em minutos:

* data: data inicial (tipo Date);
* segundos: quantidade de segundos após a meia noite;
* prazo: prazo que será aplicado em minutos (tipo int);
* periodId: código de expediente.
> Retorno: Array de Objeto, onde a primeira posição do array é a data e a segunda a hora.

> Exemplo:

```js
function afterTaskCreate(colleagueId) {
    var data = new Date();
    //Calcula o prazo
    var obj = hAPI.calculateDeadLineTime(data, 50000, 120, "Default");
    var dt = obj[0];
    var segundos = obj[1];
  
    //Recupera o numero da solicitação
    var processo = getValue("WKNumProces");
  
    // Altera o prazo do processo
    hAPI.setDueDate(processo,0,colleagueId, dt, segundos);
}
```

--- 
|método|Especificação
|---|---
addCardChild(tableName, cardData)|Adiciona um filho no formulário pai e filho do processo, onde:

* tableName: nome da tabela filha onde será criado o filho;
* cardData: mapa com os campos do filho e seus valores.


> Exemplo:

```js
function beforeStateEntry(sequenceId) {
    if (sequenceId == 4) {
        var childData = new java.util.HashMap();
        childData.put("matricula", "0041");
        childData.put("nome", "João Silva");
        childData.put("cpf", "44455889987");
        hAPI.addCardChild("funcionarios", childData);
    }
}
```


---

Exemplo de integração onde após completar uma tarefa, um usuário no fluig é criado

```js
function afterTaskComplete(colleagueId, nextSequenceId, userList) {
      
    if (nextSequenceId == 2) {
        //Busca o webservices de Colaborador
        //Servico "<url_fluig>/webdesk/ECMColleagueService?wsdl" cadastrado com o código "Colleague"
        var colleagueServiceProvider = ServiceManager.getServiceInstance("Colleague");
        var colleagueServiceLocator = colleagueServiceProvider.instantiate("com.totvs.technology.ecm.foundation.ws.ECMColleagueServiceService");
        var colleagueService = colleagueServiceLocator.getColleagueServicePort();
      
        //Cria o ColleagueDto – Verificar a lista de métodos na visualização do serviço
        var colleagueDto = colleagueServiceProvider.instantiate("com.totvs.technology.ecm.foundation.ws.ColleagueDto");
        colleagueDto.setCompanyId(1);
        colleagueDto.setColleagueId("teste");
        colleagueDto.setColleagueName("Usuario Teste");
        colleagueDto.setActive(true);
        colleagueDto.setVolumeId("Default");
        colleagueDto.setLogin("teste");
        colleagueDto.setMail("teste@empresa.com");
        colleagueDto.setPasswd("teste");
        colleagueDto.setAdminUser(false);
        colleagueDto.setEmailHtml(true);
        colleagueDto.setDialectId("pt_BR");
          
        //Cria o colleagueDtoArray e adiciona
        var colleagueDtoArray = colleagueServiceProvider.instantiate("com.totvs.technology.ecm.foundation.ws.ColleagueDtoArray");
        colleagueDtoArray.getItem().add(colleagueDto);
      
        var result = colleagueService.createColleague("adm", "adm",  1, colleagueDtoArray);
        log.info("Result: " + result);
    }
}
```

> Abaixo um outro exemplo utilizando o WebService ECMCardService para alterar o valor do campo de um registro de formulário após a entrada em uma nova atividade:

```js
function afterStateEntry(sequenceId){
    if (sequenceId == 2) {
        //Servico "<url_fluig>/webdesk/ECMCardService?wsdl"cadastrado com o código "CardService"
        var cardServiceProvider = ServiceManager.getServiceInstance("01");
        var cardServiceLocator = cardServiceProvider.instantiate("com.totvs.technology.ecm.dm.ws.ECMCardServiceService");
        var cardService = cardServiceLocator.getCardServicePort();
        var cardFieldDtoArray = cardServiceProvider.instantiate("com.totvs.technology.ecm.dm.ws.CardFieldDtoArray");
        var cardField = cardServiceProvider.instantiate("com.totvs.technology.ecm.dm.ws.CardFieldDto");
      
        //Seta valor no campo com name = 'nome'
        cardField.setField("nome");
        cardField.setValue("Valor alterado via WS dentro de um evento workflow");
  
        var vetCardFields = new Array();
        vetCardFields.push(cardField);
        cardFieldDtoArray.getItem().addAll(vetCardFields);
      
        //Altera o(s) campo(s) do registro de formulário.
        //updateCardData(tenantId, login, senha, codRegistroForm, cardFieldDtoArray);
        cardService.updateCardData(1, "adm", "adm", 8, cardFieldDtoArray);
    }
}
```



a partir de um campo imput, nomear o responsável pela atividade.
atribuir o prazo por um campo imput tmb enfim
integração datasul
