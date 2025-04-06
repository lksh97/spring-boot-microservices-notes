# Spring Boot API Development: Absolute Beginner's Guide

## 1. Introduction to Spring Boot API Development

Spring Boot API development mein ek web service create karna sikhenge jo client applications ke saath data exchange kar sake. REST (Representational State Transfer) APIs internet par data exchange ka standard way hai.

### REST API Kya Hai?

REST API ek architectural style hai jo client-server communication ke liye use hota hai:

- **Stateless**: Server client ka state nahi rakhta
- **Resource-based**: Har information ek resource hai jise URL se access kiya ja sakta hai
- **HTTP Methods**: GET, POST, PUT, DELETE operations ke liye use hote hain
- **Representation**: Resources ko JSON, XML formats mein represent kiya jata hai

```
+------------+      HTTP Requests      +------------+
|            |  GET/POST/PUT/DELETE    |            |
|  Client    | --------------------->  |   Server   |
|  (Browser/ |                         |  (Spring   |
|   Mobile)  | <---------------------  |   Boot)    |
|            |      HTTP Response      |            |
+------------+       (JSON/XML)        +------------+
```

## 2. Spring Boot Project Setup

### Dependencies Setup

Pehle hume pom.xml mein zaruri dependencies add karni hongi:

```xml
<dependencies>
    <!-- Web application ke liye -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Database connectivity ke liye -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- MySQL database ke liye -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Code simplify karne ke liye -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- Object mapping ke liye -->
    <dependency>
        <groupId>org.modelmapper</groupId>
        <artifactId>modelmapper</artifactId>
        <version>3.0.0</version>
    </dependency>
    
    <!-- API documentation ke liye -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.6.0</version>
    </dependency>
</dependencies>
```

### Database Configuration

application.properties mein database configuration setup karenge:

```properties
# Database URL
spring.datasource.url=jdbc:mysql://localhost:3306/your_database
# Database username
spring.datasource.username=root
# Database password
spring.datasource.password=your_password

# SQL queries console par dikhane ke liye
spring.jpa.show-sql=true
# Database tables automatically create/update karenge
spring.jpa.hibernate.ddl-auto=update
# SQL queries ko format karenge
spring.jpa.properties.hibernate.format_sql=true
```

## 3. Spring Boot API Project Structure

Ek standard Spring Boot API project ka structure kuch aise dikhta hai:

```
src/main/java/com/example/demo/
├── DemoApplication.java           <- Main application class
├── config/                        <- Configuration classes
├── controllers/                   <- REST API endpoints
├── dto/                           <- Data Transfer Objects
├── entities/                      <- Database entity classes
├── exceptions/                    <- Custom exceptions
├── repositories/                  <- Database access layers
├── services/                      <- Business logic
└── utils/                         <- Utility classes
```

## 4. Creating Data Models/Entities

Entities are classes that represent database tables.

```java
// Course.java
package com.elearn.app.entities;

import jakarta.persistence.*;
import lombok.Data;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Entity                // Ye class ek database table hai
@Table(name = "courses")  // Table ka name "courses" hoga
@Data                  // Lombok se getters, setters auto-generate honge
public class Course {

    @Id                 // Ye primary key hai
    private String id;  // Course ka unique identifier
    
    private String title;       // Course ka title
    
    private String shortDesc;   // Course ka short description
    
    @Column(length = 2000)      // Column ki length 2000 characters
    private String longDesc;    // Course ka detailed description
    
    private double price;       // Course ki price
    
    private boolean live = false;  // Course live hai ya nahi (default: false)
    
    private double discount;    // Course par discount
    
    private Date createdDate;   // Course create hone ki date
    
    // Course ke banner ka path store karenge
    private String banner;
    private String bannerContentType;  // Banner image ka content type (JPEG, PNG, etc.)
    
    // Course ke videos (One-to-Many relationship)
    @OneToMany(mappedBy = "course")
    private List<Video> videos = new ArrayList<>();
    
    // Course ke categories (Many-to-Many relationship)
    @ManyToMany(cascade = CascadeType.ALL)
    private List<Category> categoryList = new ArrayList<>();
    
    // Category add karne ka helper method
    public void addCategory(Category category) {
        categoryList.add(category);
        category.getCourses().add(this);
    }
    
    // Category remove karne ka helper method
    public void removeCategory(Category category) {
        categoryList.remove(category);
        category.getCourses().remove(this);
    }
}
```

## 5. DTOs (Data Transfer Objects)

DTOs are objects used for transferring data between processes, especially in API requests/responses:

```java
// CourseDto.java
package com.elearn.app.dtos;

import lombok.Data;
import java.util.Date;
import java.util.List;

@Data
public class CourseDto {
    private String id;
    private String title;
    private String shortDesc;
    private String longDesc;
    private double price;
    private boolean live;
    private double discount;
    private Date createdDate;
    private String banner;
    private List<String> videoIds;
    private List<String> categoryIds;
}
```

## 6. Creating Repositories

Repositories are interfaces that provide database operations:

```java
// CourseRepo.java
package com.elearn.app.repositories;

import com.elearn.app.entities.Course;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface CourseRepo extends JpaRepository<Course, String> {
    // Custom query method: title ya description mein keyword search karne ke liye
    List<Course> findByTitleContainingIgnoreCaseOrShortDescContainingIgnoreCase(
            String titleKeyword, String descKeyword);
    
    // Aur bhi custom queries add kar sakte hain
}
```

