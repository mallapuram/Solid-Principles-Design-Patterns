# SOLID Principles & Design Patterns in C#
### A Beginner-Friendly Guide with Real-World Analogies

---

> 💡 **How to use this guide:**
> Read it top to bottom if you're starting fresh. Each concept builds intuition before introducing code.
> The 🔥 icon marks concepts that appear *most frequently* in real-world C# projects.

---

# PART 1: SOLID PRINCIPLES

SOLID is an acronym for **5 design principles** that make your code easier to understand, maintain, and extend.

Think of SOLID like the rules of good kitchen design:
- Everything has its place
- You can add a new appliance without rewiring the whole kitchen
- One broken tool doesn't ruin everything else

---

## 🔥 S — Single Responsibility Principle (SRP)

### ✅ What is it?
> **A class should have only ONE reason to change.**

In plain English: each class should do **one thing and do it well**.

### ❓ Why do we need it?
Imagine a Swiss Army knife used as your primary kitchen knife, screwdriver, and scissors. If the blade breaks, the whole tool is unusable. Similarly, a class that does everything becomes a nightmare to maintain.

### 🕐 When should we use it?
Always. If a class has methods for fetching data, formatting it, AND saving it to a file — it's violating SRP.

### ❌ Problematic Approach

```csharp
// This class does TOO many things
public class Invoice
{
    public void CalculateTotal() { /* math logic */ }
    public void PrintInvoice() { /* printing logic */ }
    public void SaveToDatabase() { /* DB logic */ }
}
```

If the database changes, you touch `Invoice`. If the print format changes, you touch `Invoice`. Every change risks breaking everything.

### ✅ Improved Approach

```csharp
// Each class has ONE job
public class Invoice
{
    public void CalculateTotal() { /* only math */ }
}

public class InvoicePrinter
{
    public void Print(Invoice invoice) { /* only printing */ }
}

public class InvoiceRepository
{
    public void Save(Invoice invoice) { /* only DB saving */ }
}
```

Now a database change only touches `InvoiceRepository`. Clean and isolated.

### 🚫 When NOT to use it?
Don't over-split. A tiny class with 2 related methods doesn't need to be split into 2 files. Use judgment — related responsibilities can live together.

---

## 🔥 O — Open/Closed Principle (OCP)

### ✅ What is it?
> **A class should be open for extension but closed for modification.**

Real-world analogy: Think of a **power strip**. You can plug in new devices (extend it) without rewiring the strip itself (modify it).

### ❓ Why do we need it?
Every time you modify existing, tested code to add a new feature, you risk breaking what already works.

### 🕐 When should we use it?
When your system needs to support new types, behaviors, or variations — like adding payment methods, report formats, or discount types.

### ❌ Problematic Approach

```csharp
public class DiscountCalculator
{
    public double Calculate(string customerType, double price)
    {
        if (customerType == "Regular") return price * 0.95;
        if (customerType == "VIP") return price * 0.80;
        // Every new type forces you to modify this method ❌
        return price;
    }
}
```

### ✅ Improved Approach

```csharp
// Define the contract
public interface IDiscount
{
    double Apply(double price);
}

// Each discount type is its own class
public class RegularDiscount : IDiscount
{
    public double Apply(double price) => price * 0.95;
}

public class VIPDiscount : IDiscount
{
    public double Apply(double price) => price * 0.80;
}

// Add new discounts by creating a NEW class — never touch existing ones ✅
public class SeasonalDiscount : IDiscount
{
    public double Apply(double price) => price * 0.70;
}

public class DiscountCalculator
{
    public double Calculate(IDiscount discount, double price)
        => discount.Apply(price);
}
```

---

## L — Liskov Substitution Principle (LSP)

### ✅ What is it?
> **Subclasses should be replaceable for their parent class without breaking the program.**

Analogy: If a recipe says "use a bird," you should be able to use any bird. But if you use a **Penguin** and it can't fly when the recipe calls for flying, the dish is ruined.

### ❓ Why do we need it?
If your subclass behaves unexpectedly compared to its parent, polymorphism breaks and you get subtle bugs.

### ❌ Problematic Approach

