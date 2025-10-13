# Visitor

## 🎯 Intenção

O Visitor é um padrão de projeto comportamental que permite **separar algoritmos dos objetos** nos quais eles operam. Ele permite adicionar novas operações a uma estrutura de objetos existente sem modificar essas classes, usando a técnica de **Double Dispatch**.

## 🚩 Problema

Imagine que sua equipe desenvolve um app que trabalha com informações geográficas estruturadas como um grafo colossal. Cada nó pode representar uma entidade complexa como cidade, indústria, área turística, etc. Cada tipo de nó é representado por sua própria classe.

Você recebe a tarefa de implementar exportação do grafo para formato XML. A primeira ideia: adicionar método `export()` em cada classe de nó. Simples e elegante! Mas...

### Resultado problemático:
```java
// Tentativa 1: Adicionar método export em cada classe
class City {
    private String name;
    private int population;
    
    // Comportamento principal
    public void updatePopulation() { /* ... */ }
    public void addIndustry() { /* ... */ }
    
    // ❌ NOVO: Export XML
    public String exportToXML() {
        return "<city><name>" + name + "</name></city>";
    }
}

class Industry {
    private String type;
    private int employees;
    
    // Comportamento principal
    public void hire() { /* ... */ }
    public void produce() { /* ... */ }
    
    // ❌ NOVO: Export XML
    public String exportToXML() {
        return "<industry><type>" + type + "</type></industry>";
    }
}

class SightSeeing {
    private String attraction;
    private double rating;
    
    // Comportamento principal
    public void updateRating() { /* ... */ }
    
    // ❌ NOVO: Export XML
    public String exportToXML() {
        return "<sight><name>" + attraction + "</name></sight>";
    }
}
```

**Problemas:**
- **Arquiteto recusa**: Código em produção, risco de quebrar
- **Responsabilidade errada**: Export XML não é trabalho de City/Industry
- **Violação SRP**: Classes fazem trabalho com geodata + export
- **Mudanças futuras**: Marketing vai pedir JSON, CSV, PDF... 😱
- **Classes frágeis**: Cada novo formato = modificar todas classes
- **Código alien**: Export XML parece estranho em classes de geodata

### Tentativa 2: Condicionais no cliente

```java
// Cliente precisa saber tipo de cada nó
class GraphExporter {
    public void exportGraph(List<Node> nodes) {
        for (Node node : nodes) {
            // ❌ Type checking manual!
            if (node instanceof City) {
                exportCity((City) node);
            } else if (node instanceof Industry) {
                exportIndustry((Industry) node);
            } else if (node instanceof SightSeeing) {
                exportSightSeeing((SightSeeing) node);
            }
            // Adicionar novo tipo = modificar este código!
        }
    }
    
    private void exportCity(City c) { /* ... */ }
    private void exportIndustry(Industry i) { /* ... */ }
    private void exportSightSeeing(SightSeeing s) { /* ... */ }
}
```

**Mais problemas:**
- **Type checking**: Precisa checar tipo de cada objeto
- **Não escalável**: Adicionar tipo = modificar cliente
- **Sem polimorfismo**: Perdeu benefício de OOP
- **Conditional nightmare**: if/else gigante

## ✅ Solução

O padrão Visitor sugere que você coloque o novo comportamento em uma classe separada chamada **visitor**, ao invés de tentar integrá-lo nas classes existentes. O objeto original é passado para um método do visitor como argumento.

A mágica acontece com **Double Dispatch**: ao invés do cliente escolher qual método chamar, os objetos delegam essa escolha. Eles "aceitam" o visitor e dizem qual método visitar deve ser executado.

### Características-chave:
- **Separação de algoritmos**: Comportamentos em classes visitor separadas
- **Double Dispatch**: Duas chamadas polimórficas para resolver tipo
- **Método accept()**: Elementos aceitam visitor
- **Operações sem modificar**: Adiciona operações sem mudar elementos
- **Open/Closed**: Aberto para extensão (novos visitors), fechado para modificação

