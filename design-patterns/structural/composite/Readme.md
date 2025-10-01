# Composite

> **Também conhecido como:** Object Tree

## 🎯 Intenção

O Composite é um padrão de projeto estrutural que permite compor objetos em estruturas de árvore e trabalhar com essas estruturas como se fossem objetos individuais. Ele trata objetos simples e composições de objetos de forma uniforme.

## 🚩 Problema

Imagine que você tem dois tipos de objetos: `Products` e `Boxes`. Uma `Box` pode conter vários `Products` e também outras `Boxes` menores. Essas caixas menores também podem conter produtos ou caixas ainda menores, e assim por diante.

### Desafio do Sistema de Pedidos:
Você decide criar um sistema de pedidos usando essas classes. Os pedidos podem conter:
- Produtos simples sem embalagem
- Caixas com produtos 
- Caixas com outras caixas (aninhamento infinito)

**Como determinar o preço total de um pedido assim?**

```java
// Abordagem direta problemática
public double calculateTotalPrice(Object item) {
    if (item instanceof Product) {
        return ((Product) item).getPrice();
    } else if (item instanceof Box) {
        double total = 0;
        for (Object content : ((Box) item).getContents()) {
            total += calculateTotalPrice(content); // Recursão manual
        }
        return total + ((Box) item).getPackagingCost();
    }
    // Precisa conhecer todas as classes específicas!
}
```

**Problemas:**
- Precisa conhecer todas as classes específicas
- Código fica complicado com verificações de tipo
- Difícil adicionar novos tipos de objetos

## ✅ Solução

O Composite sugere trabalhar com `Products` e `Boxes` através de uma **interface comum** que declara um método para calcular o preço total.

### Como funciona:
- **Para um produto**: simplesmente retorna seu preço
- **Para uma caixa**: percorre cada item que contém, pede o preço e retorna o total
- **Recursão automática**: Se um item for uma caixa menor, ela também percorre seus conteúdos

```java
// Interface comum
interface PriceCalculable {
    double getPrice();
}

// Produto simples
class Product implements PriceCalculable {
    public double getPrice() { return price; }
}

// Caixa composta
class Box implements PriceCalculable {
    public double getPrice() {
        return items.stream()
                   .mapToDouble(PriceCalculable::getPrice)
                   .sum() + packagingCost;
    }
}
```

## 🏗️ Estrutura

```
Component (interface)
├── operation()
│
├── Leaf implements Component
│   └── operation() // faz o trabalho real
│
└── Composite implements Component
    ├── children: List<Component>
    ├── add(Component)
    ├── remove(Component)
    └── operation() // delega para filhos
```

### Componentes:
- **Component**: Interface comum para objetos simples e complexos
- **Leaf**: Elemento básico sem sub-elementos (faz o trabalho real)
- **Composite**: Elemento com sub-elementos, delega trabalho para filhos
- **Client**: Trabalha com elementos através da interface Component

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Arquivos

```java
// Component - interface comum
interface FileSystemElement {
    String getName();
    long getSize();
    void display(int depth);
}

// Leaf - arquivo simples
class File implements FileSystemElement {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public long getSize() {
        return size;
    }
    
    @Override
    public void display(int depth) {
        String indent = "  ".repeat(depth);
        System.out.printf("%s📄 %s (%d bytes)%n", indent, name, size);
    }
}

// Composite - diretório
class Directory implements FileSystemElement {
    private String name;
    private List<FileSystemElement> elements;
    
    public Directory(String name) {
        this.name = name;
        this.elements = new ArrayList<>();
    }
    
    public void add(FileSystemElement element) {
        elements.add(element);
    }
    
    public void remove(FileSystemElement element) {
        elements.remove(element);
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public long getSize() {
        return elements.stream()
                      .mapToLong(FileSystemElement::getSize)
                      .sum();
    }
    
    @Override
    public void display(int depth) {
        String indent = "  ".repeat(depth);
        System.out.printf("%s📁 %s/ (%d bytes total)%n", indent, name, getSize());
        
        for (FileSystemElement element : elements) {
            element.display(depth + 1);
        }
    }
}

// Uso
public class FileSystemExample {
    public static void main(String[] args) {
        // Criando estrutura de arquivos
        Directory root = new Directory("root");
        Directory documents = new Directory("Documents");
        Directory photos = new Directory("Photos");
        
        File resume = new File("resume.pdf", 1024);
        File photo1 = new File("vacation.jpg", 2048);
        File photo2 = new File("family.jpg", 1536);
        File readme = new File("README.txt", 512);
        
        // Montando árvore
        documents.add(resume);
        documents.add(readme);
        
        photos.add(photo1);
        photos.add(photo2);
        
        root.add(documents);
        root.add(photos);
        
        // Trabalhando uniformemente com toda a estrutura
        System.out.println("Estrutura do sistema de arquivos:");
        root.display(0);
        
        System.out.println("\nTamanho total: " + root.getSize() + " bytes");
    }
}
```

