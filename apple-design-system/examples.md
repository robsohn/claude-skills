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

---

## Empty State

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

---

## Form with Validation

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

---

## Data Visualization

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

---

## Meal Priority Badge

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

---

These examples demonstrate iOS-native patterns following Apple HIG. All examples use:
- Dynamic Type support
- Semantic colors
- Proper spacing
- Accessibility labels
- Spring animations
- Liquid Glass where appropriate
