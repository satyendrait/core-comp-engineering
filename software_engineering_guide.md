# Software Engineering & Backend Architecture Guide

A comprehensive, production-oriented reference manual covering Object-Oriented Programming (OOP), SOLID design principles, Database Systems (ACID vs. BASE), Database Normalization, and core Design Patterns implemented in Java.

---

## ☕ 1. Object-Oriented Programming (OOP) Deep Dive

Object-Oriented Programming (OOP) is a paradigm centered around **objects** (encapsulating state) and **behaviors** (methods), rather than isolated procedural logic. 

### The Foundational Blocks: Classes and Objects
* **Class:** A compile-time blueprint or template defining the structure and capabilities of a type. It occupies no heap memory.
* **Object:** A concrete runtime instance allocated on the JVM heap via the `new` keyword.

```java
// The Blueprint (Class)
public class Car {
    String model; // State
    int year;

    void drive() { // Behavior
        System.out.println(model + " is driving.");
    }
}

// Execution and Instantiation
public class Main {
    public static void main(String[] args) {
        Car myCar = new Car(); // Memory allocated on the heap
        myCar.model = "Model 3";
        myCar.drive();
    }
}
```

---

### The Four Pillars of OOP

#### 1. Encapsulation (Data Hiding)
Restricts direct access to an object's internal fields by making variables `private` and exposing them exclusively via public accessors (`getters`) and mutators (`setters`). This guarantees state validation and protection against external corruption.

```java
public class BankAccount {
    private double balance; // Hidden data field

    // Controlled Entry Point (Getter)
    public double getBalance() {
        return this.balance;
    }

    // Validation-backed modifier (Setter)
    public void deposit(double amount) {
        if (amount > 0) {
            this.balance += amount;
        }
    }
}
```

#### 2. Inheritance (Code Reusability)
Establishes an **"IS-A"** relationship allowing a subclass to inherit the fields and behaviors of a parent superclass using the `extends` keyword. Java does not support multi-class inheritance to avoid structural ambiguity (the Diamond Problem); it enforces single-inheritance but allows multiple implementation via interfaces.

```java
// Superclass
public class Vehicle {
    protected String brand = "Generic";
    
    public void honk() {
        System.out.println("Beep, beep!");
    }
}

// Subclass inheriting capabilities
public class ElectricCar extends Vehicle {
    private int batteryCapacity = 100;
    
    public void displayBrand() {
        System.out.println("Brand: " + this.brand); // Inherited field
    }
}
```

#### 3. Polymorphism (Many Forms)
Enables instructions to act differently based on contextual types.

##### A. Compile-Time Polymorphism (Method Overloading)
Methods in the same class share a name but differ in parameter signatures (count, types, or positioning). Resolved early by the compiler.

```java
public class MathOperations {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; } // Overloaded signature
}
```

##### B. Runtime Polymorphism (Dynamic Method Dispatch)
A runtime resolution protocol where an overridden method is executed based on the target execution **object type**, not the variable's **reference type**. This relies strictly on **Upcasting**.

```java
// Base Class
class PaymentProcessor {
    void processPayment(double amount) {
        System.out.println("Processing generic payment of $" + amount);
    }
}

// Subclass A
class CreditCardProcessor extends PaymentProcessor {
    @Override
    void processPayment(double amount) {
        System.out.println("Charging $" + amount + " via Credit Card with a 2% processing fee.");
    }
}

// Subclass B
class PayPalProcessor extends PaymentProcessor {
    @Override
    void processPayment(double amount) {
        System.out.println("Redirecting user to PayPal gateway to complete transaction of $" + amount);
    }
}

// Execution Loop
public class DispatchDemo {
    public static void main(String[] args) {
        // Upcasting: Reference type is parent, object type is subclass
        PaymentProcessor processor = new CreditCardProcessor();
        
        // Compiler checks reference type (PaymentProcessor has processPayment)
        // JVM checks object type (CreditCardProcessor is executing)
        processor.processPayment(150.00); 
        // Output: Charging $150.00 via Credit Card with a 2% processing fee.

        // Dynamically reassigning the reference container
        processor = new PayPalProcessor();
        processor.processPayment(150.00);
        // Output: Redirecting user to PayPal gateway to complete transaction of $150.00
    }
}
```