## 7. Creating Service Layer

Service layer contains all the business logic:

```java
// CourseService.java (Interface)
package com.elearn.app.services;

import com.elearn.app.dtos.CourseDto;
import com.elearn.app.dtos.ResourceContentType;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;
import java.util.List;

public interface CourseService {
    // Naya course create karne ke liye
    CourseDto createCourse(CourseDto courseDto);
    
    // Course update karne ke liye
    CourseDto updateCourse(String id, CourseDto courseDto);
    
    // Course ID se fetch karne ke liye
    CourseDto getCourseById(String id);
    
    // Saare courses paginated form mein fetch karne ke liye
    Page<CourseDto> getAllCourses(Pageable pageable);
    
    // Course delete karne ke liye
    void deleteCourse(String id);
    
    // Course search karne ke liye
    List<CourseDto> searchCourses(String keyword);
    
    // Course ka banner image save karne ke liye
    CourseDto saveBanner(MultipartFile file, String courseId) throws IOException;
    
    // Course ka banner fetch karne ke liye
    ResourceContentType getCourseBannerById(String courseId);
}
```

Implementation of service interface:

```java
// CourseServiceImpl.java
package com.elearn.app.services;

import com.elearn.app.config.AppConstants;
import com.elearn.app.dtos.CourseDto;
import com.elearn.app.dtos.ResourceContentType;
import com.elearn.app.entities.Course;
import com.elearn.app.exceptions.ResourceNotFoundException;
import com.elearn.app.repositories.CourseRepo;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Date;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@Service
public class CourseServiceImpl implements CourseService {

    private CourseRepo courseRepository;
    private ModelMapper modelMapper;
    private CategoryService categoryService;
    private FileService fileService;
    
    // Constructor injection for dependencies
    public CourseServiceImpl(CourseRepo courseRepository, ModelMapper modelMapper,
                             CategoryService categoryService, FileService fileService) {
        this.courseRepository = courseRepository;
        this.modelMapper = modelMapper;
        this.categoryService = categoryService;
        this.fileService = fileService;
    }
    
    @Override
    public CourseDto createCourse(CourseDto courseDto) {
        // UUID generate kar ke ID set karte hain
        courseDto.setId(UUID.randomUUID().toString());
        // Current date set karte hain
        courseDto.setCreatedDate(new Date());
        
        // DTO ko entity mein convert karte hain
        Course course = modelMapper.map(courseDto, Course.class);
        
        // Entity ko database mein save karte hain
        Course savedCourse = courseRepository.save(course);
        
        // Saved entity ko wapas DTO mein convert kar ke return karte hain
        return modelMapper.map(savedCourse, CourseDto.class);
    }
    
    @Override
    public CourseDto updateCourse(String id, CourseDto courseDto) {
        // ID se course fetch karte hain, agar nahi milta hai to exception throw karte hain
        Course course = courseRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + id));
        
        // DTO se updated data entity mein copy karte hain
        modelMapper.map(courseDto, course);
        
        // Updated entity ko save karte hain
        Course updatedCourse = courseRepository.save(course);
        
        // Updated entity ko DTO mein convert kar ke return karte hain
        return modelMapper.map(updatedCourse, CourseDto.class);
    }
    
    @Override
    public CourseDto getCourseById(String id) {
        // ID se course fetch karte hain, agar nahi milta hai to exception throw karte hain
        Course course = courseRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + id));
        
        // Entity ko DTO mein convert kar ke return karte hain
        return modelMapper.map(course, CourseDto.class);
    }
    
    @Override
    public Page<CourseDto> getAllCourses(Pageable pageable) {
        // Pageable parameters ke saath courses fetch karte hain
        Page<Course> coursesPage = courseRepository.findAll(pageable);
        
        // Courses ko DTOs mein convert karte hain
        List<CourseDto> courseDtos = coursesPage.getContent()
                .stream()
                .map(course -> modelMapper.map(course, CourseDto.class))
                .collect(Collectors.toList());
        
        // DTOs ka custom Page object create kar ke return karte hain
        return new PageImpl<>(courseDtos, pageable, coursesPage.getTotalElements());
    }
    
    @Override
    public void deleteCourse(String id) {
        // Course ko delete karte hain
        courseRepository.deleteById(id);
    }
    
    @Override
    public List<CourseDto> searchCourses(String keyword) {
        // Title ya description mein keyword search karte hain
        List<Course> courses = courseRepository
                .findByTitleContainingIgnoreCaseOrShortDescContainingIgnoreCase(keyword, keyword);
        
        // Courses ko DTOs mein convert kar ke return karte hain
        return courses.stream()
                .map(course -> modelMapper.map(course, CourseDto.class))
                .collect(Collectors.toList());
    }
    
    @Override
    public CourseDto saveBanner(MultipartFile file, String courseId) throws IOException {
        // Course fetch karte hain
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // File ko disk par save karte hain
        String filePath = fileService.save(file, AppConstants.COURSE_BANNER_UPLOAD_DIR, file.getOriginalFilename());
        
        // Course entity mein banner path aur content type set karte hain
        course.setBanner(filePath);
        course.setBannerContentType(file.getContentType());
        
        // Updated course ko save karte hain aur DTO mein convert kar ke return karte hain
        return modelMapper.map(courseRepository.save(course), CourseDto.class);
    }
    
    @Override
    public ResourceContentType getCourseBannerById(String courseId) {
        // Course fetch karte hain
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new ResourceNotFoundException("Course not found with id: " + courseId));
        
        // Banner path se Resource object create karte hain
        String bannerPath = course.getBanner();
        Path path = Paths.get(bannerPath);
        Resource resource = new FileSystemResource(path);
        
        // Resource aur content type ko wrapper object mein set kar ke return karte hain
        ResourceContentType resourceContentType = new ResourceContentType();
        resourceContentType.setResource(resource);
        resourceContentType.setContentType(course.getBannerContentType());
        return resourceContentType;
    }
}
```
---
Bahut accha sawaal hai! Chalo `ModelMapper` ke use se `CourseDto` ko `Course` entity mein kaise convert kiya jaa raha hai, usko bilkul beginner-friendly tareeke se samajhte hain — with **Hinglish comments**, **example**, aur **dry run** 🧠✨

