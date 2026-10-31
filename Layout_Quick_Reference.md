# iOS Layout 工具 - 快速参考卡

## 工具对比速览

```
┌─────────────┬──────────────┬────────┬──────────┬────────┐
│ 工具        │ 主要功能     │ 难度   │ 学习需时 │ 日常频 │
├─────────────┼──────────────┼────────┼──────────┼────────┤
│ Grid        │ 表格对齐     │ ⭐     │ 15 分    │ ⭐⭐⭐  │
│ Layout      │ 自定义布局   │ ⭐⭐⭐  │ 2 小时   │ ⭐⭐   │
│ ViewThatFits│ 响应式选择   │ ⭐⭐   │ 30 分    │ ⭐⭐   │
│ AnyLayout   │ 布局转换     │ ⭐⭐   │ 45 分    │ ⭐     │
└─────────────┴──────────────┴────────┴──────────┴────────┘
```

---

## 秒杀问题决策树

### Q1：需要表格式数据展示吗？
```
❓ 有多列、需要对齐、数据量小？
├─ YES → 用 Grid ✅
└─ NO  → 下一题
```

### Q2：内置 HStack/VStack 能满足吗？
```
❓ 只是简单横排/竖排？
├─ YES → 用内置 Stack ✅
└─ NO  → 下一题
```

### Q3：需要根据空间大小调整布局吗？
```
❓ 手机竖屏、横屏、平板需要不同布局？
├─ YES → 用 ViewThatFits ✅
└─ NO  → 下一题
```

### Q4：需要特殊测量和位置逻辑吗？
```
❓ 需要先测量子视图再决定怎么放？
├─ YES → 用 Layout Protocol ✅
└─ NO  → 下一题
```

### Q5：需要在多个布局间平滑转换吗？
```
❓ 根据状态改变布局、需要动画？
├─ YES → 用 AnyLayout ✅
└─ NO  → 重新审视需求
```

---

## Grid - 工作流

### 何时用
- ✅ 排行榜、价目表、数据对比
- ✅ 3-10 行数据、需要精确列对齐
- ✅ 不需要滚动
- ❌ 数据量 > 100（用 LazyVGrid）

### 核心语法
```swift
Grid(alignment: .leading, horizontalSpacing: 12, verticalSpacing: 8) {
    ForEach(items) { item in
        GridRow {
            Text(item.name)  // 列 1
            ProgressView(value: item.progress)  // 列 2
            Text("\(item.count)")
                .gridColumnAlignment(.trailing)  // 列 3，独立对齐
        }
        
        if item != items.last {
            Divider()  // 自动跨所有列
        }
    }
}
```

### 三个关键修饰符
| 修饰符 | 作用 | 用法 |
|------|------|------|
| `Grid(alignment:)` | 全局对齐 | `Grid(alignment: .leading)` |
| `.gridColumnAlignment()` | 单列对齐 | `Text("...").gridColumnAlignment(.trailing)` |
| `.gridCellColumns()` | 跨列 | `Divider().gridCellColumns(3)` 或直接在 GridRow 外 |

### 常见错误
```swift
// ❌ 错误：硬编码宽度（在大字体、其他语言下破裂）
GridRow {
    Text(name).frame(width: 60)
}

// ✅ 正确：让 Grid 自动计算
GridRow {
    Text(name)  // Grid 自动占据最宽的名字宽度
}
```

---

## Layout Protocol - 工作流

### 何时用
- ✅ 内置 Stack 无法满足的特殊排列
- ✅ 需要先测量子视图，再决定位置
- ✅ 可复用的自定义布局
- ❌ 简单修饰符能解决（用 frame/padding）

### 最小实现
```swift
struct MyLayout: Layout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        // 1. 测量子视图
        let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
        // 2. 计算总大小
        let totalWidth = sizes.map(\.width).reduce(0, +)
        let maxHeight = sizes.map(\.height).max() ?? 0
        // 3. 返回布局的尺寸
        return CGSize(width: totalWidth, height: maxHeight)
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        // 1. 测量子视图（同上，可提取到方法避免重复）
        let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
        // 2. 逐个放置
        var x = bounds.minX
        for (index, subview) in subviews.enumerated() {
            subview.place(
                at: CGPoint(x: x, y: bounds.midY),
                anchor: .center,
                proposal: ProposedViewSize(width: sizes[index].width, height: sizes[index].height)
            )
            x += sizes[index].width
        }
    }
}
```

