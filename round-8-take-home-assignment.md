# Round 8: Take-Home Assignment / Final Round

## **Overview**
- **Duration**: Usually 3-7 days to complete (check timeline with recruiter)
- **Format**: Complete a real-world project or refactor existing code
- **Goal**: Evaluate practical skills, code quality, problem-solving, and how you work on your own
- **Evaluation**: Code quality, architecture, testing, documentation, attention to detail
- **Level**: Hard (simulates real work)

---

## **What to Expect**

### **Common Assignment Types**

**1. Build a Feature** (Most common)
- Create a simple web application
- Example: "Build a product search feature with filters"
- Stack: Java + Angular + PostgreSQL (your stack)
- Timeframe: 3-5 hours of work typically

**2. Refactor Existing Code**
- Given a codebase with issues
- Improve architecture, performance, testability
- Fix bugs, add features
- Timeframe: 2-4 hours typically

**3. Fix Bugs**
- Given a buggy application
- Find and fix issues
- Add tests to ensure they're fixed
- Timeframe: 1-2 hours typically

**4. Optimize Performance**
- Given a slow application
- Identify bottlenecks
- Implement optimizations
- Provide before/after metrics
- Timeframe: 2-3 hours typically

---

## **What They're Evaluating**

✅ **Code Quality**: Clean, readable, maintainable
✅ **Architecture**: Proper separation of concerns
✅ **Testing**: Unit tests, integration tests if applicable
✅ **Problem-Solving**: How you approach problems
✅ **Documentation**: Clear comments, README
✅ **Git Usage**: Meaningful commits
✅ **Completeness**: Did you handle edge cases?
✅ **Performance**: Efficient algorithms/queries
✅ **Security**: No obvious vulnerabilities
✅ **Attention to Detail**: Polish, error handling

---

## **Best Practices**

### **1. Structure & Architecture**

**Backend (Java/Spring Boot):**
```
src/
├── main/java/com/yourcompany/
│   ├── controller/        # REST endpoints
│   │   └── ProductController.java
│   ├── service/           # Business logic
│   │   └── ProductService.java
│   ├── repository/        # Database access
│   │   └── ProductRepository.java
│   ├── entity/            # JPA entities
│   │   └── Product.java
│   ├── dto/               # Data transfer objects
│   │   ├── ProductDTO.java
│   │   └── CreateProductRequest.java
│   ├── exception/         # Custom exceptions
│   │   └── ResourceNotFoundException.java
│   └── config/            # Configuration classes
│       └── AppConfig.java
├── test/java/             # Tests mirror main structure
│   └── com/yourcompany/
│       ├── controller/
│       ├── service/
│       └── repository/
```

**Frontend (Angular):**
```
src/
├── app/
│   ├── modules/
│   │   ├── products/
│   │   │   ├── products-list/
│   │   │   ├── product-detail/
│   │   │   ├── products.component.ts
│   │   │   └── products.service.ts
│   │   └── shared/
│   │       ├── services/
│   │       ├── pipes/
│   │       ├── guards/
│   │       └── interceptors/
│   ├── core/              # Singleton services
│   ├── shared/            # Reusable components
│   └── app.module.ts
└── assets/
```

---

### **2. Code Quality Checklist**

**Java/Spring Boot:**

```java
// ✅ DO: Clear naming
@Service
public class ProductService {
    private final ProductRepository productRepository;
    
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public ProductDTO getProductById(Long id) {
        return productRepository.findById(id)
            .map(this::convertToDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }
}

// ❌ DON'T: Unclear naming, poor error handling
public class Service {
    public void getData(int x) {
        Repository repo = new Repository();
        Object obj = repo.query(x);
        return obj;
    }
}
```

**Angular/TypeScript:**

```typescript
// ✅ DO: Types, clear methods
@Injectable()
export class ProductService {
    constructor(private http: HttpClient) {}
    
    getProducts(filter: ProductFilter): Observable<Product[]> {
        return this.http.get<Product[]>('/api/products', { 
            params: this.buildParams(filter)
        }).pipe(
            catchError(error => {
                console.error('Error fetching products', error);
                return throwError(() => error);
            })
        );
    }
    
    private buildParams(filter: ProductFilter): HttpParams {
        let params = new HttpParams();
        if (filter.category) params = params.set('category', filter.category);
        if (filter.minPrice) params = params.set('minPrice', filter.minPrice.toString());
        return params;
    }
}

// ❌ DON'T: Any types, error handling, unclear structure
export class Service {
    constructor(private http: HttpClient) {}
    
    get(url: any): any {
        return this.http.get(url);
    }
}
```

