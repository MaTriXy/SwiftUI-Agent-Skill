# Latest SwiftUI APIs Reference

> Based on a comparison of Apple's documentation using the Sosumi MCP, we found the latest recommended APIs to use.

## Table of Contents
- [Always Use (iOS 15+)](#always-use-ios-15)
- [When Targeting iOS 16+](#when-targeting-ios-16)
- [When Targeting iOS 17+](#when-targeting-ios-17)
- [When Targeting iOS 18+](#when-targeting-ios-18)
- [When Targeting iOS 26+](#when-targeting-ios-26)

---

## Always Use (iOS 15+)

These APIs have been deprecated long enough that there is no reason to use the old variants.

### Simple Renames (drop-in replacements)

These are direct replacements with no API shape change:

- **`navigationTitle(_:)`** instead of `navigationBarTitle(_:)`
- **`toolbar { ToolbarItem(...) }`** instead of `navigationBarItems(...)`
- **`toolbarVisibility(.hidden, for: .navigationBar)`** instead of `navigationBarHidden(_:)`
- **`statusBarHidden(_:)`** instead of `statusBar(hidden:)`
- **`ignoresSafeArea(_:edges:)`** instead of `edgesIgnoringSafeArea(_:)`
- **`preferredColorScheme(_:)`** instead of `colorScheme(_:)`
- **`foregroundStyle()`** instead of `foregroundColor()`
- **`clipShape(.rect(cornerRadius:))`** instead of `cornerRadius()`
- **`textInputAutocapitalization(_:)`** instead of `autocapitalization(_:)` (note: `.never` replaces `.none`)
- **`animation(_:value:)`** instead of `animation(_:)` without value (back-deploys to iOS 13+)

### Presentation

**Always use `confirmationDialog(...)` instead of `actionSheet(...)`.**

```swift
// Modern
.confirmationDialog("Choose Option", isPresented: $showOptions) {
    Button("Option A") { selectA() }
    Button("Cancel", role: .cancel) { }
} message: {
    Text("Select your preferred option.")
}

// Deprecated — ActionSheet(title:buttons:) pattern
```

**Always use `alert(_:isPresented:actions:message:)` instead of `alert(isPresented:content:)`.**

```swift
// Modern
.alert("Delete Item", isPresented: $showAlert) {
    Button("Delete", role: .destructive) { deleteItem() }
    Button("Cancel", role: .cancel) { }
} message: {
    Text("This action cannot be undone.")
}

// Deprecated — Alert(title:message:buttons:) pattern
```

### Text Input

**Always use `onSubmit(of:_:)` and `focused(_:equals:)` instead of `TextField` `onEditingChanged`/`onCommit` callbacks.**

```swift
@FocusState private var isFocused: Bool

TextField("Search", text: $query)
    .focused($isFocused)
    .onSubmit { performSearch() }
```

### Accessibility

**Always use dedicated accessibility modifiers instead of the generic `accessibility(...)` variants.** Use `.accessibilityLabel()`, `.accessibilityValue()`, `.accessibilityHint()`, `.accessibilityAddTraits()`, `.accessibilityHidden()` instead of `.accessibility(label:)`, `.accessibility(value:)`, etc.

### Custom Environment / Container Values

**Always use the `@Entry` macro instead of manual `EnvironmentKey` conformance.** The `@Entry` macro was introduced in Xcode 16 and back-deploys to all OS versions.

```swift
// Modern — one line replaces ~10 lines of EnvironmentKey boilerplate
extension EnvironmentValues {
    @Entry var myCustomValue: String = "Default value"
}
```

### Styling

**Always use `Button` instead of `onTapGesture()` unless you need tap location or count.**

```swift
Button("Tap me") { performAction() }

// Use onTapGesture only when you need location or count
Image("photo")
    .onTapGesture(count: 2) { handleDoubleTap() }
```

---

## When Targeting iOS 16+

### Navigation

**Use `NavigationStack` (or `NavigationSplitView`) instead of `NavigationView`.** The API shape changed significantly -- value-based `NavigationLink` with `navigationDestination(for:)` replaces destination-based links.

```swift
NavigationStack {
    List(items) { item in
        NavigationLink(value: item) {
            Text(item.name)
        }
    }
    .navigationDestination(for: Item.self) { item in
        DetailView(item: item)
    }
}
```

### Simple Renames

- **`tint(_:)`** instead of `accentColor(_:)`
- **`autocorrectionDisabled(_:)`** instead of `disableAutocorrection(_:)`

### Clipboard

**Prefer `PasteButton` for user-initiated paste UI** to avoid paste prompts. It handles permissions automatically. Use `UIPasteboard` only when you need programmatic or non-`Transferable` clipboard access (triggers the paste permission prompt).

```swift
PasteButton(payloadType: String.self) { strings in
    pastedText = strings.first ?? ""
}
```

---

## When Targeting iOS 17+

### State Management

**Prefer `@Observable` over `ObservableObject` for new code.**

```swift
// Modern (iOS 17+)
@Observable
class UserProfile {
    var name: String = ""
    var email: String = ""
}

struct ProfileView: View {
    @State private var profile = UserProfile()
    var body: some View {
        TextField("Name", text: $profile.name)
    }
}

// Legacy
class UserProfile: ObservableObject {
    @Published var name: String = ""
    @Published var email: String = ""
}

struct ProfileView: View {
    @StateObject private var profile = UserProfile()
    var body: some View {
        TextField("Name", text: $profile.name)
    }
}
```

### Events

**Use `onChange(of:initial:_:)` or `onChange(of:) { }` instead of `onChange(of:perform:)`.**

The deprecated variant passes only the new value. The modern variants provide either both old and new values, or a no-parameter closure.

```swift
// Modern — no-parameter closure (most common)
.onChange(of: playState) {
    model.playStateDidChange(state: playState)
}

// Modern — old and new values
.onChange(of: selectedTab) { oldTab, newTab in
    analytics.trackTabChange(from: oldTab, to: newTab)
}

// Modern — with initial trigger
.onChange(of: searchText, initial: true) {
    performSearch()
}

// Deprecated
.onChange(of: playState) { newValue in
    model.playStateDidChange(state: newValue)
}
```

### Gestures

- **`MagnifyGesture`** instead of `MagnificationGesture` (access magnitude via `value.magnification`)
- **`RotateGesture`** instead of `RotationGesture` (access angle via `value.rotation`)

### Layout

**Consider `containerRelativeFrame()` or `visualEffect()` as alternatives to `GeometryReader` for sizing and position-based effects.** `GeometryReader` is not deprecated and remains necessary for many measurement-based layouts.

```swift
// Modern — containerRelativeFrame
Image("hero")
    .resizable()
    .containerRelativeFrame(.horizontal) { length, axis in
        length * 0.8
    }

// Modern — visualEffect for position-based effects
Text("Parallax")
    .visualEffect { content, geometry in
        content.offset(y: geometry.frame(in: .global).minY * 0.5)
    }

// Legacy — only use if necessary
GeometryReader { geometry in
    Image("hero")
        .frame(width: geometry.size.width * 0.8)
}
```

**Consider `onGeometryChange(for:of:action:)` when you need to react to geometry changes of a specific view.** It reports changes for the view you attach it to, and is useful for driving state/effects from geometry updates. `GeometryReader` is still useful when layout itself depends on geometry.

```swift
Text("Content")
    .onGeometryChange(for: CGSize.self) { $0.size } action: { newSize in
        // React to size changes
    }
```

**Use `coordinateSpace(.named(_:))` instead of `coordinateSpace(name:)`.**

---

## When Targeting iOS 18+

### Tabs

**Use the `Tab` API instead of `tabItem(_:)`.**

```swift
TabView {
    Tab("Home", systemImage: "house") { HomeView() }
    Tab("Search", systemImage: "magnifyingglass") { SearchView() }
    Tab("Profile", systemImage: "person") { ProfileView() }
}
```

When using `Tab(role:)`, all tabs must use the `Tab` syntax. Mixing `Tab(role:)` with `.tabItem()` causes compilation errors.

### Previews

**Use `@Previewable` for dynamic properties in previews.**

```swift
// Modern (iOS 18+)
#Preview {
    @Previewable @State var isOn = false
    Toggle("Setting", isOn: $isOn)
}
```

---

## When Targeting iOS 26+

For Liquid Glass APIs (`glassEffect`, `GlassEffectContainer`, glass button styles), see [liquid-glass.md](liquid-glass.md).

### Scroll Edge Effects

**Use `scrollEdgeEffectStyle(_:for:)` to configure scroll edge behavior.**

```swift
ScrollView {
    // content
}
.scrollEdgeEffectStyle(.soft, for: .top)
```

### Background Extension

**Use `backgroundExtensionEffect()` for edge-extending blurred backgrounds.**

Views behind a Liquid Glass sidebar can appear clipped. This modifier mirrors and blurs content outside the safe area so artwork remains visible.

```swift
Image("hero")
    .backgroundExtensionEffect()
```

> Source: "Build a SwiftUI app with the new design" (WWDC25, session 323)

### Tab Bar

**Use `tabBarMinimizeBehavior(_:)` to control tab bar minimization on scroll.**

```swift
TabView {
    // tabs
}
.tabBarMinimizeBehavior(.onScrollDown)
```

**Use `tabViewBottomAccessory` for persistent controls above the tab bar.** Read `tabViewBottomAccessoryPlacement` from the environment to adapt content when the accessory collapses into the tab bar area.

```swift
TabView {
    // tabs
}
.tabViewBottomAccessory {
    NowPlayingBar()
}
```

**Use `Tab(role: .search)` for a dedicated search tab.** The tab separates from the rest and morphs into a search field when selected.

```swift
TabView {
    Tab("Home", systemImage: "house") { HomeView() }
    Tab("Profile", systemImage: "person") { ProfileView() }
    Tab(role: .search) { SearchResultsView() }
}
```

> Source: "What's new in SwiftUI" (WWDC25, session 256) and "Build a SwiftUI app with the new design" (WWDC25, session 323)

### Toolbars

**Use `ToolbarSpacer` to control grouping of toolbar items.** Fixed spacers visually separate related groups; flexible spacers push items apart.

```swift
.toolbar {
    ToolbarItem(placement: .topBarTrailing) {
        Button("Up", systemImage: "chevron.up") { }
    }
    ToolbarItem(placement: .topBarTrailing) {
        Button("Down", systemImage: "chevron.down") { }
    }
    ToolbarSpacer(.fixed)
    ToolbarItem(placement: .topBarTrailing) {
        Button("Settings", systemImage: "gear") { }
    }
}
```

**Use `sharedBackgroundVisibility(.hidden)` to remove the glass group background from an individual toolbar item.**

```swift
ToolbarItem(placement: .topBarTrailing) {
    Image(systemName: "person.circle.fill")
        .sharedBackgroundVisibility(.hidden)
}
```

**Use `badge(_:)` on toolbar item content to display an indicator.**

```swift
ToolbarItem(placement: .topBarTrailing) {
    Button("Notifications", systemImage: "bell") { }
        .badge(unreadCount)
}
```

> Source: "Build a SwiftUI app with the new design" (WWDC25, session 323)

### Search

**Use `searchToolbarBehavior(.minimizable)` to opt into a minimized search button.** The system may automatically minimize search into a toolbar button depending on available space. Use this modifier to explicitly opt in.

```swift
NavigationStack {
    ContentView()
        .searchable(text: $query)
        .searchToolbarBehavior(.minimizable)
}
```

> Source: "Build a SwiftUI app with the new design" (WWDC25, session 323)

### Animations

**Use `@Animatable` macro instead of manual `animatableData` declarations.** The macro auto-synthesizes `animatableData` from all animatable properties. Use `@AnimatableIgnored` to exclude specific properties.

```swift
@Animatable
struct Wedge: Shape {
    var startAngle: Angle
    var endAngle: Angle
    @AnimatableIgnored var drawClockwise: Bool

    func path(in rect: CGRect) -> Path { /* ... */ }
}
```

> Source: "What's new in SwiftUI" (WWDC25, session 256)

### Presentations

**Use `navigationZoomTransition` to morph sheets out of their source view.** Toolbar items and buttons can serve as the transition source.

```swift
.toolbar {
    ToolbarItem {
        Button("Add", systemImage: "plus") { showSheet = true }
            .navigationTransitionSource(id: "addSheet", namespace: namespace)
    }
}
.sheet(isPresented: $showSheet) {
    AddItemView()
        .navigationTransitionDestination(id: "addSheet", namespace: namespace)
}
```

> Source: "Build a SwiftUI app with the new design" (WWDC25, session 323)

### Controls

**Use `controlSize(.extraLarge)` for extra-large prominent action buttons.**

```swift
Button("Get Started") { }
    .buttonStyle(.borderedProminent)
    .controlSize(.extraLarge)
```

**Use `concentric` corner style for buttons that match their container's corners.**

```swift
Button("Confirm") { }
    .clipShape(.rect(cornerRadius: 12, style: .concentric))
```

**Sliders now support tick marks and a neutral value.**

```swift
Slider(value: $speed, in: 0.5...2.0, step: 0.25) {
    Text("Speed")
} ticks: {
    SliderTick(value: 0.6)
    SliderTick(value: 0.9)
}
.sliderNeutralValue(1.0)
```

> Source: "Build a SwiftUI app with the new design" (WWDC25, session 323)

### Rich Text

**Use `TextEditor` with an `AttributedString` binding for rich text editing.** Supports bold, italic, underline, strikethrough, custom fonts, foreground/background colors, paragraph styles, and Genmoji.

```swift
@State private var text: AttributedString = "Hello, world!"

var body: some View {
    TextEditor(text: $text)
}
```

> Source: "Cook up a rich text experience in SwiftUI with AttributedString" (WWDC25, session 280)

### Web Content

**Use `WebView` to display web content.** For richer interaction, create a `WebPage` observable model.

```swift
// Simple URL display
WebView(url: URL(string: "https://example.com")!)

// With observable model
@State private var page = WebPage()

WebView(page)
    .onAppear { page.load(URLRequest(url: myURL)) }
    .navigationTitle(page.title ?? "")
```

> Source: "Meet WebKit for SwiftUI" (WWDC25, session 231)

### Drag and Drop

**Use `dragContainer` for multi-item drag operations.** Combine with `DragConfiguration` for custom drag behavior and `onDragSessionUpdated` to observe events.

```swift
PhotoGrid(photos: photos)
    .dragContainer(for: Photo.self) { selection in
        return selection.map { $0.transferable }
    }
    .onDragSessionUpdated { session in
        if session.phase == .endedWithDelete {
            deleteSelectedPhotos()
        }
    }
```

> Source: "What's new in SwiftUI" (WWDC25, session 256)

### Scene Bridging

**UIKit and AppKit lifecycle apps can now request SwiftUI scenes.** This enables using SwiftUI-only scene types like `MenuBarExtra` and `ImmersiveSpace` from imperative lifecycle apps via `UIApplication.shared.activateSceneSession(for:errorHandler:)`.

> Source: "What's new in SwiftUI" (WWDC25, session 256)

---

## Quick Lookup Table

| Deprecated | Recommended | Since |
|-----------|-------------|-------|
| `navigationBarTitle(_:)` | `navigationTitle(_:)` | iOS 15+ |
| `navigationBarItems(...)` | `toolbar { ToolbarItem(...) }` | iOS 15+ |
| `navigationBarHidden(_:)` | `toolbarVisibility(.hidden, for: .navigationBar)` | iOS 15+ |
| `statusBar(hidden:)` | `statusBarHidden(_:)` | iOS 15+ |
| `edgesIgnoringSafeArea(_:)` | `ignoresSafeArea(_:edges:)` | iOS 15+ |
| `colorScheme(_:)` | `preferredColorScheme(_:)` | iOS 15+ |
| `foregroundColor(_:)` | `foregroundStyle()` | iOS 15+ |
| `cornerRadius(_:)` | `clipShape(.rect(cornerRadius:))` | iOS 15+ |
| `actionSheet(...)` | `confirmationDialog(...)` | iOS 15+ |
| `alert(isPresented:content:)` | `alert(_:isPresented:actions:message:)` | iOS 15+ |
| `autocapitalization(_:)` | `textInputAutocapitalization(_:)` | iOS 15+ |
| `accessibility(label:)` etc. | `accessibilityLabel()` etc. | iOS 15+ |
| `TextField` `onCommit`/`onEditingChanged` | `onSubmit` + `focused` | iOS 15+ |
| `animation(_:)` (no value) | `animation(_:value:)` | Back-deploys (iOS 13+) |
| Manual `EnvironmentKey` | `@Entry` macro | Back-deploys (Xcode 16+) |
| `NavigationView` | `NavigationStack` / `NavigationSplitView` | iOS 16+ |
| `accentColor(_:)` | `tint(_:)` | iOS 16+ |
| `disableAutocorrection(_:)` | `autocorrectionDisabled(_:)` | iOS 16+ |
| `UIPasteboard.general` | `PasteButton` | iOS 16+ |
| `onChange(of:perform:)` | `onChange(of:) { }` or `onChange(of:) { old, new in }` | iOS 17+ |
| `MagnificationGesture` | `MagnifyGesture` | iOS 17+ |
| `RotationGesture` | `RotateGesture` | iOS 17+ |
| `coordinateSpace(name:)` | `coordinateSpace(.named(...))` | iOS 17+ |
| `ObservableObject` | `@Observable` | iOS 17+ |
| `tabItem(_:)` | `Tab` API | iOS 18+ |
| Manual `animatableData` | `@Animatable` macro | iOS 26+ |
| `presentationBackground(_:)` on sheets | Default Liquid Glass sheet material | iOS 26+ |
| Custom toolbar background hacks | `scrollEdgeEffectStyle(_:for:)` | iOS 26+ |
