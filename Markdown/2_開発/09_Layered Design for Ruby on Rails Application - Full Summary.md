## 1. Architecture Philosophy Comparison

### 1-1. "The Rails Way" (Traditional MVC)

Rails was built on the principle of **Convention over Configuration** to maximize developer happiness and productivity.  
The framework provides three core abstractions—Models, Views, and Controllers—with opinionated defaults that eliminate configuration overhead.  
ActiveRecord models serve as the central nervous system, handling persistence, validation, business logic, and even presentation concerns in a single, cohesive object.  
This monolithic approach trades purity for pragmatism: callbacks orchestrate side effects, concerns share code across contexts, and controllers coordinate everything from HTTP handling to database queries.

The brilliance of this approach lies in its **rapid prototyping capability**.  
Scaffolding generates working CRUD interfaces in seconds.  
The close coupling between layers—controllers calling models directly, views accessing instance variables, models mixing persistence with business logic—creates short, direct paths from intention to implementation.  
For MVPs, small applications, and teams learning Rails, this simplicity is a superpower.  
The framework guides you through conventions, and everything you need is within reach of a single model file.

However, as applications grow, this tight coupling becomes a liability.  
Fat Controllers balloon with orchestration logic.  
Fat Models accumulate hundreds of methods mixing persistence, validation, business rules, and presentation helpers.  
Callbacks create invisible dependencies where changing one behavior triggers cascading side effects.  
The lack of boundaries makes testing require full database fixtures, slows down test suites, and couples every component to ActiveRecord.  
What started as elegant simplicity evolves into a maintenance burden where every change risks unexpected breakage.

### 1-2. "The Extended Rails Way" (Layered Architecture)

The layered approach doesn't reject Rails—it **extends** it with explicit abstractions when complexity demands structure.  
Rather than abandoning convention, it introduces new conventions for organizing code into specialized layers: Infrastructure (logging, configuration, adapters), Data/Persistence (repositories, query objects, stripped-down ActiveRecord), Application/Service (service objects, form objects, filters), Business/Domain (policy objects, value objects, domain services), and Presentation (presenters, serializers, view components).

Each layer has a **single, clear responsibility**.  
Controllers become thin HTTP handlers that delegate to service objects.  
Models shed their bloat, separated into repositories (data access), domain models (business logic), and value objects (domain concepts).  
Business operations gain explicit names and locations—`User::RegistrationService` instead of scattered controller logic.  
Authorization centralizes in policy objects.  
Presentation logic moves to presenters and view components with explicit interfaces, not global helpers and instance variables.

This separation creates **explicit boundaries** that enable independent testing, parallel development, and confident refactoring.  
Changes to infrastructure don't ripple into business logic.  
Swapping implementations behind abstractions becomes trivial.  
Complex queries encapsulate in reusable query objects.  
Multi-model operations coordinate through service objects rather than callback chains.  
The code speaks the business language with domain-driven design principles, making intent visible and workflows documentable.

The tradeoff is **upfront complexity**.  
More files, more concepts, more decisions about which abstraction to use when.  
Initial development slows as you create multiple objects for operations that scaffolding would generate instantly.  
Teams need architectural discipline and shared understanding of layer boundaries.  
But for applications beyond MVP stage, growing teams, complex domains, or long-term maintenance horizons, this investment pays exponential returns in maintainability, testability, and scalability.

The philosophy is **pragmatic evolution, not dogmatic revolution**.  
Start with traditional Rails.  
Extract abstractions gradually when pain points emerge—when models exceed 200 lines, when controllers orchestrate complex workflows, when tests become slow and brittle.  
Follow Rails conventions in new layers, stay compatible with the ecosystem, and preserve developer happiness.  
The goal is a natural progression from simple MVC to structured layers, feeling like Rails grew up rather than being replaced.

## 2. Ecosystem Component Mapping to Layers

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

