# Паттерны и шаблоны проектирования в проекте MySpringProject

В данном документе описаны архитектурные и программные паттерны (шаблоны), примененные в исходном коде проекта.

## 1. Архитектурные паттерны

### 1.1. Layered Architecture (Слоистая архитектура)
Проект четко разделен на слои, каждый из которых имеет свою ответственность:
- **Controller Layer** (`controller/`): Обрабатывает HTTP-запросы и возвращает ответы.
- **Service Layer** (`service/`): Содержит бизнес-логику.
- **Data Access Layer** (`repository/`): Отвечает за взаимодействие с базой данных.
- **Domain Layer** (`model/`): Представляет сущности базы данных.

### 1.2. MVC (Model-View-Controller)
Spring Boot реализует этот паттерн для веб-взаимодействия.
- **Model**: Сущности в пакете `model` и DTO в пакете `dto`.
- **View**: JSON-ответы REST API (в данном случае View не является HTML-страницей, а представлением данных), а также статический `index.html`.
- **Controller**: Классы в пакете `controller` (`ControllerBook`, `ControllerAuthor` и др.).

### 1.3. REST API
Приложение построено по принципам REST:
- Используются HTTP методы (GET, POST, PUT, DELETE).
- Ресурсы имеют унифицированные URL (`/api/v2/books`).
- Stateless взаимодействие.

---

## 2. Паттерны GoF (Gang of Four)

### 2.1. Data Transfer Object (DTO)
Используется для передачи данных между подсистемами (клиент-сервер) и скрытия внутренней структуры базы данных.
- **Примеры**: `BookCreateDto`, `BookGetDto`, `AuthorUpdateDto`.
- **Где**: Пакет `src/main/java/com/example/myspringproject/dto`.

### 2.2. Repository (Репозиторий)
Абстракция над слоем доступа к данным. Скрывает сложные SQL-запросы за методами интерфейса.
- **Примеры**: `BookRepository`, `AuthorRepository`.
- **Реализация**: Spring Data JPA автоматически создает реализации этих интерфейсов во время выполнения.

### 2.3. Proxy (Заместитель)
Используется Spring Framework повсеместно.
- **AOP (Аспектно-ориентированное программирование)**: `LoggingAspect` создает прокси вокруг сервисов для внедрения логики логирования до и после выполнения методов.
- **@Transactional**: Прокси управляет открытием и закрытием транзакций в `AuthorServiceImpl`.
- **Lazy Loading**: Hibernate использует прокси для связанных сущностей (`fetch = FetchType.LAZY` в `Book.java`).

### 2.4. Singleton (Одиночка)
По умолчанию все бины Spring (`@Service`, `@RestController`, `@Component`, `@Repository`) являются синглтонами. Контейнер Spring создает один экземпляр класса и переиспользует его.
- **Примеры**: `BookServiceImpl`, `BookCache`, `VisitTrackingServiceImpl`.

### 2.5. Strategy (Стратегия)
Используется через внедрение зависимостей (Dependency Injection). Контроллер зависит от интерфейса (`BookService`), а не от конкретной реализации. Это позволяет подменять реализацию (`BookServiceImpl`) без изменения кода контроллера.

### 2.6. Builder (Строитель) / Wither Pattern
В классе `LogTaskInfo` (Java Record) используются методы `withStatus`, `withFilePath`, которые создают копию объекта с измененным полем. Это вариация паттерна Builder для иммутабельных объектов.
- **Где**: `src/main/java/com/example/myspringproject/model/LogTaskInfo.java`

### 2.7. Cache-Aside (Кэширование на стороне)
Паттерн кэширования, реализованный вручную. Приложение сначала проверяет кэш, если данных нет — идет в БД, а затем кладет данные в кэш.
- **Где**: `BookServiceImpl`, `AuthorServiceImpl`.
- **Компоненты**: `BookCache`, `AuthorCache`.

### 2.8. Decorator (Декоратор)
Аспект `LoggingAspect` декорирует методы сервисов, добавляя дополнительное поведение (логирование) без изменения самого кода сервисов.

### 2.9. Future / Promise
Используется для асинхронной обработки. Метод возвращает `CompletableFuture`, который обещает вернуть результат в будущем, не блокируя основной поток.
- **Где**: `LogGenerationServiceImpl.java` (метод `processLogGeneration`).

---

## 3. Шаблоны Enterprise (Корпоративные)

### 3.1. Inversion of Control (IoC) & Dependency Injection (DI)
Главный принцип Spring. Управление созданием объектов и их зависимостей передано контейнеру.
- **Пример**: Внедрение `BookRepository` в `BookServiceImpl` через конструктор (`@AllArgsConstructor`).

### 3.2. Global Exception Handling (Глобальная обработка исключений)
Централизованная обработка ошибок с помощью `@ControllerAdvice`.
- **Где**: `src/main/java/com/example/myspringproject/exception/GlobalExceptionHandler.java`.

### 3.3. Self-Invocation Proxy Pattern
В классе `LogGenerationServiceImpl` используется внедрение бина в самого себя (`private final LogGenerationService self`), чтобы вызовы асинхронных методов внутри того же класса проходили через прокси Spring и аннотация `@Async` срабатывала корректно.

---

## 4. Конкурентные паттерны (Concurrency)

### 4.1. Thread-Safe State
Использование потокобезопасных коллекций и атомарных переменных для хранения состояния в многопоточной среде.
- **Где**: `VisitTrackingServiceImpl.java` использует `ConcurrentHashMap` и `AtomicLong`.
- **Где**: `LogGenerationServiceImpl.java` использует `ConcurrentHashMap`.

### 4.2. Asynchronous Method Invocation
Запуск длительных задач в отдельном пуле потоков.
- **Где**: Аннотация `@Async("taskExecutor")` в `LogGenerationServiceImpl`.

---

## 5. Паттерны на Frontend (React)

### 5.1. Component-Based Architecture
Интерфейс разбит на независимые переиспользуемые компоненты.
- **Пример**: `TableSection.js` используется для отображения таблиц книг, авторов и категорий.

### 5.2. Hooks Pattern
Использование хуков React для управления состоянием и жизненным циклом.
- **Примеры**: `useState`, `useEffect`, `useCallback`, `useMemo` в `App.js`.

### 5.3. Container/Presentational Pattern
В `App.js` содержится логика загрузки данных и состояния (Container), которая передается в `TableSection.js`, отвечающий только за отображение (Presentation).