```java
// Interface Visitor
interface Visitor {
    void visitCity(City city);
    void visitIndustry(Industry industry);
    void visitSightSeeing(SightSeeing sight);
}

// Interface Element
interface Element {
    void accept(Visitor visitor);
}

// Concrete Elements
class City implements Element {
    private String name;
    private int population;
    
    public City(String name, int population) {
        this.name = name;
        this.population = population;
    }
    
    // Comportamento principal da classe
    public void updatePopulation() { /* ... */ }
    
    // DOUBLE DISPATCH: City conhece sua própria classe
    @Override
    public void accept(Visitor visitor) {
        visitor.visitCity(this); // Chama método específico para City
    }
    
    public String getName() { return name; }
    public int getPopulation() { return population; }
}

class Industry implements Element {
    private String type;
    private int employees;
    
    public Industry(String type, int employees) {
        this.type = type;
        this.employees = employees;
    }
    
    @Override
    public void accept(Visitor visitor) {
        visitor.visitIndustry(this); // Chama método específico para Industry
    }
    
    public String getType() { return type; }
    public int getEmployees() { return employees; }
}

// Concrete Visitor - XML Export
class XMLExportVisitor implements Visitor {
    private StringBuilder xml = new StringBuilder();
    
    @Override
    public void visitCity(City city) {
        xml.append("<city>")
           .append("<name>").append(city.getName()).append("</name>")
           .append("<population>").append(city.getPopulation()).append("</population>")
           .append("</city>\n");
    }
    
    @Override
    public void visitIndustry(Industry industry) {
        xml.append("<industry>")
           .append("<type>").append(industry.getType()).append("</type>")
           .append("<employees>").append(industry.getEmployees()).append("</employees>")
           .append("</industry>\n");
    }
    
    @Override
    public void visitSightSeeing(SightSeeing sight) {
        xml.append("<sight>")
           .append("<name>").append(sight.getName()).append("</name>")
           .append("</sight>\n");
    }
    
    public String getXML() {
        return xml.toString();
    }
}

// Uso - SEM type checking!
List<Element> elements = Arrays.asList(
    new City("São Paulo", 12_000_000),
    new Industry("Tech", 5000),
    new SightSeeing("Museu")
);

XMLExportVisitor xmlVisitor = new XMLExportVisitor();
for (Element element : elements) {
    element.accept(xmlVisitor); // Polimorfismo!
}
System.out.println(xmlVisitor.getXML());

// Adicionar novo formato é fácil!
JSONExportVisitor jsonVisitor = new JSONExportVisitor();
for (Element element : elements) {
    element.accept(jsonVisitor); // Mesmo código!
}
```

## 🏗️ Estrutura

```
Visitor (interface)                Element (interface)
    ↓                                      ↓
visitConcreteA(ConcreteA)             accept(Visitor)
visitConcreteB(ConcreteB)                  ↑
    ↑                                      |
ConcreteVisitor1  ConcreteVisitor2    ConcreteElementA  ConcreteElementB
       ↓                 ↓                    ↓                ↓
  visitConcreteA()  visitConcreteA()    accept(v) {      accept(v) {
  visitConcreteB()  visitConcreteB()      v.visitA(this)   v.visitB(this)
                                        }                 }

Client
   ↓
foreach element in structure:
   element.accept(visitor)  ← DOUBLE DISPATCH!
```

### Componentes:
- **Visitor**: Interface com método de visita para cada ConcreteElement
- **ConcreteVisitor**: Implementa operação específica para cada elemento
- **Element**: Interface com método `accept(Visitor)`
- **ConcreteElement**: Implementa `accept()` chamando método visitor apropriado
- **Client**: Itera elementos e chama `accept()`

### Double Dispatch explicado:
1. **Primeira dispatch**: `element.accept(visitor)` - polimorfismo escolhe implementação de `accept()` baseado no tipo de element
2. **Segunda dispatch**: `visitor.visitCity(this)` - polimorfismo escolhe método correto no visitor baseado no tipo do parâmetro

## 💻 Exemplos Práticos

### Exemplo 1: Sistema de Impostos para Diferentes Tipos de Produtos