<table>
  <thead>
    <tr>
      <th align="left">Responsibility Area</th>
      <th align="left">Components/Location</th>
      <th align="left">Architecture Pattern</th>
      <th align="left">Behavior Pattern</th>
      <th align="left">Coupling</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2"><strong>Authorization</strong></td>
      <td><code>app/controllers/</code> + <code>app/models/</code></td>
      <td>MVC</td>
      <td>Checks scattered throughout</td>
      <td>None</td>
    </tr>
    <tr>
      <td><code>app/policies/</code></td>
      <td>Layered</td>
      <td>Single source of truth</td>
      <td>Clear</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Business Logic</strong></td>
      <td><code>app/controllers/</code> + <code>app/models/</code></td>
      <td>MVC</td>
      <td>Mixed everywhere, no boundaries</td>
      <td>Scattered</td>
    </tr>
    <tr>
      <td><code>app/models/</code> (domain)<br><code>app/services/domain/</code></td>
      <td>Layered</td>
      <td>Pure logic, persistence-agnostic</td>
      <td>Domain-focused</td>
    </tr>
    <tr>
      <td><strong>Code Sharing</strong></td>
      <td><code>concerns/</code></td>
      <td>MVC</td>
      <td>Hides complexity</td>
      <td>Horizontal</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Data Access</strong></td>
      <td>Direct ActiveRecord queries</td>
      <td>MVC</td>
      <td>Direct DB access from any layer</td>
      <td>None</td>
    </tr>
    <tr>
      <td><code>app/queries/</code></td>
      <td>Layered</td>
      <td>Collection-like interface</td>
      <td>Loose</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Data Persistence</strong></td>
      <td><code>app/models/</code><br><code>app/models/concerns/</code></td>
      <td>MVC</td>
      <td>Monolithic: persistence + logic + presentation + validation</td>
      <td>High</td>
    </tr>
    <tr>
      <td><code>app/repositories/</code><br><code>app/queries/</code><br><code>app/models/</code></td>
      <td>Layered</td>
      <td>Abstracted: Repository hides ActiveRecord, Query Objects encapsulate queries, Models stripped of business logic</td>
      <td>Separated</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Form Handling</strong></td>
      <td><code>app/controllers/</code> + <code>app/models/</code></td>
      <td>MVC</td>
      <td>Context-free, all mixed</td>
      <td>None</td>
    </tr>
    <tr>
      <td><code>app/forms/</code></td>
      <td>Layered</td>
      <td>Context-sensitive validations</td>
      <td>Clear</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>HTTP Handling</strong></td>
      <td><code>app/controllers/</code></td>
      <td>MVC</td>
      <td>God objects orchestrate all concerns</td>
      <td>Monolithic</td>
    </tr>
    <tr>
      <td><code>app/controllers/</code> (thin)</td>
      <td>Layered</td>
      <td>Receive, delegate, respond</td>
      <td>Minimal</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Infrastructure</strong></td>
      <td>Mixed locations</td>
      <td>MVC</td>
      <td>No boundaries between technical &amp; business</td>
      <td>None</td>
    </tr>
    <tr>
      <td><code>app/infrastructure/</code><br><code>lib/adapters/</code></td>
      <td>Layered</td>
      <td>Technical concerns isolated</td>
      <td>Agnostic</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Presentation Logic</strong></td>
      <td><code>app/models/</code> + <code>app/helpers/</code></td>
      <td>MVC</td>
      <td>Namespace pollution, methods in models</td>
      <td>Implicit</td>
    </tr>
    <tr>
      <td><code>app/presenters/</code><br><code>app/serializers/</code></td>
      <td>Layered</td>
      <td>UI Presenters, API Serializers</td>
      <td>Explicit</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Routing</strong></td>
      <td><code>config/routes.rb</code></td>
      <td>MVC</td>
      <td>Direct to controllers</td>
      <td>Tight</td>
    </tr>
    <tr>
      <td><code>config/routes.rb</code><br><code>lib/constraints/</code></td>
      <td>Layered</td>
      <td>Business logic in constraints</td>
      <td>Loose</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Serialization</strong></td>
      <td><code>app/models/</code> + <code>app/views/</code></td>
      <td>MVC</td>
      <td>Embedded in models</td>
      <td>Tight</td>
    </tr>
    <tr>
      <td><code>app/serializers/</code></td>
      <td>Layered</td>
      <td>Version-able, independent from models</td>
      <td>Loose</td>
    </tr>
    <tr>
      <td><strong>Use Case Orchestration</strong></td>
      <td><code>app/services/</code><br><code>app/commands/</code></td>
      <td>Layered</td>
      <td>Coordinate models, transactions</td>
      <td>Clear</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>View Rendering</strong></td>
      <td><code>app/views/</code></td>
      <td>MVC</td>
      <td>Implicit dependencies</td>
      <td>Implicit</td>
    </tr>
    <tr>
      <td><code>app/components/</code><br><code>app/views/</code></td>
      <td>Layered</td>
      <td>Ruby objects + templates, testable</td>
      <td>Explicit</td>
    </tr>
  </tbody>
