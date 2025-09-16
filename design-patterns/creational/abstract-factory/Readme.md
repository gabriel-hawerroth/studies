# Abstract Factory

## 🎯 Intenção

O Abstract Factory é um padrão de projeto criacional que permite produzir famílias de objetos relacionados sem especificar suas classes concretas. Ele garante que os produtos criados sejam compatíveis entre si.

## 🚩 Problema

Imagine que você está criando um simulador de loja de móveis. Seu código consiste em classes que representam:

1. **Uma família de produtos relacionados**: `Chair` + `Sofa` + `CoffeeTable`
2. **Várias variantes dessa família**: `Modern`, `Victorian`, `ArtDeco`

**O problema:** Você precisa criar objetos individuais de móveis que combinem com outros objetos da mesma família. Os clientes ficam irritados quando recebem móveis que não combinam (ex: sofá moderno com cadeira vitoriana).

Além disso, você não quer alterar o código existente toda vez que adicionar novos produtos ou famílias de produtos.

## ✅ Solução

O padrão Abstract Factory sugere:

1. **Declarar interfaces** para cada produto distinto da família (ex: `Chair`, `Sofa`, `CoffeeTable`)
2. **Criar uma Abstract Factory** - interface com métodos de criação para todos os produtos da família
3. **Implementar fábricas concretas** para cada variante, que criam apenas produtos compatíveis

**Principais conceitos:**
- Todas as variantes de um produto seguem a mesma interface
- Cada fábrica concreta corresponde a uma variante específica
- O código cliente trabalha apenas com interfaces abstratas

## 🏗️ Estrutura

```
AbstractFactory
├── createProductA(): AbstractProductA
├── createProductB(): AbstractProductB
│
ConcreteFactory1 implements AbstractFactory
├── createProductA(): ProductA1
├── createProductB(): ProductB1
│
ConcreteFactory2 implements AbstractFactory
├── createProductA(): ProductA2
├── createProductB(): ProductB2

AbstractProductA          AbstractProductB
├── ProductA1             ├── ProductB1
├── ProductA2             ├── ProductB2
```

### Componentes:
- **AbstractProducts**: Interfaces para produtos distintos mas relacionados
- **ConcreteProducts**: Implementações específicas agrupadas por variantes
- **AbstractFactory**: Interface que declara métodos de criação para todos os produtos
- **ConcreteFactories**: Implementam métodos de criação para uma variante específica
- **Client**: Trabalha apenas com interfaces abstratas

## 💻 Exemplo Prático

```java
// Interfaces dos produtos abstratos
interface Button {
    void paint();
    void onClick();
}

interface Checkbox {
    void paint();
    void toggle();
}

// Produtos concretos - Família Windows
class WindowsButton implements Button {
    @Override
    public void paint() {
        System.out.println("Renderizando botão no estilo Windows");
    }
    
    @Override
    public void onClick() {
        System.out.println("Clique detectado - evento Windows");
    }
}

class WindowsCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("Renderizando checkbox no estilo Windows");
    }
    
    @Override
    public void toggle() {
        System.out.println("Checkbox Windows alternado");
    }
}

// Produtos concretos - Família MacOS
class MacButton implements Button {
    @Override
    public void paint() {
        System.out.println("Renderizando botão no estilo MacOS");
    }
    
    @Override
    public void onClick() {
        System.out.println("Clique detectado - evento MacOS");
    }
}

class MacCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("Renderizando checkbox no estilo MacOS");
    }
    
    @Override
    public void toggle() {
        System.out.println("Checkbox MacOS alternado");
    }
}

// Abstract Factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Fábricas concretas
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Cliente
class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    public void paint() {
        button.paint();
        checkbox.paint();
    }
}

// Uso
public class AbstractFactoryExample {
    public static void main(String[] args) {
        GUIFactory factory;
        String osName = System.getProperty("os.name").toLowerCase();
        
        if (osName.contains("mac")) {
            factory = new MacFactory();
        } else {
            factory = new WindowsFactory();
        }
        
        Application app = new Application(factory);
        app.paint();
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Trabalhar com famílias de produtos relacionados** sem depender de suas classes concretas
- **Ter uma classe com vários Factory Methods** que obscurecem sua responsabilidade principal
- **Precisar garantir compatibilidade** entre produtos criados

### 📝 Exemplo de caso de uso:
Sistema multi-plataforma que precisa criar elementos de UI (botões, checkboxes, menus) que sejam consistentes com o sistema operacional atual.

## 🚀 Como Implementar

1. **Mapeie uma matriz** de tipos de produtos distintos vs variantes
2. **Declare interfaces** para todos os tipos de produto
3. **Declare a interface Abstract Factory** com métodos de criação
4. **Implemente fábricas concretas** para cada variante
5. **Crie código de inicialização** que instancia a fábrica adequada
6. **Substitua chamadas diretas** aos construtores por chamadas à fábrica

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Compatibilidade garantida** entre produtos da mesma família
- **Desacoplamento** entre produtos concretos e código cliente
- **Princípio da Responsabilidade Única**: criação em um local
- **Princípio Aberto/Fechado**: novas variantes sem quebrar código existente

### ❌ Desvantagens:
- **Complexidade**: muitas interfaces e classes novas
- **Overhead**: pode ser excessivo para casos simples

## 🔗 Diferenças do Factory Method

| Aspecto | Factory Method | Abstract Factory |
|---------|----------------|------------------|
| **Foco** | Criação de um produto | Famílias de produtos relacionados |
| **Estrutura** | Uma classe criadora | Interface com múltiplos métodos |
| **Flexibilidade** | Subclasses definem o produto | Troca toda a família de uma vez |
| **Complexidade** | Mais simples | Mais complexo |

## 🔗 Relações com Outros Padrões

- **Factory Method**: Muitos designs começam com Factory Method e evoluem para Abstract Factory
- **Builder**: Foca na construção passo a passo vs criação imediata de famílias
- **Prototype**: Pode ser usado para compor métodos em fábricas abstratas
- **Singleton**: Abstract Factories podem ser implementadas como Singleton

## 📚 Conceitos-Chave para Lembrar

1. **Famílias de produtos**: Criar grupos de objetos relacionados
2. **Compatibilidade**: Produtos da mesma fábrica são sempre compatíveis
3. **Intercambiabilidade**: Trocar toda uma família alterando apenas a fábrica
4. **Interfaces abstratas**: Cliente não conhece implementações concretas

---

> **💡 Dica de Estudo:** Use Abstract Factory quando precisar criar "kits" completos de produtos relacionados. É como ter uma loja especializada que vende apenas produtos de uma marca/estilo específico.

> **📖 Referência:** [Refactoring Guru - Abstract Factory](https://refactoring.guru/design-patterns/abstract-factory)
