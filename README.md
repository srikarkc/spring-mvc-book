Below is a complete example of a Spring MVC application that manages a list of books. This includes domain objects, repositories, services, controllers, views, and the explanation of the Data Transfer Object (DTO).

### Domain Object

```java
// Domain object representing a Book
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String author;
    private String isbn;

    // Getters and setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }
}
```

### Repository Layer

```java
// Repository interface for accessing Book data
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Long> {
}
```

### Service Layer

```java
// Service class for Book-related business logic
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

    public List<Book> findAll() {
        return bookRepository.findAll();
    }

    public Book findById(Long id) {
        return bookRepository.findById(id).orElse(null);
    }

    public Book save(Book book) {
        return bookRepository.save(book);
    }

    public void deleteById(Long id) {
        bookRepository.deleteById(id);
    }
}
```

### Controller

```java
// Controller class for handling HTTP requests related to books
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ModelAttribute;

@Controller
public class BookController {

    @Autowired
    private BookService bookService;

    @GetMapping("/books")
    public String listBooks(Model model) {
        model.addAttribute("books", bookService.findAll());
        return "bookList";
    }

    @GetMapping("/books/new")
    public String newBookForm(Model model) {
        model.addAttribute("book", new Book());
        return "bookForm";
    }

    @PostMapping("/books")
    public String saveBook(@ModelAttribute Book book) {
        bookService.save(book);
        return "redirect:/books";
    }

    @GetMapping("/books/delete")
    public String deleteBook(@RequestParam Long id) {
        bookService.deleteById(id);
        return "redirect:/books";
    }
}
```

### View (Thymeleaf)

#### bookList.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Book List</title>
</head>
<body>
    <h1>Books</h1>
    <ul>
        <li th:each="book : ${books}">
            <span th:text="${book.title}">Title</span> by <span th:text="${book.author}">Author</span>
        </li>
    </ul>
    <a href="/books/new">Add a new book</a>
</body>
</html>
```

#### bookForm.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Add New Book</title>
</head>
<body>
    <h1>Add New Book</h1>
    <form action="/books" method="post" th:object="${book}">
        <label for="title">Title:</label>
        <input type="text" id="title" th:field="*{title}" />
        <br/>
        <label for="author">Author:</label>
        <input type="text" id="author" th:field="*{author}" />
        <br/>
        <label for="isbn">ISBN:</label>
        <input type="text" id="isbn" th:field="*{isbn}" />
        <br/>
        <button type="submit">Save</button>
    </form>
    <a href="/books">Back to book list</a>
</body>
</html>
```

### Data Transfer Object (DTO)

In this example, a separate DTO is not explicitly used because the `Book` entity directly serves as a form-backing object. However, in more complex scenarios, you might want to use a DTO to decouple the domain model from the data transfer layer.

#### Example DTO

```java
// Data Transfer Object for transferring Book data
public class BookDTO {

    private Long id;
    private String title;
    private String author;
    private String isbn;

    // Getters and setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }
}
```

### Explanation of DTO

A **Data Transfer Object (DTO)** is used to encapsulate data and transfer it between different layers of an application. DTOs are particularly useful in the following scenarios:

1. **Decoupling**: Separating the domain model from the data transfer logic. This prevents exposing the internal structure of domain entities directly to the client.

2. **Security**: Limiting the data exposed to the client, ensuring sensitive information is not transferred.

3. **Performance**: Optimizing data transfer by aggregating data from multiple sources or reducing the data size.

In the controller, you can use a DTO to handle form data:

```java
@PostMapping("/books")
public String saveBook(@ModelAttribute BookDTO bookDTO) {
    // Convert DTO to entity
    Book book = new Book();
    book.setTitle(bookDTO.getTitle());
    book.setAuthor(bookDTO.getAuthor());
    book.setIsbn(bookDTO.getIsbn());
    
    bookService.save(book);
    return "redirect:/books";
}
```

By using DTOs, you ensure that only the necessary data is transferred between the client and server, enhancing security and performance.