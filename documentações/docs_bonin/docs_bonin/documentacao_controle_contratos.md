'''
# Documentação do Aplicativo: Controle de Contratos

## 1. Visão Geral do Projeto

O aplicativo "Controle de Contratos" é uma ferramenta de gestão desenvolvida em Power Apps, projetada para centralizar o gerenciamento de contratos, aditivos, prestadores de serviço e a previsão financeira associada. Ele oferece uma visão consolidada dos dados contratuais e se integra com o Power BI para a visualização de dashboards dinâmicos.

As principais funcionalidades incluem o cadastro e a edição de contratos, o gerenciamento de prestadores de serviço (PS), o registro de aditivos e a elaboração de previsões financeiras detalhadas (aportes), que são submetidas a um fluxo de aprovação.

## 2. Estrutura de Dados e Inicialização

### 2.1. Fontes de Dados Principais

O aplicativo utiliza diversas listas do SharePoint para armazenar suas informações:

- **`Contratos`**: Lista principal com todos os dados dos contratos.
- **`Aditivo`**: Armazena os aditivos e suspensões de cada contrato.
- **`sigd_prestadores_servicos`**: Contém os registros de prestadores de serviço associados a um contrato específico.
- **`sigd_cadastro_prestadores_servicos`**: Lista mestra com o cadastro de todos os prestadores de serviço.
- **`Teste_Aportes`**: Registra as previsões financeiras e aportes para cada competência de um contrato.
- **`Clientes`**: Lista de apoio com os nomes dos clientes.

### 2.2. Lógica de Inicialização (`App.OnStart`)

Ao ser iniciado, o aplicativo executa as seguintes ações:

1.  **`colTelas`**: Cria uma coleção para definir a estrutura do menu de navegação, listando todas as telas disponíveis.
2.  **`colContratos`**: Carrega todos os itens da fonte de dados `Contratos` para uma coleção local, otimizando a performance de consultas e filtros dentro do app.
3.  **`colLinksPowerBI`**: Cria uma coleção estática que mapeia o e-mail de usuários específicos a links de relatórios do Power BI. Isso permite a exibição de dashboards personalizados na tela inicial.

## 3. Descrição das Telas

### 3.1. `HOME SCREEN`

- **Propósito**: Servir como painel de controle e tela de boas-vindas.
- **Componentes Principais**:
  - **`PowerBI1` (Componente Power BI)**: Ocupa a área principal da tela e exibe um dashboard.
- **Lógica de Negócio**:
  - O relatório do Power BI exibido é dinâmico. O aplicativo identifica o e-mail do usuário logado e busca o link correspondente na coleção `colLinksPowerBI`.
  - Caso o e-mail do usuário não esteja na lista, nenhum relatório é exibido. Isso garante que cada gestor ou usuário veja apenas os dados pertinentes à sua área.

### 3.2. `CONTRATO SCREEN`

- **Propósito**: Tela central para visualização, filtragem e gerenciamento de contratos e seus aditivos.
- **Componentes Principais**:
  - **Filtros**: Uma série de caixas de combinação permite filtrar a galeria de contratos por Nº Contrato, Contrato Bonin, Referência, Cliente, Consórcio e Responsável.
  - **`gal_EstadoCivil_6` (Galeria de Contratos)**: Exibe a lista de contratos filtrados.
  - **Ícones de Ação**: Cada item da galeria possui ícones para:
    - Visualizar/Editar informações do contrato.
    - Gerenciar aditivos e suspensões.
    - Acessar o cronograma financeiro.
    - Abrir a pasta do contrato no SharePoint.
  - **Pop-ups de Edição**: Ao clicar nos ícones, pop-ups (`locPP_Contrato`, `locPP_Aditivos`) são exibidos com formulários para editar os dados.

### 3.3. `PRESTADORES DE SERVIÇO SCREEN`

- **Propósito**: Gerenciar a alocação de prestadores de serviço (PS) aos contratos e visualizar os custos associados.
- **Componentes Principais**:
  - **`Gallery3_6` (Galeria de Contratos)**: Lista os contratos principais.
  - **`Gallery1_3` (Galeria Aninhada de PS)**: Ao expandir um contrato, esta galeria exibe todos os prestadores de serviço associados a ele, mostrando detalhes como atividade, datas e valor.
  - **Cálculos `OnVisible`**: Antes de exibir a tela, o app calcula e armazena em coleções o valor total já gasto com prestadores (`colPrestacaoServicoTotal`) e o valor total atualizado de cada contrato (`colContratosComValorAtual`), incluindo aditivos. Esses valores são usados para calcular o percentual de custo operacional.

### 3.4. `CADASTRO DE PS SCREEN`

- **Propósito**: Gerenciar a lista mestra de prestadores de serviço e associá-los a contratos específicos.
- **Componentes Principais**:
  - **`Gallery3_7`**: Exibe todos os prestadores de serviço cadastrados na lista `sigd_cadastro_prestadores_servicos`.
  - **Botão "CADASTRAR NOVO PRESTADOR"**: Abre um pop-up (`varCadastrarContratoPS`) com um formulário para adicionar um novo prestador à lista mestra.
  - **Pop-up de Edição/Associação (`varVisualizarContratoPS`)**: Ao selecionar um prestador na galeria, um pop-up é exibido para visualizar ou editar a associação daquele PS a um contrato, incluindo detalhes como objeto, valores e prazos.

### 3.5. `PREVISÃO FINANCEIRA SCREEN`

- **Propósito**: Tela dedicada ao lançamento de previsões de custos e aportes financeiros para um contrato em uma determinada competência (mês/ano).
- **Componentes Principais**:
  - **`Select_Contrato`**: Caixa de combinação para selecionar o contrato desejado.
  - **Formulário de Custos**: Diversos campos de entrada para detalhar os custos (Equipe Técnica, Despesas Indiretas, Viagens, TI, etc.).
  - **`Total_Aporte`**: Campo calculado que soma todos os custos inseridos.
  - **Botão "CALCULAR APORTE"**: Coleta todos os dados inseridos em uma coleção (`Col_json_aporte`) para revisão do usuário.
  - **Pop-up de Confirmação (`POP`)**: Exibe uma tela de revisão antes do envio final.
- **Lógica de Negócio**:
  - Após a confirmação do usuário, os dados são salvos na lista `Teste_Aportes`.
  - Um fluxo do Power Automate (`Producao_Aprovacao_Aportes.Run`) é acionado, provavelmente para iniciar um processo de aprovação do aporte financeiro.

## 4. Tema e Estilo Visual (`Themes.json`)

O aplicativo utiliza um tema visual (`defaultTheme`) consistente em todas as telas. A identidade visual é construída em torno de uma paleta de cores sóbria e profissional:

- **Cor Primária**: `RGBA(56, 96, 178, 1)` (Azul)
- **Cor de Destaque**: `RGBA(45, 65, 93, 1)` (Azul Escuro)
- **Cor de Fundo**: Branco e tons de cinza claro, com os pop-ups e áreas de formulário se destacando.

O uso de fontes como Arial e Segoe UI reforça o aspecto corporativo da aplicação.
'''