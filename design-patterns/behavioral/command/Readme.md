# Command

## 🎯 Intenção

O Command é um padrão de projeto comportamental que transforma uma requisição em um objeto autônomo que contém toda a informação sobre a requisição. Esta transformação permite parametrizar métodos com diferentes requisições, atrasar ou enfileirar a execução de requisições e suportar operações que podem ser desfeitas.

## 🚩 Problema

Imagine que você está trabalhando em um novo aplicativo de editor de texto. Sua tarefa atual é criar uma barra de ferramentas com vários botões para diferentes operações do editor. Você criou uma classe `Button` muito elegante que pode ser usada tanto para botões da barra quanto para botões genéricos em diálogos.

### Resultado problemático:
```java
// Solução ingênua: subclasses para cada botão
class CopyButton extends Button {
    public void onClick() {
        // Lógica de copiar diretamente no botão
        editor.copy();
    }
}

class PasteButton extends Button {
    public void onClick() {
        // Lógica de colar diretamente no botão
        editor.paste();
    }
}

class SaveButton extends Button {
    public void onClick() {
        // Lógica de salvar diretamente no botão
        editor.save();
    }
}

// Problema: mesma operação em múltiplos lugares
// - Botão na toolbar
// - Item no menu de contexto
// - Atalho de teclado (Ctrl+C, Ctrl+V, Ctrl+S)
// Como reutilizar o código?
```

**Problemas:**
- **Explosão de subclasses**: Uma subclasse para cada operação/local
- **Duplicação de código**: Mesma lógica em múltiplos lugares
- **Alto acoplamento**: GUI acoplada à lógica de negócio
- **Difícil manutenção**: Mudanças afetam muitas classes
- **Impossível parametrizar**: Não dá para trocar operações dinamicamente
- **Sem histórico**: Não há como implementar undo/redo

## ✅ Solução

O padrão Command sugere que objetos GUI não devem enviar requisições diretamente. Em vez disso, você deve extrair todos os detalhes da requisição (objeto chamado, nome do método, argumentos) em uma **classe command** separada com um único método que dispara esta requisição.

### Características-chave:
- **Encapsulamento**: Requisição é um objeto independente
- **Parametrização**: Objetos configurados com operações
- **Enfileiramento**: Comandos podem ser agendados
- **Histórico**: Mantém pilha de comandos executados
- **Undo/Redo**: Suporte para operações reversíveis
- **Desacoplamento**: GUI separada da lógica de negócio

```java
// Interface Command
interface Command {
    void execute();
    void undo();
}

// Command concreto
class CopyCommand implements Command {
    private Editor editor;
    private String backup;
    
    public CopyCommand(Editor editor) {
        this.editor = editor;
    }
    
    public void execute() {
        backup = editor.getSelection();
        clipboard.setText(backup);
    }
    
    public void undo() {
        // Copy não precisa desfazer
    }
}

// Botão genérico que executa qualquer comando
class Button {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void click() {
        command.execute();
    }
}

// Cliente configura botões com comandos
Button copyButton = new Button();
copyButton.setCommand(new CopyCommand(editor));
```

## 🏗️ Estrutura

```
Client → Invoker → Command (interface)
            ↓           ↓
         [command]  ConcreteCommand → Receiver
                         ↓               ↓
                    [receiver]     [business logic]
```

### Componentes:
- **Command**: Interface que declara método de execução
- **ConcreteCommand**: Implementa comando e armazena parâmetros
- **Receiver**: Contém lógica de negócio real
- **Invoker**: Armazena e dispara comandos
- **Client**: Cria e configura comandos concretos

## 🎯 Quando Usar?

### ✅ Use quando:
- **Parametrizar objetos**: Com operações a serem executadas
- **Enfileirar operações**: Agendar execução futura
- **Operações remotas**: Enviar comandos pela rede
- **Histórico de operações**: Implementar log de transações
- **Undo/Redo**: Suportar operações reversíveis
- **Transações**: Operações que podem ser canceladas

### 📝 Exemplos de aplicação:
- **Editores**: Undo/redo de operações
- **Sistemas de filas**: Job queues, task schedulers
- **Transações**: Database transactions, CQRS
- **Macro commands**: Sequência de operações
- **GUI frameworks**: Botões, menus, atalhos

### ❌ Evite quando:
- **Operação simples**: Overhead desnecessário
- **Sem necessidade de histórico**: Execução direta é suficiente
- **Performance crítica**: Overhead de objetos adicionais

