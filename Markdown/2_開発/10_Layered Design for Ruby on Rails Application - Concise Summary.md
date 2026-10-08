## 1. Philosophy

### 1-1. The Rails Way (Traditional MVC)

- **Convention over Configuration** - opinionated defaults, rapid prototyping
- **3 layers**: Models (ActiveRecord), Views (templates), Controllers (orchestration)
- **Strength**: Short path from idea to implementation, perfect for MVPs
- **Challenge**: As apps grow → Fat Models + Fat Controllers + callback hell

### 1-2. The Extended Rails Way (Layered)

- **Structured abstractions** when complexity emerges
- **5+ specialized layers**: Infrastructure, Data/Persistence, Application/Service, Business/Domain, Presentation
- **Strength**: Clear boundaries, testable, maintainable at scale
- **Challenge**: More files, upfront design, steeper learning curve

## 2. Layer Architecture

### 2-1. Traditional MVC Rails

```
╔═══════════════════════════════════════════════════════════════════╗
║                 👤 FRONTEND (Browser/Mobile/SPA)                 ║
╠═══════════════════════════════════════════════════════════════════╣
║  • HTML page rendering                                            ║
║  • JavaScript (jQuery/vanilla)                                    ║
║  • AJAX calls                                                     ║
╚═══════════════════════════════════════════════════════════════════╝
                               ▲ │
                     Response  │ │  Request
             (HTML/JSON mixed) │ │  (GET/POST/PUT/DELETE)
                               │ ▼
╔═══════════════════════════════════════════════════════════════════╗
║               🖥️  BACKEND (Rails Monolithic Server)              ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │                       🌐 ROUTING                           │  ║
║  │                     config/routes.rb                        │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║                               ↓                                   ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │                    📥 CONTROLLERS (Fat)                    │  ║
║  │  • HTTP handling + Business logic                           │  ║
║  │  • Authorization + Validation                               │  ║
║  │  • Direct model access                                      │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║          ↓                    ↓                    ↓              ║
║  ┌────────────────┐  ┌──────────────────┐  ┌─────────────────┐    ║
║  │  📄 VIEWS     │  │  💾 MODELS (Fat) │  │  🔧 OTHERS     │    ║
║  │                │  │                  │  │                 │    ║
║  │ • ERB/HAML     │  │ • Persistence    │  │ • Jobs          │    ║
║  │ • Helpers      │◄─│ • Validations    │  │ • Mailers       │    ║
║  │ • Partials     │  │ • Callbacks      │  │ • Config        │    ║
║  │ • Jbuilder     │◄─│ • Business logic │  │ • Lib           │    ║
║  │ • @vars        │  │ • Associations   │  │                 │    ║
║  │                │  │ • Scopes         │  │                 │    ║
║  │                │  │ • Presentation   │  │                 │    ║
║  │                │  │ • Serialization  │  │                 │    ║
║  └────────────────┘  └──────────────────┘  └─────────────────┘    ║
║                              ↓                                    ║
║                       💿 Database                                ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

⚠️  Issues:
  • No clear separation between frontend and backend concerns
  • Frontend tightly coupled to backend structure
  • API responses expose database schema directly
  • HTML and JSON responses mixed in same controllers/views
  • Hard to evolve frontend independently
```

### 2-2. Layered Rails

```
╔═══════════════════════════════════════════════════════════════════╗
║             👤 FRONTEND (React/Vue/Angular/Mobile)               ║
╠═══════════════════════════════════════════════════════════════════╣
║  • Component-based UI (React/Vue)                                 ║
║  • State management (Redux/Vuex/MobX)                             ║
║  • Type-safe interfaces (TypeScript)                              ║
║  • Independent deployment                                         ║
╚═══════════════════════════════════════════════════════════════════╝
                               ▲ │
                    JSON       │ │  HTTP/REST/GraphQL
          Status: 200/201/4xx  │ │  Authorization: Bearer token
       Headers: ETag, Cache    │ │  Content-Type: application/json
                               │ ▼
╔════════════════════════════════════════════════════════════════════╗
║                      📡 API CONTRACT LAYER                        ║
╠════════════════════════════════════════════════════════════════════╣
║  Versioned, Stable API Responses (Independent of DB Schema)        ║
║                                                                    ║
║  GET /api/v1/users/123  →  { "id": 123, "full_name": "...",        ║
║                              "email": "...", "avatar_url": "..." } ║
╚════════════════════════════════════════════════════════════════════╝
                              ▲
══════════════════════════════╬═══════════════════════════════════════
                              ▼
╔═══════════════════════════════════════════════════════════════════╗
║                  🖥️  BACKEND (Rails API Server)                  ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │               🌐 ROUTING (API Versioning)                  │  ║
║  │       namespace :api do namespace :v1 do ... end end        │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║                              ↓                                    ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │            📥 INTERFACE LAYER (Thin Controllers)           │   ║
║  │  • HTTP handling only                                       │  ║
║  │  • Middleware: Auth, CORS, Rate Limit                       │  ║
║  │  • Delegates to services                                    │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║                              ↓                                    ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │           🎯 APPLICATION LAYER (Orchestration)             │   ║
║  │  • Service Objects    • Form Objects                        │  ║
║  │  • Command Objects    • Filter Objects                      │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║                              ↓                                    ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │           💼 BUSINESS/DOMAIN LAYER (Pure Logic)            │   ║
║  │  • Domain Models      • Value Objects                       │  ║
║  │  • Policy Objects     • Domain Events                       │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║                              ↓                                    ║
║  ┌─────────────────────────────────────────────────────────────┐  ║
║  │            💾 DATA/PERSISTENCE LAYER                       │   ║
║  │  • Repositories       • Query Objects                       │  ║
║  │  • ActiveRecord (persistence only)                          │  ║
║  └─────────────────────────────────────────────────────────────┘  ║
║                              ↓                                    ║
║                          💿 Database                              ║
║                                                                   ║
║  ┌──────────────────────┐  ┌──────────────────────────────────┐   ║
║  │  🎨 PRESENTATION    │   │  🔧 INFRASTRUCTURE              │   ║
║  │  (Parallel Layer)    │  │  (Parallel Layer)                │   ║
║  │                      │  │                                  │   ║
║  │  • Serializers (API) │  │  • Background Jobs               │   ║
║  │  • Presenters (Web)  │  │  • Logging & Monitoring          │   ║
║  │  • Components (SSR)  │  │  • Config & Secrets              │   ║
║  │  • Version-specific  │  │  • External Service Adapters     │   ║
║  │    response formats  │  │  • Cache Management              │   ║
║  └──────────────────────┘  └──────────────────────────────────┘   ║
║           ▲                                                       ║
║           │                                                       ║
║           └─ Transforms domain objects into                       ║
║              versioned API responses (flows back up)              ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

✅  Benefits:
  • Frontend and backend completely decoupled
  • API contracts stable and versioned (v1, v2)
  • Frontend can evolve independently
  • Backend schema changes don't break frontend
  • Multiple frontend clients (web/mobile/desktop) use same API
  • Clear separation of concerns at every layer
```