---

## 🧾 Pehle samjho: DTO vs Entity

| DTO (`CourseDto`)                        | Entity (`Course`)                      |
|-----------------------------------------|----------------------------------------|
| API se data bhejne/lene ke liye hota hai | Database table ke saath link hota hai  |
| Simplified hota hai                     | Relations, JPA annotations ke saath hota hai |
| Client-facing object                    | Database-facing object                 |

---

## 💡 ModelMapper kya karta hai?

```java
Course course = modelMapper.map(courseDto, Course.class);
```

Yeh line ka simple matlab:

> "**courseDto** mein jitne bhi fields hain, unko **automatically** match karke ek **Course** entity bana do."

### ✨ Behind the scenes:

ModelMapper yeh karta hai:
- `courseDto.getId()` → `course.setId(...)`
- `courseDto.getTitle()` → `course.setTitle(...)`
- ...
- Agar names match karte hain, to wo values **automatically copy** ho jaati hain!

---

## 🔄 Dry Run Example

### Step 1: DTO bana lo

```java
CourseDto courseDto = new CourseDto();
courseDto.setId("123");
courseDto.setTitle("Java Bootcamp");
courseDto.setShortDesc("Learn Java from scratch");
courseDto.setLongDesc("Detailed Java course for beginners");
courseDto.setPrice(199.99);
courseDto.setLive(true);
courseDto.setDiscount(10.0);
courseDto.setCreatedDate(new Date());
courseDto.setBanner("java.jpg");
```

### Step 2: Convert using ModelMapper

```java
Course course = modelMapper.map(courseDto, Course.class);
```

Ab ModelMapper kya karega?
```
course.id              = "123"
course.title           = "Java Bootcamp"
course.shortDesc       = "Learn Java from scratch"
course.longDesc        = "Detailed Java course for beginners"
course.price           = 199.99
course.live            = true
course.discount        = 10.0
course.createdDate     = (current date)
course.banner          = "java.jpg"
```

### Step 3: Save to database

```java
Course savedCourse = courseRepository.save(course);
```

JPA automatically `INSERT` query chalayega aur DB mein save karega.

---

## 🧠 Important Notes

### ✅ Jab fields ka **naam aur type same** hota hai DTO aur Entity mein — tabhi ModelMapper easily kaam karta hai.

### ❌ Agar mismatch ho (jaise `List<String> categoryIds` vs `List<Category> categoryList`), tab manual mapping karni padti hai.

---

## 🎯 Pure Flow (Text Diagram)

```
Client Request --> CourseDto
                       |
                       v
        +----------------------------+
        | modelMapper.map(dto, entity)
        +----------------------------+
                       |
                       v
               Course (Entity)
                       |
                       v
       courseRepository.save(course)
                       |
                       v
         Data saved to DB (MySQL/Postgres)
```

---

## 🛠 Tips for Beginners

- **ModelMapper** is used to reduce boilerplate code like:
  ```java
  course.setTitle(courseDto.getTitle());
  course.setPrice(courseDto.getPrice());
  ...
  ```
- Bas DTO aur Entity ke field names aur types ko consistent rakho.
- Agar relations ho (jaise categories, videos), to unko manually map karna padega.

---

## 🧪 Bonus: DTO to Entity + Entity to DTO (Round Trip)

```java
Course course = modelMapper.map(courseDto, Course.class);  // DTO → Entity
Course savedCourse = courseRepository.save(course);        // Save to DB
return modelMapper.map(savedCourse, CourseDto.class);      // Entity → DTO
```

Yeh pattern commonly use hota hai controller/service layer mein.

---

## 8. Creating Controllers

Controllers handle HTTP requests and define API endpoints:

