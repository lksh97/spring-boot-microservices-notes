# Spring JDBC aur JPA: Complete Guide for Beginners

Namaskar dosto! Aaj hum Spring Framework ke do important data access approaches - Spring JDBC aur Spring JPA ke bare mein detail mein samjhenge. Ye guide absolute beginners ke liye hai aur interview preparation mein bhi help karegi.

## Introduction - Database Connectivity Kya Hai?

Database connectivity modern application ka backbone hai. Spring Framework do powerful tarike provide karta hai database ke saath communicate karne ke liye:

```
+----------------+        +-----------------+
|                |        |                 |
| Application    |        | Database        |
| (Java Code)    |<------>| (MySQL/Oracle)  |
|                |        |                 |
+----------------+        +-----------------+
        ^
        |
        v
+--------------------------------+
|                                |
| Data Access Layer             |
| (JDBC or JPA)                 |
|                                |
+--------------------------------+
```

## Spring JDBC Kya Hai?

Spring JDBC ek lightweight data access technique hai jo traditional JDBC ko simplify karta hai. Is approach mein:

- Raw SQL queries likhi jati hain
- JdbcTemplate class SQL execution manage karta hai
- Boilerplate code (connection handling, error handling) kam ho jata hai
- Developer ko result mapping manage karna padta hai

### JDBC Components

1. **JdbcTemplate**: Ye core class hai jo queries execute karti hai
   ```java
   // Yahan jdbcTemplate "SELECT COUNT(*) FROM products" query execute karta hai
   int count = jdbcTemplate.queryForObject("SELECT COUNT(*) FROM products", Integer.class);
   ```

2. **RowMapper**: ResultSet se Java objects banane ke liye interface
   ```java
   // Ye mapper database row ko Product object mein convert karta hai
   public class ProductMapper implements RowMapper<Product> {
       @Override
       public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
           Product product = new Product();
           product.setId(rs.getInt("id"));           // id column ko product id mein set karo
           product.setTitle(rs.getString("title"));  // title column ko product title mein set karo
           product.setPrice(rs.getInt("price"));     // price column ko product price mein set karo
           return product;
       }
   }
   ```

3. **DataSource**: Connection pool manage karta hai
   ```properties
   # Database settings application.properties mein
   spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
   spring.datasource.username=root
   spring.datasource.password=root1234
   ```

### Spring JDBC Implementation Kaise Karein?

#### Step 1: Dependencies Add Karna

```xml
<!-- pom.xml mein ye dependencies add karo -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### Step 2: Model Classes Banana

```java
// Product.java - ye database table ka representation hai
public class Product {
    private int id;          // product ki unique id
    private String title;    // product ka naam
    private String description; // product ka description
    private int price;       // product ki price
    private int catId;       // category ki foreign key
    
    // Getters and setters...
}
```
---

## 🔍 What does `@ManyToOne` signify?

### 🔹 **JPA mein** jab hum likhte hain `@ManyToOne`, iska matlab hota hai:

> "Many objects of this entity (Product) are linked to **one object** of another entity (Category)."

Yaani:

- **Bohot saare products** ek **category** ke andar ho sakte hain.  
- Lekin **ek product** sirf **ek hi category** se belong karega.

---

## 🧠 Real-Life Example:

- 🧴 "Shampoo", 🪒 "Razor", 🧼 "Soap" → Category: "Personal Care"
- 📱 "iPhone", 📱 "Samsung" → Category: "Mobile"

Yaha:
- Multiple products → 1 category ⇒ **ManyToOne**

---

## 📘 Java Class Example Recap:

```java
@Entity
public class Product {
    @Id
    private int productId;

    private String title;

    @ManyToOne
    private Category category;
}
```

---

## 🔗 Text Diagram:

```text
PRODUCT Table                       CATEGORY Table
------------------                 -------------------
| product_id     |                | category_id      |
| title          |                | name             |
| category_id FK |◀────────────── | (Primary Key)    |
------------------                 -------------------
     ↑
     |  Many Products
     |
     └────────► One Category
```

---

## 🧾 Summary:

| Concept        | Meaning |
|----------------|---------|
| `@ManyToOne`   | Many rows in Product table point to one row in Category table |
| Result         | Product table gets a foreign key column (category_id) |
| Direction      | Many (Product) → One (Category) |
| DB Relationship | Foreign key in `products` table referencing `category_id` in `category` table |

---

## ✅ Interview Explanation:

> "`@ManyToOne` relationship in JPA means many instances of the current entity (Product) are associated with one instance of another entity (Category). It adds a foreign key in the Product table pointing to the Category table."

---

#### Step 3: DAO Interface Banana

```java
// ProductDao.java - database operations define karta hai
public interface ProductDao {
    Product create(Product product);    // naya product add karta hai
    Product update(Product product, int productId);  // existing product update karta hai
    void delete(int productId);  // product delete karta hai
    List<Product> getAll();  // saare products fetch karta hai
    Product get(int productId);  // specific product fetch karta hai
}
```

#### Step 4: DAO Implementation with JdbcTemplate

```java
// ProductDaoImpl.java - SQL queries execute karta hai
@Repository
public class ProductDaoImpl implements ProductDao {
    
    private JdbcTemplate jdbcTemplate;  // database operations ke liye template
    
    // Constructor injection through Spring
    public ProductDaoImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        // Table create karo agar exist nahi karti
        String createQuery = "CREATE TABLE IF NOT EXISTS products(id int primary key, title varchar(200) not null)";
        jdbcTemplate.update(createQuery);  // DDL query execute karo
    }
    
    @Override
    public Product create(Product product) {
        // Insert query with prepared statement
        String query = "insert into products(id,title,description,price,cat_id) values(?,?,?,?,?)";
        int rowsUpdated = jdbcTemplate.update(
            query,  // SQL query with placeholders
            product.getId(),  // first ? ke liye value
            product.getTitle(),  // second ? ke liye value
            product.getDescription(),  // third ? ke liye value
            product.getPrice(),  // fourth ? ke liye value
            product.getCatId()   // fifth ? ke liye value
        );
        System.out.println(rowsUpdated + " rows affected");  // kitne rows update hue
        return product;  // created product return karo
    }
    
    @Override
    public List<Product> getAll() {
        // Select query with RowMapper
        String query = "select * from products";  // sare products fetch karo
        List<Product> products = jdbcTemplate.query(query, new ProductMapper());  // RowMapper se result map karo
        return products;  // list of products return karo
    }
    
    // Other methods...
}
```

## Spring JPA Kya Hai?

Spring JPA (Java Persistence API) ek higher-level abstraction hai jo:

- Object-Relational Mapping (ORM) use karta hai
- SQL queries ko mostly hide karta hai
- Annotations ka use karke database structure define karta hai
- Entities aur relationships ke concept pe based hai

### JPA Components

1. **Entity Classes**: Database tables ko represent karte hain
   ```java
   @Entity  // Ye annotation batata hai ki ye class ek database table hai
   @Table(name = "jpa_products")  // Table ka actual name
   public class Product {
       @Id  // Primary key mark karta hai
       @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment ID
       private int productId;
       
       @Column(name = "product_title", unique = true)  // Column properties
       private String title;
       
       // Fields and relationships...
       @ManyToOne  // Relationship annotation
       private Category category;  // Another entity reference
   }
   ```

2. **Repository Interfaces**: Pre-defined CRUD operations provide karte hain
   ```java
   // Sirf interface define karo, implementation automatic generate hota hai
   public interface ProductRepository extends JpaRepository<Product, Integer> {
       // Custom finder methods jo naming convention se queries generate karte hain
       List<Product> findByTitleContaining(String keyword);
       
       // Custom query with JPQL (Java Persistence Query Language)
       @Query("select p from Product p where p.price > :minPrice")
       List<Product> findExpensiveProducts(@Param("minPrice") double minPrice);
   }
   ```

### Spring JPA Implementation Kaise Karein?

#### Step 1: Dependencies Add Karna

```xml
<!-- pom.xml mein ye dependencies add karo -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### Step 2: Entity Classes Banana

```java
// Annotations se database structure define karo
@Entity
@Table(name = "jpa_category")
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment ID
    private int id;
    
    private String title;  // By default column name same as field name
    
    // One-to-Many relationship - ek category ke multiple products ho sakte hain
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<Product> productList;
    
    // Getters and setters...
}

@Entity
@Table(name = "jpa_products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int productId;
    
    @Column(name = "product_title", unique = true, nullable = false)
    private String title;
    
    private String description;
    private double price;
    
    // Many-to-One relationship - multiple products ek hi category ke ho sakte hain
    @ManyToOne
    private Category category;
    
    // Getters and setters...
}
```

