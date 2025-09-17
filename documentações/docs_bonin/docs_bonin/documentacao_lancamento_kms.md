# Documentação do Aplicativo: Lançamento de KM

## 1. Visão Geral do Aplicativo

O aplicativo de **Lançamento de KM** é uma ferramenta móvel desenvolvida para que os colaboradores da Bonin possam registrar de forma simples e eficiente as viagens realizadas com veículos da empresa ou próprios a serviço. O aplicativo centraliza os lançamentos de quilometragem, dados do condutor e contatos úteis.

As principais funcionalidades são:
- **Autenticação de Usuário:** O aplicativo valida se o usuário logado está cadastrado na base de colaboradores da empresa, garantindo que apenas pessoas autorizadas tenham acesso.
- **Dashboard Pessoal:** A tela inicial apresenta um resumo dos lançamentos de KM do próprio usuário, com a possibilidade de filtrar por mês.
- **Lançamento de Viagens:** Um formulário completo permite registrar todos os detalhes de uma viagem, incluindo motivo, datas, locais de saída/chegada, e os registros do odômetro (inicial e final) com fotos comprobatórias.
- **Edição de Lançamentos:** O usuário pode selecionar e editar um lançamento de KM já realizado.
- **Perfil do Usuário:** Uma tela dedicada permite que o colaborador visualize e atualize suas informações pessoais, como telefone, CNH e endereço.
- **Lista de Contatos:** Fornece uma lista de contatos de departamentos importantes (como Facilities e Financeiro) com atalhos para iniciar uma chamada ou uma conversa no Microsoft Teams.

## 2. Estrutura e Lógica Inicial

A configuração inicial, validação de acesso e navegação são definidas no arquivo `App.fx.yaml`.

### Lógica de Inicialização (`OnStart`)

1.  **Validação de Acesso:**
    -   O aplicativo busca o registro do usuário logado na lista `sigd_colaboradores` usando o e-mail (`LookUp(sigd_colaboradores, email = User().Email)`).
    -   Uma variável `varAcessoNegado` é definida como `true` se o usuário não for encontrado. Esta variável é usada na tela de carregamento para bloquear o acesso.
2.  **Carregamento de Dados do Usuário:** As informações do colaborador encontrado são armazenadas na variável `varUsuario` para serem usadas em todo o aplicativo.
3.  **Menu de Navegação:** Uma coleção `colTelas` é criada para alimentar o menu de navegação lateral, contendo as telas: Home, Lançamento, Contatos e Perfil.

### Tela de Carregamento (`Screen_nao_utilizar.fx.yaml`)

Apesar do nome, esta tela funciona como a tela de *splash* ou carregamento inicial.
-   Um temporizador é iniciado e, após 2 segundos, redireciona o usuário para a `Home Screen`.
-   Se a variável `varAcessoNegado` for verdadeira, o temporizador é pausado e uma mensagem de "ACESSO NEGADO" é exibida, impedindo o uso do aplicativo.

## 3. Telas do Aplicativo

### 3.1. Tela Inicial (`Home Screen`)

É o painel principal do usuário após o login.

**Objetivo:** Apresentar um resumo dos lançamentos de KM e servir como ponto de partida para as outras funcionalidades.

**Componentes e Lógica:**
-   **`OnVisible`:** Carrega para a coleção `colLancamento` todos os registros da lista `sigd_frota_lancamentos` que pertencem ao usuário logado. Também cria uma coleção `colMesAno` com os meses/anos distintos para popular o filtro.
-   **Perfil do Usuário:** Exibe a foto e o nome completo do usuário.
-   **Filtro de Mês:** Um menu suspenso (`ComboBox`) permite ao usuário filtrar os lançamentos exibidos na galeria por mês/ano.
-   **Galeria de Lançamentos (`Gallery3_3`):** Lista as viagens do usuário. Ao selecionar um item, o usuário é levado para a `Lançamento KM Screen` em modo de edição.
-   **Botão "NOVO LANÇAMENTO":** Navega para a `Lançamento KM Screen` em modo de criação para registrar uma nova viagem.

