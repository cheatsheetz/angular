# Angular Cheat Sheet

A comprehensive reference for Angular - a platform for building mobile and desktop web applications using TypeScript.

---

## Table of Contents
- [Setup and Installation](#setup-and-installation)
- [Project Structure](#project-structure)
- [Components](#components)
- [Templates and Data Binding](#templates-and-data-binding)
- [Directives](#directives)
- [Services and Dependency Injection](#services-and-dependency-injection)
- [HTTP Client](#http-client)
- [Routing](#routing)
- [Forms](#forms)
- [Pipes](#pipes)
- [Lifecycle Hooks](#lifecycle-hooks)
- [Observables and RxJS](#observables-and-rxjs)
- [Modules](#modules)
- [Testing](#testing)
- [Best Practices](#best-practices)

---

## Setup and Installation

### Angular CLI Installation
```bash
# Install Angular CLI globally
npm install -g @angular/cli

# Create new project
ng new my-app
cd my-app

# Start development server
ng serve

# Build for production
ng build --prod
```

### Project Generation Commands
```bash
# Generate component
ng generate component user-profile
ng g c user-profile

# Generate service
ng generate service user
ng g s user

# Generate module
ng generate module shared
ng g m shared

# Generate guard
ng generate guard auth
ng g g auth

# Generate pipe
ng generate pipe custom-date
ng g p custom-date
```

## Project Structure

### Basic File Structure
```
src/
├── app/
│   ├── components/
│   ├── services/
│   ├── models/
│   ├── guards/
│   ├── pipes/
│   ├── app.component.ts
│   ├── app.component.html
│   ├── app.component.css
│   ├── app.module.ts
│   └── app-routing.module.ts
├── assets/
├── environments/
└── styles.css
```

## Components

### Component Definition
```typescript
import { Component, Input, Output, EventEmitter, OnInit } from '@angular/core';

@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.css']
})
export class UserCardComponent implements OnInit {
  // Input properties
  @Input() user: User;
  @Input() showActions: boolean = true;
  
  // Output events
  @Output() userClick = new EventEmitter<User>();
  @Output() deleteUser = new EventEmitter<number>();
  
  // Component properties
  isLoading: boolean = false;
  errorMessage: string = '';
  
  constructor(private userService: UserService) {}
  
  ngOnInit(): void {
    this.loadUserData();
  }
  
  onUserClick(): void {
    this.userClick.emit(this.user);
  }
  
  onDeleteClick(): void {
    this.deleteUser.emit(this.user.id);
  }
  
  private loadUserData(): void {
    this.isLoading = true;
    this.userService.getUserDetails(this.user.id)
      .subscribe({
        next: (data) => {
          this.user = { ...this.user, ...data };
          this.isLoading = false;
        },
        error: (error) => {
          this.errorMessage = 'Failed to load user data';
          this.isLoading = false;
        }
      });
  }
}
```

### Component Template
```html
<!-- user-card.component.html -->
<div class="user-card" [class.loading]="isLoading">
  <div class="user-avatar">
    <img [src]="user.avatar" [alt]="user.name">
  </div>
  
  <div class="user-info">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <p *ngIf="user.role">Role: {{ user.role | titlecase }}</p>
  </div>
  
  <div class="user-actions" *ngIf="showActions">
    <button 
      (click)="onUserClick()"
      class="btn btn-primary"
      [disabled]="isLoading">
      View Details
    </button>
    <button 
      (click)="onDeleteClick()"
      class="btn btn-danger"
      [disabled]="isLoading">
      Delete
    </button>
  </div>
  
  <div *ngIf="errorMessage" class="error-message">
    {{ errorMessage }}
  </div>
</div>
```

### Standalone Components (Angular 14+)
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-standalone',
  standalone: true,
  imports: [CommonModule, RouterOutlet],
  template: `
    <div>
      <h1>Standalone Component</h1>
      <p>{{ message }}</p>
    </div>
  `
})
export class StandaloneComponent {
  message = 'Hello from standalone component!';
}
```

## Templates and Data Binding

### Data Binding Types
```html
<!-- Interpolation -->
<h1>{{ title }}</h1>
<p>Welcome {{ user.name }}!</p>

<!-- Property Binding -->
<img [src]="imageUrl" [alt]="imageAlt">
<button [disabled]="isLoading">Submit</button>

<!-- Attribute Binding -->
<div [attr.data-id]="user.id">Content</div>
<button [attr.aria-label]="buttonLabel">Click me</button>

<!-- Class Binding -->
<div [class.active]="isActive">Toggle class</div>
<div [class]="cssClasses">Multiple classes</div>
<div [ngClass]="{'active': isActive, 'disabled': isDisabled}">Conditional classes</div>

<!-- Style Binding -->
<div [style.color]="textColor">Colored text</div>
<div [style.font-size.px]="fontSize">Sized text</div>
<div [ngStyle]="{'color': textColor, 'font-size': fontSize + 'px'}">Multiple styles</div>

<!-- Event Binding -->
<button (click)="onClick()">Click me</button>
<input (keyup.enter)="onEnter()" (blur)="onBlur()">
<form (ngSubmit)="onSubmit()">Submit form</form>

<!-- Two-way Binding -->
<input [(ngModel)]="username" type="text">
<input [ngModel]="username" (ngModelChange)="username = $event">
```

## Directives

### Structural Directives
```html
<!-- *ngIf -->
<div *ngIf="isLoggedIn">Welcome back!</div>
<div *ngIf="user; else guestTemplate">Hello {{ user.name }}</div>
<ng-template #guestTemplate>Please log in</ng-template>

<!-- *ngIf with as -->
<div *ngIf="user$ | async as user">
  Hello {{ user.name }}
</div>

<!-- *ngFor -->
<ul>
  <li *ngFor="let item of items; let i = index; let first = first; let last = last">
    {{ i + 1 }}. {{ item.name }}
    <span *ngIf="first">(First)</span>
    <span *ngIf="last">(Last)</span>
  </li>
</ul>

<!-- *ngFor with trackBy -->
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>

<!-- *ngSwitch -->
<div [ngSwitch]="user.role">
  <div *ngSwitchCase="'admin'">Admin Panel</div>
  <div *ngSwitchCase="'user'">User Dashboard</div>
  <div *ngSwitchDefault>Guest View</div>
</div>
```

### Custom Directives
```typescript
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight: string = 'yellow';
  @Input() defaultColor: string = 'transparent';
  
  constructor(private el: ElementRef) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.appHighlight);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(this.defaultColor);
  }
  
  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}

<!-- Usage -->
<p appHighlight="lightblue" defaultColor="white">
  Highlight me on hover
</p>
```

## Services and Dependency Injection

### Creating a Service
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root' // Makes service available app-wide
})
export class UserService {
  private apiUrl = 'https://api.example.com/users';
  private usersSubject = new BehaviorSubject<User[]>([]);
  public users$ = this.usersSubject.asObservable();
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl)
      .pipe(
        map(users => users.map(user => ({ ...user, fullName: `${user.firstName} ${user.lastName}` }))),
        catchError(this.handleError)
      );
  }
  
  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
  
  createUser(user: Partial<User>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
  
  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
  
  updateUsersCache(users: User[]): void {
    this.usersSubject.next(users);
  }
  
  private handleError(error: any): Observable<never> {
    console.error('Service error:', error);
    throw error;
  }
}
```

### Service Injection
```typescript
// Component injection
constructor(
  private userService: UserService,
  private router: Router,
  private activatedRoute: ActivatedRoute
) {}

// Alternative injection methods
// Using inject() function (Angular 14+)
import { inject } from '@angular/core';

export class MyComponent {
  private userService = inject(UserService);
  private router = inject(Router);
}
```

## HTTP Client

### HTTP Service Setup
```typescript
// app.module.ts
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [HttpClientModule],
  // ...
})
export class AppModule {}

// Service with HTTP operations
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private baseUrl = 'https://api.example.com';
  
  constructor(private http: HttpClient) {}
  
  // GET request
  getItems(page: number = 1, limit: number = 10): Observable<any> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('limit', limit.toString());
    
    return this.http.get(`${this.baseUrl}/items`, { params });
  }
  
  // POST request
  createItem(item: any): Observable<any> {
    const headers = new HttpHeaders({
      'Content-Type': 'application/json'
    });
    
    return this.http.post(`${this.baseUrl}/items`, item, { headers });
  }
  
  // PUT request
  updateItem(id: number, item: any): Observable<any> {
    return this.http.put(`${this.baseUrl}/items/${id}`, item);
  }
  
  // DELETE request
  deleteItem(id: number): Observable<any> {
    return this.http.delete(`${this.baseUrl}/items/${id}`);
  }
  
  // File upload
  uploadFile(file: File): Observable<any> {
    const formData = new FormData();
    formData.append('file', file);
    
    return this.http.post(`${this.baseUrl}/upload`, formData);
  }
}
```

### HTTP Interceptors
```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler } from '@angular/common/http';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler) {
    const token = localStorage.getItem('token');
    
    if (token) {
      const authReq = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${token}`)
      });
      return next.handle(authReq);
    }
    
    return next.handle(req);
  }
}

// Register in module
import { HTTP_INTERCEPTORS } from '@angular/common/http';

@NgModule({
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
})
export class AppModule {}
```

## Routing

### Router Setup
```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AuthGuard } from './guards/auth.guard';

const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'login', component: LoginComponent },
  { 
    path: 'dashboard', 
    component: DashboardComponent,
    canActivate: [AuthGuard]
  },
  { 
    path: 'users/:id', 
    component: UserDetailComponent,
    resolve: { user: UserResolver }
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canLoad: [AdminGuard]
  },
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

### Router Navigation
```typescript
import { Router, ActivatedRoute } from '@angular/router';

export class NavigationComponent {
  constructor(
    private router: Router,
    private route: ActivatedRoute
  ) {}
  
  // Programmatic navigation
  goToUser(id: number): void {
    this.router.navigate(['/users', id]);
  }
  
  goToUserWithQuery(): void {
    this.router.navigate(['/users'], { 
      queryParams: { page: 1, filter: 'active' }
    });
  }
  
  // Relative navigation
  goToSibling(): void {
    this.router.navigate(['../sibling'], { relativeTo: this.route });
  }
  
  // Get route parameters
  ngOnInit(): void {
    // Route parameters
    this.route.params.subscribe(params => {
      const id = params['id'];
      this.loadUser(id);
    });
    
    // Query parameters
    this.route.queryParams.subscribe(queryParams => {
      const page = queryParams['page'] || 1;
      this.loadData(page);
    });
    
    // Resolved data
    this.route.data.subscribe(data => {
      this.user = data['user'];
    });
  }
}
```

### Route Guards
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(): boolean {
    if (this.authService.isLoggedIn()) {
      return true;
    } else {
      this.router.navigate(['/login']);
      return false;
    }
  }
}
```

## Forms

### Template-Driven Forms
```typescript
// Component
export class TemplateFormComponent {
  user = {
    name: '',
    email: '',
    age: null
  };
  
  onSubmit(form: NgForm): void {
    if (form.valid) {
      console.log('Form data:', this.user);
    }
  }
}
```

```html
<!-- Template -->
<form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
  <div>
    <label for="name">Name:</label>
    <input 
      type="text" 
      id="name"
      name="name"
      [(ngModel)]="user.name"
      #name="ngModel"
      required
      minlength="2">
    <div *ngIf="name.invalid && name.touched" class="error">
      <div *ngIf="name.errors?.['required']">Name is required</div>
      <div *ngIf="name.errors?.['minlength']">Name must be at least 2 characters</div>
    </div>
  </div>
  
  <div>
    <label for="email">Email:</label>
    <input 
      type="email" 
      id="email"
      name="email"
      [(ngModel)]="user.email"
      #email="ngModel"
      required
      email>
    <div *ngIf="email.invalid && email.touched" class="error">
      <div *ngIf="email.errors?.['required']">Email is required</div>
      <div *ngIf="email.errors?.['email']">Invalid email format</div>
    </div>
  </div>
  
  <button type="submit" [disabled]="userForm.invalid">Submit</button>
</form>
```

### Reactive Forms
```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators, FormArray } from '@angular/core';