```csharp
public class Bird
{
    public virtual void Fly() => Console.WriteLine("Flying...");
}

public class Penguin : Bird
{
    public override void Fly()
        => throw new NotImplementedException("Penguins can't fly!"); // ❌ Breaks LSP
}

// This will crash at runtime:
Bird bird = new Penguin();
bird.Fly(); // BOOM 💥
```

### ✅ Improved Approach

```csharp
// Separate what all birds can do from what only some can do
public abstract class Bird
{
    public abstract void Eat();
}

public interface IFlyable
{
    void Fly();
}

public class Sparrow : Bird, IFlyable
{
    public override void Eat() => Console.WriteLine("Sparrow eating.");
    public void Fly() => Console.WriteLine("Sparrow flying.");
}

public class Penguin : Bird
{
    public override void Eat() => Console.WriteLine("Penguin eating.");
    // Penguin simply doesn't implement IFlyable ✅
}
```

### 🚫 When NOT to use it?
This is a principle to follow, not opt out of. But don't over-engineer your hierarchy — sometimes composition is better than inheritance.

---

## 🔥 I — Interface Segregation Principle (ISP)

### ✅ What is it?
> **Don't force a class to implement methods it doesn't need.**

Analogy: Don't give a chef a job contract that also requires them to fix the plumbing. Keep roles focused.

### ❓ Why do we need it?
Fat interfaces force implementing classes to provide empty or fake implementations, which is confusing and error-prone.

### ❌ Problematic Approach

```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

// A Robot doesn't eat or sleep, but it's FORCED to implement them ❌
public class Robot : IWorker
{
    public void Work() => Console.WriteLine("Robot working.");
    public void Eat() => throw new NotImplementedException(); // Fake!
    public void Sleep() => throw new NotImplementedException(); // Fake!
}
```

### ✅ Improved Approach

```csharp
public interface IWorkable  { void Work(); }
public interface IEatable   { void Eat(); }
public interface ISleepable { void Sleep(); }

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work()  => Console.WriteLine("Human working.");
    public void Eat()   => Console.WriteLine("Human eating.");
    public void Sleep() => Console.WriteLine("Human sleeping.");
}

public class Robot : IWorkable
{
    public void Work() => Console.WriteLine("Robot working."); // Only what it needs ✅
}
```

---

## 🔥 D — Dependency Inversion Principle (DIP)

### ✅ What is it?
> **High-level classes should not depend on low-level classes. Both should depend on abstractions (interfaces).**

Analogy: Your laptop doesn't care if you plug in a Logitech or Microsoft mouse. It depends on the **USB standard** (abstraction), not a specific brand.

### ❓ Why do we need it?
When high-level logic is hardwired to low-level details (like a specific database or email provider), swapping them out means rewriting core code.

### ❌ Problematic Approach

```csharp
public class SqlDatabase
{
    public void Save(string data) => Console.WriteLine($"Saving '{data}' to SQL...");
}

public class OrderService
{
    private SqlDatabase _db = new SqlDatabase(); // Hardwired! ❌

    public void PlaceOrder(string order) => _db.Save(order);
}
// Want to switch to MongoDB? You must rewrite OrderService.
```

### ✅ Improved Approach

```csharp
// Define the abstraction
public interface IDatabase
{
    void Save(string data);
}

// Low-level implementations
public class SqlDatabase : IDatabase
{
    public void Save(string data) => Console.WriteLine($"SQL: Saving '{data}'");
}

public class MongoDatabase : IDatabase
{
    public void Save(string data) => Console.WriteLine($"MongoDB: Saving '{data}'");
}

// High-level class depends on the abstraction, not a concrete type ✅
public class OrderService
{
    private readonly IDatabase _db;

    public OrderService(IDatabase db) // Injected from outside
    {
        _db = db;
    }

    public void PlaceOrder(string order) => _db.Save(order);
}

// Usage — swap databases without touching OrderService:
var service = new OrderService(new MongoDatabase());
service.PlaceOrder("Laptop");
```

---

## SOLID Quick Reference

| Principle | Core Idea | Real-World Analogy |
|-----------|-----------|-------------------|
| **S**ingle Responsibility | One class, one job | Each employee has one role |
| **O**pen/Closed | Extend without modifying | Add a plugin, don't rewrite the app |
| **L**iskov Substitution | Subclasses behave like parents | Any screwdriver fits the handle |
| **I**nterface Segregation | Small, focused interfaces | Job contracts match the role |
| **D**ependency Inversion | Depend on abstractions | Laptop uses USB, not a specific mouse |

