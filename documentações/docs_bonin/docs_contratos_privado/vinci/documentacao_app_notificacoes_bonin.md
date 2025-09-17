# Documentação do Aplicativo de Notificações (Bonin/Vinci)

## 1. Visão Geral do Projeto

O aplicativo **Notificações de Área** é uma ferramenta móvel desenvolvida em Power Apps com um forte foco em **capacidade offline**. Seu principal objetivo é permitir que equipes de campo registrem, consultem e editem notificações sobre elementos em rodovias (como publicidade irregular, ocupações, etc.), mesmo em locais sem conexão com a internet.

Os principais recursos são:

-   **Criação e Edição de Notificações**: Formulários para registrar novas notificações ou alterar existentes.
-   **Consulta de Dados**: Visualização da lista de notificações com filtros dinâmicos.
-   **Funcionalidade Offline-First**: O aplicativo é projetado para funcionar primariamente com dados locais (em cache), garantindo que o usuário nunca perca seu trabalho por falta de conexão.
-   **Sincronização Inteligente**: Um mecanismo robusto para enviar dados locais para a nuvem (SharePoint) e baixar atualizações quando a conexão é restabelecida.

---

## 2. Arquitetura Offline-First e Sincronização

Esta é a característica mais importante do aplicativo. A lógica de funcionamento offline é controlada por variáveis, coleções locais e um processo de sincronização manual ou automático.

### 2.1. Inicialização (`App.fx.yaml`)

O `OnStart` do aplicativo executa uma sequência crucial para a funcionalidade offline:

1.  **Carregamento do Cache**: Usando `LoadData()`, o aplicativo tenta carregar todas as coleções de dados essenciais (`colTelas`, `colDadosLista`, `colDadosOffline`, etc.) a partir do cache local do dispositivo. Isso garante que o app possa ser aberto e utilizado mesmo sem internet.
2.  **Verificação de Conexão**: Ele verifica se há uma conexão ativa com `Connection.Connected`.
    -   **Se Online**: O aplicativo atualiza todos os dados. Ele baixa a lista de notificações do SharePoint (`db_notificacoes`) em lotes de 1000 para otimizar o desempenho, atualiza coleções auxiliares (cidades, elementos) e salva tudo no cache local com `SaveData()`.
    -   **Se Offline**: O aplicativo simplesmente utiliza os dados que foram carregados do cache, exibindo um indicador visual de que está operando em modo offline.

### 2.2. Coleções-Chave para o Modo Offline

-   **`colDadosOffline`**: A coleção mais crítica. Todos os registros novos ou editados são salvos **primeiro** aqui. Cada item nesta coleção possui um campo `StatusSync`.
-   **`colDadosLista`**: A coleção principal usada para exibir os dados nas galerias. Quando online, é um espelho da base de dados do SharePoint. Quando offline, contém os dados do último cache.
-   **`colComparacaoLista`**: Uma coleção auxiliar que armazena apenas os IDs das notificações existentes no SharePoint. É usada para verificar rapidamente se um registro criado offline já existe na nuvem, evitando duplicatas.

### 2.3. O Processo de Sincronização (`btn_sinc_1` na Home Screen)

O botão "SINCRONIZAR" dispara a lógica de envio de dados para a nuvem:

1.  **Backup**: Salva o estado atual das coleções locais no cache como medida de segurança.
2.  **Iteração**: Usa `ForAll(FirstN(Filter(colDadosOffline, StatusSync = "PENDENTE"), 50) As item, ...)` para processar os 50 primeiros itens com status "PENDENTE".
3.  **Verificar e Enviar (Create vs. Update)**:
    -   Para cada item, ele verifica (`IsBlank(LookUp(db_notificacoes, ...))`) se um registro com o mesmo `id_notificacao` já existe no SharePoint.
    -   Se **não existe**, ele cria um novo registro com `Patch(db_notificacoes, Defaults(db_notificacoes), ...)`. O `StatusSync` do item local é mudado para "ENVIADO".
    -   Se **já existe**, ele atualiza o registro existente com `Patch(db_notificacoes, LookUp(...), ...)`. O `StatusSync` local também é mudado para "ENVIADO".
