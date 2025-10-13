# Template Method

## 🎯 Intenção

O Template Method é um padrão de projeto comportamental que define o **esqueleto de um algoritmo** na superclasse, mas permite que as subclasses sobrescrevam etapas específicas do algoritmo sem alterar sua estrutura. O algoritmo permanece intacto enquanto suas partes podem ser customizadas.

## 🚩 Problema

Imagine que você está criando uma aplicação de mineração de dados que analisa documentos corporativos. Os usuários fornecem documentos em vários formatos (PDF, DOC, CSV), e o app precisa extrair dados significativos em um formato uniforme.

### Resultado problemático:
```java
// Classe para processar documentos DOC
class DOCDataMiner {
    public void mine(String path) {
        // Abre arquivo DOC
        File file = openDOCFile(path);
        
        // Extrai dados brutos
        String rawData = extractDOCData(file);
        
        // Analisa dados (CÓDIGO DUPLICADO!)
        Data parsedData = parseData(rawData);
        
        // Processa dados (CÓDIGO DUPLICADO!)
        processData(parsedData);
        
        // Cria relatório (CÓDIGO DUPLICADO!)
        Report report = generateReport(parsedData);
        
        // Fecha arquivo DOC
        closeDOCFile(file);
    }
    
    private File openDOCFile(String path) { /* ... */ }
    private String extractDOCData(File file) { /* ... */ }
    private void closeDOCFile(File file) { /* ... */ }
    
    // Métodos duplicados em todas as classes!
    private Data parseData(String raw) { /* ... */ }
    private void processData(Data data) { /* ... */ }
    private Report generateReport(Data data) { /* ... */ }
}

// Classe para processar documentos CSV
class CSVDataMiner {
    public void mine(String path) {
        File file = openCSVFile(path);
        String rawData = extractCSVData(file);
        
        // MESMO CÓDIGO de DOCDataMiner! 😱
        Data parsedData = parseData(rawData);
        processData(parsedData);
        Report report = generateReport(parsedData);
        
        closeCSVFile(file);
    }
    
    private File openCSVFile(String path) { /* ... */ }
    private String extractCSVData(File file) { /* ... */ }
    private void closeCSVFile(File file) { /* ... */ }
    
    // CÓDIGO DUPLICADO! 😱
    private Data parseData(String raw) { /* ... */ }
    private void processData(Data data) { /* ... */ }
    private Report generateReport(Data data) { /* ... */ }
}

// Classe para processar documentos PDF
class PDFDataMiner {
    public void mine(String path) {
        File file = openPDFFile(path);
        String rawData = extractPDFData(file);
        
        // MESMO CÓDIGO NOVAMENTE! 😱
        Data parsedData = parseData(rawData);
        processData(parsedData);
        Report report = generateReport(parsedData);
        
        closePDFFile(file);
    }
    
    // Mais código duplicado...
}
```

**Problemas:**
- **Duplicação massiva**: `parseData()`, `processData()`, `generateReport()` copiados em todas classes
- **Difícil manutenção**: Bug fix requer mudança em 3+ lugares
- **Código cliente complexo**: Precisa de condicionais para escolher classe certa
- **Não escalável**: Adicionar XML, JSON, etc. multiplica duplicação
- **Violação DRY**: Don't Repeat Yourself completamente ignorado
- **Alta probabilidade de bugs**: Esquecer de atualizar uma das cópias

## ✅ Solução

O padrão Template Method sugere que você quebre o algoritmo em uma série de etapas, transforme essas etapas em métodos, e coloque uma série de chamadas a esses métodos dentro de um único **template method**. As etapas podem ser `abstract` (forçando implementação nas subclasses) ou ter implementação padrão (opcional de sobrescrever).

### Características-chave:
- **Esqueleto do algoritmo**: Template method define estrutura fixa
- **Etapas customizáveis**: Subclasses implementam etapas específicas
- **Inversão de controle**: Superclasse chama métodos das subclasses
- **Hooks opcionais**: Pontos de extensão com corpo vazio
- **Reuso de código**: Código comum na superclasse
- **Estrutura preservada**: Template method não deve ser sobrescrito

