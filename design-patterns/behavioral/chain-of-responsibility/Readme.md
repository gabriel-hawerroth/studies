# Chain of Responsibility

## 🎯 Intenção

O Chain of Responsibility é um padrão de projeto comportamental que permite passar requisições ao longo de uma cadeia de handlers (manipuladores). Ao receber uma requisição, cada handler decide se processa a requisição ou a passa para o próximo handler na cadeia.

## 🚩 Problema

Imagine que você está trabalhando em um sistema de pedidos online. Você quer restringir o acesso ao sistema para que apenas usuários autenticados possam criar pedidos. Além disso, usuários com permissões administrativas devem ter acesso completo a todos os pedidos.

### Resultado problemático:
```java
// Código monolítico com todas as verificações
public class OrderSystem {
    public void createOrder(Request request) {
        // Verificação 1: Autenticação
        if (!isAuthenticated(request)) {
            System.out.println("Não autenticado!");
            return;
        }
        
        // Verificação 2: Autorização
        if (!hasPermission(request)) {
            System.out.println("Sem permissão!");
            return;
        }
        
        // Verificação 3: Validação de dados
        if (!isValidData(request)) {
            System.out.println("Dados inválidos!");
            return;
        }
        
        // Verificação 4: Rate limiting
        if (isTooManyRequests(request)) {
            System.out.println("Muitas requisições!");
            return;
        }
        
        // Verificação 5: Cache
        if (hasCachedResponse(request)) {
            System.out.println("Retornando do cache");
            return;
        }
        
        // Finalmente processa o pedido
        processOrder(request);
    }
}
```

**Problemas:**
- **Código inchado**: Método gigante com muitas responsabilidades
- **Acoplamento alto**: Todas as verificações estão juntas
- **Difícil manutenção**: Mudanças afetam múltiplas partes
- **Reutilização impossível**: Não dá para usar verificações separadamente
- **Ordem fixa**: Não é possível reordenar verificações facilmente
- **Violação SRP**: Classe faz muitas coisas diferentes

## ✅ Solução

O padrão Chain of Responsibility sugere transformar comportamentos específicos em objetos autônomos chamados **handlers**. Cada verificação deve ser extraída para sua própria classe com um único método que realiza a verificação.

### Características-chave:
- **Handlers encadeados**: Cada handler tem referência ao próximo
- **Decisão local**: Cada handler decide se processa ou passa adiante
- **Desacoplamento**: Cliente não conhece a cadeia completa
- **Ordem dinâmica**: Cadeia pode ser montada em runtime

```java
// Handler abstrato
abstract class Handler {
    protected Handler next;
    
    public Handler setNext(Handler next) {
        this.next = next;
        return next;
    }
    
    public abstract boolean handle(Request request);
}

// Handlers concretos
class AuthenticationHandler extends Handler {
    public boolean handle(Request request) {
        if (!isAuthenticated(request)) {
            return false;
        }
        return next != null ? next.handle(request) : true;
    }
}

class AuthorizationHandler extends Handler {
    public boolean handle(Request request) {
        if (!hasPermission(request)) {
            return false;
        }
        return next != null ? next.handle(request) : true;
    }
}
```

## 📊 Estrutura

```
Client → Handler (interface)
         ↓
    BaseHandler (opcional)
         ↓
    ConcreteHandler1 → ConcreteHandler2 → ConcreteHandler3
         ↓                  ↓                  ↓
      [next]            [next]            [next]
```

### Componentes:
- **Handler**: Interface que declara método para manipular requisições
- **BaseHandler**: Classe opcional com código boilerplate comum
- **ConcreteHandlers**: Contém código real para processar requisições
- **Client**: Monta cadeia e envia requisições

### Exemplo de montagem da cadeia:
```java
// Cliente monta a cadeia
Handler chain = new AuthenticationHandler();
chain.setNext(new AuthorizationHandler())
     .setNext(new ValidationHandler())
     .setNext(new RateLimitHandler());

chain.handle(request);
```

## 🤔 Quando Usar

### ✅ Use quando:
- **Múltiplos handlers**: Vários objetos podem processar uma requisição
- **Ordem variável**: Handlers devem ser executados em ordem específica
- **Conjunto dinâmico**: Handlers e ordem podem mudar em runtime
- **Processamento sequencial**: Requisição passa por múltiplos estágios
- **Handler desconhecido**: Não se sabe qual handler processará

### 📝 Exemplos de aplicação:
- **Middleware HTTP**: Express.js, ASP.NET Core
- **Event bubbling**: DOM events, GUI frameworks
- **Filtros**: Servlet filters, Spring interceptors
- **Logging pipelines**: Log4j, Serilog
- **Validação**: Múltiplas validações em sequência