---

---

# PART 2: DESIGN PATTERNS

Design patterns are **proven, reusable solutions** to common problems in software design.

Think of them as **architectural blueprints** — like how all houses have a roof, walls, and foundation, but can be customized. You don't invent from scratch; you apply a known structure.

Patterns are grouped into 3 categories:
- **Creational** — How objects are created
- **Structural** — How objects are composed/connected
- **Behavioral** — How objects communicate and share responsibilities

---

# CREATIONAL PATTERNS

---

## 🔥 1. Singleton Pattern

### ✅ What is it?
> **Ensure only ONE instance of a class exists, and provide global access to it.**

Analogy: A country can have **only one president** at a time. No matter who asks "who's president?", they all get the same answer.

### ❓ Why do we need it?
Some resources should only exist once — like a configuration manager, logger, or database connection pool. Creating multiple instances wastes memory or causes inconsistency.

### 🕐 When to use it?
- Application-wide configuration
- Logging services
- Cache managers
- Thread pools

### ✅ Implementation

```csharp
public class Logger
{
    // The single instance, created only once
    private static Logger _instance;
    private static readonly object _lock = new object();

    // Private constructor prevents direct instantiation
    private Logger() { }

    public static Logger GetInstance()
    {
        if (_instance == null)
        {
            lock (_lock) // Thread-safe
            {
                if (_instance == null)
                    _instance = new Logger();
            }
        }
        return _instance;
    }

    public void Log(string message) => Console.WriteLine($"[LOG] {message}");
}

// Usage
Logger log1 = Logger.GetInstance();
Logger log2 = Logger.GetInstance();

Console.WriteLine(log1 == log2); // True — same instance ✅
log1.Log("Application started.");
```

### 🚫 When NOT to use it?
- When you need multiple configurations (e.g., dev vs prod)
- In unit-tested code — Singletons are hard to mock
- When it holds mutable shared state that could cause race conditions

---

## 🔥 2. Factory Method Pattern

### ✅ What is it?
> **Define an interface for creating objects, but let subclasses decide which class to instantiate.**

Analogy: A **vehicle factory** — you tell it "I need a vehicle for 2 passengers on land" and it hands you the right car. You don't build the car yourself.

### ❓ Why do we need it?
When your code shouldn't need to know *exactly which class* to create — just the type of thing it needs.

### 🕐 When to use it?
- Creating different notification types (Email, SMS, Push)
- Generating different report formats (PDF, Excel, HTML)
- Instantiating different payment processors

### ✅ Implementation

```csharp
// The product interface
public interface INotification
{
    void Send(string message);
}

// Concrete products
public class EmailNotification : INotification
{
    public void Send(string message) => Console.WriteLine($"Email: {message}");
}

public class SmsNotification : INotification
{
    public void Send(string message) => Console.WriteLine($"SMS: {message}");
}

public class PushNotification : INotification
{
    public void Send(string message) => Console.WriteLine($"Push: {message}");
}

// The Factory — decides which object to create
public static class NotificationFactory
{
    public static INotification Create(string type)
    {
        return type switch
        {
            "email" => new EmailNotification(),
            "sms"   => new SmsNotification(),
            "push"  => new PushNotification(),
            _ => throw new ArgumentException($"Unknown type: {type}")
        };
    }
}

// Usage — caller doesn't know or care about concrete classes ✅
INotification notif = NotificationFactory.Create("email");
notif.Send("Your order has shipped!");
```

### 🚫 When NOT to use it?
- When you only ever create one type of object — overkill
- When construction logic is trivial and unlikely to change

---

## 3. Builder Pattern

### ✅ What is it?
> **Construct complex objects step-by-step, separating construction from representation.**

Analogy: Building a **custom burger** at a restaurant — you say "add cheese, no onions, add bacon." The kitchen (builder) assembles it step by step.

### 🕐 When to use it?
- When an object has many optional parameters (avoid telescoping constructors)
- Building SQL queries, HTTP requests, or complex configuration objects

