# Testing Strategy

## Unit Tests for Aggregate Models

Test business logic in aggregate models using mock services:

```swift
class MockEntityService: EntityService {
    var entities: [Entity] = []
    
    func fetchAll() async throws -> [Entity] {
        return entities
    }
}

class ModelTests: XCTestCase {
    @MainActor
    func testCreateEntity() async throws {
        let service = MockEntityService()
        let model = Model(service: service)
        
        let entity = Entity(name: "Test")
        try await model.create(entity)
        
        XCTAssertEqual(model.entities.count, 1)
        XCTAssertEqual(model.entities.first?.name, "Test")
    }
}
```

## Integration Tests

Test service layer against real APIs or test databases:

```swift
func testNetworkServiceFetchAll() async throws {
    let service = NetworkEntityService(baseURL: testServerURL)
    let entities = try await service.fetchAll()
    XCTAssertFalse(entities.isEmpty)
}
```

## UI Tests

Use XCTest UI testing for end-to-end validation:

```swift
func testCreateFlow() throws {
    let app = XCUIApplication()
    app.launch()
    
    app.buttons["Add"].tap()
    app.textFields["Name"].tap()
    app.textFields["Name"].typeText("Test Entity")
    app.buttons["Save"].tap()
    
    XCTAssertTrue(app.staticTexts["Test Entity"].exists)
}
```


