# Builder

## 🎯 Intenção

O Builder é um padrão de projeto criacional que permite construir objetos complexos passo a passo. O padrão permite produzir diferentes tipos e representações de um objeto usando o mesmo código de construção.

## 🚩 Problema

Imagine um objeto complexo que requer inicialização laboriosa, passo a passo, de muitos campos e objetos aninhados. Esse código de inicialização fica geralmente enterrado dentro de um construtor monstruoso com muitos parâmetros ou espalhado pelo código cliente.

### Problemas comuns:

1. **Construtor telescópico**: Muitos parâmetros opcionais levam a múltiplos construtores
2. **Subclasses infinitas**: Uma subclasse para cada configuração possível
3. **Parâmetros desnecessários**: Nem todos os parâmetros são sempre necessários

**Exemplo problemático:**
```java
// Construtor com muitos parâmetros
House house = new House(4, true, false, true, false, "Red", true, false);
// Difícil de entender o que cada boolean/valor representa
```

## ✅ Solução

O padrão Builder sugere extrair o código de construção do objeto para objetos separados chamados **builders**. O padrão organiza a construção em etapas (`buildWalls`, `buildDoor`, etc.).

**Principais conceitos:**
- **Construção passo a passo**: Execute apenas as etapas necessárias
- **Diferentes representações**: Builders distintos para produtos diferentes
- **Director (opcional)**: Define a ordem das etapas de construção
- **Interface comum**: Builders seguem a mesma interface

## 🏗️ Estrutura

```
Builder (Interface)
├── reset()
├── buildStepA()
├── buildStepB()
│
ConcreteBuilder1 implements Builder
├── reset()
├── buildStepA()
├── buildStepB()
├── getProduct(): Product1
│
ConcreteBuilder2 implements Builder
├── reset()
├── buildStepA()
├── buildStepB()
├── getProduct(): Product2

Director
├── builder: Builder
├── constructProduct()
```

### Componentes:
- **Builder**: Interface que declara etapas de construção comuns
- **ConcreteBuilder**: Implementações específicas das etapas de construção
- **Product**: Objeto resultante (pode ter interfaces diferentes)
- **Director**: Define ordem das etapas (opcional)
- **Client**: Associa builder com director e obtém resultado

## 💻 Exemplo Prático

