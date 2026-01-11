## 1. ASP.NET Core - что это, где используется, какими ОС поддерживается?

**ASP.NET Core** — это кроссплатформенный фреймворк с открытым исходным кодом для создания современных веб-приложений, API и микросервисов.

**Где используется:**
- Веб-приложения и веб-сайты
- REST API для мобильных приложений
- Микросервисная архитектура
- Real-time приложения через SignalR
- Cloud-native приложения

**Поддерживаемые ОС:**
- **Windows** (Windows 7+, Windows Server 2008+)
- **Linux** (Ubuntu, Debian, CentOS, RHEL, Alpine)
- **macOS** (macOS 10.13+)

**Ключевые особенности:**
- Высокая производительность (использует Kestrel как веб-сервер)
- Встроенная поддержка Dependency Injection
- Модульная архитектура через Middleware pipeline
- Поддержка Docker контейнеризации
- Кроссплатформенная разработка и развертывание

---

## 2. Что такое и зачем нужен Web API? Приведите примеры использования.

**Web API** (Application Programming Interface) — это набор правил и протоколов, позволяющих разным программным приложениям взаимодействовать между собой через HTTP.

**Зачем нужен:**
- Разделение клиентской и серверной логики
- Возможность обслуживать разные типы клиентов (веб, мобильные, десктоп)
- Создание микросервисной архитектуры
- Интеграция с внешними системами и сервисами

**Примеры использования:**
1. **Погодное приложение** — получает данные от OpenWeatherMap API
2. **Онлайн-магазин** — использует платежный API Stripe для обработки оплаты
3. **Социальная сеть** — интегрируется с Facebook Graph API для публикации постов
4. **Картографический сервис** — использует Google Maps API для отображения карт
5. **Банковское приложение** — получает курсы валют через API Центробанка

---

## 3. В чем преимущества и недостатки многопоточного выполнения запросов?

**Преимущества:**
1. **Повышение производительности** — одновременная обработка множества запросов
2. **Лучшее использование CPU** — загрузка всех ядер процессора
3. **Отзывчивость** — UI поток не блокируется долгими операциями
4. **Масштабируемость** — можно обрабатывать больше запросов одновременно
5. **Параллелизм** — одновременное выполнение независимых задач

**Недостатки:**
1. **Сложность разработки** — труднее писать и отлаживать многопоточный код
2. **Проблемы синхронизации** — race conditions, deadlocks, livelocks
3. **Потребление памяти** — каждый поток требует памяти (~1MB под стек)
4. **Накладные расходы** — переключение контекста между потоками
5. **Сложность отладки** — недетерминированное поведение, сложно воспроизвести ошибки

---

## 4. Зачем нужен MVC?

**MVC (Model-View-Controller)** — архитектурный паттерн, который разделяет приложение на три компонента:

**Model (Модель):**
- Содержит бизнес-логику и данные
- Отвечает за валидацию и правила предметной области
- Не зависит от представления и контроллера

**View (Представление):**
- Отображает данные пользователю
- Пассивна — только показывает информацию
- Не содержит бизнес-логику

**Controller (Контроллер):**
- Обрабатывает пользовательский ввод
- Координирует взаимодействие Model и View
- Выбирает View для отображения результата

**Преимущества MVC:**
- **Разделение ответственности** — каждый компонент решает свою задачу
- **Упрощение тестирования** — можно тестировать Model отдельно от View
- **Повторное использование** — одна Model может использоваться разными View
- **Гибкость** — легко менять View без изменения бизнес-логики
- **Поддержка** — проще понимать и поддерживать код

---

## 5. Чем примечателен слой с контроллерами?

**Контроллеры** в ASP.NET Core MVC имеют следующие особенности:

**Основные функции:**
- Принимают HTTP-запросы через систему маршрутизации
- Выполняют базовую валидацию входных данных
- Вызывают сервисы бизнес-логики (не содержат ее сами)
- Формируют HTTP-ответ (View, JSON, Redirect)
- Управляют состоянием приложения в рамках запроса

**Атрибуты и аннотации:**
```csharp
[Route("api/[controller]")]          // Маршрутизация
[ApiController]                      // Указание API контроллера
[Authorize]                          // Требование авторизации
[HttpGet("{id}")]                    // Обработка GET запроса
[ValidateAntiForgeryToken]           // Защита от CSRF
```

**Внедрение зависимостей:**
```csharp
public class ProductController : Controller
{
    private readonly IProductService _productService;
    
    // DI через конструктор
    public ProductController(IProductService productService)
    {
        _productService = productService;
    }
}
```

**Возвращаемые типы:**
- `IActionResult` — абстрактный результат действия
- `ActionResult<T>` — типизированный результат для Web API
- `ViewResult` — возврат представления с моделью
- `JsonResult` — возврат данных в формате JSON
- `RedirectResult` — перенаправление на другой URL
- `FileResult` — возврат файла
- `StatusCodeResult` — возврат HTTP статус-кода

---

## 6. Какая разница между services.AddTransient и services.AddScoped?

**Разница во времени жизни зависимостей:**

### AddTransient
```csharp
services.AddTransient<IMyService, MyService>();
```
- **Создается новый экземпляр** каждый раз, когда сервис запрашивается
- **Не сохраняет состояние** между вызовами
- **Идеально для:** легковесных, stateless сервисов

**Пример использования:**
```csharp
public class EmailService : IEmailService
{
    // Каждый раз создается новый экземпляр
    public void SendEmail(string to, string message) { ... }
}
```

### AddScoped
```csharp
services.AddScoped<IMyService, MyService>();
```
- **Один экземпляр** в рамках одного Scope (обычно HTTP-запроса)
- **Сохраняет состояние** на протяжении всего запроса
- **Идеально для:** сервисов, работающих в контексте запроса

**Пример использования:**
```csharp
public class ShoppingCartService : IShoppingCartService
{
    private List<CartItem> _items = new();
    
    // Один экземпляр на весь запрос
    public void AddItem(Product product) 
    {
        _items.Add(new CartItem(product));
    }
}
```

**Практический пример:**
```csharp
public class OrderController : Controller
{
    private readonly IOrderService _orderService1;
    private readonly IOrderService _orderService2;
    
    public OrderController(IOrderService orderService1, IOrderService orderService2)
    {
        // Для Transient: orderService1 != orderService2
        // Для Scoped: orderService1 == orderService2 (в рамках одного запроса)
    }
}
```

**Когда что использовать:**
- **Transient** — для сервисов без состояния или с дешевым созданием
- **Scoped** — для сервисов, которым нужно сохранять состояние в рамках запроса
- **Singleton** — для глобальных сервисов (кэш, конфигурация)

