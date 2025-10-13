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

## 💻 Exemplos Práticos

### Exemplo 1: Coleção de Livros com Diferentes Travessias

```java
// Classe do item
class Book {
    private String title;
    private String author;
    private String genre;
    private int year;
    
    public Book(String title, String author, String genre, int year) {
        this.title = title;
        this.author = author;
        this.genre = genre;
        this.year = year;
    }
    
    // Getters
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public String getGenre() { return genre; }
    public int getYear() { return year; }
    
    @Override
    public String toString() {
        return String.format("\"%s\" por %s (%d) - %s", title, author, year, genre);
    }
}

// Interface Iterator
interface BookIterator {
    boolean hasNext();
    Book next();
}

// Interface Collection
interface BookCollection {
    BookIterator createIterator();
    BookIterator createReverseIterator();
    BookIterator createGenreIterator(String genre);
}

// Iterator por ordem normal
class NormalIterator implements BookIterator {
    private List<Book> books;
    private int position;
    
    public NormalIterator(List<Book> books) {
        this.books = books;
        this.position = 0;
    }
    
    @Override
    public boolean hasNext() {
        return position < books.size();
    }
    
    @Override
    public Book next() {
        if (!hasNext()) {
            throw new NoSuchElementException("Não há mais elementos");
        }
        return books.get(position++);
    }
}

// Iterator por ordem reversa
class ReverseIterator implements BookIterator {
    private List<Book> books;
    private int position;
    
    public ReverseIterator(List<Book> books) {
        this.books = books;
        this.position = books.size() - 1;
    }
    
    @Override
    public boolean hasNext() {
        return position >= 0;
    }
    
    @Override
    public Book next() {
        if (!hasNext()) {
            throw new NoSuchElementException("Não há mais elementos");
        }
        return books.get(position--);
    }
}

// Iterator filtrado por gênero
class GenreIterator implements BookIterator {
    private List<Book> books;
    private String genre;
    private int position;
    
    public GenreIterator(List<Book> books, String genre) {
        this.books = books;
        this.genre = genre;
        this.position = 0;
    }
    
    @Override
    public boolean hasNext() {
        while (position < books.size()) {
            if (books.get(position).getGenre().equalsIgnoreCase(genre)) {
                return true;
            }
            position++;
        }
        return false;
    }
    
    @Override
    public Book next() {
        if (!hasNext()) {
            throw new NoSuchElementException("Não há mais elementos");
        }
        return books.get(position++);
    }
}

// Coleção concreta
class Library implements BookCollection {
    private List<Book> books;
    
    public Library() {
        this.books = new ArrayList<>();
    }
    
    public void addBook(Book book) {
        books.add(book);
        System.out.println("📚 Livro adicionado: " + book.getTitle());
    }
    
    @Override
    public BookIterator createIterator() {
        return new NormalIterator(books);
    }
    
    @Override
    public BookIterator createReverseIterator() {
        return new ReverseIterator(books);
    }
    
    @Override
    public BookIterator createGenreIterator(String genre) {
        return new GenreIterator(books, genre);
    }
    
    public int size() {
        return books.size();
    }
}

// Cliente que usa iterators
class LibraryReader {
    public void readAllBooks(Library library) {
        System.out.println("\n=== Lendo todos os livros (ordem normal) ===");
        BookIterator iterator = library.createIterator();
        int count = 1;
        
        while (iterator.hasNext()) {
            Book book = iterator.next();
            System.out.println(count++ + ". " + book);
        }
    }
    
    public void readBooksReverse(Library library) {
        System.out.println("\n=== Lendo livros (ordem reversa) ===");
        BookIterator iterator = library.createReverseIterator();
        int count = 1;
        
        while (iterator.hasNext()) {
            Book book = iterator.next();
            System.out.println(count++ + ". " + book);
        }
    }
    
    public void readBooksByGenre(Library library, String genre) {
        System.out.println("\n=== Lendo livros de " + genre + " ===");
        BookIterator iterator = library.createGenreIterator(genre);
        int count = 1;
        
        while (iterator.hasNext()) {
            Book book = iterator.next();
            System.out.println(count++ + ". " + book);
        }
    }
    
    public void demonstrateMultipleIterators(Library library) {
        System.out.println("\n=== Múltiplos iterators simultâneos ===");
        
        BookIterator iter1 = library.createIterator();
        BookIterator iter2 = library.createIterator();
        
        System.out.println("Iterator 1 (primeiro livro): " + iter1.next().getTitle());
        System.out.println("Iterator 2 (primeiro livro): " + iter2.next().getTitle());
        System.out.println("Iterator 1 (segundo livro): " + iter1.next().getTitle());
        System.out.println("Iterator 2 (segundo livro): " + iter2.next().getTitle());
        
        System.out.println("\n✅ Cada iterator mantém seu próprio estado!");
    }
}

// Uso
public class LibraryExample {
    public static void main(String[] args) {
        Library library = new Library();
        
        // Adiciona livros
        System.out.println("=== Montando biblioteca ===");
        library.addBook(new Book("1984", "George Orwell", "Ficção", 1949));
        library.addBook(new Book("O Senhor dos Anéis", "J.R.R. Tolkien", "Fantasia", 1954));
        library.addBook(new Book("Harry Potter", "J.K. Rowling", "Fantasia", 1997));
        library.addBook(new Book("Clean Code", "Robert Martin", "Técnico", 2008));
        library.addBook(new Book("O Hobbit", "J.R.R. Tolkien", "Fantasia", 1937));
        
        System.out.println("\n📖 Total de livros: " + library.size());
        
        // Usa diferentes iterators
        LibraryReader reader = new LibraryReader();
        
        reader.readAllBooks(library);
        reader.readBooksReverse(library);
        reader.readBooksByGenre(library, "Fantasia");
        reader.demonstrateMultipleIterators(library);
    }
}
```

