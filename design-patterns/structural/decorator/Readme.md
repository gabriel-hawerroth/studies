# Decorator

> **Também conhecido como:** Wrapper

## 🎯 Intenção

O Decorator é um padrão de projeto estrutural que permite anexar novos comportamentos a objetos colocando-os dentro de objetos wrapper especiais que contêm esses comportamentos. Ele estende funcionalidades sem alterar a estrutura original.

## 🚩 Problema

Imagine que você está trabalhando em uma biblioteca de notificação que permite que outros programas notifiquem seus usuários sobre eventos importantes.

A versão inicial da biblioteca era baseada na classe `Notifier` que tinha apenas alguns campos, um construtor e um método `send`. O método podia aceitar uma mensagem e enviá-la para uma lista de emails.

### Evolução do Problema:
1. **Inicial**: Apenas notificações por email
2. **Requisito**: Usuários querem SMS para questões críticas
3. **Mais requisitos**: Notificações no Facebook, Slack, etc.
4. **Problema final**: Como combinar múltiplos tipos de notificação?

```java
// Abordagem problemática com herança
class EmailNotifier { ... }
class SMSNotifier { ... }
class FacebookNotifier { ... }

// Combinações explodem exponencialmente
class EmailSMSNotifier { ... }
class EmailFacebookNotifier { ... }
class SMSFacebookNotifier { ... }
class EmailSMSFacebookNotifier { ... }
// Para N tipos = 2^N - 1 classes!
```

**Problemas da herança:**
- **Estática**: Não pode alterar comportamento em runtime
- **Limitada**: Subclasse tem apenas uma classe pai
- **Explosão combinatória**: N tipos = 2^N combinações

## ✅ Solução

O Decorator sugere usar **Agregação/Composição** ao invés de herança. Um "wrapper" é um objeto que pode ser vinculado a um objeto alvo. O wrapper contém os mesmos métodos que o alvo e delega todas as requisições para ele, mas pode alterar o resultado fazendo algo antes ou depois.

### Características-chave:
- **Mesmo interface**: Wrapper implementa a mesma interface do objeto original
- **Transparência**: Cliente não percebe a diferença
- **Empilhamento**: Pode envolver objeto em múltiplos wrappers
- **Runtime**: Comportamentos adicionados dinamicamente

```java
// Base simples
Notifier notifier = new EmailNotifier();

// Decorando com funcionalidades
notifier = new SMSDecorator(notifier);
notifier = new FacebookDecorator(notifier);
notifier = new SlackDecorator(notifier);

// Pilha: Slack -> Facebook -> SMS -> Email
notifier.send("Mensagem importante!");
```

## 🏗️ Estrutura

```
Component (interface)
├── operation()
│
├── ConcreteComponent implements Component
│   └── operation() // implementação base
│
└── BaseDecorator implements Component
    ├── component: Component (referência)
    ├── operation() // delega para component
    │
    └── ConcreteDecorator extends BaseDecorator
        └── operation() // comportamento extra + super.operation()
```

### Componentes:
- **Component**: Interface comum para wrappers e objetos originais
- **ConcreteComponent**: Classe dos objetos sendo decorados (comportamento básico)
- **BaseDecorator**: Classe base com campo para referenciar objeto wrapped
- **ConcreteDecorator**: Define comportamentos extras adicionados dinamicamente
- **Client**: Pode envolver componentes em múltiplas camadas de decorators

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Café

```java
// Component - interface comum
interface Coffee {
    String getDescription();
    double getCost();
}

// ConcreteComponent - café básico
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Café simples";
    }
    
    @Override
    public double getCost() {
        return 2.00;
    }
}

// BaseDecorator - decorator base
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
}

// ConcreteDecorators - decoradores específicos
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", com leite";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.50;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", com açúcar";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.25;
    }
}

class WhipCreamDecorator extends CoffeeDecorator {
    public WhipCreamDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", com chantilly";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.75;
    }
}

class ExtraShotDecorator extends CoffeeDecorator {
    public ExtraShotDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", shot extra";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 1.00;
    }
}

// Uso
public class CoffeeShopExample {
    public static void main(String[] args) {
        // Café básico
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + " - R$ " + coffee.getCost());
        
        // Adicionando leite
        coffee = new MilkDecorator(coffee);
        System.out.println(coffee.getDescription() + " - R$ " + coffee.getCost());
        
        // Adicionando açúcar
        coffee = new SugarDecorator(coffee);
        System.out.println(coffee.getDescription() + " - R$ " + coffee.getCost());
        
        // Adicionando chantilly
        coffee = new WhipCreamDecorator(coffee);
        System.out.println(coffee.getDescription() + " - R$ " + coffee.getCost());
        
        // Café especial completo
        Coffee specialCoffee = new ExtraShotDecorator(
            new WhipCreamDecorator(
                new MilkDecorator(
                    new SugarDecorator(
                        new SimpleCoffee()
                    )
                )
            )
        );
        
        System.out.println("\n" + specialCoffee.getDescription() + " - R$ " + specialCoffee.getCost());
        
        // Diferentes combinações
        Coffee[] orders = {
            new MilkDecorator(new SimpleCoffee()),
            new SugarDecorator(new MilkDecorator(new SimpleCoffee())),
            new ExtraShotDecorator(new ExtraShotDecorator(new SimpleCoffee()))
        };
        
        System.out.println("\nPedidos do dia:");
        for (int i = 0; i < orders.length; i++) {
            System.out.printf("Pedido %d: %s - R$ %.2f%n", 
                i + 1, orders[i].getDescription(), orders[i].getCost());
        }
    }
}
```