---

### **3. Testing**

**Backend - JUnit & Mockito:**

```java
@SpringBootTest
@ExtendWith(MockitoExtension.class)
public class ProductServiceTest {
    
    @Mock
    private ProductRepository productRepository;
    
    @InjectMocks
    private ProductService productService;
    
    @Test
    public void testGetProductById_Success() {
        // Arrange
        Long productId = 1L;
        Product product = new Product(productId, "Test Product", 99.99);
        when(productRepository.findById(productId))
            .thenReturn(Optional.of(product));
        
        // Act
        ProductDTO result = productService.getProductById(productId);
        
        // Assert
        assertThat(result.getId()).isEqualTo(productId);
        assertThat(result.getName()).isEqualTo("Test Product");
        verify(productRepository, times(1)).findById(productId);
    }
    
    @Test
    public void testGetProductById_NotFound() {
        // Arrange
        when(productRepository.findById(999L))
            .thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(ResourceNotFoundException.class, 
            () -> productService.getProductById(999L));
    }
}
```

**Frontend - Jasmine & Karma:**

```typescript
describe('ProductService', () => {
    let service: ProductService;
    let httpMock: HttpTestingController;
    
    beforeEach(() => {
        TestBed.configureTestingModule({
            providers: [ProductService],
            imports: [HttpClientTestingModule]
        });
        service = TestBed.inject(ProductService);
        httpMock = TestBed.inject(HttpTestingController);
    });
    
    afterEach(() => {
        httpMock.verify();
    });
    
    it('should fetch products', () => {
        const mockProducts = [
            { id: 1, name: 'Product 1', price: 99.99 }
        ];
        
        service.getProducts().subscribe(products => {
            expect(products.length).toBe(1);
            expect(products[0].name).toBe('Product 1');
        });
        
        const req = httpMock.expectOne('/api/products');
        expect(req.request.method).toBe('GET');
        req.flush(mockProducts);
    });
});
```

---

### **4. Error Handling**

**Backend:**

```java
// Custom exception
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException e) {
        ErrorResponse error = new ErrorResponse(
            "NOT_FOUND",
            e.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneralException(Exception e) {
        ErrorResponse error = new ErrorResponse(
            "INTERNAL_ERROR",
            "An unexpected error occurred",
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

**Frontend:**

```typescript
export class ErrorInterceptor implements HttpInterceptor {
    
    constructor(private snackBar: MatSnackBar) {}
    
    intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        return next.handle(request).pipe(
            catchError(error => {
                let message = 'An error occurred';
                
                if (error.error instanceof ErrorEvent) {
                    message = error.error.message;
                } else if (error.status === 404) {
                    message = 'Resource not found';
                } else if (error.status === 500) {
                    message = 'Server error';
                }
                
                this.snackBar.open(message, 'Close', { duration: 5000 });
                return throwError(() => error);
            })
        );
    }
}
```

---

### **5. Database Schema**

```sql
-- Well-designed schema
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    category_id BIGINT REFERENCES categories(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT price_positive CHECK (price > 0),
    CONSTRAINT stock_non_negative CHECK (stock >= 0)
);

CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_created_at ON products(created_at);

