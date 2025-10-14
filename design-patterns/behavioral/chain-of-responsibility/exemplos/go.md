# Exemplos Práticos - Chain of Responsibility (Go)

## Exemplo 1: Sistema de Suporte Técnico

```go
package main

import "fmt"

// Struct de requisição
type SupportRequest struct {
	Type        string
	Priority    string
	Description string
	Customer    string
}

// Interface Handler
type SupportHandler interface {
	SetNext(handler SupportHandler) SupportHandler
	Handle(request *SupportRequest)
}

// Handler base
type BaseSupportHandler struct {
	next        SupportHandler
	handlerName string
}

func (h *BaseSupportHandler) SetNext(handler SupportHandler) SupportHandler {
	h.next = handler
	return handler
}

func (h *BaseSupportHandler) Handle(request *SupportRequest) {
	if h.next != nil {
		fmt.Printf("%s não pode resolver. Encaminhando...\n", h.handlerName)
		h.next.Handle(request)
	} else {
		fmt.Println("❌ Nenhum handler disponível para processar a requisição.")
	}
}

// Handler de nível 1 - Atendente básico
type BasicSupportHandler struct {
	BaseSupportHandler
}

func NewBasicSupportHandler() *BasicSupportHandler {
	return &BasicSupportHandler{
		BaseSupportHandler: BaseSupportHandler{
			handlerName: "Atendente Básico",
		},
	}
}

func (h *BasicSupportHandler) Handle(request *SupportRequest) {
	if h.canHandle(request) {
		h.process(request)
	} else {
		h.BaseSupportHandler.Handle(request)
	}
}

func (h *BasicSupportHandler) canHandle(request *SupportRequest) bool {
	return request.Type == "BASIC" && request.Priority == "LOW"
}

func (h *BasicSupportHandler) process(request *SupportRequest) {
	fmt.Printf("✅ %s resolveu o problema:\n", h.handlerName)
	fmt.Printf("   Cliente: %s\n", request.Customer)
	fmt.Printf("   Problema: %s\n", request.Description)
	fmt.Println("   Solução: Consultou FAQ e resolveu rapidamente.")
}

// Handler de nível 2 - Técnico especializado
type TechnicalSupportHandler struct {
	BaseSupportHandler
}

func NewTechnicalSupportHandler() *TechnicalSupportHandler {
	return &TechnicalSupportHandler{
		BaseSupportHandler: BaseSupportHandler{
			handlerName: "Técnico Especializado",
		},
	}
}

func (h *TechnicalSupportHandler) Handle(request *SupportRequest) {
	if h.canHandle(request) {
		h.process(request)
	} else {
		h.BaseSupportHandler.Handle(request)
	}
}

func (h *TechnicalSupportHandler) canHandle(request *SupportRequest) bool {
	return request.Type == "TECHNICAL" &&
		(request.Priority == "LOW" || request.Priority == "MEDIUM")
}

func (h *TechnicalSupportHandler) process(request *SupportRequest) {
	fmt.Printf("✅ %s resolveu o problema:\n", h.handlerName)
	fmt.Printf("   Cliente: %s\n", request.Customer)
	fmt.Printf("   Problema: %s\n", request.Description)
	fmt.Println("   Solução: Analisou logs e aplicou correção técnica.")
}

// Handler de nível 3 - Engenheiro sênior
type SeniorEngineerHandler struct {
	BaseSupportHandler
}

func NewSeniorEngineerHandler() *SeniorEngineerHandler {
	return &SeniorEngineerHandler{
		BaseSupportHandler: BaseSupportHandler{
			handlerName: "Engenheiro Sênior",
		},
	}
}

func (h *SeniorEngineerHandler) Handle(request *SupportRequest) {
	if h.canHandle(request) {
		h.process(request)
	} else {
		h.BaseSupportHandler.Handle(request)
	}
}

func (h *SeniorEngineerHandler) canHandle(request *SupportRequest) bool {
	return request.Type == "TECHNICAL" && request.Priority == "HIGH"
}

func (h *SeniorEngineerHandler) process(request *SupportRequest) {
	fmt.Printf("✅ %s resolveu o problema:\n", h.handlerName)
	fmt.Printf("   Cliente: %s\n", request.Customer)
	fmt.Printf("   Problema: %s\n", request.Description)
	fmt.Println("   Solução: Bug crítico corrigido com hotfix.")
}

// Handler de nível 4 - Gerente
type ManagerHandler struct {
	BaseSupportHandler
}

func NewManagerHandler() *ManagerHandler {
	return &ManagerHandler{
		BaseSupportHandler: BaseSupportHandler{
			handlerName: "Gerente",
		},
	}
}

func (h *ManagerHandler) Handle(request *SupportRequest) {
	if h.canHandle(request) {
		h.process(request)
	} else {
		h.BaseSupportHandler.Handle(request)
	}
}

func (h *ManagerHandler) canHandle(request *SupportRequest) bool {
	return request.Priority == "CRITICAL"
}

func (h *ManagerHandler) process(request *SupportRequest) {
	fmt.Printf("✅ %s assumiu o caso:\n", h.handlerName)
	fmt.Printf("   Cliente: %s\n", request.Customer)
	fmt.Printf("   Problema: %s\n", request.Description)
	fmt.Println("   Ação: Escalado para equipe prioritária, SLA ativado.")
}

// Sistema de suporte
type SupportSystem struct {
	handlerChain SupportHandler
}

func NewSupportSystem() *SupportSystem {
	// Monta a cadeia de handlers
	basic := NewBasicSupportHandler()
	technical := NewTechnicalSupportHandler()
	senior := NewSeniorEngineerHandler()
	manager := NewManagerHandler()

	basic.SetNext(technical)
	technical.SetNext(senior)
	senior.SetNext(manager)

	return &SupportSystem{
		handlerChain: basic,
	}
}

func (s *SupportSystem) SubmitRequest(request *SupportRequest) {
	fmt.Println("\n=== Nova Requisição de Suporte ===")
	fmt.Printf("Tipo: %s | Prioridade: %s\n", request.Type, request.Priority)
	s.handlerChain.Handle(request)
}

// Uso
func main() {
	system := NewSupportSystem()

	// Caso 1: Problema básico
	system.SubmitRequest(&SupportRequest{
		Type:        "BASIC",
		Priority:    "LOW",
		Description: "Como resetar minha senha?",
		Customer:    "João Silva",
	})

	// Caso 2: Problema técnico médio
	system.SubmitRequest(&SupportRequest{
		Type:        "TECHNICAL",
		Priority:    "MEDIUM",
		Description: "Aplicação travando ao carregar relatório",
		Customer:    "Maria Santos",
	})

	// Caso 3: Problema técnico crítico
	system.SubmitRequest(&SupportRequest{
		Type:        "TECHNICAL",
		Priority:    "HIGH",
		Description: "Sistema fora do ar - erro de banco de dados",
		Customer:    "TechCorp Ltd",
	})

	// Caso 4: Problema crítico de negócio
	system.SubmitRequest(&SupportRequest{
		Type:        "BUSINESS",
		Priority:    "CRITICAL",
		Description: "Perda de dados em transação financeira",
		Customer:    "BigBank S.A.",
	})

	// Caso 5: Requisição sem handler adequado
	system.SubmitRequest(&SupportRequest{
		Type:        "UNKNOWN",
		Priority:    "UNDEFINED",
		Description: "Tipo de problema não categorizado",
		Customer:    "Cliente X",
	})
}
```

