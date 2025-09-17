# Documentação do App de Gerenciamento de Notificações (Bonin/Vinci)

## 1. Visão Geral do Projeto

O aplicativo **Gerenciamento de Notificações** é uma ferramenta administrativa, provavelmente para uso em desktop/web, que serve como o backend de gerenciamento para o sistema de notificações de campo. Seu propósito é centralizar a visualização de dados, administrar tabelas de apoio (cadastros) e controlar o acesso dos usuários ao ecossistema de aplicativos.

As funcionalidades principais são:

-   **Dashboard de Acompanhamento**: Exibe um painel do Power BI para visualização consolidada dos dados.
-   **Gerenciamento de Notificações**: Uma tela para consultar, filtrar e editar todas as notificações existentes na base de dados.
-   **Administração de Cadastros**: Telas dedicadas para gerenciar dados mestres, como Municípios e Tipos de Elementos.
-   **Controle de Acesso**: Uma interface para administrar os usuários e seus respectivos níveis de permissão (cliente ou usuário interno).

---

## 2. Arquitetura e Navegação

O aplicativo implementa uma navegação dinâmica e baseada em perfis de usuário, que é definida no `OnStart`.

### 2.1. `OnStart`: Navegação Baseada em Perfil

1.  **Verificação de Acesso**: A primeira ação é `Set(varUsuario, LookUp(db_acessos, email = User().Email, tipo_acesso))`. O aplicativo verifica o tipo de acesso do usuário logado (`cliente` ou `usuario`) na tabela `db_acessos` e armazena o resultado na variável `varUsuario`.
2.  **Carregamento de Ícones**: O app carrega uma coleção `ColIconesMenus` com dados SVG para os ícones do menu. Isso torna os ícones parte do aplicativo, eliminando a necessidade de carregar imagens externas.
3.  **Criação de Menus Dinâmicos**: A coleção de navegação `colTelas` é criada de forma condicional:
    -   **Se `varUsuario = "cliente"`**: O menu é simples, contendo apenas um link para a tela de acompanhamento (`Home Screen`).
    -   **Caso contrário (usuário interno/admin)**: O menu é completo, incluindo submenus para "NOTIFICAÇÕES" e "CONFIGURAÇÕES", que dão acesso às telas de gerenciamento de notificações, cadastros e controle de acesso.

Essa abordagem garante que os clientes vejam apenas o dashboard, enquanto a equipe interna tem acesso a todas as ferramentas administrativas.

---

## 3. Análise das Telas

### 3.1. `Home Screen`

**Objetivo**: Apresentar um dashboard consolidado com os principais indicadores do projeto.

**Componentes e Lógica**:

-   **`PowerBI1` (Integração Power BI)**: O principal elemento da tela. Ele embute um relatório específico do Power BI usando a URL do tile. Isso permite que dados complexos e visualizações ricas sejam exibidos diretamente no aplicativo.
-   **`Timer1` (Temporizador de Carregamento)**: No `OnVisible` da tela, a variável `varIniciarBICronometro` é definida como `true`, o que inicia um temporizador de 4.5 segundos. Durante esse tempo, uma animação de carregamento é exibida (`ctn_loading_2`). Isso melhora a experiência do usuário, dando tempo para o relatório do Power BI carregar em segundo plano.

### 3.2. `Notificações Screen`

**Objetivo**: Fornecer uma interface completa para visualizar, filtrar e editar todas as notificações registradas.

**Componentes e Lógica**:

