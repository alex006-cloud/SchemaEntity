# Документация SchemaEntity для CS2

## Оглавление
1. [Введение](#введение)
2. [Основные концепции](#основные-концепции)
3. [Макросы Schema System](#макросы-schema-system)
4. [Работа с сущностями](#работа-с-сущностями)
   - [CBaseEntity - базовый класс](#cbaseentity---базовый-класс)
   - [Иерархия наследования](#иерархия-наследования)
   - [Основные поля](#основные-поля)
   - [Основные методы](#основные-методы)
   - [Подклассы и компоненты](#подклассы-и-компоненты)
   - [Утилиты для поиска сущностей](#утилиты-для-поиска-сущностей)
5. [Игроки и контроллеры](#игроки-и-контроллеры)
   - [Иерархия классов игрока](#иерархия-классов-игрока)
   - [CBasePlayerController](#cbaseplayercontroller)
   - [CCSPlayerController](#ccsplayercontroller)
   - [CBasePlayerPawn](#cbaseplayerpawn)
   - [CCSPlayerPawnBase](#ccsplayerpawnbase)
   - [CCSPlayerPawn](#ccsplayerpawn)
6. [Оружие](#оружие)
   - [Иерархия классов оружия](#иерархия-классов-оружия)
   - [CEconEntity](#ceconentity)
   - [CBasePlayerWeapon](#cbaseplayerweapon)
   - [CCSWeaponBase](#ccsweaponbase)
   - [CCSWeaponBaseVData](#ccsweaponbasevdata)
7. [Сервисы](#сервисы)
   - [CPlayerPawnComponent](#cplayerpawncomponent)
   - [Иерархия сервисов](#иерархия-сервисов)
   - [CPlayer_MovementServices](#cplayer_movementservices)
   - [CCSPlayer_MovementServices](#ccsplayer_movementservices)
   - [CPlayer_WeaponServices](#cplayer_weaponservices)
   - [CCSPlayer_WeaponServices](#ccsplayer_weaponservices)
   - [CPlayer_ItemServices](#cplayer_itemservices)
   - [CCSPlayer_ItemServices](#ccsplayer_itemservices)
   - [CPlayer_CameraServices](#cplayer_cameraservices)
   - [CPlayer_ObserverServices](#cplayer_observerservices)
   - [CCSPlayerController_InGameMoneyServices](#ccsplayercontroller_ingamemoneyservices)
   - [CCSPlayerController_InventoryServices](#ccsplayercontroller_inventoryservices)
   - [CCSPlayerController_ActionTrackingServices](#ccsplayercontroller_actiontrackingservices)
   - [CCSPlayer_ActionTrackingServices](#ccsplayer_actiontrackingservices)
8. [Важные типы и перечисления](#важные-типы-и-перечисления)
   - [MoveType_t](#movetype_t)
   - [LifeState_t](#lifestate_t)
   - [gear_slot_t](#gear_slot_t)
   - [InputBitMask_t](#inputbitmask_t)
   - [DamageTypes_t](#damagetypes_t)
   - [TakeDamageFlags_t](#takedamageflags_t)
   - [ObserverMode_t](#observermode_t)
   - [CSRoundEndReason](#csroundendreason)
   - [GamePhase](#gamephase)
   - [Флаги сущностей](#флаги-сущностей-m_fflags)
   - [Группы коллизий](#группы-коллизий)
9. [Примеры использования](#примеры-использования)
10. [Дополнительные классы](#дополнительные-классы)
11. [Полезные глобальные переменные и утилиты](#полезные-глобальные-переменные-и-утилиты)
12. [Советы и лучшие практики](#советы-и-лучшие-практики)
13. [Частые ошибки](#частые-ошибки)
14. [Заключение](#заключение)

## Введение

SchemaEntity - это система для работы с игровыми сущностями Counter-Strike 2 через схему данных Source 2. Она предоставляет типобезопасный доступ к полям сущностей и автоматическую синхронизацию сетевых переменных.

### Основные файлы

- `schemasystem.h` - Ядро системы, макросы для объявления полей
- `CBaseEntity.h` - Базовый класс всех сущностей
- `CCSPlayerPawn.h` - Класс игрока (pawn)
- `CCSPlayerController.h` - Контроллер игрока
- `CCSWeaponBase.h` - Базовый класс оружия
- `services.h` - Сервисы игрока (движение, оружие, предметы)
- `globaltypes.h` - Глобальные типы и перечисления
- `virtual.h` - Утилиты для вызова виртуальных функций

## Основные концепции

### Schema System

Schema System автоматически находит смещения полей в памяти по их именам, используя метаданные движка Source 2.


### Ключевые преимущества

- **Автоматическое определение смещений**: Не нужно вручную искать адреса
- **Типобезопасность**: Компилятор проверяет типы
- **Сетевая синхронизация**: Автоматическое уведомление об изменениях
- **Удобный синтаксис**: Доступ к полям как к обычным переменным

## Макросы Schema System

### DECLARE_SCHEMA_CLASS(ClassName)

Объявляет класс как часть Schema System. Должен быть первым в теле класса.

```cpp
class CMyEntity : public CBaseEntity
{
public:
    DECLARE_SCHEMA_CLASS(CMyEntity);
    // ... поля ...
};
```

### SCHEMA_FIELD(type, varName)

Объявляет поле схемы со значением (не указатель).

```cpp
SCHEMA_FIELD(int, m_iHealth)        // int
SCHEMA_FIELD(float, m_flSpeed)      // float
SCHEMA_FIELD(Vector, m_vecOrigin)   // Vector
SCHEMA_FIELD(bool, m_bAlive)        // bool
```

**Использование:**
```cpp
// Чтение
int health = pEntity->m_iHealth();

// Запись (автоматически вызывает NetworkStateChanged)
pEntity->m_iHealth(100);

// Или через оператор =
pEntity->m_iHealth = 100;
```


### SCHEMA_FIELD_POINTER(type, varName)

Объявляет поле схемы как указатель.

```cpp
SCHEMA_FIELD_POINTER(CUtlVector<CHandle<CBasePlayerWeapon>>, m_hMyWeapons)
SCHEMA_FIELD_POINTER(char, m_szPlayerName)  // для массивов char
```

**Использование:**
```cpp
// Получить указатель
auto* weapons = pPlayer->m_pWeaponServices->m_hMyWeapons();

// После изменения данных вручную вызвать:
pPlayer->m_pWeaponServices->m_hMyWeapons.NetworkStateChanged();
```

### DECLARE_SCHEMA_CLASS_INLINE(ClassName)

Для вложенных классов (не сущностей), таких как компоненты или структуры данных.

```cpp
class CAttributeList
{
public:
    DECLARE_SCHEMA_CLASS_INLINE(CAttributeList);
    SCHEMA_FIELD(CAttributeManager*, m_pManager);
};
```

## Работа с сущностями

### CBaseEntity - базовый класс

Все игровые объекты наследуются от `CBaseEntity`.

#### Иерархия наследования

```
CEntityInstance (базовый класс движка)
    └── CBaseEntity
        ├── CBaseModelEntity
        │   ├── CBasePlayerController
        │   │   └── CCSPlayerController
        │   ├── CBasePlayerPawn
        │   │   └── CCSPlayerPawnBase
        │   │       └── CCSPlayerPawn
        │   ├── CEconEntity
        │   │   └── CBasePlayerWeapon
        │   │       └── CCSWeaponBase
        │   │           ├── CCSWeaponBaseGun
        │   │           └── CWeaponBaseItem
        │   └── CBaseViewModel
        ├── CPlantedC4
        ├── CChicken
        └── ... (другие сущности)
```

#### Основные поля

```cpp
// === Здоровье и состояние ===
SCHEMA_FIELD(int, m_iHealth)              // Текущее здоровье (0-100+)
SCHEMA_FIELD(int, m_iMaxHealth)           // Максимальное здоровье (обычно 100)
SCHEMA_FIELD(LifeState_t, m_lifeState)    // Состояние жизни (LIFE_ALIVE=0, LIFE_DYING=1, LIFE_DEAD=2)
SCHEMA_FIELD(bool, m_bTakesDamage)        // Может ли получать урон
SCHEMA_FIELD(TakeDamageFlags_t, m_nTakeDamageFlags) // Флаги получения урона
SCHEMA_FIELD(float, m_flDamageAccumulator) // Накопленный урон

// === Команда и владение ===
SCHEMA_FIELD(int, m_iTeamNum)             // Номер команды (0=None, 1=Spectator, 2=T, 3=CT)
SCHEMA_FIELD(CHandle<CBaseEntity>, m_hOwnerEntity) // Владелец сущности

// === Физика и движение ===
SCHEMA_FIELD(Vector, m_vecAbsVelocity)    // Абсолютная скорость (единиц/сек)
SCHEMA_FIELD(Vector, m_vecBaseVelocity)   // Базовая скорость (для конвейеров и т.д.)
SCHEMA_FIELD(QAngle, m_vecAngVelocity)    // Угловая скорость (градусы/сек)
SCHEMA_FIELD(MoveType_t, m_MoveType)      // Тип движения (MOVETYPE_WALK, MOVETYPE_FLY и т.д.)
SCHEMA_FIELD(MoveType_t, m_nActualMoveType) // Фактический тип движения
SCHEMA_FIELD(MoveCollide_t, m_MoveCollide) // Тип коллизии при движении
SCHEMA_FIELD(float, m_flSpeed)            // Текущая скорость
SCHEMA_FIELD(float, m_flFriction)         // Трение (0.0-1.0, обычно 1.0)
SCHEMA_FIELD(float, m_flGravityScale)     // Множитель гравитации (1.0=нормальная, 0.0=нет гравитации)
SCHEMA_FIELD(float, m_flActualGravityScale) // Фактический множитель гравитации
SCHEMA_FIELD(float, m_flTimeScale)        // Масштаб времени для анимаций

// === Флаги и состояния ===
SCHEMA_FIELD(uint32, m_fFlags)            // Флаги сущности (FL_ONGROUND, FL_DUCKING и т.д.)
SCHEMA_FIELD(uint32, m_spawnflags)        // Флаги спавна из карты
SCHEMA_FIELD(uint32, m_fEffects)          // Визуальные эффекты (EF_NODRAW и т.д.)
SCHEMA_FIELD(bool, m_bLagCompensate)      // Использовать ли лаг-компенсацию

// === Компоненты и структура ===
SCHEMA_FIELD(CBodyComponent*, m_CBodyComponent) // Компонент тела (позиция, углы, скелет)
SCHEMA_FIELD(CCollisionProperty*, m_pCollision) // Свойства коллизии
SCHEMA_FIELD_POINTER(CNetworkTransmitComponent, m_NetworkTransmitComponent) // Сетевая передача
SCHEMA_FIELD_POINTER(CUtlStringToken, m_nSubclassID) // ID подкласса для VData

// === Идентификация ===
SCHEMA_FIELD(CUtlString, m_sUniqueHammerID) // Уникальный ID из Hammer
SCHEMA_FIELD(CUtlSymbolLarge, m_target)    // Имя цели для триггеров
SCHEMA_FIELD(CUtlSymbolLarge, m_iGlobalname) // Глобальное имя сущности

// === Время и симуляция ===
SCHEMA_FIELD(float32, m_flSimulationTime) // Время последней симуляции
SCHEMA_FIELD(float, m_lastNetworkChange)  // Время последнего сетевого изменения
SCHEMA_FIELD(CBitVec<64>, m_isSteadyState) // Флаги стабильного состояния

// === Эффекты ===
SCHEMA_FIELD(CHandle<CBaseEntity>, m_hEffectEntity) // Сущность эффекта
```

#### Основные методы

```cpp
// === Индекс и идентификация ===
int entindex();                           // Получить индекс сущности (1-2048)
CHandle<CBaseEntity> GetHandle();         // Получить handle для безопасного хранения
const char* GetName() const;              // Получить имя сущности из карты

// === Позиция и углы ===
Vector GetAbsOrigin();                    // Получить абсолютную позицию
QAngle GetAngRotation();                  // Получить локальные углы
QAngle GetAbsRotation();                  // Получить абсолютные углы
Vector GetAbsVelocity();                  // Получить скорость
void SetAbsOrigin(Vector vecOrigin);      // Установить позицию
void SetAbsRotation(QAngle angAbsRotation); // Установить абсолютные углы
void SetAngRotation(QAngle angRotation);  // Установить локальные углы
void SetAbsVelocity(Vector vecVelocity);  // Установить скорость
void SetBaseVelocity(Vector vecVelocity); // Установить базовую скорость

// === Телепортация ===
// position: новая позиция (nullptr = не менять)
// angles: новые углы (nullptr = не менять)
// velocity: новая скорость (nullptr = не менять)
void Teleport(const Vector* position, const QAngle* angles, const Vector* velocity);

// === Движение ===
void SetMoveType(MoveType_t nMoveType);   // Установить тип движения

// === Урон и здоровье ===
void TakeDamage(int iDamage);             // Нанести урон (уменьшает m_iHealth)
bool IsAlive();                           // Проверить, жива ли сущность

// === Команда ===
int GetTeam();                            // Получить номер команды

// === Коллизия ===
uint8 GetCollisionGroup();                // Получить группу коллизии
void SetCollisionGroup(uint8 nCollisionGroup); // Установить группу коллизии
void CollisionRulesChanged();             // Уведомить об изменении правил коллизии

// === VData (статические данные) ===
CEntitySubclassVDataBase* GetVData();     // Получить указатель на VData класса
```

#### Подклассы и компоненты

##### CBodyComponent

Компонент тела, содержащий узел сцены.

```cpp
SCHEMA_FIELD(CGameSceneNode*, m_pSceneNode) // Узел сцены с позицией и углами
```

##### CGameSceneNode

Узел сцены для трансформации сущности.

```cpp
SCHEMA_FIELD(CEntityInstance*, m_pOwner)   // Владелец узла
SCHEMA_FIELD(CGameSceneNode*, m_pParent)   // Родительский узел
SCHEMA_FIELD(CGameSceneNode*, m_pChild)    // Дочерний узел
SCHEMA_FIELD(CNetworkOriginCellCoordQuantizedVector, m_vecOrigin) // Локальная позиция
SCHEMA_FIELD(QAngle, m_angRotation)        // Локальные углы
SCHEMA_FIELD(float, m_flScale)             // Локальный масштаб
SCHEMA_FIELD(float, m_flAbsScale)          // Абсолютный масштаб
SCHEMA_FIELD(Vector, m_vecAbsOrigin)       // Абсолютная позиция
SCHEMA_FIELD(QAngle, m_angAbsRotation)     // Абсолютные углы
SCHEMA_FIELD(Vector, m_vRenderOrigin)      // Позиция для рендеринга

// Получить матрицу трансформации
matrix3x4_t EntityToWorldTransform();
```

##### CCollisionProperty

Свойства коллизии сущности.

```cpp
// Атрибуты коллизии (доступ через m_collisionAttribute())
struct VPhysicsCollisionAttribute_t {
    uint8 m_nCollisionGroup;               // Группа коллизии
    uint8 m_nCollisionFunctionMask;        // Маска функций коллизии
    // ... другие поля
};
```

### Утилиты для поиска сущностей

```cpp
// Найти сущность по индексу
CEntityInstance* UTIL_GetEntityByIndex(int index);

// Найти первую сущность по имени класса
CEntityInstance* UTIL_FindEntityByClassname(const char* name);

// Найти все сущности по имени класса
std::vector<CEntityInstance*> UTIL_FindEntityByClassnameAll(const char* name);
```

**Пример:**
```cpp
// Найти бомбу
auto* pBomb = (CPlantedC4*)UTIL_FindEntityByClassname("planted_c4");

// Найти всех игроков
auto players = UTIL_FindEntityByClassnameAll("player");
```


## Игроки и контроллеры

В CS2 игрок состоит из двух частей:
- **CCSPlayerController** - логика игрока, счет, деньги (существует всегда)
- **CCSPlayerPawn** - физическое тело игрока (создается при спавне)

### Иерархия классов игрока

```
CBaseEntity
    └── CBaseModelEntity
        ├── CBasePlayerController
        │   └── CCSPlayerController
        └── CBasePlayerPawn
            └── CCSPlayerPawnBase
                └── CCSPlayerPawn
```

### CBasePlayerController

Базовый класс контроллера игрока.

#### Поля

```cpp
SCHEMA_FIELD(uint64, m_steamID)                    // Steam ID игрока (64-бит)
SCHEMA_FIELD(CHandle<CBasePlayerPawn>, m_hPawn)    // Handle на текущий pawn
SCHEMA_FIELD_POINTER(char, m_iszPlayerName)        // Имя игрока (массив char[128])
SCHEMA_FIELD(PlayerConnectedState, m_iConnected)   // Состояние подключения
SCHEMA_FIELD(uint32_t, m_iDesiredFOV)              // Желаемый FOV
```

#### Методы

```cpp
CBasePlayerPawn* GetPawn();                        // Получить pawn
const char* GetPlayerName();                       // Получить имя игрока
int GetPlayerSlot();                               // Получить слот (0-63)
bool IsConnected();                                // Проверить подключение
```

#### Перечисление PlayerConnectedState

```cpp
enum class PlayerConnectedState : uint32_t {
    PlayerNeverConnected = 0xFFFFFFFF,  // Никогда не подключался
    PlayerConnected = 0x0,              // Подключен
    PlayerConnecting = 0x1,             // Подключается
    PlayerReconnecting = 0x2,           // Переподключается
    PlayerDisconnecting = 0x3,          // Отключается
    PlayerDisconnected = 0x4,           // Отключен
    PlayerReserved = 0x5,               // Зарезервирован
};
```

### CCSPlayerController

Контроллер содержит информацию об игроке, которая сохраняется между спавнами.

#### Основные поля

```cpp
// === Базовая информация ===
SCHEMA_FIELD(uint64, m_steamID)                    // Steam ID (наследуется от CBasePlayerController)
SCHEMA_FIELD(CHandle<CCSPlayerPawn>, m_hPlayerPawn) // Handle на CS:GO pawn
SCHEMA_FIELD_POINTER(char, m_iszPlayerName)        // Имя игрока (массив char[128])
SCHEMA_FIELD(bool, m_bPawnIsAlive)                 // Жив ли pawn игрока
SCHEMA_FIELD(bool, m_bEverFullyConnected)          // Был ли когда-либо полностью подключен
SCHEMA_FIELD(int32_t, m_nDisconnectionTick)        // Тик отключения

// === Состояние pawn (синхронизация) ===
SCHEMA_FIELD(uint32_t, m_iPawnHealth)              // Здоровье pawn (0-100+)
SCHEMA_FIELD(int32_t, m_iPawnArmor)                // Броня pawn (0-100)
SCHEMA_FIELD(int16_t, m_nPawnCharacterDefIndex)    // Индекс модели персонажа

// === Статистика ===
SCHEMA_FIELD(int32_t, m_iScore)                    // Общий счет
SCHEMA_FIELD(int32_t, m_iRoundScore)               // Счет за раунд
SCHEMA_FIELD(int32_t, m_iRoundsWon)                // Выигранных раундов
SCHEMA_FIELD(int32_t, m_iMVPs)                     // Количество MVP

// === Сеть ===
SCHEMA_FIELD(uint32_t, m_iPing)                    // Пинг (мс)
SCHEMA_FIELD(float, m_flSmoothedPing)              // Сглаженный пинг

// === Клан ===
SCHEMA_FIELD(CUtlSymbolLarge, m_szClan)            // Тег клана (символ)
SCHEMA_FIELD_POINTER(char, m_szClanName)           // Имя клана (массив char[32])

// === Наблюдатель ===
SCHEMA_FIELD(int32, m_DesiredObserverMode)         // Желаемый режим наблюдателя
SCHEMA_FIELD(CHandle<CCSPlayerPawnBase>, m_hObserverPawn) // Pawn наблюдателя
SCHEMA_FIELD(CHandle<CCSPlayerController>, m_hOriginalControllerOfCurrentPawn) // Оригинальный контроллер

// === Команда ===
SCHEMA_FIELD(GameTime_t, m_flForceTeamTime)        // Время принудительной смены команды

// === Рейтинг ===
SCHEMA_FIELD(int32_t, m_iCompetitiveRanking)       // Рейтинг в соревновательном режиме
SCHEMA_FIELD(int8_t, m_iCompetitiveRankType)       // Тип рейтинга
SCHEMA_FIELD(int32_t, m_iCompetitiveWins)          // Побед в соревновательном режиме

// === Сервисы ===
SCHEMA_FIELD(CCSPlayerController_InGameMoneyServices*, m_pInGameMoneyServices) // Деньги
SCHEMA_FIELD(CCSPlayerController_ActionTrackingServices*, m_pActionTrackingServices) // Статистика действий
SCHEMA_FIELD(CCSPlayerController_InventoryServices*, m_pInventoryServices) // Инвентарь (скины, музыка)
```

#### Основные методы

```cpp
// === Статические методы получения контроллера ===
static CCSPlayerController* FromPawn(CCSPlayerPawn* pawn);  // Из pawn
static CCSPlayerController* FromSlot(int iSlot);            // По слоту (0-63)

// === Получение pawn ===
CCSPlayerPawn* GetPlayerPawn();                    // Получить CS:GO pawn
CBasePlayerPawn* GetPawn();                        // Получить базовый pawn (наследуется)

// === Проверки ===
bool IsBot();                                      // Проверка на бота (FL_CONTROLLER_FAKECLIENT)
bool IsConnected();                                // Проверка подключения (наследуется)

// === Управление игроком ===
void ChangeTeam(int iTeam);                        // Сменить команду (виртуальная функция 99)
void Respawn();                                    // Возродить игрока (виртуальная функция 259)

// === Состояние ===
CSPlayerState GetPawnState();                      // Получить состояние базового pawn
CSPlayerState GetPlayerPawnState();                // Получить состояние CS:GO pawn

// === Наблюдатель ===
CBaseEntity* GetObserverTarget();                  // Получить цель наблюдения
```

#### Перечисление CSPlayerState

```cpp
enum CSPlayerState {
    STATE_ACTIVE = 0x0,              // Активен в игре
    STATE_WELCOME = 0x1,             // Экран приветствия
    STATE_PICKINGTEAM = 0x2,         // Выбор команды
    STATE_PICKINGCLASS = 0x3,        // Выбор класса
    STATE_DEATH_ANIM = 0x4,          // Анимация смерти
    STATE_DEATH_WAIT_FOR_KEY = 0x5,  // Ожидание нажатия клавиши после смерти
    STATE_OBSERVER_MODE = 0x6,       // Режим наблюдателя
    STATE_GUNGAME_RESPAWN = 0x7,     // Респавн в Gun Game
    STATE_DORMANT = 0x8,             // Неактивен
    NUM_PLAYER_STATES = 0x9,
};
```


**Пример работы с контроллером:**

```cpp
// Получить контроллер по слоту
CCSPlayerController* pController = CCSPlayerController::FromSlot(0);
if (!pController) return;

// Получить имя игрока
const char* name = pController->GetPlayerName();

// Получить pawn
CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
if (!pPawn) return;

// Изменить счет
pController->m_iScore(pController->m_iScore() + 10);

// Возродить игрока
pController->Respawn();
```

### CBasePlayerPawn

Базовый класс физического тела игрока.

#### Поля

```cpp
// === Сервисы ===
SCHEMA_FIELD(CCSPlayer_MovementServices*, m_pMovementServices)   // Движение
SCHEMA_FIELD(CCSPlayer_WeaponServices*, m_pWeaponServices)       // Оружие
SCHEMA_FIELD(CCSPlayer_ItemServices*, m_pItemServices)           // Предметы
SCHEMA_FIELD(CPlayer_ObserverServices*, m_pObserverServices)     // Наблюдатель
SCHEMA_FIELD(CPlayer_CameraServices*, m_pCameraServices)         // Камера

// === Связь с контроллером ===
SCHEMA_FIELD(CHandle<CBasePlayerController>, m_hController)      // Handle на контроллер

// === HUD ===
SCHEMA_FIELD(uint32, m_iHideHUD)                                 // Скрытые элементы HUD
SCHEMA_FIELD(bool, m_fInitHUD)                                   // Инициализирован ли HUD
```

#### Методы

```cpp
void CommitSuicide(bool bExplode, bool bForce);    // Самоубийство (виртуальная функция 380)
CBasePlayerController* GetController();            // Получить контроллер
```

### CCSPlayerPawnBase

Базовый класс для CS:GO pawn с дополнительными полями.

#### Поля

```cpp
// === Состояние игрока ===
SCHEMA_FIELD(CSPlayerState, m_iPlayerState)        // Состояние игрока (активен, мертв и т.д.)
SCHEMA_FIELD(CHandle<CCSPlayerController>, m_hOriginalController) // Оригинальный контроллер

// === Прогресс-бар ===
SCHEMA_FIELD(int32_t, m_iProgressBarDuration)      // Длительность прогресс-бара (тики)

// === Ослепление ===
SCHEMA_FIELD(float, m_flFlashDuration)             // Длительность ослепления (секунды)
SCHEMA_FIELD(float, m_flFlashMaxAlpha)             // Максимальная прозрачность ослепления (0.0-1.0)
SCHEMA_FIELD(GameTime_t, m_blindUntilTime)         // Время окончания ослепления
SCHEMA_FIELD(GameTime_t, m_blindStartTime)         // Время начала ослепления

// === Gun Game ===
SCHEMA_FIELD(GameTime_t, m_fImmuneToGunGameDamageTime) // Время иммунитета к урону в Gun Game
SCHEMA_FIELD(bool, m_bGunGameImmunity)             // Иммунитет в Gun Game

// === Сервисы ===
SCHEMA_FIELD(CPlayer_ViewModelServices*, m_pViewModelServices) // Модель от первого лица
```

#### Методы

```cpp
CCSPlayerController* GetOriginalController();      // Получить оригинальный контроллер
bool IsBot();                                      // Проверка на бота (FL_PAWN_FAKECLIENT)
```

### CCSPlayerPawn

Физическое тело игрока с позицией, здоровьем и инвентарем.

#### Основные поля

```cpp
// === Взгляд и камера ===
SCHEMA_FIELD(QAngle, m_angEyeAngles)               // Углы взгляда игрока (pitch, yaw, roll)

// === Скорость ===
SCHEMA_FIELD(float, m_flVelocityModifier)          // Модификатор скорости (0.0-1.0+, 1.0=нормальная)

// === Броня ===
SCHEMA_FIELD(int32, m_ArmorValue)                  // Значение брони (0-100)

// === Зоны ===
SCHEMA_FIELD(bool, m_bInBuyZone)                   // Находится ли в зоне покупки

// === Бусты ===
SCHEMA_FIELD(GameTime_t, m_flHealthShotBoostExpirationTime) // Время окончания буста от аптечки

// === Отдача оружия ===
SCHEMA_FIELD(int, m_aimPunchTickBase)              // Базовый тик отдачи
SCHEMA_FIELD(float, m_aimPunchTickFraction)        // Дробная часть тика отдачи
SCHEMA_FIELD(QAngle, m_aimPunchAngle)              // Угол отдачи (влияет на прицел)
SCHEMA_FIELD(QAngle, m_aimPunchAngleVel)           // Скорость изменения угла отдачи

// === Экипировка (скины) ===
SCHEMA_FIELD(CEconItemView, m_EconGloves)          // Перчатки (скин)
SCHEMA_FIELD(uint8, m_nEconGlovesChanged)          // Флаг изменения перчаток
SCHEMA_FIELD(uint16, m_nCharacterDefIndex)         // Индекс модели персонажа

// === Голос ===
SCHEMA_FIELD(CUtlString, m_strVOPrefix)            // Префикс голосовых команд

// === Обнаружение ===
SCHEMA_FIELD(EntitySpottedState_t, m_entitySpottedState) // Состояние обнаружения врагами

// === Сервисы ===
SCHEMA_FIELD(CCSPlayer_ActionTrackingServices*, m_pActionTrackingServices) // Отслеживание действий
```

#### Структура EntitySpottedState_t

```cpp
struct EntitySpottedState_t {
    bool m_bSpotted;                       // Обнаружен ли игрок
    uint32_t m_bSpottedByMask[2];          // Маска игроков, которые обнаружили (64 бита)
};
```


**Пример работы с pawn:**

```cpp
CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
if (!pPawn) return;

// Установить здоровье
pPawn->m_iHealth(100);

// Установить броню
pPawn->m_ArmorValue(100);

// Телепортировать
Vector newPos(0, 0, 100);
QAngle newAng(0, 90, 0);
pPawn->Teleport(&newPos, &newAng, nullptr);

// Изменить скорость
pPawn->m_flVelocityModifier(0.5f); // 50% скорости

// Получить позицию
Vector pos = pPawn->GetAbsOrigin();
```

## Оружие

### Иерархия классов оружия

```
CBaseEntity
    └── CBaseModelEntity
        └── CEconEntity
            └── CBasePlayerWeapon
                └── CCSWeaponBase
                    ├── CCSWeaponBaseGun (огнестрельное оружие)
                    │   └── CWeaponTaser (электрошокер)
                    └── CWeaponBaseItem (предметы: гранаты, C4)
```

### CEconEntity

Базовый класс для всех экономических сущностей (оружие, перчатки).

#### Поля

```cpp
// === Атрибуты и скины ===
SCHEMA_FIELD(CAttributeContainer, m_AttributeManager) // Контейнер атрибутов (скины, StatTrak)

// === Владелец (для скинов) ===
SCHEMA_FIELD(uint32, m_OriginalOwnerXuidLow)      // Младшие 32 бита XUID владельца
SCHEMA_FIELD(uint32, m_OriginalOwnerXuidHigh)     // Старшие 32 бита XUID владельца

// === Fallback скин (когда нет инвентаря) ===
SCHEMA_FIELD(int32, m_nFallbackPaintKit)          // ID скина (paint kit)
SCHEMA_FIELD(int32, m_nFallbackSeed)              // Seed паттерна скина
SCHEMA_FIELD(float, m_flFallbackWear)             // Износ (0.0=Factory New, 1.0=Battle-Scarred)
SCHEMA_FIELD(int32, m_nFallbackStatTrak)          // Значение StatTrak (-1=нет)
```

#### Подклассы

##### CAttributeContainer

Контейнер атрибутов предмета.

```cpp
SCHEMA_FIELD(CEconItemView, m_Item)               // Представление предмета
```

##### CEconItemView

Представление экономического предмета (оружие, перчатки).

```cpp
// === Идентификация ===
SCHEMA_FIELD(uint16, m_iItemDefinitionIndex)      // Индекс определения предмета (ID оружия)
SCHEMA_FIELD(int32, m_iEntityQuality)             // Качество (обычное, StatTrak и т.д.)
SCHEMA_FIELD(uint32, m_iEntityLevel)              // Уровень предмета

// === ID предмета ===
SCHEMA_FIELD(uint64_t, m_iItemID)                 // Полный ID предмета (64 бита)
SCHEMA_FIELD(uint32, m_iItemIDHigh)               // Старшие 32 бита ID
SCHEMA_FIELD(uint32, m_iItemIDLow)                // Младшие 32 бита ID

// === Владелец ===
SCHEMA_FIELD(uint32, m_iAccountID)                // ID аккаунта владельца
SCHEMA_FIELD(uint32, m_iInventoryPosition)        // Позиция в инвентаре

// === Состояние ===
SCHEMA_FIELD(bool, m_bInitialized)                // Инициализирован ли предмет

// === Атрибуты ===
SCHEMA_FIELD(CAttributeList, m_AttributeList)     // Список статических атрибутов
SCHEMA_FIELD(CAttributeList, m_NetworkedDynamicAttributes) // Динамические атрибуты

// === Имя ===
SCHEMA_FIELD_POINTER(char, m_szCustomName)        // Пользовательское имя (Name Tag)
SCHEMA_FIELD_POINTER(char, m_szCustomNameOverride) // Переопределение имени
```

##### CAttributeList

Список атрибутов предмета.

```cpp
SCHEMA_FIELD(CAttributeManager*, m_pManager)      // Менеджер атрибутов
SCHEMA_FIELD_OLD(CUtlVector<CEconItemAttribute>, CAttributeList, m_Attributes) // Вектор атрибутов
```

##### CEconItemAttribute

Отдельный атрибут предмета.

```cpp
SCHEMA_FIELD(uint16_t, m_iAttributeDefinitionIndex) // Индекс определения атрибута
SCHEMA_FIELD(float32, m_flValue)                  // Текущее значение
SCHEMA_FIELD(float32, m_flInitialValue)           // Начальное значение
SCHEMA_FIELD(int32, m_nRefundableCurrency)        // Возвращаемая валюта
SCHEMA_FIELD(bool, m_bSetBonus)                   // Бонус от набора
```

### CBasePlayerWeapon

Базовый класс оружия игрока.

#### Поля

```cpp
// === Атака ===
SCHEMA_FIELD(GameTick_t, m_nNextPrimaryAttackTick)    // Тик следующей основной атаки
SCHEMA_FIELD(float, m_flNextPrimaryAttackTickRatio)   // Дробная часть тика основной атаки
SCHEMA_FIELD(GameTick_t, m_nNextSecondaryAttackTick)  // Тик следующей вторичной атаки
SCHEMA_FIELD(float, m_flNextSecondaryAttackTickRatio) // Дробная часть тика вторичной атаки

// === Патроны ===
SCHEMA_FIELD(int32_t, m_iClip1)                   // Патроны в основной обойме
SCHEMA_FIELD(int32_t, m_iClip2)                   // Патроны во вторичной обойме (подствольник)
SCHEMA_FIELD_POINTER(int32_t, m_pReserveAmmo)    // Запасные патроны (массив по типам)
```

#### Методы

```cpp
CCSWeaponBaseVData* GetWeaponVData();             // Получить статические данные оружия
const char* GetWeaponClassname();                 // Получить правильное имя класса
```

### CCSWeaponBase

Базовый класс для всего оружия в CS2.

#### Основные поля

```cpp
// === Перезарядка ===
SCHEMA_FIELD(bool, m_bInReload)                   // В процессе перезарядки
SCHEMA_FIELD(bool, m_bReloadVisuallyComplete)    // Визуально завершена перезарядка

// === Точность ===
SCHEMA_FIELD(float, m_fAccuracyPenalty)           // Штраф точности (увеличивается при стрельбе)

// === Отдача ===
SCHEMA_FIELD(int, m_iRecoilIndex)                 // Индекс отдачи (целое)
SCHEMA_FIELD(float, m_flRecoilIndex)              // Индекс отдачи (дробное)
```

### CCSWeaponBaseGun

Класс огнестрельного оружия.

```cpp
// Наследует все поля от CCSWeaponBase
// Дополнительных полей нет
```

### CWeaponTaser

Класс электрошокера (Zeus x27).

#### Поля

```cpp
SCHEMA_FIELD(GameTime_t, m_fFireTime)             // Время выстрела
SCHEMA_FIELD(int32, m_nLastAttackTick)            // Последний тик атаки
```

### CWeaponBaseItem

Класс предметов (гранаты, C4).

```cpp
// Наследует все поля от CCSWeaponBase
// Дополнительных полей нет
```

#### Получение информации об оружии

```cpp
// Получить VData (статические данные оружия)
CCSWeaponBaseVData* GetWeaponVData();

// Получить правильное имя класса (учитывает особые случаи)
const char* GetWeaponClassname();
```

### CBasePlayerWeaponVData

Базовые статические данные оружия.

```cpp
SCHEMA_FIELD(int, m_iMaxClip1)                    // Макс. патронов в основной обойме
SCHEMA_FIELD(int, m_iMaxClip2)                    // Макс. патронов во вторичной обойме
SCHEMA_FIELD(int, m_iDefaultClip1)                // Патронов по умолчанию в обойме
```

### CCSWeaponBaseVData

Статические данные оружия CS:GO (не изменяются во время игры).

```cpp
// === Слот и категория ===
SCHEMA_FIELD(gear_slot_t, m_GearSlot)            // Слот оружия (винтовка, пистолет и т.д.)

// === Экономика ===
SCHEMA_FIELD(int, m_nPrice)                      // Цена оружия ($)

// === Название ===
SCHEMA_FIELD(CUtlString, m_szName)               // Название оружия (локализованное)

// === Патроны ===
SCHEMA_FIELD(int, m_nPrimaryReserveAmmoMax)      // Макс. запасных патронов (основное)
SCHEMA_FIELD(int, m_nSecondaryReserveAmmoMax)    // Макс. запасных патронов (вторичное)
SCHEMA_FIELD(int, m_iMaxClip1)                   // Макс. патронов в обойме (наследуется)

// === Урон ===
SCHEMA_FIELD(int, m_nDamage)                     // Базовый урон
```

#### Перечисление gear_slot_t

```cpp
enum gear_slot_t : uint32_t {
    GEAR_SLOT_INVALID = 0xffffffff,
    GEAR_SLOT_RIFLE = 0x0,           // Основное оружие (винтовки, SMG)
    GEAR_SLOT_PISTOL = 0x1,          // Пистолет
    GEAR_SLOT_KNIFE = 0x2,           // Нож
    GEAR_SLOT_GRENADES = 0x3,        // Гранаты
    GEAR_SLOT_C4 = 0x4,              // C4
    GEAR_SLOT_RESERVED_SLOT6 = 0x5,  // Зарезервировано
    GEAR_SLOT_RESERVED_SLOT7 = 0x6,
    GEAR_SLOT_RESERVED_SLOT8 = 0x7,
    GEAR_SLOT_RESERVED_SLOT9 = 0x8,
    GEAR_SLOT_RESERVED_SLOT10 = 0x9,
    GEAR_SLOT_RESERVED_SLOT11 = 0xa,
    GEAR_SLOT_BOOSTS = 0xb,          // Бусты
    GEAR_SLOT_UTILITY = 0xc,         // Утилиты (дефьюз кит, щит)
    GEAR_SLOT_COUNT = 0xd,
    GEAR_SLOT_FIRST = 0x0,
    GEAR_SLOT_LAST = 0xc,
};
```

**Пример работы с оружием:**

```cpp
CCSWeaponBase* pWeapon = /* получить оружие */;

// Установить патроны
pWeapon->m_iClip1(30);

// Получить статические данные
auto* pVData = pWeapon->GetWeaponVData();
if (pVData) {
    int maxAmmo = pVData->m_iMaxClip1();
    int price = pVData->m_nPrice();
    const char* name = pVData->m_szName().Get();
}

// Получить имя класса
const char* className = pWeapon->GetWeaponClassname();

// Изменить скин
pWeapon->m_nFallbackPaintKit(38);  // Asiimov
pWeapon->m_nFallbackSeed(0);
pWeapon->m_flFallbackWear(0.1f);   // Factory New
```

## Сервисы

Сервисы - это компоненты игрока, отвечающие за различные аспекты. Все сервисы наследуются от `CPlayerPawnComponent`.

### CPlayerPawnComponent

Базовый класс для всех компонентов pawn.

```cpp
SCHEMA_FIELD(CNetworkVarChainer2, __m_pChainEntity) // Цепочка сетевых переменных

// Получить pawn владельца
CCSPlayerPawn* GetPawn();
```

### Иерархия сервисов

```
CPlayerPawnComponent (базовый класс)
    ├── CPlayer_MovementServices
    │   └── CPlayer_MovementServices_Humanoid
    │       └── CCSPlayer_MovementServices
    ├── CPlayer_WeaponServices
    │   └── CCSPlayer_WeaponServices
    ├── CPlayer_ItemServices
    │   └── CCSPlayer_ItemServices
    ├── CPlayer_CameraServices
    ├── CPlayer_ViewModelServices
    │   └── CCSPlayer_ViewModelServices
    ├── CPlayer_ObserverServices
    └── CCSPlayer_DamageReactServices
```

### CPlayer_MovementServices

Базовый сервис движения.

#### Поля

```cpp
// === Кнопки ===
SCHEMA_FIELD(CInButtonState, m_nButtons)          // Состояние кнопок (текущее и изменения)
SCHEMA_FIELD(uint64_t, m_nQueuedButtonDownMask)   // Маска нажатых кнопок в очереди
SCHEMA_FIELD(uint64_t, m_nQueuedButtonChangeMask) // Маска измененных кнопок в очереди
SCHEMA_FIELD(uint64_t, m_nButtonDoublePressed)    // Маска двойных нажатий
SCHEMA_FIELD(uint64_t, m_nToggleButtonDownMask)   // Маска переключаемых кнопок

// === Команды ===
SCHEMA_FIELD_POINTER(uint32_t, m_pButtonPressedCmdNumber) // Номера команд нажатия кнопок (массив[64])
SCHEMA_FIELD(uint32_t, m_nLastCommandNumberProcessed)     // Номер последней обработанной команды

// === Скорость ===
SCHEMA_FIELD(float, m_flMaxspeed)                 // Максимальная скорость (единиц/сек)
```

#### Структура CInButtonState

```cpp
// m_pButtonStates[0] - маска текущих нажатых кнопок
// m_pButtonStates[1] - маска кнопок, изменившихся в текущем кадре
// m_pButtonStates[2] - дополнительные данные
SCHEMA_FIELD_POINTER(uint64, m_pButtonStates)     // Массив состояний кнопок [3]
```

### CPlayer_MovementServices_Humanoid

Сервис движения для гуманоидов.

#### Поля

```cpp
// === Падение ===
SCHEMA_FIELD(float, m_flFallVelocity)             // Скорость падения (единиц/сек)

// === Приседание ===
SCHEMA_FIELD(float, m_bInCrouch)                  // В процессе приседания
SCHEMA_FIELD(bool, m_bDucked)                     // Присел
SCHEMA_FIELD(uint32_t, m_nCrouchState)            // Состояние приседания
SCHEMA_FIELD(bool, m_bInDuckJump)                 // Прыжок из приседа

// === Поверхность ===
SCHEMA_FIELD(float, m_flSurfaceFriction)          // Трение поверхности (0.0-1.0)
```

### CCSPlayer_MovementServices

Управление движением игрока в CS:GO.

```cpp
// === Падение ===
SCHEMA_FIELD(float, m_flMaxFallVelocity)          // Максимальная скорость падения

// === Прыжок ===
SCHEMA_FIELD(float, m_flJumpVel)                  // Скорость прыжка (единиц/сек)
SCHEMA_FIELD(float, m_flAccumulatedJumpError)     // Накопленная ошибка прыжка

// === Выносливость ===
SCHEMA_FIELD(float, m_flStamina)                  // Выносливость (0.0-100.0)

// === Приседание ===
SCHEMA_FIELD(float, m_flDuckSpeed)                // Скорость приседания
SCHEMA_FIELD(bool, m_bDuckOverride)               // Переопределение приседания

// === Вода ===
SCHEMA_FIELD(int32, m_nOldWaterLevel)             // Предыдущий уровень воды (0=нет, 1=ноги, 2=пояс, 3=голова)

// === Звуки ===
SCHEMA_FIELD(int32, m_iFootsteps)                 // Счетчик шагов
```

**Пример:**

```cpp
auto* pMovement = pPawn->m_pMovementServices();
if (!pMovement) return;

// Изменить скорость
pMovement->m_flMaxspeed(250.0f);

// Проверить нажатие кнопки
uint64_t* buttons = pMovement->m_nButtons().m_pButtonStates();
if (buttons[0] & IN_JUMP) {
    // Игрок прыгает
}

// Изменить скорость прыжка
pMovement->m_flJumpVel(300.0f);
```

### CPlayer_WeaponServices

Базовый сервис оружия.

```cpp
// === Оружие ===
SCHEMA_FIELD_POINTER(CUtlVector<CHandle<CBasePlayerWeapon>>, m_hMyWeapons) // Все оружие игрока
SCHEMA_FIELD(CHandle<CBasePlayerWeapon>, m_hActiveWeapon)   // Активное оружие
SCHEMA_FIELD(CHandle<CBasePlayerWeapon>, m_hLastWeapon)     // Последнее оружие

// === Патроны ===
SCHEMA_FIELD_POINTER(uint16_t, m_iAmmo)           // Запасные патроны по типам (массив)
```

### CCSPlayer_WeaponServices

Управление оружием игрока в CS:GO.

```cpp
// === Атака ===
SCHEMA_FIELD(GameTime_t, m_flNextAttack)          // Время следующей атаки

// === Осмотр оружия ===
SCHEMA_FIELD(bool, m_bIsLookingAtWeapon)          // Смотрит на оружие (F)
SCHEMA_FIELD(bool, m_bIsHoldingLookAtWeapon)      // Удерживает осмотр оружия

// === Сохраненное оружие ===
SCHEMA_FIELD(CHandle<CBasePlayerWeapon>, m_hSavedWeapon) // Сохраненное оружие

// === Время до переключения ===
SCHEMA_FIELD(int32_t, m_nTimeToMelee)             // Время до переключения на нож
SCHEMA_FIELD(int32_t, m_nTimeToSecondary)         // Время до переключения на пистолет
SCHEMA_FIELD(int32_t, m_nTimeToPrimary)           // Время до переключения на основное
SCHEMA_FIELD(int32_t, m_nTimeToSniperRifle)       // Время до переключения на снайперку

// === Состояние ===
SCHEMA_FIELD(bool, m_bIsBeingGivenItem)           // Получает предмет
SCHEMA_FIELD(bool, m_bIsPickingUpItemWithUse)     // Поднимает предмет кнопкой E
SCHEMA_FIELD(bool, m_bPickedUpWeapon)             // Поднял оружие
```

#### Методы

```cpp
// Выбросить оружие
// pWeapon: оружие для выброса
// pVecTarget: целевая позиция (nullptr = перед игроком)
// pVelocity: скорость выброса (nullptr = автоматически)
void DropWeapon(CBasePlayerWeapon* pWeapon, Vector* pVecTarget = nullptr, Vector* pVelocity = nullptr);
```


**Пример:**

```cpp
auto* pWeaponServices = pPawn->m_pWeaponServices();
if (!pWeaponServices) return;

// Получить активное оружие
CBasePlayerWeapon* pActiveWeapon = pWeaponServices->m_hActiveWeapon().Get();

// Получить все оружие игрока
auto* weapons = pWeaponServices->m_hMyWeapons();
for (int i = 0; i < weapons->Count(); i++) {
    CBasePlayerWeapon* pWeapon = weapons->Element(i).Get();
    if (pWeapon) {
        // Работа с оружием
    }
}

// Выбросить активное оружие
if (pActiveWeapon) {
    pWeaponServices->DropWeapon(pActiveWeapon);
}
```

### CPlayer_ItemServices

Базовый сервис предметов.

```cpp
// Базовый класс без дополнительных полей
```

### CCSPlayer_ItemServices

Управление предметами игрока (броня, дефьюз кит).

```cpp
// === Предметы ===
SCHEMA_FIELD(bool, m_bHasDefuser)                 // Есть ли дефьюз кит
SCHEMA_FIELD(bool, m_bHasHelmet)                  // Есть ли шлем
```

#### Методы (виртуальные)

```cpp
// Выдать оружие по имени (возвращает указатель)
CBasePlayerWeapon* GiveNamedItem(const char* pchName);

// Выдать оружие по имени (возвращает bool)
bool GiveNamedItemBool(const char* pchName);

// Выбросить активное оружие (параметр игнорируется, используйте CCSPlayer_WeaponServices::DropWeapon)
void DropActiveWeapon(CBasePlayerWeapon* pWeapon);

// Удалить все оружие (removeSuit: удалить ли костюм)
void StripPlayerWeapons(bool removeSuit);

// Удалить все оружие (виртуальная функция 23)
void RemoveWeapons();
```

**Пример:**

```cpp
auto* pItemServices = pPawn->m_pItemServices();
if (!pItemServices) return;

// Выдать AK-47
CBasePlayerWeapon* pWeapon = pItemServices->GiveNamedItem("weapon_ak47");

// Выдать дефьюз кит
pItemServices->m_bHasDefuser(true);

// Выдать шлем
pItemServices->m_bHasHelmet(true);

// Удалить все оружие
pItemServices->RemoveWeapons();
```

### CPlayer_CameraServices

Сервис камеры игрока.

```cpp
// === Punch (отдача камеры) ===
SCHEMA_FIELD(QAngle, m_vecCsViewPunchAngle)       // Угол отдачи камеры (CS:GO специфичный)

// === Вид ===
SCHEMA_FIELD(CHandle<CBaseEntity>, m_hViewEntity) // Сущность вида (для камер)
```

### CPlayer_ViewModelServices

Базовый сервис модели от первого лица.

```cpp
// Базовый класс без дополнительных полей
```

### CCSPlayer_ViewModelServices

Сервис модели от первого лица для CS:GO.

```cpp
// === Модели ===
SCHEMA_FIELD_POINTER(CHandle<CBaseViewModel>, m_hViewModel) // Модели от первого лица (массив)
```

### CPlayer_ObserverServices

Сервис наблюдателя.

```cpp
// === Режим наблюдения ===
SCHEMA_FIELD(ObserverMode_t, m_iObserverMode)     // Текущий режим наблюдателя
SCHEMA_FIELD(ObserverMode_t, m_iObserverLastMode) // Предыдущий режим наблюдателя
SCHEMA_FIELD(bool, m_bForcedObserverMode)         // Принудительный режим

// === Цель ===
SCHEMA_FIELD(CHandle<CBaseEntity>, m_hObserverTarget) // Цель наблюдения
```

#### Перечисление ObserverMode_t

```cpp
enum ObserverMode_t : uint8_t {
    OBS_MODE_NONE = 0x0,         // Нет наблюдения
    OBS_MODE_FIXED = 0x1,        // Фиксированная камера
    OBS_MODE_IN_EYE = 0x2,       // От первого лица
    OBS_MODE_CHASE = 0x3,        // Преследование (от третьего лица)
    OBS_MODE_ROAMING = 0x4,      // Свободная камера
    OBS_MODE_DIRECTED = 0x5,     // Режиссерская камера
    NUM_OBSERVER_MODES = 0x6,
};
```

### CCSPlayerController_InGameMoneyServices

Управление деньгами игрока.

```cpp
SCHEMA_FIELD(int, m_iAccount)                     // Деньги игрока ($0-$16000)
```

**Пример:**

```cpp
auto* pMoneyServices = pController->m_pInGameMoneyServices();
if (!pMoneyServices) return;

// Установить деньги
pMoneyServices->m_iAccount(16000);

// Добавить деньги
pMoneyServices->m_iAccount(pMoneyServices->m_iAccount() + 1000);
```

### CCSPlayerController_InventoryServices

Сервис инвентаря контроллера (скины, музыка, уровень).

```cpp
// === Опыт и уровень ===
SCHEMA_FIELD(int32_t, m_nPersonaDataXpTrailLevel) // Уровень следа XP
SCHEMA_FIELD(int32_t, m_nPersonaDataPublicLevel)  // Публичный уровень профиля

// === Музыка ===
SCHEMA_FIELD(uint16_t, m_unMusicID)               // ID музыкального набора

// === Рейтинг ===
SCHEMA_FIELD_POINTER(int, m_rank)                 // Рейтинг (массив)
```

### CCSPlayerController_ActionTrackingServices

Сервис отслеживания действий контроллера.

```cpp
SCHEMA_FIELD(CSMatchStats_t, m_matchStats)        // Статистика матча
```

#### Структура CSPerRoundStats_t

Статистика за раунд.

```cpp
SCHEMA_FIELD(int, m_iKills)                       // Убийств
SCHEMA_FIELD(int, m_iDeaths)                      // Смертей
SCHEMA_FIELD(int, m_iAssists)                     // Ассистов
SCHEMA_FIELD(int, m_iDamage)                      // Нанесено урона
SCHEMA_FIELD(int, m_iEquipmentValue)              // Стоимость экипировки
SCHEMA_FIELD(int, m_iMoneySaved)                  // Сохранено денег
SCHEMA_FIELD(int, m_iKillReward)                  // Награда за убийства
SCHEMA_FIELD(int, m_iLiveTime)                    // Время жизни (секунды)
SCHEMA_FIELD(int, m_iHeadShotKills)               // Убийств в голову
SCHEMA_FIELD(int, m_iObjective)                   // Выполнено целей
SCHEMA_FIELD(int, m_iCashEarned)                  // Заработано денег
SCHEMA_FIELD(int, m_iUtilityDamage)               // Урон утилитами
SCHEMA_FIELD(int, m_iEnemiesFlashed)              // Ослеплено врагов
```

#### Структура CSMatchStats_t

Статистика матча (наследует CSPerRoundStats_t).

```cpp
SCHEMA_FIELD(int32_t, m_iEntryWins);              // Побед в первых столкновениях
// + все поля из CSPerRoundStats_t
```

### CCSPlayer_ActionTrackingServices

Сервис отслеживания действий pawn.

```cpp
SCHEMA_FIELD(WeaponPurchaseTracker_t, m_weaponPurchasesThisRound) // Покупки оружия в раунде
```

#### Структура WeaponPurchaseTracker_t

```cpp
SCHEMA_FIELD_POINTER(CUtlVector<WeaponPurchaseCount_t>, m_weaponPurchases) // Вектор покупок
```

#### Класс WeaponPurchaseCount_t

```cpp
uint16_t m_nItemDefIndex;                         // Индекс определения предмета
uint16_t m_nCount;                                // Количество покупок
```

## Примеры использования

### Пример 1: Выдать оружие игроку

```cpp
void GiveWeaponToPlayer(CCSPlayerController* pController, const char* weaponName)
{
    if (!pController) return;
    
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn || !pPawn->IsAlive()) return;
    
    auto* pItemServices = pPawn->m_pItemServices();
    if (!pItemServices) return;
    
    // Выдать оружие
    CBasePlayerWeapon* pWeapon = pItemServices->GiveNamedItem(weaponName);
    if (pWeapon) {
        // Установить патроны
        pWeapon->m_iClip1(30);
    }
}
```


### Пример 2: Телепортация игрока

```cpp
void TeleportPlayer(CCSPlayerController* pController, Vector pos, QAngle angles)
{
    if (!pController) return;
    
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn) return;
    
    // Телепортировать
    pPawn->Teleport(&pos, &angles, nullptr);
    
    // Или установить напрямую
    pPawn->SetAbsOrigin(pos);
    pPawn->SetAbsRotation(angles);
}
```

### Пример 3: Изменить здоровье и броню

```cpp
void SetPlayerHealthArmor(CCSPlayerController* pController, int health, int armor)
{
    if (!pController) return;
    
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn) return;
    
    // Установить здоровье
    pPawn->m_iHealth(health);
    pPawn->m_iMaxHealth(health);
    
    // Установить броню
    pPawn->m_ArmorValue(armor);
    
    // Обновить информацию в контроллере
    pController->m_iPawnHealth(health);
    pController->m_iPawnArmor(armor);
}
```

### Пример 4: Удалить все оружие и выдать новое

```cpp
void ResetPlayerWeapons(CCSPlayerController* pController)
{
    if (!pController) return;
    
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn) return;
    
    auto* pItemServices = pPawn->m_pItemServices();
    if (!pItemServices) return;
    
    // Удалить все оружие
    pItemServices->RemoveWeapons();
    
    // Выдать нож
    pItemServices->GiveNamedItem("weapon_knife");
    
    // Выдать пистолет
    auto* pPistol = pItemServices->GiveNamedItem("weapon_glock");
    if (pPistol) {
        pPistol->m_iClip1(20);
    }
    
    // Выдать основное оружие
    auto* pRifle = pItemServices->GiveNamedItem("weapon_ak47");
    if (pRifle) {
        pRifle->m_iClip1(30);
    }
}
```


### Пример 5: Итерация по всем игрокам

```cpp
void ProcessAllPlayers()
{
    for (int i = 0; i < 64; i++)
    {
        CCSPlayerController* pController = CCSPlayerController::FromSlot(i);
        if (!pController || !pController->IsConnected())
            continue;
        
        CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
        if (!pPawn || !pPawn->IsAlive())
            continue;
        
        // Работа с игроком
        const char* name = pController->GetPlayerName();
        int health = pPawn->m_iHealth();
        Vector pos = pPawn->GetAbsOrigin();
        
        // Например, вылечить всех
        if (health < 100) {
            pPawn->m_iHealth(100);
        }
    }
}
```

### Пример 6: Изменить скорость игрока

```cpp
void SetPlayerSpeed(CCSPlayerController* pController, float speedMultiplier)
{
    if (!pController) return;
    
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn) return;
    
    // Метод 1: Через VelocityModifier (временный эффект)
    pPawn->m_flVelocityModifier(speedMultiplier);
    
    // Метод 2: Через MovementServices (постоянное изменение)
    auto* pMovement = pPawn->m_pMovementServices();
    if (pMovement) {
        pMovement->m_flMaxspeed(250.0f * speedMultiplier);
    }
}
```

### Пример 7: Работа с виртуальными функциями

```cpp
// Вызов виртуальной функции через CALL_VIRTUAL
void RespawnPlayer(CCSPlayerController* pController)
{
    if (!pController) return;
    
    // Вызов виртуальной функции Respawn (индекс 259)
    CALL_VIRTUAL(void, 259, pController);
    
    // Или через метод класса
    pController->Respawn();
}
```


### Пример 8: Работа со скинами оружия

```cpp
void SetWeaponSkin(CBasePlayerWeapon* pWeapon, int paintKit, float wear)
{
    if (!pWeapon) return;
    
    // Установить скин
    pWeapon->m_nFallbackPaintKit(paintKit);
    
    // Установить износ (0.0 = Factory New, 1.0 = Battle-Scarred)
    pWeapon->m_flFallbackWear(wear);
    
    // Установить seed для паттерна
    pWeapon->m_nFallbackSeed(0);
    
    // Для StatTrak
    pWeapon->m_nFallbackStatTrak(1000);
    
    // Обновить модель
    pWeapon->m_AttributeManager().m_Item().m_iItemDefinitionIndex.NetworkStateChanged();
}
```

### Пример 9: Проверка состояния игрока

```cpp
bool IsPlayerValid(CCSPlayerController* pController)
{
    if (!pController)
        return false;
    
    // Проверить подключение
    if (!pController->IsConnected())
        return false;
    
    // Получить pawn
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn)
        return false;
    
    // Проверить жизнь
    if (!pPawn->IsAlive())
        return false;
    
    // Проверить команду
    if (pPawn->GetTeam() < 2) // Не в T или CT
        return false;
    
    return true;
}
```

### Пример 10: Изменить гравитацию

```cpp
void SetPlayerGravity(CCSPlayerController* pController, float gravityScale)
{
    if (!pController) return;
    
    CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
    if (!pPawn) return;
    
    // Установить множитель гравитации
    // 1.0 = нормальная, 0.5 = половина, 2.0 = двойная
    pPawn->m_flGravityScale(gravityScale);
}
```


## Важные типы и перечисления

### MoveType_t

Типы движения сущности:

```cpp
MOVETYPE_NONE = 0,        // Не двигается (статичный объект)
MOVETYPE_WALK = 2,        // Обычное движение игрока (с гравитацией и коллизией)
MOVETYPE_FLY = 4,         // Полет (свободное движение в 3D)
MOVETYPE_NOCLIP = 8,      // Noclip (проходит сквозь стены)
MOVETYPE_LADDER = 9,      // На лестнице
MOVETYPE_OBSERVER = 10    // Наблюдатель (режим спектатора)
```

**Пример использования:**

```cpp
// Включить noclip
pPawn->SetMoveType(MOVETYPE_NOCLIP);

// Вернуть обычное движение
pPawn->SetMoveType(MOVETYPE_WALK);
```

### LifeState_t

Состояние жизни сущности:

```cpp
LIFE_ALIVE = 0,           // Жив
LIFE_DYING = 1,           // Умирает (в процессе смерти)
LIFE_DEAD = 2             // Мертв
```

**Пример использования:**

```cpp
if (pPawn->m_lifeState() == LIFE_ALIVE) {
    // Игрок жив
}
```

### gear_slot_t

Слоты оружия (см. раздел "Оружие" выше):

```cpp
GEAR_SLOT_RIFLE = 0,      // Основное оружие
GEAR_SLOT_PISTOL = 1,     // Пистолет
GEAR_SLOT_KNIFE = 2,      // Нож
GEAR_SLOT_GRENADES = 3,   // Гранаты
GEAR_SLOT_C4 = 4,         // C4
GEAR_SLOT_UTILITY = 12    // Утилиты (дефьюз кит)
```

### InputBitMask_t

Маски кнопок ввода:

```cpp
IN_NONE = 0x0,            // Нет нажатий
IN_ALL = 0xffffffffffffffff, // Все кнопки

// Основные кнопки
IN_ATTACK = 0x1,          // Левая кнопка мыши (основная атака)
IN_JUMP = 0x2,            // Прыжок (Space)
IN_DUCK = 0x4,            // Присесть (Ctrl)
IN_FORWARD = 0x8,         // Вперед (W)
IN_BACK = 0x10,           // Назад (S)
IN_USE = 0x20,            // Использовать (E)

// Повороты (устаревшие)
IN_TURNLEFT = 0x80,       // Повернуть налево
IN_TURNRIGHT = 0x100,     // Повернуть направо

// Движение влево/вправо
IN_MOVELEFT = 0x200,      // Влево (A)
IN_MOVERIGHT = 0x400,     // Вправо (D)

// Дополнительные действия
IN_ATTACK2 = 0x800,       // Правая кнопка мыши (вторичная атака)
IN_RELOAD = 0x2000,       // Перезарядка (R)
IN_SPEED = 0x10000,       // Shift (ходьба/бег)
IN_JOYAUTOSPRINT = 0x20000, // Автоспринт (джойстик)

// CS:GO специфичные
IN_FIRST_MOD_SPECIFIC_BIT = 0x100000000,
IN_USEORRELOAD = 0x100000000, // Использовать или перезарядить
IN_SCORE = 0x200000000,       // Таблица счета (Tab)
IN_ZOOM = 0x400000000,        // Зум (для снайперских винтовок)
IN_LOOK_AT_WEAPON = 0x800000000, // Осмотр оружия (F)
```

**Пример использования:**

```cpp
auto* pMovement = pPawn->m_pMovementServices();
uint64_t* buttons = pMovement->m_nButtons().m_pButtonStates();

// Проверить нажатие прыжка
if (buttons[0] & IN_JUMP) {
    // Игрок прыгает
}

// Проверить нажатие атаки
if (buttons[0] & IN_ATTACK) {
    // Игрок стреляет
}

// Проверить движение вперед
if (buttons[0] & IN_FORWARD) {
    // Игрок движется вперед
}

// Проверить приседание
if (buttons[0] & IN_DUCK) {
    // Игрок приседает
}
```

### EInButtonState

Состояние кнопки (для отслеживания изменений):

```cpp
IN_BUTTON_UP = 0x0,              // Кнопка отпущена
IN_BUTTON_DOWN = 0x1,            // Кнопка нажата
IN_BUTTON_DOWN_UP = 0x2,         // Нажата и отпущена
IN_BUTTON_UP_DOWN = 0x3,         // Отпущена и нажата
IN_BUTTON_UP_DOWN_UP = 0x4,      // Отпущена, нажата, отпущена
IN_BUTTON_DOWN_UP_DOWN = 0x5,    // Нажата, отпущена, нажата
IN_BUTTON_DOWN_UP_DOWN_UP = 0x6, // Нажата, отпущена, нажата, отпущена
IN_BUTTON_UP_DOWN_UP_DOWN = 0x7, // Отпущена, нажата, отпущена, нажата
IN_BUTTON_STATE_COUNT = 0x8,
```

### DamageTypes_t

Типы урона:

```cpp
DMG_GENERIC = 0x0,        // Общий урон
DMG_CRUSH = 0x1,          // Раздавливание
DMG_BULLET = 0x2,         // Пули
DMG_SLASH = 0x4,          // Резаный урон (нож)
DMG_BURN = 0x8,           // Огонь
DMG_VEHICLE = 0x10,       // Транспорт
DMG_FALL = 0x20,          // Падение
DMG_BLAST = 0x40,         // Взрыв
DMG_CLUB = 0x80,          // Удар тупым предметом
DMG_SHOCK = 0x100,        // Электричество
DMG_SONIC = 0x200,        // Звуковой урон
DMG_ENERGYBEAM = 0x400,   // Энергетический луч
DMG_DROWN = 0x4000,       // Утопление
DMG_POISON = 0x8000,      // Яд
DMG_RADIATION = 0x10000,  // Радиация
DMG_DROWNRECOVER = 0x20000, // Восстановление после утопления
DMG_ACID = 0x40000,       // Кислота
DMG_PHYSGUN = 0x100000,   // Физ-пушка
DMG_DISSOLVE = 0x200000,  // Растворение
DMG_BLAST_SURFACE = 0x400000, // Взрыв на поверхности
DMG_BUCKSHOT = 0x1000000, // Дробь
DMG_LASTGENERICFLAG = 0x1000000,
DMG_HEADSHOT = 0x2000000, // Выстрел в голову
DMG_DANGERZONE = 0x4000000, // Урон в Danger Zone
```

### TakeDamageFlags_t

Флаги получения урона:

```cpp
DFLAG_NONE = 0x0,                          // Нет флагов
DFLAG_SUPPRESS_HEALTH_CHANGES = 0x1,       // Подавить изменения здоровья
DFLAG_SUPPRESS_PHYSICS_FORCE = 0x2,        // Подавить физическую силу
DFLAG_SUPPRESS_EFFECTS = 0x4,              // Подавить эффекты
DFLAG_PREVENT_DEATH = 0x8,                 // Предотвратить смерть
DFLAG_FORCE_DEATH = 0x10,                  // Принудительная смерть
DFLAG_ALWAYS_GIB = 0x20,                   // Всегда расчленять
DFLAG_NEVER_GIB = 0x40,                    // Никогда не расчленять
DFLAG_REMOVE_NO_RAGDOLL = 0x80,            // Удалить без ragdoll
DFLAG_SUPPRESS_DAMAGE_MODIFICATION = 0x100, // Подавить модификацию урона
DFLAG_ALWAYS_FIRE_DAMAGE_EVENTS = 0x200,  // Всегда вызывать события урона
DFLAG_RADIUS_DMG = 0x400,                  // Радиусный урон
DMG_LASTDFLAG = 0x400,
DFLAG_IGNORE_ARMOR = 0x800,                // Игнорировать броню
```

### ObserverMode_t

Режимы наблюдателя (см. раздел "Сервисы" выше):

```cpp
OBS_MODE_NONE = 0x0,      // Нет наблюдения
OBS_MODE_FIXED = 0x1,     // Фиксированная камера
OBS_MODE_IN_EYE = 0x2,    // От первого лица
OBS_MODE_CHASE = 0x3,     // Преследование (от третьего лица)
OBS_MODE_ROAMING = 0x4,   // Свободная камера
OBS_MODE_DIRECTED = 0x5,  // Режиссерская камера
NUM_OBSERVER_MODES = 0x6,
```

### CSRoundEndReason

Причины окончания раунда:

```cpp
TargetBombed = 1,          // Цель взорвана
VIPEscaped,                // VIP сбежал (не используется в CS:GO)
VIPKilled,                 // VIP убит (не используется в CS:GO)
TerroristsEscaped,         // Террористы сбежали
CTStoppedEscape,           // КТ остановили побег
TerroristsStopped,         // Террористы остановлены
BombDefused,               // Бомба обезврежена
CTWin,                     // Победа КТ
TerroristWin,              // Победа террористов
Draw,                      // Ничья
HostagesRescued,           // Заложники спасены
TargetSaved,               // Цель спасена
HostagesNotRescued,        // Заложники не спасены
TerroristsNotEscaped,      // Террористы не сбежали
VIPNotEscaped,             // VIP не сбежал (не используется в CS:GO)
GameStart,                 // Начало игры
TerroristsSurrender,       // Террористы сдались
CTSurrender,               // КТ сдались
TerroristsPlanted,         // Террористы установили бомбу
CTsReachedHostage,         // КТ достигли заложника
SurvivalWin,               // Победа в Survival
SurvivalDraw               // Ничья в Survival
```

### GamePhase

Фазы игры:

```cpp
GAMEPHASE_WARMUP_ROUND,        // Разминка
GAMEPHASE_PLAYING_STANDARD,    // Обычная игра
GAMEPHASE_PLAYING_FIRST_HALF,  // Первая половина
GAMEPHASE_PLAYING_SECOND_HALF, // Вторая половина
GAMEPHASE_HALFTIME,            // Перерыв
GAMEPHASE_MATCH_ENDED,         // Матч окончен
GAMEPHASE_MAX
```

### ParticleAttachment_t

Типы прикрепления частиц:

```cpp
PATTACH_INVALID = 0xffffffff,
PATTACH_ABSORIGIN = 0x0,           // Спавн на позиции сущности
PATTACH_ABSORIGIN_FOLLOW = 0x1,    // Спавн и следование за позицией
PATTACH_CUSTOMORIGIN = 0x2,        // Пользовательская позиция
PATTACH_CUSTOMORIGIN_FOLLOW = 0x3, // Пользовательская позиция с следованием
PATTACH_POINT = 0x4,               // Спавн на точке прикрепления
PATTACH_POINT_FOLLOW = 0x5,        // Спавн и следование за точкой
PATTACH_EYES_FOLLOW = 0x6,         // Следование за глазами
PATTACH_OVERHEAD_FOLLOW = 0x7,     // Следование над головой
PATTACH_WORLDORIGIN = 0x8,         // Мировая позиция
PATTACH_ROOTBONE_FOLLOW = 0x9,     // Следование за корневой костью
PATTACH_RENDERORIGIN_FOLLOW = 0xa, // Следование за позицией рендера
PATTACH_MAIN_VIEW = 0xb,           // Главный вид
PATTACH_WATERWAKE = 0xc,           // След на воде
PATTACH_CENTER_FOLLOW = 0xd,       // Следование за центром
PATTACH_CUSTOM_GAME_STATE_1 = 0xe, // Пользовательское состояние игры 1
PATTACH_HEALTHBAR = 0xf,           // Полоса здоровья
MAX_PATTACH_TYPES = 0x10,
```

### gender_t

Пол персонажа (для звуков):

```cpp
GENDER_NONE = 0x0,         // Нет
GENDER_MALE = 0x1,         // Мужской
GENDER_FEMALE = 0x2,       // Женский
// ... другие значения для Left 4 Dead
GENDER_LAST = 0x14,
```

### Флаги сущностей (m_fFlags)

Основные флаги состояния сущности:

```cpp
FL_ONGROUND = (1 << 0),        // На земле
FL_DUCKING = (1 << 1),         // Приседает
FL_WATERJUMP = (1 << 2),       // Прыжок из воды
FL_ONTRAIN = (1 << 3),         // На поезде
FL_INRAIN = (1 << 4),          // Под дождем
FL_FROZEN = (1 << 5),          // Заморожен
FL_ATCONTROLS = (1 << 6),      // У управления
FL_CLIENT = (1 << 7),          // Клиент (игрок)
FL_FAKECLIENT = (1 << 8),      // Фейковый клиент (бот)
FL_INWATER = (1 << 9),         // В воде
FL_FLY = (1 << 10),            // Летает
FL_SWIM = (1 << 11),           // Плавает
FL_CONVEYOR = (1 << 12),       // На конвейере
FL_NPC = (1 << 13),            // NPC
FL_GODMODE = (1 << 14),        // Режим бога
FL_NOTARGET = (1 << 15),       // Не цель для NPC
FL_AIMTARGET = (1 << 16),      // Цель для прицеливания
FL_PARTIALGROUND = (1 << 17),  // Частично на земле
FL_STATICPROP = (1 << 18),     // Статичный проп
FL_GRAPHED = (1 << 19),        // В графе навигации
FL_GRENADE = (1 << 20),        // Граната
FL_STEPMOVEMENT = (1 << 21),   // Пошаговое движение
FL_DONTTOUCH = (1 << 22),      // Не трогать
FL_BASEVELOCITY = (1 << 23),   // Использует базовую скорость
FL_WORLDBRUSH = (1 << 24),     // Кисть мира
FL_OBJECT = (1 << 25),         // Объект
FL_KILLME = (1 << 26),         // Убить меня
FL_ONFIRE = (1 << 27),         // В огне
FL_DISSOLVING = (1 << 28),     // Растворяется
FL_TRANSRAGDOLL = (1 << 29),   // Переходный ragdoll
FL_UNBLOCKABLE_BY_PLAYER = (1 << 30), // Не блокируется игроком
```

**Пример использования:**

```cpp
// Проверить, на земле ли игрок
if (pPawn->m_fFlags() & FL_ONGROUND) {
    // Игрок на земле
}

// Проверить, в воде ли игрок
if (pPawn->m_fFlags() & FL_INWATER) {
    // Игрок в воде
}

// Проверить, бот ли это
if (pPawn->m_fFlags() & FL_FAKECLIENT) {
    // Это бот
}
```

### Флаги контроллера

```cpp
FL_CONTROLLER_FAKECLIENT = (1 << 8), // Контроллер бота
```

### Флаги pawn

```cpp
FL_PAWN_FAKECLIENT = (1 << 8),       // Pawn бота
```

### Группы коллизий

```cpp
COLLISION_GROUP_NONE = 0,              // Нет группы
COLLISION_GROUP_DEBRIS = 1,            // Обломки
COLLISION_GROUP_DEBRIS_TRIGGER = 2,    // Триггер обломков
COLLISION_GROUP_INTERACTIVE_DEBRIS = 3, // Интерактивные обломки
COLLISION_GROUP_INTERACTIVE = 4,       // Интерактивные объекты
COLLISION_GROUP_PLAYER = 5,            // Игроки
COLLISION_GROUP_BREAKABLE_GLASS = 6,   // Разбиваемое стекло
COLLISION_GROUP_VEHICLE = 7,           // Транспорт
COLLISION_GROUP_PLAYER_MOVEMENT = 8,   // Движение игрока
COLLISION_GROUP_NPC = 9,               // NPC
COLLISION_GROUP_IN_VEHICLE = 10,       // В транспорте
COLLISION_GROUP_WEAPON = 11,           // Оружие
COLLISION_GROUP_VEHICLE_CLIP = 12,     // Клип транспорта
COLLISION_GROUP_PROJECTILE = 13,       // Снаряды
COLLISION_GROUP_DOOR_BLOCKER = 14,     // Блокировщик дверей
COLLISION_GROUP_PASSABLE_DOOR = 15,    // Проходимая дверь
COLLISION_GROUP_DISSOLVING = 16,       // Растворяющиеся объекты
COLLISION_GROUP_PUSHAWAY = 17,         // Отталкиваемые объекты
COLLISION_GROUP_NPC_ACTOR = 18,        // NPC актер
COLLISION_GROUP_NPC_SCRIPTED = 19,     // Скриптовый NPC
```


## Советы и лучшие практики

### 1. Всегда проверяйте указатели

```cpp
CCSPlayerController* pController = CCSPlayerController::FromSlot(slot);
if (!pController) return; // ВАЖНО!

CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
if (!pPawn) return; // ВАЖНО!
```

### 2. Используйте NetworkStateChanged для ручных изменений

Если вы изменяете данные через указатель, вызовите `NetworkStateChanged()`:

```cpp
auto* weapons = pWeaponServices->m_hMyWeapons();
// ... изменение данных ...
pWeaponServices->m_hMyWeapons.NetworkStateChanged();
```

### 3. Разница между Controller и Pawn

- **Controller** - логика игрока (счет, деньги, имя)
- **Pawn** - физическое тело (позиция, здоровье, оружие)

Controller существует всегда, Pawn создается при спавне.

### 4. Проверка жизни игрока

```cpp
// Правильно
if (pPawn && pPawn->IsAlive()) {
    // ...
}

// Или через контроллер
if (pController->m_bPawnIsAlive()) {
    // ...
}
```

### 5. Работа с Handle

`CHandle<T>` - это умный указатель. Используйте `.Get()` для получения объекта:

```cpp
CHandle<CCSPlayerPawn> hPawn = pController->m_hPlayerPawn;
CCSPlayerPawn* pPawn = hPawn.Get();
```

### 6. Изменение сетевых переменных

Используйте оператор `()` или `=` для автоматической синхронизации:

```cpp
// Правильно - автоматическая синхронизация
pPawn->m_iHealth(100);
pPawn->m_iHealth = 100;

// Неправильно - нет синхронизации
int* pHealth = &pPawn->m_iHealth();
*pHealth = 100;
```


## Частые ошибки

### 1. Забыли проверить указатель

```cpp
// ПЛОХО
CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
pPawn->m_iHealth(100); // Может быть nullptr!

// ХОРОШО
CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
if (pPawn) {
    pPawn->m_iHealth(100);
}
```

### 2. Изменение данных без синхронизации

```cpp
// ПЛОХО
auto* weapons = pWeaponServices->m_hMyWeapons();
weapons->AddToTail(hWeapon);
// Забыли вызвать NetworkStateChanged!

// ХОРОШО
auto* weapons = pWeaponServices->m_hMyWeapons();
weapons->AddToTail(hWeapon);
pWeaponServices->m_hMyWeapons.NetworkStateChanged();
```

### 3. Путаница между Controller и Pawn

```cpp
// ПЛОХО - у контроллера нет позиции
Vector pos = pController->GetAbsOrigin(); // Ошибка!

// ХОРОШО
CCSPlayerPawn* pPawn = pController->GetPlayerPawn();
if (pPawn) {
    Vector pos = pPawn->GetAbsOrigin();
}
```

### 4. Неправильное использование Handle

```cpp
// ПЛОХО
CHandle<CCSPlayerPawn> hPawn = pController->m_hPlayerPawn;
hPawn->m_iHealth(100); // Ошибка! Handle не имеет оператора ->

// ХОРОШО
CHandle<CCSPlayerPawn> hPawn = pController->m_hPlayerPawn;
CCSPlayerPawn* pPawn = hPawn.Get();
if (pPawn) {
    pPawn->m_iHealth(100);
}
```


## Дополнительные классы

### CGameSceneNode

Узел сцены для позиции и углов сущности (подробнее см. раздел "CBaseEntity").

```cpp
SCHEMA_FIELD(CEntityInstance*, m_pOwner)          // Владелец узла
SCHEMA_FIELD(CGameSceneNode*, m_pParent)          // Родительский узел
SCHEMA_FIELD(CGameSceneNode*, m_pChild)           // Дочерний узел
SCHEMA_FIELD(CNetworkOriginCellCoordQuantizedVector, m_vecOrigin) // Локальная позиция
SCHEMA_FIELD(QAngle, m_angRotation)               // Локальные углы
SCHEMA_FIELD(float, m_flScale)                    // Локальный масштаб
SCHEMA_FIELD(float, m_flAbsScale)                 // Абсолютный масштаб
SCHEMA_FIELD(Vector, m_vecAbsOrigin)              // Абсолютная позиция
SCHEMA_FIELD(QAngle, m_angAbsRotation)            // Абсолютные углы
SCHEMA_FIELD(Vector, m_vRenderOrigin)             // Позиция для рендеринга

// Получить матрицу трансформации Entity->World
matrix3x4_t EntityToWorldTransform();
```

**Пример:**

```cpp
// Получить позицию через SceneNode
CGameSceneNode* pSceneNode = pPawn->m_CBodyComponent->m_pSceneNode;
if (pSceneNode) {
    Vector pos = pSceneNode->m_vecAbsOrigin();
    QAngle angles = pSceneNode->m_angAbsRotation();
    
    // Изменить масштаб
    pSceneNode->m_flScale(2.0f); // Увеличить в 2 раза
}
```

### CCollisionProperty

Свойства коллизии сущности.

```cpp
// Атрибуты коллизии
struct VPhysicsCollisionAttribute_t {
    uint8 m_nCollisionGroup;               // Группа коллизии
    uint8 m_nCollisionFunctionMask;        // Маска функций коллизии
    uint64 m_nInteractsAs;                 // С чем взаимодействует (как)
    uint64 m_nInteractsWith;               // С чем взаимодействует (с чем)
    uint64 m_nInteractsExclude;            // Исключения взаимодействия
    uint32 m_nEntityId;                    // ID сущности
    uint32 m_nOwnerId;                     // ID владельца
    uint16 m_nHierarchyId;                 // ID иерархии
    uint8 m_nCollisionGroupOverride;       // Переопределение группы коллизии
};

// Доступ через m_pCollision->m_collisionAttribute()
```

**Пример:**

```cpp
// Изменить группу коллизии
pPawn->SetCollisionGroup(COLLISION_GROUP_DEBRIS);

// Получить группу коллизии
uint8 group = pPawn->GetCollisionGroup();
```

### CBodyComponent

Компонент тела сущности.

```cpp
SCHEMA_FIELD(CGameSceneNode*, m_pSceneNode)       // Узел сцены
```

### CBodyComponentSkeletonInstance

Компонент тела со скелетом.

```cpp
// Наследует CBodyComponent
// Дополнительных полей нет в схеме
```

### CSkeletonInstance

Экземпляр скелета (наследует CGameSceneNode).

```cpp
SCHEMA_FIELD(CModelState, m_modelState)           // Состояние модели
```

### CModelState

Состояние модели.

```cpp
SCHEMA_FIELD(CUtlSymbolLarge, m_ModelName)        // Имя модели
SCHEMA_FIELD(uint64, m_MeshGroupMask)             // Маска групп мешей
```

### CBaseAnimGraphController

Контроллер анимационного графа.

```cpp
SCHEMA_FIELD(float, m_flPlaybackRate)             // Скорость воспроизведения анимации
```

### CBodyComponentBaseAnimGraph

Компонент тела с анимационным графом.

```cpp
SCHEMA_FIELD(CBaseAnimGraphController, m_animationController) // Контроллер анимации
```

### CBaseViewModel

Модель от первого лица (руки, оружие).

```cpp
SCHEMA_FIELD(CUtlSymbolLarge, m_sVMName)          // Имя модели от первого лица
```

### CGlowProperty

Свойства свечения сущности.

```cpp
SCHEMA_FIELD(Vector, m_fGlowColor)                // Цвет свечения (RGB, 0.0-1.0)
SCHEMA_FIELD(int, m_iGlowType)                    // Тип свечения
SCHEMA_FIELD(int, m_iGlowTeam)                    // Команда для свечения
SCHEMA_FIELD(int, m_nGlowRange)                   // Дальность свечения
SCHEMA_FIELD(int, m_nGlowRangeMin)                // Минимальная дальность
SCHEMA_FIELD(Color, m_glowColorOverride)          // Переопределение цвета
SCHEMA_FIELD(bool, m_bFlashing)                   // Мигает ли
SCHEMA_FIELD(bool, m_bGlowing)                    // Светится ли
```

**Пример:**

```cpp
// Включить свечение (требуется доступ к CGlowProperty)
// Обычно через m_Glow() если поле есть в классе
```

### GameTick_t

Игровой тик (обертка для int32).

```cpp
SCHEMA_FIELD(int32_t, m_Value)                    // Значение тика
```

### AmmoIndex_t

Индекс типа патронов (обертка для int8).

```cpp
SCHEMA_FIELD(int8_t, m_Value)                     // Значение индекса
```

### CNetworkVelocityVector

Сетевой вектор скорости.

```cpp
SCHEMA_FIELD(float, m_vecX)                       // X компонента
SCHEMA_FIELD(float, m_vecY)                       // Y компонента
SCHEMA_FIELD(float, m_vecZ)                       // Z компонента
```

### CNetworkOriginCellCoordQuantizedVector

Квантованный вектор позиции для сети.

```cpp
SCHEMA_FIELD(uint16, m_cellX)                     // Ячейка X
SCHEMA_FIELD(uint16, m_cellY)                     // Ячейка Y
SCHEMA_FIELD(uint16, m_cellZ)                     // Ячейка Z
SCHEMA_FIELD(uint16, m_nOutsideWorld)             // Вне мира
SCHEMA_FIELD(float, m_vecX)                       // X координата
SCHEMA_FIELD(float, m_vecY)                       // Y координата
SCHEMA_FIELD(float, m_vecZ)                       // Z координата
```

### EmitSound_t

Структура для воспроизведения звука.

```cpp
struct EmitSound_t {
    const char* m_pSoundName;              // Имя звука
    Vector m_vecOrigin;                    // Позиция звука
    float m_flVolume;                      // Громкость (0.0-1.0)
    float m_flSoundTime;                   // Задержка звука (секунды)
    CEntityIndex m_nSpeakerEntity;         // Индекс сущности-источника
    SoundEventGuid_t m_nForceGuid;         // Принудительный GUID
    CEntityIndex m_nSourceSoundscape;      // Источник звукового ландшафта
    uint16 m_nPitch;                       // Высота тона (обычно не используется)
    uint8 m_nFlags;                        // Флаги
};
```

### SpawnPoint

Точка спавна.

```cpp
SCHEMA_FIELD(bool, m_bEnabled)                    // Включена ли точка спавна
```

### CCSGO_TeamPreviewCharacterPosition

Позиция персонажа в предпросмотре команды.

```cpp
SCHEMA_FIELD(int32, m_nVariant)                   // Вариант
SCHEMA_FIELD(int32, m_nRandom)                    // Случайное значение
SCHEMA_FIELD(int32, m_nOrdinal)                   // Порядковый номер
SCHEMA_FIELD(CUtlString, m_sWeaponName)           // Имя оружия
SCHEMA_FIELD(uint64, m_xuid)                      // XUID игрока
SCHEMA_FIELD_POINTER(CEconItemView, m_agentItem) // Предмет агента
SCHEMA_FIELD_POINTER(CEconItemView, m_glovesItem) // Предмет перчаток
SCHEMA_FIELD_POINTER(CEconItemView, m_weaponItem) // Предмет оружия
```

### CBaseModelEntity

Базовый класс для сущностей с моделью (наследует CBaseEntity).

```cpp
// Дополнительные поля для моделей
// Точные поля зависят от версии SDK
```

## Полезные глобальные переменные и утилиты

### Глобальные переменные

```cpp
extern CEntitySystem* g_pEntitySystem;  // Система сущностей (для поиска и итерации)
```

### Утилиты для работы с сущностями

#### UTIL_GetEntityByIndex

Получить сущность по индексу.

```cpp
CEntityInstance* UTIL_GetEntityByIndex(int index);
```

**Параметры:**
- `index` - индекс сущности (1-2048)

**Возвращает:** Указатель на `CEntityInstance` или `nullptr`

**Пример:**

```cpp
// Получить сущность с индексом 5
CEntityInstance* pEntity = UTIL_GetEntityByIndex(5);
if (pEntity) {
    CBaseEntity* pBase = (CBaseEntity*)pEntity;
    // Работа с сущностью
}
```

#### UTIL_FindEntityByClassname

Найти первую сущность по имени класса.

```cpp
CEntityInstance* UTIL_FindEntityByClassname(const char* name);
```

**Параметры:**
- `name` - имя класса (например, "planted_c4", "weapon_ak47")

**Возвращает:** Указатель на первую найденную сущность или `nullptr`

**Пример:**

```cpp
// Найти бомбу
auto* pBomb = (CPlantedC4*)UTIL_FindEntityByClassname("planted_c4");
if (pBomb) {
    // Работа с бомбой
}
```

#### UTIL_FindEntityByClassnameAll

Найти все сущности по имени класса.

```cpp
std::vector<CEntityInstance*> UTIL_FindEntityByClassnameAll(const char* name);
```

**Параметры:**
- `name` - имя класса

**Возвращает:** Вектор указателей на все найденные сущности

**Пример:**

```cpp
// Найти все гранаты
auto grenades = UTIL_FindEntityByClassnameAll("hegrenade_projectile");
for (auto* pEntity : grenades) {
    CBaseEntity* pGrenade = (CBaseEntity*)pEntity;
    // Работа с гранатой
}

// Найти всех игроков (альтернативный способ)
auto players = UTIL_FindEntityByClassnameAll("player");
for (auto* pEntity : players) {
    CCSPlayerPawn* pPawn = (CCSPlayerPawn*)pEntity;
    if (pPawn && pPawn->IsAlive()) {
        // Работа с игроком
    }
}
```

#### UTIL_FindEntityByEHandle

Найти сущность по EHandle.

```cpp
CEntityInstance* UTIL_FindEntityByEHandle(CEntityInstance* pFind);
```

**Параметры:**
- `pFind` - сущность для поиска

**Возвращает:** Указатель на сущность или `nullptr`

#### UTIL_FindAllEntitiesByDesignerName

Найти все сущности по имени дизайнера (из карты).

```cpp
std::vector<CEntityInstance*> UTIL_FindAllEntitiesByDesignerName(const char* name);
```

**Параметры:**
- `name` - имя дизайнера (например, "weapon_")

**Возвращает:** Вектор указателей на все найденные сущности

### Работа с виртуальными функциями

#### CALL_VIRTUAL

Макрос для вызова виртуальных функций.

```cpp
CALL_VIRTUAL(returnType, index, thisPtr, ...args)
```

**Параметры:**
- `returnType` - тип возвращаемого значения
- `index` - индекс виртуальной функции в vtable
- `thisPtr` - указатель на объект (this)
- `...args` - аргументы функции

**Примеры:**

```cpp
// Вызов Respawn (индекс 259)
CALL_VIRTUAL(void, 259, pController);

// Вызов ChangeTeam (индекс 99)
CALL_VIRTUAL(void, 99, pController, 2); // 2 = T

// Вызов CommitSuicide (индекс 380)
CALL_VIRTUAL(void, 380, pPawn, false, true); // bExplode=false, bForce=true

// Вызов Teleport (индекс 167)
Vector pos(0, 0, 100);
QAngle ang(0, 90, 0);
CALL_VIRTUAL(void, 167, pPawn, &pos, &ang, nullptr);

// Вызов CollisionRulesChanged (индекс 190)
CALL_VIRTUAL(void, 190, pEntity);
```

### Важные индексы виртуальных функций

```cpp
// CBasePlayerController
99  - ChangeTeam(int iTeam)
259 - Respawn()

// CBaseEntity
167 - Teleport(const Vector* pos, const QAngle* ang, const Vector* vel)
190 - CollisionRulesChanged()

// CBasePlayerPawn
380 - CommitSuicide(bool bExplode, bool bForce)

// CCSPlayer_WeaponServices
24  - DropWeapon(CBasePlayerWeapon* pWeapon, Vector* pTarget, Vector* pVel)

// CCSPlayer_ItemServices
23  - RemoveWeapons()
```


---

**Версия:** 2.0
**Дата:** 2026-01-19 
