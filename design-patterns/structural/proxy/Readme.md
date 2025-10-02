# Proxy

## 🎯 Intenção

O Proxy é um padrão de projeto estrutural que permite fornecer um substituto ou placeholder para outro objeto. Um proxy controla o acesso ao objeto original, permitindo que você execute algo antes ou depois que a solicitação chegue ao objeto original.

## 🚩 Problema

Por que você gostaria de controlar o acesso a um objeto? Imagine que você tem um objeto massivo que consome uma grande quantidade de recursos do sistema. Você precisa dele de vez em quando, mas não sempre.

### Resultado problemático:
```java
// Objeto pesado que sempre consome recursos
class HeavyDatabaseObject {
    private String connectionData;
    private List<String> cache;
    
    public HeavyDatabaseObject() {
        // Conexão custosa que demora para inicializar
        this.connectionData = connectToDatabase(); // 10 segundos!
        this.cache = loadAllData(); // 500MB de dados!
        System.out.println("Objeto pesado criado - recursos consumidos!");
    }
    
    public String query(String sql) {
        // Consulta real ao banco
        return executeQuery(sql);
    }
}

// Cliente sempre paga o preço alto
public class Application {
    public static void main(String[] args) {
        // Mesmo que não use, objeto é criado
        HeavyDatabaseObject db = new HeavyDatabaseObject(); // 10s + 500MB
        
        // Talvez nem use...
        if (userWantsData()) {
            String result = db.query("SELECT * FROM users");
        }
    }
}
```

**Problemas:**
- **Inicialização custosa**: Objeto criado mesmo quando não necessário
- **Consumo de recursos**: Memória e tempo desperdiçados
- **Falta de controle**: Não há como interceptar ou modificar operações
- **Código duplicado**: Clientes repetem lógica de inicialização

## ✅ Solução

O padrão Proxy sugere que você crie uma nova classe proxy com a mesma interface do objeto de serviço original. O proxy recebe solicitações do cliente e pode executar ações **antes** ou **depois** de delegar o trabalho ao objeto real.

### Benefícios-chave:
- **Lazy loading**: Cria objeto real apenas quando necessário
- **Controle de acesso**: Adiciona verificações de segurança
- **Cache**: Armazena resultados para evitar operações custosas
- **Logging**: Registra todas as operações
- **Interface transparente**: Cliente não sabe que está usando proxy

```java
// Proxy - controla acesso ao objeto real
class DatabaseProxy implements DatabaseService {
    private HeavyDatabaseObject realDatabase;
    private String connectionString;
    
    public DatabaseProxy(String connectionString) {
        this.connectionString = connectionString;
        // NÃO cria objeto real ainda!
    }
    
    public String query(String sql) {
        // Lazy loading - cria apenas quando necessário
        if (realDatabase == null) {
            realDatabase = new HeavyDatabaseObject(connectionString);
        }
        
        // Executa controles adicionais
        if (!hasPermission(sql)) {
            throw new SecurityException("Acesso negado");
        }
        
        // Delega para objeto real
        return realDatabase.query(sql);
    }
}
```

## 🏗️ Estrutura

```
Client → ServiceInterface ← Proxy
                ↑              ↓
          RealService ←─── [reference]
```

### Componentes:
- **ServiceInterface**: Interface comum para Service e Proxy
- **Service**: Classe que fornece lógica de negócio útil
- **Proxy**: Tem campo de referência para objeto service, controla acesso
- **Client**: Trabalha com service e proxy através da mesma interface

## 💻 Exemplos Práticos

### Exemplo 1: Proxy Virtual (Lazy Loading)