---

## 7. Каковы особенности работы с Singleton зависимостями?

**Singleton** — служба создается один раз и существует все время жизни приложения.

**Регистрация:**
```csharp
services.AddSingleton<IMySingletonService, MySingletonService>();
```

**Особенности работы:**
### 1. Потокобезопасность

Singleton должен быть потокобезопасным, так как к нему могут обращаться несколько потоков одновременно:

```csharp
public class CounterService : ICounterService
{
    private int _count = 0;
    
    // НЕПРАВИЛЬНО - не потокобезопасно
    public void Increment() 
    {
        _count++; // Возможна потеря обновлений
    }
    
    // ПРАВИЛЬНО - потокобезопасная реализация
    private readonly object _lock = new object();
    public void IncrementSafe()
    {
        lock(_lock)
        {
            _count++;
        }
    }
    
    // ЛУЧШЕ - использование Interlocked
    public void IncrementBest()
    {
        Interlocked.Increment(ref _count);
    }
}
```

### 2. Время жизни зависимостей
Singleton не должен зависеть от Transient или Scoped служб:
```csharp
// ОШИБКА: Singleton зависит от Transient
services.AddSingleton<IMySingleton>(provider => 
    new MySingleton(provider.GetService<ITransientService>()));

// ПРАВИЛЬНО: Все зависимости тоже Singleton
services.AddSingleton<IMySingleton, MySingleton>();
services.AddSingleton<IDependency, SingletonDependency>();
```

### 3. Освобождение ресурсов
Singleton не освобождается до конца работы приложения:
```csharp
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    
    public DatabaseConnection()
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }
    
    public void Dispose()
    {
        _connection?.Close();
        _connection?.Dispose();
    }
}
```

### 4. Где использовать Singleton
**Хорошие кандидаты:**
- Сервисы кэширования
- Конфигурация приложения
- Логгеры
- Фабрики соединений (Connection Pool)
- Статические справочники

**Не подходит для:**
- Сервисы с состоянием пользователя
- Сервисы, зависящие от контекста запроса
- Сервисы с тяжелыми ресурсами, требующими освобождения

---

## 8. Передавать Transient зависимости в Singleton сервисы - хорошая идея? Обоснуйте свой ответ.
**Ответ: НЕТ, это плохая практика.**

### Почему это проблема?

**1. Нарушение жизненного цикла:**
```csharp
public class MyTransient : IDisposable
{
    public MyTransient() => Console.WriteLine("Transient created");
    public void Dispose() => Console.WriteLine("Transient disposed");
}

public class MySingleton
{
    private readonly MyTransient _transient;
    
    public MySingleton(MyTransient transient)
    {
        _transient = transient; // Захвачен навсегда!
    }
}

// Transient никогда не будет освобожден!
// Dispose() не вызовется → утечка ресурсов
```

**2. Transient становится фактически Singleton:**
- Transient служба предназначена для короткой жизни
- В Singleton она живет вечно
- Нарушается семантика Transient зависимостей

**3. Проблемы с потокобезопасностью:**
```csharp
public class NonThreadSafeTransient
{
    private int _counter;
    
    public void Increment() => _counter++;
}

public class MySingleton
{
    private readonly NonThreadSafeTransient _transient;
    
    public void DoWork()
    {
        // Несколько потоков работают с одним экземпляром!
        _transient.Increment(); // Race condition!
    }
}
```

### Решения проблемы:

**Решение 1: Изменить жизненный цикл зависимости**
```csharp
// Сделать зависимость тоже Singleton
services.AddSingleton<IDependency, MyDependency>();
services.AddSingleton<IMySingleton, MySingleton>();
```

**Решение 2: Использовать фабрику**
```csharp
public class MySingleton
{
    private readonly IServiceProvider _serviceProvider;
    
    public MySingleton(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public void DoWork()
    {
        using (var scope = _serviceProvider.CreateScope())
        {
            var transient = scope.ServiceProvider.GetService<ITransient>();
            // Используем и освобождаем
        }
    }
}
```

**Решение 3: Внедрять фабрику для создания зависимостей**
```csharp
public interface ITransientFactory
{
    ITransient Create();
}

public class MySingleton
{
    private readonly ITransientFactory _factory;
    
    public void DoWork()
    {
        var transient = _factory.Create();
        // Используем transient
    }
}
```

**Вывод:** Transient зависимости в Singleton нарушают принципы DI и могут привести к серьезным проблемам. Лучше пересмотреть архитектуру.

---

## 9. Передавать Scoped зависимости в Singleton сервисы - хорошая идея? Обоснуйте свой ответ.
**Ответ: НЕТ, это вызывает ошибку времени выполнения.**

### Проблема
**Scoped** служба существует только в пределах Scope (обычно одного HTTP-запроса).
**Singleton** существует все время жизни приложения.

```csharp
public class MyScopedService : IMyScopedService { }

public class MySingletonService : IMySingletonService
{
    // ОШИБКА ПРИ КОМПИЛЯЦИИ/ВЫПОЛНЕНИИ!
    public MySingletonService(IMyScopedService scopedService) { }
}
```

**Ошибка:**
```
System.InvalidOperationException: 
Cannot consume scoped service 'IMyScopedService' 
from singleton 'IMySingletonService'.
```

### Почему это запрещено?

**1. Scope может закончиться:**
- HTTP-запрос завершается → Scope уничтожается
- Singleton продолжает жить
- Scoped служба становится невалидной

**2. Утечка памяти:**
- Scoped служба не освобождается
- Ресурсы не очищаются
- Memory leak (утечка памяти)

**3. Непредсказуемое поведение:**
- Разные потоки получают доступ к одному экземпляру
- Состояние становится неконсистентным
- Трудновоспроизводимые ошибки

### Решения:

**Решение 1: Использовать IServiceScopeFactory**
```csharp
public class MySingletonService : IMySingletonService
{
    private readonly IServiceScopeFactory _scopeFactory;
    
    public MySingletonService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }
    
    public void DoWork()
    {
        // Создаем новый Scope
        using (var scope = _scopeFactory.CreateScope())
        {
            var scopedService = scope.ServiceProvider
                .GetRequiredService<IMyScopedService>();
            // Работаем со scopedService
        } // Scope уничтожается, служба освобождается
    }
}
```

**Решение 2: Внедрять зависимость через метод**
```csharp
public class MySingletonService : IMySingletonService
{
    public void DoWork(IMyScopedService scopedService)
    {
        // Получаем Scoped службу как параметр
        scopedService.DoSomething();
    }
}
```

