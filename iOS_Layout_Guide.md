# iOS SwiftUI 布局工具完全指南

## 概述

这份指南整理了 Apple 推荐的四个核心布局工具：**Grid**、**Layout Protocol**、**ViewThatFits** 和 **AnyLayout**。每个工具解决不同的布局问题，理解它们的适用场景和最佳实践能帮助你写出更高效、更灵活的 SwiftUI 代码。

---

## 一、Grid（网格布局）

### 核心定义
Grid 是一个自动计算单元格大小的二维布局容器，适合展示静态的、需要精确对齐的表格类数据。与 LazyVGrid/LazyHGrid 不同，Grid 在初始化时加载所有视图。

### 适用场景

**✅ 最佳场景：**
- **表格数据展示**：排行榜、数据对比表、列表等需要多列对齐的内容
- **固定行数内容**：数据量较小、行数可预测的界面
- **需要精确对齐**：多列间需要完美对齐（如名字列左对齐、数值列右对齐）
- **动态内容小幅变化**：数据更新时需要平滑的视觉体验

**❌ 不适合的场景：**
- 需要横向或纵向滚动的长列表
- 内容非常多且需要懒加载
- 单一维度的简单排列

### 工作原理

```
1. Grid 在布局阶段加载所有 subviews
2. 测量每一列中最宽的 view，设为该列宽度
3. 测量每一行中最高的 view，设为该行高度
4. 应用对齐规则，布局所有单元格
```

### 关键特性

| 特性 | 说明 | 用法 |
|-----|------|------|
| 自动尺寸计算 | 无需预指定行高/列宽 | 默认行为，自动生效 |
| 灵活视图支持 | ProgressView 等弹性视图占用剩余空间 | 自动分配空间，无需特殊配置 |
| 跨列合并 | 单个视图可跨多列 | `gridCellColumns(3)` 或直接写在 GridRow 外 |
| 列对齐控制 | 每列可独立设置对齐方式 | `gridColumnAlignment(.trailing)` |
| 全局对齐 | 为整个 Grid 设置默认对齐 | `Grid(alignment: .leading)` |

### 最佳实践

#### 1. 充分利用自动尺寸计算
```swift
struct Leaderboard: View {
    var pets: [Pet]
    var totalVotes: Int

    var body: some View {
        Grid(alignment: .leading) {
            ForEach(pets) { pet in
                GridRow {
                    Text(pet.type)                    // 左对齐（全局设置）
                    ProgressView(
                        value: Double(pet.votes),
                        total: Double(totalVotes)
                    )                                 // 占用剩余空间
                    Text("\(pet.votes)")
                        .gridColumnAlignment(.trailing) // 该列右对齐
                }
                Divider()                            // 自动跨所有列
            }
        }
        .padding()
    }
}
```

**为什么这样做：**
- Grid 自动计算第 1 列的宽度（最长的名字）
- Grid 自动计算第 3 列的宽度（最多的数字）
- ProgressView 占用中间剩余空间
- Divider 无需指定列数，自动跨越所有列

#### 2. 处理动态类型和国际化文本
```swift
// ❌ 避免：硬编码宽度
GridRow {
    Text(petName).frame(width: 60)  // 在大文本或其他语言下会破裂
}

// ✅ 推荐：让 Grid 自动计算
Grid {
    ForEach(pets) { pet in
        GridRow {
            Text(pet.type)  // Grid 自动适应最长的名字
        }
    }
}
```

**为什么这样做：**
- 支持动态类型（用户设置的文本大小）
- 支持国际化（某些语言文本更长）
- 一旦任何单元格改变，Grid 自动重新计算

#### 3. 分离数据逻辑和布局逻辑
```swift
// 推荐：提取出独立的 GridRow 构建器
struct LeaderboardRow: View {
    let pet: Pet
    let totalVotes: Int
    
    var body: some View {
        GridRow {
            Text(pet.type)
            ProgressView(value: Double(pet.votes), total: Double(totalVotes))
            Text("\(pet.votes)")
                .gridColumnAlignment(.trailing)
        }
    }
}

// 在主视图中使用
Grid(alignment: .leading) {
    ForEach(pets) { pet in
        LeaderboardRow(pet: pet, totalVotes: totalVotes)
        Divider()
    }
}
```

