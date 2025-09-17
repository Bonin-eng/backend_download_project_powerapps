# Documentação do Aplicativo: Lançamento de Recibos

## 1. Visão Geral do Aplicativo

O aplicativo **Lançamento de Recibos** é uma ferramenta desenvolvida em Power Apps para que os usuários possam registrar, gerenciar e acompanhar suas compras e despesas de forma digital. Cada usuário tem acesso apenas aos seus próprios lançamentos, garantindo a privacidade e a organização das informações.

As principais funcionalidades são:
- **Lançamento de Novos Recibos:** Permite que o usuário crie um novo registro de compra, inserindo detalhes como o item comprado, motivo, valor, data e forma de pagamento.
- **Anexo de Comprovantes:** O usuário pode anexar uma imagem do recibo (via câmera ou galeria) e outros arquivos relevantes para cada lançamento.
- **Histórico Pessoal:** A tela inicial exibe uma lista com todos os recibos já lançados pelo usuário logado.
- **Edição de Lançamentos:** É possível selecionar um recibo existente para corrigir ou atualizar suas informações.
- **Automação de Processos:** Ao submeter um novo recibo, o aplicativo aciona um fluxo do Power Automate (`Bonin-EnviodeRecibo`) que provavelmente notifica um gestor ou inicia um processo de reembolso/aprovação.

## 2. Estrutura e Lógica Inicial

A configuração inicial e as lógicas globais do aplicativo estão definidas no arquivo `App.fx.yaml`.

### Lógica de Inicialização (`OnStart`)

Ao ser iniciado, o aplicativo executa as seguintes ações:

1.  **`ClearCollect(colLancamentosMensais, ...)`**: Filtra a fonte de dados principal (`dbRecibo_Compra`) para carregar em uma coleção (`colLancamentosMensais`) apenas os registros onde o campo `'Criado por'.Email` é igual ao e-mail do usuário logado (`User().Email`). Isso garante que cada usuário veja apenas seus próprios lançamentos.
2.  **`ClearCollect(Col_Numero_Extenso_Dicionario, ...)`**: Cria uma coleção estática que funciona como um dicionário para converter valores numéricos (de 1 a 499) em seu correspondente em texto por extenso em português. Esta coleção é usada posteriormente para gerar o valor do recibo em texto.

### Tema do Aplicativo (`Themes.json`)

O arquivo `Themes.json` define a identidade visual do aplicativo, estabelecendo uma paleta de cores, fontes e estilos para os controles, o que garante uma experiência de usuário consistente em todas as telas.

## 3. Telas do Aplicativo

O fluxo de navegação do aplicativo é composto por três telas principais.

### 3.1. Tela Inicial (`Tela Inicial Screen`)

É a tela principal do aplicativo, servindo como um dashboard para o usuário.

**Objetivo:**
Exibir o histórico de recibos do usuário e permitir a criação de novos lançamentos.

**Componentes e Lógica:**

-   **Botão `NOVO LANÇAMENTO`:**
    -   Ao ser clicado, navega para a `Form Lançamento Screen`.
    -   Inicia o formulário `Form_LancamentoKM` em modo de criação (`NewForm`).

-   **Galeria de Registros (`RecordsGallery1`):**
    -   **Itens:** Exibe os dados da coleção `colLancamentosMensais` (os recibos do usuário).
    -   **`OnSelect`:** Ao selecionar um item na galeria, o aplicativo navega para a `Form Edição Screen`, permitindo que o usuário veja e edite os detalhes do registro selecionado.

### 3.2. Tela de Lançamento (`Form Lançamento Screen`)

Esta tela é usada para registrar uma nova despesa.

**Objetivo:**
Coletar todas as informações sobre uma nova compra e submetê-la ao sistema.

**Componentes e Lógica:**

-   **Formulário (`Form_LancamentoKM`):**
    -   **Fonte de Dados:** `dbRecibo_Compra`.
    -   **Modo:** `FormMode.New` (sempre aberto para criar um novo item).
    -   **Campos:**
        -   `Compra`: Descrição do que foi comprado.
        -   `Motivo da Compra`: Justificativa da despesa.
        -   `Data da Compra`: Data em que a compra foi realizada.
        -   `Forma de Pagamento`: Lista de opções (ex: Cartão, Dinheiro).
        -   `Valor da Compra`: Custo total da despesa.
        -   `Recibo`: Campo para adicionar uma imagem do comprovante.
        -   `Anexos`: Campo para adicionar outros arquivos.
    -   **Lógica Oculta:**
        -   O campo `Title` (título do registro) é preenchido automaticamente com a concatenação da data e do motivo da compra.
        -   O campo `Valor_Extenso` converte o valor numérico da compra em texto por extenso, utilizando a coleção `Col_Numero_Extenso_Dicionario`.

-   **Ações da Tela:**
    -   **Ícone de Check (Salvar):**
        1.  **`'Bonin-EnviodeRecibo'.Run(...)`**: Executa um fluxo do Power Automate, enviando os dados do formulário (motivo, data, valor, etc.).
        2.  **`SubmitForm(Form_LancamentoKM)`**: Salva o novo registro na lista `dbRecibo_Compra` do SharePoint.
        3.  Após o sucesso (`OnSuccess`), a coleção local `colLancamentosMensais` é atualizada e o usuário é redirecionado para a `Tela Inicial Screen`.
    -   **Ícone de Cancelar:**
        -   Navega de volta para a `Tela Inicial Screen` e reinicia o formulário, descartando as alterações.

### 3.3. Tela de Edição (`Form Edição Screen`)

Permite a modificação de um recibo já existente.

**Objetivo:**
Visualizar e corrigir os dados de um lançamento previamente salvo.

**Componentes e Lógica:**

-   **Formulário (`Form_EdicaoRecibo`):**
    -   **Fonte de Dados:** `dbRecibo_Compra`.
    -   **Item:** `RecordsGallery1.Selected`. O formulário é preenchido com os dados do item que o usuário selecionou na tela inicial.
    -   Todos os campos preenchidos no lançamento estão disponíveis para edição.

-   **Ações da Tela:**
    -   **Ícone de Cancelar (Salvar):**
        -   **`OnSelect`:** A ação configurada para este ícone é `SubmitForm(Form_EdicaoRecibo)`. Portanto, ao contrário do que o ícone sugere, ele **salva** as alterações feitas no formulário.
        -   Após o sucesso (`OnSuccess`), a coleção `colLancamentosMensais` é atualizada para refletir as mudanças.

> **Nota:** O comportamento do botão de "Cancelar" na tela de edição parece ser um erro de implementação, pois ele executa a ação de salvar. O ideal seria que ele descartasse as alterações ou que houvesse um botão de "Salvar" separado.

## 4. Fontes de Dados e Integrações

-   **SharePoint - `dbRecibo_Compra`:** É a lista principal que armazena todos os dados dos recibos lançados pelos usuários. Contém colunas para cada campo do formulário, além de metadados como "Criado por" e "Data de criação".
-   **Power Automate - `Bonin-EnviodeRecibo`:** Um fluxo de automação que é acionado a partir da tela de lançamento. Sua função é pegar os dados do novo recibo e executar uma ação externa, como enviar um e-mail de notificação para aprovação, salvar uma cópia do recibo em outro local, ou integrar com outro sistema.
