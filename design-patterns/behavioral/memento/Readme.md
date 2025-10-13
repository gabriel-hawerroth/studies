# Memento

## 🎯 Intenção

O Memento (também conhecido como **Snapshot**) é um padrão de projeto comportamental que permite salvar e restaurar o estado anterior de um objeto sem revelar os detalhes de sua implementação. Ele captura e externaliza o estado interno de um objeto de forma que o objeto possa ser restaurado para esse estado mais tarde, tudo isso sem violar o encapsulamento.

## 🚩 Problema

Imagine que você está criando um aplicativo editor de texto. Além da edição simples, seu editor pode formatar texto, inserir imagens inline, etc. Em algum momento, você decidiu permitir que os usuários desfaçam qualquer operação realizada no texto.

### Resultado problemático:
```java
// Tentativa de salvar estado expondo campos
class Editor {
    // Campos públicos ou com getters - quebra encapsulamento
    public String text;
    public int cursorX;
    public int cursorY;
    public int selectionWidth;
    public Font font;
    public Color color;
    // ... muitos outros campos
}

// Cliente precisa conhecer estrutura interna
class History {
    private List<EditorState> states = new ArrayList<>();
    
    public void saveState(Editor editor) {
        EditorState state = new EditorState();
        // Precisa acessar todos os campos
        state.text = editor.text;
        state.cursorX = editor.cursorX;
        state.cursorY = editor.cursorY;
        state.selectionWidth = editor.selectionWidth;
        state.font = editor.font;
        state.color = editor.color;
        // ... copiar todos os campos
        states.add(state);
    }
    
    public void restoreState(Editor editor, int index) {
        EditorState state = states.get(index);
        // Precisa atribuir todos os campos
        editor.text = state.text;
        editor.cursorX = state.cursorX;
        editor.cursorY = state.cursorY;
        // ... restaurar todos os campos
    }
}

// Classe de estado separada - muito acoplada
class EditorState {
    public String text;
    public int cursorX;
    public int cursorY;
    // Espelha estrutura do Editor
    // Se Editor mudar, EditorState também precisa mudar!
}
```

**Problemas:**
- **Quebra de encapsulamento**: Campos privados expostos publicamente
- **Alto acoplamento**: Cliente depende da estrutura interna
- **Difícil manutenção**: Mudanças no Editor afetam várias classes
- **Responsabilidades misturadas**: Cliente gerencia estado de outro objeto
- **Não escalável**: Adicionar campo requer mudanças em vários lugares
- **Segurança**: Estado privado acessível a todos

## ✅ Solução

O padrão Memento delega a criação dos snapshots de estado para o próprio dono desse estado, o objeto **originator**. Assim, ao invés de outros objetos tentarem copiar o estado do editor "de fora", a própria classe editor pode fazer o snapshot, já que tem acesso completo ao seu próprio estado.

### Características-chave:
- **Encapsulamento preservado**: Estado privado permanece privado
- **Originator cria memento**: Dono do estado cria snapshot
- **Memento imutável**: Snapshot não pode ser modificado
- **Caretaker gerencia**: Mantém histórico sem acessar conteúdo
- **Restauração completa**: Originator pode restaurar estado completo
- **Desacoplamento**: Cliente não conhece estrutura interna

```java
// Originator cria seu próprio memento
class Editor {
    private String text;
    private int cursorX, cursorY;
    private int selectionWidth;
    
    // Cria snapshot do estado atual
    public EditorMemento createMemento() {
        return new EditorMemento(text, cursorX, cursorY, selectionWidth);
    }
    
    // Restaura de snapshot
    public void restore(EditorMemento memento) {
        this.text = memento.getText();
        this.cursorX = memento.getCursorX();
        this.cursorY = memento.getCursorY();
        this.selectionWidth = memento.getSelectionWidth();
    }
    
    // Memento como classe interna - acesso aos campos privados
    public static class EditorMemento {
        private final String text;
        private final int cursorX, cursorY;
        private final int selectionWidth;
        
        private EditorMemento(String text, int cursorX, 
                             int cursorY, int selectionWidth) {
            this.text = text;
            this.cursorX = cursorX;
            this.cursorY = cursorY;
            this.selectionWidth = selectionWidth;
        }
        
        // Getters privados - só Editor acessa
        private String getText() { return text; }
        private int getCursorX() { return cursorX; }
        private int getCursorY() { return cursorY; }
        private int getSelectionWidth() { return selectionWidth; }
    }
}

// Caretaker gerencia mementos sem acessar conteúdo
class History {
    private Stack<EditorMemento> mementos = new Stack<>();
    
    public void save(Editor editor) {
        mementos.push(editor.createMemento());
    }
    
    public void undo(Editor editor) {
        if (!mementos.isEmpty()) {
            editor.restore(mementos.pop());
        }
    }
}
```

