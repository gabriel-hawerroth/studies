# Observer

## 🎯 Intenção

O Observer (também conhecido como **Event-Subscriber** ou **Listener**) é um padrão de projeto comportamental que permite definir um mecanismo de assinatura para notificar múltiplos objetos sobre quaisquer eventos que aconteçam ao objeto que eles estão observando. Ele estabelece uma dependência um-para-muitos entre objetos, onde quando um objeto muda de estado, todos os seus dependentes são notificados automaticamente.

## 🚩 Problema

Imagine que você tem dois tipos de objetos: um `Cliente` e uma `Loja`. O cliente está muito interessado em uma marca específica de produto (digamos, um novo modelo de iPhone) que deve ficar disponível na loja em breve.

### Resultado problemático:
```java
// Cliente verifica manualmente
class Customer {
    public void checkProduct() {
        while (true) {
            // Visita loja todos os dias
            if (store.isProductAvailable("iPhone 15")) {
                store.buyProduct("iPhone 15");
                break;
            }
            // Maioria das visitas é inútil!
            sleep(1000 * 60 * 60 * 24); // 1 dia
        }
    }
}

// Loja notifica TODOS
class Store {
    private List<Customer> allCustomers;
    
    public void newProductArrived(Product product) {
        // Envia email para TODOS os clientes
        for (Customer customer : allCustomers) {
            customer.sendEmail("Novo produto: " + product.getName());
            // Muitos não estão interessados - SPAM!
        }
    }
}
```

**Problemas:**
- **Desperdício de recursos**: Cliente verifica constantemente sem necessidade
- **Spam**: Loja notifica todos mesmo sem interesse
- **Acoplamento**: Cliente precisa conhecer loja para verificar
- **Ineficiência**: Polling constante ou notificações desnecessárias
- **Não escalável**: Adicionar verificação em outros objetos duplica código
- **Sem flexibilidade**: Cliente não pode escolher quando parar de verificar

## ✅ Solução

O objeto que tem algum estado interessante é chamado de **subject** (sujeito), mas como ele também notifica outros objetos sobre mudanças, vamos chamá-lo de **publisher** (publicador). Todos os outros objetos que querem rastrear mudanças no estado do publisher são chamados de **subscribers** (assinantes).

### Características-chave:
- **Pub/Sub**: Publisher publica eventos, subscribers se inscrevem
- **Desacoplamento**: Publisher não conhece subscribers concretamente
- **Dinâmico**: Subscribers podem entrar/sair a qualquer momento
- **Notificação automática**: Mudanças notificam automaticamente
- **Interface uniforme**: Todos subscribers implementam mesma interface
- **Um-para-muitos**: Um publisher, múltiplos subscribers

```java
// Interface Observer/Subscriber
interface ProductObserver {
    void update(String productName, boolean available);
}

// Subject/Publisher
class Store {
    private List<ProductObserver> observers = new ArrayList<>();
    private Map<String, Boolean> products = new HashMap<>();
    
    // Gerencia subscribers
    public void subscribe(ProductObserver observer) {
        observers.add(observer);
    }
    
    public void unsubscribe(ProductObserver observer) {
        observers.remove(observer);
    }
    
    // Notifica quando produto chega
    public void setProductAvailable(String productName) {
        products.put(productName, true);
        notifyObservers(productName, true);
    }
    
    private void notifyObservers(String productName, boolean available) {
        for (ProductObserver observer : observers) {
            observer.update(productName, available);
        }
    }
}

// Concrete Observer
class Customer implements ProductObserver {
    private String interestedProduct;
    
    @Override
    public void update(String productName, boolean available) {
        // Só reage se for produto de interesse
        if (productName.equals(interestedProduct) && available) {
            System.out.println("Great! " + productName + " is available!");
        }
    }
}

// Uso
Store store = new Store();
Customer alice = new Customer("iPhone 15");
Customer bob = new Customer("MacBook Pro");

store.subscribe(alice); // Alice se inscreve
store.subscribe(bob);   // Bob se inscreve

store.setProductAvailable("iPhone 15");
// Só Alice é notificada!
```

## 🏗️ Estrutura

```
Publisher/Subject                Observer/Subscriber (interface)
    ↓                                     ↓
[lista de observers]                 update(data)
subscribe(observer)                      ↑
unsubscribe(observer)                    |
notifyObservers()              ConcreteObserverA  ConcreteObserverB
    ↓                                ↓                  ↓
[notifica todos]                update()            update()
```