### Exemplo 2: Sistema de Processamento de Texto

```java
// Component - interface para processamento de texto
interface TextProcessor {
    String process(String text);
}

// ConcreteComponent - processador básico
class PlainTextProcessor implements TextProcessor {
    @Override
    public String process(String text) {
        return text;
    }
}

// BaseDecorator
abstract class TextProcessorDecorator implements TextProcessor {
    protected TextProcessor processor;
    
    public TextProcessorDecorator(TextProcessor processor) {
        this.processor = processor;
    }
    
    @Override
    public String process(String text) {
        return processor.process(text);
    }
}

// ConcreteDecorators
class UpperCaseDecorator extends TextProcessorDecorator {
    public UpperCaseDecorator(TextProcessor processor) {
        super(processor);
    }
    
    @Override
    public String process(String text) {
        return processor.process(text).toUpperCase();
    }
}

class BoldDecorator extends TextProcessorDecorator {
    public BoldDecorator(TextProcessor processor) {
        super(processor);
    }
    
    @Override
    public String process(String text) {
        return "**" + processor.process(text) + "**";
    }
}

class ItalicDecorator extends TextProcessorDecorator {
    public ItalicDecorator(TextProcessor processor) {
        super(processor);
    }
    
    @Override
    public String process(String text) {
        return "*" + processor.process(text) + "*";
    }
}

class UnderlineDecorator extends TextProcessorDecorator {
    public UnderlineDecorator(TextProcessor processor) {
        super(processor);
    }
    
    @Override
    public String process(String text) {
        return "_" + processor.process(text) + "_";
    }
}

class TimestampDecorator extends TextProcessorDecorator {
    public TimestampDecorator(TextProcessor processor) {
        super(processor);
    }
    
    @Override
    public String process(String text) {
        String timestamp = new java.util.Date().toString();
        return "[" + timestamp + "] " + processor.process(text);
    }
}

// Uso
public class TextProcessingExample {
    public static void main(String[] args) {
        String originalText = "Hello World";
        
        // Processamento básico
        TextProcessor processor = new PlainTextProcessor();
        System.out.println("Original: " + processor.process(originalText));
        
        // Adicionando formatações progressivamente
        processor = new BoldDecorator(processor);
        System.out.println("Bold: " + processor.process(originalText));
        
        processor = new ItalicDecorator(processor);
        System.out.println("Bold + Italic: " + processor.process(originalText));
        
        processor = new UpperCaseDecorator(processor);
        System.out.println("Bold + Italic + Upper: " + processor.process(originalText));
        
        // Processador com timestamp
        TextProcessor timestampProcessor = new TimestampDecorator(
            new UnderlineDecorator(
                new BoldDecorator(
                    new PlainTextProcessor()
                )
            )
        );
        
        System.out.println("\nCom timestamp: " + 
            timestampProcessor.process("Mensagem importante"));
        
        // Diferentes combinações
        System.out.println("\nDiferentes estilos:");
        
        TextProcessor[] styles = {
            new BoldDecorator(new PlainTextProcessor()),
            new ItalicDecorator(new PlainTextProcessor()),
            new UnderlineDecorator(new ItalicDecorator(new PlainTextProcessor())),
            new UpperCaseDecorator(new BoldDecorator(new ItalicDecorator(new PlainTextProcessor())))
        };
        
        String sampleText = "Design Patterns";
        for (int i = 0; i < styles.length; i++) {
            System.out.printf("Estilo %d: %s%n", i + 1, styles[i].process(sampleText));
        }
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Comportamentos extras em runtime** sem quebrar código existente
- **Herança é inadequada** (classes finais, múltiplas extensões)
- **Combinações flexíveis** de funcionalidades
- **Responsabilidades opcionais** que podem ser adicionadas/removidas

### 📝 Exemplos de uso:
- **UI Components**: Bordas, scrollbars, sombras
- **Streams de dados**: Compressão, criptografia, buffering
- **Middleware**: Autenticação, logging, caching
- **Validação**: Múltiplos validadores encadeados

### ❌ Evite quando:
- **Hierarquia simples**: Herança é suficiente
- **Comportamento único**: Não precisa de combinações
- **Performance crítica**: Múltiplas camadas podem impactar

## 🚀 Como Implementar

1. **Verifique se o domínio** pode ser representado como componente primário + camadas opcionais

2. **Identifique métodos comuns** entre componente e camadas opcionais

3. **Crie interface Component** e declare esses métodos

4. **Crie classe ConcreteComponent** com comportamento base

5. **Crie BaseDecorator** com campo para objeto wrapped (tipo Component)

6. **Garanta que todas as classes** implementem a interface Component

7. **Crie ConcreteDecorators** estendendo BaseDecorator

8. **Cliente compõe decorators** conforme necessário

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Extensão sem subclasse**: Comportamento dinâmico
- **Runtime flexibility**: Adicionar/remover responsabilidades
- **Múltiplos comportamentos**: Combinação de decorators
- **Princípio da Responsabilidade Única**: Divide classe monolítica

### ❌ Desvantagens:
- **Difícil remover wrapper específico** da pilha
- **Dependência de ordem**: Comportamento pode depender da sequência
- **Código de configuração**: Pode ficar complexo
- **Debugging**: Pilha de wrappers pode dificultar depuração

## 🔗 Diferenças de Outros Padrões

| Padrão | Interface | Estrutura | Propósito |
|--------|-----------|-----------|-----------|
| **Decorator** | Mesma ou estendida | Composição linear | Adicionar responsabilidades |
| **Adapter** | Completamente diferente | Wrapper simples | Compatibilizar interfaces |
| **Composite** | Mesma | Árvore recursiva | Tratar grupos como objetos |
| **Proxy** | Mesma | Wrapper simples | Controlar acesso |

## 🔗 Relações com Outros Padrões

- **Composite**: Estrutura similar, mas Decorator tem um filho e Composite muitos
- **Chain of Responsibility**: Estrutura similar, mas CoR pode parar a execução
- **Strategy**: Decorator muda "skin", Strategy muda "guts"
- **Proxy**: Estrutura similar, mas Proxy gerencia ciclo de vida do objeto

## 📚 Conceitos-Chave para Lembrar

1. **Wrapper transparente**: Cliente não percebe a decoração
2. **Composição > Herança**: Flexibilidade em runtime
3. **Interface consistente**: Todos implementam a mesma interface
4. **Empilhamento**: Múltiplos decorators podem ser combinados
5. **Responsabilidade única**: Cada decorator tem uma função específica
6. **Delegação**: Decorator delega para o objeto wrapped

## 🔍 Analogia do Mundo Real

**Vestir roupas**: Quando está frio, você coloca um suéter. Se ainda estiver frio, pode colocar uma jaqueta por cima. Se estiver chovendo, pode colocar uma capa de chuva. Todas essas roupas "estendem" seu comportamento básico, mas não são parte de você, e você pode facilmente tirar qualquer peça quando não precisar mais.

## ⚠️ Considerações Importantes

### Ordem dos Decorators:
```java
// Diferentes ordens, resultados diferentes
Text text1 = new UpperCase(new Bold("hello"));     // **HELLO**
Text text2 = new Bold(new UpperCase("hello"));     // **HELLO**
// Pode ou não fazer diferença dependendo da implementação
```

### Performance:
- Cada decorator adiciona uma camada de indireção
- Para operações simples, pode ser overhead
- Considere caching quando apropriado

### Alternativas modernas:
- **Functional interfaces** (Java 8+)
- **Composition functions** (linguagens funcionais)
- **Aspect-Oriented Programming** (AOP)

---

> **💡 Dica de Estudo:** Decorator é como uma "cebola" - você vai adicionando camadas de funcionalidade. Cada camada mantém a interface original, mas adiciona algo novo. Use quando quiser combinar comportamentos flexivelmente em runtime.

> **📖 Referência:** [Refactoring Guru - Decorator](https://refactoring.guru/design-patterns/decorator)
