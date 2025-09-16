# Documentação do Aplicativo: SIGD URBHIS

## 1. Visão Geral do Projeto

**Nome do Aplicativo:** Sistema Integrado de Gestão e Desenvolvimento URBHIS (SIGD URBHIS)

**Propósito:**
O SIGD URBHIS é uma aplicação de gestão abrangente, desenvolvida em Power Apps, para apoiar as operações da URBHIS. O sistema centraliza e gerencia uma variedade de processos, incluindo o atendimento a beneficiários, o planejamento de atividades em campo, a gestão de ordens de serviço, o controle de frotas e a administração de documentos oficiais.

O objetivo principal é fornecer uma plataforma unificada para equipes de campo e administrativas, otimizando a eficiência, a rastreabilidade das informações e o controle sobre as operações diárias relacionadas ao desenvolvimento urbano e social.

---

## 2. Estrutura e Lógica de Acesso

O aplicativo possui um robusto sistema de controle de acesso baseado em funções (RBAC - Role-Based Access Control), que garante que os usuários visualizem apenas as telas e funcionalidades pertinentes às suas atribuições.

### 2.1. Fontes de Dados Principais

O sistema utiliza diversas fontes de dados, provavelmente listas do SharePoint ou tabelas do Dataverse, para armazenar e gerenciar as informações. As principais são:

- `dbCadastroFuncionario`: Armazena os dados dos funcionários e seus níveis de permissão.
- `dbAtendimentos`: Registra todos os atendimentos realizados.
- `dbCadastroBeneficiario`: Mantém o cadastro de beneficiários e suas unidades familiares.
- `dbOS`: Gerencia as Ordens de Serviço.
- `dbFrota`: Controla os veículos da frota.
- `dbOficios`: Armazena informações sobre ofícios gerados.
- `dbPlanejamentoSemanal`: Guarda os dados do planejamento de atividades.
- `bdNucleos`: Cadastra os núcleos e distritos de atuação.
- Tabelas auxiliares para tipos de pesquisa, produtos, status, etc.

### 2.2. Níveis de Acesso de Usuário

Ao iniciar o aplicativo (`App.OnStart`), o sistema identifica o usuário logado e seu tipo de autorização (`TipoAutorizacao`) a partir da base `dbCadastroFuncionario`. Com base nesse nível, o menu de navegação é dinamicamente construído, liberando o acesso às telas correspondentes.

Os níveis de acesso identificados são:

- **Master:** Acesso completo a todas as funcionalidades do sistema.
- **Social (Nível 1, 2 e 3):** Focado em atendimento, agendamentos, cadastros e pesquisas. Os níveis superiores (2 e 3) possuem acesso a telas de controle e configurações.
- **Administrativo (Nível 1 e 2):** Acesso a telas de planejamento, ofícios, frotas e configurações administrativas.
- **Medição:** Focado no acompanhamento de medições, ordens de serviço e ofícios.
- **Visitante:** Acesso limitado, geralmente para consulta de informações básicas como tela inicial, plantão e cadastros.

---

## 3. Análise Detalhada das Telas

A seguir, uma descrição da finalidade e dos principais componentes de cada tela do aplicativo.

### 3.1. Tela Inicial (`Tela Inicial Screen`)

- **Propósito:** Servir como a tela de boas-vindas e ponto de partida para os usuários.
- **Lógica:**
    - Exibe uma mensagem de boas-vindas personalizada com o nome do usuário logado (`User().FullName`).
    - Apresenta o menu de navegação lateral (`cMenuLateral_1`), que é carregado com base no nível de permissão do usuário.
    - Pode conter dashboards ou resumos de atividades (a análise completa do conteúdo visual não é possível, mas a estrutura sugere isso).

### 3.2. Acompanhamento de Medição (`Acompanhamento Medição Screen`)

- **Propósito:** Visualizar métricas e indicadores de performance através de um painel de Business Intelligence.
- **Lógica:**
    - O componente principal é um `PowerBI`, que embute um relatório específico.
    - O `TileUrl` aponta para um relatório do Power BI, indicando que esta tela é usada para análise de dados e monitoramento de resultados.

### 3.3. Administrativo (`Administrativo Screen`)

