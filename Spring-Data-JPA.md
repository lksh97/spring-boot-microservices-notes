**Spring Data JPA -**  

## 1) **Introduction to Spring Data JPA**

### **Spring Data JPA** kya hai?  
- Ye **Spring Data** ka ek part hai jo **JPA (Java Persistence API)** ke through **database** se interact karna bahut aasan bana deta hai.  
- Aapko **boilerplate-free** code likhne deta hai, kyunki aapko basic CRUD (Create, Read, Update, Delete) operations manually likhne ki zarurat nahi padti.  

### **Key Features**:
1. **Simplified CRUD**: Boilerplate code se chhutkara.  
2. **Derived Queries**: Method name se query generation (e.g., `findByName(...)`).  
3. **JPQL / Native Queries**: Aap custom queries bhi likh sakte ho.  
4. **Pagination and Sorting**: Built-in support hai.  
5. **Repository Support**: `JpaRepository`, `CrudRepository` jaise interfaces se aap easily CRUD operations kar sakte ho.  

### **Advantages**:
- **Boilerplate-free**  
- **Fast Development**: Spring Boot ke saath seamlessly integrate hota hai.  
- **Scalable**: Advanced features jaise caching, transactions, lazy loading.  
- **Database Independence**: MySQL, PostgreSQL, H2, etc. sab ke saath use kar sakte ho.  

---

## 2) **Setting Up the Environment**

1. **Prerequisites**  
   - **JDK** (Java 11 ya higher)  
   - **Spring Boot** (auto-config)  
   - **Database** (MySQL, PostgreSQL, ya H2)  
   - **IDE** (IntelliJ, Eclipse, vs.)  
   - **Build Tool** (Maven/Gradle)

