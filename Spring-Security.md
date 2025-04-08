# Spring Security Complete Guide for Beginners

## 1. Introduction to Spring Security

Spring Security ek powerful aur highly customizable authentication and access-control framework hai jo Spring applications ka security handle karta hai. Ye aapko different types ke security mechanisms implement karne ki capability deta hai.

### 1.1 Spring Security ke Main Components

- **Authentication** - User ko verify karne ka process (kaun hai wo)
- **Authorization** - User ke permissions check karne ka process (kya kar sakta hai wo)
- **Protection against attacks** - CSRF, session fixation, clickjacking, etc.

## 2. Spring Security Architecture

### 2.1 Core Architectural Components

**SecurityFilterChain**
- Request aane par ye filters ki ek chain create karta hai
- Har filter specific security concern ko handle karta hai

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
    // Customization
    httpSecurity.authorizeHttpRequests(auth ->
            auth
                .requestMatchers("/api/v1/auth/login").permitAll()
                .requestMatchers(HttpMethod.POST,"/api/v1/users").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/courses/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/categories/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/videos/**").permitAll()
                .requestMatchers("/api/v1/courses/**").hasRole("ADMIN")
                .anyRequest()
                .authenticated());
                
    // Other configurations
    return httpSecurity.build();
}
```

**Authentication Manager**
- UserDetailsService aur PasswordEncoder ke through credentials verify karta hai
- Successfully validation par Authentication object return karta hai

```java
@Bean
public AuthenticationManager authenticationManager(
        AuthenticationConfiguration configuration
) throws Exception {
    return configuration.getAuthenticationManager();
}
```

**UserDetailsService**
- Database ya kisi bhi source se user information fetch karta hai
- UserDetails object return karta hai

```java
@Service
public class CustomUserDetailService implements UserDetailsService {
    private UserRepo userRepo;

    public CustomUserDetailService(UserRepo userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepo.findByEmail(username).orElseThrow(() -> 
            new BadCredentialsException("User not found in database !!"));
        return new CustomUserDetail(user);
    }
}
```

**Security Context**
- Current user ki information store karta hai
- Thread local variable ke tarah work karta hai

## 3. Authentication Implementation

### 3.1 Basic Configuration

Spring Security ko configure karne ke liye `SecurityConfig` class banani hoti hai:

```java
@Configuration
@EnableMethodSecurity(prePostEnabled = true)
@EnableWebSecurity
public class SecurityConfig {
    // Beans and configuration methods
}
```

### 3.2 Password Encoding

Passwords ko securely store karne ke liye:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### 3.3 JWT Authentication Flow

**Step 1: JWT Token Generate Karna**

`JwtUtil` class main token generation logic:

```java
@Component
public class JwtUtil {
    private Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256);
    private long jwtExpiration = 5 * 60 * 1000; // 5 minutes

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
                .signWith(key)
                .compact();
    }
    
    // Other methods for token validation and extraction
}
```

**Step 2: Login API Implement Karna**

```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
    private AuthenticationManager manager;
    private UserDetailsService userDetailsService;
    private JwtUtil util;

    @PostMapping("/login")
    public ResponseEntity<?> createToken(@RequestBody LoginRequest loginRequest) {
        try {
            // Authenticate user using AuthenticationManager
            UsernamePasswordAuthenticationToken authentication = 
                new UsernamePasswordAuthenticationToken(
                    loginRequest.getEmail(), 
                    loginRequest.getPassword()
                );
            Authentication authenticated = manager.authenticate(authentication);
        } catch (AuthenticationException ex) {
            throw new BadCredentialsException("Incorrect email or password !!");
        }

        // Generate token for authenticated user
        CustomUserDetail userDetails = 
            (CustomUserDetail) userDetailsService.loadUserByUsername(loginRequest.getEmail());
        String token = util.generateToken(userDetails.getUsername());

        // Return token and user details in response
        JwtResponse response = JwtResponse.builder()
                .token(token)
                .user(modelMapper.map(userDetails.getUser(), UserDto.class))
                .build();
        return ResponseEntity.ok(response);
    }
}
```

**Step 3: JWT Filter Implement Karna**

Har request ke sath token validate karne ke liye:

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Autowired
    private JwtUtil util;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                  HttpServletResponse response,
                                  FilterChain filterChain) throws ServletException, IOException {
        // Extract token from Authorization header
        String authorizationHeader = request.getHeader("Authorization");
        String username = null;
        String jwtToken = null;
        
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer")) {
            jwtToken = authorizationHeader.substring(7);
            username = util.extractUsername(jwtToken);

            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                // Validate token
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (util.validateToken(jwtToken, userDetails.getUsername())) {
                    // Set authentication in security context
                    UsernamePasswordAuthenticationToken authenticationToken = 
                        new UsernamePasswordAuthenticationToken(
                            userDetails, 
                            null, 
                            userDetails.getAuthorities()
                        );
                    authenticationToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                    );
                    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                }
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

**Step 4: Security Config mein Filter Add Karna**

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
    // Other configurations
    
    httpSecurity.sessionManagement(
        e -> e.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    );
    httpSecurity.addFilterBefore(
        jwtAuthenticationFilter, 
        UsernamePasswordAuthenticationFilter.class
    );
    
    return httpSecurity.build();
}
```

## 4. Authorization Implementation

### 4.1 URL Based Authorization

```java
httpSecurity.authorizeHttpRequests(auth ->
    auth
        .requestMatchers("/api/v1/auth/login").permitAll()
        .requestMatchers(HttpMethod.POST,"/api/v1/users").permitAll()
        .requestMatchers(HttpMethod.GET, "/api/v1/courses/**").permitAll()
        .requestMatchers("/api/v1/courses/**").hasRole("ADMIN")
        .requestMatchers("/api/v1/categories/**").hasRole("ADMIN")
        .requestMatchers("/api/v1/users/**").hasRole("ADMIN")
        .requestMatchers("/api/v1/videos/**").hasRole("ADMIN")
        .anyRequest()
        .authenticated());
```

### 4.2 Method Level Security

`@EnableMethodSecurity` annotation ke sath enable hota hai:

```java
// Controller or Service class mein
@PreAuthorize("hasRole('ADMIN')")
public void someAdminMethod() {
    // Only ADMIN can access
}

@PreAuthorize("hasAnyRole('ADMIN', 'USER')")
public void someUserMethod() {
    // ADMIN and USER can access
}
```

## 5. Exception Handling

### 5.1 Custom Authentication Entry Point

Unauthorized access ke case mein custom response:

```java
@Component
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request,
                       HttpServletResponse response,
                       AuthenticationException authException)
        throws IOException, ServletException {
        
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        CustomMessage customMessage = new CustomMessage();
        customMessage.setMessage(authException.getMessage());
        customMessage.setSuccess(false);

        ObjectMapper objectMapper = new ObjectMapper();
        String jsonString = objectMapper.writeValueAsString(customMessage);
        PrintWriter writer = response.getWriter();
        writer.println(jsonString);
    }
}
```

### 5.2 Access Denied Handler

```java
httpSecurity.exceptionHandling(e ->
    e.authenticationEntryPoint(authenticationEntryPoint)
     .accessDeniedHandler((request, response, accessDeniedException) -> {
         response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
         response.setContentType(MediaType.APPLICATION_JSON_VALUE);
         CustomMessage customMessage = new CustomMessage();
         customMessage.setMessage("You dont have permission to perform this operations !! " + 
                                 accessDeniedException.getMessage());
         customMessage.setSuccess(false);
         String stringMessage = new ObjectMapper().writeValueAsString(customMessage);
         response.getWriter().println(stringMessage);
     })
);
```

## 6. CORS Configuration

Cross-Origin Resource Sharing (CORS) settings:

```java
httpSecurity.cors(cor -> {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:5173", "http://localhost:4200"));
    config.addAllowedMethod("*");
    config.addAllowedHeader("*");
    config.setAllowCredentials(true);
    config.setMaxAge(300L);
    UrlBasedCorsConfigurationSource configurationSource = 
        new UrlBasedCorsConfigurationSource();
    configurationSource.registerCorsConfiguration("/**", config);

    cor.configurationSource(configurationSource);
});
```

## 7. Security Best Practices

### 7.1 Password Storage

- Passwords ko always encrypted form mein store karein
- BCryptPasswordEncoder use karein, MD5 aur SHA-1 outdated hain

### 7.2 JWT Token Security

- Token expiration time appropriate set karein
- Secret key ko secure rakhen
- HTTPS use karein token transfer ke liye

### 7.3 CSRF Protection

Modern single-page applications mein:

```java
httpSecurity.csrf(AbstractHttpConfigurer::disable);
```

Traditional web applications mein CSRF enable rakhna better hai.

## 8. Spring Security Dry Run

Let's understand how Spring Security flow works with a practical example:

### 8.1 User Login Request

1. User `POST /api/v1/auth/login` endpoint par username/password bhejta hai
2. `AuthController` mein request aati hai
3. `AuthenticationManager` credentials validate karta hai:
   - `CustomUserDetailService` database se user load karta hai
   - `PasswordEncoder` password match karta hai
4. Authentication successful hone par:
   - `JwtUtil` user ke liye token generate karta hai
   - Response mein token return hota hai

### 8.2 Protected Resource Access

1. User token ke sath `/api/v1/courses` par request bhejta hai
2. `JwtAuthenticationFilter` request intercept karta hai:
   - Token extract karta hai
   - Token validate karta hai
   - User load karta hai
   - Authentication object `SecurityContext` mein set karta hai
3. Filter ke baad, request `CourseController` tak pahunchti hai
4. Method execution se pehle authorization check hota hai
5. Authorized hone par, controller method execute hota hai aur response return hota hai

## 9. Interview Q&A on Spring Security

**Q: Spring Security kya hai aur kyun use karte hain?**  
A: Spring Security ek robust authentication and authorization framework hai jo Spring applications ko secure banane ke liye use hota hai. Ye ready-to-use components provide karta hai jisse hum authentication, authorization, aur common security vulnerabilities se protection implement kar sakte hain.

**Q: Authentication aur Authorization mein kya difference hai?**  
A: Authentication verify karta hai ki user kaun hai (identity verification), jabki Authorization determine karta hai ki user kya access kar sakta hai (permission verification).

**Q: JWT (JSON Web Token) kya hai aur kaise kaam karta hai?**  
A: JWT ek compact, self-contained way hai information ko securely parties ke beech transmit karne ka. Ye 3 parts mein hota hai: header, payload, aur signature. JWT stateless authentication provide karta hai jahan token user ki identity aur claims contain karta hai.

**Q: Spring Security mein SecurityContext kya hai?**  
A: SecurityContext current user ka authentication information store karta hai. Ye ThreadLocal implementation use karta hai jisse har thread ke liye separate security information maintain hoti hai.

**Q: Spring Security mein Filter Chain kaise kaam karti hai?**  
A: Spring Security ka core concept Filter Chain hai. HTTP requests ko process karne ke liye multiple filters daisy chain pattern mein arranged hote hain. Har filter specific security function perform karta hai, jaise authentication, authorization, CSRF prevention, etc.

**Q: Spring Security mein @PreAuthorize ka kya use hai?**  
A: @PreAuthorize annotation method level security provide karta hai. Ye SpEL (Spring Expression Language) expressions accept karta hai jo method invocation se pehle evaluate hote hain, permissions check karne ke liye.

**Q: Password encoding kyun important hai aur kaunse encoders available hain?**  
A: Password encoding ensures ki passwords plaintext mein store nahi hote, security breach ke case mein data secure rahta hai. Spring Security multiple encoders provide karta hai jaise BCryptPasswordEncoder, Pbkdf2PasswordEncoder, SCryptPasswordEncoder, etc.

**Q: CSRF attacks kya hain aur Spring Security inmein kaise protect karta hai?**  
A: CSRF (Cross-Site Request Forgery) ek attack hai jahan malicious website victim ke browser ko force karta hai authenticated requests send karne ke liye. Spring Security by default CSRF protection enable karta hai, unique tokens generate karke jo har form submission ke sath validate hote hain.

**Q: Spring Security mein CORS configuration kyun important hai?**  
A: CORS (Cross-Origin Resource Sharing) browser security feature hai jo different origins ke resources access ko restrict karta hai. Spring applications mein frontend aur backend alag domains par host hone par CORS configuration zaruri hai.

**Q: Stateless authentication kya hai aur kaise implement karte hain?**  
A: Stateless authentication mein server koi session state maintain nahi karta. Har request self-contained hoti hai with all necessary authentication info. JWT tokens stateless authentication implement karne ka common way hai. Spring Security mein `SessionCreationPolicy.STATELESS` set karke implement karte hain.

**Q: OAuth2 kya hai aur Spring Security mein kaise implement karte hain?**  
A: OAuth2 ek authorization framework hai jo third-party applications ko limited access deta hai user accounts tak. Spring Security OAuth2 support provide karta hai through Spring Security OAuth2 module, jisse hum resource server aur authorization server implement kar sakte hain.

## 10. Conclusion

Spring Security ek powerful framework hai jo authentication, authorization aur multiple security concerns handle karta hai. Is guide mein humne dekha ki JWT authentication kaise implement karte hain, URL aur method level authorization kaise configure karte hain, aur custom exception handling kaise add karte hain.

Yaad rakhen ki security ek continuous process hai, aur best practices follow karna important hai. Regular updates aur security vulnerabilities ke against testing ensure karein.

## References

1. Spring Security Documentation
2. OWASP Security Guidelines
3. JWT Authentication Best Practices

----


# Spring Security Flow in Start-Learn-Back Project with Hinglish Comments

## 1. Overall Architecture Overview

Spring Security ko project mein implement karne ke liye following components use hue hain:

1. **SecurityConfig**: Main configuration class jo security ke rules set karti hai
2. **JwtAuthenticationFilter**: JWT token ko validate aur authenticate karne ke liye filter
3. **JwtUtil**: JWT token generate, validate aur extract karne ke liye utility class
4. **CustomAuthenticationEntryPoint**: Authentication failure par response customize karne ke liye
5. **CustomUserDetailService**: Database se user ko load karne ke liye
6. **CustomUserDetail**: User entity ko Spring Security ke UserDetails mein convert karne ke liye
7. **AuthController**: Login API provide karta hai token generate karne ke liye

## 2. SecurityConfig.java File (Main Configuration)

```java
@Configuration  // Spring ko batata hai ki ye configuration class hai
@EnableMethodSecurity(prePostEnabled = true)  // Method level security enable karta hai (@PreAuthorize, etc.)
@EnableWebSecurity(debug = true)  // Web security enable karta hai aur debug mode on karta hai
public class SecurityConfig {

    private AuthenticationEntryPoint authenticationEntryPoint;
    private JwtAuthenticationFilter authenticationFilter;

    // Constructor injection - dependency injection ke through components ko inject karta hai
    public SecurityConfig(AuthenticationEntryPoint authenticationEntryPoint, JwtAuthenticationFilter authenticationFilter) {
        this.authenticationEntryPoint = authenticationEntryPoint;
        this.authenticationFilter = authenticationFilter;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();  // Password ko encode karne ke liye BCrypt algorithm use kar rahe hain
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration configuration
    ) throws Exception {
        return configuration.getAuthenticationManager();  // Authentication manager create karta hai jo username/password verify karega
    }

    // In-memory users - ye comment out hai but dikha raha hai ki kaise in-memory users create kar sakte hain testing ke liye
    /*
    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
        userDetailsManager.createUser(
                User.withDefaultPasswordEncoder()
                        .username("ram")
                        .password("ram")
                        .roles("ADMIN")
                        .build()
        );
        userDetailsManager.createUser(
                User.withDefaultPasswordEncoder()
                        .username("shyam")
                        .password("ram")
                        .roles("USER")
                        .build()
        );
        return userDetailsManager;
    }
    */

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        // CORS configuration - Cross Origin Resource Sharing
        httpSecurity.cors(cor -> {
            CorsConfiguration config = new CorsConfiguration();
            // Frontend URLs jo allow kiye gaye hain
            config.setAllowedOrigins(List.of("http://localhost:5173", "http://localhost:4200"));
            config.addAllowedMethod("*");  // Sare HTTP methods allow karta hai (GET, POST, etc.)
            config.addAllowedHeader("*");  // Sare headers allow karta hai
            config.setAllowCredentials(true);  // Credentials bhejne ki permission deta hai
            config.setMaxAge(300L);  // Preflight requests ka cache time
            UrlBasedCorsConfigurationSource configurationSource = new UrlBasedCorsConfigurationSource();
            configurationSource.registerCorsConfiguration("/**", config);
            cor.configurationSource(configurationSource);
        });
        
        // CSRF disable kiya hai - REST APIs mein common hai kyunki tokens use hote hain
        httpSecurity.csrf(AbstractHttpConfigurer::disable);

        // Authorization rules - kon sa URL kis role ke liye accessible hai
        httpSecurity.authorizeHttpRequests(auth ->
                auth
                        // Swagger UI aur OpenAPI docs publicly accessible hain
                        .requestMatchers("/doc.html", "/v3/api-docs/**", "/swagger-ui/**", "/swagger-resources/**").permitAll()
                        // Login API aur user creation API public hai
                        .requestMatchers("/api/v1/auth/login").permitAll()
                        .requestMatchers(HttpMethod.POST,"/api/v1/users").permitAll()
                        // GET requests for courses, categories and videos permitted for all
                        .requestMatchers(HttpMethod.GET, "/api/v1/courses/**").permitAll()
                        .requestMatchers(HttpMethod.GET, "/api/v1/categories/**").permitAll()
                        .requestMatchers(HttpMethod.GET, "/api/v1/videos/**").permitAll()
                        // Admin role ko hi modification access hai
                        .requestMatchers("/api/v1/courses/**").hasRole("ADMIN")
                        .requestMatchers("/api/v1/categories/**").hasRole("ADMIN")
                        .requestMatchers("/api/v1/users/**").hasRole("ADMIN")
                        .requestMatchers("/api/v1/videos/**").hasRole("ADMIN")
                        // Baki sare requests ke liye authentication jaruri hai
                        .anyRequest()
                        .authenticated());

        // Session management - STATELESS means no session will be created or used
        httpSecurity.sessionManagement(e -> e.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        
        // JWT filter ko UsernamePasswordAuthenticationFilter se pehle add karte hain
        httpSecurity.addFilterBefore(authenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        // Exception handling - authentication failures aur access denied cases ke liye
        httpSecurity.exceptionHandling(e ->
                e.authenticationEntryPoint(authenticationEntryPoint)  // Unauthorized access par invoke hota hai
                        .accessDeniedHandler((request, response, accessDeniedException) -> {
                            // Access denied handler - jab user authenticated hai par authorized nahi hai
                            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                            CustomMessage customMessage = new CustomMessage();
                            customMessage.setMessage("You dont have permission to perform this operations !! " + accessDeniedException.getMessage());
                            customMessage.setSuccess(false);
                            String stringMessage = new ObjectMapper().writeValueAsString(customMessage);
                            response.getWriter().println(stringMessage);
                        })
        );

        // Form login configuration example (commented out) - for traditional web apps
        /*
        httpSecurity.formLogin(
                form ->
                {
                    form.loginPage("/client-login");
                    form.usernameParameter("username");
                    form.passwordParameter("userpassword");
                    form.loginProcessingUrl("/client-login-process");
                    form.successForwardUrl("/success");
                }
        );
        */

        return httpSecurity.build();  // Final security configuration build karke return karta hai
    }
}
```

## 3. JwtAuthenticationFilter.java

```java
@Component  // Spring component hai, automatically detect hoga
public class JwtAuthenticationFilter extends OncePerRequestFilter {  // Har request ke liye ek baar hi chalega

    @Autowired
    private JwtUtil util;  // JWT utilities inject karta hai

    @Autowired
    private UserDetailsService userDetailsService;  // User details service inject karta hai

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        // Authorization header se token extract karna
        String authorizationHeader = request.getHeader("Authorization");
        System.out.println("Header : " + authorizationHeader);
        
        String username = null;
        String jwtToken = null;
        
        // Check karta hai ki Authorization header exist karta hai aur "Bearer" se start hota hai
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer")) {
            // "Bearer " ke baad wala part token hai
            jwtToken = authorizationHeader.substring(7);
            // Token se username extract karta hai
            username = util.extractUsername(jwtToken);

            // Agar username mil gaya aur abhi tak authenticate nahi hai
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                // User details load karta hai database se
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                // Token validate karta hai
                if (util.validateToken(jwtToken, userDetails.getUsername())) {
                    // Valid token ki case mein authentication object create karta hai
                    UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(
                            userDetails, null, userDetails.getAuthorities());
                    
                    // Request details set karta hai authentication object mein
                    authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    
                    // Security context mein authentication set karta hai - isse user authenticated ho jata hai
                    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                }
            }
        }
        
        // Request ko next filter mein forward karta hai
        filterChain.doFilter(request, response);
    }
}
```

## 4. JwtUtil.java

```java
@Component  // Spring component hai
public class JwtUtil {

