# iOS Layout 工具 - 完整代码示例

## 1. Grid 完整示例

### 基础排行榜
```swift
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

struct Leaderboard: View {
    var pets: [Pet]
    var totalVotes: Int {
        pets.reduce(0) { $0 + $1.votes }
    }

    var body: some View {
        Grid(alignment: .leading) {
            // 表头（可选）
            GridRow {
                Text("Pet").bold()
                Text("Votes %").bold()
                    .gridColumnAlignment(.trailing)
                Text("Count").bold()
                    .gridColumnAlignment(.trailing)
            }
            
            Divider()
            
            // 数据行
            ForEach(pets) { pet in
                GridRow {
                    Text(pet.type)  // 左对齐
                    
                    ProgressView(
                        value: Double(pet.votes),
                        total: Double(totalVotes)
                    )  // 占用剩余空间
                    
                    Text("\(pet.votes)")
                        .gridColumnAlignment(.trailing)  // 右对齐
                }
                
                // 分隔线自动跨所有列
                if pet.id != pets.last?.id {
                    Divider()
                }
            }
        }
        .padding()
    }
}

#Preview {
    Leaderboard(pets: Pet.exampleData)
}
```

### 高级排行榜（带排名指示）
```swift
struct AdvancedLeaderboard: View {
    @State var pets: [Pet] = Pet.exampleData
    var totalVotes: Int {
        pets.reduce(0) { $0 + $1.votes }
    }
    
    // 计算每个 pet 的排名
    func rank(for pet: Pet) -> Int {
        let sorted = pets.sorted { $0.votes > $1.votes }
        return sorted.firstIndex(where: { $0.id == pet.id }) ?? 0
    }

    var body: some View {
        VStack {
            Text("Pet Poll Results")
                .font(.title)
            
            Grid(alignment: .leading, horizontalSpacing: 12, verticalSpacing: 8) {
                ForEach(pets) { pet in
                    GridRow {
                        // 排名徽章
                        HStack {
                            Text("#\(rank(for: pet) + 1)")
                                .font(.headline)
                                .foregroundColor(.white)
                                .frame(width: 30, height: 30)
                                .background(Circle().fill(Color.blue))
                            
                            Text(pet.type)
                        }
                        
                        ProgressView(
                            value: Double(pet.votes),
                            total: Double(totalVotes)
                        )
                        
                        VStack(alignment: .trailing, spacing: 2) {
                            Text("\(pet.votes)")
                                .font(.headline)
                            Text("\(Int(Double(pet.votes) / Double(totalVotes) * 100))%")
                                .font(.caption)
                                .foregroundColor(.gray)
                        }
                    }
                }
            }
            .padding()
            .background(Color(.systemGray6))
            .cornerRadius(12)
        }
        .padding()
    }
}

#Preview {
    AdvancedLeaderboard()
}
```

---

## 2. Layout Protocol 完整示例

