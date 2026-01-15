---
name: apple-design-system
description: Design iOS interfaces following Apple Human Interface Guidelines and iOS 26+ patterns including Liquid Glass, SF Symbols, SwiftUI components, materials, accessibility. Use when designing iOS screens, choosing UI patterns, creating SwiftUI views, adapting web designs to iOS, or when user mentions iOS design, Apple HIG, Liquid Glass interface design.
---

# Apple Design System

Provides iOS-native design guidance for building interfaces that feel natural on iOS while maintaining creative distinctiveness. Works alongside the `frontend-design` skill to adapt bold web aesthetics to iOS conventions.

## Core Principles

**Apple Human Interface Guidelines:**

- **Clarity**: Legible text, precise icons, subtle adornments
- **Deference**: Content-first, UI doesn't compete with content
- **Depth**: Visual layers and motion convey hierarchy

## Quick Design Decisions

### Typography

Use semantic font styles:

```swift
.font(.title)     // 28pt section headers
.font(.headline)  // 17pt emphasized
.font(.body)      // 17pt default
```

For complete typography system, see [reference.md](reference.md#typography-system)

### Spacing

Follow 4pt grid system: 4, 8, 12, 16, 24, 32, 48
For complete spacing system, see [reference.md](reference.md#spacing-system)

### Navigation

```swift
TabView { }                           // Primary (4-tab: Dashboard, Meals, Training, Profile)
NavigationStack { }                   // Hierarchical drill-down
.sheet(isPresented:) { }              // Modal forms
```

### Buttons

```swift
.buttonStyle(.borderedProminent)      // Primary CTA
.buttonStyle(.bordered)               // Secondary action
.buttonStyle(.glass)                  // Modern iOS 26+
```

## Liquid Glass Design (iOS 26+)

Liquid Glass combines translucency with fluid, interactive effects. Use for:

- Interactive cards (meal cards, training sessions)
- Prominent CTAs
- Dashboard widgets
- Hub section headers

**Basic usage:**

```swift
// Simple glass effect
Text("Upcoming Race")
    .padding()
    .glassEffect()

// Custom shape with tint
VStack {
    // Content
}
.padding()
.glassEffect(.regular.tint(.blue).interactive())

// Button styles
Button("Save") { }
    .buttonStyle(.glass)

Button("Start Training") { }
    .buttonStyle(.glassProminent)
```

**Multiple glass elements:**

```swift
GlassEffectContainer(spacing: 20) {
    HStack(spacing: 20) {
        MealCardView().glassEffect()
        TrainingCardView().glassEffect()
    }
}
```

## SF Symbols

Use SF Symbols for all icons (5,000+ available, auto-match font weights):

```swift
Image(systemName: "house.fill")       // Dashboard
Image(systemName: "person.fill")      // Profile
```

**Common symbols:**

- Nutrition: `fork.knife`, `carrot.fill`, `drop.fill`
- Actions: `plus.circle.fill`, `checkmark.circle.fill`
- Stats: `chart.bar.fill`, `heart.fill`, `flame.fill`

## Accessibility Requirements

**Must-haves:**

- ✅ Dynamic Type support (use `.font(.headline)` not `.font(.system(size: 17))`)
- ✅ Minimum 44x44pt touch targets
- ✅ VoiceOver labels: `.accessibilityLabel("Add meal")`
- ✅ Color contrast (WCAG AA: 4.5:1 for text)
- ✅ Reduce Motion support when needed

## Adapting Frontend-Design to iOS

When `frontend-design` creates bold web concepts, adapt them:

| Web Aesthetic | iOS Adaptation |
|--------------|----------------|
| Bold gradients | Tinted materials/glass |
| Custom web fonts | SF Pro Display/Text/Rounded |
| Complex animations | Subtle spring physics |
| Sharp shapes | Rounded rectangles (12-16pt radius) |
| Custom icons | SF Symbols |

**Example:**

```swift
// Instead of bold gradient backgrounds...
// Use tinted glass with accent
VStack {
    // Content
}
.padding(16)
.background(.regularMaterial)
.glassEffect(.regular.tint(.blue))
.clipShape(RoundedRectangle(cornerRadius: 12))
```

## Animations

Use spring animations for natural, iOS-native motion:

```swift
@State private var isExpanded = false

var body: some View {
    VStack {
        if isExpanded {
            DetailView()
                .transition(.scale.combined(with: .opacity))
        }
    }
    .animation(.spring(response: 0.3, dampingFraction: 0.7), value: isExpanded)
}
```

**Common patterns:**

- Default spring: `.animation(.default, value: state)`
- Smooth spring: `.spring(response: 0.3, dampingFraction: 0.7)`
- Snappy spring: `.spring(response: 0.2, dampingFraction: 0.8)`

## Decision Flowchart

**Need to choose:**

**Navigation?**

- Primary app navigation → `TabView` (max 4-5 tabs)
- Detail/drill-down → `NavigationStack`
- Modal action → `.sheet`
- Full immersion → `.fullScreenCover`

**Background?**

- Screen background → Solid color
- Panel/card → `.regularMaterial` or `.glassEffect()`
- Overlay on image → `.ultraThinMaterial`

**Button style?**

- Primary CTA → `.borderedProminent`
- Secondary → `.bordered`
- Tertiary → `.plain`
- Modern iOS 26+ → `.glass` or `.glassProminent`

**List layout?**

- Simple rows → `List`
- Custom cards → `ScrollView + LazyVStack`
- Horizontal → `ScrollView(.horizontal) + LazyHStack`

## Learn More

For comprehensive details, see:

- **[reference.md](reference.md)** - Complete design system reference, all components, detailed guidelines
- **[examples.md](examples.md)** - Full code examples for common patterns and layouts

## External Resources

- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [SF Symbols App](https://developer.apple.com/sf-symbols/) - Browse all 5,000+ symbols
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui)
- Liquid Glass docs: `/Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md`
