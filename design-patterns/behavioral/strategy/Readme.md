# Strategy

## 🎯 Intenção

O Strategy é um padrão de projeto comportamental que permite definir uma família de algoritmos, colocar cada um deles em uma classe separada, e tornar seus objetos intercambiáveis. Ele permite que o algoritmo varie independentemente dos clientes que o utilizam.

## 🚩 Problema

Imagine que você criou um aplicativo de navegação para viajantes casuais. Uma das funcionalidades mais solicitadas era o planejamento automático de rotas. A primeira versão construía rotas apenas por estradas (carros). Com o sucesso, você adicionou rotas a pé, depois transporte público, depois para ciclistas, e depois rotas turísticas.

### Resultado problemático:
```java
// Classe Navigator com todos os algoritmos
class Navigator {
    private String routeType; // "car", "walking", "bus", "bike", "tourist"
    
    public Route buildRoute(String start, String end) {
        if (routeType.equals("car")) {
            // Algoritmo complexo para rotas de carro
            // 200 linhas de código
            return calculateCarRoute(start, end);
            
        } else if (routeType.equals("walking")) {
            // Algoritmo complexo para rotas a pé
            // 180 linhas de código
            return calculateWalkingRoute(start, end);
            
        } else if (routeType.equals("bus")) {
            // Algoritmo complexo para transporte público
            // 250 linhas de código
            return calculateBusRoute(start, end);
            
        } else if (routeType.equals("bike")) {
            // Algoritmo complexo para ciclistas
            // 190 linhas de código
            return calculateBikeRoute(start, end);
            
        } else if (routeType.equals("tourist")) {
            // Algoritmo complexo para rotas turísticas
            // 220 linhas de código
            return calculateTouristRoute(start, end);
        }
        
        return null;
    }
    
    private Route calculateCarRoute(String start, String end) {
        // Implementação gigante...
    }
    
    // Mais 4 métodos gigantes...
}
```

**Problemas:**
- **Classe inchada**: Navigator dobrava de tamanho a cada novo algoritmo
- **Difícil manutenção**: Bug fix em um algoritmo pode afetar outros
- **Merge conflicts**: Time trabalhando na mesma classe gigante
- **Violação SRP**: Classe faz demais (5+ algoritmos diferentes)
- **Difícil teste**: Testar um algoritmo requer instanciar classe inteira
- **Não extensível**: Adicionar algoritmo requer modificar classe existente

## ✅ Solução

O padrão Strategy sugere que você pegue uma classe que faz algo específico de muitas formas diferentes e extraia todos esses algoritmos para classes separadas chamadas **strategies**. A classe original, chamada **context**, deve ter um campo para armazenar uma referência a uma das strategies. O context delega o trabalho para um objeto strategy ligado ao invés de executá-lo por conta própria.

### Características-chave:
- **Família de algoritmos**: Diferentes formas de fazer a mesma coisa
- **Encapsulamento**: Cada algoritmo em sua própria classe
- **Intercambiabilidade**: Strategies trocáveis em runtime
- **Delegação**: Context delega ao strategy
- **Interface comum**: Todos strategies implementam mesma interface
- **Cliente escolhe**: Cliente seleciona strategy apropriado

```java
// Interface Strategy
interface RouteStrategy {
    Route buildRoute(String start, String end);
}

// Context
class Navigator {
    private RouteStrategy strategy;
    
    // Strategy pode ser trocado em runtime
    public void setStrategy(RouteStrategy strategy) {
        this.strategy = strategy;
    }
    
    // Delega ao strategy
    public Route buildRoute(String start, String end) {
        return strategy.buildRoute(start, end);
    }
}

// Concrete Strategy - Carro
class CarRouteStrategy implements RouteStrategy {
    @Override
    public Route buildRoute(String start, String end) {
        // Algoritmo específico para carros
        System.out.println("🚗 Calculando rota de carro");
        return new Route(/* rota otimizada para velocidade */);
    }
}

// Concrete Strategy - Caminhada
class WalkingRouteStrategy implements RouteStrategy {
    @Override
    public Route buildRoute(String start, String end) {
        // Algoritmo específico para pedestres
        System.out.println("🚶 Calculando rota a pé");
        return new Route(/* rota com calçadas e segurança */);
    }
}

// Uso
Navigator nav = new Navigator();
nav.setStrategy(new CarRouteStrategy());
Route route1 = nav.buildRoute("A", "B");

nav.setStrategy(new WalkingRouteStrategy()); // Troca strategy!
Route route2 = nav.buildRoute("A", "B");
```

