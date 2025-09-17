# Documentação do Aplicativo: Gestão de Viagens

## 1. Visão Geral do Aplicativo

O aplicativo de **Gestão de Viagens** é uma solução completa para o processo de solicitação, orçamento, aprovação e acompanhamento de viagens corporativas na empresa. Ele foi projetado para gerenciar um fluxo de trabalho com múltiplos níveis, envolvendo colaboradores, gestores e o time de Facilities.

As funcionalidades centrais são:
- **Solicitação de Viagem:** Colaboradores podem criar solicitações de viagem detalhadas, incluindo múltiplos viajantes e roteiros (passagens aéreas, terrestres, hospedagem, etc.).
- **Orçamentação:** Permite que a equipe de Facilities ou o próprio sistema anexe múltiplos orçamentos para cada item da viagem (ex: três cotações de passagens aéreas).
- **Fluxo de Aprovação:** Gestores podem aprovar as solicitações de suas equipes. O sistema valida os orçamentos com base na **Política de Viagens** da empresa e, se necessário, encaminha a solicitação para um segundo nível de aprovação (diretoria).
- **Visão por Perfil:** O aplicativo oferece uma experiência diferente para colaboradores e gestores. Gestores têm uma tela adicional para gerenciar as solicitações de sua equipe.
- **Histórico e Consulta:** Usuários podem acompanhar o status de suas viagens, e gestores podem ver o histórico de suas equipes.

## 2. Estrutura e Lógica Inicial

A configuração inicial e a lógica de navegação são definidas no arquivo `App.fx.yaml`.

### Lógica de Inicialização (`OnStart`)

Ao iniciar, o aplicativo realiza as seguintes ações:
1.  **Geração de ID Único:** Cria um ID de referência único para cada nova solicitação de viagem (`Var_ID_novo_cadastro`).
2.  **Carregamento de Coleções:**
    -   `colUFs`: Carrega uma lista com todas as siglas dos estados brasileiros.
    -   `ColTiposOrcamento`: Carrega os tipos de orçamento ativos (ex: "Aéreo - Ida", "Hotel") da lista `bgv_tipos_orcamentos`.
    -   `ColIconesMenus` e `ColLogo`: Carregam os ícones SVG e logos usados na interface do aplicativo.
3.  **Definição de Navegação por Perfil:**
    -   O sistema verifica se o e-mail do usuário logado (`User().Email`) existe na tabela de gestores (`sigd_gestores`).
    -   **Se for um Gestor:** A coleção de navegação `colTelas` é criada com acesso a quatro telas: "Solicitação de Viagem", "Histórico de Viagens", "Política de Viagens" e a tela exclusiva de gestão **"Solicitações de Viagens da Equipe"** (`Screen3`).
    -   **Se for um Colaborador Padrão:** A navegação é limitada a três telas, excluindo a de aprovação de equipes.

## 3. Perfis de Usuário

O aplicativo opera com dois níveis de permissão principais:

-   **Colaborador (Padrão):** Pode criar novas solicitações de viagem para si e outros, acompanhar o status de suas próprias viagens e consultar a política da empresa.
-   **Gestor:** Possui todas as permissões de um colaborador e, adicionalmente, pode visualizar, aprovar ou solicitar reorçamento para as viagens de sua equipe.

## 4. Telas do Aplicativo

### 4.1. Tela de Solicitação de Viagem (`Screen1`)

É a tela principal para a criação de uma nova requisição de viagem.

**Objetivo:** Permitir que o colaborador detalhe todas as necessidades de uma viagem em um único lugar.

**Componentes e Lógica:**
-   **Aviso de Antecedência:** Um banner no topo da tela informa que as solicitações devem ser feitas com 14 dias de antecedência.
-   **Formulário de Solicitação (`frm_solicitacao`):**
    -   Coleta informações gerais como motivo da viagem, centro de custo e observações.
    -   Os campos de data de início/fim e os tipos de transporte/hospedagem são preenchidos automaticamente com base nos roteiros adicionados pelo usuário.
-   **Formulário de Viajante (`frm_viajante`):**
    -   Permite adicionar um ou mais viajantes à solicitação. Ao inserir um CPF, o sistema busca os dados do colaborador na lista `sigd_colaboradores` para preencher automaticamente os campos.
-   **Seção de Roteiros:** O usuário pode adicionar múltiplos trechos e necessidades, como:
    -   Passagens aéreas (ida e volta).
    -   Passagens rodoviárias.
    -   Locação de veículo.
    -   Hospedagem (Hotel, Airbnb).
