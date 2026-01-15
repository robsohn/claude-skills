# Common Patterns

**Target Platform**: iOS 17+ / macOS 14+ using `@Observable` macro

## Core Architecture Pattern

### Observable Pattern

The `@Observable` macro from the Observation framework automatically tracks property changes:

```swift
import Observation

@Observable
class UserModel {
    var name: String = ""
    var email: String = ""
    var isLoggedIn: Bool = false
}

struct ContentView: View {
    @State private var user = UserModel()

    var body: some View {
        UserProfileView(user: user)
    }
}
```

**Key benefits**:

- No manual property wrappers needed
- Automatic change tracking for all properties
- More efficient runtime performance
- Cleaner, simpler code

### View Composition

Break complex views into smaller, focused components:

```swift
struct OrderSummaryView: View {
    let order: Order

    var body: some View {
        VStack(spacing: 16) {
            OrderHeaderView(order: order)
            OrderItemsListView(items: order.items)
            OrderTotalView(total: order.total)
        }
    }
}
```

### Prefer Value Types

Use structs for models when possible to leverage SwiftUI's efficient diffing:

```swift
struct Product: Identifiable, Equatable {
    let id: UUID
    var name: String
    var price: Decimal
    var quantity: Int
}
```

## Aggregate Root Models

### What is an Aggregate Root Model?

An aggregate model provides entities to views and coordinates data operations within a bounded context. It is NOT a view model per view—it's based on business domain boundaries.

### Basic Structure

```swift
import Observation

@MainActor
@Observable
class EntityModel {
    var entities: [Entity] = []
    var isLoading: Bool = false
    var error: Error?
    
    private let service: EntityService
    
    init(service: EntityService) {
        self.service = service
    }
    
    func loadAll() async {
        isLoading = true
        error = nil
        defer { isLoading = false }
        
        do {
            entities = try await service.fetchAll()
        } catch {
            self.error = error
        }
    }
    
    func create(_ entity: Entity) async {
        // Optimistic update: add immediately to UI
        entities.append(entity)
        
        do {
            let created = try await service.create(entity)
            // Replace with server response (may have ID, timestamps, etc.)
            if let index = entities.firstIndex(where: { $0.id == entity.id }) {
                entities[index] = created
            }
        } catch {
            // Rollback on failure
            entities.removeAll { $0.id == entity.id }
            self.error = error
        }
    }
    
    func update(_ entity: Entity) async {
        // Save original for rollback
        let originalIndex = entities.firstIndex(where: { $0.id == entity.id })
        let original = originalIndex.map { entities[$0] }
        
        // Optimistic update
        if let index = originalIndex {
            entities[index] = entity
        }
        
        do {
            let updated = try await service.update(entity)
            if let index = entities.firstIndex(where: { $0.id == entity.id }) {
                entities[index] = updated
            }
        } catch {
            // Rollback on failure
            if let index = originalIndex, let original = original {
                entities[index] = original
            }
            self.error = error
        }
    }
    
    func delete(_ entity: Entity) async {
        // Save original for rollback
        let originalIndex = entities.firstIndex(where: { $0.id == entity.id })
        
        // Optimistic update
        entities.removeAll { $0.id == entity.id }
        
        do {
            try await service.delete(entity.id)
        } catch {
            // Rollback on failure
            if let index = originalIndex {
                entities.insert(entity, at: index)
            }
            self.error = error
        }
    }
}
```

### Optimistic Updates Pattern

The examples above demonstrate **optimistic updates** - updating the UI immediately before server confirmation.

**When to use optimistic updates**:

- ✅ Create/update/delete operations where failure is rare
- ✅ Operations that should feel instant (like social media interactions)
- ✅ Non-critical operations with good error recovery

**When to avoid optimistic updates**:

- ❌ Payment processing or financial transactions
- ❌ Operations with high failure rates
- ❌ Critical operations where rollback could confuse users

**Pattern benefits**:

- Immediate UI feedback (better UX)
- App feels faster and more responsive
- Graceful degradation on network issues

**Pattern requirements**:

