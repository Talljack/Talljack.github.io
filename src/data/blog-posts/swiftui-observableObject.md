---
title: SwiftUI 中的状态管理：@Observable vs ObservableObject 深度解析
publishDate: 19 Jan 2025
description: 在 SwiftUI 的开发中，状态管理一直是一个核心话题。随着 Swift 5.9 的发布，Apple 推出了新的 @Observable 宏，为我们提供了比 ObservableObject 更简洁的状态管理方案。在实际开发中，选择合适的状态管理方案直接影响着应用的性能和可维护性。理解这两种方案的区别和适用场景，能帮助开发者做出更明智的技术选择，避免在后期重构时付出更大的代价。本文将深入探讨这两种方案的区别、使用场景以及最佳实践。
---

![SwiftUI 中的状态管理：@Observable vs ObservableObject 深度解析](/assets/blog/swiftui-observableObject.png)

## 1. 引言

在 SwiftUI 的开发中，状态管理一直是一个核心话题。随着 Swift 5.9 的发布，Apple 推出了新的 @Observable 宏，为我们提供了比 ObservableObject 更简洁的状态管理方案。在实际开发中，选择合适的状态管理方案直接影响着应用的性能和可维护性。理解这两种方案的区别和适用场景，能帮助开发者做出更明智的技术选择，避免在后期重构时付出更大的代价。本文将深入探讨这两种方案的区别、使用场景以及最佳实践。

### 版本兼容性

- @Observable：需要 iOS 17.0+、Xcode 15.0+、Swift 5.9+
- ObservableObject：支持 iOS 13.0+ 的所有版本

## 2. @Observable 的革新

@Observable 是 Swift 5.9 引入的新特性，它使用宏来自动生成所需的样板代码，使状态管理变得更加简单。它的主要优势在于：

1. 简化语法
2. 更好的性能
3. 更细粒度的更新

基本使用示例

```swift
@Observable
class UserProfile {
    var name: String
    var age: Int
    var email: String

    init(name: String, age: Int, email: String) {
        self.name = name
        self.age = age
        self.email = email
    }
}

struct ProfileView: View {
    @State private var profile = UserProfile(name: "张三", age: 30, email: "zhangsan@example.com")

    var body: some View {
        VStack {
            TextField("姓名", text: $profile.name)
            TextField("年龄", value: $profile.age, format: .number)
            TextField("邮箱", text: $profile.email)
        }
    }
}
```

## 3. ObservableObject 的传统方案

`ObservableObject`是 SwiftUI 最初提供的状态管理方案，它基于 `Combine` 框架，通过 `@Published` 属性包装器来标记需要观察的属性。
使用示例

```swift
class UserProfileViewModel: ObservableObject {
    @Published var name: String
    @Published var age: Int
    @Published var email: String

    init(name: String, age: Int, email: String) {
        self.name = name
        self.age = age
        self.email = email
    }
}

struct ProfileView: View {
    @StateObject private var viewModel = UserProfileViewModel(name: "张三", age: 30, email: "zhangsan@example.com")

    var body: some View {
        VStack {
            TextField("姓名", text: $viewModel.name)
            TextField("年龄", value: $viewModel.age, format: .number)
            TextField("邮箱", text: $viewModel.email)
        }
    }
}
```

## 4. 深入对比

### 1. 语法对比

@Observable:

- 只需要一个宏标注
- 所有属性自动成为可观察属性
- 不需要额外的属性包装器

ObservableObject:

- 需要遵循协议
- 需要使用 @Published 标记属性
- 需要使用 @StateObject 或 @ObservedObject 在视图中声明

### 2. 性能对比

@Observable:

- 更细粒度的更新机制
- 减少不必要的视图刷新
- 编译时生成代码，运行时开销更小

ObservableObject:

- 基于 Combine 框架
- 可能导致整个对象的订阅者都收到更新
- 运行时开销相对较大

### 3. 使用场景

@Observable 适用于：

- 简单的状态管理需求
- 需要细粒度更新的场景
- 性能敏感的场景

```swift
@Observable
class CounterState {
    var count = 0
    var lastUpdated = Date()

    func increment() {
        count += 1
        lastUpdated = Date()
    }
}

struct CounterView: View {
    @State private var state = CounterState()

    var body: some View {
        VStack {
            Text("Count: \(state.count)")
            Text("Last Updated: \(state.lastUpdated.formatted())")
            Button("Increment") {
                state.increment()
            }
        }
    }
}
```

ObservableObject 适用于：

- 复杂的数据流管理
- 需要与 Combine 框架深度集成
- 需要更多控制力的场景

```swift
class WeatherViewModel: ObservableObject {
    @Published private(set) var temperature: Double = 0
    @Published private(set) var humidity: Double = 0
    private var cancellables = Set<AnyCancellable>()

    func startMonitoring() {
        // 模拟天气数据更新
        Timer.publish(every: 1.0, on: .main, in: .common)
            .autoconnect()
            .sink { [weak self] _ in
                self?.temperature = Double.random(in: 20...30)
                self?.humidity = Double.random(in: 40...60)
            }
            .store(in: &cancellables)
    }
}
```