Bahut badhiya sawaal Vaibhav! Ye doubt **real-world developers** ko bhi hota hai.  
Chalo simple Hinglish mein samajhte hain:  
**Agar DB mein hi `unique` aur `not null` constraints diye hain, toh `@Column(nullable = false, unique = true)` ka kya kaam hai application code mein?**

---

## 🔍 Jab tum likhte ho:

```java
@Column(name = "product_title", unique = true, nullable = false)
private String title;
```

Ye kaam **application level pe** hota hai — aur isse:

### ✅ Hibernate / JPA:
1. **Database schema generate** karta hai (`ddl-auto=create/update`) tab ye annotations use karta hai.
2. Entity validation/metadata ke liye bhi inhe consider karta hai.

---

## 🔧 Breakdown:

| Property        | Role in App Code (`@Column`)            | Role in Database              |
|----------------|------------------------------------------|-------------------------------|
| `nullable=false` | Hibernate ko batata hai ki column `null` accept nahi karega | DB constraint: `NOT NULL`     |
| `unique=true`   | Hibernate ko batata hai ki values unique honi chahiye | DB constraint: `UNIQUE`       |

---

## 🎯 Scenario-wise Explanation:

### 1️⃣ **If Hibernate is generating your DB (ddl-auto = create/update)**  
Then `@Column(nullable = false, unique = true)` is used to:
- Generate SQL:  
  ```sql
  product_title VARCHAR(255) NOT NULL UNIQUE
  ```

✅ Very useful in new projects or auto-migrations.

---

### 2️⃣ **If DB schema is already defined and you're using ddl-auto = none**  
Then:
- Ye annotations **DDL generation ke liye nahi** use honge
- Lekin ye annotations:
  - Entity ka **metadata** define karte hain
  - Help karte hain **frameworks** jaise Swagger, JSR-303, Hibernate Validator etc. ko batane ke liye kya rules hain

---

## 🧠 Application-level benefits of `nullable` and `unique`:

| Annotation    | App Code Benefit |
|---------------|------------------|
| `nullable=false` | Tools like Spring Validator, Hibernate Validator use it to reject null inputs before hitting DB |
| `unique=true`   | Helpful for code generators, ORM mappers, and tools like Swagger/OpenAPI to auto-document the constraint |

---

### ✅ Best Practice:

- 🔒 **Critical validation (e.g. `NOT NULL`, `UNIQUE`)** hamesha **DB level par enforce** karo.
- 🧠 **Application level pe bhi annotate karo** taaki:
  - Code self-documented rahe
  - ORM tools aur APIs accurate metadata de
  - Extra safety mile during schema generation (if needed)

---

### ✅ Bonus (Validation in code):

```java
@NotBlank
@Column(nullable = false, unique = true)
private String title;
```

> Ye combo ensures:
- Not blank at API level (JSR-303 validation)
- Not null and unique at DB level
- Self-documented entity

---

## 🔚 Final Verdict:

> `@Column(nullable = false, unique = true)` in application code is **not useless**, even if the DB has those constraints.  
> It helps with **ORM schema generation**, **validation frameworks**, **code documentation**, and **self-explanatory models** ✅

---

Chaho toh main tumhare liye ek **test case likh sakta hoon** jisme ye validation fail karta hai before reaching the DB. Batau?

#### Step 3: Repository Interfaces Banana

```java
// Sirf interface define karo - implementation Spring automatically provide karega
public interface CategoryRepository extends JpaRepository<Category, Integer> {
    // Empty bhi chal jayega - basic CRUD methods automatically available hain
}

public interface ProductRepository extends JpaRepository<Product, Integer> {
    // Custom finder methods
    List<Product> findByTitleContaining(String keyword);  // SQL: WHERE title LIKE '%keyword%'
    List<Product> findByPrice(double price);              // SQL: WHERE price = ?
    
    // Complex query with JPQL
    @Query("select p from Product p WHERE p.title =:title and p.price =:price")
    List<Product> getProductByTitle(@Param("title") String title, @Param("price") double price);
    
    // Join query with JPQL
    @Query("select p from Product p JOIN fetch p.category where p.category.title =:catTitle")
    List<Product> getProductByCategoryTitle(@Param("catTitle") String title);
}
```

## JDBC vs JPA: Comparison

| Feature | Spring JDBC | Spring JPA |
|---------|------------|-----------|
| Control | Direct SQL control | Object-centric approach |
| Code Amount | More code to write | Less boilerplate code |
| Learning | Simple to start | Complex concepts |
| Use Case | Simple operations, performance critical | Complex domain models |

```
+---------------------+            +---------------------+
|                     |            |                     |
|    Spring JDBC      |            |    Spring JPA       |
|                     |            |                     |
+---------------------+            +---------------------+
| - SQL statements    |            | - Entity mapping    |
| - JdbcTemplate      |            | - Repositories      |
| - RowMapper         |            | - JPQL              |
| - Manual mapping    |            | - Automatic mapping |
+---------------------+            +---------------------+
```

## spring-jdbc-ecom Project Analysis

Aapka project `spring-jdbc-ecom` ek simple Spring JDBC implementation hai jisme:

1. **Model Classes**:
   - `Product.java` - Products ke data ko represent karta hai
   - `Category.java` - Categories ke data ko represent karta hai
   - `ProductWithCategory.java` - Join query results ko represent karta hai

2. **DAO Layer**:
   - `ProductDao.java` - Interface jo operations define karta hai
   - `ProductDaoImpl.java` - Implementation jo SQL queries execute karta hai
   - `CategoryDao.java` - Category operations ke liye interface
   - `CategoryDaoImpl.java` - Category queries execute karta hai

3. **Helper Classes**:
   - `ProductMapper.java` - ResultSet ko Product objects mein convert karta hai

### Code Walkthrough with Hinglish Comments

ProductDaoImpl.java se ek important method ka analysis:

```java
@Override
public Product create(Product product) {
    // SQL insert query with placeholders
    String query = "insert into products(id,title,description,price,cat_id) values(?,?,?,?,?)";
    
    // jdbcTemplate.update() se insert query execute karo
    int update = jdbcTemplate.update(
            query,  // query string
            product.getId(),  // id ki value placeholder ke liye
            product.getTitle(),  // title ki value
            product.getDescription(),  // description ki value
            product.getPrice(),  // price ki value
            product.getCatId()  // category id ki value
    );
    
    // Log kitne rows update hue
    System.out.println(update + " rows affected");
    
    // Created product return karo
    return product;
}
```

Join query ka example:

```java
@Override
public List<ProductWithCategory> getAllWithCategory() {
    // Join query products aur categories ke beech
    String query = "Select p.id as id, p.title as title, p.description as description, p.price as price, c.title as catTitle FROM products p INNER JOIN categories c ON p.cat_id=c.id";
    
    // Lambda expression se inline RowMapper
    return jdbcTemplate.query(query, (rs, rowNum) -> {
        // Har row ke liye ek naya ProductWithCategory object banao
        ProductWithCategory productWithCategory = new ProductWithCategory();
        productWithCategory.setId(rs.getInt("id"));  // id column ki value set karo
        productWithCategory.setTitle(rs.getString("title"));  // title set karo
        productWithCategory.setDescription(rs.getString("description"));  // description set karo
        productWithCategory.setCatTitle(rs.getString("catTitle"));  // category title set karo
        return productWithCategory;  // Populated object return karo
    });
}
```

## Interview Q&A (Hinglish mein)

### Q1: Spring JDBC aur traditional JDBC mein kya difference hai?

**A:** Spring JDBC traditional JDBC ko simplify karta hai:

1. **Exception Handling**: 
   - Traditional JDBC: Har jagah try-catch blocks lagane padte hain
   - Spring JDBC: Checked exceptions ko unchecked mein convert kar deta hai

Bilkul Vaibhav! Ye line **Spring JDBC ke sabse underrated but powerful feature** ko describe karti hai. Chalo isko **zero se samjhte hain**:

---

## ✅ Line:
> **"Spring JDBC converts checked exceptions into unchecked exceptions."**

---

## 🔍 Pehle samjho: Checked vs Unchecked Exception

| Type               | Example                        | Force try-catch? |
|--------------------|--------------------------------|------------------|
| ✅ Checked         | `SQLException`, `IOException`  | Yes              |
| ❌ Unchecked       | `NullPointerException`, `RuntimeException` | No               |

---

### ❗ Problem in Core JDBC:

```java
Connection con = DriverManager.getConnection(...);
Statement stmt = con.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM product");
```

**Har JDBC call pe `SQLException` throw ho sakta hai.**  
So we are **forced to wrap everything in try-catch**:

```java
try {
   // JDBC logic
} catch (SQLException e) {
   e.printStackTrace();
}
```

👎 **Boilerplate code + messy exception handling**

---