```java
// Interface comum
interface ImageService {
    void display();
    void rotate(int degrees);
    String getInfo();
}

// Objeto real - custoso para criar
class HighResolutionImage implements ImageService {
    private String filename;
    private byte[] imageData;
    private int width, height;
    
    public HighResolutionImage(String filename) {
        this.filename = filename;
        loadImageFromDisk(); // Operação custosa!
    }
    
    private void loadImageFromDisk() {
        System.out.println("Carregando imagem de alta resolução: " + filename);
        // Simula carregamento custoso
        try {
            Thread.sleep(2000); // 2 segundos para carregar
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // Simula carregamento de dados
        this.imageData = new byte[1024 * 1024 * 10]; // 10MB
        this.width = 4096;
        this.height = 3072;
        System.out.println("Imagem carregada: " + width + "x" + height + " (" + 
                          imageData.length / 1024 / 1024 + "MB)");
    }
    
    @Override
    public void display() {
        System.out.println("Exibindo imagem: " + filename + " (" + width + "x" + height + ")");
    }
    
    @Override
    public void rotate(int degrees) {
        System.out.println("Rotacionando imagem " + degrees + " graus");
        // Operação custosa de rotação
    }
    
    @Override
    public String getInfo() {
        return "Imagem: " + filename + " - " + width + "x" + height + 
               " (" + imageData.length / 1024 / 1024 + "MB)";
    }
}

// Proxy Virtual - lazy loading
class ImageProxy implements ImageService {
    private String filename;
    private HighResolutionImage realImage;
    
    public ImageProxy(String filename) {
        this.filename = filename;
        System.out.println("Proxy criado para: " + filename + " (imagem não carregada ainda)");
    }
    
    @Override
    public void display() {
        // Carrega imagem real apenas quando necessário
        if (realImage == null) {
            System.out.println("Primeira exibição - carregando imagem real...");
            realImage = new HighResolutionImage(filename);
        }
        realImage.display();
    }
    
    @Override
    public void rotate(int degrees) {
        if (realImage == null) {
            realImage = new HighResolutionImage(filename);
        }
        realImage.rotate(degrees);
    }
    
    @Override
    public String getInfo() {
        // Informações básicas sem carregar imagem
        if (realImage == null) {
            return "Imagem: " + filename + " (não carregada)";
        }
        return realImage.getInfo();
    }
}

// Galeria de imagens
class ImageGallery {
    private List<ImageService> images;
    
    public ImageGallery() {
        this.images = new ArrayList<>();
    }
    
    public void addImage(String filename) {
        // Usa proxy - não carrega imagem imediatamente
        images.add(new ImageProxy(filename));
    }
    
    public void showThumbnails() {
        System.out.println("=== Thumbnails da Galeria ===");
        for (int i = 0; i < images.size(); i++) {
            System.out.println((i + 1) + ". " + images.get(i).getInfo());
        }
    }
    
    public void displayImage(int index) {
        if (index >= 0 && index < images.size()) {
            System.out.println("\n=== Exibindo Imagem " + (index + 1) + " ===");
            images.get(index).display();
        }
    }
    
    public void rotateImage(int index, int degrees) {
        if (index >= 0 && index < images.size()) {
            images.get(index).rotate(degrees);
        }
    }
}

// Uso
public class VirtualProxyExample {
    public static void main(String[] args) {
        System.out.println("=== Criando Galeria de Imagens ===");
        ImageGallery gallery = new ImageGallery();
        
        // Adiciona várias imagens (proxies criados instantaneamente)
        gallery.addImage("vacation_beach.jpg");
        gallery.addImage("family_portrait.jpg");
        gallery.addImage("landscape_mountain.jpg");
        
        System.out.println("\n=== Visualizando Thumbnails ===");
        gallery.showThumbnails(); // Rápido - não carrega imagens reais
        
        System.out.println("\n=== Usuário clica na primeira imagem ===");
        gallery.displayImage(0); // Agora carrega a imagem real
        
        System.out.println("\n=== Usuário clica na primeira imagem novamente ===");
        gallery.displayImage(0); // Usa imagem já carregada
        
        System.out.println("\n=== Usuário rotaciona a primeira imagem ===");
        gallery.rotateImage(0, 90); // Usa imagem já carregada
        
        System.out.println("\n=== Usuário clica na terceira imagem ===");
        gallery.displayImage(2); // Carrega nova imagem
    }
}
```

### Exemplo 2: Proxy de Proteção (Controle de Acesso)