**为什么这样做：**
- 提高代码可读性
- 便于单独测试和复用
- 减少主视图的复杂度

#### 4. 处理跨列元素
```swift
// 分隔线跨所有列
Grid {
    ForEach(pets) { pet in
        GridRow {
            Text(pet.name)
            ProgressView(...)
            Text("\(pet.votes)")
        }
        // 方案 1：直接写在 GridRow 外（简洁，推荐）
        Divider()
        
        // 或者方案 2：显式指定跨列数
        // GridRow {
        //     Divider().gridCellColumns(3)
        // }
    }
}
```

---

## 二、Layout Protocol（自定义布局）

### 核心定义
Layout Protocol 让你创建自定义布局容器，直接与 SwiftUI 的布局引擎交互。它提供两个必需方法来控制子视图的尺寸和位置。

### 适用场景

**✅ 最佳场景：**
- **特殊对齐需求**：内置 Stack 无法满足的自定义排列逻辑
- **复杂的相对定位**：视图位置依赖于其他视图的大小或相对关系
- **自定义的测量逻辑**：需要先测量子视图，再根据测量结果决定布局方式
- **可复用的布局模式**：创建一个通用的布局类型供多处使用

**❌ 不适合的场景：**
- 简单的线性排列（用 HStack/VStack）
- 不需要测量子视图（用修饰符实现更简单）
- 一次性的、简单的调整（用 frame、padding 修饰符）

### 工作原理

```
1. sizeThatFits 阶段：
   - 接收容器的尺寸建议（proposal）
   - 测量每个子视图的理想尺寸
   - 返回整个布局容器的尺寸

2. placeSubviews 阶段：
   - 接收容器分配的具体边界（bounds）
   - 遍历每个子视图，计算其位置
   - 调用 place() 将子视图放在指定位置
```

### 必需方法详解

#### sizeThatFits 方法
```swift
func sizeThatFits(
    proposal: ProposedViewSize,      // 容器建议的尺寸（可能包含 nil）
    subviews: Subviews,              // 子视图代理集合（不是真实视图）
    cache: inout Void                // 可选缓存
) -> CGSize {
    // 需要返回这个布局的理想尺寸
}
```

**参数说明：**
- `proposal`: 包含 `width` 和 `height`，值可能为具体数字或 `nil`（表示未指定）
- `subviews`: 代理对象集合，可以询问每个子视图的理想尺寸，但无法直接访问视图本身
- 返回值: 要求的具体尺寸

#### placeSubviews 方法
```swift
func placeSubviews(
    in bounds: CGRect,               // 容器分配的实际空间
    proposal: ProposedViewSize,      // 同上
    subviews: Subviews,              // 同上
    cache: inout Void                // 同上
) {
    // 遍历 subviews，调用 place() 放置每个子视图
}
```

**bounds 坐标系统：**
- 原点（0,0）在左上角
- X 轴向右，Y 轴向下
- 注意：bounds.origin 不一定是 (0,0)，需要用 `minX`、`midY` 等属性
- RTL（从右到左）语言自动翻转处理

### 最佳实践

#### 1. 测量子视图时使用合适的建议
```swift
// ❌ 不推荐：无脑使用 unspecified
let sizes = subviews.map { $0.sizeThatFits(.unspecified) }

// ✅ 推荐：根据场景使用不同建议
// 获取理想尺寸（无约束）
let idealSize = subview.sizeThatFits(.unspecified)

// 获取最小尺寸
let minSize = subview.sizeThatFits(CGSize(width: 0, height: 0))

// 测试特定建议
let constrainedSize = subview.sizeThatFits(CGSize(width: 100, height: 100))
```