## 🏗️ Estrutura

```
Originator                    Memento
    ↓                            ↓
createMemento() --------→  [snapshot do estado]
restore(memento) ←------   [estado imutável]
[estado interno]           [acesso restrito]
    ↑
    |
Caretaker
    ↓
[mantém mementos]
[não acessa conteúdo]
```

### Componentes:
- **Originator**: Objeto que tem estado para salvar/restaurar
- **Memento**: Snapshot imutável do estado do Originator
- **Caretaker**: Gerencia mementos sem acessar seu conteúdo (histórico, comando undo)

## 💻 Exemplos Práticos

### Exemplo 1: Editor de Texto com Undo/Redo

```java
// Originator - Editor
class TextEditor {
    private String content;
    private int cursorPosition;
    private String fontName;
    private int fontSize;
    
    public TextEditor() {
        this.content = "";
        this.cursorPosition = 0;
        this.fontName = "Arial";
        this.fontSize = 12;
    }
    
    // Operações do editor
    public void type(String text) {
        String before = content.substring(0, cursorPosition);
        String after = content.substring(cursorPosition);
        content = before + text + after;
        cursorPosition += text.length();
        System.out.println("✏️  Digitado: \"" + text + "\"");
    }
    
    public void delete(int length) {
        if (cursorPosition >= length) {
            String before = content.substring(0, cursorPosition - length);
            String after = content.substring(cursorPosition);
            content = before + after;
            cursorPosition -= length;
            System.out.println("🗑️  Deletado " + length + " caracteres");
        }
    }
    
    public void moveCursor(int position) {
        if (position >= 0 && position <= content.length()) {
            cursorPosition = position;
            System.out.println("➡️  Cursor movido para posição " + position);
        }
    }
    
    public void changeFont(String fontName, int fontSize) {
        this.fontName = fontName;
        this.fontSize = fontSize;
        System.out.println("🎨 Fonte alterada: " + fontName + " " + fontSize + "pt");
    }
    
    // Cria memento
    public EditorMemento save() {
        return new EditorMemento(content, cursorPosition, fontName, fontSize);
    }
    
    // Restaura de memento
    public void restore(EditorMemento memento) {
        this.content = memento.content;
        this.cursorPosition = memento.cursorPosition;
        this.fontName = memento.fontName;
        this.fontSize = memento.fontSize;
    }
    
    public void showStatus() {
        System.out.println("\n📄 Estado atual:");
        System.out.println("   Conteúdo: \"" + content + "\"");
        System.out.println("   Cursor: posição " + cursorPosition);
        System.out.println("   Fonte: " + fontName + " " + fontSize + "pt");
    }
    
    // Memento como classe interna estática
    public static class EditorMemento {
        private final String content;
        private final int cursorPosition;
        private final String fontName;
        private final int fontSize;
        private final long timestamp;
        
        private EditorMemento(String content, int cursorPosition, 
                             String fontName, int fontSize) {
            this.content = content;
            this.cursorPosition = cursorPosition;
            this.fontName = fontName;
            this.fontSize = fontSize;
            this.timestamp = System.currentTimeMillis();
        }
        
        // Metadados públicos (OK expor)
        public long getTimestamp() {
            return timestamp;
        }
        
        public String getPreview() {
            String preview = content.substring(0, Math.min(20, content.length()));
            return preview + (content.length() > 20 ? "..." : "");
        }
    }
}

// Caretaker - Gerenciador de histórico
class EditorHistory {
    private Stack<TextEditor.EditorMemento> undoStack;
    private Stack<TextEditor.EditorMemento> redoStack;
    private TextEditor editor;
    
    public EditorHistory(TextEditor editor) {
        this.editor = editor;
        this.undoStack = new Stack<>();
        this.redoStack = new Stack<>();
    }
    
    public void backup() {
        undoStack.push(editor.save());
        redoStack.clear(); // Limpa redo ao fazer nova ação
        System.out.println("💾 Backup salvo (histórico: " + undoStack.size() + ")");
    }
    
    public void undo() {
        if (!undoStack.isEmpty()) {
            redoStack.push(editor.save());
            TextEditor.EditorMemento memento = undoStack.pop();
            editor.restore(memento);
            System.out.println("↩️  Undo executado");
        } else {
            System.out.println("⚠️  Nada para desfazer");
        }
    }
    
    public void redo() {
        if (!redoStack.isEmpty()) {
            undoStack.push(editor.save());
            TextEditor.EditorMemento memento = redoStack.pop();
            editor.restore(memento);
            System.out.println("↪️  Redo executado");
        } else {
            System.out.println("⚠️  Nada para refazer");
        }
    }
    
    public void showHistory() {
        System.out.println("\n📚 Histórico:");
        System.out.println("   Undo disponível: " + undoStack.size() + " estados");
        System.out.println("   Redo disponível: " + redoStack.size() + " estados");
        
        if (!undoStack.isEmpty()) {
            System.out.println("\n   Últimos estados (undo):");
            int count = 1;
            for (int i = undoStack.size() - 1; i >= Math.max(0, undoStack.size() - 3); i--) {
                TextEditor.EditorMemento m = undoStack.get(i);
                System.out.println("   " + count + ". \"" + m.getPreview() + "\"");
                count++;
            }
        }
    }
}

// Uso
public class TextEditorExample {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        EditorHistory history = new EditorHistory(editor);
        
        System.out.println("=== Editor de Texto com Undo/Redo ===\n");
        
        // Estado inicial
        history.backup();
        editor.showStatus();
        
        // Operação 1: Digite texto
        System.out.println("\n--- Operação 1 ---");
        history.backup();
        editor.type("Hello");
        editor.showStatus();
        
        // Operação 2: Digite mais texto
        System.out.println("\n--- Operação 2 ---");
        history.backup();
        editor.type(" World");
        editor.showStatus();
        
        // Operação 3: Mova cursor e digite
        System.out.println("\n--- Operação 3 ---");
        history.backup();
        editor.moveCursor(5);
        editor.type(" Beautiful");
        editor.showStatus();
        
        // Operação 4: Mude fonte
        System.out.println("\n--- Operação 4 ---");
        history.backup();
        editor.changeFont("Times New Roman", 14);
        editor.showStatus();
        
        // Mostra histórico
        history.showHistory();
        
        // Undo
        System.out.println("\n--- Desfazendo operações ---");
        history.undo();
        editor.showStatus();
        
        history.undo();
        editor.showStatus();
        
        history.undo();
        editor.showStatus();
        
        // Redo
        System.out.println("\n--- Refazendo operações ---");
        history.redo();
        editor.showStatus();
        
        history.redo();
        editor.showStatus();
        
        // Nova operação após undo - limpa redo
        System.out.println("\n--- Nova operação (limpa redo) ---");
        history.backup();
        editor.type("!!!");
        editor.showStatus();
        
        history.showHistory();
    }
}
```