    // Secret key generate karta hai JWT sign karne ke liye
    private Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256);

    // Token expiration time - 5 minutes
    private long jwtExpiration = 5 * 60 * 1000;

    // Token se username extract karta hai
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);  // JWT subject field mein username store hota hai
    }

    // Generic method jo kisi bhi claim ko extract karne ke liye use ho sakta hai
    public <T> T extractClaim(String token, Function<Claims, T> claimResolver) {
        final Claims claims = extractAllClaims(token);
        return claimResolver.apply(claims);
    }

    // Token se sare claims extract karta hai
    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(key)  // Secret key use karta hai verify karne ke liye
                .build()
                .parseClaimsJws(token).getBody();  // Token ko parse karta hai aur claims return karta hai
    }

    // Username se token generate karta hai
    public String generateToken(String username) {
        return createToken(username);
    }

    // Actual token creation logic
    private String createToken(String username) {
        return Jwts.builder()
                .setSubject(username)  // Username ko subject field mein set karta hai
                .setIssuedAt(new Date(System.currentTimeMillis()))  // Current time ko issued at mein set karta hai
                .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))  // Expiration time set karta hai
                .signWith(key)  // Secret key se sign karta hai
                .compact();  // Final token string generate karta hai
    }

    // Token validate karta hai - check karta hai ki token valid hai aur expire nahi hua hai
    public Boolean validateToken(String token, String username) {
        String tokenUsername = extractUsername(token);
        return (username.equals(tokenUsername) && !isTokenExpired(token));
    }

    // Check karta hai ki token expire hua hai ya nahi
    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    // Token se expiration date extract karta hai
    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
}
```

## 5. CustomAuthenticationEntryPoint.java

```java
@Component  // Spring component hai
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException authException)
            throws IOException, ServletException {

        // Authentication failure par ye method call hota hai
        
        // Response setup - JSON format mein response bhejne ke liye
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);  // 401 status code

        // Custom error message create karta hai
        CustomMessage customMessage = new CustomMessage();
        customMessage.setMessage(authException.getMessage());
        customMessage.setSuccess(false);

        // JSON string mein convert karta hai response ko
        ObjectMapper objectMapper = new ObjectMapper();
        String jsonString = objectMapper.writeValueAsString(customMessage);
        
        // JSON response client ko bhejta hai
        PrintWriter writer = response.getWriter();
        writer.println(jsonString);
    }
}
```

## 6. CustomUserDetailService.java

```java
@Service  // Ye service layer component hai
public class CustomUserDetailService implements UserDetailsService {

