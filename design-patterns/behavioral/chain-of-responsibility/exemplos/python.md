# Exemplos Práticos - Chain of Responsibility (Python)

## Exemplo 1: Sistema de Suporte Técnico

```python
from abc import ABC, abstractmethod
from typing import Optional

# Classe de requisição
class SupportRequest:
    def __init__(self, type: str, priority: str, description: str, customer: str):
        self.type = type
        self.priority = priority
        self.description = description
        self.customer = customer

# Handler abstrato
class SupportHandler(ABC):
    def __init__(self, handler_name: str):
        self._next_handler: Optional[SupportHandler] = None
        self._handler_name = handler_name
    
    def set_next(self, handler: 'SupportHandler') -> 'SupportHandler':
        self._next_handler = handler
        return handler
    
    def handle(self, request: SupportRequest) -> None:
        if self._can_handle(request):
            self._process(request)
        elif self._next_handler:
            print(f"{self._handler_name} não pode resolver. Encaminhando...")
            self._next_handler.handle(request)
        else:
            print("❌ Nenhum handler disponível para processar a requisição.")
    
    @abstractmethod
    def _can_handle(self, request: SupportRequest) -> bool:
        pass
    
    @abstractmethod
    def _process(self, request: SupportRequest) -> None:
        pass

# Handler de nível 1 - Atendente básico
class BasicSupportHandler(SupportHandler):
    def __init__(self):
        super().__init__("Atendente Básico")
    
    def _can_handle(self, request: SupportRequest) -> bool:
        return request.type == "BASIC" and request.priority == "LOW"
    
    def _process(self, request: SupportRequest) -> None:
        print(f"✅ {self._handler_name} resolveu o problema:")
        print(f"   Cliente: {request.customer}")
        print(f"   Problema: {request.description}")
        print("   Solução: Consultou FAQ e resolveu rapidamente.")

# Handler de nível 2 - Técnico especializado
class TechnicalSupportHandler(SupportHandler):
    def __init__(self):
        super().__init__("Técnico Especializado")
    
    def _can_handle(self, request: SupportRequest) -> bool:
        return (request.type == "TECHNICAL" and 
                request.priority in ["LOW", "MEDIUM"])
    
    def _process(self, request: SupportRequest) -> None:
        print(f"✅ {self._handler_name} resolveu o problema:")
        print(f"   Cliente: {request.customer}")
        print(f"   Problema: {request.description}")
        print("   Solução: Analisou logs e aplicou correção técnica.")

# Handler de nível 3 - Engenheiro sênior
class SeniorEngineerHandler(SupportHandler):
    def __init__(self):
        super().__init__("Engenheiro Sênior")
    
    def _can_handle(self, request: SupportRequest) -> bool:
        return request.type == "TECHNICAL" and request.priority == "HIGH"
    
    def _process(self, request: SupportRequest) -> None:
        print(f"✅ {self._handler_name} resolveu o problema:")
        print(f"   Cliente: {request.customer}")
        print(f"   Problema: {request.description}")
        print("   Solução: Bug crítico corrigido com hotfix.")

# Handler de nível 4 - Gerente
class ManagerHandler(SupportHandler):
    def __init__(self):
        super().__init__("Gerente")
    
    def _can_handle(self, request: SupportRequest) -> bool:
        return request.priority == "CRITICAL"
    
    def _process(self, request: SupportRequest) -> None:
        print(f"✅ {self._handler_name} assumiu o caso:")
        print(f"   Cliente: {request.customer}")
        print(f"   Problema: {request.description}")
        print("   Ação: Escalado para equipe prioritária, SLA ativado.")

# Sistema de suporte
class SupportSystem:
    def __init__(self):
        # Monta a cadeia de handlers
        self._handler_chain = BasicSupportHandler()
        self._handler_chain.set_next(TechnicalSupportHandler()) \
                          .set_next(SeniorEngineerHandler()) \
                          .set_next(ManagerHandler())
    
    def submit_request(self, request: SupportRequest) -> None:
        print("\n=== Nova Requisição de Suporte ===")
        print(f"Tipo: {request.type} | Prioridade: {request.priority}")
        self._handler_chain.handle(request)

# Uso
if __name__ == "__main__":
    system = SupportSystem()
    
    # Caso 1: Problema básico
    system.submit_request(SupportRequest(
        "BASIC", "LOW",
        "Como resetar minha senha?",
        "João Silva"
    ))
    
    # Caso 2: Problema técnico médio
    system.submit_request(SupportRequest(
        "TECHNICAL", "MEDIUM",
        "Aplicação travando ao carregar relatório",
        "Maria Santos"
    ))
    
    # Caso 3: Problema técnico crítico
    system.submit_request(SupportRequest(
        "TECHNICAL", "HIGH",
        "Sistema fora do ar - erro de banco de dados",
        "TechCorp Ltd"
    ))
    
    # Caso 4: Problema crítico de negócio
    system.submit_request(SupportRequest(
        "BUSINESS", "CRITICAL",
        "Perda de dados em transação financeira",
        "BigBank S.A."
    ))
    
    # Caso 5: Requisição sem handler adequado
    system.submit_request(SupportRequest(
        "UNKNOWN", "UNDEFINED",
        "Tipo de problema não categorizado",
        "Cliente X"
    ))
```