</table>

## 3. Layer-by-Layer Comparison

### 3-1. Infrastructure Layer

Infrastructure is explicitly separated and abstracted, not mixed with domain logic.  
Changes to infrastructure don't affect business logic layers.

#### 3-1-1. Traditional MVC Rails Approach

Mixed throughout application (no dedicated layer).

|Location |Purpose |
|:-|:-|
|`app/jobs/` |Background jobs |
|`app/mailers/` |Email sending |
|`config/` |Configuration files |
|`lib/` |Utility code |
|ENV variables |Scattered in code |

|Aspect |Description |
|:-|:-|
|Infrastructure concerns |Mixed with application code |
|Rails abstractions |Direct use (Action Mailer, Active Job, caching) |
|Logging |`Rails.logger` called everywhere |
|ENV variables |Accessed directly throughout codebase |
|Database code |Specific code in models |
|Exception handling |Scattered |
|Instrumentation |Coupled to implementation |
|Boundaries |No clear boundaries between technical and business concerns |

#### 3-1-2. Layered Rails Approach

|Component Type |Location |Purpose |
|:-|:-|:-|
|**Configuration Objects** |`config/configs/`, `app/infrastructure/config/` |Settings classes, Feature flags, API configurations |
|**Logger Objects** |`app/infrastructure/logging/` |Structured loggers, Log tags, Custom formatters |
|**Delivery Objects / Notifiers** |`app/deliveries/`, `app/notifiers/` |Email delivery, SMS notifiers, Push notification services |
|**Background Jobs** |`app/jobs/` (infrastructure-specific) |Cache warming, Log cleanup, Monitoring jobs |
|**Adapters & Wrappers** |`lib/adapters/` |Cache adapters, Search adapters, Payment gateway wrappers |
|**Exception Tracking** |`app/infrastructure/exceptions/` |Error reporters, Exception handlers |
|**Instrumentation** |`app/infrastructure/instrumentation/` |Metrics collectors, Performance monitors, Log subscribers |

|Category |Details |
|:-|:-|
|Infrastructure Layer |Dedicated - separated from business logic |
|**Abstractions for technical concerns:** |Logger objects with structured logging and log tags<br>Configuration objects (Anyway Config) separating settings from secrets<br>Exception tracking with universal error-reporting interface<br>Instrumentation via Active Support Notifications + Yabeda<br>Monitoring services abstracted from instrumentation data |
|**Database abstractions:** |Database-level features (triggers, domain types) via gems like Logidze<br>Database constraints separate from validations<br>Custom types for reusability |
|Adapter pattern |For background jobs, caching, external services |
|Interfaces |Implementation-agnostic, allowing service extraction |

#### 3-1-3. Component Mapping Comparison

|Traditional MVC |Layered Rails |Layer |
|:-|:-|:-|
|Direct `Rails.logger` calls |Logger Objects |Infrastructure |
|ENV['KEY'] everywhere |Configuration Objects |Infrastructure |
|`UserMailer.welcome(user).deliver_later` |Delivery Objects + Notifiers |Infrastructure |
|Exception handling in controllers |Exception Tracking system |Infrastructure |
|Direct cache calls |Cache Adapters |Infrastructure |
|Background jobs (mixed) |Separated: Infrastructure Jobs vs Business Jobs |Infrastructure / Application |

### 3-2. Data/Persistence Layer

#### 3-2-1. Traditional MVC Rails Approach

**Components:** `app/models/` (monolithic ActiveRecord classes)

|Component |Description |
|:-|:-|
|Single Model class |Contains everything |
|Scopes |Defined in models |
|Associations |Defined in models |
|Concerns |For code sharing |

**Fat Models:** Single ActiveRecord class contains:

|Responsibility |Description |
|:-|:-|
|Persistence logic |CRUD operations |
|Validations |All contexts mixed |
|Callbacks |Technical and business |
|Scopes |Query methods |
|Associations |has_many, belongs_to |
|Business methods |Domain logic |
|Presentation helpers |`#full_name`, `#status_label` |
|Serialization logic |`#as_json` |

|Pattern |Description |
|:-|:-|
|Database queries |Direct in controllers/views: `User.where(active: true)` |
|Scope usage |Global, everywhere |
|Domain/persistence separation |None |
|Data access pattern |ActiveRecord only |

#### 3-2-2. Layered Rails Approach

**Components:** Separated into specialized abstractions