    private UserRepo userRepo;  // User repository inject karta hai

    // Constructor injection
    public CustomUserDetailService(UserRepo userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Database se user fetch karta hai email ke based pe (email ko username ki tarah use kar rahe hain)
        User user = userRepo.findByEmail(username).orElseThrow(() -> 
            new BadCredentialsException("User not found in database !!"));
            
        System.out.println("loading user form db" + user.getEmail());

        // User entity ko UserDetails interface implement karne wale CustomUserDetail object mein convert karta hai
        return new CustomUserDetail(user);
    }
}
```

## 7. CustomUserDetail.java

```java
// UserDetails interface implement karta hai jo Spring Security ko user information provide karta hai
public class CustomUserDetail implements UserDetails {

    private User user;  // Original User entity

    public User getUser() {
        return user;
    }

    // Constructor jisme User entity pass hoti hai
    public CustomUserDetail(User user) {
        this.user = user;
    }

    // User ke roles/authorities return karta hai
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // User ke roles ko Spring Security ke GrantedAuthority objects mein convert karta hai
        return user.getRoles().
                stream().
                map(role -> new SimpleGrantedAuthority(role.getRoleName()))
                .collect(Collectors.toSet());
    }

    @Override
    public String getPassword() {
        return user.getPassword();  // User ka password return karta hai
    }

    @Override
    public String getUsername() {
        return user.getEmail();  // User ka email username ke tarah use karta hai
    }

    // Account expiration, locking, etc. ke methods - sab true return kar rahe hain (matlab ye features disable hain)
    @Override
    public boolean isAccountNonExpired() {
        return true;  // Account kabhi expire nahi hota
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;  // Account kabhi lock nahi hota
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;  // Credentials (password) kabhi expire nahi hote
    }

    @Override
    public boolean isEnabled() {
        return true;  // Account hamesha enabled rehta hai
    }
}
```

## 8. AuthController.java (Login Flow)

```java
@RestController  // REST API controller
@RequestMapping("/api/v1/auth")  // Base URL path
public class AuthController {

