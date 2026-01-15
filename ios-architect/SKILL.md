---
name: ios-architect
description: Comprehensive iOS application architecture guidance focusing on SwiftUI and the Model-View (MV) pattern. Use when Claude needs to design iOS application architecture, create models, design backend functionality, plan data flow, write code, or provide architectural recommendations for iOS apps. Covers aggregate root models, service layers, SwiftUI best practices, and Apple's recommended patterns.
---

# iOS Architect

Comprehensive guidance for architecting iOS applications using SwiftUI and the Model-View (MV) pattern, following Apple's recommendations and modern best practices.

## Core Architecture Pattern

Use the **Model-View (MV)** pattern for SwiftUI applications:

- **View**: SwiftUI views that are also their own view models
- **Model**: Aggregate root models that provide entities to views
- **Service Layer**: Network/data services that aggregate models call

### State Management Hierarchy

SwiftUI provides a hierarchy of property wrappers for different state needs:

- **@State**: Local view state, owned by the view
- **@Binding**: Two-way connection to state owned elsewhere
- **@StateObject**: Creates and owns an ObservableObject
- **@ObservedObject**: References an ObservableObject owned elsewhere
- **@EnvironmentObject**: Dependency injection through the view hierarchy
- **@Environment**: Access to system-provided values AND custom @Observable models (iOS 17+)

### Key Principle: View IS the View Model

SwiftUI views already have MVVM built-in through property wrappers. Do NOT create separate view model classes for each view. Instead:

- Views use `@State` for local state
- Views use `@Environment` or `@State` to access aggregate models
- Views handle UI validation and presentation logic directly
- Aggregate models handle business logic and data coordination

For detailed instructions about implementing design patters reference [references/common-patterns.md](references/common-patterns.md)

## Best Practices

1. **Keep Views Dumb**: Views should focus on layout and user interaction
2. **@MainActor for Models**: All aggregate models should be annotated with `@MainActor`
3. **Async/Await**: Use structured concurrency for network operations
4. **Protocol Services**: Always define service protocols for testability
5. **Single Responsibility**: Each aggregate model handles one bounded context
6. **Avoid Over-Architecture**: Don't create view models if you don't need them
7. **Direct Model Access**: Views should directly consume and display model objects
8. **UI Validation Only**: Validation in views is for UI constraints, not business rules

### Naming Convention

Name models based on bounded context, not screen count:

- Small apps: Single `Model` class
- Medium apps: `ProductCatalog`, `OrderManagement`, `UserProfile`
- Large apps: Multiple models based on domain-driven design bounded contexts

## Testing Strategy

Test business logic in aggregate models using mock services. Test service layer against real APIs or test databases. Use XCTest UI testing for end-to-end validation.

For detailed testing examples reference [references/testing-patterns.md](references/testing-patterns.md)

## Additional Resources

For detailed implementation examples and in-depth discussion of the MV pattern, see:

- `references/mv-pattern-guide.md` - Complete guide with code examples
- Apple's WWDC: Data Essentials in SwiftUI
- Apple's sample apps: Fruta, Food Truck, Meme Creator