**Решение 3: Изменить архитектуру**
```csharp
// Возможно, ваш Singleton должен быть Scoped?
services.AddScoped<IMyService, MyService>();

// Или Scoped служба должна быть Singleton?
services.AddSingleton<IMyService, MyService>();
```

**Решение 4: Использовать фабрику**
```csharp
public interface IScopedServiceFactory
{
    IMyScopedService Create();
}

public class ScopedServiceFactory : IScopedServiceFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public ScopedServiceFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IMyScopedService Create()
    {
        var scope = _serviceProvider.CreateScope();
        return scope.ServiceProvider.GetRequiredService<IMyScopedService>();
    }
}
```

**Важно:** Если архитектура требует Scoped зависимость в Singleton, это указывает на проблему в дизайне системы. Лучше пересмотреть архитектурные решения.

---

## 10. В чем отличие аутентификации от авторизации?
**Аутентификация** и **авторизация** — это два разных, но взаимосвязанных процесса безопасности:

### Аутентификация (Authentication)
**"Кто вы?"** — процесс проверки личности пользователя.

**Как работает:**
1. Пользователь предоставляет учетные данные
2. Система проверяет их корректность
3. Если проверка успешна — пользователь аутентифицирован

**Методы аутентификации:**
- **Логин/пароль** — классический способ
- **Токены** — JWT, OAuth, Bearer токены
- **Сертификаты** — SSL/TLS клиентские сертификаты
- **Биометрия** — отпечаток пальца, распознавание лица
- **Многофакторная** — комбинация методов

**Пример в ASP.NET Core:**
```csharp
// Настройка аутентификации
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true
        };
    });
```

### Авторизация (Authorization)
**"Что вам разрешено?"** — процесс проверки прав доступа.

**Как работает:**
1. Пользователь уже аутентифицирован
2. Система проверяет его права на выполнение действия
3. Если права есть — доступ разрешен

**Типы авторизации:**
1. **Ролевая (Role-based):**
   ```csharp
   [Authorize(Roles = "Admin,Manager")]
   public class AdminController : Controller
   ```

2. **На основе политик (Policy-based):**
   ```csharp
   services.AddAuthorization(options =>
   {
       options.AddPolicy("RequireAdmin", policy =>
           policy.RequireRole("Admin"));
       
       options.AddPolicy("MinimumAge", policy =>
           policy.RequireClaim("Age", "18"));
   });
   
   [Authorize(Policy = "RequireAdmin")]
   ```

3. **На основе утверждений (Claims-based):**
   ```csharp
   // Проверка конкретного claim
   if (User.HasClaim(c => c.Type == "IsAdmin" && c.Value == "true"))
   {
       // Разрешено
   }
   ```

4. **На основе ресурсов (Resource-based):**
   ```csharp
   public async Task<IActionResult> EditDocument(int id)
   {
       var document = await _context.Documents.FindAsync(id);
       
       // Проверка прав на конкретный документ
       var authResult = await _authorizationService
           .AuthorizeAsync(User, document, "EditPolicy");
       
       if (!authResult.Succeeded)
           return Forbid();
   }
   ```

### Наглядное сравнение
```
┌─────────────────────────────────────────────────────┐
│                ПРОЦЕСС ДОСТУПА                       │
├─────────────────────────────────────────────────────┤
│ 1. ВХОД В СИСТЕМУ:                                  │
│    Пользователь: "Я - user123, мой пароль: ***"     │
│    ↓                                                 │
│    АУТЕНТИФИКАЦИЯ: Проверка логина/пароля           │
│    Результат: "Да, это user123"                     │
│    ↓                                                 │
│ 2. ЗАПРОС ДЕЙСТВИЯ:                                 │
│    "Хочу удалить заказ №42"                         │
│    ↓                                                 │
│    АВТОРИЗАЦИЯ: Проверка роли и прав                │
│    Результат: "user123 - менеджер, можно удалять"   │
│    ↓                                                 │
│ 3. ВЫПОЛНЕНИЕ: "Заказ удален"                       │
└─────────────────────────────────────────────────────┘
```

### Ключевые отличия
| Аспект | Аутентификация | Авторизация |
|--------|---------------|-------------|
| **Цель** | Установить личность | Установить права |
| **Когда** | При входе в систему | При доступе к ресурсам |
| **Что проверяет** | Учетные данные | Роли, права, политики |
| **Результат** | Principal/Identity | Разрешение/запрет |
| **Пример** | Проверка пароля | Проверка роли "Admin" |
| **В ASP.NET Core** | AddAuthentication() | AddAuthorization() |

**Важно:** Сначала всегда выполняется аутентификация, потом — авторизация. Нельзя авторизовать неаутентифицированного пользователя.


## 11. Назовите основные принципы ООП.

**ООП (Объектно-Ориентированное Программирование)** базируется на четырех фундаментальных принципах:

1. **Инкапсуляция (Encapsulation)** — сокрытие внутренней реализации объекта и предоставление контролируемого доступа через публичный интерфейс.
    
2. **Наследование (Inheritance)** — создание новых классов на основе существующих, повторное использование кода.
    
3. **Полиморфизм (Polymorphism)** — возможность объектов с одинаковым интерфейсом иметь разную реализацию.
    
4. **Абстракция (Abstraction)** — выделение существенных характеристик объекта и игнорирование несущественных деталей.
    

**Дополнительные принципы (SOLID):**

- **S** - Single Responsibility (Единая ответственность)
    
- **O** - Open/Closed (Открыто/закрыто)
    
- **L** - Liskov Substitution (Подстановка Лисков)
    
- **I** - Interface Segregation (Разделение интерфейсов)
    
- **D** - Dependency Inversion (Инверсия зависимостей)
    

---

## 12. Что такое наследование, инкапсуляция, абстракция, полиморфизм: приведите примеры (желательно из собственного опыта). От какого класса неявно наследуются все классы в .NET? Разрешено ли множественное наследование в C#?

### Концепции ООП с примерами:

**1. Инкапсуляция:**


```csharp
public class BankAccount
{
    // Приватное поле - скрытая реализация
    private decimal _balance;
    
    // Публичное свойство с контролируемым доступом
    public decimal Balance 
    { 
        get => _balance;
        private set => _balance = value;
    }
    
    // Публичный метод для изменения состояния
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        
        Balance += amount;
        LogTransaction($"Deposited: {amount}");
    }
}```


**2. Наследование:**

```csharp

// Базовый класс
public class Vehicle
{
    public string Brand { get; set; }
    public int Year { get; set; }
    
    public virtual void Start() => Console.WriteLine("Vehicle started");
}

// Производный класс
public class Car : Vehicle
{
    public int Doors { get; set; }
    
    // Переопределение метода
    public override void Start() => Console.WriteLine("Car engine started");
}
```


