# **Documentação do Aplicativo: SIGD Bonin**

## 1. Visão Geral do Projeto

O **SIGD Bonin** é um aplicativo de gestão interna abrangente, projetado para centralizar e otimizar os processos de diversos departamentos da empresa Bonin. Ele funciona como um portal unificado onde os colaboradores podem acessar funcionalidades específicas de suas áreas, desde a abertura de chamados de TI e Marketing até a gestão de viagens, contratos e recursos humanos.

O aplicativo possui um sistema de permissões dinâmico, garantindo que cada usuário tenha acesso apenas às telas e funcionalidades relevantes para sua função, criando uma experiência personalizada e segura.

## 2. Arquitetura e Lógica Geral

### 2.1. Autenticação e Permissões

-   **Inicialização (App OnStart):** Quando o aplicativo é iniciado, ele identifica o usuário logado através do seu e-mail (`User().Email`).
-   **Verificação de Identidade:** O e-mail do usuário é usado para buscar seu registro na fonte de dados `sigd_colaboradores`, obtendo um `ID_Referencia` único.
-   **Controle de Acesso Baseado em Função (RBAC):** Com o ID do usuário, o aplicativo consulta a lista `sigd_din_perm_telas` para obter uma lista de telas permitidas para aquele colaborador. Essa lista é uma string de texto com os nomes das telas separados por ponto e vírgula.
-   **Menus Dinâmicos:** A barra de navegação lateral (`sidebar`) é construída dinamicamente. Ela filtra a coleção principal de telas (`colTelasGeral`) para exibir apenas os módulos e submenus que o usuário tem permissão para acessar, com base na coleção `colNomesTelasPermitidas` gerada na inicialização.

### 2.2. Componentes Reutilizáveis

O aplicativo utiliza componentes para manter a consistência visual e funcional:

-   **`sigd_bonin_sidebar` (Barra de Navegação Lateral):**
    -   É o principal elemento de navegação.
    -   Exibe os menus departamentais (Administrativo, TI, RH, etc.) com base nas permissões do usuário.
    -   Possui um menu expansível/retrátil para melhor aproveitamento do espaço.
    -   Contém submenus para navegação dentro de cada módulo.
    -   Exibe o nome e a foto do usuário logado, com um link para a tela de perfil (`US_1`).

-   **`sigd_bonin_cabecalho` (Cabeçalho):**
    -   Mostra o título da tela ativa, proporcionando contexto ao usuário.
    -   O título é obtido dinamicamente com base no nome da tela ativa (`App.ActiveScreen.Name`) e sua correspondência na coleção de navegação.

### 2.3. Estrutura de Dados e Nomenclatura

-   O aplicativo utiliza diversas listas do SharePoint como fontes de dados (prefixadas com `db_`, `sigd_`, `bgv_`).
-   As telas são nomeadas com um prefixo que indica o departamento (ex: `ADM_1`, `TI_2`, `FAC_1`), facilitando a organização e manutenção.

## 3. Módulos e Funcionalidades das Telas

O aplicativo é dividido nos seguintes módulos departamentais:

---

### **DI - Desenvolvimento e Inovação**

Módulo focado na gestão de projetos de desenvolvimento de software e BI.

-   **`DI_1`: Gestão da Informação (BI)**
    -   **Função:** Exibe um painel de Business Intelligence (Power BI) integrado, apresentando indicadores e dados relevantes sobre os projetos de desenvolvimento.
    -   **Lógica:** Carrega um relatório específico do Power BI através de uma URL de tile.

-   **`DI_2`: Kanban de Desenvolvimento**
    -   **Função:** Apresenta um quadro Kanban para visualização e gerenciamento do ciclo de vida dos chamados de desenvolvimento.
    -   **Lógica:**
        -   Os chamados são carregados da lista `db_tickets_dev`.
        -   As colunas do Kanban representam os diferentes status dos tickets (Aberto, Iniciado, Em Andamento, etc.), carregados da lista `db_tipo_evento`.
        -   Permite filtrar os chamados por solicitante, atendente e tipo de requisição.
        -   Exibe um resumo quantitativo de chamados por status.
        -   Permite visualizar detalhes de um chamado e seu histórico de eventos.

-   **`DI_3`: Abertura de Chamado**
    -   **Função:** Tela principal para a criação de novas solicitações para a equipe de Desenvolvimento e Inovação.
    -   **Lógica:**
        -   O usuário seleciona um tipo de requisição (RDA, RDB, RCA, etc.).
        -   Com base na seleção, um formulário (`Form_Abertura_Chamado_4`) é apresentado com os campos específicos para aquela requisição.
        -   Ao submeter, um novo item é criado na lista `db_tickets_dev` com um `TicketID` único gerado aleatoriamente.

-   **`DI_4`: Controle de Permissão**
    -   **Função:** Interface administrativa para gerenciar o acesso dos colaboradores às telas do aplicativo.
    -   **Lógica:**
        -   Permite associar um colaborador (da lista `sigd_colaboradores`) a um conjunto de telas permitidas.
        -   As permissões são salvas na lista `sigd_din_perm_telas` como um texto com nomes de telas separados por ";".
        -   Possui formulários para criar e editar essas permissões.

---

### **FAC - Facilities**

Gerencia as solicitações relacionadas a viagens e outras necessidades de infraestrutura.