- Must handle rollback on failure
- Must show clear error messages
- Should indicate pending state (optional)

### When to Split Models

Split into multiple aggregate models when:

- Model grows beyond 500 lines
- Distinct business contexts emerge (e.g., Orders vs Inventory)
- Different teams own different domains
- Performance requires isolated state management

## Service Layer

### Purpose

Services handle data fetching/persistence and return model objects. They do NOT:

- Format data for display (that's the view's job)
- Contain business logic (that's the aggregate model's job)
- Know about views or UI state

### Basic Service Structure

```swift
protocol EntityService {
    func fetchAll() async throws -> [Entity]
    func fetchById(_ id: UUID) async throws -> Entity
    func create(_ entity: Entity) async throws -> Entity
    func update(_ entity: Entity) async throws -> Entity
    func delete(_ id: UUID) async throws
}

class NetworkEntityService: EntityService {
    private let baseURL: URL
    
    init(baseURL: URL) {
        self.baseURL = baseURL
    }
    
    func fetchAll() async throws -> [Entity] {
        let url = baseURL.appendingPathComponent("/entities")
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Entity].self, from: data)
    }
    
    func create(_ entity: Entity) async throws -> Entity {
        var request = URLRequest(url: baseURL.appendingPathComponent("/entities"))
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(entity)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(Entity.self, from: data)
    }
}
```

### Protocol-Based Design

Always define service protocols for:

- Testing with mock implementations
- Swapping between network/local/cache implementations
- Dependency injection

## SwiftUI View Integration

### Injecting Models

Use `@Environment` for globally-accessible aggregate models:

```swift
@main
struct MyApp: App {
    @State private var model = EntityModel(service: NetworkEntityService(baseURL: /* ... */))
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(model)
        }
    }
}
```

### Accessing Models in Views

```swift
struct ContentView: View {
    @Environment(EntityModel.self) private var model 
    
    var body: some View {
        List(model.entities) { entity in
            EntityRow(entity: entity)
        }
        .task {
            await model.loadAll() 
        }
    }
}
```

### Form Validation

Views handle UI validation directly:

```swift
struct CreateEntityView: View {
    @State private var name: String = ""
    @State private var nameError: String = ""
    @Environment(EntityModel.self) private var model 
    @Environment(\.dismiss) private var dismiss
    
    private var isValid: Bool {
        nameError = ""
        if name.isEmpty {
            nameError = "Name cannot be empty"
        }
        return nameError.isEmpty
    }
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            if !nameError.isEmpty {
                Text(nameError).foregroundColor(.red)
            }
        }
        .toolbar {
            Button("Save") {
                Task {
                    guard isValid else { return }
                    let entity = Entity(name: name)
                    await model.create(entity)
                    dismiss()
                }
            }
        }
    }
}
```

For complex forms, extract validation into a separate struct:

```swift
struct EntityFormState {
    var name: String = ""
    var email: String = ""
    var nameError: String = ""
    var emailError: String = ""
    
    mutating func validate() -> Bool {
        nameError = ""
        emailError = ""
        
        if name.isEmpty {
            nameError = "Name required"
        }
        if !email.contains("@") {
            emailError = "Invalid email"
        }
        
        return nameError.isEmpty && emailError.isEmpty
    }
}
```

## Data Flow Patterns

### @State for Local View State

Use for view-specific state that doesn't need sharing:

```swift
@State private var isExpanded: Bool = false
@State private var searchText: String = ""
```

### @State for View-Owned Models

Use when a view creates and owns a model:

```swift
@State private var model = EntityModel(service: service)
```

### @Environment for Shared Models

Use for aggregate models shared across the app:

```swift
@Environment(EntityModel.self) private var model
```

## Observation Framework

The `@Observable` macro automatically tracks all property changes without any manual annotations:

```swift
import Observation

@Observable
class EntityModel {
    var entities: [Entity] = []        // Auto-tracked
    var isLoading: Bool = false        // Auto-tracked
    var error: Error?                  // Auto-tracked
}
```

**Key features**:

- ✅ All properties are automatically tracked
- ✅ No property wrappers needed
- ✅ More efficient than manual tracking
- ✅ Access via `@Environment` in views
- ✅ Cleaner, simpler code