-   **Carregamento de Dados em Lote (`OnVisible`)**: Para otimizar o desempenho com grandes volumes de dados, a tela carrega a tabela `db_notificacoes` em lotes de 500 registros. Ela usa `ForAll` e `With` para iterar sobre os lotes e carregar tudo em uma coleção local `collRegistrosUnicosFinal`.
-   **Enriquecimento de Dados Local**: Após o carregamento, a função `AddColumns` é usada para criar a coleção final `colNotificacoesTotal`, adicionando uma coluna `cidade` a cada registro. O nome da cidade é obtido através de um `LookUp` na tabela `db_cidades_cadastradas`. Fazer isso localmente evita problemas de delegação do Power Apps.
-   **Filtros**: A tela possui múltiplos ComboBoxes (`ComboBox1`, `ComboBox1_1`, etc.) que permitem ao usuário filtrar a galeria por cidade, KM, tipo de elemento e ação necessária.
-   **`galContratos` (Galeria)**: Exibe os dados da coleção local `colNotificacoesTotal`, aplicando os filtros selecionados.
-   **`cContratoSelect` (Pop-up de Edição)**:
    -   Ao clicar em um item da galeria, a variável `varContratoSelect` se torna `true`, exibindo um pop-up com os detalhes da notificação.
    -   O botão "EDITAR" (`Button2_1`) alterna a variável `varContratoEdit`, que libera um formulário (`Form_ContratoEdit`) para edição.
    -   O botão "SALVAR" (o mesmo `Button2_1`) executa `SubmitForm(Form_ContratoEdit)`, salvando as alterações diretamente na fonte de dados `db_notificacoes`.

### 3.3. `Cadastro Screen`

**Objetivo**: Gerenciar o cadastro de municípios, que serve como tabela de apoio para o restante do sistema.

**Componentes e Lógica**:

-   Esta é uma tela de CRUD (Create, Read, Update, Delete) para a tabela `db_cidades_cadastradas`.
-   **`galContratos_1` (Galeria)**: Lista os municípios cadastrados, com a opção de filtrar por nome.
-   Ao selecionar um item, um pop-up (`cContratoSelect_1`) é exibido, mostrando os detalhes.
-   Dentro do pop-up, um formulário (`Form_ContratoEdit_1`) permite a edição dos dados (Rodovia, UF, KM Inicial/Final, etc.).
-   A lógica de salvar (`SubmitForm`) e alternar entre os modos de visualização e edição é controlada por variáveis de contexto como `varContratoSelect` e `varContratoEdit`.

### 3.4. `Configuração Screen`

**Objetivo**: Gerenciar o cadastro dos "Tipos de Elemento" (ex: Placa, Outdoor, etc.) que podem ser notificados.

**Componentes e Lógica**:

-   Outra tela de CRUD, desta vez para a tabela `db_tipo_elemento`.
-   **`galContratos_15` (Galeria)**: Lista os elementos existentes.
-   **Ícones de Ação**:
    -   **Adicionar (`Icon2_8`)**: Abre um formulário (`Form_DotUnidade_1`) em modo `NewForm` para criar um novo tipo de elemento.
    -   **Editar (`Icon1_16`)**: Armazena o item selecionado em `locPP_DotUnidadeSelect` e abre o mesmo formulário em modo de edição.
    -   **Excluir (`Icon1_17`)**: Executa `RemoveIf(db_tipo_elemento, ID = varCargoObraDelet.ID)` para apagar o registro selecionado.

### 3.5. `Liberação de Acesso Screen`

**Objetivo**: Administrar os usuários do sistema e seus níveis de permissão.

**Componentes e Lógica**:

-   Tela de CRUD para a tabela `db_acessos`.
-   **`galContratos_16` (Galeria)**: Lista todos os usuários cadastrados, exibindo nome, e-mail e tipo de acesso.
-   **Ícones de Ação**: Assim como na tela de Configuração, ícones de Adicionar, Editar e Excluir permitem gerenciar os registros.
-   **`Form_DotUnidade_2` (Formulário)**: Permite criar um novo usuário ou editar um existente. O campo mais importante é o ComboBox `tipo_acesso`, que define se o usuário será um `cliente` (com acesso restrito) ou um `usuario` (com acesso administrativo).

---

## 4. Fontes de Dados

-   **`db_notificacoes`**: Tabela principal com todos os registros de notificação.
-   **`db_cidades_cadastradas`**: Tabela de apoio com informações de municípios e trechos de rodovia.
-   **`db_tipo_elemento`**: Tabela de apoio para os tipos de elementos notificáveis.
-   **`db_acessos`**: Tabela de segurança que armazena os usuários e seus níveis de permissão.
