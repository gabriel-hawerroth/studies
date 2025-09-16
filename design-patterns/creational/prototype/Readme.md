# Prototype

> **Também conhecido como:** Clone

## 🎯 Intenção

O Prototype é um padrão de projeto criacional que permite copiar objetos existentes sem tornar seu código dependente de suas classes. O padrão delega o processo de clonagem para os próprios objetos que estão sendo clonados.

## 🚩 Problema

Imagine que você tem um objeto e quer criar uma cópia exata dele. Como fazer isso?

### Problemas da abordagem direta:

1. **Campos privados**: Nem todos os campos são acessíveis de fora da classe
2. **Dependência da classe concreta**: Seu código fica acoplado à classe específica
3. **Interface desconhecida**: Às vezes você só conhece a interface, não a classe concreta

**Exemplo problemático:**
```java
// Você precisa conhecer a classe concreta para copiar
if (object instanceof Circle) {
    Circle circle = (Circle) object;
    Circle copy = new Circle();
    copy.setRadius(circle.getRadius());
    copy.setX(circle.getX());
    // ... copiar todos os campos manualmente
}
// Código se torna dependente de classes específicas
```

## ✅ Solução

O padrão Prototype sugere declarar uma interface comum para todos os objetos que suportam clonagem. Essa interface permite clonar um objeto sem acoplar seu código à classe desse objeto.

**Principais conceitos:**
- **Interface de clonagem**: Geralmente contém apenas um método `clone()`
- **Auto-clonagem**: Objetos são responsáveis por sua própria clonagem
- **Acesso a campos privados**: Objetos podem acessar campos privados de outros objetos da mesma classe
- **Protótipos pré-construídos**: Alternativa à criação de subclasses

## 🏗️ Estrutura

### Implementação Básica:
```
Prototype (Interface)
├── clone(): Prototype

ConcretePrototype1 implements Prototype
├── field1, field2, field3...
├── ConcretePrototype1(source: ConcretePrototype1)
├── clone(): Prototype

ConcretePrototype2 implements Prototype
├── fieldA, fieldB, fieldC...
├── ConcretePrototype2(source: ConcretePrototype2)
├── clone(): Prototype
```

### Com Registry (Opcional):
```
PrototypeRegistry
├── prototypes: Map<String, Prototype>
├── addPrototype(name: String, prototype: Prototype)
├── getPrototype(name: String): Prototype
```

### Componentes:
- **Prototype**: Interface que declara método de clonagem
- **ConcretePrototype**: Implementa o método de clonagem
- **Client**: Cria novos objetos clonando protótipos
- **PrototypeRegistry**: Cataloga protótipos frequentemente usados (opcional)

## 💻 Exemplo Prático

