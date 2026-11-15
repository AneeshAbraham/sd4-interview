# Round 4: Java Backend Deep Dive

## **Overview**
- **Duration**: 60-90 minutes
- **Format**: Video call, may include live coding or discussion
- **Goal**: Evaluate deep Java/Spring Boot expertise, architectural patterns, and problem-solving
- **Interviewer**: Senior backend engineer or architect
- **Level**: Hard

---

## **Core Java Concepts**

### **1. Collections Framework**

**Commonly Asked:**
- HashMap vs. ConcurrentHashMap vs. Hashtable?
- ArrayList vs. LinkedList - which to use?
- HashSet vs. TreeSet - performance differences?
- How does HashMap handle collisions?
- Why is HashMap not thread-safe?

**Key Points to Know:**

```java
// HashMap: O(1) average, not synchronized, null keys/values allowed
Map<String, Integer> map = new HashMap<>();

// ConcurrentHashMap: Thread-safe without full lock, segments
Map<String, Integer> concurrent = new ConcurrentHashMap<>();

// LinkedHashMap: Maintains insertion order
Map<String, Integer> linked = new LinkedHashMap<>();

// TreeMap: Sorted by keys, O(log n) operations
NavigableMap<String, Integer> tree = new TreeMap<>();

// Collections utility methods
Collections.sort(list);
Collections.synchronizedMap(new HashMap<>());
Collections.unmodifiableMap(map);
```

**Interview Depth:**
- Explain internal structure (buckets and linked lists)
- Load factor and resizing
- Why immutable keys are important
- Concurrent modification exception

---

### **2. Streams & Functional Programming (Java 8+)**

**Essential Operations:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// map: transform each element
numbers.stream()
    .map(n -> n * 2)
    .collect(Collectors.toList()); // [2, 4, 6, 8, 10]

// filter: keep elements matching predicate
numbers.stream()
    .filter(n -> n > 2)
    .collect(Collectors.toList()); // [3, 4, 5]

// reduce: combine all elements
int sum = numbers.stream()
    .reduce(0, Integer::sum); // 15

// flatMap: flatten nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // [1, 2, 3, 4]

// collectors: terminal operations
numbers.stream()
    .collect(Collectors.groupingBy(n -> n % 2)); // {0=[2,4], 1=[1,3,5]}

// parallel streams
numbers.parallelStream()
    .filter(n -> n > 2)
    .collect(Collectors.toList());
```

**Interview Questions:**
- Difference between map and flatMap?
- When to use parallel streams?
- What is a terminal vs. intermediate operation?
- Custom collectors?

---

### **3. Concurrency & Threading**

**Key Concepts:**
- **Thread safety**: volatile, synchronized, atomic classes
- **Locks**: ReentrantLock, ReadWriteLock
- **Executors**: Thread pools, Future, Callable
- **Concurrent collections**: CopyOnWriteArrayList, ConcurrentHashMap

```java
// ExecutorService for thread management
ExecutorService executor = Executors.newFixedThreadPool(10);

// Submit tasks
Future<Integer> future = executor.submit(() -> {
    return expensiveOperation();
});

// Get result with timeout
try {
    Integer result = future.get(5, TimeUnit.SECONDS);
} catch (TimeoutException e) {
    // Handle timeout
}

executor.shutdown();
```

**Common Patterns:**
```java
// Atomic variables
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();

// Synchronized method
synchronized void updateState() {
    // Only one thread at a time
}

// ReentrantLock for more control
Lock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}
```

**Interview Focus:**
- Race conditions and data races
- Deadlock detection
- Thread pool sizing
- Future and CompletableFuture

---

### **4. Exception Handling**

**Best Practices:**
```java
// Custom exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// Try-catch-finally vs. try-with-resources
try (InputStream input = new FileInputStream("file.txt")) {
    // Automatically closes resource
} catch (IOException e) {
    // Handle exception
} finally {
    // Optional - cleanup code
}

// Multi-catch
try {
    // code
} catch (IOException | SQLException e) {
    // Handle multiple exception types
}
```

**Questions to Prepare:**
- Checked vs. unchecked exceptions?
- When to create custom exceptions?
- How to handle exceptions in streams?
- Exception propagation in async code?

---

## **Spring Boot Deep Dive**

### **1. Dependency Injection & Bean Lifecycle**

```java
// @Component, @Service, @Repository, @Controller
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EmailService emailService;
    
    // Constructor injection (preferred)
    public UserService(UserRepository repo, EmailService email) {
        this.userRepository = repo;
        this.emailService = email;
    }
}

// Bean lifecycle
@Component
public class MyBean implements InitializingBean, DisposableBean {
    
    @PostConstruct
    public void init() {
        // Called after bean creation
    }
    
    @Override
    public void afterPropertiesSet() {
        // InitializingBean interface
    }
    
