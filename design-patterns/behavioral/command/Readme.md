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

## 💻 Exemplos Práticos

### Exemplo 1: Editor de Texto com Undo/Redo

```java
// Receiver - Editor de texto
class TextEditor {
    private StringBuilder text;
    private String clipboard;
    private int selectionStart;
    private int selectionEnd;
    
    public TextEditor() {
        this.text = new StringBuilder();
        this.clipboard = "";
    }
    
    public void write(String text) {
        this.text.append(text);
    }
    
    public void delete(int start, int end) {
        text.delete(start, end);
    }
    
    public void replaceSelection(String newText) {
        if (selectionStart >= 0 && selectionEnd <= text.length()) {
            text.replace(selectionStart, selectionEnd, newText);
        }
    }
    
    public String getSelection() {
        if (selectionStart >= 0 && selectionEnd <= text.length()) {
            return text.substring(selectionStart, selectionEnd);
        }
        return "";
    }
    
    public String getText() {
        return text.toString();
    }
    
    public void setText(String text) {
        this.text = new StringBuilder(text);
    }
    
    public void setSelection(int start, int end) {
        this.selectionStart = start;
        this.selectionEnd = end;
    }
    
    public String getClipboard() {
        return clipboard;
    }
    
    public void setClipboard(String clipboard) {
        this.clipboard = clipboard;
    }
}

// Command interface
interface Command {
    void execute();
    void undo();
}

// Comando de escrever texto
class WriteCommand implements Command {
    private TextEditor editor;
    private String textToWrite;
    private String previousText;
    
    public WriteCommand(TextEditor editor, String text) {
        this.editor = editor;
        this.textToWrite = text;
    }
    
    @Override
    public void execute() {
        previousText = editor.getText();
        editor.write(textToWrite);
        System.out.println("✏️  Escrevendo: \"" + textToWrite + "\"");
    }
    
    @Override
    public void undo() {
        editor.setText(previousText);
        System.out.println("↩️  Desfazendo escrita de: \"" + textToWrite + "\"");
    }
}

// Comando de copiar
class CopyCommand implements Command {
    private TextEditor editor;
    
    public CopyCommand(TextEditor editor) {
        this.editor = editor;
    }
    
    @Override
    public void execute() {
        String selected = editor.getSelection();
        editor.setClipboard(selected);
        System.out.println("📋 Copiando: \"" + selected + "\"");
    }
    
    @Override
    public void undo() {
        // Copy não altera estado, não precisa desfazer
        System.out.println("↩️  Copy não tem undo");
    }
}

// Comando de cortar
class CutCommand implements Command {
    private TextEditor editor;
    private String previousText;
    private String previousClipboard;
    private int selectionStart;
    private int selectionEnd;
    
    public CutCommand(TextEditor editor) {
        this.editor = editor;
    }
    
    @Override
    public void execute() {
        previousText = editor.getText();
        previousClipboard = editor.getClipboard();
        
        String selected = editor.getSelection();
        editor.setClipboard(selected);
        editor.replaceSelection("");
        
        System.out.println("✂️  Cortando: \"" + selected + "\"");
    }
    
    @Override
    public void undo() {
        editor.setText(previousText);
        editor.setClipboard(previousClipboard);
        System.out.println("↩️  Desfazendo corte");
    }
}

// Comando de colar
class PasteCommand implements Command {
    private TextEditor editor;
    private String previousText;
    
    public PasteCommand(TextEditor editor) {
        this.editor = editor;
    }
    
    @Override
    public void execute() {
        previousText = editor.getText();
        String clipboard = editor.getClipboard();
        editor.replaceSelection(clipboard);
        System.out.println("📄 Colando: \"" + clipboard + "\"");
    }
    
    @Override
    public void undo() {
        editor.setText(previousText);
        System.out.println("↩️  Desfazendo colagem");
    }
}

// Command History - gerencia histórico de comandos
class CommandHistory {
    private Stack<Command> history;
    private Stack<Command> redoStack;
    
    public CommandHistory() {
        this.history = new Stack<>();
        this.redoStack = new Stack<>();
    }
    
    public void push(Command command) {
        history.push(command);
        redoStack.clear(); // Limpa redo ao executar novo comando
    }
    
    public Command undo() {
        if (!history.isEmpty()) {
            Command command = history.pop();
            redoStack.push(command);
            return command;
        }
        return null;
    }
    
    public Command redo() {
        if (!redoStack.isEmpty()) {
            Command command = redoStack.pop();
            history.push(command);
            return command;
        }
        return null;
    }
    
    public boolean canUndo() {
        return !history.isEmpty();
    }
    
    public boolean canRedo() {
        return !redoStack.isEmpty();
    }
    
    public void showHistory() {
        System.out.println("\n📚 Histórico de comandos:");
        if (history.isEmpty()) {
            System.out.println("   (vazio)");
        } else {
            int i = 1;
            for (Command cmd : history) {
                System.out.println("   " + i++ + ". " + cmd.getClass().getSimpleName());
            }
        }
    }
}

// Invoker - Editor Application
class EditorApplication {
    private TextEditor editor;
    private CommandHistory history;
    
    public EditorApplication() {
        this.editor = new TextEditor();
        this.history = new CommandHistory();
    }
    
    public void executeCommand(Command command) {
        command.execute();
        history.push(command);
    }
    
    public void undo() {
        Command command = history.undo();
        if (command != null) {
            command.undo();
        } else {
            System.out.println("❌ Nada para desfazer");
        }
    }
    
    public void redo() {
        Command command = history.redo();
        if (command != null) {
            command.execute();
        } else {
            System.out.println("❌ Nada para refazer");
        }
    }
    
    public void showContent() {
        System.out.println("\n📄 Conteúdo do editor: \"" + editor.getText() + "\"");
    }
    
    public void showHistory() {
        history.showHistory();
    }
    
    // Métodos auxiliares para criar comandos
    public void write(String text) {
        executeCommand(new WriteCommand(editor, text));
    }
    
    public void copy(int start, int end) {
        editor.setSelection(start, end);
        executeCommand(new CopyCommand(editor));
    }
    
    public void cut(int start, int end) {
        editor.setSelection(start, end);
        executeCommand(new CutCommand(editor));
    }
    
    public void paste(int position) {
        editor.setSelection(position, position);
        executeCommand(new PasteCommand(editor));
    }
}

// Uso
public class EditorExample {
    public static void main(String[] args) {
        EditorApplication app = new EditorApplication();
        
        System.out.println("=== Editor de Texto com Undo/Redo ===");
        
        // Escreve texto
        app.write("Hello ");
        app.write("World");
        app.write("!");
        app.showContent();
        
        // Seleciona e copia "World"
        app.copy(6, 11);
        
        // Cola no final
        app.paste(12);
        app.showContent();
        
        // Corta " World!"
        app.cut(5, 12);
        app.showContent();
        
        // Mostra histórico
        app.showHistory();
        
        // Testa undo
        System.out.println("\n=== Testando Undo ===");
        app.undo();
        app.showContent();
        
        app.undo();
        app.showContent();
        
        // Testa redo
        System.out.println("\n=== Testando Redo ===");
        app.redo();
        app.showContent();
        
        app.redo();
        app.showContent();
    }
}
```

