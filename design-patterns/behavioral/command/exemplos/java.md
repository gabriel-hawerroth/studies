# Exemplos Práticos - Command (Java)

## Exemplo 1: Editor de Texto com Undo/Redo

```java
import java.util.Stack;

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

## Exemplo 2: Sistema de Smart Home

```java
import java.util.*;

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