```java
// Abstract Class - Define o Template Method
abstract class DataMiner {
    
    // TEMPLATE METHOD - Define o esqueleto do algoritmo
    // final previne sobrescrita pelas subclasses
    public final void mine(String path) {
        File file = openFile(path);
        String rawData = extractData(file);
        Data parsedData = parseData(rawData);
        analyzeData(parsedData);
        Report report = generateReport(parsedData);
        closeFile(file);
        
        // Hook opcional
        if (shouldSendReport()) {
            sendReport(report);
        }
    }
    
    // Métodos abstratos - DEVEM ser implementados pelas subclasses
    protected abstract File openFile(String path);
    protected abstract String extractData(File file);
    protected abstract void closeFile(File file);
    
    // Métodos com implementação padrão - código comum
    protected Data parseData(String rawData) {
        System.out.println("📊 Analisando dados brutos...");
        // Lógica comum de parsing
        return new Data(rawData);
    }
    
    protected void analyzeData(Data data) {
        System.out.println("🔍 Processando dados...");
        // Lógica comum de análise
    }
    
    protected Report generateReport(Data data) {
        System.out.println("📝 Gerando relatório...");
        // Lógica comum de geração de relatório
        return new Report(data);
    }
    
    // Hook - Método opcional com implementação padrão vazia
    protected boolean shouldSendReport() {
        return true; // Comportamento padrão
    }
    
    protected void sendReport(Report report) {
        System.out.println("📧 Enviando relatório...");
    }
}

// Concrete Class - DOC
class DOCDataMiner extends DataMiner {
    @Override
    protected File openFile(String path) {
        System.out.println("📄 Abrindo arquivo DOC: " + path);
        return new File(path);
    }
    
    @Override
    protected String extractData(File file) {
        System.out.println("📄 Extraindo dados do DOC...");
        return "doc_raw_data";
    }
    
    @Override
    protected void closeFile(File file) {
        System.out.println("📄 Fechando arquivo DOC");
    }
}

// Concrete Class - CSV
class CSVDataMiner extends DataMiner {
    @Override
    protected File openFile(String path) {
        System.out.println("📊 Abrindo arquivo CSV: " + path);
        return new File(path);
    }
    
    @Override
    protected String extractData(File file) {
        System.out.println("📊 Extraindo dados do CSV...");
        return "csv_raw_data";
    }
    
    @Override
    protected void closeFile(File file) {
        System.out.println("📊 Fechando arquivo CSV");
    }
    
    // Sobrescreve hook para customizar comportamento
    @Override
    protected boolean shouldSendReport() {
        return false; // CSV não envia relatório
    }
}

// Uso
DataMiner docMiner = new DOCDataMiner();
docMiner.mine("report.doc");

DataMiner csvMiner = new CSVDataMiner();
csvMiner.mine("data.csv");
```

## 🏗️ Estrutura

```
AbstractClass
    ↓
templateMethod() [final]       ← Chama os métodos na ordem certa
    ├─→ primitiveOperation1()  ← abstract
    ├─→ primitiveOperation2()  ← abstract
    ├─→ commonOperation()      ← implementação padrão
    └─→ hook()                 ← opcional (vazio ou padrão)
    ↑
ConcreteClassA              ConcreteClassB
    ↓                           ↓
primitiveOperation1()      primitiveOperation1()
primitiveOperation2()      primitiveOperation2()
hook()                     [usa padrão]
```

### Componentes:
- **AbstractClass**: Define template method e declara etapas (abstratas ou com implementação)
- **ConcreteClass**: Implementa etapas abstratas, pode sobrescrever métodos opcionais
- **Template Method**: Método final que orquestra chamada das etapas
- **Primitive Operations**: Métodos abstratos que subclasses devem implementar
- **Hooks**: Métodos opcionais com corpo vazio ou implementação padrão

### Tipos de métodos:
- **Abstract steps**: Devem ser implementados por toda subclasse
- **Optional steps**: Já têm implementação padrão, mas podem ser sobrescritos
- **Hooks**: Opcionais com corpo vazio, pontos de extensão
- **Template method**: Não deve ser sobrescrito (final)

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Processamento de Pedidos E-commerce

