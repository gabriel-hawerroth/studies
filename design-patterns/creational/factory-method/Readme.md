# Factory Method

> **Também conhecido como:** Virtual Constructor

## 🎯 Intenção

O Factory Method é um padrão de projeto criacional que fornece uma interface para criar objetos em uma superclasse, mas permite que subclasses alterem o tipo de objetos que serão criados. Ele ajuda a desacoplar o código cliente da implementação concreta dos produtos.

## 🚩 Problema

Imagine que você está criando um sistema de logística. A primeira versão só trabalha com caminhões (`Truck`), então a maior parte do código está na classe `Truck`.

Depois de um tempo, sua aplicação fica popular e você recebe pedidos para incorporar transporte marítimo (`Ship`). 

**O problema:** Adicionar uma nova classe não é simples quando o resto do código já está acoplado às classes existentes. Você acabaria com código repleto de condicionais que alteram o comportamento da aplicação dependendo da classe de transporte.

## ✅ Solução

O Factory Method sugere que você substitua chamadas diretas de construção de objetos (usando `new`) por chamadas para um método-fábrica especial. Os objetos ainda são criados via `new`, mas isso é feito dentro do método-fábrica.

**Principais conceitos:**
- Subclasses podem sobrescrever o método-fábrica para alterar a classe dos produtos criados
- Todos os produtos devem seguir uma interface comum
- O código cliente trabalha apenas com a interface abstrata

## 🏗️ Estrutura

```
Creator (Criador)
├── factoryMethod(): Product (abstrato)
├── someOperation(): void
│
ConcreteCreator1 extends Creator
├── factoryMethod(): ConcreteProduct1
│
ConcreteCreator2 extends Creator
├── factoryMethod(): ConcreteProduct2

Product (Interface)
├── operation(): void
│
ConcreteProduct1 implements Product
├── operation(): void
│
ConcreteProduct2 implements Product
├── operation(): void
```

### Componentes:
- **Product**: Interface comum para todos os produtos
- **ConcreteProduct**: Implementações específicas da interface
- **Creator**: Declara o método-fábrica que retorna objetos Product
- **ConcreteCreator**: Sobrescreve o método-fábrica para retornar produtos específicos

## 💻 Exemplo Prático

```java
// Interface do produto
interface Transport {
    String deliver();
}

// Produtos concretos
class Truck implements Transport {
    @Override
    public String deliver() {
        return "Entrega por terra em caixas";
    }
}

class Ship implements Transport {
    @Override
    public String deliver() {
        return "Entrega por mar em contêineres";
    }
}

// Criador abstrato
abstract class Logistics {
    // Método-fábrica que deve ser implementado pelas subclasses
    public abstract Transport createTransport();
    
    // Lógica de negócio que usa o produto
    public String planDelivery() {
        Transport transport = createTransport();
        String result = transport.deliver();
        return "Planejamento: " + result;
    }
}

// Criadores concretos
class RoadLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Ship();
    }
}

// Uso
public class FactoryMethodExample {
    public static void clientCode(Logistics creator) {
        System.out.println(creator.planDelivery());
    }
    
    public static void main(String[] args) {
        // Teste
        Logistics roadLogistics = new RoadLogistics();
        Logistics seaLogistics = new SeaLogistics();
        
        clientCode(roadLogistics);  // Planejamento: Entrega por terra em caixas
        clientCode(seaLogistics);   // Planejamento: Entrega por mar em contêineres
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Não souber previamente** os tipos exatos e dependências dos objetos que seu código deve trabalhar
- **Quiser fornecer aos usuários** da sua biblioteca uma forma de estender componentes internos
- **Quiser economizar recursos** reutilizando objetos existentes ao invés de reconstruí-los

### 📝 Exemplo de caso de uso:
Você tem uma UI framework que fornece botões quadrados, mas quer botões redondos. Você pode:
1. Estender `Button` criando `RoundButton`
2. Criar `UIWithRoundButtons` que sobrescreve `createButton()`
3. Usar sua nova classe no lugar da original

## 🚀 Como Implementar

1. **Padronize os produtos** com uma interface comum
2. **Adicione método-fábrica vazio** na classe criadora
3. **Substitua construtores** por chamadas ao método-fábrica
4. **Crie subclasses** para cada tipo de produto
5. **Sobrescreva o método-fábrica** em cada subclasse
6. **Torne o método abstrato** se a classe base ficar vazia

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Desacoplamento** entre criador e produtos concretos
- **Princípio da Responsabilidade Única**: código de criação em um lugar
- **Princípio Aberto/Fechado**: novos produtos sem quebrar código existente

### ❌ Desvantagens:
- **Complexidade**: pode exigir muitas subclasses novas
- **Melhor cenário**: introduzir em hierarquia de criadores já existente

## 🔗 Relações com Outros Padrões

- **Abstract Factory**: Frequentemente implementado com Factory Methods
- **Iterator**: Factory Method pode retornar iteradores compatíveis com coleções
- **Prototype**: Não usa herança, mas requer inicialização complexa (vs Factory Method)
- **Template Method**: Factory Method pode ser um passo em um Template Method maior

## 📚 Conceitos-Chave para Lembrar

1. **Desacoplamento**: Cliente não conhece classes concretas
2. **Interface comum**: Todos os produtos implementam a mesma interface
3. **Flexibilidade**: Fácil adicionar novos produtos
4. **Herança**: Subclasses definem quais produtos criar

---

> **💡 Dica de Estudo:** O Factory Method é uma evolução natural quando você percebe que tem muitos `if/else` ou `switch` para decidir qual objeto criar. É o primeiro passo para padrões mais complexos como Abstract Factory.

> **📖 Referência:** [Refactoring Guru - Factory Method](https://refactoring.guru/design-patterns/factory-method)