#### 4. Abstraction (Hiding Complexity)
Separates the semantic context of *what* an object does from the explicit structural details of *how* it performs that action. Implemented using abstract classes or interfaces.

```java
// 100% Structural Abstraction
public interface RemoteControl {
    void pressPowerButton(); 
}

public class TVRemote implements RemoteControl {
    @Override
    public void pressPowerButton() {
        System.out.println("Transmitting infrared signals to turn on the Television.");
    }
}
```

---

## 🛠️ 2. SOLID Design Principles

### S: Single Responsibility Principle (SRP)
> A class must serve one standalone operational scope. It should have only a single reason to change.

```java
// REFACTORED CLEAN DESIGN: Separation of entities and operations
public class Invoice {
    private double amount;
    public Invoice(double amount) { this.amount = amount; }
    public double getAmount() { return amount; }
}

public class InvoiceRepository {
    public void saveToDatabase(Invoice invoice) {
        System.out.println("Saving invoice worth $" + invoice.getAmount() + " to DB storage.");
    }
}
```

### O: Open/Closed Principle (OCP)
> Systems must be open for structural extension, but closed to direct modifications of verified production logic.

```java
// Interface abstraction allows unlimited strategies without modifying calculator core
public interface DiscountStrategy {
    double apply(double amount);
}

public class RegularDiscount implements DiscountStrategy {
    public double apply(double amount) { return amount * 0.9; }
}

public class VIPDiscount implements DiscountStrategy {
    public double apply(double amount) { return amount * 0.8; }
}

public class DiscountCalculator {
    public double calculate(DiscountStrategy strategy, double amount) {
        return strategy.apply(amount); // Closed for modifications, completely open to extensions
    }
}
```

### L: Liskov Substitution Principle (LSP)
> Subclasses must be completely substitutable for their parent classes without breaking application expectations or throwing unexpected exceptions.

```java
// REFACTORED CLEAN DESIGN: Split hierarchies to ensure semantic accuracy
public class Bird {
    public void eat() { System.out.println("Consuming organic energy inputs."); }
}

public class FlyingBird extends Bird {
    public void fly() { System.out.println("Propelling through spatial coordinates."); }
}

public class Ostrich extends Bird {
    // Avoids inheriting a broken fly() behavior that throws UnsupportedOperationException
}
```

### I: Interface Segregation Principle (ISP)
> Lean, role-specific interfaces are superior to broad, universal interfaces. Clients should never be forced to depend on methods they do not use.

```java
// Narrow, single-action utility boundaries
public interface Printer { void print(); }
public interface Scanner { void scan(); }

public class SimplePrinter implements Printer {
    public void print() { System.out.println("Rendering output pixels on paper."); }
}

public class MultifunctionOfficeHub implements Printer, Scanner {
    public void print() { System.out.println("Printing documents."); }
    public void scan() { System.out.println("Digitalizing physical inputs."); }
}
```

### D: Dependency Inversion Principle (DIP)
> High-level orchestration business logic must not depend on concrete low-level implementations; both must rely strictly on interfaces and abstractions.

```java
// Abstraction layer decouples dependencies
public interface Engine { void start(); }

public class ElectricEngine implements Engine {
    public void start() { System.out.println("Generating rotational magnetic field flux."); }
}

public class Car {
    private final Engine engine; // Dependent on the contract, not the detail

    // Injected via Constructor Injection
    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() { 
        engine.start(); 
    }
}
```

---

## 💾 3. Database System Architecture (ACID vs. BASE)

When evaluating persistence layer states across distributed environments, systems gravitate towards either tight compliance structures (**ACID**) or distributed, highly available configurations (**BASE**) in accordance with the **CAP Theorem** (Consistency, Availability, Partition Tolerance).

