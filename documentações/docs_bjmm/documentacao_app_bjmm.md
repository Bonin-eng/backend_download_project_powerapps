# Documentação do Aplicativo: Sistema Integrado de Gestão e Desenvolvimento BJMM

## 1. Visão Geral do Projeto

O aplicativo "Sistema Integrado de Gestão e Desenvolvimento BJMM" é uma solução abrangente desenvolvida em Power Apps para gerenciar o ciclo de vida de contratos, empreendimentos, orçamentos e fiscalizações. Ele centraliza informações críticas, permitindo o cadastro, controle e acompanhamento de todas as fases de um projeto, desde a criação do contrato até a medição e fiscalização da obra.

O sistema foi projetado para atender a diferentes perfis de usuários, como administradores, gestores de contrato e equipes de fiscalização, oferecendo uma interface organizada para cada módulo funcional.

---

## 2. Estrutura e Lógica do Aplicativo

### 2.1. Inicialização do Aplicativo (App.fx.yaml)

Ao iniciar o aplicativo, a lógica definida no `OnStart` é executada para configurar o ambiente do usuário.

- **Principais Ações:**
  - `ClearCollect(ColIconesMenus, ...)`: Carrega uma coleção de ícones SVG que são usados para identificar visualmente cada módulo no menu de navegação.
  - `ClearCollect(ColLogo, ...)`: Carrega os logotipos da empresa em formato SVG para serem exibidos na interface.
  - `ClearCollect(colTelas, ...)`: Cria a estrutura principal do menu de navegação. Esta coleção define os menus principais (ex: "CONTRATOS E EMPREENDIMENTOS", "CADASTROS", "ORÇAMENTO E MEDIÇÃO") e seus respectivos submenus, associando cada item a uma tela específica do aplicativo.

### 2.2. Tema e Estilo (Themes.json)

Este arquivo JSON define a paleta de cores, fontes e estilos visuais para todos os controles do aplicativo (botões, caixas de texto, rótulos, etc.). Isso garante uma identidade visual consistente em todas as telas, com cores primárias, de sucesso, erro, e para estados como "desabilitado" ou "hover".

---

## 3. Descrição das Telas e Funcionalidades

A seguir, uma análise detalhada de cada tela do aplicativo.

### 3.1. Tela Inicial (Tela Inicial Screen.fx.yaml)

- **Propósito:** É a tela de boas-vindas do sistema. Serve como um painel central de informações.
- **Lógica Principal:**
  - `OnVisible`: Define uma variável `varIniciarBICronometro` como `true`.
  - **PowerBI1 (Controle Power BI):** Exibe um dashboard do Power BI, fornecendo uma visão geral e consolidada dos dados mais importantes do sistema.
  - **Timer1 (Cronômetro):** Controla a exibição de uma tela de carregamento (`loading`) por 4.5 segundos enquanto o dashboard do Power BI é carregado, melhorando a experiência do usuário.

### 3.2. Módulo de Contratos

#### 3.2.1. Contratos (Contratos Screen.fx.yaml)

- **Propósito:** Tela principal para visualização e gerenciamento de contratos.
- **Lógica Principal:**
  - `OnVisible`: Calcula a vigência total de cada contrato (em dias e meses) somando os prazos dos seus "Elementos Contratuais" e armazena na coleção `colPrazoVigenciaDias`.
  - **Filtros:** Permite que o usuário filtre a lista de contratos por número, objeto, objeto reduzido e contratada.
  - **galContratos (Galeria):** Exibe a lista de contratos filtrados. Cada item da galeria mostra um resumo em HTML (`HtmlText1`) com informações como:
    - Nº do Contrato e Objeto Reduzido.
    - Dados da Contratada (Nome, CNPJ).
    - Prazos (Data de Assinatura, Término Previsto, Prazo Restante).
    - Alertas visuais ("🚨", "⚠️", "✅") para o prazo de vigência.
    - Valor e Status do Contrato.
  - **Seleção de Contrato:** Ao clicar em um contrato, o usuário é levado a uma visão detalhada (`cContratoSelect`) onde pode ver todas as informações do contrato e, se tiver permissão, entrar no modo de edição.