**3. Абстракция:**

```csharp

// Абстрактный класс
public abstract class Employee
{
    public string Name { get; set; }
    
    // Абстрактный метод - без реализации
    public abstract decimal CalculateSalary();
    
    // Конкретный метод
    public void PrintInfo() => Console.WriteLine($"Employee: {Name}");
}

// Конкретная реализация
public class Manager : Employee
{
    public decimal Bonus { get; set; }
    
    public override decimal CalculateSalary() => 5000 + Bonus;
}
````


**4. Полиморфизм:**

```csharp
public interface IShape
{
    double CalculateArea();
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public double CalculateArea() => Math.PI * Radius * Radius;
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public double CalculateArea() => Width * Height;
}

// Полиморфное использование
List<IShape> shapes = new List<IShape>
{
    new Circle { Radius = 5 },
    new Rectangle { Width = 4, Height = 6 }
};

foreach (var shape in shapes)
{
    Console.WriteLine($"Area: {shape.CalculateArea()}");
    // Вызовется правильная реализация для каждого типа
}
```
### От какого класса неявно наследуются все классы в .NET?

**Все классы в .NET неявно наследуются от `System.Object`** (или его псевдонима `object`).

**Доказательство:**

```csharp
public class MyClass
{
    // Пустой класс
}

// Эти вызовы возможны, потому что методы унаследованы от Object
var obj = new MyClass();
Console.WriteLine(obj.ToString());    // Наследуется от Object
Console.WriteLine(obj.GetHashCode()); // Наследуется от Object
Console.WriteLine(obj.GetType());     // Наследуется от Object
```


**Важные методы System.Object:**

1. `Equals()` — сравнение объектов
    
2. `GetHashCode()` — хеш-код для коллекций
    
3. `ToString()` — строковое представление
    
4. `GetType()` — получение типа объекта
    
5. `MemberwiseClone()` — поверхностное копирование
    

### Разрешено ли множественное наследование в C#?

**Множественное наследование КЛАССОВ запрещено:**


```csharp

// ОШИБКА КОМПИЛЯЦИИ - нельзя наследовать от двух классов
public class MyClass : BaseClass1, BaseClass2
{
    // Не скомпилируется!
}
```

**НО множественное наследование ИНТЕРФЕЙСОВ разрешено:**


```csharp

public interface IReadable { string Read(); }
public interface IWritable { void Write(string content); }
public interface IDeletable { void Delete(); }

// Класс может реализовать несколько интерфейсов
public class Document : IReadable, IWritable, IDeletable
{
    private string _content;
    
    public string Read() => _content;
    public void Write(string content) => _content = content;
    public void Delete() => _content = string.Empty;
}
```


**Почему множественное наследование классов запрещено:**

1. **Проблема ромба (Diamond Problem)** — неясно, какую реализацию метода использовать
    
2. **Усложнение компилятора** — разрешение конфликтов имен
    
3. **Увеличение сложности** — труднее понимать иерархию

## 13. Что такое рекурсия?
**Рекурсия** — метод решения задачи, при котором функция вызывает саму себя. Должна иметь:
5. **Базовый случай** — условие выхода
6. **Рекурсивный шаг** — вызов себя с измененными параметрами

```csharp
int Factorial(int n)
{
    if (n <= 1) return 1;  // Базовый случай
    return n * Factorial(n - 1);  // Рекурсивный шаг
}
```
**Преимущества:** Упрощает код для рекурсивных структур (деревья).
**Недостатки:** Риск StackOverflow, меньшая производительность.

---

## 14. Что такое лямбда-выражение?
**Лямбда-выражение** — краткая форма анонимной функции.
```csharp
// Синтаксис: (параметры) => выражение
Func<int, int> square = x => x * x;
Func<int, int, int> add = (x, y) => x + y;

// Использование в LINQ
var evenNumbers = numbers.Where(n => n % 2 == 0);
```
**Особенности:** Захватывает внешние переменные (closure), используется в делегатах, LINQ, деревьях выражений.

---

## 15. Что такое параллельное программирование (многопоточность) и его назначение? Какие классы используются?
**Параллельное программирование** — одновременное выполнение нескольких операций.

**Назначение:**
- Ускорение вычислений (CPU-bound задачи)
- Отзывчивость UI (не блокировать основной поток)
- Обработка множества запросов

**Основные классы:**
- `Thread` — низкоуровневый поток
- `Task` — асинхронная операция (рекомендуется)
- `Parallel` — параллельные циклы
- `ThreadPool` — пул потоков
- `CancellationTokenSource` — отмена операций
- `ConcurrentDictionary`, `ConcurrentQueue` — потокобезопасные коллекции

```csharp
// Пример с Task
Task.Run(() => {
    // Код в отдельном потоке
});

// Параллельный цикл
Parallel.For(0, 10, i => {
    Console.WriteLine($"Processing {i}");
});
```

---

## 16. Что такое JSON?
**JSON (JavaScript Object Notation)** — текстовый формат обмена данными.

**Синтаксис:**
```json
{
  "name": "John",
  "age": 30,
  "isStudent": false,
  "courses": ["Math", "Science"],
  "address": {
    "street": "123 Main St"
  }
}
```

**В C# (System.Text.Json):**
```csharp
// Сериализация
string json = JsonSerializer.Serialize(person);

// Десериализация
Person person = JsonSerializer.Deserialize<Person>(json);
```

**Особенности:** Легковесный, человекочитаемый, поддерживается всеми языками.

---

## 17. Как вы понимаете REST?
**REST (Representational State Transfer)** — архитектурный стиль для веб-сервисов.

**Принципы:**
1. Клиент-сервер архитектура
2. Stateless (без состояния)
3. Кэширование
4. Единообразный интерфейс
5. Слоистая система

**HTTP методы в REST:**
- `GET` /api/users — получить пользователей
- `POST` /api/users — создать пользователя
- `PUT` /api/users/{id} — обновить пользователя
- `DELETE` /api/users/{id} — удалить пользователя

**Статус-коды:**
- 200 OK — успешно
- 201 Created — ресурс создан
- 400 Bad Request — ошибка клиента
- 404 Not Found — не найдено
- 500 Internal Server Error — ошибка сервера

---

## 18. Какая разница между GET и POST HTTP методами?
| Характеристика | GET | POST |
|----------------|-----|------|
| **Назначение** | Получение данных | Отправка данных |
| **Параметры** | В URL (query string) | В теле запроса |
| **Длина** | Ограничена длиной URL | Практически не ограничена |
| **Кэширование** | Да | Нет |
| **Безопасность** | Параметры видны в URL | Безопаснее |
| **Идемпотентность** | Да (повторение безопасно) | Нет (может создать дубликаты) |

**Пример:**
```csharp
// GET - параметры в URL
GET /api/users?name=John&age=30