4.  **Tratamento de Erro**: Se o `Patch` falhar (por exemplo, por falta de conexão ou dados inválidos), o `StatusSync` do item local é mudado para "ERRO" e uma mensagem é gravada.
5.  **Limpeza**: Após a sincronização, `RemoveIf(colDadosOffline, StatusSync = "ENVIADO" || StatusSync = "DUPLICADO")` remove os itens já processados da fila offline, mantendo apenas os que ainda estão pendentes ou com erro.
6.  **Atualização Final**: Por fim, ele baixa novamente a lista completa do SharePoint para garantir que o aplicativo local esteja 100% atualizado com a nuvem.

---

## 3. Análise das Telas

### 3.1. `Home Screen`

**Objetivo**: Servir como painel de controle principal, exibindo o status da sincronização e permitindo o acesso à lista de notificações.

**Componentes e Lógica**:

-   **Indicadores de Status**: Labels que mostram o status da conexão (`Connection.Connected`) e a contagem de itens offline e online.
-   **`btn_sinc_1` (Botão)**: O principal gatilho para o processo de sincronização descrito na seção anterior.
-   **Filtros (`TextInput1_21`, `TextInput1`)**: Permitem filtrar a galeria principal por cidade ou descrição do local.
-   **`Gallery3_7` (Galeria de Notificações)**:
    -   **Items**: A galeria exibe os dados da `colDadosLista`, enriquecidos com o nome da cidade (`AddColumns(...)`).
    -   **`Button4_5` (Botão Sobreposto)**: Ao clicar em um item, a ação `Set(varNotificacaoSelect, ...); Navigate(...)` armazena o registro selecionado na variável `varNotificacaoSelect` e navega para a tela de edição.

### 3.2. `Nova Fiscalização Screen` (Tela de Novo Cadastro)

**Objetivo**: Fornecer um formulário para a criação de uma nova notificação.

**Componentes e Lógica**:

-   **`Form2` (Formulário)**: Configurado em `FormMode.New` e ligado à fonte de dados `db_notificacoes`.
-   **`id_notificacao_DataCard1_1` (Campo de ID da Notificação)**:
    -   Este campo é desabilitado (`DisplayMode.Disabled`) e seu valor padrão é gerado dinamicamente concatenando valores de outros campos do formulário: `Concatenate(DataCardValue12_1.Selected.Value," ", Right(DataCardValue10_2.Selected.Value, 3), ...)`.
    -   Isso cria um ID único e legível para cada notificação antes mesmo de ela ser salva.
-   **Geolocalização**: Os campos de latitude e longitude são preenchidos automaticamente com `Location.Latitude` e `Location.Longitude`.
-   **Botão "SALVAR" (dentro da `Editar Notificação Screen`, mas a lógica se aplica aqui)**:
    -   A lógica de salvar um novo item é gerenciada pelo botão na tela de edição, que é reutilizado. Ele pega os dados do formulário (`Form1.Updates`), adiciona os campos de controle (`StatusSync: "PENDENTE"`, `IndiceLote`, etc.) e o adiciona à coleção `colDadosOffline` para aguardar a sincronização.
    -   Se o app estiver online, ele chama o botão de sincronização (`Select(btn_sinc)`) para um envio imediato.

### 3.3. `Editar Notificação Screen`

**Objetivo**: Editar os detalhes de uma notificação existente ou salvar uma nova.

**Componentes e Lógica**:

-   **`Form1` (Formulário)**:
    -   **Item**: `varNotificacaoSelect` - O formulário é preenchido com os dados da notificação que o usuário selecionou na `Home Screen`.
-   **`Button1` (Botão "SALVAR")**:
    -   **OnSelect**: A lógica é praticamente idêntica à de criação de um novo item, mas aplicada a um registro existente.
    1.  Pega os dados do formulário (`Form1.Updates`).
    2.  Adiciona/atualiza os campos de controle (`StatusSync: "PENDENTE"`).
    3.  Adiciona o registro modificado à coleção `colDadosOffline`.
    4.  Salva a coleção `colDadosOffline` no cache do dispositivo com `SaveData()`.
    5.  Se online, tenta sincronizar imediatamente.
    6.  Navega de volta para a `Home Screen`.

Esta abordagem unificada garante que tanto criações quanto edições passem pela mesma fila de sincronização, tornando o sistema robusto e previsível.

---

## 4. Fontes de Dados

-   **`db_notificacoes`**: Fonte de dados principal no SharePoint, contendo todos os registros de notificação.
-   **`db_cidades_cadastradas`**: Tabela auxiliar com a relação de municípios, rodovias e KMs.
-   **`db_tipo_elemento`**: Tabela auxiliar que define os tipos de elementos que podem ser notificados (ex: outdoor, placa, etc.).

