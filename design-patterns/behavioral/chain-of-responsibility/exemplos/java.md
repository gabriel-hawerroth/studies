# Exemplos Práticos - Chain of Responsibility (Java)

## Exemplo 1: Sistema de Suporte Técnico

```java
// Classe de requisição
class SupportRequest {
    private String type;
    private String priority;
    private String description;
    private String customer;
    
    public SupportRequest(String type, String priority, String description, String customer) {
        this.type = type;
        this.priority = priority;
        this.description = description;
        this.customer = customer;
    }
    
    // Getters
    public String getType() { return type; }
    public String getPriority() { return priority; }
    public String getDescription() { return description; }
    public String getCustomer() { return customer; }
}

// Handler abstrato
abstract class SupportHandler {
    protected SupportHandler nextHandler;
    protected String handlerName;
    
    public SupportHandler(String handlerName) {
        this.handlerName = handlerName;
    }
    
    public SupportHandler setNext(SupportHandler handler) {
        this.nextHandler = handler;
        return handler;
    }
    
    public void handle(SupportRequest request) {
        if (canHandle(request)) {
            process(request);
        } else if (nextHandler != null) {
            System.out.println(handlerName + " não pode resolver. Encaminhando...");
            nextHandler.handle(request);
        } else {
            System.out.println("❌ Nenhum handler disponível para processar a requisição.");
        }
    }
    
    protected abstract boolean canHandle(SupportRequest request);
    protected abstract void process(SupportRequest request);
}

// Handler de nível 1 - Atendente básico
class BasicSupportHandler extends SupportHandler {
    public BasicSupportHandler() {
        super("Atendente Básico");
    }
    
    @Override
    protected boolean canHandle(SupportRequest request) {
        return "BASIC".equals(request.getType()) && "LOW".equals(request.getPriority());
    }
    
    @Override
    protected void process(SupportRequest request) {
        System.out.println("✅ " + handlerName + " resolveu o problema:");
        System.out.println("   Cliente: " + request.getCustomer());
        System.out.println("   Problema: " + request.getDescription());
        System.out.println("   Solução: Consultou FAQ e resolveu rapidamente.");
    }
}

// Handler de nível 2 - Técnico especializado
class TechnicalSupportHandler extends SupportHandler {
    public TechnicalSupportHandler() {
        super("Técnico Especializado");
    }
    
    @Override
    protected boolean canHandle(SupportRequest request) {
        return "TECHNICAL".equals(request.getType()) && 
               ("LOW".equals(request.getPriority()) || "MEDIUM".equals(request.getPriority()));
    }
    
    @Override
    protected void process(SupportRequest request) {
        System.out.println("✅ " + handlerName + " resolveu o problema:");
        System.out.println("   Cliente: " + request.getCustomer());
        System.out.println("   Problema: " + request.getDescription());
        System.out.println("   Solução: Analisou logs e aplicou correção técnica.");
    }
}

// Handler de nível 3 - Engenheiro sênior
class SeniorEngineerHandler extends SupportHandler {
    public SeniorEngineerHandler() {
        super("Engenheiro Sênior");
    }
    
    @Override
    protected boolean canHandle(SupportRequest request) {
        return "TECHNICAL".equals(request.getType()) && "HIGH".equals(request.getPriority());
    }
    
    @Override
    protected void process(SupportRequest request) {
        System.out.println("✅ " + handlerName + " resolveu o problema:");
        System.out.println("   Cliente: " + request.getCustomer());
        System.out.println("   Problema: " + request.getDescription());
        System.out.println("   Solução: Bug crítico corrigido com hotfix.");
    }
}

// Handler de nível 4 - Gerente
class ManagerHandler extends SupportHandler {
    public ManagerHandler() {
        super("Gerente");
    }
    
    @Override
    protected boolean canHandle(SupportRequest request) {
        return "CRITICAL".equals(request.getPriority());
    }
    
    @Override
    protected void process(SupportRequest request) {
        System.out.println("✅ " + handlerName + " assumiu o caso:");
        System.out.println("   Cliente: " + request.getCustomer());
        System.out.println("   Problema: " + request.getDescription());
        System.out.println("   Ação: Escalado para equipe prioritária, SLA ativado.");
    }
}

// Sistema de suporte
class SupportSystem {
    private SupportHandler handlerChain;
    
    public SupportSystem() {
        // Monta a cadeia de handlers
        handlerChain = new BasicSupportHandler();
        handlerChain.setNext(new TechnicalSupportHandler())
                   .setNext(new SeniorEngineerHandler())
                   .setNext(new ManagerHandler());
    }
    
    public void submitRequest(SupportRequest request) {
        System.out.println("\n=== Nova Requisição de Suporte ===");
        System.out.println("Tipo: " + request.getType() + " | Prioridade: " + request.getPriority());
        handlerChain.handle(request);
    }
}

// Uso
public class SupportExample {
    public static void main(String[] args) {
        SupportSystem system = new SupportSystem();
        
        // Caso 1: Problema básico
        system.submitRequest(new SupportRequest(
            "BASIC", "LOW", 
            "Como resetar minha senha?", 
            "João Silva"
        ));
        
        // Caso 2: Problema técnico médio
        system.submitRequest(new SupportRequest(
            "TECHNICAL", "MEDIUM", 
            "Aplicação travando ao carregar relatório", 
            "Maria Santos"
        ));
        
        // Caso 3: Problema técnico crítico
        system.submitRequest(new SupportRequest(
            "TECHNICAL", "HIGH", 
            "Sistema fora do ar - erro de banco de dados", 
            "TechCorp Ltd"
        ));
        
        // Caso 4: Problema crítico de negócio
        system.submitRequest(new SupportRequest(
            "BUSINESS", "CRITICAL", 
            "Perda de dados em transação financeira", 
            "BigBank S.A."
        ));
        
        // Caso 5: Requisição sem handler adequado
        system.submitRequest(new SupportRequest(
            "UNKNOWN", "UNDEFINED", 
            "Tipo de problema não categorizado", 
            "Cliente X"
        ));
    }
}
```