CREATE TABLE categories (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### **6. Documentation**

**README.md:**

```markdown
# Product Search API

## Overview
A full-stack application for searching and filtering products.

## Tech Stack
- Backend: Java 11+, Spring Boot 2.7+
- Frontend: Angular 14+
- Database: PostgreSQL 12+

## Prerequisites
- Java 11+
- Node.js 16+
- PostgreSQL 12+
- Git

## Getting Started

### Backend
1. Clone the repository
2. Configure database in `application.properties`
3. Run migrations: `mvn flyway:migrate`
4. Start application: `mvn spring-boot:run`
5. API available at http://localhost:8080

### Frontend
1. Navigate to frontend directory
2. Install dependencies: `npm install`
3. Start development server: `npm start`
4. Open http://localhost:4200

## Running Tests

### Backend
```bash
mvn test
```

### Frontend
```bash
ng test
```

## API Endpoints

### Products
- `GET /api/v1/products` - List all products
- `GET /api/v1/products/{id}` - Get product details
- `POST /api/v1/products` - Create product (admin)
- `PUT /api/v1/products/{id}` - Update product (admin)
- `DELETE /api/v1/products/{id}` - Delete product (admin)

## Project Structure

See ARCHITECTURE.md for detailed structure.

## Contributing

1. Create feature branch
2. Write tests
3. Submit pull request
```

---

### **7. Git Usage**

**Good commit history:**
```
✅ "Add product search API endpoint"
✅ "Implement product filter on frontend"
✅ "Add unit tests for ProductService"
✅ "Fix: handle empty search results gracefully"

❌ "WIP"
❌ "Fix stuff"
❌ "Updated files"
```

**Good commit message format:**
```
[Feature/Fix/Refactor] Brief description

Longer explanation of changes made and why.

Closes #123
```

---

## **Common Pitfalls**

❌ **Over-engineering**: Complex patterns for simple requirements
❌ **No tests**: Don't skip testing to save time
❌ **Poor error handling**: Silent failures or unclear errors
❌ **Hardcoded values**: Should be configurable
❌ **No documentation**: Unclear how to run/test
❌ **Messy commits**: Should tell a story
❌ **Security ignored**: SQL injection, auth issues
❌ **Performance ignored**: N+1 queries, inefficient algorithms
❌ **Edge cases ignored**: What if input is null/empty?
❌ **Incomplete**: Unfinished features or TODOs

---

## **If You Get Stuck**

**Approach:**
1. **Read requirements carefully** - You may have misunderstood
2. **Start simple** - Get basics working first
3. **Iteratively improve** - Add features/fixes incrementally
4. **Test frequently** - Verify each piece works
5. **Document decisions** - Comment on why you chose certain approach
6. **Don't overthink** - Good enough is better than nothing

**If time is running out:**
1. Focus on completeness over perfection
2. Document what you didn't finish
3. Explain your approach in a summary
4. Make what you have is working and tested

---

## **Before Submission**

**Checklist:**
- [ ] Code compiles/runs without errors
- [ ] All tests pass
- [ ] README is clear and complete
- [ ] No hardcoded values
- [ ] Error handling is in place
- [ ] Code is well-formatted
- [ ] No commented-out code
- [ ] Commit history is clean
- [ ] No secrets in code
- [ ] Edge cases handled

---

## **Submission**

**Include:**
- Link to GitHub repository (public or private access provided)
- Clear instructions to run
- Summary of approach and key decisions
- Any assumptions made
- Time spent on assignment

**Sample summary:**
```
## Implementation Summary

### What I Built
- REST API for product search with filtering
- Angular component for search UI
- PostgreSQL database schema

### Key Decisions
1. Used Specification pattern for dynamic queries (allows flexible filtering)
2. Implemented caching for frequently searched categories
3. Added pagination to handle large result sets

### Testing
- 15 unit tests for service layer
- 8 component tests for UI
- All critical paths covered

### Time Spent
- Backend: 2 hours
- Frontend: 1.5 hours
- Testing: 1 hour
- Documentation: 30 min
- Total: ~5 hours

### What I'd Do With More Time
- Add full-text search capability
- Implement user saved searches
- Add analytics for popular searches
```

---

## **After Submission**

- Thank them for the opportunity
- Ask about timeline for feedback
- Be available for clarifying questions
- Don't make changes unless requested
- Be ready to discuss your code in next round

---

## **Interview Tips About Your Code**

When they ask about your code:

✅ **Explain your reasoning**: "I chose X because..."
✅ **Admit limitations**: "If I had more time, I would..."
✅ **Show testing**: "Here's how I tested this..."
✅ **Discuss trade-offs**: "I chose performance over flexibility because..."
✅ **Be ready to pivot**: "If requirement changed to X, I would..."
✅ **Show growth**: "If I did this again, I would..."

---

## **Remember**

This is your chance to show:
- How you work independently
- Your code quality standards
- Your problem-solving approach
- Your attention to detail
- Your ability to think about real-world constraints

**The goal is not perfection, but professional-quality code.**
