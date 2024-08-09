# C#-Code-With-SOLID-Design-Patterns
C# code example that fully adheres to the SOLID principles and incorporates multiple design patterns requires a well-structured and somewhat complex system. Below is an example of a simple e-commerce system where we demonstrate the SOLID principles and apply various design patterns.

1. Single Responsibility Principle (SRP): Each class should have only one responsibility.
2. Open/Closed Principle (OCP): Classes should be open for extension but closed for modification.
3. Liskov Substitution Principle (LSP): Subtypes must be substitutable for their base types.
4. Interface Segregation Principle (ISP): Clients should not be forced to depend on interfaces they do not use.
5. Dependency Inversion Principle (DIP): High-level modules should not depend on low-level modules but on abstractions.

Design Patterns Used:
Factory Pattern: For creating product instances.

Strategy Pattern: For applying different discount strategies.

Observer Pattern: For notifying users about order status changes.

Decorator Pattern: For adding features to products dynamically.

Repository Pattern: For data access abstraction.

Dependency Injection: For managing dependencies between classes.

Example: E-Commerce System

            using System;
            using System.Collections.Generic;
            
            // Product Class (SRP, Decorator Pattern)
            public abstract class Product
            {
                public string Name { get; set; }
                public decimal Price { get; set; }
            
                public Product(string name, decimal price)
                {
                    Name = name;
                    Price = price;
                }
            
                public abstract decimal GetPrice();
            }
            
            // Concrete Products
            public class BasicProduct : Product
            {
                public BasicProduct(string name, decimal price) : base(name, price) { }
            
                public override decimal GetPrice() => Price;
            }
            
            // Decorator Pattern for Product
            public abstract class ProductDecorator : Product
            {
                protected Product _product;
            
                public ProductDecorator(Product product) : base(product.Name, product.Price)
                {
                    _product = product;
                }
            
                public override decimal GetPrice() => _product.GetPrice();
            }
            
            public class GiftWrapDecorator : ProductDecorator
            {
                public GiftWrapDecorator(Product product) : base(product) { }
            
                public override decimal GetPrice() => _product.GetPrice() + 5.00m;
            }
            
            // Strategy Pattern for Discounts
            public interface IDiscountStrategy
            {
                decimal ApplyDiscount(decimal price);
            }
            
            public class NoDiscountStrategy : IDiscountStrategy
            {
                public decimal ApplyDiscount(decimal price) => price;
            }
            
            public class BlackFridayDiscountStrategy : IDiscountStrategy
            {
                public decimal ApplyDiscount(decimal price) => price * 0.8m;
            }
            
            // Factory Pattern for Product Creation
            public class ProductFactory
            {
                public static Product CreateProduct(string productName, decimal price, bool giftWrap = false)
                {
                    Product product = new BasicProduct(productName, price);
            
                    if (giftWrap)
                        product = new GiftWrapDecorator(product);
            
                    return product;
                }
            }
            
            // Observer Pattern for Order Notifications
            public interface IOrderObserver
            {
                void Update(string message);
            }
            
            public class Customer : IOrderObserver
            {
                public string Name { get; set; }
            
                public Customer(string name)
                {
                    Name = name;
                }
            
                public void Update(string message)
                {
                    Console.WriteLine($"{Name} received notification: {message}");
                }
            }
            
            public class Order
            {
                private List<IOrderObserver> _observers = new List<IOrderObserver>();
            
                public void AddObserver(IOrderObserver observer)
                {
                    _observers.Add(observer);
                }
            
                public void RemoveObserver(IOrderObserver observer)
                {
                    _observers.Remove(observer);
                }
            
                public void NotifyObservers(string message)
                {
                    foreach (var observer in _observers)
                    {
                        observer.Update(message);
                    }
                }
            
                public void ProcessOrder()
                {
                    NotifyObservers("Your order has been processed.");
                }
            }
            
            // Repository Pattern for Data Access
            public interface IProductRepository
            {
                void AddProduct(Product product);
                Product GetProduct(string productName);
            }
            
            public class ProductRepository : IProductRepository
            {
                private Dictionary<string, Product> _products = new Dictionary<string, Product>();
            
                public void AddProduct(Product product)
                {
                    _products[product.Name] = product;
                }
            
                public Product GetProduct(string productName)
                {
                    return _products.ContainsKey(productName) ? _products[productName] : null;
                }
            }
            
            // Dependency Injection for Order Processing
            public class OrderService
            {
                private readonly IProductRepository _productRepository;
                private readonly IDiscountStrategy _discountStrategy;
            
                public OrderService(IProductRepository productRepository, IDiscountStrategy discountStrategy)
                {
                    _productRepository = productRepository;
                    _discountStrategy = discountStrategy;
                }
            
                public void PlaceOrder(string productName)
                {
                    var product = _productRepository.GetProduct(productName);
            
                    if (product != null)
                    {
                        var finalPrice = _discountStrategy.ApplyDiscount(product.GetPrice());
                        Console.WriteLine($"Order placed for {product.Name} with final price {finalPrice:C}");
                    }
                    else
                    {
                        Console.WriteLine("Product not found.");
                    }
                }
            }
            
            // Main Program
            class Program
            {
                static void Main(string[] args)
                {
                    // Setup
                    IProductRepository productRepository = new ProductRepository();
                    IDiscountStrategy discountStrategy = new BlackFridayDiscountStrategy();
                    OrderService orderService = new OrderService(productRepository, discountStrategy);
            
                    // Create Products
                    Product product1 = ProductFactory.CreateProduct("Laptop", 1000.00m, true);
                    productRepository.AddProduct(product1);
            
                    Product product2 = ProductFactory.CreateProduct("Smartphone", 500.00m);
                    productRepository.AddProduct(product2);
            
                    // Place Orders
                    orderService.PlaceOrder("Laptop");
            
                    // Observer Pattern Usage
                    Order order = new Order();
                    Customer customer = new Customer("John Doe");
                    order.AddObserver(customer);
                    order.ProcessOrder();
                }
            }

   
Explanation:
Single Responsibility Principle (SRP): Each class has a single responsibility, like Product, Order, Customer, etc.
Open/Closed Principle (OCP): New product types, discounts, or observers can be added without modifying existing classes.
Liskov Substitution Principle (LSP): Derived classes like GiftWrapDecorator can be used wherever the base Product class is expected.
Interface Segregation Principle (ISP): Interfaces like IDiscountStrategy, IOrderObserver, and IProductRepository ensure that classes depend only on the methods they need.
Dependency Inversion Principle (DIP): High-level modules like OrderService depend on abstractions (IDiscountStrategy, IProductRepository), not on concrete implementations.


Design Patterns:
Factory Pattern: Used to create product instances with optional gift wrapping.
Decorator Pattern: Adds functionalities (e.g., gift wrapping) to products dynamically.
Strategy Pattern: Allows switching between different discount strategies.
Observer Pattern: Notifies customers of order status changes.
Repository Pattern: Abstracts data access for products.
Dependency Injection: Injects dependencies like IDiscountStrategy and IProductRepository into OrderService.
