# Documentação do Aplicativo de Fiscalização de Obras (BJMM)

## 1. Visão Geral do Projeto

O aplicativo **Sistema de Fiscalização BJMM** é uma ferramenta móvel desenvolvida em Power Apps para dar suporte às equipes de fiscalização de obras. Ele permite que os fiscais selecionem contratos e empreendimentos, iniciem novas vistorias baseadas em checklists pré-definidos, registrem conformidades e não conformidades (com fotos e observações), e coletem assinaturas digitais para formalizar a vistoria.

O fluxo de trabalho principal é:
1.  O usuário seleciona um contrato na tela inicial.
2.  Visualiza os empreendimentos e as fiscalizações existentes para aquele contrato.
3.  Inicia uma nova fiscalização, que gera um checklist de perguntas.
4.  Responde ao checklist, registrando detalhes para itens "Não Conforme".
5.  Conclui a fiscalização com as assinaturas dos responsáveis.

O aplicativo também possui uma tela para consulta rápida de contatos da obra, com integração para chamadas e WhatsApp.

---

## 2. Estrutura do Aplicativo e Navegação

O aplicativo utiliza um componente de navegação lateral (menu hambúrguer) que é alimentado por uma coleção criada na inicialização.

-   **`App` (Inicialização)**: Ponto de entrada que configura a navegação e variáveis globais.
-   **`Home Screen`**: Tela inicial que exibe os dados do usuário e uma lista de contratos para acesso rápido.
-   **`Contratos Screen`**: Tela principal para visualizar empreendimentos e suas respectivas fiscalizações. É aqui que novas fiscalizações são iniciadas.
-   **`Nova Fiscalização Screen`**: O coração do aplicativo, onde o checklist da fiscalização é preenchido.
-   **`Contatos Screen`**: Uma lista de contatos relacionados às obras, com funcionalidades de comunicação direta.
-   **`Geral Screen`**: Uma tela que parece ser um template ou base, pois contém apenas o componente geral `cpm_Tela_Geral` e não possui lógica própria visível.

### Navegação Dinâmica (`App.fx.yaml`)

A navegação é controlada pela coleção `colTelas`, criada no `OnStart`:

```powerapps
ClearCollect(
    colTelas,
    Table(
        {NomeTela: "Home", Tela: 'Home Screen', Icon: Icon.Hamburger},
        {NomeTela: "Fiscalizações", Tela: 'Contratos Screen', Icon: Icon.Notebook},
        {NomeTela: "Contatos", Tela: 'Contatos Screen', Icon: Icon.People},
        {NomeTela: "Nova Fiscalização", Tela: 'Nova Fiscalização Screen', Icon: Icon.AddDocument}
    )
)
```

Esta coleção alimenta um componente de menu (provavelmente dentro do `cpm_Tela_Geral`), permitindo que o menu seja facilmente modificado apenas alterando esta tabela.

---

## 3. Análise das Telas

### 3.1. `Home Screen`

**Objetivo**: Servir como painel de boas-vindas e ponto de partida para a seleção de um contrato.

**Componentes e Lógica**:

-   **Informações do Usuário**: Exibe a foto (`UsuáriosdoOffice365.UserPhoto`) e o nome (`User().FullName`) do usuário logado.
-   **`TextInput1_17` (ComboBox)**: Permite filtrar a lista de contratos exibida na galeria principal.
-   **`Gallery3_3` (Galeria de Contratos)**:
    -   **Items**: `Filter(db_cont_contrato, IsBlank(TextInput1_17.Selected.n_contrato) || n_contrato = TextInput1_17.Selected.n_contrato)` - Lista os contratos da fonte de dados `db_cont_contrato`, aplicando o filtro do ComboBox, se houver.
    -   **`Button4_1` (Botão Sobreposto)**: Um botão transparente cobre cada item da galeria.
        -   **OnSelect**: `Set(varContratoSelecionado, ThisItem.n_contrato); Navigate('Contratos Screen');` - Armazena o número do contrato selecionado na variável `varContratoSelecionado` e navega para a tela de fiscalizações.

### 3.2. `Contratos Screen`

**Objetivo**: Exibir os empreendimentos de um contrato, suas fiscalizações, e permitir a criação de novas vistorias.

**Componentes e Lógica**:

-   **Filtros (`TextInput1_14`, `TextInput1_15`)**: ComboBoxes para filtrar a lista por Contrato e Empreendimento.
-   **`Button1_1` (Botão "NOVA PESQUISA")**: `OnSelect: Set(varPopupNovaFiscalização, true)` - Abre um pop-up para iniciar o processo de criação de uma nova fiscalização.
-   **`Gallery3_5` (Galeria de Empreendimentos)**:
    -   Exibe os empreendimentos de `db_cont_empreendimento` com base nos filtros.
    -   **Ícones de Mapa (`Image1`, `Image3`)**: Lançam os aplicativos Waze e Google Maps com a latitude e longitude do empreendimento.
    -   **`icoSeta_4` (Ícone de Expansão)**: `OnSelect: UpdateContext({varExpandido: If(varExpandido = ThisItem.id_contrato_empreendimento, Blank(), ThisItem.id_contrato_empreendimento)})` - Controla a visibilidade da galeria aninhada, mostrando ou escondendo as fiscalizações do empreendimento selecionado.
-   **`Gallery1_2` (Galeria Aninhada de Fiscalizações)**:
    -   **Visible**: `varExpandido = ThisItem.id_contrato_empreendimento` - Só é visível para o item expandido na galeria pai.
    -   **Items**: `Filter(db_acomp_empreendimento_fiscalizacao, id_empreendimento = ThisItem.id_contrato_empreendimento)` - Lista as fiscalizações associadas ao empreendimento.
    -   **`Button4` (Botão Sobreposto)**: `OnSelect: Navigate('Nova Fiscalização Screen'); Set(var_Nova_Fiscalizacao, ThisItem.id_fiscalizacao);` - Navega para a tela de checklist, passando o ID da fiscalização selecionada para que ela possa ser continuada.

-   **`Container16` (Pop-up de Nova Fiscalização)**:
    -   **Visibilidade**: Controlada pela variável `varPopupNovaFiscalização`.
    -   Contém campos para selecionar Contrato, Empreendimento, Data e o Tipo de Pesquisa.
    -   **`Button1` (Botão "INICIAR PESQUISA")**: Este botão contém a lógica mais crítica da tela.
        1.  **Validação**: `If(IsBlank(...), Notify(...))` - Verifica se os campos essenciais foram preenchidos.
        2.  **Geração de ID**: `Set(Var_id_fiscalizacao, Concatenate(...))` - Cria um ID único para a nova fiscalização.
        3.  **Criação do Registro Mestre**: `Patch(db_acomp_empreendimento_fiscalizacao, ...)` - Cria o registro principal da fiscalização com os dados do formulário e a geolocalização (`Location.Latitude`, `Location.Longitude`).
        4.  **Criação do Checklist**: `ClearCollect(colDadosParaPatch, ...)` cria uma coleção com todas as perguntas (`db_acomp_perguntas_pesquisas`) aplicáveis ao tipo de pesquisa selecionado. Em seguida, `ForAll(colDadosParaPatch, Patch(db_acomp_fiscalizacoes, ...))` itera sobre essa coleção e cria um registro para cada pergunta na tabela `db_acomp_fiscalizacoes`, pré-populando o checklist que será exibido na próxima tela.

### 3.3. `Nova Fiscalização Screen`

**Objetivo**: Executar a fiscalização, respondendo a um checklist dinâmico.

**Componentes e Lógica**:

-   **`OnVisible`**: `ClearCollect(col_PerguntaFiscalizacaoID, Filter(db_acomp_fiscalizacoes, id_fiscalizacao = Text(var_Nova_Fiscalizacao)))` - Carrega todas as perguntas/respostas para a fiscalização atual (identificada por `var_Nova_Fiscalizacao`) em uma coleção local.
-   **`Gallery2_1` (Galeria de Perguntas)**:
    -   **Items**: `col_PerguntaFiscalizacaoID` - Exibe o checklist.
    -   **`HtmlText1_6` (Visualizador HTML)**: Renderiza cada item da galeria. O fundo muda de cor (`#b2e8b2` para Conforme, `#f28b82` para Não Conforme) com base no status da resposta.
    -   **Botões de Resposta Rápida (`Button_C_1`, `Button_NC_1`, `Button_NV_1`)**: Botões transparentes sobrepostos ao HTML.
        -   **Conforme (C) / Não se Verifica (NV)**: Executam um `Patch` simples para atualizar o status da conformidade diretamente na fonte de dados `db_acomp_fiscalizacoes` e recarregam a coleção.
        -   **Não Conforme (NC)**: `Set(var_PopRespostaForms, true); Set(var_PerguntaFiscalizacao, ThisItem);` - Abre um pop-up para detalhar a não conformidade e armazena o item da pergunta atual em uma variável.
