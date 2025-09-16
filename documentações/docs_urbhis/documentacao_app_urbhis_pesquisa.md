# Documentação do Aplicativo: Pesquisa Socioeconômica (URBHIS)

## 1. Visão Geral do Projeto

O aplicativo **Pesquisa Socioeconômica** foi desenvolvido em Power Apps para facilitar a coleta de dados socioeconômicos de cidadãos, provavelmente no contexto de projetos de habitação e urbanismo (URBHIS).

O objetivo principal é permitir que um agente (o usuário do aplicativo) realize uma pesquisa detalhada com um indivíduo, preenchendo um formulário extenso diretamente no aplicativo. O fluxo de trabalho começa com a identificação do cidadão por meio do CPF e, em seguida, avança para o preenchimento do questionário.

## 2. Estrutura e Navegação do Aplicativo

A estrutura de navegação e as configurações globais do aplicativo são definidas no arquivo `App.fx.yaml`.

### Lógica de Inicialização (`App.OnStart`)

Ao iniciar, o aplicativo executa a seguinte ação:

```powerapps
ClearCollect(
    colTelas,
    Table(
        {NomeTela: "Fiscalizações", Tela: 'Contratos Screen', Icon: Icon.Notebook},
        {NomeTela: "Pesquisa", Tela: 'Pesquisa Screen', Icon: Icon.People}
    )
)
```

- **`ClearCollect(colTelas, ...)`**: Esta função cria uma coleção em memória chamada `colTelas`. Esta coleção funciona como um menu de navegação ou um diretório de telas para o aplicativo.
- **Telas na Coleção**:
  - **Fiscalizações**: Aponta para a tela `'Contratos Screen'`.
  - **Pesquisa**: Aponta para a tela `'Pesquisa Screen'`.

Isso sugere que a tela inicial (ou a principal) é a `'Contratos Screen'`.

### Tema Visual (`Themes.json`)

O arquivo `Themes.json` define a identidade visual do aplicativo. Ele contém uma paleta de cores customizada (`palette`) e estilos (`styles`) que são aplicados a todos os controles (telas, botões, caixas de texto, etc.). Isso garante uma aparência consistente em todo o aplicativo, com cores primárias, fontes e tamanhos de texto padronizados.

## 3. Detalhamento das Telas

O aplicativo é composto por duas telas principais.

### 3.1. `Contratos Screen` (Tela de Busca)

Esta é a tela inicial do fluxo de pesquisa.

#### **Objetivo**

O propósito desta tela é servir como ponto de entrada para uma nova pesquisa. O agente deve primeiro identificar o cidadão a ser entrevistado por meio do seu CPF.

#### **Componentes e Layout**

- **Título**: Exibe "PESQUISA SOCIOECÔNOMICA".
- **Informações do Usuário**: Mostra o nome completo (`User().FullName`) e a foto de perfil (`User().Email`) do agente logado no aplicativo, obtidos do Office 365.
- **Campo de Busca de CPF**:
  - Um campo de texto (`inpCPF_Busca_inicial_2`) para inserir o CPF do cidadão.
  - Uma máscara (`inpCPF_Busca_Mascara_2`) formata o número digitado para o padrão `000.000.000-00`.
- **Botão "BUSCAR CPF"**: O botão que inicia a navegação para a tela de pesquisa.

#### **Lógica de Funcionamento**

A lógica principal está no evento `OnSelect` do botão **"BUSCAR CPF"**:

```powerapps
NewForm(Form3);
Set(varCPF, inpCPF_Busca_Mascara_2.Text);
Navigate('Pesquisa Screen', Transition.Push);
```

1.  **`NewForm(Form3)`**: Prepara o formulário (`Form3`) na próxima tela para a criação de um novo registro.
2.  **`Set(varCPF, ...)`**: Armazena o CPF digitado em uma variável global chamada `varCPF`. Esta variável será usada para pré-preencher o campo CPF no formulário de pesquisa.
3.  **`Navigate('Pesquisa Screen', ...)`**: Redireciona o usuário para a tela `'Pesquisa Screen'`, onde o formulário de pesquisa será preenchido.

### 3.2. `Pesquisa Screen` (Tela do Formulário)

Esta tela contém o formulário principal para a coleta de dados socioeconômicos.

#### **Objetivo**

Coletar e armazenar as informações detalhadas da pesquisa para o CPF informado na tela anterior.

#### **Componentes e Layout**

- **Estrutura Similar**: Mantém o mesmo cabeçalho com o título e as informações do agente.
- **Formulário (`Form3`)**:
  - **Fonte de Dados**: O formulário está conectado à fonte de dados `dbPesquisaSocioEconomica`, que é provavelmente uma lista do SharePoint ou uma tabela do Dataverse.
  - **Modo Padrão**: `FormMode.New`, indicando que ele sempre abrirá para criar um novo registro.
  - **Campos (DataCards)**: O formulário é composto por dezenas de campos que correspondem às colunas da fonte de dados. Alguns exemplos de campos são:
    - `Nucleo` (Empreendimento)
    - `Nome_Completo`
    - `CPF` (pré-preenchido com a variável `varCPF`)
    - `Data_Nascimento`
    - `Endereco_Atual`
    - Perguntas sobre identidade étnico-racial, gênero, composição familiar, escolaridade, renda, etc.

#### **Lógica de Funcionamento**

- **Pré-preenchimento do CPF**: O campo de CPF (`DataCardValue30`) tem seu valor padrão definido como `varCPF`, garantindo que o CPF da tela anterior seja automaticamente inserido.
- **Lógica Condicional**: Vários campos são visíveis ou obrigatórios com base em respostas anteriores. Por exemplo:
  - As perguntas sobre o número de crianças em cada faixa etária (`'Quantas crianças de 0 a 3 anos?'`) só se tornam visíveis e obrigatórias se a resposta para `'Tem crianças morando com você'` for "Sim".
  - `Visible: =If(DataCardValue3.Selected.Value = "Sim", true, false)`
  - `Required: =If(DataCardValue3.Selected.Value = "Sim", true, false)`
- **Submissão do Formulário**: O formulário possui um botão de submissão (não totalmente visível no trecho, mas implícito pela estrutura do `Form`). Ao ser submetido com sucesso, o evento `OnSuccess` do formulário é acionado:

  ```powerapps
  Navigate('Contratos Screen', Transition.Push);
  Set(varCPF, Blank());
  Reset(inpCPF_Busca_inicial_2)
  ```

  1.  **`Navigate('Contratos Screen', ...)`**: Após salvar os dados, o aplicativo retorna para a tela inicial de busca.
  2.  **`Set(varCPF, Blank())`**: Limpa a variável `varCPF`.
  3.  **`Reset(inpCPF_Busca_inicial_2)`**: Limpa o campo de busca de CPF na tela inicial, deixando o aplicativo pronto para uma nova pesquisa.

