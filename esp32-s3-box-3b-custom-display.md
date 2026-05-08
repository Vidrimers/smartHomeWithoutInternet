# 🎨 ESP32-S3-BOX-3B: Кастомизация экрана

Руководство по кастомизации экрана ESP32-S3-BOX-3B — от замены иллюстраций до полного кастомного UI в стиле Pip-Boy.
Информация проверена по официальной документации HA и GitHub репозиториям (май 2026).

---

## 📋 Содержание

1. [Что такое ESPHome и нужен ли он?](#что-такое-esphome-и-нужен-ли-он)
2. [Уровни кастомизации](#1-уровни-кастомизации)
3. [Уровень 1: Замена иллюстраций (просто)](#2-уровень-1-замена-иллюстраций-просто)
4. [Уровень 2: Готовые кастомные прошивки](#3-уровень-2-готовые-кастомные-прошивки)
5. [Уровень 3: Полный кастомный UI через LVGL (сложно)](#4-уровень-3-полный-кастомный-ui-через-lvgl-сложно)
6. [Пример: Pip-Boy стиль](#5-пример-pip-boy-стиль)
7. [Полезные ссылки](#6-полезные-ссылки)

---

## Что такое ESPHome и нужен ли он?

**ESPHome** — это инструмент для создания прошивок для ESP32 устройств через YAML конфиги. Он компилирует прошивку, загружает её на устройство и обеспечивает интеграцию с HA.

Когда ты прошивал ESP32 через сайт HA — ты уже использовал **заранее скомпилированную ESPHome прошивку**. Устройство работает на ESPHome.

**"Установка без ESPHome"** (например, веб-установщик BigBobbas) означает что тебе **не нужно самому запускать ESPHome и компилировать** — просто устанавливаешь готовый `.bin` файл. Устройство всё равно работает на ESPHome прошивке, просто её скомпилировал автор за тебя.

**Влияет ли это на работу с HA?** Нет — ESP32 с любой ESPHome прошивкой подключается к HA одинаково через ESPHome интеграцию.

**Единственный минус веб-установщика:** если захочешь изменить конфиг — нужно будет установить ESPHome Dashboard и перекомпилировать самому.

---

## 1. Уровни кастомизации

| Уровень | Сложность | Что можно сделать |
|---------|-----------|-------------------|
| **1. Замена иллюстраций** | ⭐ Простой | Заменить 6 картинок состояний (idle, listening и т.д.) |
| **2. Готовые прошивки** | ⭐⭐ Средний | Установить готовый кастомный UI с дашбордом HA |
| **3. LVGL кастомный UI** | ⭐⭐⭐ Сложный | Нарисовать свой интерфейс с нуля |

---

## 2. Уровень 1: Замена иллюстраций (просто)

Самый простой способ — заменить 6 стандартных картинок состояний на свои.

**6 состояний ESP32-S3-BOX-3:**
- `loading` — загрузка
- `idle` — ожидание wake word
- `listening` — слушает команду
- `thinking` — обрабатывает
- `replying` — отвечает
- `error` — ошибка

### Шаг 1: Установи ESPHome Dashboard

```bash
docker run -d \
  --name esphome \
  --restart=unless-stopped \
  --network=host \
  -v ~/esphome:/config \
  ghcr.io/esphome/esphome
```

Открой: `http://10.0.0.3:6052`

### Шаг 2: Добавь устройство в ESPHome

В ESPHome Dashboard нажми **Adopt** на карточке ESP32-S3-BOX-3.

### Шаг 3: Используй готовые иллюстрации из сообщества

Открой конфиг устройства в ESPHome → добавь в секцию `substitutions:`:

```yaml
substitutions:
  # Иллюстрации от Frenck (официальный разработчик HA)
  loading_illustration_file: https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations/raw/main/frenck/illustrations/loading.png
  idle_illustration_file: https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations/raw/main/frenck/illustrations/idle.png
  listening_illustration_file: https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations/raw/main/frenck/illustrations/listening.png
  thinking_illustration_file: https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations/raw/main/frenck/illustrations/thinking.png
  replying_illustration_file: https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations/raw/main/frenck/illustrations/replying.png
  error_illustration_file: https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations/raw/main/frenck/illustrations/error.png
```

Нажми **Save** → **Install**.

### Шаг 4: Используй свои изображения

**Требования к изображениям:**
- Размер экрана: **320 x 240 пикселей** (соотношение 4:3)
- Форматы: PNG, JPEG, SVG
- Для `loading` и `idle` — тёмный фон
- Для `listening`, `thinking`, `replying` — светлый фон

**Скопируй изображения на сервер:**

```bash
mkdir -p ~/homeassistant/esphome/my_illustrations
# Скопируй свои PNG файлы в эту папку
```

**Добавь в конфиг:**

```yaml
substitutions:
  loading_illustration_file: my_illustrations/loading.png
  idle_illustration_file: my_illustrations/idle.png
  listening_illustration_file: my_illustrations/listening.png
  thinking_illustration_file: my_illustrations/thinking.png
  replying_illustration_file: my_illustrations/replying.png
  error_illustration_file: my_illustrations/error.png
  
  # Цвет фона (если изображения с прозрачностью)
  loading_illustration_background_color: '000000'    # Чёрный
  idle_illustration_background_color: '000000'
  listening_illustration_background_color: 'FFFFFF'  # Белый
  thinking_illustration_background_color: 'FFFFFF'
  replying_illustration_background_color: 'FFFFFF'
  error_illustration_background_color: '000000'
```

> **Официальная документация:** [home-assistant.io/voice_control/s3-box-customize](https://www.home-assistant.io/voice_control/s3-box-customize/)

> **Репозиторий с готовыми иллюстрациями от сообщества:** [github.com/jlpouffier/home-assistant-s3-box-community-illustrations](https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations)

---

## 3. Уровень 2: Готовые кастомные прошивки

Сообщество создало готовые кастомные прошивки с полноценным дашбордом HA прямо на экране ESP32.

### BigBobbas Custom Firmware (рекомендуется)

**Что умеет:**
- ✅ Сенсорный экран — управление устройствами HA
- ✅ Голосовой ассистент (wake word + Whisper/Piper)
- ✅ Медиаплеер с управлением громкостью
- ✅ Отображение сенсоров (температура, влажность)
- ✅ Интеграция с Alarmo
- ✅ Таймеры
- ✅ Скринсейвер
- ✅ Вывод аудио на внешний медиаплеер HA
- ✅ Поддержка датчика движения (mmWave)

**Ссылки:**
- 🔗 [GitHub репозиторий](https://github.com/BigBobbas/ESP32-S3-Box3-Custom-ESPHome)
- 🔗 [Веб-установщик](https://support.bbdl.co.uk/install.html) — прошивка через браузер без ESPHome
- 🔗 [Конфиг без датчика](https://github.com/BigBobbas/ESP32-S3-Box3-Custom-ESPHome) — для BOX-3B

**Установка через веб-установщик:**

1. Открой [support.bbdl.co.uk/install.html](https://support.bbdl.co.uk/install.html) в Chrome/Edge
2. Подключи ESP32-S3-BOX-3B к компьютеру по USB-C
3. Нажми **Connect** → выбери COM порт
4. Нажми **Install**
5. После прошивки настрой Wi-Fi и добавь в HA

**Установка через ESPHome:**

1. Открой ESPHome Dashboard (`http://10.0.0.3:6052`)
2. Создай новое устройство → выбери ESP32-S3-BOX-3
3. Замени содержимое конфига на содержимое из репозитория
4. Отредактируй `substitutions:` — добавь свои entity_id из HA
5. Нажми **Install**

> ⚠️ **Важно:** Компиляция кастомной прошивки требует много RAM (~2GB). На слабых машинах может упасть. Рекомендуется компилировать на мини-ПК с N100 или использовать веб-установщик.

---

### AlmostInteractive S3-BOX-3 Voice Assistant

Кастомная прошивка с поддержкой датчиков из докстанции (ИК, температура, mmWave).

- 🔗 [GitHub](https://github.com/AlmostInteractive/ESP32-S3-Box-3-Voice-Assistant-Sensor-Dock)

---

### chrisdunnname LVGL UI

Кастомный UI на LVGL для S3-BOX-3.

- 🔗 [GitHub](https://github.com/chrisdunnname/esphome-s3-box-3-lvgl)

---

## 4. Уровень 3: Полный кастомный UI через LVGL (сложно)

**LVGL** (Light and Versatile Graphics Library) — встроенная в ESPHome графическая библиотека. Позволяет рисовать любой интерфейс: кнопки, метки, иконки, анимации.

**Официальная документация LVGL в ESPHome:** [esphome.io/components/lvgl](https://esphome.io/components/lvgl/)

### Базовая структура LVGL конфига:

```yaml
esphome:
  name: s3-box-custom
  friendly_name: S3 Box Custom

esp32:
  board: esp32s3box
  framework:
    type: esp-idf

# Дисплей S3-BOX-3
display:
  - platform: ili9xxx
    model: ILI9342
    auto_clear_enabled: false
    update_interval: never

# Сенсорный экран
touchscreen:
  platform: tt21100
  interrupt_pin: GPIO3

# LVGL
lvgl:
  color_depth: 16
  bg_color: 0x000000
  
  widgets:
    - label:
        text: "Мой ассистент"
        text_color: 0xFFFFFF
        align: CENTER

# Голосовой ассистент (обязательно!)
microphone:
  - platform: i2s_audio
    i2s_din_pin: GPIO16
    adc_type: external
    pdm: false

speaker:
  - platform: i2s_audio
    i2s_dout_pin: GPIO17
    dac_type: external
    model: NS4168

voice_assistant:
  microphone: mic_id
  speaker: speaker_id
  on_listening:
    - lvgl.label.update:
        id: status_label
        text: "Слушаю..."
  on_end:
    - lvgl.label.update:
        id: status_label
        text: "Готов"
```

---

## 5. Пример: Pip-Boy стиль

Для создания интерфейса в стиле Fallout Pip-Boy нужно:

### Цветовая схема:
- Фон: `0x000000` (чёрный)
- Текст: `0x00FF41` (зелёный, как на старых мониторах)
- Акцент: `0x00CC33`

### Шрифт:
Используй моноширинный шрифт. В ESPHome можно подключить кастомный шрифт:

```yaml
font:
  - file: "fonts/PressStart2P-Regular.ttf"  # Пиксельный шрифт
    id: pixel_font
    size: 12
    glyphs: "АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯабвгдеёжзийклмнопрстуфхцчшщъыьэюяABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789:.-_/ "
```

> Скачай пиксельный шрифт: [fonts.google.com/specimen/Press+Start+2P](https://fonts.google.com/specimen/Press+Start+2P)

### Пример LVGL виджетов в стиле Pip-Boy:

```yaml
lvgl:
  color_depth: 16
  bg_color: 0x000000  # Чёрный фон
  
  widgets:
    # Заголовок
    - label:
        id: title_label
        text: "PIP-BOY 3000"
        text_color: 0x00FF41
        text_font: pixel_font
        x: 10
        y: 5
    
    # Разделитель
    - line:
        points:
          - 0, 25
          - 320, 25
        line_color: 0x00FF41
        line_width: 1
    
    # Время
    - label:
        id: time_label
        text: "TIME: 00:00"
        text_color: 0x00FF41
        text_font: pixel_font
        x: 10
        y: 35
    
    # Температура
    - label:
        id: temp_label
        text: "TEMP: --°C"
        text_color: 0x00FF41
        text_font: pixel_font
        x: 10
        y: 55
    
    # Статус ассистента
    - label:
        id: status_label
        text: "STATUS: READY"
        text_color: 0x00FF41
        text_font: pixel_font
        x: 10
        y: 75
    
    # Кнопки управления
    - button:
        x: 10
        y: 120
        width: 140
        height: 40
        bg_color: 0x003300
        border_color: 0x00FF41
        border_width: 1
        widgets:
          - label:
              text: "СВЕТ ВКЛ"
              text_color: 0x00FF41
              text_font: pixel_font
        on_click:
          - homeassistant.service:
              service: light.turn_on
              data:
                entity_id: light.люстра
    
    - button:
        x: 165
        y: 120
        width: 140
        height: 40
        bg_color: 0x003300
        border_color: 0x00FF41
        border_width: 1
        widgets:
          - label:
              text: "СВЕТ ВЫКЛ"
              text_color: 0x00FF41
              text_font: pixel_font
        on_click:
          - homeassistant.service:
              service: light.turn_off
              data:
                entity_id: light.люстра
```

### Обновление данных из HA:

```yaml
time:
  - platform: homeassistant
    id: ha_time
    on_time_sync:
      - lvgl.label.update:
          id: time_label
          text: !lambda |-
            char buf[20];
            sprintf(buf, "TIME: %02d:%02d", id(ha_time).now().hour, id(ha_time).now().minute);
            return buf;

sensor:
  - platform: homeassistant
    id: room_temp
    entity_id: sensor.температура_комната
    on_value:
      - lvgl.label.update:
          id: temp_label
          text: !lambda |-
            char buf[20];
            sprintf(buf, "TEMP: %.1f°C", x);
            return buf;
```

---

## 6. Полезные ссылки

### Официальная документация:
- 📖 [Кастомизация иллюстраций S3-BOX-3](https://www.home-assistant.io/voice_control/s3-box-customize/)
- 📖 [LVGL в ESPHome](https://esphome.io/components/lvgl/)
- 📖 [ESPHome display компонент](https://esphome.io/components/display/)

### Готовые кастомные прошивки:
- 🔗 [BigBobbas Custom Firmware](https://github.com/BigBobbas/ESP32-S3-Box3-Custom-ESPHome) — самая полная кастомная прошивка с дашбордом
- 🔗 [Веб-установщик BigBobbas](https://support.bbdl.co.uk/install.html) — установка без ESPHome
- 🔗 [chrisdunnname LVGL UI](https://github.com/chrisdunnname/esphome-s3-box-3-lvgl) — кастомный LVGL интерфейс
- 🔗 [AlmostInteractive Sensor Dock](https://github.com/AlmostInteractive/ESP32-S3-Box-3-Voice-Assistant-Sensor-Dock) — с поддержкой датчиков

### Иллюстрации от сообщества:
- 🎨 [Community illustrations](https://github.com/jlpouffier/home-assistant-s3-box-community-illustrations) — готовые наборы картинок

### Шрифты для Pip-Boy стиля:
- 🔤 [Press Start 2P](https://fonts.google.com/specimen/Press+Start+2P) — пиксельный шрифт
- 🔤 [Share Tech Mono](https://fonts.google.com/specimen/Share+Tech+Mono) — моноширинный технический шрифт