### ACID Model (Pessimistic / Exact Processing)
Optimized for transactional systems where transactional bugs cause systemic damage (e.g., financial networks).
* **Atomicity:** All operations within a unit of work succeed together, or the database rolls back to its exact pre-transaction state.
* **Consistency:** Data shifts safely between invariant states, satisfying all schema constraints, checks, and foreign keys.
* **Isolation:** Concurrently executing transactions do not bleed intermediate states into each other. Supported by isolation configurations like Read Committed or Serializable.
* **Durability:** Committed states are permanently flushed to non-volatile disk logs (Write-Ahead Logs) to withstand host system crashes.

### BASE Model (Optimistic / High Availability)
Optimized for web-scale configurations where latency or system partitions cannot dictate total service outages (e.g., streaming platform metadata, social feeds).
* **Basically Available (BA):** The persistence layer remains online and returns data from healthy nodes even amidst active infrastructure partitioning.
* **Soft State (S):** State boundaries can fluctuate continuously without synchronous external interaction while nodes sync background state changes.
* **Eventual Consistency (E):** The system completely unifies all state snapshots over asynchronous intervals, guaranteeing consistency *eventually* if no additional write inputs occur.

---

## 📐 4. Database Normalization (1NF ➡️ 2NF ➡️ 3NF)

Normalization organizes database records to systematically eliminate data structural anomalies (Insertion, Update, Deletion anomalies) while enforcing the rule: *One fact in one place.*

### First Normal Form (1NF)
Every row must contain an immutable primary identity key, and every column cell must isolate completely atomic (indivisible), non-repeating data scalar values.

### Second Normal Form (2NF)
The schema must meet 1NF criteria, and **all non-key column attributes must be fully functionally dependent on the entire primary key**. This eliminates partial key dependencies in composite key environments.

### Third Normal Form (3NF)
The schema must meet 2NF criteria, and **no non-key attribute can possess a transitive dependency on the primary identity key through an alternative non-key attribute**.

---

## 🎨 5. Structural Blueprints: Core Design Patterns

### 🏗️ Creational Patterns

#### 1. Singleton
Ensures a single operational instance exists globally within runtime loops while maintaining an encapsulated structural access channel.

```java
public class DatabaseConnectionPool {
    private static volatile DatabaseConnectionPool instance;

    private DatabaseConnectionPool() {
        // Enforce private boundary protection
    }

    public static DatabaseConnectionPool getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnectionPool.class) {
                if (instance == null) { // Double-checked locking paradigm
                    instance = new DatabaseConnectionPool();
                }
            }
        }
        return instance;
    }
}
```

#### 2. Factory Method
Defines an object-instantiation contract but moves the concrete creation routing to underlying execution context subclasses.

```java
interface Notification { void executeDispatch(); }

class SMSNotification implements Notification {
    public void executeDispatch() { System.out.println("Routing cellular carrier packet."); }
}

class EmailNotification implements Notification {
    public void executeDispatch() { System.out.println("Processing SMTP transmission pipeline."); }
}

class NotificationFactory {
    public Notification buildChannel(String contextToken) {
        if (contextToken.equalsIgnoreCase("SMS")) return new SMSNotification();
        if (contextToken.equalsIgnoreCase("EMAIL")) return new EmailNotification();
        throw new IllegalArgumentException("Unknown channel token context.");
    }
}
```

#### 3. Builder
Separates complex component orchestration from structural generation, allowing developers to safely construct intricate configurations step-by-step.

```java
public class NetworkRequest {
    private final String url;
    private final String method;
    private final int timeout;

    private NetworkRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.timeout = builder.timeout;
    }

    public static class Builder {
        private final String url; // Mandatory requirement
        private String method = "GET"; // Default configuration option
        private int timeout = 5000;

        public Builder(String url) { this.url = url; }
        public Builder withMethod(String method) { this.method = method; return this; }
        public Builder withTimeout(int timeout) { this.timeout = timeout; return this; }
        public NetworkRequest build() { return new NetworkRequest(this); }
    }
}
```

---

### 🧱 Structural Patterns

#### 1. Adapter
Converts an unconventional or incompatible application programmatic contract interface into a structural signature signature expected by target components.

