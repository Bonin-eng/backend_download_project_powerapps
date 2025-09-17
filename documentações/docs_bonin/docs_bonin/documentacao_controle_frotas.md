# Documentação do Aplicativo: Controle de Frotas

## 1. Visão Geral do Aplicativo

O aplicativo **Sistema de Gestão Integrado - Controle de Frotas** é uma ferramenta desenvolvida em Power Apps para gerenciar de forma centralizada todos os aspectos da frota de veículos da empresa, bem como as informações administrativas relacionadas, como o cadastro de funcionários.

As principais funcionalidades incluem:
- **Gestão de Veículos:** Cadastro e acompanhamento detalhado de cada veículo da frota.
- **Administração da Frota:** Controle de custos, manutenções, multas, ocorrências (acidentes) e despesas de cartão combustível (Alelo).
- **Controle de Contratos:** Gerenciamento dos contratos de locação dos veículos.
- **Cadastro de Funcionários:** Módulo administrativo para registrar e gerenciar os dados dos funcionários, indicando quais são condutores.

O aplicativo foi projetado para otimizar a gestão, centralizar informações e fornecer uma visão clara e organizada sobre o status e os custos da frota.

## 2. Estrutura e Navegação

A estrutura de navegação e o comportamento inicial do aplicativo são definidos no arquivo `App.fx.yaml`.

### Lógica de Inicialização (`OnStart`)

Ao ser iniciado, o aplicativo executa as seguintes ações:
- **`ClearCollect(colTelas, ...)`**: Cria uma coleção chamada `colTelas` que serve como fonte de dados para o menu de navegação lateral. Essa coleção mapeia o nome da tela, o objeto da tela e o ícone a ser exibido.

As telas navegáveis definidas são:
1.  **`FROTA SCREEN`**: Tela principal para o gerenciamento da frota.
2.  **`ADMINISTRATIVO SCREEN`**: Tela para tarefas administrativas, como o cadastro de funcionários.

### Tema do Aplicativo (`Themes.json`)

O arquivo `Themes.json` define a identidade visual do aplicativo. Ele contém uma paleta de cores, fontes e estilos (para botões, rótulos, campos de entrada, etc.) que são aplicados de forma consistente em todas as telas, garantindo um design coeso e profissional.

## 3. Telas do Aplicativo

O aplicativo é composto por duas telas principais, cada uma com múltiplas funcionalidades e componentes.

### 3.1. Tela de Frota (`FROTA SCREEN`)

Esta é a tela central para todas as operações relacionadas à gestão da frota.

**Objetivo:**
Permitir o acompanhamento, cadastro e administração completa dos veículos e seus respectivos custos e eventos.

**Componentes e Lógica:**

A tela é dividida em seções, cuja visibilidade é controlada por variáveis de contexto (prefixo `locPP_`):

- **Menu de Navegação (`cMenuLateral_14`):** Componente reutilizável que permite a navegação entre as telas principais (`FROTA SCREEN` e `ADMINISTRATIVO SCREEN`).
- **Botões de Seção:**
    - **`CONTROLE CONTRATO`**: Ativa a variável `locPP_ContratoFrota`.
    - **`CONTROLE DE VEÍCULO`**: Ativa a variável `locPP_CadastroVeiculo`.
    - **`ACOMPANHAMENTO DE FROTA`**: Ativa a variável `locPP_ControleFrota`.
    - **`ADMINISTRAÇÃO DA FROTA`**: Ativa a variável `locPP_AdminFrota`.

#### 3.1.1. Seção: Administração da Frota (`cAdminFrota`)

- **Visibilidade:** Controlada pela variável `locPP_AdminFrota`.
- **Funcionalidade:** Exibe uma galeria (`gal_EstadoCivil_12`) com a lista de todos os veículos cadastrados na fonte de dados `dbFrota`.
- **Galeria de Veículos:**
    - **Itens:** `dbFrota`
    - **Campos Exibidos:** Placa, ID Produto (Marca/Modelo), Cor, Cód. Renavam, Rodízio em SP, Locadora, Data de Início e Fim da Locação.