### 等宽水平栈
```swift
struct EqualWidthHStack: Layout {
    let spacing: CGFloat?
    
    init(spacing: CGFloat? = nil) {
        self.spacing = spacing
    }
    
    // 测量子视图的最大尺寸
    private func maxSize(subviews: Subviews) -> CGSize {
        let subviewSizes = subviews.map { $0.sizeThatFits(.unspecified) }
        let maxSize: CGSize = subviewSizes.reduce(.zero) { currentMax, subviewSize in
            CGSize(
                width: max(currentMax.width, subviewSize.width),
                height: max(currentMax.height, subviewSize.height)
            )
        }
        return maxSize
    }
    
    // 获取子视图间的间距
    private func spacing(subviews: Subviews) -> [CGFloat] {
        subviews.indices.map { index in
            guard index < subviews.count - 1 else { return 0 }
            
            if let customSpacing = self.spacing {
                return customSpacing
            }
            
            return subviews[index].spacing.distance(
                to: subviews[index + 1].spacing,
                along: .horizontal
            )
        }
    }
    
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        guard !subviews.isEmpty else { return .zero }
        
        let maxSize = self.maxSize(subviews: subviews)
        let spacing = self.spacing(subviews: subviews)
        let totalSpacing = spacing.reduce(0) { $0 + $1 }
        
        return CGSize(
            width: maxSize.width * CGFloat(subviews.count) + totalSpacing,
            height: maxSize.height
        )
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        guard !subviews.isEmpty else { return }
        
        let maxSize = self.maxSize(subviews: subviews)
        let spacing = self.spacing(subviews: subviews)
        
        let placementProposal = ProposedViewSize(
            width: maxSize.width,
            height: maxSize.height
        )
        
        var x = bounds.minX + maxSize.width / 2
        
        for index in subviews.indices {
            subviews[index].place(
                at: CGPoint(x: x, y: bounds.midY),
                anchor: .center,
                proposal: placementProposal
            )
            x += maxSize.width + spacing[index]
        }
    }
}

// 等宽垂直栈
struct EqualWidthVStack: Layout {
    let spacing: CGFloat?
    
    init(spacing: CGFloat? = nil) {
        self.spacing = spacing
    }
    
    private func maxSize(subviews: Subviews) -> CGSize {
        let subviewSizes = subviews.map { $0.sizeThatFits(.unspecified) }
        return subviewSizes.reduce(.zero) { currentMax, subviewSize in
            CGSize(
                width: max(currentMax.width, subviewSize.width),
                height: max(currentMax.height, subviewSize.height)
            )
        }
    }
    
    private func spacing(subviews: Subviews) -> [CGFloat] {
        subviews.indices.map { index in
            guard index < subviews.count - 1 else { return 0 }
            
            if let customSpacing = self.spacing {
                return customSpacing
            }
            
            return subviews[index].spacing.distance(
                to: subviews[index + 1].spacing,
                along: .vertical
            )
        }
    }
    
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        guard !subviews.isEmpty else { return .zero }
        
        let maxSize = self.maxSize(subviews: subviews)
        let spacing = self.spacing(subviews: subviews)
        let totalSpacing = spacing.reduce(0) { $0 + $1 }
        
        return CGSize(
            width: maxSize.width,
            height: maxSize.height * CGFloat(subviews.count) + totalSpacing
        )
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        guard !subviews.isEmpty else { return }
        
        let maxSize = self.maxSize(subviews: subviews)
        let spacing = self.spacing(subviews: subviews)
        
        let placementProposal = ProposedViewSize(
            width: maxSize.width,
            height: maxSize.height
        )
        
        var y = bounds.minY + maxSize.height / 2
        
        for index in subviews.indices {
            subviews[index].place(
                at: CGPoint(x: bounds.midX, y: y),
                anchor: .center,
                proposal: placementProposal
            )
            y += maxSize.height + spacing[index]
        }
    }
}

// 使用示例
struct EqualWidthLayoutDemo: View {
    @State var pets: [Pet] = Pet.exampleData
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Voting Buttons (Equal Width)")
                .font(.title)
            
            EqualWidthHStack {
                ForEach($pets) { $pet in
                    Button(action: { pet.votes += 1 }) {
                        Text(pet.type)
                            .frame(maxWidth: .infinity)
                    }
                    .buttonStyle(.bordered)
                }
            }
            
            Spacer()
        }
        .padding()
    }
}

#Preview {
    EqualWidthLayoutDemo()
}
```