### Componentes:
- **Publisher**: Mantém lista de subscribers e os notifica
- **Subscriber/Observer**: Interface com método de notificação (update)
- **ConcreteSubscriber**: Implementa reação às notificações
- **Client**: Cria e registra subscribers

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Notícias com Múltiplos Canais

```java
// Interface Observer
interface NewsSubscriber {
    void update(String category, String news);
}

// Publisher/Subject
class NewsAgency {
    private Map<String, List<NewsSubscriber>> subscribersByCategory;
    
    public NewsAgency() {
        this.subscribersByCategory = new HashMap<>();
    }
    
    // Subscribe para categoria específica
    public void subscribe(String category, NewsSubscriber subscriber) {
        subscribersByCategory
            .computeIfAbsent(category, k -> new ArrayList<>())
            .add(subscriber);
        System.out.println("✅ Inscrito em: " + category);
    }
    
    public void unsubscribe(String category, NewsSubscriber subscriber) {
        List<NewsSubscriber> subscribers = subscribersByCategory.get(category);
        if (subscribers != null) {
            subscribers.remove(subscriber);
            System.out.println("❌ Desinscrito de: " + category);
        }
    }
    
    // Publica notícia
    public void publishNews(String category, String news) {
        System.out.println("\n📰 PUBLICANDO: [" + category + "] " + news);
        
        List<NewsSubscriber> subscribers = subscribersByCategory.get(category);
        if (subscribers != null) {
            for (NewsSubscriber subscriber : subscribers) {
                subscriber.update(category, news);
            }
        }
    }
}

// Concrete Observer - Email
class EmailSubscriber implements NewsSubscriber {
    private String email;
    private Set<String> interests;
    
    public EmailSubscriber(String email, Set<String> interests) {
        this.email = email;
        this.interests = interests;
    }
    
    @Override
    public void update(String category, String news) {
        if (interests.contains(category)) {
            System.out.println("📧 Email para " + email + ": " + news);
        }
    }
    
    public String getEmail() { return email; }
}

// Concrete Observer - SMS
class SMSSubscriber implements NewsSubscriber {
    private String phoneNumber;
    private Set<String> interests;
    
    public SMSSubscriber(String phoneNumber, Set<String> interests) {
        this.phoneNumber = phoneNumber;
        this.interests = interests;
    }
    
    @Override
    public void update(String category, String news) {
        if (interests.contains(category)) {
            String shortNews = news.substring(0, Math.min(50, news.length()));
            System.out.println("📱 SMS para " + phoneNumber + ": " + shortNews + "...");
        }
    }
    
    public String getPhoneNumber() { return phoneNumber; }
}

// Concrete Observer - Push Notification
class PushNotificationSubscriber implements NewsSubscriber {
    private String deviceId;
    private boolean notificationsEnabled;
    
    public PushNotificationSubscriber(String deviceId) {
        this.deviceId = deviceId;
        this.notificationsEnabled = true;
    }
    
    @Override
    public void update(String category, String news) {
        if (notificationsEnabled) {
            System.out.println("🔔 Push para dispositivo " + deviceId + 
                             ": [" + category + "] " + news);
        }
    }
    
    public void setNotificationsEnabled(boolean enabled) {
        this.notificationsEnabled = enabled;
    }
}

// Concrete Observer - Logger
class NewsLogger implements NewsSubscriber {
    private String logFile;
    
    public NewsLogger(String logFile) {
        this.logFile = logFile;
    }
    
    @Override
    public void update(String category, String news) {
        String timestamp = LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        String logEntry = String.format("[%s] [%s] %s", timestamp, category, news);
        System.out.println("📝 LOG em " + logFile + ": " + logEntry);
    }
}

// Uso
public class NewsAgencyExample {
    public static void main(String[] args) {
        NewsAgency agency = new NewsAgency();
        
        System.out.println("=== Sistema de Notícias ===\n");
        
        // Cria subscribers
        EmailSubscriber alice = new EmailSubscriber(
            "alice@email.com",
            new HashSet<>(Arrays.asList("tecnologia", "ciência"))
        );
        
        EmailSubscriber bob = new EmailSubscriber(
            "bob@email.com",
            new HashSet<>(Arrays.asList("esportes", "política"))
        );
        
        SMSSubscriber charlie = new SMSSubscriber(
            "+55 11 99999-1234",
            new HashSet<>(Arrays.asList("tecnologia"))
        );
        
        PushNotificationSubscriber device1 = new PushNotificationSubscriber("device-001");
        
        NewsLogger logger = new NewsLogger("news.log");
        
        // Inscrições
        System.out.println("--- Inscrições ---");
        agency.subscribe("tecnologia", alice);
        agency.subscribe("tecnologia", charlie);
        agency.subscribe("tecnologia", device1);
        agency.subscribe("ciência", alice);
        agency.subscribe("esportes", bob);
        agency.subscribe("política", bob);
        
        // Logger em todas categorias
        agency.subscribe("tecnologia", logger);
        agency.subscribe("ciência", logger);
        agency.subscribe("esportes", logger);
        agency.subscribe("política", logger);
        
        // Publica notícias
        System.out.println("\n--- Publicações ---");
        agency.publishNews("tecnologia", 
            "Apple lança novo iPhone 16 com IA revolucionária");
        
        agency.publishNews("esportes", 
            "Brasil vence Copa do Mundo de Futebol 2026");
        
        agency.publishNews("ciência", 
            "Cientistas descobrem cura para o câncer");
        
        // Charlie desativa notificações por SMS
        System.out.println("\n--- Charlie cancela inscrição ---");
        agency.unsubscribe("tecnologia", charlie);
        
        agency.publishNews("tecnologia", 
            "Microsoft anuncia Windows 12 com recursos de IA");
        
        // Device1 desativa push notifications
        System.out.println("\n--- Device1 desativa notificações ---");
        device1.setNotificationsEnabled(false);
        
        agency.publishNews("tecnologia", 
            "Google apresenta novo motor de busca quântico");
    }
}
```