|Component Type |Location |Examples |Purpose |
|:-|:-|:-|:-|
|**Repository Objects** |`app/repositories/` |`UserRepository`, `PostRepository` |Data access abstraction |
|**Query Objects** |`app/queries/` |`ActiveUsersQuery`, `PublishedPostsQuery`, `SearchQuery` |Complex query encapsulation |
|**ActiveRecord Models** |`app/models/` (persistence only) |Database mappings, Associations, Basic scopes |Stripped of business logic |
|**Scopes** |In models or query objects |N/A |Reusable query fragments |

**Characteristics:**

**Repository Pattern:** Abstracts data access

|Aspect |Description |
|:-|:-|
|Interface |Collection-like interface to domain objects |
|Separation |Separates persistence from domain logic |
|Methods |`#find`, `#all`, `#create`, `#update`, `#delete` |
|Example |`PostRepository.published` instead of `Post.where(status: 'published')` |

**Query Objects:** Encapsulate complex queries with conventions

|Aspect |Description |
|:-|:-|
|Reusability |Reusable across contexts |
|Composability |Composable with Arel |
|Location |Separate from models |
|Naming |`<Subject><Condition>Query` or `<Action>Query` |

**Specialized patterns:**

|Pattern |Description |
|:-|:-|
|Scoping-based authorization |Authorization at data layer |
|N+1 problem handling |Handled at data layer |
|Denormalization |Through callbacks (technical, not business) |

**Separation of concerns:**

|Separation |Description |
|:-|:-|
|Domain vs. persistence |Domain logic separate from persistence |
|Validations vs. constraints |Business validations vs. database constraints |
|Events vs. callbacks |Domain events vs. ActiveRecord callbacks |

#### 3-2-3. Component Mapping Comparison

|Traditional MVC |Layered Rails |Layer |
|:-|:-|:-|
|`Post.where(status: 'published')` |`PostRepository.published` or `PublishedPostsQuery.call` |Data/Persistence |
|Complex queries in models/controllers |Query Objects |Data/Persistence |
|`Post.includes(:author).where(...)` |`PostRepository.with_authors` |Data/Persistence |
|ActiveRecord models (everything) |ActiveRecord (persistence only) |Data/Persistence |
|Scopes in models |Query Objects or Repository methods |Data/Persistence |
|Direct DB access everywhere |Repository interface |Data/Persistence |

#### 3-2-4. Key Contrast

Domain concepts are first-class citizens separated from persistence.  
Data access is controlled through repositories and query objects rather than direct ActiveRecord usage everywhere.

### 3-3. Application/Service Layer

#### 3-3-1. Traditional MVC Rails Approach

**Components:** No dedicated layer (logic scattered)

|Component |Location |Description |
|:-|:-|:-|
|Fat Controllers |`app/controllers/` |Business logic mixed in |
|Fat Models |`app/models/` |Business logic mixed in |
|Ad-hoc services |`app/services/` (if they exist) |Inconsistent patterns |
|Strong Parameters |In controllers |Input handling |

**Characteristics:**

|Issue |Description |
|:-|:-|
|Business logic location |In controllers (Fat Controllers) or pushed to models (Fat Models) |
|Service patterns |Ad-hoc with inconsistent patterns |
|Conventions |No clear conventions for service organization |
|God objects |Services often become "God objects" or utility bags |
|Model manipulation |Direct in multiple places |
|Filtering/search |In controller actions |
|Use case abstraction |No formal abstraction |

#### 3-3-2. Layered Rails Approach

**Components:** Dedicated Application/Service Layer

|Component Type |Location |Examples |
|:-|:-|:-|
|**Service Objects** |`app/services/` |`User::RegistrationService`, `Post::PublishService`, `Order::CheckoutService`, `Report::GenerateService` |
|**Command Objects** |`app/commands/` |`PublishPostCommand`, `ProcessPaymentCommand` (Alternative to service objects) |
|**Form Objects** |`app/forms/` |`RegistrationForm`, `PostPublishingForm`, `CheckoutForm`, `UserProfileForm` |
|**Filter Objects** |`app/filters/` |`ProjectsFilter`, `UsersSearchFilter`, `ProductCatalogFilter` |
|**Interactors** |`app/interactors/` (if using Interactor gem) |Chain multiple operations |

**Characteristics:**

**Service Objects:** Single business operations with clear conventions

