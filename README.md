import SwiftUI

struct Habit: Identifiable, Codable {
    var id = UUID()
    var name: String
    var unit: String
    var goal: Int
    var progress: Int
}

class HabitStore: ObservableObject {
    @Published var habits: [Habit] = []

    init() {
        load()
    }

    func load() {
        if let data = UserDefaults.standard.data(forKey: "habits"),
           let decoded = try? JSONDecoder().decode([Habit].self, from: data) {
            habits = decoded
        } else {
            habits = [
                Habit(name: "Eau", unit: "verres", goal: 8, progress: 0),
                Habit(name: "Sport", unit: "minutes", goal: 30, progress: 0),
                Habit(name: "Sommeil", unit: "heures", goal: 8, progress: 0)
            ]
        }
    }

    func save() {
        if let encoded = try? JSONEncoder().encode(habits) {
            UserDefaults.standard.set(encoded, forKey: "habits")
        }
    }

    func reset() {
        for i in habits.indices {
            habits[i].progress = 0
        }
        save()
    }

    func increment(_ habit: Habit) {
        if let index = habits.firstIndex(where: { $0.id == habit.id }) {
            if habits[index].progress < habits[index].goal {
                habits[index].progress += 1
                save()
            }
        }
    }
}

struct ContentView: View {
    @StateObject var store = HabitStore()

    var body: some View {
        NavigationView {
            List {
                ForEach(store.habits) { habit in
                    VStack(alignment: .leading) {
                        Text(habit.name)
                            .font(.headline)
                        HStack {
                            Text("\(habit.progress)/\(habit.goal) \(habit.unit)")
                            Spacer()
                            Button("+") {
                                store.increment(habit)
                            }
                            .padding(6)
                            .background(Color.blue)
                            .foregroundColor(.white)
                            .cornerRadius(6)
                        }
                        ProgressView(value: Float(habit.progress), total: Float(habit.goal))
                    }
                    .padding(.vertical, 6)
                }
            }
            .navigationTitle("Suivi d’habitudes")
            .toolbar {
                Button("Réinitialiser") {
                    store.reset()
                }
            }
        }
    }
}

@main
struct HabitApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