### 辐射形布局
```swift
// 定义排名值键
private struct Rank: LayoutValueKey {
    static let defaultValue: Int = 1
}

extension View {
    func rank(_ value: Int) -> some View {
        layoutValue(key: Rank.self, value: value)
    }
}

struct RadialLayout: Layout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        // 占用整个可用空间
        return proposal.replacingUnspecifiedDimensions()
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        let radius = min(bounds.size.width, bounds.size.height) / 3.0
        let angle = Angle.degrees(360.0 / Double(subviews.count)).radians
        
        // 根据排名计算旋转偏移
        let ranks = subviews.map { subview in
            subview[Rank.self]
        }
        let offset = calculateOffset(ranks: ranks)
        
        for (index, subview) in subviews.enumerated() {
            // 从上方开始，顺时针旋转
            var point = CGPoint(x: 0, y: -radius)
                .applying(CGAffineTransform(
                    rotationAngle: angle * Double(index) + offset
                ))
            
            // 移动到容器中心
            point.x += bounds.midX
            point.y += bounds.midY
            
            subview.place(
                at: point,
                anchor: .center,
                proposal: .unspecified
            )
        }
    }
    
    // 根据排名计算旋转角度
    private func calculateOffset(ranks: [Int]) -> Double {
        guard let firstRank = ranks.first else { return 0 }
        
        // 让排名最高的项目在顶部
        let maxRank = ranks.max() ?? 1
        let positions = ranks.indices.filter { ranks[$0] == maxRank }
        
        if positions.count == 1, let position = positions.first {
            // 单个最高排名，计算旋转使其到顶部
            let angle = Angle.degrees(360.0 / Double(ranks.count)).radians
            return -angle * Double(position)
        }
        
        return 0
    }
}

// 使用示例
struct RadialLayoutDemo: View {
    @State var pets: [Pet] = Pet.exampleData
    
    var body: some View {
        VStack {
            Text("Pet Rankings")
                .font(.title)
            
            RadialLayout {
                ForEach(pets) { pet in
                    ZStack {
                        Circle()
                            .fill(Color.blue.opacity(0.3))
                        Text(pet.type)
                            .font(.caption)
                    }
                    .frame(width: 60, height: 60)
                    .rank(rank(for: pet))
                }
            }
            
            Spacer()
        }
        .padding()
    }
    
    func rank(for pet: Pet) -> Int {
        let sorted = pets.sorted { $0.votes > $1.votes }
        return sorted.firstIndex(where: { $0.id == pet.id }) ?? 0
    }
}

#Preview {
    RadialLayoutDemo()
}
```

---

## 3. ViewThatFits 完整示例

### 基础响应式布局
```swift
struct VotingButtons: View {
    @State var pets: [Pet] = Pet.exampleData
    
    var body: some View {
        VotingButtonsContent(pets: $pets)
    }
}

// 提取可复用的按钮内容
struct VotingButtonsContent: View {
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

// 响应式容器
struct ResponsiveVotingButtons: View {
    @Binding var pets: [Pet]
    
    var body: some View {
        ViewThatFits {
            // 首选：水平等宽
            EqualWidthHStack {
                VotingButtonsContent(pets: $pets)
            }
            
            // 次选：垂直等宽
            EqualWidthVStack {
                VotingButtonsContent(pets: $pets)
            }
            
            // 最后选择：普通垂直栈（紧凑）
            VStack {
                VotingButtonsContent(pets: $pets)
            }
        }
    }
}

// 完整演示
struct ViewThatFitsDemo: View {
    @State var pets: [Pet] = Pet.exampleData
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Voting Interface")
                .font(.title)
            
            Leaderboard(pets: pets)
            
            Spacer()
            
            ResponsiveVotingButtons(pets: $pets)
        }
        .padding()
    }
}

#Preview {
    ViewThatFitsDemo()
}
```