### ✅ Implementation

```csharp
public class Pizza
{
    public string Size { get; set; }
    public bool HasCheese { get; set; }
    public bool HasPepperoni { get; set; }
    public string Crust { get; set; }

    public override string ToString()
        => $"{Size} pizza, Crust: {Crust}, Cheese: {HasCheese}, Pepperoni: {HasPepperoni}";
}

public class PizzaBuilder
{
    private Pizza _pizza = new Pizza();

    public PizzaBuilder SetSize(string size) { _pizza.Size = size; return this; }
    public PizzaBuilder AddCheese() { _pizza.HasCheese = true; return this; }
    public PizzaBuilder AddPepperoni() { _pizza.HasPepperoni = true; return this; }
    public PizzaBuilder SetCrust(string crust) { _pizza.Crust = crust; return this; }
    public Pizza Build() => _pizza;
}

// Usage — fluent, readable ✅
Pizza myPizza = new PizzaBuilder()
    .SetSize("Large")
    .SetCrust("Thin")
    .AddCheese()
    .AddPepperoni()
    .Build();

Console.WriteLine(myPizza);
// Output: Large pizza, Crust: Thin, Cheese: True, Pepperoni: True
```

---

# STRUCTURAL PATTERNS

---

## 🔥 4. Adapter Pattern

### ✅ What is it?
> **Convert the interface of a class into another interface that clients expect.**

Analogy: A **travel power adapter** — your US plug doesn't fit a European socket. The adapter bridges the gap without changing either the plug or the socket.

### 🕐 When to use it?
- Integrating third-party libraries with incompatible interfaces
- Working with legacy code that you can't modify
- Connecting two systems with different data formats

### ✅ Implementation

```csharp
// What your application expects
public interface IPaymentProcessor
{
    void ProcessPayment(double amount);
}

// A third-party library you can't change
public class StripePayment
{
    public void MakeCharge(double dollars) // Different method name!
        => Console.WriteLine($"Stripe charged ${dollars}");
}

// The Adapter — wraps Stripe and makes it look like IPaymentProcessor ✅
public class StripeAdapter : IPaymentProcessor
{
    private readonly StripePayment _stripe = new StripePayment();

    public void ProcessPayment(double amount)
        => _stripe.MakeCharge(amount); // Delegates to Stripe's method
}

// Usage — your app works with the standard interface
IPaymentProcessor processor = new StripeAdapter();
processor.ProcessPayment(99.99);
```

---

## 5. Decorator Pattern

### ✅ What is it?
> **Add new behavior to an object dynamically, without changing its class.**

Analogy: A **plain coffee** is the base. You can add milk (decorated), then add sugar (decorated again). Each add-on wraps the previous.

### 🕐 When to use it?
- Adding features to objects without subclassing
- Logging, caching, or validation wrappers around services
- UI component enhancements

### ✅ Implementation

```csharp
public interface ICoffee
{
    string GetDescription();
    double GetCost();
}

public class SimpleCoffee : ICoffee
{
    public string GetDescription() => "Coffee";
    public double GetCost() => 1.00;
}

// Base decorator
public abstract class CoffeeDecorator : ICoffee
{
    protected ICoffee _coffee;
    public CoffeeDecorator(ICoffee coffee) => _coffee = coffee;
    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual double GetCost() => _coffee.GetCost();
}

public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }
    public override string GetDescription() => _coffee.GetDescription() + ", Milk";
    public override double GetCost() => _coffee.GetCost() + 0.25;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }
    public override string GetDescription() => _coffee.GetDescription() + ", Sugar";
    public override double GetCost() => _coffee.GetCost() + 0.10;
}

// Usage ✅
ICoffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

Console.WriteLine(coffee.GetDescription()); // Coffee, Milk, Sugar
Console.WriteLine($"${coffee.GetCost()}");  // $1.35
```

---

## 6. Facade Pattern

### ✅ What is it?
> **Provide a simple interface to a complex subsystem.**

Analogy: A **TV remote** is a facade. It hides the complexity of signal processing, display hardware, and audio circuits behind simple buttons.

### 🕐 When to use it?
- Wrapping complex APIs or legacy systems
- Providing a clean entry point to a subsystem (e.g., ordering flow that involves inventory, billing, and shipping)