## 🌟 Spring JDBC ne kya kiya?

### ✅ Spring JDBC ne kaha:

> “Main tumhare liye sab try-catch handle karunga. Agar exception aaya, toh main usse `RuntimeException` mein wrap karke throw kar dunga.”

### ✅ Example:
```java
jdbcTemplate.queryForObject("SELECT * FROM product WHERE id = ?", new ProductMapper(), id);
```

Isme:
- Agar product nahi mila ⇒ `EmptyResultDataAccessException` (runtime)
- Agar query galat hai ⇒ `BadSqlGrammarException` (runtime)
- DB access issue ⇒ `DataAccessException` (runtime)

---

## 🔧 Utility (Faayda) in Real-World Code:

| Benefit                              | Explanation |
|--------------------------------------|-------------|
| ✅ Less boilerplate                  | No more `try-catch(SQLException)` every time |
| ✅ Cleaner code                      | Focus on logic, not exception plumbing |
| ✅ Common super type (`DataAccessException`) | Easily handled in global exception handler |
| ✅ Easy Integration with Spring AOP  | Can catch all DB errors uniformly |
| ✅ Runtime = Optional Handling       | You can choose to catch or not |

---

### 💡 Text Diagram:

```text
Core JDBC:
  SQLException (checked) → must handle

Spring JDBC:
  SQLException → DataAccessException (runtime)
```

---

## 📦 Example: Global Handler

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<String> handleDbError(DataAccessException ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                             .body("Database error: " + ex.getMessage());
    }
}
```

📌 Tum sab DB exceptions ko **ek hi place pe** handle kar sakte ho. That's the **real power** of converting checked → unchecked.

---

## ✅ Interview Me Kya Bolna Hai?

> “Spring JDBC simplifies error handling by converting all `SQLException` (checked) into `DataAccessException` (unchecked).  
This removes the need for boilerplate `try-catch` code and makes it easier to handle DB errors centrally in modern Spring apps.”

---


2. **Resource Cleanup**:
   - Traditional JDBC: Manually connection, statement, resultset close karna padta hai
   - Spring JDBC: Automatically resource cleanup kar deta hai

3. **Boilerplate Code**:
   - Traditional JDBC: Bahut repetitive code likhna padta hai
   - Spring JDBC: JdbcTemplate sab handle kar deta hai

### Q2: JdbcTemplate ke main methods kya hain?

**A:** JdbcTemplate ke important methods hain:

1. **update()**: INSERT, UPDATE, DELETE operations ke liye
   ```java
   // 5 rows insert karo
   int rowsAffected = jdbcTemplate.update("INSERT INTO products VALUES(?, ?)", 101, "Phone");
   ```

2. **queryForObject()**: Single row result ke liye
   ```java
   // Ek specific product fetch karo
   Product product = jdbcTemplate.queryForObject("SELECT * FROM products WHERE id=?", 
                                               new ProductMapper(), 101);
   ```

3. **query()**: Multiple rows result ke liye
   ```java
   // Sare products fetch karo
   List<Product> products = jdbcTemplate.query("SELECT * FROM products", new ProductMapper());
   ```

### Q3: JPA mein @Entity annotation ka kya use hai?

**A:** `@Entity` annotation Java class ko database table se map karta hai:

1. Class ko database table ke roop mein mark karta hai
2. JPA provider (Hibernate) ko batata hai ki isko database operations mein use karna hai
3. Is class ke instances table rows ko represent karenge
4. `@Table` annotation ke saath specific table name specify kar sakte hain

### Q4: JPA mein relationships kaise define karte hain?

**A:** JPA mein 4 types ke relationships ho sakte hain:

1. **@OneToOne** - Example: Ek User ka ek Address ho sakta hai
   ```java
   @OneToOne
   @JoinColumn(name = "address_id")
   private Address address;  // User class mein ye field address table se connect hoga
   ```

2. **@OneToMany / @ManyToOne** - Example: Ek Category ke multiple Products
   ```java
   // Category class mein
   @OneToMany(mappedBy = "category")
   private List<Product> products;  // Category ke saare products
   
   // Product class mein
   @ManyToOne
   private Category category;  // Product ki category
   ```

3. **@ManyToMany** - Example: Products aur Tags
   ```java
   @ManyToMany
   @JoinTable(name = "product_tag")  // Junction table
   private Set<Tag> tags;  // Product ke multiple tags
   ```

### Q5: Spring Data JPA mein custom queries kaise likhte hain?

**A:** Multiple ways hain:

1. **Method names se**:
   ```java
   List<Product> findByTitleContaining(String keyword);  // WHERE title LIKE '%keyword%'
   List<Product> findByPriceGreaterThan(double price);   // WHERE price > ?
   ```

2. **@Query annotation se**:
   ```java
   @Query("select p from Product p where p.price between :min and :max")
   List<Product> findByPriceRange(@Param("min") double min, @Param("max") double max);
   ```

3. **Native SQL queries**:
   ```java
   @Query(value = "SELECT * FROM products ORDER BY price DESC LIMIT 5", nativeQuery = true)
   List<Product> findTopExpensiveProducts();
   ```

### Q6: JDBC aur JPA kab use karna chahiye?

**A:** Choice depends on requirements:

**JDBC use karo jab**:
- Direct SQL control chahiye
- Simple operations karne hain
- SQL optimization critical hai
- Legacy database structure hai

**JPA use karo jab**:
- Boilerplate code kam karna hai
- Complex object relationships hain
- Database vendor independence chahiye 
- Advanced features (auditing, caching) chahiye

## Conclusion

Dosto, aaj humne Spring JDBC aur JPA ke baare mein detail mein jaana. Dono approaches ke apne advantages hain aur use case ke hisab se inko choose karna chahiye.

- **JDBC**: More control, direct SQL queries, simple approach
- **JPA**: Less code, object-oriented, powerful relationships

Interview mein, aap dono ke concepts clear rakho aur examples de paao. Best of luck!

Is there any specific part you'd like me to elaborate on further?


-------------------------


# DETAILED



-------------------------


# Spring JDBC aur JPA: Complete Guide for Beginners

## Table of Contents

1. [Introduction](#introduction)
2. [Spring JDBC](#spring-jdbc)
   - [Spring JDBC Architecture](#spring-jdbc-architecture)
   - [Spring JDBC Components](#spring-jdbc-components)
   - [JDBC Implementation Steps](#jdbc-implementation-steps)
3. [Spring JPA](#spring-jpa)
   - [JPA Architecture](#jpa-architecture)
   - [JPA Components](#jpa-components)
   - [JPA Implementation Steps](#jpa-implementation-steps)
4. [JDBC vs JPA: Comparison](#jdbc-vs-jpa-comparison)
5. [Spring JDBC Ecom Project Walkthrough](#spring-jdbc-ecom-project-walkthrough)
6. [Spring JPA Ecom Project Comparison](#spring-jpa-ecom-project-comparison)
7. [Interview Questions & Answers](#interview-questions--answers)

---

## Introduction

Database connectivity aur data access ek modern application development ka sabse important aspect hai. Spring Framework do powerful approaches provide karta hai database operations ke liye:

1. **Spring JDBC (Java Database Connectivity)**
2. **Spring JPA (Java Persistence API)**

Ye dono approaches alag-alag advantages aur use cases rakhte hain. Is guide mein, hum dono ko deep dive karenge aur ek ecommerce application project ke through inko samjhenge.

```
+----------------+        +-----------------+
|                |        |                 |
| Application    |        | Database        |
| (Java Code)    |<------>| (MySQL/Oracle)  |
|                |        |                 |
+----------------+        +-----------------+
        ^
        |
        v
+--------------------------------+
|                                |
| Data Access Layer             |
| (JDBC or JPA)                 |
|                                |
+--------------------------------+
```

## Spring JDBC

Spring JDBC traditional JDBC operations ko simplify karta hai. Ye boilerplate code ko kam karta hai aur database operations ko straightforward banata hai.

### Spring JDBC Architecture

Spring JDBC five package provide karta hai database interaction ke liye:

1. **org.springframework.jdbc.core** - Core functionality with JdbcTemplate
2. **org.springframework.jdbc.datasource** - DataSource utility classes
3. **org.springframework.jdbc.object** - SQL operations as thread-safe objects
4. **org.springframework.jdbc.support** - Exception handling
5. **org.springframework.jdbc.config** - JDBC configuration support

```
+------------------------+
|                        |
|  JdbcTemplate          |
|                        |
+------------------------+
         |
         v
+------------------------+
|                        |
|  Connection Pool       |
|  (DataSource)          |
|                        |
+------------------------+
         |
         v