```java
// CourseController.java
package com.elearn.app.controllers;

import com.elearn.app.dtos.CourseDto;
import com.elearn.app.dtos.CustomMessage;
import com.elearn.app.dtos.ResourceContentType;
import com.elearn.app.services.CourseService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import org.springframework.core.io.Resource;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;
import java.util.List;

@RestController
@RequestMapping("/api/v1/courses")
public class CourseController {

    private CourseService courseService;
    
    // Constructor injection
    public CourseController(CourseService courseService) {
        this.courseService = courseService;
    }
    
    // CREATE: Naya course create karne ke liye POST endpoint
    @Operation(
        summary = "Create New Course",
        description = "Pass course data to create a new course"
    )
    @ApiResponse(responseCode = "201", description = "Course created successfully")
    @PostMapping
    public ResponseEntity<CourseDto> createCourse(@RequestBody CourseDto courseDto) {
        // CourseService ke through course create karte hain
        CourseDto createdCourse = courseService.createCourse(courseDto);
        // 201 Created status code ke saath response return karte hain
        return ResponseEntity.status(HttpStatus.CREATED).body(createdCourse);
    }
    
    // UPDATE: Course update karne ke liye PUT endpoint
    @Operation(
        summary = "Update Course",
        description = "Update an existing course by ID"
    )
    @PutMapping("/{id}")
    public ResponseEntity<CourseDto> updateCourse(
            @PathVariable String id, 
            @RequestBody CourseDto courseDto) {
        // CourseService ke through course update karte hain
        CourseDto updatedCourse = courseService.updateCourse(id, courseDto);
        // 200 OK status code ke saath response return karte hain
        return ResponseEntity.ok(updatedCourse);
    }
    
    // READ: ID se course fetch karne ke liye GET endpoint
    @GetMapping("/{id}")
    public ResponseEntity<CourseDto> getCourseById(@PathVariable String id) {
        // CourseService ke through course fetch karte hain
        CourseDto courseDto = courseService.getCourseById(id);
        // 200 OK status code ke saath response return karte hain
        return ResponseEntity.ok(courseDto);
    }
    
    // READ: Saare courses paginated form mein fetch karne ke liye GET endpoint
    @GetMapping
    public ResponseEntity<Page<CourseDto>> getAllCourses(Pageable pageable) {
        // CourseService ke through paginated courses fetch karte hain
        Page<CourseDto> courses = courseService.getAllCourses(pageable);
        // 200 OK status code ke saath response return karte hain
        return ResponseEntity.ok(courses);
    }
    
    // DELETE: Course delete karne ke liye DELETE endpoint
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteCourse(@PathVariable String id) {
        // CourseService ke through course delete karte hain
        courseService.deleteCourse(id);
        // 204 No Content status code ke saath response return karte hain
        return ResponseEntity.noContent().build();
    }
    
    // SEARCH: Courses search karne ke liye GET endpoint
    @GetMapping("/search")
    public ResponseEntity<List<CourseDto>> searchCourses(
            @RequestParam String keyword) {
        // CourseService ke through courses search karte hain
        List<CourseDto> courses = courseService.searchCourses(keyword);
        // 200 OK status code ke saath response return karte hain
        return ResponseEntity.ok(courses);
    }
    
    // UPLOAD: Course ka banner upload karne ke liye POST endpoint
    @PostMapping("/{courseId}/banners")
    public ResponseEntity<?> uploadBanner(
            @PathVariable String courseId,
            @RequestParam("banner") MultipartFile banner) throws IOException {
        
        // File validate karte hain
        String contentType = banner.getContentType();
        if (contentType == null) {
            contentType = "image/png";
        } else if (!contentType.equalsIgnoreCase("image/png") && 
                  !contentType.equalsIgnoreCase("image/jpeg")) {
            // Agar file PNG ya JPEG nahi hai, to error return karte hain
            CustomMessage customMessage = new CustomMessage();
            customMessage.setSuccess(false);
            customMessage.setMessage("Invalid file format. Only PNG and JPEG are allowed.");
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(customMessage);
        }
        
        // CourseService ke through banner save karte hain
        CourseDto courseDto = courseService.saveBanner(banner, courseId);
        // 200 OK status code ke saath response return karte hain
        return ResponseEntity.ok(courseDto);
    }
    
    // DOWNLOAD: Course ka banner download karne ke liye GET endpoint
    @GetMapping("/{courseId}/banners")
    public ResponseEntity<Resource> serverBanner(
            @PathVariable String courseId) {
        
        // CourseService ke through banner resource fetch karte hain
        ResourceContentType resourceContentType = courseService.getCourseBannerById(courseId);
        
        // Resource aur content type ke saath response return karte hain
        return ResponseEntity
                .ok()
                .contentType(MediaType.parseMediaType(resourceContentType.getContentType()))
                .body(resourceContentType.getResource());
    }
}
```

## 9. Exception Handling

Global exception handler for consistent error responses:

```java
// GlobalExceptionHandler.java
package com.elearn.app.exceptions;

import com.elearn.app.dtos.CustomMessage;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // ResourceNotFoundException handle karne ke liye
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<CustomMessage> handleResourceNotFoundException(ResourceNotFoundException ex) {
        CustomMessage response = new CustomMessage();
        response.setMessage(ex.getMessage());
        response.setSuccess(false);
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND);
    }
    
    // Generic exceptions handle karne ke liye
    @ExceptionHandler(Exception.class)
    public ResponseEntity<CustomMessage> handleGenericException(Exception ex) {
        CustomMessage response = new CustomMessage();
        response.setMessage("An error occurred: " + ex.getMessage());
        response.setSuccess(false);
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## 10. API Security with OAuth 2.0

Security ko implement karne ke liye:

```java
// SecurityConfig.java
package com.elearn.app.config.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
@EnableMethodSecurity
@EnableWebSecurity
public class SecurityConfig {

    private CustomConverter converter;
    
    public SecurityConfig(CustomConverter converter) {
        this.converter = converter;
    }
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // CORS configuration
        http.cors(cors -> {
            CorsConfiguration config = new CorsConfiguration();
            config.setAllowedOrigins(List.of("http://localhost:3000", "http://localhost:4200"));
            config.addAllowedMethod("*");
            config.addAllowedHeader("*");
            config.setAllowCredentials(true);
            
            UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            source.registerCorsConfiguration("/**", config);
            cors.configurationSource(source);
        });
        
