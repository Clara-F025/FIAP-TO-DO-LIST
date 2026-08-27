# To-Do List (Android)

## Descrição do projeto e objetivo da aplicação

Aplicativo Android de lista de tarefas desenvolvido em **Kotlin** com **Jetpack Compose**. O objetivo da aplicação é permitir que o usuário cadastre, visualize, edite, marque como concluída e exclua tarefas.

Os dados são persistidos localmente no dispositivo utilizando o **Room**. O projeto segue uma arquitetura em camadas, separando as responsabilidades entre persistência de dados, repositório, gerenciamento de estado e interface do usuário. Essa organização facilita a manutenção, os testes e a evolução da aplicação.

## Tecnologias utilizadas

- **Kotlin:** linguagem principal utilizada no desenvolvimento do aplicativo.
- **Jetpack Compose:** framework utilizado para construção declarativa das interfaces, incluindo a tela de listagem e o formulário de tarefas.
- **Room:** biblioteca de persistência que abstrai o acesso ao banco de dados SQLite. No projeto, é utilizada por meio da entidade `Tarefa`, do `TarefaDao` e do `TarefaDatabase`.
- **Coroutines/Flow:** utilizados para executar operações de banco de dados de forma assíncrona e observar reativamente as alterações na lista de tarefas.
- **ViewModel:** responsável por manter o estado da tela e disponibilizá-lo para a interface, sobrevivendo a mudanças de configuração, como rotações de tela.
- **Navigation Compose:** responsável pela navegação entre a tela de lista e a tela de formulário, incluindo a passagem do ID da tarefa durante a edição.

## Responsabilidade de TarefaRepository

O `TarefaRepository` funciona como uma camada intermediária entre o `TarefaViewModel` e o `TarefaDao`.

Ele disponibiliza a lista de tarefas como um `Flow<List<Tarefa>>`, obtido a partir do DAO, e fornece funções de suspensão para inserir, atualizar e excluir tarefas. Essas operações são encaminhadas ao `TarefaDao`.

A principal responsabilidade do repositório é **isolar o restante da aplicação dos detalhes de acesso ao banco de dados**. Dessa forma, o `TarefaViewModel` não precisa conhecer diretamente o Room ou o DAO, comunicando-se somente com o repositório.

Essa separação facilita a manutenção e a realização de testes, além de permitir que a fonte de dados seja substituída futuramente sem grandes alterações no ViewModel ou na interface.

## Responsabilidade de TarefaViewModel

O `TarefaViewModel` é responsável por manter e expor o estado da tela de tarefas.

A lista fornecida pelo repositório é transformada em um `StateFlow` utilizando `stateIn`, com `SharingStarted.WhileSubscribed(5_000)`. Dessa forma, a interface consegue observar o estado de maneira eficiente enquanto estiver coletando os dados.

O ViewModel também disponibiliza as operações de:

- Inserir uma tarefa;
- Atualizar uma tarefa;
- Excluir uma tarefa.

Cada operação é executada dentro de uma coroutine utilizando `viewModelScope`, delegando a responsabilidade de acesso aos dados para o `TarefaRepository`.

O `TarefaViewModel` também possui uma factory responsável por criar suas dependências. Essa factory obtém o `TarefaDatabase`, recupera o DAO, cria o `TarefaRepository` e, por fim, instancia o próprio ViewModel. Assim, o Android consegue construir o ViewModel corretamente sem a necessidade de um framework de injeção de dependências.

## Como ListaTarefasScreen observa o estado e dispara ações

A `ListaTarefasScreen` observa a lista de tarefas do `TarefaViewModel` utilizando `collectAsStateWithLifecycle()`. Essa abordagem garante que a coleta respeite o ciclo de vida da tela, evitando que a interface continue coletando dados quando não estiver ativa.

O estado coletado é passado para `ListaTarefasContent`, que funciona como um componente stateless: recebe os dados necessários e callbacks para executar as ações do usuário. Essa separação também facilita a criação de previews da interface.

As principais ações realizadas pela tela são:

- **Marcar ou desmarcar uma tarefa:** o callback `onCheckedChange` cria uma cópia da tarefa utilizando `copy(concluida = ...)` e chama `viewModel.atualizar()`.
- **Excluir uma tarefa:** o callback `onDeletar` chama `viewModel.deletar(tarefa)`.
- **Editar uma tarefa:** ao tocar em um item da lista, o callback `onEditarTarefa` solicita a navegação para o formulário passando o ID da tarefa.
- **Criar uma nova tarefa:** o botão flutuante (FAB) chama `onNovaTarefa`, levando o usuário ao formulário sem uma tarefa existente.

Como a lista é observada por meio de um `StateFlow`, qualquer alteração realizada no banco de dados é propagada automaticamente para a UI. Com isso, a tela é recomposta sem a necessidade de atualizar manualmente a lista.