### Exemplo 2: Sistema de Configuração com Snapshots

```java
// Originator - Configuração do Sistema
class SystemConfiguration {
    private String theme;
    private String language;
    private int volume;
    private boolean notificationsEnabled;
    private Map<String, String> customSettings;
    
    public SystemConfiguration() {
        this.theme = "light";
        this.language = "en";
        this.volume = 50;
        this.notificationsEnabled = true;
        this.customSettings = new HashMap<>();
    }
    
    // Operações
    public void setTheme(String theme) {
        this.theme = theme;
        System.out.println("🎨 Tema alterado: " + theme);
    }
    
    public void setLanguage(String language) {
        this.language = language;
        System.out.println("🌍 Idioma alterado: " + language);
    }
    
    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
        System.out.println("🔊 Volume: " + this.volume + "%");
    }
    
    public void setNotifications(boolean enabled) {
        this.notificationsEnabled = enabled;
        System.out.println("🔔 Notificações: " + (enabled ? "ativadas" : "desativadas"));
    }
    
    public void setCustomSetting(String key, String value) {
        customSettings.put(key, value);
        System.out.println("⚙️  Configuração customizada: " + key + " = " + value);
    }
    
    // Cria memento
    public ConfigMemento createSnapshot(String description) {
        return new ConfigMemento(
            theme, language, volume, notificationsEnabled,
            new HashMap<>(customSettings), description
        );
    }
    
    // Restaura de memento
    public void restore(ConfigMemento memento) {
        this.theme = memento.theme;
        this.language = memento.language;
        this.volume = memento.volume;
        this.notificationsEnabled = memento.notificationsEnabled;
        this.customSettings = new HashMap<>(memento.customSettings);
    }
    
    public void showConfiguration() {
        System.out.println("\n⚙️  Configuração Atual:");
        System.out.println("   Tema: " + theme);
        System.out.println("   Idioma: " + language);
        System.out.println("   Volume: " + volume + "%");
        System.out.println("   Notificações: " + (notificationsEnabled ? "ON" : "OFF"));
        if (!customSettings.isEmpty()) {
            System.out.println("   Configurações customizadas: " + customSettings);
        }
    }
    
    // Memento
    public static class ConfigMemento {
        private final String theme;
        private final String language;
        private final int volume;
        private final boolean notificationsEnabled;
        private final Map<String, String> customSettings;
        private final String description;
        private final LocalDateTime timestamp;
        
        private ConfigMemento(String theme, String language, int volume,
                            boolean notificationsEnabled, 
                            Map<String, String> customSettings,
                            String description) {
            this.theme = theme;
            this.language = language;
            this.volume = volume;
            this.notificationsEnabled = notificationsEnabled;
            this.customSettings = customSettings;
            this.description = description;
            this.timestamp = LocalDateTime.now();
        }
        
        // Metadados públicos
        public String getDescription() {
            return description;
        }
        
        public LocalDateTime getTimestamp() {
            return timestamp;
        }
        
        public String getFormattedTimestamp() {
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
            return timestamp.format(formatter);
        }
    }
}

// Caretaker - Gerenciador de perfis de configuração
class ConfigurationManager {
    private Map<String, SystemConfiguration.ConfigMemento> savedConfigs;
    private SystemConfiguration config;
    
    public ConfigurationManager(SystemConfiguration config) {
        this.config = config;
        this.savedConfigs = new HashMap<>();
    }
    
    public void saveProfile(String profileName, String description) {
        SystemConfiguration.ConfigMemento memento = config.createSnapshot(description);
        savedConfigs.put(profileName, memento);
        System.out.println("💾 Perfil salvo: " + profileName);
    }
    
    public void loadProfile(String profileName) {
        SystemConfiguration.ConfigMemento memento = savedConfigs.get(profileName);
        if (memento != null) {
            config.restore(memento);
            System.out.println("📂 Perfil carregado: " + profileName);
        } else {
            System.out.println("❌ Perfil não encontrado: " + profileName);
        }
    }
    
    public void deleteProfile(String profileName) {
        if (savedConfigs.remove(profileName) != null) {
            System.out.println("🗑️  Perfil removido: " + profileName);
        } else {
            System.out.println("❌ Perfil não encontrado: " + profileName);
        }
    }
    
    public void listProfiles() {
        System.out.println("\n📋 Perfis Salvos:");
        if (savedConfigs.isEmpty()) {
            System.out.println("   Nenhum perfil salvo");
        } else {
            for (Map.Entry<String, SystemConfiguration.ConfigMemento> entry : savedConfigs.entrySet()) {
                SystemConfiguration.ConfigMemento m = entry.getValue();
                System.out.println("   • " + entry.getKey());
                System.out.println("     Descrição: " + m.getDescription());
                System.out.println("     Data: " + m.getFormattedTimestamp());
            }
        }
    }
    
    public void exportProfile(String profileName, String filename) {
        SystemConfiguration.ConfigMemento memento = savedConfigs.get(profileName);
        if (memento != null) {
            // Simulação de exportação
            System.out.println("📤 Perfil exportado para: " + filename);
            System.out.println("   (Implementação real salvaria em arquivo JSON/XML)");
        } else {
            System.out.println("❌ Perfil não encontrado: " + profileName);
        }
    }
}

// Uso
public class ConfigurationExample {
    public static void main(String[] args) {
        SystemConfiguration config = new SystemConfiguration();
        ConfigurationManager manager = new ConfigurationManager(config);
        
        System.out.println("=== Sistema de Configuração com Perfis ===\n");
        
        // Configuração padrão
        config.showConfiguration();
        manager.saveProfile("default", "Configuração padrão do sistema");
        
        // Perfil "Trabalho"
        System.out.println("\n--- Criando perfil 'Trabalho' ---");
        config.setTheme("light");
        config.setLanguage("pt-BR");
        config.setVolume(30);
        config.setNotifications(false);
        config.setCustomSetting("autoSave", "enabled");
        config.showConfiguration();
        manager.saveProfile("work", "Perfil para ambiente de trabalho");
        
        // Perfil "Casa"
        System.out.println("\n--- Criando perfil 'Casa' ---");
        config.setTheme("dark");
        config.setLanguage("en");
        config.setVolume(80);
        config.setNotifications(true);
        config.setCustomSetting("nightMode", "enabled");
        config.showConfiguration();
        manager.saveProfile("home", "Perfil para uso doméstico");
        
        // Perfil "Gaming"
        System.out.println("\n--- Criando perfil 'Gaming' ---");
        config.setTheme("dark");
        config.setVolume(100);
        config.setNotifications(false);
        config.setCustomSetting("performanceMode", "high");
        config.setCustomSetting("fps", "unlimited");
        config.showConfiguration();
        manager.saveProfile("gaming", "Perfil otimizado para jogos");
        
        // Lista perfis
        manager.listProfiles();
        
        // Carrega perfis diferentes
        System.out.println("\n--- Testando troca de perfis ---");
        manager.loadProfile("work");
        config.showConfiguration();
        
        manager.loadProfile("home");
        config.showConfiguration();
        
        manager.loadProfile("default");
        config.showConfiguration();
        
        // Exporta perfil
        System.out.println("\n--- Exportando perfil ---");
        manager.exportProfile("gaming", "gaming-profile.json");
        
        // Deleta perfil
        System.out.println("\n--- Removendo perfil ---");
        manager.deleteProfile("gaming");
        manager.listProfiles();
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Undo/Redo**: Implementar funcionalidade de desfazer
- **Snapshots**: Salvar estado de objetos periodicamente
- **Transações**: Reverter em caso de erro
- **Checkpoints**: Salvar pontos de restauração
- **Preservar encapsulamento**: Não quer expor campos internos
- **Histórico**: Manter histórico de estados

### 📝 Exemplos de aplicação:
- **Text editors**: Undo/redo operations
- **Game state**: Save points, checkpoint systems
- **Database transactions**: Rollback on error
- **Configuration management**: Profile switching
- **Drawing applications**: Step-by-step undo
- **Form wizards**: Navigation between steps

### ❌ Evite quando:
- **Estado grande**: Muita memória para snapshots
- **Estados frequentes**: Overhead de criação
- **Encapsulamento não importa**: Estado já é público

## 🚀 Como Implementar

1. **Determine Originator** - classe cujo estado deve ser salvo

2. **Crie classe Memento** - espelhe campos do Originator

3. **Torne Memento imutável** - sem setters, dados via construtor

4. **Use classe interna** (se linguagem suporta) para proteger Memento

5. **Adicione método createMemento()** no Originator

6. **Adicione método restore(memento)** no Originator

7. **Crie Caretaker** para gerenciar mementos (histórico, comandos)

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Encapsulamento preservado**: Estado privado permanece privado
- **Simplicidade**: Originator mantém responsabilidade
- **Histórico completo**: Pode salvar múltiplos estados
- **Independência**: Caretaker independente de Originator

### ❌ Desvantagens:
- **Consumo de memória**: RAM se criar mementos frequentemente
- **Lifecycle management**: Caretaker deve destruir mementos obsoletos
- **Linguagens dinâmicas**: PHP, Python, JS não garantem imutabilidade
- **Performance**: Overhead ao copiar estado grande

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Estado | Quando usar |
|--------|------|--------|-------------|
| **Memento** | Salvar/restaurar estado | Snapshot imutável | Undo, checkpoints |
| **Prototype** | Clonar objetos | Cópia completa | Criar objetos similares |
| **Command** | Encapsular ação | Comando reversível | Undo via operações inversas |
| **State** | Comportamento variável | Estado ativo | Máquina de estados |

## 🔗 Relações com Outros Padrões

- **Command + Memento**: Command armazena Memento antes de executar
  - Command: Faz ação
  - Memento: Salva estado antes da ação

- **Iterator + Memento**: Captura estado da iteração para reverter se necessário

- **Prototype vs Memento**:
  - Prototype: Clone completo (para criar novos objetos)
  - Memento: Snapshot de estado (para restaurar)
  - Memento mais simples se objeto não tem recursos externos

## 📚 Conceitos-Chave para Lembrar

1. **Encapsulamento**: Estado privado permanece privado
2. **Originator**: Cria e restaura de mementos
3. **Memento**: Snapshot imutável do estado
4. **Caretaker**: Gerencia mementos sem acessar conteúdo
5. **Classe interna**: Protege acesso ao memento
6. **Imutabilidade**: Memento não deve ser modificado

## 🔍 Analogia do Mundo Real

**Save game em videogame**: Quando você salva o progresso em um jogo, o sistema cria um "snapshot" completo do estado do jogo - posição do personagem, itens no inventário, progresso das missões, etc. Você pode carregar esse save point mais tarde para voltar exatamente àquele momento. O arquivo de save (memento) não expõe como o jogo funciona internamente, mas contém tudo necessário para restaurar o estado. O sistema de saves (caretaker) gerencia múltiplos save files sem precisar entender o conteúdo de cada um.

## ⚠️ Considerações Importantes

### Implementação com classe interna:
```java
class Editor {
    private String content;
    
