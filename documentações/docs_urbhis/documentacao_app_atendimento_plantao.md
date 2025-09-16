# Documentação do Aplicativo: URBHIS - Atendimento de Plantão Social

## 1. Visão Geral

Este documento detalha a estrutura e o funcionamento do aplicativo "URBHIS - Atendimento de Plantão Social", desenvolvido em Power Apps.

O objetivo do aplicativo é gerenciar o ciclo completo de atendimento social realizado em plantões. Ele permite que os assistentes sociais cadastrem e consultem beneficiários, registrem novos atendimentos, façam encaminhamentos e consultem o histórico de serviços prestados. O sistema possui um controle de acesso para garantir que apenas usuários autorizados possam realizar os atendimentos.

## 2. Fontes de Dados

O aplicativo se conecta a várias listas do SharePoint para armazenar e consultar dados:

-   **`dbCadastroFuncionario`**: Lista de funcionários da URBHIS, usada para verificar as permissões de acesso do usuário logado.
-   **`dbOS`**: Contém as Ordens de Serviço, de onde são filtrados os plantões planejados (`IdProduto = "P04"`).
-   **`dbCadastroBeneficiario`**: Armazena os dados cadastrais de todos os beneficiários (CPF, nome, endereço, etc.).
-   **`dbAtendimentos`**: Tabela principal que armazena cada registro de atendimento social realizado.
-   **`dbMotivoAtendimento`**: Lista com os possíveis motivos para um atendimento.
-   **`dbProdutos`**: Usada para listar os tipos de encaminhamento (`IdProduto = "P05"` ou `P06`).
-   **`bdNucleos`**: Lista de núcleos de atendimento.
-   **Outras:** `dbGenero`, `dbRacaEtnia`, `dbEstadoCivil`, `dbUnidadeFamiliar` (tabelas de apoio para os formulários).

## 3. Controle de Acesso e Inicialização (App OnStart)

Ao iniciar, o aplicativo verifica o `TipoAutorizacao` do usuário logado na base `dbCadastroFuncionario`.

-   **Lógica:** A variável global `Autorizacao_Atendimento` é definida como `true` se o tipo de autorização do usuário for "Master", "Social", "Social Nivel 2" ou "Social Nivel 3".
-   **Impacto:** Esta variável controla a visibilidade dos botões de navegação na `Home Screen`, garantindo que apenas pessoal autorizado possa acessar as funcionalidades de atendimento.

## 4. Estrutura do Aplicativo e Telas

O aplicativo é composto por um fluxo de várias telas, cada uma com uma responsabilidade clara no processo de atendimento.

---

### 4.1. Home Screen

É a tela de boas-vindas e o ponto de partida para os usuários autorizados.

-   **Objetivo:** Apresentar uma saudação personalizada e fornecer o acesso inicial ao sistema.
-   **Componentes e Lógica:**
    -   Exibe uma mensagem de boas-vindas com o nome do usuário (`User().FullName`).
    -   **`Button "INICIAR ATENDIMENTO"`**: Visível apenas se `Autorizacao_Atendimento` for `true`. Navega para a `Atendimentos Screen`, que é a tela principal de operações.

---

### 4.2. Atendimentos Screen

É o painel central do aplicativo, onde o assistente social gerencia os atendimentos.

-   **Objetivo:** Selecionar o plantão, buscar beneficiários, iniciar novos atendimentos e acessar as funções de cadastro.
-   **Componentes e Lógica:**
    -   **`cb_Reuniao_3` (ComboBox "PLANTÃO DE ATENDIMENTO"):** O usuário seleciona o plantão atual. A lista é populada com os plantões de status "Planejado" da base `dbOS`.
    -   **`inpCPF_Busca_Mascara_1` (Input "PESQUISAR CPF"):** Campo para digitar o CPF do beneficiário.
    -   **`Button "BUSCAR CPF"`**:
        -   Verifica se um plantão e um CPF foram informados.
        -   Busca o CPF na base `dbCadastroBeneficiario` e armazena o resultado na coleção `colBeneficiario`.
        -   Exibe os dados cadastrais do beneficiário encontrado (Nome, CPF, Núcleo, Selo, etc.) na parte inferior da tela.
    -   **`Button "CADASTRAR CPF"`**: Navega para a `Tela Cadastro Screen` para registrar um novo beneficiário.
    -   **`Button "EDITAR CADASTRO"`**: Navega para a `Tela Edição Cadastro Screen`, permitindo alterar os dados do beneficiário que foi buscado.
    -   **`Button "INICIAR ATENDIMENTO"`**: Fica visível após a busca de um CPF e navega para a `Tela Atendimento Screen` para registrar um novo atendimento para aquele beneficiário.
    -   **`Button "HISTÓRICO DE ATENDIMENTOS"`**: Navega para a `Tela Historico Atendimentos`.

---

### 4.3. Tela Cadastro Screen