```java
// Interface Element
interface Product {
    void accept(ProductVisitor visitor);
}

// Concrete Elements
class Food implements Product {
    private String name;
    private double price;
    private boolean organic;
    
    public Food(String name, double price, boolean organic) {
        this.name = name;
        this.price = price;
        this.organic = organic;
    }
    
    @Override
    public void accept(ProductVisitor visitor) {
        visitor.visitFood(this);
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    public boolean isOrganic() { return organic; }
}

class Electronics implements Product {
    private String name;
    private double price;
    private int warrantyYears;
    
    public Electronics(String name, double price, int warrantyYears) {
        this.name = name;
        this.price = price;
        this.warrantyYears = warrantyYears;
    }
    
    @Override
    public void accept(ProductVisitor visitor) {
        visitor.visitElectronics(this);
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getWarrantyYears() { return warrantyYears; }
}

class Clothing implements Product {
    private String name;
    private double price;
    private String size;
    private boolean imported;
    
    public Clothing(String name, double price, String size, boolean imported) {
        this.name = name;
        this.price = price;
        this.size = size;
        this.imported = imported;
    }
    
    @Override
    public void accept(ProductVisitor visitor) {
        visitor.visitClothing(this);
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    public String getSize() { return size; }
    public boolean isImported() { return imported; }
}

class Beverage implements Product {
    private String name;
    private double price;
    private boolean alcoholic;
    
    public Beverage(String name, double price, boolean alcoholic) {
        this.name = name;
        this.price = price;
        this.alcoholic = alcoholic;
    }
    
    @Override
    public void accept(ProductVisitor visitor) {
        visitor.visitBeverage(this);
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    public boolean isAlcoholic() { return alcoholic; }
}

// Interface Visitor
interface ProductVisitor {
    void visitFood(Food food);
    void visitElectronics(Electronics electronics);
    void visitClothing(Clothing clothing);
    void visitBeverage(Beverage beverage);
}

// Concrete Visitor 1 - Calculadora de Impostos
class TaxCalculatorVisitor implements ProductVisitor {
    private double totalTax = 0;
    
    @Override
    public void visitFood(Food food) {
        // Alimentos: 5% (ou isento se orgânico)
        double tax = food.isOrganic() ? 0 : food.getPrice() * 0.05;
        totalTax += tax;
        
        System.out.printf("🍎 %s: $%.2f (Imposto: $%.2f - %.0f%%)%s\n",
            food.getName(),
            food.getPrice(),
            tax,
            food.isOrganic() ? 0 : 5,
            food.isOrganic() ? " [ORGÂNICO - ISENTO]" : ""
        );
    }
    
    @Override
    public void visitElectronics(Electronics electronics) {
        // Eletrônicos: 18%
        double tax = electronics.getPrice() * 0.18;
        totalTax += tax;
        
        System.out.printf("📱 %s: $%.2f (Imposto: $%.2f - 18%%) [Garantia: %d anos]\n",
            electronics.getName(),
            electronics.getPrice(),
            tax,
            electronics.getWarrantyYears()
        );
    }
    
    @Override
    public void visitClothing(Clothing clothing) {
        // Roupas: 12% (ou 25% se importada)
        double taxRate = clothing.isImported() ? 0.25 : 0.12;
        double tax = clothing.getPrice() * taxRate;
        totalTax += tax;
        
        System.out.printf("👕 %s (%s): $%.2f (Imposto: $%.2f - %.0f%%)%s\n",
            clothing.getName(),
            clothing.getSize(),
            clothing.getPrice(),
            tax,
            taxRate * 100,
            clothing.isImported() ? " [IMPORTADA]" : ""
        );
    }
    
    @Override
    public void visitBeverage(Beverage beverage) {
        // Bebidas: 8% (ou 45% se alcoólica)
        double taxRate = beverage.isAlcoholic() ? 0.45 : 0.08;
        double tax = beverage.getPrice() * taxRate;
        totalTax += tax;
        
        System.out.printf("🥤 %s: $%.2f (Imposto: $%.2f - %.0f%%)%s\n",
            beverage.getName(),
            beverage.getPrice(),
            tax,
            taxRate * 100,
            beverage.isAlcoholic() ? " [ALCOÓLICA]" : ""
        );
    }
    
    public double getTotalTax() {
        return totalTax;
    }
    
    public void reset() {
        totalTax = 0;
    }
}

// Concrete Visitor 2 - Gerador de Etiquetas
class LabelGeneratorVisitor implements ProductVisitor {
    
    @Override
    public void visitFood(Food food) {
        System.out.println("┌─────────────────────────────────┐");
        System.out.println("│    ALIMENTO                     │");
        System.out.println("├─────────────────────────────────┤");
        System.out.printf("│ %-31s │\n", food.getName());
        System.out.printf("│ Preço: $%-22.2f │\n", food.getPrice());
        System.out.printf("│ %s │\n", 
            food.isOrganic() ? "✓ ORGÂNICO                     " : "                               ");
        System.out.println("│ Validade: Ver embalagem         │");
        System.out.println("└─────────────────────────────────┘\n");
    }
    
    @Override
    public void visitElectronics(Electronics electronics) {
        System.out.println("┌─────────────────────────────────┐");
        System.out.println("│    ELETRÔNICOS                  │");
        System.out.println("├─────────────────────────────────┤");
        System.out.printf("│ %-31s │\n", electronics.getName());
        System.out.printf("│ Preço: $%-22.2f │\n", electronics.getPrice());
        System.out.printf("│ Garantia: %d anos %-14s │\n", 
            electronics.getWarrantyYears(), "");
        System.out.println("│ ⚠️  Não jogue no lixo comum      │");
        System.out.println("└─────────────────────────────────┘\n");
    }
    
    @Override
    public void visitClothing(Clothing clothing) {
        System.out.println("┌─────────────────────────────────┐");
        System.out.println("│    VESTUÁRIO                    │");
        System.out.println("├─────────────────────────────────┤");
        System.out.printf("│ %-31s │\n", clothing.getName());
        System.out.printf("│ Tamanho: %-22s │\n", clothing.getSize());
        System.out.printf("│ Preço: $%-22.2f │\n", clothing.getPrice());
        System.out.printf("│ %s │\n",
            clothing.isImported() ? "🌍 IMPORTADO                   " : "🇧🇷 FABRICADO NO BRASIL         ");
        System.out.println("│ Instruções: Ver etiqueta        │");
        System.out.println("└─────────────────────────────────┘\n");
    }
    
    @Override
    public void visitBeverage(Beverage beverage) {
        System.out.println("┌─────────────────────────────────┐");
        System.out.println("│    BEBIDA                       │");
        System.out.println("├─────────────────────────────────┤");
        System.out.printf("│ %-31s │\n", beverage.getName());
        System.out.printf("│ Preço: $%-22.2f │\n", beverage.getPrice());
        System.out.printf("│ %s │\n",
            beverage.isAlcoholic() ? "🔞 CONTÉM ÁLCOOL               " : "                               ");
        if (beverage.isAlcoholic()) {
            System.out.println("│ ⚠️  Venda proibida p/ menores   │");
        }
        System.out.println("│ Conservar em local fresco       │");
        System.out.println("└─────────────────────────────────┘\n");
    }
}

// Concrete Visitor 3 - Calculadora de Frete
class ShippingCostVisitor implements ProductVisitor {
    private double totalShipping = 0;
    
    @Override
    public void visitFood(Food food) {
        // Alimentos: $5 fixo
        double shipping = 5.00;
        totalShipping += shipping;
        System.out.printf("📦 %s: Frete $%.2f (padrão)\n", food.getName(), shipping);
    }
    
    @Override
    public void visitElectronics(Electronics electronics) {
        // Eletrônicos: $15 + seguro
        double shipping = 15.00 + (electronics.getPrice() * 0.02);
        totalShipping += shipping;
        System.out.printf("📦 %s: Frete $%.2f ($15 base + $%.2f seguro)\n",
            electronics.getName(), shipping, electronics.getPrice() * 0.02);
    }
    
    @Override
    public void visitClothing(Clothing clothing) {
        // Roupas: $8 (ou $12 se importada)
        double shipping = clothing.isImported() ? 12.00 : 8.00;
        totalShipping += shipping;
        System.out.printf("📦 %s: Frete $%.2f%s\n",
            clothing.getName(), shipping,
            clothing.isImported() ? " (taxa internacional)" : "");
    }
    
    @Override
    public void visitBeverage(Beverage beverage) {
        // Bebidas: $7 (embalagem especial para líquidos)
        double shipping = 7.00;
        totalShipping += shipping;
        System.out.printf("📦 %s: Frete $%.2f (embalagem especial)\n",
            beverage.getName(), shipping);
    }
    
    public double getTotalShipping() {
        return totalShipping;
    }
    
    public void reset() {
        totalShipping = 0;
    }
}

// Concrete Visitor 4 - Relatório Detalhado
class DetailedReportVisitor implements ProductVisitor {
    private int foodCount = 0;
    private int electronicsCount = 0;
    private int clothingCount = 0;
    private int beverageCount = 0;
    private double totalValue = 0;
    
    @Override
    public void visitFood(Food food) {
        foodCount++;
        totalValue += food.getPrice();
    }
    
    @Override
    public void visitElectronics(Electronics electronics) {
        electronicsCount++;
        totalValue += electronics.getPrice();
    }
    
    @Override
    public void visitClothing(Clothing clothing) {
        clothingCount++;
        totalValue += clothing.getPrice();
    }
    
    @Override
    public void visitBeverage(Beverage beverage) {
        beverageCount++;
        totalValue += beverage.getPrice();
    }
    
    public void printReport() {
        System.out.println("\n═══════════════════════════════════════");
        System.out.println("         RELATÓRIO DE PRODUTOS");
        System.out.println("═══════════════════════════════════════");
        System.out.printf("🍎 Alimentos:    %d item(s)\n", foodCount);
        System.out.printf("📱 Eletrônicos:  %d item(s)\n", electronicsCount);
        System.out.printf("👕 Vestuário:    %d item(s)\n", clothingCount);
        System.out.printf("🥤 Bebidas:      %d item(s)\n", beverageCount);
        System.out.println("───────────────────────────────────────");
        System.out.printf("📊 Total:        %d item(s)\n", 
            foodCount + electronicsCount + clothingCount + beverageCount);
        System.out.printf("💰 Valor Total:  $%.2f\n", totalValue);
        System.out.println("═══════════════════════════════════════\n");
    }
}

// Uso
public class TaxSystemExample {
    public static void main(String[] args) {
        System.out.println("═══════════════════════════════════════");
        System.out.println("    SISTEMA DE GESTÃO DE PRODUTOS");
        System.out.println("═══════════════════════════════════════\n");
        
        // Cria carrinho de produtos
        List<Product> cart = Arrays.asList(
            new Food("Maçã Orgânica", 5.99, true),
            new Food("Arroz 5kg", 12.50, false),
            new Electronics("Smartphone", 1200.00, 2),
            new Electronics("Fone Bluetooth", 150.00, 1),
            new Clothing("Camiseta", 49.90, "M", false),
            new Clothing("Jaqueta Importada", 299.00, "L", true),
            new Beverage("Suco Natural", 6.50, false),
            new Beverage("Vinho Tinto", 85.00, true)
        );
        
        // Visitor 1: Cálculo de Impostos
        System.out.println("### CÁLCULO DE IMPOSTOS ###\n");
        TaxCalculatorVisitor taxVisitor = new TaxCalculatorVisitor();
        for (Product product : cart) {
            product.accept(taxVisitor);
        }
        System.out.println("───────────────────────────────────────");
        System.out.printf("💰 TOTAL DE IMPOSTOS: $%.2f\n\n", taxVisitor.getTotalTax());
        
        // Visitor 2: Geração de Etiquetas
        System.out.println("\n### GERAÇÃO DE ETIQUETAS ###\n");
        LabelGeneratorVisitor labelVisitor = new LabelGeneratorVisitor();
        for (Product product : cart) {
            product.accept(labelVisitor);
        }
        
        // Visitor 3: Cálculo de Frete
        System.out.println("\n### CÁLCULO DE FRETE ###\n");
        ShippingCostVisitor shippingVisitor = new ShippingCostVisitor();
        for (Product product : cart) {
            product.accept(shippingVisitor);
        }
        System.out.println("───────────────────────────────────────");
        System.out.printf("📦 TOTAL DE FRETE: $%.2f\n", shippingVisitor.getTotalShipping());
        
        // Visitor 4: Relatório Detalhado
        DetailedReportVisitor reportVisitor = new DetailedReportVisitor();
        for (Product product : cart) {
            product.accept(reportVisitor);
        }
        reportVisitor.printReport();
    }
}
```

