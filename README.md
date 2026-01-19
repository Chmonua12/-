## **1. Опишите кратко суть клиент-серверного взаимодействия на примере протокола HTTP. Каковы основные компоненты HTTP-запроса и ответа?**

**Суть взаимодействия:**  
Обмен сообщениями по схеме «запрос-ответ» между клиентом (например, браузером) и сервером. Клиент инициирует соединение, отправляет запрос на получение ресурса, сервер обрабатывает его и возвращает результат.

---

### **Основные компоненты HTTP-запроса:**
1. **Стартовая строка (Request Line)**  
   `Метод URI Версия_протокола`  
   *Пример:* `GET /index.html HTTP/1.1`

2. **Заголовки (Headers)**  
   Параметры запроса в формате `Ключ: Значение`  
   *Примеры:*  
   `Host: example.com`  
   `User-Agent: Mozilla/5.0`  
   `Accept-Language: ru`

3. **Тело сообщения (Body)**  
   Необязательная часть, передаёт данные (например, поля формы).

---

### **Основные компоненты HTTP-ответа:**
1. **Строка статуса (Status Line)**  
   `Версия_протокола Код_статуса Пояснение`  
   *Пример:* `HTTP/1.1 200 OK`

2. **Заголовки ответа (Headers)**  
   *Примеры:*  
   `Content-Type: text/html`  
   `Server: nginx`  
   `Set-Cookie: sessionid=abc123`

3. **Тело ответа (Body)**  
   Запрашиваемый ресурс (HTML, JSON и т.д.).

---

**Дополнительно:**  
- Протокол **статус-лесс (stateless)** – сервер не хранит состояние между запросами по умолчанию.  
- Для поддержания сессии используются **cookies**, **токены** или **сессионные ключи**.  
- Современные версии **HTTP/2** и **HTTP/3** добавляют мультиплексирование, сжатие заголовков и улучшенную производительность.

---

## **2. Что такое сокет (socket) в программировании? Опишите минимальную последовательность шагов для создания простого TCP-сервера, ожидающего подключений.**

**Сокет (socket)** – программный интерфейс (API) операционной системы для сетевого взаимодействия. Абстракция конечной точки соединения, объединяющая **IP-адрес** и **порт**.

---

### **Последовательность шагов для TCP-сервера:**

```python
1. Создание сокета
   socket(AF_INET, SOCK_STREAM)

2. Привязка к адресу и порту
   bind(('127.0.0.1', 8080))

3. Перевод в режим прослушивания
   listen(5)  # 5 - размер очереди подключений

4. Принятие подключения
   client_socket, addr = accept()  # Блокируется до подключения

5. Обмен данными
   data = recv(1024)  # Получение
   send(b'Response')   # Отправка

6. Закрытие соединений
   client_socket.close()
   server_socket.close()
```

---

**Дополнительно:**  
- Для обработки множества клиентов используют:  
  - **Многопоточность/многопроцессность**  
  - **Мультиплексирование** (`select`, `poll`, `epoll`)  
  - **Асинхронные модели** (asyncio)  
- UDP-сокеты (`SOCK_DGRAM`) не требуют установки соединения, работают по принципу «отправил и забыл».

---

## **3. В чём состоит принципиальная разница между синхронной и асинхронной моделью обработки запросов на сервере? Какие преимущества и сложности у каждого подхода?**

**Ключевое различие** – в управлении потоком выполнения во время операций **ввода-вывода (I/O)**.

| **Аспект** | **Синхронная модель** | **Асинхронная модель** |
|------------|------------------------|-------------------------|
| **Принцип работы** | Поток блокируется на время I/O операции | Поток не блокируется, операции выполняются «в фоне» |
| **Параллелизм** | Через потоки/процессы (1 поток = 1 запрос) | Через цикл событий (1 поток = множество запросов) |
| **Преимущества** | - Простота понимания и отладки<br>- Богатая экосистема библиотек | - Высокая производительность при I/O нагрузке<br>- Экономия памяти<br>- Подходит для long-polling, WebSockets |
| **Сложности** | - Накладные расходы на потоки<br>- Ограничение числа одновременных соединений | - Сложность отладки<br>- Риск блокировки цикла событий<br>- Требует специальных асинхронных библиотек |

---

**Примеры:**  
- **Синхронные фреймворки:** Django (по умолчанию), Flask  
- **Асинхронные фреймворки:** FastAPI, Sanic, aiohttp  
- **Гибридные:** Django с ASGI (Channels), Tornado

---

## **4. Что такое SQL-инъекция (SQL Injection) и какие базовые меры защиты от неё должен предусмотреть разработчик серверной части?**

**SQL-инъекция** – уязвимость, позволяющая злоумышленнику внедрить произвольный SQL-код в запрос к базе данных через неправильно обработанные пользовательские данные.

**Пример уязвимого кода:**
```python
# НЕПРАВИЛЬНО - уязвимо к инъекции
query = f"SELECT * FROM users WHERE name = '{user_input}'"
```

---

### **Базовые меры защиты:**

| **Мера** | **Описание** | **Пример в Django** |
|----------|--------------|---------------------|
| **Параметризованные запросы** | Отделение данных от кода запроса | `User.objects.raw('SELECT * FROM users WHERE name = %s', [user_input])` |
| **ORM** | Использование ORM, который автоматически экранирует данные | `User.objects.filter(name=user_input)` |
| **Валидация ввода** | Проверка типа, длины и формата данных | Django Forms, валидаторы |
| **Экранирование** | Специальное кодирование опасных символов | `connection.ops.quote_name()` |
| **Принцип наименьших привилегий** | Ограничение прав пользователя БД | Только SELECT/INSERT, без DROP |
| **Регулярное обновление** | Патчинг СУБД и фреймворков | Обновления безопасности |

---

**Дополнительные меры:**  
- **WAF (Web Application Firewall)** – фильтрация запросов на уровне сети  
- **Статический анализ кода (SAST)** – автоматическое обнаружение уязвимостей  
- **Хранимые процедуры** – с валидацией параметров  
- **Логирование и мониторинг** подозрительных запросов

---

## **5. Опишите архитектурный паттерн MVT (Model-View-Template), используемый в Django. Какую роль играет каждый из этих компонентов в обработке пользовательского запроса?**

**MVT** – вариация паттерна **MVC**, адаптированная для Django.

### **Схема обработки запроса:**
```
Запрос → URL Dispatcher → View → Model ←→ БД
                                   ↓
                             Template → HTML-ответ
```

---

### **Роль компонентов:**

| **Компонент** | **Отвечает за** | **Пример в Django** | **Аналог в MVC** |
|---------------|-----------------|---------------------|------------------|
| **Model** | Данные и бизнес-логику | `models.py`, ORM | Model |
| **View** | Обработку запроса, логику представления | `views.py`, функции/классы | Controller |
| **Template** | Отображение данных (презентационный слой) | `.html` файлы с DTL | View |

---

### **Детальный процесс:**
1. **Пользователь** отправляет запрос на URL `/articles/5/`
2. **URL Dispatcher** (`urls.py`) находит совпадение и вызывает соответствующее **View**
3. **View** обращается к **Model** для получения статьи с `id=5`
4. **Model** выполняет SQL-запрос через ORM и возвращает объект статьи
5. **View** передаёт объект статьи в **Template** вместе с контекстом
6. **Template** рендерит HTML-страницу с данными статьи
7. **View** возвращает готовый **HTTP-ответ** клиенту

---

**Важное отличие от MVC:**  
В Django **Controller** – это сам фреймворк (механизм роутинга `urls.py` + среда выполнения), а **View** – ближе к контроллеру в классическом MVC.

---

Теперь продолжу с ответами в отформатированном виде:

## **6. Каковы ключевые преимущества использования ORM (Object-Relational Mapper) Django по сравнению с написанием "сырых" SQL-запросов? Приведите пример возможного недостатка ORM.**

### **Преимущества Django ORM:**

| **Преимущество** | **Описание** | **Пример** |
|------------------|--------------|------------|
| **Продуктивность** | Меньше кода, выше скорость разработки | `User.objects.filter(is_active=True)` vs полный SQL запрос |
| **Безопасность** | Автоматическая защита от SQL-инъекций | Параметризация всех запросов "из коробки" |
| **Переносимость** | Абстракция от конкретной СУБД | Один код для SQLite, PostgreSQL, MySQL |
| **Мощные абстракции** | Сложные запросы простым синтаксисом | `annotate()`, `aggregate()`, `prefetch_related()` |
| **Миграции** | Управление схемой БД через код | `makemigrations`, `migrate` |
| **Интеграция с Django** | Глубокая интеграция со всеми компонентами | Forms, Admin, Authentication |

```python
# Пример ORM запроса
active_users = User.objects.filter(
    is_active=True,
    date_joined__year=2024
).select_related('profile')
```

---

### **Пример недостатка ORM:**

**Проблема производительности на сложных запросах:**

```python
# ORM может генерировать неоптимальный запрос
books = Book.objects.filter(
    author__country='USA',
    published_date__year__gte=2020
)

# Сырой SQL может быть эффективнее
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute("""
        SELECT b.*, a.name 
        FROM books_book b
        JOIN books_author a ON b.author_id = a.id
        WHERE a.country = %s 
        AND EXTRACT(YEAR FROM b.published_date) >= %s
        ORDER BY b.rating DESC
        LIMIT 100
    """, ['USA', 2020])
```

