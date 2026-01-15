# SwiftUI MV Pattern - Complete Reference Guide

This guide provides detailed implementation examples and best practices for the Model-View (MV) pattern in SwiftUI, based on practical experience and Apple's recommendations.

## Philosophy

**Core Principle**: SwiftUI views are already view models. Don't add unnecessary complexity by creating separate view model classes for each view.

**What This Means**:

- Views use property wrappers (`@State`, `@Environment`) for state management (iOS 17+)
- Views handle UI validation and presentation formatting
- Aggregate models coordinate business logic and data operations
- Service layers handle networking and persistence

## Architecture Layers

### Small Application (Single Aggregate Model)

```
┌─────────────────────┐
│       View          │
│   (SwiftUI View)    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Aggregate Model    │
│   (@Observable)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Service Layer     │
│  (Network/Storage)  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       Server        │
└─────────────────────┘
```

### Large Application (Multiple Aggregate Models)

```
Views communicate with multiple aggregate models based on bounded context:

OrderView ──────► OrderModel ──────► OrderService
ProductView ─────► ProductModel ────► ProductService
CustomerView ────► CustomerModel ───► CustomerService

Aggregate models can communicate with each other when needed:
OrderModel ─────────► CustomerModel (to get customer data)
```

## Complete Implementation Example

### 1. Domain Model

```swift
struct Order: Codable, Identifiable {
    let id: UUID?
    var name: String
    var coffeeName: String
    var total: Double
    var size: CoffeeSize
}

enum CoffeeSize: String, Codable, CaseIterable, Identifiable {
    case small = "Small"
    case medium = "Medium"
    case large = "Large"
    
    var id: String { rawValue }
}
```

### 2. Service Layer

```swift
enum OrderServiceError: Error {
    case badURL
    case requestFailed
    case decodingError
}

protocol OrderServiceProtocol {
    func getAllOrders() async throws -> [Order]
    func placeOrder(_ order: Order) async throws -> Order
    func updateOrder(_ order: Order) async throws -> Order
    func deleteOrder(_ id: UUID) async throws
}

class OrderService: OrderServiceProtocol {
    private let baseURL: URL
    
    init(baseURL: URL) {
        self.baseURL = baseURL
    }
    
    func getAllOrders() async throws -> [Order] {
        guard let url = URL(string: "/orders", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Order].self, from: data)
    }
    
    func placeOrder(_ order: Order) async throws -> Order {
        guard let url = URL(string: "/orders", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(order)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(Order.self, from: data)
    }
    
    func updateOrder(_ order: Order) async throws -> Order {
        guard let id = order.id,
              let url = URL(string: "/orders/\(id)", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "PUT"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(order)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(Order.self, from: data)
    }
    
    func deleteOrder(_ id: UUID) async throws {
        guard let url = URL(string: "/orders/\(id)", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "DELETE"
        
        let (_, _) = try await URLSession.shared.data(for: request)
    }
}
```

### 3. Aggregate Root Model

```swift
import Observation

enum OrderError: Error {
    case custom(String)
}

@MainActor
@Observable
class OrderModel {
    var orders: [Order] = []
    var isLoading: Bool = false
    var errorMessage: String?
    
    private let orderService: OrderServiceProtocol
    
    init(orderService: OrderServiceProtocol) {
        self.orderService = orderService
    }
    
    // MARK: - Data Operations
    
    func getAllOrders() async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        
        do {
            orders = try await orderService.getAllOrders()
        } catch {
            errorMessage = "Failed to load orders: \(error.localizedDescription)"
        }
    }
    
    func placeOrder(_ order: Order) async {
        do {
            let newOrder = try await orderService.placeOrder(order)
            orders.append(newOrder)
        } catch {
            errorMessage = "Failed to place order: \(error.localizedDescription)"
        }
    }
    
    func updateOrder(_ order: Order) async {
        do {
            let updatedOrder = try await orderService.updateOrder(order)
            
            guard let index = orders.firstIndex(where: { $0.id == updatedOrder.id }) else {
                errorMessage = "Order not found in local collection"
                return
            }
            
            orders[index] = updatedOrder
        } catch {
            errorMessage = "Failed to update order: \(error.localizedDescription)"
        }
    }
    
    func deleteOrder(_ order: Order) async {
        guard let id = order.id else {
            errorMessage = "Cannot delete order without ID"
            return
        }
        
        do {
            try await orderService.deleteOrder(id)
            orders.removeAll { $0.id == id }
        } catch {
            errorMessage = "Failed to delete order: \(error.localizedDescription)"
        }
    }
    
    // MARK: - View Support Operations
    
    func sortOrders(by criteria: OrderSortCriteria) {
        switch criteria {
        case .name:
            orders.sort { $0.name < $1.name }
        case .total:
            orders.sort { $0.total < $1.total }
        case .size:
            orders.sort { $0.size.rawValue < $1.size.rawValue }
        }
    }
    
    func filterOrders(by size: CoffeeSize?) -> [Order] {
        guard let size = size else { return orders }
        return orders.filter { $0.size == size }
    }
    
    func searchOrders(query: String) -> [Order] {
        guard !query.isEmpty else { return orders }
        return orders.filter { 
            $0.name.localizedCaseInsensitiveContains(query) ||
            $0.coffeeName.localizedCaseInsensitiveContains(query)
        }
    }
}

enum OrderSortCriteria {
    case name
    case total
    case size
}
```

