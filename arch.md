# Архитектурные принципы проекта

## Структура модуля

```
ModuleName/
├── Service/     # Сервисы модуля
├── UI/          # UI компоненты
├── Dto/         # Объекты передачи данных
├── Mediator/    # Межмодульное взаимодействие
├── Event/       # События модуля
├── Configs/     # Конфигурации
├── Model/       # Модели данных
├── Api/         # API клиент
├── Repo/        # Репозитории
└── Net/         # Сетевое взаимодействие
```

## Принципы организации кода

### 1. Разделение сервисов

#### Stateless сервисы
- Содержат только чистую бизнес-логику
- Не хранят состояние
- Используются для конкретных операций
```csharp
[Service]
public class WishlistOperationService {
    public async UniTask<WishlistOperation> Add(string itemId) {
        // Чистые операции без состояния
    }
}
```

#### Stateful сервисы
- Управляют состоянием
- Содержат события для оповещения об изменениях
- Координируют работу stateless сервисов
```csharp
[Service]
public class WishlistService {
    private ServiceState State { get; set; }
    public event Action<WishlistModel>? OnDataChanged;
}
```

### 2. UI и Presenter Pattern

- UI компоненты не содержат бизнес-логику
- Presenter отвечает за подготовку данных для UI
- UI работает только через Presenter
```csharp
[Presenter(ScopeType.SCREEN)]
public class WishlistPresenter {
    private readonly NotificationService _notificationService;
    private readonly WishlistService _wishlistService;
    
    public async UniTask<bool> HandleAdd(string itemId) {
        // Логика подготовки данных для UI
    }
}
```

### 3. Сетевое взаимодействие

- Сетевые команды выделены в отдельные классы
- Используются атрибуты для маркировки сетевых классов
- Четкое разделение клиентских и серверных команд
```csharp
[TransferClass("wl.s.bi")]
public class BoughtItemFromWishlistCommand : IServerCommand {
    [TransferField("cid")]
    public ClothesItemModel Model { get; private set; }
}
```

### 4. Межмодульное взаимодействие

- Mediator паттерн для коммуникации между модулями
- Слабая связанность модулей
- Централизованная обработка межмодульных событий
```csharp
[Mediator]
public class InventoryWishlistMediator : MonoBehaviour, IServiceInitable {
    [Inject]
    private ClothesInventoryService _clothesInventoryService;
    [Inject]
    private WishlistService _wishlistService;
}
```

### 5. Система событий

- События для асинхронной коммуникации
- Типизированные события для каждого модуля
- Константы для именования событий
```csharp
public class WishlistEvent : GameEvent {
    public const string ITEMS_ADDED = "WishlistItemsAdded";
    public const string ITEMS_REMOVED = "WishlistItemsRemoved";
}
```

### 6. Работа с данными

- Репозитории для доступа к данным
- Единый интерфейс для разных типов хранилищ
- Инкапсуляция логики хранения
```csharp
[Repository]
public class WishlistRepo : KeyMemoryRepository<long, WishlistModel> {
    // Работа с данными
}
```

### 7. Конфигурация

- Константы вынесены в отдельные классы
- Четкое разделение конфигурации и логики
```csharp
public static class WishlistConstant {
    public const int WISHLIST_LIMIT = 5;
    public const int WISHLIST_PURCHASE_LIMIT = 5;
}
```

### 8. Dependency Injection

- Использование атрибутов для внедрения зависимостей
- Четкое определение жизненного цикла сервисов
```csharp
[Inject]
public WishlistService(
    WishlistOperationService wishlistOperationService,
    WishlistDispatcher wishlistDispatcher,
    WishlistRepo wishlistRepo) {
    // Инициализация
}
```

## Рекомендации

1. **Сервисы**
   - Разделяйте на stateless и stateful
   - Не смешивайте UI логику с бизнес-логикой
   - Используйте события для оповещения об изменениях

2. **UI**
   - Используйте Presenter для подготовки данных
   - Минимум логики в UI компонентах
   - Обработка ошибок через ErrorProcessService

3. **Сетевое взаимодействие**
   - Отдельные классы для команд
   - Четкое разделение клиент/сервер
   - Валидация данных

4. **Межмодульное взаимодействие**
   - Используйте Mediator
   - Слабая связанность
   - События для асинхронной коммуникации

5. **Данные**
   - Репозитории для доступа к данным
   - DTO для передачи данных
   - Модели для бизнес-логики

## Антипаттерны

1. ❌ Прямое обращение UI к репозиториям
2. ❌ Бизнес-логика в UI компонентах
3. ❌ Сильная связанность между модулями
4. ❌ Хранение состояния в stateless сервисах
5. ❌ Прямая коммуникация между модулями без Mediator 