+------------------------+
|                        |
|  Database              |
|                        |
+------------------------+
```

### Spring JDBC Components

#### 1. JdbcTemplate

JdbcTemplate Spring JDBC ka central class hai. Ye SQL queries execute karne, result sets ko process karne, aur exceptions ko handle karne ka kaam karta hai.

```java
// JdbcTemplate ka basic usage example
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
String sql = "SELECT COUNT(*) FROM products";
int count = jdbcTemplate.queryForObject(sql, Integer.class);
```

#### 2. RowMapper

RowMapper interface ResultSet ke rows ko Java objects mein convert karta hai.

```java
// Custom RowMapper example
public class ProductMapper implements RowMapper<Product> {
    @Override
    public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
        Product product = new Product();
        product.setId(rs.getInt("id"));
        product.setTitle(rs.getString("title"));
        product.setPrice(rs.getInt("price"));
        product.setDescription(rs.getString("description"));
        return product;
    }
}
```

#### 3. DataSource

DataSource database connection pool manage karta hai. Spring Boot automatically data source configure kar deta hai application.properties file ke configurations ke basis par.

```properties
# Database Configuration Example
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=root
spring.datasource.password=root1234
```

### JDBC Implementation Steps

#### Step 1: Dependencies Add Karna

Maven project mein, pom.xml file mein necessary dependencies add karein:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### Step 2: Database Configuration

application.properties file mein database details configure karein:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/live_batch
spring.datasource.username=root
spring.datasource.password=root1234
```

#### Step 3: Model Classes Create Karna

Apne domain objects ko represent karne ke liye model classes banayein:

```java
// Product.java
public class Product {
    private int id;
    private String title;
    private String description;
    private int price;
    private int catId;
    
    // Getters, setters, constructors, toString
}

// Category.java
public class Category {
    private int id;
    private String title;
    private String description;
    
    // Getters, setters, constructors, toString
}
```

#### Step 4: DAO Interface Create Karna

Data Access Object (DAO) interfaces define karein jo database operations ko specify karein:

```java
// ProductDao.java
public interface ProductDao {
    Product create(Product product);
    Product update(Product product, int productId);
    void delete(int productId);
    List<Product> getAll();
    Product get(int productId);
    List<Product> search(String keyword);
    List<Product> getAllByCategory(int catId);
}
```

#### Step 5: DAO Implementation

JdbcTemplate ka use karke DAO interfaces ko implement karein:

```java
// ProductDaoImpl.java
@Repository
public class ProductDaoImpl implements ProductDao {
    
    private JdbcTemplate jdbcTemplate;
    
    // Constructor injection
    public ProductDaoImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        // Table create karna agar exist nahi karti
        String createQuery = "CREATE TABLE IF NOT EXISTS products(id int primary key, title varchar(200) not null, description varchar(500) not null, price int not null, cat_id int, FOREIGN KEY (cat_id) REFERENCES categories(id))";
        jdbcTemplate.update(createQuery);
    }
    
    @Override
    public Product create(Product product) {
        String query = "insert into products(id,title,description,price,cat_id) values(?,?,?,?,?)";
        int update = jdbcTemplate.update(
            query,
            product.getId(),
            product.getTitle(),
            product.getDescription(),
            product.getPrice(),
            product.getCatId()
        );
        System.out.println(update + " rows affected");
        return product;
    }
    
    @Override
    public List<Product> getAll() {
        String query = "select * from products";
        List<Product> products = jdbcTemplate.query(query, new ProductMapper());
        return products;
    }
    
    // Baki methods ki implementation
}
```

#### Step 6: RowMapper Create Karna

ResultSet ko domain objects mein convert karne ke liye RowMapper implementation:

```java
// ProductMapper.java
public class ProductMapper implements RowMapper<Product> {
    @Override
    public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
        Product product = new Product();
        product.setId(rs.getInt("id"));
        product.setTitle(rs.getString("title"));
        product.setPrice(rs.getInt("price"));
        product.setDescription(rs.getString("description"));
        return product;
    }
}
```

#### Step 7: Application Class Mein Use Karna

Main application class mein DAOs ka use karke database operations perform karein:

```java
@SpringBootApplication
public class SpringJdbcEcomApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringJdbcEcomApplication.class, args);
        
        // Get beans from context
        ProductDao productDao = context.getBean(ProductDao.class);
        CategoryDao categoryDao = context.getBean(CategoryDao.class);
        
        // Create category
        Category category = new Category();
        category.setId(1001);
        category.setTitle("mobiles");
        category.setDescription("mobile phones");
        categoryDao.create(category);
        
        // Create product
        Product product = new Product();
        product.setId(101);
        product.setTitle("iPhone 14");
        product.setDescription("Best iOS Phone");
        product.setPrice(124000);
        product.setCatId(1001);
        productDao.create(product);
        
        // Get all products
        List<Product> products = productDao.getAll();
        products.forEach(System.out::println);
    }
}
```

## Spring JPA

Spring JPA (Java Persistence API) ORM (Object-Relational Mapping) approach hai database interactions ke liye. Ye entities aur repositories ke concept pe based hai.

### JPA Architecture

JPA ka main goal hai Object-Relational impedance mismatch ko solve karna - yani Java objects aur relational database tables ke beech ka gap ko bridge karna.

```
+---------------------+
|                     |
|  Entity Classes     |
|  (Java Objects)     |
|                     |
+---------------------+
          |
          v
+---------------------+
|                     |
|  JPA Repository     |
|                     |
+---------------------+
          |
          v
+---------------------+
|                     |
|  JPA Provider       |
|  (Hibernate)        |
|                     |
+---------------------+
          |
          v
+---------------------+
|                     |
|  Database           |
|                     |
+---------------------+
```

### JPA Components

#### 1. Entities

Entities database tables ko represent karne wale Java classes hain. @Entity annotation se mark kiya jata hai.

```java
@Entity
@Table(name = "jpa_products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int productId;
    
    @Column(name = "product_title", unique = true, nullable = false)
    private String title;
    
    private String description;
    private double price;
    
    @ManyToOne
    private Category category;
    
    // Getters, setters, constructors
}
```

#### 2. Repositories

JPA repositories pre-defined CRUD operations provide karte hain, aur custom queries ko bhi support karte hain.

```java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    // Custom finder methods
    List<Product> findByTitleContaining(String keyword);
    
    // Custom query methods
    @Query("select p from Product p where p.title like %:keyword%")
    List<Product> searchProductsByTitle(@Param("keyword") String keywords);
}
```

Bilkul Vaibhav! Chalo **Spring Data JPA** ke powerful feature – **method naming convention** aur **`@Param` vs `:` syntax** – ko **zero se master karein**. Yeh topic interviews aur real-world dono ke liye important hai ✅

---

## 🔍 1. **Method Naming Convention in Spring Data JPA**

> Spring tumhare method ke naam se hi query samajh jaata hai 😎  
> No need to write SQL or JPQL manually (unless needed)

---

### 🎯 Structure:
```text
findBy + <FieldName> + [Operation]
```

---

### ✅ Basic Examples:

| Method Name                      | What it does                            |
|----------------------------------|-----------------------------------------|
| `findByTitle(String title)`      | `WHERE title = ?`                       |
| `findByTitleContaining("abc")`  | `WHERE title LIKE %abc%`                |
| `findByPriceGreaterThan(100)`   | `WHERE price > 100`                     |
| `findByTitleAndPrice(...)`      | `WHERE title = ? AND price = ?`        |
| `findByTitleOrPrice(...)`       | `WHERE title = ? OR price = ?`         |
| `findByPriceBetween(min, max)`  | `WHERE price BETWEEN ? AND ?`          |
| `findByCategory_Name("Mobile")` | `JOIN category WHERE category.name = ?`|

---

### ✅ Operators You Can Use:

| Keyword           | Meaning                   |
|------------------|---------------------------|
| `IsNull`         | `IS NULL`                 |
| `IsNotNull`      | `IS NOT NULL`             |
| `Containing`     | `LIKE %value%`            |
| `StartingWith`   | `LIKE value%`             |
| `EndingWith`     | `LIKE %value`             |
| `GreaterThan`    | `>`                       |
| `LessThan`       | `<`                       |
| `Between`        | `BETWEEN`                 |
| `In`             | `IN (...)`                |
| `OrderByFieldAsc`| Sorting                   |

---

## 🧠 Text Diagram:

```text
Method: findByTitleContaining("phone")

Query: SELECT * FROM product WHERE title LIKE '%phone%'
```

---

## 🔍 2. `@Query` + `@Param` + `:` — Custom JPQL Queries

Agar complex ya optimized query chahiye ho, toh custom JPQL likh sakte ho.

### ✅ Example:
```java
@Query("SELECT p FROM Product p WHERE p.price > :minPrice")
List<Product> findExpensiveProducts(@Param("minPrice") double minPrice);
```