### 4. SwiftUI Views

#### Main View (List)

```swift
struct OrderListView: View {
    @Environment(OrderModel.self) private var model
    @State private var showingAddOrder = false
    @State private var selectedOrder: Order?
    
    var body: some View {
        NavigationStack {
            Group {
                if model.isLoading {
                    ProgressView("Loading orders...")
                } else if let error = model.errorMessage {
                    ContentUnavailableView(
                        "Error Loading Orders",
                        systemImage: "exclamationmark.triangle",
                        description: Text(error)
                    )
                } else if model.orders.isEmpty {
                    ContentUnavailableView(
                        "No Orders",
                        systemImage: "cup.and.saucer",
                        description: Text("Add your first coffee order")
                    )
                } else {
                    ordersList
                }
            }
            .navigationTitle("Coffee Orders")
            .toolbar {
                Button {
                    showingAddOrder = true
                } label: {
                    Label("Add Order", systemImage: "plus")
                }
            }
            .sheet(isPresented: $showingAddOrder) {
                AddOrderView()
            }
            .sheet(item: $selectedOrder) { order in
                AddOrderView(order: order)
            }
            .task {
                await model.getAllOrders()
            }
        }
    }
    
    private var ordersList: some View {
        List {
            ForEach(model.orders) { order in
                OrderRow(order: order)
                    .onTapGesture {
                        selectedOrder = order
                    }
            }
            .onDelete { indexSet in
                for index in indexSet {
                    let order = model.orders[index]
                    Task {
                        await model.deleteOrder(order)
                    }
                }
            }
        }
    }
}
```

#### Row View

```swift
struct OrderRow: View {
    let order: Order
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(order.name)
                .font(.headline)
            
            HStack {
                Text(order.coffeeName)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
                
                Spacer()
                
                Text(order.size.rawValue)
                    .font(.caption)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 2)
                    .background(.quaternary)
                    .clipShape(Capsule())
            }
            
            Text("$\(order.total, specifier: "%.2f")")
                .font(.subheadline)
                .fontWeight(.medium)
        }
        .padding(.vertical, 4)
    }
}
```

#### Form View