## Exemplo 2: Middleware de Requisições HTTP

```go
package main

import (
	"fmt"
	"strings"
	"time"
)

// Struct de requisição HTTP
type HttpRequest struct {
	Method    string
	URL       string
	Headers   map[string]string
	Body      string
	Username  string
	Password  string
	IPAddress string
}

func NewHttpRequest(method, url string) *HttpRequest {
	return &HttpRequest{
		Method:  method,
		URL:     url,
		Headers: make(map[string]string),
	}
}

func (r *HttpRequest) SetAuth(username, password string) *HttpRequest {
	r.Username = username
	r.Password = password
	return r
}

func (r *HttpRequest) SetBody(body string) *HttpRequest {
	r.Body = body
	return r
}

func (r *HttpRequest) SetIPAddress(ip string) *HttpRequest {
	r.IPAddress = ip
	return r
}

// Struct de resposta HTTP
type HttpResponse struct {
	StatusCode int
	Body       string
	Success    bool
}

func NewHttpResponse(statusCode int, body string, success bool) *HttpResponse {
	return &HttpResponse{
		StatusCode: statusCode,
		Body:       body,
		Success:    success,
	}
}

// Interface Middleware
type Middleware interface {
	SetNext(middleware Middleware) Middleware
	Handle(request *HttpRequest) *HttpResponse
}

// Middleware base
type BaseMiddleware struct {
	next Middleware
}

func (m *BaseMiddleware) SetNext(middleware Middleware) Middleware {
	m.next = middleware
	return middleware
}

// Middleware de autenticação
type AuthenticationMiddleware struct {
	BaseMiddleware
	users map[string]string
}

func NewAuthenticationMiddleware() *AuthenticationMiddleware {
	return &AuthenticationMiddleware{
		users: map[string]string{
			"admin": "admin123",
			"user":  "user123",
		},
	}
}

func (m *AuthenticationMiddleware) Handle(request *HttpRequest) *HttpResponse {
	response := m.check(request)

	if !response.Success {
		return response
	}

	if m.next != nil {
		return m.next.Handle(request)
	}

	return m.processRequest(request)
}

func (m *AuthenticationMiddleware) check(request *HttpRequest) *HttpResponse {
	fmt.Println("🔐 [Auth] Verificando credenciais...")

	if request.Username == "" || request.Password == "" {
		fmt.Println("   ❌ Credenciais não fornecidas")
		return NewHttpResponse(401, "Autenticação necessária", false)
	}

	password, exists := m.users[request.Username]
	if !exists || password != request.Password {
		fmt.Println("   ❌ Credenciais inválidas")
		return NewHttpResponse(401, "Usuário ou senha incorretos", false)
	}

	fmt.Printf("   ✅ Autenticado como: %s\n", request.Username)
	return NewHttpResponse(200, "OK", true)
}

func (m *AuthenticationMiddleware) processRequest(request *HttpRequest) *HttpResponse {
	return NewHttpResponse(200, "Requisição processada com sucesso", true)
}

// Middleware de autorização
type AuthorizationMiddleware struct {
	BaseMiddleware
	adminUsers     map[string]bool
	restrictedPaths []string
}

func NewAuthorizationMiddleware() *AuthorizationMiddleware {
	return &AuthorizationMiddleware{
		adminUsers: map[string]bool{
			"admin": true,
		},
		restrictedPaths: []string{"/admin", "/config", "/users"},
	}
}

func (m *AuthorizationMiddleware) Handle(request *HttpRequest) *HttpResponse {
	response := m.check(request)

	if !response.Success {
		return response
	}

	if m.next != nil {
		return m.next.Handle(request)
	}

	return m.processRequest(request)
}

func (m *AuthorizationMiddleware) check(request *HttpRequest) *HttpResponse {
	fmt.Println("🔑 [Authorization] Verificando permissões...")

	isRestricted := false
	for _, path := range m.restrictedPaths {
		if strings.HasPrefix(request.URL, path) {
			isRestricted = true
			break
		}
	}

	if isRestricted && !m.adminUsers[request.Username] {
		fmt.Printf("   ❌ Acesso negado para: %s\n", request.URL)
		return NewHttpResponse(403, "Acesso proibido", false)
	}

	fmt.Println("   ✅ Acesso autorizado")
	return NewHttpResponse(200, "OK", true)
}

func (m *AuthorizationMiddleware) processRequest(request *HttpRequest) *HttpResponse {
	return NewHttpResponse(200, "Requisição processada com sucesso", true)
}

// Middleware de validação
type ValidationMiddleware struct {
	BaseMiddleware
}

func NewValidationMiddleware() *ValidationMiddleware {
	return &ValidationMiddleware{}
}

func (m *ValidationMiddleware) Handle(request *HttpRequest) *HttpResponse {
	response := m.check(request)

	if !response.Success {
		return response
	}

	if m.next != nil {
		return m.next.Handle(request)
	}

	return m.processRequest(request)
}

func (m *ValidationMiddleware) check(request *HttpRequest) *HttpResponse {
	fmt.Println("📋 [Validation] Validando requisição...")

	if request.Method == "" || request.URL == "" {
		fmt.Println("   ❌ Método ou URL ausentes")
		return NewHttpResponse(400, "Bad Request", false)
	}

	if request.Method == "POST" || request.Method == "PUT" {
		if request.Body == "" {
			fmt.Printf("   ❌ Body necessário para %s\n", request.Method)
			return NewHttpResponse(400, "Body é obrigatório", false)
		}
	}

	fmt.Println("   ✅ Requisição válida")
	return NewHttpResponse(200, "OK", true)
}

func (m *ValidationMiddleware) processRequest(request *HttpRequest) *HttpResponse {
	return NewHttpResponse(200, "Requisição processada com sucesso", true)
}

// Middleware de rate limiting
type RateLimitMiddleware struct {
	BaseMiddleware
	requestCounts   map[string]int
	lastRequestTime map[string]time.Time
	maxRequests     int
	timeWindow      time.Duration
}

func NewRateLimitMiddleware() *RateLimitMiddleware {
	return &RateLimitMiddleware{
		requestCounts:   make(map[string]int),
		lastRequestTime: make(map[string]time.Time),
		maxRequests:     5,
		timeWindow:      time.Minute,
	}
}

func (m *RateLimitMiddleware) Handle(request *HttpRequest) *HttpResponse {
	response := m.check(request)

	if !response.Success {
		return response
	}

	if m.next != nil {
		return m.next.Handle(request)
	}

	return m.processRequest(request)
}

func (m *RateLimitMiddleware) check(request *HttpRequest) *HttpResponse {
	fmt.Println("⏱️  [RateLimit] Verificando limite de requisições...")

	ip := request.IPAddress
	currentTime := time.Now()

	// Reseta contador se passou o tempo
	if lastTime, exists := m.lastRequestTime[ip]; exists {
		if currentTime.Sub(lastTime) > m.timeWindow {
			m.requestCounts[ip] = 0
		}
	}

	count := m.requestCounts[ip]

	if count >= m.maxRequests {
		fmt.Printf("   ❌ Limite excedido para IP: %s\n", ip)
		return NewHttpResponse(429, "Too Many Requests", false)
	}

	m.requestCounts[ip] = count + 1
	m.lastRequestTime[ip] = currentTime

	fmt.Printf("   ✅ Requisição permitida (%d/%d)\n", count+1, m.maxRequests)
	return NewHttpResponse(200, "OK", true)
}

func (m *RateLimitMiddleware) processRequest(request *HttpRequest) *HttpResponse {
	return NewHttpResponse(200, "Requisição processada com sucesso", true)
}

// Middleware de logging
type LoggingMiddleware struct {
	BaseMiddleware
}

func NewLoggingMiddleware() *LoggingMiddleware {
	return &LoggingMiddleware{}
}

func (m *LoggingMiddleware) Handle(request *HttpRequest) *HttpResponse {
	response := m.check(request)

	if !response.Success {
		return response
	}

	if m.next != nil {
		return m.next.Handle(request)
	}

	return m.processRequest(request)
}

func (m *LoggingMiddleware) check(request *HttpRequest) *HttpResponse {
	fmt.Println("📝 [Logging] Registrando requisição:")
	fmt.Printf("   Método: %s\n", request.Method)
	fmt.Printf("   URL: %s\n", request.URL)
	fmt.Printf("   IP: %s\n", request.IPAddress)
	fmt.Printf("   Usuário: %s\n", request.Username)

	return NewHttpResponse(200, "OK", true)
}

func (m *LoggingMiddleware) processRequest(request *HttpRequest) *HttpResponse {
	return NewHttpResponse(200, "Requisição processada com sucesso", true)
}

// Servidor HTTP
type HttpServer struct {
	middleware Middleware
}

func NewHttpServer() *HttpServer {
	// Monta a cadeia de middleware
	logging := NewLoggingMiddleware()
	rateLimit := NewRateLimitMiddleware()
	auth := NewAuthenticationMiddleware()
	authz := NewAuthorizationMiddleware()
	validation := NewValidationMiddleware()

	logging.SetNext(rateLimit)
	rateLimit.SetNext(auth)
	auth.SetNext(authz)
	authz.SetNext(validation)

	return &HttpServer{
		middleware: logging,
	}
}

func (s *HttpServer) HandleRequest(request *HttpRequest) {
	fmt.Println("\n" + strings.Repeat("=", 60))
	fmt.Printf("🌐 Processando requisição: %s %s\n", request.Method, request.URL)
	fmt.Println(strings.Repeat("=", 60))

	response := s.middleware.Handle(request)

	fmt.Println("\n📤 Resposta:")
	fmt.Printf("   Status: %d\n", response.StatusCode)
	fmt.Printf("   Body: %s\n", response.Body)
}

// Uso
func main() {
	server := NewHttpServer()

	// Caso 1: Requisição bem-sucedida
	request1 := NewHttpRequest("GET", "/api/products").
		SetAuth("user", "user123").
		SetIPAddress("192.168.1.1")
	server.HandleRequest(request1)

	// Caso 2: Falha de autenticação
	request2 := NewHttpRequest("GET", "/api/orders").
		SetAuth("user", "wrongpass").
		SetIPAddress("192.168.1.2")
	server.HandleRequest(request2)

	// Caso 3: Falha de autorização (usuário comum tentando acessar admin)
	request3 := NewHttpRequest("GET", "/admin/settings").
		SetAuth("user", "user123").
		SetIPAddress("192.168.1.3")
	server.HandleRequest(request3)

	// Caso 4: Sucesso com admin
	request4 := NewHttpRequest("POST", "/admin/users").
		SetAuth("admin", "admin123").
		SetBody("{\"name\":\"New User\"}").
		SetIPAddress("192.168.1.4")
	server.HandleRequest(request4)

	// Caso 5: Falha de validação (POST sem body)
	request5 := NewHttpRequest("POST", "/api/orders").
		SetAuth("user", "user123").
		SetIPAddress("192.168.1.5")
	server.HandleRequest(request5)
}
```
