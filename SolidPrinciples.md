# Solid Principles

The SOLID principles are composed of five simple but essential principles intended to make Software Design easily maintainable and scalable.

1. **S** - Single Responsibility Principle
2. **O** - Open/Closed Principle
3. **L** - Liskov's Substitution Principle
4. **I** - Interface Segregation Principle
5. **D** - Dependency Inversion Principle

- SOLID principles are intertwined and interdependent.
- SOLID principles are most effective when they are combined together.
- It is most important to get a wholesome view of all SOLID principles.

# Single Responsibility Principle
Single Responsibility Principle states that **A software component should one and only one responsibility**. Based on the used programming paradigm this component can be a function or a class or a module.

Since we will be using Swift Object Oriented Paradigm, in our case a swift class will be a software component. But if we follow Protocol Oriented Programming in Swift then every protocol will be a software component. 

So in our case, every class should have only one responsibility.

![Single Responsibility Principle](SingleResponsibilityPrinciple.png)

If we look at the above pictures
- on the left we have Swiss Army knife
- on the right we have normal kitchen knife

Although swiss army knife is very useful in day to day life, it violates Single responsibility principle. On the other hand, the kitchen knife only have one responsibility, which is to cut some items.

Above example was real life example of Single Responsibility Principle. To apply this principle in Software Design, first we need to understand two important topic

- Cohesion
- Coupling

## Cohesion
Cohesion is the core technical goal of the Single Responsibility Principle(SRP). If a class have multiple responsibility, cohesion refers to how closely related the responsibilities of a class are.

But if Single Responsibility Principle encourages to have only one responsibility per class (in OOP), then why we are measuring coherence between multiple responsibilities.

Single responsibility does not mean single method. Single responsibility is about **one reason to change**. A class with Single responsibility can have multiple properties and methods that work together toward that responsibility. And cohesion measurement helps us to identify responsibility count of a class.

If a class has a single responsibility, then cohesion of that class will be higher. But if the class has more than one responsibility, cohesion of that class will be lower.

To achieve single responsibility, our goal should be to achieve high cohesion of that class.

Let's see some coding example to understand the topic better

### Example 1

```swift
enum RotateDirection {
    case up
    case down
    case towardLeft
    case towardRight
}

// This class has dual responsibility
// cube measurement
// cube rendering
// that's why cohesion of this class will be low
class Cube {
    var side: Int = 5
    
    func calculateArea() -> Int {
        return 6 * side * side
    }
    
    func calculateVolume() -> Int {
        return side * side * side
    }
    
    func draw() {
        // renders a 3D object with the cube properties
    }
    
    func rotate(angle: Double, direction: RotateDirection) {
        // Rotate the 3D cube object according to the given parameters
    }
}
```
let's refactor the codes to increase cohesion

```swift
enum RotateDirection {
    case up
    case down
    case towardLeft
    case towardRight
}

// Now, this class has single responsibility
// cube measurement
// that's why cohesion of this class will be high
class Cube {
    var side: Int = 5
    
    func calculateArea() -> Int {
        return 6 * side * side
    }
    
    func calculateVolume() -> Int {
        return side * side * side
    }
}

// Now, this class has single responsibility
// cube rendering
// that's why cohesion of this class will be high
class CubeRenderer {
    func draw() {
        // renders a 3D object with the cube properties
    }

    func rotate(angle: Double, direction: RotateDirection) {
        // Rotate the 3D cube object according to the given parameters
    }
}
```

### Example 2
```swift
// Responsibility 1: order math (subtotal, tax, total)
// Responsibility 2: sending receipt (IO / side effects) 
// low cohesion
final class Order {
    var items: [Double] = [19.0, 5.0, 2.0]
    var taxRate: Double = 0.1
    var customerEmail: String = "user@example.com"
    
    func subtotal() -> Double { 
        return items.reduce(0, +) 
    }
    
    
    func total() -> Double { 
        let s = subtotal(); 
        return s + s * taxRate
    }
    
    func sendReceipt() {
        let message = "Thank you! Your total is \(total())"
        // imagine emailing this message to customerEmail
        print("📧 emailing to \(customerEmail): \(message)")
    }
}
```

Again, let's refactor the above code to achieve high cohesion and single responsibility

