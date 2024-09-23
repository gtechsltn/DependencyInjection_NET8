# Dependency Injection in .NET 8.0

Why Dependency Injection is Useful Even Without Mock Test Cases — .NET samples

https://medium.com/@siva.veeravarapu/why-dependency-injection-is-useful-even-without-mock-test-cases-net-samples-b74122ba89f8

## 1. Decoupling of Components: Flexibility in Code Changes

### Without DI:
```
public class HomeController : Controller
{
    private EmailService _emailService = new EmailService();

    public IActionResult SendEmail()
    {
        _emailService.SendEmail("user@example.com", "Welcome", "Thanks for registering.");
        return Ok("Email Sent!");
    }
}
```

### With DI:
```
public class HomeController : Controller
{
    private readonly IEmailService _emailService;

    public HomeController(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public IActionResult SendEmail()
    {
        _emailService.SendEmail("user@example.com", "Welcome", "Thanks for registering.");
        return Ok("Email Sent!");
    }
}
```

#### Program.cs
```
services.AddTransient<IEmailService, SmsService>(); // Replace EmailService with SmsService
```

## 2. Separation of Concerns: Managing Dependencies Across Modules
As your application grows, you might have multiple services that depend on each other. Without DI, you would need to instantiate each dependency manually, which clutters your code and makes your components tightly coupled.

### Without DI (Manually Creating Dependencies):
```
public class HomeController : Controller
{
    private EmailService _emailService;
    private LoggingService _loggingService;

    public HomeController()
    {
        _loggingService = new LoggingService();
        _emailService = new EmailService(_loggingService);
    }

    public IActionResult SendEmail()
    {
        _emailService.SendEmail("user@example.com", "Welcome", "Thanks for registering.");
        return Ok("Email Sent!");
    }
}
```

### With DI (Automatic Dependency Management):
```
public class HomeController : Controller
{
    private readonly IEmailService _emailService;

    public HomeController(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public IActionResult SendEmail()
    {
        _emailService.SendEmail("user@example.com", "Welcome", "Thanks for registering.");
        return Ok("Email Sent!");
    }
}
```

## 3. Lifetime Management: Optimized Resource Usage

### Without DI (Manual Singleton):
```
public class DatabaseService
{
    private static DatabaseService _instance;
    private static readonly object _lock = new object();

    private DatabaseService()
    {
        // Expensive initialization (e.g., opening a database connection)
    }

    public static DatabaseService GetInstance()
    {
        if (_instance == null)
        {
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = new DatabaseService();
                }
            }
        }
        return _instance;
    }
}
```

#### With DI (Built-in Singleton):
```
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<IDatabaseService, DatabaseService>(); // Singleton automatically managed
    }
}
```

## 4. Easier to Add Features and Extensions

### Without DI (Tightly Coupled):
```
public class HomeController : Controller
{
    private readonly EmailService _emailService;
    private readonly LoggingService _loggingService;

    public HomeController()
    {
        _emailService = new EmailService();
        _loggingService = new LoggingService();
    }

    public IActionResult SendEmail()
    {
        _loggingService.Log("Sending email...");
        _emailService.SendEmail("user@example.com", "Welcome", "Thanks for registering.");
        return Ok("Email Sent!");
    }
}
```

### With DI (Easily Extensible):
```
public class HomeController : Controller
{
    private readonly IEmailService _emailService;
    private readonly ILoggerService _loggerService;

    public HomeController(IEmailService emailService, ILoggerService loggerService)
    {
        _emailService = emailService;
        _loggerService = loggerService;
    }

    public IActionResult SendEmail()
    {
        _loggerService.Log("Sending email...");
        _emailService.SendEmail("user@example.com", "Welcome", "Thanks for registering.");
        return Ok("Email Sent!");
    }
}
```

#### Program.cs
```
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddTransient<ILoggerService, LoggerService>(); // Use DI to inject logger
    }
}
```

The logging feature is easily added without changing existing code that consumes the service, and it’s available throughout your application with minimal changes.

+ Advantage: DI allows you to extend functionality without overhauling your codebase, making your application more adaptable and maintainable.

# Final Thoughts:

While DI may seem like extra overhead in simple applications, it becomes extremely valuable in more complex applications because:

+ It reduces coupling, making your code easier to maintain, extend, and modify.
+ It simplifies object creation and manages lifetimes for you, reducing boilerplate code.
+ It encourages separation of concerns, leading to cleaner, more modular code.
+ It scales well with larger, more complex applications, reducing the risk of introducing bugs when adding or changing features.

Even if you’re not writing unit tests or mock cases, DI can dramatically simplify your development process and help future-proof your application.

# Conclusion:
These code snippets demonstrate how Dependency Injection (DI) improves loose coupling, maintainability, and flexibility in .NET Core applications without the need for unit testing. By configuring different service lifetimes (Transient, Scoped, Singleton) and injecting dependencies, you gain control over how objects are created and managed in your application, ensuring better scalability and maintainability.
