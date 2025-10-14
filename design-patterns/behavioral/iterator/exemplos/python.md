# Exemplos Práticos - Iterator (Python)

## Exemplo 1: Coleção de Livros com Diferentes Travessias

```python
from abc import ABC, abstractmethod
from typing import List, Optional

# Book - classe do item
class Book:
    def __init__(self, title: str, author: str, genre: str, year: int):
        self.title = title
        self.author = author
        self.genre = genre
        self.year = year
    
    def __str__(self) -> str:
        return f'"{self.title}" por {self.author} ({self.year}) - {self.genre}'

# BookIterator - interface Iterator
class BookIterator(ABC):
    @abstractmethod
    def has_next(self) -> bool:
        pass
    
    @abstractmethod
    def next(self) -> Book:
        pass

# BookCollection - interface Collection
class BookCollection(ABC):
    @abstractmethod
    def create_iterator(self) -> BookIterator:
        pass
    
    @abstractmethod
    def create_reverse_iterator(self) -> BookIterator:
        pass
    
    @abstractmethod
    def create_genre_iterator(self, genre: str) -> BookIterator:
        pass

# NormalIterator - iterator por ordem normal
class NormalIterator(BookIterator):
    def __init__(self, books: List[Book]):
        self._books = books
        self._position = 0
    
    def has_next(self) -> bool:
        return self._position < len(self._books)
    
    def next(self) -> Book:
        if not self.has_next():
            raise StopIteration("Não há mais elementos")
        book = self._books[self._position]
        self._position += 1
        return book

# ReverseIterator - iterator por ordem reversa
class ReverseIterator(BookIterator):
    def __init__(self, books: List[Book]):
        self._books = books
        self._position = len(books) - 1
    
    def has_next(self) -> bool:
        return self._position >= 0
    
    def next(self) -> Book:
        if not self.has_next():
            raise StopIteration("Não há mais elementos")
        book = self._books[self._position]
        self._position -= 1
        return book

# GenreIterator - iterator filtrado por gênero
class GenreIterator(BookIterator):
    def __init__(self, books: List[Book], genre: str):
        self._books = books
        self._genre = genre
        self._position = 0
    
    def has_next(self) -> bool:
        while self._position < len(self._books):
            if self._books[self._position].genre.lower() == self._genre.lower():
                return True
            self._position += 1
        return False
    
    def next(self) -> Book:
        if not self.has_next():
            raise StopIteration("Não há mais elementos")
        book = self._books[self._position]
        self._position += 1
        return book

# Library - coleção concreta
class Library(BookCollection):
    def __init__(self):
        self._books: List[Book] = []
    
    def add_book(self, book: Book) -> None:
        self._books.append(book)
        print(f"📚 Livro adicionado: {book.title}")
    
    def create_iterator(self) -> BookIterator:
        return NormalIterator(self._books)
    
    def create_reverse_iterator(self) -> BookIterator:
        return ReverseIterator(self._books)
    
    def create_genre_iterator(self, genre: str) -> BookIterator:
        return GenreIterator(self._books, genre)
    
    def size(self) -> int:
        return len(self._books)

# LibraryReader - cliente que usa iterators
class LibraryReader:
    def read_all_books(self, library: Library) -> None:
        print("\n=== Lendo todos os livros (ordem normal) ===")
        iterator = library.create_iterator()
        count = 1
        
        while iterator.has_next():
            book = iterator.next()
            print(f"{count}. {book}")
            count += 1
    
    def read_books_reverse(self, library: Library) -> None:
        print("\n=== Lendo livros (ordem reversa) ===")
        iterator = library.create_reverse_iterator()
        count = 1
        
        while iterator.has_next():
            book = iterator.next()
            print(f"{count}. {book}")
            count += 1
    
    def read_books_by_genre(self, library: Library, genre: str) -> None:
        print(f"\n=== Lendo livros de {genre} ===")
        iterator = library.create_genre_iterator(genre)
        count = 1
        
        while iterator.has_next():
            book = iterator.next()
            print(f"{count}. {book}")
            count += 1
    
    def demonstrate_multiple_iterators(self, library: Library) -> None:
        print("\n=== Múltiplos iterators simultâneos ===")
        
        iter1 = library.create_iterator()
        iter2 = library.create_iterator()
        
        print(f"Iterator 1 (primeiro livro): {iter1.next().title}")
        print(f"Iterator 2 (primeiro livro): {iter2.next().title}")
        print(f"Iterator 1 (segundo livro): {iter1.next().title}")
        print(f"Iterator 2 (segundo livro): {iter2.next().title}")
        
        print("\n✅ Cada iterator mantém seu próprio estado!")

# Uso
if __name__ == "__main__":
    library = Library()
    
    # Adiciona livros
    print("=== Montando biblioteca ===")
    library.add_book(Book("1984", "George Orwell", "Ficção", 1949))
    library.add_book(Book("O Senhor dos Anéis", "J.R.R. Tolkien", "Fantasia", 1954))
    library.add_book(Book("Harry Potter", "J.K. Rowling", "Fantasia", 1997))
    library.add_book(Book("Clean Code", "Robert Martin", "Técnico", 2008))
    library.add_book(Book("O Hobbit", "J.R.R. Tolkien", "Fantasia", 1937))
    
    print(f"\n📖 Total de livros: {library.size()}")
    
    # Usa diferentes iterators
    reader = LibraryReader()
    
    reader.read_all_books(library)
    reader.read_books_reverse(library)
    reader.read_books_by_genre(library, "Fantasia")
    reader.demonstrate_multiple_iterators(library)
```

