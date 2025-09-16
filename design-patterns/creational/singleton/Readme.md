# Singleton

## 🎯 Intenção

O Singleton é um padrão de projeto criacional que garante que uma classe tenha apenas uma instância, fornecendo um ponto de acesso global a essa instância.

## 🚩 Problema

O padrão Singleton resolve dois problemas ao mesmo tempo (violando o Princípio da Responsabilidade Única):

### 1. **Garantir instância única**
Por que controlar quantas instâncias uma classe possui? A razão mais comum é controlar acesso a recursos compartilhados—como banco de dados ou arquivos.

**Comportamento especial**: Quando você tenta criar um "novo" objeto, recebe o objeto já existente.

### 2. **Fornecer acesso global**
Similar a variáveis globais, mas mais seguro. O Singleton protege a instância de ser sobrescrita por outro código.

**Problema das variáveis globais:**
```java
// Perigoso: pode ser sobrescrito
public static Database database = new Database();
// Qualquer código pode fazer: database = null; ❌
```

## ✅ Solução

Todas as implementações do Singleton têm dois passos em comum:

1. **Construtor privado**: Impede uso do operador `new`
2. **Método estático de criação**: Atua como construtor, retornando sempre a mesma instância

**Funcionamento:**
- Primeira chamada: cria e armazena o objeto
- Chamadas seguintes: retorna o objeto armazenado

## 🏗️ Estrutura

```
Singleton
├── - instance: Singleton (static)
├── - Singleton() (private constructor)
├── + getInstance(): Singleton (static)
├── + businessLogic()
```

### Componentes:
- **Campo estático privado**: Armazena a instância única
- **Construtor privado**: Impede criação externa
- **Método getInstance()**: Controla acesso à instância
- **Lógica de negócio**: Funcionalidades da classe

## 💻 Exemplos Práticos

### 1. Implementação Básica (Lazy Loading)
```java
public class Database {
    private static Database instance;
    private String connectionString;
    
    // Construtor privado
    private Database() {
        // Inicialização custosa
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Conexão com banco de dados criada");
    }
    
    // Método de acesso (não thread-safe)
    public static Database getInstance() {
        if (instance == null) {
            instance = new Database();
        }
        return instance;
    }
    
    public void query(String sql) {
        System.out.println("Executando: " + sql + " em " + connectionString);
    }
}
```

### 2. Implementação Thread-Safe (Double-Checked Locking)
```java
public class Database {
    // volatile garante visibilidade entre threads
    private static volatile Database instance;
    private String connectionString;
    
    private Database() {
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Conexão com banco de dados criada");
    }
    
    public static Database getInstance() {
        if (instance == null) {
            synchronized (Database.class) {
                // Dupla verificação para thread safety
                if (instance == null) {
                    instance = new Database();
                }
            }
        }
        return instance;
    }
    
    public void query(String sql) {
        System.out.println("Executando: " + sql + " em " + connectionString);
    }
}
```

### 3. Implementação com Enum (Mais Segura)
```java
public enum DatabaseConnection {
    INSTANCE;
    
    private String connectionString;
    
    DatabaseConnection() {
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Conexão com banco de dados criada");
    }
    
    public void query(String sql) {
        System.out.println("Executando: " + sql + " em " + connectionString);
    }
}
```

### 4. Implementação Eager Loading
```java
public class Logger {
    // Instância criada no carregamento da classe
    private static final Logger instance = new Logger();
    
    private Logger() {
        System.out.println("Logger inicializado");
    }
    
    public static Logger getInstance() {
        return instance;
    }
    
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
}
```