        // CSRF disable karte hain (REST APIs ke liye typically disable hota hai)
        http.csrf().disable();
        
        // Request authorization rules
        http.authorizeHttpRequests()
            // Public endpoints
            .requestMatchers("/api/v1/auth/**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
            // GET endpoints ko GUEST aur ADMIN roles access kar sakte hain
            .requestMatchers(HttpMethod.GET, "/api/v1/**").hasAnyRole("GUEST", "ADMIN")
            // POST, PUT, DELETE endpoints ko sirf ADMIN role access kar sakta hai
            .requestMatchers(HttpMethod.POST, "/api/v1/**").hasRole("ADMIN")
            .requestMatchers(HttpMethod.PUT, "/api/v1/**").hasRole("ADMIN")
            .requestMatchers(HttpMethod.DELETE, "/api/v1/**").hasRole("ADMIN")
            // Baaki sab endpoints ko authentication ki zarurat hai
            .anyRequest().authenticated();
        
        // OAuth 2.0 Resource Server configuration
        http.oauth2ResourceServer()
            .jwt()
            .jwtAuthenticationConverter(jwtAuthConverter());
        
        return http.build();
    }
    
    @Bean
    public JwtAuthenticationConverter jwtAuthConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(this.converter);
        return converter;
    }
}
```

## 11. API Documentation with Swagger

Swagger UI se API documentation automatically generate hota hai:

```java
// Add to your application class or create a separate config class
package com.elearn.app.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI apiInfo() {
        return new OpenAPI()
            .info(new Info()
                .title("E-Learning Platform API")
                .description("API endpoints for managing courses, videos, and categories")
                .version("1.0.0")
                .license(new License().name("Apache 2.0").url("http://www.apache.org/licenses/LICENSE-2.0")));
    }
}
```

application.properties mein add karein:

```properties
# Swagger UI path
springdoc.swagger-ui.path=/swagger-ui.html
# API docs path
springdoc.api-docs.path=/v3/api-docs
```

## 12. File Upload/Download API

File operations ke liye service:

```java
// FileService.java
package com.elearn.app.services;

import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;

public interface FileService {
    // File ko specific directory mein save karne ke liye
    String save(MultipartFile file, String path, String fileName) throws IOException;
    
    // File ko fetch karne ke liye
    byte[] load(String path) throws IOException;
}

// FileServiceImpl.java
package com.elearn.app.services;

import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.UUID;

@Service
public class FileServiceImpl implements FileService {

    @Override
    public String save(MultipartFile file, String path, String fileName) throws IOException {
        // Filename ka extension extract karte hain
        String originalFilename = file.getOriginalFilename();
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        
        // Unique filename generate karte hain
        String randomFileName = UUID.randomUUID().toString() + extension;
        
        // Complete filepath create karte hain
        String filePath = path + File.separator + randomFileName;
        
        // Directory create karte hain agar exist nahi karta
        File directory = new File(path);
        if (!directory.exists()) {
            directory.mkdirs();
        }
        
        // File ko disk par save karte hain
        Files.copy(file.getInputStream(), Paths.get(filePath));
        
        return filePath;
    }

    @Override
    public byte[] load(String path) throws IOException {
        // File ko disk se read karte hain
        return Files.readAllBytes(Paths.get(path));
    }
}
```

Great question!

## 📦 What is a `MultipartFile`?

`MultipartFile` is a **Spring Framework interface** used to **handle file uploads** — especially when a client (like a browser or Postman) sends a file through an HTTP request.

---

## 🌐 Real-life Example:

Suppose you have a form like this on a website:

```html
<form method="POST" enctype="multipart/form-data" action="/upload">
  <input type="file" name="file" />
  <button type="submit">Upload</button>
</form>
```

The browser sends a `multipart/form-data` request which includes:
- File data
- Other form fields (if any)

To handle that in Spring Boot:

```java
@PostMapping("/upload")
public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
    String fileName = file.getOriginalFilename();
    long size = file.getSize();
    
    // Save to disk, database, cloud, etc.
    
    return ResponseEntity.ok("File uploaded: " + fileName + ", Size: " + size + " bytes");
}
```

---

## 🧠 Key Points:

| Feature                | Description |
|------------------------|-------------|
| `getOriginalFilename()` | Gets original file name |
| `getInputStream()`     | Access file content as stream |
| `getBytes()`           | Get file as byte array |
| `transferTo(File dest)`| Save file to disk |

---

## 🎯 When to Use

Use `MultipartFile` when:
- You want to accept file uploads from frontend (HTML forms, Postman, etc.)
- You’re building features like: profile image upload, document submission, etc.

---

## 🧪 Bonus Tip: Multiple Files Upload

```java
@PostMapping("/multi-upload")
public ResponseEntity<?> uploadMultiple(@RequestParam("files") MultipartFile[] files) {
    for (MultipartFile file : files) {
        // handle each file
    }
    return ResponseEntity.ok("Uploaded " + files.length + " files");
}
```

---


## 13. Pagination and Sorting

Pagination implementation:

```java
// Controller mein
@GetMapping
public ResponseEntity<Page<CourseDto>> getAllCourses(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "createdDate") String sortBy,
        @RequestParam(defaultValue = "desc") String direction) {
    
    // Sort direction determine karte hain
    Sort.Direction dir = direction.equalsIgnoreCase("asc") ? 
                        Sort.Direction.ASC : Sort.Direction.DESC;
    
    // Pageable object create karte hain
    Pageable pageable = PageRequest.of(page, size, Sort.by(dir, sortBy));
    
    // Service ke through paginated data fetch karte hain
    Page<CourseDto> courses = courseService.getAllCourses(pageable);
    
    return ResponseEntity.ok(courses);
}

