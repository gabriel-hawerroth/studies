# Iterator

## 🎯 Intenção

O Iterator é um padrão de projeto comportamental que permite percorrer elementos de uma coleção sem expor sua representação subjacente (lista, pilha, árvore, etc.). Ele fornece uma maneira uniforme de acessar elementos sequencialmente sem conhecer os detalhes internos da estrutura de dados.

## 🚩 Problema

Coleções são um dos tipos de dados mais usados em programação. Uma coleção é apenas um contêiner para um grupo de objetos. A maioria das coleções armazena seus elementos em listas simples, mas algumas são baseadas em pilhas, árvores, grafos e outras estruturas de dados complexas.

### Resultado problemático:
```java
// Cliente precisa conhecer estrutura interna
class Client {
    public void processArrayList(ArrayList<Item> items) {
        for (int i = 0; i < items.size(); i++) {
            Item item = items.get(i);
            process(item);
        }
    }
    
    public void processTree(TreeNode root) {
        // Precisa saber como percorrer árvore
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);
        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();
            process(node.getData());
            // Adiciona filhos...
        }
    }
    
    public void processGraph(Graph graph) {
        // Precisa implementar BFS/DFS
        Queue<Node> queue = new LinkedList<>();
        Set<Node> visited = new HashSet<>();
        // Algoritmo complexo...
    }
}
```

**Problemas:**
- **Acoplamento alto**: Cliente conhece estrutura interna da coleção
- **Código duplicado**: Lógica de travessia repetida em vários lugares
- **Difícil manutenção**: Mudanças na estrutura afetam todo código cliente
- **Sem uniformidade**: Cada coleção tem interface diferente
- **Responsabilidades misturadas**: Coleção acumula lógica de travessia
- **Impossível múltiplas travessias**: Não dá para percorrer a mesma coleção de formas diferentes simultaneamente

## ✅ Solução

A ideia principal do padrão Iterator é extrair o comportamento de travessia de uma coleção para um objeto separado chamado **iterator**. Além de implementar o algoritmo, um iterator encapsula todos os detalhes da travessia, como a posição atual e quantos elementos faltam até o fim.

### Características-chave:
- **Encapsulamento**: Esconde detalhes da travessia
- **Interface uniforme**: Mesma interface para todas coleções
- **Múltiplos iterators**: Vários podem percorrer a mesma coleção
- **Independência**: Iterators mantém estado próprio
- **Desacoplamento**: Cliente independente da estrutura
- **Travessias variadas**: Diferentes algoritmos de percurso

```java
// Interface Iterator
interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Interface Collection
interface Collection<T> {
    Iterator<T> createIterator();
}

// Cliente usa interface uniforme
void processCollection(Collection<Item> collection) {
    Iterator<Item> iterator = collection.createIterator();
    while (iterator.hasNext()) {
        Item item = iterator.next();
        process(item);
    }
}

// Funciona com qualquer coleção!
processCollection(new ArrayList<>());
processCollection(new TreeCollection<>());
processCollection(new GraphCollection<>());
```

## 🏗️ Estrutura

```
Client → Collection (interface) ← ConcreteCollection
              ↓                           ↓
       createIterator()            createIterator()
              ↓                           ↓
         Iterator (interface) ← ConcreteIterator
              ↓                           ↓
         hasNext()                   [traversal state]
         next()                      [current position]
```

### Componentes:
- **Iterator**: Interface que declara operações para travessia
- **ConcreteIterator**: Implementa algoritmo específico de travessia
- **Collection**: Interface que declara método para criar iterator
- **ConcreteCollection**: Retorna instância de iterator apropriado
- **Client**: Trabalha com coleções e iterators via interfaces

## 🎯 Quando Usar?

### ✅ Use quando:
- **Estrutura complexa**: Coleção tem estrutura interna complexa
- **Ocultar complexidade**: Não quer expor detalhes aos clientes
- **Reduzir duplicação**: Travessia repetida em vários lugares
- **Múltiplas travessias**: Diferentes formas de percorrer
- **Desconhecimento prévio**: Tipos de estruturas desconhecidos
- **Interface uniforme**: Quer tratar coleções uniformemente

### 📝 Exemplos de aplicação:
- **Collections Framework**: Java Iterator, C# IEnumerator
- **DOM Tree**: Document traversal
- **File Systems**: Directory iteration
- **Graph algorithms**: BFS, DFS iterators
- **Database cursors**: Result set iteration