#### 2. 尊重视图间距首选项（ViewSpacing）
```swift
// ❌ 避免：硬编码间距
let fixedSpacing = 16.0

// ✅ 推荐：使用 ViewSpacing 获取系统推荐的间距
private func spacing(subviews: Subviews) -> [CGFloat] {
    subviews.indices.map { index in
        guard index < subviews.count - 1 else { return 0 }
        // 获取当前视图和下一个视图之间的推荐间距
        return subviews[index].spacing.distance(
            to: subviews[index + 1].spacing,
            along: .horizontal
        )
    }
}
```

**为什么这样做：**
- 自动遵循 Apple 设计指南
- 与系统其他布局保持一致
- 支持不同平台的不同间距设置

#### 3. 构建助手方法避免代码重复
```swift
struct EqualWidthHStack: Layout {
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
    
    // 获取间距数组
    private func spacing(subviews: Subviews) -> [CGFloat] {
        subviews.indices.map { index in
            guard index < subviews.count - 1 else { return 0 }
            return subviews[index].spacing.distance(
                to: subviews[index + 1].spacing,
                along: .horizontal
            )
        }
    }
    
    // 在 sizeThatFits 和 placeSubviews 中都使用这些方法
    func sizeThatFits(...) -> CGSize {
        let maxSize = maxSize(subviews: subviews)
        let spacing = spacing(subviews: subviews)
        // ...
    }
    
    func placeSubviews(...) {
        let maxSize = maxSize(subviews: subviews)  // 复用同一逻辑
        let spacing = spacing(subviews: subviews)
        // ...
    }
}
```

#### 4. 使用坐标系统属性而不是硬编码
```swift
// ❌ 不推荐：假设原点是 (0,0)
var x = 0.0
for subview in subviews {
    subview.place(at: CGPoint(x: x, y: 0), anchor: .topLeading, proposal: proposal)
    x += width
}

// ✅ 推荐：使用 bounds 属性
var x = bounds.minX  // 正确处理非零原点
for subview in subviews {
    subview.place(at: CGPoint(x: x, y: bounds.midY), anchor: .center, proposal: proposal)
    x += width
}
```

**为什么这样做：**
- 支持布局组合（一个布局嵌套在另一个布局中）
- 正确处理所有坐标系统情况

#### 5. 使用 LayoutValueKey 传递数据到布局
```swift
// 定义一个布局值键
private struct Rank: LayoutValueKey {
    static let defaultValue: Int = 1  // 必须提供默认值，也定义了类型
}

// 创建便利修饰符
extension View {
    func rank(_ value: Int) -> some View {
        layoutValue(key: Rank.self, value: value)
    }
}

// 在视图中使用
struct RadialLayout: Layout {
    func placeSubviews(...) {
        for (index, subview) in subviews.enumerated() {
            // 读取每个子视图的 rank 值
            let rank = subview[Rank.self]
            
            // 根据 rank 计算位置
            let angle = calculateAngle(for: rank)
            // ...
        }
    }
}

// 应用到视图
RadialLayout {
    Avatar().rank(1)
    Avatar().rank(2)
    Avatar().rank(3)
}
```

**为什么这样做：**
- 布局无法访问外部数据模型
- LayoutValueKey 是布局和视图通信的标准方式
- 类型安全，编译时检查

#### 6. 考虑性能优化（缓存）
```swift
struct ComplexLayout: Layout {
    // 只在计算密集的布局中使用缓存
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Cache
    ) -> CGSize {
        // 第一次调用时计算，存储在 cache 中
        if cache.sizes == nil {
            cache.sizes = expensiveCalculation(subviews)
        }
        
        return finalSize
    }
    
    // 定义缓存结构
    struct Cache {
        var sizes: [CGSize]?
    }
}
```

**何时使用缓存：**
- 子视图很多（> 100）
- 计算逻辑复杂（涉及三角函数、循环等）
- Instruments 分析显示性能问题

---

## 三、ViewThatFits（自适应容器）

### 核心定义
ViewThatFits 是一个选择器容器，从提供的多个选项中选择第一个能在可用空间中正常显示的视图。它会依次尝试每个备选方案，直到找到适合的。

