# Documentação do Aplicativo de Arrolamento URBHIS

## 1. Visão Geral do Projeto

O aplicativo **Arrolamento URBHIS** é uma ferramenta desenvolvida em Power Apps para realizar o cadastro e gerenciamento de informações de arrolamento de imóveis em diferentes núcleos urbanos. Ele permite que os usuários de campo coletem dados detalhados sobre os imóveis, seus ocupantes e características, além de gerenciar os logradouros (ruas) de cada núcleo.

O projeto foi desenhado para ser intuitivo, guiando o usuário através de um fluxo claro:
1.  Seleção do núcleo de trabalho.
2.  Gerenciamento de logradouros (ruas) do núcleo.
3.  Criação de novos registros de arrolamento (selos).
4.  Edição e consulta dos registros existentes.

---

## 2. Estrutura do Aplicativo e Navegação

O aplicativo é composto por 5 telas principais e uma tela de inicialização que define o comportamento global.

-   **`App` (Inicialização)**: Ponto de entrada que carrega dados iniciais e define variáveis globais.
-   **`Tela_Inicio`**: A tela principal, onde o usuário seleciona o núcleo e acessa as principais funcionalidades.
-   **`Tela_Cadastro_Ruas`**: Tela dedicada ao cadastro e gerenciamento de ruas para um núcleo específico.
-   **`Tela_Novo_Formulario`**: Formulário completo para a criação de um novo registro de arrolamento (selo).
-   **`Tela_Editar_Formulario`**: Uma versão do formulário para editar um registro de arrolamento existente.
-   **`Tela_Selos`**: Tela para visualizar, filtrar e selecionar os registros de arrolamento (selos) existentes para edição ou exclusão.

O fluxo de navegação é centrado na `Tela_Inicio`, que age como um hub para as demais telas.

---

## 3. Componentes Globais (`App.fx.yaml`)

Este arquivo define o comportamento inicial e as configurações globais do aplicativo.

### Ação `OnStart`

Quando o aplicativo é iniciado, as seguintes ações são executadas:

```powerapps
ClearCollect(colDb_Nucleos, bdNucleos[@Nucleo]);
Set(varNucleo, cmb_Nucleo.Selected.Nucleo);
Set(varRecuo, 10)
```

-   **`ClearCollect(colDb_Nucleos, bdNucleos[@Nucleo])`**: Cria uma coleção local chamada `colDb_Nucleos` e a preenche com a lista de núcleos da fonte de dados `bdNucleos`. Isso otimiza o acesso aos dados de núcleos ao longo do uso do app.
-   **`Set(varNucleo, cmb_Nucleo.Selected.Nucleo)`**: Inicializa uma variável global `varNucleo` com o valor selecionado no ComboBox `cmb_Nucleo` (presente na `Tela_Inicio`).
-   **`Set(varRecuo, 10)`**: Define uma variável `varRecuo` com o valor `10`. Esta variável é usada consistentemente em todo o aplicativo para definir paddings e margens, garantindo um design coeso.

### Tema

-   **`Theme: =PowerAppsTheme`**: O aplicativo utiliza um tema customizado do Power Apps, definido no arquivo `Themes.json`, para manter a consistência visual.

---

## 4. Análise das Telas

### 4.1. `Tela_Inicio`

É a porta de entrada do aplicativo.

**Objetivo**: Permitir que o usuário selecione um núcleo de trabalho e acesse as funcionalidades de cadastro de logradouros, criação e edição de selos.

**Componentes Principais e Lógica**:

-   **`cmb_Nucleo` (ComboBox)**:
    -   **Items**: `Filter(Sort(colDb_Nucleos, Nucleo, SortOrder.Ascending),Nucleo<>"DIVERSOS NÚCLEOS")` - Exibe a lista de núcleos da coleção `colDb_Nucleos` (carregada no `OnStart`), ordenada em ordem alfabética e excluindo o item "DIVERSOS NÚCLEOS".
    -   **OnChange**:
        -   `Set(varNucleo, cmb_Nucleo.Selected.Nucleo)`: Atualiza a variável global `varNucleo` com o núcleo selecionado.
        -   `Set(varNucleoID, LookUp(bdNucleos, Nucleo = cmb_Nucleo.Selected.Nucleo, ID))`: Armazena o ID do núcleo selecionado na variável `varNucleoID`.
        -   `ClearCollect(colDb_Nucleos_Ruas, Filter(db_Nucleos_Ruas, NÚCLEO = varNucleo))`: Filtra a fonte de dados `db_Nucleos_Ruas` para carregar em uma coleção local (`colDb_Nucleos_Ruas`) apenas as ruas que pertencem ao núcleo selecionado.

-   **`btn_Novo_Logradouro` (Botão)**:
    -   **OnSelect**: `If(IsBlank(varNucleo), UpdateContext({contPopup: true}), Navigate(Tela_Cadastro_Ruas))` - Verifica se um núcleo foi selecionado. Se não, exibe um pop-up de aviso (`contPopup`). Se sim, navega para a `Tela_Cadastro_Ruas`.

