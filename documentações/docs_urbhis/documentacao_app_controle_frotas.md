# Documentação do Aplicativo: URBHIS - Controle de Frotas

## 1. Visão Geral

Este documento detalha a estrutura e o funcionamento do aplicativo "URBHIS - Controle de Frotas", desenvolvido em Power Apps.

O objetivo principal do aplicativo é permitir que os condutores da URBHIS registrem e gerenciem os trajetos realizados com os veículos da frota. Ele captura informações essenciais como quilometragem inicial e final, motivo da viagem, despesas com pedágio e registros fotográficos dos odômetros, garantindo um controle preciso e auditável do uso dos veículos.

## 2. Fontes de Dados

O aplicativo utiliza as seguintes fontes de dados (listas do SharePoint):

-   `dbLancamento_KM`: Tabela principal onde todos os registros de viagens e quilometragem são armazenados.
-   `dbFrota`: Contém a lista de veículos da frota, incluindo informações como placa, marca e modelo.
-   `dbCadastroFuncionario`: Armazena os dados dos funcionários, indicando quais deles são condutores autorizados.
-   `dbProdutos`: Tabela com a lista de produtos.

## 3. Estrutura do Aplicativo e Telas

O aplicativo é composto por três telas principais, cada uma com uma função específica no fluxo de controle de frotas.

---

### 3.1. Tela Inicial (`Tela Inicial Screen`)

É a porta de entrada do aplicativo, onde o usuário inicia o processo de registro ou consulta.

#### **Objetivo**

-   Identificar o condutor.
-   Exibir um histórico de lançamentos de KM para o condutor selecionado.
-   Fornecer acesso para criar um novo lançamento.

#### **Componentes Principais**

-   **`Select_Nome_Condutor` (ComboBox):**
    -   **Função:** Lista todos os funcionários marcados como "Condutor" na base `dbCadastroFuncionario`.
    -   **Lógica:** O usuário deve se selecionar nesta lista para que seus registros de viagem sejam exibidos. A galeria de registros permanece vazia até que um condutor seja selecionado.

-   **`RecordsGallery1` (Galeria):**
    -   **Função:** Exibe os lançamentos de KM associados ao condutor selecionado.
    -   **Lógica:** Filtra a base `dbLancamento_KM` pelo nome do condutor selecionado. Cada item na galeria mostra o título do lançamento (Data - Motivo), a placa e o modelo do veículo. Ao selecionar um item, o usuário é redirecionado para a tela de edição (`Form Edição Screen`).

-   **`Button1` ("NOVO LANÇAMENTO"):**
    -   **Função:** Inicia o processo de criação de um novo registro de viagem.
    -   **Lógica:** Antes de navegar, o botão verifica se um condutor foi selecionado. Caso contrário, exibe uma notificação. Se um condutor estiver selecionado, ele navega para a `Form Lançamento Screen` e prepara o formulário para a criação de um novo item (`NewForm(Form_LancamentoKM)`).

---

### 3.2. Tela de Lançamento (`Form Lançamento Screen`)

Este formulário é usado para registrar uma nova viagem.

#### **Objetivo**

-   Coletar todos os dados de um novo trajeto, como data, hora, motivo, locais, quilometragem e despesas.

#### **Componentes Principais**

-   **`Form_LancamentoKM` (Formulário):**
    -   **Fonte de Dados:** `dbLancamento_KM`.
    -   **Modo:** Iniciado em modo "Novo" (`NewForm`).
    -   **Campos:**
        -   `Data da Viagem/Trabalho`: Data da viagem.
        -   `Placa do Veículo`: Dropdown com as placas da base `dbFrota`.
        -   `Hora de início` e `Hora de conclusão`: Horários de início e fim.
        -   `Local da Saída` e `Local da Chegada`: Campos de texto para os endereços.
        -   `Odómetro Inicial (KM)` e `Odómetro Final (KM)`: Campos numéricos para a quilometragem.
        -   `Odómetro Inicial` e `Odómetro Final` (Imagem): Controles para anexar fotos do odômetro.
        -   `Este trajeto gerou despesas de pedágio`: Um seletor (toggle) que, quando ativado, exibe campos para detalhar os custos com pedágio.
        -   `Anexos`: Para comprovantes de pedágio ou outros documentos.
    -   **Campos Ocultos (Lógica Automática):**
        -   `Título`: É preenchido automaticamente com a data e o motivo da viagem, servindo como identificador do registro.
        -   `Condutor`: Preenchido com o nome do condutor selecionado na `Tela Inicial`.
        -   `Quilometragem percorrida`: Calculado automaticamente pela diferença entre o odômetro final e o inicial.

-   **`IconAccept1` (Ícone de Check):**
    -   **Função:** Salva o novo registro.
    -   **Lógica:** Executa `SubmitForm(Form_LancamentoKM)`, que envia os dados para a lista `dbLancamento_KM`, e depois retorna para a `Tela Inicial Screen`.

-   **`IconCancel1` (Ícone de Cancelar):**
    -   **Função:** Cancela a criação do registro.
    -   **Lógica:** Executa `ResetForm(Form_LancamentoKM)` para limpar os dados não salvos e retorna para a `Tela Inicial Screen`.

---

### 3.3. Tela de Edição (`Form Edição Screen`)

Permite a visualização e alteração de um lançamento já existente.

#### **Objetivo**

-   Visualizar os detalhes de um registro de viagem selecionado.
-   Corrigir ou atualizar informações de um lançamento.

#### **Componentes Principais**

-   **`Form_EdicaoKM` (Formulário):**
    -   **Fonte de Dados:** `dbLancamento_KM`.
    -   **Lógica do Item:** O formulário é preenchido com os dados do item que foi selecionado na galeria da `Tela Inicial` (`Item: =RecordsGallery1.Selected`).
    -   **Campos:** Apresenta os mesmos campos da tela de lançamento, mas já preenchidos com as informações do registro existente, permitindo a edição.

-   **`IconAccept1_1` (Ícone de Check):**
    -   **Função:** Salva as alterações feitas no registro.
    -   **Lógica:** Executa `SubmitForm(Form_EdicaoKM)` para atualizar o item na base `dbLancamento_KM` e retorna à `Tela Inicial Screen`.

-   **`IconCancel1_1` (Ícone de Cancelar):**
    -   **Função:** Descarta as alterações.
    -   **Lógica:** Retorna para a `Tela Inicial Screen` sem salvar nenhuma modificação.

## 4. Tema e Configurações Gerais (`Themes.json` e `App.fx.yaml`)

-   **`App.fx.yaml`**: Define configurações globais do aplicativo.
    -   `BackEnabled: =true`: Permite que o usuário utilize o botão "Voltar" do dispositivo ou navegador.
    -   `Theme: =PowerAppsTheme`: Aplica o tema de cores e estilos definido no arquivo `Themes.json`.

-   **`Themes.json`**: Contém as definições de design do aplicativo, como a paleta de cores (tons de roxo e cinza, como `RGBA(122, 111, 150, 1)`), fontes (`Font.Arial`) e estilos para os diferentes controles (botões, labels, etc.). Isso garante uma aparência consistente em todas as telas.