### 适用场景

**✅ 最佳场景：**
- **响应式设计**：同一内容在不同屏幕尺寸/方向下需要不同布局
- **动态类型支持**：用户放大文本时需要备选的更紧凑布局
- **优雅降级**：优先使用理想布局，不适合时切换到备选方案
- **一次性尝试**：不需要频繁在多个状态间切换

**❌ 不适合的场景：**
- 需要逐帧精细控制的动画
- 频繁在状态间切换（性能问题）
- 单一最优方案（直接用那个方案）

### 工作原理

```
1. ViewThatFits 接收多个视图选项（通常是不同布局的同一内容）
2. 从第一个选项开始，测试是否适合可用空间
3. 如果不适合，尝试下一个选项
4. 找到适合的选项后，使用该选项进行渲染
5. 当容器大小改变时，重新测试所有选项
```

### 最佳实践

#### 1. 组织视图选项的顺序（从最优到降级）
```swift
// ✅ 推荐：优先级从高到低
ViewThatFits {
    // 首选：水平布局（最美观）
    MyEqualWidthHStack {
        Buttons(pets: $pets)
    }
    
    // 次选：垂直布局（次优但仍然合理）
    MyEqualWidthVStack {
        Buttons(pets: $pets)
    }
    
    // 最后选择：单列滚动（紧凑备选）
    ScrollView {
        VStack {
            Buttons(pets: $pets)
        }
    }
}
```

**为什么这样做：**
- 用户会先看到最美观的版本
- 随着空间减少，优雅降级到备选方案
- 避免突然的"无法显示"情况

#### 2. 创建对应的布局变体
```swift
// 不推荐：让 ViewThatFits 选择同一个布局的不同尺寸
ViewThatFits {
    HStack { Content }
    HStack { Content }  // 两个一样的东西，无意义
}

// ✅ 推荐：为不同屏幕尺寸提供不同的布局
ViewThatFits {
    // iPad 或大屏幕：并排
    HStack(spacing: 20) { Content }
    
    // iPhone 或小屏幕：上下排列
    VStack(spacing: 12) { Content }
}
```

#### 3. 共享内容，避免重复代码
```swift
// 提取共享的按钮内容
struct ButtonsContent: View {
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

// 使用 ViewThatFits 选择布局
struct StackedButtons: View {
    @Binding var pets: [Pet]
    
    var body: some View {
        ViewThatFits {
            MyEqualWidthHStack {
                ButtonsContent(pets: $pets)
            }
            MyEqualWidthVStack {
                ButtonsContent(pets: $pets)
            }
        }
    }
}
```

**为什么这样做：**
- 按钮逻辑只定义一次
- 维护更简单（改动一处生效）
- 确保不同布局下内容一致

#### 4. 考虑边界条件
```swift
// 考虑：极小屏幕、横屏、平板等所有情况
ViewThatFits {
    // 场景 1：宽敞空间（iPad 横屏）
    HStack(spacing: 20) {
        LeftPanel()
        MainContent()
        RightPanel()
    }
    
    // 场景 2：中等空间（iPhone 横屏）
    HStack(spacing: 12) {
        MainContent()
        RightPanel()
    }
    
    // 场景 3：狭窄空间（iPhone 竖屏）
    VStack(spacing: 12) {
        MainContent()
        RightPanel()
    }
}
```

---

## 四、AnyLayout（布局转换）

### 核心定义
AnyLayout 是一个类型擦除容器，允许在保持视图结构恒定的前提下，在不同布局协议实现之间无缝切换。关键是：视图层级的身份（identity）保持不变。

### 适用场景

**✅ 最佳场景：**
- **动态布局切换**：根据状态或数据在不同布局间切换
- **平滑过渡动画**：切换时需要动画效果
- **保持视图身份**：避免重置状态（如文本输入框的内容）
- **复杂的布局决策**：布局选择依赖于数据状态