```swift
// Single responsibility: order math
// high cohesion
struct Order {
    let items: [Double]
    let taxRate: Double
    
    func subtotal() -> Double { 
        return items.reduce(0, +)
    }
    
    func total() -> Double {
        let s = subtotal()
        return s + s * taxRate
    }
}

// Single responsibility: sending receipts
// high cohesion
protocol ReceiptSender {
    func send(to email: String, body: String)
}

// Single responsibility: sending receipts
// high cohesion
struct ConsoleReceiptSender: ReceiptSender {
    func send(to email: String, body: String) {
        // placeholder for real email/notification
        print("📧 to \(email): \(body)")
    }
}

// Orchestrator for the *use case* "issue receipt" (still one reason: composing the two)
struct ReceiptService {
    let sender: ReceiptSender
    
    func issueReceipt(for order: Order, to email: String) {
        let body = "Thank you! Your total is \(order.total())"
        sender.send(to: email, body: body)
    }
}

// Usage
let order = Order(items: [19.0, 5.0, 2.0], taxRate: 0.1)
let service = ReceiptService(sender: ConsoleReceiptSender())
service.issueReceipt(for: order, to: "user@example.com")
```

### Notes on Coupling (and SRP)
- Cohesion is how closely the methods and data inside a class work toward the same
    - High Cohesion: one clear job
    - Low Cohesion: mixed, unrelated jobs inside one class.
- Symptoms of low cohesion
    - Component in this case class has multiple reasons to change
    - Tests for one feature fail when you change an unrelated one.
- Increase cohesion by:
    - Split by reason to change
    - Group behavior with the data it uses
    - Use protocols to compose cohesive collaborators without mixing concerns
    - Keep pure logic separate from IO/side-effects.
      - Pure Logic
          - Arithmetic & numeric algorithms
          - Parsing strings → values
          - Validation rules
          - Data transformations
      - IO/side-effects
          - Database operations
          - Network calls
          - File system operations
          - UI updates (SwiftUI / UIKit)

## Coupling
Coupling is defined as the level of inter dependency between various software components.

When modules have a high coupling, they are intimately linked, and modifications to one module may have an impact on other modules.  Low coupling indicates that modules are self-contained and that modifications to one module hardly affect the others.

To achieve single responsibility, our goal should be to achieve low coupling of a software component (in this case class).

### Coupling Example (Swift)
Below is a small example that first shows tight coupling, then a refactor with low coupling that also aligns with SRP.

```swift
// Tightly coupled: InvoiceService constructs and controls concrete collaborators.
// Changes to email/PDF behavior require edits here.

final class EmailClient {
    func send(to address: String, subject: String, body: String) {
        print("Email to \(address): \(subject) — \(body)")
    }
}

final class PDFRenderer {
    func render(invoiceNumber: String, amount: Double) -> Data {
        // pretend to generate a PDF
        return Data("Invoice #\(invoiceNumber): \(amount)".utf8)
    }
}

final class InvoiceServiceTight {
    // Direct construction = tight coupling
    private let mailer = EmailClient()
    private let renderer = PDFRenderer()

    func issueInvoice(number: String, amount: Double, email: String) {
        let pdf = renderer.render(invoiceNumber: number, amount: amount)
        // … store pdf somewhere …
        mailer.send(
            to: email,
            subject: "Your invoice #\(number)",
            body: "Amount due: \(amount). PDF bytes: \(pdf.count)"
        )
    }
}
```

Problems: `InvoiceServiceTight` mixes responsibilities (orchestration + choosing concrete implementations) and is hard to test or change. Any change to email or rendering requires editing this class (violates Single Responsibility Principle(SRP) and increases coupling).

Refactor to reduce coupling and align with Single Responsibility Principle(SRP):

```swift
// Abstractions decouple the service from concrete implementations
protocol InvoiceRenderer {
    func render(invoiceNumber: String, amount: Double) -> Data
}

protocol Notifier {
    func send(to address: String, subject: String, body: String)
}

// Concrete implementations can vary independently
struct PDFInvoiceRenderer: InvoiceRenderer {
    func render(invoiceNumber: String, amount: Double) -> Data {
        Data("Invoice #\(invoiceNumber): \(amount)".utf8)
    }
}

struct EmailNotifier: Notifier {
    func send(to address: String, subject: String, body: String) {
        print("Email to \(address): \(subject) — \(body)")
    }
}

// Single responsibility: orchestrate the invoicing use case
// Low coupling: depends on protocols, receives dependencies via injection
struct InvoiceService {
    let renderer: InvoiceRenderer
    let notifier: Notifier

    func issueInvoice(number: String, amount: Double, email: String) {
        let pdf = renderer.render(invoiceNumber: number, amount: amount)
        // … store pdf …
        notifier.send(
            to: email,
            subject: "Your invoice #\(number)",
            body: "Amount due: \(amount). PDF bytes: \(pdf.count)"
        )
    }
}

// Usage
let service = InvoiceService(renderer: PDFInvoiceRenderer(),
                             notifier: EmailNotifier())

service.issueInvoice(number: "INV-1001", amount: 149.0, email: "user@example.com")

// Testability: provide fakes to verify behavior without IO
struct FakeRenderer: InvoiceRenderer { 
    func render(invoiceNumber: String, amount: Double) -> Data { Data() } 
}

struct SpyNotifier: Notifier {
    private(set) var sent: [(String, String, String)] = []

    func send(to address: String, subject: String, body: String) { 
        sent.append((address, subject, body)) 
    }
}
```

