# Полное руководство по дашбордам Home Assistant

Подробный гайд по созданию, настройке и организации дашбордов (панелей) в Home Assistant.

> **⚠️ Важное обновление (2026):** Если ты используешь YAML-конфигурацию дашбордов, старый формат `lovelace: mode: yaml` устарел и будет удалён в Home Assistant 2026.8. Новый формат описан в разделе ["Создание через YAML"](#создание-нового-дашборда). YAML-дашборды продолжат работать, просто нужно обновить формат конфигурации.

---

## 📋 Содержание

1. [Основы дашбордов](#1--основы-дашбордов)
2. [UI vs YAML — что выбрать?](#2--ui-vs-yaml--что-выбрать)
3. [Создание и управление дашбордами](#3--создание-и-управление-дашбордами)
4. [Представления (Views) — вкладки внутри дашборда](#4--представления-views--вкладки-внутри-дашборда)
5. [Типы представлений (View Types)](#5--типы-представлений-view-types)
6. [Карточки (Cards)](#6--карточки-cards)
7. [Организация по комнатам](#7--организация-по-комнатам)
8. [Организация по типам устройств](#8--организация-по-типам-устройств)
9. [Практические примеры](#9--практические-примеры)
10. [Лимиты и ограничения](#10--лимиты-и-ограничения)
11. [Советы и best practices](#11--советы-и-best-practices)

---

## 1. 📊 Основы дашбордов

### Что такое дашборд?

**Дашборд (Dashboard)** — это отдельная панель в Home Assistant, которая отображается в боковом меню (сайдбаре). Каждый дашборд может содержать несколько **представлений (views)** — вкладок внутри дашборда.

### Структура

```
Home Assistant
├── Дашборд "Обзор" (Overview)
│   ├── Представление "Главная"
│   ├── Представление "Освещение"
│   └── Представление "Безопасность"
├── Дашборд "Спальня"
│   ├── Представление "Устройства"
│   └── Представление "Климат"
└── Дашборд "Камеры"
    └── Представление "Все камеры"
```

### Терминология

| Термин | Описание | Где отображается |
|--------|----------|------------------|
| **Dashboard** | Отдельная панель | В боковом меню (сайдбаре) |
| **View** | Вкладка внутри дашборда | Вверху дашборда (горизонтальные вкладки) |
| **Card** | Карточка с информацией/управлением | Внутри представления |
| **Section** | Группа карточек (в Sections view) | Внутри представления |

### Дашборд по умолчанию

При первом запуске Home Assistant создаёт дашборд **"Overview"** (Обзор), который автоматически добавляет все обнаруженные устройства.

---

## 2. 🎨 UI vs YAML — что выбрать?

### Сравнительная таблица

| Критерий | UI (веб-интерфейс) | YAML (файлы) |
|----------|-------------------|--------------|
| **Простота** | ⭐⭐⭐⭐⭐ Очень просто | ⭐⭐ Нужно знать синтаксис |
| **Скорость создания** | ⭐⭐⭐⭐⭐ 5 минут | ⭐⭐⭐ 20-30 минут |
| **Визуальное редактирование** | ✅ Drag & drop | ❌ Только текст |
| **Предпросмотр** | ✅ Сразу видишь результат | ❌ Нужно сохранить и обновить |
| **Ошибки** | ✅ UI не даст ошибиться | ❌ Легко сделать синтаксическую ошибку |
| **Массовое копирование** | ❌ По одной карточке | ✅ Копируй весь блок |
| **Версионный контроль (Git)** | ❌ Сложно | ✅ Легко отслеживать изменения |
| **Резервное копирование** | ⭐⭐⭐ Через HA backup | ⭐⭐⭐⭐⭐ Простые текстовые файлы |
| **Шаблоны и переменные** | ❌ Нет | ✅ Можно использовать |
| **Автоматизация** | ❌ Нельзя генерировать | ✅ Можно генерировать скриптами |
| **Обновления** | ✅ Новые возможности сразу | ⭐⭐⭐ Нужно обновлять вручную |

---

### ✅ Преимущества UI (рекомендуется для большинства)

**1. Визуальное редактирование**
- Видишь результат сразу, без перезагрузки
- Drag & drop для перемещения карточек
- Изменение размера карточек мышкой (в Sections view)

**2. Защита от ошибок**
- UI не даст создать неправильную конфигурацию
- Автоматическая валидация полей
- Подсказки и автодополнение

**3. Простота для новичков**
- Не нужно знать YAML синтаксис
- Интуитивный интерфейс
- Встроенная документация

**4. Быстрота**
- Создать дашборд за 5 минут
- Добавить карточку в 3 клика
- Изменить настройки без редактирования файлов

**5. Современные возможности**
- Новые типы карточек появляются автоматически
- Обновления интерфейса без действий с твоей стороны

---

### ⚙️ Когда YAML всё же нужен

**1. Массовое копирование**

Нужно создать 20 одинаковых карточек для разных комнат:

```yaml
# Скопировал один раз, заменил entity — готово
- type: tile
  entity: light.bedroom_1
- type: tile
  entity: light.bedroom_2
# ... ещё 18 раз
```

В UI пришлось бы создавать каждую карточку вручную.

**2. Версионный контроль (Git)**

Отслеживай изменения дашборда:

```bash
git diff dashboards/main.yaml
```

Видишь что изменилось, когда и кем. Можно откатить изменения.

**3. Шаблоны и переменные**

Используй YAML-якоря для повторяющихся блоков:

```yaml
# Определяем шаблон
.light_card: &light_card
  type: tile
  show_name: true
  show_icon: true

# Используем шаблон
cards:
  - <<: *light_card
    entity: light.bedroom
  - <<: *light_card
    entity: light.kitchen
```

**4. Резервное копирование**

Простые текстовые файлы легко:
- Копировать
- Хранить в облаке
- Восстанавливать
- Переносить между установками HA

**5. Автоматизация**

Генерируй дашборды скриптами:

```python
# Скрипт создаёт карточки для всех ламп автоматически
lights = ["bedroom", "kitchen", "living_room"]
for light in lights:
    print(f"- type: tile\n  entity: light.{light}")
```

**6. Обмен конфигурациями**

Легко поделиться дашбордом с сообществом:
- Скопировал YAML
- Вставил в форум/GitHub
- Другие могут использовать

---

### 🔄 Гибридный подход (лучшее из обоих миров)

Home Assistant позволяет **редактировать UI-дашборды через YAML** прямо в браузере!

**Как это работает:**

1. Создай дашборд через UI (быстро и просто)
2. Нажми **⋮** (три точки) → **Edit Dashboard**
3. Нажми **⋮** → **Raw configuration editor**
4. Редактируй YAML прямо в браузере
5. Нажми **Save**

**Что это даёт:**

✅ Удобство UI для повседневной работы  
✅ Мощь YAML для сложных задач  
✅ Можно копировать/вставлять блоки кода  
✅ Быстрое дублирование карточек  
✅ Массовые изменения (найти/заменить)  

**Пример использования:**

1. Создал 5 карточек через UI
2. Открыл Raw configuration editor
3. Скопировал блок карточки
4. Вставил 10 раз, изменил entity
5. Сохранил — готово 50 карточек за минуту!

---

### 💡 Моя рекомендация

**Для новичков и большинства пользователей:**

👉 **Используй UI** — это проще, быстрее и безопаснее.

**Когда переходить на YAML:**

- Нужно создать много похожих карточек
- Хочешь хранить конфигурацию в Git
- Нужно поделиться дашбордом с кем-то
- Делаешь массовые изменения (например, переименовал все устройства)

**Идеальный подход:**

1. **Начни с UI** — создай основу дашборда
2. **Используй Raw configuration editor** для копирования блоков
3. **Переходи на YAML-файлы** только если действительно нужно

---

### 📝 Как начать с UI

**Шаг 1:** Settings → Dashboards → + Add Dashboard  
**Шаг 2:** Открой дашборд → ⋮ → Edit Dashboard  
**Шаг 3:** + Add Card → выбери устройство  
**Шаг 4:** Настрой карточку → Add to dashboard  
**Шаг 5:** Done

Готово! Дашборд создан за 2 минуты. 🎉

---

## 3. 🛠 Создание и управление дашбордами

### Создание нового дашборда

**Способ 1: Через UI (рекомендуется)**

1. Открой боковое меню (сайдбар)
2. Внизу нажми **Settings** (⚙️)
3. Выбери **Dashboards**
4. Нажми **+ Add Dashboard** (внизу справа)
5. Заполни данные:
   - **Title:** Название дашборда (например, "Спальня")
   - **Icon:** Иконка из [Material Design Icons](https://pictogrammers.com/library/mdi/) (например, `mdi:bed`)
   - **URL:** Путь в адресной строке (например, `bedroom`)
   - **Show in sidebar:** Показывать в боковом меню (включено по умолчанию)
   - **Admin only:** Доступ только для администраторов
6. Нажми **Create**

**Способ 2: Через YAML**

> **⚠️ Важно (2026):** Старый формат `lovelace: mode: yaml` устарел и будет удалён в HA 2026.8. Используй новый формат ниже.

**Новый формат (2026+):**

Отредактируй `configuration.yaml`:

```yaml
lovelace:
  resource_mode: yaml  # Если используешь YAML для ресурсов (HACS карточки)
  dashboards:
    bedroom:
      mode: yaml
      title: Спальня
      icon: mdi:bed
      show_in_sidebar: true
      filename: dashboards/bedroom.yaml
```

Создай файл `dashboards/bedroom.yaml`:

```yaml
views:
  - title: Устройства
    cards:
      - type: entities
        entities:
          - light.bedroom_light
          - switch.bedroom_fan
```

Перезапусти Home Assistant.

**Старый формат (устарел, не используй):**

```yaml
# ❌ НЕ ИСПОЛЬЗУЙ - будет удалено в 2026.8
lovelace:
  mode: yaml
  resources:
    - url: /local/card.js
      type: module
```

---

### Редактирование дашборда

1. Открой нужный дашборд
2. Нажми **⋮** (три точки) в правом верхнем углу
3. Выбери **Edit Dashboard**
4. Внеси изменения
5. Нажми **Done**

---

### Удаление дашборда

1. **Settings → Dashboards**
2. Найди нужный дашборд
3. Нажми **⋮** → **Delete**
4. Подтверди удаление

> ⚠️ **Внимание:** Дашборд "Overview" нельзя удалить, но можно скрыть из сайдбара.

---

### Настройки видимости

Можно ограничить доступ к дашборду для определенных пользователей:

**Через UI:**
1. **Settings → Dashboards**
2. Выбери дашборд → **⋮** → **Edit**
3. Включи **Admin only** или настрой **Visible to** для конкретных пользователей

**Через YAML:**

```yaml
lovelace:
  dashboards:
    admin_panel:
      mode: yaml
      title: Админ-панель
      icon: mdi:shield-account
      show_in_sidebar: true
      require_admin: true
      filename: dashboards/admin.yaml
```

---

## 4. 📑 Представления (Views) — вкладки внутри дашборда

**View (Представление)** — это вкладка внутри дашборда. Один дашборд может содержать множество представлений.

### Создание представления

1. Открой дашборд
2. Нажми **⋮** → **Edit Dashboard**
3. Нажми **+** (плюс) в верхней панели вкладок
4. Заполни данные:
   - **Title:** Название вкладки (например, "Освещение")
   - **Icon:** Иконка (например, `mdi:lightbulb`)
   - **Show icon and title:** Показывать и иконку, и текст
   - **URL:** Путь (например, `lights`)
   - **View type:** Тип представления (см. следующий раздел)
5. Нажми **Save**

### Редактирование представления

1. Открой представление
2. Нажми **⋮** → **Edit view**
3. Внеси изменения
4. Нажми **Save**

### Удаление представления

1. Открой представление
2. Нажми **⋮** → **Edit view**
3. Нажми **Delete view**
4. Подтверди удаление

---

## 5. 🎨 Типы представлений (View Types)

Home Assistant предлагает 4 типа представлений с разными способами размещения карточек:

### 1. Sections (по умолчанию) — Рекомендуется

**Описание:** Карточки размещаются в секциях (группах) с гибкой сеткой.

**Когда использовать:**
- Для современного, структурированного интерфейса
- Когда нужно группировать карточки по смыслу
- Для адаптивного дизайна (хорошо выглядит на всех устройствах)

**Настройки:**
- **Max number of sections wide:** Максимальное количество колонок (1-4)
- **Dense section placement:** Автоматическое заполнение пробелов между карточками

**Пример:**

```yaml
views:
  - title: Гостиная
    type: sections
    max_columns: 3
    sections:
      - type: grid
        title: Освещение
        cards:
          - type: tile
            entity: light.living_room_main
          - type: tile
            entity: light.living_room_accent
      - type: grid
        title: Климат
        cards:
          - type: thermostat
            entity: climate.living_room
```

---

### 2. Masonry (классический)

**Описание:** Карточки размещаются в колонках, как кирпичная кладка. Высота карточек определяется их содержимым.

**Когда использовать:**
- Для простого, быстрого создания дашборда
- Когда не нужна строгая структура
- Для дашбордов с карточками разной высоты

**Пример:**

```yaml
views:
  - title: Обзор
    type: masonry
    cards:
      - type: weather-forecast
        entity: weather.home
      - type: entities
        entities:
          - light.kitchen
          - light.living_room
      - type: history-graph
        entities:
          - sensor.temperature
```

---

### 3. Panel (полноэкранный)

**Описание:** Одна карточка на весь экран. Идеально для карт, изображений, видео.

**Когда использовать:**
- Для отображения карты
- Для полноэкранного просмотра камеры
- Для больших графиков или изображений

**Пример:**

```yaml
views:
  - title: Карта
    type: panel
    cards:
      - type: map
        entities:
          - person.john
          - person.jane
          - zone.home
```

---

### 4. Sidebar (с боковой панелью)

**Описание:** Две колонки — широкая слева и узкая справа.

**Когда использовать:**
- Для десктопных дашбордов
- Когда нужно выделить важную информацию в боковой панели

**Пример:**

```yaml
views:
  - title: Главная
    type: sidebar
    cards:
      # Основная колонка (слева)
      - type: weather-forecast
        entity: weather.home
      - type: entities
        entities:
          - light.kitchen
    sidebar:
      # Боковая панель (справа)
      - type: picture
        image: /local/family.jpg
      - type: button
        entity: script.good_night
```

---

## 6. 🃏 Карточки (Cards)

Карточки — это основные элементы дашборда, которые отображают информацию и позволяют управлять устройствами.

### Популярные типы карточек

| Карточка | Описание | Когда использовать |
|----------|----------|-------------------|
| **Tile** | Современная карточка с большой иконкой | Для управления устройствами (свет, выключатели) |
| **Entities** | Список сущностей | Для группировки нескольких устройств |
| **Picture Entity** | Изображение с управлением | Для камер, изображений комнат |
| **Button** | Кнопка | Для быстрых действий (сцены, скрипты) |
| **Thermostat** | Термостат | Для управления климатом |
| **Weather Forecast** | Прогноз погоды | Для отображения погоды |
| **History Graph** | График истории | Для отображения данных датчиков |
| **Map** | Карта | Для отслеживания местоположения |
| **Camera** | Видеопоток | Для просмотра камер |
| **Markdown** | Текст с форматированием | Для заметок, инструкций |

### Добавление карточки

**Способ 1: Через UI**

1. Открой представление
2. Нажми **⋮** → **Edit Dashboard**
3. Нажми **+ Add Card**
4. Выбери тип карточки:
   - **By card type:** Выбрать тип карточки вручную
   - **By entity:** Выбрать устройство, HA предложит подходящую карточку
5. Настрой карточку
6. Нажми **Add to dashboard**

**Способ 2: Через YAML**

```yaml
views:
  - title: Гостиная
    cards:
      - type: tile
        entity: light.living_room
        name: Основной свет
        icon: mdi:ceiling-light
```

---

### Изменение размера карточки (Sections view)

В Sections view можно изменять размер карточек:

1. Открой карточку для редактирования
2. Перейди на вкладку **Layout**
3. Используй сетку для изменения размера:
   - **Columns:** Ширина (количество колонок)
   - **Rows:** Высота (количество строк)
4. **Full width card:** Карточка на всю ширину секции
5. **Precise mode:** Более точная сетка для размещения

---

### Условная видимость карточки

Можно показывать/скрывать карточки в зависимости от условий:

1. Открой карточку для редактирования
2. Перейди на вкладку **Visibility**
3. Нажми **Add condition**
4. Выбери тип условия:
   - **User:** Показывать только определённым пользователям
   - **State:** Показывать при определённом состоянии устройства
   - **Screen:** Показывать только на мобильных/десктопных устройствах
   - **Numeric state:** Показывать при значении датчика в диапазоне

**Пример YAML:**

```yaml
- type: tile
  entity: light.bedroom
  visibility:
    - condition: state
      entity: sun.sun
      state: below_horizon
```

Эта карточка будет видна только ночью.

---

## 7. 🏠 Организация по комнатам

Один из самых популярных способов организации — создать отдельный дашборд для каждой комнаты.

### Пример: Дашборд "Спальня"

**Структура:**
```
Дашборд "Спальня"
├── Представление "Устройства" (основное)
├── Представление "Климат"
└── Представление "Безопасность"
```

**Создание:**

1. **Settings → Dashboards → + Add Dashboard**
2. Заполни:
   - **Title:** Спальня
   - **Icon:** `mdi:bed`
   - **URL:** `bedroom`
3. Нажми **Create**

**Добавление представлений:**

**Представление 1: Устройства**

```yaml
- title: Устройства
  type: sections
  sections:
    - type: grid
      title: Освещение
      cards:
        - type: tile
          entity: light.bedroom_main
          name: Основной свет
        - type: tile
          entity: light.bedroom_bedside_left
          name: Прикроватная лампа (левая)
        - type: tile
          entity: light.bedroom_bedside_right
          name: Прикроватная лампа (правая)
    
    - type: grid
      title: Устройства
      cards:
        - type: tile
          entity: switch.bedroom_fan
          name: Вентилятор
        - type: tile
          entity: media_player.bedroom_tv
          name: Телевизор
```

**Представление 2: Климат**

```yaml
- title: Климат
  type: sections
  sections:
    - type: grid
      title: Температура и влажность
      cards:
        - type: thermostat
          entity: climate.bedroom
        - type: sensor
          entity: sensor.bedroom_temperature
          graph: line
        - type: sensor
          entity: sensor.bedroom_humidity
          graph: line
```

**Представление 3: Безопасность**

```yaml
- title: Безопасность
  type: sections
  sections:
    - type: grid
      title: Камера
      cards:
        - type: picture-entity
          entity: camera.bedroom
          camera_view: live
    
    - type: grid
      title: Датчики
      cards:
        - type: entities
          entities:
            - binary_sensor.bedroom_motion
            - binary_sensor.bedroom_door
            - binary_sensor.bedroom_window
```

---

### Пример: Дашборд "Гостиная"

```yaml
views:
  - title: Обзор
    type: sections
    sections:
      - type: grid
        title: Освещение
        cards:
          - type: tile
            entity: light.living_room_main
          - type: tile
            entity: light.living_room_accent
          - type: tile
            entity: light.living_room_floor_lamp
      
      - type: grid
        title: Развлечения
        cards:
          - type: media-control
            entity: media_player.living_room_tv
          - type: media-control
            entity: media_player.living_room_speaker
      
      - type: grid
        title: Климат
        cards:
          - type: thermostat
            entity: climate.living_room
          - type: sensor
            entity: sensor.living_room_temperature
```

---

## 8. 🔌 Организация по типам устройств

Альтернативный подход — группировать устройства по типам, а не по комнатам.

### Пример: Дашборд "Освещение"

Все лампы и выключатели в одном месте.

```yaml
title: Освещение
icon: mdi:lightbulb-group
views:
  - title: Все лампы
    type: sections
    sections:
      - type: grid
        title: Спальня
        cards:
          - type: tile
            entity: light.bedroom_main
          - type: tile
            entity: light.bedroom_bedside_left
          - type: tile
            entity: light.bedroom_bedside_right
      
      - type: grid
        title: Гостиная
        cards:
          - type: tile
            entity: light.living_room_main
          - type: tile
            entity: light.living_room_accent
          - type: tile
            entity: light.living_room_floor_lamp
      
      - type: grid
        title: Кухня
        cards:
          - type: tile
            entity: light.kitchen_main
          - type: tile
            entity: light.kitchen_under_cabinet
  
  - title: Сцены
    type: sections
    sections:
      - type: grid
        title: Быстрые сцены
        cards:
          - type: button
            entity: scene.all_lights_on
            name: Все включить
            icon: mdi:lightbulb-on
          - type: button
            entity: scene.all_lights_off
            name: Все выключить
            icon: mdi:lightbulb-off
          - type: button
            entity: scene.movie_time
            name: Кино
            icon: mdi:movie
          - type: button
            entity: scene.good_night
            name: Спокойной ночи
            icon: mdi:weather-night
```

---

### Пример: Дашборд "Камеры"

Все камеры в одном месте.

```yaml
title: Камеры
icon: mdi:cctv
views:
  - title: Все камеры
    type: sections
    max_columns: 2
    sections:
      - type: grid
        title: Входная дверь
        cards:
          - type: picture-entity
            entity: camera.front_door
            camera_view: live
            show_state: false
            show_name: false
      
      - type: grid
        title: Задний двор
        cards:
          - type: picture-entity
            entity: camera.backyard
            camera_view: live
            show_state: false
            show_name: false
      
      - type: grid
        title: Гараж
        cards:
          - type: picture-entity
            entity: camera.garage
            camera_view: live
            show_state: false
            show_name: false
      
      - type: grid
        title: Спальня
        cards:
          - type: picture-entity
            entity: camera.bedroom
            camera_view: live
            show_state: false
            show_name: false
  
  - title: События
    type: sections
    sections:
      - type: grid
        title: Последние события
        cards:
          - type: logbook
            entities:
              - camera.front_door
              - camera.backyard
              - camera.garage
              - camera.bedroom
            hours_to_show: 24
```

---

### Пример: Дашборд "Безопасность"

Датчики, замки, камеры, сигнализация.

```yaml
title: Безопасность
icon: mdi:shield-home
views:
  - title: Обзор
    type: sections
    sections:
      - type: grid
        title: Статус
        cards:
          - type: alarm-panel
            entity: alarm_control_panel.home
          - type: entities
            title: Двери и окна
            entities:
              - binary_sensor.front_door
              - binary_sensor.back_door
              - binary_sensor.garage_door
              - binary_sensor.bedroom_window
              - binary_sensor.living_room_window
      
      - type: grid
        title: Камеры
        cards:
          - type: picture-entity
            entity: camera.front_door
            camera_view: live
          - type: picture-entity
            entity: camera.backyard
            camera_view: live
      
      - type: grid
        title: Датчики движения
        cards:
          - type: entities
            entities:
              - binary_sensor.front_door_motion
              - binary_sensor.living_room_motion
              - binary_sensor.bedroom_motion
              - binary_sensor.garage_motion
```

---

## 9. 💡 Практические примеры

### Пример 1: Полный дашборд "Дом" с вкладками по этажам

```yaml
title: Дом
icon: mdi:home
views:
  # Первый этаж
  - title: Первый этаж
    icon: mdi:home-floor-1
    type: sections
    sections:
      - type: grid
        title: Гостиная
        cards:
          - type: tile
            entity: light.living_room_main
          - type: tile
            entity: media_player.living_room_tv
          - type: thermostat
            entity: climate.living_room
      
      - type: grid
        title: Кухня
        cards:
          - type: tile
            entity: light.kitchen_main
          - type: tile
            entity: switch.coffee_maker
          - type: sensor
            entity: sensor.kitchen_temperature
  
  # Второй этаж
  - title: Второй этаж
    icon: mdi:home-floor-2
    type: sections
    sections:
      - type: grid
        title: Спальня
        cards:
          - type: tile
            entity: light.bedroom_main
          - type: thermostat
            entity: climate.bedroom
      
      - type: grid
        title: Детская
        cards:
          - type: tile
            entity: light.kids_room
          - type: sensor
            entity: sensor.kids_room_temperature
  
  # Улица
  - title: Улица
    icon: mdi:home-outline
    type: sections
    sections:
      - type: grid
        title: Камеры
        cards:
          - type: picture-entity
            entity: camera.front_door
          - type: picture-entity
            entity: camera.backyard
      
      - type: grid
        title: Освещение
        cards:
          - type: tile
            entity: light.front_porch
          - type: tile
            entity: light.backyard
```

---

### Пример 2: Дашборд с условной видимостью

Показывать разные карточки в зависимости от времени суток:

```yaml
views:
  - title: Главная
    type: sections
    sections:
      - type: grid
        title: Утро
        cards:
          # Показывать только утром (6:00 - 12:00)
          - type: button
            entity: script.morning_routine
            name: Утренняя рутина
            icon: mdi:coffee
            visibility:
              - condition: numeric_state
                entity: sensor.time
                above: 6
                below: 12
      
      - type: grid
        title: Вечер
        cards:
          # Показывать только вечером (18:00 - 23:00)
          - type: button
            entity: script.evening_routine
            name: Вечерняя рутина
            icon: mdi:weather-sunset
            visibility:
              - condition: numeric_state
                entity: sensor.time
                above: 18
                below: 23
      
      - type: grid
        title: Ночь
        cards:
          # Показывать только ночью (после заката)
          - type: button
            entity: script.good_night
            name: Спокойной ночи
            icon: mdi:weather-night
            visibility:
              - condition: state
                entity: sun.sun
                state: below_horizon
```

---

### Пример 3: Мобильный дашборд

Оптимизированный для телефона (одна колонка):

```yaml
title: Мобильный
icon: mdi:cellphone
views:
  - title: Быстрый доступ
    type: sections
    max_columns: 1
    sections:
      - type: grid
        title: Сцены
        cards:
          - type: button
            entity: scene.good_morning
            name: Доброе утро
            icon: mdi:weather-sunny
          - type: button
            entity: scene.leaving_home
            name: Ухожу
            icon: mdi:exit-run
          - type: button
            entity: scene.arriving_home
            name: Пришёл
            icon: mdi:home-import-outline
          - type: button
            entity: scene.good_night
            name: Спокойной ночи
            icon: mdi:weather-night
      
      - type: grid
        title: Статус
        cards:
          - type: entities
            entities:
              - person.john
              - person.jane
              - alarm_control_panel.home
              - sensor.home_temperature
      
      - type: grid
        title: Камеры
        cards:
          - type: picture-entity
            entity: camera.front_door
            camera_view: live
```

---

## 10. 📏 Лимиты и ограничения

### Количество дашбордов

- **Технический лимит:** Нет жёсткого ограничения
- **Практический лимит:** ~20-30 дашбордов (после этого сайдбар становится перегруженным)
- **Рекомендация:** 5-10 дашбордов для удобной навигации

### Количество представлений (views) в дашборде

- **Технический лимит:** Нет жёсткого ограничения
- **Практический лимит:** ~10-15 вкладок (после этого навигация становится неудобной)
- **Рекомендация:** 3-7 представлений на дашборд

### Количество карточек в представлении

- **Технический лимит:** Нет жёсткого ограничения
- **Практический лимит:** ~50-100 карточек (зависит от типа карточек и устройства)
- **Рекомендация:** 10-30 карточек для быстрой загрузки

### Производительность

**Факторы, влияющие на производительность:**

1. **Тип карточек:**
   - Лёгкие: Button, Entities, Tile
   - Средние: History Graph, Sensor
   - Тяжёлые: Camera (live stream), Map, Picture Elements

2. **Количество камер:**
   - 1-2 камеры: Нормально
   - 3-5 камер: Может тормозить на слабых устройствах
   - 6+ камер: Рекомендуется разделить по разным представлениям

3. **Устройство:**
   - Десктоп: Может обрабатывать больше карточек
   - Планшет: Средняя производительность
   - Телефон: Ограниченная производительность

**Советы по оптимизации:**

- Используйте `camera_view: auto` вместо `live` для камер (загружается по требованию)
- Разделяйте тяжёлые карточки по разным представлениям
- Используйте условную видимость для скрытия неиспользуемых карточек
- Избегайте большого количества графиков на одном экране

---

## 11. 💡 Советы и best practices

### Организация структуры

**Вариант 1: По комнатам (рекомендуется для домов)**
```
├── Дашборд "Обзор" (главный)
├── Дашборд "Спальня"
├── Дашборд "Гостиная"
├── Дашборд "Кухня"
└── Дашборд "Улица"
```

**Вариант 2: По функциям (рекомендуется для квартир)**
```
├── Дашборд "Обзор" (главный)
├── Дашборд "Освещение"
├── Дашборд "Климат"
├── Дашборд "Безопасность"
└── Дашборд "Развлечения"
```

**Вариант 3: Гибридный (лучшее из обоих)**
```
├── Дашборд "Обзор" (главный)
├── Дашборд "Спальня" (по комнате)
├── Дашборд "Гостиная" (по комнате)
├── Дашборд "Камеры" (по функции)
└── Дашборд "Энергия" (по функции)
```

---

### Именование

**Дашборды:**
- ✅ Хорошо: "Спальня", "Гостиная", "Камеры"
- ❌ Плохо: "Dashboard 1", "Мой дашборд", "Тест"

**Представления:**
- ✅ Хорошо: "Устройства", "Климат", "Безопасность"
- ❌ Плохо: "View 1", "Всё", "Разное"

**URL:**
- ✅ Хорошо: `bedroom`, `living-room`, `cameras`
- ❌ Плохо: `view1`, `dashboard`, `test123`

---

### Иконки

Используй иконки из [Material Design Icons](https://pictogrammers.com/library/mdi/):

**Комнаты:**
- Спальня: `mdi:bed`
- Гостиная: `mdi:sofa`
- Кухня: `mdi:silverware-fork-knife`
- Ванная: `mdi:shower`
- Гараж: `mdi:garage`

**Функции:**
- Освещение: `mdi:lightbulb-group`
- Климат: `mdi:thermostat`
- Безопасность: `mdi:shield-home`
- Камеры: `mdi:cctv`
- Энергия: `mdi:lightning-bolt`
- Развлечения: `mdi:television`

---

### Мобильная vs десктопная версия

**Для мобильных:**
- Используй `max_columns: 1` или `max_columns: 2` в Sections view
- Крупные кнопки (Tile card)
- Минимум текста
- Быстрый доступ к часто используемым функциям

**Для десктопа:**
- Используй `max_columns: 3` или `max_columns: 4`
- Больше информации на экране
- Графики и детальная статистика
- Sidebar view для двухколоночного layout

**Универсальный подход:**
- Sections view автоматически адаптируется под размер экрана
- Используй условную видимость по типу устройства:

```yaml
visibility:
  - condition: screen
    media_query: "(max-width: 768px)"  # Только мобильные
```

---

### Резервное копирование

**Способ 1: Через UI (Storage mode)**

Дашборды хранятся в `.storage/lovelace.*` — включи эти файлы в резервное копирование Home Assistant.

**Способ 2: Через YAML (YAML mode)**

Дашборды хранятся в `dashboards/*.yaml` — добавь эти файлы в систему контроля версий (Git).

**Конфигурация (2026+):**

```yaml
lovelace:
  resource_mode: yaml  # Опционально, если используешь YAML для ресурсов
  dashboards:
    my_dashboard:
      mode: yaml
      title: Мой дашборд
      icon: mdi:view-dashboard
      show_in_sidebar: true
      filename: dashboards/my_dashboard.yaml
```

> **Примечание:** Старый формат `lovelace: mode: yaml` устарел и будет удалён в Home Assistant 2026.8. Используй новый формат выше.

**Рекомендация:**
- Используй YAML mode для важных дашбордов
- Храни конфигурацию в Git
- Делай резервные копии перед большими изменениями

---

### Тестирование изменений

1. Создай тестовый дашборд для экспериментов
2. Не редактируй главный дашборд "Overview" напрямую
3. Используй функцию "Duplicate dashboard" для создания копии
4. Тестируй на разных устройствах (телефон, планшет, десктоп)

---

## 📚 Дополнительные ресурсы

- [Официальная документация по дашбордам](https://www.home-assistant.io/dashboards/)
- [Список всех карточек](https://www.home-assistant.io/dashboards/cards/)
- [Material Design Icons](https://pictogrammers.com/library/mdi/)
- [Форум Home Assistant](https://community.home-assistant.io/c/projects/dashboards/)
- [Reddit r/homeassistant](https://www.reddit.com/r/homeassistant/)
- [Примеры дашбордов от сообщества](https://community.home-assistant.io/tag/dashboard-showcase)

---

## 🎯 Быстрый старт: Создание первого дашборда

### Шаг 1: Создай дашборд

1. **Settings → Dashboards → + Add Dashboard**
2. Название: "Моя комната"
3. Иконка: `mdi:home-heart`
4. URL: `my-room`
5. **Create**

### Шаг 2: Добавь представление

1. Открой дашборд "Моя комната"
2. **⋮ → Edit Dashboard**
3. Нажми **+** (добавить представление)
4. Название: "Устройства"
5. Тип: **Sections**
6. **Save**

### Шаг 3: Добавь карточки

1. **+ Add Card**
2. Выбери **By entity**
3. Выбери свои устройства (лампы, выключатели, датчики)
4. **Continue**
5. **Add to dashboard**

### Шаг 4: Организуй в секции

1. Перетащи карточки в нужные секции
2. Переименуй секции (например, "Освещение", "Климат")
3. Измени размер карточек на вкладке **Layout**
4. **Done**

Готово! Твой первый дашборд создан. 🎉

---

*Документ обновлён: май 2026*