    private AuthenticationManager manager;  // Authentication manager
    private UserDetailsService userDetailsService;  // User details service
    private JwtUtil util;  // JWT utilities
    private ModelMapper modelMapper;  // Object mapping ke liye

    // Constructor injection
    public AuthController(AuthenticationManager manager, UserDetailsService userDetailsService, JwtUtil util, ModelMapper modelMapper) {
        this.manager = manager;
        this.userDetailsService = userDetailsService;
        this.util = util;
        this.modelMapper = modelMapper;
    }

    @PostMapping("/login")  // Login endpoint - POST /api/v1/auth/login
    public ResponseEntity<?> createToken(@RequestBody LoginRequest loginRequest) {
        try {
            // Authentication object create karta hai user credentials se
            UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                    loginRequest.getEmail(), loginRequest.getPassword());
                    
            // Authentication manager se authenticate karta hai - agar credentials galat hain to exception throw hogi
            Authentication authenticated = manager.authenticate(authentication);

        } catch (AuthenticationException ex) {
            // Authentication fail hone par exception throw karta hai
            throw new BadCredentialsException("Incorrect email or password !!");
        }

        // User successfully authenticate ho gaya hai

        // UserDetails load karta hai validated email address ke liye
        CustomUserDetail userDetails = (CustomUserDetail) userDetailsService.loadUserByUsername(loginRequest.getEmail());
        