**Типичные проблемы ORM:**
- **N+1 запрос** – без `select_related`/`prefetch_related`
- **Избыточные JOIN** – когда достаточно подзапроса
- **Сложные аналитические запросы** – оконные функции, CTE
- **Безусловная выборка всех полей** – когда нужны только некоторые

---

## **7. Опишите основные типы связей между моделями в Django (OneToOne, ForeignKey, ManyToMany). Приведите по одному практическому примеру для каждого типа.**

### **Сравнительная таблица связей:**

| **Тип связи** | **Назначение** | **Реализация в БД** | **Пример** |
|---------------|----------------|---------------------|------------|
| **OneToOneField** | Строгая уникальная связь 1:1 | Внешний ключ с UNIQUE constraint | Пользователь ↔ Профиль |
| **ForeignKey** | Связь "многие к одному" (N:1) | Внешний ключ в таблице "многих" | Статья → Автор |
| **ManyToManyField** | Связь "многие ко многим" (N:M) | Промежуточная таблица | Студент ↔ Курс |

---

### **Примеры реализации:**

**1. OneToOneField (1:1)**
```python
from django.contrib.auth.models import User
from django.db import models

class UserProfile(models.Model):
    user = models.OneToOneField(
        User, 
        on_delete=models.CASCADE,
        primary_key=True  # Опционально: делает связь более явной
    )
    bio = models.TextField()
    phone = models.CharField(max_length=20)
    
# Использование
user = User.objects.get(username='john')
profile = user.userprofile  # Доступ через related_name
```

**2. ForeignKey (N:1)**
```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,  # При удалении автора удалятся его статьи
        related_name='articles'    # author.articles.all()
    )
    published_date = models.DateField()

# Использование
author = Author.objects.get(name='Достоевский')
articles = author.articles.filter(published_date__year=2024)
```

**3. ManyToManyField (N:M)**
```python
class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField(
        'Course',
        through='Enrollment',  # Промежуточная модель
        related_name='students'
    )

class Course(models.Model):
    title = models.CharField(max_length=200)
    credits = models.IntegerField()

# Промежуточная модель для дополнительных данных
class Enrollment(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrollment_date = models.DateField()
    grade = models.CharField(max_length=2, null=True, blank=True)

# Использование
student = Student.objects.get(name='Иван Иванов')
courses = student.courses.all()  # Все курсы студента
```

---

**Ключевые параметры:**
- `on_delete` – поведение при удалении связанного объекта:
  - `CASCADE` – каскадное удаление
  - `PROTECT` – запрет удаления
  - `SET_NULL` – установка NULL (требует `null=True`)
  - `SET_DEFAULT` – установка значения по умолчанию
- `related_name` – имя для обратной связи
- `through` – указание промежуточной модели для ManyToMany

---

## **8. Как Django определяет, какое представление (view) должно обработать конкретный URL-запрос пользователя? Опишите роль файла urls.py и функции path().**

### **Механизм URL Dispatcher:**

**Алгоритм работы:**
```
Запрос → ROOT_URLCONF → Поиск в urlpatterns → Совпадение → Вызов View
```

---

### **Роль файла `urls.py`:**
Главный файл конфигурации URL, содержит переменную `urlpatterns` – список путей.

**Структура проекта:**
```
myproject/
├── myproject/
│   ├── urls.py          # Главный URLconf проекта
│   └── ...
└── myapp/
    ├── urls.py          # URLconf приложения
    └── ...
```

---

### **Роль функции `path()`:**

```python
# Сигнатура
path(route, view, kwargs=None, name=None)
```

| **Параметр** | **Назначение** | **Пример** |
|--------------|----------------|------------|
| **route** | Шаблон URL с конвертерами | `'articles/<int:pk>/'` |
| **view** | Функция или класс представления | `article_detail` |
| **name** | Имя маршрута для `{% url %}` | `'article_detail'` |
| **kwargs** | Дополнительные аргументы для view | `{'template': 'custom.html'}` |

---

### **Конвертеры пути (Path Converters):**

| **Конвертер** | **Пример** | **Сопоставляет** |
|---------------|------------|-------------------|
| `str` | `<str:slug>` | Любую непустую строку без `/` |
| `int` | `<int:pk>` | Целые числа |
| `slug` | `<slug:title>` | ASCII буквы/цифры, дефисы, подчёркивания |
| `uuid` | `<uuid:uid>` | UUID строки |
| `path` | `<path:subpath>` | Любую строку, включая `/` |

---

### **Примеры конфигурации:**

```python
# myproject/urls.py
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),  # Включение URL приложения
    path('about/', views.about, name='about'),
]

# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.ArticleListView.as_view(), name='article_list'),
    path('<int:year>/', views.article_archive, name='article_archive'),
    path('<int:year>/<int:month>/', views.article_month_archive),
    path('<slug:slug>/', views.ArticleDetailView.as_view(), name='article_detail'),
    path('search/', views.search, name='search'),
]
```

---

### **Процесс разрешения URL:**

1. **Запрос** `/blog/2024/my-article/`
2. **Django** загружает `ROOT_URLCONF` (`myproject.urls`)
3. **Находит** `path('blog/', include('blog.urls'))`
4. **Переходит** в `blog.urls` и ищет совпадения
5. **Пропускает** первые два пути (не совпадают)
6. **Находит** `'<slug:slug>/'` → извлекает `slug='my-article'`
7. **Вызывает** `ArticleDetailView.as_view()(request, slug='my-article')`
8. **Возвращает** результат обработки

---

**Дополнительно:**
- `re_path()` – для сложных регулярных выражений
- `include()` – для модульности и пространств имён
- `app_name` – пространство имён приложения
- Обработка ошибок: 404 при отсутствии совпадений, 500 при ошибке в view

---

## **9. Для чего используются декораторы в контексте Function-Based Views (например, @login_required, @csrf_exempt)? Как они меняют поведение функции-представления?**

### **Что такое декораторы в Django?**

**Декоратор** – функция Python, которая принимает другую функцию и возвращает модифицированную версию, добавляя новое поведение.

---

### **Основные встроенные декораторы:**

| **Декоратор** | **Назначение** | **Пример использования** |
|---------------|----------------|--------------------------|
| **`@login_required`** | Требует аутентификации | Доступ только для авторизованных пользователей |
| **`@permission_required`** | Проверяет права | `@permission_required('app.add_article')` |
| **`@user_passes_test`** | Кастомная проверка | `@user_passes_test(lambda u: u.is_staff)` |
| **`@csrf_exempt`** | Отключает CSRF защиту | Для API эндпоинтов |
| **`@require_http_methods`** | Ограничивает HTTP-методы | `@require_http_methods(["GET", "POST"])` |
| **`@cache_page`** | Кэширование ответа | `@cache_page(60 * 15)` |

---

### **Как работают декораторы:**

**Без декоратора:**
```python
def my_view(request):
    if not request.user.is_authenticated:
        return redirect('/login/')
    # Основная логика
    return HttpResponse('Секретная страница')
```

**С декоратором:**
```python
from django.contrib.auth.decorators import login_required

@login_required(login_url='/custom-login/')
def my_view(request):
    # Код выполняется только для авторизованных
    return HttpResponse('Секретная страница')
```

---

### **Принцип работы декоратора `@login_required`:**

```python
# Упрощенная реализация декоратора
def login_required(view_func, login_url=None):
    def wrapper(request, *args, **kwargs):
        if request.user.is_authenticated:
            return view_func(request, *args, **kwargs)
        else:
            return redirect(login_url or settings.LOGIN_URL)
    return wrapper

# Применение декоратора
@login_required
def profile_view(request):
    return render(request, 'profile.html')
    
# Эквивалентно:
profile_view = login_required(profile_view)
```

---

### **Каскадное применение декораторов:**

```python
from django.views.decorators.http import require_http_methods
from django.views.decorators.cache import cache_page
from django.contrib.auth.decorators import login_required

@require_http_methods(["GET"])
@cache_page(60 * 5)  # Кэшировать на 5 минут
@login_required
def expensive_view(request):
    # 1. Проверяется метод (только GET)
    # 2. Проверяется аутентификация
    # 3. Проверяется кэш
    # 4. Выполняется тяжёлая операция
    data = perform_expensive_query()
    return JsonResponse(data)
```

**Порядок имеет значение!** Декораторы применяются снизу вверх.

---

### **Декораторы для проверки прав:**

```python
from django.contrib.auth.decorators import (
    login_required, 
    permission_required,
    user_passes_test
)

# Комбинация декораторов
@login_required
@permission_required('blog.publish_article', raise_exception=True)
def publish_article(request, article_id):
    # Только авторизованные пользователи с правом publish_article
    pass

# Кастомная проверка
def is_premium_user(user):
    return user.profile.is_premium

@user_passes_test(is_premium_user)
def premium_content(request):
    return render(request, 'premium.html')
```

---

### **Декораторы для безопасности:**

```python
from django.views.decorators.csrf import csrf_exempt, csrf_protect
from django.views.decorators.clickjacking import xframe_options_exempt

# Отключение CSRF для API (осторожно!)
@csrf_exempt
def api_endpoint(request):
    return JsonResponse({'status': 'ok'})

# Явное включение CSRF
@csrf_protect
def form_view(request):
    # CSRF проверка выполнится
    pass

# Разрешение встраивания в iframe
@xframe_options_exempt
def embeddable_widget(request):
    return render(request, 'widget.html')
```