---

### 🔎 Explain:

| Part | Meaning |
|------|--------|
| `@Query(...)` | JPQL likhne ke liye annotation |
| `:minPrice`   | Query mein named placeholder |
| `@Param("minPrice")` | Java method parameter ka naam query ke `:minPrice` se bind karta hai |

---

### 🔄 Mapping:
```text
@Query: "p.price > :minPrice"
        ↑                ↑
      Placeholder       This must match the @Param name
```

So:
```java
@Param("minPrice") double minPrice
```
binds to
```jpql
:minPrice
```

---

### 🛑 If You Miss `@Param`, Error aayega:
> "No parameter available for name 'minPrice'"

---

## 🧾 Summary Table:

| Feature           | What it does                             |
|-------------------|-------------------------------------------|
| `findBy...`       | Auto-generates query based on method name |
| `Containing`      | `LIKE %value%`                            |
| `@Query(...)`     | Custom JPQL                               |
| `:paramName`      | JPQL placeholder                          |
| `@Param("...")`   | Binds method argument to query param      |

---

## ✅ Interview Bolne Layak Line:

> “Spring Data JPA auto-generates queries from method names using naming conventions, and for custom queries, we use `@Query` with named parameters mapped via `@Param` annotations.”

---
#### 3. EntityManager

EntityManager JPA's core interface hai jo persistence operations ko handle karta hai. JpaRepository internally EntityManager use karta hai.

### JPA Implementation Steps

#### Step 1: Dependencies Add Karna

Maven project mein, pom.xml file mein necessary dependencies add karein:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### Step 2: Database Configuration

application.properties file mein database details configure karein:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=root
spring.datasource.password=root1234

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

#### Step 3: Entity Classes Create Karna

Database tables ko represent karne ke liye entity classes banayein:

```java
// Category.java
@Entity
@Table(name = "jpa_category")
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    
    private String title;
    
    @JsonIgnore
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Product> productList;
    
    // Getters, setters
}

// Product.java
@Entity
@Table(name = "jpa_products")
public class Product {
    @Id
    @Column(name = "p_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int productId;
    
    @Column(name = "product_title", unique = true, nullable = false, length = 500)
    private String title;
    
    private String description;
    private double price;
    private boolean isLive = false;
    
    @ManyToOne
    private Category category;
    
    // Getters, setters, constructors
}
```

#### Step 4: Repository Interfaces Create Karna

JpaRepository ko extend karke repository interfaces banayein:

```java
// CategoryRepository.java
public interface CategoryRepository extends JpaRepository<Category, Integer> {
    // Basic CRUD methods automatically available
}

// ProductRepository.java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    // Custom finder methods
    List<Product> findByTitleContaining(String keyword);
    List<Product> findByPrice(double price);
    
    // Custom queries
    @Query("select p from Product p WHERE p.title =:title and p.price =:price")
    List<Product> getProductByTitle(@Param("title") String title, @Param("price") double price);
    
    @Query("select p from Product p JOIN fetch p.category where p.category.title =:catTitle")
    List<Product> getProductByCategoryTitle(@Param("catTitle") String title);
}
```

#### Step 5: Service Layer Create Karna

Business logic ko implement karne ke liye service layer:

```java
// ProductService.java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    public Product createProduct(Product product) {
        return productRepository.save(product);
    }
    
    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }
    
    public Product getProductById(int productId) {
        return productRepository.findById(productId)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found with id: " + productId));
    }
    
    // Other business methods
}
```

#### Step 6: Controller Layer Create Karna (for RESTful APIs)

REST endpoints expose karne ke liye controller layer:

```java
// ProductController.java
@RestController
@RequestMapping("/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        return new ResponseEntity<>(productService.createProduct(product), HttpStatus.CREATED);
    }
    
    @GetMapping
    public ResponseEntity<List<Product>> getAllProducts() {
        return new ResponseEntity<>(productService.getAllProducts(), HttpStatus.OK);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable int id) {
        return new ResponseEntity<>(productService.getProductById(id), HttpStatus.OK);
    }
    
    // Other endpoints
}
```

## JDBC vs JPA: Comparison

| Feature | Spring JDBC | Spring JPA |
|---------|------------|-----------|
| Level of Abstraction | Low-level, SQL-centric | High-level, Object-centric |
| Coding Effort | More boilerplate code | Less code, more annotations |
| Control over SQL | Complete control | Limited direct control |
| Performance | Better for simple CRUD | Optimized for complex relations |
| Learning Curve | Simpler to learn initially | Steeper learning curve |
| Use Case | Simple applications, specific SQL needs | Complex domain models with relationships |

```
+---------------------+            +---------------------+
|                     |            |                     |
|    Spring JDBC      |            |    Spring JPA       |
|                     |            |                     |
+---------------------+            +---------------------+
| - SQL statements    |            | - Entity mapping    |
| - JdbcTemplate      |            | - Repositories      |
| - RowMapper         |            | - JPQL              |
| - Manual mapping    |            | - Automatic mapping |
+---------------------+            +---------------------+
```

## Spring JDBC Ecom Project Walkthrough

Ab hum spring-jdbc-ecom project ko step-by-step analyze karenge:

### Project Structure

```
spring-jdbc-ecom/
├── src/main/java/com/substring/jdbc/ecom/
│   ├── SpringJdbcEcomApplication.java
│   ├── dao/
│   │   ├── CategoryDao.java
│   │   ├── ProductDao.java
│   │   └── impl/
│   │       ├── CategoryDaoImpl.java
│   │       └── ProductDaoImpl.java
│   ├── helper/
│   │   └── ProductMapper.java
│   └── model/
│       ├── Category.java
│       ├── Product.java
│       └── ProductWithCategory.java
└── src/main/resources/
    └── application.properties
```

### Key Components ka Detailed Analysis

#### 1. Application.properties

```properties
spring.application.name=spring-jdbc-ecom
spring.datasource.url=jdbc:mysql://localhost:3306/live_batch
spring.datasource.username=root
spring.datasource.password=root1234
```

**Hinglish Explanation**: 
Ye file database connection ke liye configuration provide karti hai. Ismein database URL, username aur password define kiya gaya hai. Spring Boot automatically in properties ko use karke DataSource bean create karta hai.

#### 2. Model Classes

**Product.java**
```java
public class Product {
    private int id;
    private String title;
    private String description;
    private int price;
    private int catId;
    
    // Getters, setters, constructors, toString
}
```

**Hinglish Explanation**:
Product class ek simple Java bean hai jo products table ke data ko represent karta hai. Har field ek table column ko map karta hai. `catId` field Category se relationship establish karta hai.

**Category.java**
```java
public class Category {
    private int id;
    private String title;
    private String description;
    
    // Getters, setters, constructors, toString
}
```

**Hinglish Explanation**:
Category class ek simple Java bean hai jo categories table ke data ko represent karta hai.

**ProductWithCategory.java**
```java
public class ProductWithCategory {
    private int id;
    private String title;
    private String description;
    private String catTitle;
    
    // Getters, setters, constructors, toString
}
```

**Hinglish Explanation**:
Ye class Product aur Category ka join result represent karta hai. JOIN query ke results ko map karne ke liye use hota hai.

#### 3. DAO Interfaces

**ProductDao.java**
```java
public interface ProductDao {
    Product create(Product product);
    Product update(Product product, int productId);
    void delete(int productId);
    List<Product> getAll();
    Product get(int productId);
    List<Product> search(String keyword);
    List<Product> getAllByCategory(int catId);
    List<ProductWithCategory> getAllWithCategory();
}
```

**Hinglish Explanation**:
ProductDao interface Product table ke saath interact karne ke liye methods define karta hai. Database operations ko abstractions provide karta hai.

**CategoryDao.java**
```java
public interface CategoryDao {
    Category create(Category category);
}
```

**Hinglish Explanation**:
CategoryDao interface Category table ke operations ke liye methods define karta hai. Abhi sirf create method hai, but application requirements ke hisab se aur methods add kar sakte hain.

#### 4. DAO Implementations