```java
// Abstract Class
abstract class OrderProcessor {
    
    // TEMPLATE METHOD - Esqueleto do processamento de pedido
    public final void processOrder(Order order) {
        System.out.println("🛒 Iniciando processamento do pedido #" + order.getId());
        System.out.println("════════════════════════════════════");
        
        // Etapa 1: Validação
        if (!validateOrder(order)) {
            System.out.println("❌ Pedido inválido!");
            return;
        }
        
        // Etapa 2: Calcular total
        double total = calculateTotal(order);
        System.out.println("💰 Total: $" + String.format("%.2f", total));
        
        // Etapa 3: Processar pagamento
        if (!processPayment(order, total)) {
            System.out.println("❌ Falha no pagamento!");
            return;
        }
        
        // Etapa 4: Atualizar inventário
        updateInventory(order);
        
        // Etapa 5: Hook - Ações específicas do tipo de pedido
        performSpecificActions(order);
        
        // Etapa 6: Enviar notificações
        sendNotifications(order);
        
        // Etapa 7: Hook - Logging específico
        logOrderDetails(order);
        
        System.out.println("════════════════════════════════════");
        System.out.println("✅ Pedido processado com sucesso!\n");
    }
    
    // Método com implementação padrão - pode ser sobrescrito
    protected boolean validateOrder(Order order) {
        System.out.println("🔍 Validando pedido...");
        
        if (order.getItems().isEmpty()) {
            System.out.println("   ❌ Pedido vazio");
            return false;
        }
        
        for (OrderItem item : order.getItems()) {
            if (item.getQuantity() <= 0) {
                System.out.println("   ❌ Quantidade inválida para " + item.getProduct());
                return false;
            }
        }
        
        System.out.println("   ✅ Pedido válido");
        return true;
    }
    
    // Método com implementação padrão
    protected double calculateTotal(Order order) {
        System.out.println("🧮 Calculando total...");
        
        double subtotal = order.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
        
        double discount = applyDiscount(order, subtotal);
        double shipping = calculateShipping(order);
        double tax = calculateTax(subtotal - discount);
        
        double total = subtotal - discount + shipping + tax;
        
        System.out.println("   Subtotal: $" + String.format("%.2f", subtotal));
        if (discount > 0) {
            System.out.println("   Desconto: -$" + String.format("%.2f", discount));
        }
        System.out.println("   Frete: $" + String.format("%.2f", shipping));
        System.out.println("   Impostos: $" + String.format("%.2f", tax));
        
        return total;
    }
    
    // Método abstrato - DEVE ser implementado
    protected abstract boolean processPayment(Order order, double amount);
    
    // Método com implementação padrão
    protected void updateInventory(Order order) {
        System.out.println("📦 Atualizando inventário...");
        for (OrderItem item : order.getItems()) {
            System.out.println("   • " + item.getProduct() + ": -" + item.getQuantity());
        }
    }
    
    // Hook - implementação vazia, pode ser sobrescrito
    protected void performSpecificActions(Order order) {
        // Subclasses podem adicionar ações específicas
    }
    
    // Método com implementação padrão
    protected void sendNotifications(Order order) {
        System.out.println("📧 Enviando notificações...");
        System.out.println("   • Email para: " + order.getCustomerEmail());
        System.out.println("   • SMS para: " + order.getCustomerPhone());
    }
    
    // Hook - pode ser sobrescrito para logging específico
    protected void logOrderDetails(Order order) {
        System.out.println("📝 Registrando no log padrão...");
    }
    
    // Métodos auxiliares com implementação padrão
    protected double applyDiscount(Order order, double subtotal) {
        return 0; // Sem desconto por padrão
    }
    
    protected abstract double calculateShipping(Order order);
    protected abstract double calculateTax(double amount);
}

// Concrete Class - Pedido Online (Nacional)
class OnlineOrderProcessor extends OrderProcessor {
    
    @Override
    protected boolean processPayment(Order order, double amount) {
        System.out.println("💳 Processando pagamento online...");
        System.out.println("   Método: Cartão de Crédito");
        System.out.println("   ✅ Pagamento aprovado");
        return true;
    }
    
    @Override
    protected double calculateShipping(Order order) {
        // Frete calculado por região
        String region = order.getShippingAddress().getRegion();
        
        if (region.equals("Sul")) {
            return 15.00;
        } else if (region.equals("Sudeste")) {
            return 20.00;
        } else {
            return 30.00;
        }
    }
    
    @Override
    protected double calculateTax(double amount) {
        return amount * 0.15; // 15% de impostos
    }
    
    @Override
    protected double applyDiscount(Order order, double subtotal) {
        // Desconto de 10% para pedidos acima de $200
        if (subtotal > 200) {
            System.out.println("🎉 Desconto de 10% aplicado!");
            return subtotal * 0.10;
        }
        return 0;
    }
}

// Concrete Class - Pedido Internacional
class InternationalOrderProcessor extends OrderProcessor {
    
    @Override
    protected boolean validateOrder(Order order) {
        // Validação adicional para pedidos internacionais
        System.out.println("🔍 Validando pedido internacional...");
        
        if (!super.validateOrder(order)) {
            return false;
        }
        
        // Verifica restrições de exportação
        for (OrderItem item : order.getItems()) {
            if (item.getProduct().contains("Bateria")) {
                System.out.println("   ❌ Produto com restrição de exportação");
                return false;
            }
        }
        
        System.out.println("   ✅ Pedido internacional válido");
        return true;
    }
    
    @Override
    protected boolean processPayment(Order order, double amount) {
        System.out.println("💳 Processando pagamento internacional...");
        System.out.println("   Método: PayPal Internacional");
        System.out.println("   Conversão de moeda aplicada");
        System.out.println("   ✅ Pagamento aprovado");
        return true;
    }
    
    @Override
    protected double calculateShipping(Order order) {
        // Frete internacional fixo + peso
        double baseShipping = 50.00;
        int totalItems = order.getItems().stream()
            .mapToInt(OrderItem::getQuantity)
            .sum();
        return baseShipping + (totalItems * 10.00);
    }
    
    @Override
    protected double calculateTax(double amount) {
        return amount * 0.25; // 25% impostos internacionais
    }
    
    @Override
    protected void performSpecificActions(Order order) {
        System.out.println("🌎 Ações específicas internacionais:");
        System.out.println("   • Gerando documentos alfandegários");
        System.out.println("   • Registrando na Receita Federal");
        System.out.println("   • Calculando seguro de transporte");
    }
    
    @Override
    protected void logOrderDetails(Order order) {
        System.out.println("📝 Registrando em sistema internacional...");
        System.out.println("   • Log país destino: " + order.getShippingAddress().getCountry());
    }
}

// Concrete Class - Pedido Corporativo (B2B)
class CorporateOrderProcessor extends OrderProcessor {
    
    @Override
    protected boolean validateOrder(Order order) {
        System.out.println("🔍 Validando pedido corporativo...");
        
        if (!super.validateOrder(order)) {
            return false;
        }
        
        // Verifica CNPJ
        if (order.getCorporateId() == null || order.getCorporateId().isEmpty()) {
            System.out.println("   ❌ CNPJ não fornecido");
            return false;
        }
        
        // Pedidos corporativos devem ter valor mínimo
        double total = order.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
        
        if (total < 500) {
            System.out.println("   ❌ Pedido abaixo do valor mínimo ($500)");
            return false;
        }
        
        System.out.println("   ✅ Pedido corporativo válido");
        return true;
    }
    
    @Override
    protected boolean processPayment(Order order, double amount) {
        System.out.println("💼 Processando pagamento corporativo...");
        System.out.println("   Método: Boleto Bancário (30 dias)");
        System.out.println("   CNPJ: " + order.getCorporateId());
        System.out.println("   ✅ Boleto gerado");
        return true;
    }
    
    @Override
    protected double calculateShipping(Order order) {
        // Frete grátis para pedidos corporativos
        return 0.00;
    }
    
    @Override
    protected double calculateTax(double amount) {
        return amount * 0.10; // 10% impostos (nota fiscal jurídica)
    }
    
    @Override
    protected double applyDiscount(Order order, double subtotal) {
        // Desconto progressivo corporativo
        if (subtotal > 2000) {
            System.out.println("🎉 Desconto corporativo de 20%!");
            return subtotal * 0.20;
        } else if (subtotal > 1000) {
            System.out.println("🎉 Desconto corporativo de 15%!");
            return subtotal * 0.15;
        }
        return subtotal * 0.10; // Mínimo 10%
    }
    
    @Override
    protected void performSpecificActions(Order order) {
        System.out.println("💼 Ações específicas corporativas:");
        System.out.println("   • Gerando nota fiscal eletrônica");
        System.out.println("   • Emitindo boleto bancário");
        System.out.println("   • Notificando gerente de conta");
    }
    
    @Override
    protected void sendNotifications(Order order) {
        System.out.println("📧 Enviando notificações corporativas...");
        System.out.println("   • Email para: " + order.getCustomerEmail());
        System.out.println("   • Email para setor financeiro");
        System.out.println("   • Email para gerente de conta");
    }
    
    @Override
    protected void logOrderDetails(Order order) {
        System.out.println("📝 Registrando em sistema corporativo ERP...");
        System.out.println("   • CNPJ: " + order.getCorporateId());
        System.out.println("   • Centro de custo registrado");
    }
}

// Classes auxiliares
class Order {
    private String id;
    private List<OrderItem> items;
    private String customerEmail;
    private String customerPhone;
    private Address shippingAddress;
    private String corporateId; // CNPJ para pedidos corporativos
    
    // Getters
    public String getId() { return id; }
    public List<OrderItem> getItems() { return items; }
    public String getCustomerEmail() { return customerEmail; }
    public String getCustomerPhone() { return customerPhone; }
    public Address getShippingAddress() { return shippingAddress; }
    public String getCorporateId() { return corporateId; }
    
    // Constructor
    public Order(String id, String email, String phone, Address address) {
        this.id = id;
        this.customerEmail = email;
        this.customerPhone = phone;
        this.shippingAddress = address;
        this.items = new ArrayList<>();
    }
    
    public void setCorporateId(String cnpj) {
        this.corporateId = cnpj;
    }
    
    public void addItem(OrderItem item) {
        items.add(item);
    }
}

class OrderItem {
    private String product;
    private int quantity;
    private double price;
    
    public OrderItem(String product, int quantity, double price) {
        this.product = product;
        this.quantity = quantity;
        this.price = price;
    }
    
    public String getProduct() { return product; }
    public int getQuantity() { return quantity; }
    public double getPrice() { return price; }
}

class Address {
    private String street;
    private String city;
    private String region;
    private String country;
    
    public Address(String street, String city, String region, String country) {
        this.street = street;
        this.city = city;
        this.region = region;
        this.country = country;
    }
    
    public String getRegion() { return region; }
    public String getCountry() { return country; }
}

// Uso
public class OrderProcessingExample {
    public static void main(String[] args) {
        System.out.println("═══════════════════════════════════════");
        System.out.println("    Sistema de Processamento de Pedidos");
        System.out.println("═══════════════════════════════════════\n");
        
        // Pedido Online Nacional
        System.out.println("### PEDIDO ONLINE (NACIONAL) ###\n");
        Order onlineOrder = new Order(
            "ORD-001",
            "cliente@email.com",
            "(11) 98765-4321",
            new Address("Rua A, 123", "São Paulo", "Sudeste", "Brasil")
        );
        onlineOrder.addItem(new OrderItem("Notebook", 1, 2500.00));
        onlineOrder.addItem(new OrderItem("Mouse", 2, 50.00));
        
        OrderProcessor onlineProcessor = new OnlineOrderProcessor();
        onlineProcessor.processOrder(onlineOrder);
        
        // Pedido Internacional
        System.out.println("\n### PEDIDO INTERNACIONAL ###\n");
        Order intlOrder = new Order(
            "ORD-002",
            "customer@email.com",
            "+1-555-1234",
            new Address("123 Main St", "New York", "NY", "USA")
        );
        intlOrder.addItem(new OrderItem("Teclado Mecânico", 3, 150.00));
        intlOrder.addItem(new OrderItem("Webcam", 1, 200.00));
        
        OrderProcessor intlProcessor = new InternationalOrderProcessor();
        intlProcessor.processOrder(intlOrder);
        
        // Pedido Corporativo
        System.out.println("\n### PEDIDO CORPORATIVO (B2B) ###\n");
        Order corpOrder = new Order(
            "ORD-003",
            "compras@empresa.com.br",
            "(11) 3333-4444",
            new Address("Av. Paulista, 1000", "São Paulo", "Sudeste", "Brasil")
        );
        corpOrder.setCorporateId("12.345.678/0001-99");
        corpOrder.addItem(new OrderItem("Notebook", 10, 2500.00));
        corpOrder.addItem(new OrderItem("Monitor", 10, 800.00));
        
        OrderProcessor corpProcessor = new CorporateOrderProcessor();
        corpProcessor.processOrder(corpOrder);
    }
}
```