---

### **Декораторы для оптимизации:**

```python
from django.views.decorators.cache import cache_page, never_cache
from django.views.decorators.vary import vary_on_cookie, vary_on_headers

# Кэширование
@cache_page(60 * 15)  # 15 минут
@vary_on_cookie  # Разные версии для разных пользователей
def cached_view(request):
    pass

# Запрет кэширования
@never_cache
def sensitive_view(request):
    # Конфиденциальные данные
    pass
```

---

**Важно:** Для Class-Based Views декораторы применяются через `@method_decorator`:

```python
from django.utils.decorators import method_decorator
from django.contrib.auth.decorators import login_required

@method_decorator(login_required, name='dispatch')
class ProtectedView(View):
    def get(self, request):
        return HttpResponse('Защищено')
```

## **10. В чём заключаются основные преимущества использования Class-Based Views (CBV) по сравнению с Function-Based Views? Назовите несколько встроенных CBV для типовых задач.**

### **Основные преимущества CBV:**

| **Преимущество** | **Описание** | **Пример в FBV** | **Пример в CBV** |
|------------------|--------------|------------------|------------------|
| **Наследование и переиспользование** | Создание базовых классов с общей логикой | Копирование кода | Наследование от `TemplateView` |
| **Организация по HTTP-методам** | Раздельная обработка GET, POST и др. | `if request.method == 'GET':` | Методы `get()`, `post()` |
| **Встроенные generic-представления** | Готовые решения для типовых задач | Ручная реализация | `ListView`, `DetailView` |
| **Миксины (Mixin)** | Модульное добавление функциональности | Декораторы | `LoginRequiredMixin` |
| **Лучшая структурированность** | Разделение ответственности методов | Вся логика в одной функции | `get_context_data()`, `form_valid()` |

---

### **Встроенные Generic CBV для типовых задач:**

#### **1. Display Views (Отображение данных)**
```python
# Список объектов
from django.views.generic import ListView
class ArticleListView(ListView):
    model = Article
    template_name = 'articles/list.html'
    context_object_name = 'articles'
    paginate_by = 10

# Детальный просмотр объекта
from django.views.generic import DetailView
class ArticleDetailView(DetailView):
    model = Article
    template_name = 'articles/detail.html'
    
# Произвольный шаблон (без модели)
from django.views.generic import TemplateView
class AboutView(TemplateView):
    template_name = 'about.html'
```

#### **2. Editing Views (Редактирование данных)**
```python
# Создание объекта
from django.views.generic import CreateView
class ArticleCreateView(CreateView):
    model = Article
    form_class = ArticleForm
    template_name = 'articles/create.html'
    success_url = '/articles/'

# Обновление объекта
from django.views.generic import UpdateView
class ArticleUpdateView(UpdateView):
    model = Article
    form_class = ArticleForm
    template_name = 'articles/update.html'

# Удаление объекта
from django.views.generic import DeleteView
class ArticleDeleteView(DeleteView):
    model = Article
    template_name = 'articles/confirm_delete.html'
    success_url = '/articles/'
```

#### **3. Date-Based Views (Работа с датами)**
```python
from django.views.generic.dates import (
    ArchiveIndexView, YearArchiveView, 
    MonthArchiveView, DayArchiveView
)

class ArticleYearArchiveView(YearArchiveView):
    model = Article
    date_field = 'published_date'
    make_object_list = True  # Возвращает список объектов, а не только даты
```

---

### **Сравнение FBV и CBV на примере:**

**Function-Based View:**
```python
def article_list(request):
    articles = Article.objects.filter(is_published=True)
    page = request.GET.get('page', 1)
    paginator = Paginator(articles, 10)
    
    try:
        articles_page = paginator.page(page)
    except PageNotAnInteger:
        articles_page = paginator.page(1)
    except EmptyPage:
        articles_page = paginator.page(paginator.num_pages)
    
    return render(request, 'articles/list.html', {
        'articles': articles_page
    })
```

**Class-Based View:**
```python
class ArticleListView(ListView):
    model = Article
    template_name = 'articles/list.html'
    paginate_by = 10
    
    def get_queryset(self):
        # Кастомная фильтрация
        return Article.objects.filter(
            is_published=True
        ).select_related('author')
    
    def get_context_data(self, **kwargs):
        # Добавление дополнительного контекста
        context = super().get_context_data(**kwargs)
        context['now'] = timezone.now()
        return context
```

---

### **Миксины для расширения функциональности:**

```python
from django.contrib.auth.mixins import (
    LoginRequiredMixin, 
    PermissionRequiredMixin,
    UserPassesTestMixin
)

# Требует аутентификации
class ProtectedView(LoginRequiredMixin, CreateView):
    login_url = '/login/'
    redirect_field_name = 'next'
    
# Требует определённых прав
class StaffOnlyView(PermissionRequiredMixin, UpdateView):
    permission_required = 'app.change_model'
    raise_exception = True  # 403 вместо редиректа на логин
    
# Кастомная проверка
class OwnerOnlyView(UserPassesTestMixin, DetailView):
    def test_func(self):
        obj = self.get_object()
        return obj.owner == self.request.user
```

---

### **Жизненный цикл CBV (на примере CreateView):**
```
as_view() → dispatch() → http_method_not_allowed() 
→ get()/post() → get_form() → form_valid()/form_invalid()
→ get_success_url() → render_to_response()
```

**Переопределение методов:**
```python
class ArticleCreateView(CreateView):
    model = Article
    
    def form_valid(self, form):
        # Дополнительная логика перед сохранением
        form.instance.author = self.request.user
        form.instance.created_at = timezone.now()
        
        # Отправка уведомления
        send_notification(form.instance)
        
        # Вызов родительского метода
        return super().form_valid(form)
```

---

### **Когда использовать что:**

| **Ситуация** | **Рекомендация** | **Причина** |
|--------------|------------------|-------------|
| Простая страница | `TemplateView` | Минимальный код |
| CRUD операции | Generic CBV | Готовая реализация |
| Сложная бизнес-логика | FBV или кастомная CBV | Больше контроля |
| Много общих проверок | CBV с миксинами | Переиспользование |
| API endpoint | FBV или DRF views | Гибкость |

---

## **11. Как работает механизм наследования шаблонов в Django? Объясните назначение тегов {% block %} и {% extends %} и их роль в поддержке единого дизайна сайта.**

### **Концепция наследования шаблонов**

**Наследование шаблонов** позволяет создавать базовую структуру сайта (макет) и расширять её в дочерних шаблонах, переопределяя только необходимые части.

---

### **Ключевые теги:**

| **Тег** | **Назначение** | **Пример** |
|---------|----------------|------------|
| **`{% extends %}`** | Наследование от базового шаблона | `{% extends 'base.html' %}` |
| **`{% block %}`** | Определение заменяемой области | `{% block content %}{% endblock %}` |
| **`{{ block.super }}`** | Получение содержимого родительского блока | `{{ block.super }}` |

---

### **Пример базового шаблона (`base.html`):**

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    {% block meta %}
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    {% endblock %}
    
    <title>{% block title %}Мой сайт{% endblock %}</title>
    
    {% block css %}
    <link rel="stylesheet" href="/static/css/main.css">
    {% endblock %}
</head>
<body>
    <!-- Шапка сайта -->
    <header>
        {% block header %}
        <nav>Навигация</nav>
        {% endblock %}
    </header>
    
    <!-- Основной контент -->
    <main>
        {% block content %}
        <!-- Будет заменено в дочерних шаблонах -->
        {% endblock %}
    </main>
    
    <!-- Подвал -->
    <footer>
        {% block footer %}
        <p>&copy; 2024 Мой сайт</p>
        {% endblock %}
    </footer>
    
    {% block javascript %}
    <script src="/static/js/main.js"></script>
    {% endblock %}
</body>
</html>
```

---

### **Пример дочернего шаблона (`article_detail.html`):**

```html
{% extends 'base.html' %}

<!-- Переопределение заголовка -->
{% block title %}{{ article.title }} | Мой сайт{% endblock %}

<!-- Добавление дополнительных CSS -->
{% block css %}
{{ block.super }}  <!-- Сохраняем родительские стили -->
<link rel="stylesheet" href="/static/css/article.css">
{% endblock %}

<!-- Переопределение контента -->
{% block content %}
<article class="article">
    <h1>{{ article.title }}</h1>
    <div class="meta">
        Автор: {{ article.author }} | {{ article.published_date }}
    </div>
    <div class="content">
        {{ article.content|safe }}
    </div>
</article>
{% endblock %}

<!-- Добавление специфичного JavaScript -->
{% block javascript %}
{{ block.super }}
<script src="/static/js/article.js"></script>
{% endblock %}
```

---

### **Многоуровневое наследование:**

```
base.html (общая структура)
└── blog_base.html (шаблон для блога)
    ├── article_list.html (список статей)
    └── article_detail.html (детальная статья)
```

**`blog_base.html`:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="blog-container">
    <aside class="sidebar">
        {% block sidebar %}
        <!-- Боковая панель для блога -->
        {% endblock %}
    </aside>
    
    <div class="blog-content">
        {% block blog_content %}
        <!-- Контент блога -->
        {% endblock %}
    </div>
</div>
{% endblock %}
```