-   **`btn_Novo_Selo` (Botão)**:
    -   **OnSelect**: `If(IsBlank(varNucleo), UpdateContext({contPopup: true}), Navigate(Tela_Novo_Formulario))` - Similar ao botão anterior, mas navega para a `Tela_Novo_Formulario` para criar um novo selo.

-   **`btn_Editar_Selo` (Botão)**:
    -   **OnSelect**: `If(IsBlank(varNucleo), UpdateContext({contPopup: true}), Navigate(Tela_Selos))` - Verifica a seleção do núcleo e navega para a `Tela_Selos`, onde o usuário pode ver e editar os selos existentes.

-   **`grp_PopUp_Nucleo_Aviso` (Grupo de Pop-up)**:
    -   **Visible**: `contPopup` - Fica visível apenas quando a variável de contexto `contPopup` é `true`. É usado para alertar o usuário que ele precisa selecionar um núcleo antes de prosseguir.

### 4.2. `Tela_Cadastro_Ruas`

**Objetivo**: Gerenciar (cadastrar, editar e excluir) os logradouros de um núcleo específico.

**Componentes Principais e Lógica**:

-   **`lbl_Titulo_4` (Label)**: Exibe o nome do núcleo selecionado (`cmb_Nucleo.Selected.Nucleo`), informando ao usuário o contexto atual.

-   **`txt_Logradouro_Cadastro` (Caixa de Texto)**: Campo para inserir ou editar o nome de um logradouro.
    -   **Default**: `varLogradouro.NOME_RUA` - Se a variável `varLogradouro` estiver preenchida (modo de edição), exibe o nome da rua a ser editada.

-   **`btn_Salvar_Logradouro` (Botão)**:
    -   **Text**: `If(IsBlank(varLogradouro),"CADASTRAR LOGRADOURO","EDITAR LOGRADOURO")` - O texto do botão muda dinamicamente se o usuário está criando um novo logradouro ou editando um existente.
    -   **OnSelect**:
        -   `Patch(db_Nucleos_Ruas, If(IsBlank(varLogradouro), Defaults(db_Nucleos_Ruas), LookUp(db_Nucleos_Ruas, ID = varLogradouro.ID)), { ... })` - Função principal que salva os dados.
            -   Se `varLogradouro` está em branco, cria um novo registro (`Defaults`).
            -   Se `varLogradouro` contém um item, atualiza (`LookUp`) o registro existente na fonte de dados `db_Nucleos_Ruas`.
        -   `Set(varLogradouro, Blank())`: Limpa a variável `varLogradouro` após a operação, resetando o formulário para o modo de cadastro.

-   **`glr_Selecao_Ruas` (Galeria)**:
    -   **Items**: `Sort(Filter(db_Nucleos_Ruas, ID_REF = varNucleoID), NOME_RUA, SortOrder.Ascending)` - Exibe a lista de ruas do núcleo selecionado (`varNucleoID`), ordenadas pelo nome.
    -   **`icn_Editar_1` (Ícone)**: `OnSelect: Set(varLogradouro, ThisItem)` - Ao clicar, armazena o item selecionado da galeria na variável `varLogradouro`, colocando o formulário em modo de edição.
    -   **`icn_Deletar_1` (Ícone)**: `OnSelect: UpdateContext({contPopup_Selecao_Remover_Logradouro: true})` - Ativa um pop-up de confirmação para exclusão.

-   **`grp_PopUp_Edicao_Remover_Logradouro` (Grupo de Pop-up)**:
    -   Contém a lógica para confirmar a exclusão de um logradouro. O botão "SIM" executa `RemoveIf(db_Nucleos_Ruas, ID = contPopup_Selecao_Remover_Logradouro_ID)`, que remove o item da fonte de dados.

### 4.3. `Tela_Novo_Formulario`

**Objetivo**: Coletar todos os dados necessários para um novo registro de arrolamento (selo).

**Componentes Principais e Lógica**:

Esta tela é um formulário extenso com múltiplos campos de entrada. A lógica principal está no botão de salvar.

-   **Campos de Entrada**: Inclui `txt_Quadra`, `txt_Lote`, `rad_Tipo_Imovel`, `drp_Logradouro`, `txt_Numero`, informações do ocupante, proprietário, e muitos outros. A visibilidade e o comportamento de alguns campos são dinâmicos.
    -   Por exemplo, `rad_Tipo_Empreendimento` só fica visível se `rad_Tipo_Imovel.Selected.Value = "EMPREEND."`.