**ProductDaoImpl.java**
```java
@Repository
public class ProductDaoImpl implements ProductDao {
    private JdbcTemplate jdbcTemplate;
    
    // Constructor injection
    public ProductDaoImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        // Create table if not exists
        String createQuery = "CREATE TABLE IF NOT EXISTS products(id int primary key, title varchar(200) not null, description varchar(500) not null, price int not null, cat_id int, FOREIGN KEY (cat_id) REFERENCES categories(id))";
        jdbcTemplate.update(createQuery);
    }
    
    @Override
    public Product create(Product product) {
        String query = "insert into products(id,title,description,price,cat_id) values(?,?,?,?,?)";
        int update = jdbcTemplate.update(
            query,
            product.getId(),
            product.getTitle(),
            product.getDescription(),
            product.getPrice(),
            product.getCatId()
        );
        System.out.println(update + " rows affected");
        return product;
    }
    
    @Override
    public List<Product> getAll() {
        String query = "select * from products";
        List<Product> products = jdbcTemplate.query(query, new ProductMapper());
        return products;
    }
    
    @Override
    public Product get(int productId) {
        return jdbcTemplate.queryForObject("select * from products where id = ?", new ProductMapper(), productId);
    }
    
    @Override
    public List<ProductWithCategory> getAllWithCategory() {
        String query = "Select p.id as id, p.title as title, p.description as description, p.price as price, c.title as catTitle FROM products p INNER JOIN categories c ON p.cat_id=c.id";
        return jdbcTemplate.query(query, (rs, rowNum) -> {
            ProductWithCategory productWithCategory = new ProductWithCategory();
            productWithCategory.setId(rs.getInt("id"));
            productWithCategory.setTitle(rs.getString("title"));
            productWithCategory.setDescription(rs.getString("description"));
            productWithCategory.setCatTitle(rs.getString("catTitle"));
            return productWithCategory;
        });
    }
    
    // Other methods implementation
}
```

Vaibhav, chalo ab samjhte hain ye dono JDBC-based methods ka breakdown in **Hinglish (Latin script)** with clear **explanations, text diagrams**, and most importantly — **difference between `query()` and `queryForObject()`**, and what `rs` does.

---

## 🧠 Pehle samjho: `jdbcTemplate`

Ye Spring ka helper hai jo:
- SQL query chalata hai
- Result ko Java object me convert karta hai
- Boilerplate code (try-catch, connection close) sab handle karta hai

---

# 🔍 Method 1: `getAllWithCategory()` — MULTIPLE results

```java
public List<ProductWithCategory> getAllWithCategory() {
    String query = "SELECT p.id, p.title, p.description, p.price, c.title AS catTitle FROM products p INNER JOIN categories c ON p.cat_id = c.id";

    return jdbcTemplate.query(query, (rs, rowNum) -> {
        ProductWithCategory pwc = new ProductWithCategory();
        pwc.setId(rs.getInt("id"));
        pwc.setTitle(rs.getString("title"));
        pwc.setDescription(rs.getString("description"));
        pwc.setCatTitle(rs.getString("catTitle"));
        return pwc;
    });
}
```

### 🧾 Breakdown:

| Part | Explanation |
|------|-------------|
| `jdbcTemplate.query(...)` | Used for **multiple rows** |
| `(rs, rowNum) -> {}` | Row mapper — lambda function jo har row ko object mein convert karta hai |
| `rs.getXXX()` | `rs` = ResultSet ⇒ ek row ka data access karne ke liye methods (getInt, getString, etc.) |

---

### 🎯 Real-Life Analogy:
> DB ne tumhe ek Excel sheet di jisme 20 rows hain → tum har row se ek `ProductWithCategory` object banate ho.

---

### 📘 Text Diagram:

```text
ResultSet from SQL:
+----+--------+-------------+--------+-----------+
| id | title  | description | price  | catTitle  |
+----+--------+-------------+--------+-----------+
| 1  | Phone  | iPhone 13   | 70000  | Mobile    |
| 2  | Razor  | Gillette    | 200    | Grooming  |
+----+--------+-------------+--------+-----------+

Mapped to:
List<ProductWithCategory>
```

---

# 🔍 Method 2: `get(int productId)` — SINGLE result

```java
@Override
public Product get(int productId) {
    return jdbcTemplate.queryForObject(
        "SELECT * FROM products WHERE id = ?",
        new ProductMapper(),  // RowMapper implementation
        productId
    );
}
```

### 🧾 Breakdown:

| Part | Explanation |
|------|-------------|
| `queryForObject(...)` | Use when query returns **only one row** |
| `ProductMapper` | Separate class implementing `RowMapper<Product>` |
| `productId` | Query parameter (replaces `?`) |

---

### ✅ What is `ProductMapper`?

```java
public class ProductMapper implements RowMapper<Product> {
    public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
        Product p = new Product();
        p.setId(rs.getInt("id"));
        p.setTitle(rs.getString("title"));
        return p;
    }
}
```

➡ This is like a reusable **converter** to map DB row to Java object.

---

### 📘 Text Diagram:

```text
Query: SELECT * FROM products WHERE id = 1

ResultSet:
+----+--------+-------------+-------+
| id | title  | description | price |
+----+--------+-------------+-------+
| 1  | Phone  | iPhone 13   | 70000 |
+----+--------+-------------+-------+

Mapped to:
Product object
```

---

## 🆚 `query()` vs `queryForObject()` — Difference

| Feature               | `query()`                           | `queryForObject()`                |
|------------------------|-------------------------------------|-----------------------------------|
| Returns                | `List<T>` (0 or more rows)         | `T` (exactly one row)             |
| Use case               | Get all products / users list       | Get product by ID (one row only) |
| Error if 0 rows        | ❌ No error, returns empty list     | ❌ `EmptyResultDataAccessException` |
| Error if >1 rows       | ❌ No error                         | ⚠️ `IncorrectResultSizeDataAccessException` |

---

## ❓ What does `rs` do?

`rs` = **ResultSet**  
- Ye **cursor** jaisa hota hai jo DB ke result pe iterate karta hai
- `rs.getInt("id")`, `rs.getString("title")` se column values milti hain

Think of it like:

```text
ResultSet → [ row1, row2, row3... ]
rs.getString("column_name") → current row ka column value
```

---

## ✅ Interview Explain (1-min ready pitch):

> "`query()` is used when I expect **multiple rows** from the database, and I map each row to an object using a lambda or a `RowMapper`.  
> On the other hand, `queryForObject()` is for **a single row**, like fetching a product by ID.  
> `rs` is the `ResultSet` object which gives me access to each column of the current row."

---


**Hinglish Explanation**:
ProductDaoImpl class ProductDao interface ko implement karta hai. Ye JdbcTemplate ka use karke database operations perform karta hai. Constructor mein, ek "CREATE TABLE IF NOT EXISTS" query execute hoti hai table structure ensure karne ke liye.

Key points:
- `@Repository` annotation ise Spring bean banata hai
- Constructor injection se JdbcTemplate inject kiya jata hai
- `create()` method prepared statement parameters ke saath insert query execute karta hai
- `getAll()` method products fetch karne ke liye ProductMapper ka use karta hai
- `getAllWithCategory()` method JOIN query execute karke products aur unki categories fetch karta hai
- Anonymous RowMapper implementation lambda expression ke through di gayi hai

**CategoryDaoImpl.java**
```java
@Repository
public class CategoryDaoImpl implements CategoryDao {
    private JdbcTemplate jdbcTemplate;
    
    public CategoryDaoImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        String createQuery = "CREATE TABLE IF NOT EXISTS categories(id int primary key, title varchar(200) not null, description varchar(500) not null)";
        jdbcTemplate.update(createQuery);
        System.out.println("category table is created or exists");
    }
    
    @Override
    public Category create(Category category) {
        int update = jdbcTemplate.update("insert into categories(id,title,description) values(?,?,?)", 
            category.getId(), 
            category.getTitle(), 
            category.getDescription()
        );
        System.out.println(update + " categories added");
        return category;
    }
}
```

**Hinglish Explanation**:
CategoryDaoImpl class CategoryDao interface ko implement karta hai. Ye bhi JdbcTemplate ka use karke database operations perform karta hai.

#### 5. Helper Classes

**ProductMapper.java**
```java
public class ProductMapper implements RowMapper<Product> {
    @Override
    public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
        Product product = new Product();
        product.setId(rs.getInt("id"));
        product.setTitle(rs.getString("title"));
        product.setPrice(rs.getInt("price"));
        product.setDescription(rs.getString("description"));
        return product;
    }
}
```

**Hinglish Explanation**:
ProductMapper class ResultSet rows ko Product objects mein convert karta hai. Ye RowMapper interface ko implement karta hai aur mapRow method override karta hai.

#### 6. Main Application

**SpringJdbcEcomApplication.java**
```java
@SpringBootApplication
public class SpringJdbcEcomApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringJdbcEcomApplication.class, args);
        
        ProductDao productDao = context.getBean(ProductDao.class);
        CategoryDao categoryDao = context.getBean(CategoryDao.class);
        
        // Comments mein sample operations hain
        
        // GET all products with category
        productDao.getAllWithCategory().forEach(System.out::println);
    }
}
```

**Hinglish Explanation**:
Main application class Spring Boot application ka entry point hai. Ye Spring context initialize karta hai aur ProductDao aur CategoryDao beans access karta hai. Demo ke liye, ye getAllWithCategory method call karke results print karta hai.