### 关键概念
| 概念 | 说明 | 注意事项 |
|------|------|---------|
| `ProposedViewSize` | 容器建议的尺寸 | 可能包含 `nil` |
| `Subviews` | 子视图代理集合 | 只能用 `sizeThatFits()` 和 `place()` 与之交互 |
| `bounds.minX/midY` | 坐标系参考点 | 不总是 (0,0)，需要用这些属性 |
| `ViewSpacing` | 视图间距偏好 | 用 `subview.spacing.distance(to:along:)` 获取 |
| `LayoutValueKey` | 向布局传递数据 | 用 `layoutValue(key:value:)` 修饰符设置 |

### 三行代码添加间距尊重
```swift
private func spacing(subviews: Subviews) -> [CGFloat] {
    subviews.indices.map { index in
        guard index < subviews.count - 1 else { return 0 }
        return subviews[index].spacing.distance(to: subviews[index + 1].spacing, along: .horizontal)
    }
}
```

---

## ViewThatFits - 工作流

### 何时用
- ✅ 多个布局选项，根据空间大小选择
- ✅ 响应式设计：竖屏用竖排，横屏用横排
- ✅ 动态类型：大字体时用更紧凑的备选
- ❌ 需要频繁切换（性能问题）

### 核心语法
```swift
ViewThatFits {
    // 选项 1：优先级最高
    HStack { Content }
    
    // 选项 2：空间不够时尝试
    VStack { Content }
    
    // 选项 3：最后的备选
    ScrollView {
        VStack { Content }
    }
}
```

### 最佳实践
```swift
// 第一步：提取可复用的内容
struct Content: View {
    var body: some View {
        ForEach(items) { item in
            ItemView(item: item)
        }
    }
}

// 第二步：创建响应式容器
struct ResponsiveLayout: View {
    var body: some View {
        ViewThatFits {
            // 优先：宽敞布局
            HStack(spacing: 20) { Content() }
            // 次选：紧凑布局
            VStack(spacing: 8) { Content() }
        }
    }
}
```

### 坑点
```swift
// ❌ 两个一样的选项，没意义
ViewThatFits {
    HStack { Content }
    HStack { Content }  // ⚠️ 重复了
}

// ❌ 内容不一致，用户会困惑
ViewThatFits {
    HStack { ContentA() }
    VStack { ContentB() }  // ⚠️ 不同的内容
}
```

---

## AnyLayout - 工作流

### 何时用
- ✅ 布局决策来自 @State（会改变）
- ✅ 需要在布局间平滑转换和动画
- ✅ 视图内部有状态，不能重创建
- ❌ 布局选择是静态的（直接选用一个）

### 核心语法
```swift
@State var useHStack = true

var body: some View {
    // 关键：选择布局来自变动的状态
    let layout = useHStack 
        ? AnyLayout(HStackLayout())
        : AnyLayout(VStackLayout())
    
    layout {
        ForEach(items) { item in
            ItemView(item: item)  // 同一个视图树
        }
    }
    .animation(.easeInOut, value: useHStack)  // 布局改变时动画
}
```

### 与条件语句的区别

| 方式 | 代码 | 视图身份 | 状态 | 能用动画 |
|------|------|--------|------|---------|
| 条件语句 | `if A { ... } else { ... }` | 改变 | 重置 | ❌ |
| AnyLayout | `layout { ... }` | 保持 | 保留 | ✅ |

### 正确用法
```swift
// ✅ 关键：布局决策来自状态/数据
struct SmartLayout: View {
    @State var showCompact = false
    
    // 布局决策衍生自状态
    var selectedLayout: AnyLayout {
        showCompact 
            ? AnyLayout(VStackLayout(spacing: 4))
            : AnyLayout(HStackLayout(spacing: 16))
    }
    
    var body: some View {
        selectedLayout {
            Content()  // 同一个视图树
        }
        .animation(.easeInOut, value: showCompact)
    }
}

// ❌ 避免：直接在视图树中用条件
var body: some View {
    if condition {
        HStack { Content }  // ⚠️ 新视图
    } else {
        VStack { Content }  // ⚠️ 新视图（状态丢失）
    }
}
```

---

## 性能快速检查表

### Grid
- [ ] 数据量 < 100 行
- [ ] 没有硬编码的列宽/行高
- [ ] 支持动态类型（使用系统字体大小）

### Layout Protocol
- [ ] 使用了 ViewSpacing（尊重系统间距）
- [ ] 有助手方法避免代码重复
- [ ] 复杂计算时考虑了 cache 参数