Key takeaways:
- Tight coupling often coexists with SRP violations because a class both does its job and manages concrete collaborators.
- Using protocols plus dependency injection lowers coupling, isolates reasons to change, and improves testability and reuse.

### Notes on Coupling (and SRP)
- Low coupling supports SRP by keeping each class focused and less impacted by unrelated changes.
- Symptoms of tight coupling: 
    - creating dependencies inside a class (`new`/direct initializers)
    - reaching through multiple objects to call deep methods, depending on concrete types, and needing to change many classes for one change.
- Reduce coupling by: 
    - Depending on abstractions (protocols)
    - Injecting dependencies (initializer injection)
    - Following the Law of Demeter
        - Principle of least knowledge
        - A method should talk only to its immediate friends—not to friend's friends. That keeps coupling low and fits SRP.
        - function should call methods on things it directly knows: 
            - itself, its own properties
            - the parameters it received
            - objects it just created.
    - Separating pure logic from IO/side-effects.
      - Pure Logic
          - Arithmetic & numeric algorithms
          - Parsing strings → values
          - Validation rules
          - Data transformations
      - IO/side-effects
          - Database operations
          - Network calls
          - File system operations
          - UI updates (SwiftUI / UIKit)


So, it is safe to say that Single Responsibility principle is achieved by aiming for low coupling and high cohesion.

### Benefits of Single Responsibility Principle
- Easy to maintain the code base (Highly Scalable)
- Enhanced Readability
- A class with high cohesion is simpler to test
- Reduced Complexity and Dependencies (Simpler dependency graphs)

# Open/Closed Principle

**Software component should be closed for modification, but open for extension.** 

Open/Closed Principle (OCP) states that software entities should be **open for extension, but closed for modification**. You should be able to add new behavior by adding new code, not by changing existing, tested code. 

In practice this typically means depending on abstractions (protocols), using composition, and leveraging polymorphism instead of conditional explosions.

### Notes
- Extend via new types that conform to protocols; avoid modifying core algorithms when adding cases.
- Replace long `if`/`switch` conditionals on “type” with polymorphism (strategy, state, command, etc.).
- Prefer composition over inheritance; pass behavior in as dependencies.
- Combined with Single Responsibility Principle(SRP), each class has one reason to change, and new behavior lives in new classes.

### Example 1: Shapes Area (violation → Open/Closed Principle (OCP))

```swift
// Violation: adding a new shape requires modifying totalArea (edit existing code)
enum Shape {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
}

func totalArea(_ shapes: [Shape]) -> Double {
    return shapes.reduce(0) { sum, shape in
        switch shape {
        case let .circle(r):
            return sum + .pi * r * r
        case let .rectangle(w, h):
            return sum + w * h
        }
    }
}
```

Refactor with Open/Closed Principle(OCP) using polymorphism:

```swift
// Open for extension: add new Shape conformers
// Closed for modification: AreaCalculator never changes
protocol ShapeProtocol { 
    func area() -> Double 
}

struct Circle: ShapeProtocol {
    let radius: Double
    
    func area() -> Double { 
        return .pi * radius * radius 
    }
}

struct Rectangle: ShapeProtocol {
    let width: Double
    let height: Double
    
    func area() -> Double { 
        return width * height 
    }
}

struct AreaCalculator {
    func totalArea(_ shapes: [ShapeProtocol]) -> Double {
        shapes.reduce(0) { $0 + $1.area() }
    }
}

// Usage: adding Triangle requires only a new type, not changes to AreaCalculator
struct Triangle: ShapeProtocol {
    let base: Double, height: Double
    
    func area() -> Double { 
        return 0.5 * base * height 
    }
}

let areaCalculator = AreaCalculator()
let shapes: [ShapeProtocol] = [Circle(radius: 2), Rectangle(width: 3, height: 4), Triangle(base: 3, height: 5)]
let area = areaCalculator.totalArea(shapes)
```