### Exemplo 2: Sistema de Análise de Expressões Matemáticas

```java
// Interface Element - Expressões
interface Expression {
    void accept(ExpressionVisitor visitor);
}

// Concrete Elements - Números e Operações
class NumberExpression implements Expression {
    private double value;
    
    public NumberExpression(double value) {
        this.value = value;
    }
    
    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitNumber(this);
    }
    
    public double getValue() {
        return value;
    }
}

class AdditionExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public AdditionExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitAddition(this);
    }
    
    public Expression getLeft() { return left; }
    public Expression getRight() { return right; }
}

class SubtractionExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public SubtractionExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitSubtraction(this);
    }
    
    public Expression getLeft() { return left; }
    public Expression getRight() { return right; }
}

class MultiplicationExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public MultiplicationExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitMultiplication(this);
    }
    
    public Expression getLeft() { return left; }
    public Expression getRight() { return right; }
}

class DivisionExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public DivisionExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public void accept(ExpressionVisitor visitor) {
        visitor.visitDivision(this);
    }
    
    public Expression getLeft() { return left; }
    public Expression getRight() { return right; }
}

// Interface Visitor
interface ExpressionVisitor {
    void visitNumber(NumberExpression number);
    void visitAddition(AdditionExpression addition);
    void visitSubtraction(SubtractionExpression subtraction);
    void visitMultiplication(MultiplicationExpression multiplication);
    void visitDivision(DivisionExpression division);
}

// Concrete Visitor 1 - Calculadora (Avalia expressão)
class EvaluatorVisitor implements ExpressionVisitor {
    private double result;
    
    @Override
    public void visitNumber(NumberExpression number) {
        result = number.getValue();
    }
    
    @Override
    public void visitAddition(AdditionExpression addition) {
        addition.getLeft().accept(this);
        double left = result;
        
        addition.getRight().accept(this);
        double right = result;
        
        result = left + right;
        System.out.printf("   %.2f + %.2f = %.2f\n", left, right, result);
    }
    
    @Override
    public void visitSubtraction(SubtractionExpression subtraction) {
        subtraction.getLeft().accept(this);
        double left = result;
        
        subtraction.getRight().accept(this);
        double right = result;
        
        result = left - right;
        System.out.printf("   %.2f - %.2f = %.2f\n", left, right, result);
    }
    
    @Override
    public void visitMultiplication(MultiplicationExpression multiplication) {
        multiplication.getLeft().accept(this);
        double left = result;
        
        multiplication.getRight().accept(this);
        double right = result;
        
        result = left * right;
        System.out.printf("   %.2f × %.2f = %.2f\n", left, right, result);
    }
    
    @Override
    public void visitDivision(DivisionExpression division) {
        division.getLeft().accept(this);
        double left = result;
        
        division.getRight().accept(this);
        double right = result;
        
        if (right == 0) {
            System.out.println("   ❌ ERRO: Divisão por zero!");
            result = Double.NaN;
        } else {
            result = left / right;
            System.out.printf("   %.2f ÷ %.2f = %.2f\n", left, right, result);
        }
    }
    
    public double getResult() {
        return result;
    }
}

// Concrete Visitor 2 - Impressora (Formato legível)
class PrinterVisitor implements ExpressionVisitor {
    private StringBuilder output = new StringBuilder();
    
    @Override
    public void visitNumber(NumberExpression number) {
        output.append(number.getValue());
    }
    
    @Override
    public void visitAddition(AdditionExpression addition) {
        output.append("(");
        addition.getLeft().accept(this);
        output.append(" + ");
        addition.getRight().accept(this);
        output.append(")");
    }
    
    @Override
    public void visitSubtraction(SubtractionExpression subtraction) {
        output.append("(");
        subtraction.getLeft().accept(this);
        output.append(" - ");
        subtraction.getRight().accept(this);
        output.append(")");
    }
    
    @Override
    public void visitMultiplication(MultiplicationExpression multiplication) {
        output.append("(");
        multiplication.getLeft().accept(this);
        output.append(" × ");
        multiplication.getRight().accept(this);
        output.append(")");
    }
    
    @Override
    public void visitDivision(DivisionExpression division) {
        output.append("(");
        division.getLeft().accept(this);
        output.append(" ÷ ");
        division.getRight().accept(this);
        output.append(")");
    }
    
    public String getOutput() {
        return output.toString();
    }
    
    public void reset() {
        output = new StringBuilder();
    }
}

// Concrete Visitor 3 - Contador de Operações
class OperationCounterVisitor implements ExpressionVisitor {
    private int numberCount = 0;
    private int additionCount = 0;
    private int subtractionCount = 0;
    private int multiplicationCount = 0;
    private int divisionCount = 0;
    
    @Override
    public void visitNumber(NumberExpression number) {
        numberCount++;
    }
    
    @Override
    public void visitAddition(AdditionExpression addition) {
        additionCount++;
        addition.getLeft().accept(this);
        addition.getRight().accept(this);
    }
    
    @Override
    public void visitSubtraction(SubtractionExpression subtraction) {
        subtractionCount++;
        subtraction.getLeft().accept(this);
        subtraction.getRight().accept(this);
    }
    
    @Override
    public void visitMultiplication(MultiplicationExpression multiplication) {
        multiplicationCount++;
        multiplication.getLeft().accept(this);
        multiplication.getRight().accept(this);
    }
    
    @Override
    public void visitDivision(DivisionExpression division) {
        divisionCount++;
        division.getLeft().accept(this);
        division.getRight().accept(this);
    }
    
    public void printStatistics() {
        System.out.println("\n📊 ESTATÍSTICAS DA EXPRESSÃO:");
        System.out.println("───────────────────────────────────────");
        System.out.printf("🔢 Números: %d\n", numberCount);
        System.out.printf("➕ Adições: %d\n", additionCount);
        System.out.printf("➖ Subtrações: %d\n", subtractionCount);
        System.out.printf("✖️  Multiplicações: %d\n", multiplicationCount);
        System.out.printf("➗ Divisões: %d\n", divisionCount);
        System.out.printf("📈 Total de operações: %d\n",
            additionCount + subtractionCount + multiplicationCount + divisionCount);
        System.out.println("───────────────────────────────────────");
    }
}

// Concrete Visitor 4 - Conversor para código
class CodeGeneratorVisitor implements ExpressionVisitor {
    private StringBuilder code = new StringBuilder();
    
    @Override
    public void visitNumber(NumberExpression number) {
        code.append(number.getValue());
    }
    
    @Override
    public void visitAddition(AdditionExpression addition) {
        code.append("add(");
        addition.getLeft().accept(this);
        code.append(", ");
        addition.getRight().accept(this);
        code.append(")");
    }
    
    @Override
    public void visitSubtraction(SubtractionExpression subtraction) {
        code.append("subtract(");
        subtraction.getLeft().accept(this);
        code.append(", ");
        subtraction.getRight().accept(this);
        code.append(")");
    }
    
    @Override
    public void visitMultiplication(MultiplicationExpression multiplication) {
        code.append("multiply(");
        multiplication.getLeft().accept(this);
        code.append(", ");
        multiplication.getRight().accept(this);
        code.append(")");
    }
    
    @Override
    public void visitDivision(DivisionExpression division) {
        code.append("divide(");
        division.getLeft().accept(this);
        code.append(", ");
        division.getRight().accept(this);
        code.append(")");
    }
    
    public String getCode() {
        return code.toString();
    }
    
    public void reset() {
        code = new StringBuilder();
    }
}

// Uso
public class ExpressionAnalysisExample {
    public static void main(String[] args) {
        System.out.println("═══════════════════════════════════════");
        System.out.println("   SISTEMA DE ANÁLISE DE EXPRESSÕES");
        System.out.println("═══════════════════════════════════════\n");
        
        // Cria expressão: ((10 + 5) × (20 - 8)) ÷ 4
        Expression expr = new DivisionExpression(
            new MultiplicationExpression(
                new AdditionExpression(
                    new NumberExpression(10),
                    new NumberExpression(5)
                ),
                new SubtractionExpression(
                    new NumberExpression(20),
                    new NumberExpression(8)
                )
            ),
            new NumberExpression(4)
        );
        
        // Visitor 1: Imprimir expressão
        System.out.println("### EXPRESSÃO ###\n");
        PrinterVisitor printer = new PrinterVisitor();
        expr.accept(printer);
        System.out.println("📝 " + printer.getOutput() + "\n");
        
        // Visitor 2: Avaliar expressão
        System.out.println("### AVALIAÇÃO PASSO A PASSO ###\n");
        EvaluatorVisitor evaluator = new EvaluatorVisitor();
        expr.accept(evaluator);
        System.out.println("\n🎯 RESULTADO FINAL: " + evaluator.getResult() + "\n");
        
        // Visitor 3: Estatísticas
        OperationCounterVisitor counter = new OperationCounterVisitor();
        expr.accept(counter);
        counter.printStatistics();
        
        // Visitor 4: Gerar código
        System.out.println("\n### GERAÇÃO DE CÓDIGO ###\n");
        CodeGeneratorVisitor codeGen = new CodeGeneratorVisitor();
        expr.accept(codeGen);
        System.out.println("💻 Código gerado:");
        System.out.println("   " + codeGen.getCode());
        
        // Nova expressão: (100 ÷ 0) - Teste de divisão por zero
        System.out.println("\n\n═══════════════════════════════════════");
        System.out.println("   TESTE: DIVISÃO POR ZERO");
        System.out.println("═══════════════════════════════════════\n");
        
        Expression divByZero = new DivisionExpression(
            new NumberExpression(100),
            new NumberExpression(0)
        );
        
        printer.reset();
        divByZero.accept(printer);
        System.out.println("📝 " + printer.getOutput() + "\n");
        
        EvaluatorVisitor eval2 = new EvaluatorVisitor();
        divByZero.accept(eval2);
        System.out.println("\n🎯 RESULTADO: " + 
            (Double.isNaN(eval2.getResult()) ? "ERRO" : eval2.getResult()));
    }
}
```

