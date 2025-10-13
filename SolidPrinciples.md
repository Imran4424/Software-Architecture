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

#### Benefits of Single Responsibility Principle
- 

# Open/Closed Principle
# Liskov's Substitution Principle
# Interface Segregation Principle
# Dependency Inversion Principle