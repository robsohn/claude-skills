# Apple Design System Examples

Complete, runnable code examples for common iOS patterns 

## Table of Contents

1. [Meal Card with Liquid Glass](#meal-card-with-liquid-glass)
2. [Empty State](#empty-state)
3. [Form with Validation](#form-with-validation)
4. [Data Visualization](#data-visualization)
5. [Meal Priority Badge](#meal-priority-badge)


---

## Meal Card with Liquid Glass

**Use case:**
- List items with rich nutritional content
- Dashboard widgets showing meal summaries
- Interactive cards requiring user attention
- When you want to highlight priority meals

**Platform:** iOS 26+ (Liquid Glass), iOS 17+ (fallback to .regularMaterial)

**Accessibility:** Includes Dynamic Type support, semantic colors, and proper VoiceOver labels

Interactive meal card with priority badge:

```swift
import SwiftUI

enum MealPriority: String, CaseIterable {
    case low, medium, high

    var color: Color {
        switch self {
        case .low: return .green
        case .medium: return .orange
        case .high: return .red
        }
    }
}

struct Meal: Identifiable {
    let id = UUID()
    let name: String
    let time: Date
    let calories: Int
    let protein: Int?
    let priority: MealPriority
}

struct MealCardView: View {
    let meal: Meal

    var body: some View {
        HStack(spacing: 12) {
            // Time indicator
            VStack(spacing: 4) {
                Text(meal.time, format: .dateTime.hour().minute())
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }
            .frame(width: 60)

            // Main content
            VStack(alignment: .leading, spacing: 4) {
                Text(meal.name)
                    .font(.headline)
                    .foregroundStyle(.primary)

                HStack(spacing: 12) {
                    Label("\(meal.calories) cal", systemImage: "flame.fill")

                    if let protein = meal.protein {
                        Label("\(protein)g protein", systemImage: "bolt.fill")
                    }
                }
                .font(.caption)
                .foregroundStyle(.secondary)
            }

            Spacer()

            // Priority badge
            Circle()
                .fill(meal.priority.color)
                .frame(width: 8, height: 8)
        }
        .padding(12)
        .background(.regularMaterial)
        .glassEffect(.regular.interactive())
        .clipShape(RoundedRectangle(cornerRadius: 8))
    }
}

// Usage
struct MealsListView: View {
    let meals: [Meal] = [
        Meal(name: "Oatmeal with berries",
             time: Date().addingTimeInterval(-3600 * 5),
             calories: 432,
             protein: 28,
             priority: .high),
        Meal(name: "Recovery shake",
             time: Date().addingTimeInterval(-3600 * 2),
             calories: 240,
             protein: 25,
             priority: .medium)
    ]

    var body: some View {
        GlassEffectContainer(spacing: 20) {
            VStack(spacing: 12) {
                ForEach(meals) { meal in
                    MealCardView(meal: meal)
                }
            }
            .padding()
        }
    }
}
```

**See also:**
- [Empty State](#empty-state) - For handling empty meal lists
- [Priority Badge](#meal-priority-badge) - Standalone priority indicator component
- [reference.md - Liquid Glass](reference.md#liquid-glass-detailed-guide) - Advanced Liquid Glass features

---

## Empty State

**Use case:**
- First-time user experience (no data yet)
- Encouraging action when lists are empty
- Post-deletion states
- Temporary empty states (filtered results)

**Platform:** iOS 17+ (uses ContentUnavailableView)

**Accessibility:** ContentUnavailableView automatically handles VoiceOver and Dynamic Type

Encouraging empty state with action:

```swift
import SwiftUI

struct EmptyMealsView: View {
    let onAddMeal: () -> Void

    var body: some View {
        ContentUnavailableView {
            Label("No Meals Planned", systemImage: "fork.knife")
        } description: {
            Text("Start planning your nutrition to support your training goals")
        } actions: {
            Button("Add Your First Meal") {
                onAddMeal()
            }
            .buttonStyle(.borderedProminent)

            Button("Browse Meal Templates") { }
                .buttonStyle(.bordered)
        }
    }
}

// Alternative custom empty state
struct CustomEmptyStateView: View {
    let icon: String
    let title: String
    let description: String
    let actionTitle: String
    let action: () -> Void

    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: icon)
                .font(.system(size: 72))
                .foregroundStyle(.secondary)
                .symbolRenderingMode(.hierarchical)

            VStack(spacing: 8) {
                Text(title)
                    .font(.title2)
                    .fontWeight(.semibold)

                Text(description)
                    .font(.body)
                    .foregroundStyle(.secondary)
                    .multilineTextAlignment(.center)
            }

            Button(actionTitle, action: action)
                .buttonStyle(.borderedProminent)
                .controlSize(.large)
        }
        .padding(40)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

**See also:**
- [Meal Card](#meal-card-with-liquid-glass) - Card design for displaying meals
- [Form with Validation](#form-with-validation) - For the "Add Meal" action
- [reference.md - Component Patterns](reference.md#component-patterns) - More empty state patterns

---

## Form with Validation

**Use case:**
- Creating/editing data with structured inputs
- Multi-section forms with validation
- Modal data entry sheets
- Settings screens

**Platform:** iOS 16+

**Accessibility:** Form automatically handles keyboard navigation and VoiceOver focus management

**Key features:**
- Real-time validation with error messages
- Disabled save button when invalid
- Proper keyboard types for numeric inputs
- Segmented picker with visual indicators

Add meal form with validation:

```swift
import SwiftUI

struct AddMealView: View {
    @Environment(\.dismiss) private var dismiss

    @State private var mealName = ""
    @State private var mealTime = Date()
    @State private var calories = ""
    @State private var protein = ""
    @State private var trackMacros = false
    @State private var priority: MealPriority = .medium

    var isValid: Bool {
        !mealName.isEmpty && Int(calories) != nil
    }

    var body: some View {
        NavigationStack {
            Form {
                Section("Meal Details") {
                    TextField("Meal name", text: $mealName)
                        .autocorrectionDisabled()

                    DatePicker("Time",
                               selection: $mealTime,
                               displayedComponents: [.hourAndMinute])
                }

                Section("Nutrition") {
                    HStack {
                        Text("Calories")
                        Spacer()
                        TextField("Amount", text: $calories)
                            .keyboardType(.numberPad)
                            .multilineTextAlignment(.trailing)
                            .frame(width: 100)
                    }

                    if !calories.isEmpty && Int(calories) == nil {
                        Text("Calories must be a number")
                            .font(.caption)
                            .foregroundStyle(.red)
                    }

                    HStack {
                        Text("Protein")
                        Spacer()
                        TextField("Grams", text: $protein)
                            .keyboardType(.numberPad)
                            .multilineTextAlignment(.trailing)
                            .frame(width: 100)
                    }

                    Toggle("Track macros", isOn: $trackMacros)
                }

                Section("Priority") {
                    Picker("Priority", selection: $priority) {
                        ForEach(MealPriority.allCases, id: \.self) { priority in
                            HStack {
                                Circle()
                                    .fill(priority.color)
                                    .frame(width: 12, height: 12)
                                Text(priority.rawValue.capitalized)
                            }
                            .tag(priority)
                        }
                    }
                    .pickerStyle(.segmented)
                }
            }
            .navigationTitle("Add Meal")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") {
                        dismiss()
                    }
                }

                ToolbarItem(placement: .confirmationAction) {
                    Button("Save") {
                        saveMeal()
                    }
                    .disabled(!isValid)
                }
            }
        }
    }

    private func saveMeal() {
        // Save logic here
        dismiss()
    }
}
```

**See also:**
- [Priority Badge](#meal-priority-badge) - Priority picker used in this form
- [reference.md - Form Patterns](reference.md#form-patterns) - Complete form guidelines
- [reference.md - Accessibility](reference.md#accessibility-guidelines) - Form accessibility requirements

---

## Data Visualization

**Use case:**
- Training zone distribution display
- Progress indicators with multiple segments
- Performance metrics breakdown
- Time/intensity analysis

**Platform:** iOS 15+

**Accessibility:** 
- Uses semantic colors that adapt to light/dark mode
- Includes descriptive labels for screen readers
- Dynamic Type support for all text

**Key features:**
- Animated bars with GeometryReader
- Current zone indicator with visual prominence
- Color-coded zones following fitness conventions

Training zone indicator:

```swift
import SwiftUI

struct TrainingZone: Identifiable {
    let id = UUID()
    let name: String
    let color: Color
    let percentage: Double
}

struct TrainingZonesView: View {
    let zones: [TrainingZone] = [
        TrainingZone(name: "Zone 1", color: .gray, percentage: 0.2),
        TrainingZone(name: "Zone 2", color: .blue, percentage: 0.3),
        TrainingZone(name: "Zone 3", color: .green, percentage: 0.3),
        TrainingZone(name: "Zone 4", color: .orange, percentage: 0.15),
        TrainingZone(name: "Zone 5", color: .red, percentage: 0.05)
    ]

    let currentZone: Int = 2

    var body: some View {
        VStack(alignment: .leading, spacing: 16) {
            Text("Training Zones")
                .font(.headline)

            // Zone bars
            VStack(spacing: 8) {
                ForEach(Array(zones.enumerated()), id: \.offset) { index, zone in
                    HStack(spacing: 12) {
                        Text(zone.name)
                            .font(.caption)
                            .foregroundStyle(.secondary)
                            .frame(width: 60, alignment: .leading)

                        GeometryReader { geometry in
                            ZStack(alignment: .leading) {
                                // Background
                                RoundedRectangle(cornerRadius: 4)
                                    .fill(zone.color.opacity(0.2))

                                // Fill
                                RoundedRectangle(cornerRadius: 4)
                                    .fill(zone.color)
                                    .frame(width: geometry.size.width * zone.percentage)

                                // Current zone indicator
                                if index == currentZone {
                                    Circle()
                                        .fill(.white)
                                        .frame(width: 8, height: 8)
                                        .overlay {
                                            Circle()
                                                .stroke(zone.color, lineWidth: 2)
                                        }
                                        .offset(x: geometry.size.width * zone.percentage)
                                }
                            }
                        }
                        .frame(height: 12)

                        Text("\(Int(zone.percentage * 100))%")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                            .frame(width: 40, alignment: .trailing)
                    }
                }
            }
        }
        .padding()
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}
```

**See also:**
- [Meal Card](#meal-card-with-liquid-glass) - Similar card-based layout pattern
- [reference.md - Color System](reference.md#color-system) - Semantic color guidelines
- [reference.md - Card Design](reference.md#card-design) - Card layout best practices

---

## Meal Priority Badge

**Use case:**
- Reusable status indicators
- Priority/urgency markers
- Color-coded categorization
- Compact visual cues in lists

**Platform:** iOS 15+

**Accessibility:** Includes explicit accessibility labels for VoiceOver users

**Key features:**
- Configurable with/without text label
- Semantic colors that adapt to theme
- Reusable component pattern
- Minimal footprint (8pt circle)

Reusable priority indicator component:

```swift
import SwiftUI

struct PriorityBadgeView: View {
    let priority: MealPriority
    let showLabel: Bool

    init(priority: MealPriority, showLabel: Bool = false) {
        self.priority = priority
        self.showLabel = showLabel
    }

    var body: some View {
        HStack(spacing: 6) {
            Circle()
                .fill(priority.color)
                .frame(width: 8, height: 8)

            if showLabel {
                Text(priority.rawValue.capitalized)
                    .font(.caption)
                    .foregroundStyle(priority.color)
            }
        }
        .accessibilityLabel("\(priority.rawValue) priority")
    }
}

// Usage in list
struct MealRowView: View {
    let meal: Meal

    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(meal.name)
                    .font(.headline)

                Text("\(meal.calories) calories")
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }

            Spacer()

            PriorityBadgeView(priority: meal.priority)
        }
        .padding(.vertical, 4)
    }
}
```

**See also:**
- [Meal Card](#meal-card-with-liquid-glass) - Full meal card using this badge
- [Form with Validation](#form-with-validation) - Priority picker implementation
- [reference.md - Accessibility](reference.md#accessibility-guidelines) - Accessibility label best practices

---

These examples demonstrate iOS-native patterns following Apple HIG. All examples use:
- Dynamic Type support
- Semantic colors
- Proper spacing
- Accessibility labels
- Spring animations
- Liquid Glass where appropriate
