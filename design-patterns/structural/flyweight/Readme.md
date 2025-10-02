# Flyweight

## 🎯 Intenção

O Flyweight é um padrão de projeto estrutural que permite economizar memória RAM mantendo eficientemente grandes quantidades de objetos similares através do compartilhamento de partes comuns do estado entre múltiplos objetos ao invés de manter todos os dados em cada objeto.

## 🚩 Problema

Imagine que você decidiu criar um jogo simples: jogadores se movem por um mapa e atiram uns nos outros. Você implementou um sistema realista de partículas com muitas balas, mísseis e fragmentos voando pelo mapa.

### Resultado problemático:
```java
// Cada partícula é um objeto separado com todos os dados
class Particle {
    private int x, y, z;           // coordenadas (estado extrínseco)
    private Vector velocity;       // velocidade (estado extrínseco)
    private Color color;           // cor (estado intrínseco)
    private Sprite sprite;         // sprite (estado intrínseco)
    private int damage;            // dano (estado intrínseco)
    private String type;           // tipo (estado intrínseco)
    
    // Milhares de objetos = alto consumo de memória
}

// Resultado: muitos objetos com dados duplicados
List<Particle> particles = new ArrayList<>();
particles.add(new Particle(10, 20, RED, bulletSprite, 50, "bullet"));
particles.add(new Particle(15, 25, RED, bulletSprite, 50, "bullet"));
particles.add(new Particle(20, 30, RED, bulletSprite, 50, "bullet"));
// ... milhares de balas com mesma cor, sprite, dano e tipo
```

**Problemas:**
- **Alto consumo de memória**: Cada objeto armazena dados duplicados
- **Performance ruim**: Muitos objetos pequenos na heap
- **Crash do jogo**: RAM insuficiente para milhares de partículas
- **Desperdício**: Cor, sprite e tipo são iguais para muitas partículas

## ✅ Solução

O padrão Flyweight sugere que você pare de armazenar o **estado extrínseco** dentro do objeto. Em vez disso, passe este estado para métodos específicos que dependem dele. Apenas o **estado intrínseco** permanece no objeto, permitindo reutilizá-lo em diferentes contextos.

### Conceitos-chave:
- **Estado Intrínseco**: Dados constantes compartilhados (cor, sprite, tipo)
- **Estado Extrínseco**: Dados únicos contextuais (posição, velocidade)
- **Flyweight**: Objeto que armazena apenas estado intrínseco
- **Context**: Objeto que armazena estado extrínseco + referência ao flyweight

```java
// Flyweight - apenas estado intrínseco
class ParticleType {
    private Color color;
    private Sprite sprite;
    private int damage;
    private String type;
    
    public void render(Canvas canvas, int x, int y, Vector velocity) {
        // Usa dados intrínsecos + extrínsecos recebidos por parâmetro
    }
}

// Context - estado extrínseco + referência ao flyweight
class Particle {
    private int x, y, z;
    private Vector velocity;
    private ParticleType type; // referência ao flyweight
}

// Resultado: 3 flyweights para milhares de partículas
```

## 🏗️ Estrutura

```
Client → Context → Flyweight
              ↓         ↑
        [extrinsic] [intrinsic]
              ↓         ↑
         FlyweightFactory
              ↓
         [pool of flyweights]
```

### Componentes:
- **Flyweight**: Interface que define métodos através dos quais flyweights podem operar em estado extrínseco
- **ConcreteFlyweight**: Implementa interface e armazena estado intrínseco
- **Context**: Armazena estado extrínseco + referência ao flyweight
- **FlyweightFactory**: Gerencia pool de flyweights, garante reutilização
- **Client**: Calcula ou armazena estado extrínseco dos flyweights

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Partículas de Jogo