## Loading States Pattern

Show loading indicators and handle errors gracefully:

```swift
struct ContentView: View {
    @Environment(EntityModel.self) private var model
    
    var body: some View {
        Group {
            if model.isLoading {
                ProgressView("Loading...")
            } else if let error = model.error {
                ContentUnavailableView(
                    "Error Loading Data",
                    systemImage: "exclamationmark.triangle",
                    description: Text(error.localizedDescription)
                )
            } else if model.entities.isEmpty {
                ContentUnavailableView(
                    "No Data",
                    systemImage: "tray",
                    description: Text("Add your first item")
                )
            } else {
                entityList
            }
        }
    }
    
    private var entityList: some View {
        List(model.entities) { entity in
            EntityRow(entity: entity)
        }
    }
}
```

## Caching Pattern

Implement simple time-based caching to reduce network requests:

```swift
@MainActor
@Observable
class EntityModel {
    var entities: [Entity] = []
    private var cacheTimestamp: Date?
    private let cacheTimeout: TimeInterval = 300 // 5 minutes
    
    private let service: EntityService
    
    func loadAll(forceRefresh: Bool = false) async {
        // Check if cache is still valid
        if !forceRefresh,
           let timestamp = cacheTimestamp,
           Date().timeIntervalSince(timestamp) < cacheTimeout {
            return // Use cached data
        }
        
        // Fetch fresh data
        do {
            entities = try await service.fetchAll()
            cacheTimestamp = Date()
        } catch {
            // Handle error
        }
    }
}
```

## Pagination Pattern

Load data in pages for better performance with large datasets:

```swift
@MainActor
@Observable
class EntityModel {
    var entities: [Entity] = []
    var isLoading: Bool = false
    var hasMorePages: Bool = true
    private var currentPage: Int = 0
    
    private let service: EntityService
    
    func loadNextPage() async {
        guard !isLoading && hasMorePages else { return }
        
        isLoading = true
        defer { isLoading = false }
        
        do {
            let newEntities = try await service.fetchPage(currentPage)
            if newEntities.isEmpty {
                hasMorePages = false
            } else {
                entities.append(contentsOf: newEntities)
                currentPage += 1
            }
        } catch {
            // Handle error
        }
    }
    
    func refresh() async {
        currentPage = 0
        hasMorePages = true
        entities = []
        await loadNextPage()
    }
}
```

## Search and Filter Pattern

Provide search and filtering capabilities:

```swift
@MainActor
@Observable
class EntityModel {
    var entities: [Entity] = []
    var searchText: String = ""
    var selectedCategory: Category?
    
    var filteredEntities: [Entity] {
        var result = entities
        
        // Apply search
        if !searchText.isEmpty {
            result = result.filter { entity in
                entity.name.localizedCaseInsensitiveContains(searchText)
            }
        }
        
        // Apply category filter
        if let category = selectedCategory {
            result = result.filter { $0.category == category }
        }
        
        return result
    }
}

struct ContentView: View {
    @Environment(EntityModel.self) private var model
    
    var body: some View {
        List(model.filteredEntities) { entity in
            EntityRow(entity: entity)
        }
        .searchable(text: Binding(
            get: { model.searchText },
            set: { model.searchText = $0 }
        ))
    }
}
```

## Sorting Pattern

Provide sorting capabilities:

```swift
enum SortOrder {
    case nameAscending
    case nameDescending
    case dateNewest
    case dateOldest
}

@MainActor
@Observable
class EntityModel {
    var entities: [Entity] = []
    var sortOrder: SortOrder = .nameAscending
    
    var sortedEntities: [Entity] {
        switch sortOrder {
        case .nameAscending:
            return entities.sorted { $0.name < $1.name }
        case .nameDescending:
            return entities.sorted { $0.name > $1.name }
        case .dateNewest:
            return entities.sorted { $0.createdAt > $1.createdAt }
        case .dateOldest:
            return entities.sorted { $0.createdAt < $1.createdAt }
        }
    }
}
```