### Exemplo 2: Sistema de IA para Jogo de Estratégia

```java
// Abstract Class - IA Base
abstract class GameAI {
    
    // TEMPLATE METHOD - Turno da IA
    public final void takeTurn() {
        System.out.println("\n🎮 " + getRaceName() + " está jogando...");
        System.out.println("────────────────────────────────");
        
        collectResources();
        buildStructures();
        buildUnits();
        
        // Hook - Ações especiais
        performSpecialActions();
        
        attack();
        
        System.out.println("────────────────────────────────");
        System.out.println("✅ Turno finalizado!\n");
    }
    
    // Método com implementação padrão
    protected void collectResources() {
        System.out.println("💰 Coletando recursos...");
        System.out.println("   +100 ouro");
        System.out.println("   +50 madeira");
    }
    
    // Métodos abstratos - cada raça implementa diferente
    protected abstract void buildStructures();
    protected abstract void buildUnits();
    
    // Hook - Ações especiais opcionais
    protected void performSpecialActions() {
        // Subclasses podem adicionar ações especiais
    }
    
    // Método com implementação padrão
    protected void attack() {
        System.out.println("⚔️  Analisando inimigos...");
        
        String closestEnemy = findClosestEnemy();
        
        if (closestEnemy == null) {
            System.out.println("   Nenhum inimigo detectado");
            sendScouts("centro do mapa");
        } else {
            System.out.println("   Inimigo encontrado: " + closestEnemy);
            sendWarriors(closestEnemy);
        }
    }
    
    // Métodos abstratos para estratégia de ataque
    protected abstract void sendScouts(String location);
    protected abstract void sendWarriors(String location);
    
    // Métodos auxiliares
    protected abstract String getRaceName();
    
    protected String findClosestEnemy() {
        // Simula busca de inimigo
        double random = Math.random();
        return random > 0.5 ? "Base Inimiga" : null;
    }
}

// Concrete Class - Orcs (Agressivos)
class OrcsAI extends GameAI {
    
    @Override
    protected String getRaceName() {
        return "🟢 ORCS";
    }
    
    @Override
    protected void collectResources() {
        System.out.println("💰 Orcs saqueando recursos...");
        System.out.println("   +150 ouro (saque)");
        System.out.println("   +30 madeira (pilhagem)");
        System.out.println("   💀 +20 recursos de inimigos derrotados");
    }
    
    @Override
    protected void buildStructures() {
        System.out.println("🏗️  Construindo estruturas Orc...");
        System.out.println("   • Construindo Fazendas de Guerra");
        System.out.println("   • Construindo Quartéis");
        System.out.println("   • Fortificando Fortaleza");
    }
    
    @Override
    protected void buildUnits() {
        System.out.println("👥 Recrutando unidades Orc...");
        System.out.println("   • 5x Peons (trabalhadores)");
        System.out.println("   • 8x Grunts (guerreiros)");
        System.out.println("   • 3x Raiders (cavalaria)");
        System.out.println("   • 1x Catapulta");
    }
    
    @Override
    protected void performSpecialActions() {
        System.out.println("🔥 Ações especiais Orc:");
        System.out.println("   • Guerreiros entram em Fúria (+ 30% dano)");
        System.out.println("   • Xamãs lançam maldições nos inimigos");
    }
    
    @Override
    protected void sendScouts(String location) {
        System.out.println("🔍 Enviando Lobos Selvagens para: " + location);
    }
    
    @Override
    protected void sendWarriors(String location) {
        System.out.println("⚔️  ATAQUE MASSIVO a " + location + "!");
        System.out.println("   • Todos os Grunts atacando");
        System.out.println("   • Raiders flanqueando");
        System.out.println("   • Catapultas bombardeando");
        System.out.println("   💥 Estratégia: AGRESSÃO TOTAL!");
    }
}

// Concrete Class - Humanos (Equilibrados)
class HumansAI extends GameAI {
    
    @Override
    protected String getRaceName() {
        return "🔵 HUMANOS";
    }
    
    @Override
    protected void buildStructures() {
        System.out.println("🏗️  Construindo estruturas Humanas...");
        System.out.println("   • Construindo Fazendas");
        System.out.println("   • Construindo Quartéis");
        System.out.println("   • Construindo Muralhas Defensivas");
        System.out.println("   • Construindo Torres de Vigia");
    }
    
    @Override
    protected void buildUnits() {
        System.out.println("👥 Recrutando unidades Humanas...");
        System.out.println("   • 6x Camponeses");
        System.out.println("   • 5x Soldados");
        System.out.println("   • 3x Arqueiros");
        System.out.println("   • 2x Cavaleiros");
        System.out.println("   • 1x Clérigo (suporte)");
    }
    
    @Override
    protected void performSpecialActions() {
        System.out.println("✨ Ações especiais Humanas:");
        System.out.println("   • Clérigos curando unidades feridas");
        System.out.println("   • Pesquisando novas tecnologias");
        System.out.println("   • Fortificando defesas");
    }
    
    @Override
    protected void sendScouts(String location) {
        System.out.println("🔍 Enviando Arqueiros de reconhecimento para: " + location);
    }
    
    @Override
    protected void sendWarriors(String location) {
        System.out.println("⚔️  Ataque coordenado a " + location);
        System.out.println("   • Cavaleiros na vanguarda");
        System.out.println("   • Soldados protegendo arqueiros");
        System.out.println("   • Arqueiros dando cobertura");
        System.out.println("   🛡️  Estratégia: DEFESA E CONTRA-ATAQUE");
    }
}

// Concrete Class - Elfos (Mágicos)
class ElvesAI extends GameAI {
    
    @Override
    protected String getRaceName() {
        return "🟣 ELFOS";
    }
    
    @Override
    protected void collectResources() {
        System.out.println("💰 Elfos coletando recursos magicamente...");
        System.out.println("   +120 ouro");
        System.out.println("   +80 madeira (árvores encantadas)");
        System.out.println("   ✨ +50 cristais mágicos");
    }
    
    @Override
    protected void buildStructures() {
        System.out.println("🏗️  Conjurando estruturas Élficas...");
        System.out.println("   • Plantando Árvores da Vida");
        System.out.println("   • Erguendo Torres Arcanas");
        System.out.println("   • Criando Santuários Lunares");
    }
    
    @Override
    protected void buildUnits() {
        System.out.println("👥 Invocando unidades Élficas...");
        System.out.println("   • 4x Coletores de Espíritos");
        System.out.println("   • 6x Arqueiros Lunares");
        System.out.println("   • 3x Druidas");
        System.out.println("   • 2x Cavaleiros Hippogrifo");
        System.out.println("   • 1x Ancião da Floresta");
    }
    
    @Override
    protected void performSpecialActions() {
        System.out.println("✨ Ações especiais Élficas:");
        System.out.println("   • Druidas invocando Ents (árvores gigantes)");
        System.out.println("   • Lançando feitiços de proteção");
        System.out.println("   • Criando ilusões para confundir inimigos");
    }
    
    @Override
    protected void sendScouts(String location) {
        System.out.println("🔍 Enviando Corujas Espirituais para: " + location);
        System.out.println("   ✨ Visão revelando área invisível");
    }
    
    @Override
    protected void sendWarriors(String location) {
        System.out.println("⚔️  Emboscada mágica em " + location);
        System.out.println("   • Arqueiros atacando das árvores");
        System.out.println("   • Hippogrifos atacando do céu");
        System.out.println("   • Druidas lançando raízes para imobilizar");
        System.out.println("   🌙 Estratégia: EMBOSCADA E MAGIA");
    }
}

// Concrete Class - Mortos-Vivos (Especial)
class UndeadAI extends GameAI {
    
    @Override
    protected String getRaceName() {
        return "💀 MORTOS-VIVOS";
    }
    
    @Override
    protected void collectResources() {
        System.out.println("💰 Mortos-vivos não coletam recursos convencionais");
        System.out.println("   💀 +50 almas capturadas");
        System.out.println("   ⚰️  Criando recursos de cadáveres");
    }
    
    @Override
    protected void buildStructures() {
        // Mortos-vivos não constroem estruturas tradicionais
        System.out.println("🏗️  Mortos-vivos não constroem");
        System.out.println("   💀 Animando cemitérios existentes");
    }
    
    @Override
    protected void buildUnits() {
        System.out.println("👥 Ressuscitando mortos-vivos...");
        System.out.println("   • 10x Esqueletos (baratos)");
        System.out.println("   • 5x Zumbis");
        System.out.println("   • 3x Fantasmas");
        System.out.println("   • 1x Necromante");
    }
    
    @Override
    protected void performSpecialActions() {
        System.out.println("💀 Ações especiais dos Mortos-vivos:");
        System.out.println("   • Necromantes ressuscitando inimigos mortos");
        System.out.println("   • Espalhando praga nas tropas inimigas");
        System.out.println("   • Fantasmas atravessando muralhas");
    }
    
    @Override
    protected void sendScouts(String location) {
        System.out.println("🔍 Enviando Espectros invisíveis para: " + location);
    }
    
    @Override
    protected void sendWarriors(String location) {
        System.out.println("⚔️  Horda de mortos-vivos invadindo " + location);
        System.out.println("   • Horda infinita de esqueletos");
        System.out.println("   • Zumbis absorvendo dano");
        System.out.println("   • Fantasmas aterrorizando inimigos");
        System.out.println("   ☠️  Estratégia: EXAUSTÃO POR NÚMEROS");
    }
}

// Uso
public class GameAIExample {
    public static void main(String[] args) {
        System.out.println("═══════════════════════════════════════");
        System.out.println("    🎮 SIMULAÇÃO DE IA - JOGO RTS");
        System.out.println("═══════════════════════════════════════");
        
        // Cria IAs das diferentes raças
        GameAI[] players = {
            new OrcsAI(),
            new HumansAI(),
            new ElvesAI(),
            new UndeadAI()
        };
        
        // Simula 2 turnos
        for (int turn = 1; turn <= 2; turn++) {
            System.out.println("\n╔═══════════════════════════════════════╗");
            System.out.println("║          TURNO " + turn + "                      ║");
            System.out.println("╚═══════════════════════════════════════╝");
            
            for (GameAI player : players) {
                player.takeTurn();
                
                try {
                    Thread.sleep(500); // Pausa dramática
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
        
        System.out.println("\n═══════════════════════════════════════");
        System.out.println("    🏆 SIMULAÇÃO CONCLUÍDA");
        System.out.println("═══════════════════════════════════════");
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Algoritmo com variações**: Estrutura fixa, mas etapas customizáveis
- **Código duplicado**: Várias classes com código similar
- **Controle de extensão**: Quer permitir extensão apenas em pontos específicos
- **Framework/Library**: Define esqueleto, usuário implementa detalhes
- **Inversão de controle**: "Hollywood Principle" - Don't call us, we'll call you
- **Operação multi-etapas**: Processo com sequência bem definida

### 📝 Exemplos de aplicação:
- **Data parsing**: DOC, CSV, PDF, XML, JSON
- **Game AI**: Diferentes raças/facções com comportamento similar
- **Order processing**: Online, internacional, corporativo
- **Report generation**: PDF, Excel, HTML reports
- **Test frameworks**: Setup → Execute → Teardown
- **Build systems**: Configure → Compile → Test → Package

### ❌ Evite quando:
- **Algoritmo simples**: Poucas etapas, sem variação
- **Alta flexibilidade**: Precisa mudar estrutura, não só etapas
- **Composição mais apropriada**: Strategy pode ser melhor se precisar trocar em runtime

## 🚀 Como Implementar

1. **Analise algoritmo alvo** para identificar etapas comuns e variáveis

2. **Crie classe abstrata** e declare template method

3. **Declare métodos abstratos** para etapas que devem ser implementadas

4. **Adicione implementação padrão** para etapas opcionais

5. **Considere adicionar hooks** entre etapas cruciais

6. **Torne template method `final`** para prevenir sobrescrita

7. **Crie subclasses concretas** implementando etapas abstratas

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Reuso de código**: Código comum na superclasse
- **Extensibilidade controlada**: Cliente sobrescreve apenas partes específicas
- **Elimina duplicação**: Código idêntico movido para superclasse
- **Open/Closed**: Aberto para extensão, fechado para modificação
- **Inversão de controle**: Framework chama código do usuário

### ❌ Desvantagens:
- **LSP violado**: Subclasse pode suprimir etapa padrão
- **Difícil manutenção**: Quanto mais etapas, mais complexo
- **Herança**: Cliente limitado pela estrutura da superclasse
- **Menos flexível**: Estrutura fixa do algoritmo

## 🔗 Diferenças de Outros Padrões

| Padrão | Técnica | Flexibilidade | Nível | Quando mudar |
|--------|---------|---------------|-------|--------------|
| **Template Method** | Herança | Baixa (estrutura fixa) | Classe | Compile-time |
| **Strategy** | Composição | Alta (troca algoritmo) | Objeto | Runtime |
| **Factory Method** | Template Method + criação | Média | Classe | Compile-time |
| **Command** | Encapsulamento | Alta | Objeto | Runtime |

## 🔗 Relações com Outros Padrões

- **Factory Method** é especialização de Template Method:
  - Factory Method: Template Method focado em criação de objetos
  - Factory Method pode ser uma etapa dentro de Template Method maior

- **Template Method vs Strategy**:
  - Template Method: Herança, estático, nível de classe
  - Strategy: Composição, dinâmico, nível de objeto, runtime switching
  - Template Method define esqueleto fixo
  - Strategy permite trocar algoritmo inteiro

- **Template Method + Command**:
  - Comandos podem ser usados para implementar etapas do Template Method

- **Template Method em frameworks**:
  - JUnit: `setUp()` → `test()` → `tearDown()`
  - Servlets: `service()` chama `doGet()`, `doPost()`, etc.
  - React: Component lifecycle methods

## 📚 Conceitos-Chave para Lembrar

1. **Esqueleto do algoritmo**: Estrutura fixa definida na superclasse
2. **Etapas customizáveis**: Subclasses implementam detalhes específicos
3. **Inversão de controle**: "Don't call us, we'll call you"
4. **Template method final**: Não deve ser sobrescrito
5. **Hooks**: Pontos de extensão opcionais com corpo vazio
6. **Reuso de código**: Elimina duplicação movendo código comum para cima

## 🔍 Analogia do Mundo Real

**Construindo uma casa**: O plano arquitetônico (template method) define as etapas: fundar → estruturar → paredes → encanamento → elétrica → acabamento. Essa sequência é fixa e não pode ser alterada (casa sem fundação cai!). Mas você pode customizar cada etapa: tipo de fundação, material das paredes, estilo de acabamento. O arquiteto (superclasse) define O QUE fazer e QUANDO; você (subclasse) define COMO fazer cada etapa.

## ⚠️ Considerações Importantes

### Template Method com Hooks:

```java
abstract class DataProcessor {
    // Template Method
    public final void process() {
        loadData();
        
        // Hook antes do processamento
        beforeProcess();
        
        processData();
        
        // Hook depois do processamento
        afterProcess();
        
        saveData();
    }
    