```java
// Produto complexo
class Car {
    private String engine;
    private int seats;
    private boolean gps;
    private boolean tripComputer;
    
    // Construtores, getters e setters...
    
    @Override
    public String toString() {
        return String.format("Car: %s engine, %d seats, GPS: %s, Trip Computer: %s", 
            engine, seats, gps, tripComputer);
    }
}

// Builder interface
interface CarBuilder {
    void reset();
    void setEngine(String engine);
    void setSeats(int seats);
    void setGPS(boolean hasGPS);
    void setTripComputer(boolean hasTripComputer);
    Car getResult();
}

// Builder concreto
class SportsCarBuilder implements CarBuilder {
    private Car car;
    
    public SportsCarBuilder() {
        this.reset();
    }
    
    @Override
    public void reset() {
        this.car = new Car();
    }
    
    @Override
    public void setEngine(String engine) {
        car.setEngine(engine);
    }
    
    @Override
    public void setSeats(int seats) {
        car.setSeats(seats);
    }
    
    @Override
    public void setGPS(boolean hasGPS) {
        car.setGps(hasGPS);
    }
    
    @Override
    public void setTripComputer(boolean hasTripComputer) {
        car.setTripComputer(hasTripComputer);
    }
    
    @Override
    public Car getResult() {
        Car result = this.car;
        this.reset();
        return result;
    }
}

// Builder para manual (produto diferente)
class CarManualBuilder implements CarBuilder {
    private StringBuilder manual;
    
    public CarManualBuilder() {
        this.reset();
    }
    
    @Override
    public void reset() {
        this.manual = new StringBuilder();
    }
    
    @Override
    public void setEngine(String engine) {
        manual.append("Engine: ").append(engine).append("\n");
    }
    
    @Override
    public void setSeats(int seats) {
        manual.append("Seats: ").append(seats).append("\n");
    }
    
    @Override
    public void setGPS(boolean hasGPS) {
        if (hasGPS) {
            manual.append("GPS Navigation System included\n");
        }
    }
    
    @Override
    public void setTripComputer(boolean hasTripComputer) {
        if (hasTripComputer) {
            manual.append("Trip Computer included\n");
        }
    }
    
    public String getManual() {
        String result = manual.toString();
        this.reset();
        return result;
    }
    
    @Override
    public Car getResult() {
        return null; // Manual builder não produz Car
    }
}

// Director (opcional)
class CarDirector {
    public void constructSportsCar(CarBuilder builder) {
        builder.reset();
        builder.setEngine("V8 Sport Engine");
        builder.setSeats(2);
        builder.setGPS(true);
        builder.setTripComputer(true);
    }
    
    public void constructSUV(CarBuilder builder) {
        builder.reset();
        builder.setEngine("V6 Engine");
        builder.setSeats(7);
        builder.setGPS(true);
        builder.setTripComputer(false);
    }
}

// Uso
public class BuilderExample {
    public static void main(String[] args) {
        CarDirector director = new CarDirector();
        
        // Construindo um carro esportivo
        SportsCarBuilder carBuilder = new SportsCarBuilder();
        director.constructSportsCar(carBuilder);
        Car sportsCar = carBuilder.getResult();
        System.out.println(sportsCar);
        
        // Construindo manual para o mesmo carro
        CarManualBuilder manualBuilder = new CarManualBuilder();
        director.constructSportsCar(manualBuilder);
        String manual = manualBuilder.getManual();
        System.out.println("Manual:\n" + manual);
        
        // Construção manual (sem director)
        SportsCarBuilder customBuilder = new SportsCarBuilder();
        customBuilder.setEngine("Electric Motor");
        customBuilder.setSeats(4);
        customBuilder.setGPS(false);
        Car customCar = customBuilder.getResult();
        System.out.println(customCar);
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Eliminar "construtores telescópicos"** com muitos parâmetros opcionais
- **Criar diferentes representações** de um produto usando o mesmo processo
- **Construir objetos Composite** ou estruturas complexas passo a passo

### 📝 Exemplos de uso:
- **SQL Query Builder**: Construir consultas complexas passo a passo
- **HTML/XML Builder**: Criar documentos com estrutura aninhada
- **Configuration Objects**: Objetos com muitas opções configuráveis

## 🚀 Como Implementar

1. **Defina etapas de construção** comuns para todas as representações
2. **Declare etapas na interface** builder base
3. **Crie builders concretos** para cada representação
4. **Implemente método de obtenção** do resultado (pode diferir entre builders)
5. **Considere criar Director** para encapsular rotinas de construção
6. **Cliente cria builder e director**, associa-os e obtém resultado

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Construção passo a passo**: Controle total sobre o processo
- **Reutilização de código**: Mesmo código para diferentes representações
- **Princípio da Responsabilidade Única**: Separação da lógica de construção
- **Flexibilidade**: Executar etapas recursivamente ou postergar execução

### ❌ Desvantagens:
- **Complexidade**: Múltiplas classes novas
- **Overhead**: Pode ser excessivo para objetos simples

## 🔗 Diferenças de Outros Padrões

| Aspecto | Builder | Factory Method | Abstract Factory |
|---------|---------|----------------|------------------|
| **Foco** | Construção passo a passo | Criação via herança | Famílias de produtos |
| **Controle** | Controle total das etapas | Subclasses definem produto | Produtos compatíveis |
| **Flexibilidade** | Alta (etapas opcionais) | Média | Média |
| **Complexidade** | Alta | Baixa | Média |
| **Resultado** | Pode diferir entre builders | Interface comum | Interface comum |

## 🔗 Relações com Outros Padrões

- **Abstract Factory vs Builder**: Abstract Factory retorna produto imediatamente, Builder permite etapas adicionais
- **Factory Method**: Muitos designs evoluem de Factory Method para Builder
- **Composite**: Builder útil para construir árvores Composite
- **Bridge**: Director como abstração, builders como implementações

## 📚 Conceitos-Chave para Lembrar

1. **Construção vs Criação**: Builder constrói passo a passo, Factory cria de uma vez
2. **Flexibilidade**: Nem todas as etapas precisam ser executadas
3. **Produtos diferentes**: Builders podem produzir objetos sem interface comum
4. **Director opcional**: Encapsula sequências de construção comuns
5. **Fluent Interface**: Padrão comum onde métodos retornam `this` para encadeamento

---

> **💡 Dica de Estudo:** Builder é ideal quando você tem um objeto com muitos parâmetros opcionais ou quando precisa do mesmo processo de construção para produtos diferentes. Pense em "receitas de construção".

> **📖 Referência:** [Refactoring Guru - Builder](https://refactoring.guru/design-patterns/builder)