### Exemplo 2: Árvore com Diferentes Estratégias de Travessia

```java
// Nó da árvore
class TreeNode<T> {
    private T data;
    private TreeNode<T> left;
    private TreeNode<T> right;
    
    public TreeNode(T data) {
        this.data = data;
    }
    
    public void setLeft(TreeNode<T> left) { this.left = left; }
    public void setRight(TreeNode<T> right) { this.right = right; }
    public T getData() { return data; }
    public TreeNode<T> getLeft() { return left; }
    public TreeNode<T> getRight() { return right; }
}

// Interface Iterator
interface TreeIterator<T> {
    boolean hasNext();
    T next();
}

// Interface Collection
interface Tree<T> {
    TreeIterator<T> createInOrderIterator();
    TreeIterator<T> createPreOrderIterator();
    TreeIterator<T> createPostOrderIterator();
    TreeIterator<T> createBreadthFirstIterator();
}

// Iterator In-Order (esquerda, raiz, direita)
class InOrderIterator<T> implements TreeIterator<T> {
    private Stack<TreeNode<T>> stack;
    
    public InOrderIterator(TreeNode<T> root) {
        stack = new Stack<>();
        pushLeftChildren(root);
    }
    
    private void pushLeftChildren(TreeNode<T> node) {
        while (node != null) {
            stack.push(node);
            node = node.getLeft();
        }
    }
    
    @Override
    public boolean hasNext() {
        return !stack.isEmpty();
    }
    
    @Override
    public T next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        
        TreeNode<T> node = stack.pop();
        T data = node.getData();
        
        if (node.getRight() != null) {
            pushLeftChildren(node.getRight());
        }
        
        return data;
    }
}

// Iterator Pre-Order (raiz, esquerda, direita)
class PreOrderIterator<T> implements TreeIterator<T> {
    private Stack<TreeNode<T>> stack;
    
    public PreOrderIterator(TreeNode<T> root) {
        stack = new Stack<>();
        if (root != null) {
            stack.push(root);
        }
    }
    
    @Override
    public boolean hasNext() {
        return !stack.isEmpty();
    }
    
    @Override
    public T next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        
        TreeNode<T> node = stack.pop();
        
        // Push direita primeiro (será processada depois)
        if (node.getRight() != null) {
            stack.push(node.getRight());
        }
        if (node.getLeft() != null) {
            stack.push(node.getLeft());
        }
        
        return node.getData();
    }
}

// Iterator Breadth-First (por nível)
class BreadthFirstIterator<T> implements TreeIterator<T> {
    private Queue<TreeNode<T>> queue;
    
    public BreadthFirstIterator(TreeNode<T> root) {
        queue = new LinkedList<>();
        if (root != null) {
            queue.offer(root);
        }
    }
    
    @Override
    public boolean hasNext() {
        return !queue.isEmpty();
    }
    
    @Override
    public T next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        
        TreeNode<T> node = queue.poll();
        
        if (node.getLeft() != null) {
            queue.offer(node.getLeft());
        }
        if (node.getRight() != null) {
            queue.offer(node.getRight());
        }
        
        return node.getData();
    }
}

// Árvore binária concreta
class BinaryTree<T> implements Tree<T> {
    private TreeNode<T> root;
    
    public void setRoot(TreeNode<T> root) {
        this.root = root;
    }
    
    public TreeNode<T> getRoot() {
        return root;
    }
    
    @Override
    public TreeIterator<T> createInOrderIterator() {
        return new InOrderIterator<>(root);
    }
    
    @Override
    public TreeIterator<T> createPreOrderIterator() {
        return new PreOrderIterator<>(root);
    }
    
    @Override
    public TreeIterator<T> createPostOrderIterator() {
        // Implementação similar...
        throw new UnsupportedOperationException("Post-order não implementado neste exemplo");
    }
    
    @Override
    public TreeIterator<T> createBreadthFirstIterator() {
        return new BreadthFirstIterator<>(root);
    }
}

// Cliente
class TreeTraverser {
    public <T> void traverse(String type, TreeIterator<T> iterator) {
        System.out.println("\n=== Travessia " + type + " ===");
        System.out.print("Elementos: ");
        
        while (iterator.hasNext()) {
            System.out.print(iterator.next() + " ");
        }
        System.out.println();
    }
}

// Uso
public class TreeExample {
    public static void main(String[] args) {
        // Cria árvore:
        //        1
        //       / \
        //      2   3
        //     / \   \
        //    4   5   6
        
        BinaryTree<Integer> tree = new BinaryTree<>();
        
        TreeNode<Integer> root = new TreeNode<>(1);
        TreeNode<Integer> node2 = new TreeNode<>(2);
        TreeNode<Integer> node3 = new TreeNode<>(3);
        TreeNode<Integer> node4 = new TreeNode<>(4);
        TreeNode<Integer> node5 = new TreeNode<>(5);
        TreeNode<Integer> node6 = new TreeNode<>(6);
        
        root.setLeft(node2);
        root.setRight(node3);
        node2.setLeft(node4);
        node2.setRight(node5);
        node3.setRight(node6);
        
        tree.setRoot(root);
        
        System.out.println("=== Árvore Binária ===");
        System.out.println("       1");
        System.out.println("      / \\");
        System.out.println("     2   3");
        System.out.println("    / \\   \\");
        System.out.println("   4   5   6");
        
        TreeTraverser traverser = new TreeTraverser();
        
        // Diferentes travessias
        traverser.traverse("In-Order (ESQ-RAIZ-DIR)", 
            tree.createInOrderIterator());
        
        traverser.traverse("Pre-Order (RAIZ-ESQ-DIR)", 
            tree.createPreOrderIterator());
        
        traverser.traverse("Breadth-First (Por Nível)", 
            tree.createBreadthFirstIterator());
        
        // Demonstra múltiplos iterators
        System.out.println("\n=== Múltiplos Iterators Simultâneos ===");
        TreeIterator<Integer> iter1 = tree.createInOrderIterator();
        TreeIterator<Integer> iter2 = tree.createPreOrderIterator();
        
        System.out.println("In-Order primeiro: " + iter1.next());
        System.out.println("Pre-Order primeiro: " + iter2.next());
        System.out.println("In-Order segundo: " + iter1.next());
        System.out.println("Pre-Order segundo: " + iter2.next());
    }
}
```

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
