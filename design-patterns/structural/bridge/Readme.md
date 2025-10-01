# Bridge

## 🎯 Intenção

O Bridge é um padrão de projeto estrutural que permite dividir uma classe grande ou um conjunto de classes intimamente relacionadas em duas hierarquias separadas—**abstração** e **implementação**—que podem ser desenvolvidas independentemente uma da outra.

## 🚩 Problema

Imagine que você tem uma classe geométrica `Shape` com subclasses `Circle` e `Square`. Você quer estender essa hierarquia para incorporar cores, criando subclasses `Red` e `Blue`. Como já tem duas subclasses, precisará criar quatro combinações: `BlueCircle`, `RedSquare`, etc.

### Explosão Combinatória:
```
Shape
├── Circle
│   ├── RedCircle
│   └── BlueCircle
└── Square
    ├── RedSquare
    └── BlueSquare
```

**Problema**: Adicionar novas formas e cores faz a hierarquia crescer exponencialmente. Para adicionar um triângulo, você precisaria de duas subclasses (uma para cada cor). Depois, adicionar uma nova cor exigiria três subclasses (uma para cada forma).

**Exemplo problemático:**
```java
// Explosão de classes
class RedCircle extends Circle { ... }
class BlueCircle extends Circle { ... }
class RedSquare extends Square { ... }
class BlueSquare extends Square { ... }
// Para N formas e M cores = N×M classes!
```

## ✅ Solução

O Bridge resolve esse problema **trocando herança por composição**. Você extrai uma das dimensões em uma hierarquia de classes separada, fazendo a classe original referenciar um objeto da nova hierarquia.

### Separação de Responsabilidades:
- **Abstração**: Camada de controle de alto nível (ex: GUI)
- **Implementação**: Trabalho real de baixo nível (ex: API do SO)

```java
// Antes: Herança múltipla
class RedCircle extends Circle { ... }

// Depois: Composição
class Circle {
    private Color color; // Bridge para implementação
    
    public void draw() {
        color.applyColor(); // Delega para implementação
        // ... desenhar círculo
    }
}
```

## 🏗️ Estrutura

```
Client → Abstraction ←--bridge--→ Implementation
           ↑                           ↑
    RefinedAbstraction          ConcreteImplementation
```

### Componentes:
- **Abstraction**: Lógica de controle de alto nível, referencia Implementation
- **RefinedAbstraction**: Variantes da lógica de controle
- **Implementation**: Interface comum para todas as implementações concretas
- **ConcreteImplementation**: Código específico da plataforma
- **Client**: Trabalha com Abstraction, mas vincula com Implementation

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Notificações

```java
// Implementation - diferentes canais de envio
interface NotificationSender {
    void sendNotification(String title, String message);
}

class EmailSender implements NotificationSender {
    private String emailAddress;
    
    public EmailSender(String email) {
        this.emailAddress = email;
    }
    
    @Override
    public void sendNotification(String title, String message) {
        System.out.println("Email para " + emailAddress);
        System.out.println("Assunto: " + title);
        System.out.println("Mensagem: " + message);
    }
}

class SMSSender implements NotificationSender {
    private String phoneNumber;
    
    public SMSSender(String phone) {
        this.phoneNumber = phone;
    }
    
    @Override
    public void sendNotification(String title, String message) {
        System.out.println("SMS para " + phoneNumber);
        System.out.println(title + ": " + message);
    }
}

class SlackSender implements NotificationSender {
    private String channelId;
    
    public SlackSender(String channel) {
        this.channelId = channel;
    }
    
    @Override
    public void sendNotification(String title, String message) {
        System.out.println("Slack para canal " + channelId);
        System.out.println("**" + title + "**\n" + message);
    }
}

// Abstraction - diferentes tipos de notificação
abstract class Notification {
    protected NotificationSender sender;
    
    public Notification(NotificationSender sender) {
        this.sender = sender;
    }
    
    public abstract void send(String title, String message);
}

// Refined Abstractions
class SimpleNotification extends Notification {
    public SimpleNotification(NotificationSender sender) {
        super(sender);
    }
    
    @Override
    public void send(String title, String message) {
        sender.sendNotification(title, message);
    }
}

class UrgentNotification extends Notification {
    public UrgentNotification(NotificationSender sender) {
        super(sender);
    }
    
    @Override
    public void send(String title, String message) {
        sender.sendNotification("🚨 URGENTE: " + title, message.toUpperCase());
    }
}

class ReminderNotification extends Notification {
    public ReminderNotification(NotificationSender sender) {
        super(sender);
    }
    
    @Override
    public void send(String title, String message) {
        String reminderMessage = "📅 Lembrete: " + message;
        sender.sendNotification(title, reminderMessage);
    }
}

// Uso
public class NotificationExample {
    public static void main(String[] args) {
        // Diferentes combinações: tipo de notificação × canal de envio
        
        // Notificação simples por email
        Notification emailNotification = new SimpleNotification(
            new EmailSender("user@example.com")
        );
        emailNotification.send("Bem-vindo", "Conta criada com sucesso!");
        
        System.out.println();
        
        // Notificação urgente por SMS
        Notification urgentSMS = new UrgentNotification(
            new SMSSender("+5511999999999")
        );
        urgentSMS.send("Sistema", "Servidor fora do ar");
        
        System.out.println();
        
        // Lembrete por Slack
        Notification slackReminder = new ReminderNotification(
            new SlackSender("#dev-team")
        );
        slackReminder.send("Reunião", "Daily meeting em 15 minutos");
    }
}
```