### Code Dry Run with Hinglish Comments

Ab hum code ko step-by-step execute karke dekhenge kaise kaam karta hai:

1. **Application Start**:
   ```java
   // Spring Boot application start hota hai
   ConfigurableApplicationContext context = SpringApplication.run(SpringJdbcEcomApplication.class, args);
   ```
   
   *Hinglish Comment*: Sabse pehle Spring Boot application start hota hai aur ConfigurableApplicationContext create hota hai. Is context mein saare configured beans available hote hain.

2. **Bean Access**:
   ```java
   // Context se DAO beans retrieve kiye jaate hain
   ProductDao productDao = context.getBean(ProductDao.class);
   CategoryDao categoryDao = context.getBean(CategoryDao.class);
   ```
   
   *Hinglish Comment*: Context se ProductDao aur CategoryDao ke beans retrieve kiye jaate hain. In beans ko autowire kiya gaya hai aur ye database operations perform karne ke liye ready hain.

3. **Category Creation** (currently commented):
   ```java
   // Category create karne ka code (abhi comment out hai)
   Category category = new Category();
   category.setId(1001);
   category.setTitle("mobiles");
   category.setDescription("mobiles phones");
   categoryDao.create(category);
   ```
   
   *Hinglish Comment*: Agar ye execute hota toh ek new Category object create hota, properties set hoti, aur categoryDao ke through database mein save hota.

4. **Product Creation** (currently commented):
   ```java
   // Product create karne ka code (abhi comment out hai)
   Product product1 = new Product();
   product1.setId(102);
   product1.setTitle("Iphone 14");
   product1.setDescription("Best IOS Phone");
   product1.setPrice(124000);
   product1.setCatId(1001);
   productDao.create(product1);
   ```
   
   *Hinglish Comment*: Agar ye execute hota toh ek new Product object create hota, properties set hoti, aur productDao ke through database mein save hota.

5. **Join Query Execution**:
   ```java
   // Products with category information fetch karna
   productDao.getAllWithCategory().forEach(System.out::println);
   ```
   
   *Hinglish Comment*: Is line pe productDao ka getAllWithCategory method call hota hai jo JOIN query execute karta hai products aur categories ke beech. Result ProductWithCategory objects ki list hoti hai jo print ho jati hai.

## Spring JPA Ecom Project Comparison

Ab hum Spring JPA ka implementation dekhenge aur JDBC implementation se compare karenge:

### Entity Classes vs Model Classes

**JPA Entity (Product.java)**:
```java
@Entity
@Table(name = "jpa_products")
public class Product {
    @Id
    @Column(name = "p_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int productId;
    
    @Column(name = "product_title", unique = true, nullable = false, length = 500)
    private String title;
    
    private String description;
    private double price;
    private boolean isLive = false;
    
    @ManyToOne
    private Category category;
    
    // Getters, setters, constructors
}
```

**JDBC Model (Product.java)**:
```java
public class Product {
    private int id;
    private String title;
    private String description;
    private int price;
    private int catId;
    
    // Getters, setters, constructors
}
```

**Hinglish Comparison**:
- JPA mein, classes ko `@Entity` annotation se mark kiya jata hai aur `@Table` se table name specify kiya jata hai
- JPA mein, fields ko `@Column`, `@Id`, `@GeneratedValue` jaise annotations se enhance kiya jata hai 
- JPA mein, relationships `@ManyToOne`, `@OneToMany` jaise annotations se define kiye jate hain
- JDBC mein, simple POJOs use hote hain without annotations
- JDBC mein, relationships manual handling require karte hain (catId field through)

### Repository vs DAO

**JPA Repository (ProductRepository.java)**:
```java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    List<Product> findByTitleContaining(String keyword);
    
    @Query("select p from Product p WHERE p.title =:title and p.price =:price")
    List<Product> getProductByTitle(@Param("title") String title, @Param("price") double price);
    
    @Query("select p from Product p JOIN fetch p.category where p.category.title =:catTitle")
    List<Product> getProductByCategoryTitle(@Param("catTitle") String title);
}
```

**JDBC DAO (ProductDao.java & ProductDaoImpl.java)**:
```java
public interface ProductDao {
    Product create(Product product);
    List<Product> getAll();
    // Other methods
}

@Repository
public class ProductDaoImpl implements ProductDao {
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public List<Product> getAll() {
        String query = "select * from products";
        List<Product> products = jdbcTemplate.query(query, new ProductMapper());
        return products;
    }
    
    // Other methods
}
```

**Hinglish Comparison**:
- JPA mein, sirf interface define karna hota hai jo JpaRepository extend karta hai
- JPA automatically CRUD operations provide karta hai (save, findById, findAll, delete, etc.)
- JPA mein, method names se queries generate ho jati hain (findByTitleContaining)
- JPA mein, complex queries @Query annotation se define ki jati hain with JPQL
- JDBC mein, interface aur implementation dono define karne padte hain
- JDBC mein, har operation ke liye SQL directly likhni padti hai
- JDBC mein, ResultSet ko objects mein manually map karna padta hai

### Data Handling

**JPA Data Handling**:
```java
// Entity save karna
productRepository.save(product);

// Entity fetch karna
Product product = productRepository.findById(1).orElseThrow();

// Custom query
List<Product> products = productRepository.findByTitleContaining("phone");
```

**JDBC Data Handling**:
```java
// Data save karna
String query = "insert into products(id,title,description,price,cat_id) values(?,?,?,?,?)";
jdbcTemplate.update(query, product.getId(), product.getTitle(), product.getDescription(), product.getPrice(), product.getCatId());

// Data fetch karna
String query = "select * from products";
List<Product> products = jdbcTemplate.query(query, new ProductMapper());
```

**Hinglish Comparison**:
- JPA object-oriented approach follow karta hai - objects ko directly save/retrieve kiya jata hai
- JDBC SQL-centric approach follow karta hai - SQL queries explicitly likhi jati hain
- JPA mein boilerplate code kam hota hai
- JDBC mein control zyada hota hai

## Interview Questions & Answers

### Spring JDBC Questions

**Q1: Spring JDBC kya hai aur traditional JDBC se kaise different hai?**

**A:** Spring JDBC ek abstraction layer hai jo traditional JDBC programming ko simplify karta hai. Traditional JDBC mein developer ko connections manage karne, statements create karne, ResultSets process karne, aur exceptions handle karne ki responsibility hoti hai. Spring JDBC ye sab boilerplate code manage karta hai aur developers ko business logic pe focus karne deta hai.

Key differences:
1. Exception handling - Spring JDBC checked exceptions ko unchecked exceptions mein convert karta hai
2. Resource management - Spring JDBC automatically connections, statements, aur result sets close karta hai
3. Less code - Spring JDBC mein kam code likhna padta hai same functionality ke liye

**Q2: JdbcTemplate kya hai aur iske advantages kya hain?**

**A:** JdbcTemplate Spring JDBC ka central class hai jo JDBC operations execute karta hai. Ye SQL queries execute karne, result sets process karne, aur exceptions handle karne ki functionality provide karta hai.

Advantages:
1. Boilerplate code kam karta hai (connection handling, statement creation, exception handling)
2. SQL exceptions ko Spring's DataAccessException hierarchy mein translate karta hai
3. Resources (connections, statements, result sets) leak nahi hote kyunki automatically close ho jate hain
4. Thread-safe hai aur multiple threads se safely use kiya ja sakta hai

**Q3: Spring JDBC mein RowMapper kya hai aur kaise use kiya jata hai?**

**A:** RowMapper ek interface hai jo ResultSet ke rows ko Java objects mein convert karta hai. Ye mapRow() method define karta hai jo ResultSet aur row number lete hain aur Java object return karta hai.

Usage:
```java
public class ProductMapper implements RowMapper<Product> {
    @Override
    public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
        Product product = new Product();
        product.setId(rs.getInt("id"));
        product.setTitle(rs.getString("title"));
        product.setPrice(rs.getInt("price"));
        product.setDescription(rs.getString("description"));
        return product;
    }
}

// Use with JdbcTemplate
List<Product> products = jdbcTemplate.query("select * from products", new ProductMapper());
```

**Q4: Spring JDBC mein different types of JDBC operations kaise execute kiye jate hain?**

**A:** Spring JDBC mein different types ke operations ke liye JdbcTemplate different methods provide karta hai:

1. **Update operations (INSERT, UPDATE, DELETE)**:
   ```java
   int rowsAffected = jdbcTemplate.update("insert into products(id,title) values(?,?)", 101, "Phone");
   ```

2. **Query for single object**:
   ```java
   Product product = jdbcTemplate.queryForObject("select * from products where id=?", new ProductMapper(), 101);
   ```