### 三向响应式布局
```swift
struct MultiDeviceLayout: View {
    @State var pets: [Pet] = Pet.exampleData
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    
    var body: some View {
        ViewThatFits(in: .horizontal) {
            // iPad 横屏或大屏：三列
            HStack(spacing: 20) {
                VStack {
                    Text("Leaderboard")
                    Leaderboard(pets: pets)
                }
                
                VStack {
                    Text("Details")
                    DetailView(pets: pets)
                }
                
                VStack {
                    Text("Voting")
                    ResponsiveVotingButtons(pets: $pets)
                }
            }
            
            // iPhone 横屏：两列
            HStack(spacing: 12) {
                Leaderboard(pets: pets)
                
                VStack {
                    DetailView(pets: pets)
                    ResponsiveVotingButtons(pets: $pets)
                }
            }
            
            // iPhone 竖屏：单列
            VStack(spacing: 12) {
                Leaderboard(pets: pets)
                DetailView(pets: pets)
                ResponsiveVotingButtons(pets: $pets)
            }
        }
        .padding()
    }
}

struct DetailView: View {
    var pets: [Pet]
    var totalVotes: Int {
        pets.reduce(0) { $0 + $1.votes }
    }
    
    var body: some View {
        List(pets) { pet in
            HStack {
                Text(pet.type)
                Spacer()
                Text("\(Int(Double(pet.votes) / Double(totalVotes) * 100))%")
                    .font(.headline)
            }
        }
    }
}

#Preview {
    MultiDeviceLayout()
}
```

---

## 4. AnyLayout 完整示例

### 基础布局转换
```swift
struct DynamicLayoutDemo: View {
    @State var pets: [Pet] = Pet.exampleData
    @State var useHorizontal = true
    
    var body: some View {
        VStack {
            Toggle("Use Horizontal Layout", isOn: $useHorizontal)
                .padding()
            
            let layout = useHorizontal 
                ? AnyLayout(HStackLayout(spacing: 12))
                : AnyLayout(VStackLayout(spacing: 12))
            
            layout {
                ForEach(pets) { pet in
                    VStack {
                        Text(pet.type)
                            .font(.headline)
                        Text("\(pet.votes) votes")
                            .font(.caption)
                    }
                    .padding()
                    .background(Color.blue.opacity(0.3))
                    .cornerRadius(8)
                }
            }
            .animation(.easeInOut, value: useHorizontal)
            
            Spacer()
        }
    }
}

#Preview {
    DynamicLayoutDemo()
}
```

### 复杂状态驱动的布局转换
```swift
struct SmartPetLayout: View {
    @State var pets: [Pet] = Pet.exampleData
    
    // 计算是否是三向平局
    var isThreeWayTie: Bool {
        guard let maxVotes = pets.map({ $0.votes }).max() else { return false }
        let topCount = pets.filter { $0.votes == maxVotes }.count
        return topCount == 3
    }
    
    // 根据平局状态选择布局
    var selectedLayout: AnyLayout {
        if isThreeWayTie {
            // 平局时用水平展示
            return AnyLayout(HStackLayout(spacing: 12))
        } else {
            // 非平局时用辐射形
            return AnyLayout(RadialLayout())
        }
    }
    
    var body: some View {
        VStack {
            Text("Pet Poll")
                .font(.title)
            
            if isThreeWayTie {
                Text("🎯 Three-Way Tie!")
                    .font(.caption)
                    .foregroundColor(.orange)
            }
            
            selectedLayout {
                ForEach(pets) { pet in
                    ZStack {
                        Circle()
                            .fill(
                                pets.first { $0.id == pet.id }?.votes == pets.map({ $0.votes }).max()
                                ? Color.green.opacity(0.4)
                                : Color.blue.opacity(0.3)
                            )
                        VStack(spacing: 2) {
                            Text(pet.type)
                                .font(.caption)
                            Text("\(pet.votes)")
                                .font(.caption2)
                        }
                    }
                    .frame(width: 70, height: 70)
                    .rank(rank(for: pet))
                }
            }
            .animation(.spring(), value: isThreeWayTie)
            
            Spacer()
            
            // 投票按钮
            HStack(spacing: 8) {
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
            .padding()
        }
        .padding()
    }
    
    func rank(for pet: Pet) -> Int {
        let sorted = pets.sorted { $0.votes > $1.votes }
        return sorted.firstIndex(where: { $0.id == pet.id }) ?? 0
    }
}

#Preview {
    SmartPetLayout()
}
```