### ✅ Implementation

```csharp
// Complex subsystems
public class Inventory  { public bool Check(string item) { Console.WriteLine("Checking inventory..."); return true; } }
public class Payment    { public bool Charge(double amount) { Console.WriteLine("Charging payment..."); return true; } }
public class Shipping   { public void Ship(string item) => Console.WriteLine($"Shipping {item}..."); }

// The Facade — one simple method hides all complexity ✅
public class OrderFacade
{
    private Inventory _inventory = new Inventory();
    private Payment _payment = new Payment();
    private Shipping _shipping = new Shipping();

    public void PlaceOrder(string item, double price)
    {
        if (_inventory.Check(item) && _payment.Charge(price))
            _shipping.Ship(item);
        else
            Console.WriteLine("Order failed.");
    }
}

// Caller only needs to know one method:
var order = new OrderFacade();
order.PlaceOrder("Laptop", 999.99);
```

---

# BEHAVIORAL PATTERNS

---

## 🔥 7. Strategy Pattern

### ✅ What is it?
> **Define a family of algorithms, encapsulate each one, and make them interchangeable.**

Analogy: **Navigation apps** — you pick the strategy: fastest route, avoid tolls, or walking. The app swaps the algorithm based on your choice.

### ❓ Why do we need it?
Avoids massive `if/else` or `switch` blocks when selecting behavior at runtime.

### 🕐 When to use it?
- Sorting algorithms
- Payment methods
- Compression formats
- Shipping cost calculations

### ✅ Implementation

```csharp
// The strategy contract
public interface ISortStrategy
{
    void Sort(List<int> data);
}

// Concrete strategies
public class BubbleSortStrategy : ISortStrategy
{
    public void Sort(List<int> data)
    {
        // Simplified for illustration
        data.Sort();
        Console.WriteLine("Sorted with Bubble Sort.");
    }
}

public class QuickSortStrategy : ISortStrategy
{
    public void Sort(List<int> data)
    {
        data.Sort();
        Console.WriteLine("Sorted with Quick Sort.");
    }
}

// Context — uses whichever strategy is injected
public class Sorter
{
    private ISortStrategy _strategy;

    public Sorter(ISortStrategy strategy)
    {
        _strategy = strategy;
    }

    public void SetStrategy(ISortStrategy strategy) => _strategy = strategy;

    public void Sort(List<int> data) => _strategy.Sort(data);
}

// Usage ✅
var data = new List<int> { 5, 3, 1, 4, 2 };
var sorter = new Sorter(new BubbleSortStrategy());
sorter.Sort(data);

// Switch strategy at runtime — no code changes needed:
sorter.SetStrategy(new QuickSortStrategy());
sorter.Sort(data);
```

---

## 🔥 8. Observer Pattern

### ✅ What is it?
> **When one object changes state, all its dependents are automatically notified.**

Analogy: **YouTube subscriptions** — when a channel uploads a video, all subscribers are notified automatically. The channel doesn't know each subscriber personally.

### ❓ Why do we need it?
Avoids tight coupling between the source of events and the things that react to them.

### 🕐 When to use it?
- Event systems (UI button clicks, data change events)
- Stock price alerts
- Real-time dashboards
- Notification systems

### ✅ Implementation

```csharp
// Observer interface
public interface IObserver
{
    void Update(string stockName, double price);
}

// Subject (the thing being watched)
public class StockMarket
{
    private List<IObserver> _observers = new List<IObserver>();
    private string _stockName;
    private double _price;

    public void Subscribe(IObserver observer) => _observers.Add(observer);
    public void Unsubscribe(IObserver observer) => _observers.Remove(observer);

    public void SetPrice(string stockName, double price)
    {
        _stockName = stockName;
        _price = price;
        NotifyAll();
    }

    private void NotifyAll()
    {
        foreach (var observer in _observers)
            observer.Update(_stockName, _price);
    }
}

// Concrete observers
public class StockAlert : IObserver
{
    private string _name;
    public StockAlert(string name) => _name = name;

    public void Update(string stockName, double price)
        => Console.WriteLine($"[{_name}] Alert: {stockName} is now ${price}");
}

// Usage ✅
var market = new StockMarket();
market.Subscribe(new StockAlert("Alice"));
market.Subscribe(new StockAlert("Bob"));

market.SetPrice("AAPL", 182.50);
// [Alice] Alert: AAPL is now $182.5
// [Bob]   Alert: AAPL is now $182.5
```