export class ReactiveFormComponent implements OnInit {
  userForm: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit(): void {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(2)]],
      email: ['', [Validators.required, Validators.email]],
      age: ['', [Validators.required, Validators.min(18)]],
      address: this.fb.group({
        street: ['', Validators.required],
        city: ['', Validators.required],
        zipCode: ['', Validators.required]
      }),
      hobbies: this.fb.array([])
    });
  }
  
  get hobbies(): FormArray {
    return this.userForm.get('hobbies') as FormArray;
  }
  
  addHobby(): void {
    this.hobbies.push(this.fb.control('', Validators.required));
  }
  
  removeHobby(index: number): void {
    this.hobbies.removeAt(index);
  }
  
  onSubmit(): void {
    if (this.userForm.valid) {
      console.log('Form data:', this.userForm.value);
    } else {
      this.markFormGroupTouched();
    }
  }
  
  private markFormGroupTouched(): void {
    Object.keys(this.userForm.controls).forEach(field => {
      const control = this.userForm.get(field);
      control?.markAsTouched({ onlySelf: true });
    });
  }
  
  // Custom validator
  forbiddenNameValidator(control: AbstractControl): ValidationErrors | null {
    const forbidden = /admin/i.test(control.value);
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  }
}
```

```html
<!-- Reactive Form Template -->
<form [formGroup]="userForm" (ngSubmit)="onSubmit()">
  <div>
    <label for="name">Name:</label>
    <input type="text" id="name" formControlName="name">
    <div *ngIf="userForm.get('name')?.invalid && userForm.get('name')?.touched" class="error">
      <div *ngIf="userForm.get('name')?.errors?.['required']">Name is required</div>
      <div *ngIf="userForm.get('name')?.errors?.['minlength']">Name must be at least 2 characters</div>
    </div>
  </div>
  
  <div formGroupName="address">
    <h3>Address</h3>
    <input type="text" placeholder="Street" formControlName="street">
    <input type="text" placeholder="City" formControlName="city">
    <input type="text" placeholder="Zip Code" formControlName="zipCode">
  </div>
  
  <div>
    <h3>Hobbies</h3>
    <div formArrayName="hobbies">
      <div *ngFor="let hobby of hobbies.controls; let i = index">
        <input type="text" [formControlName]="i" placeholder="Hobby">
        <button type="button" (click)="removeHobby(i)">Remove</button>
      </div>
    </div>
    <button type="button" (click)="addHobby()">Add Hobby</button>
  </div>
  
  <button type="submit" [disabled]="userForm.invalid">Submit</button>