## 🎯 Quando Usar?

### ✅ Use quando:
- **Operação em estrutura complexa**: Quer executar operação sobre todos elementos (árvore de objetos)
- **Muitas operações não relacionadas**: Limpeza de código, separando comportamentos auxiliares
- **Comportamento faz sentido em apenas algumas classes**: Da hierarquia
- **Adicionar operações frequentemente**: Sem modificar classes de elementos
- **Estrutura estável, operações mudam**: Hierarquia de elementos raramente muda
- **Double Dispatch necessário**: Precisa resolver tipo em runtime

### 📝 Exemplos de aplicação:
- **Compiladores**: AST (Abstract Syntax Tree) - análise, otimização, geração de código
- **Export/Import**: XML, JSON, CSV, PDF de estrutura de dados
- **Cálculos**: Impostos, frete, preços com regras específicas por tipo
- **Report generation**: Diferentes formatos de relatório
- **Validação**: Regras específicas por tipo de elemento
- **Document processing**: Diferentes tipos de nós/elementos

### ❌ Evite quando:
- **Hierarquia instável**: Classes adicionadas/removidas frequentemente
- **Poucas operações**: 1-2 operações não justificam complexidade
- **Operações simples**: Comportamento trivial não precisa visitor
- **Acesso a membros privados**: Visitor precisa acessar estado interno privado