**`article_detail.html`:**
```html
{% extends 'blog_base.html' %}

{% block sidebar %}
<h3>Категории</h3>
<ul>
    {% for category in categories %}
    <li>{{ category.name }}</li>
    {% endfor %}
</ul>
{% endblock %}

{% block blog_content %}
<article>
    <!-- Контент статьи -->
</article>
{% endblock %}
```

---

### **Особенности работы:**

1. **Порядок тегов:** `{% extends %}` должен быть **первым** тегом в шаблоне
2. **Несколько блоков:** Можно определять любое количество блоков
3. **Вложенные блоки:** Блоки могут быть вложенными
4. **Необязательные блоки:** Если блок не переопределён, используется содержимое из базового шаблона
5. **Динамическое наследование:** Можно наследовать от разных шаблонов

```html
<!-- Динамический выбор базового шаблона -->
{% extends base_template|default:"base.html" %}

<!-- Условное наследование -->
{% if request.user.is_staff %}
    {% extends 'admin_base.html' %}
{% else %}
    {% extends 'user_base.html' %}
{% endif %}
```

---

### **Включение шаблонов (`include`):**

```html
<!-- base.html -->
<body>
    {% include 'header.html' %}
    
    {% block content %}{% endblock %}
    
    {% include 'footer.html' %}
    
    <!-- С передачей контекста -->
    {% include 'widget.html' with user=request.user only %}
</body>
```

---

### **Преимущества подхода:**

| **Преимущество** | **Описание** |
|------------------|--------------|
| **Единообразие дизайна** | Все страницы используют одну базовую структуру |
| **Легкость поддержки** | Изменения в базовом шаблоне применяются ко всем страницам |
| **Разделение ответственности** | Дизайнеры работают с базовыми шаблонами, разработчики - с контентом |
| **Повторное использование** | Общие элементы выносятся в отдельные включаемые шаблоны |
| **Гибкость** | Можно создавать различные варианты макетов |

---

## **12. Каковы основные элементы синтаксиса Django Template Language (DTL)? Приведите примеры использования переменной, тега и фильтра, объяснив их отличие.**

### **Три основных элемента DTL:**

| **Элемент** | **Синтаксис** | **Назначение** | **Пример** |
|-------------|---------------|----------------|------------|
| **Переменные** | `{{ variable }}` | Вывод значения | `{{ user.username }}` |
| **Теги** | `{% tag %}` | Логика шаблона | `{% for item in list %}` |
| **Фильтры** | `{{ variable|filter }}` | Преобразование значения | `{{ text|truncatewords:50 }}` |

---

### **1. Переменные (Variables)**

**Назначение:** Вывод значений из контекста шаблона.

```html
<!-- Простая переменная -->
<p>Привет, {{ user.first_name }}!</p>

<!-- Атрибуты объектов (через точку) -->
<p>Статья: {{ article.title }}</p>
<p>Автор: {{ article.author.username }}</p>

<!-- Вызов методов (без скобок) -->
<p>Всего статей: {{ article_list.count }}</p>

<!-- Элементы словаря -->
<p>Настройка: {{ settings.TIME_ZONE }}</p>

<!-- Элементы списка по индексу -->
<p>Первый элемент: {{ items.0 }}</p>
```

**Особенности:**
- Автоматическое экранирование HTML (XSS защита)
- Для отключения экранирования: `{{ html_content|safe }}`
- Если переменная не существует – пустая строка (не ошибка)

---

### **2. Теги (Tags)**

**Назначение:** Реализация логики в шаблонах (циклы, условия, наследование).

#### **Основные теги:**

```html
<!-- Условия -->
{% if user.is_authenticated %}
    <p>Добро пожаловать, {{ user.username }}!</p>
{% elif user.is_anonymous %}
    <p>Пожалуйста, войдите в систему.</p>
{% else %}
    <p>Неизвестный пользователь.</p>
{% endif %}

<!-- Циклы -->
<ul>
{% for article in articles %}
    <li>{{ forloop.counter }}. {{ article.title }}</li>
{% empty %}
    <li>Статьи отсутствуют.</li>
{% endfor %}
</ul>

<!-- Переменные цикла -->
{% for item in items %}
    {{ forloop.counter }}    <!-- 1-based индекс -->
    {{ forloop.counter0 }}   <!-- 0-based индекс -->
    {{ forloop.revcounter }} <!-- Обратный счёт -->
    {{ forloop.first }}      <!-- True для первого -->
    {{ forloop.last }}       <!-- True для последнего -->
{% endfor %}

<!-- URL -->
<a href="{% url 'article_detail' article.id %}">Читать</a>
<a href="{% url 'app_name:view_name' %}">Ссылка</a>

<!-- CSRF токен для форм -->
<form method="post">
    {% csrf_token %}
    <!-- поля формы -->
</form>

<!-- Комментарии -->
{% comment "Это многострочный комментарий" %}
    Этот текст не будет отображён
{% endcomment %}

{# Это однострочный комментарий #}
```

#### **Теги для работы с наследованием:**
```html
{% extends 'base.html' %}
{% block content %}...{% endblock %}
{% include 'partial.html' %}
```

#### **Пользовательские теги:**
```python
# myapp/templatetags/my_tags.py
from django import template
register = template.Library()

@register.simple_tag
def current_time(format_string):
    return datetime.now().strftime(format_string)
```

```html
{% load my_tags %}
<p>Сейчас: {% current_time "%Y-%m-%d %H:%M:%S" %}</p>
```

---

### **3. Фильтры (Filters)**

**Назначение:** Преобразование значений переменных перед выводом.

#### **Основные фильтры:**

```html
<!-- Текст -->
<p>{{ text|truncatewords:30 }}</p>          <!-- Обрезать до 30 слов -->
<p>{{ text|truncatechars:100 }}</p>         <!-- Обрезать до 100 символов -->
<p>{{ text|linebreaks }}</p>                <!-- Заменить \n на <br> -->
<p>{{ text|striptags }}</p>                 <!-- Удалить HTML теги -->
<p>{{ text|safe }}</p>                      <!-- Отключить экранирование -->
<p>{{ text|escape }}</p>                    <!-- Экранировать HTML -->

<!-- Даты -->
<p>{{ article.published_date|date:"d.m.Y" }}</p>      <!-- 15.03.2024 -->
<p>{{ article.published_date|time:"H:i" }}</p>        <!-- 14:30 -->
<p>{{ article.published_date|timesince }}</p>         <!-- 3 дня, 2 часа -->

<!-- Числа -->
<p>{{ price|floatformat:2 }}</p>            <!-- 123.45 -->
<p>{{ number|filesizeformat }}</p>          <!-- 2.5 MB -->

<!-- Списки -->
<p>{{ list|join:", " }}</p>                  <!-- item1, item2, item3 -->
<p>{{ list|first }}</p>                      <!-- Первый элемент -->
<p>{{ list|last }}</p>                       <!-- Последний элемент -->
<p>{{ list|length }}</p>                     <!-- Длина списка -->
<p>{{ list|random }}</p>                     <!-- Случайный элемент -->

<!-- Строки -->
<p>{{ title|title }}</p>                     <!-- Заглавные Буквы -->
<p>{{ title|upper }}</p>                     <!-- ВЕРХНИЙ РЕГИСТР -->
<p>{{ title|lower }}</p>                     <!-- нижний регистр -->
<p>{{ url|urlencode }}</p>                   <!-- Кодирование для URL -->
<p>{{ string|default:"Не указано" }}</p>     <!-- Значение по умолчанию -->

<!-- Логические -->
<p>{{ value|yesno:"Да,Нет,Возможно" }}</p>   <!-- True/False/None -->
```

#### **Цепочка фильтров:**
```html
<p>{{ text|truncatewords:50|linebreaks|safe }}</p>
<!-- Порядок важен! -->
```

#### **Пользовательские фильтры:**
```python
# myapp/templatetags/my_filters.py
from django import template
register = template.Library()

@register.filter
def multiply(value, arg):
    return value * arg

@register.filter(name='currency')
def currency_format(value):
    return f"${value:.2f}"
```

```html
{% load my_filters %}
<p>Цена: {{ price|currency }}</p>
<p>Удвоенное: {{ number|multiply:2 }}</p>
```

---

### **Сравнение элементов:**

| **Критерий** | **Переменные** | **Теги** | **Фильтры** |
|--------------|----------------|----------|-------------|
| **Синтаксис** | `{{ }}` | `{% %}` | `{{ variable|filter }}` |
| **Назначение** | Вывод данных | Логика | Преобразование данных |
| **Аргументы** | Нет | Есть | Могут принимать |
| **Результат** | Значение | Действие | Новое значение |
| **Пример** | `{{ title }}` | `{% if %}` | `{{ title|upper }}` |

---

### **Автоэкранирование и безопасность:**

```html
<!-- По умолчанию: экранирование включено -->
<p>{{ user_input }}</p>  <!-- <script> -> &lt;script&gt; -->

<!-- Явное отключение (осторожно!) -->
<p>{{ html_content|safe }}</p>

<!-- Условное экранирование -->
{% autoescape on %}
    {{ untrusted_data }}  <!-- Экранируется -->
{% endautoescape %}

{% autoescape off %}
    {{ trusted_html }}    <!-- Не экранируется -->
{% endautoescape %}
```

