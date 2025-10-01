# Facade

## 🎯 Intenção

O Facade é um padrão de projeto estrutural que fornece uma interface simplificada para uma biblioteca, framework ou qualquer conjunto complexo de classes. Ele esconde a complexidade do subsistema por trás de uma interface simples.

## 🚩 Problema

Imagine que você deve fazer seu código trabalhar com um amplo conjunto de objetos que pertencem a uma biblioteca ou framework sofisticado. Normalmente, você precisaria:

- **Inicializar** todos esses objetos
- **Rastrear dependências** entre eles
- **Executar métodos** na ordem correta
- **Gerenciar** o ciclo de vida dos objetos

### Resultado problemático:
```java
// Cliente precisa conhecer toda a complexidade
VideoFile file = new VideoFile("video.ogg");
CodecFactory factory = new CodecFactory();
OggCompressionCodec sourceCodec = factory.extract(file);
MPEG4CompressionCodec destinationCodec = new MPEG4CompressionCodec();
BitrateReader reader = new BitrateReader();
Buffer buffer = reader.read("video.ogg", sourceCodec);
Buffer result = reader.convert(buffer, destinationCodec);
AudioMixer mixer = new AudioMixer();
result = mixer.fix(result);
File convertedFile = new File(result);
```

**Problemas:**
- **Acoplamento forte**: Lógica de negócio acoplada aos detalhes de implementação
- **Complexidade**: Cliente precisa conhecer muitas classes
- **Manutenção difícil**: Mudanças no subsistema afetam o cliente
- **Reutilização baixa**: Código repetitivo para tarefas similares

## ✅ Solução

Um Facade é uma classe que fornece uma **interface simples** para um subsistema complexo. Embora possa oferecer funcionalidade limitada comparado ao acesso direto, inclui apenas os recursos que os clientes realmente precisam.

### Características-chave:
- **Interface simplificada**: Esconde complexidade desnecessária
- **Ponto de entrada único**: Centraliza acesso ao subsistema
- **Desacoplamento**: Cliente independente do subsistema
- **Facilita uso**: Operações comuns ficam simples

```java
// Com Facade - interface simplificada
VideoConverter converter = new VideoConverter();
File mp4 = converter.convert("video.ogg", "mp4");
```

## 🏗️ Estrutura

```
Client → Facade → ComplexSubsystem
              ├── ClassA
              ├── ClassB  
              ├── ClassC
              └── ClassD
```

### Componentes:
- **Facade**: Fornece acesso conveniente ao subsistema, conhece como operar todas as partes
- **Additional Facade**: Pode ser criado para evitar poluir o facade principal (opcional)
- **Complex Subsystem**: Conjunto de dezenas de objetos variados que precisam trabalhar juntos
- **Client**: Usa o facade ao invés de chamar objetos do subsistema diretamente

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Home Theater

```java
// Subsistema complexo - componentes do home theater
class DVDPlayer {
    public void on() {
        System.out.println("DVD Player ligado");
    }
    
    public void play(String movie) {
        System.out.println("Reproduzindo: " + movie);
    }
    
    public void stop() {
        System.out.println("DVD Player parado");
    }
    
    public void off() {
        System.out.println("DVD Player desligado");
    }
}

class Projector {
    public void on() {
        System.out.println("Projetor ligado");
    }
    
    public void setInput(String input) {
        System.out.println("Entrada do projetor: " + input);
    }
    
    public void wideScreenMode() {
        System.out.println("Modo widescreen ativado");
    }
    
    public void off() {
        System.out.println("Projetor desligado");
    }
}

class SoundSystem {
    public void on() {
        System.out.println("Sistema de som ligado");
    }
    
    public void setVolume(int volume) {
        System.out.println("Volume ajustado para: " + volume);
    }
    
    public void setSurroundSound() {
        System.out.println("Som surround ativado");
    }
    
    public void off() {
        System.out.println("Sistema de som desligado");
    }
}

class Lights {
    public void dim(int level) {
        System.out.println("Luzes ajustadas para: " + level + "%");
    }
    
    public void on() {
        System.out.println("Luzes acesas");
    }
}

class PopcornMaker {
    public void on() {
        System.out.println("Pipoqueira ligada");
    }
    
    public void pop() {
        System.out.println("Fazendo pipoca...");
    }
    
    public void off() {
        System.out.println("Pipoqueira desligada");
    }
}

// Facade - interface simplificada para o home theater
class HomeTheaterFacade {
    private DVDPlayer dvdPlayer;
    private Projector projector;
    private SoundSystem soundSystem;
    private Lights lights;
    private PopcornMaker popcornMaker;
    
    public HomeTheaterFacade(DVDPlayer dvd, Projector projector, 
                           SoundSystem sound, Lights lights, 
                           PopcornMaker popcorn) {
        this.dvdPlayer = dvd;
        this.projector = projector;
        this.soundSystem = sound;
        this.lights = lights;
        this.popcornMaker = popcorn;
    }
    
    public void watchMovie(String movie) {
        System.out.println("Preparando para assistir " + movie + "...\n");
        
        popcornMaker.on();
        popcornMaker.pop();
        
        lights.dim(10);
        
        projector.on();
        projector.wideScreenMode();
        projector.setInput("DVD");
        
        soundSystem.on();
        soundSystem.setSurroundSound();
        soundSystem.setVolume(8);
        
        dvdPlayer.on();
        dvdPlayer.play(movie);
        
        System.out.println("\nFilme iniciado! Aproveite!");
    }
    
    public void endMovie() {
        System.out.println("\nFinalizando sessão de cinema...\n");
        
        popcornMaker.off();
        lights.on();
        projector.off();
        soundSystem.off();
        dvdPlayer.stop();
        dvdPlayer.off();
        
        System.out.println("Home theater desligado.");
    }
}

// Uso
public class HomeTheaterExample {
    public static void main(String[] args) {
        // Criando componentes do subsistema
        DVDPlayer dvd = new DVDPlayer();
        Projector projector = new Projector();
        SoundSystem sound = new SoundSystem();
        Lights lights = new Lights();
        PopcornMaker popcorn = new PopcornMaker();
        
        // Criando o facade
        HomeTheaterFacade homeTheater = new HomeTheaterFacade(
            dvd, projector, sound, lights, popcorn);
        
        // Cliente usa interface simples
        homeTheater.watchMovie("Os Vingadores");
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        homeTheater.endMovie();
    }
}
```