## 🚀 Como Implementar

1. **Declare interface Visitor** com método "visit" para cada ConcreteElement

2. **Declare interface Element** com método `accept(Visitor)`

3. **Implemente accept()** em todos ConcreteElements redirecionando para método visitor apropriado

4. **Elementos trabalham com visitors** via interface Visitor

5. **Crie ConcreteVisitors** implementando todos métodos visit

6. **Cliente cria visitor** e passa para elementos via `accept()`

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Open/Closed Principle**: Novos comportamentos sem modificar elementos
- **Single Responsibility Principle**: Múltiplas versões de comportamento em uma classe
- **Acumula estado**: Visitor pode acumular informações úteis durante travessia
- **Separa algoritmos**: Comportamentos ficam em classes visitor separadas
- **Facilita adicionar operações**: Nova operação = novo visitor

### ❌ Desvantagens:
- **Atualizar todos visitors**: Quando classe adicionada/removida da hierarquia
- **Quebra encapsulamento**: Visitor pode precisar acessar campos/métodos privados
- **Complexidade**: Double dispatch não é óbvio
- **Hierarquia deve ser estável**: Adicionar elemento força mudança em todos visitors

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Quando usar | Modificação |
|--------|------|-------------|-------------|
| **Visitor** | Separar algoritmos de objetos | Estrutura estável, operações mudam | Não modifica elementos |
| **Strategy** | Algoritmos intercambiáveis | Trocar algoritmo em runtime | Modifica comportamento objeto |
| **Command** | Encapsular requisição | Undo/redo, queue, log | Operações como objetos |
| **Iterator** | Acesso sequencial | Percorrer coleção | Não executa operações |