-   **`btn_Salvar` (Botão)**:
    -   **OnSelect**: `Patch(db_Arrolamento, Defaults(db_Arrolamento), { ... })` - Cria um novo registro na fonte de dados `db_Arrolamento`, preenchendo cada coluna com os valores dos respectivos campos do formulário.
    -   A fórmula coleta dados de todos os `txt_`, `drp_`, `rad_` e outros controles para montar o registro a ser salvo.
    -   Após salvar, executa `Navigate(Tela_Inicio)` e `Reset()` em todos os campos do formulário para limpá-los para a próxima entrada.

-   **`Anexos_DataCard1` (Cartão de Anexos)**: Permite ao usuário adicionar arquivos (fotos, documentos) ao registro.

### 4.4. `Tela_Editar_Formulario`

**Objetivo**: Permitir a edição de um registro de arrolamento (selo) previamente criado.

**Componentes Principais e Lógica**:

Esta tela é uma cópia quase idêntica da `Tela_Novo_Formulario`, com as seguintes diferenças cruciais:

-   **Preenchimento de Dados**: Os campos do formulário são preenchidos com os dados do item selecionado na `Tela_Selos`, que é armazenado na variável `varThisItem`.
    -   **Default**: A propriedade `Default` de cada campo é definida para o valor correspondente em `varThisItem` (ex: `txt_Quadra_1.Default: =varThisItem.Quadra`).

-   **`btn_Salvar_1` (Botão)**:
    -   **OnSelect**: `Patch(db_Arrolamento, varThisItem, { ... })` - Em vez de criar um novo registro com `Defaults`, a função `Patch` aqui usa `varThisItem` como segundo argumento. Isso instrui o Power Apps a **atualizar** o registro existente no banco de dados `db_Arrolamento`.

-   **Campos Desabilitados**: Alguns campos que identificam unicamente o registro, como `txt_Quadra_1` e `txt_Lote_1`, estão com `DisplayMode: DisplayMode.Disabled` para impedir que sejam alterados.

### 4.5. `Tela_Selos`

**Objetivo**: Listar, filtrar e gerenciar os selos existentes de um núcleo.

**Componentes Principais e Lógica**:

-   **Filtros (`drp_Perimetro_1`, `drp_Quadra`, `drp_Lote`)**:
    -   São Dropdowns que permitem ao usuário refinar a busca por selos.
    -   Os itens de cada dropdown são populados dinamicamente com base nas seleções dos filtros anteriores usando `Distinct()` para evitar duplicatas. Por exemplo, `drp_Quadra` mostra apenas as quadras do perímetro selecionado.

-   **`glr_Selecao` (Galeria)**:
    -   **Items**: `Sort(Sort(Filter(db_Arrolamento, Nucleo = cmb_Nucleo.Selected.Nucleo, Perimetro = drp_Perimetro_1.SelectedText.Value, ...),Bloco),Imovel)` - Exibe os registros da fonte de dados `db_Arrolamento` que correspondem ao núcleo e aos filtros selecionados. A lista é ordenada por Bloco e Imóvel.
    -   A galeria exibe um resumo das informações de cada selo, como Bloco, Imóvel, Logradouro e Ocupante.

-   **`icn_Editar` (Ícone)**:
    -   **OnSelect**:
        -   `Set(varThisItem, ThisItem)`: Armazena os dados do selo selecionado na variável global `varThisItem`.
        -   `Navigate(Tela_Editar_Formulario)`: Navega para a tela de edição, que usará `varThisItem` para preencher o formulário.

-   **`icn_Deletar` (Ícone)**:
    -   **OnSelect**: `UpdateContext({contPopup_Selecao_Remover: true})` - Ativa o pop-up de confirmação de exclusão. A lógica de exclusão no pop-up é similar à da tela de cadastro de ruas, usando `RemoveIf` na fonte de dados `db_Arrolamento`.

---

## 5. Fontes de Dados

O aplicativo se conecta às seguintes fontes de dados (provavelmente listas do SharePoint ou tabelas do Dataverse):

-   **`bdNucleos`**: Armazena a lista de núcleos disponíveis.
-   **`db_Nucleos_Ruas`**: Contém o cadastro de ruas (logradouros) associadas a cada núcleo.
-   **`db_Arrolamento`**: A tabela principal onde todos os dados dos formulários de arrolamento (selos) são armazenados.

---

## 6. Tema e Estilo (`Themes.json`)

O arquivo `Themes.json` define a identidade visual do aplicativo.

-   **Paleta de Cores**: Define um conjunto de cores primárias, de texto, de fundo e de status (sucesso, erro, aviso). A cor primária principal é um tom de azul (`RGBA(56, 96, 178, 1)`).
-   **Estilos de Controle**: Fornece estilos padronizados para todos os tipos de controles (labels, botões, caixas de texto, etc.), garantindo que a aparência de fontes, tamanhos, bordas e cores seja consistente em todo o aplicativo.
-   **Variáveis de Design**: Inclui valores numéricos para tamanhos de fonte, espessura de borda e raios de contêiner, contribuindo para a uniformidade do design.

Isso garante que o aplicativo tenha uma aparência profissional e coesa, e facilita a manutenção do estilo visual.