**❌ 不适合的场景：**
- 静态布局（直接选择一种布局）
- 不需要动画的简单切换（用条件语句更简单）
- 视图集合本身改变（身份改变，用条件语句）

### 工作原理

```
1. AnyLayout 包装一个符合 Layout 协议的类型实例
2. 通过切换 AnyLayout 的值，改变内部使用的布局
3. 关键：视图的结构和身份保持不变
   - 同样的 ForEach 集合
   - 同样的视图类型和顺序
   - 内部状态（@State）保持

4. SwiftUI 检测到布局改变，触发转换动画
```

### 与条件语句的区别

```swift
// ❌ 条件语句：视图身份改变，状态被重置
if useHStack {
    HStack { Content }  // 新创建的视图
} else {
    VStack { Content }  // 新创建的视图
}
// ❌ 问题：内部的 @State 变量会被重置

// ✅ AnyLayout：视图身份保持，只改变布局
let layout = useHStack ? AnyLayout(HStackLayout()) : AnyLayout(VStackLayout())
layout {
    Content  // 同一个视图树，只是布局改变
}
// ✅ 优点：内部 @State 保持，有动画
```

### 最佳实践

#### 1. 使用状态驱动的布局选择
```swift
struct Profile: View {
    var pets: [Pet]
    
    // 计算属性决定布局
    var isThreeWayTie: Bool {
        let ranks = pets.map(rank)
        return ranks.filter { $0 == 1 }.count == 3
    }
    
    var body: some View {
        // 基于状态选择布局（而非视图结构改变）
        let layout = isThreeWayTie 
            ? AnyLayout(HStackLayout()) 
            : AnyLayout(MyRadialLayout())
        
        layout {
            ForEach(pets) { pet in
                Avatar(pet: pet)
                    .rank(rank(pet))
            }
        }
        .animation(.default, value: pets)
    }
}
```

**为什么这样做：**
- 布局决策从状态衍生（而非视图结构）
- SwiftUI 能识别出这是视图转换，而非创建新视图
- 支持平滑的过渡动画

#### 2. 添加适当的动画
```swift
// ❌ 没有动画的切换（很生硬）
let layout = condition ? AnyLayout(LayoutA()) : AnyLayout(LayoutB())
layout {
    Content
}

// ✅ 推荐：显式添加动画
let layout = condition ? AnyLayout(LayoutA()) : AnyLayout(LayoutB())
layout {
    Content
}
.animation(.default, value: pets)  // 基于数据变化添加动画

// 或者针对特定条件的动画
.animation(
    isThreeWayTie ? .spring() : .easeInOut,
    value: isThreeWayTie
)
```

**为什么这样做：**
- 即使没有明确指定 animation，AnyLayout 也会尝试转换
- 显式指定 animation 确保一致的动画行为
- 可为不同情况设置不同的动画曲线

#### 3. 结合 LayoutValueKey 传递数据
```swift
// 定义在 RadialLayout 中使用的值键
private struct Rank: LayoutValueKey {
    static let defaultValue: Int = 1
}

extension View {
    func rank(_ value: Int) -> some View {
        layoutValue(key: Rank.self, value: value)
    }
}

// 使用 AnyLayout 时，LayoutValueKey 会被正确传递
let layout = isThreeWayTie 
    ? AnyLayout(HStackLayout())  // HStack 忽略 rank
    : AnyLayout(MyRadialLayout()) // RadialLayout 使用 rank

layout {
    ForEach(pets) { pet in
        Avatar(pet: pet)
            .rank(calculateRank(for: pet))  // 值被传递到两种布局
    }
}
```

**为什么这样做：**
- 在多种布局间共享配置数据
- 某个布局可能使用数据，某个可能忽略
- 保持代码 DRY

