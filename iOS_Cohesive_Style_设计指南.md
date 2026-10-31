# iOS App 开发设计中的 Cohesive Style

## 概述

在 iOS App 设计领域，**Cohesive Style（内聚式风格、一致化设计风格）**并不是一个具体的官方组件名称，而是一种贯穿 Apple Human Interface Guidelines（HIG）的设计理念：**让整个 App 在视觉、交互、内容和品牌表达上形成一个统一、连贯的整体，而不是多个独立页面和功能的简单拼接。**

可以简单理解为：

> 用户在 App 中切换任何页面时，都应该感受到“这是同一个产品”。

---

## 1. 什么是 Cohesive Style

“Cohesive”可以理解为：

- 凝聚的
- 连贯的
- 统一的
- 内在一致的

在 App 产品设计中，它通常包含以下四个方面：

- **视觉统一**：颜色、字体、圆角、间距和图标具有一致规则。
- **交互统一**：相同类型的操作具有相同的行为和反馈。
- **品牌统一**：不同页面都能传达相同的产品气质。
- **内容统一**：文案、术语和信息表达方式保持一致。

Apple 的设计原则强调，应当使用用户已经熟悉的概念，并在整个设计中保持视觉效果和交互方式一致。这样可以帮助用户更快地学习产品，并相信新的操作会按照预期工作。

### 不一致的例子

假设一个 App 中存在以下设计：

- 页面 A 使用蓝色按钮，圆角为 12 pt。
- 页面 B 使用绿色按钮，圆角为 4 pt。
- 页面 C 使用红色按钮，圆角为 20 pt。

即使三个页面分别看起来没有明显问题，组合起来仍可能让用户感觉它们来自三个不同的产品或团队。

### Cohesive 的例子

整个 App 统一使用：

- 相同的按钮层级和样式
- 相同的颜色语义
- 相同的布局间距
- 相同的动画节奏
- 相同的导航模式
- 相同的文案语气

用户因此会把不同页面理解为同一个完整系统的一部分。

---

## 2. Cohesive Style 的主要作用

Cohesive Style 的核心目标并不只是“让界面看起来统一”，而是通过一致性降低用户理解和操作产品的成本。

| 作用 | 说明 |
|---|---|
| 降低学习成本 | 用户学会一种操作方式后，可以把经验迁移到其他页面。 |
| 提升可预测性 | 相同类型的组件和操作会产生相同结果。 |
| 减少误操作 | 清晰且稳定的交互规则可以减少判断错误。 |
| 增强品牌感 | 颜色、排版、动效和文案共同形成产品个性。 |
| 提升开发效率 | 设计系统和组件库可以减少重复开发。 |
| 降低维护成本 | 修改统一的组件或 Design Token，可以覆盖多个页面。 |
| 改善无障碍体验 | 统一的信息层级和系统行为更容易适配动态字体、VoiceOver 等能力。 |

换句话说，Cohesive Style 同时影响：

1. **用户体验质量**
2. **设计协作效率**
3. **工程实现效率**
4. **产品长期可维护性**

---

## 3. Cohesive Style 的五个层次

## 3.1 视觉一致性

这是最容易观察到的一层，包括：

- 字体与字号
- 颜色
- 图标
- 圆角
- 阴影
- 间距
- 页面背景
- 内容层级

### Typography

建议建立清晰的字体层级，例如：

- Large Title
- Title
- Headline
- Body
- Caption

在 SwiftUI 中，可以优先使用系统语义字体：

```swift
Text("页面标题")
    .font(.largeTitle)

Text("分组标题")
    .font(.headline)

Text("正文内容")
    .font(.body)

Text("辅助说明")
    .font(.caption)
```

相比在每个页面随意指定 20 pt、24 pt 或 26 pt，语义字体更容易保持统一，也能更好地支持 Dynamic Type。

### Color

颜色应当按照用途定义，而不是按照具体色值或页面定义，例如：

- Primary
- Secondary
- Accent
- Success
- Warning
- Error
- Background
- Surface

这样可以避免“登录按钮使用蓝色、保存按钮使用绿色、继续按钮又使用紫色”，但三者实际上都属于主操作的情况。

---

## 3.2 组件一致性

相同功能的组件应当具有稳定的样式和行为。例如，可以建立以下按钮层级：