## Exemplo 2: Middleware de Requisições HTTP

```java
import java.util.*;

// Classe de requisição HTTP
class HttpRequest {
    private String method;
    private String url;
    private Map<String, String> headers;
    private String body;
    private String username;
    private String password;
    private String ipAddress;
    
    public HttpRequest(String method, String url) {
        this.method = method;
        this.url = url;
        this.headers = new HashMap<>();
    }
    
    public HttpRequest addHeader(String key, String value) {
        headers.put(key, value);
        return this;
    }
    
    public HttpRequest setAuth(String username, String password) {
        this.username = username;
        this.password = password;
        return this;
    }
    
    public HttpRequest setBody(String body) {
        this.body = body;
        return this;
    }
    
    public HttpRequest setIpAddress(String ip) {
        this.ipAddress = ip;
        return this;
    }
    
    // Getters
    public String getMethod() { return method; }
    public String getUrl() { return url; }
    public Map<String, String> getHeaders() { return headers; }
    public String getBody() { return body; }
    public String getUsername() { return username; }
    public String getPassword() { return password; }
    public String getIpAddress() { return ipAddress; }
}

// Resposta HTTP
class HttpResponse {
    private int statusCode;
    private String body;
    private boolean success;
    
    public HttpResponse(int statusCode, String body, boolean success) {
        this.statusCode = statusCode;
        this.body = body;
        this.success = success;
    }
    
    public int getStatusCode() { return statusCode; }
    public String getBody() { return body; }
    public boolean isSuccess() { return success; }
}

// Middleware abstrato
abstract class Middleware {
    protected Middleware next;
    
    public Middleware setNext(Middleware next) {
        this.next = next;
        return next;
    }
    
    public HttpResponse handle(HttpRequest request) {
        HttpResponse response = check(request);
        
        if (!response.isSuccess()) {
            return response; // Para a cadeia se falhar
        }
        
        if (next != null) {
            return next.handle(request);
        }
        
        // Fim da cadeia - processa requisição
        return processRequest(request);
    }
    
    protected abstract HttpResponse check(HttpRequest request);
    
    protected HttpResponse processRequest(HttpRequest request) {
        return new HttpResponse(200, "Requisição processada com sucesso", true);
    }
}

// Middleware de autenticação
class AuthenticationMiddleware extends Middleware {
    private Map<String, String> users;
    
    public AuthenticationMiddleware() {
        users = new HashMap<>();
        users.put("admin", "admin123");
        users.put("user", "user123");
    }
    
    @Override
    protected HttpResponse check(HttpRequest request) {
        System.out.println("🔐 [Auth] Verificando credenciais...");
        
        String username = request.getUsername();
        String password = request.getPassword();
        
        if (username == null || password == null) {
            System.out.println("   ❌ Credenciais não fornecidas");
            return new HttpResponse(401, "Autenticação necessária", false);
        }
        
        if (!users.containsKey(username) || !users.get(username).equals(password)) {
            System.out.println("   ❌ Credenciais inválidas");
            return new HttpResponse(401, "Usuário ou senha incorretos", false);
        }
        
        System.out.println("   ✅ Autenticado como: " + username);
        return new HttpResponse(200, "OK", true);
    }
}

// Middleware de autorização
class AuthorizationMiddleware extends Middleware {
    private Set<String> adminUsers;
    private Set<String> restrictedPaths;
    
    public AuthorizationMiddleware() {
        adminUsers = Set.of("admin");
        restrictedPaths = Set.of("/admin", "/config", "/users");
    }
    
    @Override
    protected HttpResponse check(HttpRequest request) {
        System.out.println("🔑 [Authorization] Verificando permissões...");
        
        String username = request.getUsername();
        String url = request.getUrl();
        
        boolean isRestricted = restrictedPaths.stream()
            .anyMatch(path -> url.startsWith(path));
        
        if (isRestricted && !adminUsers.contains(username)) {
            System.out.println("   ❌ Acesso negado para: " + url);
            return new HttpResponse(403, "Acesso proibido", false);
        }
        
        System.out.println("   ✅ Acesso autorizado");
        return new HttpResponse(200, "OK", true);
    }
}

// Middleware de validação
class ValidationMiddleware extends Middleware {
    @Override
    protected HttpResponse check(HttpRequest request) {
        System.out.println("📋 [Validation] Validando requisição...");
        
        if (request.getMethod() == null || request.getUrl() == null) {
            System.out.println("   ❌ Método ou URL ausentes");
            return new HttpResponse(400, "Bad Request", false);
        }
        
        if ("POST".equals(request.getMethod()) || "PUT".equals(request.getMethod())) {
            if (request.getBody() == null || request.getBody().isEmpty()) {
                System.out.println("   ❌ Body necessário para " + request.getMethod());
                return new HttpResponse(400, "Body é obrigatório", false);
            }
        }
        
        System.out.println("   ✅ Requisição válida");
        return new HttpResponse(200, "OK", true);
    }
}

// Middleware de rate limiting
class RateLimitMiddleware extends Middleware {
    private Map<String, Integer> requestCounts;
    private Map<String, Long> lastRequestTime;
    private static final int MAX_REQUESTS = 5;
    private static final long TIME_WINDOW = 60000; // 1 minuto
    
    public RateLimitMiddleware() {
        requestCounts = new HashMap<>();
        lastRequestTime = new HashMap<>();
    }
    
    @Override
    protected HttpResponse check(HttpRequest request) {
        System.out.println("⏱️  [RateLimit] Verificando limite de requisições...");
        
        String ip = request.getIpAddress();
        long currentTime = System.currentTimeMillis();
        
        // Reseta contador se passou o tempo
        if (lastRequestTime.containsKey(ip)) {
            long timePassed = currentTime - lastRequestTime.get(ip);
            if (timePassed > TIME_WINDOW) {
                requestCounts.put(ip, 0);
            }
        }
        
        int count = requestCounts.getOrDefault(ip, 0);
        
        if (count >= MAX_REQUESTS) {
            System.out.println("   ❌ Limite excedido para IP: " + ip);
            return new HttpResponse(429, "Too Many Requests", false);
        }
        
        requestCounts.put(ip, count + 1);
        lastRequestTime.put(ip, currentTime);
        
        System.out.println("   ✅ Requisição permitida (" + (count + 1) + "/" + MAX_REQUESTS + ")");
        return new HttpResponse(200, "OK", true);
    }
}

// Middleware de logging
class LoggingMiddleware extends Middleware {
    @Override
    protected HttpResponse check(HttpRequest request) {
        System.out.println("📝 [Logging] Registrando requisição:");
        System.out.println("   Método: " + request.getMethod());
        System.out.println("   URL: " + request.getUrl());
        System.out.println("   IP: " + request.getIpAddress());
        System.out.println("   Usuário: " + request.getUsername());
        
        return new HttpResponse(200, "OK", true);
    }
}

// Servidor HTTP
class HttpServer {
    private Middleware middleware;
    
    public HttpServer() {
        // Monta a cadeia de middleware
        middleware = new LoggingMiddleware();
        middleware.setNext(new RateLimitMiddleware())
                 .setNext(new AuthenticationMiddleware())
                 .setNext(new AuthorizationMiddleware())
                 .setNext(new ValidationMiddleware());
    }
    
    public void handleRequest(HttpRequest request) {
        System.out.println("\n" + "=".repeat(60));
        System.out.println("🌐 Processando requisição: " + request.getMethod() + " " + request.getUrl());
        System.out.println("=".repeat(60));
        
        HttpResponse response = middleware.handle(request);
        
        System.out.println("\n📤 Resposta:");
        System.out.println("   Status: " + response.getStatusCode());
        System.out.println("   Body: " + response.getBody());
    }
}

// Uso
public class MiddlewareExample {
    public static void main(String[] args) {
        HttpServer server = new HttpServer();
        
        // Caso 1: Requisição bem-sucedida
        HttpRequest request1 = new HttpRequest("GET", "/api/products")
            .setAuth("user", "user123")
            .setIpAddress("192.168.1.1");
        server.handleRequest(request1);
        
        // Caso 2: Falha de autenticação
        HttpRequest request2 = new HttpRequest("GET", "/api/orders")
            .setAuth("user", "wrongpass")
            .setIpAddress("192.168.1.2");
        server.handleRequest(request2);
        
        // Caso 3: Falha de autorização (usuário comum tentando acessar admin)
        HttpRequest request3 = new HttpRequest("GET", "/admin/settings")
            .setAuth("user", "user123")
            .setIpAddress("192.168.1.3");
        server.handleRequest(request3);
        
        // Caso 4: Sucesso com admin
        HttpRequest request4 = new HttpRequest("POST", "/admin/users")
            .setAuth("admin", "admin123")
            .setBody("{\"name\":\"New User\"}")
            .setIpAddress("192.168.1.4");
        server.handleRequest(request4);
        
        // Caso 5: Falha de validação (POST sem body)
        HttpRequest request5 = new HttpRequest("POST", "/api/orders")
            .setAuth("user", "user123")
            .setIpAddress("192.168.1.5");
        server.handleRequest(request5);
    }
}
```