    // Métodos abstratos
    protected abstract void loadData();
    protected abstract void processData();
    protected abstract void saveData();
    
    // Hooks - implementação vazia
    protected void beforeProcess() {}
    protected void afterProcess() {}
}

class CachedDataProcessor extends DataProcessor {
    @Override
    protected void loadData() { /* ... */ }
    
    @Override
    protected void processData() { /* ... */ }
    
    @Override
    protected void saveData() { /* ... */ }
    
    // Sobrescreve hook para adicionar cache
    @Override
    protected void beforeProcess() {
        System.out.println("Verificando cache...");
    }
    
    @Override
    protected void afterProcess() {
        System.out.println("Salvando no cache...");
    }
}
```

### Template Method vs Strategy - Exemplo prático:

```java
// TEMPLATE METHOD - Herança, estrutura fixa
abstract class ReportGenerator {
    public final void generate() {
        loadData();
        formatData();    // Customizável
        exportData();    // Customizável
    }
    
    protected void loadData() { /* comum */ }
    protected abstract void formatData();
    protected abstract void exportData();
}

// STRATEGY - Composição, algoritmo intercambiável
interface ExportStrategy {
    void export(Data data);
}

class ReportGenerator {
    private ExportStrategy strategy;
    
    public void setStrategy(ExportStrategy s) {
        this.strategy = s;
    }
    
