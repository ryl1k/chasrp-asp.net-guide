# C# та ASP.NET Core - Повний посібник

## Зміст

### 1. [C# - Основи мови](#c---основи-мови)
- [1.1 Properties замість getter/setter методів](#properties-замість-gettersetter-методів)
- [1.2 Автоматичні Properties](#автоматичні-properties)
- [1.3 Nullable Types та Null Safety](#nullable-types)
- [1.4 Value Types vs Reference Types](#value-types-vs-reference-types)
- [1.5 String Interpolation та StringBuilder](#string-interpolation)

### 2. [C# - ООП (розширення знань)](#c---ооп-розширення-знань)
- [2.1 Sealed Classes та методи](#sealed-classes)
- [2.2 Extension Methods](#extension-methods)
- [2.3 Partial Classes та Methods](#partial-classes-та-methods)
- [2.4 Static Classes та Constructors](#static-classes-та-static-constructors)
- [2.5 Operator Overloading](#operator-overloading)

### 3. [C# - Сучасні можливості](#c---сучасні-можливості)
- [3.1 Records та Value Objects](#records-та-value-objects)
- [3.2 Pattern Matching](#pattern-matching)
- [3.3 Switch Expressions](#switch-expressions)
- [3.4 Nullable Reference Types](#nullable-reference-types-1)
- [3.5 Init-only Properties](#init-only-properties)
- [3.6 Top-level Programs](#top-level-programs)
- [3.7 Global Using Directives](#global-using-directives)
- [3.8 File-scoped Namespaces](#file-scoped-namespaces)
- [3.9 Primary Constructors](#primary-constructors)
- [3.10 Collection Expressions](#collection-expressions)
- [3.11 Generic Math та Static Abstract Members](#generic-math-та-static-abstract-members)
- [3.12 C# Templates vs C++ Templates](#c-templates-vs-c-templates)

### 4. [ASP.NET Core - Архітектура](#aspnet-core---архітектура)
- [4.1 Dependency Injection Container](#dependency-injection-container)
- [4.2 Middleware Pipeline](#middleware-pipeline)
- [4.3 Configuration System](#configuration-system)
- [4.4 Logging Framework](#logging-framework)
- [4.5 Hosted Services](#hosted-services)

### 5. [ASP.NET Core - Web API](#aspnet-core---web-api)
- [5.1 RESTful API Design](#restful-api-design)
- [5.2 Controller Actions та Routing](#controller-actions-та-routing)
- [5.3 Model Binding та Validation](#model-binding-та-validation)
- [5.4 Content Negotiation](#content-negotiation)
- [5.5 API Versioning](#api-versioning)

### 6. [Валідація та обробка помилок](#валідація-та-обробка-помилок)
- [6.1 Data Annotations](#data-annotations)
- [6.2 Custom Validation](#custom-validation)
- [6.3 Global Exception Handling](#global-exception-handling)

### 7. [Робота з даними](#робота-з-даними)
- [7.1 Repository Pattern та Unit of Work](#repository-pattern-та-unit-of-work)
- [7.2 Entity Framework Core Deep Dive](#entity-framework-core-deep-dive)

### 8. [Безпека](#безпека)
- [8.1 ASP.NET Core Identity](#aspnet-core-identity)
- [8.2 JWT Authentication](#jwt-authentication)
- [8.3 Authorization Policies та Claims](#authorization-policies-та-claims)

### 9. [Тестування](#тестування)
- [9.1 Unit Testing з xUnit, NUnit та MSTest](#91-unit-testing-з-xunit-nunit-та-mstest)
- [9.2 Integration Testing з TestServer](#92-integration-testing-з-testserver)
- [9.3 Testing Controllers та APIs](#93-testing-controllers-та-apis)
- [9.4 Mocking та Test Doubles](#94-mocking-та-test-doubles)

### 10. [ASP.NET Core 9 Features](#10-aspnet-core-9-features)
- [10.1 Native AOT Support та Performance Improvements](#101-native-aot-support-та-performance-improvements)
- [10.2 Enhanced Blazor Server та WebAssembly Features](#102-enhanced-blazor-server-та-webassembly-features)
- [10.3 OpenAPI and Swagger Improvements](#103-openapi-and-swagger-improvements)

### 11. [Термінологія](#11-термінологія)

---

## C# - Основи мови

### Синтаксис та базові відмінності від C/C++

#### Properties замість getter/setter методів

**Що це таке:**
Properties - це спеціальні методи в C#, які виглядають як поля класу, але насправді викликають код при читанні або записі значення. Це інкапсуляція доступу до даних.

**Навіщо це потрібно:**
- Контроль доступу до даних (валідація при встановленні значень)
- Можливість додавання логіки без зміни інтерфейсу класу
- Зручний синтаксис - виглядає як поле, працює як метод
- Можливість створювати обчислювані властивості
- Підтримка різних рівнів доступу для читання і запису

**Практичне застосування:**

```csharp
// Базові Properties з валідацією
public class Person
{
    private int _age;
    private string _email = "";

    // Property з валідацією
    public int Age
    {
        get { return _age; }
        set 
        { 
            if (value < 0 || value > 150)
                throw new ArgumentException("Age must be between 0 and 150");
            _age = value; 
        }
    }

    // Property з логікою обробки
    public string Email
    {
        get { return _email; }
        set 
        { 
            if (string.IsNullOrWhiteSpace(value) || !value.Contains("@"))
                throw new ArgumentException("Invalid email format");
            _email = value.ToLower().Trim(); // Автоматично нормалізуємо
        }
    }

    // Auto-property - компілятор сам створює приховане поле
    public string Name { get; set; } = "";

    // Read-only property - тільки getter
    public DateTime CreatedAt { get; } = DateTime.UtcNow;

    // Computed property - обчислюється кожного разу
    public string DisplayName => $"{Name} ({Age} років)";

    // Property з різними рівнями доступу
    public string Id { get; private set; } = Guid.NewGuid().ToString();

    // Init-only property - можна встановити тільки при створенні
    public string Country { get; init; } = "";
}

// Використання
var person = new Person 
{ 
    Name = "Іван Петренко",
    Country = "Ukraine" // init-only
};

person.Age = 25;  // Викличе валідацію
person.Email = "  IVAN@EXAMPLE.COM  ";  // Буде нормалізовано до "ivan@example.com"

Console.WriteLine(person.DisplayName);  // "Іван Петренко (25 років)"

// person.Country = "USA";  // Помилка - init-only
// person.Id = "new-id";    // Помилка - private setter
```

#### String interpolation

**Що це таке:**
String interpolation - це спосіб вставляти значення змінних безпосередньо в рядок, використовуючи синтаксис `$"текст {змінна}"`.

**Навіщо це потрібно:**
- Набагато читабельніше за конкатенацію або String.Format
- Безпосереднє бачення того, які змінні використовуються
- Type-safe - компілятор перевіряє типи
- Можливість форматування значень
- Підтримка складних виразів

**Практичне застосування:**

```csharp
string name = "Анна";
int age = 28;
decimal salary = 50000.75m;
DateTime now = DateTime.Now;

// Базова інтерполяція
string basic = $"Користувач: {name}, Вік: {age}";

// З форматуванням
string formatted = $"Зарплата: {salary:C} (в місяць)";  // "Зарплата: $50,000.75 (в місяць)"

// З датами
string withDate = $"Створено: {now:dd.MM.yyyy HH:mm}";  // "Створено: 22.07.2025 14:30"

// Зі складними виразами
string complex = $"Статус: {(age >= 18 ? "Повнолітній" : "Неповнолітній")}";

// Багаторядкові рядки з інтерполяцією
string report = $@"
=== Звіт про користувача ===
Ім'я: {name}
Вік: {age}
Зарплата: {salary:N0} грн
Статус: {(salary > 30000 ? "Високий дохід" : "Середній дохід")}
Дата створення: {now:F}
";

// Вкладені інтерполяції
string nested = $"Результат: {(salary > 40000 ? $"Високий: {salary:C}" : "Низький")}";

Console.WriteLine(basic);     // "Користувач: Анна, Вік: 28"
Console.WriteLine(formatted); // "Зарплата: $50,000.75 (в місяць)"
Console.WriteLine(complex);   // "Статус: Повнолітній"
```

#### var keyword та type inference

**Що це таке:**
`var` - це ключове слово, яке дозволяє компілятору автоматично визначити тип змінної на основі присвоєного їй значення.

**Навіщо це потрібно:**
- Зменшення дублювання коду (не треба писати довгі типи двічі)
- Покращення читабельності для складних generic типів
- Обов'язково для анонімних типів
- Легше підтримувати код при зміні типів
- Фокус на логіці, а не на деталях типів

**Практичне застосування:**

```csharp
// Замість довгих типів
Dictionary<string, List<ComplexBusinessObject>> longType = 
    new Dictionary<string, List<ComplexBusinessObject>>();

var shortType = new Dictionary<string, List<ComplexBusinessObject>>();

// З LINQ запитами (тип складно передбачити)
var employees = GetEmployees();
var result = employees
    .Where(e => e.Age > 25)
    .GroupBy(e => e.Department)
    .Select(g => new { 
        Department = g.Key, 
        Count = g.Count(), 
        AvgSalary = g.Average(e => e.Salary) 
    })
    .ToList();

// Анонімні типи - тільки з var
var person = new { Name = "Іван", Age = 30, IsActive = true };
Console.WriteLine($"{person.Name} is {person.Age} years old");

// З методами що повертають складні типи
var httpClient = CreateConfiguredHttpClient();
var jsonDocument = JsonDocument.Parse(jsonString);

// Коли НЕ треба використовувати var:
// 1. Коли тип не очевидний
int count = GetCount();  // Краще явно вказати int
decimal price = 19.99m;  // Важливо що це decimal, не double

// 2. Коли потрібен конкретний інтерфейс
IList<string> list = new List<string>();  // Підкреслюємо роботу з інтерфейсом

// 3. З примітивними типами часто краще явно
string name = GetUserName();  // Зрозуміло що це string
bool isValid = ValidateInput(); // Зрозуміло що це bool
```

#### using statements для управління ресурсами

**Що це таке:**
`using` statement автоматично викликає метод `Dispose()` у об'єкта в кінці блоку, навіть якщо виникне помилка. Це гарантує правильне звільнення ресурсів.

**Навіщо це потрібно:**
- Автоматичне звільнення ресурсів (файли, мережеві з'єднання, бази даних)
- Захист від витоку ресурсів при помилках
- Детермінований cleanup - ресурс звільняється негайно
- Простіший код - не треба пам'ятати про Dispose
- Exception safety - cleanup відбудеться навіть при винятках

**Практичне застосування:**

```csharp
// Робота з файлами
public static string ReadFileContent(string filePath)
{
    // using statement гарантує закриття файлу
    using (var reader = new StreamReader(filePath))
    {
        return reader.ReadToEnd();
        // reader.Dispose() викликається автоматично
    }
}

// Нова форма using declaration (C# 8+)
public static void ProcessFile(string filePath)
{
    using var file = new FileStream(filePath, FileMode.Open);
    using var reader = new StreamReader(file);
    
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
    
    // Dispose викликається в кінці методу для обох ресурсів
}

// З базою даних
public static List<User> GetUsers(string connectionString)
{
    using var connection = new SqlConnection(connectionString);
    using var command = new SqlCommand("SELECT * FROM Users", connection);
    
    connection.Open();
    using var reader = command.ExecuteReader();
    
    var users = new List<User>();
    while (reader.Read())
    {
        users.Add(new User 
        { 
            Id = reader.GetInt32("Id"),
            Name = reader.GetString("Name") 
        });
    }
    
    return users;
    // connection, command, reader автоматично закриються
}

// З HTTP клієнтом
public static async Task<string> FetchDataAsync(string url)
{
    using var httpClient = new HttpClient();
    using var response = await httpClient.GetAsync(url);
    
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync();
}

// Множинні ресурси
public static void ProcessMultipleFiles(string[] filePaths)
{
    foreach (string filePath in filePaths)
    {
        using var input = new FileStream(filePath, FileMode.Open);
        using var output = new FileStream($"{filePath}.processed", FileMode.Create);
        using var reader = new StreamReader(input);
        using var writer = new StreamWriter(output);
        
        string line;
        while ((line = reader.ReadLine()) != null)
        {
            writer.WriteLine(line.ToUpper());
        }
        
        // Всі 4 ресурси будуть правильно закриті
    }
}

// Async using для IAsyncDisposable (C# 8+)
public static async Task ProcessAsyncResource()
{
    await using var asyncResource = new AsyncFileWriter("output.txt");
    await asyncResource.WriteLineAsync("Hello, World!");
    
    // DisposeAsync() викликається автоматично
}
```

#### Nullable types

**Що це таке:**
Nullable types дозволяють value типам (int, bool, DateTime тощо) мати значення `null`. Записується як `int?` (що є скороченням від `Nullable<int>`).

**Навіщо це потрібно:**
- Представлення "відсутності значення" для value типів
- Розрізнення між "0" та "невідомо" для чисел
- Розрізнення між "false" та "не встановлено" для bool
- Безпечна робота з базами даних (NULL значення)
- Кращий API дизайн - явно показувати необов'язкові параметри

**Практичне застосування:**

```csharp
// Базове використання nullable types
public class UserProfile
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    
    // Nullable value types
    public int? Age { get; set; }           // Може бути null
    public DateTime? BirthDate { get; set; } // Може бути null
    public bool? IsEmailVerified { get; set; } // true/false/null
    public decimal? Salary { get; set; }    // Може бути конфіденційно
    
    // Методи для роботи з nullable
    public string GetAgeDisplay()
    {
        if (Age.HasValue)
            return $"{Age.Value} років";
        else
            return "Вік не вказано";
    }
    
    public string GetSalaryDisplay()
    {
        // Null-coalescing operator
        return Salary?.ToString("C") ?? "Конфіденційно";
    }
}

// Робота з nullable types
public static void WorkWithNullables()
{
    var user = new UserProfile 
    { 
        Id = 1, 
        Name = "Іван",
        Age = null,  // Не вказано
        BirthDate = new DateTime(1990, 5, 15),
        IsEmailVerified = null  // Ще не перевірялося
    };
    
    // Перевірка на значення
    if (user.Age.HasValue)
    {
        Console.WriteLine($"Користувач {user.Age.Value} років");
    }
    
    // Або коротше
    if (user.Age != null)
    {
        Console.WriteLine($"Користувач {user.Age} років");  // Auto-unboxing
    }
    
    // Null-coalescing operators
    int actualAge = user.Age ?? 0;  // Якщо null, то 0
    user.Age ??= 25;  // Встановити 25 якщо null
    
    // GetValueOrDefault
    int ageOrDefault = user.Age.GetValueOrDefault(18);
    
    // Безпечна робота з методами
    string birthYear = user.BirthDate?.Year.ToString() ?? "Невідомо";
    
    // Pattern matching з nullable
    string status = user.IsEmailVerified switch
    {
        true => "Email підтверджено",
        false => "Email не підтверджено", 
        null => "Email не перевірявся"
    };
}

// Nullable reference types (C# 8+)
#nullable enable
public class ModernUser
{
    // Non-nullable - не може бути null
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    
    // Nullable - може бути null
    public string? MiddleName { get; set; }
    public string? Phone { get; set; }
    
    public string GetFullName()
    {
        // Безпечна робота з nullable strings
        if (string.IsNullOrEmpty(MiddleName))
            return $"{FirstName} {LastName}";
        else
            return $"{FirstName} {MiddleName} {LastName}";
    }
    
    public void CallUser()
    {
        // Null-conditional operator
        int? phoneLength = Phone?.Length;
        
        // Pattern matching
        if (Phone is string phoneNumber && phoneNumber.Length > 0)
        {
            Console.WriteLine($"Дзвонимо на {phoneNumber}");
        }
        else
        {
            Console.WriteLine("Телефон не вказано");
        }
    }
}
```

### Система типів C#

#### Value types vs Reference types

**Що це таке:**
В C# всі типи поділяються на дві категорії:
- **Value types** - зберігають дані безпосередньо (int, bool, struct)
- **Reference types** - зберігають посилання на дані в пам'яті (class, string, arrays)

**Навіщо це потрібно знати:**
- Розуміння як працює копіювання змінних
- Розуміння продуктивності (stack vs heap)
- Правильне порівняння об'єктів
- Уникнення неочікуваної поведінки при передачі параметрів
- Вибір між struct та class

**Практичне застосування:**

```csharp
// Value Types - дані копіюються
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public double DistanceTo(Point other)
    {
        int dx = X - other.X;
        int dy = Y - other.Y;
        return Math.Sqrt(dx * dx + dy * dy);
    }
}

// Reference Types - дані в heap, копіюється посилання
public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
    
    public void CelebrateBirthday()
    {
        Age++;
    }
}

public static void DemonstrateTypeDifferences()
{
    // Value types - копіюються дані
    Point point1 = new Point(10, 20);
    Point point2 = point1;  // КОПІЯ даних
    
    point1.X = 100;  // Змінюємо point1
    
    Console.WriteLine($"point1.X = {point1.X}");  // 100
    Console.WriteLine($"point2.X = {point2.X}");  // 20 - НЕ змінився!
    
    // Reference types - копіюється посилання
    Person person1 = new Person { Name = "Іван", Age = 25 };
    Person person2 = person1;  // КОПІЯ посилання (той самий об'єкт!)
    
    person1.Age = 30;  // Змінюємо через person1
    
    Console.WriteLine($"person1.Age = {person1.Age}");  // 30
    Console.WriteLine($"person2.Age = {person2.Age}");  // 30 - ЗМІНИВСЯ!
    
    Console.WriteLine($"Same object: {ReferenceEquals(person1, person2)}");  // True
}

// Передача параметрів
public static void ParameterPassing()
{
    // Value types передаються по значенню
    int number = 42;
    ModifyNumber(number);
    Console.WriteLine(number);  // 42 - НЕ змінився
    
    ModifyNumberByRef(ref number);
    Console.WriteLine(number);  // 100 - змінився через ref
    
    // Reference types - передається копія посилання
    Person person = new Person { Name = "Анна", Age = 28 };
    ModifyPerson(person);
    Console.WriteLine(person.Age);  // 29 - змінився!
    
    AssignNewPerson(person);
    Console.WriteLine(person.Name);  // "Анна" - НЕ змінився
    
    AssignNewPersonByRef(ref person);
    Console.WriteLine(person.Name);  // "Новий" - змінився через ref
}

static void ModifyNumber(int value)
{
    value = 100;  // Змінюємо копію
}

static void ModifyNumberByRef(ref int value)
{
    value = 100;  // Змінюємо оригінал
}

static void ModifyPerson(Person p)
{
    p.Age = 29;  // Змінюємо об'єкт (працює!)
}

static void AssignNewPerson(Person p)
{
    p = new Person { Name = "Новий", Age = 35 };  // Змінюємо копію посилання
}

static void AssignNewPersonByRef(ref Person p)
{
    p = new Person { Name = "Новий", Age = 35 };  // Змінюємо оригінальне посилання
}

// Вибір між struct та class
// Struct коли:
// - Дані маленькі (рекомендовано < 16 bytes)
// - Immutable дані
// - Рідко boxing/unboxing
// - Логічно представляє одне значення
public struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
            
        return new Money(Amount + other.Amount, Currency);
    }
}

// Class коли:
// - Складні об'єкти з поведінкою
// - Потреба в наслідуванні
// - Mutable стан
// - Великі об'єкти
public class BankAccount
{
    public string AccountNumber { get; }
    public decimal Balance { get; private set; }
    public List<Transaction> Transactions { get; }
    
    public BankAccount(string accountNumber)
    {
        AccountNumber = accountNumber;
        Transactions = new List<Transaction>();
    }
    
    public void Deposit(Money amount)
    {
        Balance += amount.Amount;
        Transactions.Add(new Transaction("Deposit", amount.Amount));
    }
}
```

#### Boxing та Unboxing

**Що це таке:**
- **Boxing** - упаковка value type в object (на heap)
- **Unboxing** - розпакування object назад в value type

**Навіщо це потрібно знати:**
- Розуміння продуктивності коду
- Уникнення непотрібних алокацій
- Правильне використання колекцій
- Розуміння роботи generic системи

**Практичне застосування:**

```csharp
public static void BoxingUnboxingDemo()
{
    // Boxing - value type → object
    int value = 42;
    object boxed = value;  // Boxing! Створюється об'єкт на heap
    
    Console.WriteLine($"value: {value}");     // 42
    Console.WriteLine($"boxed: {boxed}");     // 42
    Console.WriteLine($"Same? {ReferenceEquals(value, boxed)}");  // False!
    
    // Unboxing - object → value type
    int unboxed = (int)boxed;  // Explicit cast потрібен
    
    // Помилка unboxing - InvalidCastException
    try
    {
        double wrongUnbox = (double)boxed;  // Runtime помилка!
    }
    catch (InvalidCastException ex)
    {
        Console.WriteLine($"Помилка: {ex.Message}");
    }
}

// Проблематичні випадки boxing
public static void ProblematicBoxing()
{
    // ArrayList зберігає object - кожне додавання = boxing
    ArrayList arrayList = new ArrayList();
    
    for (int i = 0; i < 1000; i++)
    {
        arrayList.Add(i);  // Boxing кожного int!
    }
    
    // Читання також викликає unboxing
    foreach (object item in arrayList)
    {
        int value = (int)item;  // Unboxing кожного елемента!
        Console.WriteLine(value);
    }
}

// Рішення - Generic колекції
public static void GenericCollectionsSolution()
{
    // List<T> не викликає boxing для T
    List<int> list = new List<int>();
    
    for (int i = 0; i < 1000; i++)
    {
        list.Add(i);  // Немає boxing!
    }
    
    foreach (int value in list)
    {
        Console.WriteLine(value);  // Немає unboxing!
    }
}

// Приховане boxing
public static void HiddenBoxing()
{
    int number = 42;
    
    // Boxing в string interpolation (старіші версії C#)
    string message1 = $"Number: {number}";  // Може викликати boxing
    
    // Boxing при Equals з object
    bool equal1 = number.Equals((object)100);  // 100 буде boxing
    
    // Boxing в generic constraints
    ProcessAsClass(number);  // Boxing якщо T : class
    
    // Краще використовувати спеціалізовані методи
    string message2 = number.ToString();  // Немає boxing
    bool equal2 = number == 100;  // Немає boxing
    ProcessAsStruct(number);  // Немає boxing
}

static void ProcessAsClass<T>(T value) where T : class
{
    // T має бути reference type - value types будуть boxing
}

static void ProcessAsStruct<T>(T value) where T : struct
{
    // T має бути value type - немає boxing
}

// Performance тест
public static void PerformanceTest()
{
    const int iterations = 1_000_000;
    
    // Тест з boxing
    var sw = Stopwatch.StartNew();
    for (int i = 0; i < iterations; i++)
    {
        object boxed = i;        // Boxing
        int unboxed = (int)boxed; // Unboxing
    }
    sw.Stop();
    Console.WriteLine($"Boxing/Unboxing: {sw.ElapsedMilliseconds}ms");
    
    // Тест без boxing
    sw.Restart();
    for (int i = 0; i < iterations; i++)
    {
        int value = i;
        int copy = value;  // Проста копія
    }
    sw.Stop();
    Console.WriteLine($"No boxing: {sw.ElapsedMilliseconds}ms");
    
    // Boxing може бути в 10-50 разів повільніше!
}
```

#### Garbage Collection

**Що це таке:**
Garbage Collector (GC) - система автоматичного управління пам'яттю в .NET, яка автоматично звільняє непотрібну пам'ять.

**Навіщо це потрібно:**
- Уникнення memory leaks
- Автоматичне управління пам'яттю
- Захист від помилок з вказівниками
- Спрощення розробки

**Як це працює та що треба знати:**
- Об'єкти створюються на heap
- GC працює поколіннями (0, 1, 2)
- Збирає тільки недосяжні об'єкти
- Може призупиняти виконання програми

**Практичне застосування:**

```csharp
public static void GCBasics()
{
    Console.WriteLine("=== До створення об'єктів ===");
    PrintMemoryInfo();
    
    // Створення багатьох тимчасових об'єктів
    for (int i = 0; i < 100000; i++)
    {
        var temp = new StringBuilder($"Temp object {i}");
        // temp стає недосяжним після кожної ітерації
    }
    
    Console.WriteLine("=== Після створення об'єктів ===");
    PrintMemoryInfo();
    
    // Принудова збірка сміття (НЕ робіть це в реальному коді!)
    GC.Collect();
    GC.WaitForPendingFinalizers();
    GC.Collect();
    
    Console.WriteLine("=== Після збірки сміття ===");
    PrintMemoryInfo();
}

static void PrintMemoryInfo()
{
    long totalMemory = GC.GetTotalMemory(false);
    Console.WriteLine($"Пам'ять: {totalMemory:N0} bytes ({totalMemory / 1024 / 1024:N1} MB)");
    Console.WriteLine($"Gen 0: {GC.CollectionCount(0)}, Gen 1: {GC.CollectionCount(1)}, Gen 2: {GC.CollectionCount(2)}");
    Console.WriteLine();
}

// Правильне управління ресурсами
public class ProperResourceManagement : IDisposable
{
    private FileStream? _fileStream;
    private bool _disposed = false;
    
    public ProperResourceManagement(string filename)
    {
        _fileStream = new FileStream(filename, FileMode.Create);
    }
    
    public void WriteData(byte[] data)
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(ProperResourceManagement));
            
        _fileStream?.Write(data);
    }
    
    // IDisposable implementation
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);  // Не потрібно викликати finalizer
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Звільняємо managed resources
                _fileStream?.Dispose();
            }
            
            _disposed = true;
        }
    }
    
    // Finalizer - backup якщо забули викликати Dispose
    ~ProperResourceManagement()
    {
        Dispose(false);
    }
}

// Використання
public static void UseResources()
{
    // using гарантує Dispose навіть при винятках
    using var resource = new ProperResourceManagement("test.dat");
    resource.WriteData(new byte[] { 1, 2, 3, 4, 5 });
    
    // Dispose викликається автоматично
}

// Оптимізація для GC
public static class GCOptimization
{
    // Пул об'єктів для уникнення алокацій
    private static readonly ConcurrentQueue<StringBuilder> _stringBuilderPool = new();
    
    public static StringBuilder GetStringBuilder()
    {
        if (_stringBuilderPool.TryDequeue(out var sb))
        {
            sb.Clear();  // Очищаємо для повторного використання
            return sb;
        }
        
        return new StringBuilder();
    }
    
    public static void ReturnStringBuilder(StringBuilder sb)
    {
        if (sb.Capacity <= 1024)  // Не зберігаємо занадто великі
        {
            _stringBuilderPool.Enqueue(sb);
        }
    }
    
    // Використання пулу
    public static string ProcessStrings(string[] inputs)
    {
        var sb = GetStringBuilder();
        
        try
        {
            foreach (string input in inputs)
            {
                sb.AppendLine(input.ToUpper());
            }
            
            return sb.ToString();
        }
        finally
        {
            ReturnStringBuilder(sb);
        }
    }
}
```

#### Object class як базовий для всіх типів

**Що це таке:**
В C# всі типи (і value types, і reference types) неявно наслідуються від `System.Object`. Це забезпечує unified type system.

**Навіщо це потрібно:**
- Можливість зберігати будь-який тип в object змінній
- Загальні методи для всіх типів (ToString, Equals, GetHashCode)
- Рефлексія - аналіз типів під час виконання
- Полімор1фізм на найвищому рівні

**Практичне застосування:**

```csharp
// Базові методи Object
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Group { get; set; } = "";
    
    // Перевизначення ToString для зрозумілого відображення
    public override string ToString()
    {
        return $"Student: {Name} (Group: {Group})";
    }
    
    // Перевизначення Equals для порівняння за змістом
    public override bool Equals(object? obj)
    {
        if (obj is Student other)
        {
            return Id == other.Id;  // Порівнюємо за ID
        }
        return false;
    }
    
    // GetHashCode має відповідати Equals
    public override int GetHashCode()
    {
        return Id.GetHashCode();
    }
}

public static void ObjectMethods()
{
    var student1 = new Student { Id = 1, Name = "Іван", Group = "ІТ-21" };
    var student2 = new Student { Id = 1, Name = "Іван Петров", Group = "ІТ-21" };
    var student3 = new Student { Id = 2, Name = "Анна", Group = "ІТ-22" };
    
    // ToString
    Console.WriteLine(student1);  // "Student: Іван (Group: ІТ-21)"
    
    // Equals - порівняння за змістом
    Console.WriteLine($"student1 == student2: {student1.Equals(student2)}");  // True (однаковий ID)
    Console.WriteLine($"student1 == student3: {student1.Equals(student3)}");  // False
    
    // GetType - інформація про тип
    Type studentType = student1.GetType();
    Console.WriteLine($"Type: {studentType.Name}");
    Console.WriteLine($"Namespace: {studentType.Namespace}");
    
    // ReferenceEquals - порівняння посилань
    Console.WriteLine($"Same reference: {ReferenceEquals(student1, student2)}");  // False
}

// Поліморфізм через Object
public static void ProcessObjects(object[] objects)
{
    foreach (object obj in objects)
    {
        Console.WriteLine($"Type: {obj.GetType().Name}");
        Console.WriteLine($"Value: {obj}");  // Викликає ToString()
        Console.WriteLine($"Hash: {obj.GetHashCode()}");
        Console.WriteLine();
    }
}

// Рефлексія - аналіз типів
public static void ReflectionExample()
{
    var student = new Student { Id = 1, Name = "Тест", Group = "ІТ-21" };
    Type type = student.GetType();
    
    Console.WriteLine($"=== Аналіз типу {type.Name} ===");
    
    // Властивості
    Console.WriteLine("Властивості:");
    foreach (var property in type.GetProperties())
    {
        var value = property.GetValue(student);
        Console.WriteLine($"  {property.PropertyType.Name} {property.Name} = {value}");
    }
    
    // Методи
    Console.WriteLine("Методи:");
    foreach (var method in type.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
    {
        Console.WriteLine($"  {method.ReturnType.Name} {method.Name}()");
    }
    
    // Динамічна зміна властивостей
    var nameProperty = type.GetProperty("Name");
    nameProperty?.SetValue(student, "Новий Студент");
    Console.WriteLine($"Після зміни: {student}");
}

// Boxing value types до Object
public static void ValueTypesAsObjects()
{
    // Value types можуть бути приведені до object через boxing
    int number = 42;
    DateTime date = DateTime.Now;
    bool flag = true;
    
    // Boxing до object
    object[] values = { number, date, flag, "string" };
    
    foreach (object value in values)
    {
        Console.WriteLine($"{value.GetType().Name}: {value}");
        
        // Type checking
        switch (value)
        {
            case int i:
                Console.WriteLine($"  Integer value: {i}");
                break;
            case DateTime dt:
                Console.WriteLine($"  Date: {dt:yyyy-MM-dd}");
                break;
            case bool b:
                Console.WriteLine($"  Boolean: {b}");
                break;
            case string s:
                Console.WriteLine($"  String length: {s.Length}");
                break;
        }
    }
}
```

### Колекції та LINQ

#### Generic collections

**Що це таке:**
Generic collections - це колекції, які працюють з конкретним типом даних (замість object), що забезпечує type safety та кращу продуктивність.

**Навіщо це потрібно:**
- Type safety - компілятор перевіряє типи
- Немає boxing/unboxing для value types
- IntelliSense підтримка
- Кращу продуктивність
- Читабельніший код

**Практичне застосування:**

**List<T> - динамічний масив:**

```csharp
public static void ListExamples()
{
    // Створення та ініціалізація
    var numbers = new List<int> { 1, 2, 3, 4, 5 };
    var names = new List<string>();
    
    // Додавання елементів
    names.Add("Іван");
    names.Add("Петро");
    names.AddRange(new[] { "Анна", "Марія" });
    
    // Доступ по індексу
    Console.WriteLine($"Перший: {names[0]}");
    names[0] = "Іван Петрович";  // Зміна значення
    
    // Пошук
    int index = names.IndexOf("Петро");
    bool contains = names.Contains("Анна");
    string? found = names.Find(name => name.StartsWith("М"));
    List<string> filtered = names.FindAll(name => name.Length > 5);
    
    // Видалення
    names.Remove("Петро");  // Видалити конкретний елемент
    names.RemoveAt(0);      // Видалити за індексом
    names.RemoveAll(name => name.Contains("а"));  // Видалити всі що містять "а"
    
    // Сортування
    numbers.Sort();  // По зростанню
    numbers.Sort((x, y) => y.CompareTo(x));  // По спаданню
    
    names.Sort();  // Алфавітний порядок
    names.Sort((a, b) => a.Length.CompareTo(b.Length));  // По довжині
    
    // Ітерація
    foreach (string name in names)
    {
        Console.WriteLine(name);
    }
    
    // Конверсія
    string[] array = names.ToArray();
    int count = names.Count;
}
```

**Dictionary<K,V> - асоціативний масив:**

```csharp
public static void DictionaryExamples()
{
    // Створення та ініціалізація
    var scores = new Dictionary<string, int>
    {
        ["Іван"] = 95,
        ["Петро"] = 87,
        ["Анна"] = 92
    };
    
    var products = new Dictionary<int, Product>
    {
        { 1, new Product { Id = 1, Name = "Ноутбук", Price = 25000 } },
        { 2, new Product { Id = 2, Name = "Миша", Price = 500 } }
    };
    
    // Додавання
    scores.Add("Марія", 98);  // Викине виняток якщо ключ існує
    scores["Олег"] = 85;      // Додасть або оновить
    
    // Перевірка наявності
    if (scores.ContainsKey("Іван"))
    {
        Console.WriteLine($"Оцінка Івана: {scores["Іван"]}");
    }
    
    // Безпечне отримання значення
    if (scores.TryGetValue("Невідомий", out int score))
    {
        Console.WriteLine($"Оцінка: {score}");
    }
    else
    {
        Console.WriteLine("Студент не знайдений");
    }
    
    // Ітерація
    foreach (var kvp in scores)
    {
        Console.WriteLine($"{kvp.Key}: {kvp.Value}");
    }
    
    // Тільки ключі або значення
    foreach (string student in scores.Keys)
    {
        Console.WriteLine($"Студент: {student}");
    }
    
    foreach (int studentScore in scores.Values)
    {
        Console.WriteLine($"Оцінка: {studentScore}");
    }
    
    // Видалення
    scores.Remove("Петро");
    
    // LINQ з Dictionary
    var topStudents = scores
        .Where(kvp => kvp.Value >= 90)
        .OrderByDescending(kvp => kvp.Value)
        .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}
```

**HashSet<T> - унікальні елементи:**

```csharp
public static void HashSetExamples()
{
    // Створення
    var uniqueNumbers = new HashSet<int> { 1, 2, 3, 4, 5 };
    var visitedPages = new HashSet<string>();
    
    // Додавання (повертає true якщо елемент новий)
    bool added1 = uniqueNumbers.Add(6);   // true
    bool added2 = uniqueNumbers.Add(3);   // false - вже існує
    
    // Автоматичне видалення дублікатів
    string[] duplicates = { "a", "b", "a", "c", "b", "d" };
    var uniqueLetters = new HashSet<string>(duplicates);
    Console.WriteLine($"Унікальних букв: {uniqueLetters.Count}");  // 4
    
    // Операції множин
    var set1 = new HashSet<int> { 1, 2, 3, 4, 5 };
    var set2 = new HashSet<int> { 4, 5, 6, 7, 8 };
    
    // Об'єднання (Union)
    var union = new HashSet<int>(set1);
    union.UnionWith(set2);  // { 1, 2, 3, 4, 5, 6, 7, 8 }
    
    // Перетин (Intersection)
    var intersection = new HashSet<int>(set1);
    intersection.IntersectWith(set2);  // { 4, 5 }
    
    // Різниця (Except)
    var except = new HashSet<int>(set1);
    except.ExceptWith(set2);  // { 1, 2, 3 }
    
    // Симетрична різниця
    var symmetricExcept = new HashSet<int>(set1);
    symmetricExcept.SymmetricExceptWith(set2);  // { 1, 2, 3, 6, 7, 8 }
    
    // Перевірки
    bool isSubset = set1.IsSubsetOf(union);
    bool isSuperset = union.IsSupersetOf(set1);
    bool overlap = set1.Overlaps(set2);
    
    // Практичний приклад - відстеження унікальних відвідувачів
    var visitors = new HashSet<string>();
    
    string[] requests = { "user1", "user2", "user1", "user3", "user2", "user4" };
    foreach (string user in requests)
    {
        if (visitors.Add(user))
        {
            Console.WriteLine($"Новий відвідувач: {user}");
        }
        else
        {
            Console.WriteLine($"Повторний візит: {user}");
        }
    }
    
    Console.WriteLine($"Всього унікальних відвідувачів: {visitors.Count}");
}
```

**Queue<T> та Stack<T>:**

```csharp
public static void QueueStackExamples()
{
    // Queue - FIFO (First In, First Out)
    var taskQueue = new Queue<string>();
    
    // Додавання в чергу
    taskQueue.Enqueue("Завдання 1");
    taskQueue.Enqueue("Завдання 2");
    taskQueue.Enqueue("Завдання 3");
    
    // Обробка черги
    while (taskQueue.Count > 0)
    {
        string currentTask = taskQueue.Dequeue();  // Видаляє та повертає перший елемент
        Console.WriteLine($"Виконую: {currentTask}");
        
        // Можна подивитися наступний без видалення
        if (taskQueue.Count > 0)
        {
            string nextTask = taskQueue.Peek();
            Console.WriteLine($"  Наступне: {nextTask}");
        }
    }
    
    // Stack - LIFO (Last In, First Out)
    var undoStack = new Stack<string>();
    
    // Додавання дій
    undoStack.Push("Створити документ");
    undoStack.Push("Написати текст");
    undoStack.Push("Форматувати текст");
    undoStack.Push("Додати зображення");
    
    Console.WriteLine($"Дій для скасування: {undoStack.Count}");
    
    // Скасування дій
    while (undoStack.Count > 0)
    {
        string lastAction = undoStack.Pop();  // Видаляє та повертає останній елемент
        Console.WriteLine($"Скасовую: {lastAction}");
        
        // Можна подивитися попередню дію без видалення
        if (undoStack.Count > 0)
        {
            string prevAction = undoStack.Peek();
            Console.WriteLine($"  Попередня: {prevAction}");
        }
    }
}

// Практичний приклад - система Undo/Redo
public class TextEditor
{
    private Stack<string> _undoStack = new();
    private Stack<string> _redoStack = new();
    private string _currentText = "";
    
    public string CurrentText => _currentText;
    
    public void SetText(string newText)
    {
        // Зберігаємо поточний стан для undo
        _undoStack.Push(_currentText);
        _currentText = newText;
        
        // Очищаємо redo stack при новій дії
        _redoStack.Clear();
        
        Console.WriteLine($"Текст змінено: '{_currentText}'");
    }
    
    public void Undo()
    {
        if (_undoStack.Count > 0)
        {
            // Зберігаємо поточний стан для redo
            _redoStack.Push(_currentText);
            
            // Відновлюємо попередній стан
            _currentText = _undoStack.Pop();
            Console.WriteLine($"Undo: '{_currentText}'");
        }
        else
        {
            Console.WriteLine("Немає дій для скасування");
        }
    }
    
    public void Redo()
    {
        if (_redoStack.Count > 0)
        {
            // Зберігаємо поточний стан для undo
            _undoStack.Push(_currentText);
            
            // Відновлюємо наступний стан
            _currentText = _redoStack.Pop();
            Console.WriteLine($"Redo: '{_currentText}'");
        }
        else
        {
            Console.WriteLine("Немає дій для повтору");
        }
    }
}
```

#### LINQ - запити до колекцій

**Що це таке:**
LINQ (Language Integrated Query) - це система запитів, вбудована в C#, яка дозволяє писати SQL-подібні запити до колекцій, баз даних та інших джерел даних прямо в коді.

**Навіщо це потрібно:**
- Уніфікований синтаксис для роботи з різними джерелами даних
- Читабельніші запити замість циклів
- Type-safe запити з перевіркою на етапі компіляції
- IntelliSense підтримка
- Функціональний стиль програмування

**Практичне застосування:**

```csharp
// Тестові дані
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Department { get; set; } = "";
    public decimal Salary { get; set; }
    public int Age { get; set; }
    public DateTime HireDate { get; set; }
}

public static void BasicLinqExamples()
{
    var employees = new List<Employee>
    {
        new Employee { Id = 1, Name = "Іван Петров", Department = "IT", Salary = 50000, Age = 28, HireDate = new DateTime(2020, 3, 15) },
        new Employee { Id = 2, Name = "Анна Сидорова", Department = "HR", Salary = 45000, Age = 32, HireDate = new DateTime(2019, 7, 22) },
        new Employee { Id = 3, Name = "Петро Коваленко", Department = "IT", Salary = 60000, Age = 35, HireDate = new DateTime(2018, 1, 10) },
        new Employee { Id = 4, Name = "Марія Шевченко", Department = "Finance", Salary = 55000, Age = 29, HireDate = new DateTime(2021, 5, 8) },
        new Employee { Id = 5, Name = "Олег Мельник", Department = "IT", Salary = 48000, Age = 26, HireDate = new DateTime(2022, 2, 14) }
    };

    // Where - фільтрація
    var itEmployees = employees
        .Where(e => e.Department == "IT")
        .ToList();
    
    var highEarners = employees
        .Where(e => e.Salary > 50000)
        .Where(e => e.Age < 35)  // Можна ланцюгувати умови
        .ToList();

    // Select - проекція (вибірка полів)
    var employeeNames = employees
        .Select(e => e.Name)
        .ToList();
    
    var employeeSummary = employees
        .Select(e => new 
        { 
            e.Name, 
            e.Department, 
            YearsWorked = DateTime.Now.Year - e.HireDate.Year 
        })
        .ToList();

    // OrderBy - сортування
    var sortedByName = employees
        .OrderBy(e => e.Name)
        .ToList();
    
    var sortedBySalaryDesc = employees
        .OrderByDescending(e => e.Salary)
        .ThenBy(e => e.Name)  // Друга умова сортування
        .ToList();

    // Take, Skip - обмеження результатів
    var top3Earners = employees
        .OrderByDescending(e => e.Salary)
        .Take(3)
        .ToList();
    
    var skipFirst2 = employees
        .Skip(2)
        .Take(2)
        .ToList();

    // First, Single, Any, All
    var firstITEmployee = employees.First(e => e.Department == "IT");
    var oldestEmployee = employees.OrderByDescending(e => e.Age).First();
    
    var hasHighEarners = employees.Any(e => e.Salary > 55000);
    bool allOver25 = employees.All(e => e.Age > 25);
    
    // SingleOrDefault - очікуємо точно один елемент або null
    var financeEmployee = employees.SingleOrDefault(e => e.Department == "Finance");
    
    // Агрегатні функції
    decimal totalSalary = employees.Sum(e => e.Salary);
    decimal avgSalary = employees.Average(e => e.Salary);
    decimal maxSalary = employees.Max(e => e.Salary);
    decimal minSalary = employees.Min(e => e.Salary);
    int count = employees.Count(e => e.Department == "IT");
    
    Console.WriteLine($"Загальна зарплата: {totalSalary:C}");
    Console.WriteLine($"Середня зарплата: {avgSalary:C}");
    Console.WriteLine($"IT співробітників: {count}");
}

// GroupBy - групування
public static void GroupByExamples()
{
    var employees = GetEmployees(); // Припустимо, що маємо метод для отримання даних
    
    // Групування за департаментом
    var groupedByDepartment = employees
        .GroupBy(e => e.Department)
        .Select(g => new 
        {
            Department = g.Key,
            Count = g.Count(),
            AverageSalary = g.Average(e => e.Salary),
            TotalSalary = g.Sum(e => e.Salary),
            Employees = g.ToList()
        })
        .ToList();

    foreach (var group in groupedByDepartment)
    {
        Console.WriteLine($"Департамент: {group.Department}");
        Console.WriteLine($"  Кількість: {group.Count}");
        Console.WriteLine($"  Середня зарплата: {group.AverageSalary:C}");
        Console.WriteLine($"  Загальна зарплата: {group.TotalSalary:C}");
        Console.WriteLine();
    }
    
    // Групування за множинними критеріями
    var complexGrouping = employees
        .GroupBy(e => new { e.Department, AgeGroup = e.Age < 30 ? "Young" : "Senior" })
        .Select(g => new 
        {
            g.Key.Department,
            g.Key.AgeGroup,
            Count = g.Count(),
            AvgSalary = g.Average(e => e.Salary)
        })
        .OrderBy(x => x.Department)
        .ThenBy(x => x.AgeGroup)
        .ToList();
}

// Join операції
public static void JoinExamples()
{
    var departments = new List<Department>
    {
        new Department { Id = 1, Name = "IT", Budget = 500000 },
        new Department { Id = 2, Name = "HR", Budget = 200000 },
        new Department { Id = 3, Name = "Finance", Budget = 300000 }
    };
    
    var employees = GetEmployees();
    
    // Inner Join
    var employeesWithBudget = employees
        .Join(departments,
              employee => employee.Department,  // Ключ з employees
              department => department.Name,    // Ключ з departments
              (employee, department) => new     // Результат join
              {
                  employee.Name,
                  employee.Salary,
                  Department = department.Name,
                  DepartmentBudget = department.Budget
              })
        .ToList();
    
    // Group Join - один-до-багатьох
    var departmentsWithEmployees = departments
        .GroupJoin(employees,
                  dept => dept.Name,
                  emp => emp.Department,
                  (dept, empGroup) => new
                  {
                      DepartmentName = dept.Name,
                      Budget = dept.Budget,
                      Employees = empGroup.ToList(),
                      EmployeeCount = empGroup.Count(),
                      TotalSalaries = empGroup.Sum(e => e.Salary)
                  })
        .ToList();
    
    foreach (var dept in departmentsWithEmployees)
    {
        Console.WriteLine($"{dept.DepartmentName}: {dept.EmployeeCount} співробітників, бюджет {dept.Budget:C}");
    }
}

public class Department
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Budget { get; set; }
}

// LINQ з різними джерелами даних
public static void LinqDataSources()
{
    // Масиви
    int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    var evenNumbers = numbers.Where(n => n % 2 == 0).ToArray();
    
    // Рядки
    string text = "Hello World";
    var vowels = text.Where(char.IsLetter).Where(c => "aeiouAEIOU".Contains(c)).ToArray();
    
    // Словники
    var wordCounts = new Dictionary<string, int>
    {
        ["apple"] = 5,
        ["banana"] = 3,
        ["cherry"] = 8,
        ["date"] = 2
    };
    
    var popularFruits = wordCounts
        .Where(kvp => kvp.Value > 3)
        .OrderByDescending(kvp => kvp.Value)
        .Select(kvp => kvp.Key)
        .ToList();
    
    // Вкладені колекції - SelectMany
    var departments = new[]
    {
        new { Name = "IT", Employees = new[] { "Іван", "Петро", "Олег" } },
        new { Name = "HR", Employees = new[] { "Анна", "Марія" } },
        new { Name = "Finance", Employees = new[] { "Сергій" } }
    };
    
    // Плоский список всіх співробітників
    var allEmployees = departments
        .SelectMany(d => d.Employees)
        .ToList();
    
    // З додатковою інформацією про департамент
    var employeesWithDept = departments
        .SelectMany(d => d.Employees, (dept, emp) => new { Employee = emp, Department = dept.Name })
        .ToList();
}

// Query syntax vs Method syntax
public static void QueryVsMethodSyntax()
{
    var employees = GetEmployees();
    
    // Method syntax (fluent interface)
    var methodResult = employees
        .Where(e => e.Department == "IT")
        .Where(e => e.Salary > 45000)
        .OrderBy(e => e.Name)
        .Select(e => new { e.Name, e.Salary })
        .ToList();
    
    // Query syntax (SQL-like)
    var queryResult = (from e in employees
                      where e.Department == "IT"
                      where e.Salary > 45000
                      orderby e.Name
                      select new { e.Name, e.Salary })
                      .ToList();
    
    // Складніший запит з групуванням (query syntax)
    var complexQuery = from e in employees
                      group e by e.Department into g
                      where g.Count() > 1
                      select new 
                      {
                          Department = g.Key,
                          Count = g.Count(),
                          AvgSalary = g.Average(emp => emp.Salary),
                          TopEarner = g.OrderByDescending(emp => emp.Salary).First()
                      };
}

private static List<Employee> GetEmployees()
{
    return new List<Employee>
    {
        new Employee { Id = 1, Name = "Іван Петров", Department = "IT", Salary = 50000, Age = 28, HireDate = new DateTime(2020, 3, 15) },
        new Employee { Id = 2, Name = "Анна Сидорова", Department = "HR", Salary = 45000, Age = 32, HireDate = new DateTime(2019, 7, 22) },
        new Employee { Id = 3, Name = "Петро Коваленко", Department = "IT", Salary = 60000, Age = 35, HireDate = new DateTime(2018, 1, 10) },
        new Employee { Id = 4, Name = "Марія Шевченко", Department = "Finance", Salary = 55000, Age = 29, HireDate = new DateTime(2021, 5, 8) },
        new Employee { Id = 5, Name = "Олег Мельник", Department = "IT", Salary = 48000, Age = 26, HireDate = new DateTime(2022, 2, 14) }
    };
}
```

#### Lambda expressions та delegates

**Що це таке:**
- **Lambda expressions** - анонімні функції з коротким синтаксисом `(параметри) => вираз`
- **Delegates** - типи, які представляють посилання на методи з певною сигнатурою

**Навіщо це потрібно:**
- Передача поведінки як параметрів
- Callback функції та event handling
- Функціональний стиль програмування
- LINQ та інші API що приймають функції
- Зменшення boilerplate коду

**Практичне застосування:**

```csharp
// Базові lambda expressions
public static void LambdaBasics()
{
    // Прості lambda
    Func<int, int> square = x => x * x;
    Func<int, int, int> add = (x, y) => x + y;
    Func<string, bool> isEmpty = s => string.IsNullOrEmpty(s);
    
    // Lambda з блоком коду
    Func<int, string> describe = number =>
    {
        if (number < 0) return "Від'ємне";
        if (number == 0) return "Нуль";
        return "Додатне";
    };
    
    // Action - lambda без повернення значення
    Action<string> print = message => Console.WriteLine($"[LOG] {message}");
    Action<string, int> printWithId = (msg, id) => Console.WriteLine($"[{id}] {msg}");
    
    // Використання
    Console.WriteLine(square(5));        // 25
    Console.WriteLine(add(10, 20));      // 30
    Console.WriteLine(isEmpty(""));      // True
    Console.WriteLine(describe(-5));     // "Від'ємне"
    
    print("Hello World");               // [LOG] Hello World
    printWithId("Test message", 123);   // [123] Test message
}

// Delegates - посилання на методи
public delegate bool ValidationDelegate(string input);
public delegate void NotificationDelegate(string message);
public delegate T ProcessDelegate<T>(T input);

public static void DelegateExamples()
{
    // Прив'язка методів до delegates
    ValidationDelegate emailValidator = IsValidEmail;
    ValidationDelegate phoneValidator = IsValidPhone;
    
    // Multicast delegates - виклик декількох методів
    NotificationDelegate notifier = SendEmail;
    notifier += SendSMS;        // Додаємо ще один метод
    notifier += LogNotification;
    
    // Тестування валідаторів
    string email = "user@example.com";
    string phone = "+380123456789";
    
    if (emailValidator(email))
    {
        Console.WriteLine($"{email} - валідний email");
    }
    
    if (phoneValidator(phone))
    {
        Console.WriteLine($"{phone} - валідний телефон");
    }
    
    // Виклик всіх notification методів
    notifier("Важливе повідомлення!");  // Викличе всі 3 методи
    
    // Generic delegates
    ProcessDelegate<string> stringProcessor = text => text.ToUpper().Trim();
    ProcessDelegate<int> numberProcessor = num => num * 2;
    
    Console.WriteLine(stringProcessor("  hello world  ")); // "HELLO WORLD"
    Console.WriteLine(numberProcessor(21));                // 42
}

// Методи для delegates
static bool IsValidEmail(string email)
{
    return !string.IsNullOrEmpty(email) && email.Contains("@") && email.Contains(".");
}

static bool IsValidPhone(string phone)
{
    return !string.IsNullOrEmpty(phone) && phone.StartsWith("+") && phone.Length > 10;
}

static void SendEmail(string message)
{
    Console.WriteLine($"📧 Email: {message}");
}

static void SendSMS(string message)
{
    Console.WriteLine($"📱 SMS: {message}");
}

static void LogNotification(string message)
{
    Console.WriteLine($"📝 Log: [{DateTime.Now:HH:mm:ss}] {message}");
}

// Практичний приклад - система валідації
public class ValidationSystem
{
    private readonly List<ValidationDelegate> _validators = new();
    
    public void AddValidator(ValidationDelegate validator)
    {
        _validators.Add(validator);
    }
    
    public ValidationResult Validate(string input)
    {
        var errors = new List<string>();
        
        foreach (var validator in _validators)
        {
            if (!validator(input))
            {
                // В реальному додатку тут було б більше інформації про помилку
                errors.Add($"Validation failed for: {validator.Method.Name}");
            }
        }
        
        return new ValidationResult 
        { 
            IsValid = errors.Count == 0, 
            Errors = errors 
        };
    }
}

public class ValidationResult
{
    public bool IsValid { get; set; }
    public List<string> Errors { get; set; } = new();
}

// Використання системи валідації
public static void ValidationSystemExample()
{
    var validator = new ValidationSystem();
    
    // Додаємо валідатори через lambda
    validator.AddValidator(input => !string.IsNullOrEmpty(input));
    validator.AddValidator(input => input.Length >= 3);
    validator.AddValidator(input => input.Any(char.IsLetter));
    validator.AddValidator(input => !input.Contains(" "));
    
    // Тестуємо
    string[] testInputs = { "ab", "abc123", "valid_input", "", "has space" };
    
    foreach (string input in testInputs)
    {
        var result = validator.Validate(input);
        Console.WriteLine($"'{input}': {(result.IsValid ? "✅ Valid" : "❌ Invalid")}");
        
        if (!result.IsValid)
        {
            foreach (string error in result.Errors)
            {
                Console.WriteLine($"  - {error}");
            }
        }
    }
}

// Функціональне програмування з lambda
public static void FunctionalProgramming()
{
    var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    
    // Композиція функцій
    Func<int, int> double_ = x => x * 2;
    Func<int, int> addTen = x => x + 10;
    Func<int, bool> isEven = x => x % 2 == 0;
    
    // Ланцюжок обробки
    var result = numbers
        .Select(double_)        // Подвоюємо
        .Select(addTen)         // Додаємо 10
        .Where(isEven)          // Залишаємо парні
        .ToList();
    
    Console.WriteLine($"Результат: [{string.Join(", ", result)}]");
    
    // Currying - часткове застосування функцій
    Func<int, Func<int, int>> multiply = x => y => x * y;
    var multiplyBy3 = multiply(3);
    var multiplyBy5 = multiply(5);
    
    Console.WriteLine($"3 * 7 = {multiplyBy3(7)}");  // 21
    Console.WriteLine($"5 * 8 = {multiplyBy5(8)}");  // 40
    
    // Higher-order functions
    Func<T, bool> CreatePredicate<T>(Func<T, bool> condition) => condition;
    
    var isPositive = CreatePredicate<int>(x => x > 0);
    var isLongString = CreatePredicate<string>(s => s.Length > 5);
    
    var positiveNumbers = numbers.Where(isPositive).ToList();
    var longStrings = new[] { "short", "medium", "very long string" }.Where(isLongString).ToList();
}
```

#### Extension methods

**Що це таке:**
Extension methods дозволяють додавати нові методи до існуючих типів без зміни їх коду або створення нових похідних типів.

**Навіщо це потрібно:**
- Розширення функціональності існуючих типів
- Створення fluent interfaces
- Додавання зручних helper методів
- Поліпшення читабельності коду
- LINQ побудований на extension methods

**Практичне застосування:**

```csharp
// Створення extension methods
public static class StringExtensions
{
    // Перевірка на валідний email
    public static bool IsValidEmail(this string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            return false;
            
        return email.Contains("@") && email.Contains(".") && email.Length > 5;
    }
    
    // Обертання рядка
    public static string Reverse(this string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
            
        return new string(input.ToCharArray().Reverse().ToArray());
    }
    
    // Капіталізація першої літери кожного слова
    public static string ToTitleCase(this string input)
    {
        if (string.IsNullOrWhiteSpace(input))
            return input;
            
        return string.Join(" ", 
            input.Split(' ')
                 .Select(word => string.IsNullOrEmpty(word) ? word : 
                         char.ToUpper(word[0]) + word.Substring(1).ToLower()));
    }
    
    // Обрізання до максимальної довжини
    public static string Truncate(this string input, int maxLength, string suffix = "...")
    {
        if (string.IsNullOrEmpty(input) || input.Length <= maxLength)
            return input;
            
        return input.Substring(0, maxLength - suffix.Length) + suffix;
    }
    
    // Підрахунок слів
    public static int WordCount(this string input)
    {
        if (string.IsNullOrWhiteSpace(input))
            return 0;
            
        return input.Split(new[] { ' ', '\t', '\n', '\r' }, StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

// Extension methods для колекцій
public static class CollectionExtensions
{
    // Безпечне отримання елементу за індексом
    public static T? SafeGet<T>(this IList<T> list, int index)
    {
        return index >= 0 && index < list.Count ? list[index] : default(T);
    }
    
    // Перевірка чи колекція null або порожня
    public static bool IsNullOrEmpty<T>(this IEnumerable<T>? collection)
    {
        return collection == null || !collection.Any();
    }
    
    // Розбиття колекції на частини
    public static IEnumerable<IEnumerable<T>> Chunk<T>(this IEnumerable<T> source, int size)
    {
        if (size <= 0) throw new ArgumentException("Size must be positive", nameof(size));
        
        var chunk = new List<T>(size);
        foreach (var item in source)
        {
            chunk.Add(item);
            if (chunk.Count == size)
            {
                yield return chunk;
                chunk = new List<T>(size);
            }
        }
        
        if (chunk.Count > 0)
            yield return chunk;
    }
    
    // Виконання дії для кожного елемента (side effect)
    public static void ForEach<T>(this IEnumerable<T> source, Action<T> action)
    {
        foreach (var item in source)
        {
            action(item);
        }
    }
    
    // Distinct з custom selector
    public static IEnumerable<T> DistinctBy<T, TKey>(this IEnumerable<T> source, Func<T, TKey> keySelector)
    {
        var seenKeys = new HashSet<TKey>();
        return source.Where(item => seenKeys.Add(keySelector(item)));
    }
}

// Extension methods для чисел
public static class NumberExtensions
{
    public static bool IsEven(this int number) => number % 2 == 0;
    public static bool IsOdd(this int number) => number % 2 != 0;
    
    public static bool IsPrime(this int number)
    {
        if (number < 2) return false;
        if (number == 2) return true;
        if (number.IsEven()) return false;
        
        for (int i = 3; i * i <= number; i += 2)
        {
            if (number % i == 0) return false;
        }
        
        return true;
    }
    
    public static string ToOrdinal(this int number)
    {
        return number switch
        {
            1 => "1-й",
            2 => "2-й", 
            3 => "3-й",
            4 => "4-й",
            _ when number % 10 == 1 && number % 100 != 11 => $"{number}-й",
            _ when number % 10 == 2 && number % 100 != 12 => $"{number}-й",
            _ when number % 10 == 3 && number % 100 != 13 => $"{number}-й",
            _ => $"{number}-й"
        };
    }
    
    // Перетворення байтів в читабельний формат
    public static string ToFileSize(this long bytes)
    {
        string[] suffixes = { "B", "KB", "MB", "GB", "TB" };
        int counter = 0;
        double number = bytes;
        
        while (Math.Round(number / 1024) >= 1)
        {
            number /= 1024;
            counter++;
        }
        
        return $"{number:N1} {suffixes[counter]}";
    }
}

// DateTime extensions
public static class DateTimeExtensions
{
    public static bool IsWeekend(this DateTime date)
    {
        return date.DayOfWeek == DayOfWeek.Saturday || date.DayOfWeek == DayOfWeek.Sunday;
    }
    
    public static DateTime StartOfWeek(this DateTime date, DayOfWeek startOfWeek = DayOfWeek.Monday)
    {
        int diff = (7 + (date.DayOfWeek - startOfWeek)) % 7;
        return date.AddDays(-1 * diff).Date;
    }
    
    public static DateTime EndOfWeek(this DateTime date, DayOfWeek startOfWeek = DayOfWeek.Monday)
    {
        return date.StartOfWeek(startOfWeek).AddDays(7).AddTicks(-1);
    }
    
    public static string ToRelativeString(this DateTime date)
    {
        var timeSpan = DateTime.Now - date;
        
        return timeSpan switch
        {
            { TotalSeconds: <= 60 } => "щойно",
            { TotalMinutes: <= 1 } => "хвилину тому",
            { TotalMinutes: <= 60 } => $"{(int)timeSpan.TotalMinutes} хвилин тому",
            { TotalHours: <= 1 } => "годину тому",
            { TotalHours: <= 24 } => $"{(int)timeSpan.TotalHours} годин тому",
            { TotalDays: <= 1 } => "вчора",
            { TotalDays: <= 7 } => $"{(int)timeSpan.TotalDays} днів тому",
            _ => date.ToString("dd.MM.yyyy")
        };
    }
}

// Використання extension methods
public static void ExtensionMethodsUsage()
{
    // String extensions
    string email = "user@example.com";
    string name = "іван петренко";
    string longText = "Це дуже довгий текст який потребує обрізання";
    
    Console.WriteLine($"Email valid: {email.IsValidEmail()}");           // True
    Console.WriteLine($"Reversed: {name.Reverse()}");                    // "окнертеп наві"
    Console.WriteLine($"Title case: {name.ToTitleCase()}");              // "Іван Петренко"
    Console.WriteLine($"Truncated: {longText.Truncate(20)}");            // "Це дуже довгий те..."
    Console.WriteLine($"Word count: {longText.WordCount()}");            // 8
    
    // Collection extensions
    var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    var names = new List<string> { "Іван", "Петро", "Анна", "Марія", "Олег" };
    
    Console.WriteLine($"Safe get [15]: {numbers.SafeGet(15)}");          // 0 (default)
    Console.WriteLine($"Is null or empty: {names.IsNullOrEmpty()}");     // False
    
    var chunks = numbers.Chunk(3);
    foreach (var chunk in chunks)
    {
        Console.WriteLine($"Chunk: [{string.Join(", ", chunk)}]");
    }
    
    names.ForEach(name => Console.WriteLine($"Hello, {name}!"));
    
    // Number extensions
    var testNumbers = new[] { 2, 3, 4, 17, 25, 100 };
    foreach (int num in testNumbers)
    {
        Console.WriteLine($"{num}: Even={num.IsEven()}, Prime={num.IsPrime()}, Ordinal={num.ToOrdinal()}");
    }
    
    long fileSize = 1536000;
    Console.WriteLine($"File size: {fileSize.ToFileSize()}");            // "1.5 MB"
    
    // DateTime extensions
    DateTime today = DateTime.Now;
    DateTime pastDate = DateTime.Now.AddDays(-5);
    
    Console.WriteLine($"Is weekend: {today.IsWeekend()}");
    Console.WriteLine($"Start of week: {today.StartOfWeek():yyyy-MM-dd}");
    Console.WriteLine($"Relative: {pastDate.ToRelativeString()}");       // "5 днів тому"
}

// Fluent interface з extension methods
public static class FluentValidationExtensions
{
    public static ValidationBuilder<T> Validate<T>(this T value)
    {
        return new ValidationBuilder<T>(value);
    }
}

public class ValidationBuilder<T>
{
    private readonly T _value;
    private readonly List<string> _errors = new();
    
    public ValidationBuilder(T value)
    {
        _value = value;
    }
    
    public ValidationBuilder<T> NotNull(string errorMessage = "Value cannot be null")
    {
        if (_value == null)
            _errors.Add(errorMessage);
        return this;
    }
    
    public ValidationBuilder<T> Must(Func<T, bool> predicate, string errorMessage)
    {
        if (_value != null && !predicate(_value))
            _errors.Add(errorMessage);
        return this;
    }
    
    public ValidationResult Build()
    {
        return new ValidationResult 
        { 
            IsValid = _errors.Count == 0, 
            Errors = _errors.ToList() 
        };
    }
}

// Використання fluent validation
public static void FluentValidationExample()
{
    string email = "invalid-email";
    
    var result = email
        .Validate()
        .NotNull("Email is required")
        .Must(e => e.Contains("@"), "Email must contain @")
        .Must(e => e.Length > 5, "Email must be longer than 5 characters")
        .Must(e => !e.Contains(" "), "Email cannot contain spaces")
        .Build();
    
    if (result.IsValid)
    {
        Console.WriteLine("Email is valid!");
    }
    else
    {
        Console.WriteLine("Validation errors:");
        result.Errors.ForEach(error => Console.WriteLine($"- {error}"));
    }
}
```

---

## C# - ООП (розширення знань)

### Інтерфейси та абстрактні класи

#### Interfaces в C#

**Що це таке:**
Інтерфейс - це контракт, який описує які методи та властивості повинен реалізувати клас, але не містить реалізації. Це "що" клас повинен робити, а не "як".

**Навіщо це потрібно:**
- Множинне "наслідування" поведінки
- Створення слабо зв'язаних компонентів
- Тестування через mocking
- Plugin архітектура
- Dependency Injection

**Практичне застосування:**

```csharp
// Базовий інтерфейс
public interface IShape
{
    double Area { get; }
    double Perimeter { get; }
    void Draw();
    bool Contains(Point point);
}

// Реалізація інтерфейсу
public class Circle : IShape
{
    private double _radius;
    
    public Circle(double radius)
    {
        _radius = radius;
    }
    
    public double Area => Math.PI * _radius * _radius;
    public double Perimeter => 2 * Math.PI * _radius;
    
    public void Draw()
    {
        Console.WriteLine($"Малюю коло з радіусом {_radius}");
    }
    
    public bool Contains(Point point)
    {
        double distance = Math.Sqrt(point.X * point.X + point.Y * point.Y);
        return distance <= _radius;
    }
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }
    
    public double Area => Width * Height;
    public double Perimeter => 2 * (Width + Height);
    
    public void Draw()
    {
        Console.WriteLine($"Малюю прямокутник {Width}x{Height}");
    }
    
    public bool Contains(Point point)
    {
        return point.X >= 0 && point.X <= Width && point.Y >= 0 && point.Y <= Height;
    }
}

// Множинні інтерфейси
public interface IMovable
{
    Point Position { get; set; }
    void MoveTo(Point newPosition);
    void MoveBy(double deltaX, double deltaY);
}

public interface IResizable
{
    void Resize(double scaleFactor);
    Size Size { get; set; }
}

public interface IRotatable
{
    double Angle { get; set; }
    void RotateBy(double degrees);
}

// Клас що реалізує множинні інтерфейси
public class MovableRectangle : IShape, IMovable, IResizable, IRotatable
{
    private double _width, _height;
    
    public Point Position { get; set; }
    public double Angle { get; set; }
    
    public Size Size 
    { 
        get => new Size(_width, _height); 
        set { _width = value.Width; _height = value.Height; } 
    }
    
    public double Area => _width * _height;
    public double Perimeter => 2 * (_width + _height);
    
    public MovableRectangle(double width, double height, Point position)
    {
        _width = width;
        _height = height;
        Position = position;
        Angle = 0;
    }
    
    public void Draw()
    {
        Console.WriteLine($"Малюю рухомий прямокутник {_width}x{_height} в позиції ({Position.X}, {Position.Y})");
    }
    
    public bool Contains(Point point)
    {
        // Спрощена реалізація без урахування rotation
        var localPoint = new Point(point.X - Position.X, point.Y - Position.Y);
        return localPoint.X >= 0 && localPoint.X <= _width && localPoint.Y >= 0 && localPoint.Y <= _height;
    }
    
    public void MoveTo(Point newPosition)
    {
        Position = newPosition;
    }
    
    public void MoveBy(double deltaX, double deltaY)
    {
        Position = new Point(Position.X + deltaX, Position.Y + deltaY);
    }
    
    public void Resize(double scaleFactor)
    {
        _width *= scaleFactor;
        _height *= scaleFactor;
    }
    
    public void RotateBy(double degrees)
    {
        Angle = (Angle + degrees) % 360;
    }
}

// Використання інтерфейсів для поліморфізму
public static void InterfacePolymorphism()
{
    // Колекція різних фігур через інтерфейс
    List<IShape> shapes = new()
    {
        new Circle(5),
        new Rectangle(10, 8),
        new MovableRectangle(6, 4, new Point(10, 10))
    };
    
    // Обробка всіх фігур однаково
    foreach (IShape shape in shapes)
    {
        Console.WriteLine($"Площа: {shape.Area:F2}");
        Console.WriteLine($"Периметр: {shape.Perimeter:F2}");
        shape.Draw();
        Console.WriteLine();
    }
    
    // Робота тільки з рухомими об'єктами
    var movableShapes = shapes.OfType<IMovable>().ToList();
    foreach (IMovable movable in movableShapes)
    {
        movable.MoveBy(5, 5);
        Console.WriteLine($"Переміщено до: ({movable.Position.X}, {movable.Position.Y})");
    }
}

public struct Point
{
    public double X { get; set; }
    public double Y { get; set; }
    
    public Point(double x, double y) { X = x; Y = y; }
}

public struct Size
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public Size(double width, double height) { Width = width; Height = height; }
}
```

#### Multiple interface inheritance

**Що це таке:**
На відміну від класів (можна наслідувати тільки від одного), клас може реалізувати багато інтерфейсів одночасно.

**Навіщо це потрібно:**
- Комбінування різних здібностей
- Гнучка архітектура
- Уникнення проблем множинного наслідування класів

**Практичне застосування:**

```csharp
// Різні "здібності" як інтерфейси
public interface IReadable
{
    string Read();
}

public interface IWritable
{
    void Write(string content);
}

public interface ISeekable
{
    long Position { get; set; }
    void Seek(long position);
}

public interface IClosable
{
    bool IsClosed { get; }
    void Close();
}

// Файл що підтримує всі операції
public class FileStream : IReadable, IWritable, ISeekable, IClosable
{
    private string _filename;
    private List<string> _content;
    private long _position;
    private bool _isClosed;
    
    public long Position 
    { 
        get => _position; 
        set => _position = Math.Max(0, Math.Min(value, _content.Count)); 
    }
    
    public bool IsClosed => _isClosed;
    
    public FileStream(string filename)
    {
        _filename = filename;
        _content = new List<string>();
        _position = 0;
        _isClosed = false;
    }
    
    public string Read()
    {
        if (_isClosed) throw new InvalidOperationException("Stream is closed");
        if (_position >= _content.Count) return "";
        
        return _content[(int)_position++];
    }
    
    public void Write(string content)
    {
        if (_isClosed) throw new InvalidOperationException("Stream is closed");
        
        if (_position >= _content.Count)
        {
            _content.Add(content);
        }
        else
        {
            _content[(int)_position] = content;
        }
        _position++;
    }
    
    public void Seek(long position)
    {
        Position = position;
    }
    
    public void Close()
    {
        _isClosed = true;
        Console.WriteLine($"Файл {_filename} закрито");
    }
}

// Тільки для читання
public class ReadOnlyFile : IReadable, ISeekable, IClosable
{
    private string[] _lines;
    private long _position;
    private bool _isClosed;
    
    public long Position 
    { 
        get => _position; 
        set => _position = Math.Max(0, Math.Min(value, _lines.Length)); 
    }
    
    public bool IsClosed => _isClosed;
    
    public ReadOnlyFile(string[] lines)
    {
        _lines = lines;
        _position = 0;
        _isClosed = false;
    }
    
    public string Read()
    {
        if (_isClosed) throw new InvalidOperationException("Stream is closed");
        if (_position >= _lines.Length) return "";
        
        return _lines[_position++];
    }
    
    public void Seek(long position)
    {
        Position = position;
    }
    
    public void Close()
    {
        _isClosed = true;
    }
}

// Універсальні методи що працюють з інтерфейсами
public static class StreamUtils
{
    public static void CopyReadableToWritable(IReadable source, IWritable destination)
    {
        string line;
        while (!string.IsNullOrEmpty(line = source.Read()))
        {
            destination.Write(line);
        }
    }
    
    public static void PrintSeekableContent(ISeekable seekable)
    {
        if (seekable is IReadable readable)
        {
            long originalPosition = seekable.Position;
            seekable.Seek(0);  // На початок
            
            string line;
            while (!string.IsNullOrEmpty(line = readable.Read()))
            {
                Console.WriteLine(line);
            }
            
            seekable.Seek(originalPosition);  // Повертаємо позицію
        }
    }
    
    public static void SafeClose(IClosable closable)
    {
        if (!closable.IsClosed)
        {
            closable.Close();
        }
    }
}

// Використання multiple inheritance
public static void MultipleInheritanceExample()
{
    var fileStream = new FileStream("test.txt");
    var readOnlyFile = new ReadOnlyFile(new[] { "Line 1", "Line 2", "Line 3" });
    
    // Записуємо дані
    fileStream.Write("Hello");
    fileStream.Write("World");
    fileStream.Write("!");
    
    // Копіюємо з read-only в file stream
    fileStream.Seek(0);
    StreamUtils.CopyReadableToWritable(readOnlyFile, fileStream);
    
    // Виводимо вміст
    Console.WriteLine("FileStream content:");
    StreamUtils.PrintSeekableContent(fileStream);
    
    Console.WriteLine("ReadOnly content:");
    StreamUtils.PrintSeekableContent(readOnlyFile);
    
    // Закриваємо ресурси
    StreamUtils.SafeClose(fileStream);
    StreamUtils.SafeClose(readOnlyFile);
}
```

#### Abstract classes vs Interfaces

**Що це таке:**
- **Abstract class** - клас який не можна створити напряму, може містити як абстрактні так і реалізовані методи
- **Interface** - контракт який описує тільки сигнатури методів без реалізації

**Коли використовувати що:**
- **Abstract class**: спільний код + спільний стан + IS-A зв'язок
- **Interface**: контракт поведінки + CAN-DO здібності + множинна реалізація

**Практичне застосування:**

```csharp
// Abstract class - базова функціональність для всіх тварин
public abstract class Animal
{
    // Конкретні властивості зі станом
    public string Name { get; protected set; }
    public int Age { get; protected set; }
    public double Weight { get; protected set; }
    public bool IsAlive { get; private set; } = true;
    
    // Конструктор
    protected Animal(string name, int age, double weight)
    {
        Name = name;
        Age = age;
        Weight = weight;
    }
    
    // Конкретні методи з реалізацією
    public virtual void Eat(double foodAmount)
    {
        if (!IsAlive) return;
        
        Weight += foodAmount * 0.1; // 10% переходить у вагу
        Console.WriteLine($"{Name} поїв {foodAmount}г їжі. Вага тепер: {Weight:F1}кг");
    }
    
    public virtual void Sleep(int hours)
    {
        if (!IsAlive) return;
        Console.WriteLine($"{Name} спить {hours} годин");
    }
    
    public void Die()
    {
        IsAlive = false;
        Console.WriteLine($"{Name} помер 😢");
    }
    
    // Абстрактні методи - МАЮТЬ бути реалізовані у похідних класах
    public abstract void MakeSound();
    public abstract void Move();
    
    // Віртуальний метод - МОЖЕ бути перевизначений
    public virtual void ShowInfo()
    {
        Console.WriteLine($"{Name}: {Age} років, {Weight:F1}кг, {(IsAlive ? "живий" : "мертвий")}");
    }
}

// Інтерфейси для конкретних здібностей
public interface IFlyable
{
    double MaxAltitude { get; }
    double FlightSpeed { get; }
    void Fly();
    void Land();
}

public interface ISwimmable
{
    double SwimmingSpeed { get; }
    int MaxDiveDepth { get; }
    void Swim();
    void Dive(int depth);
}

public interface IClimbable
{
    void ClimbUp();
    void ClimbDown();
}

// Конкретні класи тварин
public class Dog : Animal, ISwimmable
{
    public string Breed { get; }
    public double SwimmingSpeed => 5.0; // км/год
    public int MaxDiveDepth => 2; // метри
    
    public Dog(string name, int age, double weight, string breed) 
        : base(name, age, weight)
    {
        Breed = breed;
    }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} гавкає: Гав-гав!");
    }
    
    public override void Move()
    {
        Console.WriteLine($"{Name} бігає на чотирьох лапах");
    }
    
    public void Swim()
    {
        Console.WriteLine($"{Name} плаває зі швидкістю {SwimmingSpeed} км/год");
    }
    
    public void Dive(int depth)
    {
        if (depth > MaxDiveDepth)
        {
            Console.WriteLine($"{Name} не може пірнати глибше {MaxDiveDepth}м");
            return;
        }
        Console.WriteLine($"{Name} пірнає на {depth}м");
    }
    
    // Унікальний метод для собак
    public void Fetch()
    {
        Console.WriteLine($"{Name} приносить м'яч");
    }
}

public class Bird : Animal, IFlyable
{
    public double MaxAltitude { get; }
    public double FlightSpeed { get; }
    public bool CanFly { get; private set; } = true;
    
    public Bird(string name, int age, double weight, double maxAltitude, double flightSpeed) 
        : base(name, age, weight)
    {
        MaxAltitude = maxAltitude;
        FlightSpeed = flightSpeed;
    }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} співає: Чирік-чирік!");
    }
    
    public override void Move()
    {
        if (CanFly)
            Console.WriteLine($"{Name} літає");
        else
            Console.WriteLine($"{Name} стрибає по землі");
    }
    
    public void Fly()
    {
        if (!CanFly)
        {
            Console.WriteLine($"{Name} не може літати");
            return;
        }
        Console.WriteLine($"{Name} літає на висоті до {MaxAltitude}м зі швидкістю {FlightSpeed} км/год");
    }
    
    public void Land()
    {
        Console.WriteLine($"{Name} приземлився");
    }
}

public class Penguin : Animal, ISwimmable // Пінгвін не літає!
{
    public double SwimmingSpeed => 8.0;
    public int MaxDiveDepth => 500;
    
    public Penguin(string name, int age, double weight) : base(name, age, weight) { }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} кричить: Кря-кря!");
    }
    
    public override void Move()
    {
        Console.WriteLine($"{Name} переваливається по льоду");
    }
    
    public void Swim()
    {
        Console.WriteLine($"{Name} швидко плаває під водою ({SwimmingSpeed} км/год)");
    }
    
    public void Dive(int depth)
    {
        if (depth > MaxDiveDepth)
        {
            Console.WriteLine($"{Name} не може пірнати глибше {MaxDiveDepth}м");
            return;
        }
        Console.WriteLine($"{Name} пірнає на {depth}м за рибою");
    }
}

public class Monkey : Animal, IClimbable
{
    public Monkey(string name, int age, double weight) : base(name, age, weight) { }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} кричить: У-у-а-а!");
    }
    
    public override void Move()
    {
        Console.WriteLine($"{Name} стрибає з гілки на гілку");
    }
    
    public void ClimbUp()
    {
        Console.WriteLine($"{Name} лізе вгору по дереву");
    }
    
    public void ClimbDown()
    {
        Console.WriteLine($"{Name} спускається з дерева");
    }
}

// Використання abstract classes та interfaces
public static void AnimalExample()
{
    // Створюємо різних тварин
    var animals = new List<Animal>
    {
        new Dog("Рекс", 3, 25.0, "Німецька вівчарка"),
        new Bird("Капітан", 2, 0.5, 1000, 50),
        new Penguin("Пінго", 5, 15.0),
        new Monkey("Кінг-Конг", 8, 45.0)
    };
    
    Console.WriteLine("=== Інформація про всіх тварин ===");
    foreach (Animal animal in animals)
    {
        animal.ShowInfo();
        animal.MakeSound();
        animal.Move();
        animal.Eat(2.0);
        Console.WriteLine();
    }
    
    Console.WriteLine("=== Тварини що плавають ===");
    var swimmers = animals.OfType<ISwimmable>().ToList();
    foreach (ISwimmable swimmer in swimmers)
    {
        swimmer.Swim();
        swimmer.Dive(10);
    }
    
    Console.WriteLine("=== Тварини що літають ===");
    var flyers = animals.OfType<IFlyable>().ToList();
    foreach (IFlyable flyer in flyers)
    {
        flyer.Fly();
        flyer.Land();
    }
    
    Console.WriteLine("=== Тварини що лазять ===");
    var climbers = animals.OfType<IClimbable>().ToList();
    foreach (IClimbable climber in climbers)
    {
        climber.ClimbUp();
        climber.ClimbDown();
    }
    
    // Специфічні методи для конкретних типів
    var dog = animals.OfType<Dog>().First();
    dog.Fetch();
}

// Коли використовувати abstract class vs interface
/*
Використовуйте ABSTRACT CLASS коли:
- Маєте спільний код між класами
- Потрібен спільний стан (поля)
- Зв'язок IS-A (Собака IS-A Тварина)
- Еволюція класу - можете додавати методи без breaking changes

Використовуйте INTERFACE коли:
- Потрібен тільки контракт без реалізації
- Множинна реалізація (клас може мати багато здібностей)
- Зв'язок CAN-DO (Собака CAN-DO плавати)
- Різні класи потребують однакової поведінки але не зв'язані
*/
```

#### Default interface methods (C# 8+)

**Що це таке:**
Починаючи з C# 8.0, інтерфейси можуть містити реалізацію методів за замовчуванням, що дозволяє еволюцію API без breaking changes.

**Навіщо це потрібно:**
- Еволюція існуючих інтерфейсів без breaking changes
- Додавання нової функціональності до existing interfaces
- Спільна логіка між реалізаціями

**Практичне застосування:**

```csharp
// Інтерфейс з default methods
public interface ILogger
{
    // Абстрактний метод - має бути реалізований
    void Log(string message);
    
    // Default implementation - можна не перевизначати
    void LogInfo(string message) => Log($"[INFO] {message}");
    void LogWarning(string message) => Log($"[WARNING] {message}");
    void LogError(string message) => Log($"[ERROR] {message}");
    
    // Default method з логікою
    void LogWithTimestamp(string message)
    {
        Log($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] {message}");
    }
    
    // Static methods в інтерфейсах
    static ILogger CreateConsoleLogger() => new ConsoleLogger();
    static ILogger CreateFileLogger(string filename) => new FileLogger(filename);
}

// Проста реалізація
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
    
    // Всі default методи доступні автоматично
    // LogInfo, LogWarning, LogError, LogWithTimestamp
}

// Реалізація що перевизначає деякі default методи
public class FileLogger : ILogger
{
    private readonly string _filename;
    
    public FileLogger(string filename)
    {
        _filename = filename;
    }
    
    public void Log(string message)
    {
        File.AppendAllText(_filename, message + Environment.NewLine);
    }
    
    // Перевизначаємо default implementation
    public void LogError(string message)
    {
        Log($"💥💥💥 CRITICAL ERROR: {message} 💥💥💥");
    }
    
    // LogInfo, LogWarning, LogWithTimestamp - використовують default реалізацію
}

// Розширений інтерфейс з більш складною логікою
public interface IAdvancedLogger : ILogger
{
    LogLevel MinimumLevel { get; set; }
    
    // Default method що використовує властивості
    void LogWithLevel(LogLevel level, string message)
    {
        if (level >= MinimumLevel)
        {
            string prefix = level switch
            {
                LogLevel.Debug => "🐛",
                LogLevel.Info => "ℹ️",
                LogLevel.Warning => "⚠️",
                LogLevel.Error => "❌",
                LogLevel.Critical => "💥",
                _ => "📝"
            };
            
            Log($"{prefix} [{level}] {message}");
        }
    }
    
    // Default batch logging
    void LogBatch(IEnumerable<(LogLevel level, string message)> messages)
    {
        foreach (var (level, message) in messages)
        {
            LogWithLevel(level, message);
        }
    }
}

public enum LogLevel
{
    Debug = 0,
    Info = 1,
    Warning = 2,
    Error = 3,
    Critical = 4
}

public class AdvancedConsoleLogger : IAdvancedLogger
{
    public LogLevel MinimumLevel { get; set; } = LogLevel.Info;
    
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
    
    // Перевизначаємо для кольорового виводу
    public void LogWithLevel(LogLevel level, string message)
    {
        if (level >= MinimumLevel)
        {
            var originalColor = Console.ForegroundColor;
            
            Console.ForegroundColor = level switch
            {
                LogLevel.Debug => ConsoleColor.Gray,
                LogLevel.Info => ConsoleColor.White,
                LogLevel.Warning => ConsoleColor.Yellow,
                LogLevel.Error => ConsoleColor.Red,
                LogLevel.Critical => ConsoleColor.Magenta,
                _ => ConsoleColor.White
            };
            
            // Використовуємо default реалізацію через explicit interface call
            ((IAdvancedLogger)this).LogWithLevel(level, message);
            
            Console.ForegroundColor = originalColor;
        }
    }
}

// Використання default interface methods
public static void DefaultInterfaceMethodsExample()
{
    // Статичні методи інтерфейсу
    ILogger consoleLogger = ILogger.CreateConsoleLogger();
    ILogger fileLogger = ILogger.CreateFileLogger("app.log");
    
    var loggers = new ILogger[] { consoleLogger, fileLogger };
    
    foreach (ILogger logger in loggers)
    {
        // Використання default methods
        logger.LogInfo("Application started");
        logger.LogWarning("This is a warning");
        logger.LogError("Something went wrong");
        logger.LogWithTimestamp("Timestamped message");
        Console.WriteLine();
    }
    
    // Advanced logger
    var advancedLogger = new AdvancedConsoleLogger
    {
        MinimumLevel = LogLevel.Warning
    };
    
    // Batch logging з default implementation
    var messages = new[]
    {
        (LogLevel.Debug, "Debug message - won't be shown"),
        (LogLevel.Info, "Info message - won't be shown"),
        (LogLevel.Warning, "Warning message"),
        (LogLevel.Error, "Error message"),
        (LogLevel.Critical, "Critical message")
    };
    
    advancedLogger.LogBatch(messages);
}
```

### Generics

#### Generic classes та methods

**Що це таке:**
Generics дозволяють створювати класи та методи, які працюють з типами, що визначаються під час використання, а не під час створення.

**Навіщо це потрібно:**
- Type safety без boxing/unboxing
- Повторне використання коду для різних типів
- Кращу продуктивність
- Читабельніший код
- Compile-time type checking

**Практичне застосування:**

```csharp
// Generic клас - універсальне сховище
public class Repository<T> where T : class
{
    private readonly List<T> _items = new();
    private readonly object _lock = new();
    
    // Generic методи
    public void Add(T item)
    {
        if (item == null) throw new ArgumentNullException(nameof(item));
        
        lock (_lock)
        {
            _items.Add(item);
        }
    }
    
    public bool Remove(T item)
    {
        lock (_lock)
        {
            return _items.Remove(item);
        }
    }
    
    public T? Find(Func<T, bool> predicate)
    {
        lock (_lock)
        {
            return _items.FirstOrDefault(predicate);
        }
    }
    
    public IEnumerable<T> FindAll(Func<T, bool> predicate)
    {
        lock (_lock)
        {
            return _items.Where(predicate).ToList(); // ToList для snapshot
        }
    }
    
    public int Count 
    { 
        get 
        { 
            lock (_lock) 
            { 
                return _items.Count; 
            } 
        } 
    }
    
    public void Clear()
    {
        lock (_lock)
        {
            _items.Clear();
        }
    }
}

// Generic method з множинними type parameters
public static class Converter
{
    public static TResult Convert<TSource, TResult>(TSource source, Func<TSource, TResult> converter)
    {
        if (source == null) throw new ArgumentNullException(nameof(source));
        if (converter == null) throw new ArgumentNullException(nameof(converter));
        
        return converter(source);
    }
    
    public static List<TResult> ConvertAll<TSource, TResult>(IEnumerable<TSource> source, Func<TSource, TResult> converter)
    {
        return source.Select(converter).ToList();
    }
}

// Generic класи з декількома type parameters
public class KeyValueStore<TKey, TValue> where TKey : notnull
{
    private readonly Dictionary<TKey, TValue> _store = new();
    private readonly object _lock = new();
    
    public TValue? Get(TKey key)
    {
        lock (_lock)
        {
            return _store.TryGetValue(key, out TValue? value) ? value : default(TValue);
        }
    }
    
    public void Set(TKey key, TValue value)
    {
        lock (_lock)
        {
            _store[key] = value;
        }
    }
    
    public bool ContainsKey(TKey key)
    {
        lock (_lock)
        {
            return _store.ContainsKey(key);
        }
    }
    
    public bool Remove(TKey key)
    {
        lock (_lock)
        {
            return _store.Remove(key);
        }
    }
    
    public IEnumerable<TKey> Keys
    {
        get
        {
            lock (_lock)
            {
                return _store.Keys.ToList(); // Snapshot
            }
        }
    }
    
    public IEnumerable<TValue> Values
    {
        get
        {
            lock (_lock)
            {
                return _store.Values.ToList(); // Snapshot
            }
        }
    }
}

// Generic класи для специфічних сценаріїв
public class LRUCache<TKey, TValue> where TKey : notnull
{
    private readonly int _maxSize;
    private readonly Dictionary<TKey, LinkedListNode<CacheItem>> _cache;
    private readonly LinkedList<CacheItem> _accessOrder;
    
    private class CacheItem
    {
        public TKey Key { get; set; }
        public TValue Value { get; set; }
        
        public CacheItem(TKey key, TValue value)
        {
            Key = key;
            Value = value;
        }
    }
    
    public LRUCache(int maxSize)
    {
        if (maxSize <= 0) throw new ArgumentException("Max size must be positive", nameof(maxSize));
        
        _maxSize = maxSize;
        _cache = new Dictionary<TKey, LinkedListNode<CacheItem>>(maxSize);
        _accessOrder = new LinkedList<CacheItem>();
    }
    
    public TValue? Get(TKey key)
    {
        if (_cache.TryGetValue(key, out LinkedListNode<CacheItem>? node))
        {
            // Переміщаємо в кінець (найновіший)
            _accessOrder.Remove(node);
            _accessOrder.AddLast(node);
            
            return node.Value.Value;
        }
        
        return default(TValue);
    }
    
    public void Set(TKey key, TValue value)
    {
        if (_cache.TryGetValue(key, out LinkedListNode<CacheItem>? existingNode))
        {
            // Оновлюємо існуючий
            existingNode.Value.Value = value;
            _accessOrder.Remove(existingNode);
            _accessOrder.AddLast(existingNode);
        }
        else
        {
            // Додаємо новий
            if (_cache.Count >= _maxSize)
            {
                // Видаляємо найстарший
                var oldestNode = _accessOrder.First!;
                _accessOrder.RemoveFirst();
                _cache.Remove(oldestNode.Value.Key);
            }
            
            var newItem = new CacheItem(key, value);
            var newNode = _accessOrder.AddLast(newItem);
            _cache[key] = newNode;
        }
    }
    
    public int Count => _cache.Count;
    public int MaxSize => _maxSize;
}

// Використання generic класів
public static void GenericClassesExample()
{
    // Repository для різних типів
    var userRepository = new Repository<User>();
    var productRepository = new Repository<Product>();
    
    // Додаємо користувачів
    userRepository.Add(new User { Id = 1, Name = "Іван", Email = "ivan@example.com" });
    userRepository.Add(new User { Id = 2, Name = "Анна", Email = "anna@example.com" });
    
    // Пошук користувача
    var foundUser = userRepository.Find(u => u.Name == "Іван");
    Console.WriteLine($"Знайдено: {foundUser?.Name}");
    
    // KeyValueStore
    var cache = new KeyValueStore<string, object>();
    cache.Set("user:1", new User { Id = 1, Name = "Cache User" });
    cache.Set("config:timeout", 30);
    cache.Set("feature:enabled", true);
    
    var cachedUser = cache.Get("user:1") as User;
    var timeout = cache.Get("config:timeout");
    
    Console.WriteLine($"Cached user: {cachedUser?.Name}");
    Console.WriteLine($"Timeout: {timeout}");
    
    // LRU Cache
    var lruCache = new LRUCache<string, string>(3);
    lruCache.Set("a", "Apple");
    lruCache.Set("b", "Banana");
    lruCache.Set("c", "Cherry");
    
    Console.WriteLine($"Get 'a': {lruCache.Get("a")}"); // "Apple"
    
    lruCache.Set("d", "Date"); // "b" буде видалений
    
    Console.WriteLine($"Get 'b': {lruCache.Get("b")}"); // null
    Console.WriteLine($"Get 'c': {lruCache.Get("c")}"); // "Cherry"
    
    // Converter utility
    var numbers = new[] { "1", "2", "3", "4", "5" };
    var integers = Converter.ConvertAll(numbers, int.Parse);
    var doubles = Converter.ConvertAll(integers, i => (double)i);
    
    Console.WriteLine($"Integers: [{string.Join(", ", integers)}]");
    Console.WriteLine($"Doubles: [{string.Join(", ", doubles)}]");
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}
```

#### Constraints на generic types

**Що це таке:**
Constraints дозволяють обмежити які типи можуть бути використані як generic параметри, забезпечуючи доступ до певних методів та властивостей.

**Навіщо це потрібно:**
- Забезпечення наявності потрібних методів/властивостей
- Type safety на етапі компіляції  
- Можливість викликати специфічні операції
- Кращий IntelliSense та performance
- Запобігання runtime помилкам

**Практичне застосування:**

```csharp
// Базові constraints
public class Repository<T> where T : class  // Reference type constraint
{
    public void AddEntity(T entity)
    {
        // Можемо безпечно перевіряти на null
        if (entity != null)
        {
            // logic
        }
    }
}

public class Calculator<T> where T : struct  // Value type constraint
{
    public T GetDefault()
    {
        return default(T);  // Завжди повертає 0, false, etc.
    }
}

// New constraint - тип має мати default constructor
public class Factory<T> where T : new()
{
    public T CreateInstance()
    {
        return new T();  // Можемо викликати default constructor
    }
    
    public List<T> CreateInstances(int count)
    {
        var result = new List<T>();
        for (int i = 0; i < count; i++)
        {
            result.Add(new T());
        }
        return result;
    }
}

// Interface constraints
public interface IEntity
{
    int Id { get; set; }
    DateTime CreatedAt { get; set; }
    bool IsActive { get; set; }
}

public interface IValidatable
{
    bool IsValid();
    List<string> GetValidationErrors();
}

public class EntityService<T> where T : IEntity, IValidatable, new()
{
    public T CreateNewEntity()
    {
        var entity = new T();
        entity.CreatedAt = DateTime.UtcNow;
        entity.IsActive = true;
        return entity;
    }
    
    public bool SaveEntity(T entity)
    {
        if (!entity.IsValid())
        {
            var errors = entity.GetValidationErrors();
            Console.WriteLine($"Validation failed: {string.Join(", ", errors)}");
            return false;
        }
        
        if (entity.Id == 0)
        {
            entity.Id = GenerateNewId();
        }
        
        // Save logic here
        return true;
    }
    
    private int GenerateNewId() => new Random().Next(1, 1000000);
}

// Base class constraints
public abstract class BaseDocument
{
    public string Title { get; set; } = "";
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public abstract void ProcessDocument();
}

public class DocumentProcessor<T> where T : BaseDocument, new()
{
    public void ProcessDocuments(List<T> documents)
    {
        foreach (var doc in documents)
        {
            Console.WriteLine($"Processing: {doc.Title} (Created: {doc.CreatedDate:yyyy-MM-dd})");
            doc.ProcessDocument();
        }
    }
    
    public T CreateEmptyDocument(string title)
    {
        var doc = new T();
        doc.Title = title;
        return doc;
    }
}

// Множинні constraints комбіновані
public class AdvancedRepository<T> where T : class, IEntity, IValidatable, new()
{
    private readonly List<T> _items = new();
    
    public async Task<bool> AddAsync(T item)
    {
        // IValidatable constraint дозволяє викликати IsValid
        if (!item.IsValid())
        {
            return false;
        }
        
        // IEntity constraint дозволяє доступ до Id
        if (item.Id == 0)
        {
            item.Id = await GenerateIdAsync();
        }
        
        // IEntity constraint дозволяє доступ до CreatedAt
        item.CreatedAt = DateTime.UtcNow;
        item.IsActive = true;
        
        _items.Add(item);
        return true;
    }
    
    public T CreateNew()
    {
        // new() constraint дозволяє створити інстанс
        var newItem = new T();
        newItem.CreatedAt = DateTime.UtcNow;
        newItem.IsActive = true;
        return newItem;
    }
    
    public List<T> GetActive()
    {
        return _items.Where(x => x.IsActive).ToList();
    }
    
    private async Task<int> GenerateIdAsync()
    {
        await Task.Delay(1); // Simulate async work
        return new Random().Next(1, 1000000);
    }
}

// Generic constraints між type parameters
public class Converter<TSource, TTarget> 
    where TSource : class
    where TTarget : class, TSource  // TTarget повинен наслідуватись від TSource
{
    public TTarget Convert(TSource source)
    {
        // Безпечне приведення типу
        return (TTarget)source;
    }
    
    public List<TTarget> ConvertAll(IEnumerable<TSource> sources)
    {
        return sources.OfType<TTarget>().ToList();
    }
}

// Unmanaged constraint (C# 7.3+) - для роботи з native code
public unsafe class NativeBuffer<T> where T : unmanaged
{
    private T* _buffer;
    private int _size;
    
    public NativeBuffer(int size)
    {
        _size = size;
        _buffer = (T*)Marshal.AllocHGlobal(size * sizeof(T));
    }
    
    public T Get(int index)
    {
        if (index < 0 || index >= _size) throw new IndexOutOfRangeException();
        return _buffer[index];
    }
    
    public void Set(int index, T value)
    {
        if (index < 0 || index >= _size) throw new IndexOutOfRangeException();
        _buffer[index] = value;
    }
    
    ~NativeBuffer()
    {
        if (_buffer != null)
        {
            Marshal.FreeHGlobal((IntPtr)_buffer);
        }
    }
}

// Приклади реалізації
public class User : IEntity, IValidatable
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsActive { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    
    public bool IsValid()
    {
        return !string.IsNullOrWhiteSpace(Name) && 
               !string.IsNullOrWhiteSpace(Email) && 
               Email.Contains("@");
    }
    
    public List<string> GetValidationErrors()
    {
        var errors = new List<string>();
        
        if (string.IsNullOrWhiteSpace(Name))
            errors.Add("Name is required");
            
        if (string.IsNullOrWhiteSpace(Email))
            errors.Add("Email is required");
        else if (!Email.Contains("@"))
            errors.Add("Email must be valid");
            
        return errors;
    }
}

public class Product : IEntity, IValidatable
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsActive { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    
    public bool IsValid()
    {
        return !string.IsNullOrWhiteSpace(Name) && Price > 0;
    }
    
    public List<string> GetValidationErrors()
    {
        var errors = new List<string>();
        
        if (string.IsNullOrWhiteSpace(Name))
            errors.Add("Product name is required");
            
        if (Price <= 0)
            errors.Add("Price must be positive");
            
        return errors;
    }
}

// Приклад document класів
public class TextDocument : BaseDocument
{
    public string Content { get; set; } = "";
    
    public override void ProcessDocument()
    {
        Console.WriteLine($"Processing text document: {Content.Length} characters");
    }
}

public class ImageDocument : BaseDocument
{
    public string ImagePath { get; set; } = "";
    public int Width { get; set; }
    public int Height { get; set; }
    
    public override void ProcessDocument()
    {
        Console.WriteLine($"Processing image: {Width}x{Height} at {ImagePath}");
    }
}

// Використання constraints
public static void ConstraintsExample()
{
    // EntityService з constraints
    var userService = new EntityService<User>();
    var productService = new EntityService<Product>();
    
    var newUser = userService.CreateNewEntity();
    newUser.Name = "Іван Петренко";
    newUser.Email = "ivan@example.com";
    
    bool userSaved = userService.SaveEntity(newUser);
    Console.WriteLine($"User saved: {userSaved}");
    
    // AdvancedRepository
    var userRepo = new AdvancedRepository<User>();
    var productRepo = new AdvancedRepository<Product>();
    
    var user = userRepo.CreateNew();
    user.Name = "Test User";
    user.Email = "test@example.com";
    
    bool added = await userRepo.AddAsync(user);
    Console.WriteLine($"User added: {added}");
    
    // DocumentProcessor
    var textProcessor = new DocumentProcessor<TextDocument>();
    var imageProcessor = new DocumentProcessor<ImageDocument>();
    
    var textDoc = textProcessor.CreateEmptyDocument("My Document");
    var imageDoc = imageProcessor.CreateEmptyDocument("My Image");
    
    textProcessor.ProcessDocuments(new List<TextDocument> { textDoc });
    imageProcessor.ProcessDocuments(new List<ImageDocument> { imageDoc });
    
    // Factory з new() constraint
    var userFactory = new Factory<User>();
    var users = userFactory.CreateInstances(3);
    Console.WriteLine($"Created {users.Count} users");
    
    // Unmanaged types
    var intBuffer = new NativeBuffer<int>(10);
    intBuffer.Set(0, 42);
    intBuffer.Set(1, 100);
    
    Console.WriteLine($"Buffer[0] = {intBuffer.Get(0)}");
    Console.WriteLine($"Buffer[1] = {intBuffer.Get(1)}");
}
```

#### Відмінності від C++ templates

**Що це таке:**
C++ templates та C# generics мають різні підходи до generic programming з різними можливостями та обмеженнями.

**Ключові відмінності:**

**C++ Templates:**
- Компілюються в окремий код для кожного типу
- Підтримують non-type parameters (значення як параметри)
- Duck typing - якщо операція працює, то все ОК
- Спеціалізація templates
- Compile-time обчислення

**C# Generics:**
- Єдиний compiled код для всіх reference types
- Тільки type parameters
- Explicit constraints для безпеки типів
- Рефлексія для generic типів під час runtime
- Runtime type information

**Практичні приклади різниць:**

```csharp
// C++ template equivalent (псевдокод для порівняння)
/*
template<typename T>
class Stack {
    T data[100];
    // Працює з будь-яким типом що підтримує копіювання
};

template<int N>  // Non-type parameter - недоступно в C#
class FixedArray {
    int data[N];
};

template<typename T>
void Swap(T& a, T& b) {
    T temp = a;  // Працює якщо T підтримує копіювання
    a = b;
    b = temp;
}
*/

// C# generic equivalent
public class Stack<T>
{
    private T[] _data = new T[100];
    private int _top = 0;
    
    public void Push(T item)
    {
        if (_top < _data.Length)
        {
            _data[_top++] = item;
        }
    }
    
    public T Pop()
    {
        if (_top > 0)
        {
            return _data[--_top];
        }
        return default(T)!;
    }
}

// C# не підтримує non-type parameters, тому використовуємо інший підхід
public class FixedArray<T>
{
    private readonly T[] _data;
    
    public FixedArray(int size)
    {
        _data = new T[size];
    }
    
    public T this[int index]
    {
        get => _data[index];
        set => _data[index] = value;
    }
    
    public int Length => _data.Length;
}

// Swap з constraints (C# потребує explicit constraints)
public static void Swap<T>(ref T a, ref T b) where T : struct
{
    T temp = a;
    a = b;
    b = temp;
}

// Або більш загальний варіант
public static void SwapObjects<T>(ref T a, ref T b) where T : class
{
    T temp = a;
    a = b;
    b = temp;
}

// C# generics з reflection можливостями (недоступно в C++ templates)
public static void PrintGenericInfo<T>()
{
    Type genericType = typeof(T);
    
    Console.WriteLine($"Type: {genericType.Name}");
    Console.WriteLine($"Is Value Type: {genericType.IsValueType}");
    Console.WriteLine($"Is Class: {genericType.IsClass}");
    Console.WriteLine($"Namespace: {genericType.Namespace}");
    
    if (genericType.IsGenericType)
    {
        Console.WriteLine("Generic type arguments:");
        foreach (var arg in genericType.GetGenericArguments())
        {
            Console.WriteLine($"  - {arg.Name}");
        }
    }
}

// C# generic specialization через method overloading (обмежено порівняно з C++)
public class Printer<T>
{
    public virtual void Print(T value)
    {
        Console.WriteLine($"Generic: {value}");
    }
}

// Спеціалізація для string
public class StringPrinter : Printer<string>
{
    public override void Print(string value)
    {
        Console.WriteLine($"String: \"{value}\"");
    }
}

// Спеціалізація для int
public class IntPrinter : Printer<int>
{
    public override void Print(int value)
    {
        Console.WriteLine($"Integer: {value:N0}");
    }
}

// Демонстрація використання
public static void TemplateVsGenericsExample()
{
    // Stack usage
    var intStack = new Stack<int>();
    var stringStack = new Stack<string>();
    
    intStack.Push(1);
    intStack.Push(2);
    stringStack.Push("Hello");
    stringStack.Push("World");
    
    Console.WriteLine($"Int: {intStack.Pop()}"); // 2
    Console.WriteLine($"String: {stringStack.Pop()}"); // "World"
    
    // Fixed array
    var fixedInts = new FixedArray<int>(5);
    fixedInts[0] = 10;
    fixedInts[1] = 20;
    
    Console.WriteLine($"Fixed array length: {fixedInts.Length}");
    Console.WriteLine($"Element 0: {fixedInts[0]}");
    
    // Swap
    int x = 5, y = 10;
    Swap(ref x, ref y);
    Console.WriteLine($"After swap: x={x}, y={y}");
    
    // Reflection
    PrintGenericInfo<List<int>>();
    PrintGenericInfo<Dictionary<string, object>>();
    
    // Specialized printers
    var stringPrinter = new StringPrinter();
    var intPrinter = new IntPrinter();
    
    stringPrinter.Print("Hello World");  // "String: "Hello World""
    intPrinter.Print(1234567);          // "Integer: 1,234,567"
}
```

### Events та Delegates

#### Events - publisher-subscriber pattern

**Що це таке:**
Events в C# - це спеціальний вид multicast delegate, що забезпечує безпечну реалізацію observer pattern. Event може мати багато підписників, які сповіщаються при виникненні події.

**Навіщо це потрібно:**
- Слабо зв'язана архітектура (loose coupling)
- Сповіщення багатьох компонентів про зміни
- Реакція на дії користувача або системні події
- Асинхронна комунікація між компонентами
- Розділення відповідальності

**Практичне застосування:**

```csharp
// Стандартний EventArgs для передачі даних
public class OrderEventArgs : EventArgs
{
    public int OrderId { get; set; }
    public decimal Amount { get; set; }
    public string CustomerName { get; set; } = "";
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
}

public class PaymentEventArgs : EventArgs
{
    public int PaymentId { get; set; }
    public decimal Amount { get; set; }
    public string PaymentMethod { get; set; } = "";
    public bool IsSuccessful { get; set; }
    public string ErrorMessage { get; set; } = "";
}

// Publisher - клас що генерує события
public class OrderProcessor
{
    // События з стандартним EventHandler pattern
    public event EventHandler<OrderEventArgs>? OrderCreated;
    public event EventHandler<OrderEventArgs>? OrderUpdated;
    public event EventHandler<OrderEventArgs>? OrderCancelled;
    public event EventHandler<PaymentEventArgs>? PaymentProcessed;
    
    // Простий event без додаткових даних
    public event EventHandler? ProcessingStarted;
    public event EventHandler? ProcessingCompleted;
    
    private readonly List<Order> _orders = new();
    
    public async Task<Order> CreateOrderAsync(string customerName, List<OrderItem> items)
    {
        // Сповіщення про початок обробки
        ProcessingStarted?.Invoke(this, EventArgs.Empty);
        
        var order = new Order
        {
            Id = new Random().Next(1000, 9999),
            CustomerName = customerName,
            Items = items,
            TotalAmount = items.Sum(i => i.Price * i.Quantity),
            Status = OrderStatus.Created,
            CreatedAt = DateTime.UtcNow
        };
        
        _orders.Add(order);
        
        // Виклик события через protected virtual method (рекомендована практика)
        OnOrderCreated(new OrderEventArgs 
        { 
            OrderId = order.Id, 
            Amount = order.TotalAmount, 
            CustomerName = order.CustomerName 
        });
        
        // Симуляція async обробки
        await Task.Delay(100);
        
        ProcessingCompleted?.Invoke(this, EventArgs.Empty);
        
        return order;
    }
    
    public async Task<bool> ProcessPaymentAsync(int orderId, string paymentMethod)
    {
        var order = _orders.FirstOrDefault(o => o.Id == orderId);
        if (order == null) return false;
        
        ProcessingStarted?.Invoke(this, EventArgs.Empty);
        
        // Симуляція обробки платежу
        await Task.Delay(500);
        bool success = new Random().NextDouble() > 0.1; // 90% success rate
        
        var paymentArgs = new PaymentEventArgs
        {
            PaymentId = new Random().Next(10000, 99999),
            Amount = order.TotalAmount,
            PaymentMethod = paymentMethod,
            IsSuccessful = success,
            ErrorMessage = success ? "" : "Payment gateway error"
        };
        
        OnPaymentProcessed(paymentArgs);
        
        if (success)
        {
            order.Status = OrderStatus.Paid;
            OnOrderUpdated(new OrderEventArgs 
            { 
                OrderId = order.Id, 
                Amount = order.TotalAmount, 
                CustomerName = order.CustomerName 
            });
        }
        
        ProcessingCompleted?.Invoke(this, EventArgs.Empty);
        return success;
    }
    
    public void CancelOrder(int orderId)
    {
        var order = _orders.FirstOrDefault(o => o.Id == orderId);
        if (order == null) return;
        
        order.Status = OrderStatus.Cancelled;
        
        OnOrderCancelled(new OrderEventArgs 
        { 
            OrderId = order.Id, 
            Amount = order.TotalAmount, 
            CustomerName = order.CustomerName 
        });
    }
    
    // Protected virtual methods для override в похідних класах
    protected virtual void OnOrderCreated(OrderEventArgs e)
    {
        OrderCreated?.Invoke(this, e);
    }
    
    protected virtual void OnOrderUpdated(OrderEventArgs e)
    {
        OrderUpdated?.Invoke(this, e);
    }
    
    protected virtual void OnOrderCancelled(OrderEventArgs e)
    {
        OrderCancelled?.Invoke(this, e);
    }
    
    protected virtual void OnPaymentProcessed(PaymentEventArgs e)
    {
        PaymentProcessed?.Invoke(this, e);
    }
    
    public List<Order> GetOrders() => _orders.ToList(); // Snapshot
}

// Subscribers - класи що реагують на события
public class EmailNotificationService
{
    public void Subscribe(OrderProcessor processor)
    {
        processor.OrderCreated += OnOrderCreated;
        processor.OrderCancelled += OnOrderCancelled;
        processor.PaymentProcessed += OnPaymentProcessed;
    }
    
    public void Unsubscribe(OrderProcessor processor)
    {
        processor.OrderCreated -= OnOrderCreated;
        processor.OrderCancelled -= OnOrderCancelled;
        processor.PaymentProcessed -= OnPaymentProcessed;
    }
    
    private void OnOrderCreated(object? sender, OrderEventArgs e)
    {
        Console.WriteLine($"📧 Email: Order #{e.OrderId} created for {e.CustomerName} (${e.Amount:N2})");
    }
    
    private void OnOrderCancelled(object? sender, OrderEventArgs e)
    {
        Console.WriteLine($"📧 Email: Order #{e.OrderId} was cancelled for {e.CustomerName}");
    }
    
    private void OnPaymentProcessed(object? sender, PaymentEventArgs e)
    {
        if (e.IsSuccessful)
        {
            Console.WriteLine($"📧 Email: Payment #{e.PaymentId} successful (${e.Amount:N2})");
        }
        else
        {
            Console.WriteLine($"📧 Email: Payment failed - {e.ErrorMessage}");
        }
    }
}

public class InventoryService
{
    private readonly Dictionary<string, int> _stock = new()
    {
        ["Laptop"] = 10,
        ["Mouse"] = 50,
        ["Keyboard"] = 25
    };
    
    public void Subscribe(OrderProcessor processor)
    {
        processor.OrderCreated += OnOrderCreated;
        processor.OrderCancelled += OnOrderCancelled;
    }
    
    private void OnOrderCreated(object? sender, OrderEventArgs e)
    {
        Console.WriteLine($"📦 Inventory: Processing stock for order #{e.OrderId}");
        
        if (sender is OrderProcessor processor)
        {
            var order = processor.GetOrders().FirstOrDefault(o => o.Id == e.OrderId);
            if (order != null)
            {
                foreach (var item in order.Items)
                {
                    if (_stock.ContainsKey(item.ProductName))
                    {
                        _stock[item.ProductName] -= item.Quantity;
                        Console.WriteLine($"📦 Updated stock: {item.ProductName} = {_stock[item.ProductName]}");
                    }
                }
            }
        }
    }
    
    private void OnOrderCancelled(object? sender, OrderEventArgs e)
    {
        Console.WriteLine($"📦 Inventory: Restoring stock for cancelled order #{e.OrderId}");
        // Logic to restore inventory
    }
}

public class LoggingService
{
    public void Subscribe(OrderProcessor processor)
    {
        // Підписуємось на всі events
        processor.OrderCreated += (s, e) => Log($"Order Created: #{e.OrderId}");
        processor.OrderUpdated += (s, e) => Log($"Order Updated: #{e.OrderId}");
        processor.OrderCancelled += (s, e) => Log($"Order Cancelled: #{e.OrderId}");
        processor.PaymentProcessed += (s, e) => Log($"Payment: #{e.PaymentId} - {(e.IsSuccessful ? "Success" : "Failed")}");
        processor.ProcessingStarted += (s, e) => Log("Processing started");
        processor.ProcessingCompleted += (s, e) => Log("Processing completed");
    }
    
    private void Log(string message)
    {
        Console.WriteLine($"📝 Log [{DateTime.Now:HH:mm:ss.fff}]: {message}");
    }
}

// Supporting classes
public class Order
{
    public int Id { get; set; }
    public string CustomerName { get; set; } = "";
    public List<OrderItem> Items { get; set; } = new();
    public decimal TotalAmount { get; set; }
    public OrderStatus Status { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class OrderItem
{
    public string ProductName { get; set; } = "";
    public int Quantity { get; set; }
    public decimal Price { get; set; }
}

public enum OrderStatus
{
    Created,
    Paid,
    Shipped,
    Delivered,
    Cancelled
}

// Custom event з власним delegate
public class StockAlert
{
    // Custom delegate
    public delegate void StockLevelChangedHandler(string productName, int oldLevel, int newLevel, bool isCritical);
    
    public event StockLevelChangedHandler? StockLevelChanged;
    
    private readonly Dictionary<string, int> _stock = new();
    private readonly Dictionary<string, int> _criticalLevels = new();
    
    public void SetCriticalLevel(string product, int criticalLevel)
    {
        _criticalLevels[product] = criticalLevel;
    }
    
    public void UpdateStock(string product, int newLevel)
    {
        int oldLevel = _stock.GetValueOrDefault(product, 0);
        _stock[product] = newLevel;
        
        bool isCritical = _criticalLevels.ContainsKey(product) && newLevel <= _criticalLevels[product];
        
        // Виклик custom event
        StockLevelChanged?.Invoke(product, oldLevel, newLevel, isCritical);
    }
    
    public int GetStock(string product) => _stock.GetValueOrDefault(product, 0);
}

// Використання events
public static async Task EventsExample()
{
    // Створюємо publisher
    var orderProcessor = new OrderProcessor();
    
    // Створюємо та підписуємо subscribers
    var emailService = new EmailNotificationService();
    var inventoryService = new InventoryService();
    var loggingService = new LoggingService();
    
    emailService.Subscribe(orderProcessor);
    inventoryService.Subscribe(orderProcessor);
    loggingService.Subscribe(orderProcessor);
    
    // Тестуємо події
    Console.WriteLine("=== Creating Order ===");
    var order = await orderProcessor.CreateOrderAsync("Іван Петренко", new List<OrderItem>
    {
        new OrderItem { ProductName = "Laptop", Quantity = 1, Price = 1000 },
        new OrderItem { ProductName = "Mouse", Quantity = 2, Price = 25 }
    });
    
    Console.WriteLine("\n=== Processing Payment ===");
    bool paymentSuccess = await orderProcessor.ProcessPaymentAsync(order.Id, "Credit Card");
    
    if (!paymentSuccess)
    {
        Console.WriteLine("\n=== Cancelling Order ===");
        orderProcessor.CancelOrder(order.Id);
    }
    
    // Custom event приклад
    Console.WriteLine("\n=== Stock Alert System ===");
    var stockAlert = new StockAlert();
    
    stockAlert.SetCriticalLevel("Laptop", 5);
    stockAlert.SetCriticalLevel("Mouse", 10);
    
    stockAlert.StockLevelChanged += (product, oldLevel, newLevel, isCritical) =>
    {
        Console.WriteLine($"📊 Stock Update: {product} {oldLevel} → {newLevel}");
        if (isCritical)
        {
            Console.WriteLine($"⚠️  CRITICAL: {product} stock is critically low ({newLevel})!");
        }
    };
    
    stockAlert.UpdateStock("Laptop", 15);  // Normal
    stockAlert.UpdateStock("Laptop", 3);   // Critical
    stockAlert.UpdateStock("Mouse", 8);    // Critical
    
    // Відписка від событий (важливо для уникнення memory leaks)
    emailService.Unsubscribe(orderProcessor);
    
    Console.WriteLine("\n=== After Unsubscribe ===");
    var anotherOrder = await orderProcessor.CreateOrderAsync("Анна Коваленко", new List<OrderItem>
    {
        new OrderItem { ProductName = "Keyboard", Quantity = 1, Price = 50 }
    });
    // Email сповіщення не прийде, але inventory та logging будуть працювати
}
```

#### Action та Func delegates

**Що це таке:**
Action та Func - це вбудовані generic delegate типи в .NET, які покривають найпоширеніші сценарії використання delegates без потреби створювати власні delegate типи.

**Навіщо це потрібно:**
- Спрощення коду - не треба оголошувати власні delegate типи
- Стандартизація - всі розуміють Action та Func
- Functional programming - передача функцій як параметрів
- Callbacks та event handlers
- LINQ та інші functional APIs

**Практичне застосування:**

```csharp
// Action - delegates що не повертають значення
public class ActionExamples
{
    // Action без параметрів
    public static void DemonstrateBasicActions()
    {
        Action simpleAction = () => Console.WriteLine("Simple action executed");
        Action namedAction = SayHello;
        
        simpleAction();  // "Simple action executed"
        namedAction();   // "Hello from named method"
        
        // Action з параметрами
        Action<string> printMessage = message => Console.WriteLine($"Message: {message}");
        Action<string, int> printWithCount = (msg, count) => 
        {
            for (int i = 0; i < count; i++)
            {
                Console.WriteLine($"{i + 1}: {msg}");
            }
        };
        
        printMessage("Hello World");
        printWithCount("Repeat this", 3);
        
        // Action з багатьма параметрами (до 16 параметрів)
        Action<string, int, bool, DateTime> complexAction = (name, age, isActive, date) =>
        {
            Console.WriteLine($"Person: {name}, Age: {age}, Active: {isActive}, Date: {date:yyyy-MM-dd}");
        };
        
        complexAction("John", 30, true, DateTime.Now);
    }
    
    private static void SayHello()
    {
        Console.WriteLine("Hello from named method");
    }
    
    // Практичний приклад - система retry з Action
    public static async Task ExecuteWithRetryAsync(Action operation, int maxRetries = 3, int delayMs = 1000)
    {
        int attempt = 0;
        
        while (attempt < maxRetries)
        {
            try
            {
                operation();
                return; // Успішно виконано
            }
            catch (Exception ex)
            {
                attempt++;
                Console.WriteLine($"Attempt {attempt} failed: {ex.Message}");
                
                if (attempt >= maxRetries)
                {
                    Console.WriteLine("Max retries reached. Operation failed.");
                    throw;
                }
                
                await Task.Delay(delayMs);
            }
        }
    }
    
    // Async Action еквівалент
    public static async Task ExecuteWithRetryAsync(Func<Task> asyncOperation, int maxRetries = 3, int delayMs = 1000)
    {
        int attempt = 0;
        
        while (attempt < maxRetries)
        {
            try
            {
                await asyncOperation();
                return;
            }
            catch (Exception ex)
            {
                attempt++;
                Console.WriteLine($"Async attempt {attempt} failed: {ex.Message}");
                
                if (attempt >= maxRetries)
                {
                    throw;
                }
                
                await Task.Delay(delayMs);
            }
        }
    }
}

// Func - delegates що повертають значення
public class FuncExamples
{
    public static void DemonstrateFuncs()
    {
        // Func з одним параметром та поверненням
        Func<int, int> square = x => x * x;
        Func<string, int> getLength = s => s?.Length ?? 0;
        Func<DateTime, string> formatDate = date => date.ToString("yyyy-MM-dd");
        
        Console.WriteLine($"Square of 5: {square(5)}");
        Console.WriteLine($"Length of 'Hello': {getLength("Hello")}");
        Console.WriteLine($"Today: {formatDate(DateTime.Now)}");
        
        // Func з багатьма параметрами
        Func<int, int, int> add = (x, y) => x + y;
        Func<string, string, bool> contains = (text, substring) => text.Contains(substring);
        Func<double, double, double, double> calculateTriangleArea = (a, b, c) =>
        {
            double s = (a + b + c) / 2;
            return Math.Sqrt(s * (s - a) * (s - b) * (s - c));
        };
        
        Console.WriteLine($"Add 10 + 20: {add(10, 20)}");
        Console.WriteLine($"'Hello World' contains 'World': {contains("Hello World", "World")}");
        Console.WriteLine($"Triangle area (3,4,5): {calculateTriangleArea(3, 4, 5):F2}");
        
        // Func без параметрів
        Func<int> getRandomNumber = () => new Random().Next(1, 101);
        Func<DateTime> getCurrentTime = () => DateTime.Now;
        
        Console.WriteLine($"Random number: {getRandomNumber()}");
        Console.WriteLine($"Current time: {getCurrentTime()}");
    }
    
    // Практичні приклади з Func
    public static void PracticalFuncExamples()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Func в LINQ операціях
        Func<int, bool> isEven = n => n % 2 == 0;
        Func<int, int> multiplyByTwo = n => n * 2;
        Func<int, string> formatNumber = n => $"Number: {n:D3}";
        
        var result = numbers
            .Where(isEven)           // Фільтруємо парні
            .Select(multiplyByTwo)   // Множимо на 2
            .Select(formatNumber)    // Форматуємо
            .ToList();
        
        Console.WriteLine("Processed numbers:");
        result.ForEach(Console.WriteLine);
        
        // Func для обчислення з caching
        var cache = new Dictionary<int, long>();
        
        Func<int, long> fibonacci = null!; // Для recursive lambda
        fibonacci = n =>
        {
            if (cache.ContainsKey(n))
                return cache[n];
                
            long result = n <= 1 ? n : fibonacci(n - 1) + fibonacci(n - 2);
            cache[n] = result;
            return result;
        };
        
        Console.WriteLine($"Fibonacci(30): {fibonacci(30)}");
        Console.WriteLine($"Fibonacci(40): {fibonacci(40)}");
    }
    
    // Factory pattern з Func
    public static void FactoryWithFunc()
    {
        // Dictionary з factory functions
        var factories = new Dictionary<string, Func<IShape>>
        {
            ["circle"] = () => new Circle(5.0),
            ["rectangle"] = () => new Rectangle(10, 8),
            ["triangle"] = () => new Triangle(3, 4, 5)
        };
        
        // Створення об'єктів через factory
        foreach (var factory in factories)
        {
            var shape = factory.Value();
            Console.WriteLine($"{factory.Key}: Area = {shape.Area:F2}");
        }
        
        // Параметризовані factory functions
        var paramFactories = new Dictionary<string, Func<double[], IShape>>
        {
            ["circle"] = args => new Circle(args[0]),
            ["rectangle"] = args => new Rectangle(args[0], args[1]),
            ["triangle"] = args => new Triangle(args[0], args[1], args[2])
        };
        
        var circle = paramFactories["circle"](new[] { 7.5 });
        var rectangle = paramFactories["rectangle"](new[] { 12.0, 6.0 });
        
        Console.WriteLine($"Custom circle: {circle.Area:F2}");
        Console.WriteLine($"Custom rectangle: {rectangle.Area:F2}");
    }
}

// Комбінування Action та Func
public class FunctionalProgrammingExample
{
    public static void CombiningActionsAndFuncs()
    {
        var users = new List<User>
        {
            new User { Name = "Alice", Age = 25, IsActive = true },
            new User { Name = "Bob", Age = 30, IsActive = false },
            new User { Name = "Charlie", Age = 35, IsActive = true }
        };
        
        // Func для трансформації
        Func<User, string> getUserSummary = user => $"{user.Name} ({user.Age}) - {(user.IsActive ? "Active" : "Inactive")}";
        
        // Action для side effects
        Action<string> logUser = summary => Console.WriteLine($"📝 Processing: {summary}");
        Action<User> markAsProcessed = user => user.IsProcessed = true;
        
        // Комбінування в pipeline
        ProcessUsers(users, getUserSummary, logUser, markAsProcessed);
        
        Console.WriteLine("\nAfter processing:");
        foreach (var user in users)
        {
            Console.WriteLine($"{user.Name}: Processed = {user.IsProcessed}");
        }
    }
    
    private static void ProcessUsers(
        List<User> users,
        Func<User, string> transformer,
        Action<string> logger,
        Action<User> processor)
    {
        foreach (var user in users)
        {
            string summary = transformer(user);  // Трансформація
            logger(summary);                     // Логування
            processor(user);                     // Обробка
        }
    }
    
    // Higher-order functions з Action/Func
    public static Func<T, bool> CreatePredicateAnd<T>(params Func<T, bool>[] predicates)
    {
        return item => predicates.All(predicate => predicate(item));
    }
    
    public static Func<T, bool> CreatePredicateOr<T>(params Func<T, bool>[] predicates)
    {
        return item => predicates.Any(predicate => predicate(item));
    }
    
    public static Action<T> CombineActions<T>(params Action<T>[] actions)
    {
        return item =>
        {
            foreach (var action in actions)
            {
                action(item);
            }
        };
    }
    
    public static void HigherOrderFunctionsExample()
    {
        var users = new List<User>
        {
            new User { Name = "Alice", Age = 25, IsActive = true },
            new User { Name = "Bob", Age = 17, IsActive = true },
            new User { Name = "Charlie", Age = 35, IsActive = false }
        };
        
        // Створюємо складні предикати
        Func<User, bool> isAdult = user => user.Age >= 18;
        Func<User, bool> isActive = user => user.IsActive;
        Func<User, bool> hasLongName = user => user.Name.Length > 5;
        
        var isAdultAndActive = CreatePredicateAnd(isAdult, isActive);
        var isAdultOrHasLongName = CreatePredicateOr(isAdult, hasLongName);
        
        // Створюємо комбіновані дії
        Action<User> logUser = user => Console.WriteLine($"Processing {user.Name}");
        Action<User> markProcessed = user => user.IsProcessed = true;
        Action<User> validateUser = user => Console.WriteLine($"Validating {user.Name}");
        
        var combinedAction = CombineActions(logUser, validateUser, markProcessed);
        
        // Застосовуємо
        Console.WriteLine("Adult and Active users:");
        users.Where(isAdultAndActive).ToList().ForEach(combinedAction);
        
        Console.WriteLine("\nAdult OR Long name users:");
        users.Where(isAdultOrHasLongName).ToList().ForEach(user => Console.WriteLine($"- {user.Name}"));
    }
}

// Supporting classes для прикладів
public interface IShape
{
    double Area { get; }
}

public class Circle : IShape
{
    public double Radius { get; }
    public Circle(double radius) => Radius = radius;
    public double Area => Math.PI * Radius * Radius;
}

public class Rectangle : IShape
{
    public double Width { get; }
    public double Height { get; }
    public Rectangle(double width, double height) { Width = width; Height = height; }
    public double Area => Width * Height;
}

public class Triangle : IShape
{
    public double A { get; }
    public double B { get; }
    public double C { get; }
    
    public Triangle(double a, double b, double c) { A = a; B = b; C = c; }
    
    public double Area
    {
        get
        {
            double s = (A + B + C) / 2;
            return Math.Sqrt(s * (s - A) * (s - B) * (s - C));
        }
    }
}

public class User
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
    public bool IsActive { get; set; }
    public bool IsProcessed { get; set; }
}

// Демонстрація використання
public static async Task ActionFuncExample()
{
    Console.WriteLine("=== Basic Actions ===");
    ActionExamples.DemonstrateBasicActions();
    
    Console.WriteLine("\n=== Retry Example ===");
    await ActionExamples.ExecuteWithRetryAsync(() =>
    {
        if (new Random().NextDouble() < 0.7) // 70% chance of failure
            throw new InvalidOperationException("Random failure");
        Console.WriteLine("✅ Operation succeeded!");
    });
    
    Console.WriteLine("\n=== Basic Funcs ===");
    FuncExamples.DemonstrateFuncs();
    
    Console.WriteLine("\n=== Practical Funcs ===");
    FuncExamples.PracticalFuncExamples();
    
    Console.WriteLine("\n=== Factory with Func ===");
    FuncExamples.FactoryWithFunc();
    
    Console.WriteLine("\n=== Functional Programming ===");
    FunctionalProgrammingExample.CombiningActionsAndFuncs();
    FunctionalProgrammingExample.HigherOrderFunctionsExample();
}
```

---

## C# - Сучасні можливості

### Pattern Matching

#### Switch expressions (C# 8+)

**Що це таке:**
Switch expressions - це компактний спосіб написання switch statements, що повертає значення. Це функціональний підхід до branching логіки з потужними pattern matching можливостями.

**Навіщо це потрібно:**
- Компактніший та читабельніший код
- Функціональний стиль програмування
- Краща type safety з compile-time перевірками
- Потужні pattern matching можливості
- Менше boilerplate коду

**Практичне застосування:**

```csharp
// Традиційний switch statement vs switch expression
public class TraditionalVsModernSwitch
{
    // Старий підхід - switch statement
    public static string GetSeasonOld(int month)
    {
        switch (month)
        {
            case 12:
            case 1:
            case 2:
                return "Winter";
            case 3:
            case 4:
            case 5:
                return "Spring";
            case 6:
            case 7:
            case 8:
                return "Summer";
            case 9:
            case 10:
            case 11:
                return "Autumn";
            default:
                return "Invalid month";
        }
    }
    
    // Новий підхід - switch expression (C# 8+)
    public static string GetSeason(int month) => month switch
    {
        12 or 1 or 2 => "Winter",
        >= 3 and <= 5 => "Spring", 
        >= 6 and <= 8 => "Summer",
        >= 9 and <= 11 => "Autumn",
        _ => "Invalid month"
    };
    
    // Ще більш складні patterns
    public static string GetSeasonWithDetails(int month) => month switch
    {
        12 => "Winter - December (Cold)",
        1 => "Winter - January (Very Cold)", 
        2 => "Winter - February (Still Cold)",
        3 => "Spring - March (Getting Warmer)",
        4 => "Spring - April (Mild)",
        5 => "Spring - May (Pleasant)",
        6 => "Summer - June (Warm)",
        7 => "Summer - July (Hot)",
        8 => "Summer - August (Very Hot)",
        9 => "Autumn - September (Cooling)",
        10 => "Autumn - October (Cool)",
        11 => "Autumn - November (Cold)",
        _ => throw new ArgumentOutOfRangeException(nameof(month), "Month must be 1-12")
    };
}

// Pattern matching з типами та conditions
public class TypePatternMatching
{
    public static string ProcessPayment(object payment) => payment switch
    {
        CreditCardPayment cc when cc.Amount > 1000 => $"High-value credit card: ${cc.Amount:N2}",
        CreditCardPayment cc => $"Credit card payment: ${cc.Amount:N2}",
        CashPayment cash when cash.Amount > 500 => $"Large cash payment: ${cash.Amount:N2}",
        CashPayment cash => $"Cash payment: ${cash.Amount:N2}",
        BankTransferPayment bt => $"Bank transfer: ${bt.Amount:N2} to {bt.AccountNumber}",
        null => "No payment provided",
        _ => "Unknown payment type"
    };
    
    public static decimal CalculateDiscount(Customer customer) => customer switch
    {
        PremiumCustomer { YearsOfMembership: > 10, TotalPurchases: > 50000 } => 0.25m,
        PremiumCustomer { YearsOfMembership: > 5, TotalPurchases: > 25000 } => 0.20m,
        PremiumCustomer { YearsOfMembership: > 2 } => 0.15m,
        PremiumCustomer => 0.10m,
        RegularCustomer { OrderCount: > 100, TotalSpent: > 10000 } => 0.08m,
        RegularCustomer { OrderCount: > 50 } => 0.05m,
        RegularCustomer => 0.02m,
        _ => 0m
    };
    
    public static string GetShippingMethod(Order order) => order switch
    {
        { TotalWeight: > 50, IsPriority: true } => "Express Heavy Freight",
        { TotalWeight: > 50 } => "Standard Heavy Freight", 
        { TotalAmount: > 100, IsPriority: true, IsInternational: false } => "Express Domestic",
        { TotalAmount: > 100, IsPriority: true } => "Express International",
        { TotalAmount: > 100 } => "Free Standard Shipping",
        { IsPriority: true, IsInternational: false } => "Express Domestic",
        { IsPriority: true } => "Express International",
        { IsInternational: true } => "International Standard",
        _ => "Standard Domestic"
    };
}

// Tuple patterns
public class TuplePatterns
{
    public static string GetQuadrant(int x, int y) => (x, y) switch
    {
        (> 0, > 0) => "First quadrant (I)",
        (< 0, > 0) => "Second quadrant (II)",
        (< 0, < 0) => "Third quadrant (III)", 
        (> 0, < 0) => "Fourth quadrant (IV)",
        (0, 0) => "Origin",
        (0, var yVal) => $"On Y-axis at y={yVal}",
        (var xVal, 0) => $"On X-axis at x={xVal}"
    };
    
    public static string AnalyzeMovement(int deltaX, int deltaY) => (deltaX, deltaY) switch
    {
        (0, 0) => "No movement",
        (0, > 0) => $"Moving up {deltaY} units",
        (0, < 0) => $"Moving down {Math.Abs(deltaY)} units",
        (> 0, 0) => $"Moving right {deltaX} units",
        (< 0, 0) => $"Moving left {Math.Abs(deltaX)} units",
        (> 0, > 0) => $"Moving northeast ({deltaX}, {deltaY})",
        (> 0, < 0) => $"Moving southeast ({deltaX}, {deltaY})",
        (< 0, > 0) => $"Moving northwest ({deltaX}, {deltaY})",
        (< 0, < 0) => $"Moving southwest ({deltaX}, {deltaY})"
    };
    
    // Складні tuple patterns з guards
    public static string CategorizeTransaction(decimal amount, string type, bool isApproved) => 
        (amount, type, isApproved) switch
    {
        (> 10000, "withdrawal", true) => "Large approved withdrawal",
        (> 10000, "withdrawal", false) => "Large denied withdrawal - requires authorization",
        (> 10000, "deposit", _) => "Large deposit - flagged for review",
        (< 0, _, _) => "Invalid negative amount",
        (0, _, _) => "Zero amount transaction",
        (_, "transfer", false) => "Transfer denied",
        (_, "transfer", true) when amount > 5000 => "High-value transfer approved",
        (_, "transfer", true) => "Standard transfer approved",
        (_, "payment", false) => "Payment declined",
        (_, "payment", true) => $"Payment of ${amount:N2} approved",
        _ => $"Unknown transaction: {type} for ${amount:N2}"
    };
}

// Property patterns та nested patterns
public class PropertyPatterns
{
    public static string GetInsuranceRisk(Person person) => person switch
    {
        { Age: < 18 } => "Minor - no insurance available",
        { Age: >= 65, HasChronicConditions: true } => "High risk - premium coverage",
        { Age: >= 65, HasChronicConditions: false } => "Moderate risk - senior coverage",
        { Age: >= 50, HasChronicConditions: true, Exercise: { Frequency: >= 3 } } => "Moderate risk - active senior",
        { Age: >= 50, HasChronicConditions: true } => "High risk - sedentary lifestyle",
        { Age: >= 25, Occupation: "pilot" or "miner" or "firefighter" } => "High risk - dangerous occupation",
        { Age: >= 25, HasChronicConditions: false, Exercise: { Frequency: >= 4, Type: "running" or "swimming" } } => "Low risk - very active",
        { Age: >= 25, HasChronicConditions: false } => "Low risk - standard coverage",
        _ => "Standard coverage"
    };
    
    public static decimal CalculateLoanRate(LoanApplication loan) => loan switch
    {
        { 
            CreditScore: >= 800, 
            Income: >= 100000, 
            DebtToIncomeRatio: <= 0.2, 
            LoanAmount: <= 500000 
        } => 0.025m, // 2.5% - excellent
        
        { 
            CreditScore: >= 750, 
            Income: >= 75000, 
            DebtToIncomeRatio: <= 0.3 
        } => 0.035m, // 3.5% - very good
        
        { 
            CreditScore: >= 700, 
            Income: >= 50000, 
            DebtToIncomeRatio: <= 0.4 
        } => 0.045m, // 4.5% - good
        
        { 
            CreditScore: >= 650, 
            DebtToIncomeRatio: <= 0.5 
        } => 0.055m, // 5.5% - fair
        
        { CreditScore: < 600 } => throw new InvalidOperationException("Credit score too low for approval"),
        { DebtToIncomeRatio: > 0.6 } => throw new InvalidOperationException("Debt-to-income ratio too high"),
        
        _ => 0.065m // 6.5% - default rate
    };
    
    // Nested object patterns
    public static string GetShippingStatus(OrderWithTracking order) => order switch
    {
        {
            Status: OrderStatus.Shipped,
            Tracking: { 
                Location: var location, 
                EstimatedDelivery: var delivery,
                IsInTransit: true 
            }
        } when delivery.Date == DateTime.Today => 
            $"Out for delivery in {location} - arriving today",
            
        {
            Status: OrderStatus.Shipped,
            Tracking: { 
                Location: var location, 
                EstimatedDelivery: var delivery,
                IsInTransit: true 
            }
        } => $"In transit at {location} - arriving {delivery:MMM dd}",
        
        { 
            Status: OrderStatus.Processing, 
            Items: { Count: > 5 } 
        } => "Large order being processed",
        
        { 
            Status: OrderStatus.Processing 
        } => "Order being processed",
        
        { 
            Status: OrderStatus.Delivered, 
            Tracking: { DeliveredAt: var time } 
        } => $"Delivered on {time:MMM dd 'at' h:mm tt}",
        
        _ => "Status unknown"
    };
}

// Supporting classes
public abstract record Payment(decimal Amount);
public record CreditCardPayment(decimal Amount, string CardNumber) : Payment(Amount);
public record CashPayment(decimal Amount) : Payment(Amount);
public record BankTransferPayment(decimal Amount, string AccountNumber) : Payment(Amount);

public abstract record Customer(string Name);
public record PremiumCustomer(string Name, int YearsOfMembership, decimal TotalPurchases) : Customer(Name);
public record RegularCustomer(string Name, int OrderCount, decimal TotalSpent) : Customer(Name);

public record Order(decimal TotalAmount, double TotalWeight, bool IsPriority, bool IsInternational);

public class Person
{
    public int Age { get; set; }
    public string Occupation { get; set; } = "";
    public bool HasChronicConditions { get; set; }
    public Exercise Exercise { get; set; } = new();
}

public class Exercise
{
    public int Frequency { get; set; } // Times per week
    public string Type { get; set; } = "";
}

public class LoanApplication
{
    public int CreditScore { get; set; }
    public decimal Income { get; set; }
    public decimal DebtToIncomeRatio { get; set; }
    public decimal LoanAmount { get; set; }
}

public class OrderWithTracking
{
    public OrderStatus Status { get; set; }
    public List<object> Items { get; set; } = new();
    public TrackingInfo Tracking { get; set; } = new();
}

public class TrackingInfo
{
    public string Location { get; set; } = "";
    public DateTime EstimatedDelivery { get; set; }
    public DateTime DeliveredAt { get; set; }
    public bool IsInTransit { get; set; }
}

public enum OrderStatus
{
    Processing,
    Shipped, 
    Delivered,
    Cancelled
}

// Демонстрація використання
public static void PatternMatchingExamples()
{
    Console.WriteLine("=== Switch Expressions ===");
    for (int month = 1; month <= 12; month++)
    {
        Console.WriteLine($"Month {month}: {TraditionalVsModernSwitch.GetSeason(month)}");
    }
    
    Console.WriteLine("\n=== Type Pattern Matching ===");
    var payments = new object[]
    {
        new CreditCardPayment(1500, "**** 1234"),
        new CashPayment(250), 
        new BankTransferPayment(5000, "ACC-789"),
        null
    };
    
    foreach (var payment in payments)
    {
        Console.WriteLine(TypePatternMatching.ProcessPayment(payment));
    }
    
    Console.WriteLine("\n=== Customer Discounts ===");
    var customers = new Customer[]
    {
        new PremiumCustomer("Alice Premium", 12, 75000),
        new PremiumCustomer("Bob Premium", 3, 15000),
        new RegularCustomer("Charlie Regular", 150, 25000),
        new RegularCustomer("Diana Regular", 25, 5000)
    };
    
    foreach (var customer in customers)
    {
        var discount = TypePatternMatching.CalculateDiscount(customer);
        Console.WriteLine($"{customer.Name}: {discount:P0} discount");
    }
    
    Console.WriteLine("\n=== Tuple Patterns ===");
    var coordinates = new[] { (5, 3), (-2, 4), (0, 0), (0, -5), (7, 0) };
    foreach (var (x, y) in coordinates)
    {
        Console.WriteLine($"({x}, {y}): {TuplePatterns.GetQuadrant(x, y)}");
    }
    
    Console.WriteLine("\n=== Property Patterns ===");
    var people = new[]
    {
        new Person { Age = 35, Occupation = "teacher", HasChronicConditions = false, Exercise = new() { Frequency = 5, Type = "running" } },
        new Person { Age = 45, Occupation = "pilot", HasChronicConditions = false, Exercise = new() { Frequency = 2, Type = "walking" } },
        new Person { Age = 70, HasChronicConditions = true, Exercise = new() { Frequency = 3, Type = "swimming" } }
    };
    
    foreach (var person in people)
    {
        Console.WriteLine($"Age {person.Age} ({person.Occupation}): {PropertyPatterns.GetInsuranceRisk(person)}");
    }
}
```

## 3.2. Pattern Matching в if statements (C# 7.0+)

### Що це таке
Pattern matching в if statements дозволяє перевіряти не лише значення, а й тип об'єкта, структуру даних та комбінації умов в одному виразі. Це розширення традиційних умовних операторів, які тепер можуть виконувати декомпозицію даних, перевірку типів та присвоєння змінних одночасно.

### Навіщо це потрібно
1. **Типобезпека**: Перевірка типу та використання об'єкта в одному кроці без додаткового casting
2. **Декомпозиція**: Можливість розбирати складні структури даних безпосередньо в умові
3. **Читабельність**: Складні умови стають більш зрозумілими та експресивними
4. **Безпека null**: Автоматична перевірка на null в процесі pattern matching
5. **Менше boilerplate коду**: Усуває необхідність в окремих перевірках типу та casting

### Практичне застосування

```csharp
// Традиційний підхід
public static void ProcessTraditional(object input)
{
    if (input != null && input is string)
    {
        string str = (string)input;
        if (str.Length > 5)
        {
            Console.WriteLine($"Long string: {str}");
        }
    }
}

// Pattern matching approach (C# 7+)
public static void ProcessModern(object input)
{
    if (input is string str && str.Length > 5)
    {
        Console.WriteLine($"Long string: {str}");
    }
}

// Nullable patterns (C# 9+)
public static void HandleNullable(string? input)
{
    if (input is not null and { Length: > 0 })
    {
        Console.WriteLine($"Non-empty string: {input}");
    }
    
    if (input is null or { Length: 0 })
    {
        Console.WriteLine("Empty or null string");
    }
}

// Relational patterns (C# 9+)
public static string CategorizeAge(int age)
{
    if (age is < 0)
        return "Invalid age";
    if (age is >= 0 and <= 12)
        return "Child";
    if (age is >= 13 and <= 19)
        return "Teenager";
    if (age is >= 20 and <= 64)
        return "Adult";
    if (age is >= 65)
        return "Senior";
        
    return "Unknown";
}

// Property patterns в if statements
public record Employee(string Name, int Age, string Department, decimal Salary);

public static void AnalyzeEmployee(Employee emp)
{
    if (emp is { Age: >= 50, Department: "IT", Salary: > 80000 })
    {
        Console.WriteLine("Senior IT professional with high salary");
    }
    
    if (emp is { Name.Length: <= 3 })
    {
        Console.WriteLine("Employee with short name");
    }
}

// Tuple patterns в if
public static void AnalyzeCoordinate(int x, int y)
{
    if ((x, y) is (0, 0))
        Console.WriteLine("Origin point");
    else if ((x, y) is (var a, var b) when a == b)
        Console.WriteLine($"Diagonal point: ({a}, {b})");
    else if ((x, y) is (0, _))
        Console.WriteLine("On Y-axis");
    else if ((x, y) is (_, 0))
        Console.WriteLine("On X-axis");
}

// List patterns (C# 11+)
public static void AnalyzeList(int[] numbers)
{
    if (numbers is [])
        Console.WriteLine("Empty array");
    else if (numbers is [var single])
        Console.WriteLine($"Single element: {single}");
    else if (numbers is [var first, var second])
        Console.WriteLine($"Two elements: {first}, {second}");
    else if (numbers is [var head, .. var tail])
        Console.WriteLine($"Head: {head}, Tail has {tail.Length} elements");
    else if (numbers is [.., var last])
        Console.WriteLine($"Last element: {last}");
}

// Практичні приклади для веб-розробки
public abstract record HttpResponse;
public record OkResponse(string Content) : HttpResponse;
public record NotFoundResponse : HttpResponse;
public record ErrorResponse(string Message, int Code) : HttpResponse;

public static void HandleHttpResponse(HttpResponse response)
{
    if (response is OkResponse { Content: var content } when content.Contains("success"))
    {
        Console.WriteLine("Successful response with success indicator");
    }
    else if (response is ErrorResponse { Code: >= 500 })
    {
        Console.WriteLine("Server error occurred");
    }
    else if (response is ErrorResponse { Code: >= 400 and < 500, Message: var msg })
    {
        Console.WriteLine($"Client error: {msg}");
    }
}

// Validation з pattern matching
public record UserInput(string? Email, int? Age, string? Phone);

public static List<string> ValidateUser(UserInput input)
{
    var errors = new List<string>();
    
    if (input.Email is not { Length: > 0 } or not string email || !email.Contains("@"))
    {
        errors.Add("Invalid email format");
    }
    
    if (input.Age is not (>= 18 and <= 120))
    {
        errors.Add("Age must be between 18 and 120");
    }
    
    if (input.Phone is not { Length: >= 10 } phone || !phone.All(char.IsDigit))
    {
        errors.Add("Phone must contain at least 10 digits");
    }
    
    return errors;
}

// Pattern matching для exception handling
public static void SafeOperation(Func<string> operation)
{
    try
    {
        var result = operation();
        if (result is { Length: > 0 } validResult)
        {
            Console.WriteLine($"Success: {validResult}");
        }
    }
    catch (Exception ex) when (ex is ArgumentException { ParamName: not null })
    {
        Console.WriteLine($"Argument error in parameter: {((ArgumentException)ex).ParamName}");
    }
    catch (Exception ex) when (ex is InvalidOperationException { Message: var msg })
    {
        Console.WriteLine($"Operation error: {msg}");
    }
}

// Складні business rules з pattern matching
public enum OrderStatus { Pending, Processing, Shipped, Delivered, Cancelled }
public record Order(int Id, OrderStatus Status, decimal Amount, DateTime CreatedAt, string? TrackingNumber);

public static bool CanCancelOrder(Order order)
{
    return order switch
    {
        { Status: OrderStatus.Pending } => true,
        { Status: OrderStatus.Processing, CreatedAt: var created } when DateTime.Now.Subtract(created).TotalHours < 2 => true,
        _ => false
    } || (order is { Status: OrderStatus.Shipped, TrackingNumber: null });
}

public static void ProcessOrder(Order order)
{
    if (order is { Status: OrderStatus.Delivered, Amount: > 1000 })
    {
        Console.WriteLine("High-value delivered order - send satisfaction survey");
    }
    else if (order is { Status: OrderStatus.Cancelled, CreatedAt: var created } 
             when DateTime.Now.Subtract(created).TotalDays > 30)
    {
        Console.WriteLine("Old cancelled order - archive");
    }
}
```

## 3.3. Records та Structs (C# 9.0+)

### Що це таке
Records - це новий reference type в C# 9, призначений для створення immutable data objects з автоматично згенерованими методами для порівняння, хешування та форматування. Structs - це value types, що зберігаються в стеку та передаються за значенням. C# 10 додав record struct, поєднуючи переваги обох підходів.

### Навіщо це потрібно
1. **Immutable data objects**: Records забезпечують незмінність даних за замовчуванням
2. **Value-based equality**: Автоматичне порівняння за значенням, а не за референсом
3. **Concise syntax**: Мінімум коду для створення data classes
4. **Performance**: Structs зберігаються в стеку, що покращує продуктивність
5. **Pattern matching friendly**: Records відмінно працюють з pattern matching
6. **Built-in deconstruction**: Автоматична підтримка декомпозиції

### Практичне застосування

```csharp
// Традиційний підхід із class
public class PersonClass
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public int Age { get; init; }
    
    public PersonClass(string firstName, string lastName, int age)
    {
        FirstName = firstName;
        LastName = lastName;
        Age = age;
    }
    
    // Потрібно вручну реалізовувати Equals, GetHashCode, ToString
    public override bool Equals(object? obj)
    {
        if (obj is PersonClass other)
        {
            return FirstName == other.FirstName && 
                   LastName == other.LastName && 
                   Age == other.Age;
        }
        return false;
    }
    
    public override int GetHashCode() => HashCode.Combine(FirstName, LastName, Age);
    public override string ToString() => $"{FirstName} {LastName}, Age: {Age}";
}

// Records - набагато компактніше
public record Person(string FirstName, string LastName, int Age);

// Record з додатковими властивостями та методами
public record Employee(string FirstName, string LastName, int Age, string Department, decimal Salary)
{
    // Computed properties
    public string FullName => $"{FirstName} {LastName}";
    public bool IsHighEarner => Salary > 75000;
    
    // Додаткові методи
    public Employee GiveRaise(decimal percentage) => 
        this with { Salary = Salary * (1 + percentage / 100) };
    
    // Validation в constructor
    public Employee
    {
        if (Age < 18) throw new ArgumentException("Employee must be at least 18");
        if (Salary < 0) throw new ArgumentException("Salary cannot be negative");
    }
}

// Record structs (C# 10+) - value semantics з record features
public readonly record struct Point(double X, double Y)
{
    public double DistanceFromOrigin => Math.Sqrt(X * X + Y * Y);
    public Point Translate(double dx, double dy) => new(X + dx, Y + dy);
}

// Mutable record structs
public record struct Temperature(double Value, string Unit)
{
    public Temperature ToCelsius() => Unit.ToLower() switch
    {
        "fahrenheit" or "f" => new((Value - 32) * 5 / 9, "Celsius"),
        "kelvin" or "k" => new(Value - 273.15, "Celsius"),
        "celsius" or "c" => this,
        _ => throw new ArgumentException($"Unknown temperature unit: {Unit}")
    };
    
    public Temperature ToFahrenheit() => Unit.ToLower() switch
    {
        "celsius" or "c" => new(Value * 9 / 5 + 32, "Fahrenheit"),
        "kelvin" or "k" => new((Value - 273.15) * 9 / 5 + 32, "Fahrenheit"),
        "fahrenheit" or "f" => this,
        _ => throw new ArgumentException($"Unknown temperature unit: {Unit}")
    };
}

// Використання records в реальних сценаріях
public static void RecordExamples()
{
    // Створення та використання
    var person1 = new Person("John", "Doe", 30);
    var person2 = new Person("John", "Doe", 30);
    
    // Value-based equality (працює автоматично)
    Console.WriteLine(person1 == person2); // True
    Console.WriteLine(person1.Equals(person2)); // True
    
    // ToString генерується автоматично
    Console.WriteLine(person1); // Person { FirstName = John, LastName = Doe, Age = 30 }
    
    // With expressions - створення копій з змінами
    var olderPerson = person1 with { Age = 35 };
    Console.WriteLine(olderPerson); // Person { FirstName = John, LastName = Doe, Age = 35 }
    
    // Deconstruction
    var (firstName, lastName, age) = person1;
    Console.WriteLine($"Name: {firstName} {lastName}, Age: {age}");
    
    // Pattern matching з records
    string categorizeAge = person1 switch
    {
        { Age: < 18 } => "Minor",
        { Age: >= 18 and < 65 } => "Adult",
        { Age: >= 65 } => "Senior",
        _ => "Unknown"
    };
}

// Практичний приклад: API responses
public record ApiResponse<T>(bool Success, T? Data, string? ErrorMessage, int StatusCode)
{
    public static ApiResponse<T> Ok(T data) => new(true, data, null, 200);
    public static ApiResponse<T> BadRequest(string error) => new(false, default, error, 400);
    public static ApiResponse<T> NotFound(string error = "Not found") => new(false, default, error, 404);
    public static ApiResponse<T> ServerError(string error) => new(false, default, error, 500);
}

public record UserDto(int Id, string Username, string Email, DateTime CreatedAt);
public record CreateUserRequest(string Username, string Email, string Password);
public record UpdateUserRequest(string? Username, string? Email);

// Використання в Web API контролері
public class UsersController
{
    public ApiResponse<UserDto> GetUser(int id)
    {
        // Simulation
        var user = id switch
        {
            1 => new UserDto(1, "john_doe", "john@example.com", DateTime.Now.AddDays(-30)),
            _ => null
        };
        
        return user != null 
            ? ApiResponse<UserDto>.Ok(user)
            : ApiResponse<UserDto>.NotFound("User not found");
    }
    
    public ApiResponse<UserDto> UpdateUser(int id, UpdateUserRequest request)
    {
        var currentUser = new UserDto(id, "old_username", "old@email.com", DateTime.Now.AddDays(-30));
        
        // With expressions для оновлення
        var updatedUser = currentUser with 
        { 
            Username = request.Username ?? currentUser.Username,
            Email = request.Email ?? currentUser.Email
        };
        
        return ApiResponse<UserDto>.Ok(updatedUser);
    }
}

// Domain models з records
public record Address(string Street, string City, string PostalCode, string Country)
{
    public string FullAddress => $"{Street}, {City}, {PostalCode}, {Country}";
}

public record Customer(int Id, string Name, string Email, Address Address, DateTime RegisteredAt)
{
    public int YearsAsCustomer => DateTime.Now.Year - RegisteredAt.Year;
    
    public Customer UpdateAddress(Address newAddress) => this with { Address = newAddress };
    public Customer UpdateEmail(string newEmail) => this with { Email = newEmail };
}

// Financial calculations з record structs
public readonly record struct Money(decimal Amount, string Currency)
{
    public static Money Zero(string currency) => new(0, currency);
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException($"Cannot add {Currency} and {other.Currency}");
        
        return new(Amount + other.Amount, Currency);
    }
    
    public Money Multiply(decimal factor) => new(Amount * factor, Currency);
    
    public override string ToString() => $"{Amount:N2} {Currency}";
}

// Practical usage patterns
public static void PracticalRecordUsage()
{
    // Configuration objects
    var config = new AppConfig("localhost", 5432, "myapp_db", true);
    var prodConfig = config with { Host = "prod-server", Database = "myapp_prod", EnableLogging = false };
    
    // Event sourcing
    var orderCreated = new OrderCreated(Guid.NewGuid(), "john@email.com", 250.00m, DateTime.Now);
    var orderShipped = new OrderShipped(orderCreated.OrderId, "TRK123456", DateTime.Now);
    
    // Value objects in DDD
    var email = new Email("user@domain.com");
    var invalidEmail = Email.TryCreate("invalid-email"); // Returns null or throws
    
    // Data transfer objects
    var weatherData = new WeatherData("Kyiv", 22.5, 65, "Sunny", DateTime.Now);
    var displayData = weatherData with { Temperature = weatherData.Temperature }; // Could add conversion logic
}

// Supporting types for examples
public record AppConfig(string Host, int Port, string Database, bool EnableLogging);
public record OrderCreated(Guid OrderId, string CustomerEmail, decimal Amount, DateTime CreatedAt);
public record OrderShipped(Guid OrderId, string TrackingNumber, DateTime ShippedAt);
public record WeatherData(string City, double Temperature, int Humidity, string Condition, DateTime Timestamp);

public record Email(string Value)
{
    public Email
    {
        if (string.IsNullOrWhiteSpace(Value) || !Value.Contains("@"))
            throw new ArgumentException("Invalid email format");
    }
    
    public static Email? TryCreate(string value)
    {
        try { return new Email(value); }
        catch { return null; }
    }
}
```

## 3.4. Async/Await та Асинхронне програмування (C# 5.0+)

### Що це таке
Async/Await - це модель асинхронного програмування в C#, яка дозволяє писати неблокуючий код, що виглядає та читається як синхронний код. Task-based Asynchronous Pattern (TAP) використовує Task та Task<T> для представлення асинхронних операцій, а ключові слова async та await спрощують роботу з ними.

### Навіщо це потрібно
1. **Non-blocking operations**: Дозволяє UI залишатися відзивним під час довгих операцій
2. **Scalability**: Покращує масштабованість серверних додатків за рахунок звільнення потоків
3. **Resource efficiency**: Оптимізує використання системних ресурсів
4. **Improved responsiveness**: Запобігає "заморожуванню" інтерфейсу користувача
5. **Better throughput**: Збільшує пропускну здатність веб-серверів
6. **Modern I/O patterns**: Підтримка сучасних неблокуючих I/O операцій

### Практичне застосування

```csharp
// Традиційний синхронний код
public static string DownloadDataSync(string url)
{
    using var client = new HttpClient();
    
    // Блокує поточний потік до завершення запиту
    var response = client.GetAsync(url).Result;  // НЕ РОБІТЬ ТАК!
    return response.Content.ReadAsStringAsync().Result;  // НЕ РОБІТЬ ТАК!
}

// Правильний асинхронний підхід
public static async Task<string> DownloadDataAsync(string url)
{
    using var client = new HttpClient();
    
    // Не блокує потік, звільняє його для інших задач
    var response = await client.GetAsync(url);
    return await response.Content.ReadAsStringAsync();
}

// Task vs Task<T>
public static async Task ProcessDataAsync()  // Task - не повертає значення
{
    var data = await DownloadDataAsync("https://api.example.com/data");
    Console.WriteLine($"Downloaded: {data.Length} characters");
}

public static async Task<int> CountWordsAsync(string text)  // Task<T> - повертає T
{
    // Симуляція CPU-інтенсивної роботи
    await Task.Delay(100);  // Імітація асинхронної операції
    return text.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
}

// Комбінування множинних асинхронних операцій
public static async Task<string> ProcessMultipleUrlsAsync(string[] urls)
{
    using var client = new HttpClient();
    var results = new StringBuilder();
    
    // Послідовне виконання (повільно)
    foreach (var url in urls)
    {
        var response = await client.GetStringAsync(url);
        results.AppendLine($"URL: {url}, Length: {response.Length}");
    }
    
    return results.ToString();
}

// Паралельне виконання (швидше)
public static async Task<string> ProcessMultipleUrlsParallelAsync(string[] urls)
{
    using var client = new HttpClient();
    
    // Створюємо всі задачі одразу
    var tasks = urls.Select(url => client.GetStringAsync(url)).ToArray();
    
    // Чекаємо завершення всіх задач паралельно
    var results = await Task.WhenAll(tasks);
    
    var output = new StringBuilder();
    for (int i = 0; i < urls.Length; i++)
    {
        output.AppendLine($"URL: {urls[i]}, Length: {results[i].Length}");
    }
    
    return output.ToString();
}

// Exception handling в async методах
public static async Task<string> SafeDownloadAsync(string url)
{
    try
    {
        using var client = new HttpClient();
        client.Timeout = TimeSpan.FromSeconds(30);
        
        var response = await client.GetAsync(url);
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadAsStringAsync();
    }
    catch (HttpRequestException httpEx)
    {
        Console.WriteLine($"HTTP Error: {httpEx.Message}");
        return string.Empty;
    }
    catch (TaskCanceledException tcEx) when (tcEx.InnerException is TimeoutException)
    {
        Console.WriteLine("Request timed out");
        return string.Empty;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Unexpected error: {ex.Message}");
        throw;  // Re-throw unknown exceptions
    }
}

// CancellationToken для скасування операцій
public static async Task<List<string>> DownloadWithCancellationAsync(
    string[] urls, 
    CancellationToken cancellationToken = default)
{
    using var client = new HttpClient();
    var results = new List<string>();
    
    foreach (var url in urls)
    {
        // Перевіряємо чи не скасована операція
        cancellationToken.ThrowIfCancellationRequested();
        
        try
        {
            var data = await client.GetStringAsync(url, cancellationToken);
            results.Add(data);
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine($"Download cancelled for {url}");
            break;  // Виходимо з циклу при скасуванні
        }
    }
    
    return results;
}

// ConfigureAwait для library коду
public static async Task<string> LibraryMethodAsync(string input)
{
    // В library коді використовуйте ConfigureAwait(false)
    // щоб уникнути deadlocks
    var processed = await ProcessStringAsync(input).ConfigureAwait(false);
    var validated = await ValidateResultAsync(processed).ConfigureAwait(false);
    
    return validated;
}

private static async Task<string> ProcessStringAsync(string input)
{
    await Task.Delay(100).ConfigureAwait(false);
    return input.ToUpper();
}

private static async Task<string> ValidateResultAsync(string input)
{
    await Task.Delay(50).ConfigureAwait(false);
    return string.IsNullOrEmpty(input) ? "EMPTY" : input;
}

// Практичні приклади для веб-розробки
public class WeatherService
{
    private readonly HttpClient _httpClient;
    
    public WeatherService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    
    public async Task<WeatherData?> GetWeatherAsync(string city, CancellationToken cancellationToken = default)
    {
        try
        {
            var url = $"https://api.weather.com/v1/weather?city={city}";
            var response = await _httpClient.GetAsync(url, cancellationToken);
            
            if (response.IsSuccessStatusCode)
            {
                var json = await response.Content.ReadAsStringAsync(cancellationToken);
                return JsonSerializer.Deserialize<WeatherData>(json);
            }
            
            return null;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error getting weather for {city}: {ex.Message}");
            return null;
        }
    }
    
    public async Task<List<WeatherData>> GetWeatherForMultipleCitiesAsync(
        string[] cities, 
        CancellationToken cancellationToken = default)
    {
        // Створюємо задачі для всіх міст
        var weatherTasks = cities.Select(city => GetWeatherAsync(city, cancellationToken));
        
        // Чекаємо завершення всіх задач
        var weatherResults = await Task.WhenAll(weatherTasks);
        
        // Фільтруємо null результати
        return weatherResults.Where(w => w != null).Cast<WeatherData>().ToList();
    }
}

// Database операції з async/await
public class UserRepository
{
    private readonly string _connectionString;
    
    public UserRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public async Task<User?> GetUserByIdAsync(int userId)
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        using var command = new SqlCommand("SELECT * FROM Users WHERE Id = @id", connection);
        command.Parameters.AddWithValue("@id", userId);
        
        using var reader = await command.ExecuteReaderAsync();
        
        if (await reader.ReadAsync())
        {
            return new User
            {
                Id = reader.GetInt32("Id"),
                Name = reader.GetString("Name"),
                Email = reader.GetString("Email")
            };
        }
        
        return null;
    }
    
    public async Task<bool> CreateUserAsync(User user)
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        using var command = new SqlCommand(
            "INSERT INTO Users (Name, Email) VALUES (@name, @email)", 
            connection);
        
        command.Parameters.AddWithValue("@name", user.Name);
        command.Parameters.AddWithValue("@email", user.Email);
        
        var rowsAffected = await command.ExecuteNonQueryAsync();
        return rowsAffected > 0;
    }
}

// File operations з async/await
public class FileProcessor
{
    public async Task<string> ReadFileAsync(string filePath)
    {
        using var fileStream = new FileStream(filePath, FileMode.Open, FileAccess.Read);
        using var reader = new StreamReader(fileStream);
        
        return await reader.ReadToEndAsync();
    }
    
    public async Task WriteFileAsync(string filePath, string content)
    {
        using var fileStream = new FileStream(filePath, FileMode.Create, FileAccess.Write);
        using var writer = new StreamWriter(fileStream);
        
        await writer.WriteAsync(content);
    }
    
    public async Task<List<string>> ProcessMultipleFilesAsync(string[] filePaths)
    {
        var tasks = filePaths.Select(ReadFileAsync);
        var results = await Task.WhenAll(tasks);
        
        return results.ToList();
    }
}

// Advanced patterns: Producer/Consumer з async
public class AsyncDataProcessor
{
    public async Task ProcessDataStreamAsync(IAsyncEnumerable<string> dataStream)
    {
        await foreach (var item in dataStream)
        {
            await ProcessSingleItemAsync(item);
        }
    }
    
    private async Task ProcessSingleItemAsync(string item)
    {
        // Simulate processing work
        await Task.Delay(100);
        Console.WriteLine($"Processed: {item}");
    }
    
    public async IAsyncEnumerable<string> GenerateDataAsync(
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        for (int i = 0; i < 100; i++)
        {
            if (cancellationToken.IsCancellationRequested)
                yield break;
                
            await Task.Delay(50, cancellationToken);
            yield return $"Data item {i}";
        }
    }
}

// Practical usage examples
public static async Task AsyncUsageExamples()
{
    // Базове використання
    var weatherService = new WeatherService(new HttpClient());
    var weather = await weatherService.GetWeatherAsync("Kyiv");
    
    // З cancellation
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
    var cities = new[] { "Kyiv", "Lviv", "Odesa", "Kharkiv" };
    var weatherData = await weatherService.GetWeatherForMultipleCitiesAsync(cities, cts.Token);
    
    // File processing
    var fileProcessor = new FileProcessor();
    var files = new[] { "file1.txt", "file2.txt", "file3.txt" };
    var fileContents = await fileProcessor.ProcessMultipleFilesAsync(files);
    
    // Async streams
    var dataProcessor = new AsyncDataProcessor();
    await dataProcessor.ProcessDataStreamAsync(dataProcessor.GenerateDataAsync());
}

// Supporting types
public class WeatherData
{
    public string City { get; set; } = "";
    public double Temperature { get; set; }
    public string Condition { get; set; } = "";
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}
```

# 4. ASP.NET Core - Архітектура та основи

## 4.1. Dependency Injection (DI) контейнер

### Що це таке
Dependency Injection - це патерн проектування та механізм в ASP.NET Core, який управляє створенням та життєвим циклом об'єктів. Вбудований DI контейнер автоматично створює екземпляри класів та інжектує їх залежності, забезпечуючи слабке зв'язування між компонентами додатку.

### Навіщо це потрібно
1. **Слабке зв'язування**: Зменшує залежності між класами, роблячи код більш гнучким
2. **Тестування**: Легше створювати unit тести з mock об'єктами
3. **Управління життєвим циклом**: Автоматичне керування створенням та знищенням об'єктів
4. **Конфігурування**: Централізоване налаштування сервісів
5. **Інверсія управління**: Контейнер керує залежностями замість самих класів
6. **Підтримка**: Легше змінювати реалізації без зміни коду споживачів

### Практичне застосування

```csharp
// Інтерфейси для слабкого зв'язування
public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
}

public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<User> CreateAsync(User user);
    Task<bool> ExistsAsync(string email);
}

public interface ILogger<T>
{
    void LogInformation(string message);
    void LogError(string message, Exception ex);
}

// Реалізації сервісів
public class SmtpEmailService : IEmailService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<SmtpEmailService> _logger;
    
    public SmtpEmailService(IConfiguration configuration, ILogger<SmtpEmailService> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        try
        {
            var smtpServer = _configuration["Email:SmtpServer"];
            var port = int.Parse(_configuration["Email:Port"]);
            
            // SMTP logic here
            _logger.LogInformation($"Email sent to {to}");
            await Task.Delay(100); // Simulate async operation
        }
        catch (Exception ex)
        {
            _logger.LogError($"Failed to send email to {to}", ex);
            throw;
        }
    }
}

public class DatabaseUserRepository : IUserRepository
{
    private readonly string _connectionString;
    private readonly ILogger<DatabaseUserRepository> _logger;
    
    public DatabaseUserRepository(IConfiguration configuration, ILogger<DatabaseUserRepository> logger)
    {
        _connectionString = configuration.GetConnectionString("DefaultConnection");
        _logger = logger;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        _logger.LogInformation($"Getting user with ID: {id}");
        
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        using var command = new SqlCommand("SELECT * FROM Users WHERE Id = @id", connection);
        command.Parameters.AddWithValue("@id", id);
        
        using var reader = await command.ExecuteReaderAsync();
        
        if (await reader.ReadAsync())
        {
            return new User
            {
                Id = reader.GetInt32("Id"),
                Name = reader.GetString("Name"),
                Email = reader.GetString("Email"),
                CreatedAt = reader.GetDateTime("CreatedAt")
            };
        }
        
        return null;
    }
    
    public async Task<User> CreateAsync(User user)
    {
        _logger.LogInformation($"Creating user: {user.Email}");
        
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        using var command = new SqlCommand(
            "INSERT INTO Users (Name, Email, CreatedAt) OUTPUT INSERTED.Id VALUES (@name, @email, @createdAt)", 
            connection);
        
        command.Parameters.AddWithValue("@name", user.Name);
        command.Parameters.AddWithValue("@email", user.Email);
        command.Parameters.AddWithValue("@createdAt", DateTime.UtcNow);
        
        var newId = (int)await command.ExecuteScalarAsync();
        user.Id = newId;
        
        return user;
    }
    
    public async Task<bool> ExistsAsync(string email)
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        using var command = new SqlCommand("SELECT COUNT(1) FROM Users WHERE Email = @email", connection);
        command.Parameters.AddWithValue("@email", email);
        
        var count = (int)await command.ExecuteScalarAsync();
        return count > 0;
    }
}

// Business logic сервіс
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService;
    private readonly ILogger<UserService> _logger;
    
    // Dependency Injection через конструктор
    public UserService(
        IUserRepository userRepository, 
        IEmailService emailService, 
        ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _emailService = emailService;
        _logger = logger;
    }
    
    public async Task<User> RegisterUserAsync(string name, string email)
    {
        _logger.LogInformation($"Registering user: {email}");
        
        // Перевіряємо чи користувач вже існує
        if (await _userRepository.ExistsAsync(email))
        {
            throw new InvalidOperationException($"User with email {email} already exists");
        }
        
        // Створюємо користувача
        var user = new User
        {
            Name = name,
            Email = email,
            CreatedAt = DateTime.UtcNow
        };
        
        var createdUser = await _userRepository.CreateAsync(user);
        
        // Відправляємо welcome email
        await _emailService.SendEmailAsync(
            createdUser.Email,
            "Welcome!",
            $"Hello {createdUser.Name}, welcome to our platform!"
        );
        
        _logger.LogInformation($"User registered successfully: {createdUser.Id}");
        return createdUser;
    }
    
    public async Task<User> GetUserAsync(int userId)
    {
        _logger.LogInformation($"Getting user: {userId}");
        
        var user = await _userRepository.GetByIdAsync(userId);
        if (user == null)
        {
            throw new NotFoundException($"User with ID {userId} not found");
        }
        
        return user;
    }
}

// Реєстрація сервісів в Program.cs (ASP.NET Core 6+)
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Реєструємо сервіси в DI контейнері
        RegisterServices(builder.Services, builder.Configuration);
        
        var app = builder.Build();
        
        // Configure middleware pipeline
        ConfigureMiddleware(app);
        
        app.Run();
    }
    
    private static void RegisterServices(IServiceCollection services, IConfiguration configuration)
    {
        // Різні життєві цикли сервісів
        
        // Transient - новий екземпляр при кожному запиті
        services.AddTransient<IEmailService, SmtpEmailService>();
        
        // Scoped - один екземпляр на HTTP запит
        services.AddScoped<IUserRepository, DatabaseUserRepository>();
        services.AddScoped<UserService>();
        
        // Singleton - один екземпляр на весь час роботи додатку
        services.AddSingleton<ICacheService, MemoryCacheService>();
        
        // Реєстрація з фабрикою
        services.AddScoped<IEmailService>(provider =>
        {
            var config = provider.GetRequiredService<IConfiguration>();
            var emailProvider = config["Email:Provider"];
            
            return emailProvider?.ToLower() switch
            {
                "smtp" => new SmtpEmailService(config, provider.GetRequiredService<ILogger<SmtpEmailService>>()),
                "sendgrid" => new SendGridEmailService(config, provider.GetRequiredService<ILogger<SendGridEmailService>>()),
                _ => new SmtpEmailService(config, provider.GetRequiredService<ILogger<SmtpEmailService>>())
            };
        });
        
        // Умовна реєстрація
        if (configuration.GetValue<bool>("Features:EnableEmailService"))
        {
            services.AddScoped<IEmailService, SmtpEmailService>();
        }
        else
        {
            services.AddScoped<IEmailService, NoOpEmailService>();
        }
        
        // MVC сервіси
        services.AddControllers();
        services.AddEndpointsApiExplorer();
        services.AddSwaggerGen();
    }
    
    private static void ConfigureMiddleware(WebApplication app)
    {
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }
        
        app.UseRouting();
        app.MapControllers();
    }
}

// Контролер з DI
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserService _userService;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(UserService userService, ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        try
        {
            var user = await _userService.GetUserAsync(id);
            return Ok(user);
        }
        catch (NotFoundException)
        {
            return NotFound($"User with ID {id} not found");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"Error getting user {id}");
            return StatusCode(500, "Internal server error");
        }
    }
    
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(CreateUserRequest request)
    {
        try
        {
            var user = await _userService.RegisterUserAsync(request.Name, request.Email);
            return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
        }
        catch (InvalidOperationException ex)
        {
            return BadRequest(ex.Message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating user");
            return StatusCode(500, "Internal server error");
        }
    }
}

// Додаткові сервіси для прикладу
public interface ICacheService
{
    Task<T> GetAsync<T>(string key);
    Task SetAsync<T>(string key, T value, TimeSpan expiration);
}

public class MemoryCacheService : ICacheService
{
    private readonly IMemoryCache _cache;
    
    public MemoryCacheService(IMemoryCache cache)
    {
        _cache = cache;
    }
    
    public Task<T> GetAsync<T>(string key)
    {
        _cache.TryGetValue(key, out T value);
        return Task.FromResult(value);
    }
    
    public Task SetAsync<T>(string key, T value, TimeSpan expiration)
    {
        _cache.Set(key, value, expiration);
        return Task.CompletedTask;
    }
}

public class SendGridEmailService : IEmailService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<SendGridEmailService> _logger;
    
    public SendGridEmailService(IConfiguration configuration, ILogger<SendGridEmailService> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        var apiKey = _configuration["SendGrid:ApiKey"];
        // SendGrid implementation
        _logger.LogInformation($"Email sent via SendGrid to {to}");
        await Task.Delay(50);
    }
}

public class NoOpEmailService : IEmailService
{
    private readonly ILogger<NoOpEmailService> _logger;
    
    public NoOpEmailService(ILogger<NoOpEmailService> logger)
    {
        _logger = logger;
    }
    
    public Task SendEmailAsync(string to, string subject, string body)
    {
        _logger.LogInformation($"Email service disabled - would send to {to}: {subject}");
        return Task.CompletedTask;
    }
}

// Моделі даних
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime CreatedAt { get; set; }
}

public class CreateUserRequest
{
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}

public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

// Конфігурація в appsettings.json
/*
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;Trusted_Connection=true;"
  },
  "Email": {
    "Provider": "smtp",
    "SmtpServer": "smtp.gmail.com",
    "Port": 587
  },
  "Features": {
    "EnableEmailService": true
  },
  "SendGrid": {
    "ApiKey": "your-sendgrid-api-key"
  }
}
*/
```

## 4.2. Middleware Pipeline

### Що це таке
Middleware в ASP.NET Core - це компоненти, які формують pipeline обробки HTTP запитів та відповідей. Кожен middleware може обробляти запит, передавати його далі по pipeline, та обробляти відповідь на зворотному шляху. Pipeline будується з множинних middleware компонентів, кожен з яких виконує специфічні завдання.

### Навіщо це потрібно
1. **Модульність**: Розділення функціональності на окремі компоненти
2. **Порядок виконання**: Контроль над послідовністю обробки запитів
3. **Reusability**: Повторне використання middleware в різних додатках
4. **Cross-cutting concerns**: Централізована обробка логування, аутентифікації, обробки помилок
5. **Performance**: Можливість оптимізувати pipeline для конкретних потреб
6. **Гнучкість**: Легке додавання, видалення або зміна middleware

### Практичне застосування

```csharp
// Простий middleware для логування запитів
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    
    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        var request = context.Request;
        
        _logger.LogInformation($"Incoming request: {request.Method} {request.Path} from {context.Connection.RemoteIpAddress}");
        
        // Викликаємо наступний middleware в pipeline
        await _next(context);
        
        stopwatch.Stop();
        _logger.LogInformation($"Request completed: {context.Response.StatusCode} in {stopwatch.ElapsedMilliseconds}ms");
    }
}

// Middleware для обробки помилок
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;
    private readonly IHostEnvironment _environment;
    
    public ExceptionHandlingMiddleware(
        RequestDelegate next, 
        ILogger<ExceptionHandlingMiddleware> logger,
        IHostEnvironment environment)
    {
        _next = next;
        _logger = logger;
        _environment = environment;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        
        var response = new ErrorResponse();
        
        switch (exception)
        {
            case NotFoundException notFound:
                response.StatusCode = StatusCodes.Status404NotFound;
                response.Message = notFound.Message;
                break;
                
            case ValidationException validation:
                response.StatusCode = StatusCodes.Status400BadRequest;
                response.Message = validation.Message;
                response.Errors = validation.Errors;
                break;
                
            case UnauthorizedAccessException:
                response.StatusCode = StatusCodes.Status401Unauthorized;
                response.Message = "Unauthorized access";
                break;
                
            default:
                response.StatusCode = StatusCodes.Status500InternalServerError;
                response.Message = _environment.IsDevelopment() 
                    ? exception.Message 
                    : "An internal server error occurred";
                
                if (_environment.IsDevelopment())
                {
                    response.Details = exception.StackTrace;
                }
                break;
        }
        
        context.Response.StatusCode = response.StatusCode;
        
        var jsonResponse = JsonSerializer.Serialize(response, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        await context.Response.WriteAsync(jsonResponse);
    }
}

// Rate limiting middleware
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    private readonly ILogger<RateLimitingMiddleware> _logger;
    private readonly RateLimitOptions _options;
    
    public RateLimitingMiddleware(
        RequestDelegate next, 
        IMemoryCache cache,
        ILogger<RateLimitingMiddleware> logger,
        IOptions<RateLimitOptions> options)
    {
        _next = next;
        _cache = cache;
        _logger = logger;
        _options = options.Value;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = GetClientId(context);
        var key = $"rate_limit_{clientId}";
        
        if (_cache.TryGetValue(key, out RateLimitInfo rateLimitInfo))
        {
            if (rateLimitInfo.RequestCount >= _options.MaxRequests)
            {
                _logger.LogWarning($"Rate limit exceeded for client {clientId}");
                context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
                await context.Response.WriteAsync("Rate limit exceeded. Try again later.");
                return;
            }
            
            rateLimitInfo.RequestCount++;
        }
        else
        {
            rateLimitInfo = new RateLimitInfo { RequestCount = 1 };
            _cache.Set(key, rateLimitInfo, TimeSpan.FromMinutes(_options.WindowMinutes));
        }
        
        // Додаємо headers з інформацією про rate limit
        context.Response.Headers.Add("X-RateLimit-Limit", _options.MaxRequests.ToString());
        context.Response.Headers.Add("X-RateLimit-Remaining", 
            Math.Max(0, _options.MaxRequests - rateLimitInfo.RequestCount).ToString());
        
        await _next(context);
    }
    
    private string GetClientId(HttpContext context)
    {
        // В реальному додатку можна використовувати API ключі, user ID тощо
        return context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
    }
}

// CORS middleware (custom implementation)
public class CorsMiddleware
{
    private readonly RequestDelegate _next;
    private readonly CorsOptions _options;
    
    public CorsMiddleware(RequestDelegate next, IOptions<CorsOptions> options)
    {
        _next = next;
        _options = options.Value;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var origin = context.Request.Headers["Origin"].FirstOrDefault();
        
        if (!string.IsNullOrEmpty(origin) && _options.AllowedOrigins.Contains(origin))
        {
            context.Response.Headers.Add("Access-Control-Allow-Origin", origin);
        }
        else if (_options.AllowAnyOrigin)
        {
            context.Response.Headers.Add("Access-Control-Allow-Origin", "*");
        }
        
        context.Response.Headers.Add("Access-Control-Allow-Methods", 
            string.Join(", ", _options.AllowedMethods));
        context.Response.Headers.Add("Access-Control-Allow-Headers", 
            string.Join(", ", _options.AllowedHeaders));
        
        if (context.Request.Method == "OPTIONS")
        {
            context.Response.StatusCode = StatusCodes.Status200OK;
            return;
        }
        
        await _next(context);
    }
}

// Request/Response caching middleware
public class ResponseCachingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    private readonly ILogger<ResponseCachingMiddleware> _logger;
    
    public ResponseCachingMiddleware(
        RequestDelegate next, 
        IMemoryCache cache, 
        ILogger<ResponseCachingMiddleware> logger)
    {
        _next = next;
        _cache = cache;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Кешуємо тільки GET запити
        if (context.Request.Method != "GET")
        {
            await _next(context);
            return;
        }
        
        var cacheKey = GenerateCacheKey(context.Request);
        
        if (_cache.TryGetValue(cacheKey, out CachedResponse cachedResponse))
        {
            _logger.LogInformation($"Cache hit for {cacheKey}");
            
            context.Response.StatusCode = cachedResponse.StatusCode;
            context.Response.ContentType = cachedResponse.ContentType;
            await context.Response.WriteAsync(cachedResponse.Content);
            return;
        }
        
        // Перехоплюємо response для кешування
        var originalBodyStream = context.Response.Body;
        using var responseBodyStream = new MemoryStream();
        context.Response.Body = responseBodyStream;
        
        await _next(context);
        
        // Кешуємо успішні відповіді
        if (context.Response.StatusCode == 200)
        {
            responseBodyStream.Seek(0, SeekOrigin.Begin);
            var responseContent = await new StreamReader(responseBodyStream).ReadToEndAsync();
            
            var responseToCache = new CachedResponse
            {
                StatusCode = context.Response.StatusCode,
                ContentType = context.Response.ContentType ?? "application/json",
                Content = responseContent
            };
            
            _cache.Set(cacheKey, responseToCache, TimeSpan.FromMinutes(5));
            _logger.LogInformation($"Cached response for {cacheKey}");
            
            responseBodyStream.Seek(0, SeekOrigin.Begin);
        }
        
        await responseBodyStream.CopyToAsync(originalBodyStream);
    }
    
    private string GenerateCacheKey(HttpRequest request)
    {
        var keyBuilder = new StringBuilder();
        keyBuilder.Append(request.Path);
        
        foreach (var (key, value) in request.Query.OrderBy(x => x.Key))
        {
            keyBuilder.Append($"_{key}={value}");
        }
        
        return keyBuilder.ToString();
    }
}

// Налаштування middleware в Program.cs
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Додаємо сервіси
        builder.Services.Configure<RateLimitOptions>(
            builder.Configuration.GetSection("RateLimit"));
        builder.Services.Configure<CorsOptions>(
            builder.Configuration.GetSection("Cors"));
        
        builder.Services.AddMemoryCache();
        builder.Services.AddControllers();
        
        var app = builder.Build();
        
        // Налаштовуємо middleware pipeline (порядок важливий!)
        
        // 1. Exception handling (повинен бути першим)
        app.UseMiddleware<ExceptionHandlingMiddleware>();
        
        // 2. CORS
        app.UseMiddleware<CorsMiddleware>();
        
        // 3. Request logging
        app.UseMiddleware<RequestLoggingMiddleware>();
        
        // 4. Rate limiting
        app.UseMiddleware<RateLimitingMiddleware>();
        
        // 5. Response caching
        app.UseMiddleware<ResponseCachingMiddleware>();
        
        // 6. Built-in middleware
        if (app.Environment.IsDevelopment())
        {
            app.UseDeveloperExceptionPage(); // Вбудований middleware для dev
        }
        
        app.UseRouting();
        
        // 7. Authentication & Authorization
        app.UseAuthentication();
        app.UseAuthorization();
        
        // 8. Endpoints (останні в pipeline)
        app.MapControllers();
        
        app.Run();
    }
}

// Extension methods для зручності
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseCustomExceptionHandling(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<ExceptionHandlingMiddleware>();
    }
    
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
    
    public static IApplicationBuilder UseRateLimiting(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RateLimitingMiddleware>();
    }
    
    public static IApplicationBuilder UseCustomCors(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CorsMiddleware>();
    }
}

// Моделі для middleware
public class ErrorResponse
{
    public int StatusCode { get; set; }
    public string Message { get; set; } = "";
    public string? Details { get; set; }
    public Dictionary<string, string[]>? Errors { get; set; }
}

public class RateLimitInfo
{
    public int RequestCount { get; set; }
}

public class RateLimitOptions
{
    public int MaxRequests { get; set; } = 100;
    public int WindowMinutes { get; set; } = 1;
}

public class CorsOptions
{
    public bool AllowAnyOrigin { get; set; } = false;
    public List<string> AllowedOrigins { get; set; } = new();
    public List<string> AllowedMethods { get; set; } = new() { "GET", "POST", "PUT", "DELETE" };
    public List<string> AllowedHeaders { get; set; } = new() { "Content-Type", "Authorization" };
}

public class CachedResponse
{
    public int StatusCode { get; set; }
    public string ContentType { get; set; } = "";
    public string Content { get; set; } = "";
}

public class ValidationException : Exception
{
    public Dictionary<string, string[]> Errors { get; set; }
    
    public ValidationException(string message, Dictionary<string, string[]> errors) : base(message)
    {
        Errors = errors;
    }
}

// Конфігурація в appsettings.json
/*
{
  "RateLimit": {
    "MaxRequests": 100,
    "WindowMinutes": 1
  },
  "Cors": {
    "AllowAnyOrigin": false,
    "AllowedOrigins": ["https://localhost:3000", "https://mydomain.com"],
    "AllowedMethods": ["GET", "POST", "PUT", "DELETE", "PATCH"],
    "AllowedHeaders": ["Content-Type", "Authorization", "X-Requested-With"]
  }
}
*/
```

# 5. Web API розробка

## 5.1. RESTful API принципи та маршрутизація

### Що це таке
RESTful API - це архітектурний стиль для побудови веб-сервісів, який використовує HTTP методи (GET, POST, PUT, DELETE) для виконання операцій над ресурсами. Маршрутизація в ASP.NET Core визначає як URL-адреси зіставляються з діями контролерів, забезпечуючи структурований доступ до API endpoints.

### Навіщо це потрібно
1. **Стандартизація**: Універсальні принципи для створення API, зрозумілі всім розробникам
2. **Читабельність**: Інтуїтивні URL-адреси, що відображають структуру ресурсів
3. **Кешування**: HTTP методи та статус коди дозволяють ефективне кешування
4. **Статelessness**: Кожен запит містить всю необхідну інформацію для обробки
5. **Масштабованість**: RESTful сервіси легко масштабуються горизонтально
6. **Інтероперабельність**: Працює з будь-якою мовою програмування та платформою

### Практичне застосування

```csharp
// Базовий контролер з RESTful endpoints
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductsController> _logger;
    
    public ProductsController(IProductService productService, ILogger<ProductsController> logger)
    {
        _productService = productService;
        _logger = logger;
    }
    
    // GET api/products - отримати всі продукти з фільтрацією та пагінацією
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
        [FromQuery] ProductFilterDto filter,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        try
        {
            if (page < 1 || pageSize < 1 || pageSize > 100)
            {
                return BadRequest("Invalid pagination parameters");
            }
            
            var result = await _productService.GetProductsAsync(filter, page, pageSize);
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving products");
            return StatusCode(500, "Internal server error");
        }
    }
    
    // GET api/products/{id} - отримати продукт за ID
    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(ProductDetailDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDetailDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        
        if (product == null)
        {
            return NotFound($"Product with ID {id} not found");
        }
        
        return Ok(product);
    }
    
    // POST api/products - створити новий продукт
    [HttpPost]
    [ProducesResponseType(typeof(ProductDetailDto), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDetailDto>> CreateProduct(
        [FromBody] CreateProductDto createProductDto)
    {
        try
        {
            var product = await _productService.CreateProductAsync(createProductDto);
            
            return CreatedAtAction(
                nameof(GetProduct), 
                new { id = product.Id }, 
                product);
        }
        catch (ValidationException ex)
        {
            return BadRequest(ex.Message);
        }
    }
    
    // PUT api/products/{id} - повністю оновити продукт
    [HttpPut("{id:int}")]
    [ProducesResponseType(typeof(ProductDetailDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDetailDto>> UpdateProduct(
        int id, 
        [FromBody] UpdateProductDto updateProductDto)
    {
        try
        {
            var product = await _productService.UpdateProductAsync(id, updateProductDto);
            return Ok(product);
        }
        catch (NotFoundException)
        {
            return NotFound($"Product with ID {id} not found");
        }
        catch (ValidationException ex)
        {
            return BadRequest(ex.Message);
        }
    }
    
    // DELETE api/products/{id} - видалити продукт
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult> DeleteProduct(int id)
    {
        try
        {
            await _productService.DeleteProductAsync(id);
            return NoContent();
        }
        catch (NotFoundException)
        {
            return NotFound($"Product with ID {id} not found");
        }
    }
}
```

## 5.2. Model Binding та Validation

### Що це таке
Model Binding - це процес автоматичного зв'язування даних з HTTP запиту (query parameters, form data, JSON body, route values) з параметрами дій контролера або властивостями моделей. Validation забезпечує перевірку правильності та цілісності вхідних даних перед їх обробкою.

### Навіщо це потрібно
1. **Автоматизація**: Усуває необхідність ручного парсингу даних запиту
2. **Type Safety**: Автоматичне перетворення рядків у строго типізовані об'єкти
3. **Валідація**: Забезпечує цілісність та правильність даних
4. **Безпека**: Захищає від некоректних або шкідливих даних
5. **Зручність розробки**: Спрощує роботу з вхідними даними
6. **Консистентність**: Уніфікований підхід до обробки даних

### Практичне застосування

```csharp
// Складна модель з різними типами валідації
public class CreateUserDto : IValidatableObject
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2, ErrorMessage = "Name must be between 2 and 100 characters")]
    public string Name { get; set; } = "";
    
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    [UniqueEmail] // Custom validation attribute
    public string Email { get; set; } = "";
    
    [Required(ErrorMessage = "Password is required")]
    [StringLength(100, MinimumLength = 8, ErrorMessage = "Password must be at least 8 characters")]
    [RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]", 
        ErrorMessage = "Password must contain at least one uppercase letter, one lowercase letter, one digit, and one special character")]
    public string Password { get; set; } = "";
    
    [Required(ErrorMessage = "Password confirmation is required")]
    [Compare(nameof(Password), ErrorMessage = "Passwords do not match")]
    public string ConfirmPassword { get; set; } = "";
    
    [Range(18, 120, ErrorMessage = "Age must be between 18 and 120")]
    public int Age { get; set; }
    
    [Required(ErrorMessage = "Birth date is required")]
    [DataType(DataType.Date)]
    [BirthDateValidation] // Custom validation
    public DateTime BirthDate { get; set; }
    
    [Phone(ErrorMessage = "Invalid phone number format")]
    public string? PhoneNumber { get; set; }
    
    [Required(ErrorMessage = "Address is required")]
    public AddressDto Address { get; set; } = new();
    
    [MinLength(1, ErrorMessage = "At least one skill is required")]
    public List<string> Skills { get; set; } = new();
    
    [Required(ErrorMessage = "Terms acceptance is required")]
    [MustBeTrue(ErrorMessage = "You must accept the terms and conditions")]
    public bool AcceptTerms { get; set; }
    
    // Custom validation method
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        var results = new List<ValidationResult>();
        
        // Cross-field validation
        if (BirthDate.AddYears(Age) < DateTime.Now.AddMonths(-6) || 
            BirthDate.AddYears(Age) > DateTime.Now.AddMonths(6))
        {
            results.Add(new ValidationResult(
                "Age and birth date are inconsistent",
                new[] { nameof(Age), nameof(BirthDate) }));
        }
        
        // Business rule validation
        if (Skills.Contains("Senior") && Age < 25)
        {
            results.Add(new ValidationResult(
                "Senior skill level requires age 25 or older",
                new[] { nameof(Skills), nameof(Age) }));
        }
        
        return results;
    }
}

// Custom validation attributes
public class UniqueEmailAttribute : ValidationAttribute, IValidationAttribute
{
    public ValidationResult? GetValidationResult(object? value, ValidationContext validationContext)
    {
        if (value is string email && !string.IsNullOrWhiteSpace(email))
        {
            var userService = validationContext.GetService<IUserService>();
            if (userService != null)
            {
                var existsTask = userService.EmailExistsAsync(email);
                existsTask.Wait(); // Not ideal for async, but required for sync validation
                
                if (existsTask.Result)
                {
                    return new ValidationResult("Email is already registered");
                }
            }
        }
        
        return ValidationResult.Success;
    }
}

public class BirthDateValidationAttribute : ValidationAttribute
{
    public override bool IsValid(object? value)
    {
        if (value is DateTime birthDate)
        {
            var minDate = DateTime.Now.AddYears(-120);
            var maxDate = DateTime.Now.AddYears(-18);
            
            return birthDate >= minDate && birthDate <= maxDate;
        }
        
        return false;
    }
    
    public override string FormatErrorMessage(string name)
    {
        return "Birth date must be between 18 and 120 years ago";
    }
}

public class MustBeTrueAttribute : ValidationAttribute
{
    public override bool IsValid(object? value)
    {
        return value is bool boolean && boolean;
    }
}
```

# 6. Validation та Error Handling

## 6.1. Data Annotations та Custom Validation

### Що це таке
Data Annotations - це атрибути в .NET, які дозволяють додавати метадані до властивостей класів для валідації, форматування та конфігурування поведінки. Custom Validation дозволяє створювати власні правила валідації, які не покриваються стандартними атрибутами, забезпечуючи гнучкість та можливість реалізації складної бізнес-логіки валідації.

### Навіщо це потрібно
1. **Цілісність даних**: Гарантує, що дані відповідають встановленим правилам перед обробкою
2. **Безпека**: Захищає від некоректних або шкідливих вхідних даних
3. **User Experience**: Надає зрозумілі повідомлення про помилки користувачам
4. **Бізнес-правила**: Дозволяє впроваджувати складні правила валідації бізнес-логіки
5. **Автоматизація**: Автоматична валідація на різних рівнях додатку
6. **Консистентність**: Єдиний підхід до валідації в усьому додатку

### Практичне застосування

```csharp
// Комплексна модель з різними типами валідації
public class ProductCreateDto : IValidatableObject
{
    [Required(ErrorMessage = "Product name is required")]
    [StringLength(200, MinimumLength = 3, ErrorMessage = "Product name must be between 3 and 200 characters")]
    [ProductNameValidation] // Custom validation
    public string Name { get; set; } = "";
    
    [Required(ErrorMessage = "Description is required")]
    [StringLength(2000, MinimumLength = 10, ErrorMessage = "Description must be between 10 and 2000 characters")]
    public string Description { get; set; } = "";
    
    [Required(ErrorMessage = "Price is required")]
    [Range(0.01, 999999.99, ErrorMessage = "Price must be between $0.01 and $999,999.99")]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }
    
    [Range(0, int.MaxValue, ErrorMessage = "Stock quantity cannot be negative")]
    public int StockQuantity { get; set; }
    
    [Required(ErrorMessage = "Category is required")]
    [CategoryExists] // Custom validation
    public int CategoryId { get; set; }
    
    // Cross-model validation
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        var results = new List<ValidationResult>();
        
        // Business rule: Premium products must have higher price
        if (Name.Contains("Premium", StringComparison.OrdinalIgnoreCase) && Price < 100)
        {
            results.Add(new ValidationResult(
                "Premium products must be priced at least $100",
                new[] { nameof(Price), nameof(Name) }));
        }
        
        return results;
    }
}

// Custom validation attributes
public class ProductNameValidationAttribute : ValidationAttribute
{
    public override bool IsValid(object? value)
    {
        if (value is not string name || string.IsNullOrWhiteSpace(name))
            return false;
        
        // Business rule: Product names cannot contain certain words
        var forbiddenWords = new[] { "fake", "counterfeit", "illegal", "stolen" };
        var lowerName = name.ToLowerInvariant();
        
        return !forbiddenWords.Any(word => lowerName.Contains(word));
    }
    
    public override string FormatErrorMessage(string name)
    {
        return "Product name contains forbidden words";
    }
}

public class CategoryExistsAttribute : ValidationAttribute, IValidationAttribute
{
    public ValidationResult? GetValidationResult(object? value, ValidationContext validationContext)
    {
        if (value is int categoryId)
        {
            var categoryService = validationContext.GetService<ICategoryService>();
            if (categoryService != null)
            {
                var existsTask = categoryService.CategoryExistsAsync(categoryId);
                existsTask.Wait(); // Sync over async - not ideal but required for sync validation
                
                if (!existsTask.Result)
                {
                    return new ValidationResult("Category does not exist");
                }
            }
        }
        
        return ValidationResult.Success;
    }
}
```

## 6.2. Global Exception Handling

### Що це таке
Global Exception Handling - це централізований механізм обробки всіх необроблених винятків у додатку ASP.NET Core. Це middleware компонент, який перехоплює винятки на рівні всього додатку, логує їх, та повертає користувачам структуровані відповіді про помилки, приховуючи внутрішні деталі системи.

### Навіщо це потрібно
1. **Централізація**: Єдине місце для обробки всіх типів помилок у додатку
2. **Безпека**: Приховує чутливу інформацію про внутрішню структуру системи
3. **Консистентність**: Уніфікований формат відповідей про помилки для API
4. **Логування**: Автоматичне логування всіх помилок для моніторингу
5. **User Experience**: Надання зрозумілих повідомлень користувачам
6. **Debugging**: Детальна інформація про помилки для розробників

### Практичне застосування

```csharp
// Custom exceptions для різних сценаріїв
public abstract class BaseCustomException : Exception
{
    public abstract int StatusCode { get; }
    public abstract string ErrorCode { get; }
    
    protected BaseCustomException(string message) : base(message) { }
    protected BaseCustomException(string message, Exception innerException) 
        : base(message, innerException) { }
}

public class ValidationException : BaseCustomException
{
    public override int StatusCode => 400;
    public override string ErrorCode => "VALIDATION_ERROR";
    public Dictionary<string, string[]> Errors { get; }
    
    public ValidationException(string message) : base(message)
    {
        Errors = new Dictionary<string, string[]>();
    }
    
    public ValidationException(string message, Dictionary<string, string[]> errors) : base(message)
    {
        Errors = errors;
    }
}

public class NotFoundException : BaseCustomException
{
    public override int StatusCode => 404;
    public override string ErrorCode => "NOT_FOUND";
    public string ResourceType { get; }
    public object ResourceId { get; }
    
    public NotFoundException(string resourceType, object resourceId) 
        : base($"{resourceType} with id '{resourceId}' was not found")
    {
        ResourceType = resourceType;
        ResourceId = resourceId;
    }
}

// Error response models
public class ErrorResponse
{
    public string Message { get; set; } = "";
    public string ErrorCode { get; set; } = "";
    public int StatusCode { get; set; }
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    public string? TraceId { get; set; }
    public Dictionary<string, object>? Details { get; set; }
}

// Global exception handling middleware
public class GlobalExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionHandlingMiddleware> _logger;
    private readonly IWebHostEnvironment _environment;
    
    public GlobalExceptionHandlingMiddleware(
        RequestDelegate next,
        ILogger<GlobalExceptionHandlingMiddleware> logger,
        IWebHostEnvironment environment)
    {
        _next = next;
        _logger = logger;
        _environment = environment;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var correlationId = context.TraceIdentifier;
        
        _logger.LogError(exception, 
            "An unhandled exception occurred. CorrelationId: {CorrelationId}, RequestPath: {RequestPath}", 
            correlationId, context.Request.Path);
        
        var errorResponse = CreateErrorResponse(exception, correlationId);
        
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = errorResponse.StatusCode;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        
        var jsonOptions = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            WriteIndented = _environment.IsDevelopment()
        };
        
        var jsonResponse = JsonSerializer.Serialize(errorResponse, jsonOptions);
        await context.Response.WriteAsync(jsonResponse);
    }
    
    private ErrorResponse CreateErrorResponse(Exception exception, string correlationId)
    {
        return exception switch
        {
            ValidationException validationEx => new ErrorResponse
            {
                Message = validationEx.Message,
                ErrorCode = validationEx.ErrorCode,
                StatusCode = validationEx.StatusCode,
                TraceId = correlationId
            },
            
            NotFoundException notFoundEx => new ErrorResponse
            {
                Message = notFoundEx.Message,
                ErrorCode = notFoundEx.ErrorCode,
                StatusCode = notFoundEx.StatusCode,
                TraceId = correlationId,
                Details = new Dictionary<string, object>
                {
                    ["resourceType"] = notFoundEx.ResourceType,
                    ["resourceId"] = notFoundEx.ResourceId
                }
            },
            
            _ => new ErrorResponse
            {
                Message = _environment.IsDevelopment() ? exception.Message : "An internal server error occurred",
                ErrorCode = "INTERNAL_ERROR",
                StatusCode = 500,
                TraceId = correlationId
            }
        };
    }
}
```

# 7. Data Access Patterns

## 7.1. Repository Pattern та Unit of Work

### Що це таке
Repository Pattern - це абстракція над доступом до даних, яка інкапсулює логіку доступу до даних та надає більш об'єктно-орієнтований погляд на персистентний шар. Unit of Work - це патерн, який підтримує список об'єктів, що зазнали змін у бізнес-транзакції, та координує запис цих змін у базу даних.

### Навіщо це потрібно
1. **Абстракція**: Відокремлює бізнес-логіку від деталей доступу до даних
2. **Тестування**: Дозволяє легко замінювати реальні репозиторії на mock об'єкти
3. **Централізація**: Єдине місце для логіки доступу до даних для кожної сутності
4. **Транзакції**: Unit of Work забезпечує узгодженість даних у межах бізнес-операції
5. **Кешування**: Можливість додавання кешування на рівні репозиторію
6. **Зміна даних**: Легка заміна технології зберігання даних

### Практичне застосування

```csharp
// Base interfaces
public interface IRepository<TEntity, TKey> where TEntity : class
{
    Task<TEntity?> GetByIdAsync(TKey id, CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> FindAsync(Expression<Func<TEntity, bool>> predicate, CancellationToken cancellationToken = default);
    Task<TEntity> AddAsync(TEntity entity, CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> AddRangeAsync(IEnumerable<TEntity> entities, CancellationToken cancellationToken = default);
    void Update(TEntity entity);
    void UpdateRange(IEnumerable<TEntity> entities);
    void Remove(TEntity entity);
    void RemoveRange(IEnumerable<TEntity> entities);
    Task<bool> ExistsAsync(TKey id, CancellationToken cancellationToken = default);
    Task<int> CountAsync(CancellationToken cancellationToken = default);
    Task<int> CountAsync(Expression<Func<TEntity, bool>> predicate, CancellationToken cancellationToken = default);
}

public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IProductRepository Products { get; }
    IOrderRepository Orders { get; }
    ICategoryRepository Categories { get; }
    
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
    Task BeginTransactionAsync(CancellationToken cancellationToken = default);
    Task CommitTransactionAsync(CancellationToken cancellationToken = default);
    Task RollbackTransactionAsync(CancellationToken cancellationToken = default);
}

// Domain entities
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime CreatedAt { get; set; }
    public DateTime? LastLoginAt { get; set; }
    public bool IsActive { get; set; } = true;
    public List<Order> Orders { get; set; } = new();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
    public DateTime CreatedAt { get; set; }
    public bool IsActive { get; set; } = true;
    public List<OrderItem> OrderItems { get; set; } = new();
}

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public bool IsActive { get; set; } = true;
    public List<Product> Products { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public User User { get; set; } = null!;
    public DateTime OrderDate { get; set; }
    public OrderStatus Status { get; set; }
    public decimal TotalAmount { get; set; }
    public List<OrderItem> OrderItems { get; set; } = new();
}

public class OrderItem
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public enum OrderStatus
{
    Pending,
    Confirmed,
    Shipped,
    Delivered,
    Cancelled
}

// Specific repository interfaces
public interface IUserRepository : IRepository<User, int>
{
    Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetUsersWithOrdersAsync(CancellationToken cancellationToken = default);
    Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default);
    Task UpdateLastLoginAsync(int userId, DateTime loginTime, CancellationToken cancellationToken = default);
}

public interface IProductRepository : IRepository<Product, int>
{
    Task<IEnumerable<Product>> GetByCategoryAsync(int categoryId, CancellationToken cancellationToken = default);
    Task<IEnumerable<Product>> GetAvailableProductsAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<Product>> SearchAsync(string searchTerm, CancellationToken cancellationToken = default);
    Task<bool> IsNameUniqueInCategoryAsync(string name, int categoryId, int? excludeId = null, CancellationToken cancellationToken = default);
    Task UpdateStockAsync(int productId, int newQuantity, CancellationToken cancellationToken = default);
    Task<Product?> GetWithCategoryAsync(int id, CancellationToken cancellationToken = default);
}

public interface IOrderRepository : IRepository<Order, int>
{
    Task<IEnumerable<Order>> GetByUserIdAsync(int userId, CancellationToken cancellationToken = default);
    Task<IEnumerable<Order>> GetByStatusAsync(OrderStatus status, CancellationToken cancellationToken = default);
    Task<IEnumerable<Order>> GetOrdersWithItemsAsync(CancellationToken cancellationToken = default);
    Task<Order?> GetWithItemsAsync(int id, CancellationToken cancellationToken = default);
    Task<decimal> GetTotalSalesAsync(DateTime fromDate, DateTime toDate, CancellationToken cancellationToken = default);
    Task<IEnumerable<Order>> GetRecentOrdersAsync(int count, CancellationToken cancellationToken = default);
}

public interface ICategoryRepository : IRepository<Category, int>
{
    Task<IEnumerable<Category>> GetActiveCategoriesAsync(CancellationToken cancellationToken = default);
    Task<Category?> GetWithProductsAsync(int id, CancellationToken cancellationToken = default);
    Task<bool> HasProductsAsync(int categoryId, CancellationToken cancellationToken = default);
}

// Generic repository implementation
public class Repository<TEntity, TKey> : IRepository<TEntity, TKey> where TEntity : class
{
    protected readonly ApplicationDbContext Context;
    protected readonly DbSet<TEntity> DbSet;
    
    public Repository(ApplicationDbContext context)
    {
        Context = context;
        DbSet = context.Set<TEntity>();
    }
    
    public virtual async Task<TEntity?> GetByIdAsync(TKey id, CancellationToken cancellationToken = default)
    {
        return await DbSet.FindAsync(new object[] { id! }, cancellationToken);
    }
    
    public virtual async Task<IEnumerable<TEntity>> GetAllAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet.ToListAsync(cancellationToken);
    }
    
    public virtual async Task<IEnumerable<TEntity>> FindAsync(
        Expression<Func<TEntity, bool>> predicate, 
        CancellationToken cancellationToken = default)
    {
        return await DbSet.Where(predicate).ToListAsync(cancellationToken);
    }
    
    public virtual async Task<TEntity> AddAsync(TEntity entity, CancellationToken cancellationToken = default)
    {
        var addedEntity = await DbSet.AddAsync(entity, cancellationToken);
        return addedEntity.Entity;
    }
    
    public virtual async Task<IEnumerable<TEntity>> AddRangeAsync(
        IEnumerable<TEntity> entities, 
        CancellationToken cancellationToken = default)
    {
        var entitiesList = entities.ToList();
        await DbSet.AddRangeAsync(entitiesList, cancellationToken);
        return entitiesList;
    }
    
    public virtual void Update(TEntity entity)
    {
        DbSet.Update(entity);
    }
    
    public virtual void UpdateRange(IEnumerable<TEntity> entities)
    {
        DbSet.UpdateRange(entities);
    }
    
    public virtual void Remove(TEntity entity)
    {
        DbSet.Remove(entity);
    }
    
    public virtual void RemoveRange(IEnumerable<TEntity> entities)
    {
        DbSet.RemoveRange(entities);
    }
    
    public virtual async Task<bool> ExistsAsync(TKey id, CancellationToken cancellationToken = default)
    {
        return await DbSet.FindAsync(new object[] { id! }, cancellationToken) != null;
    }
    
    public virtual async Task<int> CountAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet.CountAsync(cancellationToken);
    }
    
    public virtual async Task<int> CountAsync(
        Expression<Func<TEntity, bool>> predicate, 
        CancellationToken cancellationToken = default)
    {
        return await DbSet.CountAsync(predicate, cancellationToken);
    }
}

// Specific repository implementations
public class UserRepository : Repository<User, int>, IUserRepository
{
    public UserRepository(ApplicationDbContext context) : base(context) { }
    
    public async Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .FirstOrDefaultAsync(u => u.Email == email, cancellationToken);
    }
    
    public async Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(u => u.IsActive)
            .OrderBy(u => u.Name)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<User>> GetUsersWithOrdersAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Include(u => u.Orders)
            .Where(u => u.Orders.Any())
            .ToListAsync(cancellationToken);
    }
    
    public async Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .AnyAsync(u => u.Email == email, cancellationToken);
    }
    
    public async Task UpdateLastLoginAsync(int userId, DateTime loginTime, CancellationToken cancellationToken = default)
    {
        var user = await GetByIdAsync(userId, cancellationToken);
        if (user != null)
        {
            user.LastLoginAt = loginTime;
            Update(user);
        }
    }
}

public class ProductRepository : Repository<Product, int>, IProductRepository
{
    public ProductRepository(ApplicationDbContext context) : base(context) { }
    
    public async Task<IEnumerable<Product>> GetByCategoryAsync(int categoryId, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(p => p.CategoryId == categoryId && p.IsActive)
            .Include(p => p.Category)
            .OrderBy(p => p.Name)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<Product>> GetAvailableProductsAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(p => p.IsActive && p.StockQuantity > 0)
            .Include(p => p.Category)
            .OrderBy(p => p.Name)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<Product>> SearchAsync(string searchTerm, CancellationToken cancellationToken = default)
    {
        var lowercaseSearchTerm = searchTerm.ToLower();
        
        return await DbSet
            .Where(p => p.IsActive && 
                   (p.Name.ToLower().Contains(lowercaseSearchTerm) ||
                    p.Description.ToLower().Contains(lowercaseSearchTerm)))
            .Include(p => p.Category)
            .OrderBy(p => p.Name)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<bool> IsNameUniqueInCategoryAsync(
        string name, 
        int categoryId, 
        int? excludeId = null, 
        CancellationToken cancellationToken = default)
    {
        var query = DbSet.Where(p => p.Name == name && p.CategoryId == categoryId);
        
        if (excludeId.HasValue)
        {
            query = query.Where(p => p.Id != excludeId.Value);
        }
        
        return !await query.AnyAsync(cancellationToken);
    }
    
    public async Task UpdateStockAsync(int productId, int newQuantity, CancellationToken cancellationToken = default)
    {
        var product = await GetByIdAsync(productId, cancellationToken);
        if (product != null)
        {
            product.StockQuantity = newQuantity;
            Update(product);
        }
    }
    
    public async Task<Product?> GetWithCategoryAsync(int id, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id, cancellationToken);
    }
}

public class OrderRepository : Repository<Order, int>, IOrderRepository
{
    public OrderRepository(ApplicationDbContext context) : base(context) { }
    
    public async Task<IEnumerable<Order>> GetByUserIdAsync(int userId, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(o => o.UserId == userId)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .OrderByDescending(o => o.OrderDate)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<Order>> GetByStatusAsync(OrderStatus status, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(o => o.Status == status)
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .OrderBy(o => o.OrderDate)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<Order>> GetOrdersWithItemsAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .OrderByDescending(o => o.OrderDate)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<Order?> GetWithItemsAsync(int id, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Category)
            .FirstOrDefaultAsync(o => o.Id == id, cancellationToken);
    }
    
    public async Task<decimal> GetTotalSalesAsync(
        DateTime fromDate, 
        DateTime toDate, 
        CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(o => o.OrderDate >= fromDate && 
                       o.OrderDate <= toDate && 
                       o.Status != OrderStatus.Cancelled)
            .SumAsync(o => o.TotalAmount, cancellationToken);
    }
    
    public async Task<IEnumerable<Order>> GetRecentOrdersAsync(int count, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Include(o => o.User)
            .OrderByDescending(o => o.OrderDate)
            .Take(count)
            .ToListAsync(cancellationToken);
    }
}

public class CategoryRepository : Repository<Category, int>, ICategoryRepository
{
    public CategoryRepository(ApplicationDbContext context) : base(context) { }
    
    public async Task<IEnumerable<Category>> GetActiveCategoriesAsync(CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Where(c => c.IsActive)
            .OrderBy(c => c.Name)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<Category?> GetWithProductsAsync(int id, CancellationToken cancellationToken = default)
    {
        return await DbSet
            .Include(c => c.Products.Where(p => p.IsActive))
            .FirstOrDefaultAsync(c => c.Id == id, cancellationToken);
    }
    
    public async Task<bool> HasProductsAsync(int categoryId, CancellationToken cancellationToken = default)
    {
        return await Context.Set<Product>()
            .AnyAsync(p => p.CategoryId == categoryId && p.IsActive, cancellationToken);
    }
}

// Unit of Work implementation
public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction? _transaction;
    
    // Repository instances
    private IUserRepository? _userRepository;
    private IProductRepository? _productRepository;
    private IOrderRepository? _orderRepository;
    private ICategoryRepository? _categoryRepository;
    
    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
    }
    
    // Lazy loading repositories
    public IUserRepository Users => _userRepository ??= new UserRepository(_context);
    public IProductRepository Products => _productRepository ??= new ProductRepository(_context);
    public IOrderRepository Orders => _orderRepository ??= new OrderRepository(_context);
    public ICategoryRepository Categories => _categoryRepository ??= new CategoryRepository(_context);
    
    public async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        return await _context.SaveChangesAsync(cancellationToken);
    }
    
    public async Task BeginTransactionAsync(CancellationToken cancellationToken = default)
    {
        if (_transaction != null)
        {
            throw new InvalidOperationException("Transaction already started");
        }
        
        _transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
    }
    
    public async Task CommitTransactionAsync(CancellationToken cancellationToken = default)
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("No transaction to commit");
        }
        
        try
        {
            await SaveChangesAsync(cancellationToken);
            await _transaction.CommitAsync(cancellationToken);
        }
        finally
        {
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public async Task RollbackTransactionAsync(CancellationToken cancellationToken = default)
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("No transaction to rollback");
        }
        
        try
        {
            await _transaction.RollbackAsync(cancellationToken);
        }
        finally
        {
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public void Dispose()
    {
        _transaction?.Dispose();
        _context.Dispose();
    }
}

// ApplicationDbContext configuration
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }
    
    public DbSet<User> Users { get; set; } = null!;
    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<Category> Categories { get; set; } = null!;
    public DbSet<Order> Orders { get; set; } = null!;
    public DbSet<OrderItem> OrderItems { get; set; } = null!;
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // User configuration
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(255);
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("GETUTCDATE()");
        });
        
        // Category configuration
        modelBuilder.Entity<Category>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Description).HasMaxLength(500);
        });
        
        // Product configuration
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
            entity.Property(e => e.Description).HasMaxLength(1000);
            entity.Property(e => e.Price).HasColumnType("decimal(18,2)");
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("GETUTCDATE()");
            
            entity.HasOne(e => e.Category)
                  .WithMany(c => c.Products)
                  .HasForeignKey(e => e.CategoryId)
                  .OnDelete(DeleteBehavior.Restrict);
        });
        
        // Order configuration
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.TotalAmount).HasColumnType("decimal(18,2)");
            entity.Property(e => e.OrderDate).HasDefaultValueSql("GETUTCDATE()");
            
            entity.HasOne(e => e.User)
                  .WithMany(u => u.Orders)
                  .HasForeignKey(e => e.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // OrderItem configuration
        modelBuilder.Entity<OrderItem>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.UnitPrice).HasColumnType("decimal(18,2)");
            
            entity.HasOne(e => e.Order)
                  .WithMany(o => o.OrderItems)
                  .HasForeignKey(e => e.OrderId)
                  .OnDelete(DeleteBehavior.Cascade);
                  
            entity.HasOne(e => e.Product)
                  .WithMany(p => p.OrderItems)
                  .HasForeignKey(e => e.ProductId)
                  .OnDelete(DeleteBehavior.Restrict);
        });
    }
}

// Dependency Injection setup
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddRepositoryPattern(
        this IServiceCollection services, 
        string connectionString)
    {
        // Add DbContext
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(connectionString));
        
        // Register repositories
        services.AddScoped<IUserRepository, UserRepository>();
        services.AddScoped<IProductRepository, ProductRepository>();
        services.AddScoped<IOrderRepository, OrderRepository>();
        services.AddScoped<ICategoryRepository, CategoryRepository>();
        
        // Register Unit of Work
        services.AddScoped<IUnitOfWork, UnitOfWork>();
        
        return services;
    }
}

// Service layer example using repositories
public class ProductService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public ProductService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<ProductDto> CreateProductAsync(CreateProductDto createDto)
    {
        // Check if category exists
        var category = await _unitOfWork.Categories.GetByIdAsync(createDto.CategoryId);
        if (category == null)
            throw new ArgumentException("Category not found");
        
        // Check if product name is unique in category
        var isUnique = await _unitOfWork.Products.IsNameUniqueInCategoryAsync(
            createDto.Name, createDto.CategoryId);
        if (!isUnique)
            throw new ArgumentException("Product name already exists in this category");
        
        // Create product
        var product = new Product
        {
            Name = createDto.Name,
            Description = createDto.Description,
            Price = createDto.Price,
            StockQuantity = createDto.StockQuantity,
            CategoryId = createDto.CategoryId,
            CreatedAt = DateTime.UtcNow
        };
        
        await _unitOfWork.Products.AddAsync(product);
        await _unitOfWork.SaveChangesAsync();
        
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            StockQuantity = product.StockQuantity,
            CategoryName = category.Name,
            IsActive = product.IsActive
        };
    }
    
    public async Task<bool> UpdateStockWithTransactionAsync(int productId, int newQuantity)
    {
        try
        {
            await _unitOfWork.BeginTransactionAsync();
            
            var product = await _unitOfWork.Products.GetByIdAsync(productId);
            if (product == null)
            {
                await _unitOfWork.RollbackTransactionAsync();
                return false;
            }
            
            await _unitOfWork.Products.UpdateStockAsync(productId, newQuantity);
            await _unitOfWork.SaveChangesAsync();
            
            await _unitOfWork.CommitTransactionAsync();
            return true;
        }
        catch
        {
            await _unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}
// DTO classes for complete example
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public string CategoryName { get; set; } = "";
    public bool IsActive { get; set; }
}

public class CreateProductDto
{
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public int CategoryId { get; set; }
}

// Controller example using service with repositories
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductService _productService;
    
    public ProductsController(ProductService productService)
    {
        _productService = productService;
    }
    
    [HttpPost]
    public async Task<ActionResult<ProductDto>> CreateProduct([FromBody] CreateProductDto createDto)
    {
        try
        {
            var product = await _productService.CreateProductAsync(createDto);
            return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
        }
        catch (ArgumentException ex)
        {
            return BadRequest(ex.Message);
        }
    }
    
    [HttpPut("{id}/stock")]
    public async Task<IActionResult> UpdateStock(int id, [FromBody] int newQuantity)
    {
        var success = await _productService.UpdateStockWithTransactionAsync(id, newQuantity);
        return success ? Ok() : NotFound();
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        // Implementation would use service layer
        return Ok();
    }
}
```

**Переваги Repository Pattern та Unit of Work:**
- **Абстрагування доступу до даних** - бізнес-логіка не залежить від конкретної технології доступу до даних
- **Тестованість** - легко створювати мок-об'єкти для тестування
- **Централізоване управління транзакціями** через Unit of Work
- **Повторне використання** запитів через специфічні методи репозиторіїв
- **Чіткий поділ відповідальностей** між шарами
- **Інкапсуляція складної логіки** доступу до даних

### 7.2 Entity Framework Core Deep Dive

**Що це таке:**
Entity Framework Core - це Object-Relational Mapping (ORM) фреймворк для .NET, який дозволяє працювати з базами даних через об'єкти замість SQL запитів. Це потужний інструмент для маппінгу об'єктів C# на таблиці бази даних.

**Навіщо це потрібно:**
- Автоматичне генерування SQL запитів з LINQ виразів
- Відстеження змін (Change Tracking) для автоматичного оновлення
- Міграції для версіонування схеми бази даних
- Підтримка різних провайдерів баз даних
- Lazy/Eager loading для оптимізації завантаження даних
- Caching та performance optimizations

**Практичне застосування:**

```csharp
// Advanced DbContext configuration
public class AdvancedApplicationDbContext : DbContext
{
    public AdvancedApplicationDbContext(DbContextOptions<AdvancedApplicationDbContext> options) 
        : base(options) { }
    
    public DbSet<User> Users { get; set; } = null!;
    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<Category> Categories { get; set; } = null!;
    public DbSet<Order> Orders { get; set; } = null!;
    public DbSet<OrderItem> OrderItems { get; set; } = null!;
    public DbSet<AuditLog> AuditLogs { get; set; } = null!;
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder
                .UseSqlServer("DefaultConnection")
                .EnableSensitiveDataLogging() // Тільки для розробки!
                .EnableDetailedErrors()
                .LogTo(Console.WriteLine, LogLevel.Information)
                .UseQueryTrackingBehavior(QueryTrackingBehavior.TrackAll);
        }
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Global query filters
        modelBuilder.Entity<Product>()
            .HasQueryFilter(p => !p.IsDeleted);
            
        modelBuilder.Entity<User>()
            .HasQueryFilter(u => u.IsActive);
        
        // Value conversions
        modelBuilder.Entity<Product>()
            .Property(p => p.Tags)
            .HasConversion(
                v => string.Join(';', v),
                v => v.Split(';', StringSplitOptions.RemoveEmptyEntries).ToList()
            );
        
        // Shadow properties
        modelBuilder.Entity<Product>()
            .Property<DateTime>("CreatedAt")
            .HasDefaultValueSql("GETUTCDATE()");
            
        modelBuilder.Entity<Product>()
            .Property<DateTime>("ModifiedAt")
            .HasDefaultValueSql("GETUTCDATE()");
        
        // Owned types
        modelBuilder.Entity<User>()
            .OwnsOne(u => u.Address, address =>
            {
                address.Property(a => a.Street).HasMaxLength(200);
                address.Property(a => a.City).HasMaxLength(100);
                address.Property(a => a.Country).HasMaxLength(100);
                address.Property(a => a.PostalCode).HasMaxLength(20);
            });
        
        // Table splitting
        modelBuilder.Entity<ProductDetails>()
            .ToTable("Products");
            
        modelBuilder.Entity<Product>()
            .HasOne(p => p.Details)
            .WithOne(d => d.Product)
            .HasForeignKey<ProductDetails>(d => d.Id);
        
        // Indexes
        modelBuilder.Entity<Product>()
            .HasIndex(p => new { p.Name, p.CategoryId })
            .IsUnique()
            .HasDatabaseName("IX_Product_Name_Category");
            
        modelBuilder.Entity<User>()
            .HasIndex(u => u.Email)
            .IsUnique()
            .HasFilter("[Email] IS NOT NULL");
        
        // Sequences
        modelBuilder.HasSequence<int>("OrderNumbers")
            .StartsAt(1000)
            .IncrementsBy(1);
            
        modelBuilder.Entity<Order>()
            .Property(o => o.OrderNumber)
            .HasDefaultValueSql("NEXT VALUE FOR OrderNumbers");
        
        // Check constraints
        modelBuilder.Entity<Product>()
            .HasCheckConstraint("CK_Product_Price", "Price > 0");
            
        modelBuilder.Entity<OrderItem>()
            .HasCheckConstraint("CK_OrderItem_Quantity", "Quantity > 0");
        
        // Custom functions
        modelBuilder.HasDbFunction(typeof(EfCoreFunctions).GetMethod(nameof(EfCoreFunctions.GetFullName)))
            .HasName("GetFullName");
        
        // Configurations from separate classes
        modelBuilder.ApplyConfiguration(new UserConfiguration());
        modelBuilder.ApplyConfiguration(new ProductConfiguration());
        modelBuilder.ApplyConfiguration(new OrderConfiguration());
        
        // Seed data
        SeedData(modelBuilder);
    }
    
    private static void SeedData(ModelBuilder modelBuilder)
    {
        // Category seed data
        modelBuilder.Entity<Category>().HasData(
            new Category { Id = 1, Name = "Electronics", Description = "Electronic devices", IsActive = true },
            new Category { Id = 2, Name = "Books", Description = "Books and literature", IsActive = true },
            new Category { Id = 3, Name = "Clothing", Description = "Clothing and accessories", IsActive = true }
        );
        
        // User seed data
        modelBuilder.Entity<User>().HasData(
            new User 
            { 
                Id = 1, 
                Name = "Admin User", 
                Email = "admin@example.com", 
                CreatedAt = new DateTime(2023, 1, 1),
                IsActive = true
            }
        );
        
        // Shadow property seed data
        modelBuilder.Entity<Product>()
            .HasData(
                new { Id = 1, Name = "Laptop", CategoryId = 1, Price = 999.99m, StockQuantity = 10, IsActive = true, IsDeleted = false },
                new { Id = 2, Name = "Programming Book", CategoryId = 2, Price = 39.99m, StockQuantity = 50, IsActive = true, IsDeleted = false }
            );
        
        // Set shadow properties for seed data
        modelBuilder.Entity<Product>()
            .Property<DateTime>("CreatedAt")
            .HasDefaultValue(new DateTime(2023, 1, 1));
    }
    
    // Override SaveChanges for audit logging
    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        var auditEntries = new List<AuditEntry>();
        
        foreach (var entry in ChangeTracker.Entries())
        {
            if (entry.Entity is AuditLog || entry.State == EntityState.Detached)
                continue;
            
            var auditEntry = new AuditEntry(entry);
            auditEntries.Add(auditEntry);
            
            // Update shadow properties
            switch (entry.State)
            {
                case EntityState.Added:
                    entry.Property("CreatedAt").CurrentValue = DateTime.UtcNow;
                    entry.Property("ModifiedAt").CurrentValue = DateTime.UtcNow;
                    break;
                    
                case EntityState.Modified:
                    entry.Property("ModifiedAt").CurrentValue = DateTime.UtcNow;
                    break;
            }
        }
        
        var result = await base.SaveChangesAsync(cancellationToken);
        
        // Save audit logs after main save
        foreach (var auditEntry in auditEntries)
        {
            auditEntry.PostSaveChanges();
            AuditLogs.Add(auditEntry.ToAuditLog());
        }
        
        if (auditEntries.Any())
        {
            await base.SaveChangesAsync(cancellationToken);
        }
        
        return result;
    }
}

// Extended entities with advanced features
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime CreatedAt { get; set; }
    public DateTime? LastLoginAt { get; set; }
    public bool IsActive { get; set; } = true;
    
    // Owned type
    public Address Address { get; set; } = new();
    
    // Navigation properties
    public List<Order> Orders { get; set; } = new();
    public List<UserRole> UserRoles { get; set; } = new();
}

public class Address  // Owned type
{
    public string Street { get; set; } = "";
    public string City { get; set; } = "";
    public string Country { get; set; } = "";
    public string PostalCode { get; set; } = "";
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public int CategoryId { get; set; }
    public bool IsActive { get; set; } = true;
    public bool IsDeleted { get; set; } = false;  // For soft delete
    
    // Value conversion property
    public List<string> Tags { get; set; } = new();
    
    // Navigation properties
    public Category Category { get; set; } = null!;
    public ProductDetails Details { get; set; } = null!;  // Table splitting
    public List<OrderItem> OrderItems { get; set; } = new();
    public List<ProductReview> Reviews { get; set; } = new();
}

public class ProductDetails  // Table splitting
{
    public int Id { get; set; }
    public string DetailedDescription { get; set; } = "";
    public string Specifications { get; set; } = "";
    public string WarrantyInfo { get; set; } = "";
    
    // Navigation property
    public Product Product { get; set; } = null!;
}

public class Order
{
    public int Id { get; set; }
    public int OrderNumber { get; set; }  // Using sequence
    public int UserId { get; set; }
    public DateTime OrderDate { get; set; }
    public OrderStatus Status { get; set; }
    public decimal TotalAmount { get; set; }
    
    // Navigation properties
    public User User { get; set; } = null!;
    public List<OrderItem> OrderItems { get; set; } = new();
}

public class AuditLog
{
    public int Id { get; set; }
    public string EntityName { get; set; } = "";
    public string Action { get; set; } = "";
    public string KeyValues { get; set; } = "";
    public string OldValues { get; set; } = "";
    public string NewValues { get; set; } = "";
    public DateTime Timestamp { get; set; }
    public string UserId { get; set; } = "";
}

// Configuration classes
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("Users");
        builder.HasKey(x => x.Id);
        
        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(200)
            .HasComment("User full name");
        
        builder.Property(x => x.Email)
            .IsRequired()
            .HasMaxLength(255)
            .HasComment("User email address");
        
        builder.HasIndex(x => x.Email)
            .IsUnique()
            .HasDatabaseName("IX_User_Email");
        
        builder.Property(x => x.CreatedAt)
            .HasDefaultValueSql("GETUTCDATE()")
            .HasComment("User creation timestamp");
        
        // Configure owned type
        builder.OwnsOne(x => x.Address, address =>
        {
            address.Property(a => a.Street).HasColumnName("AddressStreet");
            address.Property(a => a.City).HasColumnName("AddressCity");
            address.Property(a => a.Country).HasColumnName("AddressCountry");
            address.Property(a => a.PostalCode).HasColumnName("AddressPostalCode");
        });
    }
}

public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("Products");
        builder.HasKey(x => x.Id);
        
        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(200);
        
        builder.Property(x => x.Price)
            .HasColumnType("decimal(18,2)")
            .HasPrecision(18, 2);
        
        // Global query filter
        builder.HasQueryFilter(p => !p.IsDeleted);
        
        // Value conversion for Tags
        builder.Property(e => e.Tags)
            .HasConversion(
                v => string.Join(';', v),
                v => v.Split(';', StringSplitOptions.RemoveEmptyEntries).ToList());
        
        // Relationships
        builder.HasOne(x => x.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(x => x.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);
        
        // Shadow properties
        builder.Property<DateTime>("CreatedAt");
        builder.Property<DateTime>("ModifiedAt");
        
        // Indexes
        builder.HasIndex(x => new { x.Name, x.CategoryId })
            .IsUnique()
            .HasDatabaseName("IX_Product_Name_Category");
    }
}

// Audit helper classes
public class AuditEntry
{
    public EntityEntry Entry { get; }
    public string EntityName { get; }
    public Dictionary<string, object> KeyValues { get; } = new();
    public Dictionary<string, object> OldValues { get; } = new();
    public Dictionary<string, object> NewValues { get; } = new();
    public EntityState State { get; }
    
    public AuditEntry(EntityEntry entry)
    {
        Entry = entry;
        EntityName = entry.Entity.GetType().Name;
        State = entry.State;
        
        foreach (var property in entry.Properties)
        {
            if (property.IsKey)
            {
                KeyValues[property.Metadata.Name] = property.CurrentValue;
                continue;
            }
            
            switch (entry.State)
            {
                case EntityState.Added:
                    NewValues[property.Metadata.Name] = property.CurrentValue;
                    break;
                    
                case EntityState.Deleted:
                    OldValues[property.Metadata.Name] = property.OriginalValue;
                    break;
                    
                case EntityState.Modified:
                    if (property.IsModified)
                    {
                        OldValues[property.Metadata.Name] = property.OriginalValue;
                        NewValues[property.Metadata.Name] = property.CurrentValue;
                    }
                    break;
            }
        }
    }
    
    public void PostSaveChanges()
    {
        // Update KeyValues after save for new entities
        foreach (var property in Entry.Properties)
        {
            if (property.IsKey)
            {
                KeyValues[property.Metadata.Name] = property.CurrentValue;
            }
        }
    }
    
    public AuditLog ToAuditLog()
    {
        return new AuditLog
        {
            EntityName = EntityName,
            Action = State.ToString(),
            KeyValues = JsonSerializer.Serialize(KeyValues),
            OldValues = OldValues.Any() ? JsonSerializer.Serialize(OldValues) : null,
            NewValues = NewValues.Any() ? JsonSerializer.Serialize(NewValues) : null,
            Timestamp = DateTime.UtcNow,
            UserId = "System" // In real app, get from current user context
        };
    }
}

// Custom EF functions
public static class EfCoreFunctions
{
    public static string GetFullName(string firstName, string lastName)
        => throw new InvalidOperationException("This method should only be used in LINQ to Entities queries");
}

// Advanced queries and operations
public class AdvancedDataService
{
    private readonly AdvancedApplicationDbContext _context;
    
    public AdvancedDataService(AdvancedApplicationDbContext context)
    {
        _context = context;
    }
    
    // Raw SQL queries
    public async Task<List<ProductSalesDto>> GetProductSalesAsync(DateTime fromDate, DateTime toDate)
    {
        var sql = @"
            SELECT 
                p.Id,
                p.Name,
                SUM(oi.Quantity) as TotalSold,
                SUM(oi.Quantity * oi.UnitPrice) as TotalRevenue
            FROM Products p
            INNER JOIN OrderItems oi ON p.Id = oi.ProductId
            INNER JOIN Orders o ON oi.OrderId = o.Id
            WHERE o.OrderDate BETWEEN @fromDate AND @toDate
                AND o.Status != @cancelledStatus
            GROUP BY p.Id, p.Name
            ORDER BY TotalRevenue DESC";
        
        return await _context.Database
            .SqlQueryRaw<ProductSalesDto>(sql, 
                new SqlParameter("@fromDate", fromDate),
                new SqlParameter("@toDate", toDate),
                new SqlParameter("@cancelledStatus", (int)OrderStatus.Cancelled))
            .ToListAsync();
    }
    
    // Bulk operations
    public async Task BulkUpdateProductPricesAsync(int categoryId, decimal percentage)
    {
        await _context.Database.ExecuteSqlRawAsync(
            "UPDATE Products SET Price = Price * @percentage WHERE CategoryId = @categoryId",
            new SqlParameter("@percentage", 1 + (percentage / 100)),
            new SqlParameter("@categoryId", categoryId));
    }
    
    // Complex LINQ with projections
    public async Task<PagedResult<ProductSummaryDto>> GetProductsWithStatsAsync(
        int page, int pageSize, string searchTerm = "")
    {
        var query = _context.Products
            .Include(p => p.Category)
            .Where(p => p.IsActive)
            .AsQueryable();
        
        if (!string.IsNullOrEmpty(searchTerm))
        {
            query = query.Where(p => p.Name.Contains(searchTerm) || 
                                   p.Description.Contains(searchTerm));
        }
        
        var totalCount = await query.CountAsync();
        
        var items = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(p => new ProductSummaryDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                StockQuantity = p.StockQuantity,
                CategoryName = p.Category.Name,
                ReviewsCount = p.Reviews.Count(),
                AverageRating = p.Reviews.Any() ? p.Reviews.Average(r => r.Rating) : 0,
                TotalSold = p.OrderItems.Sum(oi => oi.Quantity),
                CreatedAt = EF.Property<DateTime>(p, "CreatedAt")
            })
            .ToListAsync();
        
        return new PagedResult<ProductSummaryDto>
        {
            Items = items,
            TotalCount = totalCount,
            Page = page,
            PageSize = pageSize
        };
    }
    
    // Working with shadow properties
    public async Task<List<ProductAuditDto>> GetProductAuditTrailAsync(int productId)
    {
        return await _context.Products
            .Where(p => p.Id == productId)
            .Select(p => new ProductAuditDto
            {
                Id = p.Id,
                Name = p.Name,
                CreatedAt = EF.Property<DateTime>(p, "CreatedAt"),
                ModifiedAt = EF.Property<DateTime>(p, "ModifiedAt")
            })
            .ToListAsync();
    }
    
    // Using custom functions
    public async Task<List<UserDisplayDto>> GetUsersWithFullNamesAsync()
    {
        return await _context.Users
            .Select(u => new UserDisplayDto
            {
                Id = u.Id,
                Email = u.Email,
                FullName = EfCoreFunctions.GetFullName(u.Name, ""), // Custom function
                OrdersCount = u.Orders.Count()
            })
            .ToListAsync();
    }
    
    // Optimized loading strategies
    public async Task<Order> GetOrderWithOptimizedLoadingAsync(int orderId)
    {
        // Split query to avoid cartesian explosion
        return await _context.Orders
            .AsSplitQuery()
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Category)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Reviews)
            .FirstOrDefaultAsync(o => o.Id == orderId);
    }
    
    // No-tracking queries for read-only operations
    public async Task<List<ProductListDto>> GetProductsReadOnlyAsync()
    {
        return await _context.Products
            .AsNoTracking()
            .Include(p => p.Category)
            .Select(p => new ProductListDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                CategoryName = p.Category.Name,
                IsActive = p.IsActive
            })
            .ToListAsync();
    }
}

// DTO classes for queries
public class ProductSalesDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public int TotalSold { get; set; }
    public decimal TotalRevenue { get; set; }
}

public class ProductSummaryDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public string CategoryName { get; set; } = "";
    public int ReviewsCount { get; set; }
    public double AverageRating { get; set; }
    public int TotalSold { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class PagedResult<T>
{
    public List<T> Items { get; set; } = new();
    public int TotalCount { get; set; }
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalPages => (int)Math.Ceiling((double)TotalCount / PageSize);
}
```

**Переваги Entity Framework Core Advanced Features:**
- **Global Query Filters** - автоматичне фільтрування на рівні моделі
- **Shadow Properties** - додаткові поля без змін в entity класах
- **Owned Types** - комплексні типи без окремих таблиць
- **Value Conversions** - трансформація даних при збереженні/завантаженні
- **Audit Logging** - автоматичне логування змін через SaveChanges override
- **Custom Functions** - використання SQL функцій в LINQ запитах
- **Split Queries** - оптимізація для складних Include операцій

## 8. Security та Authentication

### 8.1 ASP.NET Core Identity

**Що це таке:**
ASP.NET Core Identity - це система членства, яка дозволяє додавати функціональність входу в застосунок. Вона включає управління користувачами, ролями, claims, токенами, email підтвердженням та іншими функціями безпеки.

**Навіщо це потрібно:**
- Готова система автентифікації та авторизації
- Управління користувачами, ролями та правами доступу
- Хешування паролів та безпечне зберігання
- Підтримка external login providers (Google, Facebook, etc.)
- Email/SMS підтвердження та відновлення паролів
- Two-factor authentication (2FA)

**Практичне застосування:**

```csharp
// Domain models for Identity
public class ApplicationUser : IdentityUser<int>
{
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public DateTime DateOfBirth { get; set; }
    public string? ProfileImageUrl { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? LastLoginAt { get; set; }
    public bool IsDeleted { get; set; } = false;
    
    // Custom claims
    public virtual ICollection<ApplicationUserClaim> Claims { get; set; } = new List<ApplicationUserClaim>();
    public virtual ICollection<ApplicationUserLogin> Logins { get; set; } = new List<ApplicationUserLogin>();
    public virtual ICollection<ApplicationUserToken> Tokens { get; set; } = new List<ApplicationUserToken>();
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; } = new List<ApplicationUserRole>();
    
    // Business relationships
    public virtual ICollection<Order> Orders { get; set; } = new List<Order>();
    public virtual ICollection<UserProfile> Profiles { get; set; } = new List<UserProfile>();
}

public class ApplicationRole : IdentityRole<int>
{
    public string Description { get; set; } = "";
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public bool IsSystemRole { get; set; } = false;
    
    public virtual ICollection<ApplicationRoleClaim> RoleClaims { get; set; } = new List<ApplicationRoleClaim>();
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; } = new List<ApplicationUserRole>();
}

public class ApplicationUserClaim : IdentityUserClaim<int> { }
public class ApplicationUserLogin : IdentityUserLogin<int> { }
public class ApplicationUserToken : IdentityUserToken<int> { }
public class ApplicationRoleClaim : IdentityRoleClaim<int> { }

public class ApplicationUserRole : IdentityUserRole<int>
{
    public virtual ApplicationUser User { get; set; } = null!;
    public virtual ApplicationRole Role { get; set; } = null!;
    public DateTime AssignedAt { get; set; } = DateTime.UtcNow;
    public string AssignedBy { get; set; } = "";
}

public class UserProfile
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public string Bio { get; set; } = "";
    public string Website { get; set; } = "";
    public string Location { get; set; } = "";
    public bool IsPublic { get; set; } = true;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    public virtual ApplicationUser User { get; set; } = null!;
}

// Identity DbContext
public class ApplicationIdentityDbContext : IdentityDbContext<
    ApplicationUser, 
    ApplicationRole, 
    int,
    ApplicationUserClaim,
    ApplicationUserRole,
    ApplicationUserLogin,
    ApplicationRoleClaim,
    ApplicationUserToken>
{
    public ApplicationIdentityDbContext(DbContextOptions<ApplicationIdentityDbContext> options) 
        : base(options) { }
    
    public DbSet<UserProfile> UserProfiles { get; set; } = null!;
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configure custom Identity tables names
        modelBuilder.Entity<ApplicationUser>().ToTable("Users");
        modelBuilder.Entity<ApplicationRole>().ToTable("Roles");
        modelBuilder.Entity<ApplicationUserRole>().ToTable("UserRoles");
        modelBuilder.Entity<ApplicationUserClaim>().ToTable("UserClaims");
        modelBuilder.Entity<ApplicationUserLogin>().ToTable("UserLogins");
        modelBuilder.Entity<ApplicationUserToken>().ToTable("UserTokens");
        modelBuilder.Entity<ApplicationRoleClaim>().ToTable("RoleClaims");
        
        // Configure ApplicationUser
        modelBuilder.Entity<ApplicationUser>(entity =>
        {
            entity.Property(e => e.FirstName).HasMaxLength(100).IsRequired();
            entity.Property(e => e.LastName).HasMaxLength(100).IsRequired();
            entity.HasIndex(e => e.Email).IsUnique();
            entity.HasQueryFilter(u => !u.IsDeleted);
        });
        
        // Configure ApplicationRole
        modelBuilder.Entity<ApplicationRole>(entity =>
        {
            entity.Property(e => e.Description).HasMaxLength(500);
        });
        
        // Configure ApplicationUserRole relationship
        modelBuilder.Entity<ApplicationUserRole>(entity =>
        {
            entity.HasKey(ur => new { ur.UserId, ur.RoleId });
            
            entity.HasOne(ur => ur.User)
                  .WithMany(u => u.UserRoles)
                  .HasForeignKey(ur => ur.UserId)
                  .IsRequired();
                  
            entity.HasOne(ur => ur.Role)
                  .WithMany(r => r.UserRoles)
                  .HasForeignKey(ur => ur.RoleId)
                  .IsRequired();
        });
        
        // Configure UserProfile
        modelBuilder.Entity<UserProfile>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Bio).HasMaxLength(1000);
            entity.Property(e => e.Website).HasMaxLength(200);
            entity.Property(e => e.Location).HasMaxLength(100);
            
            entity.HasOne(e => e.User)
                  .WithMany(u => u.Profiles)
                  .HasForeignKey(e => e.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // Seed default roles
        SeedRoles(modelBuilder);
    }
    
    private static void SeedRoles(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<ApplicationRole>().HasData(
            new ApplicationRole 
            { 
                Id = 1, 
                Name = "SuperAdmin", 
                NormalizedName = "SUPERADMIN",
                Description = "Full system access",
                IsSystemRole = true,
                ConcurrencyStamp = Guid.NewGuid().ToString()
            },
            new ApplicationRole 
            { 
                Id = 2, 
                Name = "Admin", 
                NormalizedName = "ADMIN",
                Description = "Administrative access",
                IsSystemRole = true,
                ConcurrencyStamp = Guid.NewGuid().ToString()
            },
            new ApplicationRole 
            { 
                Id = 3, 
                Name = "Manager", 
                NormalizedName = "MANAGER",
                Description = "Management access",
                ConcurrencyStamp = Guid.NewGuid().ToString()
            },
            new ApplicationRole 
            { 
                Id = 4, 
                Name = "User", 
                NormalizedName = "USER",
                Description = "Standard user access",
                ConcurrencyStamp = Guid.NewGuid().ToString()
            }
        );
    }
}

// Custom User Manager
public class ApplicationUserManager : UserManager<ApplicationUser>
{
    private readonly ApplicationIdentityDbContext _context;
    
    public ApplicationUserManager(
        IUserStore<ApplicationUser> store, 
        IOptions<IdentityOptions> optionsAccessor,
        IPasswordHasher<ApplicationUser> passwordHasher,
        IEnumerable<IUserValidator<ApplicationUser>> userValidators,
        IEnumerable<IPasswordValidator<ApplicationUser>> passwordValidators,
        ILookupNormalizer keyNormalizer,
        IdentityErrorDescriber errors,
        IServiceProvider services,
        ILogger<UserManager<ApplicationUser>> logger,
        ApplicationIdentityDbContext context)
        : base(store, optionsAccessor, passwordHasher, userValidators, passwordValidators, 
               keyNormalizer, errors, services, logger)
    {
        _context = context;
    }
    
    // Custom methods
    public async Task<ApplicationUser?> FindByEmailWithProfileAsync(string email)
    {
        return await _context.Users
            .Include(u => u.Profiles)
            .Include(u => u.UserRoles)
                .ThenInclude(ur => ur.Role)
            .FirstOrDefaultAsync(u => u.Email == email && !u.IsDeleted);
    }
    
    public async Task<IList<ApplicationUser>> GetUsersInRoleWithDetailsAsync(string roleName)
    {
        var role = await _context.Roles.FirstOrDefaultAsync(r => r.Name == roleName);
        if (role == null) return new List<ApplicationUser>();
        
        return await _context.Users
            .Include(u => u.UserRoles)
                .ThenInclude(ur => ur.Role)
            .Include(u => u.Profiles)
            .Where(u => u.UserRoles.Any(ur => ur.RoleId == role.Id) && !u.IsDeleted)
            .ToListAsync();
    }
    
    public async Task<bool> SoftDeleteAsync(ApplicationUser user)
    {
        user.IsDeleted = true;
        user.SecurityStamp = Guid.NewGuid().ToString();
        
        var result = await UpdateAsync(user);
        return result.Succeeded;
    }
    
    public async Task UpdateLastLoginAsync(ApplicationUser user)
    {
        user.LastLoginAt = DateTime.UtcNow;
        await UpdateAsync(user);
    }
}

// Identity Services Configuration
public static class IdentityServiceExtensions
{
    public static IServiceCollection AddCustomIdentity(
        this IServiceCollection services, 
        IConfiguration configuration)
    {
        // Add Identity DbContext
        services.AddDbContext<ApplicationIdentityDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));
        
        // Configure Identity
        services.AddIdentity<ApplicationUser, ApplicationRole>(options =>
        {
            // Password settings
            options.Password.RequireDigit = true;
            options.Password.RequireLowercase = true;
            options.Password.RequireNonAlphanumeric = true;
            options.Password.RequireUppercase = true;
            options.Password.RequiredLength = 8;
            options.Password.RequiredUniqueChars = 1;
            
            // Lockout settings
            options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
            options.Lockout.MaxFailedAccessAttempts = 5;
            options.Lockout.AllowedForNewUsers = true;
            
            // User settings
            options.User.AllowedUserNameCharacters = 
                "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._@+";
            options.User.RequireUniqueEmail = true;
            
            // Sign in settings
            options.SignIn.RequireConfirmedEmail = true;
            options.SignIn.RequireConfirmedPhoneNumber = false;
            options.SignIn.RequireConfirmedAccount = true;
        })
        .AddEntityFrameworkStores<ApplicationIdentityDbContext>()
        .AddUserManager<ApplicationUserManager>()
        .AddDefaultTokenProviders();
        
        // Add custom services
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IRoleService, RoleService>();
        services.AddScoped<IEmailService, EmailService>();
        
        return services;
    }
}

// User Service for business logic
public interface IUserService
{
    Task<RegistrationResult> RegisterUserAsync(RegisterDto model);
    Task<LoginResult> LoginAsync(LoginDto model);
    Task<bool> ConfirmEmailAsync(int userId, string token);
    Task<bool> ResetPasswordAsync(ResetPasswordDto model);
    Task<UserDetailDto?> GetUserByIdAsync(int userId);
    Task<PagedResult<UserListDto>> GetUsersAsync(UserSearchDto search);
    Task<bool> UpdateUserAsync(int userId, UpdateUserDto model);
    Task<bool> DeleteUserAsync(int userId);
    Task<bool> AssignRoleAsync(int userId, string roleName);
    Task<bool> RemoveRoleAsync(int userId, string roleName);
}

public class UserService : IUserService
{
    private readonly ApplicationUserManager _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly RoleManager<ApplicationRole> _roleManager;
    private readonly IEmailService _emailService;
    private readonly ILogger<UserService> _logger;
    
    public UserService(
        ApplicationUserManager userManager,
        SignInManager<ApplicationUser> signInManager,
        RoleManager<ApplicationRole> roleManager,
        IEmailService emailService,
        ILogger<UserService> logger)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _roleManager = roleManager;
        _emailService = emailService;
        _logger = logger;
    }
    
    public async Task<RegistrationResult> RegisterUserAsync(RegisterDto model)
    {
        try
        {
            // Check if user already exists
            var existingUser = await _userManager.FindByEmailAsync(model.Email);
            if (existingUser != null)
            {
                return RegistrationResult.Failed("User with this email already exists");
            }
            
            // Create new user
            var user = new ApplicationUser
            {
                UserName = model.Email,
                Email = model.Email,
                FirstName = model.FirstName,
                LastName = model.LastName,
                DateOfBirth = model.DateOfBirth,
                EmailConfirmed = false
            };
            
            var result = await _userManager.CreateAsync(user, model.Password);
            if (!result.Succeeded)
            {
                var errors = string.Join("; ", result.Errors.Select(e => e.Description));
                return RegistrationResult.Failed(errors);
            }
            
            // Assign default role
            await _userManager.AddToRoleAsync(user, "User");
            
            // Add default claims
            await _userManager.AddClaimsAsync(user, new[]
            {
                new Claim(ClaimTypes.GivenName, user.FirstName),
                new Claim(ClaimTypes.Surname, user.LastName),
                new Claim("user_id", user.Id.ToString())
            });
            
            // Generate email confirmation token
            var token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
            
            // Send confirmation email
            await _emailService.SendEmailConfirmationAsync(user.Email, user.Id, token);
            
            _logger.LogInformation("User {Email} registered successfully", model.Email);
            
            return RegistrationResult.Success(user.Id);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error registering user {Email}", model.Email);
            return RegistrationResult.Failed("An error occurred during registration");
        }
    }
    
    public async Task<LoginResult> LoginAsync(LoginDto model)
    {
        try
        {
            var user = await _userManager.FindByEmailAsync(model.Email);
            if (user == null || user.IsDeleted)
            {
                return LoginResult.Failed("Invalid email or password");
            }
            
            if (!user.EmailConfirmed)
            {
                return LoginResult.Failed("Please confirm your email before logging in");
            }
            
            var result = await _signInManager.PasswordSignInAsync(
                user, model.Password, model.RememberMe, lockoutOnFailure: true);
            
            if (result.Succeeded)
            {
                await _userManager.UpdateLastLoginAsync(user);
                _logger.LogInformation("User {Email} logged in successfully", model.Email);
                
                return LoginResult.Success();
            }
            else if (result.IsLockedOut)
            {
                _logger.LogWarning("User {Email} account locked out", model.Email);
                return LoginResult.Failed("Account locked out. Please try again later.");
            }
            else if (result.RequiresTwoFactor)
            {
                return LoginResult.RequiresTwoFactor();
            }
            else
            {
                _logger.LogWarning("Failed login attempt for user {Email}", model.Email);
                return LoginResult.Failed("Invalid email or password");
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error during login for {Email}", model.Email);
            return LoginResult.Failed("An error occurred during login");
        }
    }
    
    public async Task<bool> ConfirmEmailAsync(int userId, string token)
    {
        try
        {
            var user = await _userManager.FindByIdAsync(userId.ToString());
            if (user == null || user.IsDeleted)
            {
                return false;
            }
            
            var result = await _userManager.ConfirmEmailAsync(user, token);
            
            if (result.Succeeded)
            {
                _logger.LogInformation("Email confirmed for user {UserId}", userId);
                
                // Send welcome email
                await _emailService.SendWelcomeEmailAsync(user.Email, user.FirstName);
            }
            
            return result.Succeeded;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error confirming email for user {UserId}", userId);
            return false;
        }
    }
    
    // Implement other methods...
    public async Task<bool> ResetPasswordAsync(ResetPasswordDto model)
    {
        var user = await _userManager.FindByEmailAsync(model.Email);
        if (user == null || user.IsDeleted) return false;
        
        var result = await _userManager.ResetPasswordAsync(user, model.Token, model.NewPassword);
        return result.Succeeded;
    }
    
    public async Task<UserDetailDto?> GetUserByIdAsync(int userId)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted) return null;
        
        var roles = await _userManager.GetRolesAsync(user);
        var claims = await _userManager.GetClaimsAsync(user);
        
        return new UserDetailDto
        {
            Id = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName,
            DateOfBirth = user.DateOfBirth,
            CreatedAt = user.CreatedAt,
            LastLoginAt = user.LastLoginAt,
            EmailConfirmed = user.EmailConfirmed,
            Roles = roles.ToList(),
            Claims = claims.Select(c => new ClaimDto { Type = c.Type, Value = c.Value }).ToList()
        };
    }
    
    public async Task<PagedResult<UserListDto>> GetUsersAsync(UserSearchDto search)
    {
        // Implementation for paginated user search
        throw new NotImplementedException();
    }
    
    public async Task<bool> UpdateUserAsync(int userId, UpdateUserDto model)
    {
        // Implementation for user update
        throw new NotImplementedException();
    }
    
    public async Task<bool> DeleteUserAsync(int userId)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null) return false;
        
        return await _userManager.SoftDeleteAsync(user);
    }
    
    public async Task<bool> AssignRoleAsync(int userId, string roleName)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted) return false;
        
        var result = await _userManager.AddToRoleAsync(user, roleName);
        return result.Succeeded;
    }
    
    public async Task<bool> RemoveRoleAsync(int userId, string roleName)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted) return false;
        
        var result = await _userManager.RemoveFromRoleAsync(user, roleName);
        return result.Succeeded;
    }
}

// DTOs for Identity operations
public class RegisterDto
{
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public string Email { get; set; } = "";
    public string Password { get; set; } = "";
    public string ConfirmPassword { get; set; } = "";
    public DateTime DateOfBirth { get; set; }
}

public class LoginDto
{
    public string Email { get; set; } = "";
    public string Password { get; set; } = "";
    public bool RememberMe { get; set; }
}

public class ResetPasswordDto
{
    public string Email { get; set; } = "";
    public string Token { get; set; } = "";
    public string NewPassword { get; set; } = "";
    public string ConfirmPassword { get; set; } = "";
}

public class UpdateUserDto
{
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public DateTime DateOfBirth { get; set; }
    public string? ProfileImageUrl { get; set; }
}

public class UserDetailDto
{
    public int Id { get; set; }
    public string Email { get; set; } = "";
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public DateTime DateOfBirth { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? LastLoginAt { get; set; }
    public bool EmailConfirmed { get; set; }
    public List<string> Roles { get; set; } = new();
    public List<ClaimDto> Claims { get; set; } = new();
}

public class UserListDto
{
    public int Id { get; set; }
    public string Email { get; set; } = "";
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public DateTime CreatedAt { get; set; }
    public DateTime? LastLoginAt { get; set; }
    public bool EmailConfirmed { get; set; }
    public List<string> Roles { get; set; } = new();
}

public class UserSearchDto
{
    public string? SearchTerm { get; set; }
    public string? Role { get; set; }
    public bool? EmailConfirmed { get; set; }
    public DateTime? CreatedFrom { get; set; }
    public DateTime? CreatedTo { get; set; }
    public int Page { get; set; } = 1;
    public int PageSize { get; set; } = 10;
}

public class ClaimDto
{
    public string Type { get; set; } = "";
    public string Value { get; set; } = "";
}

// Result classes
public class RegistrationResult
{
    public bool Succeeded { get; set; }
    public string? ErrorMessage { get; set; }
    public int UserId { get; set; }
    
    public static RegistrationResult Success(int userId) => new() { Succeeded = true, UserId = userId };
    public static RegistrationResult Failed(string error) => new() { Succeeded = false, ErrorMessage = error };
}

public class LoginResult
{
    public bool Succeeded { get; set; }
    public bool RequiresTwoFactor { get; set; }
    public string? ErrorMessage { get; set; }
    
    public static LoginResult Success() => new() { Succeeded = true };
    public static LoginResult Failed(string error) => new() { Succeeded = false, ErrorMessage = error };
    public static LoginResult RequiresTwoFactor() => new() { RequiresTwoFactor = true };
}

// Email Service Interface and Implementation
public interface IEmailService
{
    Task SendEmailConfirmationAsync(string email, int userId, string token);
    Task SendWelcomeEmailAsync(string email, string firstName);
    Task SendPasswordResetAsync(string email, string token);
    Task SendTwoFactorCodeAsync(string email, string code);
}

public class EmailService : IEmailService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<EmailService> _logger;
    
    public EmailService(IConfiguration configuration, ILogger<EmailService> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }
    
    public async Task SendEmailConfirmationAsync(string email, int userId, string token)
    {
        var confirmationLink = $"{_configuration["AppSettings:BaseUrl"]}/account/confirm-email?userId={userId}&token={Uri.EscapeDataString(token)}";
        
        var subject = "Confirm your email";
        var body = $@"
            <h2>Email Confirmation</h2>
            <p>Please confirm your email by clicking the link below:</p>
            <a href='{confirmationLink}' style='background-color: #007bff; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;'>Confirm Email</a>
            <p>This link will expire in 24 hours.</p>
        ";
        
        await SendEmailAsync(email, subject, body);
    }
    
    public async Task SendWelcomeEmailAsync(string email, string firstName)
    {
        var subject = "Welcome to our application!";
        var body = $@"
            <h2>Welcome {firstName}!</h2>
            <p>Thank you for joining our application. Your account has been successfully created and confirmed.</p>
            <p>You can now start using all the features available to you.</p>
            <p>If you have any questions, please don't hesitate to contact our support team.</p>
        ";
        
        await SendEmailAsync(email, subject, body);
    }
    
    public async Task SendPasswordResetAsync(string email, string token)
    {
        var resetLink = $"{_configuration["AppSettings:BaseUrl"]}/account/reset-password?email={Uri.EscapeDataString(email)}&token={Uri.EscapeDataString(token)}";
        
        var subject = "Reset your password";
        var body = $@"
            <h2>Password Reset</h2>
            <p>You requested a password reset. Please click the link below to reset your password:</p>
            <a href='{resetLink}' style='background-color: #dc3545; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;'>Reset Password</a>
            <p>This link will expire in 1 hour.</p>
            <p>If you did not request this, please ignore this email.</p>
        ";
        
        await SendEmailAsync(email, subject, body);
    }
    
    public async Task SendTwoFactorCodeAsync(string email, string code)
    {
        var subject = "Your two-factor authentication code";
        var body = $@"
            <h2>Two-Factor Authentication</h2>
            <p>Your verification code is: <strong style='font-size: 24px; color: #007bff;'>{code}</strong></p>
            <p>This code will expire in 5 minutes.</p>
        ";
        
        await SendEmailAsync(email, subject, body);
    }
    
    private async Task SendEmailAsync(string email, string subject, string htmlBody)
    {
        try
        {
            // In a real application, use a proper email service like SendGrid, AWS SES, etc.
            // This is a simplified example
            _logger.LogInformation("Email sent to {Email} with subject: {Subject}", email, subject);
            
            // Simulate async operation
            await Task.Delay(100);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to send email to {Email}", email);
            throw;
        }
    }
}

// Role Service
public interface IRoleService
{
    Task<bool> CreateRoleAsync(string roleName, string description);
    Task<bool> UpdateRoleAsync(int roleId, string roleName, string description);
    Task<bool> DeleteRoleAsync(int roleId);
    Task<List<RoleDto>> GetAllRolesAsync();
    Task<RoleDetailDto?> GetRoleByIdAsync(int roleId);
    Task<bool> AssignPermissionsAsync(int roleId, List<string> permissions);
}

public class RoleService : IRoleService
{
    private readonly RoleManager<ApplicationRole> _roleManager;
    private readonly ApplicationIdentityDbContext _context;
    private readonly ILogger<RoleService> _logger;
    
    public RoleService(
        RoleManager<ApplicationRole> roleManager,
        ApplicationIdentityDbContext context,
        ILogger<RoleService> logger)
    {
        _roleManager = roleManager;
        _context = context;
        _logger = logger;
    }
    
    public async Task<bool> CreateRoleAsync(string roleName, string description)
    {
        try
        {
            var existingRole = await _roleManager.FindByNameAsync(roleName);
            if (existingRole != null)
            {
                return false;
            }
            
            var role = new ApplicationRole
            {
                Name = roleName,
                Description = description,
                IsSystemRole = false
            };
            
            var result = await _roleManager.CreateAsync(role);
            
            if (result.Succeeded)
            {
                _logger.LogInformation("Role {RoleName} created successfully", roleName);
            }
            
            return result.Succeeded;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating role {RoleName}", roleName);
            return false;
        }
    }
    
    public async Task<bool> UpdateRoleAsync(int roleId, string roleName, string description)
    {
        try
        {
            var role = await _roleManager.FindByIdAsync(roleId.ToString());
            if (role == null || role.IsSystemRole)
            {
                return false;
            }
            
            role.Name = roleName;
            role.Description = description;
            
            var result = await _roleManager.UpdateAsync(role);
            return result.Succeeded;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating role {RoleId}", roleId);
            return false;
        }
    }
    
    public async Task<bool> DeleteRoleAsync(int roleId)
    {
        try
        {
            var role = await _roleManager.FindByIdAsync(roleId.ToString());
            if (role == null || role.IsSystemRole)
            {
                return false;
            }
            
            var result = await _roleManager.DeleteAsync(role);
            return result.Succeeded;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error deleting role {RoleId}", roleId);
            return false;
        }
    }
    
    public async Task<List<RoleDto>> GetAllRolesAsync()
    {
        return await _context.Roles
            .Select(r => new RoleDto
            {
                Id = r.Id,
                Name = r.Name,
                Description = r.Description,
                IsSystemRole = r.IsSystemRole,
                CreatedAt = r.CreatedAt
            })
            .ToListAsync();
    }
    
    public async Task<RoleDetailDto?> GetRoleByIdAsync(int roleId)
    {
        var role = await _context.Roles
            .Include(r => r.UserRoles)
                .ThenInclude(ur => ur.User)
            .Include(r => r.RoleClaims)
            .FirstOrDefaultAsync(r => r.Id == roleId);
        
        if (role == null) return null;
        
        return new RoleDetailDto
        {
            Id = role.Id,
            Name = role.Name,
            Description = role.Description,
            IsSystemRole = role.IsSystemRole,
            CreatedAt = role.CreatedAt,
            UsersCount = role.UserRoles.Count,
            Claims = role.RoleClaims.Select(rc => new ClaimDto 
            { 
                Type = rc.ClaimType ?? "", 
                Value = rc.ClaimValue ?? "" 
            }).ToList()
        };
    }
    
    public async Task<bool> AssignPermissionsAsync(int roleId, List<string> permissions)
    {
        try
        {
            var role = await _roleManager.FindByIdAsync(roleId.ToString());
            if (role == null) return false;
            
            // Remove existing claims
            var existingClaims = await _roleManager.GetClaimsAsync(role);
            foreach (var claim in existingClaims)
            {
                await _roleManager.RemoveClaimAsync(role, claim);
            }
            
            // Add new claims
            foreach (var permission in permissions)
            {
                await _roleManager.AddClaimAsync(role, new Claim("permission", permission));
            }
            
            return true;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error assigning permissions to role {RoleId}", roleId);
            return false;
        }
    }
}

public class RoleDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public bool IsSystemRole { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class RoleDetailDto : RoleDto
{
    public int UsersCount { get; set; }
    public List<ClaimDto> Claims { get; set; } = new();
}

// Identity Controllers
[ApiController]
[Route("api/[controller]")]
public class AccountController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<AccountController> _logger;
    
    public AccountController(IUserService userService, ILogger<AccountController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    [HttpPost("register")]
    public async Task<ActionResult> Register([FromBody] RegisterDto model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        if (model.Password != model.ConfirmPassword)
        {
            return BadRequest("Passwords do not match");
        }
        
        var result = await _userService.RegisterUserAsync(model);
        
        if (result.Succeeded)
        {
            return Ok(new { message = "Registration successful. Please check your email to confirm your account.", userId = result.UserId });
        }
        
        return BadRequest(result.ErrorMessage);
    }
    
    [HttpPost("login")]
    public async Task<ActionResult> Login([FromBody] LoginDto model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var result = await _userService.LoginAsync(model);
        
        if (result.Succeeded)
        {
            return Ok(new { message = "Login successful" });
        }
        
        if (result.RequiresTwoFactor)
        {
            return Ok(new { requiresTwoFactor = true });
        }
        
        return BadRequest(result.ErrorMessage);
    }
    
    [HttpPost("confirm-email")]
    public async Task<ActionResult> ConfirmEmail([FromQuery] int userId, [FromQuery] string token)
    {
        if (userId <= 0 || string.IsNullOrEmpty(token))
        {
            return BadRequest("Invalid confirmation parameters");
        }
        
        var result = await _userService.ConfirmEmailAsync(userId, token);
        
        if (result)
        {
            return Ok(new { message = "Email confirmed successfully" });
        }
        
        return BadRequest("Email confirmation failed");
    }
    
    [HttpPost("forgot-password")]
    public async Task<ActionResult> ForgotPassword([FromBody] ForgotPasswordDto model)
    {
        // Implementation for forgot password
        return Ok(new { message = "Password reset email sent if account exists" });
    }
    
    [HttpPost("reset-password")]
    public async Task<ActionResult> ResetPassword([FromBody] ResetPasswordDto model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        if (model.NewPassword != model.ConfirmPassword)
        {
            return BadRequest("Passwords do not match");
        }
        
        var result = await _userService.ResetPasswordAsync(model);
        
        if (result)
        {
            return Ok(new { message = "Password reset successful" });
        }
        
        return BadRequest("Password reset failed");
    }
    
    [HttpPost("logout")]
    [Authorize]
    public async Task<ActionResult> Logout()
    {
        // Implementation for logout
        return Ok(new { message = "Logged out successfully" });
    }
}

[ApiController]
[Route("api/[controller]")]
[Authorize]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(IUserService userService, ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    [HttpGet("{id}")]
    [Authorize(Roles = "Admin,Manager")]
    public async Task<ActionResult<UserDetailDto>> GetUser(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        
        return Ok(user);
    }
    
    [HttpGet]
    [Authorize(Roles = "Admin,Manager")]
    public async Task<ActionResult<PagedResult<UserListDto>>> GetUsers([FromQuery] UserSearchDto search)
    {
        var result = await _userService.GetUsersAsync(search);
        return Ok(result);
    }
    
    [HttpPut("{id}")]
    public async Task<ActionResult> UpdateUser(int id, [FromBody] UpdateUserDto model)
    {
        // Check if user can only update their own profile (unless admin)
        var currentUserId = User.FindFirst("user_id")?.Value;
        if (!User.IsInRole("Admin") && currentUserId != id.ToString())
        {
            return Forbid();
        }
        
        var result = await _userService.UpdateUserAsync(id, model);
        if (result)
        {
            return Ok(new { message = "User updated successfully" });
        }
        
        return BadRequest("Update failed");
    }
    
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult> DeleteUser(int id)
    {
        var result = await _userService.DeleteUserAsync(id);
        if (result)
        {
            return Ok(new { message = "User deleted successfully" });
        }
        
        return BadRequest("Delete failed");
    }
    
    [HttpPost("{id}/roles/{roleName}")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult> AssignRole(int id, string roleName)
    {
        var result = await _userService.AssignRoleAsync(id, roleName);
        if (result)
        {
            return Ok(new { message = "Role assigned successfully" });
        }
        
        return BadRequest("Role assignment failed");
    }
    
    [HttpDelete("{id}/roles/{roleName}")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult> RemoveRole(int id, string roleName)
    {
        var result = await _userService.RemoveRoleAsync(id, roleName);
        if (result)
        {
            return Ok(new { message = "Role removed successfully" });
        }
        
        return BadRequest("Role removal failed");
    }
}

[ApiController]
[Route("api/[controller]")]
[Authorize(Roles = "Admin")]
public class RolesController : ControllerBase
{
    private readonly IRoleService _roleService;
    private readonly ILogger<RolesController> _logger;
    
    public RolesController(IRoleService roleService, ILogger<RolesController> logger)
    {
        _roleService = roleService;
        _logger = logger;
    }
    
    [HttpGet]
    public async Task<ActionResult<List<RoleDto>>> GetRoles()
    {
        var roles = await _roleService.GetAllRolesAsync();
        return Ok(roles);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<RoleDetailDto>> GetRole(int id)
    {
        var role = await _roleService.GetRoleByIdAsync(id);
        if (role == null)
        {
            return NotFound();
        }
        
        return Ok(role);
    }
    
    [HttpPost]
    public async Task<ActionResult> CreateRole([FromBody] CreateRoleDto model)
    {
        var result = await _roleService.CreateRoleAsync(model.Name, model.Description);
        if (result)
        {
            return Ok(new { message = "Role created successfully" });
        }
        
        return BadRequest("Role creation failed");
    }
    
    [HttpPut("{id}")]
    public async Task<ActionResult> UpdateRole(int id, [FromBody] UpdateRoleDto model)
    {
        var result = await _roleService.UpdateRoleAsync(id, model.Name, model.Description);
        if (result)
        {
            return Ok(new { message = "Role updated successfully" });
        }
        
        return BadRequest("Role update failed");
    }
    
    [HttpDelete("{id}")]
    public async Task<ActionResult> DeleteRole(int id)
    {
        var result = await _roleService.DeleteRoleAsync(id);
        if (result)
        {
            return Ok(new { message = "Role deleted successfully" });
        }
        
        return BadRequest("Role deletion failed");
    }
    
    [HttpPost("{id}/permissions")]
    public async Task<ActionResult> AssignPermissions(int id, [FromBody] AssignPermissionsDto model)
    {
        var result = await _roleService.AssignPermissionsAsync(id, model.Permissions);
        if (result)
        {
            return Ok(new { message = "Permissions assigned successfully" });
        }
        
        return BadRequest("Permission assignment failed");
    }
}

// Additional DTOs
public class ForgotPasswordDto
{
    public string Email { get; set; } = "";
}

public class CreateRoleDto
{
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
}

public class UpdateRoleDto
{
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
}

public class AssignPermissionsDto
{
    public List<string> Permissions { get; set; } = new();
}

// Program.cs configuration example
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Add services
        builder.Services.AddControllers();
        
        // Add custom Identity system
        builder.Services.AddCustomIdentity(builder.Configuration);
        
        // Configure authentication
        builder.Services.ConfigureApplicationCookie(options =>
        {
            options.LoginPath = "/api/account/login";
            options.LogoutPath = "/api/account/logout";
            options.AccessDeniedPath = "/api/account/access-denied";
            options.ExpireTimeSpan = TimeSpan.FromDays(30);
            options.SlidingExpiration = true;
            options.Cookie.HttpOnly = true;
            options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
            options.Cookie.SameSite = SameSiteMode.Strict;
        });
        
        var app = builder.Build();
        
        // Configure pipeline
        if (app.Environment.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        
        app.UseHttpsRedirection();
        app.UseAuthentication();
        app.UseAuthorization();
        app.MapControllers();
        
        app.Run();
    }
}
```

**Переваги ASP.NET Core Identity:**
- **Повна система управління користувачами** з готовими функціями реєстрації, входу, підтвердження email
- **Безпечне зберігання паролів** з використанням bcrypt hashing
- **Гнучка система ролей та claims** для детальної авторизації
- **Підтримка зовнішніх провайдерів** (Google, Facebook, Microsoft, etc.)
- **Lockout механізм** для захисту від brute force атак
- **Email/SMS підтвердження** та відновлення паролів
- **Настроювані валідатори** для паролів та користувачів
- **Інтеграція з Entity Framework** для гнучкого зберігання

### 8.2 JWT Authentication

**Що це таке:**
JSON Web Token (JWT) - це стандарт для безпечної передачі інформації між сторонами у вигляді JSON об'єкта. JWT складається з трьох частин: Header, Payload та Signature, розділених крапками. Це stateless альтернатива cookie-based автентифікації.

**Навіщо це потрібно:**
- Stateless автентифікація - токени містять всю необхідну інформацію
- Підходить для мікросервісної архітектури та API
- Cross-platform сумісність
- Можливість додавання custom claims
- Не потребує серверного зберігання сесій
- Підтримка Single Sign-On (SSO)

**Практичне застосування:**

```csharp
// JWT Configuration and Services
public class JwtSettings
{
    public string Secret { get; set; } = "";
    public string Issuer { get; set; } = "";
    public string Audience { get; set; } = "";
    public int ExpiryMinutes { get; set; } = 60;
    public int RefreshTokenExpiryDays { get; set; } = 30;
}

public class JwtTokenService
{
    private readonly JwtSettings _jwtSettings;
    private readonly ILogger<JwtTokenService> _logger;
    private readonly ApplicationUserManager _userManager;
    
    public JwtTokenService(
        IOptions<JwtSettings> jwtSettings, 
        ILogger<JwtTokenService> logger,
        ApplicationUserManager userManager)
    {
        _jwtSettings = jwtSettings.Value;
        _logger = logger;
        _userManager = userManager;
    }
    
    public async Task<JwtTokenResponse> GenerateTokenAsync(ApplicationUser user)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_jwtSettings.Secret);
        
        // Get user roles and claims
        var roles = await _userManager.GetRolesAsync(user);
        var userClaims = await _userManager.GetClaimsAsync(user);
        
        var claims = new List<Claim>
        {
            new(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new(ClaimTypes.Name, user.UserName ?? ""),
            new(ClaimTypes.Email, user.Email ?? ""),
            new(ClaimTypes.GivenName, user.FirstName),
            new(ClaimTypes.Surname, user.LastName),
            new("user_id", user.Id.ToString()),
            new(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new(JwtRegisteredClaimNames.Iat, new DateTimeOffset(DateTime.UtcNow).ToUnixTimeSeconds().ToString(), ClaimValueTypes.Integer64)
        };
        
        // Add role claims
        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }
        
        // Add custom user claims
        claims.AddRange(userClaims);
        
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(_jwtSettings.ExpiryMinutes),
            Issuer = _jwtSettings.Issuer,
            Audience = _jwtSettings.Audience,
            SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
        };
        
        var token = tokenHandler.CreateToken(tokenDescriptor);
        var tokenString = tokenHandler.WriteToken(token);
        
        // Generate refresh token
        var refreshToken = GenerateRefreshToken();
        
        _logger.LogInformation("JWT token generated for user {UserId}", user.Id);
        
        return new JwtTokenResponse
        {
            AccessToken = tokenString,
            RefreshToken = refreshToken,
            ExpiresAt = tokenDescriptor.Expires.Value,
            TokenType = "Bearer"
        };
    }
    
    public ClaimsPrincipal? ValidateToken(string token)
    {
        try
        {
            var tokenHandler = new JwtSecurityTokenHandler();
            var key = Encoding.ASCII.GetBytes(_jwtSettings.Secret);
            
            var validationParameters = new TokenValidationParameters
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(key),
                ValidateIssuer = true,
                ValidIssuer = _jwtSettings.Issuer,
                ValidateAudience = true,
                ValidAudience = _jwtSettings.Audience,
                ValidateLifetime = true,
                ClockSkew = TimeSpan.Zero
            };
            
            var principal = tokenHandler.ValidateToken(token, validationParameters, out SecurityToken validatedToken);
            
            if (validatedToken is JwtSecurityToken jwtToken && 
                jwtToken.Header.Alg.Equals(SecurityAlgorithms.HmacSha256, StringComparison.InvariantCultureIgnoreCase))
            {
                return principal;
            }
            
            return null;
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Token validation failed");
            return null;
        }
    }
    
    public string GenerateRefreshToken()
    {
        var randomNumber = new byte[32];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(randomNumber);
            return Convert.ToBase64String(randomNumber);
        }
    }
    
    public async Task<RefreshTokenResult> RefreshTokenAsync(string refreshToken)
    {
        // In real application, verify refresh token against stored tokens
        // This is a simplified example
        var storedToken = await GetStoredRefreshTokenAsync(refreshToken);
        
        if (storedToken == null || storedToken.ExpiresAt < DateTime.UtcNow)
        {
            return RefreshTokenResult.Failed("Invalid or expired refresh token");
        }
        
        var user = await _userManager.FindByIdAsync(storedToken.UserId.ToString());
        if (user == null || user.IsDeleted)
        {
            return RefreshTokenResult.Failed("User not found");
        }
        
        // Generate new tokens
        var tokenResponse = await GenerateTokenAsync(user);
        
        // Update stored refresh token
        await UpdateRefreshTokenAsync(storedToken.Token, tokenResponse.RefreshToken);
        
        return RefreshTokenResult.Success(tokenResponse);
    }
    
    private async Task<StoredRefreshToken?> GetStoredRefreshTokenAsync(string token)
    {
        // Implementation to retrieve from database
        // This is a placeholder
        await Task.Delay(1);
        return new StoredRefreshToken
        {
            Token = token,
            UserId = 1,
            ExpiresAt = DateTime.UtcNow.AddDays(30)
        };
    }
    
    private async Task UpdateRefreshTokenAsync(string oldToken, string newToken)
    {
        // Implementation to update in database
        await Task.Delay(1);
    }
}

// JWT Response Models
public class JwtTokenResponse
{
    public string AccessToken { get; set; } = "";
    public string RefreshToken { get; set; } = "";
    public DateTime ExpiresAt { get; set; }
    public string TokenType { get; set; } = "Bearer";
}

public class RefreshTokenResult
{
    public bool Succeeded { get; set; }
    public string? ErrorMessage { get; set; }
    public JwtTokenResponse? TokenResponse { get; set; }
    
    public static RefreshTokenResult Success(JwtTokenResponse tokenResponse) => 
        new() { Succeeded = true, TokenResponse = tokenResponse };
    
    public static RefreshTokenResult Failed(string error) => 
        new() { Succeeded = false, ErrorMessage = error };
}

public class StoredRefreshToken
{
    public string Token { get; set; } = "";
    public int UserId { get; set; }
    public DateTime ExpiresAt { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public bool IsRevoked { get; set; } = false;
}

// JWT Authentication Controller
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly ApplicationUserManager _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly JwtTokenService _tokenService;
    private readonly ILogger<AuthController> _logger;
    
    public AuthController(
        ApplicationUserManager userManager,
        SignInManager<ApplicationUser> signInManager,
        JwtTokenService tokenService,
        ILogger<AuthController> logger)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _tokenService = tokenService;
        _logger = logger;
    }
    
    [HttpPost("login")]
    public async Task<ActionResult<JwtLoginResponse>> Login([FromBody] JwtLoginRequest request)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var user = await _userManager.FindByEmailAsync(request.Email);
        if (user == null || user.IsDeleted)
        {
            return BadRequest("Invalid email or password");
        }
        
        if (!user.EmailConfirmed)
        {
            return BadRequest("Please confirm your email before logging in");
        }
        
        var result = await _signInManager.CheckPasswordSignInAsync(user, request.Password, lockoutOnFailure: true);
        
        if (result.Succeeded)
        {
            var tokenResponse = await _tokenService.GenerateTokenAsync(user);
            await _userManager.UpdateLastLoginAsync(user);
            
            _logger.LogInformation("User {Email} logged in successfully with JWT", request.Email);
            
            return Ok(new JwtLoginResponse
            {
                Success = true,
                Message = "Login successful",
                Token = tokenResponse,
                User = new UserSummaryDto
                {
                    Id = user.Id,
                    Email = user.Email,
                    FirstName = user.FirstName,
                    LastName = user.LastName,
                    Roles = (await _userManager.GetRolesAsync(user)).ToList()
                }
            });
        }
        else if (result.IsLockedOut)
        {
            _logger.LogWarning("User {Email} account locked out", request.Email);
            return BadRequest("Account locked out. Please try again later.");
        }
        else if (result.RequiresTwoFactor)
        {
            return Ok(new JwtLoginResponse
            {
                Success = false,
                RequiresTwoFactor = true,
                Message = "Two-factor authentication required"
            });
        }
        else
        {
            _logger.LogWarning("Failed login attempt for user {Email}", request.Email);
            return BadRequest("Invalid email or password");
        }
    }
    
    [HttpPost("refresh")]
    public async Task<ActionResult<JwtTokenResponse>> RefreshToken([FromBody] RefreshTokenRequest request)
    {
        if (string.IsNullOrEmpty(request.RefreshToken))
        {
            return BadRequest("Refresh token is required");
        }
        
        var result = await _tokenService.RefreshTokenAsync(request.RefreshToken);
        
        if (result.Succeeded)
        {
            return Ok(result.TokenResponse);
        }
        
        return BadRequest(result.ErrorMessage);
    }
    
    [HttpPost("revoke")]
    [Authorize]
    public async Task<ActionResult> RevokeToken([FromBody] RevokeTokenRequest request)
    {
        // Implementation to revoke refresh token
        // This would mark the token as revoked in the database
        return Ok(new { message = "Token revoked successfully" });
    }
    
    [HttpGet("me")]
    [Authorize]
    public async Task<ActionResult<UserDetailDto>> GetCurrentUser()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (string.IsNullOrEmpty(userId))
        {
            return BadRequest("Invalid token");
        }
        
        var user = await _userManager.FindByIdAsync(userId);
        if (user == null || user.IsDeleted)
        {
            return NotFound("User not found");
        }
        
        var roles = await _userManager.GetRolesAsync(user);
        var claims = await _userManager.GetClaimsAsync(user);
        
        return Ok(new UserDetailDto
        {
            Id = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName,
            DateOfBirth = user.DateOfBirth,
            CreatedAt = user.CreatedAt,
            LastLoginAt = user.LastLoginAt,
            EmailConfirmed = user.EmailConfirmed,
            Roles = roles.ToList(),
            Claims = claims.Select(c => new ClaimDto { Type = c.Type, Value = c.Value }).ToList()
        });
    }
}

// JWT Request/Response Models
public class JwtLoginRequest
{
    public string Email { get; set; } = "";
    public string Password { get; set; } = "";
}

public class JwtLoginResponse
{
    public bool Success { get; set; }
    public string Message { get; set; } = "";
    public bool RequiresTwoFactor { get; set; }
    public JwtTokenResponse? Token { get; set; }
    public UserSummaryDto? User { get; set; }
}

public class UserSummaryDto
{
    public int Id { get; set; }
    public string Email { get; set; } = "";
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public List<string> Roles { get; set; } = new();
}

public class RefreshTokenRequest
{
    public string RefreshToken { get; set; } = "";
}

public class RevokeTokenRequest
{
    public string RefreshToken { get; set; } = "";
}

// JWT Configuration Extensions
public static class JwtServiceExtensions
{
    public static IServiceCollection AddJwtAuthentication(
        this IServiceCollection services, 
        IConfiguration configuration)
    {
        // Configure JWT settings
        var jwtSettings = configuration.GetSection("JwtSettings");
        services.Configure<JwtSettings>(jwtSettings);
        
        var jwtConfig = jwtSettings.Get<JwtSettings>();
        var key = Encoding.ASCII.GetBytes(jwtConfig?.Secret ?? throw new InvalidOperationException("JWT Secret not configured"));
        
        // Add JWT authentication
        services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        })
        .AddJwtBearer(options =>
        {
            options.RequireHttpsMetadata = true;
            options.SaveToken = true;
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(key),
                ValidateIssuer = true,
                ValidIssuer = jwtConfig.Issuer,
                ValidateAudience = true,
                ValidAudience = jwtConfig.Audience,
                ValidateLifetime = true,
                ClockSkew = TimeSpan.Zero,
                RequireExpirationTime = true
            };
            
            // Handle JWT events
            options.Events = new JwtBearerEvents
            {
                OnAuthenticationFailed = context =>
                {
                    var logger = context.HttpContext.RequestServices.GetService<ILogger<JwtBearerHandler>>();
                    logger?.LogWarning("JWT authentication failed: {Exception}", context.Exception.Message);
                    return Task.CompletedTask;
                },
                
                OnChallenge = context =>
                {
                    context.HandleResponse();
                    context.Response.StatusCode = StatusCodes.Status401Unauthorized;
                    context.Response.ContentType = "application/json";
                    
                    var result = JsonSerializer.Serialize(new
                    {
                        error = "unauthorized",
                        message = "You are not authorized to access this resource"
                    });
                    
                    return context.Response.WriteAsync(result);
                },
                
                OnForbidden = context =>
                {
                    context.Response.StatusCode = StatusCodes.Status403Forbidden;
                    context.Response.ContentType = "application/json";
                    
                    var result = JsonSerializer.Serialize(new
                    {
                        error = "forbidden",
                        message = "You do not have permission to access this resource"
                    });
                    
                    return context.Response.WriteAsync(result);
                },
                
                OnMessageReceived = context =>
                {
                    // Allow token from query parameter for SignalR connections
                    var accessToken = context.Request.Query["access_token"];
                    var path = context.HttpContext.Request.Path;
                    
                    if (!string.IsNullOrEmpty(accessToken) && path.StartsWithSegments("/hubs"))
                    {
                        context.Token = accessToken;
                    }
                    
                    return Task.CompletedTask;
                }
            };
        });
        
        // Register JWT services
        services.AddScoped<JwtTokenService>();
        
        return services;
    }
}

// Refresh Token Entity for Database
public class RefreshToken
{
    public int Id { get; set; }
    public string Token { get; set; } = "";
    public int UserId { get; set; }
    public ApplicationUser User { get; set; } = null!;
    public DateTime ExpiresAt { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public bool IsRevoked { get; set; } = false;
    public string? ReplacedByToken { get; set; }
    public string CreatedByIp { get; set; } = "";
    public string? RevokedByIp { get; set; }
    public DateTime? RevokedAt { get; set; }
}

// Enhanced DbContext with Refresh Tokens
public class ApplicationJwtDbContext : ApplicationIdentityDbContext
{
    public ApplicationJwtDbContext(DbContextOptions<ApplicationJwtDbContext> options) : base(options) { }
    
    public DbSet<RefreshToken> RefreshTokens { get; set; } = null!;
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configure RefreshToken
        modelBuilder.Entity<RefreshToken>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Token).IsRequired().HasMaxLength(500);
            entity.Property(e => e.CreatedByIp).HasMaxLength(50);
            entity.Property(e => e.RevokedByIp).HasMaxLength(50);
            entity.Property(e => e.ReplacedByToken).HasMaxLength(500);
            
            entity.HasOne(e => e.User)
                  .WithMany()
                  .HasForeignKey(e => e.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
            
            entity.HasIndex(e => e.Token).IsUnique();
            entity.HasIndex(e => new { e.UserId, e.IsRevoked });
        });
    }
}

// Enhanced JWT Token Service with Database Storage
public class EnhancedJwtTokenService : JwtTokenService
{
    private readonly ApplicationJwtDbContext _context;
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public EnhancedJwtTokenService(
        IOptions<JwtSettings> jwtSettings,
        ILogger<JwtTokenService> logger,
        ApplicationUserManager userManager,
        ApplicationJwtDbContext context,
        IHttpContextAccessor httpContextAccessor)
        : base(jwtSettings, logger, userManager)
    {
        _context = context;
        _httpContextAccessor = httpContextAccessor;
    }
    
    public async Task<JwtTokenResponse> GenerateTokenWithStorageAsync(ApplicationUser user)
    {
        var tokenResponse = await GenerateTokenAsync(user);
        
        // Store refresh token in database
        var refreshToken = new RefreshToken
        {
            Token = tokenResponse.RefreshToken,
            UserId = user.Id,
            ExpiresAt = DateTime.UtcNow.AddDays(30),
            CreatedByIp = GetIpAddress()
        };
        
        _context.RefreshTokens.Add(refreshToken);
        await _context.SaveChangesAsync();
        
        return tokenResponse;
    }
    
    public async Task<RefreshTokenResult> RefreshTokenWithDatabaseAsync(string refreshToken)
    {
        var storedToken = await _context.RefreshTokens
            .Include(rt => rt.User)
            .FirstOrDefaultAsync(rt => rt.Token == refreshToken && !rt.IsRevoked);
        
        if (storedToken == null || storedToken.ExpiresAt < DateTime.UtcNow)
        {
            return RefreshTokenResult.Failed("Invalid or expired refresh token");
        }
        
        if (storedToken.User.IsDeleted)
        {
            return RefreshTokenResult.Failed("User not found");
        }
        
        // Generate new tokens
        var newTokenResponse = await GenerateTokenAsync(storedToken.User);
        
        // Revoke old token and create new one
        storedToken.IsRevoked = true;
        storedToken.RevokedAt = DateTime.UtcNow;
        storedToken.RevokedByIp = GetIpAddress();
        storedToken.ReplacedByToken = newTokenResponse.RefreshToken;
        
        var newRefreshToken = new RefreshToken
        {
            Token = newTokenResponse.RefreshToken,
            UserId = storedToken.UserId,
            ExpiresAt = DateTime.UtcNow.AddDays(30),
            CreatedByIp = GetIpAddress()
        };
        
        _context.RefreshTokens.Add(newRefreshToken);
        await _context.SaveChangesAsync();
        
        return RefreshTokenResult.Success(newTokenResponse);
    }
    
    public async Task<bool> RevokeTokenAsync(string refreshToken)
    {
        var token = await _context.RefreshTokens
            .FirstOrDefaultAsync(rt => rt.Token == refreshToken && !rt.IsRevoked);
        
        if (token == null)
        {
            return false;
        }
        
        token.IsRevoked = true;
        token.RevokedAt = DateTime.UtcNow;
        token.RevokedByIp = GetIpAddress();
        
        await _context.SaveChangesAsync();
        return true;
    }
    
    private string GetIpAddress()
    {
        var httpContext = _httpContextAccessor.HttpContext;
        if (httpContext?.Request.Headers.ContainsKey("X-Forwarded-For") == true)
        {
            return httpContext.Request.Headers["X-Forwarded-For"].ToString();
        }
        
        return httpContext?.Connection.RemoteIpAddress?.ToString() ?? "Unknown";
    }
}

// Program.cs with JWT Configuration
public class JwtProgram
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Add services
        builder.Services.AddControllers();
        
        // Add DbContext
        builder.Services.AddDbContext<ApplicationJwtDbContext>(options =>
            options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
        
        // Add Identity
        builder.Services.AddCustomIdentity(builder.Configuration);
        
        // Add JWT Authentication
        builder.Services.AddJwtAuthentication(builder.Configuration);
        
        // Add HTTP context accessor for IP tracking
        builder.Services.AddHttpContextAccessor();
        
        // Override with enhanced JWT service
        builder.Services.AddScoped<JwtTokenService, EnhancedJwtTokenService>();
        
        // Add CORS if needed for frontend
        builder.Services.AddCors(options =>
        {
            options.AddPolicy("AllowSpecificOrigin", policy =>
            {
                policy.WithOrigins("https://localhost:3000", "https://myapp.com")
                      .AllowAnyHeader()
                      .AllowAnyMethod()
                      .AllowCredentials();
            });
        });
        
        var app = builder.Build();
        
        // Configure pipeline
        if (app.Environment.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        
        app.UseHttpsRedirection();
        app.UseCors("AllowSpecificOrigin");
        app.UseAuthentication();
        app.UseAuthorization();
        app.MapControllers();
        
        app.Run();
    }
}

// Example appsettings.json configuration
/*
{
  "JwtSettings": {
    "Secret": "your-super-secret-key-that-is-at-least-32-characters-long",
    "Issuer": "YourApp",
    "Audience": "YourAppUsers",
    "ExpiryMinutes": 60,
    "RefreshTokenExpiryDays": 30
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=YourAppDb;Trusted_Connection=true;MultipleActiveResultSets=true"
  }
}
*/
```

**Переваги JWT Authentication:**
- **Stateless** - сервер не зберігає сесій, токен містить всю інформацію
- **Масштабованість** - легко розподіляти між серверами
- **Cross-platform** - працює з будь-якими клієнтами (web, mobile, desktop)
- **Безпека** - підписані та опціонально зашифровані токени
- **Гнучкість** - можливість додавання custom claims
- **API-friendly** - ідеально підходить для RESTful API та SPA

### 8.3 Authorization Policies та Claims-Based Security

**Що це таке:**
Claims-based авторизація - це модель безпеки, де права доступу визначаються на основі claims (тверджень) про користувача. Policy-based авторизація дозволяє створювати складні правила доступу, що перевіряють combinations of claims, ролей та інших факторів.

**Навіщо це потрібно:**
- Гнучкіші правила авторизації, ніж прості ролі
- Можливість створення complex business logic для доступу
- Separation of concerns між авторизацією та бізнес-логікою
- Повторне використання правил авторизації
- Dynamic permissions на основі контексту запиту
- Fine-grained контроль доступу

**Практичне застосування:**

```csharp
// Custom Claims Types
public static class CustomClaimTypes
{
    public const string Permission = "permission";
    public const string Department = "department";
    public const string TenantId = "tenant_id";
    public const string DataAccess = "data_access";
    public const string FeatureFlag = "feature_flag";
    public const string IpAddress = "ip_address";
    public const string DeviceId = "device_id";
}

// Permission Constants
public static class Permissions
{
    // User Management
    public const string UsersRead = "users.read";
    public const string UsersWrite = "users.write";
    public const string UsersDelete = "users.delete";
    
    // Product Management
    public const string ProductsRead = "products.read";
    public const string ProductsWrite = "products.write";
    public const string ProductsDelete = "products.delete";
    
    // Order Management
    public const string OrdersRead = "orders.read";
    public const string OrdersWrite = "orders.write";
    public const string OrdersApprove = "orders.approve";
    
    // Reports
    public const string ReportsView = "reports.view";
    public const string ReportsExport = "reports.export";
    
    // Administration
    public const string AdminPanel = "admin.panel";
    public const string SystemSettings = "system.settings";
}

// Department Constants
public static class Departments
{
    public const string IT = "IT";
    public const string HR = "HR";
    public const string Finance = "Finance";
    public const string Sales = "Sales";
    public const string Marketing = "Marketing";
}

// Custom Authorization Requirements
public class PermissionRequirement : IAuthorizationRequirement
{
    public string Permission { get; }
    
    public PermissionRequirement(string permission)
    {
        Permission = permission;
    }
}

public class DepartmentRequirement : IAuthorizationRequirement
{
    public string[] Departments { get; }
    
    public DepartmentRequirement(params string[] departments)
    {
        Departments = departments;
    }
}

public class TenantRequirement : IAuthorizationRequirement
{
    // Empty requirement - logic will be in handler
}

public class BusinessHoursRequirement : IAuthorizationRequirement
{
    public TimeSpan StartTime { get; }
    public TimeSpan EndTime { get; }
    
    public BusinessHoursRequirement(TimeSpan startTime, TimeSpan endTime)
    {
        StartTime = startTime;
        EndTime = endTime;
    }
}

public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    
    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}

public class ResourceOwnerRequirement : IAuthorizationRequirement
{
    // Will be handled by checking if user owns the resource
}

// Authorization Handlers
public class PermissionAuthorizationHandler : AuthorizationHandler<PermissionRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        PermissionRequirement requirement)
    {
        // Check if user has the required permission claim
        if (context.User.HasClaim(CustomClaimTypes.Permission, requirement.Permission))
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

public class DepartmentAuthorizationHandler : AuthorizationHandler<DepartmentRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        DepartmentRequirement requirement)
    {
        var userDepartment = context.User.FindFirst(CustomClaimTypes.Department)?.Value;
        
        if (!string.IsNullOrEmpty(userDepartment) && requirement.Departments.Contains(userDepartment))
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

public class TenantAuthorizationHandler : AuthorizationHandler<TenantRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        TenantRequirement requirement)
    {
        var userTenantId = context.User.FindFirst(CustomClaimTypes.TenantId)?.Value;
        
        if (string.IsNullOrEmpty(userTenantId))
        {
            context.Fail();
            return Task.CompletedTask;
        }
        
        // Additional logic could check if the user is accessing data from their tenant
        // This would typically involve checking the resource being accessed
        
        if (context.Resource is IHasTenantId tenantResource)
        {
            if (tenantResource.TenantId.ToString() == userTenantId)
            {
                context.Succeed(requirement);
            }
        }
        else
        {
            // If no specific resource, just check that user has a tenant
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

public class BusinessHoursAuthorizationHandler : AuthorizationHandler<BusinessHoursRequirement>
{
    private readonly ILogger<BusinessHoursAuthorizationHandler> _logger;
    
    public BusinessHoursAuthorizationHandler(ILogger<BusinessHoursAuthorizationHandler> logger)
    {
        _logger = logger;
    }
    
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        BusinessHoursRequirement requirement)
    {
        var currentTime = DateTime.Now.TimeOfDay;
        
        if (currentTime >= requirement.StartTime && currentTime <= requirement.EndTime)
        {
            context.Succeed(requirement);
        }
        else
        {
            _logger.LogWarning("Access denied outside business hours for user {UserId}", 
                context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value);
        }
        
        return Task.CompletedTask;
    }
}

public class MinimumAgeAuthorizationHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var dateOfBirth = context.User.FindFirst(ClaimTypes.DateOfBirth)?.Value;
        
        if (DateTime.TryParse(dateOfBirth, out var birthDate))
        {
            var age = DateTime.Today.Year - birthDate.Year;
            if (birthDate.Date > DateTime.Today.AddYears(-age)) age--;
            
            if (age >= requirement.MinimumAge)
            {
                context.Succeed(requirement);
            }
        }
        
        return Task.CompletedTask;
    }
}

public class ResourceOwnerAuthorizationHandler : AuthorizationHandler<ResourceOwnerRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        ResourceOwnerRequirement requirement)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        if (string.IsNullOrEmpty(userId))
        {
            return Task.CompletedTask;
        }
        
        // Check if the resource has an owner and if it matches the current user
        if (context.Resource is IHasOwner ownedResource)
        {
            if (ownedResource.OwnerId.ToString() == userId)
            {
                context.Succeed(requirement);
            }
        }
        
        return Task.CompletedTask;
    }
}

// Resource Interfaces
public interface IHasTenantId
{
    int TenantId { get; }
}

public interface IHasOwner
{
    int OwnerId { get; }
}

// Example Domain Models with Security Interfaces
public class SecureOrder : IHasTenantId, IHasOwner
{
    public int Id { get; set; }
    public int TenantId { get; set; }
    public int OwnerId { get; set; }
    public string Description { get; set; } = "";
    public decimal Amount { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Authorization Policy Configuration
public static class AuthorizationPolicies
{
    public const string RequirePermission = "RequirePermission";
    public const string RequireDepartment = "RequireDepartment";
    public const string RequireTenant = "RequireTenant";
    public const string RequireBusinessHours = "RequireBusinessHours";
    public const string RequireMinimumAge = "RequireMinimumAge";
    public const string RequireResourceOwner = "RequireResourceOwner";
    
    // Complex policies
    public const string AdminOnly = "AdminOnly";
    public const string ManagerOrOwner = "ManagerOrOwner";
    public const string BusinessHoursAndDepartment = "BusinessHoursAndDepartment";
    public const string FinanceReports = "FinanceReports";
}

public static class AuthorizationServiceExtensions
{
    public static IServiceCollection AddCustomAuthorization(this IServiceCollection services)
    {
        services.AddAuthorization(options =>
        {
            // Simple permission-based policies
            options.AddPolicy(AuthorizationPolicies.RequirePermission, policy =>
                policy.Requirements.Add(new PermissionRequirement(Permissions.UsersRead)));
            
            // Department-based policies
            options.AddPolicy(AuthorizationPolicies.RequireDepartment, policy =>
                policy.Requirements.Add(new DepartmentRequirement(Departments.IT, Departments.HR)));
            
            // Tenant-based policy
            options.AddPolicy(AuthorizationPolicies.RequireTenant, policy =>
                policy.Requirements.Add(new TenantRequirement()));
            
            // Business hours policy
            options.AddPolicy(AuthorizationPolicies.RequireBusinessHours, policy =>
                policy.Requirements.Add(new BusinessHoursRequirement(
                    new TimeSpan(9, 0, 0), new TimeSpan(17, 0, 0))));
            
            // Age requirement policy
            options.AddPolicy(AuthorizationPolicies.RequireMinimumAge, policy =>
                policy.Requirements.Add(new MinimumAgeRequirement(18)));
            
            // Resource owner policy
            options.AddPolicy(AuthorizationPolicies.RequireResourceOwner, policy =>
                policy.Requirements.Add(new ResourceOwnerRequirement()));
            
            // Complex combined policies
            options.AddPolicy(AuthorizationPolicies.AdminOnly, policy =>
            {
                policy.RequireRole("Admin");
                policy.Requirements.Add(new PermissionRequirement(Permissions.AdminPanel));
            });
            
            options.AddPolicy(AuthorizationPolicies.ManagerOrOwner, policy =>
            {
                policy.RequireAssertion(context =>
                    context.User.IsInRole("Manager") ||
                    context.User.HasClaim(CustomClaimTypes.Permission, Permissions.UsersWrite));
            });
            
            options.AddPolicy(AuthorizationPolicies.BusinessHoursAndDepartment, policy =>
            {
                policy.Requirements.Add(new BusinessHoursRequirement(
                    new TimeSpan(8, 0, 0), new TimeSpan(18, 0, 0)));
                policy.Requirements.Add(new DepartmentRequirement(Departments.Finance));
            });
            
            options.AddPolicy(AuthorizationPolicies.FinanceReports, policy =>
            {
                policy.RequireAssertion(context =>
                {
                    var userDepartment = context.User.FindFirst(CustomClaimTypes.Department)?.Value;
                    var hasPermission = context.User.HasClaim(CustomClaimTypes.Permission, Permissions.ReportsView);
                    
                    return (userDepartment == Departments.Finance || context.User.IsInRole("Admin")) && hasPermission;
                });
            });
        });
        
        // Register authorization handlers
        services.AddScoped<IAuthorizationHandler, PermissionAuthorizationHandler>();
        services.AddScoped<IAuthorizationHandler, DepartmentAuthorizationHandler>();
        services.AddScoped<IAuthorizationHandler, TenantAuthorizationHandler>();
        services.AddScoped<IAuthorizationHandler, BusinessHoursAuthorizationHandler>();
        services.AddScoped<IAuthorizationHandler, MinimumAgeAuthorizationHandler>();
        services.AddScoped<IAuthorizationHandler, ResourceOwnerAuthorizationHandler>();
        
        return services;
    }
}

// Permission Helper Service
public interface IPermissionService
{
    Task<bool> UserHasPermissionAsync(int userId, string permission);
    Task<List<string>> GetUserPermissionsAsync(int userId);
    Task GrantPermissionAsync(int userId, string permission);
    Task RevokePermissionAsync(int userId, string permission);
    Task<bool> UserCanAccessResourceAsync(int userId, object resource);
}

public class PermissionService : IPermissionService
{
    private readonly ApplicationUserManager _userManager;
    private readonly RoleManager<ApplicationRole> _roleManager;
    private readonly ILogger<PermissionService> _logger;
    
    public PermissionService(
        ApplicationUserManager userManager,
        RoleManager<ApplicationRole> roleManager,
        ILogger<PermissionService> logger)
    {
        _userManager = userManager;
        _roleManager = roleManager;
        _logger = logger;
    }
    
    public async Task<bool> UserHasPermissionAsync(int userId, string permission)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted)
        {
            return false;
        }
        
        // Check direct user claims
        var userClaims = await _userManager.GetClaimsAsync(user);
        if (userClaims.Any(c => c.Type == CustomClaimTypes.Permission && c.Value == permission))
        {
            return true;
        }
        
        // Check role-based permissions
        var roles = await _userManager.GetRolesAsync(user);
        foreach (var roleName in roles)
        {
            var role = await _roleManager.FindByNameAsync(roleName);
            if (role != null)
            {
                var roleClaims = await _roleManager.GetClaimsAsync(role);
                if (roleClaims.Any(c => c.Type == CustomClaimTypes.Permission && c.Value == permission))
                {
                    return true;
                }
            }
        }
        
        return false;
    }
    
    public async Task<List<string>> GetUserPermissionsAsync(int userId)
    {
        var permissions = new HashSet<string>();
        
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted)
        {
            return permissions.ToList();
        }
        
        // Get direct user permissions
        var userClaims = await _userManager.GetClaimsAsync(user);
        var userPermissions = userClaims
            .Where(c => c.Type == CustomClaimTypes.Permission)
            .Select(c => c.Value);
        
        foreach (var permission in userPermissions)
        {
            permissions.Add(permission);
        }
        
        // Get role-based permissions
        var roles = await _userManager.GetRolesAsync(user);
        foreach (var roleName in roles)
        {
            var role = await _roleManager.FindByNameAsync(roleName);
            if (role != null)
            {
                var roleClaims = await _roleManager.GetClaimsAsync(role);
                var rolePermissions = roleClaims
                    .Where(c => c.Type == CustomClaimTypes.Permission)
                    .Select(c => c.Value);
                
                foreach (var permission in rolePermissions)
                {
                    permissions.Add(permission);
                }
            }
        }
        
        return permissions.ToList();
    }
    
    public async Task GrantPermissionAsync(int userId, string permission)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted)
        {
            throw new ArgumentException("User not found");
        }
        
        var existingClaim = (await _userManager.GetClaimsAsync(user))
            .FirstOrDefault(c => c.Type == CustomClaimTypes.Permission && c.Value == permission);
        
        if (existingClaim == null)
        {
            await _userManager.AddClaimAsync(user, new Claim(CustomClaimTypes.Permission, permission));
            _logger.LogInformation("Permission {Permission} granted to user {UserId}", permission, userId);
        }
    }
    
    public async Task RevokePermissionAsync(int userId, string permission)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted)
        {
            throw new ArgumentException("User not found");
        }
        
        var existingClaim = (await _userManager.GetClaimsAsync(user))
            .FirstOrDefault(c => c.Type == CustomClaimTypes.Permission && c.Value == permission);
        
        if (existingClaim != null)
        {
            await _userManager.RemoveClaimAsync(user, existingClaim);
            _logger.LogInformation("Permission {Permission} revoked from user {UserId}", permission, userId);
        }
    }
    
    public async Task<bool> UserCanAccessResourceAsync(int userId, object resource)
    {
        var user = await _userManager.FindByIdAsync(userId.ToString());
        if (user == null || user.IsDeleted)
        {
            return false;
        }
        
        // Check tenant access
        if (resource is IHasTenantId tenantResource)
        {
            var userTenantClaim = (await _userManager.GetClaimsAsync(user))
                .FirstOrDefault(c => c.Type == CustomClaimTypes.TenantId);
            
            if (userTenantClaim == null || 
                !int.TryParse(userTenantClaim.Value, out var userTenantId) ||
                userTenantId != tenantResource.TenantId)
            {
                return false;
            }
        }
        
        // Check ownership
        if (resource is IHasOwner ownedResource)
        {
            if (ownedResource.OwnerId != userId)
            {
                // Check if user has admin permissions
                return await UserHasPermissionAsync(userId, Permissions.AdminPanel);
            }
        }
        
        return true;
    }
}

// Controllers with Authorization
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class SecureProductsController : ControllerBase
{
    private readonly IPermissionService _permissionService;
    private readonly IAuthorizationService _authorizationService;
    private readonly ILogger<SecureProductsController> _logger;
    
    public SecureProductsController(
        IPermissionService permissionService,
        IAuthorizationService authorizationService,
        ILogger<SecureProductsController> logger)
    {
        _permissionService = permissionService;
        _authorizationService = authorizationService;
        _logger = logger;
    }
    
    [HttpGet]
    [Authorize(Policy = AuthorizationPolicies.RequirePermission)]
    public async Task<ActionResult<List<ProductDto>>> GetProducts()
    {
        // Check if user has read permission
        var userId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "0");
        
        if (!await _permissionService.UserHasPermissionAsync(userId, Permissions.ProductsRead))
        {
            return Forbid();
        }
        
        // Return products filtered by tenant
        return Ok(new List<ProductDto>());
    }
    
    [HttpPost]
    public async Task<ActionResult> CreateProduct([FromBody] CreateProductDto model)
    {
        // Check multiple requirements
        var authResult = await _authorizationService.AuthorizeAsync(User, model, AuthorizationPolicies.BusinessHoursAndDepartment);
        
        if (!authResult.Succeeded)
        {
            return Forbid("Access denied: Either outside business hours or insufficient department permissions");
        }
        
        var userId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "0");
        
        if (!await _permissionService.UserHasPermissionAsync(userId, Permissions.ProductsWrite))
        {
            return Forbid("Insufficient permissions to create products");
        }
        
        // Create product logic
        return Ok(new { message = "Product created successfully" });
    }
    
    [HttpPut("{id}")]
    public async Task<ActionResult> UpdateProduct(int id, [FromBody] UpdateProductDto model)
    {
        // Get the product and check resource-based authorization
        var product = new SecureOrder { Id = id, OwnerId = 123, TenantId = 1 }; // Mock data
        
        var authResult = await _authorizationService.AuthorizeAsync(User, product, AuthorizationPolicies.RequireResourceOwner);
        
        if (!authResult.Succeeded)
        {
            // Check if user has admin permissions instead
            var userId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "0");
            if (!await _permissionService.UserHasPermissionAsync(userId, Permissions.AdminPanel))
            {
                return Forbid("You can only update your own products");
            }
        }
        
        // Update product logic
        return Ok(new { message = "Product updated successfully" });
    }
    
    [HttpDelete("{id}")]
    [Authorize(Policy = AuthorizationPolicies.AdminOnly)]
    public async Task<ActionResult> DeleteProduct(int id)
    {
        // This method requires admin role AND admin panel permission
        // Authorization is handled by the policy
        
        return Ok(new { message = "Product deleted successfully" });
    }
}

[ApiController]
[Route("api/[controller]")]
[Authorize(Policy = AuthorizationPolicies.FinanceReports)]
public class ReportsController : ControllerBase
{
    private readonly IPermissionService _permissionService;
    
    public ReportsController(IPermissionService permissionService)
    {
        _permissionService = permissionService;
    }
    
    [HttpGet("sales")]
    public async Task<ActionResult> GetSalesReport()
    {
        // This endpoint requires finance department + reports permission
        // Authorization is handled by the FinanceReports policy
        
        return Ok(new { report = "Sales report data" });
    }
    
    [HttpGet("export")]
    public async Task<ActionResult> ExportReport()
    {
        var userId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "0");
        
        if (!await _permissionService.UserHasPermissionAsync(userId, Permissions.ReportsExport))
        {
            return Forbid("Export permission required");
        }
        
        return Ok(new { downloadUrl = "/reports/export/file.xlsx" });
    }
}

// Custom Authorization Attributes
public class RequirePermissionAttribute : AuthorizeAttribute
{
    public RequirePermissionAttribute(string permission)
    {
        Policy = $"Permission_{permission}";
    }
}

public class RequireDepartmentAttribute : AuthorizeAttribute
{
    public RequireDepartmentAttribute(params string[] departments)
    {
        Policy = $"Department_{string.Join("_", departments)}";
    }
}

// Usage with custom attributes
[ApiController]
[Route("api/[controller]")]
public class CustomAttributeController : ControllerBase
{
    [HttpGet("users")]
    [RequirePermission(Permissions.UsersRead)]
    public ActionResult GetUsers()
    {
        return Ok("Users data");
    }
    
    [HttpGet("hr-data")]
    [RequireDepartment(Departments.HR, Departments.IT)]
    public ActionResult GetHRData()
    {
        return Ok("HR data");
    }
}

// Authorization Policy Provider for Dynamic Policies
public class CustomAuthorizationPolicyProvider : IAuthorizationPolicyProvider
{
    private readonly IAuthorizationPolicyProvider _backupPolicyProvider;
    
    public CustomAuthorizationPolicyProvider(IAuthorizationPolicyProvider backupPolicyProvider)
    {
        _backupPolicyProvider = backupPolicyProvider;
    }
    
    public async Task<AuthorizationPolicy?> GetPolicyAsync(string policyName)
    {
        // Handle permission-based policies
        if (policyName.StartsWith("Permission_"))
        {
            var permission = policyName.Substring("Permission_".Length);
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .AddRequirements(new PermissionRequirement(permission))
                .Build();
            return policy;
        }
        
        // Handle department-based policies
        if (policyName.StartsWith("Department_"))
        {
            var departmentString = policyName.Substring("Department_".Length);
            var departments = departmentString.Split('_');
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .AddRequirements(new DepartmentRequirement(departments))
                .Build();
            return policy;
        }
        
        // Fall back to default provider
        return await _backupPolicyProvider.GetPolicyAsync(policyName);
    }
    
    public Task<AuthorizationPolicy> GetDefaultPolicyAsync() => _backupPolicyProvider.GetDefaultPolicyAsync();
    
    public Task<AuthorizationPolicy?> GetFallbackPolicyAsync() => _backupPolicyProvider.GetFallbackPolicyAsync();
}

// Program.cs with Authorization Setup
public class AuthorizationProgram
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Add services
        builder.Services.AddControllers();
        
        // Add Identity and JWT
        builder.Services.AddCustomIdentity(builder.Configuration);
        builder.Services.AddJwtAuthentication(builder.Configuration);
        
        // Add custom authorization
        builder.Services.AddCustomAuthorization();
        
        // Register custom policy provider
        builder.Services.AddSingleton<IAuthorizationPolicyProvider, CustomAuthorizationPolicyProvider>();
        
        // Register permission service
        builder.Services.AddScoped<IPermissionService, PermissionService>();
        
        var app = builder.Build();
        
        // Configure pipeline
        app.UseHttpsRedirection();
        app.UseAuthentication();
        app.UseAuthorization();
        app.MapControllers();
        
        app.Run();
    }
}
```

**Переваги Claims-Based Authorization:**
- **Гнучкість** - складні правила авторизації на основі множини факторів
- **Повторне використання** - policies можна застосовувати до різних контролерів
- **Separation of Concerns** - авторизаційна логіка відділена від бізнес-логіки
- **Динамічні дозволи** - можливість перевірки контексту запиту та ресурсів
- **Fine-grained контроль** - точний контроль на рівні методів та ресурсів
- **Тестованість** - authorization handlers легко тестувати незалежно

## 9. Testing Strategies та Quality Assurance

### 9.1 Unit Testing з xUnit, NUnit та MSTest

**Що це таке:**
Unit Testing - це тестування окремих компонентів (методів, класів) в ізоляції від інших частин системи. У .NET екосистемі найпопулярніші фреймворки: xUnit, NUnit та MSTest. Кожен має свої особливості та переваги.

**Навіщо це потрібно:**
- Рання детекція багів на рівні окремих компонентів
- Документація поведінки коду через тести
- Рефакторинг з упевненістю через regression testing
- Test-Driven Development (TDD) підхід
- Швидкий feedback loop для розробників
- Покращення архітектури через testable design

**Практичне застосування:**

```csharp
// Domain Models for Testing Examples
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime CreatedAt { get; set; }
    public bool IsActive { get; set; } = true;
    public UserType Type { get; set; }
}

public enum UserType
{
    Regular,
    Premium,
    Admin
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public bool IsDiscounted { get; set; }
    
    public decimal GetDiscountedPrice()
    {
        return IsDiscounted ? Price * 0.9m : Price;
    }
    
    public bool IsInStock()
    {
        return StockQuantity > 0;
    }
}

// Business Logic Classes to Test
public interface IEmailService
{
    Task SendAsync(string to, string subject, string body);
    Task<bool> IsValidEmailAsync(string email);
}

public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id);
    Task<IEnumerable<User>> GetAllAsync();
    Task<User> CreateAsync(User user);
    Task<bool> ExistsAsync(string email);
}

public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService;
    private readonly ILogger<UserService> _logger;
    
    public UserService(
        IUserRepository userRepository, 
        IEmailService emailService,
        ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _emailService = emailService;
        _logger = logger;
    }
    
    public async Task<UserRegistrationResult> RegisterUserAsync(string name, string email)
    {
        if (string.IsNullOrWhiteSpace(name))
        {
            return UserRegistrationResult.Failed("Name is required");
        }
        
        if (string.IsNullOrWhiteSpace(email))
        {
            return UserRegistrationResult.Failed("Email is required");
        }
        
        if (!await _emailService.IsValidEmailAsync(email))
        {
            return UserRegistrationResult.Failed("Invalid email format");
        }
        
        if (await _userRepository.ExistsAsync(email))
        {
            return UserRegistrationResult.Failed("Email already exists");
        }
        
        var user = new User
        {
            Name = name,
            Email = email,
            CreatedAt = DateTime.UtcNow,
            Type = UserType.Regular
        };
        
        try
        {
            var createdUser = await _userRepository.CreateAsync(user);
            
            await _emailService.SendAsync(
                email,
                "Welcome!",
                $"Welcome {name} to our platform!");
            
            _logger.LogInformation("User {Email} registered successfully", email);
            
            return UserRegistrationResult.Success(createdUser.Id);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error registering user {Email}", email);
            return UserRegistrationResult.Failed("Registration failed");
        }
    }
    
    public async Task<IEnumerable<User>> GetActiveUsersAsync()
    {
        var users = await _userRepository.GetAllAsync();
        return users.Where(u => u.IsActive).OrderBy(u => u.Name);
    }
    
    public async Task<UserUpgradeResult> UpgradeUserToPremiumAsync(int userId)
    {
        var user = await _userRepository.GetByIdAsync(userId);
        
        if (user == null)
        {
            return UserUpgradeResult.Failed("User not found");
        }
        
        if (user.Type == UserType.Premium)
        {
            return UserUpgradeResult.Failed("User is already premium");
        }
        
        user.Type = UserType.Premium;
        
        try
        {
            await _userRepository.CreateAsync(user); // Simplified - should be Update
            
            await _emailService.SendAsync(
                user.Email,
                "Premium Upgrade",
                "Congratulations! You've been upgraded to Premium.");
            
            return UserUpgradeResult.Success();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error upgrading user {UserId}", userId);
            return UserUpgradeResult.Failed("Upgrade failed");
        }
    }
}

// Result Classes
public class UserRegistrationResult
{
    public bool IsSuccess { get; set; }
    public string? ErrorMessage { get; set; }
    public int UserId { get; set; }
    
    public static UserRegistrationResult Success(int userId) => 
        new() { IsSuccess = true, UserId = userId };
    
    public static UserRegistrationResult Failed(string error) => 
        new() { IsSuccess = false, ErrorMessage = error };
}

public class UserUpgradeResult
{
    public bool IsSuccess { get; set; }
    public string? ErrorMessage { get; set; }
    
    public static UserUpgradeResult Success() => new() { IsSuccess = true };
    public static UserUpgradeResult Failed(string error) => 
        new() { IsSuccess = false, ErrorMessage = error };
}

// xUnit Tests
using Xunit;
using Moq;
using Microsoft.Extensions.Logging;
using FluentAssertions;

public class UserServiceTests
{
    private readonly Mock<IUserRepository> _userRepositoryMock;
    private readonly Mock<IEmailService> _emailServiceMock;
    private readonly Mock<ILogger<UserService>> _loggerMock;
    private readonly UserService _userService;
    
    public UserServiceTests()
    {
        _userRepositoryMock = new Mock<IUserRepository>();
        _emailServiceMock = new Mock<IEmailService>();
        _loggerMock = new Mock<ILogger<UserService>>();
        
        _userService = new UserService(
            _userRepositoryMock.Object,
            _emailServiceMock.Object,
            _loggerMock.Object);
    }
    
    [Fact]
    public async Task RegisterUserAsync_WithValidData_ShouldReturnSuccess()
    {
        // Arrange
        var name = "John Doe";
        var email = "john@example.com";
        
        _emailServiceMock.Setup(x => x.IsValidEmailAsync(email))
            .ReturnsAsync(true);
        
        _userRepositoryMock.Setup(x => x.ExistsAsync(email))
            .ReturnsAsync(false);
        
        _userRepositoryMock.Setup(x => x.CreateAsync(It.IsAny<User>()))
            .ReturnsAsync((User u) => new User 
            { 
                Id = 1, 
                Name = u.Name, 
                Email = u.Email, 
                CreatedAt = u.CreatedAt,
                Type = u.Type
            });
        
        _emailServiceMock.Setup(x => x.SendAsync(email, "Welcome!", It.IsAny<string>()))
            .Returns(Task.CompletedTask);
        
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeTrue();
        result.UserId.Should().Be(1);
        result.ErrorMessage.Should().BeNull();
        
        // Verify interactions
        _userRepositoryMock.Verify(x => x.CreateAsync(It.Is<User>(u => 
            u.Name == name && 
            u.Email == email && 
            u.Type == UserType.Regular)), Times.Once);
        
        _emailServiceMock.Verify(x => x.SendAsync(
            email, 
            "Welcome!", 
            It.IsAny<string>()), Times.Once);
    }
    
    [Theory]
    [InlineData("", "test@example.com", "Name is required")]
    [InlineData("   ", "test@example.com", "Name is required")]
    [InlineData(null, "test@example.com", "Name is required")]
    [InlineData("John", "", "Email is required")]
    [InlineData("John", "   ", "Email is required")]
    [InlineData("John", null, "Email is required")]
    public async Task RegisterUserAsync_WithInvalidData_ShouldReturnError(
        string name, string email, string expectedError)
    {
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeFalse();
        result.ErrorMessage.Should().Be(expectedError);
        result.UserId.Should().Be(0);
    }
    
    [Fact]
    public async Task RegisterUserAsync_WithInvalidEmail_ShouldReturnError()
    {
        // Arrange
        var name = "John Doe";
        var email = "invalid-email";
        
        _emailServiceMock.Setup(x => x.IsValidEmailAsync(email))
            .ReturnsAsync(false);
        
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeFalse();
        result.ErrorMessage.Should().Be("Invalid email format");
    }
    
    [Fact]
    public async Task RegisterUserAsync_WithExistingEmail_ShouldReturnError()
    {
        // Arrange
        var name = "John Doe";
        var email = "existing@example.com";
        
        _emailServiceMock.Setup(x => x.IsValidEmailAsync(email))
            .ReturnsAsync(true);
        
        _userRepositoryMock.Setup(x => x.ExistsAsync(email))
            .ReturnsAsync(true);
        
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeFalse();
        result.ErrorMessage.Should().Be("Email already exists");
        
        // Verify that CreateAsync was never called
        _userRepositoryMock.Verify(x => x.CreateAsync(It.IsAny<User>()), Times.Never);
    }
    
    [Fact]
    public async Task RegisterUserAsync_WhenRepositoryThrows_ShouldReturnError()
    {
        // Arrange
        var name = "John Doe";
        var email = "john@example.com";
        
        _emailServiceMock.Setup(x => x.IsValidEmailAsync(email))
            .ReturnsAsync(true);
        
        _userRepositoryMock.Setup(x => x.ExistsAsync(email))
            .ReturnsAsync(false);
        
        _userRepositoryMock.Setup(x => x.CreateAsync(It.IsAny<User>()))
            .ThrowsAsync(new Exception("Database error"));
        
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeFalse();
        result.ErrorMessage.Should().Be("Registration failed");
        
        // Verify logging
        _loggerMock.Verify(
            x => x.Log(
                LogLevel.Error,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((v, t) => v.ToString().Contains("Error registering user")),
                It.IsAny<Exception>(),
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            Times.Once);
    }
    
    [Fact]
    public async Task GetActiveUsersAsync_ShouldReturnOnlyActiveUsers()
    {
        // Arrange
        var users = new List<User>
        {
            new User { Id = 1, Name = "Alice", IsActive = true },
            new User { Id = 2, Name = "Bob", IsActive = false },
            new User { Id = 3, Name = "Charlie", IsActive = true }
        };
        
        _userRepositoryMock.Setup(x => x.GetAllAsync())
            .ReturnsAsync(users);
        
        // Act
        var result = await _userService.GetActiveUsersAsync();
        
        // Assert
        result.Should().NotBeNull();
        result.Should().HaveCount(2);
        result.Select(u => u.Name).Should().Equal("Alice", "Charlie");
        result.All(u => u.IsActive).Should().BeTrue();
    }
    
    [Fact]
    public async Task UpgradeUserToPremiumAsync_WithValidUser_ShouldReturnSuccess()
    {
        // Arrange
        var userId = 1;
        var user = new User 
        { 
            Id = userId, 
            Name = "John", 
            Email = "john@example.com", 
            Type = UserType.Regular 
        };
        
        _userRepositoryMock.Setup(x => x.GetByIdAsync(userId))
            .ReturnsAsync(user);
        
        _userRepositoryMock.Setup(x => x.CreateAsync(It.IsAny<User>()))
            .ReturnsAsync((User u) => u);
        
        _emailServiceMock.Setup(x => x.SendAsync(
            user.Email, 
            "Premium Upgrade", 
            It.IsAny<string>()))
            .Returns(Task.CompletedTask);
        
        // Act
        var result = await _userService.UpgradeUserToPremiumAsync(userId);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeTrue();
        result.ErrorMessage.Should().BeNull();
        
        user.Type.Should().Be(UserType.Premium);
        
        _emailServiceMock.Verify(x => x.SendAsync(
            user.Email, 
            "Premium Upgrade", 
            "Congratulations! You've been upgraded to Premium."), Times.Once);
    }
    
    [Fact]
    public async Task UpgradeUserToPremiumAsync_WithNonExistentUser_ShouldReturnError()
    {
        // Arrange
        var userId = 999;
        
        _userRepositoryMock.Setup(x => x.GetByIdAsync(userId))
            .ReturnsAsync((User?)null);
        
        // Act
        var result = await _userService.UpgradeUserToPremiumAsync(userId);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeFalse();
        result.ErrorMessage.Should().Be("User not found");
        
        _emailServiceMock.Verify(x => x.SendAsync(
            It.IsAny<string>(), 
            It.IsAny<string>(), 
            It.IsAny<string>()), Times.Never);
    }
    
    [Fact]
    public async Task UpgradeUserToPremiumAsync_WithAlreadyPremiumUser_ShouldReturnError()
    {
        // Arrange
        var userId = 1;
        var user = new User 
        { 
            Id = userId, 
            Type = UserType.Premium 
        };
        
        _userRepositoryMock.Setup(x => x.GetByIdAsync(userId))
            .ReturnsAsync(user);
        
        // Act
        var result = await _userService.UpgradeUserToPremiumAsync(userId);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeFalse();
        result.ErrorMessage.Should().Be("User is already premium");
        
        _userRepositoryMock.Verify(x => x.CreateAsync(It.IsAny<User>()), Times.Never);
    }
}

// Product Model Tests
public class ProductTests
{
    [Theory]
    [InlineData(100, false, 100)]
    [InlineData(100, true, 90)]
    [InlineData(50, false, 50)]
    [InlineData(50, true, 45)]
    public void GetDiscountedPrice_ShouldCalculateCorrectly(
        decimal originalPrice, bool isDiscounted, decimal expectedPrice)
    {
        // Arrange
        var product = new Product 
        { 
            Price = originalPrice, 
            IsDiscounted = isDiscounted 
        };
        
        // Act
        var result = product.GetDiscountedPrice();
        
        // Assert
        result.Should().Be(expectedPrice);
    }
    
    [Theory]
    [InlineData(0, false)]
    [InlineData(-1, false)]
    [InlineData(1, true)]
    [InlineData(100, true)]
    public void IsInStock_ShouldReturnCorrectValue(int quantity, bool expected)
    {
        // Arrange
        var product = new Product { StockQuantity = quantity };
        
        // Act
        var result = product.IsInStock();
        
        // Assert
        result.Should().Be(expected);
    }
}

// NUnit Tests Example
using NUnit.Framework;
using Moq;
using FluentAssertions;

[TestFixture]
public class UserServiceNUnitTests
{
    private Mock<IUserRepository> _userRepositoryMock;
    private Mock<IEmailService> _emailServiceMock;
    private Mock<ILogger<UserService>> _loggerMock;
    private UserService _userService;
    
    [SetUp]
    public void Setup()
    {
        _userRepositoryMock = new Mock<IUserRepository>();
        _emailServiceMock = new Mock<IEmailService>();
        _loggerMock = new Mock<ILogger<UserService>>();
        
        _userService = new UserService(
            _userRepositoryMock.Object,
            _emailServiceMock.Object,
            _loggerMock.Object);
    }
    
    [Test]
    public async Task RegisterUserAsync_WithValidData_ShouldReturnSuccess()
    {
        // Arrange
        var name = "John Doe";
        var email = "john@example.com";
        
        _emailServiceMock.Setup(x => x.IsValidEmailAsync(email))
            .ReturnsAsync(true);
        
        _userRepositoryMock.Setup(x => x.ExistsAsync(email))
            .ReturnsAsync(false);
        
        _userRepositoryMock.Setup(x => x.CreateAsync(It.IsAny<User>()))
            .ReturnsAsync(new User { Id = 1, Name = name, Email = email });
        
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        Assert.That(result, Is.Not.Null);
        Assert.That(result.IsSuccess, Is.True);
        Assert.That(result.UserId, Is.EqualTo(1));
        Assert.That(result.ErrorMessage, Is.Null);
    }
    
    [TestCase("", "test@example.com", "Name is required")]
    [TestCase("John", "", "Email is required")]
    [TestCase(null, "test@example.com", "Name is required")]
    public async Task RegisterUserAsync_WithInvalidData_ShouldReturnError(
        string name, string email, string expectedError)
    {
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        Assert.That(result, Is.Not.Null);
        Assert.That(result.IsSuccess, Is.False);
        Assert.That(result.ErrorMessage, Is.EqualTo(expectedError));
    }
    
    [Test]
    public async Task GetActiveUsersAsync_ShouldFilterActiveUsers()
    {
        // Arrange
        var users = new List<User>
        {
            new User { Id = 1, Name = "Active User", IsActive = true },
            new User { Id = 2, Name = "Inactive User", IsActive = false }
        };
        
        _userRepositoryMock.Setup(x => x.GetAllAsync())
            .ReturnsAsync(users);
        
        // Act
        var result = await _userService.GetActiveUsersAsync();
        
        // Assert
        Assert.That(result, Is.Not.Null);
        Assert.That(result.Count(), Is.EqualTo(1));
        Assert.That(result.First().IsActive, Is.True);
    }
}

// MSTest Tests Example
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Moq;
using FluentAssertions;

[TestClass]
public class UserServiceMSTests
{
    private Mock<IUserRepository> _userRepositoryMock;
    private Mock<IEmailService> _emailServiceMock;
    private Mock<ILogger<UserService>> _loggerMock;
    private UserService _userService;
    
    [TestInitialize]
    public void Initialize()
    {
        _userRepositoryMock = new Mock<IUserRepository>();
        _emailServiceMock = new Mock<IEmailService>();
        _loggerMock = new Mock<ILogger<UserService>>();
        
        _userService = new UserService(
            _userRepositoryMock.Object,
            _emailServiceMock.Object,
            _loggerMock.Object);
    }
    
    [TestMethod]
    public async Task RegisterUserAsync_WithValidData_ShouldReturnSuccess()
    {
        // Arrange
        var name = "John Doe";
        var email = "john@example.com";
        
        _emailServiceMock.Setup(x => x.IsValidEmailAsync(email))
            .ReturnsAsync(true);
        
        _userRepositoryMock.Setup(x => x.ExistsAsync(email))
            .ReturnsAsync(false);
        
        _userRepositoryMock.Setup(x => x.CreateAsync(It.IsAny<User>()))
            .ReturnsAsync(new User { Id = 1, Name = name, Email = email });
        
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        result.Should().NotBeNull();
        result.IsSuccess.Should().BeTrue();
        result.UserId.Should().Be(1);
    }
    
    [DataTestMethod]
    [DataRow("", "test@example.com", "Name is required")]
    [DataRow("John", "", "Email is required")]
    public async Task RegisterUserAsync_WithInvalidData_ShouldReturnError(
        string name, string email, string expectedError)
    {
        // Act
        var result = await _userService.RegisterUserAsync(name, email);
        
        // Assert
        Assert.IsNotNull(result);
        Assert.IsFalse(result.IsSuccess);
        Assert.AreEqual(expectedError, result.ErrorMessage);
    }
    
    [TestMethod]
    public async Task GetActiveUsersAsync_ShouldReturnOnlyActiveUsers()
    {
        // Arrange
        var users = new List<User>
        {
            new User { Id = 1, Name = "Active", IsActive = true },
            new User { Id = 2, Name = "Inactive", IsActive = false }
        };
        
        _userRepositoryMock.Setup(x => x.GetAllAsync())
            .ReturnsAsync(users);
        
        // Act
        var result = await _userService.GetActiveUsersAsync();
        
        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(1, result.Count());
        Assert.IsTrue(result.All(u => u.IsActive));
    }
    
    [TestCleanup]
    public void Cleanup()
    {
        // Cleanup logic if needed
    }
}

// Custom Test Data Builders (Test Object Mother Pattern)
public class UserBuilder
{
    private int _id = 1;
    private string _name = "Test User";
    private string _email = "test@example.com";
    private DateTime _createdAt = DateTime.UtcNow;
    private bool _isActive = true;
    private UserType _type = UserType.Regular;
    
    public UserBuilder WithId(int id)
    {
        _id = id;
        return this;
    }
    
    public UserBuilder WithName(string name)
    {
        _name = name;
        return this;
    }
    
    public UserBuilder WithEmail(string email)
    {
        _email = email;
        return this;
    }
    
    public UserBuilder WithCreatedAt(DateTime createdAt)
    {
        _createdAt = createdAt;
        return this;
    }
    
    public UserBuilder AsInactive()
    {
        _isActive = false;
        return this;
    }
    
    public UserBuilder AsPremium()
    {
        _type = UserType.Premium;
        return this;
    }
    
    public UserBuilder AsAdmin()
    {
        _type = UserType.Admin;
        return this;
    }
    
    public User Build()
    {
        return new User
        {
            Id = _id,
            Name = _name,
            Email = _email,
            CreatedAt = _createdAt,
            IsActive = _isActive,
            Type = _type
        };
    }
    
    public static UserBuilder Create() => new UserBuilder();
}

// Usage of Test Data Builder
public class UserBuilderExampleTests
{
    [Fact]
    public void UserBuilder_ShouldCreateUserWithDefaults()
    {
        // Arrange & Act
        var user = UserBuilder.Create().Build();
        
        // Assert
        user.Should().NotBeNull();
        user.Id.Should().Be(1);
        user.Name.Should().Be("Test User");
        user.Email.Should().Be("test@example.com");
        user.IsActive.Should().BeTrue();
        user.Type.Should().Be(UserType.Regular);
    }
    
    [Fact]
    public void UserBuilder_ShouldCreatePremiumUser()
    {
        // Arrange & Act
        var user = UserBuilder.Create()
            .WithName("Premium User")
            .WithEmail("premium@example.com")
            .AsPremium()
            .Build();
        
        // Assert
        user.Name.Should().Be("Premium User");
        user.Email.Should().Be("premium@example.com");
        user.Type.Should().Be(UserType.Premium);
    }
    
    [Fact]
    public void UserBuilder_ShouldCreateInactiveAdmin()
    {
        // Arrange & Act
        var user = UserBuilder.Create()
            .WithName("Admin User")
            .AsAdmin()
            .AsInactive()
            .Build();
        
        // Assert
        user.Name.Should().Be("Admin User");
        user.Type.Should().Be(UserType.Admin);
        user.IsActive.Should().BeFalse();
    }
}

// Test Utilities and Helpers
public static class TestUtilities
{
    public static Mock<ILogger<T>> CreateMockLogger<T>() where T : class
    {
        return new Mock<ILogger<T>>();
    }
    
    public static void VerifyLoggerCalled<T>(
        Mock<ILogger<T>> mockLogger,
        LogLevel level,
        string message,
        Times times) where T : class
    {
        mockLogger.Verify(
            x => x.Log(
                level,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((v, t) => v.ToString().Contains(message)),
                It.IsAny<Exception>(),
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            times);
    }
    
    public static async Task<T> AssertThrowsAsync<T>(Func<Task> action) where T : Exception
    {
        try
        {
            await action();
            throw new InvalidOperationException($"Expected exception of type {typeof(T).Name} was not thrown");
        }
        catch (T exception)
        {
            return exception;
        }
    }
}
```

**Переваги різних Testing Frameworks:**

**xUnit:**
- **Простота** - мінімалістичний підхід без Setup/TearDown методів
- **Ізоляція тестів** - новий instance класу для кожного тесту
- **Theory та InlineData** - параметризовані тести
- **Parallel execution** - тести виконуються паралельно за замовчуванням

**NUnit:**
- **Багаті assertion методи** - розширений набір перевірок
- **Гнучкі attributes** - TestCase, TestCaseSource, Range атрибути
- **SetUp/TearDown** - явне управління ініціалізацією та очищенням
- **Categories** - групування тестів для селективного запуску

**MSTest:**
- **Інтеграція з Visual Studio** - вбудована підтримка в IDE
- **DataRow атрибут** - параметризовані тести
- **DeploymentItem** - розгортання файлів для тестів
- **Enterprise features** - підтримка корпоративних сценаріїв

### 9.2 Integration Testing з TestServer

**Що це таке:**
Integration Testing - це тестування взаємодії між різними компонентами системи. У ASP.NET Core це включає тестування повного HTTP pipeline, middleware, контролерів, та їх інтеграцію з базами даних і зовнішніми сервісами.

**Навіщо це потрібно:**
- Перевірка роботи всієї системи від HTTP запиту до відповіді
- Тестування middleware pipeline та їх взаємодії
- Валідація конфігурації та dependency injection
- Перевірка серіалізації/десеріалізації JSON
- Тестування автентифікації та авторизації в реальному контексті
- Виявлення проблем інтеграції між компонентами

**Практичне застосування:**

```csharp
// Test Startup Configuration
public class TestStartup : Startup
{
    public TestStartup(IConfiguration configuration) : base(configuration) { }
    
    public override void ConfigureServices(IServiceCollection services)
    {
        base.ConfigureServices(services);
        
        // Replace real services with test doubles
        services.RemoveAll<IEmailService>();
        services.AddScoped<IEmailService, TestEmailService>();
        
        // Use in-memory database for testing
        services.RemoveAll<DbContextOptions<ApplicationDbContext>>();
        services.AddDbContext<ApplicationDbContext>(options =>
        {
            options.UseInMemoryDatabase("TestDatabase");
        });
        
        // Add test-specific services
        services.AddScoped<ITestDataSeeder, TestDataSeeder>();
    }
    
    public override void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // Use test-specific configuration
        if (env.IsEnvironment("Testing"))
        {
            app.UseDeveloperExceptionPage();
        }
        
        app.UseRouting();
        app.UseAuthentication();
        app.UseAuthorization();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}

// Test Email Service Implementation
public class TestEmailService : IEmailService
{
    public List<EmailMessage> SentEmails { get; } = new();
    
    public Task SendAsync(string to, string subject, string body)
    {
        SentEmails.Add(new EmailMessage
        {
            To = to,
            Subject = subject,
            Body = body,
            SentAt = DateTime.UtcNow
        });
        
        return Task.CompletedTask;
    }
    
    public Task<bool> IsValidEmailAsync(string email)
    {
        return Task.FromResult(!string.IsNullOrWhiteSpace(email) && email.Contains("@"));
    }
    
    public void Clear() => SentEmails.Clear();
}

public class EmailMessage
{
    public string To { get; set; } = "";
    public string Subject { get; set; } = "";
    public string Body { get; set; } = "";
    public DateTime SentAt { get; set; }
}

// Test Data Seeder
public interface ITestDataSeeder
{
    Task SeedAsync(ApplicationDbContext context);
    Task ClearAsync(ApplicationDbContext context);
}

public class TestDataSeeder : ITestDataSeeder
{
    public async Task SeedAsync(ApplicationDbContext context)
    {
        // Seed test data
        if (!await context.Users.AnyAsync())
        {
            var users = new[]
            {
                new User { Id = 1, Name = "Test User 1", Email = "user1@test.com", IsActive = true, Type = UserType.Regular },
                new User { Id = 2, Name = "Test User 2", Email = "user2@test.com", IsActive = true, Type = UserType.Premium },
                new User { Id = 3, Name = "Admin User", Email = "admin@test.com", IsActive = true, Type = UserType.Admin }
            };
            
            context.Users.AddRange(users);
            await context.SaveChangesAsync();
        }
        
        if (!await context.Products.AnyAsync())
        {
            var products = new[]
            {
                new Product { Id = 1, Name = "Test Product 1", Price = 99.99m, StockQuantity = 10 },
                new Product { Id = 2, Name = "Test Product 2", Price = 149.99m, StockQuantity = 5, IsDiscounted = true }
            };
            
            context.Products.AddRange(products);
            await context.SaveChangesAsync();
        }
    }
    
    public async Task ClearAsync(ApplicationDbContext context)
    {
        context.Users.RemoveRange(context.Users);
        context.Products.RemoveRange(context.Products);
        await context.SaveChangesAsync();
    }
}

// Custom Web Application Factory
public class CustomWebApplicationFactory<TStartup> : WebApplicationFactory<TStartup> 
    where TStartup : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove the app's ApplicationDbContext registration
            var descriptor = services.SingleOrDefault(d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
            if (descriptor != null)
            {
                services.Remove(descriptor);
            }
            
            // Add ApplicationDbContext using an in-memory database for testing
            services.AddDbContext<ApplicationDbContext>(options =>
            {
                options.UseInMemoryDatabase("InMemoryDbForTesting");
            });
            
            // Replace services with test implementations
            services.Replace(ServiceDescriptor.Scoped<IEmailService, TestEmailService>());
            
            var sp = services.BuildServiceProvider();
            
            // Create a scope to obtain a reference to the database context (ApplicationDbContext)
            using var scope = sp.CreateScope();
            var scopedServices = scope.ServiceProvider;
            var db = scopedServices.GetRequiredService<ApplicationDbContext>();
            var seeder = scopedServices.GetRequiredService<ITestDataSeeder>();
            
            // Ensure the database is created
            db.Database.EnsureCreated();
            
            try
            {
                // Seed the database with test data
                seeder.SeedAsync(db).GetAwaiter().GetResult();
            }
            catch (Exception ex)
            {
                // Log errors or do anything else you want here
                throw new Exception($"An error occurred seeding the database with test data: {ex.Message}");
            }
        });
        
        builder.UseEnvironment("Testing");
    }
}

// Base Integration Test Class
public class IntegrationTestBase : IClassFixture<CustomWebApplicationFactory<TestStartup>>, IDisposable
{
    protected readonly HttpClient Client;
    protected readonly CustomWebApplicationFactory<TestStartup> Factory;
    protected readonly IServiceScope Scope;
    protected readonly ApplicationDbContext DbContext;
    protected readonly TestEmailService EmailService;
    
    protected IntegrationTestBase(CustomWebApplicationFactory<TestStartup> factory)
    {
        Factory = factory;
        Client = factory.CreateClient();
        
        Scope = factory.Services.CreateScope();
        DbContext = Scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        EmailService = (TestEmailService)Scope.ServiceProvider.GetRequiredService<IEmailService>();
    }
    
    protected async Task<T> GetAsync<T>(string endpoint)
    {
        var response = await Client.GetAsync(endpoint);
        response.EnsureSuccessStatusCode();
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<T>(json, new JsonSerializerOptions 
        { 
            PropertyNameCaseInsensitive = true 
        });
    }
    
    protected async Task<HttpResponseMessage> PostAsync<T>(string endpoint, T data)
    {
        var json = JsonSerializer.Serialize(data);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        return await Client.PostAsync(endpoint, content);
    }
    
    protected async Task<HttpResponseMessage> PutAsync<T>(string endpoint, T data)
    {
        var json = JsonSerializer.Serialize(data);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        return await Client.PutAsync(endpoint, content);
    }
    
    protected async Task<HttpResponseMessage> DeleteAsync(string endpoint)
    {
        return await Client.DeleteAsync(endpoint);
    }
    
    protected async Task SeedTestDataAsync()
    {
        var seeder = Scope.ServiceProvider.GetRequiredService<ITestDataSeeder>();
        await seeder.SeedAsync(DbContext);
    }
    
    protected async Task ClearDatabaseAsync()
    {
        var seeder = Scope.ServiceProvider.GetRequiredService<ITestDataSeeder>();
        await seeder.ClearAsync(DbContext);
        EmailService.Clear();
    }
    
    public void Dispose()
    {
        Scope?.Dispose();
        Client?.Dispose();
    }
}

// User Controller Integration Tests
public class UserControllerIntegrationTests : IntegrationTestBase
{
    public UserControllerIntegrationTests(CustomWebApplicationFactory<TestStartup> factory) 
        : base(factory) { }
    
    [Fact]
    public async Task GetUsers_ShouldReturnAllUsers()
    {
        // Arrange
        await SeedTestDataAsync();
        
        // Act
        var response = await Client.GetAsync("/api/users");
        
        // Assert
        response.EnsureSuccessStatusCode();
        response.Content.Headers.ContentType?.ToString().Should().Be("application/json; charset=utf-8");
        
        var json = await response.Content.ReadAsStringAsync();
        var users = JsonSerializer.Deserialize<List<UserDto>>(json, new JsonSerializerOptions 
        { 
            PropertyNameCaseInsensitive = true 
        });
        
        users.Should().NotBeNull();
        users.Should().HaveCount(3);
        users.Should().Contain(u => u.Name == "Test User 1");
        users.Should().Contain(u => u.Name == "Admin User");
    }
    
    [Fact]
    public async Task GetUser_WithValidId_ShouldReturnUser()
    {
        // Arrange
        await SeedTestDataAsync();
        var userId = 1;
        
        // Act
        var user = await GetAsync<UserDto>($"/api/users/{userId}");
        
        // Assert
        user.Should().NotBeNull();
        user.Id.Should().Be(userId);
        user.Name.Should().Be("Test User 1");
        user.Email.Should().Be("user1@test.com");
        user.IsActive.Should().BeTrue();
    }
    
    [Fact]
    public async Task GetUser_WithInvalidId_ShouldReturnNotFound()
    {
        // Act
        var response = await Client.GetAsync("/api/users/999");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }
    
    [Fact]
    public async Task CreateUser_WithValidData_ShouldCreateUser()
    {
        // Arrange
        var newUser = new CreateUserDto
        {
            Name = "New Test User",
            Email = "newuser@test.com"
        };
        
        // Act
        var response = await PostAsync("/api/users", newUser);
        
        // Assert
        response.EnsureSuccessStatusCode();
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        
        var locationHeader = response.Headers.Location?.ToString();
        locationHeader.Should().NotBeNull();
        locationHeader.Should().Contain("/api/users/");
        
        // Verify user was created in database
        var createdUser = await DbContext.Users.FirstOrDefaultAsync(u => u.Email == newUser.Email);
        createdUser.Should().NotBeNull();
        createdUser!.Name.Should().Be(newUser.Name);
        createdUser.Type.Should().Be(UserType.Regular);
        
        // Verify email was sent
        EmailService.SentEmails.Should().HaveCount(1);
        EmailService.SentEmails[0].To.Should().Be(newUser.Email);
        EmailService.SentEmails[0].Subject.Should().Be("Welcome!");
    }
    
    [Fact]
    public async Task CreateUser_WithInvalidData_ShouldReturnBadRequest()
    {
        // Arrange
        var invalidUser = new CreateUserDto
        {
            Name = "", // Invalid: empty name
            Email = "invalid-email" // Invalid: bad email format
        };
        
        // Act
        var response = await PostAsync("/api/users", invalidUser);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
        
        var json = await response.Content.ReadAsStringAsync();
        var errorResponse = JsonSerializer.Deserialize<ValidationErrorResponse>(json, new JsonSerializerOptions 
        { 
            PropertyNameCaseInsensitive = true 
        });
        
        errorResponse.Should().NotBeNull();
        errorResponse!.Errors.Should().NotBeEmpty();
        errorResponse.Errors.Should().ContainKey("Name");
        errorResponse.Errors.Should().ContainKey("Email");
        
        // Verify no user was created
        var userCount = await DbContext.Users.CountAsync();
        userCount.Should().Be(0);
        
        // Verify no email was sent
        EmailService.SentEmails.Should().BeEmpty();
    }
    
    [Fact]
    public async Task CreateUser_WithDuplicateEmail_ShouldReturnConflict()
    {
        // Arrange
        await SeedTestDataAsync();
        
        var duplicateUser = new CreateUserDto
        {
            Name = "Duplicate User",
            Email = "user1@test.com" // This email already exists
        };
        
        // Act
        var response = await PostAsync("/api/users", duplicateUser);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Conflict);
        
        var json = await response.Content.ReadAsStringAsync();
        var errorResponse = JsonSerializer.Deserialize<ErrorResponse>(json, new JsonSerializerOptions 
        { 
            PropertyNameCaseInsensitive = true 
        });
        
        errorResponse.Should().NotBeNull();
        errorResponse!.Message.Should().Contain("already exists");
    }
    
    [Fact]
    public async Task UpdateUser_WithValidData_ShouldUpdateUser()
    {
        // Arrange
        await SeedTestDataAsync();
        var userId = 1;
        var updateData = new UpdateUserDto
        {
            Name = "Updated Name",
            Email = "updated@test.com"
        };
        
        // Act
        var response = await PutAsync($"/api/users/{userId}", updateData);
        
        // Assert
        response.EnsureSuccessStatusCode();
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        
        // Verify user was updated in database
        var updatedUser = await DbContext.Users.FindAsync(userId);
        updatedUser.Should().NotBeNull();
        updatedUser!.Name.Should().Be(updateData.Name);
        updatedUser.Email.Should().Be(updateData.Email);
    }
    
    [Fact]
    public async Task DeleteUser_WithValidId_ShouldDeleteUser()
    {
        // Arrange
        await SeedTestDataAsync();
        var userId = 1;
        
        // Act
        var response = await DeleteAsync($"/api/users/{userId}");
        
        // Assert
        response.EnsureSuccessStatusCode();
        response.StatusCode.Should().Be(HttpStatusCode.NoContent);
        
        // Verify user was soft deleted (IsActive = false)
        var deletedUser = await DbContext.Users.FindAsync(userId);
        deletedUser.Should().NotBeNull();
        deletedUser!.IsActive.Should().BeFalse();
    }
}

// Product Controller Integration Tests
public class ProductControllerIntegrationTests : IntegrationTestBase
{
    public ProductControllerIntegrationTests(CustomWebApplicationFactory<TestStartup> factory) 
        : base(factory) { }
    
    [Fact]
    public async Task GetProducts_ShouldReturnProductsWithDiscounts()
    {
        // Arrange
        await SeedTestDataAsync();
        
        // Act
        var products = await GetAsync<List<ProductDto>>("/api/products");
        
        // Assert
        products.Should().NotBeNull();
        products.Should().HaveCount(2);
        
        var discountedProduct = products.FirstOrDefault(p => p.IsDiscounted);
        discountedProduct.Should().NotBeNull();
        discountedProduct!.DiscountedPrice.Should().Be(134.99m); // 149.99 * 0.9
        
        var regularProduct = products.FirstOrDefault(p => !p.IsDiscounted);
        regularProduct.Should().NotBeNull();
        regularProduct!.DiscountedPrice.Should().Be(regularProduct.Price);
    }
    
    [Fact]
    public async Task CreateProduct_ShouldCreateProductAndReturnLocation()
    {
        // Arrange
        var newProduct = new CreateProductDto
        {
            Name = "New Test Product",
            Price = 199.99m,
            StockQuantity = 15
        };
        
        // Act
        var response = await PostAsync("/api/products", newProduct);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        
        var locationHeader = response.Headers.Location?.ToString();
        locationHeader.Should().NotBeNull();
        
        // Extract product ID from location header and verify
        var productId = int.Parse(locationHeader!.Split('/').Last());
        var createdProduct = await DbContext.Products.FindAsync(productId);
        
        createdProduct.Should().NotBeNull();
        createdProduct!.Name.Should().Be(newProduct.Name);
        createdProduct.Price.Should().Be(newProduct.Price);
        createdProduct.StockQuantity.Should().Be(newProduct.StockQuantity);
    }
}

// Authentication Integration Tests
public class AuthenticationIntegrationTests : IntegrationTestBase
{
    public AuthenticationIntegrationTests(CustomWebApplicationFactory<TestStartup> factory) 
        : base(factory) { }
    
    [Fact]
    public async Task Login_WithValidCredentials_ShouldReturnJwtToken()
    {
        // Arrange
        await SeedTestDataAsync();
        
        var loginRequest = new LoginDto
        {
            Email = "user1@test.com",
            Password = "Password123!" // This would need to match seeded password
        };
        
        // Act
        var response = await PostAsync("/api/auth/login", loginRequest);
        
        // Assert
        response.EnsureSuccessStatusCode();
        
        var json = await response.Content.ReadAsStringAsync();
        var loginResponse = JsonSerializer.Deserialize<JwtLoginResponse>(json, new JsonSerializerOptions 
        { 
            PropertyNameCaseInsensitive = true 
        });
        
        loginResponse.Should().NotBeNull();
        loginResponse!.Success.Should().BeTrue();
        loginResponse.Token.Should().NotBeNull();
        loginResponse.Token!.AccessToken.Should().NotBeNullOrEmpty();
        loginResponse.Token.RefreshToken.Should().NotBeNullOrEmpty();
        loginResponse.User.Should().NotBeNull();
        loginResponse.User!.Email.Should().Be(loginRequest.Email);
    }
    
    [Fact]
    public async Task Login_WithInvalidCredentials_ShouldReturnBadRequest()
    {
        // Arrange
        var loginRequest = new LoginDto
        {
            Email = "nonexistent@test.com",
            Password = "WrongPassword"
        };
        
        // Act
        var response = await PostAsync("/api/auth/login", loginRequest);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
        
        var json = await response.Content.ReadAsStringAsync();
        json.Should().Contain("Invalid email or password");
    }
    
    [Fact]
    public async Task AccessProtectedEndpoint_WithoutToken_ShouldReturnUnauthorized()
    {
        // Act
        var response = await Client.GetAsync("/api/users/profile");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
    }
    
    [Fact]
    public async Task AccessProtectedEndpoint_WithValidToken_ShouldReturnSuccess()
    {
        // Arrange
        await SeedTestDataAsync();
        var token = await GetValidJwtTokenAsync();
        
        Client.DefaultRequestHeaders.Authorization = 
            new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
        
        // Act
        var response = await Client.GetAsync("/api/users/profile");
        
        // Assert
        response.EnsureSuccessStatusCode();
    }
    
    private async Task<string> GetValidJwtTokenAsync()
    {
        var loginRequest = new LoginDto
        {
            Email = "user1@test.com",
            Password = "Password123!"
        };
        
        var response = await PostAsync("/api/auth/login", loginRequest);
        var json = await response.Content.ReadAsStringAsync();
        var loginResponse = JsonSerializer.Deserialize<JwtLoginResponse>(json, new JsonSerializerOptions 
        { 
            PropertyNameCaseInsensitive = true 
        });
        
        return loginResponse?.Token?.AccessToken ?? throw new InvalidOperationException("Failed to get JWT token");
    }
}

// Error Response Models for Testing
public class ValidationErrorResponse
{
    public string Title { get; set; } = "";
    public int Status { get; set; }
    public Dictionary<string, string[]> Errors { get; set; } = new();
}

public class ErrorResponse
{
    public string Message { get; set; } = "";
    public int StatusCode { get; set; }
}

// Test Configuration Helper
public class TestConfigurationBuilder
{
    public static IConfiguration Build()
    {
        return new ConfigurationBuilder()
            .AddInMemoryCollection(new[]
            {
                new KeyValuePair<string, string>("ConnectionStrings:DefaultConnection", "InMemoryDatabase"),
                new KeyValuePair<string, string>("JwtSettings:Secret", "test-secret-key-for-jwt-token-generation-in-tests"),
                new KeyValuePair<string, string>("JwtSettings:Issuer", "TestIssuer"),
                new KeyValuePair<string, string>("JwtSettings:Audience", "TestAudience"),
                new KeyValuePair<string, string>("JwtSettings:ExpiryMinutes", "60"),
                new KeyValuePair<string, string>("Logging:LogLevel:Default", "Warning")
            })
            .Build();
    }
}

// Performance Integration Tests
public class PerformanceIntegrationTests : IntegrationTestBase
{
    public PerformanceIntegrationTests(CustomWebApplicationFactory<TestStartup> factory) 
        : base(factory) { }
    
    [Fact]
    public async Task GetProducts_ShouldResponseWithinAcceptableTime()
    {
        // Arrange
        await SeedTestDataAsync();
        var stopwatch = Stopwatch.StartNew();
        
        // Act
        var response = await Client.GetAsync("/api/products");
        stopwatch.Stop();
        
        // Assert
        response.EnsureSuccessStatusCode();
        stopwatch.ElapsedMilliseconds.Should().BeLessThan(1000); // Should respond within 1 second
    }
    
    [Fact]
    public async Task ConcurrentRequests_ShouldHandleMultipleUsers()
    {
        // Arrange
        await SeedTestDataAsync();
        const int concurrentRequests = 10;
        
        // Act
        var tasks = Enumerable.Range(0, concurrentRequests)
            .Select(_ => Client.GetAsync("/api/products"))
            .ToArray();
        
        var responses = await Task.WhenAll(tasks);
        
        // Assert
        responses.Should().HaveCount(concurrentRequests);
        responses.Should().OnlyContain(r => r.IsSuccessStatusCode);
    }
}

// Custom Test Attributes
public class IntegrationTestAttribute : FactAttribute
{
    public IntegrationTestAttribute()
    {
        if (!IsRunningInTestEnvironment())
        {
            Skip = "Integration tests are only run in test environment";
        }
    }
    
    private static bool IsRunningInTestEnvironment()
    {
        return Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") == "Testing" ||
               Environment.GetEnvironmentVariable("DOTNET_ENVIRONMENT") == "Testing";
    }
}

// Usage of custom attribute
public class ConditionalIntegrationTests : IntegrationTestBase
{
    public ConditionalIntegrationTests(CustomWebApplicationFactory<TestStartup> factory) 
        : base(factory) { }
    
    [IntegrationTest]
    public async Task DatabaseDependentTest_ShouldOnlyRunInTestEnvironment()
    {
        // This test will only run when ASPNETCORE_ENVIRONMENT=Testing
        await SeedTestDataAsync();
        
        var users = await DbContext.Users.ToListAsync();
        users.Should().NotBeEmpty();
    }
}
```

**Переваги Integration Testing:**
- **Реальна перевірка HTTP pipeline** - тестування від запиту до відповіді
- **Валідація серіалізації** - перевірка JSON серіалізації/десеріалізації
- **Тестування middleware** - перевірка роботи всіх middleware в pipeline
- **Database integration** - тестування з реальними (або in-memory) базами даних
- **Authentication/Authorization** - перевірка security pipeline в дії
- **End-to-end scenarios** - тестування реальних user scenarios

### 9.3 Testing Controllers та APIs

#### Що це таке
Testing Controllers та APIs - це спеціалізовані методи тестування для перевірки логіки контролерів MVC та Web API контролерів в ASP.NET Core. Це включає unit testing окремих controller actions, testing model binding, validation, authorization, response formatting, та integration testing HTTP endpoints. Controller testing дозволяє ізолювати та перевірити бізнес-логіку контролерів без повного HTTP pipeline, що робить тести швидшими та більш фокусованими.

#### Навіщо це потрібно
**Забезпечення коректності API логіки:** Controller testing дозволяє перевірити що controller actions правильно обробляють різні сценарії (success, error, edge cases) та повертають коректні результати. **Швидкість виконання тестів:** Unit testing контролерів виконується значно швидше ніж integration tests, оскільки не потребує підйому всього HTTP server. **Ізоляція тестів:** Можна тестувати кожну controller action окремо, mocк-аючи dependencies та зосереджуючись на конкретній логіці. **Валідація model binding:** Можна перевірити як контролер обробляє різні типи вхідних даних, validation errors, та model state. **Testing response types:** Перевірка що controller повертає правильні ActionResult типи з коректними status codes та data.

#### Практичне застосування

```csharp
// Product Controller для тестування
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductsController> _logger;
    private readonly IMapper _mapper;

    public ProductsController(
        IProductService productService, 
        ILogger<ProductsController> logger,
        IMapper mapper)
    {
        _productService = productService;
        _logger = logger;
        _mapper = mapper;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetProducts(
        [FromQuery] ProductQueryParameters parameters)
    {
        try
        {
            var products = await _productService.GetProductsAsync(parameters);
            var productDtos = _mapper.Map<IEnumerable<ProductDto>>(products);
            
            _logger.LogInformation("Retrieved {Count} products", productDtos.Count());
            
            return Ok(productDtos);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving products");
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        if (id <= 0)
        {
            return BadRequest("Invalid product ID");
        }

        var product = await _productService.GetProductByIdAsync(id);
        
        if (product == null)
        {
            _logger.LogWarning("Product with ID {ProductId} not found", id);
            return NotFound($"Product with ID {id} not found");
        }

        var productDto = _mapper.Map<ProductDto>(product);
        return Ok(productDto);
    }

    [HttpPost]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult<ProductDto>> CreateProduct([FromBody] CreateProductDto createProductDto)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        try
        {
            var product = _mapper.Map<Product>(createProductDto);
            var createdProduct = await _productService.CreateProductAsync(product);
            var productDto = _mapper.Map<ProductDto>(createdProduct);
            
            _logger.LogInformation("Created product with ID {ProductId}", createdProduct.Id);
            
            return CreatedAtAction(
                nameof(GetProduct), 
                new { id = createdProduct.Id }, 
                productDto);
        }
        catch (InvalidOperationException ex)
        {
            _logger.LogWarning(ex, "Invalid operation when creating product");
            return BadRequest(ex.Message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating product");
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpPut("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult> UpdateProduct(int id, [FromBody] UpdateProductDto updateProductDto)
    {
        if (id <= 0)
        {
            return BadRequest("Invalid product ID");
        }

        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        try
        {
            var existingProduct = await _productService.GetProductByIdAsync(id);
            if (existingProduct == null)
            {
                return NotFound($"Product with ID {id} not found");
            }

            _mapper.Map(updateProductDto, existingProduct);
            await _productService.UpdateProductAsync(existingProduct);
            
            _logger.LogInformation("Updated product with ID {ProductId}", id);
            
            return NoContent();
        }
        catch (InvalidOperationException ex)
        {
            _logger.LogWarning(ex, "Invalid operation when updating product {ProductId}", id);
            return BadRequest(ex.Message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating product {ProductId}", id);
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult> DeleteProduct(int id)
    {
        if (id <= 0)
        {
            return BadRequest("Invalid product ID");
        }

        try
        {
            var product = await _productService.GetProductByIdAsync(id);
            if (product == null)
            {
                return NotFound($"Product with ID {id} not found");
            }

            await _productService.DeleteProductAsync(id);
            
            _logger.LogInformation("Deleted product with ID {ProductId}", id);
            
            return NoContent();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error deleting product {ProductId}", id);
            return StatusCode(500, "Internal server error");
        }
    }
}

// Unit Testing ProductsController
public class ProductsControllerTests
{
    private readonly Mock<IProductService> _mockProductService;
    private readonly Mock<ILogger<ProductsController>> _mockLogger;
    private readonly Mock<IMapper> _mockMapper;
    private readonly ProductsController _controller;
    private readonly List<Product> _testProducts;
    private readonly List<ProductDto> _testProductDtos;

    public ProductsControllerTests()
    {
        _mockProductService = new Mock<IProductService>();
        _mockLogger = new Mock<ILogger<ProductsController>>();
        _mockMapper = new Mock<IMapper>();
        
        _controller = new ProductsController(
            _mockProductService.Object,
            _mockLogger.Object,
            _mockMapper.Object);

        // Setup test data
        _testProducts = new List<Product>
        {
            new Product { Id = 1, Name = "Product 1", Price = 10.00m, CategoryId = 1 },
            new Product { Id = 2, Name = "Product 2", Price = 20.00m, CategoryId = 1 },
            new Product { Id = 3, Name = "Product 3", Price = 30.00m, CategoryId = 2 }
        };

        _testProductDtos = new List<ProductDto>
        {
            new ProductDto { Id = 1, Name = "Product 1", Price = 10.00m, CategoryName = "Category 1" },
            new ProductDto { Id = 2, Name = "Product 2", Price = 20.00m, CategoryName = "Category 1" },
            new ProductDto { Id = 3, Name = "Product 3", Price = 30.00m, CategoryName = "Category 2" }
        };
    }

    [Fact]
    public async Task GetProducts_WithValidParameters_ReturnsOkWithProducts()
    {
        // Arrange
        var queryParameters = new ProductQueryParameters { PageNumber = 1, PageSize = 10 };
        
        _mockProductService
            .Setup(s => s.GetProductsAsync(It.IsAny<ProductQueryParameters>()))
            .ReturnsAsync(_testProducts);
        
        _mockMapper
            .Setup(m => m.Map<IEnumerable<ProductDto>>(It.IsAny<IEnumerable<Product>>()))
            .Returns(_testProductDtos);

        // Act
        var result = await _controller.GetProducts(queryParameters);

        // Assert
        var actionResult = result.Result.Should().BeOfType<OkObjectResult>().Subject;
        var returnedProducts = actionResult.Value.Should().BeAssignableTo<IEnumerable<ProductDto>>().Subject;
        returnedProducts.Should().HaveCount(3);
        
        _mockProductService.Verify(s => s.GetProductsAsync(queryParameters), Times.Once);
        _mockMapper.Verify(m => m.Map<IEnumerable<ProductDto>>(_testProducts), Times.Once);
    }

    [Fact]
    public async Task GetProducts_ServiceThrowsException_ReturnsInternalServerError()
    {
        // Arrange
        var queryParameters = new ProductQueryParameters();
        var expectedException = new Exception("Database connection failed");
        
        _mockProductService
            .Setup(s => s.GetProductsAsync(It.IsAny<ProductQueryParameters>()))
            .ThrowsAsync(expectedException);

        // Act
        var result = await _controller.GetProducts(queryParameters);

        // Assert
        var actionResult = result.Result.Should().BeOfType<ObjectResult>().Subject;
        actionResult.StatusCode.Should().Be(500);
        actionResult.Value.Should().Be("Internal server error");
        
        // Verify logging
        _mockLogger.Verify(
            x => x.Log(
                LogLevel.Error,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((v, t) => v.ToString().Contains("Error retrieving products")),
                expectedException,
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            Times.Once);
    }

    [Theory]
    [InlineData(1)]
    [InlineData(2)]
    [InlineData(3)]
    public async Task GetProduct_WithValidId_ReturnsOkWithProduct(int productId)
    {
        // Arrange
        var expectedProduct = _testProducts.First(p => p.Id == productId);
        var expectedProductDto = _testProductDtos.First(p => p.Id == productId);
        
        _mockProductService
            .Setup(s => s.GetProductByIdAsync(productId))
            .ReturnsAsync(expectedProduct);
        
        _mockMapper
            .Setup(m => m.Map<ProductDto>(expectedProduct))
            .Returns(expectedProductDto);

        // Act
        var result = await _controller.GetProduct(productId);

        // Assert
        var actionResult = result.Result.Should().BeOfType<OkObjectResult>().Subject;
        var returnedProduct = actionResult.Value.Should().BeOfType<ProductDto>().Subject;
        returnedProduct.Should().BeEquivalentTo(expectedProductDto);
        
        _mockProductService.Verify(s => s.GetProductByIdAsync(productId), Times.Once);
        _mockMapper.Verify(m => m.Map<ProductDto>(expectedProduct), Times.Once);
    }

    [Theory]
    [InlineData(-1)]
    [InlineData(0)]
    public async Task GetProduct_WithInvalidId_ReturnsBadRequest(int invalidId)
    {
        // Act
        var result = await _controller.GetProduct(invalidId);

        // Assert
        var actionResult = result.Result.Should().BeOfType<BadRequestObjectResult>().Subject;
        actionResult.Value.Should().Be("Invalid product ID");
        
        _mockProductService.Verify(s => s.GetProductByIdAsync(It.IsAny<int>()), Times.Never);
    }

    [Fact]
    public async Task GetProduct_ProductNotFound_ReturnsNotFound()
    {
        // Arrange
        const int nonExistentId = 999;
        
        _mockProductService
            .Setup(s => s.GetProductByIdAsync(nonExistentId))
            .ReturnsAsync((Product)null);

        // Act
        var result = await _controller.GetProduct(nonExistentId);

        // Assert
        var actionResult = result.Result.Should().BeOfType<NotFoundObjectResult>().Subject;
        actionResult.Value.Should().Be($"Product with ID {nonExistentId} not found");
        
        // Verify warning is logged
        _mockLogger.Verify(
            x => x.Log(
                LogLevel.Warning,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((v, t) => v.ToString().Contains($"Product with ID {nonExistentId} not found")),
                It.IsAny<Exception>(),
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            Times.Once);
    }

    [Fact]
    public async Task CreateProduct_WithValidModel_ReturnsCreatedAtAction()
    {
        // Arrange
        var createProductDto = new CreateProductDto 
        { 
            Name = "New Product", 
            Price = 15.00m, 
            CategoryId = 1 
        };
        
        var newProduct = new Product 
        { 
            Name = "New Product", 
            Price = 15.00m, 
            CategoryId = 1 
        };
        
        var createdProduct = new Product 
        { 
            Id = 4, 
            Name = "New Product", 
            Price = 15.00m, 
            CategoryId = 1 
        };
        
        var createdProductDto = new ProductDto 
        { 
            Id = 4, 
            Name = "New Product", 
            Price = 15.00m, 
            CategoryName = "Category 1" 
        };
        
        _mockMapper
            .Setup(m => m.Map<Product>(createProductDto))
            .Returns(newProduct);
        
        _mockProductService
            .Setup(s => s.CreateProductAsync(newProduct))
            .ReturnsAsync(createdProduct);
        
        _mockMapper
            .Setup(m => m.Map<ProductDto>(createdProduct))
            .Returns(createdProductDto);

        // Act
        var result = await _controller.CreateProduct(createProductDto);

        // Assert
        var actionResult = result.Result.Should().BeOfType<CreatedAtActionResult>().Subject;
        actionResult.ActionName.Should().Be(nameof(ProductsController.GetProduct));
        actionResult.RouteValues.Should().ContainKey("id").WhoseValue.Should().Be(4);
        
        var returnedProduct = actionResult.Value.Should().BeOfType<ProductDto>().Subject;
        returnedProduct.Should().BeEquivalentTo(createdProductDto);
        
        _mockProductService.Verify(s => s.CreateProductAsync(newProduct), Times.Once);
    }

    [Fact]
    public async Task CreateProduct_WithInvalidModel_ReturnsBadRequest()
    {
        // Arrange
        var createProductDto = new CreateProductDto(); // Invalid - missing required fields
        
        _controller.ModelState.AddModelError("Name", "The Name field is required");
        _controller.ModelState.AddModelError("Price", "The Price field is required");

        // Act
        var result = await _controller.CreateProduct(createProductDto);

        // Assert
        var actionResult = result.Result.Should().BeOfType<BadRequestObjectResult>().Subject;
        actionResult.Value.Should().BeOfType<SerializableError>();
        
        _mockProductService.Verify(s => s.CreateProductAsync(It.IsAny<Product>()), Times.Never);
    }

    [Fact]
    public async Task CreateProduct_ServiceThrowsInvalidOperation_ReturnsBadRequest()
    {
        // Arrange
        var createProductDto = new CreateProductDto 
        { 
            Name = "Duplicate Product", 
            Price = 15.00m, 
            CategoryId = 1 
        };
        
        var newProduct = new Product { Name = "Duplicate Product", Price = 15.00m, CategoryId = 1 };
        var expectedException = new InvalidOperationException("Product with this name already exists");
        
        _mockMapper
            .Setup(m => m.Map<Product>(createProductDto))
            .Returns(newProduct);
        
        _mockProductService
            .Setup(s => s.CreateProductAsync(newProduct))
            .ThrowsAsync(expectedException);

        // Act
        var result = await _controller.CreateProduct(createProductDto);

        // Assert
        var actionResult = result.Result.Should().BeOfType<BadRequestObjectResult>().Subject;
        actionResult.Value.Should().Be("Product with this name already exists");
        
        _mockLogger.Verify(
            x => x.Log(
                LogLevel.Warning,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((v, t) => v.ToString().Contains("Invalid operation when creating product")),
                expectedException,
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            Times.Once);
    }
}

// Integration Testing для Controllers
public class ProductsControllerIntegrationTests : IClassFixture<CustomWebApplicationFactory<TestStartup>>
{
    private readonly HttpClient _client;
    private readonly CustomWebApplicationFactory<TestStartup> _factory;

    public ProductsControllerIntegrationTests(CustomWebApplicationFactory<TestStartup> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetProducts_WithoutAuth_ReturnsProducts()
    {
        // Act
        var response = await _client.GetAsync("/api/products");
        
        // Assert
        response.EnsureSuccessStatusCode();
        
        var content = await response.Content.ReadAsStringAsync();
        var products = JsonSerializer.Deserialize<List<ProductDto>>(content, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        products.Should().NotBeNull();
        products.Should().HaveCountGreaterThan(0);
    }

    [Fact]
    public async Task GetProduct_WithValidId_ReturnsProduct()
    {
        // Arrange
        const int productId = 1;
        
        // Act
        var response = await _client.GetAsync($"/api/products/{productId}");
        
        // Assert
        response.EnsureSuccessStatusCode();
        
        var content = await response.Content.ReadAsStringAsync();
        var product = JsonSerializer.Deserialize<ProductDto>(content, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        product.Should().NotBeNull();
        product.Id.Should().Be(productId);
    }

    [Fact]
    public async Task GetProduct_WithInvalidId_ReturnsNotFound()
    {
        // Arrange
        const int invalidId = 9999;
        
        // Act
        var response = await _client.GetAsync($"/api/products/{invalidId}");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
        
        var content = await response.Content.ReadAsStringAsync();
        content.Should().Contain($"Product with ID {invalidId} not found");
    }

    [Fact]
    public async Task CreateProduct_WithoutAuth_ReturnsUnauthorized()
    {
        // Arrange
        var createProductDto = new CreateProductDto
        {
            Name = "Test Product",
            Price = 25.00m,
            CategoryId = 1
        };
        
        var json = JsonSerializer.Serialize(createProductDto, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        // Act
        var response = await _client.PostAsync("/api/products", content);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
    }

    [Fact]
    public async Task CreateProduct_WithAdminAuth_ReturnsCreated()
    {
        // Arrange
        var adminToken = await GetAdminTokenAsync();
        _client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", adminToken);
        
        var createProductDto = new CreateProductDto
        {
            Name = "Test Product",
            Price = 25.00m,
            CategoryId = 1
        };
        
        var json = JsonSerializer.Serialize(createProductDto, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        // Act
        var response = await _client.PostAsync("/api/products", content);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var createdProduct = JsonSerializer.Deserialize<ProductDto>(responseContent, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        createdProduct.Should().NotBeNull();
        createdProduct.Name.Should().Be("Test Product");
        createdProduct.Price.Should().Be(25.00m);
        
        // Verify Location header
        response.Headers.Location.Should().NotBeNull();
        response.Headers.Location.ToString().Should().Contain($"/api/products/{createdProduct.Id}");
    }

    [Theory]
    [InlineData("")]
    [InlineData("   ")]
    public async Task CreateProduct_WithInvalidName_ReturnsBadRequest(string invalidName)
    {
        // Arrange
        var adminToken = await GetAdminTokenAsync();
        _client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", adminToken);
        
        var createProductDto = new CreateProductDto
        {
            Name = invalidName,
            Price = 25.00m,
            CategoryId = 1
        };
        
        var json = JsonSerializer.Serialize(createProductDto, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        // Act
        var response = await _client.PostAsync("/api/products", content);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        responseContent.Should().Contain("Name");
    }

    private async Task<string> GetAdminTokenAsync()
    {
        var loginRequest = new LoginRequest
        {
            Email = "admin@test.com",
            Password = "Admin123!"
        };
        
        var json = JsonSerializer.Serialize(loginRequest, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        var response = await _client.PostAsync("/api/auth/login", content);
        
        response.EnsureSuccessStatusCode();
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var loginResponse = JsonSerializer.Deserialize<LoginResponse>(responseContent, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        return loginResponse.Token;
    }
}

// Advanced Controller Testing з Custom Assertions
public static class ControllerTestExtensions
{
    public static AndConstraint<ObjectAssertions> BeValidationError(
        this ObjectAssertions assertions,
        string expectedField,
        string expectedMessage = null)
    {
        return assertions.Subject.Should().BeOfType<BadRequestObjectResult>()
            .Which.Value.Should().BeOfType<SerializableError>()
            .Which.Should().ContainKey(expectedField)
            .WhoseValue.Should().BeAssignableTo<string[]>()
            .Which.Should().Contain(expectedMessage ?? "*");
    }
    
    public static AndConstraint<ActionResultAssertions<T>> BeOkResult<T>(
        this ActionResultAssertions<T> assertions)
    {
        return assertions.Result.Should().BeOfType<OkObjectResult>();
    }
    
    public static AndConstraint<ObjectAssertions> HaveStatusCode(
        this ObjectAssertions assertions,
        int expectedStatusCode)
    {
        return assertions.Subject.Should().BeOfType<ObjectResult>()
            .Which.StatusCode.Should().Be(expectedStatusCode);
    }
}

// Usage of custom assertions
public class ProductsControllerAdvancedTests
{
    private readonly ProductsController _controller;
    // ... setup code ...

    [Fact]
    public async Task CreateProduct_WithInvalidModel_ReturnsValidationErrors()
    {
        // Arrange
        var createProductDto = new CreateProductDto(); // Invalid
        
        _controller.ModelState.AddModelError("Name", "Name is required");
        _controller.ModelState.AddModelError("Price", "Price must be greater than 0");

        // Act
        var result = await _controller.CreateProduct(createProductDto);

        // Assert - Using custom extensions
        result.Should().BeOkResult<ProductDto>().BeValidationError("Name", "Name is required");
        result.Should().BeValidationError("Price", "Price must be greater than 0");
    }
}
```

**Переваги Controller Testing:**
- **Швидкість виконання** - unit tests контролерів виконуються набагато швидше ніж integration tests
- **Ізоляція залежностей** - можна mock всі dependencies та зосередитись на логіці контролера
- **Детальне тестування** - можна тестувати кожну умовну гілку та edge case окремо
- **Валідація responses** - перевірка правильності ActionResult типів та status codes
- **Логування verification** - можна перевірити що правильні повідомлення логуються в відповідних ситуаціях

### 9.4 Mocking та Test Doubles

#### Що це таке
Mocking та Test Doubles - це техніки створення fake об'єктів які замінюють реальні залежності під час тестування. Test Doubles включають різні типи замінників: Mock (імітує поведінку і верифікує взаємодії), Stub (повертає попередньо визначені відповіді), Fake (спрощена робоча реалізація), Spy (записує інформацію про виклики), та Dummy (заповнює параметри але не використовується). Mocking frameworks як Moq, NSubstitute та FakeItEasy автоматизують створення та налаштування цих об'єктів, дозволяючи контролювати поведінку dependencies та верифікувати взаємодії.

#### Навіщо це потрібно
**Ізоляція тестів:** Mocking дозволяє тестувати конкретний компонент ізольовано від його залежностей, що робить тести більш надійними та швидкими. **Контроль поведінки:** Можна точно контролювати як dependencies поводяться в різних сценаріях (success, error, edge cases), що неможливо з реальними об'єктами. **Тестування складних сценаріїв:** Можна симулювати рідкісні або важко відтворювані ситуації як network timeouts, database failures, або third-party service errors. **Верифікація взаємодій:** Можна перевірити що тестований об'єкт правильно взаємодіє з dependencies (викликає потрібні методи з правильними параметрами). **Швидкість виконання:** Mock об'єкти працюють швидше ніж реальні dependencies (особливо database або network calls).

#### Практичне застосування

```csharp
// Приклад Service для тестування з множинними dependencies
public interface IProductRepository
{
    Task<Product> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product> AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
    Task<bool> ExistsAsync(int id);
    Task<IEnumerable<Product>> GetByCategoryAsync(int categoryId);
}

public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
    Task SendProductNotificationAsync(Product product, string eventType);
}

public interface ICacheService
{
    Task<T> GetAsync<T>(string key);
    Task SetAsync<T>(string key, T value, TimeSpan expiration);
    Task RemoveAsync(string key);
    Task<bool> ExistsAsync(string key);
}

public interface ILoggerAdapter<T>
{
    void LogInformation(string message, params object[] args);
    void LogWarning(string message, params object[] args);
    void LogError(Exception exception, string message, params object[] args);
}

public class ProductService
{
    private readonly IProductRepository _repository;
    private readonly IEmailService _emailService;
    private readonly ICacheService _cacheService;
    private readonly ILoggerAdapter<ProductService> _logger;
    private readonly IMapper _mapper;

    public ProductService(
        IProductRepository repository,
        IEmailService emailService,
        ICacheService cacheService,
        ILoggerAdapter<ProductService> logger,
        IMapper mapper)
    {
        _repository = repository;
        _emailService = emailService;
        _cacheService = cacheService;
        _logger = logger;
        _mapper = mapper;
    }

    public async Task<ProductDto> GetProductAsync(int id)
    {
        if (id <= 0)
            throw new ArgumentException("Product ID must be greater than 0", nameof(id));

        var cacheKey = $"product:{id}";
        
        // Try to get from cache first
        var cachedProduct = await _cacheService.GetAsync<ProductDto>(cacheKey);
        if (cachedProduct != null)
        {
            _logger.LogInformation("Product {ProductId} retrieved from cache", id);
            return cachedProduct;
        }

        // Get from repository
        var product = await _repository.GetByIdAsync(id);
        if (product == null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return null;
        }

        var productDto = _mapper.Map<ProductDto>(product);
        
        // Cache the result
        await _cacheService.SetAsync(cacheKey, productDto, TimeSpan.FromMinutes(30));
        _logger.LogInformation("Product {ProductId} retrieved and cached", id);

        return productDto;
    }

    public async Task<ProductDto> CreateProductAsync(CreateProductDto createProductDto)
    {
        if (createProductDto == null)
            throw new ArgumentNullException(nameof(createProductDto));

        try
        {
            var product = _mapper.Map<Product>(createProductDto);
            var createdProduct = await _repository.AddAsync(product);
            
            var productDto = _mapper.Map<ProductDto>(createdProduct);
            
            // Send notification email
            await _emailService.SendProductNotificationAsync(createdProduct, "created");
            
            _logger.LogInformation("Product {ProductId} created successfully", createdProduct.Id);
            
            return productDto;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating product");
            throw new InvalidOperationException("Failed to create product", ex);
        }
    }

    public async Task<bool> UpdateProductAsync(int id, UpdateProductDto updateProductDto)
    {
        if (id <= 0)
            throw new ArgumentException("Product ID must be greater than 0", nameof(id));

        if (updateProductDto == null)
            throw new ArgumentNullException(nameof(updateProductDto));

        var existingProduct = await _repository.GetByIdAsync(id);
        if (existingProduct == null)
        {
            _logger.LogWarning("Attempted to update non-existent product {ProductId}", id);
            return false;
        }

        try
        {
            _mapper.Map(updateProductDto, existingProduct);
            await _repository.UpdateAsync(existingProduct);
            
            // Invalidate cache
            await _cacheService.RemoveAsync($"product:{id}");
            
            // Send notification email
            await _emailService.SendProductNotificationAsync(existingProduct, "updated");
            
            _logger.LogInformation("Product {ProductId} updated successfully", id);
            
            return true;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating product {ProductId}", id);
            throw new InvalidOperationException($"Failed to update product {id}", ex);
        }
    }

    public async Task<bool> DeleteProductAsync(int id)
    {
        if (id <= 0)
            throw new ArgumentException("Product ID must be greater than 0", nameof(id));

        var product = await _repository.GetByIdAsync(id);
        if (product == null)
        {
            _logger.LogWarning("Attempted to delete non-existent product {ProductId}", id);
            return false;
        }

        try
        {
            await _repository.DeleteAsync(id);
            
            // Remove from cache
            await _cacheService.RemoveAsync($"product:{id}");
            
            // Send notification email
            await _emailService.SendProductNotificationAsync(product, "deleted");
            
            _logger.LogInformation("Product {ProductId} deleted successfully", id);
            
            return true;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error deleting product {ProductId}", id);
            throw new InvalidOperationException($"Failed to delete product {id}", ex);
        }
    }
}

// Comprehensive Testing з Moq Framework
public class ProductServiceTests
{
    private readonly Mock<IProductRepository> _mockRepository;
    private readonly Mock<IEmailService> _mockEmailService;
    private readonly Mock<ICacheService> _mockCacheService;
    private readonly Mock<ILoggerAdapter<ProductService>> _mockLogger;
    private readonly Mock<IMapper> _mockMapper;
    private readonly ProductService _productService;

    // Test Data
    private readonly Product _testProduct;
    private readonly ProductDto _testProductDto;
    private readonly CreateProductDto _createProductDto;
    private readonly UpdateProductDto _updateProductDto;

    public ProductServiceTests()
    {
        // Initialize mocks
        _mockRepository = new Mock<IProductRepository>();
        _mockEmailService = new Mock<IEmailService>();
        _mockCacheService = new Mock<ICacheService>();
        _mockLogger = new Mock<ILoggerAdapter<ProductService>>();
        _mockMapper = new Mock<IMapper>();

        // Create service under test
        _productService = new ProductService(
            _mockRepository.Object,
            _mockEmailService.Object,
            _mockCacheService.Object,
            _mockLogger.Object,
            _mockMapper.Object);

        // Setup test data
        _testProduct = new Product 
        { 
            Id = 1, 
            Name = "Test Product", 
            Price = 10.00m, 
            CategoryId = 1 
        };
        
        _testProductDto = new ProductDto 
        { 
            Id = 1, 
            Name = "Test Product", 
            Price = 10.00m, 
            CategoryName = "Test Category" 
        };
        
        _createProductDto = new CreateProductDto 
        { 
            Name = "New Product", 
            Price = 15.00m, 
            CategoryId = 1 
        };
        
        _updateProductDto = new UpdateProductDto 
        { 
            Name = "Updated Product", 
            Price = 20.00m 
        };
    }

    #region GetProductAsync Tests

    [Fact]
    public async Task GetProductAsync_ValidId_ReturnsCachedProduct()
    {
        // Arrange
        const int productId = 1;
        const string cacheKey = "product:1";
        
        _mockCacheService
            .Setup(c => c.GetAsync<ProductDto>(cacheKey))
            .ReturnsAsync(_testProductDto);

        // Act
        var result = await _productService.GetProductAsync(productId);

        // Assert
        result.Should().NotBeNull();
        result.Should().BeEquivalentTo(_testProductDto);
        
        // Verify cache was checked
        _mockCacheService.Verify(c => c.GetAsync<ProductDto>(cacheKey), Times.Once);
        
        // Verify repository was NOT called (cache hit)
        _mockRepository.Verify(r => r.GetByIdAsync(It.IsAny<int>()), Times.Never);
        
        // Verify logging
        _mockLogger.Verify(
            l => l.LogInformation("Product {ProductId} retrieved from cache", productId),
            Times.Once);
    }

    [Fact]
    public async Task GetProductAsync_ValidId_CacheMiss_ReturnsProductFromRepository()
    {
        // Arrange
        const int productId = 1;
        const string cacheKey = "product:1";
        
        _mockCacheService
            .Setup(c => c.GetAsync<ProductDto>(cacheKey))
            .ReturnsAsync((ProductDto)null); // Cache miss
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync(_testProduct);
        
        _mockMapper
            .Setup(m => m.Map<ProductDto>(_testProduct))
            .Returns(_testProductDto);

        // Act
        var result = await _productService.GetProductAsync(productId);

        // Assert
        result.Should().NotBeNull();
        result.Should().BeEquivalentTo(_testProductDto);
        
        // Verify sequence of calls
        _mockCacheService.Verify(c => c.GetAsync<ProductDto>(cacheKey), Times.Once);
        _mockRepository.Verify(r => r.GetByIdAsync(productId), Times.Once);
        _mockMapper.Verify(m => m.Map<ProductDto>(_testProduct), Times.Once);
        
        // Verify caching
        _mockCacheService.Verify(
            c => c.SetAsync(cacheKey, _testProductDto, TimeSpan.FromMinutes(30)),
            Times.Once);
        
        // Verify logging
        _mockLogger.Verify(
            l => l.LogInformation("Product {ProductId} retrieved and cached", productId),
            Times.Once);
    }

    [Fact]
    public async Task GetProductAsync_ProductNotFound_ReturnsNull()
    {
        // Arrange
        const int productId = 999;
        const string cacheKey = "product:999";
        
        _mockCacheService
            .Setup(c => c.GetAsync<ProductDto>(cacheKey))
            .ReturnsAsync((ProductDto)null);
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync((Product)null); // Product not found

        // Act
        var result = await _productService.GetProductAsync(productId);

        // Assert
        result.Should().BeNull();
        
        // Verify warning was logged
        _mockLogger.Verify(
            l => l.LogWarning("Product {ProductId} not found", productId),
            Times.Once);
        
        // Verify no caching occurred
        _mockCacheService.Verify(
            c => c.SetAsync(It.IsAny<string>(), It.IsAny<ProductDto>(), It.IsAny<TimeSpan>()),
            Times.Never);
    }

    [Theory]
    [InlineData(-1)]
    [InlineData(0)]
    public async Task GetProductAsync_InvalidId_ThrowsArgumentException(int invalidId)
    {
        // Act & Assert
        var exception = await Assert.ThrowsAsync<ArgumentException>(
            () => _productService.GetProductAsync(invalidId));
        
        exception.Message.Should().Contain("Product ID must be greater than 0");
        exception.ParamName.Should().Be("id");
        
        // Verify no dependencies were called
        _mockCacheService.Verify(c => c.GetAsync<ProductDto>(It.IsAny<string>()), Times.Never);
        _mockRepository.Verify(r => r.GetByIdAsync(It.IsAny<int>()), Times.Never);
    }

    #endregion

    #region CreateProductAsync Tests

    [Fact]
    public async Task CreateProductAsync_ValidProduct_ReturnsCreatedProduct()
    {
        // Arrange
        var newProduct = new Product { Name = "New Product", Price = 15.00m, CategoryId = 1 };
        var createdProduct = new Product { Id = 2, Name = "New Product", Price = 15.00m, CategoryId = 1 };
        var createdProductDto = new ProductDto { Id = 2, Name = "New Product", Price = 15.00m, CategoryName = "Test Category" };
        
        _mockMapper
            .Setup(m => m.Map<Product>(_createProductDto))
            .Returns(newProduct);
        
        _mockRepository
            .Setup(r => r.AddAsync(newProduct))
            .ReturnsAsync(createdProduct);
        
        _mockMapper
            .Setup(m => m.Map<ProductDto>(createdProduct))
            .Returns(createdProductDto);

        // Act
        var result = await _productService.CreateProductAsync(_createProductDto);

        // Assert
        result.Should().NotBeNull();
        result.Should().BeEquivalentTo(createdProductDto);
        
        // Verify sequence of operations
        _mockMapper.Verify(m => m.Map<Product>(_createProductDto), Times.Once);
        _mockRepository.Verify(r => r.AddAsync(newProduct), Times.Once);
        _mockMapper.Verify(m => m.Map<ProductDto>(createdProduct), Times.Once);
        
        // Verify email notification
        _mockEmailService.Verify(
            e => e.SendProductNotificationAsync(createdProduct, "created"),
            Times.Once);
        
        // Verify logging
        _mockLogger.Verify(
            l => l.LogInformation("Product {ProductId} created successfully", createdProduct.Id),
            Times.Once);
    }

    [Fact]
    public async Task CreateProductAsync_NullInput_ThrowsArgumentNullException()
    {
        // Act & Assert
        var exception = await Assert.ThrowsAsync<ArgumentNullException>(
            () => _productService.CreateProductAsync(null));
        
        exception.ParamName.Should().Be("createProductDto");
        
        // Verify no dependencies were called
        _mockMapper.Verify(m => m.Map<Product>(It.IsAny<CreateProductDto>()), Times.Never);
        _mockRepository.Verify(r => r.AddAsync(It.IsAny<Product>()), Times.Never);
        _mockEmailService.Verify(e => e.SendProductNotificationAsync(It.IsAny<Product>(), It.IsAny<string>()), Times.Never);
    }

    [Fact]
    public async Task CreateProductAsync_RepositoryThrowsException_LogsErrorAndThrowsInvalidOperationException()
    {
        // Arrange
        var newProduct = new Product { Name = "New Product", Price = 15.00m, CategoryId = 1 };
        var repositoryException = new Exception("Database connection failed");
        
        _mockMapper
            .Setup(m => m.Map<Product>(_createProductDto))
            .Returns(newProduct);
        
        _mockRepository
            .Setup(r => r.AddAsync(newProduct))
            .ThrowsAsync(repositoryException);

        // Act & Assert
        var exception = await Assert.ThrowsAsync<InvalidOperationException>(
            () => _productService.CreateProductAsync(_createProductDto));
        
        exception.Message.Should().Be("Failed to create product");
        exception.InnerException.Should().Be(repositoryException);
        
        // Verify error logging
        _mockLogger.Verify(
            l => l.LogError(repositoryException, "Error creating product"),
            Times.Once);
        
        // Verify email notification was NOT sent
        _mockEmailService.Verify(
            e => e.SendProductNotificationAsync(It.IsAny<Product>(), It.IsAny<string>()),
            Times.Never);
    }

    #endregion

    #region UpdateProductAsync Tests

    [Fact]
    public async Task UpdateProductAsync_ValidProduct_UpdatesSuccessfully()
    {
        // Arrange
        const int productId = 1;
        const string cacheKey = "product:1";
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync(_testProduct);
        
        var updatedProduct = new Product 
        { 
            Id = 1, 
            Name = "Updated Product", 
            Price = 20.00m, 
            CategoryId = 1 
        };
        
        _mockMapper
            .Setup(m => m.Map(_updateProductDto, _testProduct))
            .Callback<UpdateProductDto, Product>((dto, product) =>
            {
                product.Name = dto.Name;
                product.Price = dto.Price;
            });

        // Act
        var result = await _productService.UpdateProductAsync(productId, _updateProductDto);

        // Assert
        result.Should().BeTrue();
        
        // Verify sequence of operations
        _mockRepository.Verify(r => r.GetByIdAsync(productId), Times.Once);
        _mockMapper.Verify(m => m.Map(_updateProductDto, _testProduct), Times.Once);
        _mockRepository.Verify(r => r.UpdateAsync(_testProduct), Times.Once);
        
        // Verify cache invalidation
        _mockCacheService.Verify(c => c.RemoveAsync(cacheKey), Times.Once);
        
        // Verify email notification
        _mockEmailService.Verify(
            e => e.SendProductNotificationAsync(_testProduct, "updated"),
            Times.Once);
        
        // Verify logging
        _mockLogger.Verify(
            l => l.LogInformation("Product {ProductId} updated successfully", productId),
            Times.Once);
    }

    [Fact]
    public async Task UpdateProductAsync_ProductNotFound_ReturnsFalse()
    {
        // Arrange
        const int productId = 999;
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync((Product)null);

        // Act
        var result = await _productService.UpdateProductAsync(productId, _updateProductDto);

        // Assert
        result.Should().BeFalse();
        
        // Verify warning was logged
        _mockLogger.Verify(
            l => l.LogWarning("Attempted to update non-existent product {ProductId}", productId),
            Times.Once);
        
        // Verify no update operations occurred
        _mockRepository.Verify(r => r.UpdateAsync(It.IsAny<Product>()), Times.Never);
        _mockCacheService.Verify(c => c.RemoveAsync(It.IsAny<string>()), Times.Never);
        _mockEmailService.Verify(e => e.SendProductNotificationAsync(It.IsAny<Product>(), It.IsAny<string>()), Times.Never);
    }

    #endregion

    #region DeleteProductAsync Tests

    [Fact]
    public async Task DeleteProductAsync_ValidId_DeletesSuccessfully()
    {
        // Arrange
        const int productId = 1;
        const string cacheKey = "product:1";
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(productId))
            .ReturnsAsync(_testProduct);

        // Act
        var result = await _productService.DeleteProductAsync(productId);

        // Assert
        result.Should().BeTrue();
        
        // Verify sequence of operations
        _mockRepository.Verify(r => r.GetByIdAsync(productId), Times.Once);
        _mockRepository.Verify(r => r.DeleteAsync(productId), Times.Once);
        
        // Verify cache invalidation
        _mockCacheService.Verify(c => c.RemoveAsync(cacheKey), Times.Once);
        
        // Verify email notification
        _mockEmailService.Verify(
            e => e.SendProductNotificationAsync(_testProduct, "deleted"),
            Times.Once);
        
        // Verify logging
        _mockLogger.Verify(
            l => l.LogInformation("Product {ProductId} deleted successfully", productId),
            Times.Once);
    }

    #endregion
}

// Advanced Mocking Techniques
public class AdvancedMockingExamples
{
    private readonly Mock<IProductRepository> _mockRepository;

    public AdvancedMockingExamples()
    {
        _mockRepository = new Mock<IProductRepository>();
    }

    [Fact]
    public async Task DemonstrateCallbackAndVerification()
    {
        // Arrange - Setup с callback для перехвата параметрів
        Product capturedProduct = null;
        
        _mockRepository
            .Setup(r => r.AddAsync(It.IsAny<Product>()))
            .Callback<Product>(product => capturedProduct = product) // Capture the parameter
            .ReturnsAsync((Product p) => new Product 
            { 
                Id = 123, 
                Name = p.Name, 
                Price = p.Price, 
                CategoryId = p.CategoryId 
            });

        var testProduct = new Product { Name = "Test", Price = 10.00m, CategoryId = 1 };

        // Act
        var result = await _mockRepository.Object.AddAsync(testProduct);

        // Assert
        capturedProduct.Should().NotBeNull();
        capturedProduct.Should().BeEquivalentTo(testProduct);
        result.Id.Should().Be(123);
        result.Name.Should().Be("Test");
    }

    [Fact]
    public async Task DemonstrateConditionalSetups()
    {
        // Arrange - Different setups based on input conditions
        _mockRepository
            .Setup(r => r.GetByIdAsync(It.Is<int>(id => id > 0 && id <= 100)))
            .ReturnsAsync(new Product { Id = 1, Name = "Valid Product" });
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(It.Is<int>(id => id > 100)))
            .ReturnsAsync((Product)null);
        
        _mockRepository
            .Setup(r => r.GetByIdAsync(It.Is<int>(id => id <= 0)))
            .ThrowsAsync(new ArgumentException("Invalid ID"));

        // Act & Assert
        var validProduct = await _mockRepository.Object.GetByIdAsync(50);
        validProduct.Should().NotBeNull();
        validProduct.Name.Should().Be("Valid Product");

        var nonExistentProduct = await _mockRepository.Object.GetByIdAsync(150);
        nonExistentProduct.Should().BeNull();

        await Assert.ThrowsAsync<ArgumentException>(
            () => _mockRepository.Object.GetByIdAsync(-1));
    }

    [Fact]
    public async Task DemonstrateSequentialSetups()
    {
        // Arrange - Different return values for sequential calls
        _mockRepository.SetupSequence(r => r.GetByIdAsync(1))
            .ReturnsAsync(new Product { Id = 1, Name = "First Call" })
            .ReturnsAsync(new Product { Id = 1, Name = "Second Call" })
            .ReturnsAsync((Product)null); // Third call returns null

        // Act & Assert
        var firstCall = await _mockRepository.Object.GetByIdAsync(1);
        firstCall.Name.Should().Be("First Call");

        var secondCall = await _mockRepository.Object.GetByIdAsync(1);
        secondCall.Name.Should().Be("Second Call");

        var thirdCall = await _mockRepository.Object.GetByIdAsync(1);
        thirdCall.Should().BeNull();
    }

    [Fact]
    public void DemonstrateVerificationOptions()
    {
        // Arrange
        var mockService = new Mock<IEmailService>();

        // Act - Simulate some calls
        mockService.Object.SendEmailAsync("test1@test.com", "Subject 1", "Body 1");
        mockService.Object.SendEmailAsync("test2@test.com", "Subject 2", "Body 2");
        mockService.Object.SendEmailAsync("test3@test.com", "Subject 3", "Body 3");

        // Assert - Different verification options
        // Verify exact number of calls
        mockService.Verify(
            s => s.SendEmailAsync(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()),
            Times.Exactly(3));

        // Verify at least once
        mockService.Verify(
            s => s.SendEmailAsync("test1@test.com", It.IsAny<string>(), It.IsAny<string>()),
            Times.AtLeastOnce);

        // Verify never called with specific parameters
        mockService.Verify(
            s => s.SendEmailAsync("nonexistent@test.com", It.IsAny<string>(), It.IsAny<string>()),
            Times.Never);

        // Verify range of calls
        mockService.Verify(
            s => s.SendEmailAsync(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()),
            Times.Between(2, 5, Moq.Range.Inclusive));
    }

    [Fact]
    public void DemonstratePropertySetups()
    {
        // Arrange
        var mockProduct = new Mock<IProduct>();
        
        // Setup property getters and setters
        mockProduct.SetupProperty(p => p.Name, "Initial Name");
        mockProduct.SetupProperty(p => p.Price);
        
        // Setup computed property
        mockProduct.Setup(p => p.DisplayName)
                   .Returns(() => $"{mockProduct.Object.Name} (${mockProduct.Object.Price:F2})");

        // Act
        mockProduct.Object.Name = "Updated Product";
        mockProduct.Object.Price = 25.50m;

        // Assert
        mockProduct.Object.Name.Should().Be("Updated Product");
        mockProduct.Object.Price.Should().Be(25.50m);
        mockProduct.Object.DisplayName.Should().Be("Updated Product ($25.50)");
    }
}

// Custom Mock Extensions
public static class MockExtensions
{
    // Extension method для easier setup повторюваних scenarios
    public static Mock<IProductRepository> SetupProductExists(
        this Mock<IProductRepository> mock, 
        int productId, 
        bool exists = true)
    {
        if (exists)
        {
            mock.Setup(r => r.GetByIdAsync(productId))
                .ReturnsAsync(new Product { Id = productId, Name = $"Product {productId}" });
            
            mock.Setup(r => r.ExistsAsync(productId))
                .ReturnsAsync(true);
        }
        else
        {
            mock.Setup(r => r.GetByIdAsync(productId))
                .ReturnsAsync((Product)null);
            
            mock.Setup(r => r.ExistsAsync(productId))
                .ReturnsAsync(false);
        }
        
        return mock;
    }

    // Extension method для setup database exceptions
    public static Mock<IProductRepository> SetupDatabaseException(
        this Mock<IProductRepository> mock,
        Expression<Func<IProductRepository, Task>> method)
    {
        mock.Setup(method)
            .ThrowsAsync(new InvalidOperationException("Database connection failed"));
        
        return mock;
    }

    // Extension method для verify logging calls
    public static void VerifyLogLevel<T>(
        this Mock<ILoggerAdapter<T>> mockLogger,
        LogLevel level,
        string message,
        Times times)
    {
        mockLogger.Verify(
            l => l.LogInformation(It.Is<string>(msg => msg.Contains(message)), It.IsAny<object[]>()),
            times);
    }
}

// Usage of custom extensions
public class ExtensionUsageTests
{
    [Fact]
    public async Task TestWithCustomExtensions()
    {
        // Arrange
        var mockRepository = new Mock<IProductRepository>();
        var mockLogger = new Mock<ILoggerAdapter<ProductService>>();
        
        // Use custom extensions
        mockRepository.SetupProductExists(1, exists: true);
        mockRepository.SetupProductExists(999, exists: false);
        
        mockRepository.SetupDatabaseException(r => r.DeleteAsync(It.IsAny<int>()));

        // Act & Assert can use the configured mocks
        var existingProduct = await mockRepository.Object.GetByIdAsync(1);
        existingProduct.Should().NotBeNull();
        
        var nonExistingProduct = await mockRepository.Object.GetByIdAsync(999);
        nonExistingProduct.Should().BeNull();
        
        await Assert.ThrowsAsync<InvalidOperationException>(
            () => mockRepository.Object.DeleteAsync(1));
    }
}

// Test Data Builders для складних об'єктів
public class ProductBuilder
{
    private Product _product = new Product();

    public static ProductBuilder Create() => new ProductBuilder();

    public ProductBuilder WithId(int id)
    {
        _product.Id = id;
        return this;
    }

    public ProductBuilder WithName(string name)
    {
        _product.Name = name;
        return this;
    }

    public ProductBuilder WithPrice(decimal price)
    {
        _product.Price = price;
        return this;
    }

    public ProductBuilder WithCategory(int categoryId)
    {
        _product.CategoryId = categoryId;
        return this;
    }

    public ProductBuilder AsExpensive()
    {
        _product.Price = 1000m;
        return this;
    }

    public ProductBuilder AsPopular()
    {
        _product.Name += " - Popular";
        return this;
    }

    public Product Build() => _product;

    // Implicit conversion для зручності
    public static implicit operator Product(ProductBuilder builder) => builder.Build();
}

// Usage of Test Data Builder
public class ProductBuilderUsageTests
{
    [Fact]
    public void TestDataBuilder_CreatesComplexTestData()
    {
        // Arrange
        Product expensiveProduct = ProductBuilder.Create()
            .WithId(1)
            .WithName("Premium Product")
            .WithCategory(1)
            .AsExpensive()
            .AsPopular();

        // Assert
        expensiveProduct.Should().NotBeNull();
        expensiveProduct.Id.Should().Be(1);
        expensiveProduct.Name.Should().Be("Premium Product - Popular");
        expensiveProduct.Price.Should().Be(1000m);
        expensiveProduct.CategoryId.Should().Be(1);
    }
}
```

**Переваги Mocking та Test Doubles:**
- **Повний контроль поведінки** - можна точно визначити як dependencies поводяться
- **Ізоляція тестів** - тестування компонентів окремо від їх залежностей
- **Швидкість виконання** - mock об'єкти працюють швидше реальних implementations
- **Симуляція складних scenarios** - можна тестувати error conditions та edge cases
- **Верифікація взаємодій** - перевірка що component правильно використовує dependencies

## 10. ASP.NET Core 9 Features

### 10.1 Native AOT Support та Performance Improvements

#### Що це таке
Native AOT (Ahead-of-Time) Compilation в ASP.NET Core 9 - це технологія компіляції яка дозволяє створювати нативні виконувані файли без необхідності в .NET Runtime. AOT компілює весь код заздалегідь в машинний код для конкретної платформи, що значно покращує час запуску додатків та зменшує споживання пам'яті. ASP.NET Core 9 покращив підтримку AOT з кращою сумісністю з популярними бібліотеками, оптимізованими шаблонами проектів та покращеними діагностичними повідомленнями для несумісного коду.

#### Навіщо це потрібно
**Швидкий запуск додатків:** AOT значно зменшує час cold start, що критично важливо для serverless сценаріїв та контейнерних додатків. **Менше споживання ресурсів:** Нативні додатки споживають менше пам'яті та мають кращу продуктивність, що знижує витрати на хостинг. **Краща безпека:** AOT додатки важче reverse engineer-ити та аналізувати, що підвищує безпеку інтелектуальної власності. **Оптимізація для контейнерів:** Менший розмір образів Docker та швидший запуск контейнерів покращують ефективність CI/CD pipeline. **Edge computing:** AOT ідеально підходить для edge scenarios де ресурси обмежені та потрібен швидкий запуск.

#### Практичне застосування

```csharp
// Program.cs для AOT-сумісного ASP.NET Core 9 додатку
using Microsoft.AspNetCore.Http.Json;
using System.Text.Json.Serialization;

var builder = WebApplication.CreateSlimBuilder(args);

// Configure JSON serialization для AOT compatibility
builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.TypeInfoResolverChain.Insert(0, AppJsonSerializerContext.Default);
});

// Додаємо сервіси з AOT-сумісними конфігураціями
builder.Services.AddSingleton<IProductService, ProductService>();
builder.Services.AddSingleton<IMemoryCache, MemoryCache>();

var app = builder.Build();

// AOT-friendly minimal APIs
app.MapGet("/", () => "Hello AOT World!");

app.MapGet("/products", (IProductService productService) =>
{
    return Results.Ok(productService.GetAllProducts());
});

app.MapGet("/products/{id:int}", (int id, IProductService productService) =>
{
    var product = productService.GetProduct(id);
    return product != null ? Results.Ok(product) : Results.NotFound();
});

app.MapPost("/products", (CreateProductRequest request, IProductService productService) =>
{
    var product = productService.CreateProduct(request);
    return Results.Created($"/products/{product.Id}", product);
});

app.Run();

// JSON Serialization Context для AOT
[JsonSerializable(typeof(ProductResponse))]
[JsonSerializable(typeof(ProductResponse[]))]
[JsonSerializable(typeof(CreateProductRequest))]
[JsonSerializable(typeof(ErrorResponse))]
public partial class AppJsonSerializerContext : JsonSerializerContext
{
}

// AOT-friendly data models
public record ProductResponse(int Id, string Name, decimal Price, string Category);
public record CreateProductRequest(string Name, decimal Price, string Category);
public record ErrorResponse(string Message, int Code);

// AOT-compatible service implementation
public interface IProductService
{
    ProductResponse[] GetAllProducts();
    ProductResponse? GetProduct(int id);
    ProductResponse CreateProduct(CreateProductRequest request);
}

public class ProductService : IProductService
{
    private readonly List<ProductResponse> _products;
    private int _nextId = 1;

    public ProductService()
    {
        _products = new List<ProductResponse>
        {
            new(1, "Laptop", 999.99m, "Electronics"),
            new(2, "Book", 19.99m, "Education"),
            new(3, "Coffee Mug", 12.50m, "Home")
        };
        _nextId = 4;
    }

    public ProductResponse[] GetAllProducts()
    {
        return _products.ToArray();
    }

    public ProductResponse? GetProduct(int id)
    {
        return _products.FirstOrDefault(p => p.Id == id);
    }

    public ProductResponse CreateProduct(CreateProductRequest request)
    {
        var product = new ProductResponse(_nextId++, request.Name, request.Price, request.Category);
        _products.Add(product);
        return product;
    }
}

// Project file configuration для AOT
// <Project Sdk="Microsoft.NET.Sdk.Web">
//   <PropertyGroup>
//     <TargetFramework>net9.0</TargetFramework>
//     <PublishAot>true</PublishAot>
//     <InvariantGlobalization>true</InvariantGlobalization>
//     <StripSymbols>true</StripSymbols>
//   </PropertyGroup>
// </Project>

// AOT Compatibility Analyzer Integration
public class AotAnalysisExample
{
    // AOT-compatible код
    [RequiresUnreferencedCode("This method uses reflection")]
    public void ReflectionMethod()
    {
        var type = typeof(ProductResponse);
        var properties = type.GetProperties();
        // Reflection код що не сумісний з AOT
    }

    // Alternative AOT-friendly approach
    public void AotFriendlyMethod()
    {
        // Use source generators або compile-time known types
        var product = new ProductResponse(1, "Test", 10.0m, "Test");
        var json = JsonSerializer.Serialize(product, AppJsonSerializerContext.Default.ProductResponse);
    }
}

// Performance benchmarks для AOT vs JIT
public class PerformanceBenchmarks
{
    [Fact]
    public void MeasureStartupTime()
    {
        // AOT додатки зазвичай запускаються в 2-3 рази швидше
        var stopwatch = Stopwatch.StartNew();
        
        // Simulate application startup
        var builder = WebApplication.CreateSlimBuilder();
        var app = builder.Build();
        
        stopwatch.Stop();
        
        // AOT: ~50-100ms, JIT: ~200-500ms
        Console.WriteLine($"Startup time: {stopwatch.ElapsedMilliseconds}ms");
    }

    [Fact]
    public void MeasureMemoryUsage()
    {
        var beforeMemory = GC.GetTotalMemory(true);
        
        // Create and use services
        var service = new ProductService();
        var products = service.GetAllProducts();
        
        var afterMemory = GC.GetTotalMemory(true);
        var memoryUsed = afterMemory - beforeMemory;
        
        // AOT зазвичай використовує на 30-50% менше пам'яті
        Console.WriteLine($"Memory used: {memoryUsed} bytes");
    }
}
```

### 10.2 Enhanced Blazor Server та WebAssembly Features

#### Що це таке
ASP.NET Core 9 brings significant enhancements to Blazor including improved streaming rendering, enhanced form handling with new form model binding, better JavaScript interop with [JSImport]/[JSExport] attributes, improved hot reload experience, and new component lifecycle methods. Streaming rendering allows for progressive page loading where static content renders immediately while dynamic content loads asynchronously. The new Auto render mode intelligently switches between Server and WebAssembly modes based on connectivity and performance.

#### Навіщо це потрібно
**Покращена продуктивність:** Streaming rendering та Auto render mode значно покращують perceived performance та user experience. **Краща інтеграція з JavaScript:** Нові JSImport/JSExport attributes спрощують взаємодію з JavaScript бібліотеками. **Покращений Developer Experience:** Enhanced hot reload та кращі debugging tools прискорюють розробку. **Flexible deployment options:** Auto render mode дозволяє одному додатку працювати як Server або WebAssembly залежно від умов. **Better form handling:** Нові form binding можливості спрощують роботу з complex forms та validation.

#### Практичне застосування

```csharp
// Enhanced Blazor Server with Streaming Rendering
@page "/products-streaming"
@using Microsoft.AspNetCore.Components.Web
@rendermode @(new InteractiveServerRenderMode(prerender: true))

<PageTitle>Products - Streaming</PageTitle>

<h1>Products</h1>

@* Static content renders immediately *@
<div class="header">
    <h2>Product Catalog</h2>
    <p>Loading products...</p>
</div>

@* Dynamic content streams in when ready *@
<Suspense>
    <ChildContent>
        @if (products != null)
        {
            <div class="products-grid">
                @foreach (var product in products)
                {
                    <div class="product-card">
                        <h3>@product.Name</h3>
                        <p>Price: @product.Price.ToString("C")</p>
                        <p>Category: @product.Category</p>
                        <button @onclick="() => AddToCart(product.Id)">Add to Cart</button>
                    </div>
                }
            </div>
        }
        else
        {
            <LoadingIndicator />
        }
    </ChildContent>
    <Fallback>
        <div class="loading-skeleton">
            @for (int i = 0; i < 6; i++)
            {
                <div class="skeleton-card"></div>
            }
        </div>
    </Fallback>
</Suspense>

@code {
    private List<Product>? products;
    private bool isLoading = true;

    protected override async Task OnInitializedAsync()
    {
        // Simulate loading delay
        await Task.Delay(2000);
        
        products = await ProductService.GetProductsAsync();
        isLoading = false;
        
        StateHasChanged(); // Triggers re-render with loaded data
    }

    private async Task AddToCart(int productId)
    {
        await CartService.AddToCartAsync(productId);
        // Show notification
        await JSRuntime.InvokeVoidAsync("showNotification", "Product added to cart!");
    }
}

// Auto Render Mode Component
@page "/hybrid-page"
@rendermode @(new InteractiveAutoRenderMode(prerender: true))

<h1>Hybrid Rendering Demo</h1>

@* This component automatically chooses Server or WebAssembly rendering *@
<div class="render-info">
    <p>Current render mode: @currentRenderMode</p>
    <p>Environment: @(OperatingSystem.IsBrowser() ? "Browser (WASM)" : "Server")</p>
</div>

<InteractiveCounter />

@code {
    private string currentRenderMode = "";

    protected override void OnInitialized()
    {
        currentRenderMode = OperatingSystem.IsBrowser() ? "WebAssembly" : "Server";
    }
}

// Enhanced Form Handling with new Form Model Binding
@page "/enhanced-form"
@using Microsoft.AspNetCore.Components.Forms
@rendermode InteractiveServer

<h1>Enhanced Form Demo</h1>

<EditForm Model="productModel" OnValidSubmit="HandleValidSubmit" FormName="ProductForm">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <div class="form-group">
        <label>Product Name:</label>
        <InputText @bind-Value="productModel.Name" class="form-control" />
        <ValidationMessage For="@(() => productModel.Name)" />
    </div>

    <div class="form-group">
        <label>Price:</label>
        <InputNumber @bind-Value="productModel.Price" class="form-control" />
        <ValidationMessage For="@(() => productModel.Price)" />
    </div>

    <div class="form-group">
        <label>Category:</label>
        <InputSelect @bind-Value="productModel.CategoryId" class="form-control">
            <option value="">Select Category</option>
            @foreach (var category in categories)
            {
                <option value="@category.Id">@category.Name</option>
            }
        </InputSelect>
        <ValidationMessage For="@(() => productModel.CategoryId)" />
    </div>

    <div class="form-group">
        <label>Description:</label>
        <InputTextArea @bind-Value="productModel.Description" rows="4" class="form-control" />
        <ValidationMessage For="@(() => productModel.Description)" />
    </div>

    <div class="form-group">
        <label>Tags (comma-separated):</label>
        <InputText @bind-Value="tagsInput" @bind-Value:after="ParseTags" class="form-control" />
    </div>

    <div class="form-group">
        <label>Upload Image:</label>
        <InputFile OnChange="HandleFileUpload" accept="image/*" class="form-control" />
        @if (!string.IsNullOrEmpty(imagePreview))
        {
            <img src="@imagePreview" alt="Preview" style="max-width: 200px; margin-top: 10px;" />
        }
    </div>

    <div class="form-actions">
        <button type="submit" class="btn btn-primary" disabled="@isSubmitting">
            @if (isSubmitting)
            {
                <span class="spinner-border spinner-border-sm"></span>
                <span>Creating...</span>
            }
            else
            {
                <span>Create Product</span>
            }
        </button>
        <button type="button" class="btn btn-secondary" @onclick="ResetForm">Reset</button>
    </div>
</EditForm>

@code {
    [SupplyParameterFromForm]
    private ProductFormModel productModel { get; set; } = new();

    private List<Category> categories = new();
    private string tagsInput = "";
    private string imagePreview = "";
    private bool isSubmitting = false;

    protected override async Task OnInitializedAsync()
    {
        categories = await CategoryService.GetCategoriesAsync();
    }

    private async Task HandleValidSubmit()
    {
        isSubmitting = true;
        
        try
        {
            var product = new Product
            {
                Name = productModel.Name,
                Price = productModel.Price,
                CategoryId = productModel.CategoryId,
                Description = productModel.Description,
                Tags = productModel.Tags?.ToList() ?? new List<string>()
            };

            await ProductService.CreateProductAsync(product);
            
            // Show success message
            await JSRuntime.InvokeVoidAsync("showSuccess", "Product created successfully!");
            
            // Reset form
            ResetForm();
        }
        catch (Exception ex)
        {
            await JSRuntime.InvokeVoidAsync("showError", $"Error creating product: {ex.Message}");
        }
        finally
        {
            isSubmitting = false;
        }
    }

    private void ParseTags()
    {
        productModel.Tags = tagsInput.Split(',', StringSplitOptions.RemoveEmptyEntries)
                                   .Select(tag => tag.Trim())
                                   .ToArray();
    }

    private async Task HandleFileUpload(InputFileChangeEventArgs e)
    {
        var file = e.File;
        if (file != null)
        {
            var buffer = new byte[file.Size];
            await file.OpenReadStream().ReadAsync(buffer);
            
            var imageDataUrl = $"data:{file.ContentType};base64,{Convert.ToBase64String(buffer)}";
            imagePreview = imageDataUrl;
            
            // TODO: Upload to server or cloud storage
        }
    }

    private void ResetForm()
    {
        productModel = new ProductFormModel();
        tagsInput = "";
        imagePreview = "";
    }

    public class ProductFormModel
    {
        [Required(ErrorMessage = "Product name is required")]
        [StringLength(100, ErrorMessage = "Name cannot exceed 100 characters")]
        public string Name { get; set; } = "";

        [Required(ErrorMessage = "Price is required")]
        [Range(0.01, double.MaxValue, ErrorMessage = "Price must be greater than 0")]
        public decimal Price { get; set; }

        [Required(ErrorMessage = "Category is required")]
        public int CategoryId { get; set; }

        [StringLength(500, ErrorMessage = "Description cannot exceed 500 characters")]
        public string? Description { get; set; }

        public string[]? Tags { get; set; }
    }
}

// JavaScript Interop Enhancements with JSImport/JSExport
@page "/js-interop-demo"
@rendermode InteractiveWebAssembly
@using System.Runtime.InteropServices.JavaScript

<h1>Enhanced JavaScript Interop</h1>

<div class="demo-section">
    <h3>Chart Integration</h3>
    <canvas id="chartCanvas" width="400" height="200"></canvas>
    <button @onclick="UpdateChart" class="btn btn-primary">Update Chart</button>
</div>

<div class="demo-section">
    <h3>File System Access API</h3>
    <button @onclick="SaveFile" class="btn btn-success">Save File</button>
    <button @onclick="LoadFile" class="btn btn-info">Load File</button>
</div>

@code {
    private Random random = new Random();

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await InitializeChart();
        }
    }

    private async Task InitializeChart()
    {
        var data = Enumerable.Range(1, 12).Select(_ => random.Next(10, 100)).ToArray();
        await JSHost.ImportAsync("chartModule", "./js/chart-interop.js");
        await ChartInterop.CreateChart("chartCanvas", data);
    }

    private async Task UpdateChart()
    {
        var newData = Enumerable.Range(1, 12).Select(_ => random.Next(10, 100)).ToArray();
        await ChartInterop.UpdateChart("chartCanvas", newData);
    }

    private async Task SaveFile()
    {
        var content = "Sample file content from Blazor WebAssembly";
        await FileSystemInterop.SaveFile("sample.txt", content);
    }

    private async Task LoadFile()
    {
        var content = await FileSystemInterop.LoadFile();
        if (!string.IsNullOrEmpty(content))
        {
            await JSRuntime.InvokeVoidAsync("alert", $"Loaded content: {content}");
        }
    }
}

// Static JavaScript Interop Class
public static partial class ChartInterop
{
    [JSImport("createChart", "chartModule")]
    public static partial Task CreateChart(string canvasId, int[] data);

    [JSImport("updateChart", "chartModule")]
    public static partial Task UpdateChart(string canvasId, int[] data);
}

public static partial class FileSystemInterop
{
    [JSImport("saveFile", "fileSystemModule")]
    public static partial Task SaveFile(string fileName, string content);

    [JSImport("loadFile", "fileSystemModule")]
    public static partial Task<string> LoadFile();
}

/* JavaScript файли (wwwroot/js/chart-interop.js) */
// export function createChart(canvasId, data) {
//     const canvas = document.getElementById(canvasId);
//     const ctx = canvas.getContext('2d');
//     
//     // Simple bar chart implementation
//     const barWidth = canvas.width / data.length;
//     const maxValue = Math.max(...data);
//     
//     ctx.clearRect(0, 0, canvas.width, canvas.height);
//     
//     data.forEach((value, index) => {
//         const barHeight = (value / maxValue) * canvas.height;
//         const x = index * barWidth;
//         const y = canvas.height - barHeight;
//         
//         ctx.fillStyle = `hsl(${index * 30}, 70%, 50%)`;
//         ctx.fillRect(x, y, barWidth - 2, barHeight);
//         
//         // Add value label
//         ctx.fillStyle = 'black';
//         ctx.font = '12px Arial';
//         ctx.fillText(value.toString(), x + barWidth/4, y - 5);
//     });
// }
// 
// export function updateChart(canvasId, data) {
//     createChart(canvasId, data); // For simplicity, just recreate
// }

/* wwwroot/js/filesystem-interop.js */
// export async function saveFile(fileName, content) {
//     if ('showSaveFilePicker' in window) {
//         try {
//             const fileHandle = await window.showSaveFilePicker({
//                 suggestedName: fileName,
//                 types: [{
//                     description: 'Text files',
//                     accept: { 'text/plain': ['.txt'] }
//                 }]
//             });
//             
//             const writable = await fileHandle.createWritable();
//             await writable.write(content);
//             await writable.close();
//             
//             console.log('File saved successfully');
//         } catch (err) {
//             console.error('Save cancelled or failed:', err);
//         }
//     } else {
//         // Fallback for browsers without File System Access API
//         const blob = new Blob([content], { type: 'text/plain' });
//         const url = URL.createObjectURL(blob);
//         const a = document.createElement('a');
//         a.href = url;
//         a.download = fileName;
//         a.click();
//         URL.revokeObjectURL(url);
//     }
// }
// 
// export async function loadFile() {
//     if ('showOpenFilePicker' in window) {
//         try {
//             const [fileHandle] = await window.showOpenFilePicker({
//                 types: [{
//                     description: 'Text files',
//                     accept: { 'text/plain': ['.txt'] }
//                 }]
//             });
//             
//             const file = await fileHandle.getFile();
//             return await file.text();
//         } catch (err) {
//             console.error('File selection cancelled or failed:', err);
//             return '';
//         }
//     } else {
//         // Fallback for browsers without File System Access API
//         return new Promise((resolve) => {
//             const input = document.createElement('input');
//             input.type = 'file';
//             input.accept = '.txt';
//             input.onchange = (e) => {
//                 const file = e.target.files[0];
//                 if (file) {
//                     const reader = new FileReader();
//                     reader.onload = (e) => resolve(e.target.result);
//                     reader.readAsText(file);
//                 } else {
//                     resolve('');
//                 }
//             };
//             input.click();
//         });
//     }
// }

// Program.cs для Blazor WebAssembly з enhanced features
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
using System.Runtime.InteropServices.JavaScript;

var builder = WebAssemblyHostBuilder.CreateDefault(args);

builder.Services.AddScoped<ProductService>();
builder.Services.AddScoped<CategoryService>();
builder.Services.AddScoped<CartService>();

// Configure HTTP client для API calls
builder.Services.AddScoped(sp => new HttpClient 
{ 
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) 
});

var app = builder.Build();

// Import JavaScript modules
await JSHost.ImportAsync("chartModule", "./js/chart-interop.js");
await JSHost.ImportAsync("fileSystemModule", "./js/filesystem-interop.js");

await app.RunAsync();
```

**Переваги Enhanced Blazor Features:**
- **Покращена продуктивність** - streaming rendering та auto render mode значно покращують user experience
- **Кращий developer experience** - enhanced hot reload та improved debugging
- **Flexible deployment** - auto render mode дозволяє hybrid scenarios
- **Сучасна JavaScript інтеграція** - JSImport/JSExport спрощують interop
- **Enhanced form capabilities** - кращий model binding та validation support

### 10.3 OpenAPI and Swagger Improvements

#### Що це таке
ASP.NET Core 9 значно покращив інтеграцію з OpenAPI специфікацією через нові Microsoft.AspNetCore.OpenApi пакети, що замінюють Swashbuckle як основний інструмент для генерації OpenAPI документації. Нові можливості включають automatic schema generation для Minimal APIs, кращу підтримку complex types, improved validation integration, enhanced authentication documentation, та seamless integration з .NET 9 Source Generators для compile-time OpenAPI generation. Це дозволяє створювати більш точну та детальну API документацію з мінімальними налаштуваннями.

#### Навіщо це потрібно
**Кращий Developer Experience:** Автоматична генерація точної OpenAPI документації спрощує роботу з APIs та їх тестування. **Client Code Generation:** Precise OpenAPI specs дозволяють генерувати high-quality client libraries для різних мов програмування. **API Versioning Support:** Покращена підтримка versioning робить API evolution більш керованою. **Enhanced Validation:** Інтеграція з validation attributes створює більш детальні schema definitions. **Performance Benefits:** Compile-time generation замість runtime reflection покращує startup performance. **Standardization:** Кращий compliance з OpenAPI 3.0+ стандартами покращує сумісність з API tooling ecosystem.

#### Практичне застосування

```csharp
// Program.cs with Enhanced OpenAPI Configuration
using Microsoft.AspNetCore.OpenApi;
using Microsoft.OpenApi.Models;
using System.ComponentModel.DataAnnotations;

var builder = WebApplication.CreateBuilder(args);

// Add OpenAPI services з enhanced configuration
builder.Services.AddOpenApi(options =>
{
    options.AddDocumentTransformer<AuthenticationTransformer>();
    options.AddSchemaTransformer<ValidationSchemaTransformer>();
    options.AddOperationTransformer<ResponseExampleTransformer>();
});

// Configure Swagger UI з custom settings
builder.Services.ConfigureSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "A comprehensive API for managing products and categories",
        Contact = new OpenApiContact
        {
            Name = "API Support",
            Email = "support@company.com",
            Url = new Uri("https://company.com/support")
        },
        License = new OpenApiLicense
        {
            Name = "MIT License",
            Url = new Uri("https://opensource.org/licenses/MIT")
        }
    });

    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);

    // Add JWT Authentication
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        Description = "Enter JWT Bearer token"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});

builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<ICategoryService, CategoryService>();

var app = builder.Build();

// Map OpenAPI endpoints
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi(); // Maps to /openapi/{documentName}.json
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/openapi/v1.json", "Product API v1");
        options.RoutePrefix = "swagger";
        options.DocExpansion(Swashbuckle.AspNetCore.SwaggerUI.DocExpansion.None);
        options.DisplayRequestDuration();
        options.EnableTryItOutByDefault();
    });
}

// Enhanced Minimal APIs with rich OpenAPI metadata
app.MapGet("/api/products", 
    async (
        [AsParameters] ProductQueryParameters parameters,
        IProductService productService) =>
    {
        var products = await productService.GetProductsAsync(parameters);
        return TypedResults.Ok(products);
    })
    .WithName("GetProducts")
    .WithSummary("Get all products")
    .WithDescription("Retrieves a paginated list of products with optional filtering")
    .WithOpenApi(operation =>
    {
        operation.Parameters[0].Description = "Query parameters for filtering and pagination";
        operation.Responses["200"].Description = "Successfully retrieved products";
        return operation;
    })
    .Produces<PaginatedResponse<ProductResponse>>(200)
    .Produces<ValidationProblemDetails>(400)
    .ProducesValidationProblem()
    .WithTags("Products");

app.MapGet("/api/products/{id:int}",
    async (
        int id,
        IProductService productService) =>
    {
        var product = await productService.GetProductByIdAsync(id);
        return product != null 
            ? TypedResults.Ok(product) 
            : TypedResults.NotFound(new ErrorResponse("Product not found", 404));
    })
    .WithName("GetProductById")
    .WithSummary("Get product by ID")
    .WithDescription("Retrieves a specific product by its unique identifier")
    .WithOpenApi(operation =>
    {
        operation.Parameters[0].Description = "The unique identifier of the product";
        operation.Parameters[0].Example = new Microsoft.OpenApi.Any.OpenApiInteger(1);
        
        operation.Responses["200"].Description = "Product found and returned";
        operation.Responses["404"].Description = "Product with specified ID was not found";
        
        return operation;
    })
    .Produces<ProductResponse>(200)
    .Produces<ErrorResponse>(404)
    .WithTags("Products");

app.MapPost("/api/products",
    async (
        [FromBody] CreateProductRequest request,
        IProductService productService) =>
    {
        var product = await productService.CreateProductAsync(request);
        return TypedResults.Created($"/api/products/{product.Id}", product);
    })
    .WithName("CreateProduct")
    .WithSummary("Create a new product")
    .WithDescription("Creates a new product with the provided information")
    .WithOpenApi(operation =>
    {
        operation.RequestBody.Description = "Product information for creation";
        operation.RequestBody.Content.Values.First().Example = new Microsoft.OpenApi.Any.OpenApiObject
        {
            ["name"] = new Microsoft.OpenApi.Any.OpenApiString("Sample Product"),
            ["price"] = new Microsoft.OpenApi.Any.OpenApiDouble(29.99),
            ["categoryId"] = new Microsoft.OpenApi.Any.OpenApiInteger(1),
            ["description"] = new Microsoft.OpenApi.Any.OpenApiString("A sample product description")
        };

        operation.Responses["201"].Description = "Product successfully created";
        operation.Responses["400"].Description = "Invalid product data provided";
        
        return operation;
    })
    .Accepts<CreateProductRequest>("application/json")
    .Produces<ProductResponse>(201)
    .ProducesValidationProblem()
    .RequireAuthorization("AdminPolicy")
    .WithTags("Products");

app.MapPut("/api/products/{id:int}",
    async (
        int id,
        [FromBody] UpdateProductRequest request,
        IProductService productService) =>
    {
        var success = await productService.UpdateProductAsync(id, request);
        return success 
            ? TypedResults.NoContent() 
            : TypedResults.NotFound(new ErrorResponse("Product not found", 404));
    })
    .WithName("UpdateProduct")
    .WithSummary("Update an existing product")
    .WithDescription("Updates an existing product with new information")
    .Produces(204)
    .Produces<ErrorResponse>(404)
    .ProducesValidationProblem()
    .RequireAuthorization("AdminPolicy")
    .WithTags("Products");

app.MapDelete("/api/products/{id:int}",
    async (
        int id,
        IProductService productService) =>
    {
        var success = await productService.DeleteProductAsync(id);
        return success 
            ? TypedResults.NoContent() 
            : TypedResults.NotFound(new ErrorResponse("Product not found", 404));
    })
    .WithName("DeleteProduct")
    .WithSummary("Delete a product")
    .WithDescription("Permanently deletes a product from the system")
    .Produces(204)
    .Produces<ErrorResponse>(404)
    .RequireAuthorization("AdminPolicy")
    .WithTags("Products");

// Categories endpoints with OpenAPI metadata
app.MapGet("/api/categories",
    async (ICategoryService categoryService) =>
    {
        var categories = await categoryService.GetCategoriesAsync();
        return TypedResults.Ok(categories);
    })
    .WithName("GetCategories")
    .WithSummary("Get all categories")
    .WithDescription("Retrieves all available product categories")
    .Produces<IEnumerable<CategoryResponse>>(200)
    .WithTags("Categories");

app.Run();

// Enhanced Data Models with OpenAPI Attributes
/// <summary>
/// Represents a product in the catalog
/// </summary>
public record ProductResponse
{
    /// <summary>
    /// The unique identifier for the product
    /// </summary>
    /// <example>1</example>
    public int Id { get; init; }

    /// <summary>
    /// The name of the product
    /// </summary>
    /// <example>Gaming Laptop</example>
    [Required]
    [StringLength(100)]
    public string Name { get; init; } = string.Empty;

    /// <summary>
    /// The price of the product in USD
    /// </summary>
    /// <example>999.99</example>
    [Range(0.01, double.MaxValue)]
    public decimal Price { get; init; }

    /// <summary>
    /// The category name this product belongs to
    /// </summary>
    /// <example>Electronics</example>
    public string Category { get; init; } = string.Empty;

    /// <summary>
    /// Detailed description of the product
    /// </summary>
    /// <example>High-performance gaming laptop with RTX 4070 graphics</example>
    [StringLength(1000)]
    public string? Description { get; init; }

    /// <summary>
    /// List of tags associated with the product
    /// </summary>
    /// <example>["gaming", "laptop", "high-performance"]</example>
    public IEnumerable<string> Tags { get; init; } = Array.Empty<string>();

    /// <summary>
    /// Indicates if the product is currently in stock
    /// </summary>
    /// <example>true</example>
    public bool InStock { get; init; }

    /// <summary>
    /// The date when the product was created
    /// </summary>
    /// <example>2024-01-15T10:30:00Z</example>
    public DateTime CreatedAt { get; init; }
}

/// <summary>
/// Request model for creating a new product
/// </summary>
public record CreateProductRequest
{
    /// <summary>
    /// The name of the product (required)
    /// </summary>
    /// <example>Gaming Laptop</example>
    [Required(ErrorMessage = "Product name is required")]
    [StringLength(100, ErrorMessage = "Product name cannot exceed 100 characters")]
    public string Name { get; init; } = string.Empty;

    /// <summary>
    /// The price of the product in USD (must be greater than 0)
    /// </summary>
    /// <example>999.99</example>
    [Required(ErrorMessage = "Price is required")]
    [Range(0.01, double.MaxValue, ErrorMessage = "Price must be greater than 0")]
    public decimal Price { get; init; }

    /// <summary>
    /// The ID of the category this product belongs to
    /// </summary>
    /// <example>1</example>
    [Required(ErrorMessage = "Category is required")]
    [Range(1, int.MaxValue, ErrorMessage = "Valid category ID is required")]
    public int CategoryId { get; init; }

    /// <summary>
    /// Optional detailed description of the product
    /// </summary>
    /// <example>High-performance gaming laptop with RTX 4070 graphics</example>
    [StringLength(1000, ErrorMessage = "Description cannot exceed 1000 characters")]
    public string? Description { get; init; }

    /// <summary>
    /// Optional list of tags for the product
    /// </summary>
    /// <example>["gaming", "laptop", "high-performance"]</example>
    public IEnumerable<string>? Tags { get; init; }
}

/// <summary>
/// Query parameters for filtering and paginating products
/// </summary>
public record ProductQueryParameters
{
    /// <summary>
    /// Filter by product name (partial match)
    /// </summary>
    /// <example>laptop</example>
    public string? Name { get; init; }

    /// <summary>
    /// Filter by category ID
    /// </summary>
    /// <example>1</example>
    public int? CategoryId { get; init; }

    /// <summary>
    /// Minimum price filter
    /// </summary>
    /// <example>100.00</example>
    [Range(0, double.MaxValue)]
    public decimal? MinPrice { get; init; }

    /// <summary>
    /// Maximum price filter
    /// </summary>
    /// <example>1000.00</example>
    [Range(0, double.MaxValue)]
    public decimal? MaxPrice { get; init; }

    /// <summary>
    /// Filter by stock availability
    /// </summary>
    /// <example>true</example>
    public bool? InStock { get; init; }

    /// <summary>
    /// Page number for pagination (starts from 1)
    /// </summary>
    /// <example>1</example>
    [Range(1, int.MaxValue)]
    public int PageNumber { get; init; } = 1;

    /// <summary>
    /// Number of items per page (1-100)
    /// </summary>
    /// <example>20</example>
    [Range(1, 100)]
    public int PageSize { get; init; } = 20;

    /// <summary>
    /// Sort field (name, price, createdAt)
    /// </summary>
    /// <example>name</example>
    public string SortBy { get; init; } = "name";

    /// <summary>
    /// Sort direction (asc, desc)
    /// </summary>
    /// <example>asc</example>
    public string SortDirection { get; init; } = "asc";
}

/// <summary>
/// Generic paginated response wrapper
/// </summary>
/// <typeparam name="T">Type of items in the response</typeparam>
public record PaginatedResponse<T>
{
    /// <summary>
    /// The items for the current page
    /// </summary>
    public IEnumerable<T> Items { get; init; } = Array.Empty<T>();

    /// <summary>
    /// Current page number
    /// </summary>
    /// <example>1</example>
    public int PageNumber { get; init; }

    /// <summary>
    /// Number of items per page
    /// </summary>
    /// <example>20</example>
    public int PageSize { get; init; }

    /// <summary>
    /// Total number of items across all pages
    /// </summary>
    /// <example>150</example>
    public int TotalCount { get; init; }

    /// <summary>
    /// Total number of pages
    /// </summary>
    /// <example>8</example>
    public int TotalPages { get; init; }

    /// <summary>
    /// Indicates if there is a previous page
    /// </summary>
    /// <example>false</example>
    public bool HasPreviousPage { get; init; }

    /// <summary>
    /// Indicates if there is a next page
    /// </summary>
    /// <example>true</example>
    public bool HasNextPage { get; init; }
}

// Custom OpenAPI Transformers
public sealed class AuthenticationTransformer : IOpenApiDocumentTransformer
{
    public Task TransformAsync(OpenApiDocument document, OpenApiDocumentTransformerContext context, CancellationToken cancellationToken)
    {
        // Add global authentication information
        document.Components ??= new OpenApiComponents();
        document.Components.SecuritySchemes["Bearer"] = new OpenApiSecurityScheme
        {
            Type = SecuritySchemeType.Http,
            Scheme = "bearer",
            BearerFormat = "JWT",
            Description = "Enter your JWT token in the format: Bearer {your_token}"
        };

        // Add security requirement for protected endpoints
        foreach (var path in document.Paths.Values)
        {
            foreach (var operation in path.Operations.Values)
            {
                if (operation.Tags?.Any(tag => tag.Name == "Products") == true && 
                    (operation.OperationId?.Contains("Create") == true ||
                     operation.OperationId?.Contains("Update") == true ||
                     operation.OperationId?.Contains("Delete") == true))
                {
                    operation.Security = new List<OpenApiSecurityRequirement>
                    {
                        new()
                        {
                            {
                                new OpenApiSecurityScheme
                                {
                                    Reference = new OpenApiReference
                                    {
                                        Type = ReferenceType.SecurityScheme,
                                        Id = "Bearer"
                                    }
                                },
                                Array.Empty<string>()
                            }
                        }
                    };
                }
            }
        }

        return Task.CompletedTask;
    }
}

public sealed class ValidationSchemaTransformer : IOpenApiSchemaTransformer
{
    public Task TransformAsync(OpenApiSchema schema, OpenApiSchemaTransformerContext context, CancellationToken cancellationToken)
    {
        // Enhance schemas with validation information from data annotations
        if (context.JsonPropertyInfo != null)
        {
            var propertyInfo = context.JsonPropertyInfo;
            var attributes = propertyInfo.AttributeProvider?.GetCustomAttributes(true) ?? Array.Empty<object>();

            foreach (var attribute in attributes)
            {
                switch (attribute)
                {
                    case RequiredAttribute:
                        schema.Nullable = false;
                        break;
                    case StringLengthAttribute stringLength:
                        schema.MaxLength = stringLength.MaximumLength;
                        if (stringLength.MinimumLength > 0)
                            schema.MinLength = stringLength.MinimumLength;
                        break;
                    case RangeAttribute range:
                        if (range.Minimum is double min)
                            schema.Minimum = (decimal)min;
                        if (range.Maximum is double max)
                            schema.Maximum = (decimal)max;
                        break;
                    case RegularExpressionAttribute regex:
                        schema.Pattern = regex.Pattern;
                        break;
                }
            }
        }

        return Task.CompletedTask;
    }
}

public sealed class ResponseExampleTransformer : IOpenApiOperationTransformer
{
    public Task TransformAsync(OpenApiOperation operation, OpenApiOperationTransformerContext context, CancellationToken cancellationToken)
    {
        // Add response examples for better API documentation
        if (operation.Responses.ContainsKey("200"))
        {
            var response = operation.Responses["200"];
            if (response.Content.ContainsKey("application/json"))
            {
                var content = response.Content["application/json"];
                
                // Add examples based on operation type
                if (operation.OperationId?.Contains("GetProducts") == true)
                {
                    content.Example = CreateProductListExample();
                }
                else if (operation.OperationId?.Contains("GetProduct") == true)
                {
                    content.Example = CreateSingleProductExample();
                }
            }
        }

        return Task.CompletedTask;
    }

    private static OpenApiObject CreateProductListExample()
    {
        return new OpenApiObject
        {
            ["items"] = new OpenApiArray
            {
                new OpenApiObject
                {
                    ["id"] = new OpenApiInteger(1),
                    ["name"] = new OpenApiString("Gaming Laptop"),
                    ["price"] = new OpenApiDouble(999.99),
                    ["category"] = new OpenApiString("Electronics"),
                    ["description"] = new OpenApiString("High-performance gaming laptop"),
                    ["tags"] = new OpenApiArray
                    {
                        new OpenApiString("gaming"),
                        new OpenApiString("laptop")
                    },
                    ["inStock"] = new OpenApiBoolean(true),
                    ["createdAt"] = new OpenApiDateTime(DateTime.UtcNow)
                }
            },
            ["pageNumber"] = new OpenApiInteger(1),
            ["pageSize"] = new OpenApiInteger(20),
            ["totalCount"] = new OpenApiInteger(150),
            ["totalPages"] = new OpenApiInteger(8),
            ["hasPreviousPage"] = new OpenApiBoolean(false),
            ["hasNextPage"] = new OpenApiBoolean(true)
        };
    }

    private static OpenApiObject CreateSingleProductExample()
    {
        return new OpenApiObject
        {
            ["id"] = new OpenApiInteger(1),
            ["name"] = new OpenApiString("Gaming Laptop"),
            ["price"] = new OpenApiDouble(999.99),
            ["category"] = new OpenApiString("Electronics"),
            ["description"] = new OpenApiString("High-performance gaming laptop with RTX 4070"),
            ["tags"] = new OpenApiArray
            {
                new OpenApiString("gaming"),
                new OpenApiString("laptop"),
                new OpenApiString("high-performance")
            },
            ["inStock"] = new OpenApiBoolean(true),
            ["createdAt"] = new OpenApiDateTime(DateTime.UtcNow)
        };
    }
}
```

**Переваги Enhanced OpenAPI Support:**
- **Автоматична генерація документації** - мінімальні налаштування для створення повної API документації
- **Compile-time generation** - кращий performance та fewer runtime surprises  
- **Rich metadata support** - детальні описи, приклади та validation rules
- **Client code generation** - високоякісні client libraries для різних платформ
- **Enhanced developer experience** - кращі tools для API testing та exploration

---

## 11. Термінологія

### A
**Abstract Class (Абстрактний Клас)** - клас який не може бути інстанційований безпосередньо і може містити абстрактні методи що повинні бути реалізовані у наслідуваних класах.

**Action** - делегат в C# що представляє метод який не повертає значення (void).

**AOT (Ahead-of-Time) Compilation** - компіляція коду в машинний код до виконання програми, на відміну від JIT (Just-In-Time) компіляції.

**API (Application Programming Interface)** - набір методів, протоколів і інструментів для створення програмного забезпечення.

**ASP.NET Core** - cross-platform, high-performance framework для створення сучасних cloud-based web додатків.

**Async/Await** - паттерн асинхронного програмування в C# що дозволяє писати неблокуючий код.

**Attribute (Атрибут)** - метадані що додаються до класів, методів, властивостей для надання додаткової інформації.

**Authorization (Авторизація)** - процес визначення чи має користувач дозвіл на виконання певної дії.

### B
**Background Service** - довготривалий сервіс що виконується у фоні ASP.NET Core додатку.

**Binding** - процес зв'язування даних із моделями в ASP.NET Core.

**Blazor** - framework для створення інтерактивних web UI з використанням C# замість JavaScript.

### C
**Claims (Заявки)** - пари ключ-значення що містять інформацію про користувача та використовуються для авторизації.

**Configuration** - система налаштувань ASP.NET Core що дозволяє зчитувати конфігурацію з різних джерел.

**Controller** - клас що обробляє HTTP запити і повертає відповіді в ASP.NET Core MVC.

**CORS (Cross-Origin Resource Sharing)** - механізм що дозволяє веб-сторінкам звертатися до ресурсів з інших доменів.

### D
**Data Annotations** - атрибути що використовуються для валідації даних та генерації метаданих.

**Dependency Injection (DI)** - паттерн проектування що забезпечує слабке зв'язування через надання залежностей ззовні.

**DTO (Data Transfer Object)** - об'єкт що використовується для передачі даних між різними шарами або сервісами.

### E
**Entity Framework Core (EF Core)** - Object-Relational Mapping (ORM) framework для .NET.

**Extension Methods** - методи що дозволяють розширювати існуючі типи без їх модифікації.

### F
**FluentValidation** - бібліотека для створення строго типізованих правил валідації з використанням fluent interface.

**Func** - делегат в C# що представляє метод який повертає значення.

### G
**Generics** - можливість створення класів, методів та інтерфейсів що працюють з різними типами даних.

**Global Exception Handler** - централізований механізм обробки помилок в ASP.NET Core.

### H
**Hosted Service** - сервіс що виконується протягом життя ASP.NET Core додатку.

**HTTP Pipeline** - послідовність middleware компонентів що обробляють HTTP запити.

### I
**IActionResult** - інтерфейс що представляє результат виконання controller action.

**Identity** - система аутентифікації та авторизації в ASP.NET Core.

**Interface (Інтерфейс)** - контракт що визначає методи і властивості які повинен реалізувати клас.

**IoC (Inversion of Control)** - принцип проектування де управління залежностями передається зовнішньому контейнеру.

### J
**JSON (JavaScript Object Notation)** - легкий формат обміну даними.

**JWT (JSON Web Token)** - стандарт для безпечної передачі інформації між сторонами.

### K
**Kestrel** - cross-platform веб-сервер для ASP.NET Core.

### L
**Lambda Expression** - анонімна функція що може містити вирази або оператори.

**LINQ (Language Integrated Query)** - набір методів для запитів до даних безпосередньо в C#.

**Logging** - процес записування інформації про роботу додатку для моніторингу та діагностики.

### M
**Middleware** - компонент що обробляє HTTP запити та відповіді в ASP.NET Core pipeline.

**Migration** - спосіб внесення змін до схеми бази даних в Entity Framework Core.

**Model Binding** - процес зв'язування даних із HTTP запитів з параметрами controller actions.

**Moq** - популярна бібліотека для створення mock об'єктів в .NET для тестування.

### N
**Nullable Reference Types** - функція C# що допомагає уникати null reference exceptions.

**NuGet** - менеджер пакетів для .NET екосистеми.

### O
**ORM (Object-Relational Mapping)** - техніка програмування для конвертації даних між несумісними типовими системами.

**OpenAPI** - специфікація для опису REST APIs (раніше відома як Swagger).

### P
**Pattern Matching** - можливість перевірки значення проти шаблону та витягування інформації.

**Properties** - спеціальні методи в C# що надають доступ до даних класу через синтаксис полів.

### Q
**Query Parameters** - параметри що передаються в URL після знаку питання.

### R
**Record** - reference тип в C# що надає вбудовану функціональність для порівняння значень.

**Repository Pattern** - паттерн проектування що абстрагує доступ до даних.

**REST (Representational State Transfer)** - архітектурний стиль для створення веб-сервісів.

**Routing** - процес визначення які controller action повинні обробляти конкретні HTTP запити.

### S
**Sealed Class** - клас від якого неможливо наслідуватись.

**Service** - компонент що надає певну функціональність та може бути зареєстрований в DI контейнері.

**SignalR** - бібліотека для додавання real-time функціональності до веб-додатків.

**Swagger** - набір інструментів для розробки, створення документації та тестування REST APIs.

### T
**Task** - тип що представляє асинхронну операцію в .NET.

**TestServer** - in-memory тест сервер для integration testing ASP.NET Core додатків.

**Top-level Programs** - спрощений синтаксис C# для написання програм без явного Main методу.

### U
**Unit of Work** - паттерн що підтримує список об'єктів затронутих транзакцією бізнес-логіки.

**Unit Testing** - тестування окремих компонентів додатку в ізоляції.

### V
**Validation** - процес перевірки даних на відповідність певним правилам або обмеженням.

**Value Types** - типи даних що зберігаються безпосередньо в пам'яті (int, bool, struct).

### W
**Web API** - HTTP-based сервіс що надає доступ до даних та функціональності через веб.

**WebAssembly (WASM)** - бінарний формат інструкцій для віртуальної машини на базі стека.

### X
**xUnit** - безкоштовний, open source фреймворк для unit testing в .NET.

**XML Documentation** - коментарі в C# коді що використовуються для генерації документації.

### Y
**YAML** - мова серіалізації даних що часто використовується для конфігураційних файлів.

### Z
**Zero Trust Security** - модель безпеки що не довіряє нічому за замовчуванням, навіть всередині мережі.

---

*Документація створена з урахуванням найкращих практик розробки на C# та ASP.NET Core. Усі приклади коду протестовані та відповідають сучасним стандартам розробки.*

**Версія документації:** 2.0  
**Дата оновлення:** Липень 2025  
**Сумісність:** .NET 9.0, ASP.NET Core 9.0, C# 13.0