// POST - данные в теле
POST /api/users
Content-Type: application/json
{"name": "John", "age": 30}
```

---

## 19. Что такое Exception?
**Exception (исключение)** — ошибка времени выполнения, прерывающая нормальный поток программы.

**Иерархия:**
- `System.Exception` (базовый класс)
  - `System.SystemException` (системные исключения)
    - `NullReferenceException` — обращение к null
    - `ArgumentException` — неверный аргумент
    - `IndexOutOfRangeException` — выход за границы массива
  - `ApplicationException` (пользовательские исключения)

**Свойства Exception:**
- `Message` — описание ошибки
- `StackTrace` — цепочка вызовов методов
- `InnerException` — вложенное исключение
- `Source` — источник исключения

```csharp
// Пользовательское исключение
public class InsufficientFundsException : Exception
{
    public decimal CurrentBalance { get; }
    public decimal RequiredAmount { get; }
    
    public InsufficientFundsException(decimal current, decimal required)
        : base($"Insufficient funds. Current: {current}, Required: {required}")
    {
        CurrentBalance = current;
        RequiredAmount = required;
    }
}
```

---

## 20. Для чего служат try, catch, finally? В каком случае может не выполниться блок finally?
**try-catch-finally** — обработка исключений:
- `try` — код, который может выбросить исключение
- `catch` — обработка исключений
- `finally` — код, выполняющийся всегда (очистка ресурсов)

```csharp
try
{
    // Код, который может вызвать исключение
    File.Open("file.txt", FileMode.Open);
}
catch (FileNotFoundException ex)
{
    // Обработка конкретного исключения
    Console.WriteLine($"File not found: {ex.FileName}");
}
catch (Exception ex)
{
    // Обработка всех остальных исключений
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    // Выполняется всегда (даже при исключении)
    // Закрытие файлов, соединений и т.д.
}
```

**Finally НЕ выполнится при:**
- `Environment.FailFast()` — аварийное завершение
- Необработанном `StackOverflowException`
- Бесконечном цикле в try/catch
- Принудительном завершении процесса (kill -9)

---

## 21. Что такое call stack?
**Call Stack (стек вызовов)** — структура данных LIFO, отслеживающая активные вызовы методов.

**Как работает:**
1. При вызове метода — добавляется в стек
2. При возврате из метода — удаляется из стека
3. При исключении — показывается StackTrace

**Пример:**
```csharp
void MethodA() { MethodB(); }
void MethodB() { MethodC(); }
void MethodC() { throw new Exception("Error!"); }

// StackTrace при исключении:
// at MethodC()
// at MethodB()
// at MethodA()
// at Main()
```

**Важность:**
- Понимание потока выполнения
- Отладка исключений
- Анализ рекурсии (глубина стека)

---

## 22. Что такое ASP.NET?
**ASP.NET** — платформа для создания веб-приложений на .NET Framework (только Windows).

**Компоненты:**
- **Web Forms** — событийная модель (как Windows Forms)
- **MVC** — паттерн Model-View-Controller
- **Web API** — создание RESTful сервисов
- **SignalR** — real-time коммуникация

**Отличие от ASP.NET Core:**
- Только Windows
- Не кроссплатформенный
- Медленнее
- Нет встроенного DI (требуются сторонние библиотеки)

---

## 23. Что такое CLR?
**CLR (Common Language Runtime)** — виртуальная машина .NET, выполняющая управляемый код.

**Функции:**
1. **JIT-компиляция** — преобразование IL кода в машинный
2. **Управление памятью** — Garbage Collection
3. **Безопасность типов** — проверка корректности операций
4. **Обработка исключений** — механизм try-catch-finally
5. **Управление потоками** — многопоточность

**Процесс выполнения:**
1. Компиляция C# → IL код
2. Загрузка сборки в CLR
3. JIT-компиляция IL → машинный код
4. Выполнение
5. Сборка мусора

---

## 24. Что такое сборщик мусора (Garbage Collector)?
**Garbage Collector (GC)** — автоматическое управление памятью в .NET.

**Как работает:**
1. **Mark** — определение используемых объектов
2. **Sweep** — удаление неиспользуемых объектов
3. **Compact** — дефрагментация памяти

**Поколения (Generations):**
- **Gen 0** — молодые объекты (быстрая сборка)
- **Gen 1** — пережившие одну сборку
- **Gen 2** — долгоживущие объекты (медленная сборка)

**Триггеры сборки мусора:**
1. Недостаточно памяти для нового объекта
2. Вызов `GC.Collect()` (не рекомендуется)
3. Система в режиме низкой памяти

**IDisposable для освобождения ресурсов:**
```csharp
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    
    public void Dispose()
    {
        _connection?.Close();
        _connection?.Dispose();
        GC.SuppressFinalize(this); // Отменить финализацию
    }
    
    // Использование
    using (var conn = new DatabaseConnection())
    {
        // Работа с соединением
    } // Dispose() вызывается автоматически
}
```

---

## 25. Что такое делегат?
**Делегат** — типобезопасный указатель на метод.

```csharp
// Объявление делегата
public delegate void MyDelegate(string message);

// Метод, соответствующий делегату
void PrintMessage(string msg) => Console.WriteLine(msg);

// Создание экземпляра делегата
MyDelegate handler = PrintMessage;

// Вызов
handler("Hello!"); // Выведет "Hello!"
```

**Встроенные делегаты:**
- `Action` — без возвращаемого значения
- `Func<T>` — с возвращаемым значением типа T
- `Predicate<T>` — возвращает bool (для фильтрации)

```csharp
Action<string> print = msg => Console.WriteLine(msg);
Func<int, int, int> add = (x, y) => x + y;
Predicate<int> isEven = n => n % 2 == 0;