```java
// Interface Prototype
interface Shape {
    Shape clone();
    void draw();
}

// Classe base abstrata
abstract class BaseShape implements Shape {
    protected int x;
    protected int y;
    protected String color;
    
    public BaseShape() {}
    
    // Construtor de cópia
    public BaseShape(BaseShape source) {
        if (source != null) {
            this.x = source.x;
            this.y = source.y;
            this.color = source.color;
        }
    }
    
    // Getters e setters...
    public void setX(int x) { this.x = x; }
    public void setY(int y) { this.y = y; }
    public void setColor(String color) { this.color = color; }
}

// Protótipo concreto - Círculo
class Circle extends BaseShape {
    private int radius;
    
    public Circle() {}
    
    // Construtor de cópia
    public Circle(Circle source) {
        super(source);
        if (source != null) {
            this.radius = source.radius;
        }
    }
    
    @Override
    public Shape clone() {
        return new Circle(this);
    }
    
    @Override
    public void draw() {
        System.out.printf("Círculo: pos=(%d,%d), cor=%s, raio=%d%n", 
                         x, y, color, radius);
    }
    
    public void setRadius(int radius) { this.radius = radius; }
    public int getRadius() { return radius; }
}

// Protótipo concreto - Retângulo
class Rectangle extends BaseShape {
    private int width;
    private int height;
    
    public Rectangle() {}
    
    // Construtor de cópia
    public Rectangle(Rectangle source) {
        super(source);
        if (source != null) {
            this.width = source.width;
            this.height = source.height;
        }
    }
    
    @Override
    public Shape clone() {
        return new Rectangle(this);
    }
    
    @Override
    public void draw() {
        System.out.printf("Retângulo: pos=(%d,%d), cor=%s, dim=%dx%d%n", 
                         x, y, color, width, height);
    }
    
    public void setWidth(int width) { this.width = width; }
    public void setHeight(int height) { this.height = height; }
}

// Registry de protótipos (opcional)
class ShapeRegistry {
    private Map<String, Shape> shapes = new HashMap<>();
    
    public void addShape(String key, Shape shape) {
        shapes.put(key, shape);
    }
    
    public Shape getShape(String key) {
        Shape prototype = shapes.get(key);
        return prototype != null ? prototype.clone() : null;
    }
    
    public void loadPredefinedShapes() {
        Circle circle = new Circle();
        circle.setX(10);
        circle.setY(10);
        circle.setRadius(5);
        circle.setColor("Azul");
        addShape("círculo-azul", circle);
        
        Rectangle rectangle = new Rectangle();
        rectangle.setX(20);
        rectangle.setY(20);
        rectangle.setWidth(10);
        rectangle.setHeight(5);
        rectangle.setColor("Vermelho");
        addShape("retângulo-vermelho", rectangle);
    }
}

// Uso
public class PrototypeExample {
    public static void main(String[] args) {
        // Clonagem direta
        Circle originalCircle = new Circle();
        originalCircle.setX(5);
        originalCircle.setY(5);
        originalCircle.setRadius(3);
        originalCircle.setColor("Verde");
        
        Circle clonedCircle = (Circle) originalCircle.clone();
        clonedCircle.setX(15); // Modificar apenas a posição
        
        System.out.println("=== Clonagem Direta ===");
        originalCircle.draw();
        clonedCircle.draw();
        
        // Usando Registry
        ShapeRegistry registry = new ShapeRegistry();
        registry.loadPredefinedShapes();
        
        System.out.println("\n=== Usando Registry ===");
        Shape shape1 = registry.getShape("círculo-azul");
        Shape shape2 = registry.getShape("retângulo-vermelho");
        Shape shape3 = registry.getShape("círculo-azul"); // Outro clone
        
        if (shape1 != null) shape1.draw();
        if (shape2 != null) shape2.draw();
        if (shape3 != null) shape3.draw();
        
        // Polimorfismo com protótipos
        System.out.println("\n=== Lista Polimórfica ===");
        List<Shape> shapes = Arrays.asList(
            originalCircle.clone(),
            registry.getShape("retângulo-vermelho")
        );
        
        for (Shape shape : shapes) {
            shape.draw();
        }
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Código não deve depender** de classes concretas dos objetos a serem copiados
- **Reduzir subclasses** que diferem apenas na inicialização
- **Trabalhar com objetos de terceiros** através de interfaces
- **Configurações complexas** que são custosas de recriar

### 📝 Exemplos de uso:
- **Editores gráficos**: Clonar formas geométricas
- **Jogos**: Clonar unidades, itens, configurações
- **Configurações de sistema**: Templates pré-definidos
- **Cache de objetos**: Evitar recriação custosa

## 🚀 Como Implementar

1. **Crie interface Prototype** com método `clone()`
2. **Adicione construtor alternativo** que aceita objeto da mesma classe
3. **Implemente método clone** chamando construtor com `new`
4. **Opcionalmente, crie registry** para protótipos frequentes
5. **Substitua chamadas de construtores** por clonagem

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Desacoplamento**: Clonar sem depender de classes concretas
- **Eliminação de código repetitivo**: Evita inicializações complexas
- **Conveniência**: Produzir objetos complexos facilmente
- **Alternativa à herança**: Para presets de configuração

### ❌ Desvantagens:
- **Referências circulares**: Clonar objetos complexos pode ser complicado
- **Implementação complexa**: Para objetos com muitas dependências

## 🔗 Diferenças de Outros Padrões

| Aspecto | Prototype | Factory Method | Abstract Factory |
|---------|-----------|----------------|------------------|
| **Criação** | Por clonagem | Por herança | Por composição |
| **Configuração** | Objeto pré-configurado | Lógica na subclasse | Família de produtos |
| **Herança** | Não obrigatória | Baseado em herança | Interface comum |
| **Flexibilidade** | Alta (runtime) | Média (compile-time) | Média (família) |
| **Inicialização** | Cópia de estado | Desde zero | Desde zero |

## 🔗 Relações com Outros Padrões

- **Factory Method**: Muitos designs começam com Factory Method e evoluem para Prototype
- **Abstract Factory**: Pode usar Prototype para compor métodos
- **Command**: Prototype útil para salvar comandos no histórico
- **Composite/Decorator**: Clonar estruturas complexas ao invés de reconstruir
- **Memento**: Prototype pode ser alternativa mais simples para estados

## 📚 Conceitos-Chave para Lembrar

1. **Auto-clonagem**: Objetos são responsáveis por se clonar
2. **Construtor de cópia**: Padrão comum para implementar clonagem
3. **Shallow vs Deep Copy**: Atenção para referências aninhadas
4. **Registry pattern**: Catálogo de protótipos pré-configurados
5. **Polimorfismo**: Cliente trabalha com interface, não classes concretas

## 🔍 Analogia do Mundo Real

**Divisão celular**: Uma célula se divide criando uma cópia idêntica de si mesma. A célula original atua como protótipo e participa ativamente na criação da cópia, similar ao padrão Prototype onde o objeto original é responsável por sua própria clonagem.

---

> **💡 Dica de Estudo:** Prototype é útil quando criar um objeto do zero é mais caro que cloná-lo, ou quando você precisa de várias variações de um objeto complexo. Pense em "carimbos" ou "moldes" que você pode usar repetidamente.

> **📖 Referência:** [Refactoring Guru - Prototype](https://refactoring.guru/design-patterns/prototype)