2. **Project Creation**  
   - **Spring Initializr** (https://start.spring.io)  
   - **IDE** ke through (Spring project wizard)

3. **Dependencies** (pom.xml mein)  
   - Spring Web (`spring-boot-starter-web`)  
   - Spring Data JPA (`spring-boot-starter-data-jpa`)  
   - Database Driver (e.g. MySQL)  
   - (Optional) DevTools

```xml
<!-- pom.xml: Spring Boot + Spring Data JPA + MySQL example -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- project metadata (groupId, artifactId, version) -->
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>  <!-- Hinglish comment: 'groupId' aapke project ka domain group represent karta hai. -->
    <artifactId>spring-data-jpa-demo</artifactId> <!-- 'artifactId' project ka naam. -->
    <version>1.0.0</version> <!-- version number of your project -->

    <!-- Parent Spring Boot version define kar rahe hain -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version> <!-- Ye version Spring Boot ki version specify karta hai. -->
        <relativePath/> 
    </parent>

    <!-- Java version specify kar sakte hain yahan -->
    <properties>
        <java.version>17</java.version> <!-- JDK 17 use kar rahe hain. -->
    </properties>

    <dependencies>
        <!-- Spring Boot Web Starter: isse hum apne REST endpoints bana sakte hain, web server use kar sakte hain. -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Starter Data JPA: isse hum Spring Data JPA functionalities, repositories, etc. use kar sakte hain. -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- MySQL Driver: Ye library MySQL database se connect hone ke liye zaruri hai. -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope> <!-- runtime means jab application chalega tab ye jar load hoga. -->
        </dependency>

        <!-- (Optional) DevTools: isse aapke code mein changes hote hi auto-restart ho sakta hai. Dev time par bahut helpful. -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <!-- build ke time ke settings -->
    <build>
        <plugins>
            <!-- Spring Boot Maven plugin: run, build, package sab handle karega. -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal> <!-- repackage se jar banega executable -->
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

4. **application.properties** (MySQL example):

```
# application.properties

# ============= Database Configuration =============
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce 
# Hinglish comment: 'jdbc:mysql://localhost:3306/ecommerce' matlab hum local MySQL server pe 'ecommerce' naam ka DB use kar rahe hain.

spring.datasource.username=root 
# Hinglish comment: Database username, e.g. 'root'.

spring.datasource.password=my_password 
# Hinglish comment: Database ka password.

# ============= JPA (Hibernate) Settings =============
spring.jpa.hibernate.ddl-auto=update 
# Hinglish comment: 'update' matlab table schema automatically create/update karega but existing data rehga. 
# Production me kabhi 'create-drop' mat use karo unless testing. 
# (DDL = Data Definition Language).

spring.jpa.show-sql=true 
# Hinglish comment: Ye true rakhenge to console me actual SQL queries dikhengi.

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect 
# Hinglish comment: Hibernate ko batata hai hum MySQL database use kar rahe hain. 
```

- `ddl-auto=update` ka matlab DB schema ko auto-update karna, **production** me dhyaan se use karein.

---

## 3) **Basics of JPA Entities**

### **JPA Entity** kya hai?  
- Ek **Java class** jo **database table** ko represent karti hai.  
- JPA (yaani Hibernate, etc.) ke through ye class **table ke records** ko Java objects me map karti hai.  

### **Key Annotations**:
- `@Entity`: Class ko entity banata hai.  
- `@Table`: Table ka naam specify karta hai.  
- `@Id`: Primary key field.  
- `@GeneratedValue`: Primary key ki generation strategy (IDENTITY, AUTO, etc.)  
- `@Column`: Field ko table column se map karta hai.  
- `@Transient`: Iss field ko persist mat karo.  

### **Code Example**: _Basic JPA Entity_
```java
package com.example.springdatajpademo.entity;

import jakarta.persistence.*;

@Entity // Hinglish comment: Ye bata raha hai ki ye class DB table se map hogi.
@Table(name = "products") // Is table ka naam 'products' rakha hai.
public class Product {

    @Id // Ye primary key define karta hai.
    @GeneratedValue(strategy = GenerationType.IDENTITY) 
    // Hinglish comment: Ye MySQL ke auto-increment behavior ko use karega.
    private Long id;

    @Column(nullable = false, unique = true) 
    // Hinglish comment: 'nullable=false' means ye column empty nahin reh sakta. 
    // 'unique=true' means koi duplicate value nahin hogi.
    private String name;

    private double price; 
    // Hinglish comment: isko 'price' column se map karega. By default column name 'price' hoga.

    private int stock; 
    // Hinglish comment: isko 'stock' column se map karega.

    @Transient 
    // Hinglish comment: Ye '@Transient' ka matlab hai 
    // 'is field ko DB mein mat store karna/na hi read karna'.
    // e.g. aap koi temporary computed field rakhna chahte ho, 
    // jo DB se relate nahin hai, to '@Transient' use kar sakte ho.
    private String notPersistedField;

    // Hinglish comment: Aage getters/setters

    public Long getId() {
        return id; // Humein yahan sirf id return karni hai
    }

    public void setId(Long id) {
        this.id = id; 
    }

    public String getName() {
        return name; 
    }

    public void setName(String name) {
        this.name = name; 
    }

    public double getPrice() {
        return price; 
    }

    public void setPrice(double price) {
        this.price = price; 
    }

    public int getStock() {
        return stock; 
    }

    public void setStock(int stock) {
        this.stock = stock; 
    }

    public String getNotPersistedField() {
        return notPersistedField;
    }

    public void setNotPersistedField(String notPersistedField) {
        this.notPersistedField = notPersistedField;
    }
}
```

**Text Diagram**:

```
+-------------------+
|   products (DB)   |
+---------+---------+
| id      | BIGINT  |
| name    | VARCHAR |
| price   | DOUBLE  |
| stock   | INT     |
+---------+---------+
       ^
       |
  Product (Entity)
  @Entity
  - id, name, price, stock
```

### **Common PK Generation Strategies**  
- `IDENTITY`: DB apne aap increment karega.  
- `SEQUENCE`: Database sequence use hota hai.  
- `AUTO`: JPA khud decide karega DB ke hisaab se.  

### **Testing the Entity**  
Ek simple `JpaRepository` banate hain:

```java
package com.example.springdatajpademo.repository;

import com.example.springdatajpademo.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository // Hinglish comment: Mark karega is interface ko as a bean for data access layer.
public interface ProductRepository extends JpaRepository<Product, Long> {
    // Hinglish comment: 'Product' is the entity type
    // 'Long' is the type of primary key (id).
    // Ab hum derived queries ya custom queries yahan define kar sakte hain 
    // e.g. List<Product> findByName(String name);
}
```

Aur ek `CommandLineRunner` / `TestRunner` banake CRUD test kar sakte hain:

```java
package com.example.springdatajpademo;

import com.example.springdatajpademo.entity.Product;
import com.example.springdatajpademo.repository.ProductRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component 
// Hinglish comment: Isse ye class Spring container me ek bean ke tarah register ho jayegi, 
// jisse run method call ho sake on startup.
public class DemoRunner implements CommandLineRunner {

    private final ProductRepository productRepository;
    // Hinglish comment: hum ProductRepository ko inject kar rahe hain 
    // taaki hum database pe CRUD operations kar sakein.

    // Constructor Injection
    public DemoRunner(ProductRepository productRepository) {
        this.productRepository = productRepository; 
    }

    @Override
    public void run(String... args) throws Exception {
        // Hinglish comment: 'run' method tab call hota hai jab application fully startup ho jata hai.

        System.out.println("**** Application Started - Now Inserting Test Data ****");

        // 1) Naya product banate hain
        Product product1 = new Product();
        product1.setName("Laptop");
        product1.setPrice(1000.00);
        product1.setStock(10);

        // 2) save() method se DB me insert ho jayega
        productRepository.save(product1);
        // Hinglish comment: Ye 'save' method JPA repository ka default method hai.

        // 3) sab products retrieve karke print karte hain
        System.out.println("All products from DB: " + productRepository.findAll());

        // 4) ek or product
        Product product2 = new Product();
        product2.setName("Smartphone");
        product2.setPrice(500.00);
        product2.setStock(5);

        productRepository.save(product2);

        // 5) 'findAll' fir se, updated list
        System.out.println("All products after second insert: " + productRepository.findAll());
        
        // Aap further findById, deleteById waghera sab check kar sakte ho
    }
}
```

---

## 4) **Query Methods in Repositories**

### 1) **Derived Query Methods**  
- Spring Data method names se hi query bana deta hai.  
- Example: `findByName(String name)` => `SELECT * FROM product WHERE name = ?`

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByName(String name);
    long countByPriceGreaterThan(double price);
}
```

### 2) **Query Creation**  
Agar derived method se kaam nahi chal raha, to aap `@Query` use kar sakte ho.

```java
@Query("SELECT p FROM Product p WHERE p.name = :name AND p.price > :price")
List<Product> findExpensiveProducts(@Param("name") String productName, @Param("price") double price);
```

### 3) **Native Queries**  
Jab aapko raw SQL chahiye:

```java
@Query(value = "SELECT * FROM products WHERE name = ?1", nativeQuery = true)
Product findByNameNative(String name);
```

### 4) **JPQL (Java Persistence Query Language)**  
SQL jaise hi hota hai, bas tables ke bajay **entities** pe operate karta hai.

```java
@Query("SELECT p FROM Product p WHERE p.stock < :stockLevel")
List<Product> findLowStockProducts(@Param("stockLevel") int stock);
```

---

## 5) **Pagination and Sorting in Spring Data**

### 1) **Pagination** (`Pageable`, `Page` Objects)

- Aap repository method me `Pageable` pass karke paginated results le sakte ho.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    Page<Product> findAll(Pageable pageable);
}
```

_Controller Example_:
```java
@GetMapping("/products")
public Page<Product> getProducts(@RequestParam int page, @RequestParam int size) {
    Pageable pageable = PageRequest.of(page, size);
    return productRepository.findAll(pageable);
}
```

### 2) **Sorting** (`Sort` class)

```java
Pageable pageable = PageRequest.of(page, size, Sort.by("price").descending());
```

Ya multiple fields:
```java
Sort sort = Sort.by("name").ascending().and(Sort.by("price").descending());
Pageable pageable = PageRequest.of(page, size, sort);
```

---

## 6) **Relationships and Mappings**

**JPA** mein aap `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany` annotations se entities ke beech relationship define kar sakte ho.

### **1) One-to-One**

```java
@Entity
public class Person {
    @Id
    private Long id;