</form>
```

## Pipes

### Built-in Pipes
```html
<!-- String pipes -->
{{ 'hello world' | uppercase }}
{{ 'HELLO WORLD' | lowercase }}
{{ 'hello world' | titlecase }}

<!-- Number pipes -->
{{ 123.456 | number:'1.2-2' }}
{{ 0.123 | percent }}
{{ 1234.56 | currency:'USD':'symbol':'1.2-2' }}

<!-- Date pipes -->
{{ today | date }}
{{ today | date:'shortDate' }}
{{ today | date:'yyyy-MM-dd' }}

<!-- JSON pipe -->
{{ user | json }}

<!-- Async pipe -->
{{ user$ | async }}
<div *ngIf="user$ | async as user">
  {{ user.name }}
</div>

<!-- Slice pipe -->
{{ items | slice:0:3 }}
```

### Custom Pipes
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate'
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 20, trail: string = '...'): string {
    return value.length > limit ? value.substring(0, limit) + trail : value;
  }
}

// Usage
{{ longText | truncate:50:'...' }}
```

## Lifecycle Hooks

### Component Lifecycle
```typescript
import { 
  OnInit, 
  OnDestroy, 
  OnChanges, 
  DoCheck,
  AfterContentInit,
  AfterContentChecked,
  AfterViewInit,
  AfterViewChecked
} from '@angular/core';

export class LifecycleComponent implements 
  OnInit, OnDestroy, OnChanges, DoCheck,
  AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked {
  
  ngOnInit(): void {
    console.log('Component initialized');
  }
  
  ngOnChanges(changes: SimpleChanges): void {
    console.log('Input properties changed:', changes);
  }
  
  ngDoCheck(): void {
    console.log('Change detection ran');
  }
  
  ngAfterContentInit(): void {
    console.log('Content projected');
  }
  
  ngAfterContentChecked(): void {
    console.log('Content checked');
  }
  
  ngAfterViewInit(): void {
    console.log('View initialized');
  }
  
  ngAfterViewChecked(): void {
    console.log('View checked');
  }
  
  ngOnDestroy(): void {
    console.log('Component destroyed');
    // Cleanup subscriptions
  }
}
```