```java
interface TargetProtocol { void performModernRequest(); }

class LegacySystem {
    public void processOldExecutionFormat() { System.out.println("Handling raw string buffers."); }
}

class SystemAdapter implements TargetProtocol {
    private final LegacySystem legacyService;
    
    public SystemAdapter(LegacySystem service) { this.legacyService = service; }
    
    @Override
    public void performModernRequest() {
        // Wraps legacy data logic into modern interface execution context
        legacyService.processOldExecutionFormat();
    }
}
```

#### 2. Decorator
Dynamically attaches new behaviors and features to an object at runtime without altering its underlying interface or structural definition.

```java
interface HTTPResponse { String writePayload(); }

class BaseHTMLResponse implements HTTPResponse {
    public String writePayload() { return "Raw HTML Data String"; }
}

abstract class ResponseDecorator implements HTTPResponse {
    protected final HTTPResponse wrappedComponent;
    public ResponseDecorator(HTTPResponse component) { this.wrappedComponent = component; }
}

class GZipCompressionDecorator extends ResponseDecorator {
    public GZipCompressionDecorator(HTTPResponse component) { super(component); }
    
    @Override
    public String writePayload() {
        return "GZIP_COMPRESSED[" + wrappedComponent.writePayload() + "]";
    }
}
```

#### 3. Facade
Synthesizes complex, tightly coupled low-level subsystem interfaces behind a single high-level unified interface layer.

```java
class StorageSubsystem { void initializeDisk() {} }
class SecuritySubsystem { void configureAccessTokens() {} }

public class CloudDeploymentFacade {
    private final StorageSubsystem storage = new StorageSubsystem();
    private final SecuritySubsystem security = new SecuritySubsystem();

    public void launchCloudEnvironment() {
        storage.initializeDisk();
        security.configureAccessTokens();
        System.out.println("Subsystems abstracted. Production environment safe and active.");
    }
}
```

---

### ⚙️ Behavioral Patterns

#### 1. Observer
Builds subscription hooks allowing state modification actions within one runtime target to automatically publish updates to listening dependants.

```java
import java.util.*;

interface Observer { void receiveNotification(String stateUpdate); }

class CentralTelemetryService {
    private final List<Observer> activeSubscribers = new ArrayList<>();
    
    public void attach(Observer observer) { activeSubscribers.add(observer); }
    
    public void broadcastTelemetry(String eventData) {
        for (Observer sub : activeSubscribers) {
            sub.receiveNotification(eventData);
        }
    }
}
```

#### 2. Strategy
Encapsulates a family of operational algorithms, rendering them entirely interchangeable at runtime inside host processes.

```java
interface SortingStrategy { void executeSort(int[] collection); }

class QuickSortAlgorithm implements SortingStrategy {
    public void executeSort(int[] collection) { /* High-performance processing logic */ }
}

class ContextExecutionManager {
    private SortingStrategy currentStrategy;
    
    public void updateStrategy(SortingStrategy targetConfig) { this.currentStrategy = targetConfig; }
    
    public void processData(int[] dataset) {
        currentStrategy.executeSort(dataset); // Strategy algorithm swapped fluidly at runtime
    }
}
```

#### 3. Command
Encapsulates an explicit action or data operation inside an independent, structured command object, providing native capabilities for request queuing and transaction playbacks.

```java
interface TransactionCommand { void executeAction(); }

class DatabaseBroker {
    public void executeCommit() { System.out.println("Writing database commit state."); }
}

class CommitTransaction implements TransactionCommand {
    private final DatabaseBroker dbBroker;
    public CommitTransaction(DatabaseBroker broker) { this.dbBroker = broker; }
    
    public void executeAction() {
        dbBroker.executeCommit();
    }
}

class CommandInvokerPipeline {
    private final List<TransactionCommand> commandLogQueue = new ArrayList<>();
    
    public void stageCommand(TransactionCommand dynamicCmd) {
        commandLogQueue.add(dynamicCmd);
    }
    
    public void processStagedPipelines() {
        for (TransactionCommand cmd : commandLogQueue) {
            cmd.executeAction();
        }
    }
}