### ❌ Evite quando:
- **Coleções simples**: Array ou lista simples já basta
- **Acesso aleatório**: Precisa de acesso direto por índice
- **Performance crítica**: Overhead de objetos pode impactar

## 🚀 Como Implementar

1. **Declare interface Iterator** com métodos `hasNext()` e `next()`

2. **Declare interface Collection** com método `createIterator()`

3. **Implemente iterators concretos** para cada coleção

4. **Implemente interface Collection** nas classes de coleção

5. **Cliente usa iterators** ao invés de acessar coleção diretamente

6. **Considere iterators bidirecionais** se necessário (previous())

## ⚖️ Prós e Contras

### ✅ Vantagens:
- **Single Responsibility**: Separa travessia de coleção
- **Open/Closed**: Novos iterators sem mudar código
- **Iteração paralela**: Múltiplos iterators independentes
- **Iteração diferida**: Pode pausar e continuar depois
- **Interface uniforme**: Mesmo código para diferentes coleções

### ❌ Desvantagens:
- **Overkill**: Para coleções simples pode ser excessivo
- **Performance**: Menos eficiente que acesso direto
- **Complexidade**: Mais classes e interfaces

## 🔗 Diferenças de Outros Padrões

| Padrão | Foco | Travessia | Estado |
|--------|------|-----------|--------|
| **Iterator** | Percorrer coleção | Externa (cliente controla) | Mantém posição |
| **Visitor** | Operação em elementos | Interna (visitor percorre) | Stateless |
| **Composite** | Estrutura árvore | Recursiva | Hierárquica |
| **Strategy** | Algoritmo variável | N/A | Algoritmo encapsulado |

## 🔗 Relações com Outros Padrões

- **Composite**: Iterators podem percorrer árvores Composite
- **Factory Method**: Coleções podem usar Factory para criar iterators
- **Memento**: Pode salvar estado da iteração
- **Visitor**: Visitor percorre estrutura, Iterator dá acesso aos elementos

## 📚 Conceitos-Chave para Lembrar

1. **Separação de responsabilidades**: Coleção armazena, iterator percorre
2. **Interface uniforme**: Mesma interface para todas coleções
3. **Estado independente**: Cada iterator mantém posição própria
4. **Múltiplas travessias**: Vários iterators na mesma coleção
5. **Encapsulamento**: Oculta estrutura interna da coleção
6. **Travessias variadas**: Diferentes algoritmos de percurso

## 🔍 Analogia do Mundo Real

**Guia turístico em Roma**: Você pode explorar Roma de várias formas - caminhando aleatoriamente (acesso direto), usando app de navegação (iterator simples), ou contratando guia local (iterator especializado). Cada forma de percorrer a cidade é um iterator diferente sobre a mesma coleção (pontos turísticos de Roma). O guia conhece a estrutura (ruas, trajetos) mas você só vê os pontos turísticos em sequência.

## ⚠️ Considerações Importantes

### Iterator em Java:
```java
// Java fornece interface Iterator padrão
interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException();
    }
}

// E interface Iterable para coleções
interface Iterable<E> {
    Iterator<E> iterator();
}

// Uso com enhanced for
for (Item item : collection) {
    // Usa iterator internamente
}
```

### Iterator externo vs interno:

#### Externo (External Iterator)
```java
// Cliente controla iteração
Iterator<Item> iter = collection.iterator();
while (iter.hasNext()) {
    Item item = iter.next();
    process(item);
}
```

#### Interno (Internal Iterator)
```java
// Coleção controla iteração
collection.forEach(item -> process(item));
```

### Fail-fast iterators:
```java
// Lança ConcurrentModificationException se coleção mudar
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");

Iterator<String> iter = list.iterator();
iter.next();
list.add("C"); // Modifica durante iteração
iter.next(); // ❌ Lança exceção!
```

### Design Guidelines:
- **Imutabilidade**: Iterator não deve modificar coleção
- **Fail-fast**: Detecte modificações concorrentes
- **Lazy evaluation**: Calcule elementos sob demanda se possível
- **Resource cleanup**: Feche recursos ao finalizar iteração

---

> **💡 Dica de Estudo:** Iterator é como um "controle remoto" para sua TV (coleção) - você aperta os botões (hasNext/next) sem precisar saber como a TV funciona internamente. Diferentes controles remotos (iterators) podem ter funções diferentes, mas todos controlam a mesma TV.

> **📖 Referência:** [Refactoring Guru - Iterator](https://refactoring.guru/design-patterns/iterator)
