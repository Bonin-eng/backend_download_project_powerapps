'''
# Documentação do Aplicativo: Abertura de Chamados T.I. e D.I.

## 1. Visão Geral do Projeto

O aplicativo "Abertura de Chamados T.I. e D.I." é uma ferramenta desenvolvida em Power Apps para centralizar e gerenciar as solicitações de suporte e desenvolvimento para os departamentos de Tecnologia da Informação (T.I.) e Desenvolvimento e Inovação (D.I.).

O sistema permite que os usuários abram novos chamados, acompanhem o histórico de suas solicitações e respondam a pesquisas de satisfação (NPS) para os atendimentos concluídos. O aplicativo é dividido em dois fluxos principais, um para cada departamento, e possui regras de negócio para garantir que as pesquisas de satisfação sejam respondidas antes da abertura de novos chamados.

## 2. Estrutura de Dados e Inicialização

### 2.1. Fontes de Dados

O aplicativo utiliza várias listas do SharePoint para armazenar os dados:

- **T.I.**: 
  - `db_tickets_infra`: Armazena os detalhes dos chamados de T.I.
  - `db_eventos_infra`: Registra os eventos e atualizações de cada chamado de T.I.
  - `db_nps_infra`: Armazena os dados das pesquisas de satisfação dos chamados de T.I.

- **D.I.**:
  - `db_tickets_dev`: Armazena os detalhes dos chamados de D.I.
  - `db_eventos_dev`: Registra os eventos de cada chamado de D.I.
  - `db_nps_dev`: Armazena os dados das pesquisas de satisfação dos chamados de D.I.

- `db_Tipo_Ticket`: Lista de configuração que contém os tipos de ticket e seus respectivos SLAs (Service Level Agreement).

### 2.2. Lógica de Inicialização (`App.OnStart`)

Ao iniciar, o aplicativo executa uma série de ações para preparar o ambiente para o usuário:

1.  **Identificação do Usuário**: Coleta as informações do usuário logado (Nome, E-mail, Empresa) e as armazena na variável `VarUsuario`.
2.  **Criação de Coleções**: Cria coleções locais (`colTiposRequisicaoTI` e `colTiposRequisicaoDI`) que armazenam os tipos de requisições disponíveis para cada departamento, incluindo o SLA correspondente buscado da lista `db_Tipo_Ticket`.
3.  **Carregamento de Dados em Lotes**: Para otimizar a performance, o aplicativo carrega os dados das listas do SharePoint (`tickets`, `eventos` e `nps`) em lotes de 1000 itens. Ele busca apenas os registros pertencentes ao usuário logado, reduzindo o volume de dados trafegados e acelerando o carregamento.

## 3. Descrição das Telas

### 3.1. `Home`

- **Propósito**: É a tela principal e o ponto de partida do aplicativo.
- **Funcionalidades**: Apresenta ao usuário as opções para interagir com os departamentos de T.I. e D.I.
  - **Abertura de Chamados**: Botões para iniciar a criação de um novo chamado de T.I. ou D.I.
  - **Histórico**: Acesso ao histórico de chamados de cada departamento.
  - **Pesquisa de Satisfação**: Acesso às pesquisas de satisfação pendentes.
- **Lógica de Negócio Chave**:
  - Antes de navegar para a tela de abertura de chamados, o sistema verifica duas condições:
    1.  **Dia da Semana**: Impede a abertura de novos chamados aos sábados e domingos, exibindo uma notificação.
    2.  **Pesquisas Pendentes**: Verifica se o usuário possui pesquisas de satisfação pendentes (`Tk_Status_NPS = "Aguardando Pesquisa"`). Se houver, o usuário é notificado e redirecionado para a tela de pesquisa, sendo impedido de abrir um novo chamado até que as pendências sejam resolvidas.

### 3.2. `TI_Chamados` e `DI_Chamados`

- **Propósito**: Telas para a criação de novos chamados para T.I. e D.I., respectivamente.
- **Componentes Principais**:
  - **Seleção de Requisição**: O usuário escolhe o tipo de requisição que deseja abrir (ex: "Suporte Técnico" para T.I., "Requisição de Ajuste e Melhoria" para D.I.).
  - **Formulário Dinâmico**: Com base no tipo de requisição selecionado, um formulário (`Form_TI_Chamados` ou `Form_DI_Chamados`) é exibido com os campos pertinentes.
  - **Informações do Chamado**: A tela exibe o número do ticket (gerado aleatoriamente via `GUID()`) e a previsão de conclusão com base no SLA.
- **Lógica de Negócio**:
  - Ao submeter o formulário, os dados são salvos na lista do SharePoint correspondente (`db_tickets_infra` ou `db_tickets_dev`).
  - Um fluxo do Power Automate (`BONINHELPDASK-SOLICITAÇÃOTICKET...`) é acionado para notificar a equipe responsável sobre o novo chamado.
  - Após o sucesso, o aplicativo recarrega os dados do SharePoint e exibe uma notificação de sucesso ao usuário.

### 3.3. `TI_Historico` e `DI_Historico`

- **Propósito**: Exibir o histórico de todos os chamados abertos pelo usuário para cada departamento.
- **Componentes Principais**:
  - **Galeria de Chamados (`gal_TI_Historico` / `gal_DI_Historico`)**: Lista todos os chamados do usuário, exibindo informações como número do ticket, tipo, atendente, data de abertura, conclusão e status.
  - **Detalhes Expansíveis**: O usuário pode clicar em um ícone para expandir um chamado e visualizar todos os eventos associados a ele (atualizações, comentários, etc.), que são carregados da lista de eventos (`col_db_eventos_infra` ou `col_db_eventos_dev`).
- **Lógica de Negócio**: Os dados são apresentados em ordem cronológica decrescente (chamados mais recentes primeiro).

### 3.4. `TI_Pesquisa` e `DI_Pesquisa`

- **Propósito**: Permitir que o usuário responda às pesquisas de satisfação (NPS) dos chamados que já foram concluídos.
- **Componentes Principais**:
  - **Galeria de Pesquisas Pendentes (`gal_TI_NPS` / `gal_DI_NPS`)**: Lista todos os chamados que estão com o status `Aguardando Pesquisa`.
  - **Formulário de Pesquisa**: Ao selecionar um item na galeria, um formulário (`Form_TI_NPS_Chamados` ou `Form_DI_NPS_Chamados`) é exibido.
  - **Campos de Avaliação**: O usuário avalia o atendimento em diferentes quesitos, como "Solução", "Agilidade" e "Comunicação", usando uma escala de avaliação (rating) e campos de texto para feedback.
- **Lógica de Negócio**:
  - Após o envio, o status do NPS na lista (`db_nps_infra` ou `db_nps_dev`) é atualizado para "Pesquisa Concluída".
  - O usuário recebe uma notificação de agradecimento e a lista de pesquisas pendentes é atualizada.

## 4. Tema e Estilo Visual (`Themes.json`)

O aplicativo utiliza um tema customizado (`defaultTheme`) para manter a consistência visual. A paleta de cores principal inclui:

- **Cor Primária**: `RGBA(56, 96, 178, 1)` (Azul)
- **Cor de Destaque**: `RGBA(45, 65, 93, 1)` (Azul Escuro)
- **Cor de Fundo Principal**: `RGBA(243, 244, 246, 1)` (Cinza Claro)

Essas cores são aplicadas em cabeçalhos, botões e outros elementos para criar uma identidade visual coesa e profissional, alinhada à marca Bonin.
'''