```java
// Interface do serviço
interface FileService {
    String readFile(String filename);
    void writeFile(String filename, String content);
    void deleteFile(String filename);
    List<String> listFiles();
}

// Serviço real
class FileSystemService implements FileService {
    @Override
    public String readFile(String filename) {
        System.out.println("Lendo arquivo: " + filename);
        return "Conteúdo do arquivo " + filename;
    }
    
    @Override
    public void writeFile(String filename, String content) {
        System.out.println("Escrevendo arquivo: " + filename);
        System.out.println("Conteúdo: " + content);
    }
    
    @Override
    public void deleteFile(String filename) {
        System.out.println("Deletando arquivo: " + filename);
    }
    
    @Override
    public List<String> listFiles() {
        System.out.println("Listando arquivos do sistema");
        return Arrays.asList("config.txt", "data.json", "backup.zip");
    }
}

// Proxy de proteção
class SecureFileProxy implements FileService {
    private FileSystemService realService;
    private String currentUser;
    private Set<String> adminUsers;
    private Set<String> readOnlyUsers;
    
    public SecureFileProxy(String currentUser) {
        this.currentUser = currentUser;
        this.realService = new FileSystemService();
        this.adminUsers = Set.of("admin", "superuser");
        this.readOnlyUsers = Set.of("guest", "viewer");
    }
    
    private boolean hasReadPermission() {
        return adminUsers.contains(currentUser) || readOnlyUsers.contains(currentUser);
    }
    
    private boolean hasWritePermission() {
        return adminUsers.contains(currentUser);
    }
    
    private void logAccess(String operation, String resource) {
        System.out.println("[LOG] Usuário '" + currentUser + "' executou '" + 
                          operation + "' em '" + resource + "'");
    }
    
    @Override
    public String readFile(String filename) {
        if (!hasReadPermission()) {
            throw new SecurityException("Usuário '" + currentUser + 
                                      "' não tem permissão de leitura");
        }
        
        logAccess("READ", filename);
        return realService.readFile(filename);
    }
    
    @Override
    public void writeFile(String filename, String content) {
        if (!hasWritePermission()) {
            throw new SecurityException("Usuário '" + currentUser + 
                                      "' não tem permissão de escrita");
        }
        
        // Validações adicionais
        if (filename.contains("system") && !adminUsers.contains(currentUser)) {
            throw new SecurityException("Apenas admins podem modificar arquivos do sistema");
        }
        
        logAccess("WRITE", filename);
        realService.writeFile(filename, content);
    }
    
    @Override
    public void deleteFile(String filename) {
        if (!hasWritePermission()) {
            throw new SecurityException("Usuário '" + currentUser + 
                                      "' não tem permissão de exclusão");
        }
        
        // Proteção extra para arquivos críticos
        if (filename.contains("config") || filename.contains("system")) {
            if (!adminUsers.contains(currentUser)) {
                throw new SecurityException("Apenas admins podem deletar arquivos críticos");
            }
        }
        
        logAccess("DELETE", filename);
        realService.deleteFile(filename);
    }
    
    @Override
    public List<String> listFiles() {
        if (!hasReadPermission()) {
            throw new SecurityException("Usuário '" + currentUser + 
                                      "' não tem permissão de listagem");
        }
        
        logAccess("LIST", "directory");
        List<String> files = realService.listFiles();
        
        // Filtra arquivos baseado em permissões
        if (!adminUsers.contains(currentUser)) {
            files = files.stream()
                        .filter(file -> !file.contains("system"))
                        .collect(Collectors.toList());
        }
        
        return files;
    }
}

// Sistema de arquivos com diferentes usuários
class FileManager {
    private Map<String, FileService> userServices;
    
    public FileManager() {
        this.userServices = new HashMap<>();
    }
    
    public void loginUser(String username) {
        FileService service = new SecureFileProxy(username);
        userServices.put(username, service);
        System.out.println("Usuário '" + username + "' conectado");
    }
    
    public void performOperation(String username, String operation, String... params) {
        FileService service = userServices.get(username);
        if (service == null) {
            System.out.println("Usuário não conectado");
            return;
        }
        
        try {
            switch (operation.toLowerCase()) {
                case "read":
                    String content = service.readFile(params[0]);
                    System.out.println("Resultado: " + content);
                    break;
                case "write":
                    service.writeFile(params[0], params[1]);
                    System.out.println("Arquivo escrito com sucesso");
                    break;
                case "delete":
                    service.deleteFile(params[0]);
                    System.out.println("Arquivo deletado com sucesso");
                    break;
                case "list":
                    List<String> files = service.listFiles();
                    System.out.println("Arquivos: " + files);
                    break;
                default:
                    System.out.println("Operação desconhecida");
            }
        } catch (SecurityException e) {
            System.out.println("ERRO DE SEGURANÇA: " + e.getMessage());
        }
    }
}

// Uso
public class ProtectionProxyExample {
    public static void main(String[] args) {
        FileManager fileManager = new FileManager();
        
        System.out.println("=== Conectando usuários ===");
        fileManager.loginUser("admin");
        fileManager.loginUser("guest");
        fileManager.loginUser("hacker");
        
        System.out.println("\n=== Teste com Admin ===");
        fileManager.performOperation("admin", "list");
        fileManager.performOperation("admin", "read", "config.txt");
        fileManager.performOperation("admin", "write", "new_file.txt", "conteúdo");
        fileManager.performOperation("admin", "delete", "backup.zip");
        
        System.out.println("\n=== Teste com Guest ===");
        fileManager.performOperation("guest", "list");
        fileManager.performOperation("guest", "read", "data.json");
        fileManager.performOperation("guest", "write", "guest_file.txt", "dados");
        fileManager.performOperation("guest", "delete", "config.txt");
        
        System.out.println("\n=== Teste com Usuário sem permissão ===");
        fileManager.performOperation("hacker", "list");
        fileManager.performOperation("hacker", "read", "sensitive.txt");
    }
}
```

