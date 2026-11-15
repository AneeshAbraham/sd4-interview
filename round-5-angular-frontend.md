# Round 5: Angular Frontend Deep Dive

## **Overview**
- **Duration**: 60-90 minutes
- **Format**: Video call, may include live coding or discussion
- **Goal**: Evaluate deep Angular expertise, RxJS knowledge, and frontend architecture
- **Interviewer**: Senior frontend engineer or full-stack tech lead
- **Level**: Hard

---

## **Angular Fundamentals**

### **1. Component Lifecycle**

```typescript
// Component with all lifecycle hooks
@Component({
  selector: 'app-example',
  template: `<div>{{data}}</div>`,
  styles: ['.class { }']
})
export class ExampleComponent implements OnInit, OnChanges, OnDestroy {
  
  @Input() data: string;
  
  // 1. Constructor - dependency injection
  constructor(private service: MyService) {
    console.log('1. Constructor');
  }
  
  // 2. Before any input properties
  ngOnChanges(changes: SimpleChanges) {
    console.log('2. ngOnChanges', changes);
  }
  
  // 3. After component initialized
  ngOnInit() {
    console.log('3. ngOnInit');
    // Fetch data from API
    this.service.getData().subscribe(data => {
      this.data = data;
    });
  }
  
  // 4. After view initialized
  ngAfterViewInit() {
    console.log('4. ngAfterViewInit');
  }
  
  // 5. After content projected initialized
  ngAfterContentInit() {
    console.log('5. ngAfterContentInit');
  }
  
  // 6. Every change detection cycle
  ngDoCheck() {
    console.log('6. ngDoCheck');
  }
  
  // 7. Before destroy
  ngOnDestroy() {
    console.log('7. ngOnDestroy');
    // Cleanup: unsubscribe, clear timers
  }
}
```

**Interview Questions:**
- Lifecycle execution order?
- When to use which hook?
- OnInit vs. constructor?
- Memory leak prevention in ngOnDestroy?

---

### **2. Change Detection**

```typescript
// Default: OnPush - only on input change or events
@Component({
  selector: 'app-optimized',
  template: `<p>{{data}}</p>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent {
  @Input() data: string;
}

// Manual change detection
@Component({
  selector: 'app-manual',
  template: `<p>{{counter}}</p>`
})
export class ManualComponent implements OnDestroy {
  counter = 0;
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  increment() {
    this.counter++;
    this.cdr.markForCheck(); // Mark for next check
  }
  
  ngOnDestroy() {
    this.cdr.detach(); // Detach from change detection
  }
}
```

**Key Concepts:**
- Default change detection: checks everything
- OnPush: only checks on inputs or events
- Immutable patterns improve performance
- Angular Zone for performance optimization

---

### **3. Dependency Injection**

```typescript
// Service with dependency injection
@Injectable({
  providedIn: 'root' // Singleton at root level
})
export class UserService {
  
  constructor(private http: HttpClient) {}
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
}

// Component using service
@Component({
  selector: 'app-user',
  template: `<div>{{user | async}}</div>`
})
export class UserComponent {
  user: Observable<User>;
  
  constructor(private userService: UserService) {
    this.user = this.userService.getUser(1);
  }
}

// Custom provider
providers: [
  { provide: UserService, useClass: UserService },
  { provide: 'API_URL', useValue: 'https://api.example.com' },
  { provide: Logger, useFactory: loggerFactory }
]
```

**Interview Topics:**
- Injection hierarchy
- providedIn: 'root' vs. providers array
- Custom providers and factory functions
- Multi token providers

---

## **RxJS & Observables Deep Dive**

### **1. Observable Fundamentals**

```typescript
// Creating observables
import { Observable, Subject, BehaviorSubject, ReplaySubject } from 'rxjs';

// From subscription
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.complete();
});

// From array
const arrObservable = from([1, 2, 3]);

// From promise
const promiseObservable = from(Promise.resolve('data'));

// From events
const clickObservable = fromEvent(button, 'click');

// Subjects - both observer and observable
const subject = new Subject<string>();
subject.next('value');
subject.subscribe(value => console.log(value));

// BehaviorSubject - stores latest value
const behaviorSubject = new BehaviorSubject<string>('initial');
const current = behaviorSubject.getValue();

// ReplaySubject - replays n values
const replaySubject = new ReplaySubject<string>(2);
```

**Interview Concepts:**
- Hot vs. cold observables
- Subscription management
- Subject use cases
- Memory leaks with observables

---

### **2. Common Operators**

```typescript
import { map, filter, switchMap, mergeMap, catchError, retry, debounceTime, distinctUntilChanged } from 'rxjs/operators';

// map: transform data
observable.pipe(
  map(value => value * 2)
);