## 3. MVC vs Layered: Quick Comparison

|Responsibility |MVC |Layered |
|:-|:-|:-|
|**Authorization** |Scattered in controllers/models |`app/policies/` - single source of truth |
|**Business Logic** |Mixed everywhere |Domain models + `app/services/` |
|**Data Access** |Direct ActiveRecord everywhere |Repositories + Query Objects |
|**Form Handling** |Strong params + model validations |Form Objects (context-sensitive) |
|**Presentation** |`@instance_vars` + helpers + `#as_json` |Presenters + Serializers + View Components |
|**Use Cases** |Hidden in controller actions |Explicit Service Objects |

## 4. Key Components on The Extended Rails Way (Layered)

### 4-1. Application/Service Layer

- **Service Objects**: `User::RegistrationService.call(params)` - single business operation
- **Form Objects**: `CheckoutForm.new(params).save` - multi-model operations, context validations
- **Filter Objects**: `ProjectsFilter.new(params).apply(scope)` - user-driven queries

### 4-2. Business/Domain Layer

- **Domain Models**: Rich behavior, no persistence details
- **Policy Objects**: `PostPolicy.new(user, post).publish?` - authorization rules
- **Value Objects**: `Money.new(100, 'USD')` - immutable domain concepts

### 4-3. Data/Persistence Layer

- **Repositories**: `PostRepository.published` - data access abstraction
- **Query Objects**: `ActiveUsersQuery.call` - reusable complex queries
- **ActiveRecord**: Stripped down to persistence only

### 4-4. Presentation Layer

- **View Components**: `render SearchBox::Component.new(url:, placeholder:)` - explicit interfaces
- **Presenters**: `UserPresenter.new(user).display_name` - context-aware formatting
- **Serializers**: `UserSerializer.new(user).as_json` - API contracts, versioning

## 5. When to Use Layered Approach?

### 5-1. ✅ Good Fit

- Beyond MVP stage
- Teams of 3+ developers
- Complex business domains
- Long-term maintenance
- Multiple API clients
- Performance requirements

### 5-2. ❌ Overkill

- Simple CRUD apps
- Prototypes/MVPs
- 1-2 person teams
- Short-lived projects

## 6. Pros & Cons

### 6-1. Pros (Benefits of Layered Approach)

- **Maintainability**: Clear separation, easy to locate code
- **Testability**: Isolated layers, less framework coupling
- **Scalability**: Linear growth, independent layers
- **Flexibility**: Swap implementations, context-sensitive behavior
- **Clarity**: Code speaks business language

### 6-2. Cons (Challenges/Tradeoffs)

- **Initial Complexity**: More files, steeper learning curve
- **Slower Start**: No scaffolding support, overhead for simple features
- **Team Alignment**: Everyone must follow patterns consistently
- **Debugging**: Longer call stacks, more indirection
- **Migration Cost**: Refactoring effort for existing apps

## 7. Pragmatic Adoption Strategy

1. **Start with Rails Way** - traditional MVC for new projects
2. **Watch for pain points**:
   - Models > 200 lines
   - Controllers with complex workflows
   - Slow, brittle tests
3. **Extract gradually** - refactor specific areas as needed
4. **Follow conventions** - maintain Rails idioms in new layers
5. **Keep hybrid** - layering where complex, traditional where simple

## 8. Key Takeaway

**Traditional MVC**: Rapid initial development, struggles with scale
**Layered Approach**: Upfront cost, long-term maintainability

The best approach is **pragmatic evolution** - start simple, add structure when complexity demands it.  
Not dogma, but disciplined refactoring guided by real pain points.