---

## 🔥 9. Command Pattern

### ✅ What is it?
> **Encapsulate a request as an object, allowing you to queue, log, or undo operations.**

Analogy: A **restaurant order slip** — the waiter writes down your order (command object) and hands it to the kitchen. The kitchen executes it. You can also cancel an order before it's prepared (undo).

### 🕐 When to use it?
- Undo/Redo functionality (text editors, drawing apps)
- Task queuing and scheduling
- Macro recording
- Transaction logs

### ✅ Implementation

```csharp
// Command interface
public interface ICommand
{
    void Execute();
    void Undo();
}

// The thing being operated on
public class Light
{
    public void TurnOn()  => Console.WriteLine("Light ON");
    public void TurnOff() => Console.WriteLine("Light OFF");
}

// Concrete commands
public class TurnOnCommand : ICommand
{
    private Light _light;
    public TurnOnCommand(Light light) => _light = light;
    public void Execute() => _light.TurnOn();
    public void Undo()    => _light.TurnOff();
}

// The invoker — executes and tracks commands
public class RemoteControl
{
    private Stack<ICommand> _history = new Stack<ICommand>();

    public void PressButton(ICommand command)
    {
        command.Execute();
        _history.Push(command);
    }

    public void PressUndo()
    {
        if (_history.Count > 0)
            _history.Pop().Undo();
    }
}

// Usage ✅
var light = new Light();
var remote = new RemoteControl();

remote.PressButton(new TurnOnCommand(light)); // Light ON
remote.PressUndo();                           // Light OFF (undo!)
```

---

## 10. Template Method Pattern

### ✅ What is it?
> **Define the skeleton of an algorithm in a base class, letting subclasses fill in the details.**

Analogy: A **recipe template** — "Prepare ingredients → Cook → Plate" is always the order. But what you cook changes.

### 🕐 When to use it?
- Data export pipelines (read → transform → write)
- Game loops (initialize → play → end)
- Report generation with different formats

### ✅ Implementation

```csharp
public abstract class DataExporter
{
    // Template method — defines the steps ✅
    public void Export()
    {
        ReadData();
        ProcessData();
        WriteOutput();
    }

    protected abstract void ReadData();
    protected abstract void ProcessData();
    protected abstract void WriteOutput();
}

public class CsvExporter : DataExporter
{
    protected override void ReadData()    => Console.WriteLine("Reading from database...");
    protected override void ProcessData() => Console.WriteLine("Formatting as CSV...");
    protected override void WriteOutput() => Console.WriteLine("Writing to .csv file.");
}

public class JsonExporter : DataExporter
{
    protected override void ReadData()    => Console.WriteLine("Reading from API...");
    protected override void ProcessData() => Console.WriteLine("Serializing to JSON...");
    protected override void WriteOutput() => Console.WriteLine("Writing to .json file.");
}

// Usage
DataExporter exporter = new CsvExporter();
exporter.Export();
```

---

## 11. Iterator Pattern

### ✅ What is it?
> **Provide a standard way to traverse a collection without exposing its internal structure.**

Analogy: A **TV remote's channel up/down button** — you cycle through channels without knowing how they're stored internally.

C# has this built-in via `IEnumerable` and `foreach`, but understanding the pattern helps when building custom collections.

### ✅ Implementation (Custom Iterator)

```csharp
public class NumberCollection : IEnumerable<int>
{
    private List<int> _numbers = new List<int>();

    public void Add(int number) => _numbers.Add(number);

    // Returns built-in iterator — IEnumerable handles traversal ✅
    public IEnumerator<int> GetEnumerator() => _numbers.GetEnumerator();
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}

// Usage
var collection = new NumberCollection();
collection.Add(10);
collection.Add(20);
collection.Add(30);

foreach (var num in collection) // Clean, no knowledge of internals
    Console.WriteLine(num);
```

---

## 12. State Pattern

### ✅ What is it?
> **Allow an object to change its behavior when its internal state changes.**

