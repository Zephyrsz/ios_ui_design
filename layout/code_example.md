4:28 - Grid with explicit rows

struct Leaderboard: View {
    var body: some View {
        Grid {
            GridRow {
                Text("Cat")
                ProgressView(value: 0.5)
                Text("25")
            }
            GridRow {
                Text("Goldfish")
                ProgressView(value: 0.2)
                Text("9")
            }
            GridRow {
                Text("Dog")
                ProgressView(value: 0.3)
                Text("16")
            }
        }
    }
}
Copy Code
5:16 - Data model

struct Pet: Identifiable, Equatable {
    let type: String
    var votes: Int = 0
    var id: String { type }

    static var exampleData: [Pet] = [
        Pet(type: "Cat", votes: 25),
        Pet(type: "Goldfish", votes: 9),
        Pet(type: "Dog", votes: 16)
    ]
}
Copy Code
5:41 - Final Leaderboard

struct Leaderboard: View {
    var pets: [Pet]
    var totalVotes: Int

    var body: some View {
        Grid(alignment: .leading) {
            ForEach(pets) { pet in
                GridRow {
                    Text(pet.type)
                    ProgressView(
                        value: Double(pet.votes),
                        total: Double(totalVotes))
                    Text("\(pet.votes)")
                        .gridColumnAlignment(.trailing)
                }

                Divider()
            }
        }
        .padding()
    }
}
Copy Code
10:53 - Layout protocol stubs for required methods

struct MyEqualWidthHStack: Layout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        // Return a size.
    }

    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        // Place child views.
    }
}
Copy Code
13:44 - Maximum size helper method

private func maxSize(subviews: Subviews) -> CGSize {
    let subviewSizes = subviews.map { $0.sizeThatFits(.unspecified) }
    let maxSize: CGSize = subviewSizes.reduce(.zero) { currentMax, subviewSize in
        CGSize(
            width: max(currentMax.width, subviewSize.width),
            height: max(currentMax.height, subviewSize.height))
    }

    return maxSize
}
Copy Code
15:40 - Spacing helper method

private func spacing(subviews: Subviews) -> [CGFloat] {
    subviews.indices.map { index in
        guard index < subviews.count - 1 else { return 0 }
        return subviews[index].spacing.distance(
            to: subviews[index + 1].spacing,
            along: .horizontal)
    }
}
Copy Code
16:33 - Size that fits implementation

func sizeThatFits(
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) -> CGSize {
    // Return a size.
    guard !subviews.isEmpty else { return .zero }

    let maxSize = maxSize(subviews: subviews)
    let spacing = spacing(subviews: subviews)
    let totalSpacing = spacing.reduce(0) { $0 + $1 }

    return CGSize(
        width: maxSize.width * CGFloat(subviews.count) + totalSpacing,
        height: maxSize.height)
}
Copy Code
16:51 - Place subviews implementation

func placeSubviews(
    in bounds: CGRect,
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) {
    // Place child views.
    guard !subviews.isEmpty else { return }
  
    let maxSize = maxSize(subviews: subviews)
    let spacing = spacing(subviews: subviews)

    let placementProposal = ProposedViewSize(width: maxSize.width, height: maxSize.height)
    var x = bounds.minX + maxSize.width / 2
  
    for index in subviews.indices {
        subviews[index].place(
            at: CGPoint(x: x, y: bounds.midY),
            anchor: .center,
            proposal: placementProposal)
        x += maxSize.width + spacing[index]
    }
}
Copy Code
18:07 - Custom layout instantiation

MyEqualWidthHStack {
    ForEach($pets) { $pet in
        Button {
            pet.votes += 1
        } label: {
            Text(pet.type)
                .frame(maxWidth: .infinity)
        }
        .buttonStyle(.bordered)
    }
}
Copy Code
20:12 - Buttons helper view

struct Buttons: View {
    @Binding var pets: [Pet]

    var body: some View {
        ForEach($pets) { $pet in
            Button {
                pet.votes += 1
            } label: {
                Text(pet.type)
                    .frame(maxWidth: .infinity)
            }
            .buttonStyle(.bordered)
        }
    }
}
Copy Code
21:08 - Final voting buttons view

struct StackedButtons: View {
    @Binding var pets: [Pet]

    var body: some View {
        ViewThatFits {
            MyEqualWidthHStack {
                Buttons(pets: $pets)
            }
            MyEqualWidthVStack {
                Buttons(pets: $pets)
            }
        }
    }
}
Copy Code
22:30 - Radial size that fits

func sizeThatFits(
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
)  -> CGSize {
    // Take whatever space is offered.
    return proposal.replacingUnspecifiedDimensions()
}
Copy Code
22:52 - Radial place subviews without offsets

func placeSubviews(
    in bounds: CGRect,
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) {
    let radius = min(bounds.size.width, bounds.size.height) / 3.0
    let angle = Angle.degrees(360.0 / Double(subviews.count)).radians
    let offset = 0 // This depends on rank...

    for (index, subview) in subviews.enumerated() {
        var point = CGPoint(x: 0, y: -radius)
            .applying(CGAffineTransform(
                rotationAngle: angle * Double(index) + offset))

        point.x += bounds.midX
        point.y += bounds.midY

        subview.place(at: point, anchor: .center, proposal: .unspecified)
    }
}
Copy Code
23:42 - Rank value

private struct Rank: LayoutValueKey {
    static let defaultValue: Int = 1
}

extension View {
    func rank(_ value: Int) -> some View {
        layoutValue(key: Rank.self, value: value)
    }
}
Copy Code
24:21 - Radial place subviews with offsets

func placeSubviews(
    in bounds: CGRect,
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) {
    let radius = min(bounds.size.width, bounds.size.height) / 3.0
    let angle = Angle.degrees(360.0 / Double(subviews.count)).radians

    let ranks = subviews.map { subview in
        subview[Rank.self]
    }
    let offset = getOffset(ranks)

    for (index, subview) in subviews.enumerated() {
        var point = CGPoint(x: 0, y: -radius)
            .applying(CGAffineTransform(
                rotationAngle: angle * Double(index) + offset))
        point.x += bounds.midX
        point.y += bounds.midY
        subview.place(at: point, anchor: .center, proposal: .unspecified)
    }
}
Copy Code
25:18 - Final profile view

struct Profile: View {
    var pets: [Pet]
    var isThreeWayTie: Bool

    var body: some View {
        let layout = isThreeWayTie ? AnyLayout(HStackLayout()) : AnyLayout(MyRadialLayout())

        Podium() // Creates the background that shows ranks.
            .overlay(alignment: .top) {
                layout {
                    ForEach(pets) { pet in
                        Avatar(pet: pet)
                            .rank(rank(pet))
                    }
                }
                .animation(.default, value: pets)
            }
    }
}