-   **`FAC_1`: Chamados Facilities**
    -   **Função:** Apresenta um quadro Kanban para acompanhamento de chamados direcionados ao time de Facilities.
    -   **Lógica:**
        -   Carrega e exibe os chamados da lista `sigd_facilites_chamados`.
        -   As colunas representam as etapas do atendimento: "Abertura do Ticket", "Atendimento" e "Concluído".
        -   Permite avançar o status de um chamado.

-   **`FAC_2`: Controle de Viagens**
    -   **Função:** Tela para gestão completa do fluxo de solicitações e orçamentos de viagens.
    -   **Lógica:**
        -   Exibe as solicitações da lista `bgv_viagens` em um layout Kanban.
        -   As colunas representam os estágios do processo: "Solicitações", "Análises dos Gestores", "Ações do Facilities" e "Processo Finalizado".
        -   Permite visualizar detalhes da viagem, incluindo viajantes (`bgv_viajantes`), roteiros (`bgv_roteiros`) e orçamentos (`sigd_viagens_orcamento`).
        -   Possui pop-ups para aprovação, reprovação, solicitação de reorçamento e registro de compras.

-   **`FAC_3`: Configurações**
    -   **Função:** Tela administrativa para configurar parâmetros utilizados no módulo de Facilities.
    -   **Lógica:** Permite o cadastro e a edição de "Motivos de Viagem" (salvos na lista `bgv_tipos_motivo`), que serão utilizados nos formulários de solicitação.

---

### **RH - Recursos Humanos**

Módulo para gerenciamento de colaboradores e gestores.

-   **`RH_1`: Novos Colaboradores**
    -   **Função:** Formulário para iniciar o processo de onboarding de um novo colaborador.
    -   **Lógica:**
        -   Permite o cadastro de informações pessoais e contratuais do novo colaborador.
        -   Dispara automaticamente a criação de chamados para os departamentos de TI, Marketing e Facilities para providenciar os recursos necessários (notebook, kit de boas-vindas, etc.).
        -   Os dados do colaborador são salvos na lista `sigd_colaboradores`.
        -   Os chamados gerados são registrados nas listas `db_tickets_infra`, `sigd_mkt_chamados` e `sigd_facilites_chamados`.

-   **`RH_2`: Cadastro de Colaboradores**
    -   **Função:** Tela para visualização e edição dos dados de todos os colaboradores cadastrados.
    -   **Lógica:**
        -   Exibe uma galeria com os colaboradores da lista `sigd_colaboradores`.
        -   Permite filtrar por nome, departamento, tipo de contrato e status (ativo/inativo).
        -   Ao selecionar um colaborador, abre um formulário para edição de suas informações.

-   **`RH_3`: Cadastro de Gestores**
    -   **Função:** Interface para definir quais colaboradores são gestores.
    -   **Lógica:**
        -   Permite adicionar ou remover colaboradores da função de gestor.
        -   Os dados são salvos na lista `sigd_gestores`.
        -   A lista de gestores é utilizada em outros módulos para processos de aprovação.

---

### **TI - Tecnologia da Informação**

Centraliza a gestão de chamados e a visualização de dados de infraestrutura.

-   **`TI_1`: Gestão da Informação**
    -   **Função:** Apresenta um dashboard do Power BI com indicadores sobre a infraestrutura de TI e o volume de chamados.
    -   **Lógica:** Carrega um relatório específico do Power BI.

-   **`TI_2`: Backups**
    -   **Função:** Exibe um painel de Power BI focado no monitoramento e status dos backups.
    -   **Lógica:** Carrega um relatório específico do Power BI.

-   **`TI_3`: Kanban de Atendimentos**
    -   **Função:** Quadro Kanban para o gerenciamento de chamados de suporte técnico e infraestrutura.
    -   **Lógica:**
        -   Funcionalidade muito similar à tela `DI_2`, mas utiliza a lista `db_tickets_infra` como fonte de dados.
        -   Permite filtrar, visualizar detalhes, atualizar status e registrar o histórico de eventos dos chamados de TI.

---

### **MKT - Marketing**

-   **`MKT_1`: Chamados Marketing**
    -   **Função:** Quadro Kanban para gestão de solicitações do departamento de marketing.
    -   **Lógica:** Similar aos outros Kanbans, exibe chamados da lista `sigd_mkt_chamados` e permite a progressão de status.

-   **`MKT_2`:** Tela de placeholder para futuras funcionalidades do Marketing.

---

### **US - Perfil do Usuário**

-   **`US_1`: Perfil do Usuário**
    -   **Função:** Permite que o usuário visualize e edite suas próprias informações cadastrais.
    -   **Lógica:**
        -   Carrega os dados do usuário logado a partir da lista `sigd_colaboradores`.
        -   Apresenta um formulário (`frm_colaboradores_RH_1_3`) que permite a atualização de dados como telefone, endereço, etc.

---

### **Outros Módulos (ADM, COM, CON, CTL, DIR, INS, PMO, REC, SGI)**

Atualmente, a maioria das telas nesses módulos (`ADM_1`, `ADM_2`, `COM_1`, `COM_2`, etc.) funcionam como placeholders, contendo a estrutura de navegação padrão (sidebar e cabeçalho), mas sem funcionalidades específicas implementadas. Elas estão prontas para receber futuros relatórios, formulários ou outros conteúdos.

-   **`CON_1`:** Contém um elemento para exibir um relatório do Power BI relacionado a contratos.