```java
// Flyweight - armazena apenas estado intrínseco
class ParticleType {
    private String name;
    private Color color;
    private String sprite;
    private int damage;
    
    public ParticleType(String name, Color color, String sprite, int damage) {
        this.name = name;
        this.color = color;
        this.sprite = sprite;
        this.damage = damage;
    }
    
    // Método que opera com estado extrínseco passado por parâmetro
    public void render(Graphics graphics, int x, int y, Vector velocity) {
        // 1. Desenha sprite na posição (x, y)
        graphics.setColor(color);
        graphics.drawImage(sprite, x, y);
        
        // 2. Aplica efeitos visuais baseados na velocidade
        if (velocity.getSpeed() > 100) {
            graphics.drawTrail(x, y, velocity);
        }
        
        System.out.println("Renderizando " + name + " em (" + x + ", " + y + ")");
    }
    
    public void move(Particle particle, double deltaTime) {
        // Atualiza posição baseada na velocidade (estado extrínseco)
        Vector velocity = particle.getVelocity();
        particle.setX(particle.getX() + velocity.getX() * deltaTime);
        particle.setY(particle.getY() + velocity.getY() * deltaTime);
    }
    
    // Getters para estado intrínseco
    public int getDamage() { return damage; }
    public String getName() { return name; }
}

// Context - armazena estado extrínseco
class Particle {
    private double x, y;
    private Vector velocity;
    private ParticleType type; // referência ao flyweight
    private double life;
    
    public Particle(double x, double y, Vector velocity, ParticleType type) {
        this.x = x;
        this.y = y;
        this.velocity = velocity;
        this.type = type;
        this.life = 1.0;
    }
    
    public void render(Graphics graphics) {
        type.render(graphics, (int)x, (int)y, velocity);
    }
    
    public void update(double deltaTime) {
        type.move(this, deltaTime);
        life -= deltaTime * 0.1; // partícula perde vida com o tempo
    }
    
    public boolean isAlive() {
        return life > 0;
    }
    
    // Getters e setters
    public double getX() { return x; }
    public void setX(double x) { this.x = x; }
    public double getY() { return y; }
    public void setY(double y) { this.y = y; }
    public Vector getVelocity() { return velocity; }
}

// Factory para gerenciar flyweights
class ParticleTypeFactory {
    private static Map<String, ParticleType> particleTypes = new HashMap<>();
    
    public static ParticleType getParticleType(String name, Color color, 
                                             String sprite, int damage) {
        String key = name + "_" + color + "_" + sprite + "_" + damage;
        
        ParticleType type = particleTypes.get(key);
        if (type == null) {
            System.out.println("Criando novo tipo de partícula: " + name);
            type = new ParticleType(name, color, sprite, damage);
            particleTypes.put(key, type);
        } else {
            System.out.println("Reutilizando tipo de partícula: " + name);
        }
        
        return type;
    }
    
    public static void printStats() {
        System.out.println("Tipos de partículas criados: " + particleTypes.size());
    }
}

// Classe auxiliar para velocidade
class Vector {
    private double x, y;
    
    public Vector(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    public double getSpeed() { return Math.sqrt(x*x + y*y); }
}

// Sistema principal do jogo
class ParticleSystem {
    private List<Particle> particles;
    private Graphics graphics;
    
    public ParticleSystem(Graphics graphics) {
        this.particles = new ArrayList<>();
        this.graphics = graphics;
    }
    
    public void createExplosion(double x, double y) {
        Random random = new Random();
        
        // Cria muitos fragmentos (mesmo tipo de flyweight)
        for (int i = 0; i < 50; i++) {
            ParticleType shrapnelType = ParticleTypeFactory.getParticleType(
                "Shrapnel", Color.GRAY, "shrapnel.png", 10);
            
            Vector velocity = new Vector(
                random.nextDouble() * 200 - 100,
                random.nextDouble() * 200 - 100
            );
            
            particles.add(new Particle(x, y, velocity, shrapnelType));
        }
        
        // Cria algumas balas (outro tipo de flyweight)
        for (int i = 0; i < 10; i++) {
            ParticleType bulletType = ParticleTypeFactory.getParticleType(
                "Bullet", Color.YELLOW, "bullet.png", 25);
            
            Vector velocity = new Vector(
                random.nextDouble() * 300 - 150,
                random.nextDouble() * 300 - 150
            );
            
            particles.add(new Particle(x, y, velocity, bulletType));
        }
    }
    
    public void update(double deltaTime) {
        Iterator<Particle> iterator = particles.iterator();
        while (iterator.hasNext()) {
            Particle particle = iterator.next();
            particle.update(deltaTime);
            
            if (!particle.isAlive()) {
                iterator.remove();
            }
        }
    }
    
    public void render() {
        for (Particle particle : particles) {
            particle.render(graphics);
        }
    }
    
    public void printStats() {
        System.out.println("Partículas ativas: " + particles.size());
        ParticleTypeFactory.printStats();
    }
}

// Uso
public class GameExample {
    public static void main(String[] args) {
        Graphics graphics = new MockGraphics();
        ParticleSystem particleSystem = new ParticleSystem(graphics);
        
        System.out.println("=== Criando explosões ===");
        
        // Simula várias explosões
        particleSystem.createExplosion(100, 100);
        particleSystem.createExplosion(200, 150);
        particleSystem.createExplosion(300, 200);
        
        particleSystem.printStats();
        
        System.out.println("\n=== Simulando alguns frames ===");
        
        // Simula alguns frames do jogo
        for (int frame = 0; frame < 3; frame++) {
            System.out.println("\n--- Frame " + (frame + 1) + " ---");
            particleSystem.update(0.016); // ~60 FPS
            particleSystem.render();
        }
        
        System.out.println("\n=== Estatísticas finais ===");
        particleSystem.printStats();
    }
}

// Mock classes para o exemplo
class MockGraphics implements Graphics {
    public void setColor(Color color) {}
    public void drawImage(String sprite, int x, int y) {}
    public void drawTrail(int x, int y, Vector velocity) {}
}

interface Graphics {
    void setColor(Color color);
    void drawImage(String sprite, int x, int y);
    void drawTrail(int x, int y, Vector velocity);
}

enum Color { RED, GRAY, YELLOW }
```