## 🔗 Relações com Outros Padrões

- **Visitor vs Command**:
  - Visitor: Versão poderosa de Command
  - Visitor pode executar operações sobre objetos de classes diferentes
  - Command: Operação como objeto

- **Visitor + Composite**:
  - Visitor pode executar operação sobre árvore Composite inteira
  - Composite fornece estrutura, Visitor percorre e opera

- **Visitor + Iterator**:
  - Visitor percorre estrutura complexa
  - Iterator fornece acesso aos elementos
  - Visitor executa operação, mesmo se classes diferentes

## 📚 Conceitos-Chave para Lembrar

1. **Separação de algoritmos**: Comportamentos em classes visitor separadas
2. **Double Dispatch**: Duas chamadas polimórficas resolvem tipo
3. **Método accept()**: Elementos aceitam visitor e delegam chamada
4. **Open/Closed**: Adiciona operações sem modificar elementos
5. **Estrutura estável**: Funciona melhor quando hierarquia de elementos é estável
6. **Quebra encapsulamento**: Trade-off de expor estado interno

## 🔍 Analogia do Mundo Real

**Agente de seguros experiente**: Ele visita cada prédio de um bairro tentando vender seguros. Dependendo do tipo de organização no prédio, ele oferece apólices especializadas:
- **Residencial**: Vende seguro médico
- **Banco**: Vende seguro contra roubo
- **Cafeteria**: Vende seguro contra incêndio e inundação