---

### **Контекстные процессоры:**

```python
# Добавляют переменные во все шаблоны
def user_context(request):
    return {
        'site_name': 'Мой сайт',
        'current_year': datetime.now().year,
    }
```

```html
<!-- Доступно во всех шаблонах -->
<title>{{ site_name }}</title>
<footer>&copy; {{ current_year }}</footer>
```

---
## **13. Опишите жизненный цикл Django-формы: от момента создания в представлении до сохранения данных. Какие основные этапы обработки проходит форма?**
### **Полный жизненный цикл Django Form:**

```
1. Создание формы (GET-запрос)
   ├── Инициализация формы
   ├── Рендеринг в шаблоне
   └── Отображение пользователю

2. Обработка данных (POST-запрос)
   ├── Валидация данных
   ├── Обработка валидных данных
   ├── Сохранение в БД (для ModelForm)
   └── Редирект или повторное отображение
```

---

### **Этап 1: Инициализация формы (GET-запрос)**

```python
# views.py
from django import forms
from .models import Article
from django.shortcuts import render, redirect

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content', 'category']
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10}),
        }

def create_article(request):
    # 1. Создание пустой формы для GET-запроса
    if request.method == 'GET':
        form = ArticleForm()
        
        # Контекст с формой для шаблона
        context = {
            'form': form,
            'title': 'Создание статьи'
        }
        return render(request, 'articles/form.html', context)
```

---

### **Этап 2: Рендеринг формы в шаблоне**

```html
<!-- templates/articles/form.html -->
<form method="post">
    {% csrf_token %}
    
    <!-- Автоматический рендеринг всех полей -->
    {{ form.as_p }}
    
    <!-- Или ручной рендеринг -->
    <div class="field">
        {{ form.title.label_tag }}
        {{ form.title }}
        {% if form.title.errors %}
            <div class="error">{{ form.title.errors }}</div>
        {% endif %}
    </div>
    
    <button type="submit">Сохранить</button>
</form>
```

---

### **Этап 3: Обработка POST-запроса**

```python
def create_article(request):
    if request.method == 'POST':
        # 1. Создание формы с данными из request.POST
        form = ArticleForm(request.POST)
        
        # 2. Валидация данных (самый важный этап)
        if form.is_valid():
            # 3. Обработка валидных данных
            
            # Для ModelForm:
            article = form.save(commit=False)  # Создаем объект, но не сохраняем в БД
            article.author = request.user      # Добавляем дополнительные данные
            article.published_date = timezone.now()
            article.save()                     # Сохраняем в БД
            
            # Для обычной Form:
            # cleaned_data = form.cleaned_data
            # Article.objects.create(
            #     title=cleaned_data['title'],
            #     content=cleaned_data['content']
            # )
            
            # 4. Редирект после успешного сохранения
            return redirect('article_detail', pk=article.pk)
        else:
            # 5. Если данные невалидны - повторное отображение формы с ошибками
            context = {
                'form': form,
                'title': 'Создание статьи'
            }
            return render(request, 'articles/form.html', context)
```

---

### **Детализация процесса валидации:**

```python
# Процесс валидации form.is_valid():
# 1. to_python() - конвертация в Python-тип
# 2. validate() - проверка типа
# 3. run_validators() - запуск всех валидаторов
# 4. clean_<fieldname>() - полевая очистка
# 5. clean() - межполевая валидация
```

---

### **Кастомная валидация:**

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content', 'category']
    
    # Полевая валидация
    def clean_title(self):
        title = self.cleaned_data.get('title')
        if len(title) < 10:
            raise forms.ValidationError(
                "Заголовок должен содержать минимум 10 символов"
            )
        return title
    
    # Межполевая валидация
    def clean(self):
        cleaned_data = super().clean()
        title = cleaned_data.get('title')
        content = cleaned_data.get('content')
        
        if title and content:
            if title.lower() in content.lower():
                raise forms.ValidationError(
                    "Заголовок не должен дублироваться в содержимом"
                )
        
        return cleaned_data
```

---

### **Этап 4: Обработка файлов (File Upload)**

```python
class ArticleWithImageForm(forms.ModelForm):
    image = forms.ImageField(required=False)
    
    class Meta:
        model = Article
        fields = ['title', 'content', 'image']
    
    def save(self, commit=True):
        instance = super().save(commit=False)
        
        # Обработка загруженного файла
        if self.cleaned_data.get('image'):
            instance.image = self.cleaned_data['image']
        
        if commit:
            instance.save()
        return instance

def create_article_with_image(request):
    if request.method == 'POST':
        # Важно: request.FILES для загрузки файлов
        form = ArticleWithImageForm(
            request.POST, 
            request.FILES  # ← Обработка файлов
        )
        
        if form.is_valid():
            article = form.save()
            return redirect('article_detail', pk=article.pk)
```

```html
<!-- В шаблоне важно добавить enctype -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Загрузить</button>
</form>
```

---

### **Этап 5: Редактирование существующего объекта**

```python
def edit_article(request, pk):
    article = get_object_or_404(Article, pk=pk)
    
    if request.method == 'POST':
        # Передаем instance для редактирования
        form = ArticleForm(
            request.POST, 
            instance=article  # ← Редактирование существующего
        )
        
        if form.is_valid():
            form.save()
            return redirect('article_detail', pk=article.pk)
    else:
        # Предзаполнение формы данными объекта
        form = ArticleForm(instance=article)
    
    return render(request, 'articles/form.html', {'form': form})
```

---

### **Промежуточные состояния формы:**

| **Состояние** | `is_valid()` | `cleaned_data` | `errors` |
|---------------|--------------|----------------|----------|
| **Новая (пустая)** | `False` | `{}` | `{}` |
| **С ошибками валидации** | `False` | Частично заполнен | Содержит ошибки |
| **Валидная** | `True` | Полностью заполнен | `{}` |
| **Привязанная (bound)** | Зависит | Зависит | Зависит |

---

### **Упрощенный шаблон обработки:**

```python
def form_view(request, pk=None):
    # Получение объекта (для редактирования)
    instance = None
    if pk:
        instance = get_object_or_404(Article, pk=pk)
    
    if request.method == 'POST':
        form = ArticleForm(request.POST, instance=instance)
        if form.is_valid():
            form.save()
            return redirect('success_url')
    else:
        form = ArticleForm(instance=instance)
    
    return render(request, 'template.html', {'form': form})
```

---

### **Class-Based View для форм:**

```python
from django.views.generic import CreateView, UpdateView

class ArticleCreateView(CreateView):
    model = Article
    form_class = ArticleForm
    template_name = 'articles/form.html'
    success_url = '/articles/'
    
    def form_valid(self, form):
        # Дополнительная логика перед сохранением
        form.instance.author = self.request.user
        return super().form_valid(form)

class ArticleUpdateView(UpdateView):
    model = Article
    form_class = ArticleForm
    template_name = 'articles/form.html'
    
    def get_success_url(self):
        return reverse('article_detail', args=[self.object.pk])
```

---

### **Жизненный цикл в виде схемы:**

```
[ Пользователь открывает страницу ]
         ↓
[ Django создает пустую форму ]
         ↓
[ Форма рендерится в HTML ]
         ↓
[ Пользователь заполняет и отправляет ]
         ↓
[ Django создает bound-форму с данными ]
         ↓
[ Валидация: полевая → межполевая ]
         ↓
       ┌─────────────────────┐
       │ Данные валидны?     │
       └──────────┬──────────┘
                 ↓
       ┌─────────┴──────────┐
      Да                    Нет
       ↓                    ↓
[ Обработка данных ]  [ Показать ошибки ]
       ↓                    │
[ Сохранение в БД ]         │
       ↓                    │
[ Редирект на успех ] ←─────┘
```

---

## **14. В чём разница между классами Form и ModelForm? В какой ситуации целесообразно использовать каждый из них?**

### **Сравнительная таблица:**

| **Критерий** | **Form** | **ModelForm** |
|-------------|----------|---------------|
| **Назначение** | Общая форма, не привязанная к модели | Форма, привязанная к модели Django |
| **Создание полей** | Ручное определение каждого поля | Автоматическая генерация из модели |
| **Связь с моделью** | Нет прямой связи | Прямая связь через `Meta.model` |
| **Сохранение данных** | Ручная обработка `cleaned_data` | Автоматическое через `save()` |
| **Валидация** | Кастомные правила | + Валидация на основе модели |
| **Использование** | Контактные формы, поиск, логин | CRUD операции с моделями |

---

### **1. Django Form (базовая форма)**

**Когда использовать:**
- Формы, не связанные с моделями (поиск, контакты, логин)
- Сложные формы с вычисляемыми полями
- Формы, объединяющие данные из нескольких моделей

**Пример:**

```python
from django import forms

class ContactForm(forms.Form):
    # Ручное определение всех полей
    name = forms.CharField(
        max_length=100,
        label='Ваше имя',
        widget=forms.TextInput(attrs={'placeholder': 'Введите имя'})
    )
    email = forms.EmailField(
        label='Email',
        help_text='Мы отправим ответ на этот адрес'
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={'rows': 5}),
        label='Сообщение'
    )
    subscribe = forms.BooleanField(
        required=False,
        label='Подписаться на рассылку'
    )
    
    # Кастомная валидация
    def clean_email(self):
        email = self.cleaned_data.get('email')
        if 'spam' in email:
            raise forms.ValidationError("Email содержит запрещённое слово")
        return email