## 🏗️ Estrutura

```
Context                          Strategy (interface)
    ↓                                   ↓
[strategy: Strategy]              algorithm()
setStrategy(strategy)                  ↑
executeStrategy() ----→               |
    [delega]            ConcreteStrategyA  ConcreteStrategyB
                              ↓                  ↓
                         algorithm()        algorithm()
```

### Componentes:
- **Context**: Mantém referência ao strategy, delega execução
- **Strategy**: Interface comum para todos algoritmos
- **ConcreteStrategy**: Implementa variação específica do algoritmo
- **Client**: Cria strategy específico e passa ao context

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Pagamento com Múltiplos Métodos

```java
// Interface Strategy
interface PaymentStrategy {
    boolean pay(double amount);
    String getPaymentMethod();
}

// Concrete Strategy - Cartão de Crédito
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String cvv;
    private String expiryDate;
    
    public CreditCardPayment(String cardNumber, String cvv, String expiryDate) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("💳 Processando pagamento com Cartão de Crédito");
        System.out.println("   Valor: $" + String.format("%.2f", amount));
        System.out.println("   Cartão: **** **** **** " + cardNumber.substring(12));
        
        // Validações
        if (!validateCard()) {
            System.out.println("   ❌ Cartão inválido");
            return false;
        }
        
        // Simula processamento
        System.out.println("   ✅ Pagamento aprovado!");
        return true;
    }
    
    private boolean validateCard() {
        // Validação simplificada
        return cardNumber.length() == 16 && cvv.length() == 3;
    }
    
    @Override
    public String getPaymentMethod() {
        return "Cartão de Crédito";
    }
}

// Concrete Strategy - PayPal
class PayPalPayment implements PaymentStrategy {
    private String email;
    private String password;
    
    public PayPalPayment(String email, String password) {
        this.email = email;
        this.password = password;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("🅿️  Processando pagamento com PayPal");
        System.out.println("   Valor: $" + String.format("%.2f", amount));
        System.out.println("   Conta: " + email);
        
        // Autenticação
        if (!authenticate()) {
            System.out.println("   ❌ Falha na autenticação");
            return false;
        }
        
        // Simula processamento
        System.out.println("   ✅ Pagamento via PayPal aprovado!");
        return true;
    }
    
    private boolean authenticate() {
        // Autenticação simplificada
        return email.contains("@") && password.length() >= 6;
    }
    
    @Override
    public String getPaymentMethod() {
        return "PayPal";
    }
}

// Concrete Strategy - PIX
class PixPayment implements PaymentStrategy {
    private String pixKey;
    
    public PixPayment(String pixKey) {
        this.pixKey = pixKey;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("📱 Processando pagamento com PIX");
        System.out.println("   Valor: $" + String.format("%.2f", amount));
        System.out.println("   Chave PIX: " + pixKey);
        
        // Gera QR Code (simulado)
        String qrCode = generateQRCode(amount);
        System.out.println("   📊 QR Code gerado: " + qrCode);
        System.out.println("   ⏳ Aguardando confirmação...");
        
        // Simula confirmação
        System.out.println("   ✅ Pagamento PIX confirmado!");
        return true;
    }
    
    private String generateQRCode(double amount) {
        return "00020126580014br.gov.bcb.pix" + pixKey.hashCode();
    }
    
    @Override
    public String getPaymentMethod() {
        return "PIX";
    }
}

// Concrete Strategy - Boleto
class BoletoPayment implements PaymentStrategy {
    private String cpf;
    
    public BoletoPayment(String cpf) {
        this.cpf = cpf;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("🧾 Processando pagamento com Boleto");
        System.out.println("   Valor: $" + String.format("%.2f", amount));
        System.out.println("   CPF: " + maskCPF(cpf));
        
        // Gera código de barras (simulado)
        String barcode = generateBarcode(amount);
        System.out.println("   📋 Código de barras: " + barcode);
        System.out.println("   📅 Vencimento: 3 dias");
        System.out.println("   ✅ Boleto gerado! Pague até o vencimento.");
        return true;
    }
    
    private String generateBarcode(double amount) {
        return "34191.79001 01043.510047 91020.150008 1 84560000" + 
               String.format("%010d", (int)(amount * 100));
    }
    
    private String maskCPF(String cpf) {
        return cpf.substring(0, 3) + ".***.***-" + cpf.substring(9);
    }
    
    @Override
    public String getPaymentMethod() {
        return "Boleto Bancário";
    }
}

// Context - Carrinho de Compras
class ShoppingCart {
    private List<Item> items;
    private PaymentStrategy paymentStrategy;
    
    public ShoppingCart() {
        this.items = new ArrayList<>();
    }
    
    public void addItem(Item item) {
        items.add(item);
        System.out.println("🛒 Adicionado: " + item.getName() + 
                         " ($" + String.format("%.2f", item.getPrice()) + ")");
    }
    
    public void setPaymentMethod(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
        System.out.println("💰 Método de pagamento: " + strategy.getPaymentMethod());
    }
    
    public void checkout() {
        if (items.isEmpty()) {
            System.out.println("❌ Carrinho vazio!");
            return;
        }
        
        if (paymentStrategy == null) {
            System.out.println("❌ Selecione um método de pagamento!");
            return;
        }
        
        double total = calculateTotal();
        System.out.println("\n💵 Total: $" + String.format("%.2f", total));
        System.out.println("─────────────────────────────────");
        
        boolean success = paymentStrategy.pay(total);
        
        if (success) {
            System.out.println("─────────────────────────────────");
            System.out.println("🎉 Compra finalizada com sucesso!");
            items.clear();
        } else {
            System.out.println("─────────────────────────────────");
            System.out.println("❌ Falha no pagamento. Tente novamente.");
        }
    }
    
    private double calculateTotal() {
        return items.stream()
            .mapToDouble(Item::getPrice)
            .sum();
    }
    
    public void showCart() {
        System.out.println("\n🛒 Carrinho de Compras:");
        for (Item item : items) {
            System.out.println("   • " + item.getName() + 
                             " - $" + String.format("%.2f", item.getPrice()));
        }
        System.out.println("   Total: $" + String.format("%.2f", calculateTotal()));
    }
}

// Item
class Item {
    private String name;
    private double price;
    
    public Item(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
}

// Uso
public class PaymentExample {
    public static void main(String[] args) {
        System.out.println("=== Sistema de Pagamento E-commerce ===\n");
        
        ShoppingCart cart = new ShoppingCart();
        
        // Adiciona itens
        System.out.println("--- Adicionando itens ---");
        cart.addItem(new Item("Notebook", 2999.99));
        cart.addItem(new Item("Mouse", 49.99));
        cart.addItem(new Item("Teclado", 199.99));
        
        cart.showCart();
        
        // Checkout com Cartão de Crédito
        System.out.println("\n\n--- Checkout 1: Cartão de Crédito ---");
        cart.setPaymentMethod(new CreditCardPayment(
            "1234567890123456",
            "123",
            "12/2025"
        ));
        cart.checkout();
        
        // Adiciona novos itens
        System.out.println("\n\n--- Nova compra ---");
        cart.addItem(new Item("Headset", 299.99));
        cart.addItem(new Item("Webcam", 399.99));
        
        // Checkout com PayPal
        System.out.println("\n--- Checkout 2: PayPal ---");
        cart.setPaymentMethod(new PayPalPayment(
            "user@email.com",
            "securePassword123"
        ));
        cart.checkout();
        
        // Nova compra com PIX
        System.out.println("\n\n--- Nova compra ---");
        cart.addItem(new Item("Monitor", 899.99));
        
        System.out.println("\n--- Checkout 3: PIX ---");
        cart.setPaymentMethod(new PixPayment("user@email.com"));
        cart.checkout();
        
        // Nova compra com Boleto
        System.out.println("\n\n--- Nova compra ---");
        cart.addItem(new Item("Impressora", 599.99));
        
        System.out.println("\n--- Checkout 4: Boleto ---");
        cart.setPaymentMethod(new BoletoPayment("12345678901"));
        cart.checkout();
    }
}
```