|Aspect |Description |
|:-|:-|
|Naming |`<Subject>::<Action>Service` or `<Subject><Verb>` |
|Interface |Callable (`.call` method) |
|Return values |Specific result objects or use dry-monads |
|Principle |Single responsibility |
|Example |`User::RegistrationService.call(params)` instead of user creation in controller |

**Form Objects:** Handle user input outside models

|Aspect |Description |
|:-|:-|
|Operations |Multi-model (e.g., User registration + Project creation) |
|Validations |Context-sensitive |
|Compliance |ActiveModel API (Rails Way compatible) |
|Usage |Replace strong parameters for complex scenarios |
|Example |`RegistrationForm.new(params).save` handles User + Project + Welcome email |

**Filter Objects:** User-driven query building

|Aspect |Description |
|:-|:-|
|Purpose |Parameter-based filtering abstraction |
|Distinction |Separate from query objects (different purpose) |
|Convention |Convention-based (e.g., Rubanok gem) |
|Example |`ProjectsFilter.new(params).apply(Project.all)` |

**Command Objects:** Alternative pattern for operations

|Aspect |Description |
|:-|:-|
|Purpose |Explicit commands representing user intentions |
|Features |Can be queued, logged, undone |

#### 3-3-3. Component Mapping Comparison

|Traditional MVC |Layered Rails |Layer |
|:-|:-|:-|
|Controller actions with business logic |Service Objects / Command Objects |Application |
|Strong Parameters + Model validations |Form Objects |Application |
|Filtering in controller (`params[:status]`) |Filter Objects |Application |
|Multi-model operations in callbacks |Service Objects coordinating Domain Models |Application |
|`User.create(params)` in controller |`User::RegistrationService.call(params)` |Application |
|Order processing in controller |`Order::CheckoutService.call(order, params)` |Application |

#### 3-3-4. Key Contrast

Business operations are explicit, named, and follow conventions.  
Each service/form/filter has a single, clear responsibility rather than being scattered across controllers and models.

### 3-4. Business/Domain Logic Layer

#### 3-4-1. Traditional MVC Rails Approach

**Components:** Mixed in Models and Controllers

|Component |Location |
|:-|:-|
|Model validations |`app/models/` |
|Model callbacks |`app/models/` |
|Model methods (business logic) |`app/models/` |
|Helper methods |`app/helpers/` |
|Concerns |`app/models/concerns/` |

**Characteristics:**

Business rules embedded in:

|Location |Description |
|:-|:-|
|Model validations |Context-free, all mixed together |
|Callbacks |Mixing technical and business concerns |
|Controller actions |Business logic in controllers |
|Helper methods |Business logic scattered |

|Issue |Description |
|:-|:-|
|State transitions |Handled via callbacks |
|Business workflows |Implicit (hidden in callbacks) |
|Domain concepts |Not explicitly modeled |
|God objects |Accumulating responsibilities |
|Authorization logic |Scattered |
|Domain language |Not clear in code |

#### 3-4-2. Layered Rails Approach

**Components:** Dedicated Business/Domain Layer

|Component Type |Location |Examples |Description |
|:-|:-|:-|:-|
|**Domain Models** |`app/models/` or `app/domain/models/` |`User`, `Post`, `Order` |Business logic only, Rich domain behavior, No persistence details |
|**Policy Objects** |`app/policies/` |`PostPolicy`, `User::PublishPolicy` |Authorization rules (RBAC/ABAC models) |
|**Value Objects** |`app/values/` or `app/models/values/` |`Money`, `Email`, `Address`, `PhoneNumber`, `DateRange`, `Coordinates`, `Price` |Immutable domain concepts |
|**Domain Services** |`app/services/domain/` or `app/domain/services/` |`OrderFulfillmentService`, `InventoryManagementService` |Complex business workflows |
|**State Machines** |`app/models/` (with state_machine gems) |Order states, Post publishing workflow |State transition management |
|**Domain Events** |`app/events/` or with Rails Event Store |`UserRegistered`, `PostPublished`, `OrderPlaced` |Business event tracking |
|**Aggregates** |DDD pattern |Order + LineItems, Post + Comments |Entity clusters |

**Characteristics:**

**Explicit Domain Models:** Business entities with clear boundaries

|Aspect |Description |Example |
|:-|:-|:-|
|Persistence concerns |Free from persistence details |N/A |
|Domain behavior |Rich domain behavior |`Order#add_item(product)` instead of `OrderItem.create` |
|Domain concepts |Value objects for domain concepts |N/A |

**Policy Objects:** Authorization and business rules