    @PreDestroy
    public void cleanup() {
        // Called before bean destruction
    }
}
```

**Interview Topics:**
- Singleton vs. prototype scope
- Circular dependency issues
- Bean initialization order
- Configuration classes vs. XML

---

### **2. RESTful API Design**

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    // GET - retrieve resource
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // POST - create resource
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDTO user = userService.createUser(request);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(user);
    }
    
    // PUT - update entire resource
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody UpdateUserRequest request
    ) {
        return ResponseEntity.ok(userService.updateUser(id, request));
    }
    
    // PATCH - partial update
    @PatchMapping("/{id}")
    public ResponseEntity<UserDTO> partialUpdate(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates
    ) {
        return ResponseEntity.ok(userService.partialUpdate(id, updates));
    }
    
    // DELETE - remove resource
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Interview Points:**
- HTTP status codes (200, 201, 204, 400, 404, 500)
- Request/response validation
- Content negotiation
- HATEOAS principles
- Pagination implementation

---

### **3. Spring Data JPA**

```java
// Repository with common CRUD operations
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived query methods
    Optional<User> findByEmail(String email);
    List<User> findByFirstNameContainingIgnoreCase(String firstName);
    List<User> findByAgeGreaterThan(int age);
    List<User> findByRoleIn(List<String> roles);
    
    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findUserByEmail(@Param("email") String email);
    
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE age > ?1", nativeQuery = true)
    List<User> findAdults(int minAge);
    
    // Pagination
    Page<User> findByRole(String role, Pageable pageable);
    
    // Bulk operations
    @Modifying
    @Query("DELETE FROM User u WHERE u.age < ?1")
    void deleteMinors(int age);
}

// Entity with relationships
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String email;
    private String firstName;
    private String lastName;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;
    
    @ManyToOne
    @JoinColumn(name = "role_id")
    private Role role;
}
```

**Questions to Prepare:**
- N+1 query problem and solutions
- Eager vs. lazy loading
- Cascade types
- Transaction boundaries
- Dirty checking and change tracking

---

### **4. Spring Security**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .formLogin()
                .loginPage("/login")
            .and()
            .logout()
                .logoutUrl("/logout");
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

// JWT Authentication
@Component
public class JwtTokenProvider {
    
    @Value("${app.jwtSecret}")
    private String jwtSecret;
    
    @Value("${app.jwtExpirationInMs}")
    private int jwtExpirationInMs;
    
    public String generateToken(String userId) {
        return Jwts.builder()
            .setSubject(userId)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpirationInMs))
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }
    
    public String getUserIdFromToken(String token) {
        return Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }
}
```

**Interview Topics:**
- Authentication vs. authorization
- JWT claims and validation
- CORS configuration
- CSRF protection
- Security headers

---

### **5. Transaction Management**

```java
@Service
public class PaymentService {
    
    @Autowired
    private PaymentRepository paymentRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    // Declarative transaction management
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        rollbackFor = Exception.class
    )
    public void processPayment(Order order, Payment payment) {
        // Both operations succeed or both rollback
        order.setStatus("PAID");
        orderRepository.save(order);
        
        payment.setStatus("COMPLETED");
        paymentRepository.save(payment);
    }
    
    // Nested transaction
    @Transactional
    public void mainTransaction() {
        // Main work
        nestedTransaction();
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void nestedTransaction() {
        // Separate transaction
    }
}
```

**Concepts:**
- Propagation types (REQUIRED, REQUIRES_NEW, NESTED)
- Isolation levels (READ_UNCOMMITTED to SERIALIZABLE)
- Rollback rules
- Declarative vs. programmatic transactions

---

## **Common Interview Problems**

### **1. Implement a LRU Cache**
```java
public class LRUCache<K, V> {
    private int capacity;
    private Map<K, V> cache;
    private LinkedHashMap<K, V> lru;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>(capacity);
        this.lru = new LinkedHashMap<K, V>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LRUCache.this.capacity;
            }
        };
    }
    
    public V get(K key) {
        return lru.getOrDefault(key, null);
    }
    
    public void put(K key, V value) {
        lru.put(key, value);
    }
}
```

### **2. Implement a Retry Logic**
```java
public class RetryTemplate {
    
    public <T> T execute(Callable<T> task, int maxRetries) throws Exception {
        int attempt = 0;
        Exception lastException = null;
        
        while (attempt < maxRetries) {
            try {
                return task.call();
            } catch (Exception e) {
                lastException = e;
                attempt++;
                Thread.sleep((long) Math.pow(2, attempt) * 100); // Exponential backoff
            }
        }
        
        throw lastException;
    }
}
```

### **3. Implement a Circuit Breaker**
```java
public class CircuitBreaker {
    private enum State { CLOSED, OPEN, HALF_OPEN }
    
    private State state = State.CLOSED;
    private int failureThreshold = 5;
    private int failureCount = 0;
    private long lastFailureTime = 0;
    private long timeout = 60000; // 1 minute
    
    public <T> T execute(Supplier<T> operation) {
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - lastFailureTime > timeout) {
                state = State.HALF_OPEN;
            } else {
                throw new CircuitBreakerOpenException();
            }
        }
        
        try {
            T result = operation.get();
            onSuccess();
            return result;
        } catch (Exception e) {
            onFailure();
            throw e;
        }
    }
    
    private void onSuccess() {
        failureCount = 0;
        state = State.CLOSED;
    }
    
    private void onFailure() {
        failureCount++;
        lastFailureTime = System.currentTimeMillis();
        if (failureCount >= failureThreshold) {
            state = State.OPEN;
        }
    }
}
```

---

## **What They're Evaluating**

✅ Deep understanding of Java internals
✅ Spring Boot best practices
✅ Ability to design scalable APIs
✅ Security awareness
✅ Transaction handling
✅ Performance optimization
✅ Debugging skills
✅ Software design principles

---

## **Tips**

✅ **Know your tools**: IDE shortcuts, debugging, profiling
✅ **Explain trade-offs**: Why choose this pattern over another?
✅ **Real-world examples**: Reference projects you've worked on
✅ **Performance mindset**: Think about efficiency
✅ **Testing mindset**: How would you test this?
✅ **Security first**: Always consider security implications