### Exemplo 2: Sistema de Compressão de Arquivos

```java
// Interface Strategy
interface CompressionStrategy {
    byte[] compress(byte[] data);
    byte[] decompress(byte[] compressedData);
    String getAlgorithmName();
    double getCompressionRatio(byte[] original, byte[] compressed);
}

// Concrete Strategy - ZIP
class ZipCompression implements CompressionStrategy {
    @Override
    public byte[] compress(byte[] data) {
        System.out.println("📦 Comprimindo com ZIP...");
        // Simulação de compressão ZIP
        byte[] compressed = simulateCompression(data, 0.6);
        System.out.println("   Tamanho original: " + data.length + " bytes");
        System.out.println("   Tamanho comprimido: " + compressed.length + " bytes");
        System.out.println("   Taxa: " + 
            String.format("%.1f%%", getCompressionRatio(data, compressed)));
        return compressed;
    }
    
    @Override
    public byte[] decompress(byte[] compressedData) {
        System.out.println("📂 Descomprimindo ZIP...");
        return simulateDecompression(compressedData, 1.67);
    }
    
    @Override
    public String getAlgorithmName() {
        return "ZIP";
    }
    
    @Override
    public double getCompressionRatio(byte[] original, byte[] compressed) {
        return (1.0 - (double) compressed.length / original.length) * 100;
    }
    
    private byte[] simulateCompression(byte[] data, double ratio) {
        int newSize = (int) (data.length * ratio);
        return new byte[newSize];
    }
    
    private byte[] simulateDecompression(byte[] data, double ratio) {
        int newSize = (int) (data.length * ratio);
        return new byte[newSize];
    }
}

// Concrete Strategy - RAR
class RarCompression implements CompressionStrategy {
    @Override
    public byte[] compress(byte[] data) {
        System.out.println("📦 Comprimindo com RAR...");
        // RAR geralmente tem melhor compressão
        byte[] compressed = simulateCompression(data, 0.5);
        System.out.println("   Tamanho original: " + data.length + " bytes");
        System.out.println("   Tamanho comprimido: " + compressed.length + " bytes");
        System.out.println("   Taxa: " + 
            String.format("%.1f%%", getCompressionRatio(data, compressed)));
        return compressed;
    }
    
    @Override
    public byte[] decompress(byte[] compressedData) {
        System.out.println("📂 Descomprimindo RAR...");
        return simulateDecompression(compressedData, 2.0);
    }
    
    @Override
    public String getAlgorithmName() {
        return "RAR";
    }
    
    @Override
    public double getCompressionRatio(byte[] original, byte[] compressed) {
        return (1.0 - (double) compressed.length / original.length) * 100;
    }
    
    private byte[] simulateCompression(byte[] data, double ratio) {
        int newSize = (int) (data.length * ratio);
        return new byte[newSize];
    }
    
    private byte[] simulateDecompression(byte[] data, double ratio) {
        int newSize = (int) (data.length * ratio);
        return new byte[newSize];
    }
}

// Concrete Strategy - 7z
class SevenZipCompression implements CompressionStrategy {
    @Override
    public byte[] compress(byte[] data) {
        System.out.println("📦 Comprimindo com 7z...");
        // 7z geralmente tem a melhor compressão
        byte[] compressed = simulateCompression(data, 0.45);
        System.out.println("   Tamanho original: " + data.length + " bytes");
        System.out.println("   Tamanho comprimido: " + compressed.length + " bytes");
        System.out.println("   Taxa: " + 
            String.format("%.1f%%", getCompressionRatio(data, compressed)));
        return compressed;
    }
    
    @Override
    public byte[] decompress(byte[] compressedData) {
        System.out.println("📂 Descomprimindo 7z...");
        return simulateDecompression(compressedData, 2.22);
    }
    
    @Override
    public String getAlgorithmName() {
        return "7-Zip";
    }
    
    @Override
    public double getCompressionRatio(byte[] original, byte[] compressed) {
        return (1.0 - (double) compressed.length / original.length) * 100;
    }
    
    private byte[] simulateCompression(byte[] data, double ratio) {
        int newSize = (int) (data.length * ratio);
        return new byte[newSize];
    }
    
    private byte[] simulateDecompression(byte[] data, double ratio) {
        int newSize = (int) (data.length * ratio);
        return new byte[newSize];
    }
}

// Context - Compressor de Arquivos
class FileCompressor {
    private CompressionStrategy strategy;
    
    public void setCompressionStrategy(CompressionStrategy strategy) {
        this.strategy = strategy;
        System.out.println("🔧 Algoritmo selecionado: " + strategy.getAlgorithmName());
    }
    
    public byte[] compressFile(String filename, byte[] data) {
        if (strategy == null) {
            System.out.println("❌ Nenhum algoritmo de compressão selecionado!");
            return data;
        }
        
        System.out.println("\n📁 Comprimindo arquivo: " + filename);
        long startTime = System.currentTimeMillis();
        
        byte[] compressed = strategy.compress(data);
        
        long endTime = System.currentTimeMillis();
        System.out.println("   ⏱️  Tempo: " + (endTime - startTime) + "ms");
        System.out.println("   ✅ Compressão concluída!");
        
        return compressed;
    }
    
    public byte[] decompressFile(byte[] compressedData) {
        if (strategy == null) {
            System.out.println("❌ Nenhum algoritmo de compressão selecionado!");
            return compressedData;
        }
        
        System.out.println("\n📂 Descomprimindo arquivo...");
        long startTime = System.currentTimeMillis();
        
        byte[] decompressed = strategy.decompress(compressedData);
        
        long endTime = System.currentTimeMillis();
        System.out.println("   ⏱️  Tempo: " + (endTime - startTime) + "ms");
        System.out.println("   ✅ Descompressão concluída!");
        
        return decompressed;
    }
    
    public void compareAlgorithms(String filename, byte[] data) {
        System.out.println("\n📊 Comparando algoritmos de compressão");
        System.out.println("   Arquivo: " + filename);
        System.out.println("   Tamanho: " + data.length + " bytes");
        System.out.println("─────────────────────────────────────");
        
        CompressionStrategy[] algorithms = {
            new ZipCompression(),
            new RarCompression(),
            new SevenZipCompression()
        };
        
        for (CompressionStrategy algo : algorithms) {
            setCompressionStrategy(algo);
            byte[] compressed = algo.compress(data);
            System.out.println();
        }
    }
}

// Uso
public class CompressionExample {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("=== Sistema de Compressão de Arquivos ===\n");
        
        FileCompressor compressor = new FileCompressor();
        
        // Simula dados de arquivo
        byte[] fileData = new byte[10000]; // 10KB
        
        // Compressão com ZIP
        System.out.println("--- Teste 1: ZIP ---");
        compressor.setCompressionStrategy(new ZipCompression());
        byte[] zipCompressed = compressor.compressFile("document.txt", fileData);
        Thread.sleep(50);
        compressor.decompressFile(zipCompressed);
        
        // Compressão com RAR
        System.out.println("\n\n--- Teste 2: RAR ---");
        compressor.setCompressionStrategy(new RarCompression());
        byte[] rarCompressed = compressor.compressFile("image.jpg", fileData);
        Thread.sleep(50);
        compressor.decompressFile(rarCompressed);
        
        // Compressão com 7z
        System.out.println("\n\n--- Teste 3: 7-Zip ---");
        compressor.setCompressionStrategy(new SevenZipCompression());
        byte[] sevenZipCompressed = compressor.compressFile("video.mp4", fileData);
        Thread.sleep(50);
        compressor.decompressFile(sevenZipCompressed);
        
        // Comparação de algoritmos
        System.out.println("\n\n─────────────────────────────────────");
        compressor.compareAlgorithms("large-file.bin", fileData);
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Variantes de algoritmo**: Quer usar diferentes versões de um algoritmo
- **Runtime switching**: Precisa trocar algoritmo em tempo de execução
- **Classes similares**: Classes que diferem apenas no comportamento
- **Isolar lógica**: Separar lógica de negócio de detalhes de implementação
- **Condicionais massivas**: Muitos if/switch para diferentes algoritmos
- **Open/Closed**: Quer adicionar algoritmos sem modificar código existente

### 📝 Exemplos de aplicação:
- **Payment methods**: Cartão, PayPal, PIX, boleto
- **Sorting algorithms**: QuickSort, MergeSort, BubbleSort
- **Compression**: ZIP, RAR, 7z, GZIP
- **Navigation**: Car, walking, bike, public transport
- **Validation**: Email, phone, CPF, credit card
- **Export formats**: PDF, Excel, CSV, JSON

### ❌ Evite quando:
- **Poucos algoritmos**: 1-2 algoritmos simples
- **Algoritmos raramente mudam**: Não há necessidade de flexibilidade
- **Linguagem funcional**: Lambdas/funções anônimas são mais simples

## 🚀 Como Implementar

1. **Identifique algoritmo** que muda frequentemente

2. **Declare interface Strategy** comum para todas variantes

3. **Extraia algoritmos** para classes separadas implementando Strategy

4. **Adicione campo Strategy** no Context

5. **Context delega** execução ao Strategy

6. **Cliente cria** Strategy específico e passa ao Context

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Runtime switching**: Troca algoritmo em tempo de execução
- **Isolamento**: Detalhes de implementação isolados
- **Composição sobre herança**: Favorece composição
- **Open/Closed**: Novos strategies sem mudar context
- **Testabilidade**: Fácil testar cada strategy isoladamente

### ❌ Desvantagens:
- **Mais classes**: Uma classe por strategy
- **Cliente precisa saber**: Diferenças entre strategies
- **Overhead**: Se poucos algoritmos, pode ser excessivo
- **Linguagens modernas**: Lambdas podem ser mais simples

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Quem escolhe | Relacionamento |
|--------|------|--------------|----------------|
| **Strategy** | Diferentes formas de fazer | Cliente escolhe | Strategies independentes |
| **State** | Comportamento por estado | Estado muda automaticamente | Estados conhecem uns aos outros |
| **Template Method** | Esqueleto de algoritmo | Subclasses implementam passos | Herança (estático) |
| **Command** | Encapsula operação | Cliente cria comando | Commands independentes |

## 🔗 Relações com Outros Padrões

- **Bridge, State, Strategy** (e Adapter):
  - Estrutura similar (composição/delegação)
  - Strategy: Cliente escolhe algoritmo
  - State: Estado interno muda comportamento
  - Bridge: Separa abstração de implementação

- **Command vs Strategy**:
  - Command: Qualquer operação como objeto (undo, queue, etc)
  - Strategy: Diferentes formas de fazer mesma coisa

- **Decorator vs Strategy**:
  - Decorator: Muda aparência (skin)
  - Strategy: Muda comportamento interno (guts)

- **Template Method vs Strategy**:
  - Template Method: Herança, estático
  - Strategy: Composição, dinâmico (runtime)

- **State vs Strategy**:
  - State: Extensão de Strategy
  - State: Estados interdependentes
  - Strategy: Strategies completamente independentes

## 📚 Conceitos-Chave para Lembrar

1. **Família de algoritmos**: Diferentes formas de fazer mesma coisa
2. **Intercambiabilidade**: Strategies trocáveis em runtime
3. **Encapsulamento**: Cada algoritmo isolado em classe própria
4. **Delegação**: Context delega ao strategy
5. **Cliente escolhe**: Cliente seleciona strategy apropriado
6. **Composição**: Favorece composição sobre herança

## 🔍 Analogia do Mundo Real

**Ir ao aeroporto**: Você precisa chegar ao aeroporto. Pode pegar um ônibus (barato, lento), chamar um táxi (médio custo, médio tempo), ou ir de bicicleta (grátis, depende da distância). Essas são suas "strategies" de transporte. Você escolhe uma baseado em fatores como orçamento ou tempo. Todas levam ao mesmo destino (objetivo comum), mas de formas diferentes. Você pode mudar de ideia no último minuto e escolher outra strategy!

## ⚠️ Considerações Importantes

### Strategy com Lambdas (Java 8+):

```java
// Tradicional
interface SortStrategy {
    void sort(int[] array);
}