    @OneToOne
    private Address address; // One Person has One Address
}

@Entity
public class Address {
    @Id
    private Long id;

    @OneToOne(mappedBy = "address")
    private Person person;
}
```

### **2) One-to-Many / Many-to-One**

```java
@Entity
public class Course {
    @Id
    private Long id;

    @OneToMany(mappedBy = "course")
    private List<Review> reviews;
}

@Entity
public class Review {
    @Id
    private Long id;

    @ManyToOne
    private Course course; // Many Reviews to One Course
}
```

### **3) Many-to-Many**

```java
@Entity
public class Student {
    @Id
    private Long id;

    @ManyToMany
    @JoinTable(name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id"))
    private List<Course> courses;
}

@Entity
public class Course {
    @Id
    private Long id;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students;
}
```

### **4) Fetch Types (EAGER vs LAZY)**  
- `EAGER`: Relationship entity ko turant load karega.  
- `LAZY`: Tab load karega jab needed ho (performance ke liye accha).

```java
@OneToOne(fetch = FetchType.LAZY)
private Department department;
```

### **5) Cascade Types**  
- `PERSIST`, `MERGE`, `REMOVE`, `ALL`  
- Matlab aap parent entity pe operation karte ho, to child entities me bhi reflect ho.

```java
@OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
private List<Employee> employees;
```

### **6) `mappedBy`**  
- **Owning side** vs. **Inverse side**.  
- `mappedBy` ka matlab entity kaunse field ke through own ho rahi hai.

```java
public class Student {
    @ManyToOne
    @JoinColumn(name = "teacher_id")
    private Teacher teacher; // Owning side
}

public class Teacher {
    @OneToMany(mappedBy = "teacher")
    private List<Student> students; // Inverse side
}
```

---

## 7) **Entity Lifecycle Callbacks**

- Kabhi aapko entity **persist** hone se pehle kuch karna ho, ya persist hone ke baad, to lifecycle callbacks use karte ho (`@PrePersist`, `@PostPersist`, `@PreUpdate`, etc.).

#### Example: `@PrePersist`
```java
@Entity
public class Order {

