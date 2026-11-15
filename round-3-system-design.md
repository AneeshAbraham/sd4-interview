# Round 3: System Design Round

## **Overview**
- **Duration**: 45-60 minutes
- **Format**: Video call, whiteboard or drawing tool
- **Goal**: Evaluate architectural thinking, scalability, and full-stack understanding
- **Interviewer**: Senior engineer or tech lead
- **Difficulty**: Medium to hard conceptually

---

## **Typical System Design Problems**

### **Common Scenarios for Full-Stack Developers**

1. **E-commerce Product Catalog**
   - Search, filter, pagination
   - Product recommendations
   - Shopping cart system
   - Order processing

2. **Social Media Feed**
   - Real-time updates
   - Following relationships
   - Feed ranking/personalization

3. **URL Shortener / Link Service**
   - Billions of short URLs
   - Redirection performance
   - Analytics tracking

4. **Chat / Messaging System**
   - Real-time messages
   - Typing indicators
   - Message search
   - Notification delivery

5. **Photo Sharing Platform**
   - Image upload/storage
   - Feed display
   - Search/tagging
   - Real-time notifications

6. **Rating/Review System**
   - User-generated content
   - Aggregation & ranking
   - Sorting & filtering
   - Real-time updates

---

## **System Design Approach (RESHADED)**

### **R - Requirements Clarification**
1. **Functional Requirements:**
   - What are the core features?
   - User workflows?
   - Data that needs to persist?

2. **Non-Functional Requirements:**
   - Scale (users, QPS, data size)?
   - Availability (99.9%? 99.99%)?
   - Latency requirements?
   - Consistency vs. Availability?

**Example questions to ask:**
- "How many users? Daily active users?"
- "What's the read-to-write ratio?"
- "What's the target response time?"
- "Should this be eventually consistent or strongly consistent?"

---

### **E - Estimation & Back-of-Envelope**
Calculate:
- **QPS (Queries Per Second)**
  - Example: 1 million DAU → ~100 million requests/day → ~1.2K QPS peak
- **Storage needed**
  - Example: 1M users × 1KB profile data = 1GB
- **Bandwidth**
  - Example: 1K QPS × 1KB response = ~1MB/sec
- **Concurrent connections**
  - Example: 10K concurrent users for real-time chat

---

### **S - System Architecture**
Discuss high-level components:

**Frontend (Angular):**
- Responsive web application
- Lazy loading, pagination
- Real-time updates (WebSocket)
- State management

**Backend (Java/Spring Boot):**
- REST API / GraphQL
- Business logic services
- Authentication/Authorization
- Caching layer

**Database (PostgreSQL):**
- Schema design
- Indexing strategy
- Query optimization
- Read replicas for scaling

**Supporting Services:**
- Cache (Redis)
- Message queue (RabbitMQ, Kafka)
- Search engine (Elasticsearch)
- File storage (S3)

---

### **H - High-Level Design**
Draw a diagram with:
- Clients (web, mobile, etc.)
- Load balancer
- API servers (multiple instances)
- Cache layer
- Database (primary + replicas)
- External services

```
[Users/Clients]
       ↓
[Load Balancer]
       ↓
[API Server 1] [API Server 2] [API Server 3]
       ↓
[Cache Layer (Redis)]
       ↓
[Primary DB] → [Read Replicas]
       ↓
[External Services: S3, Search, Queue]
```

---

### **A - API Design / Contracts**
Define key APIs:

**REST Endpoints Example (E-commerce):**
```
GET /api/v1/products?page=1&limit=20&category=electronics
GET /api/v1/products/{id}
POST /api/v1/orders
GET /api/v1/orders/{id}
GET /api/v1/users/{id}/cart
POST /api/v1/users/{id}/cart/items
DELETE /api/v1/users/{id}/cart/items/{itemId}
```

**Request/Response Format:**
```json
GET /api/v1/products/123

Response:
{
  "id": 123,
  "name": "Product Name",
  "price": 99.99,
  "description": "...",
  "inventory": 50,
  "rating": 4.5,
  "reviews_count": 120
}
```

---

### **D - Database Schema**
Design tables with proper normalization:

**E-commerce Example:**
```sql
Users Table:
- id (PK)
- username
- email
- password_hash
- created_at

Products Table:
- id (PK)
- name
- description
- price
- category_id (FK)
- inventory_count
- created_at

Orders Table:
- id (PK)
- user_id (FK)
- total_price
- status (pending, completed, cancelled)
- created_at

OrderItems Table:
- id (PK)
- order_id (FK)
- product_id (FK)
- quantity
- unit_price
```

**Indexes:**
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

---

### **E - Evolution & Scale**
Discuss how to handle growth:

1. **Caching Strategy:**
   - Redis for hot data
   - Cache invalidation patterns
   - Cache-aside vs. write-through

2. **Database Scaling:**
   - Read replicas for read-heavy workloads
   - Sharding strategy (by user_id, geographic, etc.)
   - When to consider NoSQL?

3. **Asynchronous Processing:**
   - Message queues for long operations
   - Background jobs (order processing)
   - Event-driven architecture

4. **Search & Analytics:**
   - Elasticsearch for full-text search
   - Data warehouse for analytics
   - Separate reporting cluster

5. **Real-time Features:**
   - WebSockets for live updates
   - Message queue (Kafka) for events
   - Push notifications

---

### **D - Deep Dive into Components**

Pick 1-2 areas to go deeper:

**Authentication & Authorization:**
- JWT tokens vs. Session-based
- OAuth2 for third-party login
- Role-based access control (RBAC)

**API Design:**
- Versioning strategy (URL vs. header)
- Error handling & status codes
- Rate limiting & throttling

**Database Optimization:**
- Query execution plans
- Indexing strategies
- Connection pooling

**Angular Frontend:**
- State management (NgRx)
- Error handling & retry logic
- Pagination & infinite scroll
- Real-time updates (RxJS, WebSocket)

---

## **Java/Spring Boot Specific Points**

**Discuss:**
- Spring Boot autoconfiguration
- Spring Data JPA for database access
- Spring Security for authentication
- REST Controller design
- Exception handling patterns
- Dependency injection
- Transaction management

**Example Spring Boot Setup:**
```java
@SpringBootApplication
@EnableCaching
public class Application { }

@RestController
@RequestMapping("/api/v1/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping
    @Cacheable("products")
    public Page<Product> getProducts(
        @RequestParam int page,
        @RequestParam int limit
    ) {
        return productService.getProducts(page, limit);
    }
    
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.getProductById(id)
            .orElseThrow(() -> new ResourceNotFoundException());
    }
}
```

---

## **Angular Specific Points**

**Discuss:**
- Service layer for API calls
- HttpClient with interceptors
- State management (NgRx/Akita)
- Error handling & retry logic
- Real-time updates with RxJS
- Pagination & lazy loading
- Change detection strategies

**Example Service:**
```typescript
@Injectable()
export class ProductService {
  constructor(private http: HttpClient) {}
  
  getProducts(page: number, limit: number): Observable<Page<Product>> {
    return this.http.get<Page<Product>>(
      '/api/v1/products',
      { params: { page, limit } }
    ).pipe(
      retry(2),
      catchError(error => {
        // Handle error
        return throwError(error);
      })
    );
  }
}
```

---

## **PostgreSQL Specific Points**

**Discuss:**
- Query optimization techniques
- Index strategies (B-tree, Hash, BRIN)
- EXPLAIN/ANALYZE for query plans
- Connection pooling (PgBouncer)
- Replication for high availability
- Partitioning for large tables
- JSONB for semi-structured data
- Full-text search capabilities

---

## **Common Mistakes to Avoid**

❌ Jumping to solutions without understanding requirements
❌ Over-engineering for day-one requirements
❌ Ignoring scalability from the start
❌ Poor database schema design
❌ Not discussing trade-offs
❌ Assuming infinite resources
❌ Forgetting about monitoring/logging
❌ Not mentioning security
❌ Too focused on one layer (only backend or frontend)

---

## **What They're Evaluating**

✅ Do you understand full-stack architecture?
✅ Can you make reasonable estimates?
✅ Do you think about scalability?
✅ Can you design extensible systems?
✅ Do you understand trade-offs?
✅ Can you communicate complex ideas clearly?
✅ How deep is your knowledge in each layer?
✅ Do you consider security, monitoring, failover?

---

## **Follow-up Questions Often Asked**

1. "How would you handle 10x traffic tomorrow?"
2. "How would you ensure data consistency in distributed system?"
3. "How would you implement caching without cache invalidation issues?"
4. "How would you handle failures in the message queue?"
5. "How would you migrate this to microservices?"
6. "How would you implement pagination efficiently?"
7. "How would you secure the API?"
8. "How would you monitor this system?"

---

## **Tips**

✅ **Draw diagrams**: Visual communication is crucial
✅ **Ask questions**: Show you think critically
✅ **Discuss trade-offs**: Show maturity (speed vs. consistency, cost vs. complexity)
✅ **Mention monitoring**: Production systems need observability
✅ **Think about security**: Authentication, authorization, data protection
✅ **Consider failure modes**: What happens when components fail?
✅ **Scale incrementally**: Don't over-engineer day 1
✅ **Use correct terminology**: Show you know the domain