### ViewThatFits
- [ ] 选项 < 4 个
- [ ] 选项从优到差排序
- [ ] 各选项内容相同，只是布局不同

### AnyLayout
- [ ] 布局决策来自可变状态
- [ ] 不频繁改变布局选择（不是每帧）
- [ ] 添加了 animation 修饰符

---

## 常见场景一句话判断

| 场景 | 判断 |
|------|------|
| 排行榜 | → Grid |
| 按钮等宽 | → Layout Protocol |
| 响应屏幕尺寸 | → ViewThatFits |
| 点击按钮切换水平/竖 | → AnyLayout |
| 简单横排 | → HStack |
| 简单竖排 | → VStack |

---

## 代码模板库

### Grid 模板
```swift
struct TableView: View {
    var items: [Item]
    
    var body: some View {
        Grid(alignment: .leading) {
            ForEach(items) { item in
                GridRow {
                    Text(item.name)
                    ProgressView(value: item.value)
                    Text("\(item.count)")
                        .gridColumnAlignment(.trailing)
                }
                if item.id != items.last?.id {
                    Divider()
                }
            }
        }
    }
}
```

### Layout Protocol 模板
```swift
struct CustomLayout: Layout {
    private func calculateSize(subviews: Subviews) -> CGSize {
        let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
        return CGSize(
            width: sizes.map(\.width).reduce(0, +),
            height: sizes.map(\.height).max() ?? 0
        )
    }
    
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout Void) -> CGSize {
        calculateSize(subviews: subviews)
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout Void) {
        let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
        var x = bounds.minX
        for (index, subview) in subviews.enumerated() {
            subview.place(at: CGPoint(x: x, y: bounds.midY), anchor: .center, proposal: ProposedViewSize(width: sizes[index].width, height: sizes[index].height))
            x += sizes[index].width
        }
    }
}
```

### ViewThatFits 模板
```swift
struct ResponsiveLayout: View {
    var body: some View {
        ViewThatFits {
            HStack { Content() }
            VStack { Content() }
        }
    }
}
```

### AnyLayout 模板
```swift
struct AdaptiveLayout: View {
    @State var useHStack = true
    
    var body: some View {
        let layout = useHStack ? AnyLayout(HStackLayout()) : AnyLayout(VStackLayout())
        layout { Content() }
            .animation(.easeInOut, value: useHStack)
    }
}
```

---

## 调试技巧

### Grid 调试
```swift
// 查看每列的实际宽度
Grid(alignment: .leading) {
    ForEach(items) { item in
        GridRow {
            Text(item.name)
                .background(.red.opacity(0.3))  // 查看列宽
        }
    }
}
```

### Layout Protocol 调试
```swift
func placeSubviews(...) {
    // 添加日志查看计算结果
    let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
    print("Calculated sizes: \(sizes)")
    
    for (index, subview) in subviews.enumerated() {
        let position = CGPoint(x: x, y: bounds.midY)
        print("Placing subview \(index) at \(position)")
        subview.place(...)
    }
}
```

### ViewThatFits 调试
```swift
ViewThatFits {
    HStack { Content() }
        .background(.red.opacity(0.3))  // 红色=选中了这个
    VStack { Content() }
        .background(.blue.opacity(0.3))
}
```

### AnyLayout 调试
```swift
let layout = condition ? AnyLayout(LayoutA()) : AnyLayout(LayoutB())
layout {
    Content()
}
.border(condition ? .red : .blue)  // 清晰看到布局切换
```

---

## 何时不该用这些工具

| 工具 | 何时不要用 |
|-----|---------|
| Grid | 需要无限滚动（改用 LazyVGrid） |
| Layout Protocol | 只是简单的间距调整（用 padding） |
| ViewThatFits | 只有一个布局选项（直接用那个） |
| AnyLayout | 视图集合本身改变（用条件语句） |

---

## 学习路径建议

### Day 1（30 分）
- 理解 Grid 的自动尺寸计算
- 实现一个简单的排行榜

### Day 2（1 小时）
- 理解 ViewThatFits 的试用逻辑
- 实现一个响应屏幕尺寸的界面

### Day 3（2 小时）
- 理解 Layout Protocol 的两个方法
- 实现一个等宽 HStack

### Day 4（1.5 小时）
- 理解 AnyLayout 的身份保持机制
- 实现一个可以在 HStack/VStack 间转换的视图

### Day 5（1 小时）
- 用四个工具组合实现一个完整功能
- 理解何时选用哪个工具
