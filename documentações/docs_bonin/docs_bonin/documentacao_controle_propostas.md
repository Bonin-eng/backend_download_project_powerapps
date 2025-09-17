'''
# Documentação do Aplicativo: Controle de Propostas

## 1. Visão Geral do Projeto

O aplicativo "Controle de Propostas" é uma ferramenta desenvolvida em Power Apps para gerenciar o ciclo de vida de propostas comerciais, separando-as em duas categorias principais: **Públicas** e **Privadas**. O sistema permite aos usuários criar novas propostas, visualizar o status atual, filtrar registros e gerenciar os diferentes lotes associados a cada proposta, bem como seus respectivos históricos.

A principal funcionalidade está concentrada em uma única tela, que se adapta para exibir propostas públicas ou privadas, proporcionando uma interface centralizada para todo o processo de gestão.

## 2. Estrutura de Dados e Inicialização

### 2.1. Fontes de Dados

O aplicativo utiliza principalmente duas listas do SharePoint:

- **`Proposta`**: Armazena os dados mestres de cada proposta, incluindo ID, cliente, edital, modalidade, e se é uma proposta preferencial.
- **`Historico_Propostas`**: Registra todas as atualizações de status e etapa para cada lote de uma proposta, criando um histórico detalhado de todo o processo.

### 2.2. Lógica de Inicialização (`App.OnStart`)

Ao iniciar, o aplicativo executa as seguintes ações para preparar os dados:

1.  **`loc_PropostaPrivada`**: Uma variável de controle é inicializada como `false`, definindo a visualização padrão para "Propostas Públicas".
2.  **Separação de Propostas**: A lista `Proposta` é consultada e dividida em duas coleções locais:
    - **`Col_Propostas_Publicas`**: Contém propostas cujo `idProposta` começa com "P.0".
    - **`Col_Propostas_Privadas`**: Contém propostas cujo `idProposta` começa com "P.1".
3.  **`colPropostas`**: Esta é a coleção principal usada pela galeria. Inicialmente, ela é populada com os dados de `Col_Propostas_Publicas`.
4.  **`ColLoteRecentes`**: O aplicativo processa a lista `Historico_Propostas` para criar uma coleção que contém apenas o registro de status mais recente para cada lote de proposta (`IdPropostaLote`). Isso otimiza a exibição do status atual na galeria principal sem a necessidade de consultar o histórico completo a todo momento.

## 3. Descrição da Tela Principal (`Propostas Públicas Screen`)

Esta é a tela central do aplicativo e agrega todas as funcionalidades de visualização e gerenciamento.

### 3.1. Funcionalidades e Componentes

- **Alternância de Visualização**:
  - **Botões "PROPOSTAS PÚBLICAS" e "PROPOSTAS PRIVADA"**: Permitem ao usuário alternar a fonte de dados da galeria principal. Ao clicar, a variável `loc_PropostaPrivada` é atualizada e a coleção `colPropostas` é recarregada com os dados da coleção pública ou privada correspondente.

- **Criação de Nova Proposta**:
  - **Botão "CRIAR NOVA PROPOSTA"**: Inicia o fluxo de criação de uma nova proposta.
  - **Integração com Power Automate**: Antes de abrir o formulário, um fluxo (`'APP-BONIN-CONTROLEDEPROPOSTAS...-ListarPastaPréAnálise'`) é executado para buscar, no SharePoint, pastas de propostas que ainda não foram formalmente cadastradas no aplicativo. Isso evita a duplicidade de registros.
  - **Pop-up de Criação (`locPP_Criar_Proposta`)**: Exibe um formulário (`Form_CriacaoProposta`) para que o usuário preencha os detalhes da nova proposta.

- **Filtragem e Visualização**:
  - **Campos de Filtro**: O usuário pode filtrar a lista de propostas por ID da Proposta, Cliente, Edital, Prazo e Modalidade.
  - **Galeria Principal (`gal_EstadoCivil_5`)**: Exibe as propostas (públicas ou privadas) com base nos filtros aplicados. As colunas mostram o status e a etapa mais recentes, obtidos da coleção `ColLoteRecentes`.

- **Gerenciamento de Lotes e Histórico (Pop-ups)**:
  - **Ícone de Lotes**: Na galeria principal, um ícone abre um pop-up (`locPP_Lotes`) que lista todos os lotes associados à proposta selecionada.
  - **Visualizar/Editar Lote (`locPP_EdicaoProposta`)**: Permite editar os detalhes de um lote específico.
  - **Visualizar Histórico (`locPP_VisulizarHistoricoProposta`)**: Abre um pop-up que exibe o histórico completo de um lote, mostrando todas as mudanças de etapa e status ao longo do tempo.
  - **Atualizar Histórico (`locPP_AtualizarHistorico`)**: Abre um formulário para adicionar um novo registro ao histórico de um lote, permitindo atualizar sua etapa e status.
  - **Criar Novo Lote (`locPP_Criar_Lotes`)**: Permite adicionar um novo lote a uma proposta já existente.

## 4. Tema e Estilo Visual (`Themes.json`)

O aplicativo utiliza um tema visual (`defaultTheme`) que padroniza a aparência dos componentes, garantindo uma experiência de usuário consistente. A paleta de cores é focada em tons de azul e cinza, conferindo um aspecto profissional e corporativo:

- **Cor Primária**: `RGBA(56, 96, 178, 1)` (Azul)
- **Cor de Destaque**: `RGBA(45, 65, 93, 1)` (Azul Escuro)
- **Fonte Principal**: Arial
'''