'''
# Documentação do Aplicativo: Gestão de Pessoa Jurídica

## 1. Visão Geral do Projeto

O aplicativo "Gestão de Pessoa Jurídica" é uma ferramenta administrativa desenvolvida em Power Apps para gerenciar o ciclo de vida de colaboradores (PJs), seus dados cadastrais, lançamentos financeiros mensais e as configurações gerais do sistema. O aplicativo serve como uma central de controle para o departamento administrativo e financeiro, permitindo desde o cadastro inicial de um colaborador até a automação de envio de e-mails.

## 2. Estrutura de Dados e Inicialização

### 2.1. Fontes de Dados Principais

O aplicativo se conecta a diversas listas do SharePoint para armazenar e gerenciar os dados:

- **`GPJ_Colaboradores`**: Lista principal que armazena os dados cadastrais de cada colaborador (pessoa jurídica).
- **`GPJ_Historico_Mensal`**: Registra os lançamentos financeiros e de atividades de cada colaborador a cada mês.
- **`GPJ_Historico_Anexos`**: Armazena os anexos relacionados aos lançamentos mensais (Notas Fiscais, RDs, etc.).
- **Listas de Opções (`GPJ_OP_*`)**: Um conjunto de listas (ex: `GPJ_OP_Tipo_Colaborador`, `GPJ_OP_Convenio_Medico`) que funcionam como tabelas de lookup para popular as opções de seleção nos formulários, permitindo uma gestão centralizada das opções disponíveis.

### 2.2. Lógica de Inicialização (`App.OnStart`)

Ao iniciar, o aplicativo realiza uma série de configurações essenciais:

1.  **Controle de Carregamento**: Variáveis como `VarInicial` e `VarTemporizador` são usadas para exibir uma tela de carregamento inicial, melhorando a experiência do usuário.
2.  **Criação de Coleções**: 
    - `ColDatas`: Gera uma coleção de meses e anos desde 01/01/2023 até a data atual, usada nos filtros de competência.
    - `ColIconesMenus`: Carrega os SVGs dos ícones usados na interface, centralizando os recursos visuais.
    - `colTelas`: Define a estrutura do menu de navegação, associando cada item de menu à sua respectiva tela e ícone.
3.  **Navegação Inicial**: Após a inicialização, um temporizador navega automaticamente para a tela `Screen1` (Cadastro de Pessoas), indicando que a `Home` funciona como uma tela de entrada e dashboard.

## 3. Descrição das Telas

### 3.1. `Home` (Tela Inicial)

- **Propósito**: Servir como um dashboard visual e tela de carregamento inicial.
- **Componentes**: A tela é composta por um componente reutilizável (`gpj_cpn_tela`) que define o layout padrão e um elemento Power BI (`PowerBI2`) que exibe um relatório.
- **Lógica**: Após um temporizador de 3 segundos (`tmr_loading_1`), o aplicativo navega automaticamente para a `Screen1`. O relatório do Power BI exibido é estático, fornecendo uma visão geral dos dados.

### 3.2. `Screen1` (Cadastro de Pessoas)

- **Propósito**: Gerenciar o cadastro completo dos colaboradores (PJs).
- **Componentes**: 
  - **Galeria de Colaboradores**: Exibe todos os colaboradores cadastrados, permitindo a busca por nome.
  - **Formulário de Cadastro (`frm_novo_cadastro`)**: Um formulário abrangente para criar ou editar os dados de um colaborador, incluindo informações pessoais, de contrato, regime, e valores de remuneração.
- **Lógica de Negócio**:
  - **CRUD Completo**: Permite criar, ler, atualizar e desativar colaboradores.
  - **Atualização de Vínculo**: Ao atualizar um colaborador (por exemplo, em uma renovação), o aplicativo tem a lógica para desativar o registro antigo e criar um novo, mantendo o histórico intacto.
  - **Controle de Visibilidade**: A tela alterna entre a visualização da galeria (`VarCtnDadosGal`) e a do formulário (`VarCtnDadosForm`).

### 3.3. `Screen2` (Lançamentos Mensais)

- **Propósito**: Realizar e consultar os lançamentos financeiros mensais para cada colaborador ativo.
- **Componentes**:
  - **Galeria de Colaboradores**: Lista os colaboradores ativos para seleção.
  - **Galeria de Histórico**: Ao selecionar um colaborador, exibe o histórico de seus lançamentos mensais.
  - **Formulário de Lançamento (`frm_mensal_cadastro`)**: Permite registrar os valores do mês para diversas categorias (Nota, RT, KM, Plano Médico, Alimentação, Comissões, etc.).
  - **Formulários de Anexos**: Seções dedicadas para anexar documentos como Nota Fiscal (NF), Recibo de Despesas (RD) e Comprovante (CONF).
- **Lógica de Negócio**:
  - O usuário seleciona um colaborador e uma competência (mês/ano) para ver ou criar um lançamento.
  - O formulário calcula automaticamente os totais (`Total_RD` e `Total_Geral`) com base nos valores inseridos.
  - Ao salvar, os dados são registrados na lista `GPJ_Historico_Mensal` e os anexos na `GPJ_Historico_Anexos`.

### 3.4. `Screen3` (Configurações)

- **Propósito**: Tela administrativa para gerenciar as listas de opções usadas nos formulários do aplicativo.
- **Componentes**: A tela é dividida em seções, cada uma com uma galeria e um formulário para gerenciar uma lista de lookup específica:
  - Perfil do Colaborador
  - Contrato de Atuação
  - Perfil de RT (Responsabilidade Técnica)
  - Plano Médico
  - Alimentação
- **Lógica de Negócio**:
  - Permite que um administrador adicione, edite ou desative opções que aparecerão nos menus suspensos das outras telas.
  - Isso torna o aplicativo flexível, permitindo a adição de novos tipos de contrato, perfis ou benefícios sem a necessidade de alterar o código do aplicativo.

### 3.5. `Screen4` (Envio de E-mail)

- **Propósito**: Automatizar o envio de e-mails em massa para os colaboradores (PJ) e para o departamento financeiro.
- **Componentes**:
  - **Seleção de Mês**: Um menu suspenso para escolher a competência desejada.
  - **Galeria de Lançamentos**: Exibe os colaboradores com lançamentos no mês selecionado.
  - **Checkboxes de Seleção**: Permitem selecionar todos ou colaboradores específicos para o envio.
- **Lógica de Negócio**:
  - **Dois Fluxos de E-mail**: 
    1.  **E-mail PJ**: Envia um e-mail para o colaborador com os detalhes de seus lançamentos mensais.
    2.  **E-mail Financeiro**: Envia um e-mail consolidado para o departamento financeiro, contendo os dados de todos os colaboradores selecionados.
  - **Integração com Power Automate**: Ao clicar no botão de envio, um fluxo (`GPJ_BONIN_AUTOMACAO_EMAIL` ou `GPJ_BONIN_AUTOMACAO_FINANCEIRO_EMAIL`) é acionado, recebendo os dados dos colaboradores selecionados em formato JSON para processar e enviar os e-mails.

## 4. Tema e Estilo Visual (`Themes.json`)

O aplicativo utiliza um tema visual (`defaultTheme`) que garante uma identidade visual coesa. A paleta de cores é baseada em tons de azul e cinza, transmitindo uma imagem profissional e organizada:

- **Cor Primária**: `RGBA(56, 96, 178, 1)` (Azul)
- **Cor de Destaque**: `RGBA(45, 65, 93, 1)` (Azul Escuro)
- **Fonte Principal**: Segoe UI e Arial.
'''