O agente (visitor) adapta sua abordagem (operação) baseado no tipo de organização (elemento) que ele visita. As organizações "aceitam" a visita permitindo que o agente faça sua proposta específica.

## ⚠️ Considerações Importantes

### Double Dispatch explicado:

```java
// SEM Visitor - Type checking manual
for (Shape shape : shapes) {
    if (shape instanceof Circle) {
        exportCircle((Circle) shape);
    } else if (shape instanceof Rectangle) {
        exportRectangle((Rectangle) shape);
    }
}

// COM Visitor - Double Dispatch
// 1ª dispatch: shape.accept() - polimorfismo baseado em tipo de shape
// 2ª dispatch: visitor.visitCircle(this) - polimorfismo baseado em tipo do visitor

for (Shape shape : shapes) {
    shape.accept(exportVisitor); // SEM type checking!
}

// Circle
public void accept(Visitor v) {
    v.visitCircle(this); // Circle SABE que é Circle
}

// Rectangle
public void accept(Visitor v) {
    v.visitRectangle(this); // Rectangle SABE que é Rectangle
}
```

### Visitor com Composite (caso comum):

```java
// Composite Element
class CompoundShape implements Shape {
    private List<Shape> children = new ArrayList<>();
    
    @Override
    public void accept(Visitor visitor) {
        // Visita a si mesmo
        visitor.visitCompoundShape(this);
        
        // Visita todos filhos
        for (Shape child : children) {
            child.accept(visitor);
        }
    }
}

// Visitor percorre árvore inteira automaticamente
XMLExportVisitor visitor = new XMLExportVisitor();
compoundShape.accept(visitor); // Percorre toda árvore!
```

### Problema de acesso a membros privados:

```java
// ❌ PROBLEMA: Visitor precisa de dados privados
class Product {
    private double cost;        // Privado
    private double markup;      // Privado
    
    public void accept(Visitor v) {
        v.visitProduct(this);
    }
}

class PriceCalculator implements Visitor {
    public void visitProduct(Product p) {
        // ❌ Não consegue acessar cost e markup!
        // double price = p.cost * (1 + p.markup);
    }
}

// ✅ SOLUÇÃO 1: Adicionar getters (quebra encapsulamento)
class Product {
    private double cost;
    private double markup;
    
    public double getCost() { return cost; }
    public double getMarkup() { return markup; }
}

// ✅ SOLUÇÃO 2: Nested class (se linguagem suporta)
class Product {
    private double cost;
    private double markup;
    
    // Visitor interno tem acesso a membros privados
    public static class ProductVisitor implements Visitor {
        public void visitProduct(Product p) {
            double price = p.cost * (1 + p.markup); // OK!
        }
    }
}
```

### Visitor retornando valores:

```java
// Visitor tradicional não retorna valor
interface Visitor {
    void visitElement(Element e);
}

// Visitor genérico com retorno
interface Visitor<T> {
    T visitElement(Element e);
}

class AreaCalculator implements Visitor<Double> {
    @Override
    public Double visitCircle(Circle c) {
        return Math.PI * c.getRadius() * c.getRadius();
    }
    
    @Override
    public Double visitRectangle(Rectangle r) {
        return r.getWidth() * r.getHeight();
    }
}

// Uso
Double area = shape.accept(areaCalculator);
```

### Design Guidelines:
- **Interface estável**: Hierarquia de elementos deve ser estável
- **Visitors mudam**: Adicionar novos visitors deve ser fácil
- **Um método por tipo**: Visitor tem um método para cada ConcreteElement
- **Sem null checks**: Todos visitors devem implementar todos métodos
- **Estado no visitor**: Visitor pode acumular estado durante visitas
- **Composite-friendly**: Funciona muito bem com árvores/composites
- **Accept em todos**: Todos ConcreteElements devem implementar accept()

---

> **💡 Dica de Estudo:** Visitor é como um inspetor de saúde que visita diferentes estabelecimentos (restaurante, padaria, mercado). Ele executa inspeções (operação) específicas para cada tipo, mas os estabelecimentos não precisam saber como fazer inspeção - apenas "aceitam" o inspetor e ele sabe o que verificar em cada lugar!

> **🔑 Conceito-chave:** Double Dispatch = elemento conhece seu tipo → chama método visitor específico → visitor executa operação apropriada. Sem if/instanceof!

> **📖 Referência:** [Refactoring Guru - Visitor](https://refactoring.guru/design-patterns/visitor)
