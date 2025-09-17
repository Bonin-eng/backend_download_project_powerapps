'''
# Documentação do Aplicativo: Controle de Arquivos COSAN

## 1. Visão Geral do Projeto

O aplicativo "Controle de Arquivos COSAN" é uma ferramenta desenvolvida em Power Apps para gerenciar o fluxo de análise de arquivos entre diferentes perfis de usuários (Gestão, Analistas, Projetistas e Clientes). A principal função do aplicativo é permitir o envio, a análise e a devolução de arquivos, mantendo um registro detalhado de cada etapa do processo.

O sistema controla o acesso e a visibilidade das funcionalidades com base no perfil do usuário logado, garantindo que cada um tenha acesso apenas às telas e ações pertinentes à sua função.

## 2. Perfis de Usuário e Permissões

O acesso às funcionalidades do aplicativo é determinado pelo perfil do usuário, que é verificado no início da sessão (`App.OnStart`). Os perfis são obtidos das listas `Controle_de_Analistas` e `Controle_de_Projetistas`.

- **Gestão**: Possui acesso total a todas as telas e funcionalidades do aplicativo, incluindo:
  - `Controle de Arquivos`: Tela principal para visualização e cadastro de todos os arquivos.
  - `Controle de Analistas`: Gerenciamento (cadastro, edição e exclusão) dos usuários analistas.
  - `Controle de Projetistas`: Gerenciamento dos usuários projetistas.
  - `Arquivos para Análise`: Visualização dos arquivos pendentes de análise.
  - `Arquivos Analisados`: Visualização dos arquivos que já foram analisados.

- **Analista**: Perfil focado na análise dos arquivos. Possui acesso a:
  - `Arquivos para Análise`: Tela para visualizar os arquivos que precisam ser analisados.

- **Projetista**: Perfil focado no envio e recebimento de arquivos analisados. Possui acesso a:
  - `Arquivos Analisados`: Tela para visualizar o resultado e baixar os arquivos após a análise.

- **Cliente**: Perfil com visibilidade sobre o processo. Possui acesso a:
  - `Arquivos para Análise`: Visualização dos arquivos pendentes de análise.
  - `Arquivos Analisados`: Visualização dos arquivos que já foram analisados.

## 3. Estrutura de Navegação

A navegação é controlada por um menu lateral (`cmpMenuLateral`) que exibe as telas disponíveis para o usuário com base no seu perfil, definido na coleção `colTelas` durante a inicialização do aplicativo.

1.  **Tela Inicial (`Controle de Arquivos_Inicial`)**: É a primeira tela carregada, servindo como ponto de entrada que direciona o usuário para as funcionalidades permitidas.
2.  **Menu Lateral**: A partir da tela inicial, o usuário utiliza o menu lateral para navegar entre as diferentes seções do aplicativo.

## 4. Descrição das Telas

### 4.1. `Controle de Arquivos_Inicial`

- **Propósito**: Servir como tela de boas-vindas e ponto de partida para a navegação.
- **Lógica**: No `OnVisible`, a variável `VarVisibilidadeInicial` é definida como `true`. Esta tela contém o componente de menu lateral (`cmpMenuLateral_6`) que permite ao usuário navegar para outras telas com base em suas permissões.

### 4.2. `Controle de Arquivos`

- **Propósito**: Tela principal para a equipe de **Gestão**. Permite o cadastro de novos arquivos e a visualização de todos os registros.
- **Componentes Principais**:
  - **`btn "Novo Arquivo"`**: Abre um formulário para o cadastro de um novo arquivo.
  - **`frm_arquivos` e `frm_arquivos_entrada_anexos`**: Formulários para inserir os dados do novo arquivo (Projetista, Disciplina, Assunto, Revisão, etc.) e anexar o documento a ser analisado. Os dados são salvos nas listas `Controle_Entrada_Arquivos` e `Controle_Entrada_Anexos`.
  - **`gal_Arquivos`**: Galeria que exibe todos os arquivos registrados, com informações detalhadas e filtros.
  - **Filtros**: Permitem filtrar a galeria por nome do arquivo, etapa, disciplina, analista, projetista e status.
  - **`frm_arquivos_conclusao`**: Formulário para registrar a conclusão de uma análise, inserindo a data de saída, horas de análise e o status final.
- **Lógica de Negócio**:
  - Ao cadastrar um novo arquivo, o status padrão é definido como "Em análise".
  - A revisão (`Revisao`) é calculada automaticamente, incrementando a última revisão existente para o mesmo arquivo.
  - A tela permite o envio de e-mails para analistas e projetistas.

### 4.3. `Controle de Analistas`

- **Propósito**: Tela de gerenciamento de usuários com perfil de **Analista**, acessível apenas pela **Gestão**.
- **Componentes Principais**:
  - **`frm_Analistas`**: Formulário para cadastrar ou editar informações de um analista (Nome, E-mail, Telefone, Disciplina e Perfil).
  - **`gal_Analistas`**: Galeria que exibe a lista de todos os analistas cadastrados.
  - **Filtros**: Permitem filtrar a lista de analistas por nome, disciplina e perfil.
  - **Ícones de Ação**: Permitem editar ou excluir um analista (com um pop-up de confirmação para exclusão).
- **Lógica de Negócio**:
  - Os dados são salvos na lista `Controle_de_Analistas`.
  - A edição de um registro é feita selecionando um item na galeria, que preenche o formulário com os dados existentes.

### 4.4. `Controle de Projetistas`

- **Propósito**: Tela de gerenciamento de usuários com perfil de **Projetista**, acessível apenas pela **Gestão**.
- **Lógica e Componentes**: A estrutura é praticamente idêntica à da tela `Controle de Analistas`, mas os dados são lidos e gravados na lista `Controle_de_Projetistas`.

### 4.5. `Controle de Arquivos_Analise`

- **Propósito**: Tela destinada aos **Analistas** (e Clientes/Gestão) para visualizar os arquivos que estão com o status "Em análise" ou pendentes.
- **Componentes Principais**:
  - **`gal_Arquivos_1`**: Galeria que exibe os arquivos que precisam ser analisados.
  - **Filtros**: Permitem refinar a busca por etapa, disciplina, analista, etc.
  - **Botão "Baixar"**: Dentro da galeria, um botão permite que o usuário baixe o anexo original (`Controle_Entrada_Anexos`) para realizar a análise.
- **Lógica de Negócio**:
  - A galeria é filtrada para exibir, por padrão, apenas os itens com `Status.Value = "Em análise"`.
  - A visualização é focada em fornecer ao analista as informações e o arquivo necessários para o seu trabalho.

### 4.6. `Controle de Arquivos_Analisados`

- **Propósito**: Tela destinada aos **Projetistas** (e Clientes/Gestão) para visualizar os arquivos cujo processo de análise foi concluído.
- **Componentes Principais**:
  - **`gal_Arquivos_2`**: Galeria que exibe os arquivos com status diferente de "Em análise" (ex: "Analisado", "Aprovado com comentários").
  - **Filtros**: Permitem refinar a busca.
  - **Botão "Baixar"**: Permite baixar o arquivo de saída (`Controle_Saida_Anexos`) que contém o resultado da análise.
- **Lógica de Negócio**:
  - A galeria é filtrada para não exibir arquivos "Em análise".
  - O foco é permitir que o projetista ou cliente acesse o resultado final do processo de análise.

## 5. Tema e Estilo Visual (`Themes.json`)

O aplicativo utiliza um tema customizado chamado `defaultTheme`. A paleta de cores principal contribui para a identidade visual da COSAN, com destaque para:

- **Cor Primária 1**: `RGBA(56, 96, 178, 1)` (Azul)
- **Cor Primária 2**: `RGBA(0, 18, 107, 1)` (Azul Escuro)
- **Cor de Fundo**: `RGBA(255, 255, 255, 1)` (Branco)

As fontes, tamanhos e estilos dos componentes (botões, caixas de texto, etc.) são padronizados neste arquivo, garantindo uma experiência de usuário coesa e profissional em todo o aplicativo.
'''