### 5. 使用结论

1. `@Observable` 提供了更简洁、高效的状态管理方案，特别适合简单的状态管理需求
2. `ObservableObject` 仍然在复杂场景中有其价值，特别是需要与 `Combine` 框架集成时
3. 在实际应用中，可以根据具体需求组合使用这两种方案
4. 关注点分离和清晰的架构设计比选择使用哪种状态管理方案更重要
   建议在新项目中优先考虑使用 `@Observable`，但不要急于重构现有的 `ObservableObject` 代码，除非有明确的性能优化需求。

## 4. 最佳实践

### 场景一：全局状态管理

在需要在整个应用程序中共享状态时，我们有以下几种推荐方案：

#### 1. Environment 注入方案

**适用于需要在多个视图层级中共享状态，并支持依赖注入和测试的场景**。

##### ObservableObject 方式

```swift
class StorageManager: ObservableObject {
    @Published var totalSize: Int64 = 0
    @Published var isProcessing = false

    func calculateStorage() {
        isProcessing = true
        // 异步处理存储计算
        DispatchQueue.global().async {
            // 模拟计算过程
            Thread.sleep(forTimeInterval: 2)
            DispatchQueue.main.async {
                self.totalSize = 1024 * 1024 * 100 // 100MB
                self.isProcessing = false
            }
        }
    }
}

// App入口注入
@main
struct YourApp: App {
    @StateObject private var storageManager = StorageManager()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(storageManager)
        }
    }
}

// 在任意子视图中使用
struct StorageView: View {
    @EnvironmentObject private var storageManager: StorageManager

    var body: some View {
        VStack {
            if storageManager.isProcessing {
                ProgressView()
            } else {
                Text("存储空间：\(storageManager.totalSize) bytes")
            }
            Button("刷新存储状态") {
                storageManager.calculateStorage()
            }
        }
    }
}
```

##### @Observable 方式

```swift
@Observable class StorageManager {
    var totalSize: Int64 = 0
    var isProcessing = false

    func calculateStorage() {
        isProcessing = true
        Task {
            // 异步处理存储计算
            try? await Task.sleep(for: .seconds(2))
            await MainActor.run {
                totalSize = 1024 * 1024 * 100 // 100MB
                isProcessing = false
            }
        }
    }
}

// App入口注入
@main
struct YourApp: App {
    @State private var storageManager = StorageManager()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(storageManager)
        }
    }
}

// 在任意子视图中使用
struct StorageView: View {
    @Environment(StorageManager.self) private var storageManager

    var body: some View {
        VStack {
            if storageManager.isProcessing {
                ProgressView()
            } else {
                Text("存储空间：\(storageManager.totalSize) bytes")
            }
            Button("刷新存储状态") {
                storageManager.calculateStorage()
            }
        }
    }
}
```

#### 2. 单例模式方案

**适用于确实需要全局唯一实例，且状态需要在整个应用生命周期内保持的场景**。

##### ObservableObject 方式

```swift
class UserManager: ObservableObject {
    static let shared = UserManager()
    private init() {} // 添加私有初始化器

    @Published var isLoggedIn = false
    @Published var currentUser: User?

    struct User {
        var id: String
        var name: String
    }

    func login(username: String) {
        currentUser = User(id: UUID().uuidString, name: username)
        isLoggedIn = true
    }

    func logout() {
        currentUser = nil
        isLoggedIn = false
    }
}

// 使用示例
struct ProfileView: View {
    @StateObject private var userManager = UserManager.shared

    var body: some View {
        Group {
            if userManager.isLoggedIn {
                Text("欢迎回来，\(userManager.currentUser?.name ?? "")")
                Button("退出登录") {
                    userManager.logout()
                }
            } else {
                Button("登录") {
                    userManager.login(username: "张三")
                }
            }
        }
    }
}
```

##### @Observable 方式

```swift
@Observable class UserManager {
    static let shared = UserManager()
    private init() {} // 添加私有初始化器

    var isLoggedIn = false
    var currentUser: User?

    struct User {
        var id: String
        var name: String
    }

    func login(username: String) {
        currentUser = User(id: UUID().uuidString, name: username)
        isLoggedIn = true
    }

    func logout() {
        currentUser = nil
        isLoggedIn = false
    }
}

// 使用示例
struct ProfileView: View {
    // 方式1：直接使用单例
    let userManager = UserManager.shared  // 不需要 @State
    // 或者方式2：如果需要在视图更新时保持状态
    @State private var userManager = UserManager.shared

    var body: some View {
        Group {
            if userManager.isLoggedIn {
                Text("欢迎回来，\(userManager.currentUser?.name ?? "")")
                Button("退出登录") {
                    userManager.logout()
                }
            } else {
                Button("登录") {
                    userManager.login(username: "张三")
                }
            }
        }
    }
}
```