### Exemplo 2: Sistema de Smart Home

```java
// Receivers - dispositivos da casa
class Light {
    private String location;
    private boolean isOn;
    private int brightness;
    
    public Light(String location) {
        this.location = location;
        this.isOn = false;
        this.brightness = 0;
    }
    
    public void on() {
        isOn = true;
        brightness = 100;
        System.out.println("💡 Luz da " + location + " ligada (100%)");
    }
    
    public void off() {
        isOn = false;
        brightness = 0;
        System.out.println("🌑 Luz da " + location + " desligada");
    }
    
    public void dim(int level) {
        isOn = true;
        brightness = level;
        System.out.println("🔅 Luz da " + location + " ajustada para " + level + "%");
    }
    
    public boolean isOn() { return isOn; }
    public int getBrightness() { return brightness; }
}

class Thermostat {
    private int temperature;
    
    public Thermostat() {
        this.temperature = 22;
    }
    
    public void setTemperature(int temp) {
        temperature = temp;
        System.out.println("🌡️  Termostato ajustado para " + temp + "°C");
    }
    
    public int getTemperature() { return temperature; }
}

class SecuritySystem {
    private boolean armed;
    
    public SecuritySystem() {
        this.armed = false;
    }
    
    public void arm() {
        armed = true;
        System.out.println("🔒 Sistema de segurança ativado");
    }
    
    public void disarm() {
        armed = false;
        System.out.println("🔓 Sistema de segurança desativado");
    }
    
    public boolean isArmed() { return armed; }
}

class GarageDoor {
    private boolean open;
    
    public GarageDoor() {
        this.open = false;
    }
    
    public void open() {
        open = true;
        System.out.println("🚪 Portão da garagem abrindo...");
    }
    
    public void close() {
        open = false;
        System.out.println("🚪 Portão da garagem fechando...");
    }
    
    public boolean isOpen() { return open; }
}

// Command interface
interface HomeCommand {
    void execute();
    void undo();
    String getDescription();
}

// Comandos para luz
class LightOnCommand implements HomeCommand {
    private Light light;
    private int previousBrightness;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        previousBrightness = light.getBrightness();
        light.on();
    }
    
    @Override
    public void undo() {
        if (previousBrightness == 0) {
            light.off();
        } else {
            light.dim(previousBrightness);
        }
    }
    
    @Override
    public String getDescription() {
        return "Ligar luz";
    }
}

class LightOffCommand implements HomeCommand {
    private Light light;
    private int previousBrightness;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        previousBrightness = light.getBrightness();
        light.off();
    }
    
    @Override
    public void undo() {
        if (previousBrightness > 0) {
            light.dim(previousBrightness);
        }
    }
    
    @Override
    public String getDescription() {
        return "Desligar luz";
    }
}

// Comando para termostato
class ThermostatCommand implements HomeCommand {
    private Thermostat thermostat;
    private int newTemperature;
    private int previousTemperature;
    
    public ThermostatCommand(Thermostat thermostat, int temperature) {
        this.thermostat = thermostat;
        this.newTemperature = temperature;
    }
    
    @Override
    public void execute() {
        previousTemperature = thermostat.getTemperature();
        thermostat.setTemperature(newTemperature);
    }
    
    @Override
    public void undo() {
        thermostat.setTemperature(previousTemperature);
    }
    
    @Override
    public String getDescription() {
        return "Ajustar temperatura para " + newTemperature + "°C";
    }
}

// Macro Command - executa múltiplos comandos
class MacroCommand implements HomeCommand {
    private List<HomeCommand> commands;
    private String description;
    
    public MacroCommand(String description, HomeCommand... commands) {
        this.description = description;
        this.commands = Arrays.asList(commands);
    }
    
    @Override
    public void execute() {
        System.out.println("🎬 Executando macro: " + description);
        for (HomeCommand command : commands) {
            command.execute();
        }
    }
    
    @Override
    public void undo() {
        System.out.println("⏪ Desfazendo macro: " + description);
        // Desfaz em ordem reversa
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
    
    @Override
    public String getDescription() {
        return "Macro: " + description;
    }
}

// Remote Control (Invoker)
class SmartHomeRemote {
    private Map<String, HomeCommand> commands;
    private Stack<HomeCommand> history;
    
    public SmartHomeRemote() {
        this.commands = new HashMap<>();
        this.history = new Stack<>();
    }
    
    public void setCommand(String button, HomeCommand command) {
        commands.put(button, command);
    }
    
    public void pressButton(String button) {
        HomeCommand command = commands.get(button);
        if (command != null) {
            System.out.println("\n▶️  Botão pressionado: " + button);
            command.execute();
            history.push(command);
        } else {
            System.out.println("❌ Botão não configurado: " + button);
        }
    }
    
    public void pressUndo() {
        if (!history.isEmpty()) {
            HomeCommand command = history.pop();
            System.out.println("\n↩️  Desfazendo: " + command.getDescription());
            command.undo();
        } else {
            System.out.println("\n❌ Nada para desfazer");
        }
    }
    
    public void showCommands() {
        System.out.println("\n🎮 Comandos configurados:");
        commands.forEach((button, command) -> 
            System.out.println("   " + button + ": " + command.getDescription())
        );
    }
}

// Uso
public class SmartHomeExample {
    public static void main(String[] args) {
        // Cria dispositivos (receivers)
        Light livingRoomLight = new Light("sala");
        Light bedroomLight = new Light("quarto");
        Thermostat thermostat = new Thermostat();
        SecuritySystem security = new SecuritySystem();
        GarageDoor garage = new GarageDoor();
        
        // Cria comandos
        HomeCommand livingRoomOn = new LightOnCommand(livingRoomLight);
        HomeCommand livingRoomOff = new LightOffCommand(livingRoomLight);
        HomeCommand bedroomOn = new LightOnCommand(bedroomLight);
        HomeCommand bedroomOff = new LightOffCommand(bedroomLight);
        HomeCommand heatUp = new ThermostatCommand(thermostat, 25);
        HomeCommand coolDown = new ThermostatCommand(thermostat, 18);
        
        // Cria macro commands
        HomeCommand movieMode = new MacroCommand(
            "Modo Cinema",
            livingRoomOff,
            new ThermostatCommand(thermostat, 20)
        );
        
        HomeCommand sleepMode = new MacroCommand(
            "Modo Dormir",
            livingRoomOff,
            bedroomOff,
            new ThermostatCommand(thermostat, 18),
            new HomeCommand() {
                public void execute() { security.arm(); }
                public void undo() { security.disarm(); }
                public String getDescription() { return "Ativar segurança"; }
            }
        );
        
        // Configura controle remoto
        SmartHomeRemote remote = new SmartHomeRemote();
        remote.setCommand("Sala ON", livingRoomOn);
        remote.setCommand("Sala OFF", livingRoomOff);
        remote.setCommand("Quarto ON", bedroomOn);
        remote.setCommand("Quarto OFF", bedroomOff);
        remote.setCommand("Esquentar", heatUp);
        remote.setCommand("Esfriar", coolDown);
        remote.setCommand("Cinema", movieMode);
        remote.setCommand("Dormir", sleepMode);
        
        remote.showCommands();
        
        // Simula uso
        System.out.println("\n=== Chegando em casa ===");
        remote.pressButton("Sala ON");
        remote.pressButton("Esquentar");
        
        System.out.println("\n=== Hora do cinema ===");
        remote.pressButton("Cinema");
        
        System.out.println("\n=== Hora de dormir ===");
        remote.pressButton("Dormir");
        
        System.out.println("\n=== Testando undo ===");
        remote.pressUndo(); // Desfaz modo dormir
        remote.pressUndo(); // Desfaz modo cinema
    }
}
```

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