### 多条件布局选择
```swift
struct AdaptiveLayoutView: View {
    @State var pets: [Pet] = Pet.exampleData
    @State var displayMode: DisplayMode = .grid
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    
    enum DisplayMode {
        case list
        case grid
        case carousel
    }
    
    var selectedLayout: AnyLayout {
        switch displayMode {
        case .list:
            return AnyLayout(VStackLayout(alignment: .leading, spacing: 8))
        case .grid:
            return AnyLayout(HStackLayout(spacing: 12))
        case .carousel:
            return AnyLayout(RadialLayout())
        }
    }
    
    var body: some View {
        VStack {
            Picker("Display Mode", selection: $displayMode) {
                Text("List").tag(DisplayMode.list)
                Text("Grid").tag(DisplayMode.grid)
                Text("Carousel").tag(DisplayMode.carousel)
            }
            .pickerStyle(.segmented)
            .padding()
            
            selectedLayout {
                ForEach(pets) { pet in
                    PetCard(pet: pet, displayMode: displayMode)
                }
            }
            .animation(.easeInOut, value: displayMode)
            
            Spacer()
        }
        .padding()
    }
}

struct PetCard: View {
    let pet: Pet
    let displayMode: AdaptiveLayoutView.DisplayMode
    
    var body: some View {
        VStack {
            Text(pet.type)
                .font(.headline)
            Text("\(pet.votes) votes")
                .font(.caption)
                .foregroundColor(.gray)
        }
        .frame(maxWidth: displayMode == .list ? .infinity : nil)
        .padding()
        .background(Color.blue.opacity(0.1))
        .cornerRadius(8)
    }
}

#Preview {
    AdaptiveLayoutView()
}
```

---

## 5. 综合示例：完整的投票应用

```swift
// 完整的宠物投票应用
struct PetPollApp: View {
    @State var pets: [Pet] = Pet.exampleData
    @State var showResults = false
    
    var totalVotes: Int {
        pets.reduce(0) { $0 + $1.votes }
    }
    
    var isThreeWayTie: Bool {
        guard let maxVotes = pets.map({ $0.votes }).max() else { return false }
        return pets.filter { $0.votes == maxVotes }.count == 3
    }
    
    var body: some View {
        ZStack {
            VStack(spacing: 20) {
                // 标题
                VStack(spacing: 4) {
                    Text("🐾 Pet Poll 2024")
                        .font(.title)
                    Text("Vote for your favorite companion")
                        .font(.caption)
                        .foregroundColor(.gray)
                }
                
                // 排行榜
                if !showResults {
                    Leaderboard(pets: pets)
                        .transition(.scale)
                } else {
                    AdvancedLeaderboard()
                        .transition(.scale)
                }
                
                Spacer()
                
                // 投票区域
                VStack(spacing: 12) {
                    Text("Cast Your Vote")
                        .font(.headline)
                    
                    ResponsiveVotingButtons(pets: $pets)
                }
                
                // 切换按钮
                Button(action: { showResults.toggle() }) {
                    Label(
                        showResults ? "Show Simple" : "Show Advanced",
                        systemImage: showResults ? "list.bullet" : "chart.bar.fill"
                    )
                    .frame(maxWidth: .infinity)
                }
                .buttonStyle(.bordered)
            }
            .padding()
        }
    }
}

#Preview {
    PetPollApp()
}
```

---

## 使用建议

### 代码组织
- 将布局类型定义在单独的文件中，便于复用
- 使用扩展为 View 添加便利修饰符（如 `.rank()`）
- 提取可复用的内容视图（如 `VotingButtonsContent`）

### 性能优化
- Grid：数据量 > 100 时改用 LazyVGrid
- Layout Protocol：复杂计算时使用 cache
- ViewThatFits：选项 > 4 个时需要测试性能
- AnyLayout：避免每帧改变布局条件

### 测试覆盖
- 在 iPhone、iPad、Mac 上都测试过
- 测试动态类型（小、标准、大、超大）
- 测试国际化（RTL 语言等）
- 测试旋转和屏幕大小变化
