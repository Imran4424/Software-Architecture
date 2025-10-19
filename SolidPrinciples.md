# Solid Principles

The SOLID principles are composed of five simple but essential principles intended to make Software Design easily maintainable and scalable.

1. **S** - Single Responsibility Principle
2. **O** - Open/Closed Principle
3. **L** - Liskov's Substitution Principle
4. **I** - Interface Segregation Principle
5. **D** - Dependency Inversion Principle

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
## Coupling
Coupling is defined as the level of inter dependency between various software components.

When modules have a high coupling, they are intimately linked, and modifications to one module may have an impact on other modules.  Low coupling indicates that modules are self-contained and that modifications to one module hardly affect the others.

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
    - creating dependencies inside a class (`new`/direct initializers), 
    - reaching through multiple objects to call deep methods, depending on concrete types, and needing to change many classes for one change.
- Reduce coupling by: 
    - Depending on abstractions (protocols)
    - Injecting dependencies (initializer injection)
    - Following the Law of Demeter
    - Separating pure logic from IO/side-effects.




#### Benefits of Single Responsibility Principle
- Easy to maintain the code base
- Enhanced Readability
- A class with high cohesion is simpler to test
- Reduced Complexity and Dependencies  

# Open/Closed Principle
# Liskov's Substitution Principle
# Interface Segregation Principle
# Dependency Inversion Principle
