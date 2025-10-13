# State

## 🎯 Intenção

O State é um padrão de projeto comportamental que permite um objeto alterar seu comportamento quando seu estado interno muda. Parece como se o objeto mudasse sua classe. O padrão está intimamente relacionado ao conceito de **Máquina de Estados Finitos** (Finite-State Machine).

## 🚩 Problema

O padrão State está relacionado ao conceito de uma Máquina de Estados Finitos. A ideia principal é que, em qualquer momento, há um número finito de estados nos quais um programa pode estar. Dentro de cada estado único, o programa se comporta de forma diferente, e pode mudar instantaneamente de um estado para outro.

### Resultado problemático:
```java
// Documento com estados usando condicionais
class Document {
    private String state; // "draft", "moderation", "published"
    private String content;
    private User currentUser;
    
    public void publish() {
        switch (state) {
            case "draft":
                state = "moderation";
                System.out.println("Documento enviado para moderação");
                break;
                
            case "moderation":
                if (currentUser.isAdmin()) {
                    state = "published";
                    System.out.println("Documento publicado!");
                } else {
                    System.out.println("Apenas admin pode publicar");
                }
                break;
                
            case "published":
                System.out.println("Já está publicado");
                break;
        }
    }
    
    public void edit() {
        switch (state) {
            case "draft":
                // Permite edição
                content = "novo conteúdo";
                break;
                
            case "moderation":
                // Não permite
                System.out.println("Em moderação, não pode editar");
                break;
                
            case "published":
                // Não permite
                System.out.println("Publicado, não pode editar");
                break;
        }
    }
    
    // Mais métodos com switches gigantes...
    public void requestReview() { /* switch state... */ }
    public void approve() { /* switch state... */ }
    public void reject() { /* switch state... */ }
}
```

**Problemas:**
- **Condicionais monstruosas**: Cada método tem switch/if gigante
- **Difícil manutenção**: Adicionar estado requer mudar todos os métodos
- **Lógica espalhada**: Comportamento de um estado está em vários lugares
- **Crescimento caótico**: Estados e transições crescem sem controle
- **Difícil teste**: Testar todas combinações de estado/ação
- **Violação SRP**: Classe faz demais

## ✅ Solução

O padrão State sugere que você crie novas classes para todos os estados possíveis de um objeto e extraia todos os comportamentos específicos do estado para essas classes. Ao invés de implementar todos os comportamentos por conta própria, o objeto original, chamado **context**, armazena uma referência a um dos objetos de estado que representa seu estado atual, e delega todo o trabalho relacionado ao estado para esse objeto.

### Características-chave:
- **Objeto de estado**: Cada estado é uma classe separada
- **Context**: Mantém referência ao estado atual
- **Delegação**: Context delega comportamento ao estado
- **Transição**: Estados podem trocar o estado do context
- **Interface comum**: Todos estados implementam mesma interface
- **Eliminação de condicionais**: Substituídos por polimorfismo

```java
// Interface State
interface DocumentState {
    void publish(Document doc);
    void edit(Document doc, String content);
    void requestReview(Document doc);
}

// Context
class Document {
    private DocumentState state;
    private String content;
    
    public Document() {
        this.state = new DraftState(); // Estado inicial
    }
    
    // Context delega ao estado
    public void publish() {
        state.publish(this);
    }
    
    public void edit(String content) {
        state.edit(this, content);
    }
    
    // Permite estado trocar o estado
    public void setState(DocumentState state) {
        this.state = state;
        System.out.println("Estado mudou para: " + state.getClass().getSimpleName());
    }
    
    // Getters/setters
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
}

// Concrete State - Draft
class DraftState implements DocumentState {
    @Override
    public void publish(Document doc) {
        doc.setState(new ModerationState());
    }
    
    @Override
    public void edit(Document doc, String content) {
        doc.setContent(content);
        System.out.println("Conteúdo editado");
    }
    
    @Override
    public void requestReview(Document doc) {
        doc.setState(new ModerationState());
    }
}

// Concrete State - Moderation
class ModerationState implements DocumentState {
    @Override
    public void publish(Document doc) {
        // Verifica permissão (simplificado)
        doc.setState(new PublishedState());
    }
    
    @Override
    public void edit(Document doc, String content) {
        System.out.println("Não pode editar em moderação");
    }
    
    @Override
    public void requestReview(Document doc) {
        System.out.println("Já está em moderação");
    }
}

// Concrete State - Published
class PublishedState implements DocumentState {
    @Override
    public void publish(Document doc) {
        System.out.println("Já está publicado");
    }
    
    @Override
    public void edit(Document doc, String content) {
        System.out.println("Não pode editar publicado");
    }
    
    @Override
    public void requestReview(Document doc) {
        System.out.println("Já está publicado");
    }
}
```