### Uso dos Exemplos:
```java
public class SingletonExample {
    public static void main(String[] args) {
        // Teste básico
        Database db1 = Database.getInstance();
        Database db2 = Database.getInstance();
        
        System.out.println("Same instance? " + (db1 == db2)); // true
        
        db1.query("SELECT * FROM users");
        db2.query("SELECT * FROM products");
        
        // Usando enum
        DatabaseConnection.INSTANCE.query("SELECT * FROM orders");
        
        // Logger
        Logger logger = Logger.getInstance();
        logger.log("Sistema iniciado");
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Controlar acesso a recursos compartilhados** (BD, arquivos, cache)
- **Substituir variáveis globais** de forma mais segura
- **Garantir uma única instância** por questões de performance ou integridade

### 📝 Exemplos de uso:
- **Connection Pool**: Gerenciador de conexões de banco
- **Cache Global**: Sistema de cache compartilhado
- **Logger**: Sistema de log único
- **Configuration Manager**: Configurações da aplicação

### ❌ Evite quando:
- **Apenas para agrupar funções** (use classes estáticas)
- **Testabilidade é crítica** (dificulta mocks)
- **Múltiplas instâncias podem ser úteis**

## 🚀 Como Implementar

1. **Adicione campo estático privado** para armazenar a instância
2. **Declare método público estático** para obter a instância
3. **Implemente "lazy initialization"** no método estático
4. **Torne o construtor privado**
5. **Substitua chamadas diretas** ao construtor por chamadas ao método getInstance

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Garantia de instância única**
- **Ponto de acesso global controlado**
- **Inicialização tardia** (lazy loading)
- **Controle sobre criação de objetos**

### ❌ Desvantagens:
- **Viola Princípio da Responsabilidade Única**
- **Pode mascarar design ruim** (acoplamento excessivo)
- **Complexidade em ambientes multi-thread**
- **Dificulta testes unitários** (hard to mock)
- **Problemas com herança**

## 🔗 Variações e Considerações

### Thread Safety:
| Abordagem | Pros | Contras |
|-----------|------|---------|
| **Lazy (sem sync)** | Simples, rápido | Não thread-safe |
| **Synchronized method** | Thread-safe | Performance ruim |
| **Double-checked locking** | Performance boa | Complexo |
| **Enum** | Mais seguro | Menos flexível |
| **Eager loading** | Simples, thread-safe | Pode desperdiçar memória |

### Problemas de Serialização:
```java
public class Singleton implements Serializable {
    private static Singleton instance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return instance;
    }
    
    // Previne criação de nova instância na deserialização
    private Object readResolve() {
        return instance;
    }
}
```

## 🔗 Relações com Outros Padrões

- **Facade**: Pode ser transformado em Singleton
- **Flyweight**: Similar, mas Flyweight pode ter múltiplas instâncias
- **Abstract Factory/Builder/Prototype**: Podem ser implementados como Singleton
- **State/Strategy**: Objetos de estado podem ser Singletons

## 📚 Conceitos-Chave para Lembrar

1. **Uma instância, acesso global**: Essência do padrão
2. **Construtor privado**: Impede criação externa
3. **Thread safety**: Consideração crítica em apps multi-thread
4. **Lazy vs Eager**: Quando criar a instância
5. **Enum é mais seguro**: Previne reflection e serialização
6. **Testabilidade**: Principal desvantagem do padrão

## 🔍 Analogia do Mundo Real

**Governo de um país**: Um país pode ter apenas um governo oficial. Independente das pessoas que formam o governo, o título "Governo de X" é um ponto de acesso global que identifica o grupo responsável. É impossível ter dois governos oficiais ao mesmo tempo.

## ⚠️ Considerações Importantes

### Anti-Patterns:
- **God Object**: Singleton que faz muitas coisas
- **Global State**: Abuso do acesso global
- **Hidden Dependencies**: Dependências não explícitas

### Alternativas:
- **Dependency Injection**: Container controla instâncias
- **Static Classes**: Para funcionalidades stateless
- **Factory Pattern**: Para controlar criação sem singleton

---

> **💡 Dica de Estudo:** Singleton é controverso - útil para recursos únicos, mas pode indicar design problem. Use com moderação e sempre considere alternativas como Dependency Injection.

> **📖 Referência:** [Refactoring Guru - Singleton](https://refactoring.guru/design-patterns/singleton)