- **Propósito:** Centralizar as funções administrativas, como gestão de funcionários e ordens de serviço específicas.
- **Lógica:**
    - **Cadastro de Funcionário:** Contém um formulário (`FormCad_Funcionario`) para criar e editar registros na base `dbCadastroFuncionario`.
    - **OIEPC:** Permite a gestão de Ordens de Serviço do tipo "OIEPC", filtrando e exibindo-as em uma galeria.
    - **Reuniões:** Inclui um formulário (`FormCad_ReuniaoAgendamento`) para planejar e agendar reuniões.

### 3.4. Agendamentos (`Agendamentos Screen`)

- **Propósito:** Gerenciar e controlar os agendamentos de visitas e vistorias.
- **Lógica:**
    - Exibe uma galeria (`gal_EstadoCivil_8`) com os atendimentos planejados ou concluídos da base `dbAtendimentos`.
    - Oferece múltiplos filtros para refinar a busca por plantão, status, nome do beneficiário, motivo, etc.
    - Permite a atualização (`Form_AtualizarAgendamento`), baixa (`Form_BaixarAgendamento`) e cancelamento de agendamentos.

### 3.5. Atendimento Emergencial (`Atendimento Emergencial Screen`)

- **Propósito:** Registrar e gerenciar atendimentos de caráter emergencial.
- **Lógica:**
    - O fluxo inicia com a seleção de um núcleo/subprefeitura e a busca de um beneficiário pelo CPF.
    - Permite cadastrar um novo beneficiário caso não seja encontrado.
    - Exibe o histórico de eventos da unidade familiar, incluindo atendimentos anteriores e agendamentos.
    - Contém um formulário (`FormCad_AtendimentoEmergencial`) para registrar os detalhes do novo atendimento emergencial.

### 3.6. Atendimento Plantão (`Atendimento Plantao Screen`)

- **Propósito:** Registrar atendimentos realizados em postos de plantão social.
- **Lógica:**
    - Similar à tela de Atendimento Emergencial, o fluxo começa pela identificação do local de atendimento e do beneficiário (via CPF).
    - Exibe informações cadastrais do beneficiário e o histórico de eventos da sua unidade familiar.
    - O formulário `FormCad_AtendimentoPlantao` é usado para registrar os detalhes do atendimento.

### 3.7. Cadastro de Beneficiário (`Cadastro Beneficiario Screen`)

- **Propósito:** Manter a base de dados de beneficiários e suas unidades familiares.
- **Lógica:**
    - Permite a busca de beneficiários por múltiplos filtros (Nome, CPF, Tipo Familiar, etc.).
    - Exibe os beneficiários de um núcleo selecionado em uma galeria (`gal_EstadoCivil_6`).
    - Contém formulários para **cadastrar um novo beneficiário** (`FormEdit_Beneficiario_2`) e **editar um cadastro existente**.
    - Oferece uma funcionalidade para visualizar e gerenciar a **unidade familiar**, vinculando múltiplos membros a um responsável.

### 3.8. Configurações (`Configurações Screen`)

- **Propósito:** Tela administrativa para gerenciar as tabelas e opções que alimentam o aplicativo.
- **Lógica:**
    - **Produtos:** Gerencia a lista de produtos/serviços (`dbProdutos`).
    - **Núcleos:** Permite o cadastro e a edição de núcleos e subprefeituras (`bdNucleos`).
    - **Formulário de Beneficiário:** Permite configurar as opções de campos de seleção, como Gênero (`dbGenero`), Raça/Etnia (`dbRacaEtnia`) e Estado Civil (`dbEstadoCivil`).

### 3.9. Controle de Agendamentos (`Controle Agendamentos Screen`)

- **Propósito:** Oferecer uma visão gerencial sobre todos os agendamentos.
- **Lógica:**
    - Possui duas abas principais: "Todos os Agendamentos" e "Agendamentos em Aberto (Sem Ordem)".
    - Permite que gestores filtrem agendamentos por núcleo e visualizem detalhes para acompanhamento e criação de ordens de serviço complementares.

### 3.10. Controle de Atendimentos (`Controle Atendimentos Screen`)

- **Propósito:** Permitir que cada usuário visualize os atendimentos que ele mesmo registrou.
- **Lógica:**
    - A galeria de atendimentos (`gal_EstadoCivil_23`) é filtrada pelo email do usuário logado (`'Criado por'.Email = User().Email`).
    - Oferece filtros por núcleo, nome, CPF, motivo, etc., para que o usuário possa encontrar seus próprios registros facilmente.

