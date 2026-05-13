# 🤖 Frigate + Home Assistant: Автоматизации

Практическое руководство по созданию автоматизаций на основе детекции объектов Frigate.
Информация проверена по официальной документации Frigate (docs.frigate.video, май 2026).

---

## 📋 Содержание

1. [Важно: MQTT обязателен для интеграции](#1-важно-mqtt-обязателен-для-интеграции)
2. [Настройка MQTT для Frigate](#2-настройка-mqtt-для-frigate)
3. [Доступные сенсоры Frigate в HA](#3-доступные-сенсоры-frigate-в-ha)
4. [MQTT топики Frigate](#4-mqtt-топики-frigate)
5. [Примеры автоматизаций](#5-примеры-автоматизаций)
   - [5.1. Уведомление когда нет дома](#51-уведомление-когда-нет-дома)
   - [5.2. Уведомление с фото через HA API](#52-уведомление-с-фото-через-ha-api)
   - [5.3. Уведомление ночью](#53-уведомление-ночью)
   - [5.4. Включить свет при появлении человека](#54-включить-свет-при-появлении-человека)
   - [5.5. Включить монитор при движении](#55-включить-монитор-при-движении)
   - [5.6. Запись только когда нет дома](#56-запись-только-когда-нет-дома)
   - [5.7. Уведомление при обнаружении машины](#57-уведомление-при-обнаружении-машины)
   - [5.8. Уведомление через frigate/reviews (рекомендуется)](#58-уведомление-через-frigatereviews-рекомендуется)
6. [Notification API — ссылки на фото и видео](#6-notification-api--ссылки-на-фото-и-видео)
7. [Управление камерами через MQTT](#7-управление-камерами-через-mqtt)
8. [Полезные советы](#8-полезные-советы)

---

## 1. Важно: MQTT обязателен для интеграции

Согласно официальной документации Frigate, **MQTT должен быть включён** для корректной работы большинства сенсоров интеграции:

> "The Frigate integration requires the mqtt integration to be installed and manually configured first. MQTT must be enabled in your Frigate configuration file and Frigate must be connected to the same MQTT server as Home Assistant for many of the entities created by the integration to function."

Без MQTT:
- ❌ Сенсоры `binary_sensor` не будут обновляться
- ❌ Счётчики объектов не будут работать
- ✅ Только базовый просмотр камер и медиабраузер

---

## 2. Настройка MQTT для Frigate

У тебя уже запущен Mosquitto (MQTT брокер). Нужно подключить Frigate к нему.

### Шаг 1: Включи MQTT в Frigate

```bash
nano ~/frigate/config/config.yml
```

Измени секцию `mqtt`:

```yaml
mqtt:
  enabled: true
  host: 127.0.0.1   # Mosquitto на том же сервере
  port: 1883        # Стандартный порт Mosquitto
  # user: mqtt_user       # Если настроена аутентификация в Mosquitto
  # password: mqtt_pass   # Если настроена аутентификация в Mosquitto
```

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`

Перезапусти Frigate:

```bash
cd ~/frigate
docker compose restart
```

### Шаг 2: Настрой MQTT интеграцию в HA

1. Settings → Devices & Services → Add Integration → **MQTT**
2. Broker: `127.0.0.1` (или `10.0.0.3`)
3. Port: `1883`
4. Submit

### Шаг 3: Проверь что сенсоры появились

После настройки MQTT сенсоры Frigate в HA должны показывать реальные значения (не "Недоступно").

---

## 3. Доступные сенсоры Frigate в HA

После настройки MQTT интеграции появляются следующие сенсоры:

### Binary sensors (Да/Нет):

| Сенсор | Описание |
|--------|----------|
| `binary_sensor.camera_outside_1_person_occupancy` | Есть ли человек в кадре |
| `binary_sensor.camera_outside_1_car_occupancy` | Есть ли машина в кадре |
| `binary_sensor.camera_outside_1_dog_occupancy` | Есть ли собака в кадре |
| `binary_sensor.camera_outside_1_motion` | Есть ли движение в кадре |

> **Примечание:** Occupancy сенсоры имеют меньше фильтрации ложных срабатываний — они быстрые, но могут иногда давать ложные триггеры. Для важных автоматизаций используй `frigate/reviews` MQTT топик.

### Sensors (числовые):

| Сенсор | Описание |
|--------|----------|
| `sensor.camera_outside_1_person_count` | Сколько людей в кадре |
| `sensor.camera_outside_1_car_count` | Сколько машин в кадре |
| `sensor.camera_outside_1_all_count` | Всего объектов в кадре |

### Switches (переключатели):

| Switch | Описание |
|--------|----------|
| `switch.camera_outside_1_detect` | Включить/выключить детекцию |
| `switch.camera_outside_1_recordings` | Включить/выключить запись |
| `switch.camera_outside_1_snapshots` | Включить/выключить снимки |

### Cameras:

| Сенсор | Описание |
|--------|----------|
| `camera.camera_outside_1` | Живое видео (RTSP) |
| `camera.camera_outside_1_person` | Снимок последнего обнаруженного человека |

> Замени `camera_outside_1` на название своей камеры из конфига Frigate.

---

## 4. MQTT топики Frigate

### Основные топики:

| Топик | Описание |
|-------|----------|
| `frigate/reviews` | ⭐ Рекомендуемый топик для уведомлений (alerts и detections) |
| `frigate/events` | Все события детекции объектов (подробные данные) |
| `frigate/available` | Статус Frigate: `online` / `offline` |

### Топики камер:

| Топик | Описание |
|-------|----------|
| `frigate/<camera>/person` | Количество людей на камере |
| `frigate/<camera>/motion` | Движение: `ON` / `OFF` |
| `frigate/<camera>/detect/state` | Статус детекции: `ON` / `OFF` |
| `frigate/<camera>/recordings/state` | Статус записи: `ON` / `OFF` |
| `frigate/<camera>/review_status` | Статус: `NONE`, `DETECTION`, `ALERT` |

### Управляющие топики (для отправки команд):

| Топик | Команда |
|-------|---------|
| `frigate/<camera>/detect/set` | `ON` или `OFF` — включить/выключить детекцию |
| `frigate/<camera>/recordings/set` | `ON` или `OFF` — включить/выключить запись |
| `frigate/<camera>/snapshots/set` | `ON` или `OFF` — включить/выключить снимки |

---

## 5. Примеры автоматизаций

### 5.1. Уведомление когда нет дома

**Сценарий:** Ты уехал — пришёл человек — получил уведомление в Telegram.

```yaml
alias: "Человек у двери когда нет дома"
trigger:
  - platform: state
    entity_id: binary_sensor.camera_outside_1_person_occupancy
    to: "on"
condition:
  - condition: state
    entity_id: person.твоё_имя
    state: "not_home"
action:
  - service: notify.telegram
    data:
      message: "🚨 Обнаружен человек у входной двери!"
mode: single
```

---

### 5.2. Уведомление с фото через HA API

Frigate integration создаёт специальные API эндпоинты для уведомлений. Используй их вместо прямых ссылок на Frigate — они работают и удалённо через HA.

```yaml
alias: "Уведомление с фото (через HA API)"
trigger:
  - platform: mqtt
    topic: frigate/reviews
    payload: alert
    value_template: "{{ value_json['after']['severity'] }}"
condition:
  - condition: state
    entity_id: person.твоё_имя
    state: "not_home"
action:
  - service: notify.telegram
    data:
      message: >
        🚨 Обнаружен: {{ trigger.payload_json["after"]["data"]["objects"] | join(", ") }}
        Камера: {{ trigger.payload_json["after"]["camera"] }}
      data:
        photo:
          - url: >-
              https://АДРЕС_HA/api/frigate/notifications/{{ trigger.payload_json["after"]["data"]["detections"][0] }}/thumbnail.jpg
            caption: "Frigate Alert"
mode: single
```

> Замени `АДРЕС_HA` на адрес твоего Home Assistant (например `http://10.0.0.3:8123`).

---

### 5.3. Уведомление ночью

**Сценарий:** Ночью любое движение подозрительно.

```yaml
alias: "Движение ночью"
trigger:
  - platform: state
    entity_id: binary_sensor.camera_outside_1_person_occupancy
    to: "on"
condition:
  - condition: time
    after: "23:00:00"
    before: "06:00:00"
action:
  - service: notify.telegram
    data:
      message: "🌙 Ночное движение на камере 1!"
mode: single
```

---

### 5.4. Включить свет при появлении человека

**Сценарий:** Человек подходит к дому — автоматически включается уличный свет на 5 минут.

```yaml
alias: "Свет при появлении человека"
trigger:
  - platform: state
    entity_id: binary_sensor.camera_outside_1_person_occupancy
    to: "on"
action:
  - service: light.turn_on
    target:
      entity_id: light.уличный_свет
    data:
      brightness: 255
  - delay: "00:05:00"
  - service: light.turn_off
    target:
      entity_id: light.уличный_свет
mode: restart  # Перезапускать таймер если снова появился человек
```

---

### 5.5. Включить монитор при движении

**Сценарий:** Кто-то подошёл к двери — монитор автоматически включился.

```yaml
alias: "Монитор при движении у двери"
trigger:
  - platform: state
    entity_id: binary_sensor.camera_outside_1_person_occupancy
    to: "on"
action:
  - service: shell_command.monitor_on   # Включить монитор через HDMI-CEC
mode: single
```

В `configuration.yaml` добавь:

```yaml
shell_command:
  monitor_on: 'echo "on 0" | cec-client -s -d 1'
  monitor_off: 'echo "standby 0" | cec-client -s -d 1'
```

---

### 5.6. Запись только когда нет дома

**Сценарий:** Frigate записывает только когда ты уехал — экономия места на диске.

```yaml
alias: "Включить запись когда уехал"
trigger:
  - platform: state
    entity_id: person.твоё_имя
    to: "not_home"
action:
  - service: switch.turn_on
    target:
      entity_id:
        - switch.camera_outside_1_recordings
        - switch.camera_outside_2_recordings
        - switch.home_usb_recordings
mode: single

---

alias: "Выключить запись когда вернулся"
trigger:
  - platform: state
    entity_id: person.твоё_имя
    to: "home"
action:
  - service: switch.turn_off
    target:
      entity_id:
        - switch.camera_outside_1_recordings
        - switch.camera_outside_2_recordings
        - switch.home_usb_recordings
mode: single
```

---

### 5.7. Уведомление при обнаружении машины

**Сценарий:** Незнакомая машина припарковалась у дома.

```yaml
alias: "Машина у дома"
trigger:
  - platform: state
    entity_id: binary_sensor.camera_outside_1_car_occupancy
    to: "on"
condition:
  - condition: state
    entity_id: person.твоё_имя
    state: "not_home"
action:
  - service: notify.telegram
    data:
      message: "🚗 Машина у дома!"
mode: single
```

---

### 5.8. Уведомление через frigate/reviews (рекомендуется)

Официальная документация рекомендует использовать топик `frigate/reviews` для уведомлений — он даёт больше деталей и меньше ложных срабатываний.

**Разница severity:**
- `detection` — Frigate обнаружил объект
- `alert` — объект попал в зону (более важное событие)

```yaml
alias: "Frigate Alert уведомление"
trigger:
  - platform: mqtt
    topic: frigate/reviews
    payload: alert
    value_template: "{{ value_json['after']['severity'] }}"
action:
  - service: notify.telegram
    data:
      message: >
        🎯 Alert на камере {{ trigger.payload_json["after"]["camera"] }}!
        Объекты: {{ trigger.payload_json["after"]["data"]["objects"] | join(", ") }}
        Зоны: {{ trigger.payload_json["after"]["data"]["zones"] | join(", ") }}
mode: queued
```

---

## 6. Notification API — ссылки на фото и видео

Frigate integration создаёт специальные эндпоинты в HA для получения медиафайлов. Это удобно для уведомлений — ссылки работают через HA даже удалённо.

| Тип | URL |
|-----|-----|
| Миниатюра объекта | `https://HA_URL/api/frigate/notifications/<event-id>/thumbnail.jpg` |
| Снимок объекта | `https://HA_URL/api/frigate/notifications/<event-id>/snapshot.jpg` |
| Видеоклип (Android) | `https://HA_URL/api/frigate/notifications/<event-id>/clip.mp4` |
| Видеоклип (iOS) | `https://HA_URL/api/frigate/notifications/<event-id>/master.m3u8` |
| Превью GIF | `https://HA_URL/api/frigate/notifications/<event-id>/event_preview.gif` |

`event-id` берётся из MQTT события:

```yaml
# Из frigate/reviews:
trigger.payload_json["after"]["data"]["detections"][0]

# Из frigate/events:
trigger.payload_json["after"]["id"]
```

---

## 7. Управление камерами через MQTT

Можно включать/выключать детекцию и запись через MQTT прямо из автоматизаций HA.

### Выключить детекцию на камере:

```yaml
action:
  - service: mqtt.publish
    data:
      topic: "frigate/camera_outside_1/detect/set"
      payload: "OFF"
```

### Включить запись:

```yaml
action:
  - service: mqtt.publish
    data:
      topic: "frigate/camera_outside_1/recordings/set"
      payload: "ON"
```

### Или через switch (проще):

```yaml
action:
  - service: switch.turn_off
    target:
      entity_id: switch.camera_outside_1_detect
```

---

## 8. Полезные советы

### Оптимизация USB камер (снижение CPU)

Если USB камера использует много CPU (15-25%), можно снизить нагрузку:

**Проблема:** USB камеры используют формат MJPEG, который требует больше CPU чем H.264 у IP камер.

**Решение:**

1. **Снизить разрешение:** 640x480 вместо 1280x720
2. **Снизить FPS:** 20 fps вместо 30 fps
3. **Снизить FPS детекции:** `fps: 10` вместо `fps: 15`

**Пример оптимизированной конфигурации:**

```yaml
indoor_usb:
  ffmpeg:
    hwaccel_args: []
    inputs:
      - path: /dev/video0
        input_args: -f v4l2 -input_format mjpeg -video_size 640x480 -framerate 20
        roles:
          - detect
          - record
  detect:
    width: 640
    height: 480
    fps: 10  # Анализировать 10 кадров из 20
```

**Результат:** CPU снизится с 21% до ~8-10%

> **Важно:** Разрешение в `detect` (width/height) **должно совпадать** с разрешением в `input_args` (-video_size).

Подробнее: см. `observer-video-system.md` → раздел "Оптимизация USB камеры"

---

### Избегай спама уведомлений

```yaml
alias: "Уведомление без спама"
trigger:
  - platform: state
    entity_id: binary_sensor.camera_outside_1_person_occupancy
    to: "on"
action:
  - service: notify.telegram
    data:
      message: "Обнаружен человек!"
  - delay: "00:05:00"   # Не отправлять повторно 5 минут
mode: single   # Не запускать если уже выполняется
```

---

### Используй зоны для точности

Вместо всей камеры реагируй только на конкретную зону:

```yaml
trigger:
  - platform: state
    entity_id: binary_sensor.detection_zone_cam_web_person_occupancy
    to: "on"
```

---

### Комбинируй условия

```yaml
condition:
  - condition: and
    conditions:
      - condition: state
        entity_id: person.твоё_имя
        state: "not_home"
      - condition: time
        after: "22:00:00"
        before: "08:00:00"
```

---

### Отладка автоматизаций

Если автоматизация не срабатывает:

1. Settings → Automations → найди автоматизацию
2. Нажми на неё → **Traces** — история запусков
3. Видно почему не сработало (условие не выполнено, триггер не сработал)

Проверь состояние сенсора:
- Developer Tools → States → найди `binary_sensor.camera_outside_1_person_occupancy`

Проверь MQTT сообщения:
- Developer Tools → MQTT → подпишись на `frigate/#` и смотри все события в реальном времени