-   **Seção de Orçamentos:** Após a criação do roteiro, a equipe de Facilities anexa até três orçamentos para cada item, que serão posteriormente avaliados pelo gestor.

### 4.2. Tela de Histórico de Viagens (`Screen2`)

Dashboard pessoal para o acompanhamento das viagens solicitadas.

**Objetivo:** Dar visibilidade ao colaborador sobre o andamento de suas solicitações.

**Componentes e Lógica:**
-   **`OnVisible`:** A tela filtra as listas `bgv_viajantes` e `bgv_viagens` para encontrar todas as viagens associadas ao usuário logado e as carrega na coleção `colRegistroViagens`.
-   **Galeria de Histórico (`glr_historico`):**
    -   Exibe um resumo de cada viagem solicitada, ordenado da mais recente para a mais antiga.
    -   Cada item pode ser expandido para exibir um painel detalhado (renderizado com HTML) contendo todos os viajantes, roteiros e orçamentos daquela solicitação.

### 4.3. Tela de Aprovação do Gestor (`Screen3`)

Esta é a tela exclusiva para gestores, onde o fluxo de aprovação acontece.

**Objetivo:** Permitir que gestores analisem e aprovem as despesas de viagem de suas equipes.

**Componentes e Lógica:**
-   **`OnVisible`:** A lógica é similar à da tela de histórico, mas aqui o sistema busca todas as viagens solicitadas por colaboradores que estão sob a gestão do usuário logado.
-   **Galeria de Solicitações (`glr_historico_1`):**
    -   Lista as solicitações pendentes da equipe.
    -   Ao expandir um item, o gestor visualiza os detalhes da viagem e, mais importante, os **cards de orçamento** para cada despesa (passagem, hotel, etc.).
-   **Lógica de Aprovação de Orçamento:**
    -   Para cada item (ex: passagem aérea de ida), o gestor vê de 1 a 3 orçamentos.
    -   **Validação de Política:** O sistema compara o valor de cada orçamento com o teto definido na lista `sigd_politicas_viagens`. Se um orçamento excede o limite, ele é marcado com um aviso (`⚠️`).
    -   O gestor clica no botão **"✅ Aprovar este orçamento"** no card desejado.
-   **Finalizar Aprovação:** Após aprovar um orçamento para cada item da viagem, o gestor clica em **"Finalizar Aprovação"**.
    -   **Se todos os orçamentos aprovados estiverem DENTRO da política:** O status da viagem (`bgv_viagens`) é atualizado para **"Compra em andamento pelo facilities"**.
    -   **Se ALGUM orçamento aprovado estiver FORA da política:** O status da viagem é atualizado para **"Cotação enviada para aprovação do diretor"**, indicando que um segundo nível de aprovação é necessário.
-   **Solicitar Reorçamento:** O gestor pode clicar em **"Solicitar Reorçamento"**, o que invalida os orçamentos atuais e notifica a equipe de Facilities para que forneça novas cotações.

### 4.4. Tela de Política de Viagens (`Screen4`)

Uma tela estática que serve como guia de referência para todos os usuários.

**Objetivo:** Exibir de forma clara e organizada as regras e limites para viagens corporativas.

**Componentes e Lógica:**
-   A tela contém um único controle `HtmlText` que renderiza a política de viagens da empresa. O conteúdo é pré-formatado em HTML, incluindo tabelas de valores (limites para refeições, diárias de hotel) e regras sobre prazos, aprovações e reembolsos.

## 5. Fontes de Dados Principais

O aplicativo utiliza um conjunto robusto de listas do SharePoint para funcionar:

-   **`bgv_viagens`**: Tabela principal que armazena cada solicitação de viagem como um registro único.
-   **`bgv_viajantes`**: Armazena os dados de cada colaborador incluído em uma viagem, vinculando-os à solicitação principal.
-   **`bgv_roteiros`**: Detalha cada trecho da viagem (ex: um voo, uma reserva de hotel), também vinculado à solicitação principal.
-   **`sigd_viagens_orcamento`**: Tabela onde são registrados os orçamentos (até 3 por item de roteiro).
-   **`sigd_colaboradores`**: Lista mestra com os dados de todos os colaboradores.
-   **`sigd_gestores`**: Lista que define a hierarquia, mapeando quem são os gestores.
-   **`sigd_politicas_viagens`**: Armazena os tetos de gastos para cada tipo de despesa, servindo como base para o fluxo de aprovação.
-   **`bgv_tipos_orcamentos`**: Tabela de tipos para os orçamentos.
-   **`sigd_contratos`**: Lista de contratos para serem usados como centro de custo.