// Multicast делегаты
Action multi = () => Console.Write("Hello");
multi += () => Console.WriteLine(" World");
multi(); // Выведет "Hello World"
```

---

## 26. Что такое LINQ и для чего используется? Приведите несколько примеров применения LINQ.
**LINQ (Language Integrated Query)** — технология для запросов к данным прямо в C#.

**Источники данных:**
- Коллекции (List, Array)
- XML документы
- Базы данных (Entity Framework)
- JSON, CSV

**Примеры:**

**1. Фильтрация (Where):**
```csharp
var adults = people.Where(p => p.Age >= 18);
var activeUsers = users.Where(u => u.IsActive);
```

**2. Проекция (Select):**
```csharp
var names = people.Select(p => p.Name);
var nameAndAge = people.Select(p => new { p.Name, p.Age });
```

**3. Сортировка (OrderBy):**
```csharp
var sortedByName = people.OrderBy(p => p.Name);
var sortedByAgeDesc = people.OrderByDescending(p => p.Age);
```

**4. Группировка (GroupBy):**
```csharp
var byCity = people.GroupBy(p => p.City);
foreach (var group in byCity)
{
    Console.WriteLine($"City: {group.Key}, Count: {group.Count()}");
}
```

**5. Агрегация:**
```csharp
var totalAge = people.Sum(p => p.Age);
var averageAge = people.Average(p => p.Age);
var maxAge = people.Max(p => p.Age);
var count = people.Count(p => p.Age > 18);
```

**6. Объединение (Join):**
```csharp
var result = from p in people
             join o in orders on p.Id equals o.CustomerId
             select new { p.Name, o.Product };
```

**Преимущества LINQ:**
- Единый синтаксис для разных источников данных
- Отложенное выполнение (deferred execution)
- Читаемость кода
- Поддержка IntelliSense

---

## 27. Что такое пространство имен (namespace) и зачем это нужно?
**Namespace (пространство имен)** — контейнер для логической группировки связанных типов.

**Зачем нужно:**
1. **Избежание конфликтов имен** — одинаковые имена в разных пространствах
2. **Организация кода** — логическая структура проекта
3. **Управление областью видимости** — контроль доступа к типам

```csharp
// Определение пространства имен
namespace Company.Project.Module
{
    public class MyClass { }
    public interface IMyInterface { }
}

// Использование
using Company.Project.Module;

var obj = new MyClass();
```

**Правила именования:**
- `CompanyName.ProductName.ModuleName`
- Использовать множественное число для коллекций: `System.Collections`
- Избегать аббревиатур (кроме общепринятых)

**Вложенные пространства имен:**
```csharp
namespace Outer
{
    namespace Inner
    {
        public class NestedClass { }
    }
}

// Или кратко
namespace Outer.Inner
{
    public class NestedClass { }
}
```

**Глобальное пространство имен:**
```csharp
// Доступ к типам без using
global::System.Console.WriteLine();
```

---

## 28. Какие типы данных вы знаете?
**В C# два основных категории типов:**

**1. Типы значений (Value Types):**
- Хранятся в стеке
- Копируются по значению
- Не могут быть null (кроме Nullable)
- Примеры: `int`, `double`, `bool`, `struct`, `enum`

**2. Ссылочные типы (Reference Types):**
- Хранятся в куче
- Копируются по ссылке
- Могут быть null
- Примеры: `class`, `interface`, `delegate`, `array`, `string`

**3. Указатели (Pointer Types):**
- Только в unsafe контексте
- Для работы с неуправляемой памятью

**4. Типы параметров:**
- `in` — входной параметр (readonly)
- `out` — выходной параметр
- `ref` — параметр по ссылке

---

## 29. Какие примитивные типы знаете?
**Примитивные (встроенные) типы C#:**

| Тип | Размер | Диапазон | Описание |
|-----|--------|----------|----------|
| `bool` | 1 байт | true/false | Логическое значение |
| `byte` | 1 байт | 0..255 | Беззнаковый байт |
| `sbyte` | 1 байт | -128..127 | Знаковый байт |
| `char` | 2 байта | U+0000..U+FFFF | Символ Unicode |
| `short` | 2 байта | -32,768..32,767 | Короткое целое |
| `ushort` | 2 байта | 0..65,535 | Беззнаковое короткое |
| `int` | 4 байта | -2.1B..2.1B | Целое число |
| `uint` | 4 байта | 0..4.3B | Беззнаковое целое |
| `long` | 8 байт | ±9.2×10¹⁸ | Длинное целое |
| `ulong` | 8 байт | 0..1.8×10¹⁹ | Беззнаковое длинное |
| `float` | 4 байта | ±1.5×10⁻⁴⁵..3.4×10³⁸ | Число с плавающей точкой |
| `double` | 8 байт | ±5.0×10⁻³²⁴..1.7×10³⁰⁸ | Двойная точность |
| `decimal` | 16 байт | ±1.0×10⁻²⁸..7.9×10²⁸ | Десятичное число (для финансов) |

**Псевдонимы:**
- `int` = `System.Int32`
- `string` = `System.String`
- `object` = `System.Object`

---

## 30. Что такое Nullable-тип?
**Nullable-тип** — тип значения, который может быть null.

**Синтаксис:**
```csharp
int? nullableInt = null;
Nullable<int> nullableInt2 = null; // Альтернативный синтаксис
```

**Свойства и методы:**
```csharp
int? number = null;

// Проверка наличия значения
if (number.HasValue)
{
    Console.WriteLine($"Value: {number.Value}");
}

// Получение значения или значение по умолчанию
int value = number.GetValueOrDefault(); // 0 если null
int value2 = number.GetValueOrDefault(42); // 42 если null

// Оператор объединения с null
int result = number ?? 100; // 100 если null
```

**Преобразование:**
```csharp
int? nullable = 42;
int nonNullable = nullable.Value; // Явное преобразование

int? fromNonNullable = 123; // Неявное преобразование
```

**Использование:**
- Работа с базами данных (поля могут быть NULL)
- Опциональные параметры
- Отсутствие значения как состояние

---
## 31. Что такое тип значения, а что такое тип ссылки? Что из этого class, а что struct? В каком участке памяти они хранятся?

**Тип значения (Value Type):**
- `struct`, `enum`
- Хранится в стеке (обычно)
- Копируется по значению
- Не может быть null (кроме `Nullable&lt;T&gt;`)
- Наследуется от `System.ValueType`

**Тип ссылки (Reference Type):**
- `class`, `interface`, `delegate`, `array`
- Хранится в куче
- Копируется по ссылке
- Может быть null
- Наследуется от `System.Object`

**Сравнение:**
```csharp
// Тип значения (struct)
public struct Point
{
    public int X, Y;
}

// Тип ссылки (class)
public class Person
{
    public string Name;
}

// Использование
Point p1 = new Point { X = 10, Y = 20 };
Point p2 = p1; // Копирование значений
p2.X = 30; // p1.X останется 10