### Exemplo 2: Sistema de Desenho Multiplataforma

```java
// Implementation - diferentes APIs gráficas
interface DrawingAPI {
    void drawCircle(int x, int y, int radius);
    void drawRectangle(int x, int y, int width, int height);
}

class WindowsAPI implements DrawingAPI {
    @Override
    public void drawCircle(int x, int y, int radius) {
        System.out.printf("Windows API: Círculo em (%d,%d) raio %d%n", x, y, radius);
    }
    
    @Override
    public void drawRectangle(int x, int y, int width, int height) {
        System.out.printf("Windows API: Retângulo em (%d,%d) %dx%d%n", x, y, width, height);
    }
}

class LinuxAPI implements DrawingAPI {
    @Override
    public void drawCircle(int x, int y, int radius) {
        System.out.printf("Linux API: Círculo em (%d,%d) raio %d%n", x, y, radius);
    }
    
    @Override
    public void drawRectangle(int x, int y, int width, int height) {
        System.out.printf("Linux API: Retângulo em (%d,%d) %dx%d%n", x, y, width, height);
    }
}

// Abstraction - formas geométricas
abstract class Shape {
    protected DrawingAPI drawingAPI;
    
    protected Shape(DrawingAPI drawingAPI) {
        this.drawingAPI = drawingAPI;
    }
    
    public abstract void draw();
    public abstract void resize(double factor);
}

// Refined Abstractions
class Circle extends Shape {
    private int x, y, radius;
    
    public Circle(int x, int y, int radius, DrawingAPI drawingAPI) {
        super(drawingAPI);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }
    
    @Override
    public void draw() {
        drawingAPI.drawCircle(x, y, radius);
    }
    
    @Override
    public void resize(double factor) {
        radius = (int) (radius * factor);
    }
}

class Rectangle extends Shape {
    private int x, y, width, height;
    
    public Rectangle(int x, int y, int width, int height, DrawingAPI drawingAPI) {
        super(drawingAPI);
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    
    @Override
    public void draw() {
        drawingAPI.drawRectangle(x, y, width, height);
    }
    
    @Override
    public void resize(double factor) {
        width = (int) (width * factor);
        height = (int) (height * factor);
    }
}

// Uso
public class DrawingExample {
    public static void main(String[] args) {
        // Formas podem usar qualquer API
        Shape[] shapes = {
            new Circle(10, 10, 5, new WindowsAPI()),
            new Rectangle(20, 20, 30, 15, new WindowsAPI()),
            new Circle(50, 50, 8, new LinuxAPI()),
            new Rectangle(60, 60, 25, 20, new LinuxAPI())
        };
        
        for (Shape shape : shapes) {
            shape.draw();
            shape.resize(1.5);
            shape.draw();
            System.out.println();
        }
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Dividir classe monolítica** com várias variantes de funcionalidade
- **Estender classe em dimensões ortogonais** (independentes)
- **Trocar implementações em runtime**
- **Desenvolver partes independentemente** (equipes diferentes)

### 📝 Exemplos de uso:
- **Aplicações multiplataforma**: GUI + API do SO
- **Drivers de dispositivos**: Interface comum + implementações específicas
- **Sistemas de persistência**: DAO + diferentes bancos de dados
- **Protocolos de comunicação**: Interface + TCP/UDP/HTTP

### ❌ Evite quando:
- **Classe é altamente coesa**: Pode aumentar complexidade desnecessariamente
- **Apenas uma dimensão de variação**: Use Strategy ou State
- **Hierarquia simples**: Pode ser over-engineering

## 🚀 Como Implementar

1. **Identifique dimensões ortogonais**: abstração/plataforma, domínio/infraestrutura, front-end/back-end

2. **Defina operações do cliente** na classe abstração base

3. **Determine operações da plataforma** e declare na interface de implementação

4. **Crie implementações concretas** para todas as plataformas

5. **Adicione campo de referência** na abstração para o tipo de implementação

6. **Crie abstrações refinadas** estendendo a abstração base

7. **Cliente associa implementação** à abstração via construtor

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Classes/apps independentes de plataforma**
- **Cliente trabalha com abstrações** de alto nível
- **Princípio Aberto/Fechado**: Novas abstrações e implementações independentes
- **Princípio da Responsabilidade Única**: Lógica vs detalhes de plataforma
- **Desenvolvimento paralelo**: Equipes independentes

### ❌ Desvantagens:
- **Complexidade aumenta** em classes altamente coesas
- **Overhead de indireção**: Camada extra de abstração
- **Pode ser over-engineering** para casos simples

## 🔗 Diferenças de Outros Padrões

| Padrão | Quando usar | Estrutura | Foco |
|--------|-------------|-----------|------|
| **Bridge** | Design antecipado | Duas hierarquias | Separar abstração/implementação |
| **Adapter** | Código existente | Wrapper | Compatibilizar interfaces |
| **Strategy** | Runtime behavior | Algoritmos intercambiáveis | Trocar comportamento |
| **State** | Estado do objeto | Estados como classes | Mudar comportamento por estado |

## 🔗 Relações com Outros Padrões

- **Abstract Factory + Bridge**: Encapsular relações entre abstrações e implementações
- **Builder + Bridge**: Director como abstração, builders como implementações
- **Strategy**: Estrutura similar, mas Strategy foca em algoritmos intercambiáveis
- **State**: Estrutura similar, mas State foca em mudanças de comportamento

## 📚 Conceitos-Chave para Lembrar

1. **Composição > Herança**: Evita explosão combinatória
2. **Duas hierarquias**: Abstração (controle) + Implementação (trabalho)
3. **Design antecipado**: Planejado desde o início vs Adapter (adaptação)
4. **Independência**: Desenvolver abstrações e implementações separadamente
5. **Runtime switching**: Trocar implementações dinamicamente
6. **Dimensões ortogonais**: Variações independentes

## 🔍 Analogia do Mundo Real

**Controle remoto e dispositivos**: Um controle remoto (abstração) pode operar diferentes dispositivos como TV, rádio, ar-condicionado (implementações). O controle define operações de alto nível (ligar, volume, canal), enquanto cada dispositivo implementa essas operações de forma específica. Você pode ter controles básicos ou avançados funcionando com qualquer dispositivo.

## ⚠️ Considerações Importantes

### Design antecipado:
- Bridge é projetado desde o início
- Permite desenvolvimento independente de equipes
- Facilita adição de novas plataformas/abstrações

### Vs Strategy Pattern:
- **Bridge**: Separação estrutural permanente
- **Strategy**: Intercambio comportamental temporário

### Performance:
- Indireção adicional pode impactar performance
- Benefício de flexibilidade geralmente compensa

---

> **💡 Dica de Estudo:** Bridge separa "o que faz" (abstração) de "como faz" (implementação). É como ter um controle remoto universal que funciona com qualquer aparelho - cada botão tem a mesma função, mas cada aparelho responde diferente.

> **📖 Referência:** [Refactoring Guru - Bridge](https://refactoring.guru/design-patterns/bridge)