## Observables and RxJS

### Common RxJS Operators
```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subject, Observable, combineLatest } from 'rxjs';
import { takeUntil, map, filter, debounceTime, switchMap } from 'rxjs/operators';

export class RxJSComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit(): void {
    // Basic subscription
    this.userService.getUsers()
      .pipe(takeUntil(this.destroy$))
      .subscribe(users => {
        this.users = users;
      });
    
    // Search with debounce
    this.searchControl.valueChanges
      .pipe(
        debounceTime(300),
        filter(term => term.length > 2),
        switchMap(term => this.searchService.search(term)),
        takeUntil(this.destroy$)
      )
      .subscribe(results => {
        this.searchResults = results;
      });
    
    // Combine multiple observables
    combineLatest([
      this.userService.currentUser$,
      this.settingsService.settings$
    ])
    .pipe(takeUntil(this.destroy$))
    .subscribe(([user, settings]) => {
      this.updateUI(user, settings);
    });
  }
  
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

## Modules

### Feature Module
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

import { UserRoutingModule } from './user-routing.module';
import { UserListComponent } from './components/user-list.component';
import { UserDetailComponent } from './components/user-detail.component';
import { UserService } from './services/user.service';

@NgModule({
  declarations: [
    UserListComponent,
    UserDetailComponent
  ],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    UserRoutingModule
  ],
  providers: [UserService],
  exports: [UserListComponent] // Components to export
})
export class UserModule {}
```

