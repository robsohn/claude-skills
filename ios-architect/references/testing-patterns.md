# Testing Patterns

## Unit Tests for Aggregate Models

Test business logic in aggregate models using mock services:

```swift
class MockEntityService: EntityService {
    var entities: [Entity] = []
    var shouldThrowError = false
    
    func fetchAll() async throws -> [Entity] {
        if shouldThrowError {
            throw ServiceError.networkError
        }
        return entities
    }
    
    func create(_ entity: Entity) async throws -> Entity {
        if shouldThrowError {
            throw ServiceError.networkError
        }
        let created = Entity(id: UUID(), name: entity.name)
        entities.append(created)
        return created
    }
}

@MainActor
class EntityModelTests: XCTestCase {
    func testCreateEntity() async throws {
        // Arrange
        let service = MockEntityService()
        let model = EntityModel(service: service)
        
        // Act
        let entity = Entity(name: "Test")
        await model.create(entity)
        
        // Assert
        XCTAssertEqual(model.entities.count, 1)
        XCTAssertEqual(model.entities.first?.name, "Test")
    }
    
    func testCreateEntityOptimisticUpdate() async throws {
        // Arrange
        let service = MockEntityService()
        service.shouldThrowError = true
        let model = EntityModel(service: service)
        
        // Act
        let entity = Entity(name: "Test")
        await model.create(entity)
        
        // Assert - should rollback on error
        XCTAssertEqual(model.entities.count, 0)
        XCTAssertNotNil(model.error)
    }
    
    func testLoadAll() async throws {
        // Arrange
        let service = MockEntityService()
        service.entities = [
            Entity(id: UUID(), name: "Entity 1"),
            Entity(id: UUID(), name: "Entity 2")
        ]
        let model = EntityModel(service: service)
        
        // Act
        await model.loadAll()
        
        // Assert
        XCTAssertEqual(model.entities.count, 2)
        XCTAssertFalse(model.isLoading)
    }
    
    func testUpdateEntity() async throws {
        // Arrange
        let service = MockEntityService()
        let original = Entity(id: UUID(), name: "Original")
        service.entities = [original]
        let model = EntityModel(service: service)
        await model.loadAll()
        
        // Act
        var updated = original
        updated.name = "Updated"
        await model.update(updated)
        
        // Assert
        XCTAssertEqual(model.entities.first?.name, "Updated")
    }
    
    func testDeleteEntity() async throws {
        // Arrange
        let service = MockEntityService()
        let entity = Entity(id: UUID(), name: "To Delete")
        service.entities = [entity]
        let model = EntityModel(service: service)
        await model.loadAll()
        
        // Act
        await model.delete(entity)
        
        // Assert
        XCTAssertEqual(model.entities.count, 0)
    }
}
```

## Integration Tests

Test service layer against real APIs or test databases:

```swift
class NetworkEntityServiceTests: XCTestCase {
    func testFetchAll() async throws {
        // Arrange
        let testServerURL = URL(string: "https://test-api.example.com")!
        let service = NetworkEntityService(baseURL: testServerURL)
        
        // Act
        let entities = try await service.fetchAll()
        
        // Assert
        XCTAssertFalse(entities.isEmpty)
    }
    
    func testCreateEntity() async throws {
        // Arrange
        let testServerURL = URL(string: "https://test-api.example.com")!
        let service = NetworkEntityService(baseURL: testServerURL)
        let entity = Entity(name: "Test Entity")
        
        // Act
        let created = try await service.create(entity)
        
        // Assert
        XCTAssertNotNil(created.id)
        XCTAssertEqual(created.name, "Test Entity")
    }
}
```

## UI Tests

Use XCTest UI testing for end-to-end validation:

```swift
class EntityAppUITests: XCTestCase {
    func testCreateFlow() throws {
        // Arrange
        let app = XCUIApplication()
        app.launch()
        
        // Act
        app.buttons["Add"].tap()
        
        let nameField = app.textFields["Name"]
        nameField.tap()
        nameField.typeText("Test Entity")
        
        app.buttons["Save"].tap()
        
        // Assert
        XCTAssertTrue(app.staticTexts["Test Entity"].exists)
    }
    
    func testDeleteFlow() throws {
        // Arrange
        let app = XCUIApplication()
        app.launch()
        
        // Ensure an entity exists
        if !app.staticTexts["Test Entity"].exists {
            app.buttons["Add"].tap()
            app.textFields["Name"].tap()
            app.textFields["Name"].typeText("Test Entity")
            app.buttons["Save"].tap()
        }
        
        // Act
        let entityCell = app.staticTexts["Test Entity"]
        entityCell.swipeLeft()
        app.buttons["Delete"].tap()
        
        // Assert
        XCTAssertFalse(app.staticTexts["Test Entity"].exists)
    }
    
    func testSearchFlow() throws {
        // Arrange
        let app = XCUIApplication()
        app.launch()
        
        // Act
        let searchField = app.searchFields.firstMatch
        searchField.tap()
        searchField.typeText("Test")
        
        // Assert
        XCTAssertTrue(app.staticTexts.containing(NSPredicate(format: "label CONTAINS 'Test'")).firstMatch.exists)
    }
    
    func testErrorHandling() throws {
        // Arrange
        let app = XCUIApplication()
        app.launchArguments = ["--testing", "--network-error"]
        app.launch()
        
        // Act
        app.buttons["Refresh"].tap()
        
        // Assert
        XCTAssertTrue(app.staticTexts["Error Loading Data"].exists)
    }
}
```

## Testing Best Practices

### Use Dependency Injection

Always inject services to enable testing:

```swift
// Good
class EntityModel {
    private let service: EntityService
    
    init(service: EntityService) {
        self.service = service
    }
}

// Bad
class EntityModel {
    private let service = NetworkEntityService() // Hard to test
}
```

### Test Business Logic, Not Implementation

Focus on behavior, not internal implementation:

```swift
// Good - tests behavior
func testCreateEntityAddsToList() async {
    await model.create(entity)
    XCTAssertEqual(model.entities.count, 1)
}

// Bad - tests implementation details
func testCreateEntityCallsService() async {
    // Don't test that it calls the service, test the outcome
}
```

### Use @MainActor for Model Tests

Aggregate models run on MainActor, so tests must too:

```swift
@MainActor
class EntityModelTests: XCTestCase {
    func testSomething() async {
        // Test code here
    }
}
```

### Test Error Scenarios

Always test both success and failure paths:

```swift
func testCreateEntitySuccess() async {
    mockService.shouldThrowError = false
    await model.create(entity)
    XCTAssertNotNil(model.entities.first)
}

func testCreateEntityFailure() async {
    mockService.shouldThrowError = true
    await model.create(entity)
    XCTAssertNotNil(model.error)
}
```

### Use Test Doubles

Create focused test doubles for different scenarios:

```swift
class AlwaysFailingService: EntityService {
    func fetchAll() async throws -> [Entity] {
        throw ServiceError.networkError
    }
}

class AlwaysEmptyService: EntityService {
    func fetchAll() async throws -> [Entity] {
        return []
    }
}

class SlowService: EntityService {
    func fetchAll() async throws -> [Entity] {
        try await Task.sleep(for: .seconds(2))
        return []
    }
}
```

### Test Async Operations

Use async/await in tests for async code:

```swift
func testAsyncOperation() async throws {
    // Act
    await model.loadAll()
    
    // Assert
    XCTAssertFalse(model.isLoading)
    XCTAssertNotNil(model.entities)
}
```

## Snapshot Testing (Optional)

For UI consistency, consider snapshot testing:

```swift
import SnapshotTesting

class EntityRowSnapshotTests: XCTestCase {
    func testEntityRowAppearance() {
        let entity = Entity(id: UUID(), name: "Test Entity")
        let view = EntityRow(entity: entity)
        
        assertSnapshot(matching: view, as: .image)
    }
}
```