Formulário para a inclusão de novos beneficiários no sistema.

-   **Objetivo:** Coletar e salvar os dados de um beneficiário que ainda não possui cadastro.
-   **Componentes e Lógica:**
    -   **`FormCad_Beneficiario` (Formulário):**
        -   **Fonte de Dados:** `dbCadastroBeneficiario`.
        -   **Modo:** `FormMode.New`.
        -   **Campos:** Inclui CPF, Nome Completo, Data de Nascimento, Gênero, Endereço, Contato, Vínculo Familiar, etc. Campos obrigatórios são validados antes do envio.
    -   **`Button "CADASTRAR BENEFICIÁRIO"`**: Executa `SubmitForm(FormCad_Beneficiario)` para salvar os dados. Realiza uma validação para garantir que os campos obrigatórios (`CPF`, `NomeCompleto`, `TelefoneContato`, etc.) foram preenchidos.

---

### 4.4. Tela Edição Cadastro Screen

Formulário para a edição dos dados de um beneficiário existente.

-   **Objetivo:** Permitir a correção e atualização das informações cadastrais de um beneficiário.
-   **Componentes e Lógica:**
    -   **`FormCad_Beneficiario_1` (Formulário):**
        -   **Fonte de Dados:** `dbCadastroBeneficiario`.
        -   **Modo:** `FormMode.Edit`. O item a ser editado é definido pela variável `locPP_EditCadastro`, que foi populada na `Atendimentos Screen`.
        -   **Campos:** Os mesmos da tela de cadastro, mas pré-preenchidos com os dados do beneficiário selecionado.
    -   **`Button "EDITAR CADASTRO"`**: Executa `SubmitForm` para salvar as alterações no registro do beneficiário.

*(Observação: O arquivo `Tela Edição Screen.fx.yaml` parece ser uma duplicata ou versão antiga da `Tela Edição Cadastro Screen` e não é utilizado no fluxo principal de navegação.)*

---

### 4.5. Tela Atendimento Screen

Formulário para registrar um novo atendimento para um beneficiário.

-   **Objetivo:** Documentar os detalhes de um atendimento social.
-   **Componentes e Lógica:**
    -   **`FormCad_AtendimentoPlantao` (Formulário):**
        -   **Fonte de Dados:** `dbAtendimentos`.
        -   **Modo:** `FormMode.New`.
        -   **Campos:** Inclui Local, Tipo e Motivo do Atendimento, Data, Descrição detalhada (campo Rich Text), Encaminhamento e um campo para **Assinatura** (`PenInput`).
        -   **Lógica Oculta:** Campos como `NomeCompleto`, `CPF`, `Nucleo`, `Selo`, `Plantao` e `OSNumero` são preenchidos automaticamente com base no beneficiário e no plantão selecionados nas telas anteriores.
    -   **`Button "CONCLUIR ATENDIMENTO"`**:
        -   Valida se os campos principais foram preenchidos.
        -   Salva o novo registro na base `dbAtendimentos` usando a função `Patch`, definindo o `StatusAtendimento` como "Planejado".
        -   Limpa as variáveis e retorna para a `Atendimentos Screen`.

---

### 4.6. Tela Atendimento Edição Screen

Formulário para visualizar e editar um registro de atendimento existente.

-   **Objetivo:** Corrigir ou adicionar informações a um atendimento já registrado.
-   **Componentes e Lógica:**
    -   **`FormCad_AtendimentoPlantao_1` (Formulário):**
        -   **Fonte de Dados:** `dbAtendimentos`.
        -   **Modo:** `FormMode.Edit`. O item a ser editado é definido pela variável `locPP_EditAtendimentoGeral`, populada na `Tela Historico Atendimentos`.
        -   **Campos:** Os mesmos da tela de criação de atendimento, pré-preenchidos com os dados do registro selecionado.
    -   **`Button "CONCLUIR ATENDIMENTO"`**: Executa `SubmitForm` para salvar as alterações no registro de atendimento.

---

### 4.7. Tela Historico Atendimentos

Tela para consulta e filtragem do histórico de atendimentos.

-   **Objetivo:** Permitir que o assistente social visualize os atendimentos que realizou, com opções de filtro.
-   **Componentes e Lógica:**
    -   **Filtros:** Permite filtrar os atendimentos por `Núcleo`, `Nome Completo` do beneficiário e `CPF`. A busca é realizada dinamicamente conforme o usuário digita nos campos.
    -   **`gal_EstadoCivil_23` (Galeria):**
        -   **Itens:** Exibe os registros da coleção `colAtendimentoRealizado`, que é filtrada com base nos critérios de busca e no usuário logado (`'Criado por'.Email = User().Email`).
        -   **Ação:** Ao clicar em um item do histórico, o usuário é levado para a `Tela Atendimento Edição Screen` para ver ou editar os detalhes daquele atendimento.
