# 🚀 Руководство по моддингу Astroneer (на русском)

> **Важно:** Astroneer не имеет официальной поддержки модов — всё это создано силами сообщества. Этот сайт никак не связан с компанией System Era.
> Если нужна помощь — заходи на [Discord сервер по моддингу Astroneer](https://discord.gg/bBqdVYxu4k).

---

## 📋 Содержание

1. [Как устроены моды в Astroneer](#как-устроены-моды)
2. [Необходимые инструменты](#необходимые-инструменты)
3. [Первоначальная настройка](#первоначальная-настройка)
4. [Твой первый мод — изменение существующего предмета](#твой-первый-мод)
5. [Настройка Modding Kit (для сложных модов)](#настройка-modding-kit)
6. [Создание нового предмета через Unreal Editor](#создание-нового-предмета)
7. [Как добавить предмет в принтер](#как-добавить-предмет-в-принтер)
8. [Как создать платформу](#как-создать-платформу)
9. [Миссии и панели миссий](#миссии-и-панели-миссий)
10. [Придумывание названий для мода](#придумывание-названий)
11. [Частые вопросы и проблемы](#частые-вопросы)
12. [Таблица: где искать файлы предметов](#таблица-файлов-предметов)

---

## Как устроены моды

Astroneer работает на движке **Unreal Engine 4.27.2** (немного модифицированном). Официальной поддержки модов нет, но сообщество разработало собственные инструменты.

### Что такое .pak файл?

Мод — это файл с расширением `.pak`. Он похож на ZIP-архив: внутри лежат файлы с данными игры. Когда ты кидаешь `.pak` в нужную папку, игра подхватывает его при запуске и применяет изменения.

Файлы загружаются после основного файла игры `pakchunk0-WindowsNoEditor.pak` и могут:
- Заменять существующие части игры (например, изменять характеристики предметов)
- Добавлять новые предметы и файлы

Внутри каждого `.pak` файла обязательно должен быть файл `metadata.json` — он рассказывает загрузчику модов, что это за мод и как его правильно применить.

### Что такое .uasset файл?

Все игровые данные (числа, текстуры, модели, часть кода) хранятся в файлах `.uasset`. Они бывают двух видов:

| Тип | Где встречается | Как открыть |
|-----|----------------|-------------|
| **Uncooked** (несжатый) | В Unreal Editor во время разработки | Прямо в Unreal Editor |
| **Cooked** (сжатый) | В готовой игре | Только через UAssetGUI или аналоги |

> ⚠️ Начиная с обновления Xenobiology (Snail Update), каждый `.uasset` файл сопровождается файлом `.uexp` — они всегда должны лежать вместе!

### Структура папок внутри .pak

```
root/
├── Astro/
│   └── Content/          ← здесь всё важное
│       ├── Items/
│       │   ├── ItemTypes/  ← данные предметов (_IT файлы)
│       │   └── ...
│       ├── Components_Small/
│       ├── Components_Medium/
│       ├── Components_Large/
│       └── Mods/           ← сюда кладутся файлы твоих модов
│           └── ТвойНик/
│               └── НазваниеМода/
└── Engine/
```

> **Важная деталь:** В самих файлах пути пишутся как `/Game/...` — это сокращение для `/Astro/Content/...`. Например, `/Game/Items/ItemTypes/FloodLight_IT` = файл `Astro/Content/Items/ItemTypes/FloodLight_IT.uasset`.

### Как устроены предметы в Astroneer

Каждый предмет состоит из **двух файлов**:

| Файл | Суффикс | Что хранит | Где лежит |
|------|---------|-----------|-----------|
| **ItemType** | `_IT` | Рецепт крафта, иконка, описание, стоимость в байтах | `/Game/Items/ItemTypes/` |
| **PhysicalItem** | `_BP` | Сам объект в мире игры (3D модель, слоты, логика) | `/Game/Components_*/` или `/Game/Items/` |

---

## Необходимые инструменты

### Для простых модов (изменение числовых значений)

| Инструмент | Для чего | Ссылка |
|-----------|---------|--------|
| **repak** | Распаковка и упаковка `.pak` файлов | [GitHub](https://github.com/trumank/repak) |
| **UAssetGUI** | Просмотр и редактирование `.uasset` файлов | [GitHub](https://github.com/atenfyr/UAssetGUI/releases) |

### Для сложных модов (новые предметы, новая логика)

| Инструмент | Для чего |
|-----------|---------|
| **Visual Studio 2017+** | Нужен для работы с Unreal Engine |
| **Unreal Engine 4.27.2** | Редактор для создания новых ассетов |
| **AstroTechies ModdingKit** | Шаблон проекта с игровыми классами |
| **Wwise 2019.1.8** | Только если хочешь добавить звуки (необязательно) |

### Загрузчики модов (для установки)

- **astro_modloader** (Rust) — [GitHub](https://github.com/AstroTechies/astro_modloader)
- **AstroModLoader Classic** — [GitHub](https://github.com/atenfyr/AstroModLoader-Classic)

Оба совместимы с большинством модов. Мод устанавливается просто: перетащи `.pak` файл на окно загрузчика.

---

## Первоначальная настройка

### Шаг 1: Создай папку для работы

Создай папку `AstroneerModding` в удобном месте (например, в Документах). На диске должно быть **минимум 10 ГБ свободного места** (если планируешь работать с Unreal Engine — ещё +30 ГБ).

### Шаг 2: Установи repak

1. Скачай [repak_for_astroneer.zip](https://astroneermodding.readthedocs.io/en/latest/_downloads/c73277f4c6d7d821b10c49cc49e7973f/repak_for_astroneer.zip)
2. Распакуй в папку `AstroneerModding`
3. В папке `repak_for_astroneer` найди два `.bat` файла
4. Кликни правой кнопкой по каждому → **Создать ярлык**
5. Нажми `Win + R` → введи `Shell:sendto` → перетащи оба ярлыка в открывшееся окно
6. Переименуй ярлыки: `Repack folder with repak` и `Unpack .pak with repak`

Теперь ты сможешь упаковывать/распаковывать `.pak` прямо из контекстного меню!

### Шаг 3: Извлеки файлы игры

1. Создай папку `GameFiles` внутри `AstroneerModding`
2. Найди файл `pakchunk0-WindowsNoEditor.pak` по пути:
   - Steam: `C:\Program Files (x86)\Steam\steamapps\common\ASTRONEER\Astro\Content\Paks\`
   - Или: ПКМ на игре в Steam → Свойства → Локальные файлы → Обзор
3. Скопируй этот файл в папку `GameFiles`
4. ПКМ на файле → **Отправить → Unpack .pak with repak**

Распаковка займёт пару минут. После этого у тебя будут все игровые файлы для изучения.

### Шаг 4: Установи UAssetGUI

1. Скачай последнюю версию с [GitHub](https://github.com/atenfyr/UAssetGUI/releases)
2. Распакуй в папку `AstroneerModding`
3. Открой любой `.uasset` файл, выбери UAssetGUI как программу по умолчанию

> **Проверка:** Открой `AstroneerModding\GameFiles\WindowsNoEditor\Astro\Content\Items\ItemTypes\FloodLight_IT.uasset` — должен открыться UAssetGUI.

---

## Твой первый мод

Разберём на примере: изменим стоимость Прожектора (Floodlight) в байтах.

### Шаг 1: Создай структуру папок мода

```
AstroneerModding/
└── TutorialMod/
    └── 000-TutorialMod-0.1.0_P/   ← это будущее имя .pak файла
        ├── metadata.json
        └── Astro/
            └── Content/
                └── Items/
                    └── ItemTypes/
                        ├── FloodLight_IT.uasset  ← скопировано из GameFiles
                        └── FloodLight_IT.uexp    ← скопировано из GameFiles
```

> ⚠️ Структура папок должна **точно совпадать** с оригинальной в игре! Оба файла (`.uasset` и `.uexp`) всегда держи вместе.

> ⚠️ **Никогда не редактируй файлы прямо в `GameFiles`!** Только копируй их в папку мода.

### Шаг 2: Создай файл metadata.json

В папке `000-TutorialMod-0.1.0_P` создай файл `metadata.json`:

```json
{
    "schema_version": 2,
    "name": "Tutorial Mod",
    "mod_id": "TutorialMod",
    "author": "ТВОЁ_ИМЯ",
    "description": "Мой первый мод.",
    "version": "0.1.0",
    "sync": "serverclient"
}
```

Параметр `sync` определяет, где должен быть установлен мод:
- `"serverclient"` — на клиенте и на сервере
- `"client"` — только на клиенте
- `"server"` — только на сервере

### Шаг 3: Измени нужный параметр

1. Открой `FloodLight_IT.uasset` из **папки мода** в UAssetGUI
2. При первом открытии выбери версию движка: **4.27**
3. Перейди: **View → Expand All**
4. Найди экспорт `ItemCatalogData` — там хранится стоимость предмета
5. Измени значение `UnlockCost` на нужное (например, `500`)
6. Сохрани: `Ctrl + S`

> Поиск по файлу: `Ctrl + F` — очень полезно в больших файлах!

### Шаг 4: Упакуй мод

1. Вернись в папку `TutorialMod`
2. ПКМ на папке `000-TutorialMod-0.1.0_P` → **Отправить → Repack folder with repak**
3. Появится файл `000-TutorialMod-0.1.0_P.pak`

### Шаг 5: Установи и проверь

1. Перетащи `.pak` файл на окно загрузчика модов
2. Убедись, что мод включён (галочка стоит)
3. Запусти игру → открой каталог → проверь стоимость Прожектора

---

## Настройка Modding Kit

Это нужно, если ты хочешь создавать **новые предметы**, добавлять **новую логику**, делать что-то сложнее, чем просто менять числа.

### Что понадобится

- Минимум **40 ГБ свободного места**
- **Visual Studio 2017** или новее
- **Unreal Engine 4.27.2**
- **AstroTechies ModdingKit**

### Шаг 1: Установи Visual Studio

1. Скачай [Visual Studio](https://visualstudio.microsoft.com/downloads/) (Community версия — бесплатная)
2. При установке отметь два пакета:
   - **Desktop & Mobile → Desktop development with C++**
   - **Gaming → Game development with C++**

### Шаг 2: Установи Unreal Engine 4.27.2

1. Открой Epic Games Launcher
2. Перейди во вкладку **Unreal Engine → Library**
3. Нажми `+` рядом с версиями движка
4. Выбери **4.27.2** и установи

### Шаг 3: Скачай ModdingKit

1. Перейди на [GitHub AstroTechies/ModdingKit](https://github.com/AstroTechies/ModdingKit)
2. Нажми **Code → Download ZIP**
3. Распакуй в любое удобное место

### Шаг 4: Добавить звуки (необязательно — только если нужны звуки)

1. Перейди на [сайт Wwise](https://www.audiokinetic.com/en/products/wwise)
2. Скачай и установи версию **2019.1.8.7173**
3. Во время установки выбери: Authoring, SDK (C++), Windows, macOS, Visual Studio 2017/2019
4. В Wwise перейди во вкладку Unreal Engine → Browse for Project → выбери `Astro.uproject`
5. Нажми **Integrate Wwise into project**

### Шаг 5: Генерация файлов Visual Studio (необязательно)

Если хочешь открывать проект в Visual Studio:
1. ПКМ на файле `Astro.uproject` → **Generate Visual Studio project files**
2. Открой появившийся файл `Astro.sln`

Или через командную строку:
```
"C:\Program Files\Epic Games\UE_4.27\Engine\Binaries\DotNET\UnrealBuildTool.exe" -projectfiles -project="ПУТЬ_К_ПРОЕКТУ\Astro.uproject" -game -rocket -progress
```

### Шаг 6: Первый запуск

1. Дважды кликни на `Astro.uproject`
2. Если появится вопрос "Пересобрать недостающие модули?" — нажми **Да** и подожди
3. Если файл не открывается: ПКМ → Открыть с помощью → найди `UE_INSTALL_PATH\Engine\Binaries\Win64\UE4Editor.exe`

---

## Создание нового предмета

После настройки Modding Kit ты можешь создавать полностью новые предметы.

### Шаг 1: Создай папку мода в редакторе

В **Content Browser** открой папку `Mods`, создай папку с твоим ником, внутри — папку `TutorialMod`.

### Шаг 2: Создай PhysicalItem (объект в мире)

1. В `TutorialMod` нажми **Create a blueprint class**
2. Нажми **Open All Classes**
3. Найди и выбери класс `PhysicalItem`
4. Назови файл `ExampleItem_BP` (суффикс `_BP` обязателен!)
5. Дважды кликни на созданный файл

#### Импорт 3D модели

1. Скачай пример модели [`exampleMesh.fbx`](https://astroneermodding.readthedocs.io/en/latest/_downloads/66c51874e321e45a4c8cddbe2c0ea702/exampleMesh.fbx)
2. Перетащи `.fbx` в Content Browser → нажми **Import All** (настройки не меняй)
3. Открой импортированную модель → в деталях найди **Has Navigation Data** → **сними галочку!**

> ⛔ Если не снять галочку с **Has Navigation Data** — игра будет вылетать при загрузке!

4. Снова открой `ExampleItem_BP` → кликни на **StaticMeshComponent**
5. В Details найди поле **Static Mesh** → выбери импортированную модель
6. Нажми **Compile** → **Save**

### Шаг 3: Создай ItemType (карточка предмета)

1. Снова нажми **Create a blueprint class**
2. Выбери класс `ItemType`
3. Назови файл `ExampleItem_IT` (суффикс `_IT` обязателен!)
4. Открой и настрой:

| Параметр | Что делать |
|---------|-----------|
| **Pickup Actor** | Выбери `ExampleItem_BP` |
| **Construction Recipe → Ingredients → +** | Добавь ингредиент (например, Astronium x1) |
| **Catalog Data** | Выбери `Item Catalog Data` |
| **Is Base Item** | Сними галочку (если используешь существующую строку каталога) |
| **Base Item Type** | Выбери, рядом с каким предметом показывать в каталоге |
| **Variation Sequence Number** | Порядок в строке каталога |
| **Catalog Mesh** | Модель для отображения в каталоге |
| **Crate Overlay Texture** | Иконка на упаковке (например, `ui_icon_package_drill`) |
| **Widget Icon** | Иконка в каталоге (например, `ui_icon_comp_drill`) |

В разделе **Control Symbol** заполни:

```
Name:             Example Item
All caps Name:    EXAMPLE ITEM  
Tooltip Subtitle: Example Item
Description:      Это пример предмета.
```

> ⚠️ Если включить `Is Base Item` и одновременно задать `Base Item Type` равным другому предмету — предмет **не появится** в каталоге!

### Шаг 4: Свяжи ItemType и PhysicalItem

1. Открой `ExampleItem_BP` снова
2. Нажми на **ItemComponent**
3. В Details → **Item Component → Item Type** → выбери `ExampleItem_IT`

---

## Как добавить предмет в принтер

Чтобы предмет можно было напечатать, нужно правильно настроить `metadata.json`.

### Добавление в рюкзак-принтер (Backpack Printer)

```json
{
    "schema_version": 2,
    "name": "My Item Mod",
    "mod_id": "MyItemMod",
    "author": "ТвойНик",
    "description": "Новый предмет.",
    "version": "1.0.0",
    "sync": "serverclient",
    "integrator": {
        "item_list_entries": {
            "/Game/Items/ItemTypes/MasterItemList": {
                "ItemTypes": [
                    "/Game/Mods/ТвойНик/НазваниеМода/ExampleItem_IT"
                ]
            },
            "/Game/Items/BackpackRail": {
                "PrinterComponent.Blueprints": [
                    "/Game/Mods/ТвойНик/НазваниеМода/ExampleItem_BP"
                ]
            }
        }
    }
}
```

### Что делает каждая строка

| Ключ в integrator | Что означает |
|-------------------|-------------|
| `MasterItemList → ItemTypes` | Регистрирует ItemType в общем списке предметов игры |
| `BackpackRail → PrinterComponent.Blueprints` | Добавляет предмет в список для печати в рюкзаке |

### Добавление в другие принтеры

Чтобы добавить предмет в принтеры других размеров, используй пути к соответствующим активам:

| Принтер | Примерный путь в metadata |
|---------|--------------------------|
| Маленький принтер | `/Game/Components_Small/Printer_Breadboards_T1` |
| Средний принтер | `/Game/Components_Medium/Printer_Breadboards_T2` |
| Большой принтер | `/Game/Components_Large/Printer_Breadboards_T3` |
| Гараж (транспорт) | `/Game/Components_Large/Printer_Vehicles` |

---

## Как создать платформу

Платформы в Astroneer — это тоже обычные предметы с классом `PhysicalItem`. Разница в базовом классе, от которого они наследуются.

### Где искать существующие платформы

Файлы платформ лежат в `/Game/Slots/Pads/`. Примеры:

| Платформа | Путь к файлу |
|-----------|-------------|
| Medium Platform A | `/Game/Slots/Pads/Platform_Standard_T2x1_P4` |
| Large Platform A | `/Game/Slots/Pads/Platform_Standard_T3x1_P4` |
| Extra Large Platform A | `/Game/Slots/Pads/Platform_Standard_T4x1_P8` |
| Curved Platform | `/Game/Slots/Pads/Platfrom_Curve_T3x1_P4` |

### Как сделать кастомную платформу (через UAssetGUI)

1. Найди похожую платформу в `GameFiles` (в папке `Astro/Content/Slots/Pads/`)
2. Скопируй оба файла (`.uasset` + `.uexp`) в папку мода, сохранив структуру путей
3. Открой в UAssetGUI
4. Измени нужные параметры (количество слотов, размер и т.д.)
5. Добавь ItemType для этой платформы в `metadata.json` → `MasterItemList`

### Как сделать платформу через Unreal Editor

1. Создай **Blueprint class** на основе существующей платформы (наследуй от класса нужного размера)
2. Настрой количество и расположение слотов в **Component** разделе
3. Свяжи с ItemType так же, как и для обычного предмета

---

## Миссии и панели миссий

### Добавление новой миссии

Миссии добавляются через `metadata.json` в разделе `integrator`. Для этого нужен **Mission Trailhead** — специальный актор, который запускает миссию.

Пример в `metadata.json`:
```json
"integrator": {
    "mission_trailheads": [
        "/Game/Mods/ТвойНик/НазваниеМода/MyMissionTrailhead"
    ]
}
```

### Панель миссий на предмете

Если хочешь, чтобы предмет запускал миссию при взаимодействии:

1. В Unreal Editor добавь на `PhysicalItem` компонент **Mission Panel**
2. Создай точки поставки (Supply Drop Points) и привяжи их к мисии
3. Это позволяет игре доставлять предметы в рамках миссии прямо к твоему предмету

---

## Упаковка и финальные шаги

### Запечь (cook) мод в Unreal Editor

1. Сохрани все изменённые файлы (обязательно!)
2. Нажми **File → Cook Content for Windows**
3. Дождись завершения

> 💡 Чтобы ускорить повторную запекку: **Edit → Project Settings → Project → Packaging** → снять галочку с **Full Rebuild**

### Собери папку мода

```
000-TutorialMod-0.1.0_P/
├── metadata.json
└── Astro/
    └── Content/
        └── Mods/
            └── ТвойНик/
                └── TutorialMod/
                    ├── ExampleItem_BP.uasset
                    ├── ExampleItem_BP.uexp
                    ├── ExampleItem_IT.uasset
                    └── ExampleItem_IT.uexp
```

> ⚠️ Копируй файлы из `Saved/Cooked/WindowsNoEditor/Astro/Content/Mods/...` — это **запечённые** файлы. Из `Content/Mods/...` — **не запечённые**, игра их не примет!

### Упакуй в .pak

ПКМ на папке `000-TutorialMod-0.1.0_P` → **Отправить → Repack folder with repak**

---

## Придумывание названий

### Правила именования

- Только латинские буквы (A–Z, a–z) и цифры (0–9)
- Никаких пробелов, никаких символов (кроме одной точки для разделения ID мода и ID автора)

### Что нужно придумать

| Что | Описание | Пример |
|-----|---------|--------|
| **Author ID** | Твой уникальный ник | `Konsti`, `atenfyr` |
| **Mod ID** | Короткое ID мода | `RocketLauncher`, `MoreTradables` |
| **Mod Name** | Полное читаемое название | `Rocket Launcher Mod` |

### Хорошие примеры Mod ID

- `RocketLauncher` → для мода "Rocket Launcher"
- `LavaLamp` → для мода "Lava Lamp"
- `MoreTradables` → для мода "Mo' Tradables"

### Плохие примеры

- `QTRTG` — непонятно, что делает мод
- `Pumpkin` — непонятно, что именно с тыквой
- `6A1S` — что это вообще?

### Формат имени .pak файла

```
ПРИОРИТЕТ-ModID-ВЕРСИЯ_P.pak
```

Пример: `000-TutorialMod-0.1.0_P.pak`

- **ПРИОРИТЕТ** — трёхзначное число (000–999), определяет порядок загрузки
- **ModID** — ID твоего мода
- **ВЕРСИЯ** — версия в формате X.Y.Z

### Уникальный Mod ID (если совпадают имена)

Если другой автор уже использует такой же Mod ID, можно добавить свой Author ID через точку:
```
PickupRovers.Konsti
```

---

## Частые вопросы

### Общие вопросы

**Можно ли использовать моды на выделенных серверах?**
Да! Для этого установи AstroModLoader Classic в корневую папку сервера и запусти его с параметром `--server`.

**Работают ли моды на Linux?**
Да. Используй `astro_modloader` (Rust) — у него есть нативный Linux-бинарник. AstroModLoader Classic работает через Wine.

**Совместимы ли старые моды с новой версией игры?**
Моды, выпущенные до **21 ноября 2025 года**, несовместимы с текущей версией — из-за обновления движка с UE4.23 до UE4.27 (обновление MEGATECH 1.36.42.0).

**Нужна ли мне лицензионная копия игры?**
Да — поддержку пиратских копий сообщество не оказывает.

**Что такое Mod Integrator?**
Это программа, которая автоматически изменяет файлы базовой игры от имени нескольких модов, избегая конфликтов. Работает каждый раз, когда ты добавляешь или удаляешь мод. Результат сохраняется в файл `999-AstroModIntegrator_P.pak`.

### Вопросы по файлам игры

**Как найти файл конкретного предмета?**
Используй поиск Windows или команду в командной строке:
```
findstr /S /I /M /C:"jetpack" *.uasset
```
Это найдёт все файлы, содержащие слово "jetpack".

**Почему в экспорте нет некоторых свойств?**
Это называется **delta serialization** — движок не записывает свойства, которые равны значению по умолчанию. Чтобы изменить такое свойство, просто добавь новую строку с нужным именем и значением в UAssetGUI.

**Как изменить рецепт существующего предмета?**
Открой файл `_IT` предмета в UAssetGUI → найди экспорт с рецептом → измени ингредиенты и их количество.

**Как добавить предмет в существующий принтер через UAssetGUI?**
Найди файл принтера → найди список `PrinterComponent.Blueprints` → добавь путь к своему `_BP` файлу. Или используй `integrator` в `metadata.json` — это безопаснее и не конфликтует с другими модами.

**Как сделать ресурс хранимым в канистрах?**
Открой файл ресурса в UAssetGUI и добавь нужный флаг в свойствах. Для обычных канистр и газовых — разные флаги.

**Как сделать предмет разблокированным по умолчанию в каталоге?**
В ItemType найди параметр, отвечающий за начальную разблокировку, и установи значение по умолчанию.

### Вопросы по Unreal Editor

**Ошибка "Astro could not be compiled" при открытии .uproject?**
Убедись, что Visual Studio установлен с пакетами C++ (см. раздел установки выше). Попробуй регенерировать файлы проекта.

**Мой предмет не появляется в каталоге?**
Проверь:
- Зарегистрирован ли ItemType в `MasterItemList` в `metadata.json`
- Не стоит ли `Is Base Item` = true при заданном `Base Item Type`

**Мой предмет перезаписывает другой в меню принтера?**
Убедись, что `Variation Sequence Number` не совпадает с уже существующими предметами в той же строке каталога.

**Как добавить слот к предмету?**
В Blueprint редакторе добавь компонент `SlotComponent` на `PhysicalItem`. Тип слота определяет, что в него можно класть.

**Как сделать, чтобы предмет потреблял или производил энергию?**
Добавь на `PhysicalItem` компонент `PowerComponent` и настрой параметры потребления/производства.

**Как добавить поддержку кислорода предмету?**
Добавь соответствующий компонент и настрой его параметры хранения кислорода.

**Как запустить код Blueprint при загрузке игры?**
Используй специальный Actor класс, который игра загружает при старте уровня. Подробности — в Discord.

**Можно ли тестировать моды прямо в Unreal Editor?**
Ограниченно — полную функциональность можно проверить только в самой игре.

---

## Таблица файлов предметов

Ниже — часть справочной таблицы соответствия названий предметов и путей к их файлам.

### Принтеры

| Название | ItemType (_IT) | PhysicalItem (_BP) |
|---------|---------------|-------------------|
| Small Printer | `/Game/Items/ItemTypes/Components/BreadboardPrinter_T1` | `/Game/Components_Small/Printer_Breadboards_T1` |
| Medium Printer | `/Game/Items/ItemTypes/Components/BreadboardPrinter_T2` | `/Game/Components_Medium/Printer_Breadboards_T2` |
| Large Printer | `/Game/Items/ItemTypes/Components/BreadboardPrinter_T3` | `/Game/Components_Large/Printer_Breadboards_T3` |
| Vehicle Bay | `/Game/Items/ItemTypes/Components/Printer_Vehicles` | `/Game/Components_Large/Printer_Vehicles` *(примерно)* |

### Платформы

| Название | ItemType путь |
|---------|--------------|
| Medium Platform A | `/Game/Items/ItemTypes/Components/Platform_Standard_T2x1_P4` |
| Medium Platform B | `/Game/Items/ItemTypes/Components/Breadboard_Tier2` |
| Large Platform A | `/Game/Items/ItemTypes/Components/Platform_Standard_T3x1_P4` |
| Extra Large Platform A | `/Game/Items/ItemTypes/Components/Platform_Standard_T4x1_P8` |
| Extra Large Platform B | `/Game/Items/ItemTypes/Components/Platform_Standard_T2x10_P4` |

### Хранилища и ресурсы

| Название | ItemType путь |
|---------|--------------|
| Medium Storage | `/Game/Items/ItemTypes/Components/StorageRack_Medium` |
| Large Storage | `/Game/Items/ItemTypes/Components/StorageRack_Large` |
| Extra Large Storage | `/Game/Items/ItemTypes/Components/StorageRack_XL` |
| Medium Resource Canister | `/Game/Items/ItemTypes/Components/MediumResourceCanister_IT` |
| Large Resource Canister | `/Game/Items/ItemTypes/Components/LargeResourceCanister_IT` |

### Популярные предметы

| Название | ItemType путь |
|---------|--------------|
| Floodlight | `/Game/Items/ItemTypes/FloodLight_IT` |
| RTG | `/Game/Items/ItemTypes/Components/RTG_T2` |
| QT-RTG | `/Game/Items/ItemTypes/Components/RTG_T1_IT` |
| Chemistry Lab | `/Game/Items/ItemTypes/Components/ResourceCombiner_IT` |
| Research Chamber | `/Game/Items/ItemTypes/Components/ResearchModule_Tier2` |
| Trade Platform | `/Game/Items/ItemTypes/Components/Trade_Module` |
| Packager | `/Game/Items/ItemTypes/Components/Repackager_T1` |
| Hoverboard | `/Game/Vehicles/Hoverboard_IT` |
| Hydrazine Jet Pack | `/Game/Items/ItemTypes/Components/HydrazineJetpack_IT` |

> 💡 Полная таблица всех предметов игры доступна в разделе FAQ оригинальной документации.

---

## Полезные ссылки

| Ресурс | Ссылка |
|--------|--------|
| Оригинальная документация | [astroneermodding.readthedocs.io](https://astroneermodding.readthedocs.io) |
| Discord сообщества | [discord.gg/bBqdVYxu4k](https://discord.gg/bBqdVYxu4k) |
| ModdingKit (GitHub) | [AstroTechies/ModdingKit](https://github.com/AstroTechies/ModdingKit) |
| UAssetGUI | [atenfyr/UAssetGUI](https://github.com/atenfyr/UAssetGUI/releases) |
| repak | [trumank/repak](https://github.com/trumank/repak) |
| astro_modloader | [AstroTechies/astro_modloader](https://github.com/AstroTechies/astro_modloader) |
| AstroModLoader Classic | [atenfyr/AstroModLoader-Classic](https://github.com/atenfyr/AstroModLoader-Classic) |

---

*Документ основан на материалах сообщества AstroTechies. Astroneer не имеет официальной поддержки модов.*