# Использование в представлении
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Ручная обработка данных
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']
            
            # Отправка email, сохранение в БД и т.д.
            send_contact_email(name, email, message)
            
            return redirect('thank_you')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})
```

---

### **2. Django ModelForm (форма на основе модели)**

**Когда использовать:**
- Создание, редактирование, удаление объектов моделей
- Когда форма напрямую соответствует полям модели
- Административные интерфейсы (админка Django использует ModelForm)

**Пример:**

```python
from django import forms
from .models import Article, Author

class ArticleForm(forms.ModelForm):
    # Автоматическая генерация полей из модели
    class Meta:
        model = Article  # Связь с моделью
        fields = ['title', 'content', 'author', 'category']
        # Или: exclude = ['created_at', 'updated_at']
        # Или: fields = '__all__' (все поля)
        
        # Кастомизация полей
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10, 'class': 'rich-text'}),
            'published_date': forms.DateInput(attrs={'type': 'date'}),
        }
        
        labels = {
            'title': 'Заголовок статьи',
            'content': 'Текст статьи',
        }
        
        help_texts = {
            'title': 'Не более 200 символов',
        }
    
    # Дополнительные поля, не из модели
    tags = forms.CharField(
        required=False,
        help_text='Введите теги через запятую'
    )
    
    def save(self, commit=True):
        # Переопределение сохранения
        instance = super().save(commit=False)
        
        # Обработка дополнительных полей
        if self.cleaned_data.get('tags'):
            instance.tags_list = self.cleaned_data['tags'].split(',')
        
        if commit:
            instance.save()
            self.save_m2m()  # Для ManyToMany полей
        
        return instance

# Использование
def create_article(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            # Автоматическое сохранение в модель
            article = form.save()  # Создает объект Article в БД
            return redirect('article_detail', pk=article.pk)
```

---

### **3. Form vs ModelForm: практические примеры**

**Ситуация 1: Регистрация пользователя (Form)**
```python
class RegistrationForm(forms.Form):
    username = forms.CharField(max_length=150)
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    password_confirm = forms.CharField(widget=forms.PasswordInput)
    
    def clean(self):
        # Проверка совпадения паролей
        if self.cleaned_data.get('password') != self.cleaned_data.get('password_confirm'):
            raise forms.ValidationError("Пароли не совпадают")
        
        # Проверка уникальности username
        if User.objects.filter(username=self.cleaned_data.get('username')).exists():
            raise forms.ValidationError("Пользователь с таким именем уже существует")
```

**Ситуация 2: Профиль пользователя (ModelForm)**
```python
class UserProfileForm(forms.ModelForm):
    class Meta:
        model = UserProfile
        fields = ['bio', 'avatar', 'birth_date', 'website']
    
    # Поле из связанной модели User
    email = forms.EmailField()
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Предзаполнение email из связанного User
        if self.instance and self.instance.user:
            self.fields['email'].initial = self.instance.user.email
    
    def save(self, commit=True):
        profile = super().save(commit=False)
        # Обновление email в модели User
        if self.cleaned_data.get('email'):
            profile.user.email = self.cleaned_data['email']
            profile.user.save()
        
        if commit:
            profile.save()
        
        return profile
```

---

### **4. Кастомизация ModelForm**

```python
class CustomModelForm(forms.ModelForm):
    class Meta:
        model = MyModel
        
    def __init__(self, *args, **kwargs):
        # Передача дополнительных аргументов
        self.user = kwargs.pop('user', None)
        super().__init__(*args, **kwargs)
        
        # Динамическое изменение полей
        if self.user and not self.user.is_staff:
            # Скрыть поле для не-админов
            self.fields['secret_field'].widget = forms.HiddenInput()
        
        # Добавление CSS классов
        for field_name, field in self.fields.items():
            field.widget.attrs['class'] = 'form-control'
            
            if field_name == 'title':
                field.widget.attrs['placeholder'] = 'Введите заголовок'
```

---

### **5. Выбор подхода: рекомендации**

| **Ситуация** | **Рекомендация** | **Обоснование** |
|--------------|------------------|-----------------|
| Создание объекта модели | **ModelForm** | Автоматическая генерация полей, валидация, save() |
| Редактирование объекта | **ModelForm** | Предзаполнение через `instance`, простое обновление |
| Поиск по сайту | **Form** | Не связан с моделью, простые поля |
| Многостраничная форма | **Form** | Контроль над потоком данных |
| Форма с вычислениями | **Form** | Сложная логика обработки |
| Админ-интерфейс | **ModelForm** | Интеграция с Django Admin |
| Объединение нескольких моделей | **Form** или несколько **ModelForm** | Гибкость композиции |

---

### **6. Гибридный подход**

```python
class OrderForm(forms.Form):
    # Поля из модели Order
    shipping_address = forms.CharField(max_length=255)
    payment_method = forms.ChoiceField(choices=PAYMENT_METHODS)
    
    # Поля из модели OrderItem (связанная модель)
    product = forms.ModelChoiceField(queryset=Product.objects.all())
    quantity = forms.IntegerField(min_value=1)
    
    def save(self, user):
        # Создание Order
        order = Order.objects.create(
            user=user,
            shipping_address=self.cleaned_data['shipping_address'],
            payment_method=self.cleaned_data['payment_method'],
        )
        
        # Создание OrderItem
        OrderItem.objects.create(
            order=order,
            product=self.cleaned_data['product'],
            quantity=self.cleaned_data['quantity'],
            price=self.cleaned_data['product'].price,
        )
        
        return order
```

---

## **15. Какую основную проблему решает сериализатор (Serializer) в Django REST Framework? Опишите его две ключевые функции.**

### **Основная проблема:**

**Преобразование сложных типов данных** (модели Django, QuerySets) в **нативные типы Python**, которые можно легко преобразовать в JSON/XML, и наоборот.

**Без сериализатора:**
```python
def api_view(request):
    articles = Article.objects.all()
    # Ручное преобразование
    data = []
    for article in articles:
        data.append({
            'id': article.id,
            'title': article.title,
            'content': article.content,
            'author': article.author.username,  # Проблема: связанные объекты
            'comments_count': article.comments.count(),  # Проблема: агрегация
        })
    return JsonResponse(data, safe=False)
```

**Проблемы ручного подхода:**
- Трудоемкость для сложных объектов
- Нет стандартизации
- Сложность валидации входящих данных
- Нет поддержки вложенных отношений

---

### **Две ключевые функции Serializer:**

#### **1. Сериализация (Serialization)**
**Преобразование объектов Django/QuerySets → Python datatypes → JSON/XML**

```python
from rest_framework import serializers
from .models import Article, Comment

class ArticleSerializer(serializers.ModelSerializer):
    # Дополнительные поля (read-only)
    comments_count = serializers.SerializerMethodField()
    author_name = serializers.CharField(source='author.username', read_only=True)
    
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author', 'author_name', 
                 'published_date', 'comments_count']
    
    def get_comments_count(self, obj):
        return obj.comments.count()

# Использование
article = Article.objects.get(pk=1)
serializer = ArticleSerializer(article)
print(serializer.data)
# {'id': 1, 'title': '...', 'content': '...', 'author': 1, 
#  'author_name': 'john', 'comments_count': 5}
```

#### **2. Десериализация (Deserialization)**
**Преобразование входящих данных (JSON/XML) → Python datatypes → Валидация → Объекты Django**

```python
class ArticleCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['title', 'content', 'category']
    
    def validate_title(self, value):
        """Полевая валидация"""
        if len(value) < 10:
            raise serializers.ValidationError(
                "Заголовок должен содержать минимум 10 символов"
            )
        return value
    
    def validate(self, data):
        """Межполевая валидация"""
        if data['title'].lower() in data['content'].lower():
            raise serializers.ValidationError(
                "Заголовок не должен дублироваться в содержании"
            )
        return data
    
    def create(self, validated_data):
        """Создание объекта"""
        # Добавление текущего пользователя
        validated_data['author'] = self.context['request'].user
        return super().create(validated_data)

# Использование
data = {'title': 'Новая статья', 'content': 'Текст...', 'category': 1}
serializer = ArticleCreateSerializer(data=data, context={'request': request})

if serializer.is_valid():
    article = serializer.save()  # Создает объект в БД
    return Response(serializer.data, status=201)
else:
    return Response(serializer.errors, status=400)
```

---

### **Типы сериализаторов в DRF:**

| **Тип** | **Назначение** | **Пример** |
|---------|----------------|------------|
| **Serializer** | Базовый сериализатор | Для не-модельных данных |
| **ModelSerializer** | Для моделей Django | Автогенерация полей |
| **HyperlinkedModelSerializer** | С гиперссылками | Для HATEOAS API |
| **ListSerializer** | Для списков объектов | bulk operations |

---

### **Практические примеры:**

#### **Пример 1: Вложенные сериализаторы**
```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ['id', 'text', 'author', 'created_at']

class ArticleDetailSerializer(serializers.ModelSerializer):
    # Вложенное представление комментариев
    comments = CommentSerializer(many=True, read_only=True)
    
    # Поле-источник
    category_name = serializers.CharField(source='category.name')
    
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author', 'category', 
                 'category_name', 'comments', 'published_date']