### 场景二：局部状态管理

适用于组件内部或有限范围内的状态管理。

#### 1. 父子组件状态传递

适用于需要在父子组件间共享状态的场景。

```swift
// 共享的状态模型
@Observable class FormState {
    var username = ""
    var email = ""
    var isValid: Bool {
        !username.isEmpty && !email.isEmpty && email.contains("@")
    }
}

// 父组件
struct FormContainer: View {
    @State private var formState = FormState()

    var body: some View {
        VStack {
            FormInputs(formState: formState)
            FormActions(formState: formState)
        }
    }
}

// 子组件：表单输入
struct FormInputs: View {
    let formState: FormState

    var body: some View {
        VStack {
            TextField("用户名", text: $formState.username)
            TextField("邮箱", text: $formState.email)
        }
        .textFieldStyle(RoundedBorderTextFieldStyle())
        .padding()
    }
}

// 子组件：表单操作
struct FormActions: View {
    let formState: FormState

    var body: some View {
        Button("提交") {
            // 处理表单提交
            print("提交表单：\(formState.username), \(formState.email)")
        }
        .disabled(!formState.isValid)
    }
}
```

#### 2. 视图组件的本地状态

适用于组件内部的状态管理，不需要与外部共享的场景。

```swift
struct ExpandableCard: View {
    @State private var isExpanded = false
    let title: String
    let content: String

    var body: some View {
        VStack {
            Button {
                withAnimation {
                    isExpanded.toggle()
                }
            } label: {
                HStack {
                    Text(title)
                    Spacer()
                    Image(systemName: isExpanded ? "chevron.up" : "chevron.down")
                }
            }

            if isExpanded {
                Text(content)
                    .padding(.top)
                    .transition(.opacity)
            }
        }
        .padding()
        .background(Color.gray.opacity(0.1))
        .cornerRadius(8)
    }
}

// 使用示例
struct ContentView: View {
    var body: some View {
        VStack(spacing: 20) {
            ExpandableCard(
                title: "SwiftUI 介绍",
                content: "SwiftUI 是一个创新的、简单的构建用户界面的框架..."
            )
            ExpandableCard(
                title: "状态管理",
                content: "在 SwiftUI 中，我们有多种方式管理状态..."
            )
        }
        .padding()
    }
}
```

### 场景三：混合使用策略

在实际项目中，我们经常需要组合使用不同的状态管理方案。以下是一个综合示例：

```swift
// 全局状态管理（使用 @Observable）
@Observable class AppState {
    static let shared = AppState()
    private init() {}

    var theme: Theme = .light
    var language: Language = .chinese

    enum Theme {
        case light, dark
    }

    enum Language {
        case chinese, english
    }
}

// 业务模块状态管理（使用 ObservableObject）
class ProductViewModel: ObservableObject {
    @Published var products: [Product] = []
    @Published var isLoading = false

    struct Product: Identifiable {
        let id: UUID
        var name: String
        var price: Double
    }

    func fetchProducts() {
        isLoading = true
        // 模拟网络请求
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            self.products = [
                Product(id: UUID(), name: "产品1", price: 99.9),
                Product(id: UUID(), name: "产品2", price: 199.9)
            ]
            self.isLoading = false
        }
    }
}

// 视图实现
struct ProductListView: View {
    let appState = AppState.shared
    @StateObject private var viewModel = ProductViewModel()
    @State private var selectedProduct: ProductViewModel.Product?

    var body: some View {
        NavigationView {
            Group {
                if viewModel.isLoading {
                    ProgressView()
                } else {
                    List(viewModel.products) { product in
                        ProductRow(product: product)
                            .onTapGesture {
                                selectedProduct = product
                            }
                    }
                }
            }
            .navigationTitle(appState.language == .chinese ? "产品列表" : "Products")
            .sheet(item: $selectedProduct) { product in
                ProductDetailView(product: product)
            }
        }
        .onAppear {
            viewModel.fetchProducts()
        }
    }
}

struct ProductRow: View {
    let product: ProductViewModel.Product

    var body: some View {
        HStack {
            Text(product.name)
            Spacer()
            Text("¥\(product.price, specifier: "%.2f")")
        }
    }
}
```

## 6. 常见陷阱和注意事项

1. 内存管理

- @Observable 中的循环引用问题
- ObservableObject 中的 Combine 订阅清理

2. 线程安全

- 属性更新的线程考虑
- 异步操作的处理方式

3. 调试技巧

- 如何追踪状态变化
- 常用的调试工具和方法

## 7. 迁移指南

具体的迁移指南可查看[ Apple developer 官方文档](https://developer.apple.com/documentation/swiftui/migrating-from-the-observable-object-protocol-to-the-observable-macro)