    @Id
    private Long id;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @PrePersist
    private void setCreationDate() {
        this.createdDate = new Date();
    }
}
```

#### Example: `@PostPersist`
```java
@PostPersist
private void sendNotification() {
    System.out.println("Order created! Sending notification...");
}
```

#### Example: `@PreUpdate`
```java
@PreUpdate
private void checkSomethingBeforeUpdate() {
    // e.g., salary can't be negative
}
```

---

## 8) **JPA Mappings, Paging & Sorting, Auditing, and Soft Deletes**

### **1) JPA Mappings**  
- Already covered above.  
- Best practice: Samajh ke rakho `@Entity`, `@Table`, `@Column`, `@Embedded`, etc.  

### **2) Paging & Sorting**  
- Reiterate: Use `Pageable`, `Page`, `Sort` classes.  
- Ye code hum pichle section me dekh chuke hain.  

### **3) Auditing**  
- Aap automatically record kar sakte ho ki `entity` kab create hui, kab update hui, kis user ne ki, etc.  
- Steps:  
  1. `@EnableJpaAuditing` in a `@Configuration` class.  
  2. Ek base class banao, e.g. `@MappedSuperclass` with fields: `@CreatedDate`, `@LastModifiedDate`.  
  3. Apni entity us base class ko extend kar le.

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}

@Entity
public class Product extends Auditable {
    @Id
    private Long id;
    ...
}
```

### **4) Soft Deletes**  
- Matlab record physically DB se delete nahi hota, bas **flag** set ho jata hai (`deleted = true`).  

#### Base Class
```java
@MappedSuperclass
public abstract class SoftDeletable {
    @Column(name = "deleted", nullable = false)
    private boolean deleted = false;
    
    public void setDeleted(boolean deleted) {
        this.deleted = deleted;
    }
}
```

#### Entity Example
```java
@Entity
public class Product extends SoftDeletable {
    @Id
    private Long id;
    private String name;
}
```

#### Repository Example
```java
@Query("SELECT p FROM Product p WHERE p.deleted = false")
List<Product> findAllNotDeleted();
```

#### Service Example
```java
@Service
public class ProductService {
    @Autowired
    private ProductRepository productRepository;
    
    public void deleteProduct(Long id) {
        Product prod = productRepository.findById(id).orElseThrow(...);
        prod.setDeleted(true);
        productRepository.save(prod);
    }
}
```

#### **Combining Soft Deletes & Auditing**  
- Aap ek hi base class me `deleted`, `deletedDate`, `createdDate`, etc. sab combine kar sakte ho.

---

## **Conclusion / Recap**

1. **Spring Data JPA** aapki **DB operations** ko bahut simplify kar deta hai.  
2. **Entities** banane ke liye aap `@Entity` use karo, fields ko `@Id`, `@Column` se map karo.  
3. **Repositories** me `JpaRepository` extend karo, aur derived queries, `@Query`, `nativeQuery` se advanced retrieval kar sakte ho.  
4. **Relationships** (One-to-One, One-to-Many, etc.) define karke aap multi-table data handle kar sakte ho.  
5. **Pagination & Sorting** se large data handle ho jata hai.  
6. **Auditing** se aap entity ke create / update timestamps track kar sakte ho.  
7. **Soft Deletes** me record ko physically delete na karke bas flag set kar dete ho.  

_Is tarah aap ek real-world project me **Spring Data JPA** use karke code likh sakte ho, sab samajh ke aur asaani se interview me explain bhi kar sakte ho!_