## 🏗️ Estrutura

```
Context                          State (interface)
    ↓                                   ↓
[state: State]                     handle()
setState(state)                        ↑
request() ----→ state.handle()         |
                                       |
                    ConcreteStateA  ConcreteStateB
                         ↓               ↓
                    handle()        handle()
                    [pode mudar     [pode mudar
                     estado]         estado]
```

### Componentes:
- **Context**: Mantém referência ao estado atual, delega requisições
- **State**: Interface comum para todos estados concretos
- **ConcreteState**: Implementa comportamento específico do estado, pode trocar estado do context

## 💻 Exemplos Práticos

### Exemplo 1: Pedido de E-commerce com Ciclo de Vida

```java
// Interface State
interface OrderState {
    void payOrder(Order order);
    void shipOrder(Order order);
    void deliverOrder(Order order);
    void cancelOrder(Order order);
    String getStatus();
}

// Context
class Order {
    private String orderId;
    private double amount;
    private OrderState state;
    
    public Order(String orderId, double amount) {
        this.orderId = orderId;
        this.amount = amount;
        this.state = new NewOrderState(); // Estado inicial
        System.out.println("🛒 Pedido criado: " + orderId + " ($" + amount + ")");
    }
    
    // Delegação ao estado
    public void pay() {
        state.payOrder(this);
    }
    
    public void ship() {
        state.shipOrder(this);
    }
    
    public void deliver() {
        state.deliverOrder(this);
    }
    
    public void cancel() {
        state.cancelOrder(this);
    }
    
    // Mudança de estado
    public void setState(OrderState state) {
        this.state = state;
        System.out.println("   ➡️  Estado: " + state.getStatus());
    }
    
    // Getters
    public String getOrderId() { return orderId; }
    public double getAmount() { return amount; }
    public String getStatus() { return state.getStatus(); }
}

// Concrete State - New Order
class NewOrderState implements OrderState {
    @Override
    public void payOrder(Order order) {
        System.out.println("💳 Processando pagamento de $" + order.getAmount());
        order.setState(new PaidOrderState());
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("❌ Não pode enviar pedido não pago");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("❌ Não pode entregar pedido não pago");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("🚫 Pedido cancelado");
        order.setState(new CancelledOrderState());
    }
    
    @Override
    public String getStatus() {
        return "NOVO";
    }
}

// Concrete State - Paid
class PaidOrderState implements OrderState {
    @Override
    public void payOrder(Order order) {
        System.out.println("❌ Pedido já está pago");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("📦 Preparando envio...");
        order.setState(new ShippedOrderState());
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("❌ Não pode entregar antes de enviar");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("💰 Processando reembolso de $" + order.getAmount());
        order.setState(new CancelledOrderState());
    }
    
    @Override
    public String getStatus() {
        return "PAGO";
    }
}

// Concrete State - Shipped
class ShippedOrderState implements OrderState {
    @Override
    public void payOrder(Order order) {
        System.out.println("❌ Pedido já está pago");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("❌ Pedido já foi enviado");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("✅ Pedido entregue ao cliente!");
        order.setState(new DeliveredOrderState());
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("⚠️  Cancelando envio e processando reembolso");
        order.setState(new CancelledOrderState());
    }
    
    @Override
    public String getStatus() {
        return "ENVIADO";
    }
}

// Concrete State - Delivered
class DeliveredOrderState implements OrderState {
    @Override
    public void payOrder(Order order) {
        System.out.println("❌ Pedido já foi entregue e pago");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("❌ Pedido já foi entregue");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("❌ Pedido já foi entregue");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("❌ Não pode cancelar pedido entregue");
        System.out.println("   💡 Use o processo de devolução");
    }
    
    @Override
    public String getStatus() {
        return "ENTREGUE";
    }
}

// Concrete State - Cancelled
class CancelledOrderState implements OrderState {
    @Override
    public void payOrder(Order order) {
        System.out.println("❌ Pedido foi cancelado");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("❌ Pedido foi cancelado");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("❌ Pedido foi cancelado");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("❌ Pedido já está cancelado");
    }
    
    @Override
    public String getStatus() {
        return "CANCELADO";
    }
}

// Uso
public class OrderExample {
    public static void main(String[] args) {
        System.out.println("=== Sistema de Pedidos E-commerce ===\n");
        
        // Fluxo normal
        System.out.println("--- Fluxo Normal ---");
        Order order1 = new Order("ORD-001", 299.99);
        order1.pay();
        order1.ship();
        order1.deliver();
        
        // Tentativa de ação inválida
        System.out.println("\n--- Tentando cancelar pedido entregue ---");
        order1.cancel();
        
        // Cancelamento após pagamento
        System.out.println("\n--- Cancelamento após pagamento ---");
        Order order2 = new Order("ORD-002", 149.99);
        order2.pay();
        order2.cancel();
        
        // Tentativa de enviar sem pagar
        System.out.println("\n--- Tentando enviar sem pagar ---");
        Order order3 = new Order("ORD-003", 89.99);
        order3.ship();
        order3.pay();
        order3.ship();
        
        // Cancelamento durante envio
        System.out.println("\n--- Cancelamento durante envio ---");
        Order order4 = new Order("ORD-004", 199.99);
        order4.pay();
        order4.ship();
        order4.cancel();
    }
}
```