- **Primary Button**：页面最重要的操作
- **Secondary Button**：次要操作
- **Tertiary Button**：低优先级或文字型操作
- **Destructive Button**：删除、清空等危险操作

组件一致性不仅要求按钮“看起来相同”，还要求以下内容一致：

- 点击区域
- 禁用状态
- 加载状态
- 按压反馈
- 色彩对比度
- 错误反馈
- VoiceOver Label

设计稳定的组件库后，开发者不需要在每个页面重新决定这些规则。

---

## 3.3 交互一致性

交互一致性决定了用户能否预测 App 的行为。

例如，页面返回应尽量遵循统一规则：

- 导航栈中的页面使用系统返回按钮和侧滑返回。
- 临时任务使用 Sheet，并通过下滑或关闭按钮退出。
- 沉浸式全屏任务使用 Full Screen Cover，并提供明确的退出入口。

应避免在没有明确语义差异时出现以下混合行为：

- 页面 A 通过左滑返回
- 页面 B 通过右滑返回
- 页面 C 必须点击右上角关闭
- 页面 D 点击背景区域才能退出

一致的交互能够让用户把注意力放在任务本身，而不是反复研究界面怎么操作。

---

## 3.4 动效一致性

动效不仅是装饰，也是界面状态变化的说明。

统一的动效语言通常包括：

- 相似的动画时长
- 相似的缓动曲线
- 一致的进入和退出方向
- 与操作来源相匹配的空间关系
- 对 Reduce Motion 设置的支持

例如，Sheet 从屏幕下方出现，也从下方退出。这种一致性可以帮助用户理解界面之间的层级关系。

好的 Cohesive Motion 应当回答三个问题：

1. 内容从哪里来？
2. 当前界面发生了什么变化？
3. 用户接下来可以做什么？

---

## 3.5 内容与文案一致性

Cohesive Style 也包括文字表达。

应统一：

- 功能名称
- 按钮动词
- 错误提示格式
- 标点规则
- 专业术语
- 语气和礼貌程度

例如，同一个操作不应在不同页面分别称为：

- 保存
- 提交
- 完成修改
- 确认变更

除非这些词确实代表不同操作。

简洁的按钮文案通常更容易理解，例如：

- 继续
- 保存
- 完成
- 重试
- 取消

不要把简单动作写成冗长的系统命令式表达。

---

## 4. Cohesive Style 与 Apple 系统设计的关系

Apple 平台强调 Familiarity，即利用用户已经掌握的系统经验。采用系统组件通常能自动获得：

- 熟悉的外观和操作方式
- 深色模式适配
- Dynamic Type 支持
- VoiceOver 等无障碍能力
- 本地化方向支持
- 平台升级后的样式改进

因此，Cohesive Style 并不意味着所有 App 都必须长得完全一样。更合理的做法是：

> 以系统行为作为基础，再通过颜色、排版、图像、内容和适度定制建立自己的品牌风格。

如果为了“差异化”而重做导航栏、开关、菜单、返回手势等基础交互，常常会损害熟悉感和可用性。

---

## 5. SwiftUI 中的实现方法

## 5.1 建立 Design Token

Design Token 用来集中管理设计中的基础变量，例如颜色、间距、圆角和字体。

```swift
enum AppSpacing {
    static let xSmall: CGFloat = 4
    static let small: CGFloat = 8
    static let medium: CGFloat = 16
    static let large: CGFloat = 24
    static let xLarge: CGFloat = 32
}

enum AppRadius {
    static let small: CGFloat = 8
    static let medium: CGFloat = 12
    static let large: CGFloat = 20
}
```

建议使用有限且有规律的间距值，例如：

- 4 pt
- 8 pt
- 16 pt
- 24 pt
- 32 pt

避免每个页面随意出现 13 pt、17 pt、19 pt 和 22 pt 等缺少规则的数值。

## 5.2 使用语义颜色

```swift
extension Color {
    static let appPrimary = Color("AppPrimary")
    static let appBackground = Color("AppBackground")
    static let appSurface = Color("AppSurface")
    static let appError = Color("AppError")
}
```

Asset Catalog 中可以分别配置浅色和深色外观，业务页面只引用颜色的语义名称。

## 5.3 创建统一组件

```swift
struct PrimaryButton: View {
    let title: String
    let action: () -> Void

    var body: some View {
        Button(title, action: action)
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
    }
}
```