// Service mein
@Override
public Page<CourseDto> getAllCourses(Pageable pageable) {
    Page<Course> coursesPage = courseRepository.findAll(pageable);
    
    List<CourseDto> courseDtos = coursesPage.getContent()
            .stream()
            .map(course -> modelMapper.map(course, CourseDto.class))
            .collect(Collectors.toList());
    
    return new PageImpl<>(courseDtos, pageable, coursesPage.getTotalElements());
}
```

Bahut badhiya sawaal Vaibhav! Chalo isko ekdum **beginner-friendly Hinglish** (Latin script) mein detail se samjhte hain – including:

1. `@RequestParam(...)` ko wrap karna
2. `Pageable`, `PageRequest.of(...)`, `Sort.by(...)` kya karte hain
3. Diagram + Real Example + Bonus Tips ✅

---

## 🧾 Ye Code Kya Kar Raha Hai?

```java
@GetMapping
public ResponseEntity<Page<CourseDto>> getAllCourses(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "createdDate") String sortBy,
    @RequestParam(defaultValue = "desc") String direction)
```

Yeh controller method client se **pagination** aur **sorting** ke liye query params accept karta hai.

### 🌐 URL Example:
```
GET /courses?page=1&size=5&sortBy=title&direction=asc
```

Agar koi param nahi bhejta, to default values use hongi:
- `page = 0` (pehla page)
- `size = 10` (10 items per page)
- `sortBy = createdDate`
- `direction = desc`

---

## 🔁 Can We Wrap These Params in a Single Global Param?

### ✅ YES! Use a Wrapper Class like `PaginationRequest`

```java
public class PaginationRequest {
    private int page = 0;
    private int size = 10;
    private String sortBy = "createdDate";
    private String direction = "desc";
    
    // Getters and setters
}
```

Then update your controller like this:

```java
@GetMapping
public ResponseEntity<Page<CourseDto>> getAllCourses(PaginationRequest request) {
    Sort.Direction dir = request.getDirection().equalsIgnoreCase("asc") ? 
                        Sort.Direction.ASC : Sort.Direction.DESC;

    Pageable pageable = PageRequest.of(request.getPage(), request.getSize(), Sort.by(dir, request.getSortBy()));

    Page<CourseDto> courses = courseService.getAllCourses(pageable);
    return ResponseEntity.ok(courses);
}
```

**✨ Cleaner Code + Easy to Maintain!**

---

## 📦 Pageable, PageRequest, Sort – Full Breakdown

### 🔹 `Pageable`
Ek **interface** hai jo page number, size, sort ka combination batata hai.
Used to **request paginated data** from DB/service.

```java
Pageable pageable = ...
```

### 🔹 `PageRequest.of(...)`
Factory method to create a `PageRequest` (which implements `Pageable`).

```java
PageRequest.of(page, size)
```

Adds sorting:

```java
PageRequest.of(page, size, Sort.by(Sort.Direction.DESC, "createdDate"))
```

### 🔹 `Sort.by(...)`
Sorting logic define karta hai:

```java
Sort.by(Sort.Direction.ASC, "title")
```

---

## 📊 Diagram

```
Client Request
     |
     | GET /courses?page=1&size=5&sortBy=title&direction=asc
     |
     v
[Controller]
     |
     v
 PageRequest.of(1, 5, Sort.by(ASC, "title"))
     |
     v
[Service Layer]
     |
     v
[Repository] → Automatically fetches paginated + sorted data
     |
     v
 Page<CourseDto> returned to client
```

---

## 🧪 Dry Run

**Request:**
```
GET /courses?page=1&size=2&sortBy=price&direction=asc
```

**Controller executes:**

```java
PageRequest.of(1, 2, Sort.by(Sort.Direction.ASC, "price"))
```

Means:
- 2 courses per page
- Start from **second page** (page index 1)
- Sorted by price ASCENDING

---

## 🧠 BONUS TIPS

✅ `Page<CourseDto>` object ke paas useful info hoti hai:
```java
courses.getContent();      // List<CourseDto>
courses.getTotalPages();   // Total pages
courses.getTotalElements();// Total records
courses.getNumber();       // Current page index
```

✅ Swagger mein `@ModelAttribute PaginationRequest` use karke params neatly dikh jaate hain.

---

Bahut badiya observation Vaibhav! 🔥  
Tumne bilkul sahi point uthaya — agar hum controller mein likhte hain:

```java
public ResponseEntity<Page<CourseDto>> getAllCourses(PaginationRequest request)
```

...to yeh `request` object mein values kaise aayengi agar `@RequestParam` use hi nahi ho raha?

---

## 🤔 Samasya (Problem)

Spring Boot by default sirf **query parameters** ko individual `@RequestParam` ke through map karta hai.  
Agar hum ek custom class (like `PaginationRequest`) use karte hain, to **usmein values tabhi bind hoti hain** jab:

> ✅ `@ModelAttribute` lagaya gaya ho – ya  
> ✅ class ke fields ke naam query parameters ke naam se match karte ho.

---

## ✅ Solution: Use `@ModelAttribute`

```java
@GetMapping
public ResponseEntity<Page<CourseDto>> getAllCourses(@ModelAttribute PaginationRequest request)
```

> `@ModelAttribute` tells Spring:  
> "**Jo query parameters aaye hain, unko is object ke fields mein map kar do.**"

### 🔹 URL Example:

```
GET /courses?page=1&size=5&sortBy=price&direction=asc
```

### 🔹 Mapping:

| Query Param | Class Field         |
|-------------|---------------------|
| `page=1`    | `request.page = 1`  |
| `size=5`    | `request.size = 5`  |
| ...         | ...                 |

Agar koi parameter missing hai, to `PaginationRequest` ke default value use hogi.

---

## 🔍 Full Working Example:

### 🔸 `PaginationRequest.java`

```java
public class PaginationRequest {
    private int page = 0;
    private int size = 10;
    private String sortBy = "createdDate";
    private String direction = "desc";