|Aspect |Description |
|:-|:-|
|Usage |Can user publish post? `PostPolicy.new(user, post).publish?` |
|Logic centralization |Access control logic centralized |
|Models |ABAC (Attribute-Based) or RBAC (Role-Based) |

**Value Objects:** Domain concepts without identity

|Aspect |Description |
|:-|:-|
|Characteristics |Immutable, compared by value |
|Example |`Money.new(100, 'USD')` instead of separate amount/currency fields |

**Domain Services:** Complex business operations

|Aspect |Description |
|:-|:-|
|Coordination |Multi-model coordination |
|Orchestration |Business workflow orchestration |
|Event handling |Domain event handling |

**Business Rules Separation:**

|Rule Type |Implementation |
|:-|:-|
|Context-sensitive validations |In Form objects |
|Invariant rules |As database constraints |
|Complex transitions |State machines |
|Business rules |Policy objects |

**Domain Events:**

|Aspect |Description |
|:-|:-|
|Purpose |Replace callbacks for business workflows |
|Side effects |Event handlers |
|Pattern support |Event sourcing (Rails Event Store) |

**Domain-Driven Design (DDD) principles:**

|Principle |Description |
|:-|:-|
|Bounded contexts |Clear context boundaries |
|Aggregates and entities |Entity clustering |
|Domain language |In code |
|Ubiquitous language |Shared team vocabulary |

#### 3-4-3. Component Mapping Comparison

|Traditional MVC |Layered Rails |Layer |
|:-|:-|:-|
|Model business methods |Domain Models |Business/Domain |
|Authorization checks everywhere |Policy Objects |Business/Domain |
|Primitive values (strings, numbers) |Value Objects |Business/Domain |
|Complex workflows in callbacks |Domain Services |Business/Domain |
|`can? :publish, post` checks |`PostPolicy.new(user, post).publish?` |Business/Domain |
|`user.email` (string) |`Email.new(user.email_string)` (value object) |Business/Domain |
|State in database column |State Machine + Domain Events |Business/Domain |

#### 3-4-4. Key Contrast

Business logic is explicitly modeled using domain concepts and patterns.  
The code speaks the business language rather than being hidden in framework callbacks and validations.

### 3-5. Presentation Layer

#### 3-5-1. Traditional MVC Rails Approach

**Components:** Views and Helpers (mixed responsibilities)

|Component Type |Location |Examples |
|:-|:-|:-|
|**Views** |`app/views/` |ERB/HAML templates, Partials (`_user.html.erb`), Layouts |
|**Helpers** |`app/helpers/` |Global utility methods, Mixed concerns |
|**Jbuilder** |`app/views/*.json.jbuilder` |JSON templates |
|**Model methods** |In models |`#full_name`, `#status_badge`, `#formatted_date` (presentation logic) |

**Characteristics:**

**Views:** ERB templates with instance variables

|Aspect |Description |Example |
|:-|:-|:-|
|Interface contracts |None |N/A |
|Coupling |Tight coupling to controllers (via `@instance_variables`) |`<%= @user.full_name %>` directly from controller |
|Helpers |Global utility bags |N/A |
|Partials |Implicit dependencies |N/A |

**JSON Responses:**

|Aspect |Description |
|:-|:-|
|Methods |`#as_json` methods in models |
|Templates |Jbuilder templates (views) |
|Coupling |Tight coupling to model structure |

**Presentation logic in models:**

|Issue |Description |
|:-|:-|
|Methods |`User#full_name`, `Post#admin_status_icon` |
|Context awareness |Context-unaware methods |
|Method explosion |Multiple contexts → method name explosion (`#admin_display_name`, `#public_display_name`) |

|Problem |Description |
|:-|:-|
|Partials |Without clear interfaces |
|Component isolation |No isolation or testing |
|Helpers |Global (namespace pollution) |

#### 3-5-2. Layered Rails Approach

**Components:** Dedicated Presentation Layer with specialized abstractions

|Component Type |Location |Examples |
|:-|:-|:-|
|**Presenters/Decorators** |`app/presenters/` or `app/decorators/` |`UserPresenter`, `PostPresenter`, `DashboardPresenter`, `ReportPresenter` |
|**Serializers** |`app/serializers/` |`UserSerializer`, `PostSerializer`, `Api::V1::UserSerializer`, `Api::V2::UserSerializer` |
|**View Components** |`app/components/` |`SearchBox::Component`, `Card::Component`, `User::Profile::Component` |
|**View Models** |`app/view_models/` |Alternative to presenters |
|**Templates** |`app/views/` (simplified) |Strict locals, Component-based composition |
|**Partials** |With strict interface |Clear local variables defined |