### Exemplo 2: Sistema de Monitoramento de Ações (Stock Market)

```java
// Interface Observer
interface StockObserver {
    void priceChanged(String symbol, double oldPrice, double newPrice);
}

// Publisher/Subject
class StockMarket {
    private Map<String, Double> stockPrices;
    private Map<String, List<StockObserver>> observersByStock;
    
    public StockMarket() {
        this.stockPrices = new HashMap<>();
        this.observersByStock = new HashMap<>();
    }
    
    public void addStock(String symbol, double initialPrice) {
        stockPrices.put(symbol, initialPrice);
        observersByStock.put(symbol, new ArrayList<>());
        System.out.println("📊 Ação adicionada: " + symbol + " = $" + initialPrice);
    }
    
    public void subscribe(String symbol, StockObserver observer) {
        List<StockObserver> observers = observersByStock.get(symbol);
        if (observers != null) {
            observers.add(observer);
            System.out.println("✅ Observer inscrito em " + symbol);
        }
    }
    
    public void unsubscribe(String symbol, StockObserver observer) {
        List<StockObserver> observers = observersByStock.get(symbol);
        if (observers != null) {
            observers.remove(observer);
            System.out.println("❌ Observer desinscrito de " + symbol);
        }
    }
    
    public void updatePrice(String symbol, double newPrice) {
        Double oldPrice = stockPrices.get(symbol);
        if (oldPrice != null && oldPrice != newPrice) {
            stockPrices.put(symbol, newPrice);
            notifyObservers(symbol, oldPrice, newPrice);
        }
    }
    
    private void notifyObservers(String symbol, double oldPrice, double newPrice) {
        List<StockObserver> observers = observersByStock.get(symbol);
        if (observers != null) {
            for (StockObserver observer : observers) {
                observer.priceChanged(symbol, oldPrice, newPrice);
            }
        }
    }
    
    public double getPrice(String symbol) {
        return stockPrices.getOrDefault(symbol, 0.0);
    }
}

// Concrete Observer - Investidor Individual
class Investor implements StockObserver {
    private String name;
    private Map<String, Integer> portfolio; // símbolo -> quantidade
    private double cashBalance;
    
    public Investor(String name, double initialCash) {
        this.name = name;
        this.cashBalance = initialCash;
        this.portfolio = new HashMap<>();
    }
    
    @Override
    public void priceChanged(String symbol, double oldPrice, double newPrice) {
        double changePercent = ((newPrice - oldPrice) / oldPrice) * 100;
        String trend = newPrice > oldPrice ? "📈" : "📉";
        
        System.out.printf("%s %s viu %s: $%.2f → $%.2f (%.2f%%)\n",
            trend, name, symbol, oldPrice, newPrice, changePercent);
        
        // Decisão de compra/venda
        if (portfolio.containsKey(symbol)) {
            int shares = portfolio.get(symbol);
            double portfolioValue = shares * newPrice;
            System.out.printf("   💰 Posição: %d ações, valor: $%.2f\n", 
                shares, portfolioValue);
        }
        
        // Estratégia simples: vende se cair 5%, compra se subir 5%
        if (changePercent <= -5.0 && portfolio.containsKey(symbol)) {
            sellStock(symbol, portfolio.get(symbol), newPrice);
        } else if (changePercent >= 5.0 && cashBalance > newPrice * 10) {
            buyStock(symbol, 10, newPrice);
        }
    }
    
    private void buyStock(String symbol, int quantity, double price) {
        double cost = quantity * price;
        if (cashBalance >= cost) {
            cashBalance -= cost;
            portfolio.put(symbol, portfolio.getOrDefault(symbol, 0) + quantity);
            System.out.printf("   ✅ %s COMPROU %d %s @ $%.2f (Total: $%.2f)\n",
                name, quantity, symbol, price, cost);
        }
    }
    
    private void sellStock(String symbol, int quantity, double price) {
        if (portfolio.getOrDefault(symbol, 0) >= quantity) {
            double revenue = quantity * price;
            cashBalance += revenue;
            portfolio.put(symbol, portfolio.get(symbol) - quantity);
            System.out.printf("   ❌ %s VENDEU %d %s @ $%.2f (Total: $%.2f)\n",
                name, quantity, symbol, price, revenue);
        }
    }
    
    public void showPortfolio(StockMarket market) {
        System.out.println("\n💼 Portfólio de " + name + ":");
        System.out.println("   Cash: $" + String.format("%.2f", cashBalance));
        
        double totalValue = cashBalance;
        for (Map.Entry<String, Integer> entry : portfolio.entrySet()) {
            String symbol = entry.getKey();
            int shares = entry.getValue();
            double currentPrice = market.getPrice(symbol);
            double value = shares * currentPrice;
            totalValue += value;
            
            System.out.printf("   %s: %d ações @ $%.2f = $%.2f\n",
                symbol, shares, currentPrice, value);
        }
        System.out.printf("   TOTAL: $%.2f\n", totalValue);
    }
}

// Concrete Observer - Sistema de Alertas
class PriceAlertSystem implements StockObserver {
    private Map<String, Double> alertThresholds; // símbolo -> preço alerta
    
    public PriceAlertSystem() {
        this.alertThresholds = new HashMap<>();
    }
    
    public void setAlert(String symbol, double targetPrice) {
        alertThresholds.put(symbol, targetPrice);
        System.out.println("🔔 Alerta configurado: " + symbol + 
                         " quando atingir $" + targetPrice);
    }
    
    @Override
    public void priceChanged(String symbol, double oldPrice, double newPrice) {
        Double threshold = alertThresholds.get(symbol);
        if (threshold != null) {
            if ((oldPrice < threshold && newPrice >= threshold) ||
                (oldPrice > threshold && newPrice <= threshold)) {
                System.out.println("🚨 ALERTA! " + symbol + 
                    " atingiu o preço alvo de $" + threshold + 
                    " (atual: $" + newPrice + ")");
                alertThresholds.remove(symbol); // Remove alerta após disparar
            }
        }
    }
}

// Concrete Observer - Analytics Dashboard
class MarketAnalytics implements StockObserver {
    private Map<String, List<Double>> priceHistory;
    
    public MarketAnalytics() {
        this.priceHistory = new HashMap<>();
    }
    
    @Override
    public void priceChanged(String symbol, double oldPrice, double newPrice) {
        priceHistory
            .computeIfAbsent(symbol, k -> new ArrayList<>())
            .add(newPrice);
        
        List<Double> history = priceHistory.get(symbol);
        if (history.size() >= 5) {
            double avg = history.stream()
                .skip(history.size() - 5)
                .mapToDouble(Double::doubleValue)
                .average()
                .orElse(0.0);
            
            System.out.printf("📊 Analytics: %s - Média últimos 5: $%.2f\n",
                symbol, avg);
        }
    }
}

// Uso
public class StockMarketExample {
    public static void main(String[] args) throws InterruptedException {
        StockMarket market = new StockMarket();
        
        System.out.println("=== Sistema de Monitoramento de Ações ===\n");
        
        // Adiciona ações
        market.addStock("AAPL", 150.00);
        market.addStock("GOOGL", 2800.00);
        market.addStock("MSFT", 300.00);
        
        // Cria observers
        Investor warren = new Investor("Warren", 100000.0);
        Investor elon = new Investor("Elon", 50000.0);
        
        PriceAlertSystem alertSystem = new PriceAlertSystem();
        MarketAnalytics analytics = new MarketAnalytics();
        
        // Inscrições
        System.out.println("\n--- Inscrições ---");
        market.subscribe("AAPL", warren);
        market.subscribe("AAPL", elon);
        market.subscribe("AAPL", alertSystem);
        market.subscribe("AAPL", analytics);
        
        market.subscribe("GOOGL", warren);
        market.subscribe("MSFT", elon);
        
        // Configura alertas
        alertSystem.setAlert("AAPL", 160.00);
        
        // Simulação de mudanças de preço
        System.out.println("\n--- Mudanças de Preço ---");
        
        Thread.sleep(100);
        market.updatePrice("AAPL", 157.50); // +5%
        
        Thread.sleep(100);
        market.updatePrice("AAPL", 163.00); // +3.5%
        
        Thread.sleep(100);
        market.updatePrice("AAPL", 154.85); // -5%
        
        Thread.sleep(100);
        market.updatePrice("GOOGL", 2940.00); // +5%
        
        Thread.sleep(100);
        market.updatePrice("MSFT", 315.00); // +5%
        
        // Portfólios finais
        System.out.println("\n--- Portfólios Finais ---");
        warren.showPortfolio(market);
        elon.showPortfolio(market);
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Mudanças em um objeto**: Afetam outros objetos desconhecidos
- **Objetos dinâmicos**: Set de observers muda em runtime
- **Notificações automáticas**: Mudanças devem notificar automaticamente
- **Acoplamento fraco**: Objetos não devem conhecer uns aos outros
- **Eventos**: Sistema orientado a eventos
- **Broadcast**: Um objeto notifica muitos

### 📝 Exemplos de aplicação:
- **GUI frameworks**: Event listeners (button clicks, etc)
- **Model-View**: MVC, MVVM patterns
- **Pub/Sub systems**: Message brokers, event buses
- **Reactive programming**: RxJava, Reactive Streams
- **Social media**: Followers/notifications
- **Stock market**: Price change notifications

### ❌ Evite quando:
- **Notificação rara**: Observers raramente precisam notificação
- **Performance crítica**: Overhead de notificar todos
- **Ordem importa**: Subscribers devem ser notificados em ordem específica

## 🚀 Como Implementar

1. **Divida lógica** em publisher (core) e subscribers (reações)

2. **Declare interface Subscriber** com método `update()`

3. **Declare interface Publisher** com subscribe/unsubscribe

4. **Decida onde colocar lista** de subscribers (geralmente no publisher)

5. **Crie concrete publishers** que notificam ao mudar estado

6. **Implemente update()** em concrete subscribers

7. **Cliente cria e registra** subscribers apropriados

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Open/Closed Principle**: Novos subscribers sem mudar publisher
- **Runtime relationships**: Estabelece relações em tempo de execução
- **Desacoplamento**: Publisher não conhece subscribers concretos
- **Broadcast**: Um evento notifica múltiplos objetos
- **Dinâmico**: Subscribe/unsubscribe a qualquer momento

### ❌ Desvantagens:
- **Ordem aleatória**: Subscribers notificados em ordem não determinística
- **Memory leaks**: Esquecer de unsubscribe pode causar vazamentos
- **Performance**: Notificar muitos subscribers pode ser custoso
- **Cascata**: Notificações podem gerar outras notificações (cuidado!)

## 🔗 Diferenças de Outros Padrões

| Padrão | Comunicação | Direção | Conhecimento |
|--------|-------------|---------|--------------|
| **Observer** | Pub/Sub assíncrono | Um-para-muitos | Publisher não conhece subscribers |
| **Mediator** | Centralizada via mediador | Muitos-para-muitos | Mediador conhece todos |
| **Command** | Encapsula requisição | Um-para-um | Comando conhece receptor |
| **Chain of Responsibility** | Sequencial em cadeia | Um-para-um | Só conhece próximo |

## 🔗 Relações com Outros Padrões

- **Mediator vs Observer**:
  - Mediator: Elimina dependências mútuas, comunicação bidirecional
  - Observer: Dependência unidirecional, comunicação um-para-muitos
  - Mediator pode usar Observer internamente

- **Chain of Responsibility, Command, Mediator, Observer**:
  - Chain: Passa requisição sequencialmente
  - Command: Conexão unidirecional
  - Mediator: Comunicação indireta via mediador
  - Observer: Subscrição dinâmica

## 📚 Conceitos-Chave para Lembrar

1. **Pub/Sub**: Publisher publica, subscribers se inscrevem
2. **Desacoplamento**: Publisher não conhece subscribers concretamente
3. **Dinâmico**: Subscribe/unsubscribe em runtime
4. **Notificação automática**: Mudanças notificam automaticamente
5. **Um-para-muitos**: Um publisher, múltiplos subscribers
6. **Interface uniforme**: Todos subscribers implementam mesma interface

## 🔍 Analogia do Mundo Real

**Assinatura de revista/jornal**: Quando você assina uma revista, não precisa ir à banca todo dia verificar se saiu nova edição. A editora (publisher) mantém lista de assinantes (subscribers) e envia automaticamente cada nova edição para sua caixa de correio. Você pode cancelar a assinatura (unsubscribe) quando quiser. A editora não precisa conhecer detalhes sobre cada assinante - apenas precisa do endereço de entrega e usa a interface uniforme do correio.

## ⚠️ Considerações Importantes

### Push vs Pull:
```java
// PUSH: Publisher envia dados
interface Observer {
    void update(String data); // Dados vêm do publisher
}