### Exemplo 3: Proxy de Cache

```java
// Interface do serviço
interface WeatherService {
    WeatherData getWeather(String city);
    List<WeatherData> getForecast(String city, int days);
}

// Dados do clima
class WeatherData {
    private String city;
    private double temperature;
    private String condition;
    private int humidity;
    private long timestamp;
    
    public WeatherData(String city, double temperature, String condition, int humidity) {
        this.city = city;
        this.temperature = temperature;
        this.condition = condition;
        this.humidity = humidity;
        this.timestamp = System.currentTimeMillis();
    }
    
    public boolean isExpired(long cacheTimeMs) {
        return System.currentTimeMillis() - timestamp > cacheTimeMs;
    }
    
    @Override
    public String toString() {
        return String.format("%s: %.1f°C, %s, %d%% umidade", 
                           city, temperature, condition, humidity);
    }
    
    // Getters
    public String getCity() { return city; }
    public double getTemperature() { return temperature; }
    public String getCondition() { return condition; }
    public int getHumidity() { return humidity; }
}

// Serviço real - faz chamadas de API caras
class ExternalWeatherService implements WeatherService {
    private Random random = new Random();
    
    @Override
    public WeatherData getWeather(String city) {
        System.out.println("🌐 Fazendo chamada de API para: " + city);
        simulateNetworkDelay();
        
        // Simula dados da API
        double temp = 15 + random.nextDouble() * 20; // 15-35°C
        String[] conditions = {"Ensolarado", "Nublado", "Chuvoso", "Tempestade"};
        String condition = conditions[random.nextInt(conditions.length)];
        int humidity = 30 + random.nextInt(50); // 30-80%
        
        return new WeatherData(city, temp, condition, humidity);
    }
    
    @Override
    public List<WeatherData> getForecast(String city, int days) {
        System.out.println("🌐 Fazendo chamada de API para previsão de " + days + " dias: " + city);
        simulateNetworkDelay();
        
        List<WeatherData> forecast = new ArrayList<>();
        for (int i = 0; i < days; i++) {
            double temp = 15 + random.nextDouble() * 20;
            String[] conditions = {"Ensolarado", "Nublado", "Chuvoso"};
            String condition = conditions[random.nextInt(conditions.length)];
            int humidity = 30 + random.nextInt(50);
            
            forecast.add(new WeatherData(city + " (+" + i + " dias)", temp, condition, humidity));
        }
        
        return forecast;
    }
    
    private void simulateNetworkDelay() {
        try {
            Thread.sleep(1000 + random.nextInt(2000)); // 1-3 segundos
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// Proxy com cache
class CachedWeatherProxy implements WeatherService {
    private ExternalWeatherService realService;
    private Map<String, WeatherData> weatherCache;
    private Map<String, List<WeatherData>> forecastCache;
    private final long CACHE_DURATION = 5 * 60 * 1000; // 5 minutos
    
    public CachedWeatherProxy() {
        this.realService = new ExternalWeatherService();
        this.weatherCache = new HashMap<>();
        this.forecastCache = new HashMap<>();
    }
    
    @Override
    public WeatherData getWeather(String city) {
        // Verifica cache primeiro
        WeatherData cached = weatherCache.get(city);
        if (cached != null && !cached.isExpired(CACHE_DURATION)) {
            System.out.println("📋 Cache HIT para: " + city);
            return cached;
        }
        
        // Cache miss ou expirado
        System.out.println("📋 Cache MISS para: " + city);
        WeatherData fresh = realService.getWeather(city);
        weatherCache.put(city, fresh);
        
        return fresh;
    }
    
    @Override
    public List<WeatherData> getForecast(String city, int days) {
        String key = city + "_" + days;
        List<WeatherData> cached = forecastCache.get(key);
        
        if (cached != null && !cached.isEmpty() && 
            !cached.get(0).isExpired(CACHE_DURATION)) {
            System.out.println("📋 Cache HIT para previsão: " + key);
            return cached;
        }
        
        System.out.println("📋 Cache MISS para previsão: " + key);
        List<WeatherData> fresh = realService.getForecast(city, days);
        forecastCache.put(key, fresh);
        
        return fresh;
    }
    
    public void clearCache() {
        weatherCache.clear();
        forecastCache.clear();
        System.out.println("🗑️ Cache limpo");
    }
    
    public void printCacheStats() {
        System.out.println("📊 Cache stats - Weather: " + weatherCache.size() + 
                          " entries, Forecast: " + forecastCache.size() + " entries");
    }
}

// Aplicação do clima
class WeatherApp {
    private WeatherService weatherService;
    
    public WeatherApp(WeatherService weatherService) {
        this.weatherService = weatherService;
    }
    
    public void checkWeather(String city) {
        System.out.println("\n=== Consultando clima para " + city + " ===");
        long start = System.currentTimeMillis();
        
        WeatherData weather = weatherService.getWeather(city);
        
        long end = System.currentTimeMillis();
        System.out.println("Resultado: " + weather);
        System.out.println("Tempo: " + (end - start) + "ms");
    }
    
    public void checkForecast(String city, int days) {
        System.out.println("\n=== Consultando previsão para " + city + " (" + days + " dias) ===");
        long start = System.currentTimeMillis();
        
        List<WeatherData> forecast = weatherService.getForecast(city, days);
        
        long end = System.currentTimeMillis();
        System.out.println("Previsão:");
        forecast.forEach(data -> System.out.println("  " + data));
        System.out.println("Tempo: " + (end - start) + "ms");
    }
}

// Uso
public class CacheProxyExample {
    public static void main(String[] args) {
        CachedWeatherProxy weatherProxy = new CachedWeatherProxy();
        WeatherApp app = new WeatherApp(weatherProxy);
        
        // Primeira consulta - cache miss
        app.checkWeather("São Paulo");
        
        // Segunda consulta - cache hit
        app.checkWeather("São Paulo");
        
        // Consulta diferente - cache miss
        app.checkWeather("Rio de Janeiro");
        
        // Consulta anterior novamente - cache hit
        app.checkWeather("São Paulo");
        
        // Previsão - cache miss
        app.checkForecast("São Paulo", 3);
        
        // Mesma previsão - cache hit
        app.checkForecast("São Paulo", 3);
        
        weatherProxy.printCacheStats();
        
        // Limpa cache e testa novamente
        System.out.println("\n=== Limpando cache ===");
        weatherProxy.clearCache();
        app.checkWeather("São Paulo"); // Novamente cache miss
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:

#### 1. **Lazy Initialization (Virtual Proxy)**
- Objetos custosos que nem sempre são usados
- Carregamento sob demanda de recursos
- Aplicações que precisam inicializar rapidamente

#### 2. **Controle de Acesso (Protection Proxy)**
- Verificação de permissões de usuário
- Validação de credenciais
- Filtragem de operações baseada em contexto

#### 3. **Cache (Caching Proxy)**
- Resultados de operações custosas
- Dados que mudam infrequentemente
- Redução de chamadas de rede/banco

#### 4. **Logging e Monitoramento**
- Rastreamento de uso de objetos
- Auditoria de operações
- Métricas de performance

### 📝 Exemplos de aplicação:
- **ORM/Hibernate**: Lazy loading de relacionamentos
- **Spring AOP**: Proxies para aspectos transversais
- **CDN**: Cache de conteúdo estático
- **API Gateways**: Controle de acesso e rate limiting

### ❌ Evite quando:
- **Interface simples**: Overhead desnecessário para objetos simples
- **Performance crítica**: Latência adicional pode ser problemática
- **Debugging complexo**: Pode dificultar rastreamento de problemas

## 🚀 Como Implementar

1. **Crie interface comum** se não existir (Service Interface)

2. **Implemente classe Proxy** com referência ao service

3. **Adicione lógica do proxy** conforme o tipo:
   - Virtual: lazy loading
   - Protection: verificação de acesso
   - Cache: armazenamento de resultados

4. **Gerencie ciclo de vida** do objeto real

5. **Considere factory method** para decidir entre proxy ou service real

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Controle transparente**: Cliente não sabe da existência do proxy
- **Lazy loading**: Otimização de recursos
- **Segurança**: Controle de acesso granular
- **Cache**: Melhoria de performance
- **Separação de responsabilidades**: Concerns transversais isolados
- **Open/Closed**: Novos proxies sem alterar service

### ❌ Desvantagens:
- **Complexidade**: Mais classes no sistema
- **Latência**: Overhead de chamadas indiretas
- **Debugging**: Mais difícil rastrear fluxo
- **Memória**: Proxy adicional consome recursos

## 🔗 Diferenças de Outros Padrões

| Padrão | Interface | Propósito | Relação |
|--------|-----------|-----------|---------|
| **Proxy** | Mesma interface | Controlar acesso | Substituto transparente |
| **Adapter** | Interface diferente | Compatibilizar | Tradução de interface |
| **Decorator** | Interface estendida | Adicionar funcionalidade | Enriquecimento |
| **Facade** | Interface simplificada | Simplificar complexidade | Abstração |

## 🔗 Relações com Outros Padrões

- **Adapter**: Proxy mantém interface, Adapter a modifica
- **Decorator**: Ambos usam composição, mas Proxy gerencia ciclo de vida
- **Facade**: Proxy tem mesma interface, Facade simplifica interface
- **Singleton**: Proxy pode implementar Singleton para cache global

## 📚 Conceitos-Chave para Lembrar

1. **Transparência**: Cliente não sabe que está usando proxy
2. **Controle de acesso**: Principal benefício do padrão
3. **Mesma interface**: Proxy e service implementam mesma interface
4. **Lazy loading**: Criação sob demanda do objeto real
5. **Ciclo de vida**: Proxy gerencia quando criar/destruir service
6. **Tipos diferentes**: Virtual, Protection, Cache, Remote, etc.

## 🔍 Analogia do Mundo Real

**Cartão de crédito**: É um proxy para sua conta bancária. Tem a mesma interface (fazer pagamentos), mas adiciona controles (limite, verificação de segurança), cache (pre-autorização), e logging (histórico de transações). O comerciante não precisa saber se você está pagando com dinheiro físico ou cartão - a interface é a mesma.

## ⚠️ Considerações Importantes

### Tipos de Proxy:

#### 1. **Virtual Proxy**
```java
class VirtualProxy implements Service {
    private Service realService;
    