    public Memento save() {
        return new Memento(content);
    }
    
    public void restore(Memento m) {
        content = m.getContent(); // Acessa método privado!
    }
    
    // Classe interna - Editor pode acessar membros privados
    public static class Memento {
        private final String content;
        
        private Memento(String content) { // Construtor privado
            this.content = content;
        }
        
        private String getContent() { // Getter privado
            return content;
        }
    }
}
```

### Implementação com interface:
```java
// Interface pública (metadados)
interface Memento {
    String getName();
    long getTimestamp();
}

// Implementação privada (estado)
class ConcreteMemento implements Memento {
    private final String state;
    private final String name;
    private final long timestamp;
    
    public ConcreteMemento(String state, String name) {
        this.state = state;
        this.name = name;
        this.timestamp = System.currentTimeMillis();
    }
    
    String getState() { return state; } // package-private
    public String getName() { return name; }
    public long getTimestamp() { return timestamp; }
}
```

### Otimizações de memória:
```java
// Incremental memento - salva apenas mudanças
class IncrementalMemento {
    private final Memento baseState;
    private final List<Change> changes;
    
    // Restaura aplicando mudanças sobre base
    void restore(Editor editor) {
        editor.restore(baseState);
        for (Change change : changes) {
            change.apply(editor);
        }
    }
}