#### 3.2.2. Cadastro de Contratos (Cadastro de Contratos Screen.fx.yaml)

- **Propósito:** Formulário para registrar um novo contrato no sistema.
- **Lógica Principal:**
  - **Form_ContratoNew (Formulário):** Vinculado à fonte de dados `db_cont_contrato`. Contém campos para todas as informações contratuais, como número, objeto, partes envolvidas, valores, prazos e dotação orçamentária.
  - `OnSuccess`: Após o envio bem-sucedido do formulário:
    1.  Cria automaticamente um "Elemento Contratual" inicial na tabela `db_cont_econtratual`, registrando o valor e o prazo originais do contrato.
    2.  Reseta o formulário.
    3.  Navega de volta para a tela 'Contratos Screen'.

#### 3.2.3. Elemento Contratual (Elemento Contratual Screen.fx.yaml)

- **Propósito:** Gerenciar aditivos, reajustes ou outros componentes que alteram o contrato original.
- **Lógica Principal:**
  - **Filtros:** Permite buscar por número do contrato ou objeto reduzido.
  - **galContratos_1 (Galeria):** Lista todos os elementos contratuais, exibindo detalhes como descrição, data de assinatura, prazo e valor.
  - **Seleção e Edição:** Ao selecionar um item, um painel de detalhes (`cContratoSelect_1`) é exibido, permitindo a visualização e edição das informações do elemento contratual através do formulário `Form_ContratoEdit_1`.

#### 3.2.4. Cadastro Elemento Contratual (Cadastro Elemento Contratual Screen.fx.yaml)

- **Propósito:** Formulário para adicionar um novo elemento a um contrato existente (ex: um aditivo de prazo ou valor).
- **Lógica Principal:**
  - **Form_ElementoContratualNew (Formulário):** Vinculado à fonte de dados `db_cont_econtratual`. O usuário seleciona um contrato existente e preenche os dados do novo elemento, como tipo, descrição, data, prazo e valor.
  - `OnSuccess`: Após o cadastro, o formulário é resetado e o usuário retorna à tela 'Elemento Contratual Screen'.

### 3.3. Módulo de Empreendimentos

#### 3.3.1. Empreendimentos (Empreendimentos Screen.fx.yaml)

- **Propósito:** Visualizar e gerenciar os empreendimentos (obras) associados aos contratos.
- **Lógica Principal:**
  - **Filtros:** Permite buscar por número do contrato, objeto reduzido ou nome do empreendimento.
  - **galContratos_9 (Galeria):** Lista os empreendimentos, exibindo informações como construtora, nome do empreendimento, número de unidades habitacionais e endereço.
  - **Seleção e Edição:** Ao selecionar um item, um painel de detalhes (`cContratoSelect_3`) é exibido, permitindo a visualização e edição das informações através do formulário `Form_EmpreendimentoEdit`.

#### 3.3.2. Cadastro de Empreendimento (Cadastro de Empreendimento Screen.fx.yaml)

- **Propósito:** Formulário para registrar um novo empreendimento e associá-lo a um contrato.
- **Lógica Principal:**
  - **Form_EmpreendimentoNew (Formulário):** Vinculado à fonte de dados `db_cont_empreendimento`. O usuário seleciona um contrato e preenche os detalhes do empreendimento, como nome, objeto, endereço, número de unidades, QRE e coordenadas geográficas.
  - `OnSuccess`: Após o cadastro, o usuário é redirecionado para a tela 'Empreendimentos Screen'.

### 3.4. Módulo de Orçamento e Medição

#### 3.4.1. Base Orçamentária (Base Orçamentária Screen.fx.yaml)