3. **Query for list of objects**:
   ```java
   List<Product> products = jdbcTemplate.query("select * from products", new ProductMapper());
   ```

4. **Query for single value**:
   ```java
   int count = jdbcTemplate.queryForObject("select count(*) from products", Integer.class);
   ```

5. **Batch operations**:
   ```java
   int[] batchUpdate = jdbcTemplate.batchUpdate("insert into products values(?,?,?)", 
       new BatchPreparedStatementSetter() {
           @Override
           public void setValues(PreparedStatement ps, int i) throws SQLException {
               ps.setInt(1, products.get(i).getId());
               ps.setString(2, products.get(i).getTitle());
               ps.setString(3, products.get(i).getDescription());
           }
           
           @Override
           public int getBatchSize() {
               return products.size();
           }
       });
   ```

### Spring JPA Questions

**Q1: Spring Data JPA kya hai aur iske advantages kya hain?**

**A:** Spring Data JPA ek framework hai jo JPA-based repositories implement karne ko simplify karta hai. Ye boilerplate code ko reduce karta hai aur standard CRUD operations automatically provide karta hai.

Advantages:
1. Boilerplate code significantly reduce hota hai
2. Method naming conventions se queries automatically generate hoti hain
3. Custom queries @Query annotation se easily define ki ja sakti hain
4. Pagination aur sorting support built-in hai
5. Auditing support (@CreatedDate, @LastModifiedDate) available hai

**Q2: JPA mein Entity kya hai aur kaise define kiya jata hai?**

**A:** Entity ek Java class hai jo database table ko represent karta hai. Har entity object table ka ek row represent karta hai.

Entity define karne ke liye:
1. Class ko @Entity annotation se mark karein
2. @Id annotation se primary key field mark karein
3. Optionally @Table annotation se table name specify karein
4. Optionally @Column annotations se column properties specify karein

Example:
```java
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "product_name", nullable = false)
    private String name;
    
    private double price;
    
    // Getters, setters
}
```

**Q3: JPA mein different types ke relationships kaise handle kiye jate hain?**

**A:** JPA entities ke beech four types ke relationships support karta hai:

1. **@OneToOne** - Ek entity dusre entity se one-to-one relationship rakhti hai:
   ```java
   @OneToOne
   @JoinColumn(name = "address_id")
   private Address address;
   ```

2. **@OneToMany / @ManyToOne** - Ek entity dusre entities ke collection se related hai:
   ```java
   // In Category class
   @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
   private List<Product> products;
   
   // In Product class
   @ManyToOne
   @JoinColumn(name = "category_id")
   private Category category;
   ```

3. **@ManyToMany** - Entities ke beech many-to-many relationship:
   ```java
   @ManyToMany
   @JoinTable(
       name = "product_tag",
       joinColumns = @JoinColumn(name = "product_id"),
       inverseJoinColumns = @JoinColumn(name = "tag_id")
   )
   private Set<Tag> tags;
   ```

**Q4: Spring Data JPA mein custom queries kaise define kiye jate hain?**

**A:** Spring Data JPA mein custom queries define karne ke multiple ways hain:

1. **Method naming conventions**:
   ```java
   List<Product> findByTitleContaining(String keyword);
   List<Product> findByPriceGreaterThan(double price);
   List<Product> findByTitleAndPrice(String title, double price);
   ```

2. **@Query annotation with JPQL**:
   ```java
   @Query("select p from Product p where p.title like %:keyword%")
   List<Product> searchProducts(@Param("keyword") String keyword);
   ```

3. **@Query annotation with native SQL**:
   ```java
   @Query(value = "select * from products where price > ?1", nativeQuery = true)
   List<Product> findExpensiveProducts(double minPrice);
   ```

4. **Query by Example**:
   ```java
   Product probe = new Product();
   probe.setTitle("Phone");
   
   Example<Product> example = Example.of(probe);
   List<Product> products = productRepository.findAll(example);
   ```

### Comparison Questions

**Q1: Spring JDBC aur Spring JPA ke beech main differences kya hain?**

**A:** Spring JDBC aur Spring JPA ke beech key differences:

1. **Abstraction Level**:
   - JDBC: Low-level abstraction, SQL-centric approach
   - JPA: High-level abstraction, object-centric approach

2. **Code Quantity**:
   - JDBC: More boilerplate code (SQL writing, mapping, exception handling)
   - JPA: Less code, annotations-based approach

3. **SQL Control**:
   - JDBC: Complete control over SQL queries
   - JPA: Limited direct SQL control, mostly generated queries

4. **Learning Curve**:
   - JDBC: Easier to learn initially
   - JPA: Steeper learning curve with concepts like entities, relationships, JPQL

5. **Performance**:
   - JDBC: Can be more performant for simple operations and specific optimizations
   - JPA: May have overhead but optimized for complex object relationships

6. **Use Cases**:
   - JDBC: Simple applications, specific SQL optimization needs
   - JPA: Complex domain models with relationships

**Q2: Kab Spring JDBC use karna chahiye aur kab Spring JPA use karna chahiye?**

**A:** 

**Spring JDBC use karein when**:
1. Complete SQL control chahiye
2. Simple CRUD operations hi required hain
3. Performance critical hai aur SQL optimizations zaruri hain
4. Legacy database structures ke saath work karna hai
5. Direct stored procedures calls karne hain
6. Learning curve kam rakhi jani chahiye

**Spring JPA use karein when**:
1. Complex domain model hai relationships ke saath
2. Boilerplate code ko minimize karna hai
3. Database vendor independence chahiye
4. Maintainability aur readability priority hai
5. Advanced features like auditing, versioning, caching chahiye
6. Object-oriented approach prefer hai

**Q3: Kya Spring JDBC aur Spring JPA ek saath use kiye ja sakte hain? Kaise?**

**A:** Yes, Spring JDBC aur Spring JPA ek saath use kiye ja sakte hain. 

**Implementation approach**:
1. JPA repositories use karein regular CRUD operations ke liye
2. Custom JDBC operations define karein complex queries ya performance-critical parts ke liye
3. @PersistenceContext se EntityManager inject karein aur specific JPA operations perform karein

Example:
```java
@Service
public class ProductService {
    @Autowired
    private ProductRepository productRepository; // JPA repository
    
    @Autowired
    private JdbcTemplate jdbcTemplate; // JDBC template
    
    // JPA for regular operations
    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }
    
    // JDBC for complex query
    public List<Map<String, Object>> getProductStatistics() {
        String sql = "SELECT category_id, AVG(price) as avg_price, COUNT(*) as product_count " +
                     "FROM products GROUP BY category_id";
        return jdbcTemplate.queryForList(sql);
    }
}
```

**Q4: Transaction management Spring JDBC aur Spring JPA mein kaise handle kiya jata hai?**

**A:** 

**Spring JDBC mein transaction management**:
```java
@Service
@Transactional
public class ProductServiceJdbc {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public void transferFunds(int fromAccount, int toAccount, double amount) {
        jdbcTemplate.update("UPDATE accounts SET balance = balance - ? WHERE id = ?", amount, fromAccount);
        
        // Agar koi exception throw hoti hai to transaction rollback ho jayega
        if (someCondition) {
            throw new RuntimeException("Transfer failed");
        }
        
        jdbcTemplate.update("UPDATE accounts SET balance = balance + ? WHERE id = ?", amount, toAccount);
    }
}
```

**Spring JPA mein transaction management**:
```java
@Service
@Transactional
public class ProductServiceJpa {
    @Autowired
    private ProductRepository productRepository;
    @Autowired
    private CategoryRepository categoryRepository;
    
    public void createProductWithCategory(Product product, Category category) {
        categoryRepository.save(category);
        
        // Agar koi exception throw hoti hai to transaction rollback ho jayega
        if (someCondition) {
            throw new RuntimeException("Creation failed");
        }
        
        product.setCategory(category);
        productRepository.save(product);
    }
}
```

**Key differences**:
1. Basic usage same hai - @Transactional annotation both approaches mein work karta hai
2. JPA mein persistence context transaction ke scope se bind hai
3. JDBC mein direct connection operations transaction ke part hote hain
4. JPA automatically dirty checking karta hai changes detect karne ke liye
5. JDBC mein explicit updates necessary hote hain

## Conclusion

Spring JDBC aur Spring JPA dono powerful approaches hain database connectivity ke liye, with different advantages.

**Spring JDBC**:
- Direct SQL control
- Simple approach
- Perfect for simpler applications
- Good when SQL optimization crucial hai

**Spring JPA**:
- Object-oriented approach
- Less boilerplate code
- Perfect for complex domain models
- Great when maintainability priority hai

Project requirements ke basis par approach choose karein. Aur yaad rakhein, dono approaches ek application mein mix bhi kiye ja sakte hain when needed!