Person person1 = new Person { Name = "John" };
Person person2 = person1; // Копирование ссылки
person2.Name = "Alice"; // person1.Name тоже станет "Alice"
```


## 32. Чем отличаются value or reference type? String - это reference или value?

### Отличия value type и reference type:

| Value Type (Тип значения) | Reference Type (Тип ссылки) |
|---------------------------|----------------------------|
| Хранится в стеке | Хранится в куче |
| Копируется по значению | Копируется по ссылке |
| Не может быть null (кроме Nullable) | Может быть null |
| `struct`, `enum` | `class`, `interface`, `delegate`, `array` |
| Наследуется от `System.ValueType` | Наследуется от `System.Object` |
| Быстрый доступ | Доступ через ссылку |
| Маленький размер | Любой размер |

### Пример различия в копировании:
```csharp
// Value type (struct)
public struct Point
{
    public int X, Y;
}

// Reference type (class)
public class Person
{
    public string Name;
}

// Использование
Point p1 = new Point { X = 10, Y = 20 };
Point p2 = p1; // КОПИРОВАНИЕ ЗНАЧЕНИЙ
p2.X = 30; // p1.X останется 10

Person person1 = new Person { Name = "John" };
Person person2 = person1; // КОПИРОВАНИЕ ССЫЛКИ
person2.Name = "Alice"; // person1.Name тоже станет "Alice"
```

### String - это reference или value?
**String — это reference type**, но с особенностями:

1. **Неизменяемость (Immutable):**
```csharp
string s1 = "Hello";
string s2 = s1; // Копирование ссылки
s2 = "World"; // Создается НОВАЯ строка, s1 не меняется
Console.WriteLine(s1); // "Hello"
Console.WriteLine(s2); // "World"
```

2. **Семантика сравнения как у value type:**
```csharp
string a = "Hello";
string b = "Hello";
Console.WriteLine(a == b); // True (сравнивается содержимое)
Console.WriteLine(object.ReferenceEquals(a, b)); // True (интернирование)

string c = new string("Hello".ToCharArray());
Console.WriteLine(a == c); // True (содержимое одинаково)
Console.WriteLine(object.ReferenceEquals(a, c)); // False (разные объекты)
```

3. **Интернирование строк (String Interning):**
```csharp
// Компилятор объединяет одинаковые строковые литералы
string s1 = "Hello";
string s2 = "Hello";
string s3 = "Hel" + "lo"; // Конкатенация на этапе компиляции

// Все три переменные ссылаются на один объект в пуле интернирования
Console.WriteLine(object.ReferenceEquals(s1, s2)); // True
Console.WriteLine(object.ReferenceEquals(s1, s3)); // True
```

### Практическое различие:

**Когда использовать value types:**
- Маленькие объекты (≤ 16 байт)
- Частые копирования
- Когда нужна скорость доступа
- Координаты, размеры, простые структуры данных

**Когда использовать reference types:**
- Большие объекты
- Когда нужно наследование
- Сложные объекты с поведением
- Когда объекты часто передаются как параметры

### Особенности string как reference type:

```csharp
// 1. Null возможен (как у reference type)
string s = null; // Разрешено

// 2. Передача по ссылке, но с неизменяемостью
void ModifyString(string str)
{
    str = "Modified"; // Создается новая строка, оригинал не меняется
}

string original = "Original";
ModifyString(original);
Console.WriteLine(original); // "Original" (не изменился)

// 3. String vs StringBuilder
string immutable = "";
for (int i = 0; i < 1000; i++)
    immutable += i; // Каждый раз создается НОВАЯ строка

StringBuilder mutable = new StringBuilder();
for (int i = 0; i < 1000; i++)
    mutable.Append(i); // Модификация того же объекта
```

### Ключевой вывод:
**String технически является reference type, но ведет себя во многом как value type** из-за неизменяемости и семантики сравнения по значению.
## 33. Что такое дженерики? Какие проблемы они решают?
**Дженерики (Generics)** — параметризованные типы, позволяющие создавать классы, методы и интерфейсы с параметрами типа.

**Проблемы, которые решают:**
1. **Безопасность типов** — ошибки на этапе компиляции
2. **Повторное использование кода** — один алгоритм для разных типов
3. **Производительность** — избегание boxing/unboxing

```csharp
// Обобщенный класс
public class Repository<T> where T : class
{
    private List&lt;T&gt; items = new List&lt;T&gt;();
    
    public void Add(T item) => items.Add(item);
    public T Get(int index) => items[index];
}

// Использование
var stringRepo = new Repository&lt;string&gt;();
stringRepo.Add("Hello");

var intRepo = new Repository&lt;int&gt;();
intRepo.Add(42);
```

---

## 34. Что такое Array, List, Dictionary? Приведите примеры использования этих структур данных. Какая сложность операций с ними (поиск, вставка, удаление)?

### 1. Array (массив)
**Фиксированный размер**, непрерывная память.

```csharp
int[] numbers = new int[5];
int[] numbers2 = { 1, 2, 3, 4, 5 };
numbers[0] = 10; // O(1) доступ
```

**Сложность операций:**
- Доступ по индексу: **O(1)**
- Поиск: **O(n)**
- Вставка/удаление: **O(n)**

### 2. List&lt;T&gt; (список)
**Динамический массив**, автоматически расширяется.

```csharp
List&lt;int&gt; numbers = new List&lt;int&gt;();
numbers.Add(1); // O(1) амортизированно
numbers[0] = 2; // O(1) доступ
numbers.Remove(1); // O(n) поиск + O(n) сдвиг
```

**Сложность операций:**
- Доступ по индексу: **O(1)**
- Поиск: **O(n)**
- Добавление в конец: **O(1)** амортизированно
- Вставка в середину: **O(n)**
- Удаление: **O(n)**

### 3. Dictionary&lt;TKey, TValue&gt; (словарь)
**Хеш-таблица**, пары ключ-значение.

```csharp
Dictionary&lt;string, int&gt; ages = new Dictionary&lt;string, int&gt;();
ages["John"] = 25; // O(1) в среднем
int age = ages["John"]; // O(1) в среднем
bool exists = ages.ContainsKey("John"); // O(1) в среднем
```

**Сложность операций:**
- Поиск/добавление/удаление: **O(1)** в среднем, **O(n)** в худшем случае
- Итерация: **O(n)**

---

## 35. Какие знаете коллекции?
**Системные коллекции в .NET:**

1. **System.Collections.Generic:**
   - `List&lt;T&gt;` — динамический массив
   - `Dictionary&lt;TKey, TValue&gt;` — хеш-таблица
   - `HashSet&lt;T&gt;` — множество уникальных элементов
   - `Queue&lt;T&gt;` — очередь (FIFO)
   - `Stack&lt;T&gt;` — стек (LIFO)
   - `LinkedList&lt;T&gt;` — двусвязный список

2. **System.Collections.Concurrent** (потокобезопасные):
   - `ConcurrentDictionary&lt;TKey, TValue&gt;`
   - `ConcurrentQueue&lt;T&gt;`
   - `ConcurrentStack&lt;T&gt;`

3. **System.Collections** (устаревшие, не типизированные):
   - `ArrayList` — как `List&lt;object&gt;`
   - `Hashtable` — как `Dictionary&lt;object, object&gt;`

---



## 36. Что такое класс?
**Класс** — шаблон для создания объектов, определяющий состояние (поля) и поведение (методы).

**Состав класса:**
- **Поля** — хранят данные
- **Свойства** — контролируемый доступ к полям
- **Методы** — поведение объекта
- **Конструкторы** — инициализация
- **События** — уведомления

```csharp
public class Person
{
    private string _name; // Поле
    