## Exemplo 2: Middleware de Requisições HTTP

```python
from abc import ABC, abstractmethod
from typing import Optional, Dict
from datetime import datetime, timedelta

# Classe de requisição HTTP
class HttpRequest:
    def __init__(self, method: str, url: str):
        self.method = method
        self.url = url
        self.headers: Dict[str, str] = {}
        self.body: Optional[str] = None
        self.username: Optional[str] = None
        self.password: Optional[str] = None
        self.ip_address: Optional[str] = None
    
    def set_auth(self, username: str, password: str) -> 'HttpRequest':
        self.username = username
        self.password = password
        return self
    
    def set_body(self, body: str) -> 'HttpRequest':
        self.body = body
        return self
    
    def set_ip_address(self, ip: str) -> 'HttpRequest':
        self.ip_address = ip
        return self

# Classe de resposta HTTP
class HttpResponse:
    def __init__(self, status_code: int, body: str, success: bool):
        self.status_code = status_code
        self.body = body
        self.success = success

# Middleware abstrato
class Middleware(ABC):
    def __init__(self):
        self._next: Optional[Middleware] = None
    
    def set_next(self, middleware: 'Middleware') -> 'Middleware':
        self._next = middleware
        return middleware
    
    def handle(self, request: HttpRequest) -> HttpResponse:
        response = self._check(request)
        
        if not response.success:
            return response  # Para a cadeia se falhar
        
        if self._next:
            return self._next.handle(request)
        
        # Fim da cadeia - processa requisição
        return self._process_request(request)
    
    @abstractmethod
    def _check(self, request: HttpRequest) -> HttpResponse:
        pass
    
    def _process_request(self, request: HttpRequest) -> HttpResponse:
        return HttpResponse(200, "Requisição processada com sucesso", True)

# Middleware de autenticação
class AuthenticationMiddleware(Middleware):
    def __init__(self):
        super().__init__()
        self._users = {
            "admin": "admin123",
            "user": "user123"
        }
    
    def _check(self, request: HttpRequest) -> HttpResponse:
        print("🔐 [Auth] Verificando credenciais...")
        
        if not request.username or not request.password:
            print("   ❌ Credenciais não fornecidas")
            return HttpResponse(401, "Autenticação necessária", False)
        
        if (request.username not in self._users or 
            self._users[request.username] != request.password):
            print("   ❌ Credenciais inválidas")
            return HttpResponse(401, "Usuário ou senha incorretos", False)
        
        print(f"   ✅ Autenticado como: {request.username}")
        return HttpResponse(200, "OK", True)

# Middleware de autorização
class AuthorizationMiddleware(Middleware):
    def __init__(self):
        super().__init__()
        self._admin_users = {"admin"}
        self._restricted_paths = {"/admin", "/config", "/users"}
    
    def _check(self, request: HttpRequest) -> HttpResponse:
        print("🔑 [Authorization] Verificando permissões...")
        
        is_restricted = any(
            request.url.startswith(path) 
            for path in self._restricted_paths
        )
        
        if is_restricted and request.username not in self._admin_users:
            print(f"   ❌ Acesso negado para: {request.url}")
            return HttpResponse(403, "Acesso proibido", False)
        
        print("   ✅ Acesso autorizado")
        return HttpResponse(200, "OK", True)

# Middleware de validação
class ValidationMiddleware(Middleware):
    def _check(self, request: HttpRequest) -> HttpResponse:
        print("📋 [Validation] Validando requisição...")
        
        if not request.method or not request.url:
            print("   ❌ Método ou URL ausentes")
            return HttpResponse(400, "Bad Request", False)
        
        if request.method in ["POST", "PUT"]:
            if not request.body:
                print(f"   ❌ Body necessário para {request.method}")
                return HttpResponse(400, "Body é obrigatório", False)
        
        print("   ✅ Requisição válida")
        return HttpResponse(200, "OK", True)

# Middleware de rate limiting
class RateLimitMiddleware(Middleware):
    def __init__(self):
        super().__init__()
        self._request_counts: Dict[str, int] = {}
        self._last_request_time: Dict[str, datetime] = {}
        self._max_requests = 5
        self._time_window = timedelta(minutes=1)
    
    def _check(self, request: HttpRequest) -> HttpResponse:
        print("⏱️  [RateLimit] Verificando limite de requisições...")
        
        ip = request.ip_address
        current_time = datetime.now()
        
        # Reseta contador se passou o tempo
        if ip in self._last_request_time:
            time_passed = current_time - self._last_request_time[ip]
            if time_passed > self._time_window:
                self._request_counts[ip] = 0
        
        count = self._request_counts.get(ip, 0)
        
        if count >= self._max_requests:
            print(f"   ❌ Limite excedido para IP: {ip}")
            return HttpResponse(429, "Too Many Requests", False)
        
        self._request_counts[ip] = count + 1
        self._last_request_time[ip] = current_time
        
        print(f"   ✅ Requisição permitida ({count + 1}/{self._max_requests})")
        return HttpResponse(200, "OK", True)

# Middleware de logging
class LoggingMiddleware(Middleware):
    def _check(self, request: HttpRequest) -> HttpResponse:
        print("📝 [Logging] Registrando requisição:")
        print(f"   Método: {request.method}")
        print(f"   URL: {request.url}")
        print(f"   IP: {request.ip_address}")
        print(f"   Usuário: {request.username}")
        
        return HttpResponse(200, "OK", True)

# Servidor HTTP
class HttpServer:
    def __init__(self):
        # Monta a cadeia de middleware
        self._middleware = LoggingMiddleware()
        self._middleware.set_next(RateLimitMiddleware()) \
                       .set_next(AuthenticationMiddleware()) \
                       .set_next(AuthorizationMiddleware()) \
                       .set_next(ValidationMiddleware())
    
    def handle_request(self, request: HttpRequest) -> None:
        print("\n" + "=" * 60)
        print(f"🌐 Processando requisição: {request.method} {request.url}")
        print("=" * 60)
        
        response = self._middleware.handle(request)
        
        print("\n📤 Resposta:")
        print(f"   Status: {response.status_code}")
        print(f"   Body: {response.body}")

# Uso
if __name__ == "__main__":
    server = HttpServer()
    
    # Caso 1: Requisição bem-sucedida
    request1 = HttpRequest("GET", "/api/products") \
        .set_auth("user", "user123") \
        .set_ip_address("192.168.1.1")
    server.handle_request(request1)
    
    # Caso 2: Falha de autenticação
    request2 = HttpRequest("GET", "/api/orders") \
        .set_auth("user", "wrongpass") \
        .set_ip_address("192.168.1.2")
    server.handle_request(request2)
    
    # Caso 3: Falha de autorização (usuário comum tentando acessar admin)
    request3 = HttpRequest("GET", "/admin/settings") \
        .set_auth("user", "user123") \
        .set_ip_address("192.168.1.3")
    server.handle_request(request3)
    
    # Caso 4: Sucesso com admin
    request4 = HttpRequest("POST", "/admin/users") \
        .set_auth("admin", "admin123") \
        .set_body('{"name":"New User"}') \
        .set_ip_address("192.168.1.4")
    server.handle_request(request4)
    
    # Caso 5: Falha de validação (POST sem body)
    request5 = HttpRequest("POST", "/api/orders") \
        .set_auth("user", "user123") \
        .set_ip_address("192.168.1.5")
    server.handle_request(request5)
```