Context context = new Context();
context.setStrategy(new BubbleSortStrategy());

// Com Lambda
interface SortStrategy {
    void sort(int[] array);
}

Context context = new Context();
context.setStrategy(array -> Arrays.sort(array)); // Lambda!

// Ou method reference
context.setStrategy(Arrays::sort);
```

### Strategy com dados do Context:

```java
// Strategy precisa de dados do Context
interface DiscountStrategy {
    double calculateDiscount(Order order);
}

class SeasonalDiscount implements DiscountStrategy {
    public double calculateDiscount(Order order) {
        // Acessa dados do order
        double total = order.getTotal();
        LocalDate date = order.getDate();
        
        if (date.getMonth() == Month.DECEMBER) {
            return total * 0.2; // 20% desconto natal
        }
        return 0;
    }
}
```

### Factory para criar Strategies:

```java
// Factory simplifica criação
class PaymentStrategyFactory {
    public static PaymentStrategy create(String type) {
        switch (type) {
            case "CREDIT_CARD":
                return new CreditCardPayment();
            case "PAYPAL":
                return new PayPalPayment();
            case "PIX":
                return new PixPayment();
            default:
                throw new IllegalArgumentException("Unknown type");
        }
    }
}

// Uso
PaymentStrategy strategy = PaymentStrategyFactory.create("PIX");
cart.setPaymentMethod(strategy);
```

### Null Object Strategy:

```java
// Evita null checks
class NoOpCompressionStrategy implements CompressionStrategy {
    public byte[] compress(byte[] data) {
        return data; // Sem compressão
    }
    
    public byte[] decompress(byte[] data) {
        return data;
    }
}

// Context
class Compressor {
    private CompressionStrategy strategy = new NoOpCompressionStrategy();
    
    public void setStrategy(CompressionStrategy strategy) {
        this.strategy = strategy; // Nunca null
    }
}
```

### Design Guidelines:
- **Interface mínima**: Strategy interface deve ser focado
- **Stateless**: Strategies devem ser stateless se possível (reusáveis)
- **Cliente informado**: Cliente precisa saber qual strategy escolher
- **Context passa dados**: Via parâmetros ou permitindo acesso
- **Nomenclatura clara**: Nome do strategy deve indicar o que faz
- **Considere Lambdas**: Em linguagens modernas, podem ser mais simples

---

> **💡 Dica de Estudo:** Strategy é como escolher meio de transporte para ir ao trabalho - ônibus (barato/lento), carro (caro/rápido), ou bicicleta (grátis/saudável). Todos levam ao mesmo lugar, mas de formas diferentes. Você escolhe baseado em suas necessidades (budget, tempo, clima). Pode mudar de ideia a qualquer momento!

> **📖 Referência:** [Refactoring Guru - Strategy](https://refactoring.guru/design-patterns/strategy)