### Shared Module
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

import { LoaderComponent } from './components/loader.component';
import { ConfirmDialogComponent } from './components/confirm-dialog.component';
import { TruncatePipe } from './pipes/truncate.pipe';
import { HighlightDirective } from './directives/highlight.directive';

@NgModule({
  declarations: [
    LoaderComponent,
    ConfirmDialogComponent,
    TruncatePipe,
    HighlightDirective
  ],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule
  ],
  exports: [
    // Re-export common modules
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    
    // Export shared components
    LoaderComponent,
    ConfirmDialogComponent,
    TruncatePipe,
    HighlightDirective
  ]
})
export class SharedModule {}
```

## Testing

### Unit Testing with Jasmine & Karma
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { By } from '@angular/platform-browser';
import { of } from 'rxjs';

import { UserComponent } from './user.component';
import { UserService } from '../services/user.service';

describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;
  let userService: jasmine.SpyObj<UserService>;

  beforeEach(async () => {
    const spy = jasmine.createSpyObj('UserService', ['getUsers']);

    await TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [
        { provide: UserService, useValue: spy }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should load users on init', () => {
    const mockUsers = [{ id: 1, name: 'John' }];
    userService.getUsers.and.returnValue(of(mockUsers));

    component.ngOnInit();

    expect(userService.getUsers).toHaveBeenCalled();
    expect(component.users).toEqual(mockUsers);
  });

  it('should emit user click event', () => {
    const user = { id: 1, name: 'John' };
    spyOn(component.userClick, 'emit');

    component.onUserClick(user);

    expect(component.userClick.emit).toHaveBeenCalledWith(user);
  });
});
```

### Service Testing
```typescript
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should fetch users', () => {
    const mockUsers = [{ id: 1, name: 'John' }];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(1);
      expect(users).toEqual(mockUsers);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
});
```

## Best Practices

### Code Organization
```typescript
// Use interfaces for type safety
export interface User {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest'
}

// Use OnPush change detection for performance
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `...`
})
export class UserListComponent {
  // Use trackBy for ngFor performance
  trackByUserId(index: number, user: User): number {
    return user.id;
  }
}

// Unsubscribe from observables
export class Component implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit(): void {
    this.service.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => { /* handle data */ });
  }
  
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### Performance Tips
```typescript
// Use async pipe instead of manual subscription
// Good
@Component({
  template: `
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
    </div>
  `
})
export class Component {
  users$ = this.userService.getUsers();
}

// Lazy load modules
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
];
```

---

## Build and Deployment

### Build Commands
```bash
# Development build
ng build

# Production build
ng build --prod

# Serve with specific configuration
ng serve --configuration=staging

# Test
ng test
ng e2e

# Lint
ng lint
```

---

## Resources
- [Official Angular Documentation](https://angular.io/docs)
- [Angular CLI](https://cli.angular.io)
- [RxJS Documentation](https://rxjs.dev)
- [Angular Material](https://material.angular.io)
- [Angular DevTools](https://angular.io/guide/devtools)
- [Angular Update Guide](https://update.angular.io)

---
*Originally compiled from various sources. Contributions welcome!*