### Example 2: Discount Strategy (extensible without edits)

```swift
// Violation: adding a new tier requires editing this function
enum CustomerTier { 
    case standard 
    case gold 
}

struct Order { 
    let total: Double 
}

func discountedTotal(order: Order, tier: CustomerTier) -> Double {
    switch tier {
    case .standard: 
        return order.total
    case .gold: 
        return order.total * 0.9
    }
}
```

Refactor to Open/Closed Principle(OCP) with strategies and registration:

```swift
protocol DiscountStrategy { 
    func apply(to total: Double) -> Double 
}

struct NoDiscount: DiscountStrategy { 
    func apply(to total: Double) -> Double { 
        return total 
    } 
}

struct TenPercent: DiscountStrategy { 
    func apply(to total: Double) -> Double { 
        return total * 0.9 
    } 
}

// Key-based registry allows adding new strategies without modifying engine
typealias DiscountCode = String

struct DiscountEngine {
    private var strategies: [DiscountCode: any DiscountStrategy]

    init(strategies: [DiscountCode: any DiscountStrategy]) {
        self.strategies = strategies
    }

    func total(for order: Order, code: DiscountCode?) -> Double {
        guard let code, let s = strategies[code] else { 
            return order.total 
        }

        return s.apply(to: order.total)
    }

    mutating func register(_ strategy: any DiscountStrategy, for code: DiscountCode) {
        strategies[code] = strategy
    }
}

// Usage
var engine = DiscountEngine(strategies: [
    "STANDARD": NoDiscount(),
    "GOLD": TenPercent()
])

let order = Order(total: 200)
let goldTotal = engine.total(for: order, code: "GOLD") // 180

// Add a new strategy later (no edits to DiscountEngine)
struct Flat20Off: DiscountStrategy { 
    func apply(to total: Double) -> Double { 
        return max(0, total - 20)    
    } 
}

engine.register(Flat20Off(), for: "PROMO20")
```

### Example 3
Scenario - One state insurance (currently handles only health type of insurance)

```swift
class InsurancePremiumDiscountCalculator {
    func calculatePremiumDiscountPercent(HealthInsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }
}

class HealthInsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}
```
Next Scenario - One state insurance extended themselves to support vehicle insurance too
Using method overloading to accommodate two insurance support in same class

```swift
class InsurancePremiumDiscountCalculator {
    func calculatePremiumDiscountPercent(HealthInsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }

    func calculatePremiumDiscountPercent(VehicleInsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }
}

class HealthInsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}

class VehicleInsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}
```

Next Scenario - One state insurance extended themselves to support Home insurance too
Now supporting Health, Vehicle and Home Insurance
Using method overloading to accommodate three insurance support in same class

```swift
class InsurancePremiumDiscountCalculator {
    func calculatePremiumDiscountPercent(HealthInsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }

    func calculatePremiumDiscountPercent(VehicleInsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }

    func calculatePremiumDiscountPercent(HomeInsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }
}

class HealthInsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}

class VehicleInsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}

class HomeInsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}
```

This is a very bad design because every time the Insurance company extend or modify their business 
not only we need to implement the new or modified features but also we need to modify the main class too.

Let's use protocol to fix this issue which follows open closed principle
```swift
protocol InsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool
}

class InsurancePremiumDiscountCalculator {
    func calculatePremiumDiscountPercent(InsuranceCustomerProfile customer) {
        if customer.isLoyalCustomer() -> Int {
            return 20
        }

        return 0
    }
}

class HealthInsuranceCustomerProfile: InsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}

class VehicleInsuranceCustomerProfile: InsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}

class HomeInsuranceCustomerProfile: InsuranceCustomerProfile {
    func isLoyalCustomer() -> Bool {
        // return true or false based on the assessment
    }
}
```

With this approach code is now easily expandable as business grows.

#### Benefits of Open/Closed Principle
- Improves maintainability and stability by protecting existing, tested code.
- Enables safer feature growth by adding new types/strategies instead of editing core logic.
- Ease of adding new features leads to minimal cost of developing and testing software.
- Open Closed Principle often requires decoupling which in turn automatically follows the Single responsibility principle.
- Reduces regression risk and merge conflicts when extending behavior.
- Encourages clean boundaries, protocols, and composition.