### 3.11. Controle Geral de Atendimentos (`Controle Geral Atendimentos Screen`)

- **Propósito:** Fornecer uma visão completa de todos os atendimentos realizados, destinada a perfis de gestão e administração.
- **Lógica:**
    - Similar à tela "Controle de Atendimentos", mas sem o filtro por usuário logado, exibindo todos os registros da base `dbAtendimentos`.
    - Permite a edição e visualização detalhada de cada atendimento.

### 3.12. Frotas (`Frotas Screen`)

- **Propósito:** Gerenciar o ciclo de vida e o uso dos veículos da organização.
- **Lógica:**
    - **Controle de Contrato:** Gerencia os contratos de locação de veículos.
    - **Controle de Veículo:** Contém um formulário (`FormCad_Veiculo`) para cadastrar e editar informações detalhadas de cada veículo na base `dbFrota` (placa, renavam, chassi, modelo, etc.).
    - **Acompanhamento de Frota:** Provavelmente exibe um dashboard ou galeria para monitorar o status e a alocação dos veículos.
    - **Administração da Frota:** Funções administrativas avançadas relacionadas à frota.

### 3.13. Ofícios (`Oficios Screen`)

- **Propósito:** Controlar a emissão e o status de ofícios relacionados às ordens de serviço.
- **Lógica:**
    - Permite filtrar ofícios por Ordem de Serviço (`cb_OrdemServico`).
    - Exibe uma lista de ofícios (`gal_EstadoCivil_9`) com seus detalhes (produto, status, data, etc.).
    - Contém formulários para atualizar (`Form_AtualizarOficio`), baixar (`Form_BaixarOfício`) e cancelar ofícios, atualizando o status na base `dbOficios`.

### 3.14. Ordens de Serviço (`Ordens Serviço Screen`)

- **Propósito:** Gerenciar Ordens de Serviço (OS) de diferentes tipos.
- **Lógica:**
    - **Abas de Controle:** A tela é dividida por tipo de OS: OEIP (Oficina Trimestral), OIEPC (Produto Complementar) e Ordens em Aberto.
    - **Criação e Edição:** Permite criar novas OS e atualizar existentes através de formulários como `Form_AtualizarOrdemServico`.
    - **Gestão de Equipe:** Possui funcionalidade para alocar equipes às ordens de serviço.
    - **Baixa e Cancelamento:** Permite alterar o status de uma OS para "Baixado" ou "Cancelado", com formulários para justificar o motivo.

### 3.15. Pesquisa (`Pesquisa Screen`)

- **Propósito:** Coordenar e registrar atividades de pesquisa de campo.
- **Lógica:**
    - **Agendamento de Ação Regional:** Permite agendar a aplicação de pesquisas em determinadas regiões, usando o formulário `Form_AtualizarOficio_2` (reutilizado para `dbAgendamentoRegional`).
    - **Controle de Aplicação:** Exibe os agendamentos em uma galeria e permite o registro dos resultados da aplicação das pesquisas.

### 3.16. Planejamento Semanal (`Planejamento Semanal Screen`)

- **Propósito:** Visualizar e organizar as atividades planejadas para a semana em um formato de calendário/Kanban.
- **Lógica:**
    - Exibe as atividades da semana em uma galeria horizontal (`galPortKanban_2`), dividida por dia.
    - As atividades são carregadas da base `dbPlanejamentoSemanal`.
    - Permite filtrar as atividades por núcleo e DTS (Distrito).
    - Um pop-up (`cPP_Cad_Atividade`) com o formulário `Form_PlanejamentoSemanal` permite o planejamento de novas atividades.

### 3.17. Produtos Cancelados (`Produtos Cancelados Screen`)

- **Propósito:** Monitorar e gerenciar produtos (serviços) que foram cancelados.
- **Lógica:**
    - Exibe uma galeria (`gal_EstadoCivil_20`) com todas as OS de status "Cancelado" da base `dbOS`.
    - Oferece uma aba para visualizar produtos cancelados que ainda não possuem uma justificativa.
    - Contém um formulário (`Form_BaixarOrdemServico_9`) para que o usuário insira o motivo do cancelamento no campo "Observações".