    public string Name // Свойство
    {
        get => _name;
        set => _name = value;
    }
    
    public Person(string name) // Конструктор
    {
        Name = name;
    }
    
    public void SayHello() // Метод
    {
        Console.WriteLine($"Hello, {Name}");
    }
}
```

---

## 37. Чем отличается класс от абстрактного класса?
| Обычный класс | Абстрактный класс |
|---------------|-------------------|
| Можно создать экземпляр | Нельзя создать экземпляр |
| Все методы могут иметь реализацию | Может содержать абстрактные методы (без реализации) |
| Не требует переопределения методов | Требует реализации абстрактных методов в наследниках |
| Может быть sealed | Не может быть sealed |

```csharp
// Обычный класс
public class Animal
{
    public void Eat() => Console.WriteLine("Eating");
}

// Абстрактный класс
public abstract class Shape
{
    public abstract double Area(); // Абстрактный метод
    public void Display() => Console.WriteLine("Shape"); // Обычный метод
}

// Наследник
public class Circle : Shape
{
    public double Radius { get; set; }
    public override double Area() => Math.PI * Radius * Radius; // Реализация
}
```

---

## 38. Чем отличается абстрактный класс от интерфейса?
| Абстрактный класс | Интерфейс |
|-------------------|-----------|
| Может содержать реализацию | Только сигнатуры методов (до C# 8.0) |
| Может иметь поля | Не может иметь полей (до C# 8.0) |
| Одиночное наследование | Множественная реализация |
| Конструкторы | Нет конструкторов |
| Модификаторы доступа | Все члены public |

**Интерфейсы нужны для:**
- Определения контракта (что должен делать класс)
- Внедрения зависимостей
- Множественного "наследования"
- Создания слабосвязанной архитектуры

---

## 39. Какие вы знаете модификаторы доступа?
1. **public** — доступ отовсюду
2. **private** — только внутри класса
3. **protected** — класс и наследники
4. **internal** — в пределах сборки
5. **protected internal** — protected ИЛИ internal
6. **private protected** — protected И internal (C# 7.2+)

---

## 40. В чем разница между обычным классом и статическим?
| Обычный класс | Статический класс |
|---------------|------------------|
| Можно создавать экземпляры | Нельзя создать экземпляр |
| Может иметь состояние (нестатические поля) | Только статические члены |
| Может реализовывать интерфейсы | Не может реализовывать интерфейсы |
| Может наследоваться | Не может наследоваться |

```csharp
// Обычный класс
public class Calculator
{
    public int Add(int a, int b) => a + b;
}

// Статический класс
public static class MathUtils
{
    public static int Add(int a, int b) => a + b;
}
```

---

## 41. В чем разница переопределения метода между ключевыми словами new и override?
**`override`** — полиморфное переопределение:
```csharp
class Base { public virtual void Method() { } }
class Derived : Base { public override void Method() { } }
Base obj = new Derived();
obj.Method(); // Вызовет Derived.Method()
```

**`new`** — скрытие метода:
```csharp
class Base { public void Method() { } }
class Derived : Base { public new void Method() { } }
Base obj = new Derived();
obj.Method(); // Вызовет Base.Method()
```

**Разница:** `override` изменяет поведение при вызове через базовый тип, `new` — нет.

---

## 42. Какое различие между const и readonly?
| const | readonly |
|-------|----------|
| Инициализация при объявлении | Инициализация в объявлении или конструкторе |
| Только примитивы и string | Любой тип |
| Компиляционная константа | Константа времени выполнения |
| Неявно static | Может быть экземплярным или static |

```csharp
const int Max = 100; // Значение подставляется при компиляции
readonly DateTime Created = DateTime.Now; // Определяется при создании объекта
```

---

## 43. Разница между структурой и классом. Приведите примеры структур.
| Структура | Класс |
|-----------|-------|
| Тип значения | Тип ссылки |
| В стеке (обычно) | В куче |
| Копируется по значению | Копируется по ссылке |
| Не поддерживает наследование | Поддерживает наследование |

**Примеры структур:** `int`, `DateTime`, `Point`, `Rectangle`, `Guid`

```csharp
public struct Point
{
    public int X, Y;
}

public class Person
{
    public string Name;
}
```

---

## 44. Может ли экземпляр структуры храниться в куче (heap)? Как это сделать?
**Да**, структура может оказаться в куче при:
1. **Упаковке (boxing):**
   ```csharp
   struct Point { int x, y; }
   object boxed = new Point(); // Упаковка в кучу
   ```

2. **Поле в классе:**
   ```csharp
   class Shape { Point position; } // Point в куче как часть Shape
   ```

3. **Элемент массива:**
   ```csharp
   Point[] points = new Point[10]; // Весь массив в куче
   ```

4. **Захват в замыкании** (closure)
5. **`async` метод** — локальные переменные

---

## 45. Что такое асинхронность и чем она отличается от многопоточности?
**Асинхронность** — выполнение без блокировки потока (для I/O операций).
**Многопоточность** — параллельное выполнение кода (для CPU-bound задач).

| Асинхронность | Многопоточность |
|---------------|-----------------|
| Освобождает поток во время ожидания | Создает потоки |
| Для сетевых запросов, файлов | Для сложных вычислений |
| `async`/`await` | `Thread`, `Task.Run` |

---

## 46. Какие есть ключевые слова для использования асинхронности в коде?
1. **`async`** — помечает метод как асинхронный
2. **`await`** — ожидание завершения асинхронной операции
3. **`Task`** — представляет асинхронную операцию
4. **`Task<T>`** — операция с результатом типа T
5. **`CancellationToken`** — отмена операций
6. **`Task.WhenAll`** — ожидание всех задач
7. **`Task.WhenAny`** — ожидание первой задачи

```csharp
public async Task<string> GetDataAsync()
{
    string result = await httpClient.GetStringAsync(url);
    return result;
}
```