    public String operation() {
        if (realService == null) {
            realService = new ExpensiveService(); // lazy load
        }
        return realService.operation();
    }
}
```

#### 2. **Protection Proxy**
```java
class ProtectionProxy implements Service {
    private Service realService;
    private User currentUser;
    
    public String operation() {
        if (!hasPermission(currentUser)) {
            throw new SecurityException("Access denied");
        }
        return realService.operation();
    }
}
```

#### 3. **Cache Proxy**
```java
class CacheProxy implements Service {
    private Service realService;
    private Map<String, String> cache = new HashMap<>();
    
    public String operation(String input) {
        if (!cache.containsKey(input)) {
            cache.put(input, realService.operation(input));
        }
        return cache.get(input);
    }
}
```

### Design Guidelines:
- **Interface consistency**: Proxy deve ter exatamente mesma interface
- **Error handling**: Proxy deve tratar erros apropriadamente
- **Performance monitoring**: Meça impacto do proxy na performance
- **Resource management**: Gerencie recursos adequadamente

### Frameworks que usam Proxy:
- **Spring Framework**: AOP proxies, transactional proxies
- **Hibernate**: Lazy loading proxies
- **Java Dynamic Proxy**: Reflection-based proxies
- **CGLIB**: Code generation proxies

---

> **💡 Dica de Estudo:** Proxy é como um "assistente pessoal" - ele tem a mesma autoridade (interface) que seu chefe, mas adiciona controles, otimizações e verificações antes de passar as tarefas adiante. O cliente nem percebe que está falando com o assistente, não com o chefe diretamente.

> **📖 Referência:** [Refactoring Guru - Proxy](https://refactoring.guru/design-patterns/proxy)