### Exemplo 2: Sistema de E-commerce

```java
// Subsistema complexo - componentes do e-commerce
class InventoryService {
    public boolean checkStock(String productId, int quantity) {
        System.out.println("Verificando estoque para produto: " + productId);
        return true; // Simplificado
    }
    
    public void reserveProduct(String productId, int quantity) {
        System.out.println("Reservando " + quantity + " unidades do produto: " + productId);
    }
}

class PaymentService {
    public boolean processPayment(String cardNumber, double amount) {
        System.out.println("Processando pagamento de R$ " + amount);
        return true; // Simplificado
    }
    
    public String generateTransactionId() {
        return "TXN" + System.currentTimeMillis();
    }
}

class ShippingService {
    public String calculateShipping(String address, double weight) {
        System.out.println("Calculando frete para: " + address);
        return "Frete: R$ 15,00";
    }
    
    public void scheduleDelivery(String address, String productId) {
        System.out.println("Agendando entrega para: " + address);
    }
}

class NotificationService {
    public void sendOrderConfirmation(String email, String orderId) {
        System.out.println("Enviando confirmação para: " + email + " - Pedido: " + orderId);
    }
    
    public void sendShippingNotification(String email, String trackingCode) {
        System.out.println("Enviando código de rastreamento: " + trackingCode);
    }
}

class OrderService {
    public String createOrder(String customerId, String productId, int quantity) {
        String orderId = "ORD" + System.currentTimeMillis();
        System.out.println("Criando pedido: " + orderId);
        return orderId;
    }
}

// Facade - interface simplificada para e-commerce
class EcommerceFacade {
    private InventoryService inventory;
    private PaymentService payment;
    private ShippingService shipping;
    private NotificationService notification;
    private OrderService orderService;
    
    public EcommerceFacade() {
        this.inventory = new InventoryService();
        this.payment = new PaymentService();
        this.shipping = new ShippingService();
        this.notification = new NotificationService();
        this.orderService = new OrderService();
    }
    
    public boolean placeOrder(String customerId, String customerEmail, 
                            String productId, int quantity, 
                            String cardNumber, double price, 
                            String address) {
        
        System.out.println("=== Processando Pedido ===\n");
        
        // 1. Verificar estoque
        if (!inventory.checkStock(productId, quantity)) {
            System.out.println("Produto fora de estoque!");
            return false;
        }
        
        // 2. Reservar produto
        inventory.reserveProduct(productId, quantity);
        
        // 3. Calcular frete
        String shippingInfo = shipping.calculateShipping(address, quantity * 0.5);
        System.out.println(shippingInfo);
        
        // 4. Processar pagamento
        double totalAmount = price * quantity + 15.00; // Preço + frete
        if (!payment.processPayment(cardNumber, totalAmount)) {
            System.out.println("Falha no pagamento!");
            return false;
        }
        
        // 5. Criar pedido
        String orderId = orderService.createOrder(customerId, productId, quantity);
        
        // 6. Agendar entrega
        shipping.scheduleDelivery(address, productId);
        
        // 7. Enviar notificações
        notification.sendOrderConfirmation(customerEmail, orderId);
        
        String transactionId = payment.generateTransactionId();
        notification.sendShippingNotification(customerEmail, "TRACK" + orderId);
        
        System.out.println("\n=== Pedido Processado com Sucesso! ===");
        System.out.println("ID do Pedido: " + orderId);
        System.out.println("Total pago: R$ " + totalAmount);
        
        return true;
    }
    
    public void getOrderStatus(String orderId) {
        System.out.println("Status do pedido " + orderId + ": Em processamento");
    }
}

// Uso
public class EcommerceExample {
    public static void main(String[] args) {
        EcommerceFacade ecommerce = new EcommerceFacade();
        
        // Cliente usa interface simples para operação complexa
        boolean success = ecommerce.placeOrder(
            "CUST001", 
            "cliente@email.com",
            "PROD123", 
            2, 
            "**** **** **** 1234", 
            99.90,
            "Rua das Flores, 123 - São Paulo, SP"
        );
        
        if (success) {
            System.out.println("\n✅ Compra realizada com sucesso!");
        } else {
            System.out.println("\n❌ Falha na compra.");
        }
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Interface limitada mas direta** para subsistema complexo
- **Estruturar subsistema em camadas** definindo pontos de entrada
- **Reduzir acoplamento** entre cliente e subsistema
- **Simplificar integração** com bibliotecas complexas

### 📝 Exemplos de uso:
- **APIs de terceiros**: Simplificar bibliotecas complexas
- **Sistemas legados**: Interface moderna para código antigo
- **Microservices**: Agregador de serviços relacionados
- **Frameworks**: Interface simplificada para funcionalidades comuns

### ❌ Evite quando:
- **Subsistema é simples**: Pode adicionar complexidade desnecessária
- **Cliente precisa de controle fino**: Facade pode ser limitante
- **Único ponto de falha**: Facade concentra muito poder

## 🚀 Como Implementar

1. **Analise se é possível** fornecer interface mais simples que o subsistema existente

2. **Declare e implemente** esta interface em nova classe facade

3. **Redirecione chamadas** do cliente para objetos apropriados do subsistema

4. **Gerencie ciclo de vida** do subsistema (inicialização, cleanup)

5. **Force comunicação** apenas via facade para proteger cliente de mudanças

6. **Considere facades adicionais** se o facade principal ficar muito grande

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Isola código** da complexidade do subsistema
- **Interface simples** para operações comuns
- **Reduz dependências** entre cliente e subsistema
- **Facilita manutenção** e evolução do sistema

### ❌ Desvantagens:
- **God Object**: Facade pode se tornar objeto acoplado a todas as classes
- **Limitação**: Pode não expor toda funcionalidade necessária
- **Ponto único de falha**: Concentra responsabilidades

## 🔗 Diferenças de Outros Padrões

| Padrão | Interface | Escopo | Propósito |
|--------|-----------|--------|-----------|
| **Facade** | Nova interface | Subsistema inteiro | Simplificar acesso |
| **Adapter** | Compatibilizar interface | Objeto único | Tornar compatível |
| **Proxy** | Mesma interface | Objeto único | Controlar acesso |
| **Decorator** | Mesma/estendida | Objeto único | Adicionar comportamento |

## 🔗 Relações com Outros Padrões

- **Abstract Factory**: Alternativa ao Facade para esconder criação de objetos
- **Mediator**: Facade simplifica interface, Mediator centraliza comunicação
- **Singleton**: Facade geralmente implementado como Singleton
- **Adapter**: Facade define nova interface, Adapter torna interface existente usável

## 📚 Conceitos-Chave para Lembrar

1. **Simplificação**: Esconde complexidade por trás de interface simples
2. **Ponto de entrada único**: Centraliza acesso ao subsistema
3. **Desacoplamento**: Cliente independente do subsistema
4. **Facilita uso**: Operações complexas ficam simples
5. **Não adiciona funcionalidade**: Apenas organiza acesso existente
6. **Subsistema independente**: Não sabe da existência do facade

## 🔍 Analogia do Mundo Real

**Atendente de loja por telefone**: Quando você liga para uma loja para fazer um pedido, o atendente é seu facade para todos os serviços e departamentos da loja. O atendente fornece uma interface de voz simples para o sistema de pedidos, gateways de pagamento e vários serviços de entrega. Você não precisa falar diretamente com cada departamento.

## ⚠️ Considerações Importantes

### Design Guidelines:
- **Interface mínima**: Exponha apenas o necessário
- **Operações de alto nível**: Combine operações do subsistema
- **Gerenciamento de estado**: Facade pode manter estado se necessário
- **Error handling**: Trate erros do subsistema adequadamente

### Múltiplos Facades:
```java
// Diferentes facades para diferentes aspectos
class MediaFacade {
    // Operações de mídia
}

class SecurityFacade {
    // Operações de segurança  
}

class AdminFacade {
    // Operações administrativas
}
```

### Facade vs API Gateway:
- **Facade**: Padrão de design para código
- **API Gateway**: Infraestrutura para microservices
- **Conceito similar**: Ambos simplificam acesso a sistemas complexos

---

> **💡 Dica de Estudo:** Facade é como um "menu executivo" - oferece as opções mais populares de forma simples, escondendo a complexidade da cozinha. Use quando quiser que o cliente tenha acesso fácil a operações complexas sem se preocupar com os detalhes.

> **📖 Referência:** [Refactoring Guru - Facade](https://refactoring.guru/design-patterns/facade)