#### Caution
- Do not follow the open closed principle blindly.
- You will end up with a huge number of classes that can complicate overall design.

# Liskov's Substitution Principle

Objects should be replaceable with their subtypes without affecting the correctness of the program.

What it means that, the functions that use pointers or references to base/super classes must be able to use objects of derived/sub classes without knowing it.

Although Liskov's substitution principle build around Object Oriented Programming(OOP) approach, In swift we can use this approach with two programming approach.

- Object Oriented Programming - where relationship will evolve around Super class and sub class
- Protocol Oriented Programming - where relationship will evolve around protocol and classes or protocol and structs (SwiftUI preferred way)

Now let's talk about the pattern of problems that Liskov's Substition principle helps to solve.

## Hierarchical Problem
Liskov's Substitution Principle is often violated by forcing inheritance where behavioral contracts don’t actually align.

Let's consider a classic pitfall between Car and Formula one racing car.

```swift
class Car {
    func getCabinWidth() -> Double {
        // return cabinWidth
    }
}

class FormulaOneRacingCar: Car {
    override func getCabinWidth() -> Double {
        fatalError("Error: Formula  one racing car don't have cabin width")
    }

    func getCockpitWidth() -> Double {
        // return cockpit width
    }
}

let cars: [Car] = [
    Car(),
    Car(),
    FormulaOneRacingCar()
]

for car in cars {
    // this line will crash
    // since FormulaOneRacingCar throw fatal error when calling getCabinWidth
    // which violates Liskov's Substitution Principle
    print("Cabin width:", cars.getCabinWidth())
}
```
Let's address the by breaking the Hierarchy

```swift
class Vehicle {
    func getInteriorWidth() -> Double {
        // return interior Width
    }
}

class Car: Vehicle {
    override func getInteriorWidth() -> Double {
        return self.getCabinWidth()
    }

    func getCabinWidth() -> Double {
        // return cabinWidth
    }
}

class FormulaOneRacingCar: Vehicle {
    override func getInteriorWidth() -> Double {
        return self.getCockpitWidth()
    }

    func getCockpitWidth() -> Double {
        // return cockpit width
    }
}

let vehicles: [Vehicle] = [
    Car(),
    Car(),
    FormulaOneRacingCar()
]

for car in vehicles {
    print("Interior width:", cars.getInteriorWidth())
}
```

## Tell Don't ask
Tell Don’t Ask complements Liskov's Substitution Principle(LSP) and Open/Closed Principle(OCP): instead of asking an object for data and branching on its “type,” tell the object to perform the behavior via a shared abstraction. This keeps clients closed to change and preserves substitutability.

```swift
class Product {
    var discount: Double

    init(discount: Double = 5.0) {
        self.discount = discount
    }

    func getDiscount() -> Double {
        return discount
    }
}

class InHouseProduct: Product {
    init(discount: Double = 5.0) {
        super.init(discount: discount)
    }

    func applyExtraDiscount() {
        discount = discount * 2
    }
}

let products: [Product] = [
    Product(),
    Product(),
    InHouseProduct()
]

for product in products {
    if let inHouse = product as? InHouseProduct {
        inHouse.applyExtraDiscount()
    }

    print("Discount amount:", product.getDiscount())
}
```

The above code runs without crash but it violates Liskov's Substitution Principle(LSP). According to Liskov's Substitution Principle(LSP), we should have been able to deal with all the objects without checking and typecasting into the sub-class types.

We can re-structure the above code to follow Liskov's Substitution Principle(LSP).

```swift
class Product {
    var discount: Double

    init(discount: Double = 5.0) {
        self.discount = discount
    }

    func getDiscount() -> Double {
        return discount
    }
}

class InHouseProduct: Product {
    init(discount: Double = 5.0) {
        super.init(discount: discount)
    }

    override func getDiscount() -> Double {
        self.applyExtraDiscount()
        return discount
    }

    func applyExtraDiscount() {
        discount = discount * 2
    }
}

let products: [Product] = [
    Product(),
    Product(),
    InHouseProduct()
]

for product in products {
    print("Discount amount:", product.getDiscount())
}
```

Now the above code follows the Liskov's Substitution Principle(LSP).

#### Benefits of Liskov's Substitution Principle