- **Ações por Veículo:** Cada item na galeria possui ícones que permitem ao usuário acessar detalhes específicos do veículo, alterando o estado da interface para exibir diferentes seções:
    - **Financeiro (Ícone `Money`):**
        - Ativa `locPP_FinanceiroVeiculo`.
        - Filtra e armazena os dados financeiros do veículo selecionado na coleção `colfinVeiculo`.
    - **Manutenção (Ícone `Tools`):**
        - Ativa `locPP_ManutencaoVeiculo`.
    - **Multas (Ícone `DocumentWithContent`):**
        - Ativa `locPP_MultasVeiculos`.
    - **Ocorrências (Ícone `Cars`):**
        - Ativa `locPP_AcidentesVeiculo`.
    - **Cartão Alelo (Ícone `Tablet`):**
        - Ativa `locPP_FinanceiroAlelo`.

#### 3.1.2. Sub-seções de Administração (Pop-ups)

Quando um dos ícones de ação é clicado, um contêiner correspondente se torna visível.

- **Controle Mensal - Financeiro (`cFinanceiroVeiculo`):**
    - Exibe os lançamentos financeiros mensais do veículo (da fonte `dbFrotasFinanceiro`).
    - Permite a criação de novos lançamentos (`NewForm(Form1)`) e a edição de existentes.
- **Multas do Veículo (`cMultasVeiculos`):**
    - Exibe uma galeria com as multas associadas ao veículo (da fonte `dbFrotaMultas`).
    - Permite lançar novas multas e editar registros existentes.
- **Manutenções do Veículo (`cManutencaoVeiculo`):**
    - Mostra o histórico de manutenções (da fonte `dbFrota_Manutencao`).
    - Permite agendar e registrar novas manutenções.

### 3.2. Tela Administrativa (`ADMINISTRATIVO SCREEN`)

**Objetivo:**
Gerenciar dados de suporte ao sistema, com foco no cadastro de funcionários.

**Componentes e Lógica:**

- **Menu de Navegação (`cMenuLateral_9`):** Permite retornar à tela de frota.
- **Botão "CADASTRO DE FUNCIONÁRIO":** Controla a visibilidade da seção de cadastro.

#### 3.2.1. Seção: Cadastro de Funcionário (`cCadastroFuncionario`)

- **Formulário (`FormCad_Funcionario`):**
    - **Fonte de Dados:** `dbCadastroFuncionario`.
    - **Funcionalidade:** Utilizado para criar um novo funcionário (`NewForm`) ou para editar um existente. O modo do formulário é controlado pela variável `locPP_AtualizarFuncionario`.
    - **Campos:** Nome Completo, CPF, E-mail, Telefone, Cargo, Equipe, Tipo de Contratação, Condutor (Sim/Não), etc.
    - **Ações:**
        - **`CADASTRAR` / `ATUALIZAR CADASTRO`:** O botão `SubmitForm(FormCad_Funcionario)` salva os dados no SharePoint.
        - **`CANCELAR`:** Limpa o formulário e as variáveis de edição.

- **Galeria de Funcionários Cadastrados (`Gallery1_5`):**
    - **Itens:** `SortByColumns(dbCadastroFuncionario, "NomeCompleto", SortOrder.Ascending)` - exibe os funcionários em ordem alfabética.
    - **Ações:**
        - **Editar (Ícone `Edit`):**
            1.  Filtra o funcionário selecionado na coleção `colCadFuncionario`.
            2.  Define o item a ser editado no formulário: `UpdateContext({loc_EditFuncionario: First(colCadFuncionario)})`.
            3.  Muda o modo do formulário para edição: `UpdateContext({locPP_AtualizarFuncionario: true})`.

## 4. Fontes de Dados

O aplicativo se conecta a diversas listas do SharePoint para armazenar e ler os dados. As principais fontes são:

- **`dbCadastroFuncionario`**: Armazena as informações dos funcionários da empresa.
- **`dbFrota`**: Lista principal contendo os detalhes de cada veículo da frota.
- **`dbFrotaContratos`**: Registros dos contratos de locação dos veículos.
- **`dbFrotasFinanceiro`**: Lançamentos financeiros mensais, como custos de locação e outros.
- **`dbFrota_Manutencao`**: Histórico de manutenções preventivas e corretivas.
- **`dbFrotaMultas`**: Registros de multas de trânsito.
- **`dbFrotaOcorrencia`**: Registros de acidentes e outras ocorrências com os veículos.
- **`dbFrotaAlelo`**: Controle de despesas do cartão combustível Alelo.
- **`dbLancamento_KM`**: Histórico de quilometragem rodada.