        // JWT token generate karta hai
        String token = util.generateToken(userDetails.getUsername());

        // User entity get karta hai
        User user = userDetails.getUser();

        // Response object build karta hai (token + user details)
        JwtResponse build = JwtResponse.builder()
                .token(token)
                .user(modelMapper.map(user, UserDto.class))  // User entity ko DTO mein convert karta hai
                .build();
                
        // Response return karta hai client ko
        return ResponseEntity.ok(build);
    }
}
```

## 9. Complete Authentication Flow (Dry Run)

### Registration Flow:
1. User `/api/v1/users` endpoint par POST request bhejta hai apne details ke saath.
2. UserController request handle karta hai aur UserService ko call karta hai.
3. UserService user ka password encode karta hai BCryptPasswordEncoder ke through.
4. Naya User entity create hota hai aur database mein save hota hai.
5. User create ho gaya hai aur ab login kar sakta hai.

### Login Flow:
1. User `/api/v1/auth/login` endpoint par POST request bhejta hai email aur password ke saath.
2. AuthController request receive karta hai aur AuthenticationManager ko call karta hai credentials verify karne ke liye.
3. AuthenticationManager internally CustomUserDetailService ko call karta hai database se user load karne ke liye.
4. CustomUserDetailService database se user fetch karta hai aur CustomUserDetail object return karta hai.
5. AuthenticationManager password compare karta hai aur verify karta hai.
6. Agar credentials valid hain, to JwtUtil token generate karta hai.
7. Generated token aur user details client ko response mein bheje jate hain.
8. Client token ko store karta hai (localStorage, cookies, etc. mein) aur future requests mein use karta hai.

### Protected Resource Access Flow:
1. Client kisi protected endpoint ke liye request bhejta hai, HTTP header mein token add karke: `Authorization: Bearer {jwt_token}`.
2. Request sabse pehle JwtAuthenticationFilter ke pass jata hai.
3. Filter Authorization header se token extract karta hai.
4. JwtUtil token validate karta hai - signature check karta hai aur expiration check karta hai.
5. Agar token valid hai, to filter database se user load karta hai aur SecurityContext mein authentication set karta hai.
6. Request controller tak pahunchta hai jahan role-based authorization check hote hain.
7. Agar user ke pass required roles hain, to controller request process karta hai aur response return karta hai.
8. Agar user ke pass required roles nahi hain, to access denied response generate hota hai.

## 10. Important Components and Their Roles

1. **SecurityConfig**: 
   - Security rules configure karta hai
   - URL patterns ke liye authorization rules set karta hai
   - Password encoder provide karta hai
   - CORS aur CSRF configuration set karta hai
   - JWT filter register karta hai
   - Exception handling configure karta hai

2. **JwtAuthenticationFilter**:
   - HTTP requests ko intercept karta hai
   - Authorization header se JWT token extract karta hai
   - Token validate karta hai
   - Valid token ke case mein user ko authenticate karta hai

3. **JwtUtil**:
   - JWT tokens generate karta hai
   - Tokens validate karta hai
   - Tokens se information extract karta hai

4. **CustomAuthenticationEntryPoint**:
   - Authentication failure par custom response generate karta hai

5. **CustomUserDetailService**:
   - Database se users load karta hai
   - UserDetails objects provide karta hai authentication process ke liye

6. **CustomUserDetail**:
   - User entity ko Spring Security ke UserDetails interface se adapt karta hai
   - User ke authorities (roles) provide karta hai

7. **AuthController**:
   - Login API expose karta hai
   - Credentials validate karta hai
   - JWT token generate karta hai aur client ko response mein bhejta hai

---

# Spring Security Interview Questions and Answers

## Basic Spring Security Concepts

### Authentication vs Authorization

**Question:** Spring Security mein Authentication aur Authorization mein kya difference hai?

**Answer:**
- **Authentication (Kaun hai?)**: 
  - User ki identity verify karne ka process hai
  - "Aap kaun ho?" - is sawaal ka jawab deta hai
  - Login credentials (username/password) ya tokens se verify hota hai
  - Is project mein JWT token se authentication ho raha hai

- **Authorization (Kya kar sakta hai?)**: 
  - Authenticated user ko kya access hai, yeh decide karta hai
  - "Aap kya kar sakte ho?" - is sawaal ka jawab deta hai
  - Roles aur permissions ke basis par decide hota hai
  - Is project mein hasRole() methods se authorization ho raha hai

### JWT Token

**Question:** JWT token kya hai aur isme kya advantages hain traditional session-based authentication se?

**Answer:**
- JWT (JSON Web Token) ek compact, self-contained way hai securely information transmit karne ka parties ke beech
- JWT structure: Header + Payload + Signature
- **Advantages:**
  - **Stateless**: Server ko user session store nahi karna padta
  - **Scalability**: Multiple servers handle kar sakte hain requests without sharing session information
  - **Cross-domain/CORS**: Different domains ke beech easily work karta hai
  - **Mobile-friendly**: Mobile applications ke liye suitable hai
  - **Performance**: Database lookups kam hote hain token validation ke liye

- Is project mein JwtUtil class JWT tokens ko generate, validate aur extract karne ke liye use ho rahi hai

### Spring Security Filter Chain

**Question:** Spring Security mein filter chain kaise kaam karti hai? JwtAuthenticationFilter kahan fit hota hai is chain mein?

**Answer:**
- Spring Security multiple filters ka use karta hai HTTP requests ko process karne ke liye
- Filters chain mein arrange hote hain aur sequential order mein execute hote hain
- Standard filters:
  - SecurityContextPersistenceFilter: Security context load/save karta hai
  - UsernamePasswordAuthenticationFilter: Form-based authentication handle karta hai
  - BasicAuthenticationFilter: HTTP Basic authentication handle karta hai
  - And many more...

- **JwtAuthenticationFilter** custom filter hai jo:
  - OncePerRequestFilter extend karta hai (har request ke liye ek baar execute hota hai)
  - UsernamePasswordAuthenticationFilter se pehle chain mein add hota hai
  - Authorization header se JWT token extract karta hai aur validate karta hai
  - Valid token ke case mein SecurityContext mein authentication set karta hai

### Password Encoding

**Question:** Spring Security mein password encoding kyun important hai aur is project mein kaunsa encoder use ho raha hai?

**Answer:**
- Password encoding security ka critical aspect hai - plain text passwords store karna extremely risky hai
- Encoding prevents:
  - Database breaches mein passwords ka exposure
  - Rainbow table attacks
  - Brute force attacks

- **BCryptPasswordEncoder** is project mein use ho raha hai:
  - One-way hashing algorithm hai with salting
  - Adaptive hashing - multiple rounds of hashing apply karta hai
  - Automatically generates different hash for same password (due to different salts)
  - CPU-intensive by design - brute force attacks ko slow karta hai

### Authentication Manager

**Question:** AuthenticationManager ka kya role hai Spring Security mein aur yeh authentication process mein kaise kaam karta hai?

**Answer:**
- **AuthenticationManager** main interface hai jo authentication process ko handle karta hai
- Default implementation **ProviderManager** hai
- Main responsibilities:
  - Authentication request ko validate karna
  - Valid credentials par authenticated Authentication object return karna
  - Invalid credentials par AuthenticationException throw karna

- Working in our project:
  1. AuthController authentication request receive karta hai (email/password)
  2. UsernamePasswordAuthenticationToken create karta hai
  3. AuthenticationManager.authenticate() ko call karta hai
  4. AuthenticationManager internally UserDetailsService use karta hai user ko load karne ke liye
  5. Password comparison automatically hota hai
  6. Success par authenticated object return hota hai, failure par exception throw hoti hai

### UserDetailsService

**Question:** UserDetailsService kya hai aur Spring Security mein iska kya role hai? CustomUserDetailService kaise implement hui hai?

**Answer:**
- **UserDetailsService** ek core interface hai Spring Security mein
- Single method hai: `UserDetails loadUserByUsername(String username)`
- Primary responsibility: username se user details fetch karna authentication ke liye

- **CustomUserDetailService** implementation in our project:
  - Database se user ko find karta hai email ke through
  - User entity ko CustomUserDetail object mein wrap karta hai jo UserDetails interface implement karta hai
  - User ke roles/authorities provide karta hai authorization ke liye
  - If user not found, BadCredentialsException throw karta hai

## Project-Specific Questions

### SecurityConfig Annotations

**Question:** SecurityConfig class par aapne kaun kaun se annotations use kiye hain aur unka kya purpose hai?

**Answer:**
- `@Configuration`: Spring ko batata hai ki ye class beans define karne wali configuration class hai
- `@EnableWebSecurity`: Web security features ko enable karta hai, SecurityFilterChain configure karne ke liye required hai
- `@EnableMethodSecurity(prePostEnabled = true)`: Method level security ko enable karta hai, jisse hum methods par @PreAuthorize/@PostAuthorize annotations use kar sakte hain

```java
@Configuration
@EnableMethodSecurity(prePostEnabled = true)
@EnableWebSecurity(debug = true)
public class SecurityConfig {
    // Configuration code
}
```

### CORS Configuration

**Question:** CORS (Cross-Origin Resource Sharing) kya hai aur aapne apne project mein ise kaise configure kiya hai?

**Answer:**
- **CORS** ek security feature hai browsers mein jo restrict karta hai ki ek domain se dusre domain ke resources access karne ko
- By default, web browsers same-origin policy follow karte hain - ek website dusre domain ke resources access nahi kar sakti
- API ke case mein, frontend aur backend different domains par host ho sakte hain

- Our project mein CORS configuration:
  ```java
  httpSecurity.cors(cor -> {
      CorsConfiguration config = new CorsConfiguration();
      config.setAllowedOrigins(List.of("http://localhost:5173", "http://localhost:4200"));
      config.addAllowedMethod("*");  // Sare HTTP methods allow karta hai
      config.addAllowedHeader("*");  // Sare headers allow karta hai
      config.setAllowCredentials(true);  // Credentials bhejne ki permission deta hai
      config.setMaxAge(300L);  // Preflight requests ka cache time
      UrlBasedCorsConfigurationSource configurationSource = new UrlBasedCorsConfigurationSource();
      configurationSource.registerCorsConfiguration("/**", config);
      cor.configurationSource(configurationSource);
  });
  ```

### JWT Authentication Filter Implementation

**Question:** JwtAuthenticationFilter kaise implement kiya gaya hai aur kya challenges the implementation mein?

**Answer:**
- JwtAuthenticationFilter `OncePerRequestFilter` ko extend karta hai - ensures filter exactly once per request chalega
- Implementation steps:
  1. Request se "Authorization" header extract karta hai
  2. Check karta hai ki header "Bearer" prefix se start hota hai
  3. JWT token extract karta hai aur validate karta hai
  4. Valid token se username extract karta hai aur database se user load karta hai
  5. Authentication object create karta hai aur SecurityContext mein set karta hai

- Challenges:
  - Token extraction aur validation proper error handling ke saath implement karna
  - Security context mein proper authentication object set karna
  - Performance considerations - har request par filter chalega
  - Token expiration aur refresh mechanism implement karna

### User Roles and Authorization

**Question:** Spring Security mein roles kaise implement kiye gaye hain aur authorization kaise kaam karta hai?

**Answer:**
- Roles database mein Role entity ke through store hote hain (roleName field)
- User aur Role ke beech Many-to-Many relationship hai
- CustomUserDetail class user ke roles ko GrantedAuthority objects mein convert karta hai:

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    return user.getRoles().
            stream().
            map(role -> new SimpleGrantedAuthority(role.getRoleName()))
            .collect(Collectors.toSet());
}
```