```

#### **Пример 2: Кастомные поля и валидация**
```python
class UserRegistrationSerializer(serializers.Serializer):
    email = serializers.EmailField()
    password = serializers.CharField(write_only=True, min_length=8)
    password_confirm = serializers.CharField(write_only=True)
    
    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError("Email уже зарегистрирован")
        return value
    
    def validate(self, data):
        if data['password'] != data['password_confirm']:
            raise serializers.ValidationError({
                'password_confirm': 'Пароли не совпадают'
            })
        return data
    
    def create(self, validated_data):
        user = User.objects.create_user(
            email=validated_data['email'],
            password=validated_data['password']
        )
        return user
```

#### **Пример 3: Динамические поля**
```python
class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author']
    
    def __init__(self, *args, **kwargs):
        # Получение параметра из контекста
        fields = kwargs.pop('fields', None)
        super().__init__(*args, **kwargs)
        
        # Динамическое ограничение полей
        if fields:
            allowed = set(fields)
            existing = set(self.fields.keys())
            for field_name in existing - allowed:
                self.fields.pop(field_name)

# Использование
serializer = ArticleSerializer(article, fields=['id', 'title'])
```

---

### **Методы сериализатора:**

| **Метод** | **Назначение** | **Пример** |
|-----------|----------------|------------|
| `is_valid()` | Проверка валидности данных | `serializer.is_valid()` |
| `validated_data` | Валидированные данные | `serializer.validated_data` |
| `save()` | Сохранение объекта | `serializer.save()` |
| `create()` | Создание нового объекта | Определение в сериализаторе |
| `update()` | Обновление существующего | Определение в сериализаторе |
| `to_representation()` | Кастомизация вывода | Переопределение в сериализаторе |
| `to_internal_value()` | Кастомизация ввода | Переопределение в сериализаторе |

---

### **Проблемы, которые решает сериализатор:**

1. **Сложные преобразования данных**
   ```python
   # Автоматическая обработка ForeignKey, ManyToMany
   # Преобразование дат в строки
   # Обработка FileField, ImageField
   ```

2. **Валидация на разных уровнях**
   ```python
   # Полевая валидация
   # Межполевая валидация
   # Валидация уникальности
   # Валидация связанных объектов
   ```

3. **Гибкое управление выводом**
   ```python
   # Разные сериализаторы для разных представлений
   # Динамическое включение/исключение полей
   # Вложенные представления
   # Гиперссылки (HATEOAS)
   ```

4. **Безопасность**
   ```python
   # Автоматическое экранирование
   # Контроль доступа к полям (read_only, write_only)
   # Валидация входящих данных
   ```

---

## **16. В чём заключается идея и преимущество использования ViewSet и Router в DRF по сравнению с написанием отдельных представлений для каждой операции API?**

### **Проблема традиционного подхода:**

**Без ViewSet и Router (классический подход):**
```python
# urls.py
urlpatterns = [
    path('articles/', ArticleListView.as_view(), name='article-list'),
    path('articles/create/', ArticleCreateView.as_view(), name='article-create'),
    path('articles/<int:pk>/', ArticleDetailView.as_view(), name='article-detail'),
    path('articles/<int:pk>/update/', ArticleUpdateView.as_view(), name='article-update'),
    path('articles/<int:pk>/delete/', ArticleDeleteView.as_view(), name='article-delete'),
]

# views.py
class ArticleListView(APIView):
    def get(self, request):
        articles = Article.objects.all()
        serializer = ArticleSerializer(articles, many=True)
        return Response(serializer.data)

class ArticleCreateView(APIView):
    def post(self, request):
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)

class ArticleDetailView(APIView):
    def get(self, request, pk):
        article = get_object_or_404(Article, pk=pk)
        serializer = ArticleSerializer(article)
        return Response(serializer.data)

# ... и так для каждого метода/операции
```

**Проблемы:**
- Много повторяющегося кода
- Сложность поддержки
- Ручная регистрация каждого URL
- Нет стандартизации

---

### **Решение: ViewSet + Router**

#### **1. ViewSet - объединение операций**

**ViewSet** - класс, который объединяет логику для набора связанных представлений (list, create, retrieve, update, partial_update, destroy).

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response

class ArticleViewSet(viewsets.ModelViewSet):
    """
    ViewSet для статей со всеми CRUD операциями
    Автоматически предоставляет:
    - list (GET /articles/)
    - create (POST /articles/)
    - retrieve (GET /articles/{id}/)
    - update (PUT /articles/{id}/)
    - partial_update (PATCH /articles/{id}/)
    - destroy (DELETE /articles/{id}/)
    """
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    
    # Дополнительные настройки
    permission_classes = [IsAuthenticatedOrReadOnly]
    filter_backends = [DjangoFilterBackend, SearchFilter]
    filterset_fields = ['category', 'author']
    search_fields = ['title', 'content']
    
    # Кастомные действия (custom actions)
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        """Публикация статьи"""
        article = self.get_object()
        article.is_published = True
        article.save()
        serializer = self.get_serializer(article)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def recent(self, request):
        """Последние опубликованные статьи"""
        recent_articles = Article.objects.filter(
            is_published=True
        ).order_by('-published_date')[:10]
        serializer = self.get_serializer(recent_articles, many=True)
        return Response(serializer.data)
    
    # Переопределение стандартных методов
    def perform_create(self, serializer):
        """Дополнительная логика при создании"""
        serializer.save(author=self.request.user)
    
    def get_queryset(self):
        """Кастомный queryset"""
        queryset = super().get_queryset()
        if not self.request.user.is_staff:
            queryset = queryset.filter(is_published=True)
        return queryset
```

#### **2. Router - автоматическая маршрутизация**

**Router** автоматически генерирует URL patterns для ViewSet.

```python
# urls.py
from rest_framework.routers import DefaultRouter
from .views import ArticleViewSet, CommentViewSet

router = DefaultRouter()
router.register(r'articles', ArticleViewSet, basename='article')
router.register(r'comments', CommentViewSet, basename='comment')

urlpatterns = router.urls

# Автоматически создаются URL:
# /articles/              - список статей (GET)
# /articles/              - создание статьи (POST)
# /articles/{pk}/         - детали статьи (GET)
# /articles/{pk}/         - обновление статьи (PUT)
# /articles/{pk}/         - частичное обновление (PATCH)
# /articles/{pk}/         - удаление статьи (DELETE)
# /articles/{pk}/publish/ - кастомное действие (POST)
# /articles/recent/       - кастомное действие (GET)
```

---

### **Преимущества подхода:**

| **Преимущество** | **Описание** | **Пример** |
|------------------|--------------|------------|
| **Меньше кода** | Автоматизация CRUD операций | 1 ViewSet вместо 6 View классов |
| **Стандартизация** | Единый стиль API endpoints | RESTful по умолчанию |
| **Автоматическая документация** | Генерация схемы OpenAPI | Поддержка drf-yasg, spectacular |
| **Гибкость** | Кастомные действия через `@action` | Любые дополнительные endpoints |
| **Простота маршрутизации** | Автоматическое создание URL | `router.register()` |
| **Встроенные возможности** | Пагинация, фильтрация, поиск | `filter_backends`, `pagination_class` |

---

### **Типы ViewSet:**

| **Тип ViewSet** | **Предоставляет** | **Использование** |
|-----------------|-------------------|-------------------|
| **ModelViewSet** | Все CRUD операции | Полное управление моделью |
| **ReadOnlyModelViewSet** | Только чтение (list, retrieve) | Публичный доступ |
| **ViewSet** | Базовый класс без предопределенных действий | Полная кастомизация |
| **GenericViewSet** | Наследует от GenericAPIView | Mixins + кастомные действия |

---

### **Пример с несколькими ViewSet:**

```python
# views.py
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    
    @action(detail=True, methods=['get'])
    def comments(self, request, pk=None):
        """Комментарии к статье"""
        article = self.get_object()
        comments = article.comments.all()
        serializer = CommentSerializer(comments, many=True)
        return Response(serializer.data)

class CommentViewSet(viewsets.ModelViewSet):
    queryset = Comment.objects.all()
    serializer_class = CommentSerializer
    
    def get_queryset(self):
        # Фильтрация по статье, если передан article_pk
        queryset = super().get_queryset()
        article_pk = self.kwargs.get('article_pk')
        if article_pk:
            queryset = queryset.filter(article_id=article_pk)
        return queryset
    
    def perform_create(self, serializer):
        # Автоматическое связывание со статьей
        article_pk = self.kwargs.get('article_pk')
        if article_pk:
            article = Article.objects.get(pk=article_pk)
            serializer.save(article=article, author=self.request.user)
```

```python
# urls.py - вложенные роутеры
from rest_framework_nested import routers

router = routers.DefaultRouter()
router.register(r'articles', ArticleViewSet, basename='article')

# Вложенный роутер для комментариев
articles_router = routers.NestedDefaultRouter(
    router, r'articles', lookup='article'
)
articles_router.register(
    r'comments', CommentViewSet, basename='article-comments'
)

urlpatterns = [
    path('api/', include(router.urls)),
    path('api/', include(articles_router.urls)),
]

# Создаются URL:
# /api/articles/              - статьи
# /api/articles/{article_pk}/comments/  - комментарии статьи
# /api/articles/{article_pk}/comments/{pk}/ - конкретный комментарий
```