### Exemplo 2: Máquina de Venda Automática (Vending Machine)

```java
// Interface State
interface VendingMachineState {
    void insertCoin(VendingMachine machine, double amount);
    void selectProduct(VendingMachine machine, String product);
    void dispense(VendingMachine machine);
    void cancel(VendingMachine machine);
    String getStateName();
}

// Context
class VendingMachine {
    private VendingMachineState state;
    private double balance;
    private Map<String, Product> inventory;
    private String selectedProduct;
    
    public VendingMachine() {
        this.state = new IdleState();
        this.balance = 0.0;
        this.inventory = new HashMap<>();
        initializeInventory();
        System.out.println("🏪 Máquina de venda inicializada");
    }
    
    private void initializeInventory() {
        inventory.put("A1", new Product("Coca-Cola", 3.50, 10));
        inventory.put("A2", new Product("Água", 2.00, 15));
        inventory.put("B1", new Product("Snickers", 4.00, 8));
        inventory.put("B2", new Product("Doritos", 5.50, 12));
        inventory.put("C1", new Product("KitKat", 3.00, 20));
    }
    
    // Delegação ao estado
    public void insertCoin(double amount) {
        state.insertCoin(this, amount);
    }
    
    public void selectProduct(String productCode) {
        state.selectProduct(this, productCode);
    }
    
    public void dispense() {
        state.dispense(this);
    }
    
    public void cancel() {
        state.cancel(this);
    }
    
    // Mudança de estado
    public void setState(VendingMachineState state) {
        this.state = state;
        System.out.println("   🔄 Estado: " + state.getStateName());
    }
    
    // Operações internas
    public void addBalance(double amount) {
        this.balance += amount;
    }
    
    public void returnChange() {
        if (balance > 0) {
            System.out.println("💰 Devolvendo troco: $" + String.format("%.2f", balance));
            this.balance = 0.0;
        }
    }
    
    public void resetSelection() {
        this.selectedProduct = null;
        this.balance = 0.0;
    }
    
    // Getters e setters
    public double getBalance() { return balance; }
    public void setBalance(double balance) { this.balance = balance; }
    public Map<String, Product> getInventory() { return inventory; }
    public String getSelectedProduct() { return selectedProduct; }
    public void setSelectedProduct(String productCode) { this.selectedProduct = productCode; }
    
    public void showStatus() {
        System.out.println("\n📊 Status da Máquina:");
        System.out.println("   Estado: " + state.getStateName());
        System.out.println("   Saldo: $" + String.format("%.2f", balance));
        if (selectedProduct != null) {
            Product p = inventory.get(selectedProduct);
            System.out.println("   Produto selecionado: " + p.getName());
        }
    }
}

// Produto
class Product {
    private String name;
    private double price;
    private int quantity;
    
    public Product(String name, double price, int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public void decrementQuantity() { this.quantity--; }
}

// Concrete State - Idle (Aguardando)
class IdleState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, double amount) {
        System.out.println("💵 Moeda inserida: $" + String.format("%.2f", amount));
        machine.addBalance(amount);
        machine.setState(new HasMoneyState());
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("❌ Insira dinheiro primeiro");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("❌ Nenhum produto selecionado");
    }
    
    @Override
    public void cancel(VendingMachine machine) {
        System.out.println("❌ Nada para cancelar");
    }
    
    @Override
    public String getStateName() {
        return "AGUARDANDO";
    }
}

// Concrete State - Has Money
class HasMoneyState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, double amount) {
        System.out.println("💵 Moeda adicional inserida: $" + String.format("%.2f", amount));
        machine.addBalance(amount);
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String productCode) {
        Product product = machine.getInventory().get(productCode);
        
        if (product == null) {
            System.out.println("❌ Código inválido: " + productCode);
            return;
        }
        
        if (product.getQuantity() == 0) {
            System.out.println("❌ " + product.getName() + " está esgotado");
            return;
        }
        
        if (machine.getBalance() < product.getPrice()) {
            double missing = product.getPrice() - machine.getBalance();
            System.out.println("❌ Saldo insuficiente. Faltam $" + 
                             String.format("%.2f", missing));
            return;
        }
        
        System.out.println("✅ Produto selecionado: " + product.getName() + 
                         " ($" + String.format("%.2f", product.getPrice()) + ")");
        machine.setSelectedProduct(productCode);
        machine.setState(new DispensingState());
        machine.dispense();
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("❌ Selecione um produto primeiro");
    }
    
    @Override
    public void cancel(VendingMachine machine) {
        System.out.println("🚫 Compra cancelada");
        machine.returnChange();
        machine.resetSelection();
        machine.setState(new IdleState());
    }
    
    @Override
    public String getStateName() {
        return "PRONTO PARA SELEÇÃO";
    }
}

// Concrete State - Dispensing
class DispensingState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, double amount) {
        System.out.println("❌ Aguarde, dispensando produto...");
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("❌ Aguarde, dispensando produto...");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        String productCode = machine.getSelectedProduct();
        Product product = machine.getInventory().get(productCode);
        
        // Debita do saldo
        double newBalance = machine.getBalance() - product.getPrice();
        machine.setBalance(newBalance);
        
        // Remove do estoque
        product.decrementQuantity();
        
        // Dispensa
        System.out.println("📤 Dispensando: " + product.getName());
        System.out.println("✅ Aproveite seu " + product.getName() + "!");
        
        // Retorna troco se houver
        if (machine.getBalance() > 0) {
            machine.returnChange();
        }
        
        // Reset e volta ao idle
        machine.resetSelection();
        machine.setState(new IdleState());
    }
    
    @Override
    public void cancel(VendingMachine machine) {
        System.out.println("❌ Não pode cancelar durante dispensa");
    }
    
    @Override
    public String getStateName() {
        return "DISPENSANDO";
    }
}

// Uso
public class VendingMachineExample {
    public static void main(String[] args) throws InterruptedException {
        VendingMachine machine = new VendingMachine();
        
        System.out.println("\n=== Máquina de Venda Automática ===\n");
        
        // Compra bem-sucedida
        System.out.println("--- Compra 1: Coca-Cola ---");
        machine.insertCoin(2.00);
        machine.insertCoin(2.00);
        machine.selectProduct("A1");
        Thread.sleep(100);
        
        // Tentativa sem dinheiro
        System.out.println("\n--- Tentativa sem dinheiro ---");
        machine.selectProduct("B1");
        
        // Compra com troco
        System.out.println("\n--- Compra 2: Água (com troco) ---");
        machine.insertCoin(5.00);
        machine.selectProduct("A2");
        Thread.sleep(100);
        
        // Cancelamento
        System.out.println("\n--- Cancelamento ---");
        machine.insertCoin(10.00);
        machine.showStatus();
        machine.cancel();
        Thread.sleep(100);
        
        // Saldo insuficiente
        System.out.println("\n--- Saldo insuficiente ---");
        machine.insertCoin(3.00);
        machine.selectProduct("B2"); // Doritos custa $5.50
        machine.insertCoin(3.00);
        machine.selectProduct("B2");
        Thread.sleep(100);
        
        // Produto esgotado (simulação)
        System.out.println("\n--- Produto esgotado ---");
        Product kitkat = machine.getInventory().get("C1");
        int originalQty = kitkat.getQuantity();
        // Zera estoque temporariamente
        while (kitkat.getQuantity() > 0) {
            kitkat.decrementQuantity();
        }
        machine.insertCoin(5.00);
        machine.selectProduct("C1");
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Comportamento varia por estado**: Objeto age diferente conforme estado
- **Muitos estados**: Número enorme de estados
- **Mudanças frequentes**: Lógica de estado muda com frequência
- **Condicionais massivas**: Classes poluídas com switch/if
- **Código duplicado**: Similar em estados/transições
- **Máquina de estados**: Implementar FSM

### 📝 Exemplos de aplicação:
- **Order processing**: Estados do pedido (novo, pago, enviado, entregue)
- **Document workflow**: Rascunho, revisão, aprovado, publicado
- **Connection management**: Conectado, desconectado, reconectando
- **Game states**: Menu, playing, paused, game over
- **TCP connection**: Listen, established, closed
- **UI components**: Enabled, disabled, hidden, focused

### ❌ Evite quando:
- **Poucos estados**: Simples if/else é suficiente
- **Estados raramente mudam**: Overhead desnecessário
- **Lógica de estado simples**: Complexidade não justificada

## 🚀 Como Implementar

1. **Decida qual classe será Context** (que tem comportamento dependente de estado)

2. **Declare interface State** com métodos que representam ações

3. **Crie classe para cada estado** implementando a interface

4. **Extraia código específico** de cada estado das condicionais

5. **Adicione campo de referência State** no Context

6. **Substitua condicionais** por chamadas aos métodos do estado

7. **Crie instâncias de estado** e passe ao Context (Context ou estados podem fazer isso)

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Single Responsibility**: Organiza código relacionado a estados
- **Open/Closed**: Novos estados sem mudar existentes
- **Elimina condicionais**: Substitui switch/if por polimorfismo
- **Simplifica Context**: Remove lógica complexa
- **Transições claras**: Estados controlam transições explicitamente

### ❌ Desvantagens:
- **Overkill**: Para poucos estados pode ser excessivo
- **Muitas classes**: Uma classe para cada estado
- **Complexidade inicial**: Setup mais complexo que if/else

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Mudança | Conhecimento entre objetos |
|--------|------|---------|----------------------------|
| **State** | Comportamento por estado | Estado muda comportamento | Estados conhecem uns aos outros |
| **Strategy** | Algoritmo intercambiável | Cliente escolhe estratégia | Estratégias independentes |
| **Command** | Encapsula requisição | Parametriza ação | Commands independentes |
| **Chain of Responsibility** | Sequência de handlers | Handler passa requisição | Só conhece próximo |

## 🔗 Relações com Outros Padrões

- **Bridge, State, Strategy** (e Adapter):
  - Estrutura similar (composição/delegação)
  - Resolvem problemas diferentes
  - State: Comportamento depende de estado interno
  - Strategy: Cliente escolhe algoritmo externamente
  - Bridge: Separa abstração de implementação

- **State vs Strategy**:
  - State: Estados conhecem uns aos outros, podem trocar estado
  - Strategy: Estratégias completamente independentes
  - State pode ser visto como extensão de Strategy

## 📚 Conceitos-Chave para Lembrar

1. **FSM**: Baseado em Máquina de Estados Finitos
2. **Delegação**: Context delega comportamento ao estado
3. **Eliminação de condicionais**: Substitui switch/if por polimorfismo
4. **Transições**: Estados podem mudar o estado do Context
5. **Interface comum**: Todos estados implementam mesma interface
6. **Estado atual**: Context mantém referência ao estado ativo

## 🔍 Analogia do Mundo Real

**Smartphone**: Os botões do seu smartphone se comportam diferentemente dependendo do estado atual do aparelho. Quando desbloqueado, apertar botões executa várias funções. Quando bloqueado, qualquer botão leva à tela de desbloqueio. Quando a bateria está baixa, qualquer botão mostra a tela de carregamento. O telefone "sabe" em qual estado está e delega o comportamento dos botões para esse estado específico.

## ⚠️ Considerações Importantes

### Quem controla transições?

```java
// Opção 1: Context controla transições
class Context {
    public void request() {
        if (someCondition) {
            state = new StateA();
        } else {
            state = new StateB();
        }
        state.handle();
    }
}