### 3.2. Tela de Lançamento de KM (`Lançamento KM Screen`)

Esta tela possui uma dupla função: criar novos lançamentos ou editar existentes.

**Objetivo:** Capturar de forma detalhada as informações de uma viagem.

**Componentes e Lógica:**
-   **Controle de Modo (Novo vs. Edição):** A visibilidade dos contêineres `cLançar` (novo) e `cEditar` é controlada pela variável `varEditarViagem`, definida na `Home Screen`.
-   **Formulário (`Form_lancamento_viagem`):
    -   **Fonte de Dados:** `sigd_frota_lancamentos`.
    -   **Campos do Formulário:**
        -   Motivo da Viagem
        -   Data da Viagem
        -   Placa do Veículo
        -   Centro de Custo
        -   Local de Saída e Chegada
        -   Odômetro Inicial e Final (campos numéricos)
        -   Imagem do Odômetro Inicial e Final (campos de imagem obrigatórios)
        -   Observação
    -   **Captura de Localização:** Os campos `latitude_saida` e `longitude_saida` são preenchidos automaticamente com as coordenadas do GPS do dispositivo no momento do lançamento.
-   **Ações:**
    -   **`CONCLUIR ✅`:** Salva os dados do formulário na lista do SharePoint (`SubmitForm`).
    -   **`↩️ VOLTAR`:** Retorna à tela inicial sem salvar as alterações.

### 3.3. Tela de Perfil (`Perfil Screen`)

Permite ao usuário gerenciar suas próprias informações cadastrais.

**Objetivo:** Manter os dados do colaborador atualizados na base de dados da empresa.

**Componentes e Lógica:**
-   **Formulário (`frm_novos_colaboradores`):
    -   **Fonte de Dados:** `sigd_colaboradores`.
    -   **Item:** O formulário é preenchido com os dados da variável `varUsuario`, que contém o registro do usuário logado.
    -   **Campos Editáveis:** Nome, Telefone, CPF, RG, Data de Nascimento, CNH, Validade da CNH e Endereço.
-   **Ações:**
    -   **`CONCLUIR ✅`:** Salva as alterações no registro do usuário na lista `sigd_colaboradores`.
    -   **`↩️ VOLTAR`:** Retorna à tela inicial.

### 3.4. Tela de Contatos (`Contato Screen`)

Funciona como uma pequena agenda de contatos internos.

**Objetivo:** Facilitar a comunicação do usuário com departamentos-chave da empresa.

**Componentes e Lógica:**
-   **`OnVisible`:** Carrega uma coleção `colDepartamentos` com os departamentos de Facilities e Financeiro.
-   **Filtro por Departamento:** Um menu suspenso permite filtrar a lista de contatos.
-   **Galeria de Contatos:** Uma galeria aninhada exibe os departamentos e, dentro de cada um, os colaboradores correspondentes da lista `sigd_colaboradores`.
-   **Ações Rápidas:** Para cada contato, há dois ícones:
    -   **Microsoft Teams:** Inicia uma conversa no Teams com o contato (`Launch("msteams://...")`).
    -   **Telefone:** Inicia uma chamada telefônica para o número do contato (`Launch("tel:...")`).

## 4. Fontes de Dados

-   **`sigd_frota_lancamentos`**: Lista principal que armazena todos os registros de viagens e quilometragem.
-   **`sigd_colaboradores`**: Lista mestra com os dados de todos os colaboradores, usada para autenticação e para preenchimento de dados de perfil e contatos.
-   **`sigd_departamentos`**: Lista com os departamentos da empresa, usada para organizar os contatos.
-   **`sigd_contratos`**: Contém os centros de custo que podem ser associados a uma viagem.
-   **`UsuáriosdoOffice365`**: Conector do Office 365 usado para obter a foto de perfil do usuário logado.