### ❌ Evite quando:
- **Handler único**: Só um objeto pode processar
- **Ordem irrelevante**: Não importa a sequência
- **Performance crítica**: Overhead de passar pela cadeia
- **Requisição deve ser processada**: Não pode ser ignorada

## 🚀 Como Implementar

1. **Declare interface handler** com método para processar requisições

2. **Crie classe base opcional** com campo para próximo handler

3. **Implemente handlers concretos**:
   - Decidir se processa requisição
   - Decidir se passa para próximo

4. **Cliente monta cadeia** ou recebe cadeia pronta

5. **Cliente envia requisição** para qualquer handler da cadeia

6. **Prepare-se para cenários**:
   - Cadeia com único link
   - Requisições não processadas
   - Requisições que chegam ao fim

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Controle de ordem**: Define sequência de processamento
- **Single Responsibility**: Cada handler faz uma coisa
- **Open/Closed**: Adiciona novos handlers sem quebrar código
- **Desacoplamento**: Cliente não conhece handlers concretos
- **Flexibilidade**: Monta cadeias diferentes dinamicamente

### ❌ Desvantagens:
- **Requisições não processadas**: Podem passar sem ser tratadas
- **Debugging difícil**: Difícil rastrear fluxo pela cadeia
- **Performance**: Overhead de passar por múltiplos handlers

## 🔗 Diferenças de Outros Padrões

| Padrão | Conexão | Processamento | Uso Principal |
|--------|---------|---------------|---------------|
| **Chain of Responsibility** | Sequencial dinâmica | Um handler processa | Pipeline de processamento |
| **Command** | Unidirecional | Executa comando | Encapsular operações |
| **Mediator** | Via mediador central | Mediador coordena | Comunicação entre objetos |
| **Observer** | Broadcast dinâmico | Todos observadores | Notificação de eventos |

## 🔗 Relações com Outros Padrões

- **Command**: Handlers podem ser implementados como Commands
- **Composite**: Frequentemente usado com Composite em estruturas de árvore
- **Decorator**: Similar estruturalmente, mas Decorator não pode quebrar fluxo
- **Mediator**: CoR passa sequencialmente, Mediator coordena entre múltiplos

## 📚 Conceitos-Chave para Lembrar

1. **Cadeia de handlers**: Handlers encadeados processam requisição
2. **Decisão local**: Cada handler decide se processa ou passa adiante
3. **Desacoplamento**: Cliente não conhece estrutura da cadeia
4. **Ordem importante**: Sequência afeta comportamento
5. **Parada opcional**: Handler pode parar propagação
6. **Configuração dinâmica**: Cadeia montada em runtime

## 🔍 Analogia do Mundo Real

**Suporte técnico por telefone**: Quando você liga para suporte, primeiro fala com um atendimento automatizado (handler 1), que tenta resolver com FAQ. Se não resolver, passa para atendente humano (handler 2). Se ainda não resolver, escala para técnico especializado (handler 3). Se for muito crítico, vai para engenheiro sênior (handler 4). Cada nível decide se consegue resolver ou passa adiante.

## ⚠️ Considerações Importantes

### Variações do padrão:

#### 1. **Passing Chain** (Cadeia que passa)
```java
// Todos os handlers processam
public boolean handle(Request request) {
    doSomething(request);
    if (next != null) {
        next.handle(request);
    }
    return true;
}
```

#### 2. **Stopping Chain** (Cadeia que para)
```java
// Para quando um handler processa
public boolean handle(Request request) {
    if (canHandle(request)) {
        process(request);
        return true; // Para aqui
    }
    return next != null && next.handle(request);
}
```

### Design Guidelines:
- **Interface consistente**: Todos handlers mesma interface
- **Imutabilidade**: Considere handlers imutáveis
- **Default behavior**: Handler base pode ter comportamento padrão
- **Error handling**: Defina como tratar erros na cadeia

### Montagem da cadeia:
```java
// Fluent interface para montar cadeia
Handler chain = new Handler1();
chain.setNext(new Handler2())
     .setNext(new Handler3())
     .setNext(new Handler4());

// Ou usando builder
HandlerChain chain = HandlerChain.builder()
    .addHandler(new Handler1())
    .addHandler(new Handler2())
    .build();
```

### Frameworks que usam:
- **Express.js**: Middleware chain
- **ASP.NET Core**: Request pipeline
- **Java Servlets**: Filter chain
- **Spring**: Interceptor chain

---

> **💡 Dica de Estudo:** Chain of Responsibility é como uma "linha de montagem de decisões" - cada estação (handler) olha o produto (requisição) e decide: "Eu resolvo isso" ou "Passa para o próximo". Use quando tiver múltiplos candidatos para processar uma requisição e não souber qual será antecipadamente.

> **📖 Referência:** [Refactoring Guru - Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)