Analogy: A **traffic light** — the same light behaves differently depending on its current state (Red, Yellow, Green).

### 🕐 When to use it?
- Order lifecycle (Pending → Processing → Shipped → Delivered)
- User authentication states
- Game character states (Idle, Running, Jumping)

### ✅ Implementation

```csharp
public interface IOrderState
{
    void Handle(Order order);
}

public class Order
{
    public IOrderState State { get; set; } = new PendingState();
    public string Name { get; set; }

    public Order(string name) => Name = name;

    public void Proceed() => State.Handle(this);
}

public class PendingState : IOrderState
{
    public void Handle(Order order)
    {
        Console.WriteLine($"{order.Name}: Pending → Processing");
        order.State = new ProcessingState();
    }
}

public class ProcessingState : IOrderState
{
    public void Handle(Order order)
    {
        Console.WriteLine($"{order.Name}: Processing → Shipped");
        order.State = new ShippedState();
    }
}

public class ShippedState : IOrderState
{
    public void Handle(Order order)
        => Console.WriteLine($"{order.Name}: Already shipped. No further transitions.");
}

// Usage ✅
var order = new Order("Laptop");
order.Proceed(); // Pending → Processing
order.Proceed(); // Processing → Shipped
order.Proceed(); // Already shipped.
```

---

# PART 3: PATTERNS AT A GLANCE

## Design Patterns Quick Reference

| Pattern | Category | Core Idea | Use When |
|---------|----------|-----------|----------|
| 🔥 **Singleton** | Creational | One instance globally | Logger, config, cache |
| 🔥 **Factory Method** | Creational | Delegate object creation | Multiple types of the same thing |
| **Builder** | Creational | Build complex objects step-by-step | Many optional parameters |
| 🔥 **Adapter** | Structural | Bridge incompatible interfaces | Third-party integration |
| **Decorator** | Structural | Wrap objects to add behavior | Add features without subclassing |
| **Facade** | Structural | Simple interface to complex system | Hide subsystem complexity |
| 🔥 **Strategy** | Behavioral | Swap algorithms at runtime | Multiple ways to do one thing |
| 🔥 **Observer** | Behavioral | Notify dependents on state change | Events, subscriptions |
| 🔥 **Command** | Behavioral | Encapsulate requests as objects | Undo/redo, task queue |
| **Template Method** | Behavioral | Define skeleton, let subclasses fill in | Pipelines with fixed steps |
| **Iterator** | Behavioral | Standard traversal of collections | Custom collections |
| **State** | Behavioral | Change behavior based on state | Workflows, lifecycles |

---

## 🗺️ When to Apply What — Decision Guide

```
Is the problem about OBJECT CREATION?
├── Need only ONE instance?           → Singleton
├── Don't know which class to create? → Factory Method
└── Complex object with many parts?   → Builder

Is the problem about CONNECTING CLASSES?
├── Incompatible interfaces?          → Adapter
├── Need to add behavior dynamically? → Decorator
└── Hiding complex subsystem?         → Facade

Is the problem about BEHAVIOR / COMMUNICATION?
├── Multiple ways to do one thing?    → Strategy
├── Reacting to changes in another?   → Observer
├── Need undo / queuing?              → Command
├── Fixed steps, varying details?     → Template Method
└── Behavior changes based on state?  → State
```

---

## 💡 Final Tips for Real-World Application

1. **Don't force patterns.** Patterns solve specific problems. If you don't have the problem, don't apply the pattern.

2. **Start with SOLID.** Clean SOLID code often naturally leads you toward the right patterns.

3. **The most used patterns in enterprise C#:**
   - Strategy + Factory (business rules)
   - Observer (event-driven systems)
   - Singleton (services, config)
   - Command (CQRS in .NET)
   - Adapter (third-party integrations)

4. **Patterns + Dependency Injection:** In .NET, the built-in DI container works beautifully with Factory, Strategy, and Singleton patterns. Learn `IServiceCollection` once you're comfortable with these patterns.

5. **Learn by refactoring.** Take old messy code and ask: "Which principle is violated here? Which pattern would fix it?"

---

*Happy coding! 🚀 The goal isn't to memorize patterns — it's to recognize the problems they solve.*
