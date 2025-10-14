# Exemplos Práticos - Iterator (Go)

## Exemplo 1: Coleção de Livros com Diferentes Travessias

```go
package main

import (
	"fmt"
	"strings"
)

// Book - classe do item
type Book struct {
	Title  string
	Author string
	Genre  string
	Year   int
}

func NewBook(title, author, genre string, year int) *Book {
	return &Book{
		Title:  title,
		Author: author,
		Genre:  genre,
		Year:   year,
	}
}

func (b *Book) String() string {
	return fmt.Sprintf("\"%s\" por %s (%d) - %s", b.Title, b.Author, b.Year, b.Genre)
}

// BookIterator - interface Iterator
type BookIterator interface {
	HasNext() bool
	Next() *Book
}

// BookCollection - interface Collection
type BookCollection interface {
	CreateIterator() BookIterator
	CreateReverseIterator() BookIterator
	CreateGenreIterator(genre string) BookIterator
}

// NormalIterator - iterator por ordem normal
type NormalIterator struct {
	books    []*Book
	position int
}

func NewNormalIterator(books []*Book) *NormalIterator {
	return &NormalIterator{
		books:    books,
		position: 0,
	}
}

func (it *NormalIterator) HasNext() bool {
	return it.position < len(it.books)
}

func (it *NormalIterator) Next() *Book {
	if !it.HasNext() {
		panic("Não há mais elementos")
	}
	book := it.books[it.position]
	it.position++
	return book
}

// ReverseIterator - iterator por ordem reversa
type ReverseIterator struct {
	books    []*Book
	position int
}

func NewReverseIterator(books []*Book) *ReverseIterator {
	return &ReverseIterator{
		books:    books,
		position: len(books) - 1,
	}
}

func (it *ReverseIterator) HasNext() bool {
	return it.position >= 0
}

func (it *ReverseIterator) Next() *Book {
	if !it.HasNext() {
		panic("Não há mais elementos")
	}
	book := it.books[it.position]
	it.position--
	return book
}

// GenreIterator - iterator filtrado por gênero
type GenreIterator struct {
	books    []*Book
	genre    string
	position int
}

func NewGenreIterator(books []*Book, genre string) *GenreIterator {
	return &GenreIterator{
		books:    books,
		genre:    genre,
		position: 0,
	}
}

func (it *GenreIterator) HasNext() bool {
	for it.position < len(it.books) {
		if strings.EqualFold(it.books[it.position].Genre, it.genre) {
			return true
		}
		it.position++
	}
	return false
}

func (it *GenreIterator) Next() *Book {
	if !it.HasNext() {
		panic("Não há mais elementos")
	}
	book := it.books[it.position]
	it.position++
	return book
}

// Library - coleção concreta
type Library struct {
	books []*Book
}

func NewLibrary() *Library {
	return &Library{
		books: make([]*Book, 0),
	}
}

func (l *Library) AddBook(book *Book) {
	l.books = append(l.books, book)
	fmt.Printf("📚 Livro adicionado: %s\n", book.Title)
}

func (l *Library) CreateIterator() BookIterator {
	return NewNormalIterator(l.books)
}

func (l *Library) CreateReverseIterator() BookIterator {
	return NewReverseIterator(l.books)
}

func (l *Library) CreateGenreIterator(genre string) BookIterator {
	return NewGenreIterator(l.books, genre)
}

func (l *Library) Size() int {
	return len(l.books)
}

// LibraryReader - cliente que usa iterators
type LibraryReader struct{}

func NewLibraryReader() *LibraryReader {
	return &LibraryReader{}
}

func (r *LibraryReader) ReadAllBooks(library *Library) {
	fmt.Println("\n=== Lendo todos os livros (ordem normal) ===")
	iterator := library.CreateIterator()
	count := 1
	
	for iterator.HasNext() {
		book := iterator.Next()
		fmt.Printf("%d. %s\n", count, book)
		count++
	}
}

func (r *LibraryReader) ReadBooksReverse(library *Library) {
	fmt.Println("\n=== Lendo livros (ordem reversa) ===")
	iterator := library.CreateReverseIterator()
	count := 1
	
	for iterator.HasNext() {
		book := iterator.Next()
		fmt.Printf("%d. %s\n", count, book)
		count++
	}
}

func (r *LibraryReader) ReadBooksByGenre(library *Library, genre string) {
	fmt.Printf("\n=== Lendo livros de %s ===\n", genre)
	iterator := library.CreateGenreIterator(genre)
	count := 1
	
	for iterator.HasNext() {
		book := iterator.Next()
		fmt.Printf("%d. %s\n", count, book)
		count++
	}
}

func (r *LibraryReader) DemonstrateMultipleIterators(library *Library) {
	fmt.Println("\n=== Múltiplos iterators simultâneos ===")
	
	iter1 := library.CreateIterator()
	iter2 := library.CreateIterator()
	
	fmt.Printf("Iterator 1 (primeiro livro): %s\n", iter1.Next().Title)
	fmt.Printf("Iterator 2 (primeiro livro): %s\n", iter2.Next().Title)
	fmt.Printf("Iterator 1 (segundo livro): %s\n", iter1.Next().Title)
	fmt.Printf("Iterator 2 (segundo livro): %s\n", iter2.Next().Title)
	
	fmt.Println("\n✅ Cada iterator mantém seu próprio estado!")
}

// Uso
func main() {
	library := NewLibrary()
	
	// Adiciona livros
	fmt.Println("=== Montando biblioteca ===")
	library.AddBook(NewBook("1984", "George Orwell", "Ficção", 1949))
	library.AddBook(NewBook("O Senhor dos Anéis", "J.R.R. Tolkien", "Fantasia", 1954))
	library.AddBook(NewBook("Harry Potter", "J.K. Rowling", "Fantasia", 1997))
	library.AddBook(NewBook("Clean Code", "Robert Martin", "Técnico", 2008))
	library.AddBook(NewBook("O Hobbit", "J.R.R. Tolkien", "Fantasia", 1937))
	
	fmt.Printf("\n📖 Total de livros: %d\n", library.Size())
	
	// Usa diferentes iterators
	reader := NewLibraryReader()
	
	reader.ReadAllBooks(library)
	reader.ReadBooksReverse(library)
	reader.ReadBooksByGenre(library, "Fantasia")
	reader.DemonstrateMultipleIterators(library)
}
```