```swift
struct AddOrderView: View {
    @Environment(OrderModel.self) private var model
    @Environment(\.dismiss) private var dismiss
    
    let order: Order?
    @State private var formState = OrderFormState()
    
    init(order: Order? = nil) {
        self.order = order
    }
    
    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("Name", text: $formState.name)
                    if !formState.nameError.isEmpty {
                        Text(formState.nameError)
                            .font(.caption)
                            .foregroundStyle(.red)
                    }
                }
                
                Section {
                    TextField("Coffee", text: $formState.coffeeName)
                    if !formState.coffeeNameError.isEmpty {
                        Text(formState.coffeeNameError)
                            .font(.caption)
                            .foregroundStyle(.red)
                    }
                }
                
                Section {
                    TextField("Total", text: $formState.total)
                        .keyboardType(.decimalPad)
                    if !formState.totalError.isEmpty {
                        Text(formState.totalError)
                            .font(.caption)
                            .foregroundStyle(.red)
                    }
                }
                
                Section {
                    Picker("Size", selection: $formState.size) {
                        ForEach(CoffeeSize.allCases) { size in
                            Text(size.rawValue).tag(size)
                        }
                    }
                    .pickerStyle(.segmented)
                }
            }
            .navigationTitle(order == nil ? "New Order" : "Edit Order")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") {
                        dismiss()
                    }
                }
                
                ToolbarItem(placement: .confirmationAction) {
                    Button(order == nil ? "Add" : "Update") {
                        Task {
                            await saveOrder()
                        }
                    }
                }
            }
            .onAppear {
                if let order {
                    formState.name = order.name
                    formState.coffeeName = order.coffeeName
                    formState.total = String(order.total)
                    formState.size = order.size
                }
            }
        }
    }
    
    private func saveOrder() async {
        guard formState.validate() else { return }
        
        let orderData = Order(
            id: order?.id,
            name: formState.name,
            coffeeName: formState.coffeeName,
            total: Double(formState.total)!,
            size: formState.size
        )
        
        if order != nil {
            await model.updateOrder(orderData)
        } else {
            await model.placeOrder(orderData)
        }
        dismiss()
    }
}
```

#### Form State

```swift
struct OrderFormState {
    var name: String = ""
    var coffeeName: String = ""
    var total: String = ""
    var size: CoffeeSize = .medium
    
    var nameError: String = ""
    var coffeeNameError: String = ""
    var totalError: String = ""
    
    mutating func validate() -> Bool {
        nameError = ""
        coffeeNameError = ""
        totalError = ""
        
        if name.isEmpty {
            nameError = "Name is required"
        }
        
        if coffeeName.isEmpty {
            coffeeNameError = "Coffee name is required"
        }
        
        if total.isEmpty {
            totalError = "Total is required"
        } else if Double(total) == nil {
            totalError = "Total must be a valid number"
        } else if let value = Double(total), value <= 0 {
            totalError = "Total must be greater than 0"
        }
        
        return nameError.isEmpty && coffeeNameError.isEmpty && totalError.isEmpty
    }
}
```

### 5. App Entry Point

```swift
import SwiftUI

@main
struct CoffeeApp: App {
    private let orderService = OrderService(
        baseURL: URL(string: "https://api.example.com")!
    )
    @State private var orderModel: OrderModel
    
    init() {
        let service = OrderService(
            baseURL: URL(string: "https://api.example.com")!
        )
        _orderModel = State(initialValue: OrderModel(orderService: service))
    }
    
    var body: some Scene {
        WindowGroup {
            OrderListView()
                .environment(orderModel)
        }
    }
}
```

## UI Validation vs Business Rules

**UI Validation** (handled in views):

- Field not empty
- Valid format (email, phone)
- Number ranges
- Required fields

**Business Rules** (handled in aggregate model or server):

- Credit score → APR calculation
- Inventory availability
- User permissions
- Complex domain logic

Example of UI validation in HTML:

```html
<input type="text" required minlength="5" maxlength="100" />
```

This is purely UI validation, not business logic.

## Testing Patterns

### Mock Service for Testing

```swift
class MockOrderService: OrderServiceProtocol {
    var mockOrders: [Order] = []
    var shouldFail = false
    
    func getAllOrders() async throws -> [Order] {
        if shouldFail {
            throw OrderServiceError.requestFailed
        }
        return mockOrders
    }
    
    func placeOrder(_ order: Order) async throws -> Order {
        if shouldFail {
            throw OrderServiceError.requestFailed
        }
        let newOrder = Order(
            id: UUID(),
            name: order.name,
            coffeeName: order.coffeeName,
            total: order.total,
            size: order.size
        )
        mockOrders.append(newOrder)
        return newOrder
    }
    
    func updateOrder(_ order: Order) async throws -> Order {
        if shouldFail {
            throw OrderServiceError.requestFailed
        }
        guard let index = mockOrders.firstIndex(where: { $0.id == order.id }) else {
            throw OrderServiceError.requestFailed
        }
        mockOrders[index] = order
        return order
    }
    
    func deleteOrder(_ id: UUID) async throws {
        if shouldFail {
            throw OrderServiceError.requestFailed
        }
        mockOrders.removeAll { $0.id == id }
    }
}
```