// Compressão
class CompressedMemento {
    private final byte[] compressedState;
    
    private CompressedMemento(Object state) {
        this.compressedState = compress(serialize(state));
    }
}

// Limit de histórico
class BoundedHistory {
    private static final int MAX_SIZE = 20;
    private LinkedList<Memento> history = new LinkedList<>();
    
    void save(Memento m) {
        if (history.size() >= MAX_SIZE) {
            history.removeFirst(); // Remove mais antigo
        }
        history.add(m);
    }
}
```

### Design Guidelines:
- **Imutabilidade**: Memento deve ser imutável
- **Encapsulamento**: Use classes internas ou interfaces
- **Metadados**: OK expor informações sobre o memento
- **Estado**: NUNCA exponha estado interno
- **Gerenciamento**: Caretaker cuida do lifecycle
- **Otimização**: Considere compressão/incremental para estados grandes

---

> **💡 Dica de Estudo:** Memento é como um "save point" em videogame - você salva o estado completo do jogo (memento) sem expor como o jogo funciona internamente. Pode carregar esse save depois para voltar exatamente àquele momento. O sistema de saves (caretaker) gerencia os arquivos sem precisar entender o conteúdo!

> **📖 Referência:** [Refactoring Guru - Memento](https://refactoring.guru/design-patterns/memento)