#### 4. 处理复杂的布局转换逻辑
```swift
// 推荐：提取布局选择逻辑到计算属性
struct ComplexLayoutView: View {
    var data: [Item]
    @State var isCompact = false
    
    var selectedLayout: AnyLayout {
        if isCompact {
            return AnyLayout(VStackLayout(alignment: .leading, spacing: 8))
        } else if data.count > 10 {
            return AnyLayout(LazyVStackLayout(spacing: 12))
        } else {
            return AnyLayout(VStackLayout(alignment: .center, spacing: 16))
        }
    }
    
    var body: some View {
        selectedLayout {
            ForEach(data) { item in
                ItemView(item: item)
            }
        }
        .animation(.easeInOut, value: isCompact)
        .animation(.easeInOut, value: data.count)
    }
}
```

**为什么这样做：**
- 布局选择逻辑清晰独立
- 易于添加新的布局条件
- 易于测试布局选择逻辑

---

## 五、工具对比矩阵

| 工具 | 主要用途 | 加载方式 | 适合内容量 | 学习难度 |
|-----|---------|---------|----------|---------|
| **Grid** | 表格式数据对齐 | 全量加载 | 小（< 100 行） | ⭐ |
| **Layout Protocol** | 自定义布局逻辑 | 按需实现 | 任意 | ⭐⭐⭐ |
| **ViewThatFits** | 响应式布局选择 | 支持懒加载 | 任意 | ⭐⭐ |
| **AnyLayout** | 布局动画转换 | 按内部布局 | 任意 | ⭐⭐ |

---

## 六、常见场景决策树

```
需要布局吗？
├─ 不需要
│  └─ 直接使用 Stack 或其他内置容器
│
├─ 是的，有多列数据
│  ├─ 数据量小（< 100）且需要精确对齐？
│  │  └─ 使用 Grid ✅
│  │
│  └─ 数据量大且需要滚动？
│     └─ 使用 LazyVGrid 或 LazyHGrid
│
├─ 是的，需要特殊排列逻辑
│  ├─ 内置 Stack 能满足？
│  │  └─ 使用 Stack
│  │
│  ├─ 需要先测量子视图？
│  │  └─ 使用 Layout Protocol ✅
│  │
│  └─ 简单的修饰符调整？
│     └─ 使用 frame、padding 等修饰符
│
├─ 是的，多个布局选项
│  ├─ 根据空间大小选择？
│  │  └─ 使用 ViewThatFits ✅
│  │
│  └─ 需要平滑转换动画？
│     └─ 使用 AnyLayout ✅
```

---

## 七、性能考虑

### Grid
- **优点**：全量加载让对齐完美
- **缺点**：大数据量时会卡顿
- **优化**：数据量 > 100 时考虑 LazyVGrid

### Layout Protocol
- **优点**：完全控制，灵活度高
- **缺点**：复杂计算会影响性能
- **优化**：使用 cache 参数存储中间结果，用 Instruments 测试

### ViewThatFits
- **优点**：自动选择，无需手动判断
- **缺点**：每次尝试都要测量所有选项
- **优化**：选项数量不要超过 3-4 个

### AnyLayout
- **优点**：平滑转换，视图身份保持
- **缺点**：频繁切换布局会有开销
- **优化**：避免每帧都改变布局决策

---

## 八、总结建议

### 快速选择指南

1. **展示排行榜、对比表？** → **Grid**
2. **内置布局无法满足需求？** → **Layout Protocol**
3. **根据屏幕大小调整布局？** → **ViewThatFits**
4. **在多个布局间平滑转换？** → **AnyLayout**

### 学习路径
1. 先掌握 Grid（最简单，日常最常用）
2. 再学 ViewThatFits（需要理解响应式设计）
3. 然后学 Layout Protocol（需要理解布局引擎）
4. 最后学 AnyLayout（高级特性，组合使用）

### 代码质量检查表

- [ ] 使用 Grid 时，没有硬编码列宽/行高
- [ ] 使用 Layout Protocol 时，尊重了 ViewSpacing
- [ ] 使用 ViewThatFits 时，选项从优到差排序
- [ ] 使用 AnyLayout 时，视图结构保持不变
- [ ] 所有布局都支持动态类型和国际化
- [ ] 在 iPad/Mac 和手机上都测试过