- **Propósito:** Consultar a planilha orçamentária detalhada de um contrato/empreendimento.
- **Lógica Principal:**
  - **Filtros:** O usuário seleciona um contrato para carregar os dados.
  - **Estrutura Hierárquica:** A tela exibe uma galeria aninhada. A primeira galeria (`Gallery3_4`) lista os empreendimentos do contrato. Ao expandir um empreendimento, a segunda galeria (`Gallery2`) mostra os itens da base orçamentária agrupados.
  - **Visualização Detalhada:** É possível expandir cada item para ver a planilha orçamentária completa, renderizada com HTML (`htmltxt_historico_viajantes_1`), mostrando colunas como Item, Descrição, Unidade, Quantidade, Preço Unitário, Valor Total e Valor com BDI.

#### 3.4.2. Planejamento Orçamentário (Planejamento Orçamentário Screen.fx.yaml)

- **Propósito:** Visualizar o planejamento financeiro de um empreendimento ao longo do tempo.
- **Lógica Principal:**
  - **Seleção de Contrato:** O usuário filtra os contratos para encontrar o desejado.
  - **Visualização em Tabela:** Ao expandir um empreendimento, a tela exibe uma tabela complexa (`htmltxt_historico_viajantes_2`) que mostra o planejamento orçamentário. A tabela é organizada por grupos de serviço e detalha o valor e o percentual previsto para cada mês do cronograma.

#### 3.4.3. Boletim de Medição (Boletim Medição Screen.fx.yaml)

- **Propósito:** Acompanhar as medições de serviços executados em um contrato.
- **Lógica Principal:**
  - **Carregamento de Dados:** Ao selecionar um contrato, a tela carrega todos os registros de medição da base `db_fin_medicao` de forma paginada para otimizar a performance.
  - **Visualização de Boletins:** Uma galeria (`Gallery1_3`) lista os boletins de medição, agrupados por número. Cada item exibe o período, competência, valor medido no período e valor acumulado.
  - **Detalhes da Medição:** O usuário pode expandir um boletim para ver os detalhes dos itens medidos.

#### 3.4.4. Resumo Orçamentário (Resumo Orçamentário Screen.fx.yaml)

- **Propósito:** Fornecer uma visão consolidada e resumida dos orçamentos dos empreendimentos de um contrato.
- **Lógica Principal:**
  - **Filtros:** Permite buscar por contrato, objeto ou contratada.
  - **Galeria de Contratos (`Gallery3_3`):** Lista os contratos.
  - **Expansão de Detalhes:** Ao expandir um contrato, uma galeria aninhada (`Gallery1`) mostra os empreendimentos associados.
  - **Resumo Financeiro:** Ao expandir um empreendimento, a tela exibe uma tabela HTML (`htmltxt_historico_viajantes`) com o resumo da planilha orçamentária, agrupando os valores por item principal.
  - **Gerenciamento de Orçamento:** Oferece opções para "Desativar Orçamento" ou "Apagar Orçamento", ações que exigem confirmação do usuário.

### 3.5. Módulo de Acompanhamento e Fiscalização

#### 3.5.1. Acompanhamento de Pesquisas (Acompanhamento de Pesquisas Screen.fx.yaml)

- **Propósito:** Registrar e consultar as fiscalizações (pesquisas de campo) realizadas nos empreendimentos.
- **Lógica Principal:**
  - **Filtros:** O usuário pode filtrar as fiscalizações por Contrato, tipo de Ficha (pesquisa) e Data.
  - **Galeria de Contratos (`Gallery3_8`):** Exibe os contratos. Ao expandir um, a galeria aninhada (`Gallery1_5`) mostra as fiscalizações realizadas.
  - **Visualização de Perguntas:** Ao selecionar uma fiscalização, um painel (`cContratoSelect_5`) é aberto, listando as perguntas daquela ficha de fiscalização.
  - **Registro de Respostas:** O usuário pode selecionar uma pergunta e preencher os detalhes em um formulário (`Form1`), indicando a conformidade, criticidade (se não conforme), observações e anexando imagens como evidência.

#### 3.5.2. Controle de Pesquisas (Controle de Pesquisas Screen.fx.yaml)

