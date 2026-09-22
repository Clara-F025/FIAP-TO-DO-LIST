# Evidências - Diálogo de Confirmação de Exclusão de Tarefas

Este documento registra as evidências de funcionamento do fluxo seguro de exclusão de tarefas no aplicativo **To-Do List (FIAP)**.

A implementação garante que, ao tocar no ícone de lixeira de uma tarefa, seja exibido um diálogo de confirmação (`AlertDialog` do Jetpack Compose Material 3) contendo o título da tarefa selecionada. O usuário pode optar por **Cancelar** (mantendo a tarefa na lista) ou **Excluir** (removendo a tarefa definitivamente).

---

## Sequência de Evidências

### 1. Lista de Tarefas Antes da Exclusão
Exibição da lista de tarefas cadastradas na aplicação antes de solicitar qualquer exclusão.

![1. Lista antes da exclusão](docs/images/exclusao/1_lista_antes.png)

---

### 2. Diálogo de Confirmação Aberto com a Tarefa Selecionada
Ao tocar no ícone de lixeira da tarefa *"Estudar Room"*, o diálogo de confirmação é exibido em camada sobre a lista (`AlertDialog`), apresentando o título da tarefa e informando claramente que ela será excluída.

![2. Diálogo aberto](docs/images/exclusao/2_dialogo_aberto.png)

---

### 3. Resultado após Ação de Cancelar
Ao selecionar a opção **Cancelar**, o diálogo é fechado sem realizar nenhuma alteração no banco de dados e a tarefa *"Estudar Room"* permanece intacta na lista.

![3. Resultado ao cancelar](docs/images/exclusao/3_resultado_cancelar.png)

---

### 4. Nova Abertura do Diálogo de Confirmação
O usuário toca novamente no ícone de lixeira da tarefa *"Estudar Room"*, reabrindo o diálogo de confirmação.

![4. Nova abertura do diálogo](docs/images/exclusao/4_nova_abertura_dialogo.png)

---

### 5. Resultado após Confirmar a Exclusão
Ao selecionar a opção **Excluir**, a tarefa *"Estudar Room"* é removida do banco de dados Room e a lista é atualizada automaticamente por meio do `StateFlow`, mantendo apenas as demais tarefas.

![5. Resultado após confirmar exclusão](docs/images/exclusao/5_resultado_confirmar.png)

---

## Resumo Técnico da Implementação

1. **Jetpack Compose Material 3:** Utilização do componente `AlertDialog` com título (`title`), mensagem contextualizada (`text`), e botões de ação `TextButton` para **Cancelar** e **Excluir**.
2. **Componente Stateless / State Management:** O estado do diálogo (`tarefaParaExcluir`) é controlado reativamente pela tela, garantindo que apenas a tarefa selecionada seja passada para a deleção.
3. **Preservação de Funcionalidades:** Manteve-se intacta toda a funcionalidade existente de cadastro, edição, prazos/atrasos, ordenação e MVVM com Room e Coroutines.
4. **Previews:** Foram adicionados novos `@Preview` no `ListaTarefasScreen.kt` demonstrando isoladamente e integradamente o estado do diálogo de confirmação de exclusão.