-   **`Container12` (Pop-up de Não Conformidade)**:
    -   **`Form1` (Formulário)**: Vinculado a `db_acomp_fiscalizacoes` e ao item `var_PerguntaFiscalizacao`.
    -   Permite ao usuário registrar a **Criticidade**, adicionar **Anexos** (fotos), definir um **Prazo de Resolução** e escrever **Observações**.
    -   O botão "CONCLUIR ✅" (`Button3_1`) executa `SubmitForm(Form1)` para salvar os detalhes.
-   **`Container7_2` (Pop-up de Conclusão da Fiscalização)**:
    -   Aberto pelo botão "CONCLUIR FISCALIZAÇÃO" (`Button1_3`).
    -   **`Form1_1` (Formulário)**: Vinculado ao registro mestre da fiscalização em `db_acomp_empreendimento_fiscalizacao`.
    -   Permite registrar o status final ("Liberado pra Obra?"), o responsável da área e o fiscal da gerenciadora.
    -   **`AddPicture3` e `AddPicture4` (Controles de Tinta)**: Capturam as assinaturas digitais.
    -   O botão "CONCLUIR ✅" (`Button3_3`) submete o formulário, finalizando a vistoria, e navega de volta para a tela de contratos.

### 3.4. `Contatos Screen`

**Objetivo**: Fornecer uma lista de contatos das obras com acesso rápido a meios de comunicação.

**Componentes e Lógica**:

-   **`Gallery3_6` (Galeria de Contratos)**: Funciona de forma similar às outras telas, listando contratos e permitindo a expansão para ver detalhes.
-   **`Gallery1_3` (Galeria Aninhada de Contatos)**:
    -   **Items**: `Filter(db_cont_pessoas_obra, id_contrato = ThisItem.n_contrato && status = "Ativo")` - Lista os contatos ativos para o contrato expandido.
    -   **`Image4` (Ícone do WhatsApp)**: `OnSelect: Launch("whatsapp://send?phone=...")` - Abre o WhatsApp para iniciar uma conversa com o contato. A fórmula limpa caracteres especiais do número de telefone.
    -   **`Image6` (Ícone de Telefone)**: `OnSelect: Launch("tel:...")` - Abre o discador do celular com o número do contato.

---

## 4. Fontes de Dados

O aplicativo utiliza um conjunto de tabelas inter-relacionadas, que provavelmente residem no Dataverse ou SharePoint:

-   **`db_cont_contrato`**: Tabela mestre de contratos.
-   **`db_cont_empreendimento`**: Tabela de empreendimentos, ligada aos contratos.
-   **`db_cont_pessoas_obra`**: Armazena os contatos (pessoas) de cada obra/contrato.
-   **`db_cont_pessoas_obra_cargo`**: Tabela de cargos/responsabilidades.
-   **`db_acomp_tipo_pesquisa`**: Define os diferentes tipos de fiscalização/checklist disponíveis.
-   **`db_acomp_perguntas_pesquisas`**: Contém todas as perguntas possíveis para todos os tipos de checklist.
-   **`db_acomp_empreendimento_fiscalizacao`**: Tabela mestre para cada instância de fiscalização realizada.
-   **`db_acomp_fiscalizacoes`**: Tabela de detalhes que armazena a resposta para cada pergunta de cada fiscalização.
-   **`dbCadastroFuncionario`**: Lista de funcionários (fiscais) para seleção no momento da conclusão.

---

## 5. Tema e Estilo (`Themes.json`)

Assim como o aplicativo anterior, este projeto utiliza um arquivo `Themes.json` para garantir uma identidade visual consistente. A paleta de cores é dominada por um azul escuro (`RGBA(45, 65, 93, 1)`) e branco, criando um visual profissional e de alto contraste, adequado para um aplicativo corporativo.