- URL-based authorization SecurityConfig mein define hui hai:
```java
httpSecurity.authorizeHttpRequests(auth ->
        auth
                .requestMatchers("/api/v1/auth/login").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/courses/**").permitAll()
                .requestMatchers("/api/v1/courses/**").hasRole("ADMIN")
                // More rules...
                .anyRequest()
                .authenticated());
```

- Method-level authorization bhi use kar sakte hain `@PreAuthorize` annotation ke saath

### JWT Token Security

**Question:** JWT tokens ko secure karne ke liye aapne kya measures implement kiye hain?

**Answer:**
- **Strong Signature Key**: JwtUtil class mein secure random key generate hota hai signing ke liye
  ```java
  private Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256);
  ```

- **Token Expiration**: Tokens limited lifetime ke saath generate hote hain (5 minutes)
  ```java
  private long jwtExpiration = 5 * 60 * 1000;
  ```

- **Token Validation**: Multiple validations hote hain:
  - Signature validation - token tampered nahi hua hai
  - Expiration check - token expire nahi hua hai
  - Username validation - token correct user ke liye hai

- **Transport Security**: HTTPS use karna recommend kiya jata hai production mein

- **CSRF Protection**: JWT-based authentication mein CSRF protection generally not needed hai because tokens are passed in headers, not cookies