- Increased Code Reusability: Generic reference uses.
- Improve Code Quality and Reliability - No need to worry about unexpected behavior of generic reference uses.
- Enhanced Flexibility and Extensibility: Systems can be extended by adding new subclasses without needing to modify existing client code that uses the base class.
- Improves Code Maintainability.
- Reduced Coupling: LSP helps lower the coupling between objects, which makes your code more modular and less prone to bugs when changes are made.
- Easier for testing: We can use dependency injection using mocks for testing purposes.

# Interface Segregation Principle

Interface Segregation Principle(ISP) states that **No client should be forced to depend on others methods it does not use**

This is the first SOLID principle which talks about interface(in swift case protocol) rather than class.

Consider a real life scenario to understand this problem.

A office has three devices

| Xerox Machine    | Printer and Scanner | Printer |
| ---------------- | -------------------- | -------------------- |
| ![Xerox Machine](Xerox.png) | ![Printer and Scanner](PrintNScan.png) | ![Printer](Printer.png) |

Now, we as programmer need to design a system by which user's of the office can use these devices.

Let's declare a protocol which will contain all the functions required for all these devices

```swift
protocol MultiFunction {
    func print()
    func scan()
    func fax(isInternet: Bool)
}
```
Xerox machine functionality implementation

```swift
class XeroxMachine: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        // implement the scan functionality
    }

    func fax(isInternet: Bool) {
        // implement the fax functionality
    }
}
```

So far everything looks fine.

How about implementing the Scanner and  Printer combo machine.

```swift
class PrintScanCombo: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        // implement the scan functionality
    }

    func fax(isInternet: Bool) {
        fatalError("Error! fax is not supported")
    }
}
```

Since, scanner and printer combo don't support fax functionality, if user attempt to use fax function it will crush and show an error message.

And Implementation of Printer as follows 

```swift
class Printer: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        fatalError("Error! scan is not supported")
    }

    func fax(isInternet: Bool) {
        fatalError("Error! fax is not supported")
    }
}
```

Same way, since printer don't support scan and fax functionality, upon calling these methods system will crush and show an error message.

Above implementation forces all classes to implement all methods of the protocol even if they don't use all of them which violates Interface Segregation Principle(ISP).

### Workaround of Interface Segregation Principle (Protocol Oriented Programming)

In Protocol Oriented Programming we can provide default implementation of protocol methods, so the if a class don't need certain methods, it can easily skip the method implementation.

```swift
protocol MultiFunction {
    func print()
    func scan()
    func fax(isInternet: Bool)
}

extension MultiFunction {
    func print() {
        print("Error! print is not supported")
    }

    func scan() {
        print("Error! scan is not supported")
    }

    func fax(isInternet: Bool) {
        print("Error! fax is not supported")
    }
}

// Xerox machine class implementing all the methods since it supports all the function provided by the interface
class XeroxMachine: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        // implement the scan functionality
    }

    func fax(isInternet: Bool) {
        // implement the fax functionality
    }
}

// PrintScanCombo machine is not implementing fax method since it don't support the functionality
class PrintScanCombo: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        // implement the scan functionality
    }
}

// Printer only implementing print method since it needs only that
class Printer: MultiFunction {
    func print() {
        // implement the print functionality
    }
}
```

The above implementation does not violates Interface Segregation Principle(ISP) since it the protocol is not forcing anyone to implement all the methods.

But there's still a issue, every classes who conforms the protocols will have the reference to the methods and their default implementation.

There can be an ambiguous situation where user might call the method and wondering why the functionality is not working. And if it is not working why it is there.

[![Image alt text](WhatThisButtonDo.jpg)](https://www.youtube.com/watch?v=pMuVm1Y669U)
**What does this button do?**

### Application of Interface Segregation Principle

To Properly address the issue, let's  separate the protocol according to the functionality

```swift
protocol PrintFunction {
    func print()
}

protocol ScanFunction {
    func scan()
}

protocol FaxFunction {
    func fax(isInternet: Bool)
}
```

Now, implements the classes according to their functionality needs

```swift
// Xerox machine class implementing 
class XeroxMachine: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        // implement the scan functionality
    }

    func fax(isInternet: Bool) {
        // implement the fax functionality
    }
}

// PrintScanCombo machine is not implementing fax method since it don't support the functionality
class PrintScanCombo: MultiFunction {
    func print() {
        // implement the print functionality
    }

    func scan() {
        // implement the scan functionality
    }
}

// Printer only implementing print method since it needs only that
class Printer: MultiFunction {
    func print() {
        // implement the print functionality
    }
}
```

# Dependency Inversion Principle