在实际项目中，还可以继续统一：

- Loading Button
- Empty State
- Error View
- List Row
- Form Field
- Card
- Toolbar
- Confirmation Dialog

## 5.4 统一导航结构

建议明确不同容器的用途：

- `NavigationStack`：层级式内容浏览
- `TabView`：顶级功能分区
- `sheet`：临时任务或补充流程
- `fullScreenCover`：需要完整注意力的沉浸式流程
- `popover`：与当前内容相关的轻量选择

不要只根据视觉偏好选择容器，而应根据内容层级和用户任务选择。

## 5.5 统一状态设计

每一个重要页面都应考虑以下状态：

- Loading
- Empty
- Content
- Error
- Offline
- Disabled

如果页面 A 使用进度环，页面 B 使用骨架屏，页面 C 在加载时完全空白，用户会感到产品缺乏统一规则。可以根据任务类型定义少量标准状态组件。

---

## 6. Apple App 中的常见表现

### Settings

Settings 使用一致的列表结构、分组方式、导航层级和系统控件。用户进入新的设置页面时，通常不需要重新学习操作方式。

### Apple Music

Apple Music 在播放控制、迷你播放器、专辑图像、页面层级和转场动画上维持统一的视觉及交互语言。

### App Store

App Store 的不同内容分区可能承载不同类型的信息，但排版层级、卡片逻辑、导航结构和品牌表达仍然保持一致。

这些案例说明，Cohesive Style 并不要求每个页面布局完全相同，而是要求不同页面遵循同一套设计规则。

---

## 7. Cohesive Style 不等于所有页面完全一样

需要注意，**一致性不等于机械重复**。

不同任务可能需要不同组件：

- 阅读页面强调内容和排版。
- 编辑页面强调输入和保存状态。
- 播放页面强调媒体控制。
- 数据页面强调图表和筛选。

它们可以采用不同的布局，但仍应共享：

- 相同的视觉语言
- 相同的组件规则
- 相同的导航逻辑
- 相同的反馈机制
- 相同的品牌语气

Cohesive Style 的目标是“有规则的变化”，不是“所有页面复制同一个模板”。

---

## 8. 实际项目检查清单

### 视觉

- [ ] 是否定义了统一的颜色语义？
- [ ] 是否使用稳定的字体层级？
- [ ] 圆角、间距和阴影是否来自统一规则？
- [ ] 图标是否优先使用一致的 SF Symbols 风格？
- [ ] 是否支持浅色和深色模式？

### 组件

- [ ] 主按钮和次按钮是否容易区分？
- [ ] 相同组件在不同页面是否具有相同行为？
- [ ] 是否统一了加载、空状态和错误状态？
- [ ] 组件是否支持禁用和无障碍状态？

### 交互

- [ ] 返回和关闭操作是否符合用户预期？
- [ ] 相同任务是否采用相同流程？
- [ ] 危险操作是否使用一致的确认方式？
- [ ] 操作完成后是否提供清晰反馈？

### 内容

- [ ] 相同概念是否使用相同术语？
- [ ] 按钮文案是否简洁、可执行？
- [ ] 错误信息是否说明问题和解决方式？
- [ ] 整个 App 的语气是否一致？

### 工程

- [ ] 是否建立 Design Token？
- [ ] 是否把高频模式封装为可复用组件？
- [ ] 是否避免在业务页面硬编码颜色、字号和间距？
- [ ] 是否为组件建立 Preview 和测试状态？

---

## 9. 总结

对于 SwiftUI 和现代 iOS App 架构，可以把 Cohesive Style 理解为：

```text
Design System
+ Component Library
+ Interaction Patterns
+ Content Guidelines
= Cohesive App Experience
```

它的核心价值不是单纯追求视觉整齐，而是让用户感受到：

- 界面容易理解
- 操作结果可以预测
- 产品具有统一个性
- 不同功能属于同一个整体

最终目标是让用户感觉自己不是在使用几十个互不相关的页面，而是在使用一个结构清晰、行为稳定并具有明确设计语言的完整产品。

---

## 参考资料

1. Apple Developer, Human Interface Guidelines  
   https://developer.apple.com/design/human-interface-guidelines/

2. Apple Developer, 设计原则  
   https://developer.apple.com/cn/design/human-interface-guidelines/design-principles