### Exemplo 2: Organização Empresarial

```java
// Component - funcionário genérico
abstract class Employee {
    protected String name;
    protected String position;
    protected double salary;
    
    public Employee(String name, String position, double salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    
    public abstract double getTotalSalary();
    public abstract void showHierarchy(int level);
    public abstract int getTeamSize();
}

// Leaf - funcionário individual
class Developer extends Employee {
    public Developer(String name, double salary) {
        super(name, "Developer", salary);
    }
    
    @Override
    public double getTotalSalary() {
        return salary;
    }
    
    @Override
    public void showHierarchy(int level) {
        String indent = "  ".repeat(level);
        System.out.printf("%s👨‍💻 %s - %s (R$ %.2f)%n", 
                         indent, name, position, salary);
    }
    
    @Override
    public int getTeamSize() {
        return 1;
    }
}

// Composite - gerente com equipe
class Manager extends Employee {
    private List<Employee> team;
    
    public Manager(String name, double salary) {
        super(name, "Manager", salary);
        this.team = new ArrayList<>();
    }
    
    public void addTeamMember(Employee employee) {
        team.add(employee);
    }
    
    public void removeTeamMember(Employee employee) {
        team.remove(employee);
    }
    
    @Override
    public double getTotalSalary() {
        double total = salary; // Salário do gerente
        for (Employee employee : team) {
            total += employee.getTotalSalary();
        }
        return total;
    }
    
    @Override
    public void showHierarchy(int level) {
        String indent = "  ".repeat(level);
        System.out.printf("%s👔 %s - %s (R$ %.2f) - Equipe: %d pessoas%n", 
                         indent, name, position, salary, getTeamSize());
        
        for (Employee employee : team) {
            employee.showHierarchy(level + 1);
        }
    }
    
    @Override
    public int getTeamSize() {
        int size = 1; // O próprio gerente
        for (Employee employee : team) {
            size += employee.getTeamSize();
        }
        return size;
    }
}

// Uso
public class OrganizationExample {
    public static void main(String[] args) {
        // Criando a organização
        Manager ceo = new Manager("Alice Silva", 15000);
        
        Manager devManager = new Manager("Bob Santos", 8000);
        Manager qaManager = new Manager("Carol Lima", 7000);
        
        Developer dev1 = new Developer("David Costa", 5000);
        Developer dev2 = new Developer("Eva Rocha", 5500);
        Developer dev3 = new Developer("Frank Alves", 4800);
        
        Developer qa1 = new Developer("Grace Pereira", 4500);
        Developer qa2 = new Developer("Hugo Ferreira", 4200);
        
        // Montando a hierarquia
        devManager.addTeamMember(dev1);
        devManager.addTeamMember(dev2);
        devManager.addTeamMember(dev3);
        
        qaManager.addTeamMember(qa1);
        qaManager.addTeamMember(qa2);
        
        ceo.addTeamMember(devManager);
        ceo.addTeamMember(qaManager);
        
        // Trabalhando com a organização como um todo
        System.out.println("Estrutura Organizacional:");
        ceo.showHierarchy(0);
        
        System.out.println("\nResumo:");
        System.out.println("Total de funcionários: " + ceo.getTeamSize());
        System.out.printf("Folha salarial total: R$ %.2f%n", ceo.getTotalSalary());
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Estrutura em árvore**: Modelo da aplicação pode ser representado como árvore
- **Tratamento uniforme**: Quer tratar objetos simples e complexos da mesma forma
- **Composição recursiva**: Objetos podem conter outros objetos do mesmo tipo
- **Hierarquias aninhadas**: Estruturas com múltiplos níveis de profundidade

### 📝 Exemplos de uso:
- **Sistemas de arquivos**: Diretórios e arquivos
- **Interfaces gráficas**: Painéis, botões, grupos
- **Estruturas organizacionais**: Departamentos, equipes, funcionários
- **Expressões matemáticas**: Operadores e operandos
- **Documentos**: Seções, parágrafos, textos

### ❌ Evite quando:
- **Estrutura muito simples**: Adiciona complexidade desnecessária
- **Funcionalidade muito diferente**: Interface comum fica muito genérica
- **Performance crítica**: Recursão pode ser custosa para árvores profundas

## 🚀 Como Implementar

1. **Verifique se o modelo** pode ser representado como estrutura de árvore

2. **Declare interface Component** com métodos comuns para elementos simples e complexos

3. **Crie classe Leaf** para elementos simples (pode ter múltiplas classes Leaf)

4. **Crie classe Composite** para elementos complexos:
   - Array de sub-elementos do tipo Component
   - Métodos delegam trabalho para sub-elementos

5. **Defina métodos de gerenciamento** (add/remove) no Composite

6. **Considere colocar métodos de gerenciamento** na interface Component (princípio vs praticidade)

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Trabalha com estruturas complexas**: Polimorfismo e recursão
- **Princípio Aberto/Fechado**: Novos tipos sem quebrar código existente
- **Simplicidade para cliente**: Trata tudo uniformemente
- **Flexibilidade**: Facilmente reorganiza estruturas

### ❌ Desvantagens:
- **Interface muito genérica**: Pode ser difícil criar interface comum
- **Overgeneralização**: Interface pode ficar difícil de entender
- **Performance**: Recursão em árvores muito profundas

## 🔗 Diferenças de Outros Padrões

| Padrão | Estrutura | Propósito | Características |
|--------|-----------|-----------|----------------|
| **Composite** | Árvore recursiva | Tratar grupos como objetos | Soma resultados dos filhos |
| **Decorator** | Lista linear | Adicionar responsabilidades | Estende comportamento |
| **Chain of Responsibility** | Lista linear | Processar requisições | Passa requisição adiante |
| **Facade** | Objeto único | Simplificar interface | Esconde complexidade |

## 🔗 Relações com Outros Padrões

- **Builder**: Construção recursiva de árvores Composite complexas
- **Chain of Responsibility**: Leaf pode passar requisição pela cadeia de pais
- **Iterator**: Percorrer árvores Composite
- **Visitor**: Executar operações na árvore inteira
- **Flyweight**: Folhas compartilhadas para economizar memória
- **Decorator**: Estrutura similar, mas Decorator adiciona responsabilidades

## 📚 Conceitos-Chave para Lembrar

1. **Estrutura em árvore**: Essencial para aplicar o padrão
2. **Interface comum**: Leaf e Composite implementam a mesma interface
3. **Recursão transparente**: Cliente não precisa saber se é simples ou composto
4. **Delegação**: Composite delega trabalho para filhos
5. **Soma de resultados**: Composite "agrega" resultados dos filhos
6. **Uniformidade**: Tratar objetos individuais e composições igualmente

## 🔍 Analogia do Mundo Real

**Estrutura militar**: Um exército consiste em divisões; uma divisão é um conjunto de brigadas; uma brigada consiste em pelotões; e pelotões podem ser divididos em esquadrões. Finalmente, um esquadrão é um pequeno grupo de soldados reais. Ordens são dadas no topo da hierarquia e passadas para cada nível até cada soldado saber o que precisa fazer.

## ⚠️ Considerações Importantes

### Interface Design:
- **Segurança vs Transparência**: Métodos add/remove em Component?
- **Princípio da Segregação de Interface**: Leaves não precisam de add/remove
- **Transparência**: Cliente trata tudo igual, mas métodos podem ser vazios

### Performance:
- **Caching**: Armazenar resultados calculados
- **Lazy evaluation**: Calcular apenas quando necessário
- **Profundidade da árvore**: Pode impactar performance da recursão

### Implementação:
- **Parent references**: Facilita navegação para cima
- **Iterator pattern**: Para percorrer a árvore
- **Visitor pattern**: Para operações complexas na árvore

---

> **💡 Dica de Estudo:** Composite é como uma "boneca russa" - cada peça pode conter outras peças menores, mas você trata todas da mesma forma. Use quando tiver estruturas aninhadas que precisam ser processadas uniformemente.

> **📖 Referência:** [Refactoring Guru - Composite](https://refactoring.guru/design-patterns/composite)