// filter: filter by condition
observable.pipe(
  filter(value => value > 5)
);

// switchMap: switch to new observable (cancels previous)
searchInput$.pipe(
  switchMap(term => this.api.search(term))
);

// mergeMap: merge results (subscribe to all)
observable.pipe(
  mergeMap(id => this.api.getDetails(id))
);

// concatMap: order matters (sequential)
observable.pipe(
  concatMap(id => this.api.getDetails(id))
);

// debounceTime: wait before emitting
searchInput$.pipe(
  debounceTime(300),
  switchMap(term => this.api.search(term))
);

// distinctUntilChanged: emit only on change
observable.pipe(
  distinctUntilChanged()
);

// retry: retry on error
observable.pipe(
  retry(3),
  catchError(error => of(null))
);

// takeUntil: unsubscribe when other observable emits
observable.pipe(
  takeUntil(this.destroy$)
);

// tap: side effects without modifying
observable.pipe(
  tap(value => console.log('Debug:', value))
);

// combineLatest: wait for all observables
combineLatest([obs1$, obs2$, obs3$]).pipe(
  map(([val1, val2, val3]) => ({ val1, val2, val3 }))
);

// forkJoin: wait for all to complete (parallel)
forkJoin([
  this.api.getUser(),
  this.api.getPosts(),
  this.api.getComments()
]).subscribe(([user, posts, comments]) => {
  // All completed
});
```

**Interview Questions:**
- switchMap vs. mergeMap vs. concatMap?
- How to prevent memory leaks?
- Understanding hot vs. cold operators?
- Error handling strategies?

---

### **3. Subscription Management**

```typescript
// Pattern 1: Manually unsubscribe
export class Component implements OnDestroy {
  private subscription: Subscription;
  
  constructor(private service: MyService) {
    this.subscription = this.service.getData().subscribe(data => {
      // Handle data
    });
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}

// Pattern 2: takeUntil
export class Component implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  constructor(private service: MyService) {}
  
  ngOnInit() {
    this.service.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => {
        // Handle data
      });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Pattern 3: async pipe (Angular handles unsubscribe)
@Component({
  template: `<div>{{data | async}}</div>`
})
export class Component {
  data = this.service.getData();
  
  constructor(private service: MyService) {}
}
```

---

## **Angular Services & Architecture**

### **1. HTTP Service Pattern**

```typescript
@Injectable({
  providedIn: 'root'
})
export class ApiService {
  
  constructor(private http: HttpClient) {}
  
  get<T>(url: string): Observable<T> {
    return this.http.get<T>(url);
  }
  
  post<T>(url: string, data: any): Observable<T> {
    return this.http.post<T>(url, data);
  }
  
  put<T>(url: string, data: any): Observable<T> {
    return this.http.put<T>(url, data);
  }
  
  delete<T>(url: string): Observable<T> {
    return this.http.delete<T>(url);
  }
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  
  constructor(private api: ApiService) {}
  
  getUsers(): Observable<User[]> {
    return this.api.get<User[]>('/api/users');
  }
  
  getUser(id: number): Observable<User> {
    return this.api.get<User>(`/api/users/${id}`);
  }
  
  createUser(user: CreateUserRequest): Observable<User> {
    return this.api.post<User>('/api/users', user);
  }
}
```

### **2. HTTP Interceptors**

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  
  constructor(private auth: AuthService) {}
  
  intercept(
    request: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    
    // Add token to request
    const token = this.auth.getToken();
    if (token) {
      request = request.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }
    
    return next.handle(request).pipe(
      catchError(error => {
        if (error.status === 401) {
          this.auth.logout();
        }
        return throwError(error);
      })
    );
  }
}

// Register interceptor
providers: [
  { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true }
]
```

### **3. Error Handling**

```typescript
@Injectable({
  providedIn: 'root'
})
export class ErrorHandlerService {
  
  handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = 'An error occurred';
    
    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Error: ${error.error.message}`;
    } else {
      // Server-side error
      errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
    }
    
    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}

// Using in service
getUser(id: number): Observable<User> {
  return this.http.get<User>(`/api/users/${id}`).pipe(
    catchError(error => this.errorHandler.handleError(error))
  );
}
```

---

## **State Management**

### **1. NgRx (Store Pattern)**

```typescript
// 1. Define state
export interface AppState {
  users: UserState;
}

export interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

// 2. Define actions
export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction(
  '[User] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User] Load Users Failure',
  props<{ error: string }>()
);

// 3. Create reducer
export const userReducer = createReducer(
  initialState,
  on(loadUsers, (state) => ({ ...state, loading: true })),
  on(loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false,
    error: null
  })),
  on(loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  }))
);