## Como FormularioTarefaScreen diferencia cadastro e edição

A `FormularioTarefaScreen` recebe um parâmetro chamado `tarefaId`, obtido a partir da navegação.

A regra utilizada é:

- `tarefaId == 0`: o formulário está no modo de **cadastro** de uma nova tarefa.
- `tarefaId != 0`: o formulário está no modo de **edição** de uma tarefa existente.

Quando o formulário está em modo de edição, a tela procura a tarefa correspondente dentro da lista atual:

```kotlin
tarefas.find { it.id == tarefaId }
```

Os dados encontrados são utilizados para preencher previamente os campos de título e descrição.

O `FormularioTarefaContent` recebe uma flag `isEdicao`, utilizada para alterar elementos visuais, como o título da barra superior entre **"Nova Tarefa"** e **"Editar Tarefa"**.

Ao salvar:

- No cadastro, uma nova instância de `Tarefa` é criada e enviada para `viewModel.inserir()`.
- Na edição, a tarefa existente é atualizada utilizando `copy(titulo = ..., descricao = ...)`, preservando o ID e os demais dados, como o estado de conclusão e a data de criação.

Após salvar ou cancelar, a tela retorna para a lista.

## Rotas configuradas em AppNavigation e passagem do ID da tarefa

O `AppNavigation` utiliza um `NavHost` para controlar a navegação entre as telas. O destino inicial é a tela de lista.

São utilizadas duas rotas principais.

### Rota `lista`

Responsável por exibir a `ListaTarefasScreen`.

Quando o usuário solicita uma nova tarefa, a aplicação navega para:

```text
formulario/0
```

Quando o usuário seleciona uma tarefa para edição, o ID correspondente é inserido na rota:

```text
formulario/{id}
```

### Rota `formulario/{tarefaId}`

Responsável por exibir a `FormularioTarefaScreen`.

O `tarefaId` é declarado como um argumento da rota e recuperado por meio de `backStackEntry.arguments`. Como os argumentos de navegação são recebidos como `String`, o valor é convertido para `Int`, utilizando `0` como valor padrão.

Esse ID é então repassado para o formulário, que utiliza seu valor para determinar se deve apresentar o modo de cadastro ou de edição.

A navegação de volta é realizada com:

```kotlin
navController.popBackStack()
```

Assim, tanto o cancelamento quanto o salvamento retornam o usuário à tela de lista.

## Como MainActivity cria a ViewModel e inicia a navegação

Na `MainActivity`, dentro de `setContent`, o tema `FiaptodolistTheme` envolve o conteúdo principal da aplicação.

A instância do `TarefaViewModel` é criada utilizando a função `viewModel()` do Compose e a factory definida no próprio ViewModel:

```kotlin
TarefaViewModel.factory(applicationContext)
```

A factory é responsável por:

1. Obter a instância do `TarefaDatabase`;
2. Obter o `TarefaDao`;
3. Criar o `TarefaRepository`;
4. Criar o `TarefaViewModel`.

Depois que o ViewModel é criado, ele é compartilhado com o `AppNavigation`:

```kotlin
AppNavigation(viewModel = viewModel)
```

Dessa forma, as telas de lista e formulário utilizam a mesma instância do ViewModel, mantendo o estado e as operações centralizados.

## Instruções básicas para executar o projeto

### Pré-requisitos

- Android Studio em versão compatível com **AGP 9.2.1** e **Kotlin 2.2.10**.
- Dispositivo Android físico com depuração USB habilitada ou emulador Android.
- API mínima **24**.
- API alvo **37**.

### Executando o projeto

1. Abra o projeto no Android Studio.
2. Aguarde a sincronização automática do Gradle.
3. Conecte um dispositivo físico ou inicie um emulador Android.
4. Selecione o módulo `app`.
5. Clique em **Run** para compilar e instalar a aplicação.

Também é possível instalar a versão de debug pelo terminal, na raiz do projeto:

```bash
./gradlew installDebug
```

No Windows:

```bash
gradlew.bat installDebug
```

Após iniciar o aplicativo, a tela principal exibirá a lista de tarefas. O usuário poderá utilizar o botão **+** para criar uma tarefa, tocar em uma tarefa para editá-la, utilizar a caixa de seleção para alterar seu estado de conclusão e utilizar o botão de lixeira para excluí-la.


```bash
./gradlew connectedAndroidTest
```

Os testes instrumentados exigem um emulador ou dispositivo Android conectado. Entre eles está o `TarefaDaoTest`, responsável por testar o acesso aos dados por meio do DAO.



## Evidências do projeto

As evidências de funcionamento do projeto estão disponíveis no arquivo:

`docs/evidencias - Evidências do Projeto.pdf`

O documento contém registros que demonstram o funcionamento da aplicação e das principais funcionalidades implementadas.