void notifyObservers() {
    for (Observer o : observers) {
        o.update(this.data); // Push
    }
}

// PULL: Observer busca dados
interface Observer {
    void update(Publisher publisher); // Recebe referência
}

void notifyObservers() {
    for (Observer o : observers) {
        o.update(this); // Observer puxa dados se necessário
    }
}

class ConcreteObserver implements Observer {
    public void update(Publisher pub) {
        String data = pub.getData(); // Pull
    }
}
```

### Prevenindo memory leaks:
```java
// ❌ Problema: Esquecer de unsubscribe
class Window {
    void close() {
        // VAZAMENTO! Observer ainda registrado
        dispose();
    }
}

// ✅ Solução: Sempre unsubscribe
class Window {
    private Observer myObserver;
    
    void init() {
        myObserver = new MyObserver();
        publisher.subscribe(myObserver);
    }
    
    void close() {
        publisher.unsubscribe(myObserver); // Limpa!
        dispose();
    }
}

// ✅ Solução: WeakReference
class Publisher {
    private List<WeakReference<Observer>> observers;
    
    void notifyObservers() {
        observers.removeIf(ref -> ref.get() == null); // Limpa automaticamente
        for (WeakReference<Observer> ref : observers) {
            Observer o = ref.get();
            if (o != null) {
                o.update(this);
            }
        }
    }
}
```

### Event types/categories:
```java
// Subscribers por tipo de evento
class EventManager {
    private Map<String, List<Observer>> subscribers;
    
