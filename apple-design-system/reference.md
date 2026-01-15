# Apple Design System Reference

Complete reference for iOS-native design patterns, components, and guidelines.

## Table of Contents

1. [Visual Design System](#visual-design-system)
2. [Liquid Glass Detailed Guide](#liquid-glass-detailed-guide)
3. [Component Patterns](#component-patterns)
4. [Accessibility Guidelines](#accessibility-guidelines)
5. [Design Tokens](#design-tokens)

---

## Visual Design System

### Typography System

**Dynamic Type Support**

Dynamic Type is CRITICAL for accessibility. Always use semantic font styles:

**Available Semantic Styles:**

- `.largeTitle` - 34pt (default) - Page titles
- `.title` - 28pt - Section headers
- `.title2` - 22pt - Subsection headers
- `.title3` - 20pt - Group headers
- `.headline` - 17pt semibold - Emphasized content
- `.body` - 17pt regular - Default reading text
- `.callout` - 16pt - Secondary content
- `.subheadline` - 15pt - De-emphasized content
- `.footnote` - 13pt - Tertiary information
- `.caption` - 12pt - Minimum readable size
- `.caption2` - 11pt - Labels and annotations

**Line Spacing:**

- SwiftUI handles automatically with Dynamic Type
- Custom spacing: `.lineSpacing(4)` for dense lists
- Comfortable reading: `.lineSpacing(8)`
- Use `.minimumScaleFactor(0.8)` sparingly for single-line text

**SF Pro Font Family:**

- **SF Pro Text** - Body text (13pt and below)
- **SF Pro Display** - Headlines (14pt+)
- **SF Pro Rounded** - Friendly interfaces (use sparingly)
- System font applies automatically

### Color System

**Semantic Color Hierarchy:**

```swift
// Primary - Main text, icons (highest contrast)
Color.primary

// Secondary - Supporting text, secondary icons (medium contrast)
Color.secondary

// Tertiary - Disabled text, placeholders (lowest contrast)
Color.tertiary
```

**System Colors:**

```swift
.blue      // Default tint, primary actions
.green     // Success, positive indicators
.orange    // Warnings, caution
.red       // Errors, destructive actions
.gray      // Neutral, inactive states
.purple    // Creative, alternative
.pink      // Warm, inviting
.teal      // Fresh, modern
.indigo    // Deep, sophisticated
.mint      // Light, refreshing
.cyan      // Technical, precise
.brown     // Natural, earthy
```

**Custom Colors with Assets:**

1. Create color set in `Assets.xcassets`
2. Add "Any Appearance" and "Dark" variants
3. Reference: `Color("AppTextPrimary")`

### Spacing System

**4pt Grid:**

```swift
enum Spacing {
    static let xxs: CGFloat = 4   // Tight: icon-to-text
    static let xs: CGFloat = 8     // Compact: within components
    static let sm: CGFloat = 12    // Comfortable: related items
    static let md: CGFloat = 16    // Standard: general padding
    static let lg: CGFloat = 24    // Generous: section separation
    static let xl: CGFloat = 32    // Spacious: major sections
    static let xxl: CGFloat = 48   // Dramatic: hero spacing
}
```

**Standard Patterns:**

- Card/panel padding: 16pt
- List row padding: 12-16pt horizontal, 8-12pt vertical
- Screen edge margins: 16-20pt
- Button internal padding: 12pt vertical, 16pt horizontal
- Section spacing: 24-32pt

**Safe Areas:**

```swift
// Extend background into safe areas
.ignoresSafeArea(edges: .top)

// Respect safe areas for content (default)
.padding() // Auto-insets from safe area
```

### Materials & Backgrounds

**Material Thickness Levels:**

```swift
.ultraThinMaterial   // Delicate overlays, maintains image visibility
.thinMaterial        // Subtle but present, floating toolbars
.regularMaterial     // Standard depth, panels and cards
.thickMaterial       // Strong separation, modal sheets
.ultraThickMaterial  // Maximum separation
```

**Material Vibrancy:**
Materials automatically adjust:

- Light mode: subtle blur with light tint
- Dark mode: deeper blur with dark tint
- Content behind remains partially visible
- Text on materials gains automatic vibrancy

**Usage Guidelines:**

- **Screen backgrounds:** Solid colors
- **Cards/panels:** `.regularMaterial`
- **Toolbars:** `.thinMaterial`
- **Modal sheets:** `.regularMaterial` or `.thickMaterial`
- **Image overlays:** `.ultraThinMaterial`

---

## Liquid Glass Detailed Guide

### Overview

Liquid Glass (iOS 26+) combines:

- Glass-like translucency
- Fluid morphing effects
- Interactive response to touch/pointer
- Automatic blending of nearby glass elements

### Basic Implementation

**Simple Glass Effect:**

```swift
Text("Hello")
    .padding()
    .glassEffect() // Default: Capsule shape
```

**Custom Shapes:**

```swift
.glassEffect(in: .rect(cornerRadius: 16))
.glassEffect(in: .capsule)
.glassEffect(in: .circle)
```

**Glass Variants:**

```swift
// Regular glass
.glassEffect(.regular)

// Tinted glass (suggests prominence)
.glassEffect(.regular.tint(.orange))

// Interactive glass (responds to touch/pointer)
.glassEffect(.regular.interactive())

// Combined
.glassEffect(.regular.tint(.blue).interactive())
```

### GlassEffectContainer

Use for multiple glass elements to enable blending and morphing:

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .glassEffect()

        Image(systemName: "eraser.fill")
            .glassEffect()
    }
}
```

**Spacing parameter:**

- Smaller values: Views must be closer to merge
- Larger values: Effects merge at greater distances
- Typical range: 20-60pt

### Uniting Glass Effects

Combine multiple views into single glass effect:

```swift
@Namespace private var namespace

GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        ForEach(items.indices, id: \.self) { index in
            ItemView(items[index])
                .glassEffect()
                .glassEffectUnion(id: index < 2 ? "group1" : "group2",
                                  namespace: namespace)
        }
    }
}
```

### Morphing Transitions

Create fluid morphing effects during view hierarchy changes:

```swift
@State private var isExpanded: Bool = false
@Namespace private var namespace

var body: some View {
    GlassEffectContainer(spacing: 40.0) {
        HStack(spacing: 40.0) {
            Image(systemName: "scribble.variable")
                .glassEffect()
                .glassEffectID("pencil", in: namespace)

            if isExpanded {
                Image(systemName: "eraser.fill")
                    .glassEffect()
                    .glassEffectID("eraser", in: namespace)
            }
        }
    }

    Button("Toggle") {
        withAnimation {
            isExpanded.toggle()
        }
    }
}
```

### Glass Button Styles

```swift
// Standard glass button
Button("Click Me") { }
    .buttonStyle(.glass)

// Prominent glass button
Button("Important Action") { }
    .buttonStyle(.glassProminent)
```

### Performance Considerations

- Liquid Glass uses GPU effects
- Limit to 5-10 prominent elements
- Avoid on long scrolling lists (60+ items)
- Test on older devices (iPhone 12+)
- For large lists, use `.regularMaterial` instead

### Best Practices

1. **Container Usage:** Always use `GlassEffectContainer` for multiple glass views
2. **Effect Order:** Apply `.glassEffect()` after appearance modifiers
3. **Spacing:** Choose spacing values carefully to control merging
4. **Animation:** Use `withAnimation` for smooth morphing
5. **Interactivity:** Add `.interactive()` for user-interactive elements
6. **Consistency:** Maintain shapes/styles across app

---

## Component Patterns

### Navigation Patterns

**TabView (Primary Navigation):**

**Guidelines:**

- Maximum 5 tabs (4 is ideal)
- Use SF Symbols for tab icons
- Use `Label` for text + icon
- Set app-wide tint color

**NavigationStack (Hierarchical):**

```swift
NavigationStack {
    List(meals) { meal in
        NavigationLink(value: meal) {
            MealRowView(meal: meal)
        }
    }
    .navigationDestination(for: Meal.self) { meal in
        MealDetailView(meal: meal)
    }
    .navigationTitle("Meal Plans")
    .navigationBarTitleDisplayMode(.large)
}
```

**Modal Presentations:**

```swift
// Sheet - Standard modals
.sheet(isPresented: $showingAddMeal) {
    AddMealView()
        .presentationDetents([.medium, .large])
        .presentationDragIndicator(.visible)
}

// Full Screen Cover - Immersive
.fullScreenCover(isPresented: $showingOnboarding) {
    OnboardingView()
}
```

### List Patterns

**Standard List:**

```swift
List {
    Section("Breakfast") {
        ForEach(breakfastMeals) { meal in
            MealRowView(meal: meal)
        }
    }

    Section("Lunch") {
        ForEach(lunchMeals) { meal in
            MealRowView(meal: meal)
        }
    }
}
.listStyle(.insetGrouped)
```

**List Styles:**

- `.plain` - No background
- `.insetGrouped` - Rounded, inset sections (iOS standard)
- `.grouped` - Full-width sections
- `.sidebar` - For sidebar navigation

**Custom Card Layouts:**

```swift
ScrollView {
    LazyVStack(spacing: 16) {
        ForEach(meals) { meal in
            MealCardView(meal: meal)
                .background(.regularMaterial)
                .clipShape(RoundedRectangle(cornerRadius: 12))
        }
    }
    .padding()
}
```

### Button Patterns

**Button Style Hierarchy:**

```swift
// Primary CTA - Most prominent
Button("Save Changes") { }
    .buttonStyle(.borderedProminent)

// Secondary - Less emphasis
Button("Cancel") { }
    .buttonStyle(.bordered)

// Tertiary - Minimal
Button("Skip") { }
    .buttonStyle(.plain)

// Glass (iOS 26+)
Button("Continue") { }
    .buttonStyle(.glass)

Button("Get Started") { }
    .buttonStyle(.glassProminent)
```

**Button Roles:**

```swift
Button("Delete", role: .destructive) { }
Button("Cancel", role: .cancel) { }
```

**Minimum Touch Target:**

```swift
Button { } label: {
    Image(systemName: "plus")
}
.frame(minWidth: 44, minHeight: 44)
```

### Form Patterns

**Standard Form:**

```swift
Form {
    Section("Meal Details") {
        TextField("Meal name", text: $mealName)
        DatePicker("Time", selection: $mealTime,
                   displayedComponents: .hourAndMinute)
    }

    Section("Nutrition") {
        HStack {
            Text("Calories")
            Spacer()
            TextField("Amount", value: $calories, format: .number)
                .keyboardType(.numberPad)
                .multilineTextAlignment(.trailing)
        }

        Toggle("Track macros", isOn: $trackMacros)
    }

    Section {
        Button("Save") {
            saveMeal()
        }
    }
}
```

**Input Components:**

- `TextField` - Single-line text
- `TextEditor` - Multi-line text
- `SecureField` - Password input
- `Picker` - Selection from options
- `DatePicker` - Date/time selection
- `Toggle` - Binary on/off
- `Stepper` - Incremental numeric
- `Slider` - Continuous value

### Card Design

**Standard Card:**

```swift
VStack(alignment: .leading, spacing: 12) {
    // Header
    HStack {
        Image(systemName: "fork.knife")
            .font(.title2)
        Text("Breakfast")
            .font(.headline)
        Spacer()
        Text("7:00 AM")
            .font(.subheadline)
            .foregroundStyle(.secondary)
    }

    // Content
    Text("High-protein oatmeal with berries")
        .font(.body)

    // Footer
    HStack(spacing: 16) {
        Label("432 cal", systemImage: "flame.fill")
        Label("28g protein", systemImage: "bolt.fill")
    }
    .font(.caption)
    .foregroundStyle(.secondary)
}
.padding(16)
.background(.regularMaterial)
.clipShape(RoundedRectangle(cornerRadius: 12))
```

**Card Guidelines:**

- Corner radius: 12-16pt
- Internal padding: 16pt
- Background: `.regularMaterial` or `.glassEffect()`
- Shadow alternative: Materials provide depth without shadows

---

## Accessibility Guidelines

### Dynamic Type (CRITICAL)

**Always support Dynamic Type:**

```swift
// Correct
Text("Title").font(.title)
Text("Body").font(.body)

// Wrong - doesn't scale
Text("Title").font(.system(size: 28))
```

**Test with:**

- Settings → Accessibility → Display & Text Size → Larger Text
- Xcode Environment Overrides
- Test at minimum and maximum sizes

**Custom Dynamic Type:**

```swift
Text("Special")
    .font(.system(.body, design: .rounded))
    .dynamicTypeSize(...DynamicTypeSize.xxxLarge) // Cap if needed
```

### VoiceOver Support

**Accessibility Labels:**

```swift
Button {
    addMeal()
} label: {
    Image(systemName: "plus.circle.fill")
}
.accessibilityLabel("Add meal")
.accessibilityHint("Opens meal creation form")
```

**Combine Elements:**

```swift
HStack {
    Image(systemName: "fork.knife")
    VStack(alignment: .leading) {
        Text("Breakfast")
        Text("432 calories")
    }
}
.accessibilityElement(children: .combine)
.accessibilityLabel("Breakfast, 432 calories")
```

**Hide Decorative Elements:**

```swift
Image("decorative-pattern")
    .accessibilityHidden(true)
```

### Touch Targets

**Minimum Size: 44x44pt**

```swift
// Ensure minimum size
Button { } label: {
    Image(systemName: "heart")
}
.frame(minWidth: 44, minHeight: 44)

// Check with inspector
.accessibilityShowsLargeSizes()
```

### Color Contrast

**WCAG AA Requirements:**

- Normal text: 4.5:1 contrast ratio
- Large text (18pt+): 3:1 contrast ratio
- Graphical elements: 3:1 contrast ratio

**Semantic colors meet requirements automatically**

**Test custom colors:**

- Use online contrast checker
- Test in light and dark mode
- Consider colorblind users

### Reduce Motion

**Support users who need reduced motion:**

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

var animation: Animation? {
    reduceMotion ? nil : .spring(response: 0.3, dampingFraction: 0.7)
}

.animation(animation, value: someState)
```

**Alternative approach:**

```swift
if reduceMotion {
    // Instant transition
    DetailView()
} else {
    // Animated transition
    DetailView()
        .transition(.scale)
}
```

---

## Design Tokens

### Spacing Tokens

```swift
enum Spacing {
    static let xxs: CGFloat = 4
    static let xs: CGFloat = 8
    static let sm: CGFloat = 12
    static let md: CGFloat = 16
    static let lg: CGFloat = 24
    static let xl: CGFloat = 32
    static let xxl: CGFloat = 48
}
```

### Corner Radius Tokens

```swift
enum CornerRadius {
    static let xs: CGFloat = 4
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let xl: CGFloat = 24
    static let circle: CGFloat = .infinity
}
```

### Shadow Tokens

```swift
enum Shadow {
    static let sm = (radius: 2.0, x: 0.0, y: 1.0)
    static let md = (radius: 4.0, x: 0.0, y: 2.0)
    static let lg = (radius: 8.0, x: 0.0, y: 4.0)
}

// Usage
.shadow(radius: Shadow.md.radius, x: Shadow.md.x, y: Shadow.md.y)
```

**Note:** Consider using materials instead of shadows for iOS-native feel

### Animation Tokens

```swift
enum AnimationTiming {
    static let quick: Double = 0.2
    static let standard: Double = 0.3
    static let slow: Double = 0.5

    static let springResponse: Double = 0.3
    static let springDamping: Double = 0.7
}

// Usage
.animation(.spring(response: AnimationTiming.springResponse,
                   dampingFraction: AnimationTiming.springDamping),
           value: state)
```

---

## Additional Resources

- [Apple HIG](https://developer.apple.com/design/human-interface-guidelines/)
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui)
- [Accessibility Documentation](https://developer.apple.com/accessibility/)
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- Liquid Glass: `/Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md`