### Exemplo 2: Editor de Texto

```java
// Flyweight - formatação de caractere (estado intrínseco)
class CharacterFormat {
    private String fontFamily;
    private int fontSize;
    private Color color;
    private boolean bold;
    private boolean italic;
    
    public CharacterFormat(String fontFamily, int fontSize, Color color, 
                          boolean bold, boolean italic) {
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
        this.color = color;
        this.bold = bold;
        this.italic = italic;
    }
    
    public void render(Graphics graphics, char character, int x, int y) {
        // Aplica formatação e desenha caractere
        Font font = new Font(fontFamily, fontSize, bold, italic);
        graphics.setFont(font);
        graphics.setColor(color);
        graphics.drawChar(character, x, y);
        
        System.out.println("Renderizando '" + character + "' em (" + x + ", " + y + 
                          ") com fonte " + fontFamily + " " + fontSize);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        CharacterFormat that = (CharacterFormat) obj;
        return fontSize == that.fontSize &&
               bold == that.bold &&
               italic == that.italic &&
               Objects.equals(fontFamily, that.fontFamily) &&
               Objects.equals(color, that.color);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(fontFamily, fontSize, color, bold, italic);
    }
    
    @Override
    public String toString() {
        return fontFamily + " " + fontSize + "pt " + 
               (bold ? "Bold " : "") + (italic ? "Italic " : "") + color;
    }
}

// Context - caractere com posição (estado extrínseco)
class Character {
    private char value;
    private int x, y;
    private CharacterFormat format; // referência ao flyweight
    
    public Character(char value, int x, int y, CharacterFormat format) {
        this.value = value;
        this.x = x;
        this.y = y;
        this.format = format;
    }
    
    public void render(Graphics graphics) {
        format.render(graphics, value, x, y);
    }
    
    // Getters e setters
    public char getValue() { return value; }
    public int getX() { return x; }
    public void setX(int x) { this.x = x; }
    public int getY() { return y; }
    public void setY(int y) { this.y = y; }
    public CharacterFormat getFormat() { return format; }
}

// Factory para formatos de caractere
class CharacterFormatFactory {
    private static Map<String, CharacterFormat> formats = new HashMap<>();
    
    public static CharacterFormat getFormat(String fontFamily, int fontSize, 
                                          Color color, boolean bold, boolean italic) {
        String key = fontFamily + "_" + fontSize + "_" + color + "_" + bold + "_" + italic;
        
        CharacterFormat format = formats.get(key);
        if (format == null) {
            System.out.println("Criando novo formato: " + key);
            format = new CharacterFormat(fontFamily, fontSize, color, bold, italic);
            formats.put(key, format);
        }
        
        return format;
    }
    
    public static void printStats() {
        System.out.println("Formatos de caractere criados: " + formats.size());
        formats.values().forEach(format -> 
            System.out.println("  - " + format.toString()));
    }
}

// Editor de texto
class TextEditor {
    private List<Character> characters;
    private Graphics graphics;
    
    public TextEditor(Graphics graphics) {
        this.characters = new ArrayList<>();
        this.graphics = graphics;
    }
    
    public void addText(String text, int startX, int startY, 
                       String fontFamily, int fontSize, Color color) {
        int x = startX;
        
        for (char ch : text.toCharArray()) {
            if (ch == ' ') {
                x += 10; // espaço entre palavras
                continue;
            }
            
            // Determina formatação baseada no caractere
            boolean bold = Character.isUpperCase(ch);
            boolean italic = Character.isDigit(ch);
            
            CharacterFormat format = CharacterFormatFactory.getFormat(
                fontFamily, fontSize, color, bold, italic);
            
            characters.add(new Character(ch, x, startY, format));
            x += 12; // espaçamento entre caracteres
        }
    }
    
    public void addFormattedText(String text, int startX, int startY,
                               String fontFamily, int fontSize, Color color,
                               boolean bold, boolean italic) {
        int x = startX;
        
        CharacterFormat format = CharacterFormatFactory.getFormat(
            fontFamily, fontSize, color, bold, italic);
        
        for (char ch : text.toCharArray()) {
            if (ch == ' ') {
                x += 10;
                continue;
            }
            
            characters.add(new Character(ch, x, startY, format));
            x += 12;
        }
    }
    
    public void render() {
        System.out.println("=== Renderizando documento ===");
        for (Character character : characters) {
            character.render(graphics);
        }
    }
    
    public void printStats() {
        System.out.println("Caracteres no documento: " + characters.size());
        CharacterFormatFactory.printStats();
    }
}

// Uso
public class TextEditorExample {
    public static void main(String[] args) {
        Graphics graphics = new MockTextGraphics();
        TextEditor editor = new TextEditor(graphics);
        
        System.out.println("=== Adicionando texto ao documento ===");
        
        // Adiciona diferentes tipos de texto
        editor.addText("Hello World 123", 10, 50, "Arial", 12, Color.BLACK);
        editor.addFormattedText("Título Principal", 10, 100, "Times", 18, Color.BLUE, true, false);
        editor.addFormattedText("Subtítulo em itálico", 10, 130, "Times", 14, Color.GRAY, false, true);
        editor.addText("Mais texto normal", 10, 160, "Arial", 12, Color.BLACK);
        
        System.out.println("\n=== Estatísticas ===");
        editor.printStats();
        
        System.out.println("\n=== Renderização ===");
        editor.render();
    }
}

// Mock classes para texto
class MockTextGraphics implements Graphics {
    public void setFont(Font font) {}
    public void setColor(Color color) {}
    public void drawChar(char character, int x, int y) {}
}

class Font {
    private String family;
    private int size;
    private boolean bold;
    private boolean italic;
    
    public Font(String family, int size, boolean bold, boolean italic) {
        this.family = family;
        this.size = size;
        this.bold = bold;
        this.italic = italic;
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Aplicação precisa gerar** grande número de objetos similares
- **Custo de armazenamento** é alto devido a quantidade de objetos
- **Objetos contêm estado duplicado** que pode ser extraído e compartilhado
- **RAM disponível** é limitada no dispositivo alvo

### 📝 Exemplos de uso:
- **Jogos**: Sistemas de partículas, sprites, terrenos
- **Editores de texto**: Formatação de caracteres
- **Interfaces gráficas**: Componentes visuais reutilizáveis
- **Sistemas de cache**: Objetos compartilhados frequentemente acessados

### ❌ Evite quando:
- **Poucos objetos similares**: Overhead desnecessário
- **Estado majoritariamente único**: Pouco para compartilhar
- **Complexidade > benefício**: Ganho de memória não justifica complexidade
- **CPU limitada**: Recálculo de contexto pode ser custoso

## 🚀 Como Implementar

1. **Divida campos** da classe em duas partes:
   - **Estado intrínseco**: dados imutáveis duplicados
   - **Estado extrínseco**: dados contextuais únicos

2. **Mantenha campos intrínsecos** na classe, garantindo imutabilidade

3. **Revise métodos** que usam estado extrínseco - adicione parâmetros

4. **Crie factory** para gerenciar pool de flyweights

5. **Cliente deve armazenar** estado extrínseco para chamar métodos do flyweight

6. **Considere classe Context** para agrupar estado extrínseco + referência

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Economia de RAM**: Redução significativa no uso de memória
- **Performance**: Menos objetos na heap, melhor cache locality
- **Compartilhamento**: Reutilização eficiente de objetos
- **Escalabilidade**: Suporta grande quantidade de objetos

### ❌ Desvantagens:
- **Complexidade**: Código mais complexo para entender
- **CPU vs RAM**: Pode trocar RAM por ciclos de CPU
- **Recálculo**: Estado extrínseco pode precisar ser recalculado
- **Debugging**: Mais difícil rastrear estado distribuído

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Compartilhamento | Imutabilidade |
|--------|------|------------------|---------------|
| **Flyweight** | Economia de memória | Estado intrínseco | Flyweights imutáveis |
| **Singleton** | Instância única | Instância completa | Pode ser mutável |
| **Object Pool** | Reutilização | Objetos completos | Podem ser modificados |
| **Prototype** | Clonagem | Templates | Clones independentes |

## 🔗 Relações com Outros Padrões

- **Composite**: Nós folha podem ser implementados como flyweights
- **Facade**: Flyweight mostra como fazer muitos objetos pequenos, Facade um objeto grande
- **Singleton**: Flyweight pode ter múltiplas instâncias com estados intrínsecos diferentes
- **State/Strategy**: Flyweights são imutáveis, enquanto States/Strategies podem mudar

## 📚 Conceitos-Chave para Lembrar

1. **Separação de estado**: Intrínseco (compartilhado) vs Extrínseco (contextual)
2. **Imutabilidade**: Flyweights não podem ser modificados após criação
3. **Factory pattern**: Gerencia pool e garante reutilização
4. **Economia de memória**: Principal benefício do padrão
5. **Contexto**: Cliente mantém estado extrínseco
6. **Trade-off**: RAM vs CPU - pode recalcular dados

## 🔍 Analogia do Mundo Real

**Biblioteca pública**: Os livros são como flyweights (estado intrínseco - conteúdo, autor, ISBN). O estado extrínseco são as informações de empréstimo (quem pegou, quando deve devolver, histórico). Milhares de pessoas podem "usar" o mesmo livro, mas cada uma tem seu próprio contexto de empréstimo. A biblioteca não precisa de uma cópia do livro para cada pessoa - apenas mantém as informações de empréstimo separadamente.

## ⚠️ Considerações Importantes

### Design Guidelines:
- **Identifique candidatos**: Classes com muitas instâncias similares
- **Análise de estado**: Separe claramente intrínseco vs extrínseco
- **Factory obrigatório**: Essencial para controlar instâncias
- **Documentação**: Estado extrínseco deve ser bem documentado

### Implementação eficiente:
```java
// Use collections eficientes para o pool
Map<String, Flyweight> flyweights = new ConcurrentHashMap<>();

// Considere weak references se apropriado
Map<String, WeakReference<Flyweight>> flyweights = new WeakHashMap<>();

// Implemente equals/hashCode corretamente
@Override
public boolean equals(Object obj) {
    // Compare apenas estado intrínseco
}
```

### Medição de benefício:
- **Antes**: Memory = Objects × (IntrinsicSize + ExtrinsicSize)
- **Depois**: Memory = Flyweights × IntrinsicSize + Objects × ExtrinsicSize
- **Benefício**: Significativo quando Flyweights << Objects

---

> **💡 Dica de Estudo:** Flyweight é como um "molde reutilizável" - você tem poucos moldes (flyweights) que podem criar muitos produtos diferentes dependendo do material (contexto) que você coloca neles. Use quando tiver muitos objetos similares consumindo muita memória.

> **📖 Referência:** [Refactoring Guru - Flyweight](https://refactoring.guru/design-patterns/flyweight)