// 4. Create effects
@Injectable()
export class UserEffects {
  
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })),
          catchError(error => of(loadUsersFailure({ error: error.message })))
        )
      )
    )
  );
  
  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}

// 5. Create selectors
export const selectUserState = (state: AppState) => state.users;
export const selectUsers = createSelector(
  selectUserState,
  (state: UserState) => state.users
);
export const selectLoading = createSelector(
  selectUserState,
  (state: UserState) => state.loading
);

// 6. Use in component
@Component({
  template: `
    <div *ngIf="(loading$ | async)">Loading...</div>
    <div *ngFor="let user of (users$ | async)">
      {{user.name}}
    </div>
  `
})
export class UserListComponent {
  users$ = this.store.select(selectUsers);
  loading$ = this.store.select(selectLoading);
  
  constructor(private store: Store<AppState>) {
    this.store.dispatch(loadUsers());
  }
}
```

### **2. Simple State Management (Alternative)**

```typescript
@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  
  private usersSubject = new BehaviorSubject<User[]>([]);
  users$ = this.usersSubject.asObservable();
  
  constructor(private api: ApiService) {}
  
  loadUsers() {
    this.api.getUsers().subscribe(users => {
      this.usersSubject.next(users);
    });
  }
  
  addUser(user: User) {
    const current = this.usersSubject.value;
    this.usersSubject.next([...current, user]);
  }
}
```

---

## **Common Interview Problems**

### **1. Implement a Search with Debounce**

```typescript
@Component({
  selector: 'app-search',
  template: `
    <input [formControl]="searchControl" placeholder="Search...">
    <ul>
      <li *ngFor="let result of (results$ | async)">
        {{result}}
      </li>
    </ul>
  `
})
export class SearchComponent {
  searchControl = new FormControl('');
  
  results$ = this.searchControl.valueChanges.pipe(
    debounceTime(300),
    distinctUntilChanged(),
    switchMap(term => this.search(term)),
    startWith([])
  );
  
  constructor(private api: ApiService) {}
  
  search(term: string): Observable<any[]> {
    return this.api.get(`/search?q=${term}`);
  }
}
```

### **2. Implement Pagination**

```typescript
@Component({
  selector: 'app-paginated-list',
  template: `
    <div *ngFor="let item of (items$ | async)">
      {{item}}
    </div>
    <button (click)="nextPage()">Next</button>
  `
})
export class PaginatedListComponent {
  private pageSubject = new BehaviorSubject<number>(1);
  page$ = this.pageSubject.asObservable();
  
  items$ = this.page$.pipe(
    switchMap(page => this.api.getItems(page))
  );
  
  constructor(private api: ApiService) {}
  
  nextPage() {
    this.pageSubject.next(this.pageSubject.value + 1);
  }
}
```

### **3. Implement Form Validation**

```typescript
@Component({
  selector: 'app-form',
  template: `
    <form [formGroup]="form">
      <input formControlName="email" />
      <span *ngIf="form.get('email')?.hasError('required')">
        Email required
      </span>
      <span *ngIf="form.get('email')?.hasError('email')">
        Invalid email
      </span>
      <button [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class FormComponent {
  form = new FormGroup({
    email: new FormControl('', [Validators.required, Validators.email]),
    password: new FormControl('', [Validators.required, Validators.minLength(8)])
  });
  
  submit() {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

---

## **Performance Optimization**

### **1. OnPush Change Detection**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

### **2. TrackBy in *ngFor**
```typescript
@Component({
  template: `
    <div *ngFor="let item of items; trackBy: trackByFn">
      {{item.id}}
    </div>
  `
})
export class ListComponent {
  items: Item[];
  
  trackByFn(index: number, item: Item) {
    return item.id;
  }
}
```

### **3. Lazy Loading**
```typescript
const routes: Routes = [
  { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) }
];
```

---

## **What They're Evaluating**

✅ Deep RxJS understanding
✅ Angular patterns and best practices
✅ Change detection optimization
✅ State management
✅ Performance awareness
✅ TypeScript proficiency
✅ HTTP handling and interceptors
✅ Component architecture

---

## **Common Mistakes to Avoid**

❌ Memory leaks (forgetting unsubscribe)
❌ Not using trackBy in loops
❌ Unnecessary subscriptions in templates
❌ Not understanding operator differences
❌ Ignoring OnPush change detection
❌ Overly complex component logic
❌ Not handling errors
❌ Blocking operations in main thread

---

## **Tips**

✅ **Use async pipe**: Handles unsubscription automatically
✅ **Prefer observables**: Over promises for consistency
✅ **Immutability**: Improves change detection
✅ **Lazy load**: Improve initial load time
✅ **Share operators**: Prevent multiple subscriptions
✅ **Type safety**: Use TypeScript generics
✅ **Testing mindset**: How would you test this?