---

### **Кастомизация Router:**

```python
# Кастомный Router с дополнительными путями
from rest_framework.routers import Route, DynamicRoute, SimpleRouter

class CustomRouter(SimpleRouter):
    """
    Кастомный router с дополнительными действиями
    """
    routes = [
        # Стандартные маршруты
        Route(
            url=r'^{prefix}/$',
            mapping={'get': 'list', 'post': 'create'},
            name='{basename}-list',
            detail=False,
            initkwargs={'suffix': 'List'}
        ),
        Route(
            url=r'^{prefix}/{lookup}/$',
            mapping={
                'get': 'retrieve',
                'put': 'update',
                'patch': 'partial_update',
                'delete': 'destroy'
            },
            name='{basename}-detail',
            detail=True,
            initkwargs={'suffix': 'Instance'}
        ),
        # Динамические маршруты для кастомных действий
        DynamicRoute(
            url=r'^{prefix}/{lookup}/{url_path}/$',
            name='{basename}-{url_name}',
            detail=True,
            initkwargs={}
        ),
        # Дополнительный маршрут для действий без pk
        DynamicRoute(
            url=r'^{prefix}/{url_path}/$',
            name='{basename}-{url_name}',
            detail=False,
            initkwargs={}
        ),
    ]

# Использование
router = CustomRouter()
router.register(r'articles', ArticleViewSet)
```

---

### **Сравнение подходов:**

**До ViewSet:**
```python
# 5+ классов View
# 5+ URL patterns
# Много повторяющегося кода
# Разная реализация одинаковых операций
```

**После ViewSet:**
```python
# 1 класс ViewSet
# Автоматические URL
# Единая логика
# Стандартизированный API
```

---

### **Дополнительные возможности:**

1. **Базовые классы для permission, authentication:**
```python
class ArticleViewSet(viewsets.ModelViewSet):
    authentication_classes = [TokenAuthentication, SessionAuthentication]
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    throttle_classes = [UserRateThrottle]
```

2. **Пагинация:**
```python
class LargeResultsSetPagination(PageNumberPagination):
    page_size = 100
    page_size_query_param = 'page_size'
    max_page_size = 1000

class ArticleViewSet(viewsets.ModelViewSet):
    pagination_class = LargeResultsSetPagination
```

3. **Фильтрация и поиск:**
```python
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters

class ArticleViewSet(viewsets.ModelViewSet):
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['category', 'author', 'is_published']
    search_fields = ['title', 'content', 'author__username']
    ordering_fields = ['published_date', 'title', 'views']
    ordering = ['-published_date']  # сортировка по умолчанию
```

Извините за недоразумение! Я отвечал на все 20 вопросов, но форматирование могло сбить. Вот вопросы **17-20** в том же красивом markdown формате, что и предыдущие:

## **17. Объясните концепцию аутентификации на основе JWT-токенов. Каковы его основные преимущества для API перед традиционной сессионной аутентификацией?**

### **Концепция JWT-аутентификации:**
**JWT (JSON Web Token)** — стандарт для создания токенов доступа, состоящих из трех частей: заголовка, полезной нагрузки (payload) и подписи. Токен самодостаточен и содержит всю информацию для аутентификации.

### **Как работает:**
1. **Логин**: Пользователь отправляет учетные данные → сервер проверяет и генерирует JWT
2. **Хранение**: Клиент сохраняет токен (localStorage/cookies)
3. **Использование**: Клиент отправляет токен в заголовке `Authorization: Bearer <token>`
4. **Верификация**: Сервер проверяет подпись и срок действия токена

### **Преимущества перед сессионной аутентификацией:**

| **Аспект** | **JWT** | **Сессии** |
|------------|---------|------------|
| **Stateless** | ✅ Сервер не хранит состояние | ❌ Хранит сессии в БД/Redis |
| **Масштабируемость** | ✅ Любой сервер может проверить токен | ❌ Требуется общее хранилище сессий |
| **CORS/мобильные приложения** | ✅ Легко работает | ❌ Проблемы с cookies |
| **Производительность** | ✅ Нет запросов к БД для проверки | ❌ Проверка сессии в хранилище |
| **Микросервисы** | ✅ Идеально подходит | ❌ Сложная интеграция |

### **Недостатки JWT:**
- Невозможность отзыва токена без blacklist
- Больший размер передаваемых данных
- Безопасность зависит от реализации хранения

---

## **18. Какие встроенные механизмы защиты от CSRF и XSS атак предоставляет Django и как они работают на базовом уровне?**

### **Защита от CSRF (Cross-Site Request Forgery):**

**Механизм:**
- `CsrfViewMiddleware` включен по умолчанию
- Генерирует уникальный токен для каждого пользователя
- Токен хранится в cookie и добавляется в формы

**Как работает:**
1. При GET-запросе Django генерирует CSRF-токен
2. Токен сохраняется в cookie и в контексте шаблона
3. В форме добавляется скрытое поле с токеном
4. При POST-запросе сравниваются токены из cookie и формы
5. Несовпадение → ошибка 403 Forbidden

```html
<!-- Автоматически в формах -->
<form method="post">
  {% csrf_token %}
  <!-- ... -->
</form>
```

### **Защита от XSS (Cross-Site Scripting):**

**Механизмы:**
1. **Автоэкранирование в шаблонах**: Все переменные автоматически экранируются
   ```html
   {{ user_input }}  <!-- <script> → &lt;script&gt; -->
   ```

2. **Ручное управление**:
   ```html
   {{ safe_html|safe }}  <!-- Явное отключение экранирования -->
   {% autoescape off %}...{% endautoescape %}
   ```

3. **Валидация форм**: Очистка входящих данных
4. **Заголовки безопасности**: `X-XSS-Protection`, `Content-Security-Policy`

### **Дополнительные защиты:**
- `X_FRAME_OPTIONS = 'DENY'` — защита от clickjacking
- `SECURE_BROWSER_XSS_FILTER = True` — включение фильтра XSS в браузере
- Валидация URL при редиректах

---

## **19. Опишите типичный production-стек для развертывания Django-приложения. Какую роль в нём выполняют веб-сервер (Nginx) и сервер приложений (Gunicorn)?**

### **Типичный стек:**
```
Пользователь → Nginx → Gunicorn → Django → PostgreSQL/Redis
```

### **Роль Nginx (веб-сервер):**
1. **Обратный прокси** — принимает запросы и передает их Gunicorn
2. **Обслуживание статических файлов** — отдает CSS, JS, изображения напрямую
3. **SSL/TLS termination** — обработка HTTPS
4. **Кэширование** — кэширование статики и динамического контента
5. **Балансировка нагрузки** — распределение между экземплярами Gunicorn
6. **Rate limiting** — ограничение частоты запросов

```nginx
# Пример конфигурации
location /static/ {
    alias /path/to/static/files;
    expires 30d;
}

location / {
    proxy_pass http://127.0.0.1:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### **Роль Gunicorn (сервер приложений):**
1. **WSGI-сервер** — запускает Django-приложение
2. **Управление процессами** — создает воркеров для обработки запросов
3. **Балансировка** — распределяет запросы между воркерами
4. **Graceful shutdown/restart** — плавная перезагрузка

```bash
# Запуск Gunicorn
gunicorn myproject.wsgi:application \
  --workers 4 \
  --bind 127.0.0.1:8000 \
  --access-logfile -
```

### **Почему два сервера?**
- **Nginx** эффективен для статики и SSL
- **Gunicorn** оптимизирован для Python/Django
- Разделение ответственности повышает безопасность и производительность

---

## **20. В чём заключаются ключевые преимущества использования Docker для деплоя серверного приложения? Опишите эту концепцию на уровне контейнеризации.**

### **Ключевые преимущества Docker:**

| **Преимущество** | **Описание** | **Пример** |
|------------------|--------------|------------|
| **Консистентность** | Одинаковое окружение везде | На dev, staging, production |
| **Изоляция** | Приложения не мешают друг другу | Отдельные контейнеры для БД, кэша, приложения |
| **Переносимость** | Работает на любой системе с Docker | Локально, на сервере, в облаке |
| **Быстрое развертывание** | Секунды вместо часов | `docker-compose up -d` |
| **Масштабируемость** | Легкое клонирование контейнеров | Горизонтальное масштабирование |
| **Версионирование** | Точное воспроизведение состояний | Разные версии приложения одновременно |

### **Концепция контейнеризации:**

**Контейнер** — изолированное окружение для приложения со всеми зависимостями.

**Отличие от виртуальных машин:**
- **VM**: Полная ОС + гипервизор + приложение (тяжело)
- **Контейнер**: Только приложение + зависимости (легко)

### **Основные компоненты:**
1. **Dockerfile** — инструкция для сборки образа
2. **Image** — неизменяемый шаблон контейнера
3. **Container** — запущенный экземпляр образа
4. **Docker Compose** — оркестрация нескольких контейнеров

### **Пример для Django:**
```dockerfile
# Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "myproject.wsgi:application"]
```

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:15
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
```

### **Рабочий процесс:**
1. Разработка в контейнере
2. Тестирование в изолированном окружении
3. Сборка образа
4. Деплой одной командой

**Итог:** Docker стандартизирует и упрощает весь цикл разработки и деплоя.