## Exemplo 2: Árvore com Diferentes Estratégias de Travessia

```python
from abc import ABC, abstractmethod
from typing import Optional, List
from collections import deque

# TreeNode - nó da árvore
class TreeNode:
    def __init__(self, data: int):
        self.data = data
        self.left: Optional[TreeNode] = None
        self.right: Optional[TreeNode] = None

# TreeIterator - interface Iterator
class TreeIterator(ABC):
    @abstractmethod
    def has_next(self) -> bool:
        pass
    
    @abstractmethod
    def next(self) -> int:
        pass

# Tree - interface Collection
class Tree(ABC):
    @abstractmethod
    def create_in_order_iterator(self) -> TreeIterator:
        pass
    
    @abstractmethod
    def create_pre_order_iterator(self) -> TreeIterator:
        pass
    
    @abstractmethod
    def create_breadth_first_iterator(self) -> TreeIterator:
        pass

# InOrderIterator - iterator in-order (esquerda, raiz, direita)
class InOrderIterator(TreeIterator):
    def __init__(self, root: Optional[TreeNode]):
        self._stack: List[TreeNode] = []
        self._push_left_children(root)
    
    def _push_left_children(self, node: Optional[TreeNode]) -> None:
        while node:
            self._stack.append(node)
            node = node.left
    
    def has_next(self) -> bool:
        return len(self._stack) > 0
    
    def next(self) -> int:
        if not self.has_next():
            raise StopIteration("Não há mais elementos")
        
        node = self._stack.pop()
        data = node.data
        
        if node.right:
            self._push_left_children(node.right)
        
        return data

# PreOrderIterator - iterator pre-order (raiz, esquerda, direita)
class PreOrderIterator(TreeIterator):
    def __init__(self, root: Optional[TreeNode]):
        self._stack: List[TreeNode] = []
        if root:
            self._stack.append(root)
    
    def has_next(self) -> bool:
        return len(self._stack) > 0
    
    def next(self) -> int:
        if not self.has_next():
            raise StopIteration("Não há mais elementos")
        
        node = self._stack.pop()
        
        # Push direita primeiro (será processada depois)
        if node.right:
            self._stack.append(node.right)
        if node.left:
            self._stack.append(node.left)
        
        return node.data

# BreadthFirstIterator - iterator breadth-first (por nível)
class BreadthFirstIterator(TreeIterator):
    def __init__(self, root: Optional[TreeNode]):
        self._queue: deque = deque()
        if root:
            self._queue.append(root)
    
    def has_next(self) -> bool:
        return len(self._queue) > 0
    
    def next(self) -> int:
        if not self.has_next():
            raise StopIteration("Não há mais elementos")
        
        node = self._queue.popleft()
        
        if node.left:
            self._queue.append(node.left)
        if node.right:
            self._queue.append(node.right)
        
        return node.data

# BinaryTree - árvore binária concreta
class BinaryTree(Tree):
    def __init__(self):
        self._root: Optional[TreeNode] = None
    
    def set_root(self, root: TreeNode) -> None:
        self._root = root
    
    def get_root(self) -> Optional[TreeNode]:
        return self._root
    
    def create_in_order_iterator(self) -> TreeIterator:
        return InOrderIterator(self._root)
    
    def create_pre_order_iterator(self) -> TreeIterator:
        return PreOrderIterator(self._root)
    
    def create_breadth_first_iterator(self) -> TreeIterator:
        return BreadthFirstIterator(self._root)

# TreeTraverser - cliente
class TreeTraverser:
    def traverse(self, traversal_type: str, iterator: TreeIterator) -> None:
        print(f"\n=== Travessia {traversal_type} ===")
        print("Elementos: ", end="")
        
        while iterator.has_next():
            print(iterator.next(), end=" ")
        print()

# Uso
if __name__ == "__main__":
    # Cria árvore:
    #        1
    #       / \
    #      2   3
    #     / \   \
    #    4   5   6
    
    tree = BinaryTree()
    
    root = TreeNode(1)
    node2 = TreeNode(2)
    node3 = TreeNode(3)
    node4 = TreeNode(4)
    node5 = TreeNode(5)
    node6 = TreeNode(6)
    
    root.left = node2
    root.right = node3
    node2.left = node4
    node2.right = node5
    node3.right = node6
    
    tree.set_root(root)
    
    print("=== Árvore Binária ===")
    print("       1")
    print("      / \\")
    print("     2   3")
    print("    / \\   \\")
    print("   4   5   6")
    
    traverser = TreeTraverser()
    
    # Diferentes travessias
    traverser.traverse("In-Order (ESQ-RAIZ-DIR)", 
        tree.create_in_order_iterator())
    
    traverser.traverse("Pre-Order (RAIZ-ESQ-DIR)", 
        tree.create_pre_order_iterator())
    
    traverser.traverse("Breadth-First (Por Nível)", 
        tree.create_breadth_first_iterator())
    
    # Demonstra múltiplos iterators
    print("\n=== Múltiplos Iterators Simultâneos ===")
    iter1 = tree.create_in_order_iterator()
    iter2 = tree.create_pre_order_iterator()
    
    print(f"In-Order primeiro: {iter1.next()}")
    print(f"Pre-Order primeiro: {iter2.next()}")
    print(f"In-Order segundo: {iter1.next()}")
    print(f"Pre-Order segundo: {iter2.next()}")
```