- **Propósito:** Tela administrativa para criar e gerenciar os modelos de fichas de fiscalização (tipos de pesquisa) e suas respectivas perguntas.
- **Lógica Principal:**
  - **Lista de Pesquisas (`galContratos_10`):** Exibe todos os tipos de pesquisa cadastrados.
  - **Gerenciamento de Perguntas:** Ao selecionar um tipo de pesquisa, um painel (`cContratoSelect_4`) mostra as perguntas associadas. O usuário pode:
    - **Criar Nova Pergunta:** Abre um formulário (`cCriarPergunta`) para adicionar uma nova pergunta à ficha.
    - **Editar Pergunta:** Abre um formulário (`cEditarPergunta`) para modificar uma pergunta existente.
    - **Excluir Pergunta:** Remove uma pergunta da ficha, com uma caixa de diálogo de confirmação.

#### 3.5.3. Cadastro de Ficha de Fiscalização (Cadastro de Ficha de Fiscalização Screen.fx.yaml)

- **Propósito:** Formulário para criar um novo tipo de pesquisa (ficha de fiscalização).
- **Lógica Principal:**
  - **Form_FiscalizacaoNew (Formulário):** Vinculado à fonte de dados `db_acomp_tipo_pesquisa`. Gera um ID sequencial para a nova ficha e permite ao usuário definir o nome (ex: "Checklist de Segurança do Trabalho").

### 3.6. Módulo Administrativo e Cadastros

#### 3.6.1. Controle de Funcionários (Controle de Funcionarios Screen.fx.yaml)

- **Propósito:** Gerenciar os colaboradores cadastrados no sistema.
- **Lógica Principal:**
  - **Filtros:** Permite buscar funcionários por nome ou área/equipe.
  - **galContratos_8 (Galeria):** Lista os funcionários com suas principais informações (Nome, CPF, Email, Cargo, etc.).
  - **Seleção e Edição:** Ao selecionar um funcionário, um painel (`cContratoSelect_2`) é exibido com o formulário `Form_ColaboradorEdit` para visualização e edição dos dados.

#### 3.6.2. Cadastro de Funcionários (Cadastro de Funcionarios Screen.fx.yaml)

- **Propósito:** Formulário para registrar novos colaboradores.
- **Lógica Principal:**
  - **Form_ColaboradorNew (Formulário):** Vinculado à fonte de dados `dbCadastroFuncionario`. Contém campos para dados pessoais e profissionais, como Nome, CPF, Email, Cargo, Equipe e Tipo de Contratação.

#### 3.6.3. Cadastro de Contato na Obra (Cadastro de Contato na Obra Screen.fx.yaml)

- **Propósito:** Registrar contatos específicos de uma obra (ex: engenheiros, fiscais).
- **Lógica Principal:**
  - **Form_CadastroPessoaObra (Formulário):** Vinculado à fonte de dados `db_cont_pessoas_obra`. O usuário seleciona o contrato, o cargo do contato e preenche os dados como nome, telefone e email.

#### 3.6.4. Cadastro Cargo Obra (Cadastro Cargo Obra Screen.fx.yaml)

- **Propósito:** Tela administrativa para gerenciar os cargos que podem ser atribuídos aos contatos da obra.
- **Lógica Principal:**
  - **Visualização e Gerenciamento:** A tela é dividida em duas seções:
    - À esquerda, uma galeria (`galContratos_15`) lista os cargos existentes.
    - À direita, um formulário (`Form_DotUnidade_1`) permite criar um novo cargo ou editar um cargo selecionado.

#### 3.6.5. Dotação Orçamentária (Dotação Orçamentária Screen.fx.yaml)

- **Propósito:** Tela de configuração para gerenciar as categorias usadas na dotação orçamentária dos contratos.
- **Lógica Principal:**
  - **Navegação por Abas:** A tela utiliza botões para alternar a visibilidade de diferentes seções (Unidade, Órgão, Programática, Despesa, Recurso, Crédito).
  - **Gerenciamento de Categorias:** Para cada seção, há uma galeria que lista os itens cadastrados e um formulário que permite adicionar ou editar registros, garantindo que os contratos possam ser classificados corretamente em termos orçamentários.