**Characteristics:**

**View Components:** Component-based architecture

|Aspect |Description |Example |
|:-|:-|:-|
|Structure |Ruby objects + templates |N/A |
|Interfaces |Explicit (initialize parameters) |`render SearchBox::Component.new(url: search_path, placeholder: "Find...")` |
|Type safety |Strict locals |N/A |
|Previews |Component previews (like mailer previews) |N/A |
|Testing |Isolated testing |N/A |
|Composition |Slots API |N/A |
|Life cycle |Events (before_render) |N/A |

**Presenters/Decorators:**

|Aspect |Description |Example |
|:-|:-|:-|
|Separation |Presentation logic separated from models |`UserPresenter.new(user).display_name` instead of `user.full_name` |
|Context awareness |Context-aware representations |N/A |
|Multiple projections |Via different presenter classes |N/A |
|Types |Open (decorator pattern) vs. Closed presenters |N/A |
|Multi-model |For complex UIs (dashboards, reports) |N/A |

**Serializers:** API-specific presenters

|Aspect |Description |Example |
|:-|:-|:-|
|Implementation |Plain Ruby serializers |`UserSerializer.new(user).as_json` vs. `user#as_json` |
|Contracts |Clear serialization contracts |N/A |
|Independence |Independent from model structure |N/A |
|Versioning |Version-able API responses |N/A |

**Design Systems on Rails:**

|Aspect |Description |
|:-|:-|
|UI kit |Via components |
|Patterns |Reusable |
|Documentation |Lookbook for component documentation |
|Linting |With erb-lint |

**Authorization in views:**

|Aspect |Description |
|:-|:-|
|Control |Policy objects control visibility |
|Enforcement |Seamless authorization enforcement |
|View policies |View policy objects |
|Separation |Separate from form fields |

#### 3-5-3. Component Mapping Comparison

|Traditional MVC |Layered Rails |Layer |
|:-|:-|:-|
|`user.full_name` in views |`UserPresenter.new(user).display_name` |Presentation |
|`user#as_json` |`UserSerializer.new(user).as_json` |Presentation |
|Partials with implicit deps |View Components with explicit interface |Presentation |
|Jbuilder templates |Serializers |Presentation |
|Helper methods (`format_date`) |Presenter methods |Presentation |
|`@user` instance variable in views |Presenter passed explicitly |Presentation |
|Multiple `#*_display_name` methods |Different Presenter classes per context |Presentation |
|`<%= render 'user', user: @user %>` |`<%= render User::Card::Component.new(user: @user) %>` |Presentation |

#### 3-5-4. Key Contrast

Presentation logic is explicitly separated, component-based, testable, and context-aware.  
Views have clear interfaces rather than relying on controller instance variables and global helpers.

## 4. Pros and Cons of Layered Rails Application

### 4-1. Pros (Benefits of Layered Approach)

|Benefit |Description |
|:-|:-|
|**1. Maintainability** |• Clear separation of concerns<br>• Single responsibility for each class<br>• Easy to locate and modify specific functionality<br>• Reduced conceptual overhead per layer |
|**2. Testability** |• Layers can be tested in isolation<br>• Less coupling to framework<br>• Faster test suites (less database dependency)<br>• Mocking/stubbing at layer boundaries |
|**3. Scalability (Code)** |• Linear growth instead of exponential complexity<br>• Each layer can grow independently<br>• Abstraction distance prevents tight coupling<br>• Easier to onboard new developers (focused learning) |
|**4. Scalability (Performance)** |• Implementation-agnostic abstractions enable service extraction<br>• Can optimize hot paths without affecting business logic<br>• Easier to identify performance bottlenecks<br>• Infrastructure changes don't require business logic changes |
|**5. Flexibility** |• Swap implementations behind abstractions<br>• Multiple representations of same data (presenters)<br>• Context-sensitive behavior (form objects, validations)<br>• Adapter pattern enables library changes |
|**6. Code Reusability** |• Query objects composable across contexts<br>• Presenters reusable in different views<br>• Service objects callable from multiple entry points<br>• Value objects shared across domains |
|**7. Business Logic Clarity** |• Domain concepts explicitly modeled<br>• Business operations have clear names<br>• Workflows visible and documented in code<br>• Code speaks business language (DDD) |
|**8. Better Collaboration** |• Frontend/backend separation via presenters<br>• Clear contracts between layers<br>• Parallel development on different layers<br>• Less merge conflicts (focused changes) |
|**9. Gradual Adoption** |• Can be introduced incrementally<br>• Doesn't require rewriting entire application<br>• Works alongside existing Rails Way code<br>• Low risk refactoring |
|**10. Long-term Productivity** |• Reduced churn rate<br>• Fewer bugs from side effects<br>• Easier to add features in mature applications<br>• Technical debt managed systematically |