### Security Context

**Question:** SecurityContextHolder kya hai aur authentication process mein iska kya role hai?

**Answer:**
- **SecurityContextHolder** central component hai Spring Security mein jo current authenticated user ki details store karta hai
- ThreadLocal variable use karta hai jo per-thread information store karta hai
- JwtAuthenticationFilter mein authentication set hoti hai:

```java
SecurityContextHolder.getContext().setAuthentication(authenticationToken);
```

- Benefits:
  - Application ke kisi bhi part mein current user access kar sakte hain
  - Automatically request ke duration tak valid rehta hai
  - Thread-safe implementation - multiple concurrent requests handle kar sakta hai

- Controller methods mein user access karna:
```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
CustomUserDetail userDetail = (CustomUserDetail) authentication.getPrincipal();
User currentUser = userDetail.getUser();
```

## Advanced Spring Security Concepts

### Session Management

**Question:** JWT-based authentication mein session management kaise handle hota hai aur STATELESS ka kya matlab hai?

**Answer:**
- JWT-based authentication typically stateless hota hai:
  - Server koi session state maintain nahi karta
  - Har request self-contained hoti hai with JWT token
  - No session cookies or server-side session storage

- Project mein STATELESS session management configure hai:
```java
httpSecurity.sessionManagement(e -> e.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

- STATELESS benefits:
  - Scalability - server instances ko easily scale kar sakte hain
  - No session synchronization needed in cluster environments
  - Less server memory usage
  - Client-side state management - client responsible hai token store karne ke liye

### Custom Exception Handling

**Question:** Spring Security mein authentication failures aur access denied scenarios ko aapne kaise handle kiya hai?

**Answer:**
- Two types of security exceptions handle kiye hain:

1. **Authentication Failures**: CustomAuthenticationEntryPoint class implement ki hai
   ```java
   public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
       @Override
       public void commence(HttpServletRequest request, HttpServletResponse response, 
                          AuthenticationException authException) throws IOException {
           response.setContentType(MediaType.APPLICATION_JSON_VALUE);
           response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
           CustomMessage customMessage = new CustomMessage();
           customMessage.setMessage(authException.getMessage());
           customMessage.setSuccess(false);
           // Write JSON response
       }
   }
   ```

2. **Access Denied**: SecurityConfig mein accessDeniedHandler define kiya hai
   ```java
   httpSecurity.exceptionHandling(e ->
           e.authenticationEntryPoint(authenticationEntryPoint)
                   .accessDeniedHandler((request, response, accessDeniedException) -> {
                       response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                       // Custom JSON response
                   })
   );
   ```

- Benefits of custom exception handling:
  - Better user experience with proper error messages
  - Consistent response format
  - Security information leakage prevention
  - Easier client-side error handling

### Spring Security Testing

**Question:** Spring Security components ko aap kaise test karenge?

**Answer:**
- **Unit Testing**:
  - JwtUtil ke methods unit tests se test karna
  - CustomUserDetailService ke methods mock UserRepo ke saath test karna
  - AuthController ke methods mock services ke saath test karna

- **Integration Testing**:
  - Spring Security test utilities use karna: `@WithMockUser`, `@WithUserDetails`
  - MockMvc se protected endpoints test karna

- Example test case for secured endpoint:
```java
@SpringBootTest
@AutoConfigureMockMvc
public class SecurityTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    @WithMockUser(roles = "ADMIN")
    public void whenAdminAccessAdminEndpoint_thenSuccess() throws Exception {
        mockMvc.perform(get("/api/v1/users"))
               .andExpect(status().isOk());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    public void whenUserAccessAdminEndpoint_thenForbidden() throws Exception {
        mockMvc.perform(get("/api/v1/users"))
               .andExpect(status().isForbidden());
    }
}
```

### Security Best Practices

**Question:** Spring Security implement karte time aapne kaun kaun se security best practices follow kiye?

**Answer:**
- **Password Encoding**: BCryptPasswordEncoder use kiya hai plain text passwords store karne se bachne ke liye
- **Role-Based Access Control**: APIs ko proper roles se secure kiya hai
- **JWT Implementation**: Proper signing, expiration aur validation mechanisms implement kiye hain
- **CORS Configuration**: Explicitly configured hai permitted origins ke liye
- **CSRF Protection**: APIs ke liye disabled hai kyunki JWT tokens headers mein pass hote hain
- **Custom Error Responses**: Minimal information leak karta hai failures par
- **Stateless Authentication**: No server-side sessions - scalable implementation
- **Request Filtering**: OncePerRequestFilter use kiya hai secure handling ke liye
- **Method Security**: @EnableMethodSecurity use kiya hai fine-grained control ke liye

### Spring Security 6 Features

**Question:** Spring Security 6 mein kya new features hain aur aapke project mein kaunse features use ho rahe hain?

**Answer:**
- Spring Security 6 main features:
  - Lambda DSL for configuration (improved readability)
  - Simplified authorization configuration
  - Better method security
  - Enhanced CSRF protection
  - Improved OAuth 2.0 client support
  - Java 17 support

- Our project using these features:
  - Lambda DSL for HttpSecurity configuration:
  ```java
  httpSecurity.cors(cor -> {
      // Configuration
  });
  
  httpSecurity.authorizeHttpRequests(auth -> {
      // Rules
  });
  ```
  
  - Method security with @EnableMethodSecurity
  - Modern exception handling approach
  - New style for filter registration