    // Getters and Setters
}
```

### 🔸 Controller

```java
@GetMapping("/courses")
public ResponseEntity<Page<CourseDto>> getAllCourses(@ModelAttribute PaginationRequest request) {
    Sort.Direction dir = request.getDirection().equalsIgnoreCase("asc") ? 
                         Sort.Direction.ASC : Sort.Direction.DESC;

    Pageable pageable = PageRequest.of(request.getPage(), request.getSize(), Sort.by(dir, request.getSortBy()));

    Page<CourseDto> courses = courseService.getAllCourses(pageable);

    return ResponseEntity.ok(courses);
}
```

---

## ✅ Output:

**URL:**
```
GET /courses?page=2&size=3&sortBy=price&direction=asc
```

**Mapped:**
```java
request.page     = 2
request.size     = 3
request.sortBy   = "price"
request.direction= "asc"
```

**Result:**
Paginated and sorted list of courses from page 2 with 3 items, sorted by price ASC.

---

## 💡 BONUS: Agar `@ModelAttribute` na bhi likho...

> Spring still tries to bind query parameters to object if it’s a **GET method** — but **explicitly** using `@ModelAttribute` is recommended for clarity and Swagger docs.

---




## 14. Interview Preparation

### Top REST API Interview Questions

1. **REST API kya hai?**
   - REST (Representational State Transfer) ek architectural style hai jo web services ke design ke liye use hota hai. REST APIs stateless hote hain, resources ko URLs ke through expose karte hain, aur HTTP methods (GET, POST, PUT, DELETE) ka use karte hain.

2. **REST ki main principles kya hain?**
   - Client-Server architecture
   - Stateless communication
   - Cacheable responses
   - Uniform Interface
   - Layered System
   - Code on Demand (optional)

3. **HTTP Methods ki roles kya hain?**
   - GET: Data retrieve karna
   - POST: New resource create karna
   - PUT/PATCH: Existing resource update karna
   - DELETE: Resource delete karna

4. **RESTful API mein status codes ka kya significance hai?**
   - 200 OK: Request successful tha
   - 201 Created: Resource successfully create ho gaya
   - 400 Bad Request: Client request mein error hai
   - 401 Unauthorized: Authentication required hai
   - 403 Forbidden: Authenticated hai lekin permission nahi hai
   - 404 Not Found: Resource nahi mila
   - 500 Internal Server Error: Server mein error hai

5. **@RequestBody aur @ResponseBody annotations ka kya use hai?**
   - @RequestBody: HTTP request body ko Java object mein convert karta hai
   - @ResponseBody: Method ke return value ko HTTP response body mein convert karta hai

6. **ResponseEntity class ka kya use hai?**
   - Complete HTTP response (status code, headers, body) ko wrap karne ke liye use hota hai

7. **@PathVariable aur @RequestParam mein kya difference hai?**
   - @PathVariable: URL path se values extract karta hai (e.g., /users/{id})
   - @RequestParam: Query parameters se values extract karta hai (e.g., /users?id=123)

8. **Spring Boot mein exception handling kaise karte hain?**
   - @ExceptionHandler: Specific exceptions handle karne ke liye
   - @ControllerAdvice/@RestControllerAdvice: Global exception handling ke liye
   - ResponseEntityExceptionHandler: Common exceptions handle karne ke liye

9. **API versioning kyu aur kaise karte hain?**
   - URL versioning (/api/v1/users)
   - Request parameter versioning (/api/users?version=1)
   - Header versioning (X-API-Version: 1)
   - Media type versioning (Accept: application/vnd.company.v1+json)

10. **Spring Security mein OAuth2 kaise implement karte hain?**
    - Dependencies add karein
    - ResourceServer configure karein
    - JwtAuthenticationConverter customize karein
    - Endpoints ko secure karein based on roles/scopes

11. **DTO (Data Transfer Object) ka kya purpose hai?**
    - Client aur server ke beech data transfer ke liye specialized objects
    - Entity classes se alag hote hain, security ke liye sensitive data hide karte hain
    - Multiple entities ko ek response mein combine kar sakte hain

12. **API pagination kyu important hai aur kaise implement karte hain?**
    - Large datasets ko handle karne ke liye important hai
    - Spring Data JPA Pageable interface ka use karte hain
    - Page size, number, aur sorting options provide karte hain

13. **API rate limiting kyu aur kaise implement karte hain?**
    - Server ko overload hone se bachane ke liye
    - Client ka fair usage ensure karne ke liye
    - Spring Cloud Gateway, Bucket4j, ya third-party solutions use kar sakte hain

14. **API documentation kaise generate karte hain?**
    - SpringDoc/OpenAPI/Swagger use karke
    - Controller methods par annotations add karke
    - API documentation automatically generate hoti hai

15. **REST API testing kaise karte hain?**
    - Unit tests: Controller methods ke liye MockMvc use karke
    - Integration tests: TestRestTemplate ya RestAssured libraries use karke
    - End-to-end tests: Postman ya other API testing tools use karke

## 15. Spring Boot API Project Dry Run

Aaiye hmare code ko step-by-step execute karte hain:

1. **Application Start**
   ```
   $ mvn spring-boot:run
   ```
   - Spring Boot application start hoga
   - Embedded Tomcat server port 8081 par start hoga
   - Database connection establish hoga
   - JPA entities se tables create/update honge

2. **Course Create API Call**
   ```
   POST http://localhost:8081/api/v1/courses
   
   {
     "title": "Spring Boot Complete Guide",
     "shortDesc": "Learn Spring Boot from scratch",
     "longDesc": "Comprehensive guide to Spring Boot application development",
     "price": 5999,
     "discount": 10,
     "live": true
   }
   ```
   
   **Process Flow**:
   - Request CourseController.createCourse() method par jayega
   - Method CourseDto object receive karega via @RequestBody
   - Controller CourseService.createCourse() ko call karega
   - Service UUID generate karega, current date set karega
   - ModelMapper CourseDto ko Course entity mein convert karega
   - Repository Course entity ko database mein save karega
   - Service Course ko wapas CourseDto mein convert karega
   - Controller 201 Created status code ke saath response send karega

3. **Course Get API Call**
   ```
   GET http://localhost:8081/api/v1/courses/{id}
   ```
   
   **Process Flow**:
   - Request CourseController.getCourseById() method par jayega
   - Method path variable ID extract karega
   - Controller CourseService.getCourseById() ko call karega
   - Service Repository.findById() ko call karega
   - Repository database se Course fetch karega
   - Agar course nahi milta, ResourceNotFoundException throw hoga
   - Service Course ko CourseDto mein convert karega
   - Controller 200 OK status code ke saath response send karega

4. **Course Update API Call**
   ```
   PUT http://localhost:8081/api/v1/courses/{id}
   
   {
     "title": "Updated Course Title",
     "price": 4999
   }
   ```
   
   **Process Flow**:
   - Request CourseController.updateCourse() method par jayega
   - Method path variable ID aur request body extract karega
   - Controller CourseService.updateCourse() ko call karega
   - Service Repository.findById() se course fetch karega
   - ModelMapper updated fields ko entity mein map karega
   - Repository updated entity ko save karega
   - Service updated Course ko CourseDto mein convert karega
   - Controller 200 OK status code ke saath response send karega

5. **Course Delete API Call**
   ```
   DELETE http://localhost:8081/api/v1/courses/{id}
   ```
   
   **Process Flow**:
   - Request CourseController.deleteCourse() method par jayega
   - Method path variable ID extract karega
   - Controller CourseService.deleteCourse() ko call karega
   - Service Repository.deleteById() ko call karega
   - Repository course ko database se delete karega
   - Controller 204 No Content status code ke saath response send karega

6. **Course Search API Call**
   ```
   GET http://localhost:8081/api/v1/courses/search?keyword=Spring
   ```
   
   **Process Flow**:
   - Request CourseController.searchCourses() method par jayega
   - Method query parameter 'keyword' extract karega
   - Controller CourseService.searchCourses() ko call karega
   - Service Repository.findByTitleContaining...() ko call karega
   - Repository database se matching courses fetch karega
   - Service Courses ko CourseDtos mein convert karega
   - Controller 200 OK status code ke saath response send karega

7. **Course Banner Upload API Call**
   ```
   POST http://localhost:8081/api/v1/courses/{id}/banners
   Content-Type: multipart/form-data
   
   banner=@file.jpg
   ```
   
   **Process Flow**:
   - Request CourseController.uploadBanner() method par jayega
   - Method path variable ID aur MultipartFile extract karega
   - Content type validate hoga
   - Controller CourseService.saveBanner() ko call karega
   - FileService file ko disk par save karega
   - Course entity mein file path aur content type update hoga
   - Repository updated course ko save karega
   - Controller 200 OK status code ke saath response send karega

8. **Course Banner Download API Call**
   ```
   GET http://localhost:8081/api/v1/courses/{id}/banners
   ```
   
   **Process Flow**:
   - Request CourseController.serverBanner() method par jayega
   - Method path variable ID extract karega
   - Controller CourseService.getCourseBannerById() ko call karega
   - Service Course fetch karke banner path extract karega
   - FileSystemResource create hoga
   - Controller content-type header set karke Resource return karega

## Conclusion

Spring Boot se REST API development karne ke liye yeh sab components important hain:

1. **Entity Classes**: Database tables ko represent karte hain
2. **DTO Classes**: API requests/responses ko represent karte hain
3. **Repository Interfaces**: Database operations provide karte hain
4. **Service Classes**: Business logic implement karte hain
5. **Controller Classes**: API endpoints define karte hain
6. **Exception Handling**: Errors ko properly handle karte hain
7. **Security Configuration**: API ko secure karte hain

In sab components ko proper tarike se implement karke, aap ek robust, secure, aur scalable REST API develop kar sakte hain.

Spring Boot ke baare mein aur jaankaari ke liye official Spring documentation ya baaki online resources explore karein.