    public void generate() {
        Data data = loadData();
        strategy.export(data); // Delega totalmente
    }
}
```

### Liskov Substitution Principle:

```java
// ❌ VIOLAÇÃO LSP
abstract class DataMiner {
    public void mine() {
        openFile();
        extractData();
        closeFile();
    }
    
    protected abstract void extractData();
}

class NoOpMiner extends DataMiner {
    @Override
    protected void extractData() {
        // Não faz nada! Viola expectativa do template method
    }
}

// ✅ MELHOR - Use hook
abstract class DataMiner {
    public void mine() {
        openFile();
        if (shouldExtract()) {  // Hook com lógica
            extractData();
        }
        closeFile();
    }
    
    protected boolean shouldExtract() {
        return true; // Padrão
    }
}
```

### Template Method em Testes:

```java
// Framework de teste usa Template Method
abstract class TestCase {
    // Template Method
    public final void run() {
        setUp();
        try {
            runTest();
        } finally {
            tearDown();
        }
    }
    
    protected void setUp() {}        // Hook
    protected abstract void runTest();
    protected void tearDown() {}     // Hook
}

class MyTest extends TestCase {
    private Database db;
    
    @Override
    protected void setUp() {
        db = new Database();
        db.connect();
    }
    
    @Override
    protected void runTest() {
        // Test code
        assert db.query("SELECT * FROM users").size() > 0;
    }
    
    @Override
    protected void tearDown() {
        db.disconnect();
    }
}
```

### Design Guidelines:
- **Minimize abstract methods**: Apenas o necessário
- **Maximize reuso**: Código comum na superclasse
- **Use hooks**: Para pontos de extensão opcionais
- **final no template method**: Previne sobrescrita acidental
- **protected para etapas**: Não devem ser chamadas externamente
- **Documente ordem**: Deixe clara a sequência das etapas
- **Considere Strategy**: Se precisar trocar algoritmo em runtime

---

> **💡 Dica de Estudo:** Template Method é como uma receita de bolo - a sequência é fixa (misturar → assar → decorar), mas você pode customizar ingredientes (chocolate/baunilha) e decoração. O processo (receita) garante que o bolo ficará bom, mas você controla o sabor!

> **📖 Referência:** [Refactoring Guru - Template Method](https://refactoring.guru/design-patterns/template-method)
