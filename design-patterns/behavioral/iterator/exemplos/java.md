# Exemplos Práticos - Iterator (Java)

## Exemplo 1: Coleção de Livros com Diferentes Travessias

```java
import java.util.*;

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

## Exemplo 2: Árvore com Diferentes Estratégias de Travessia

```java
import java.util.*;

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
