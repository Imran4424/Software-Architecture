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

## Coupling

# Open/Closed Principle
# Liskov's Substitution Principle
# Interface Segregation Principle
# Dependency Inversion Principle