### Testing Aggregate Model

```swift
import XCTest

@MainActor
class OrderModelTests: XCTestCase {
    func testPlaceOrder() async throws {
        // Arrange
        let mockService = MockOrderService()
        let model = OrderModel(orderService: mockService)
        let order = Order(
            id: nil,
            name: "John",
            coffeeName: "Latte",
            total: 5.99,
            size: .medium
        )
        
        // Act
        await model.placeOrder(order)
        
        // Assert
        XCTAssertEqual(model.orders.count, 1)
        XCTAssertEqual(model.orders.first?.name, "John")
        XCTAssertNil(model.errorMessage)
    }
    
    func testSortOrders() async throws {
        // Arrange
        let mockService = MockOrderService()
        mockService.mockOrders = [
            Order(id: UUID(), name: "Zoe", coffeeName: "Cappuccino", total: 4.50, size: .small),
            Order(id: UUID(), name: "Alice", coffeeName: "Espresso", total: 3.00, size: .small),
            Order(id: UUID(), name: "Bob", coffeeName: "Americano", total: 3.50, size: .medium)
        ]
        let model = OrderModel(orderService: mockService)
        await model.getAllOrders()
        
        // Act
        model.sortOrders(by: .name)
        
        // Assert
        XCTAssertEqual(model.orders.first?.name, "Alice")
        XCTAssertEqual(model.orders.last?.name, "Zoe")
    }
    
    func testErrorHandling() async throws {
        // Arrange
        let mockService = MockOrderService()
        mockService.shouldFail = true
        let model = OrderModel(orderService: mockService)
        
        // Act
        await model.getAllOrders()
        
        // Assert
        XCTAssertNotNil(model.errorMessage)
        XCTAssertTrue(model.orders.isEmpty)
    }
}
```

## Common Pitfalls to Avoid

### ❌ Don't Do This

```swift
// Creating a view model per view
class OrderListViewModel: ObservableObject {
    @Published var orders: [Order] = []
    // This is unnecessary complexity
}

// Passing EnvironmentObject to a separate view model
class OrderViewModel: ObservableObject {
    let model: Model
    
    init(model: Model) {
        self.model = model  // Don't do this
    }
}
```

### ✅ Do This Instead

```swift
// Access model directly in view
struct OrderListView: View {
    @Environment(OrderModel.self) private var model
    
    var body: some View {
        List(model.orders) { order in
            OrderRow(order: order)
        }
    }
}
```

## Performance Considerations

### View Re-evaluation vs Re-rendering

```swift
struct DemoView: View {
    @State private var name: String = ""
    
    var body: some View {
        let _ = Self._printChanges()  // Debug view updates
        
        VStack {
            List(1...20, id: \.self) { index in
                Text("\(index)")  // Not re-rendered when name changes
            }
            
            TextField("Name", text: $name)  // Only this re-renders
        }
    }
}
```

When `name` changes:

- Body is **re-evaluated** (fast)
- Only changed views are **re-rendered** (TextField)
- Static views (List items) are not re-rendered

### Splitting Environment Models

If performance issues arise, split large models:

```swift
@Environment(OrderModel.self) private var orderModel
@Environment(ProductModel.self) private var productModel
@Environment(UserModel.self) private var userModel
```

This prevents unnecessary re-evaluations when only one model changes.

## Key Takeaways

1. **View = View Model**: Don't create separate ViewModels
2. **Aggregate Models**: Based on bounded context, not screen count
3. **Protocol Services**: Always use protocols for testability
4. **Direct Access**: Views directly consume model objects
5. **UI Validation**: Keep in views, business rules in models/server
6. **@MainActor**: Always annotate aggregate models
7. **Testing**: Focus on meaningful behavior tests, not mocks
8. **iOS 17+**: Use `@Observable` and `@Environment` for modern apps
9. **Error Handling**: Use `defer` pattern and handle errors internally
10. **Dependency Injection**: Use protocols and `@State` for models

## Additional Resources

- [Apple WWDC: Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/)
- [Apple WWDC: Discover Observation in SwiftUI](https://developer.apple.com/videos/play/wwdc2023/10149/)
- [Apple Sample: Fruta App](https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui)
- [Apple Sample: Food Truck App](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app)