// Opção 2: Estados controlam transições (mais comum)
class StateA implements State {
    public void handle(Context ctx) {
        // Faz algo
        ctx.setState(new StateB()); // Muda estado
    }
}
```

### State com dados compartilhados:

```java
// Context mantém dados compartilhados
class Order {
    private OrderState state;
    private double amount;
    private String customerEmail;
    
    // Estados acessam dados via getters
    public double getAmount() { return amount; }
    public String getCustomerEmail() { return customerEmail; }
}

class PaidOrderState implements OrderState {
    public void ship(Order order) {
        // Acessa dados do context
        sendEmail(order.getCustomerEmail(), 
                  "Pedido de $" + order.getAmount() + " enviado");
    }
}
```

### Estados com hierarquia:

```java
// Estado base abstrato
abstract class OrderState {
    // Comportamento comum
    protected void logStateChange(Order order, String newState) {
        System.out.println("Order " + order.getId() + 
                         " changed to " + newState);
    }
    
    // Métodos abstratos
    abstract void pay(Order order);
    abstract void ship(Order order);
}

// Estados concretos herdam
class NewOrderState extends OrderState {
    @Override
    void pay(Order order) {
        logStateChange(order, "PAID");
        order.setState(new PaidOrderState());
    }
    
    @Override
    void ship(Order order) {
        System.out.println("Cannot ship unpaid order");
    }
}
```

### Singleton states:

```java
// Estados podem ser singleton se não mantêm dados
class IdleState implements State {
    private static IdleState instance;
    
    private IdleState() {}
    
    public static IdleState getInstance() {
        if (instance == null) {
            instance = new IdleState();
        }
        return instance;
    }
}

// Uso
context.setState(IdleState.getInstance());
```

### Design Guidelines:
- **Interface completa**: Todos métodos relevantes na interface State
- **Transições explícitas**: Deixe claro quais transições são válidas
- **Context mínimo**: Context não deve ter lógica de estado
- **Estados imutáveis**: Se possível, use singleton
- **Nomenclatura clara**: Nomes de estados óbvios (NewOrder, PaidOrder)
- **Documentação**: Diagrama de estados ajuda muito

---

> **💡 Dica de Estudo:** State é como um semáforo - o mesmo "objeto" (luz) se comporta diferente em cada estado (vermelho/amarelo/verde). Quando vermelho, carros param; quando verde, carros passam. O semáforo muda de estado automaticamente, e cada estado sabe para qual outro pode transitar. Não precisa de switch/if gigante!

> **📖 Referência:** [Refactoring Guru - State](https://refactoring.guru/design-patterns/state)