## Exemplo 2: Árvore com Diferentes Estratégias de Travessia

```go
package main

import "fmt"

// TreeNode - nó da árvore
type TreeNode struct {
	Data  int
	Left  *TreeNode
	Right *TreeNode
}

func NewTreeNode(data int) *TreeNode {
	return &TreeNode{Data: data}
}

// TreeIterator - interface Iterator
type TreeIterator interface {
	HasNext() bool
	Next() int
}

// Tree - interface Collection
type Tree interface {
	CreateInOrderIterator() TreeIterator
	CreatePreOrderIterator() TreeIterator
	CreateBreadthFirstIterator() TreeIterator
}

// InOrderIterator - iterator in-order (esquerda, raiz, direita)
type InOrderIterator struct {
	stack []*TreeNode
}

func NewInOrderIterator(root *TreeNode) *InOrderIterator {
	it := &InOrderIterator{stack: make([]*TreeNode, 0)}
	it.pushLeftChildren(root)
	return it
}

func (it *InOrderIterator) pushLeftChildren(node *TreeNode) {
	for node != nil {
		it.stack = append(it.stack, node)
		node = node.Left
	}
}

func (it *InOrderIterator) HasNext() bool {
	return len(it.stack) > 0
}

func (it *InOrderIterator) Next() int {
	if !it.HasNext() {
		panic("Não há mais elementos")
	}
	
	// Pop do stack
	lastIndex := len(it.stack) - 1
	node := it.stack[lastIndex]
	it.stack = it.stack[:lastIndex]
	
	data := node.Data
	
	if node.Right != nil {
		it.pushLeftChildren(node.Right)
	}
	
	return data
}

// PreOrderIterator - iterator pre-order (raiz, esquerda, direita)
type PreOrderIterator struct {
	stack []*TreeNode
}

func NewPreOrderIterator(root *TreeNode) *PreOrderIterator {
	stack := make([]*TreeNode, 0)
	if root != nil {
		stack = append(stack, root)
	}
	return &PreOrderIterator{stack: stack}
}

func (it *PreOrderIterator) HasNext() bool {
	return len(it.stack) > 0
}

func (it *PreOrderIterator) Next() int {
	if !it.HasNext() {
		panic("Não há mais elementos")
	}
	
	// Pop do stack
	lastIndex := len(it.stack) - 1
	node := it.stack[lastIndex]
	it.stack = it.stack[:lastIndex]
	
	// Push direita primeiro (será processada depois)
	if node.Right != nil {
		it.stack = append(it.stack, node.Right)
	}
	if node.Left != nil {
		it.stack = append(it.stack, node.Left)
	}
	
	return node.Data
}

// BreadthFirstIterator - iterator breadth-first (por nível)
type BreadthFirstIterator struct {
	queue []*TreeNode
}

func NewBreadthFirstIterator(root *TreeNode) *BreadthFirstIterator {
	queue := make([]*TreeNode, 0)
	if root != nil {
		queue = append(queue, root)
	}
	return &BreadthFirstIterator{queue: queue}
}

func (it *BreadthFirstIterator) HasNext() bool {
	return len(it.queue) > 0
}

func (it *BreadthFirstIterator) Next() int {
	if !it.HasNext() {
		panic("Não há mais elementos")
	}
	
	// Dequeue
	node := it.queue[0]
	it.queue = it.queue[1:]
	
	if node.Left != nil {
		it.queue = append(it.queue, node.Left)
	}
	if node.Right != nil {
		it.queue = append(it.queue, node.Right)
	}
	
	return node.Data
}

// BinaryTree - árvore binária concreta
type BinaryTree struct {
	root *TreeNode
}

func NewBinaryTree() *BinaryTree {
	return &BinaryTree{}
}

func (t *BinaryTree) SetRoot(root *TreeNode) {
	t.root = root
}

func (t *BinaryTree) GetRoot() *TreeNode {
	return t.root
}

func (t *BinaryTree) CreateInOrderIterator() TreeIterator {
	return NewInOrderIterator(t.root)
}

func (t *BinaryTree) CreatePreOrderIterator() TreeIterator {
	return NewPreOrderIterator(t.root)
}

func (t *BinaryTree) CreateBreadthFirstIterator() TreeIterator {
	return NewBreadthFirstIterator(t.root)
}

// TreeTraverser - cliente
type TreeTraverser struct{}

func NewTreeTraverser() *TreeTraverser {
	return &TreeTraverser{}
}

func (t *TreeTraverser) Traverse(traversalType string, iterator TreeIterator) {
	fmt.Printf("\n=== Travessia %s ===\n", traversalType)
	fmt.Print("Elementos: ")
	
	for iterator.HasNext() {
		fmt.Printf("%d ", iterator.Next())
	}
	fmt.Println()
}

// Uso
func main() {
	// Cria árvore:
	//        1
	//       / \
	//      2   3
	//     / \   \
	//    4   5   6
	
	tree := NewBinaryTree()
	
	root := NewTreeNode(1)
	node2 := NewTreeNode(2)
	node3 := NewTreeNode(3)
	node4 := NewTreeNode(4)
	node5 := NewTreeNode(5)
	node6 := NewTreeNode(6)
	
	root.Left = node2
	root.Right = node3
	node2.Left = node4
	node2.Right = node5
	node3.Right = node6
	
	tree.SetRoot(root)
	
	fmt.Println("=== Árvore Binária ===")
	fmt.Println("       1")
	fmt.Println("      / \\")
	fmt.Println("     2   3")
	fmt.Println("    / \\   \\")
	fmt.Println("   4   5   6")
	
	traverser := NewTreeTraverser()
	
	// Diferentes travessias
	traverser.Traverse("In-Order (ESQ-RAIZ-DIR)", 
		tree.CreateInOrderIterator())
	
	traverser.Traverse("Pre-Order (RAIZ-ESQ-DIR)", 
		tree.CreatePreOrderIterator())
	
	traverser.Traverse("Breadth-First (Por Nível)", 
		tree.CreateBreadthFirstIterator())
	
	// Demonstra múltiplos iterators
	fmt.Println("\n=== Múltiplos Iterators Simultâneos ===")
	iter1 := tree.CreateInOrderIterator()
	iter2 := tree.CreatePreOrderIterator()
	
	fmt.Printf("In-Order primeiro: %d\n", iter1.Next())
	fmt.Printf("Pre-Order primeiro: %d\n", iter2.Next())
	fmt.Printf("In-Order segundo: %d\n", iter1.Next())
	fmt.Printf("Pre-Order segundo: %d\n", iter2.Next())
}
```