## 🚀 Como Implementar

1. **Declare interface Command** com método `execute()`

2. **Extraia requisições** para classes command concretas

3. **Identifique invokers** (senders) que dispararão comandos

4. **Modifique senders** para executar comando ao invés de requisição direta

5. **Cliente inicializa objetos**:
   - Cria receivers
   - Cria commands (associa com receivers)
   - Associa commands com senders

6. **Implemente undo** se necessário:
   - Adicione método `undo()` na interface
   - Salve estado anterior em cada comando

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Single Responsibility**: Desacopla invocação de execução
- **Open/Closed**: Novos comandos sem alterar código existente
- **Undo/Redo**: Implementa reversão de operações
- **Execução diferida**: Agenda ou enfileira operações
- **Composição**: Monta comandos complexos de simples
- **Histórico**: Mantém log de operações

### ❌ Desvantagens:
- **Complexidade**: Nova camada entre sender e receiver
- **Mais classes**: Um command para cada operação
- **Overhead**: Objetos adicionais em memória

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Relação | Estado |
|--------|------|---------|--------|
| **Command** | Encapsular requisição | Sender → Receiver unidirecional | Mantém parâmetros |
| **Strategy** | Algoritmos intercambiáveis | Contexto usa estratégia | Stateless geralmente |
| **Chain of Responsibility** | Passar por cadeia | Múltiplos handlers | Não mantém estado |
| **Observer** | Notificar múltiplos | Broadcast para observadores | Event-driven |

## 🔗 Relações com Outros Padrões

- **Chain of Responsibility**: Handlers podem ser implementados como Commands
- **Memento**: Usado junto com Command para implementar undo
- **Prototype**: Pode salvar cópias de Commands no histórico
- **Strategy**: Command transforma operação em objeto, Strategy algorítimos
- **Visitor**: Command executável sobre vários objetos

## 📚 Conceitos-Chave para Lembrar

1. **Requisição como objeto**: Encapsula tudo sobre a requisição
2. **Desacoplamento**: Sender não conhece receiver
3. **Parametrização**: Configura objetos com operações
4. **Undo/Redo**: Suporte natural para reversão
5. **Enfileiramento**: Permite agendar execução
6. **Macro commands**: Combina múltiplos comandos

## 🔍 Analogia do Mundo Real

**Pedido em restaurante**: O garçom (sender) anota seu pedido em um papel (command object) sem precisar saber cozinhar. O pedido contém toda informação necessária (prato, quantidade, observações). O pedido vai para fila (queue), é entregue ao chef (receiver) que executa. O pedido pode ser cancelado (undo) antes de ficar pronto, e fica arquivado para histórico.

## ⚠️ Considerações Importantes

### Implementação de Undo:

#### Abordagem 1: Salvar estado anterior
```java
class Command {
    private Object previousState;
    
    void execute() {
        previousState = receiver.getState();
        receiver.action();
    }
    
    void undo() {
        receiver.setState(previousState);
    }
}
```

#### Abordagem 2: Operação inversa
```java
class IncrementCommand {
    void execute() {
        receiver.increment();
    }
    
    void undo() {
        receiver.decrement(); // Operação inversa
    }
}
```

### Macro Commands:
```java
class MacroCommand implements Command {
    private List<Command> commands;
    
    void execute() {
        commands.forEach(Command::execute);
    }
    
    void undo() {
        // Desfaz em ordem reversa
        Collections.reverse(commands);
        commands.forEach(Command::undo);
    }
}
```

### Command Queue:
```java
class CommandQueue {
    private Queue<Command> queue = new LinkedList<>();
    
    void add(Command cmd) {
        queue.offer(cmd);
    }
    
    void processAll() {
        while (!queue.isEmpty()) {
            queue.poll().execute();
        }
    }
}
```

### Frameworks que usam:
- **Spring**: `@Command` annotation
- **CQRS**: Command/Query separation
- **Redux**: Actions são commands
- **Java Swing**: Action interface

---

> **💡 Dica de Estudo:** Command é como um "pedido escrito" - ao invés de pedir diretamente (acoplamento), você escreve o pedido em um papel (command object) que pode ser guardado, enfileirado, enviado ou até cancelado. Use quando precisar tratar requisições como objetos de primeira classe.

> **📖 Referência:** [Refactoring Guru - Command](https://refactoring.guru/design-patterns/command)