    void subscribe(String eventType, Observer observer) {
        subscribers
            .computeIfAbsent(eventType, k -> new ArrayList<>())
            .add(observer);
    }
    
    void notify(String eventType, Object data) {
        List<Observer> eventSubscribers = subscribers.get(eventType);
        if (eventSubscribers != null) {
            for (Observer o : eventSubscribers) {
                o.update(data);
            }
        }
    }
}

// Uso
eventManager.subscribe("user.login", loginLogger);
eventManager.subscribe("user.login", analyticsTracker);
eventManager.subscribe("user.logout", sessionManager);

eventManager.notify("user.login", userData);
```

### Design Guidelines:
- **Weak references**: Considere para evitar memory leaks
- **Thread safety**: Sincronize se multi-threaded
- **Event types**: Categorize eventos para granularidade
- **Push vs Pull**: Push para dados pequenos, Pull para grandes
- **Unsubscribe**: Sempre limpe observers quando não mais necessários
- **Exception handling**: Um observer falhando não deve afetar outros

---

> **💡 Dica de Estudo:** Observer é como assinatura de Netflix - você se inscreve (subscribe) e recebe notificações automáticas quando novos episódios chegam, sem precisar ficar checando manualmente. Pode cancelar (unsubscribe) quando quiser. A Netflix não precisa conhecer detalhes de cada assinante, apenas envia notificações para todos da lista!

> **📖 Referência:** [Refactoring Guru - Observer](https://refactoring.guru/design-patterns/observer)