### 4-2. Cons (Challenges/Tradeoffs)

|Challenge |Description |
|:-|:-|
|**1. Initial Complexity** |• More files and directories to navigate<br>• Steeper learning curve for team<br>• Requires understanding of abstraction principles<br>• More upfront design decisions |
|**2. Slower Initial Development** |• Scaffolding doesn't create layered structure<br>• Need to create multiple objects for simple operations<br>• Takes longer to implement first features<br>• Overhead for prototyping/MVPs |
|**3. Increased Cognitive Load (Initially)** |• Must understand layer boundaries<br>• Need to know which abstraction to use when<br>• More concepts to keep in mind<br>• Requires architectural discipline |
|**4. Potential Over-Abstraction** |• Risk of creating abstractions too early<br>• Can lead to unnecessary complexity<br>• "Architecture astronaut" syndrome<br>• Premature optimization of code structure |
|**5. More Boilerplate Code** |• Multiple files for related functionality<br>• Interface definitions and contracts<br>• Adapter/wrapper objects<br>• Configuration for abstractions |
|**6. Team Alignment Required** |• Whole team must understand and follow patterns<br>• Inconsistent application breaks benefits<br>• Requires code review discipline<br>• Need conventions documentation |
|**7. Testing Overhead** |• More test files to maintain<br>• Need to test layer boundaries<br>• Integration tests still necessary<br>• Mock/stub management complexity |
|**8. Debugging Challenges** |• Call stack goes through multiple layers<br>• Harder to trace execution flow<br>• More indirection to follow<br>• Stack traces can be longer |
|**9. Deviation from Standard Rails** |• Less support from Rails generators<br>• Community resources focus on traditional approach<br>• Some gems assume Fat Models pattern<br>• May confuse Rails beginners |
|**10. Migration Cost** |• Effort to refactor existing applications<br>• Risk of breaking changes during migration<br>• Team training required<br>• Temporary coexistence of both patterns |

### 4-3. When to Use Layered Approach

|Category |Scenarios |
|:-|:-|
|**Good fit for:** |• Applications beyond MVP stage<br>• Growing code bases showing complexity signs<br>• Teams of 3+ developers<br>• Long-term maintenance expected<br>• Complex business domains<br>• Multiple API clients/consumers<br>• Applications with performance requirements<br>• Projects needing testability |
|**May be overkill for:** |• Simple CRUD applications<br>• Prototypes and MVPs<br>• Small teams (1-2 developers)<br>• Short-lived projects<br>• Applications with simple business logic<br>• Internal tools with limited scope |

### 4-4. The "Extended Rails Way" Philosophy

The book advocates for a **gradual, pragmatic approach**:

|Principle |Description |
|:-|:-|
|Start with traditional Rails Way |Begin with standard patterns |
|Extract abstractions when complexity emerges |Refactor only when needed |
|Follow Rails conventions in new layers |Maintain Rails idioms |
|Stay compatible with Rails ecosystem |Use ecosystem tools |
|Use gems that support layered approach |Leverage supporting libraries |
|Keep developer happiness as priority |Maintain productivity focus |

The goal is not to abandon Rails but to **extend it thoughtfully** when the application outgrows simple MVC structure. The layered approach should feel like a natural evolution of Rails, not a fight against the framework.

## 5. Key Takeaway

The **Traditional MVC Rails** approach excels at rapid initial development and simplicity, making it perfect for MVPs and small applications.  
However, it struggles with complexity as applications grow, leading to Fat Controllers, Fat Models, and maintenance challenges.

The **Layered Rails** approach trades some initial simplicity for long-term maintainability, testability, and scalability.  
It's the "Extended Rails Way" - respecting Rails conventions while introducing structured abstractions that make large applications manageable.

The transition should be **gradual and pragmatic**, not dogmatic.  
Extract abstractions when pain points emerge, not preemptively.  
The best Rails applications often use a hybrid approach, applying layering where complexity demands it while keeping simple CRUD operations traditional.
