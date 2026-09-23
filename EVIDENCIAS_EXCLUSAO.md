# Evidências - Diálogo de Confirmação de Exclusão de Tarefas
---

## Sequência de Evidências

### 1. Lista de Tarefas Antes da Exclusão
Exibição da lista de tarefas cadastradas na aplicação antes de solicitar qualquer exclusão.

![img.png](img.png)
---

### 2. Diálogo de Confirmação Aberto com a Tarefa Selecionada
Ao tocar no ícone de lixeira da tarefa *"Fazer checkpoint de kotlin"*, o diálogo de confirmação é exibido em camada sobre a lista, apresentando o título da tarefa e informando claramente que ela será excluída.

![img_3.png](img_3.png)
---

### 3. Resultado após Ação de Cancelar
Ao selecionar a opção **Cancelar**, o diálogo é fechado sem realizar nenhuma alteração no banco de dados e a tarefa *"Fazer checkpoint de kotlin"* permanece intacta na lista.

![img_4.png](img_4.png)
---

### 4. Nova Abertura do Diálogo de Confirmação
O usuário toca novamente no ícone de lixeira da tarefa *"Estudar"*, reabrindo o diálogo de confirmação.

![img_5.png](img_5.png)
---

### 5. Resultado após Confirmar a Exclusão
Ao selecionar a opção **Excluir**, a tarefa *"Estudar"* é removida do banco de dados Room e a lista é atualizada automaticamente por meio do `StateFlow`, mantendo apenas as demais tarefas.

![img_6.png](img_6.png)
---

