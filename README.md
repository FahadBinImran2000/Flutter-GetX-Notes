<div align="center">
  <img height="60" src="https://img.icons8.com/color/256/flutter.png" alt="Flutter Logo">
  <h1>Flutter GetX Notes</h1>
  <p>Structured conceptual notes on GetX state management in Flutter.</p>
</div>

---

## Table of Contents

### 📘 Beginner
- [Overview](#overview)
- [What is GetX?](#what-is-getx)
- [What is Boilerplate Code?](#what-is-boilerplate-code)
- [Core Principles of GetX](#core-principles-of-getx)
- [Core Responsibilities of GetX](#core-responsibilities-of-getx)

> **Checkpoint:** If you are new to GetX, stop here and understand the basics above before moving forward. The sections below go deeper into how GetX handles state internally.

---

### 📗 Intermediate
- [State Management in GetX](#state-management-in-getx)
    - [What is State?](#1-what-is-state)
    - [Why Traditional State Managers Become Complex](#2-why-traditional-state-managers-become-complex)
    - [How GetX Solves It](#3-how-getx-solves-it)
    - [Reactive System Internals](#4-reactive-system-internals)
    - [Full Reactive Flow](#5-full-reactive-flow)
    - [Obx vs GetBuilder](#6-obx-vs-getbuilder)
- [Simple State Manager](#simple-state-manager)
    - [How It Works](#1-how-it-works)
    - [Key Difference](#2-key-difference)
    - [Why GetBuilder Exists](#3-why-getbuilder-exists)
    - [Real Example: Cart Scenario](#4-real-example-cart-scenario)
    - [Why It Is Called Simple](#5-why-it-is-called-simple)
    - [Performance Advantage](#6-performance-advantage)
    - [When to Use GetBuilder](#7-when-to-use-getbuilder)
    - [Rebuilding Specific Parts with IDs](#8-rebuilding-specific-parts-with-ids)
    - [GetBuilder Does Not Use ChangeNotifier](#9-getbuilder-does-not-use-changenotifier)

---

### 📕 Advanced
- [Deep Dive into Reactive State Manager](#deep-dive-into-reactive-state-manager)
    - [Using Reactive Values in the Controller](#1-using-reactive-values-in-the-controller)
    - [In the View](#2-in-the-view)
    - [Why Do Only Some Widgets Rebuild?](#3-why-do-only-some-widgets-rebuild)
    - [Dependency Tracking: How Getters Work Reactively](#4-dependency-tracking-how-getters-work-reactively)
    - [First Rebuild Behavior](#5-first-rebuild-behavior)
    - [ever(): Listening to Changes](#6-ever-listening-to-changes)
    - [Conditional Updates: addIf()](#7-conditional-updates-addif)
    - [Where .obs Can Be Used](#8-where-obs-can-be-used)
    - [Accessing Values from a Reactive Object](#9-accessing-values-from-a-reactive-object)
    - [Reactive Lists](#10-reactive-lists)
    - [assign() and assignAll()](#11-assign-and-assignall)
    - [Why .value Exists](#12-why-value-exists)
    - [Obx() vs GetX() Widget](#13-obx-vs-getx-widget)
    - [Workers](#14-workers)
    - [Types of Workers](#15-types-of-workers)
    - [Where Workers Should Be Used](#16-where-workers-should-be-used)

---

## Overview

This repository contains conceptual notes about GetX.

The goal is to understand:
- Why GetX exists
- What problems it solves
- How it simplifies Flutter development

---

## What is GetX?

GetX is a Flutter package designed to simplify application development.

It focuses on reducing complexity and boilerplate by providing built-in solutions for:
- State management
- Dependency injection
- Route management

---

## What is Boilerplate Code?

Boilerplate code means code that you must write again and again even though it does almost the same thing every time. It is required setup, not the main logic of your feature.

**Real-life example:** Writing a letter always requires date, greeting, subject, and signature before your actual message. That repeated format is boilerplate.

**Flutter example:**

```dart
class MyScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text("Hello"), // ← your actual content
      ),
    );
  }
}
```

Only `Text("Hello")` is your real content. The rest is required setup. That is boilerplate.

Why developers don't like too much boilerplate:
- It makes code longer
- It slows development
- It increases chances of mistakes
- It makes projects harder to read

GetX is designed to reduce this type of repetitive setup.

---

## Core Principles of GetX

GetX is built around three main principles:

### 1. Productivity

Productivity means you can build apps faster with less effort.

With GetX:
- Less repetitive code
- No complex setup
- Navigate, manage state, and inject dependencies easily

> Less code → Faster development → Less stress.

### 2. Performance

Performance means your app runs smoothly and efficiently.

GetX:
- Updates only the part of the screen that needs updating
- Avoids unnecessary rebuilds
- Is very lightweight (small package size)

> Only update what changes → App stays fast.

### 3. Organization

Organization means keeping your code clean, structured, and easy to manage.

GetX encourages:
- Separating UI from business logic (using Controllers)
- Managing dependencies properly
- Structured navigation

> Logic in controllers, UI in widgets → Better separation of concerns.

---

## Core Responsibilities of GetX

GetX primarily focuses on three core areas of Flutter development:

### 1. State Management

State means data that can change in your app, like a counter value increasing, a user name updating, or a loading spinner appearing.

In traditional Flutter, `setState()` is commonly used, but in larger applications it can lead to tightly coupled and harder-to-maintain code. With GetX:
- No `setState()` needed
- UI updates automatically when data changes
- Code becomes cleaner and shorter

```dart
// Normal Flutter
setState(() { counter++; });

// With GetX
var counter = 0.obs;
counter++; // UI updates automatically
```

> Change the value → UI updates automatically.

### 2. Dependency Injection

Dependency Injection means creating objects like controllers or services once and making them available anywhere in your app.

```dart
// Normal Flutter
var controller = MyController();

// With GetX
Get.put(MyController());
var controller = Get.find<MyController>();
```

> Create once → Use everywhere.

### 3. Route Management

Navigation means moving from one screen to another.

```dart
// Normal Flutter
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondScreen()),
);

// With GetX
Get.to(SecondScreen());
```

With GetX you don't need `BuildContext` or complex navigation setup.

> Go to the next screen with one simple line.

---

> **Checkpoint:** You have completed the foundational concepts. The sections below go deeper into how GetX handles state management internally. Come back when you are comfortable with the basics above.

---

## State Management in GetX

GetX makes state management simple, reactive, and fast. Now let's go deeper into how GetX handles state internally.

### 1. What is State?

State = data that changes in your app.

### 2. Why Traditional State Managers Become Complex

Many Flutter state managers depend on:
- **Streams** (used in BLoC) require `StreamController`, `sink`, `listen`, and manual dispose
- **ChangeNotifier** (used in Provider) requires `notifyListeners()` and widget wrapping
- **Code generators** (used in Riverpod) require running `build_runner` every time you make changes

These are powerful but introduce architectural complexity and additional setup.

GetX does not rely on those traditional approaches. It uses its own lightweight system designed to be fast, low memory, and simple to write.

> Less complexity → Faster and cleaner code.

### 3. How GetX Solves It

#### No Code Generators Needed

```bash
# Other state managers require this every time you change something
flutter pub run build_runner build
```

With GetX there is no code generation, no `build_runner`. Everything works directly.

> Change variable → Done. No extra commands.

#### No Need for BuildContext Everywhere

In normal Flutter you often need `BuildContext` to access controllers, navigate, or show dialogs, sometimes passing it through many layers.

With GetX:
- Controllers don't depend on context
- Business logic stays separate from UI

> Controllers work independently. No tight coupling.

#### Very Precise Updates

In many state managers, a single change rebuilds all listening widgets even if only one thing changed.

With GetX, only the widget that actually changed rebuilds.

> Only what changes gets rebuilt.

#### Rebuilds Only If Value Really Changes

If a Text widget shows `"John"` and you set the value again to `"John"`, GetX will not rebuild because the value didn't actually change.

> No real change → No rebuild.

### 4. Reactive System Internals

GetX powers its state management through four things:

#### GetValue

A lightweight internal container that stores one value and detects when it changes. You don't use it directly. It works behind `.obs` internally.

#### GetStream

GetX's lightweight version of a Stream. Unlike Dart Streams, it requires no `StreamController`, no `sink`, no manual dispose.

> Note: Internally GetX is still stream-based. It just hides that complexity behind `.obs` and `Obx`. The complexity exists but you don't manage it.

#### .obs (Reactive Variables)

`.obs` makes a variable observable, meaning the UI can watch it and react when it changes.

```dart
// Normal variable
var name = "John";

// Reactive variable
var name = "John".obs; // becomes RxString
```

Internally GetX wraps it in a stream, stores the initial value, tracks which widgets use it, and rebuilds only those widgets when the value changes.

**Three ways to create reactive variables:**

```dart
// Method 1
final name = RxString('');
final count = RxInt(0);

// Method 2
final name = Rx<String>('');
final count = Rx<int>(0);

// Method 3 - Recommended
final name = ''.obs;
final count = 0.obs;
final isLogged = false.obs;
```

Always give an initial value. Dart prefers non-nullable values.

#### Obx Widget

`Obx` watches reactive variables and rebuilds only when they change.

```dart
Obx(() => Text("${count.value}"))
```

When `count` changes, only this widget rebuilds. If you assign the same value twice, no rebuild happens because GetX compares old and new values.

### 5. Full Reactive Flow

```dart
var count = 0.obs;  // 1. make variable reactive

count.value++;      // 2. value changes
```

```
count changes
↓
Obx detects change
↓
Only that widget rebuilds
```

### 6. Obx vs GetBuilder

GetX provides two approaches for state management:

#### Obx (Reactive)

Automatically updates UI when a reactive variable changes. No manual call needed.

```dart
final count = 0.obs;

Obx(() => Text("${count.value}"))
```

Use `Obx` when:
- The value changes frequently (counter, cart quantity, loading indicator)
- You want only a small part of UI to update automatically

#### GetBuilder (Simple Updater)

Closer to `setState`. Rebuilds a block of UI only when you manually call `update()`.

```dart
GetBuilder<MyController>(
  builder: (controller) {
    return Text(controller.name);
  },
);
```

```dart
void changeName() {
  name = "John";
  update(); // manually trigger rebuild
}
```

Use `GetBuilder` when:
- You update many things at once (after API call, after login)
- The state changes rarely
- You want simple manual control

#### Decision Rule

- Value changes frequently and independently → `Obx`
- Screen updates as one unit after an action → `GetBuilder`

> Do not make everything `.obs`. Only make what truly needs reactivity.

---

## Deep Dive into Reactive State Manager

The sections below go into advanced reactive concepts. These build on everything covered above.

### 1. Using Reactive Values in the Controller

```dart
final count1 = 0.obs;
final count2 = 0.obs;
int get sum => count1.value + count2.value;
```

What is happening here?
- `count1` → reactive variable
- `count2` → reactive variable
- `sum` → normal Dart getter

**Important:** `sum` is NOT `.obs`. It is just calculated from `count1` and `count2`.

### 2. In the View

```dart
GetX<Controller>(
  builder: (controller) {
    return Text('${controller.count1.value}');
  },
),
```

Each `GetX<Controller>` widget:
- Listens to reactive variables used inside its builder
- Rebuilds only if those variables change

> Think of it like: "I will only rebuild if the reactive variable I use changes."

### 3. Why Do Only Some Widgets Rebuild?

If you run:

```dart
count1.value++;
```

What changes?
- `count1` changes
- `sum` changes (because sum depends on count1)

So:
- count1 widget rebuilds
- sum widget rebuilds
- count2 widget does NOT rebuild

Why? Because `count2` didn't change.

If you run:

```dart
count2.value++;
```

- count2 widget rebuilds
- sum widget rebuilds
- count1 widget does NOT rebuild

This is called **fine-grained dependency tracking**. GetX tracks which reactive variables are used inside each builder.

### 4. Dependency Tracking: How Getters Work Reactively

Even though `sum` is not `.obs`:

```dart
Text('${controller.sum}');
```

GetX sees that `sum` uses `count1.value` and `count2.value`, so it automatically listens to both.

> This is smart tracking. You don't manually subscribe. GetX figures it out.

### 5. First Rebuild Behavior

**Line Definition:** The first time you manually assign `.value` to an Rx variable, GetX triggers listeners once, even if the value didn't change.

#### When It Occurs

It occurs when ALL of these are true:
1. The variable is reactive (`.obs`)
2. You manually assign `.value = something`
3. It is the FIRST assignment after creation

**Example:**

```dart
var isLogged = false.obs;
```

Later:

```dart
isLogged.value = false;   // FIRST assignment
```

Even though: `false → false`

GetX will:
- Trigger `Obx`
- Trigger `ever()`
- Trigger `GetX<Controller>` rebuild

#### When It Does NOT Occur

**During declaration:**
```dart
var isLogged = false.obs;  // does NOT trigger anything
```
No listener is attached yet.

**After the first assignment:**
```dart
isLogged.value = false;  // triggers (first time)
isLogged.value = false;  // ignored (second time)
```
Only the first assignment forces a rebuild. After that, GetX behaves normally.

**If value actually changes, these are normal rebuilds, not first rebuild behavior:**
```dart
isLogged.value = true;   // triggers (real change)
isLogged.value = false;  // triggers (real change)
```

**If you disable it:**
```dart
isLogged.firstRebuild = false;
```

Now even the first assignment will NOT trigger if the value is the same.

#### Where This Usually Happens

**In `onInit()`:**
```dart
@override
void onInit() {
  isLogged.value = checkToken();
}
```

Without first rebuild behavior, startup logic might never run if the value happens to be the same as the default.

Also common in:
- App startup logic
- `ever()` listeners

#### Where It Usually Does NOT Matter

It rarely matters in counter apps, UI toggle buttons, form inputs, or small widgets because values actually change. It mostly matters in login, token checking, app initialization, and boolean flags.

### 6. `ever()`: Listening to Changes

`ever()` is a listener. It means: "Whenever this reactive variable changes, run this function."

```dart
ever(isLogged, fireRoute);
```

Whenever `isLogged` changes, `fireRoute()` runs.

Useful for:
- Navigation
- Showing dialogs
- Triggering side effects

### 7. Conditional Updates: `addIf()`

```dart
list.addIf(item < limit, item);
```

This means: only add the item if the condition is true.

Instead of writing:

```dart
if (item < limit) {
  list.add(item);
}
```

GetX gives a shortcut.

### 8. Where `.obs` Can Be Used

Earlier you learned basic `.obs` on simple variables. It can also be used in two more ways:

#### Method 1: Make Each Property Reactive

```dart
class RxUser {
  final name = "Alex".obs;
  final age = 18.obs;
}
```

Here `name` and `age` are each reactive individually. If `name` changes, only widgets using `name` rebuild. If `age` changes, only widgets using `age` rebuild.

```dart
user.name.value = "John"; // only name widgets rebuild
```

#### Method 2: Make the Whole Object Reactive

Instead of making each field reactive, you can make the entire object reactive.

```dart
class User {
  User({this.name = '', this.age = 0});
  String name;
  int age;
}
```

Controller:

```dart
final user = User(name: "Alex", age: 18).obs;
```

Now the whole `User` object is reactive.

**Updating the object:**

```dart
user.update((user) {
  user.name = "John";
  user.age = 20;
});
```

**Replacing the entire object:**

```dart
user(User(name: "Ali", age: 25));
```

This replaces the old user object with a completely new one.

### 9. Accessing Values from a Reactive Object

In the UI:

```dart
Obx(() => Text("Name ${user.value.name}"))
```

**Why `.value`?** Because `user` is `Rx<User>`, not `User`. So `.value` gives you the real object inside.

**Shortcut way:**

```dart
user().name
```

`user()` is just a shorthand for `user.value`. Both do the same thing.

### 10. Reactive Lists

Lists behave differently from primitive variables.

```dart
final list = <User>[].obs;
```

Now the entire list is reactive. When you add items:

```dart
list.add(User(name: "John"));
```

Widgets using the list rebuild automatically.

**Lists do not need `.value`:**

```dart
controller.list.length        // correct
controller.list.value.length  // not needed
```

**Why lists don't need `.value`:** Dart allows extending collection APIs directly, but primitive types cannot be extended. That is why:

| Type | Needs `.value` |
|------|---------------|
| `int` | yes |
| `String` | yes |
| `bool` | yes |
| `List` | no |
| `Map` | no |

### 11. `assign()` and `assignAll()`

These are special methods for reactive lists.

**`assign()`** clears the list and adds one item:

```dart
list.assign(User(name: "John"));
```

Equivalent to:

```dart
list.clear();
list.add(User(name: "John"));
```

**`assignAll()`** clears the list and adds multiple items:

```dart
list.assignAll(newUsers);
```

Equivalent to:

```dart
list.clear();
list.addAll(newUsers);
```

Useful when loading fresh data from an API.

### 12. Why `.value` Exists

You might wonder why you write:

```dart
count.value++;
```

Instead of just:

```dart
count++;
```

The reason is that GetX avoids code generators. Libraries like MobX remove `.value` using code generation, but GetX philosophy is:

- No code generation
- No extra tools
- Just add the package and start coding

So `.value` is the trade-off for keeping things simple and generator-free.

### 13. `Obx()` vs `GetX()` Widget

Both widgets listen to reactive variables and rebuild when they change. The difference is syntax.

`GetX<Controller>` requires controller typing:

```dart
GetX<Controller>(
  builder: (controller) {
    return Text('${controller.count.value}');
  },
)
```

`Obx` is simpler and requires no typing:

```dart
Obx(() => Text('${controller.count.value}'))
```

Both do the same thing. Most developers prefer `Obx` because it is shorter and cleaner. `GetX<Controller>` is useful when you need explicit access to the controller inside the builder.

### 14. Workers

Workers are automatic listeners for reactive variables. They run a function when something changes.

```dart
ever(count1, (_) => print("count changed"));
```

```
count1 changes
↓
function runs automatically
```

You already saw `ever()` in Section 6. Workers is the broader concept. `ever()` is one type of worker in GetX.

### 15. Types of Workers

#### ever

Runs every time the variable changes.

```dart
ever(count, (_) => print("count changed"));
```

```
count changes → function runs
count changes again → function runs again
```

#### once

Runs only the first time the variable changes.

```dart
once(count, (_) => print("changed once"));
```

```
first change → runs
second change → ignored
```

#### debounce

Waits until the user stops changing the value, then runs once.

```dart
debounce(searchText, (_) => fetchResults(), time: Duration(seconds: 1));
```

Example: search field. User types `J`, `Jo`, `Joh`, `John` quickly. Without debounce, 4 API calls are made. With debounce, GetX waits until the user stops typing and makes only one API call.

```
user stops typing
↓
only one API call
```

#### interval

Limits how often a function runs regardless of how many times the variable changes.

```dart
interval(count, (_) => print("interval"), time: Duration(seconds: 1));
```

Example: user taps a button 100 times per second. Without interval, 100 actions fire. With interval set to 1 second, only one action fires per second.

Useful for:
- Coin reward systems
- Spam protection

### 16. Where Workers Should Be Used

Workers should be created inside `onInit()`:

```dart
@override
void onInit() {
  ever(count, (_) => print("changed"));
}
```

This is the recommended place because `onInit()` runs when the controller is first created, ensuring the worker is registered before any changes happen.

---

## Simple State Manager

GetX provides a second approach to state management that gives you full manual control over when the UI rebuilds.

GetX has two ways to manage state:
1. Reactive → `Obx` (covered in previous sections)
2. Simple → `GetBuilder` (this section)

Simple State Manager means `GetBuilder`. It is a manual update system, similar to `setState()`. Instead of automatic updates like `.obs`, you control when the UI rebuilds.

### 1. How It Works

**Controller:**

```dart
class CartController extends GetxController {
  int totalPrice = 0;
  int itemCount = 0;
 
  void removeItem() {
    itemCount--;
    totalPrice -= 100;
    update(); // manually trigger UI rebuild
  }
}
```

**UI:**

```dart
GetBuilder<CartController>(
  builder: (controller) {
    return Column(
      children: [
        Text("Items: ${controller.itemCount}"),
        Text("Price: ${controller.totalPrice}"),
      ],
    );
  },
)
```

**What happens:**

```
removeItem() called
↓
values change
↓
update() called
↓
whole GetBuilder rebuilds
```

### 2. Key Difference

| Approach | Update Type |
|----------|------------|
| Reactive (`Obx`) | Automatic |
| Simple (`GetBuilder`) | Manual |

That is the core difference.

### 3. Why GetBuilder Exists

Sometimes you don't need reactive variables, automatic tracking, or fine-grained updates. Sometimes you just want:

> "Update everything together when I say so."

`GetBuilder` is the right tool for those cases.

### 4. Real Example: Cart Scenario

Imagine a shopping cart where deleting one product causes multiple things to update at the same time:
- Product list updates
- Total price updates
- Cart badge updates

With `GetBuilder` you handle all of this with one call:

```dart
void removeItem() {
  cart.remove(item);
  calculateTotal();
  update(); // one call updates everything
}
```

```
Multiple data changes
↓
Single update() call
↓
Entire UI block rebuilds
```

No need to manage each variable separately.

### 5. Why It Is Called Simple

Because there is no `.obs`, no `.value`, no `Obx`, and no reactive thinking involved. Just:

```
Change data → call update()
```

### 6. Performance Advantage

`GetBuilder` rebuilds once for multiple changes instead of triggering many small rebuilds.

With `Obx`, each reactive variable triggers its own rebuild:

```
count changes → rebuild
price changes → rebuild
badge changes → rebuild
```

With `GetBuilder`, all changes trigger a single rebuild:

```
all changes
↓
one update()
↓
one rebuild
```

This makes `GetBuilder` more efficient when multiple values change together as one unit.

### 7. When to Use GetBuilder

- Many values change together (API response, cart update)
- You want simple logic without reactive complexity
- State changes rarely

### 8. Rebuilding Specific Parts with IDs

You can target specific parts of the UI to rebuild instead of the entire `GetBuilder` block:

```dart
update(['price']);
```

```dart
GetBuilder<MyController>(
  id: 'price',
  builder: (_) => Text("Price"),
)
```

Only the widget with that matching `id` rebuilds.

### 9. GetBuilder Does Not Use ChangeNotifier

Most Flutter state managers like Provider use `ChangeNotifier` internally. That means they store listeners, notify all of them on any change, and can become heavy in large apps.

`GetBuilder` does not use `ChangeNotifier`. Instead it uses its own ID-based system and only updates widgets linked to a specific ID.

Result:
- Less memory usage
- Faster updates
- More control over what rebuilds

### 10. Multiple State Control

The core strength of `GetBuilder` is that you don't need to track relationships between variables. No dependency tracking, no reactive linking, no observers.

```dart
void removeItem() {
  cart.remove(item);
  totalPrice -= item.price;
  itemCount--;
  update();
}
```

You just change everything and call `update()`. This is what GetX means by updating state "without computational logic" — no extra tracking system is running behind the scenes.

### 11. Ephemeral State

Ephemeral state means temporary UI state that doesn't need a full state manager.

Examples:
- Button color change
- Toggle switch
- Animation state

For these, `setState()` is enough. You don't need GetBuilder or Obx.

```
Small temporary change → use setState
Bigger shared change → use GetBuilder
```

### 12. Forget StatefulWidget

Normally in Flutter, a `StatefulWidget` stores the entire screen state. Even if only one small widget changes, the whole widget tree is stateful.

GetX suggests a different approach:

- Make everything `StatelessWidget`
- Only wrap the parts that change with `GetBuilder`

```
StatelessWidget
↓
GetBuilder → only this part updates
```

Result: less memory usage and a cleaner widget structure.

### 13. initState and dispose Inside GetBuilder

Normally `initState()` and `dispose()` belong to `StatefulWidget`. With `GetBuilder` you can run lifecycle code directly inside it:

```dart
GetBuilder(
  initState: (_) => controller.fetchData(),
  dispose: (_) => controller.cleanUp(),
)
```

However the better and more professional approach is to use the controller lifecycle instead:

```dart
@override
void onInit() {
  fetchData();
}
 
@override
void onClose() {
  cleanUp();
}
```

This keeps lifecycle logic inside the controller where it belongs, not scattered in the UI.

### 14. Controller Lifecycle

Every GetX controller goes through three lifecycle stages:

**`onInit()`** runs when the controller is first created. Use it for setup logic like API calls or initializing data.

```dart
@override
void onInit() {
  fetchData();
}
```

**`onClose()`** runs when the controller is removed. Use it for cleanup like closing streams or cancelling timers.

```dart
@override
void onClose() {
  cleanUp();
}
```

**Deleted** — after `onClose()` the controller is removed from memory automatically. You don't control this manually.

### 15. Automatic Memory Management

GetX automatically creates a controller when it is needed and removes it when it is no longer needed. You do not manually dispose or remove controllers.

```
Screen A → no controller
↓
Screen B → controller created
↓
Screen C → same controller reused
↓
Back to A → controller removed
```

Memory is optimized automatically without any extra code from you.

### 16. Initialize Controller Only Once

When using `GetBuilder`, only pass `init` the first time:

```dart
GetBuilder(
  init: MyController(), // only on first use
  builder: (_) => Text(...),
)
```

If you use `GetBuilder` again elsewhere, do not pass `init` again:

```dart
GetBuilder<MyController>(
  builder: (_) => Text(...), // no init here
)
```

If you reinitialize on every use, you will create multiple controller instances which causes unexpected behavior and memory issues.

### 17. Access Controller Globally

You can create a shortcut inside the controller to access it from anywhere:

```dart
static Controller get to => Get.find();
```

Then use it like this:

```dart
Controller.to.increment();
```

This is just a shortcut for:

```dart
Get.find<Controller>().increment();
```

Both do the same thing. The `to` getter just makes it cleaner to read.