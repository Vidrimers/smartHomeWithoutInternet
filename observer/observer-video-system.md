# Observer-video-system.md

# 📹 Система видеонаблюдения с детекцией человека, уведомлениями и удалённым доступом

---

## 📋 Содержание

1. [🛒 Что нужно купить](#1--что-нужно-купить)
   - [1.1. Мини‑ПК](#11-мини‑пк)
   - [1.2. Камеры (RTSP + хорошая детекция)](#12-камеры-rtsp--хорошая-детекция)
   - [1.3. Питание и сеть](#13-питание-и-сеть)
   - [1.4. Хранилище](#14-хранилище)
   - [1.5. Дополнительно](#15-дополнительно)
2. [🧩 Общая архитектура системы](#2--общая-архитектура-системы)
3. [🖥 Установка ОС](#3--установка-ос)
4. [🐳 Установка Docker + Docker Compose](#4--установка-docker--docker-compose)
5. [📦 Установка Frigate](#5--установка-frigate)
6. [🎥 Настройка камер в Frigate](#6--настройка-камер-в-frigate)
   - [6.1. IP камеры (RTSP)](#61-ip-камеры-rtsp)
   - [6.2. USB веб-камеры](#62-usb-веб-камеры)
7. [▶ Запуск Frigate](#7--запуск-frigate)
   - [7.1. Первый запуск](#71-первый-запуск)
   - [7.2. Настройка аутентификации (порт 8971)](#72-настройка-аутентификации-порт-8971)
8. [🏠 Установка Home Assistant](#8--установка-home-assistant)
9. [🔔 Уведомления в Telegram](#9--уведомления-в-telegram)
   - [9.1. Создать бота](#91-создать-бота)
   - [9.2. Добавить в Home Assistant](#92-добавить-в-home-assistant)
10. [👤 Автоматизация уведомлений](#10--автоматизация-уведомлений)
11. [🖥 Автовключение монитора (HDMI‑CEC)](#11--автовключение-монитора-hdmicec)
    - [11.1. Альтернатива: BroadLink RM4C Mini (без HDMI‑CEC)](#111-альтернатива-broadlink-rm4c-mini-без-hdmicec)
12. [🌍 Удалённый доступ через Xray VPN](#12--удалённый-доступ-через-xray-vpn)
    - [12.1. Установка Xray client](#121-установка-xray-client)
    - [12.2. Конфиг клиента](#122-конфиг-клиента)
13. [🔐 Удалённый доступ](#13--удалённый-доступ)
14. [⚙️ Оптимизация производительности](#14--оптимизация-производительности)
15. [🔧 Troubleshooting / Решение проблем](#15--troubleshooting--решение-проблем)
    - [15.1. Ошибки авторизации (401 Unauthorized)](#151-ошибки-авторизации-401-unauthorized)
    - [15.2. Конфликты устройств (Device busy)](#152-конфликты-устройств-device-busy)
    - [15.3. Проблемы с портами](#153-проблемы-с-портами)
    - [15.4. Проверка логов](#154-проверка-логов)
16. [❓ FAQ / Частые вопросы](#16--faq--частые-вопросы)
17. [🎉 Готово](#17--готово)

---

**Архитектура:** Мини‑ПК на Intel N100 дома + Xray VPN + Frigate + Home Assistant

---

# 1. 🛒 Что нужно купить

## 1.1. Мини‑ПК

**⭐ Рекомендуемый выбор: Beelink Mini S12 Pro (Intel N100)**

- Процессор: Intel N100
- ОЗУ: минимум 8GB, рекомендовано 16GB
- SSD: минимум 128GB, рекомендовано 256GB (видео с камер занимает место)
- Цена: ~80–100€
- Плюсы: тихий, экономичный (~10W), легко тянет HA + Frigate + голос одновременно

**Другие варианты на Intel N100 (тоже подойдут):**

- Beelink EQ12
- GMKtec G3
- Minisforum UN100
- Intel NUC
- Любой мини‑ПК на Intel N100 / N95

> ⚠️ Raspberry Pi не рекомендуется для связки HA + Frigate + голосовое управление —
> процессор ARM слабее, и система будет работать на пределе при нескольких камерах.

---

## 1.2. Камеры (RTSP + хорошая детекция)

### Бюджетные:

- Reolink RLC‑520A  
- Reolink RLC‑810A  
- TP‑Link Tapo C310

### Средний сегмент:

- Hikvision DS‑2CD2043G2‑I  
- Dahua IPC‑HFW2431S‑S‑S2

### Премиум:

- UniFi G4 Pro  
- UniFi G5 Bullet

---

## 1.3. Питание и сеть

Если камеры PoE:

- PoE‑инжектор или PoE‑коммутатор  
  - TP‑Link TL‑SG1005P  
  - Mikrotik RB260GSP  

Если Wi‑Fi — ничего не нужно.

---

## 1.4. Хранилище

- SSD 128–512 GB  
- USB‑корпус для SSD (если Raspberry Pi)

---

## 1.5. Дополнительно

- Google Coral USB TPU (ускоряет детекцию)  
- HDMI‑CEC монитор

---

# 2. 🧩 Общая архитектура системы

```
IP Камеры → Raspberry Pi / Мини‑ПК → Frigate → Home Assistant → Telegram
                                              ↓
                                           HDMI‑CEC
                                              ↓
                                       Включение монитора

Удалённый доступ → Xray VPN → Frigate / Home Assistant / Камеры
```

---

# 3. 🖥 Установка ОС

## Raspberry Pi:

- Raspberry Pi OS Lite (64-bit)  
- Включить SSH (файл `ssh` в корне SD)

## Мини‑ПК:

- **Ubuntu Server 24.04 LTS** (рекомендуется)
- Или Ubuntu Server 22.04 LTS

---

# 4. 🐳 Установка Docker + Docker Compose

> **Важно:** Устанавливаем Docker из официального репозитория, а не из стандартных репозиториев Ubuntu.

## Шаг 1: Обновление системы и установка зависимостей

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

## Шаг 2: Добавление официального GPG-ключа Docker

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

## Шаг 3: Добавление репозитория Docker

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Шаг 4: Установка Docker Engine и Docker Compose

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Шаг 5: Добавление пользователя в группу docker

```bash
sudo usermod -aG docker $USER
```

## Шаг 6: Перезагрузка

```bash
sudo reboot
```

## Проверка установки

После перезагрузки проверь, что Docker работает:

```bash
docker --version
docker compose version
```

Должно вывести версии Docker и Docker Compose.

---

# 5. 📦 Установка Frigate

```bash
mkdir -p ~/frigate/config
mkdir -p ~/frigate/media
cd ~/frigate
```

Создай файл `docker-compose.yml`:

```bash
nano docker-compose.yml
```

Вставь следующее содержимое:

```yaml
services:
  frigate:
    container_name: frigate
    image: ghcr.io/blakeblackshear/frigate:stable
    privileged: true
    restart: unless-stopped
    shm_size: "512mb"
    volumes:
      - ./config:/config
      - ./media:/media/frigate
      - /etc/localtime:/etc/localtime:ro
      - /dev/bus/usb:/dev/bus/usb  # Для USB Coral (если используешь)
    ports:
      - "8971:8971"  # Authenticated UI (рекомендуется)
      - "5000:5000"  # Internal unauthenticated UI
      - "8554:8554"  # RTSP restreaming
      - "8555:8555/tcp"  # WebRTC
      - "8555:8555/udp"  # WebRTC
    environment:
      FRIGATE_RTSP_PASSWORD: "password"  # Измени на свой пароль
```

Сохрани файл: `Ctrl+O`, `Enter`, `Ctrl+X`.

> **Примечание:** Начиная с Frigate 0.13+, основной порт для UI — `8971` (с аутентификацией). Порт `5000` — для внутреннего использования без аутентификации.

---

# 6. 🎥 Настройка камер в Frigate

## 6.1. IP камеры (RTSP)

### Подготовка камер (на примере HiWatch DS-I200)

Перед настройкой Frigate нужно правильно настроить камеры.

### Шаг 1: Проверить RTSP (обычно включен по умолчанию)

В большинстве камер HiWatch/Hikvision **RTSP включен по умолчанию** и работает на порту 554.

**Проверь, работает ли RTSP:**

```bash
# Установи ffmpeg (если ещё не установлен)
sudo apt install ffmpeg

# Попробуй получить кадр с камеры
ffmpeg -i rtsp://admin:твой_пароль@10.0.0.21:554/Streaming/Channels/102 -frames:v 1 test.jpg
```

Если команда выполнилась и создался файл `test.jpg` — **RTSP уже работает**, переходи к Шагу 2.

**Если RTSP не работает**, зайди в веб-интерфейс камеры и проверь настройки:

Зайди в веб-интерфейс камеры:
- Камера 1: `http://10.0.0.21`
- Камера 2: `http://10.0.0.22`

Логин/пароль: обычно `admin/admin` или тот, что установил при первой настройке.

**Возможные пути в меню (зависит от прошивки):**

**Вариант 1 (английский интерфейс):**
```
Configuration → Network → Advanced Settings → RTSP
```

**Вариант 2 (русский интерфейс):**
```
Настройки → Сеть → Доп. настройки → RTSP
```

**Вариант 3 (если не нашёл RTSP):**

RTSP может быть включен всегда без возможности отключения. В этом случае просто убедись, что порт 554 открыт:

```
Настройки → Сеть → Базовые настройки → Порт RTSP: 554
```

**Проверь:**
- ✅ Порт RTSP: `554` (по умолчанию)
- ✅ RTSP включен (если есть такая опция)

> **Примечание:** На скриншоте ты видишь "Enable RTCP" — это другой протокол (вспомогательный для RTSP). Его можно оставить как есть.

---

### Шаг 2: Настроить потоки видео

**Путь в меню:**
```
Configuration → Video/Audio → Stream Settings
```

**Main Stream (основной поток) — для записи:**
- Разрешение: 1920x1080 (Full HD) или 1280x720 (HD)
- Кодек: H.264
- Битрейт: 2048 Kbps
- FPS: 15-20

**Sub Stream (дополнительный поток) — для детекции:**
- Разрешение: 640x480 или 704x576
- Кодек: H.264
- Битрейт: 512 Kbps
- FPS: 10-15

> **Важно:** Sub Stream используется для детекции в Frigate (меньше нагрузка на CPU), Main Stream — для записи высокого качества.

---

### Шаг 3: Создать пользователя для Frigate (рекомендуется)

**Путь в меню:**
```
Configuration → System → User Management
```

- Имя: `frigate`
- Пароль: `твой_надёжный_пароль`
- Права: **Operator** (достаточно для просмотра потока)

---

### Шаг 4: RTSP URL для камер

**Формат URL для Hikvision/HiWatch:**

```
rtsp://username:password@IP:554/Streaming/Channels/101  # Main stream
rtsp://username:password@IP:554/Streaming/Channels/102  # Sub stream
```

**Для твоих камер:**

**Камера 1 (10.0.0.21):**
```
Main:  rtsp://frigate:password@10.0.0.21:554/Streaming/Channels/101
Sub:   rtsp://frigate:password@10.0.0.21:554/Streaming/Channels/102
```

**Камера 2 (10.0.0.22):**
```
Main:  rtsp://frigate:password@10.0.0.22:554/Streaming/Channels/101
Sub:   rtsp://frigate:password@10.0.0.22:554/Streaming/Channels/102
```

> **Примечание:** Замени `frigate:password` на свои учётные данные.

---

### ⚠️ Специальные символы в пароле

Если в пароле есть специальные символы (например, `!`, `@`, `#`, `$`, `%`, `&`), их нужно **URL-кодировать**, иначе возникнут ошибки.

#### Таблица URL-кодирования:

| Символ | URL-код | Пример пароля | URL-кодированный |
|--------|---------|---------------|------------------|
| `!` | `%21` | `Pass!123` | `Pass%21123` |
| `@` | `%40` | `My@Pass` | `My%40Pass` |
| `#` | `%23` | `Secure#99` | `Secure%2399` |
| `$` | `%24` | `Money$100` | `Money%24100` |
| `%` | `%25` | `Test%Pass` | `Test%25Pass` |
| `&` | `%26` | `You&Me` | `You%26Me` |
| `=` | `%3D` | `A=B` | `A%3DB` |
| `?` | `%3F` | `What?` | `What%3F` |
| ` ` (пробел) | `%20` | `My Pass` | `My%20Pass` |

#### Примеры использования:

**Пример 1: Пароль с восклицательным знаком**

Исходный пароль: `MyPass!2026`  
URL-кодированный: `MyPass%212026`

```
rtsp://admin:MyPass%212026@10.0.0.21:554/Streaming/Channels/102
```

**Пример 2: Пароль с несколькими спецсимволами**

Исходный пароль: `Secure@Pass!123`  
URL-кодированный: `Secure%40Pass%21123`

```
rtsp://frigate:Secure%40Pass%21123@10.0.0.22:554/Streaming/Channels/102
```

**Пример 3: Пароль с пробелом**

Исходный пароль: `My Password`  
URL-кодированный: `My%20Password`

```
rtsp://admin:My%20Password@10.0.0.21:554/Streaming/Channels/101
```

#### Как использовать в командной строке:

**Вариант 1: Одинарные кавычки (для bash)**

```bash
ffmpeg -i 'rtsp://admin:MyPass!2026@10.0.0.21:554/Streaming/Channels/102' -frames:v 1 test.jpg
```

**Вариант 2: URL-кодирование (работает везде)**

```bash
ffmpeg -i rtsp://admin:MyPass%212026@10.0.0.21:554/Streaming/Channels/102 -frames:v 1 test.jpg
```

#### Как использовать в конфигурации Frigate:

В файле `config.yml` **всегда используй URL-кодирование**:

```yaml
cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://admin:MyPass%212026@10.0.0.21:554/Streaming/Channels/102
          roles:
            - detect
```

> **Рекомендация:** Для упрощения настройки используй пароль без специальных символов (только буквы и цифры), например: `FrigatePass2026`.

---

### Шаг 5: Проверка RTSP потока

Перед настройкой Frigate проверь, что потоки работают:

```bash
# Установи ffmpeg (если ещё не установлен)
sudo apt install ffmpeg

# Проверь поток камеры 1 (Sub Stream для детекции)
ffmpeg -i rtsp://frigate:password@10.0.0.21:554/Streaming/Channels/102 -frames:v 1 -update 1 test1.jpg

# Проверь поток камеры 2 (Sub Stream для детекции)
ffmpeg -i rtsp://frigate:password@10.0.0.22:554/Streaming/Channels/102 -frames:v 1 -update 1 test2.jpg
```

> **Примечание:** Если в пароле есть спецсимволы, используй URL-кодирование (см. раздел выше).

Если команды выполнились без ошибок и создались файлы `test1.jpg` и `test2.jpg` — всё работает!

Посмотри изображения:

```bash
ls -lh test*.jpg
```

**Важно:** Обрати внимание на разрешение в выводе ffmpeg:

```
Stream #0:0: Video: h264 (Main), yuvj420p(pc, bt709, progressive), 640x360, 25 fps
                                                                     ^^^^^^^^
                                                                     Это разрешение!
```

Запомни разрешение (например, `640x360`) — оно понадобится для конфигурации Frigate.

**Для HiWatch DS-I200:** Sub Stream обычно имеет разрешение **640x360**.

---

### Что НЕ нужно менять в камерах:

❌ Не включай облачные сервисы  
❌ Не включай P2P  
❌ Не включай Hik-Connect  
❌ Не открывай порты на роутере (если используешь VPN)  
❌ Не включай UPnP

---

## Конфигурация Frigate

Создай файл конфигурации:

```bash
nano ~/frigate/config/config.yml
```

Вставь следующее содержимое:

```yaml
mqtt:
  enabled: False

detectors:
  cpu1:
    type: cpu

cameras:
  front_door:  # Камера 1 - входная дверь
    ffmpeg:
      inputs:
        # Sub stream для детекции (меньше нагрузка на CPU)
        - path: rtsp://frigate:password@10.0.0.21:554/Streaming/Channels/102
          roles:
            - detect
        # Main stream для записи (высокое качество)
        - path: rtsp://frigate:password@10.0.0.21:554/Streaming/Channels/101
          roles:
            - record
    detect:
      width: 640   # Для HiWatch DS-I200: 640x360 (проверь вывод ffmpeg!)
      height: 360  # Другие камеры могут иметь 704x576 или иное разрешение
      fps: 10
      enabled: True
    objects:
      track:
        - person
        - car
        - dog
        - cat
    record:
      enabled: True
      retain:
        days: 7  # Хранить записи 7 дней
        mode: motion  # Записывать только при движении
    snapshots:
      enabled: True
      retain:
        default: 14  # Хранить снимки 14 дней
  
  backyard:  # Камера 2 - задний двор
    ffmpeg:
      inputs:
        - path: rtsp://frigate:password@10.0.0.22:554/Streaming/Channels/102
          roles:
            - detect
        - path: rtsp://frigate:password@10.0.0.22:554/Streaming/Channels/101
          roles:
            - record
    detect:
      width: 640   # Для HiWatch DS-I200: 640x360 (проверь вывод ffmpeg!)
      height: 360  # Другие камеры могут иметь 704x576 или иное разрешение
      fps: 10
      enabled: True
    objects:
      track:
        - person
        - car
        - dog
        - cat
    record:
      enabled: True
      retain:
        days: 7
        mode: motion
    snapshots:
      enabled: True
      retain:
        default: 14
```

Сохрани файл: `Ctrl+O`, `Enter`, `Ctrl+X`.

> **Важно:** 
> - Замени `frigate:password` на свои учётные данные (используй URL-кодирование для спецсимволов)
> - Измени названия камер (`front_door`, `backyard`) на свои
> - **Укажи правильное разрешение** `width` и `height` из вывода команды ffmpeg (см. Шаг 5)

**Как узнать разрешение:**

Посмотри на вывод команды `ffmpeg` из Шага 5:

```
Stream #0:0: Video: h264 (Main), yuvj420p(pc, bt709, progressive), 640x360, 25 fps
                                                                     ^^^^^^^^
                                                                     width x height
```

В этом примере: `width: 640`, `height: 360`.

---

### Альтернативные RTSP URL (если основной не работает)

Некоторые модели HiWatch/Hikvision используют другие форматы URL:

**Вариант 1 (основной, описан выше):**
```
rtsp://user:pass@IP:554/Streaming/Channels/101
rtsp://user:pass@IP:554/Streaming/Channels/102
```

**Вариант 2:**
```
rtsp://user:pass@IP:554/h264/ch1/main/av_stream
rtsp://user:pass@IP:554/h264/ch1/sub/av_stream
```

**Вариант 3:**
```
rtsp://user:pass@IP:554/cam/realmonitor?channel=1&subtype=0  # Main
rtsp://user:pass@IP:554/cam/realmonitor?channel=1&subtype=1  # Sub
```

Если первый вариант не работает, попробуй другие.

---

## 6.2. USB веб-камеры

USB веб-камеры можно добавить в Frigate для детекции объектов и записи. Это полезно для внутреннего видеонаблюдения.

### Шаг 1: Проверка USB камеры

Проверь, что камера определяется системой:

```bash
ls -l /dev/video*
```

Должно показать устройства типа `/dev/video0`, `/dev/video1`.

Проверь поддерживаемые форматы:

```bash
# Установи утилиты (если ещё не установлены)
sudo apt install v4l-utils

# Проверь форматы камеры
v4l2-ctl --list-formats-ext -d /dev/video0
```

Ищи формат **MJPEG** — он лучше всего подходит для Frigate.

**Пример вывода:**

```
[0]: 'MJPG' (Motion-JPEG, compressed)
    Size: Discrete 640x480
        Interval: Discrete 0.033s (30.000 fps)
    Size: Discrete 1280x720
        Interval: Discrete 0.033s (30.000 fps)
    Size: Discrete 1920x1080
        Interval: Discrete 0.033s (30.000 fps)
```

Запомни поддерживаемые разрешения и FPS.

---

### Шаг 2: Остановить конфликтующие программы

USB камера может использоваться только одной программой одновременно. Если камера уже используется (например, mjpg-streamer или Home Assistant), останови эти программы.

**Проверь, что использует камеру:**

```bash
sudo lsof /dev/video0
```

**Если там mjpg-streamer:**

```bash
# Останови процесс
sudo kill <PID>

# Отключи автозапуск
sudo systemctl stop mjpg-streamer
sudo systemctl disable mjpg-streamer
```

**Если камера настроена в Home Assistant:**

Удали интеграцию MJPEG Camera из Home Assistant (Settings → Devices & Services).

---

### Шаг 3: Добавить USB камеру в docker-compose.yml

```bash
nano ~/frigate/docker-compose.yml
```

Добавь `/dev/video0` в секцию `volumes:`:

```yaml
services:
  frigate:
    container_name: frigate
    image: ghcr.io/blakeblackshear/frigate:stable
    privileged: true
    restart: unless-stopped
    shm_size: "512mb"
    volumes:
      - ./config:/config
      - ./media:/media/frigate
      - /etc/localtime:/etc/localtime:ro
      - /dev/bus/usb:/dev/bus/usb
      - /dev/video0:/dev/video0  # USB камера
    ports:
      - "8971:8971"
      - "5000:5000"
      - "8554:8554"
      - "8555:8555/tcp"
      - "8555:8555/udp"
    environment:
      FRIGATE_RTSP_PASSWORD: "password"
```

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`.

---

### Шаг 4: Добавить USB камеру в config.yml

```bash
nano ~/frigate/config/config.yml
```

Добавь камеру в секцию `cameras:` (после IP камер):

```yaml
cameras:
  # ... IP камеры ...
  
  indoor_usb:  # USB веб-камера
    ffmpeg:
      hwaccel_args: []  # Отключить аппаратное ускорение для USB камер
      inputs:
        - path: /dev/video0
          input_args: -f v4l2 -input_format mjpeg -video_size 640x480 -framerate 30
          roles:
            - detect
            - record
    detect:
      width: 640
      height: 480
      fps: 15  # Для детекции достаточно 15 fps (меньше нагрузка на CPU)
      enabled: true
    objects:
      track:
        - person
        - dog
        - cat
    record:
      enabled: true
      continuous:
        days: 0
      motion:
        days: 7
    snapshots:
      enabled: true
      retain:
        default: 14
```

**Параметры:**

- `hwaccel_args: []` — отключает GPU (USB камеры не поддерживают аппаратное ускорение)
- `-video_size 640x480` — разрешение (можно изменить на 1280x720 или 1920x1080)
- `-framerate 30` — FPS захвата (камера захватывает 30 кадров в секунду)
- `fps: 15` — FPS детекции (Frigate анализирует только 15 кадров для экономии CPU)

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`.

---

### Шаг 5: Перезапустить Frigate

```bash
cd ~/frigate
docker compose down
docker compose up -d
```

Проверь логи:

```bash
docker compose logs -f frigate | grep indoor_usb
```

Если всё работает, увидишь:

```
[INFO] Camera processor started for indoor_usb
```

---

### Выбор разрешения для USB камеры

| Разрешение | FPS | Качество | Нагрузка CPU | Рекомендация |
|------------|-----|----------|--------------|--------------|
| **640x480** | 30 | Среднее | Низкая | ✅ Оптимально для детекции |
| **1280x720** | 30 | Хорошее | Средняя | ✅ Баланс качества и производительности |
| **1920x1080** | 30 | Отличное | Высокая | ⚠️ Только для мощных CPU |

**Для просмотра live stream:** используется разрешение из `input_args` (например, 640x480 @ 30fps).  
**Для детекции:** используется параметр `fps` (например, 15 fps) — Frigate анализирует каждый второй кадр.

---

### Шаг 6: Оптимизация USB камеры — снижение нагрузки CPU

#### Почему USB камера использует больше CPU чем IP камеры?

**Типичная нагрузка:**
- IP камера (RTSP, H.264): 3-5% CPU
- USB камера (MJPEG): **15-25% CPU**

**Причины:**

1. **Формат MJPEG требует больше CPU**
   - Каждый кадр = отдельное JPEG изображение
   - FFmpeg декодирует каждый кадр отдельно
   - IP камеры используют H.264 (более эффективный)

2. **Нет аппаратного ускорения**
   - `hwaccel_args: []` — GPU не используется
   - Вся обработка на CPU

3. **Один поток для детекции И записи**
   - IP камеры: 2 потока (Sub Stream + Main Stream)
   - USB камера: 1 поток для всего

---

#### ⚠️ Важно: Разрешение в detect должно совпадать с input_args

**Нельзя делать так:**

```yaml
# ❌ НЕПРАВИЛЬНО
input_args: ... -video_size 1280x720 ...
detect:
  width: 640   # Не совпадает!
  height: 480
```

**Почему это не работает:**
- FFmpeg захватывает видео с разрешением из `input_args`
- Frigate получает кадры **без изменения размера**
- Если разрешения не совпадают → ошибка или искажение

**Правильно:**

```yaml
# ✅ ПРАВИЛЬНО
input_args: ... -video_size 640x480 ...
detect:
  width: 640   # Совпадает!
  height: 480
```

---

#### Как проверить поддерживаемые форматы камеры

```bash
# Установи утилиты
sudo apt install v4l-utils

# Проверь форматы
v4l2-ctl --list-formats-ext -d /dev/video0
```

**Пример вывода:**

```
[0]: 'MJPG' (Motion-JPEG, compressed)
    Size: Discrete 640x480
        Interval: Discrete 0.033s (30.000 fps)
    Size: Discrete 1280x720
        Interval: Discrete 0.033s (30.000 fps)
    Size: Discrete 1920x1080
        Interval: Discrete 0.033s (30.000 fps)
```

Используй формат **MJPEG** — он лучше всего подходит для Frigate.

---

#### Проверка наличия второго устройства

Некоторые USB камеры создают два устройства (`/dev/video0` и `/dev/video1`):

```bash
ls -l /dev/video*
v4l2-ctl --list-formats-ext -d /dev/video1
```

**Если `/dev/video1` показывает ошибку `ioctl: VIDIOC_ENUM_FMT`** — это устройство управления, а не видеопоток. Используй только `/dev/video0`.

---

#### 📊 Три варианта конфигурации

##### Вариант 1: Максимальная экономия CPU (рекомендуется)

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
    enabled: true
  objects:
    track:
      - person
  record:
    enabled: true
    continuous:
      days: 0
    motion:
      days: 3
  snapshots:
    enabled: true
    retain:
      default: 7
```

**Результат:**
- CPU: ~8-10% (вместо 21%)
- Качество: Достаточно для детекции человека
- Место на диске: ~300-500 MB/сутки

---

##### Вариант 2: Баланс качества и производительности

```yaml
indoor_usb:
  ffmpeg:
    hwaccel_args: []
    inputs:
      - path: /dev/video0
        input_args: -f v4l2 -input_format mjpeg -video_size 1280x720 -framerate 20
        roles:
          - detect
          - record
  detect:
    width: 1280
    height: 720
    fps: 10
    enabled: true
  objects:
    track:
      - person
  record:
    enabled: true
    continuous:
      days: 0
    motion:
      days: 3
  snapshots:
    enabled: true
    retain:
      default: 7
```

**Результат:**
- CPU: ~15-18%
- Качество: Хорошее (HD)
- Место на диске: ~1-1.5 GB/сутки

---

##### Вариант 3: Максимальное качество

```yaml
indoor_usb:
  ffmpeg:
    hwaccel_args: []
    inputs:
      - path: /dev/video0
        input_args: -f v4l2 -input_format mjpeg -video_size 1920x1080 -framerate 20
        roles:
          - detect
          - record
  detect:
    width: 1920
    height: 1080
    fps: 10
    enabled: true
  objects:
    track:
      - person
  record:
    enabled: true
    continuous:
      days: 0
    motion:
      days: 3
  snapshots:
    enabled: true
    retain:
      default: 7
```

**Результат:**
- CPU: ~25-30%
- Качество: Отличное (Full HD)
- Место на диске: ~2-3 GB/сутки

---

#### 💡 Почему 20 fps вместо 30?

1. **Для детекции человека 10 fps достаточно** — человек не телепортируется
2. **Framerate 20 → анализ 10 fps** = экономия 33% CPU
3. **Запись в 20 fps всё ещё плавная** (кино снимают в 24 fps)

---

#### Сравнительная таблица

| Параметр | 640x480 @ 20fps | 1280x720 @ 20fps | 1920x1080 @ 20fps |
|----------|-----------------|------------------|-------------------|
| CPU FFmpeg | ~8-10% | ~15-18% | ~25-30% |
| Качество записи | Среднее | Хорошее | Отличное |
| Качество детекции | ✅ Отлично | ✅ Отлично | ✅ Отлично |
| Место на диске (сутки) | ~300-500 MB | ~1-1.5 GB | ~2-3 GB |

**Вывод:** Для детекции человека **640x480 достаточно**. Высокое разрешение нужно только если хочешь рассмотреть мелкие детали на записи (номера машин, лица).

---

#### Как применить изменения

1. Открой конфиг:
```bash
nano ~/frigate/config/config.yml
```

2. Измени параметры камеры

3. Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`

4. Перезапусти Frigate:
```bash
cd ~/frigate
docker compose restart
```

5. Проверь нагрузку через 1-2 минуты в Frigate UI (System → Stats)

---

# 7. ▶ Запуск Frigate

## 7.1. Первый запуск

```bash
cd ~/frigate
docker compose up -d
```

> **Примечание:** Используй `docker compose` (с пробелом), а не `docker-compose` (с дефисом). Новая версия Docker Compose — это плагин.

Проверка логов:

```bash
docker compose logs -f frigate
```

Нажми `Ctrl+C` чтобы выйти из просмотра логов (контейнер продолжит работать).

---

## 7.2. Настройка аутентификации (порт 8971)

Frigate имеет два порта для веб-интерфейса:

- **Порт 5000** — HTTP без аутентификации (для локальной сети)
- **Порт 8971** — HTTPS с аутентификацией (для удалённого доступа)

### Почему порт 8971 не работает сразу?

При первом запуске порт 8971 требует настройки аутентификации. Если попытаешься зайти на `http://IP:8971`, увидишь ошибку **400 Bad Request**.

**Причина:** Порт 8971 работает только по **HTTPS**, а не HTTP.

---

### Настройка аутентификации

**Шаг 1: Включи аутентификацию в config.yml**

```bash
nano ~/frigate/config/config.yml
```

Добавь в **самое начало** файла (перед `mqtt:`):

```yaml
auth:
  enabled: true
  reset_admin_password: true

mqtt:
  enabled: false
...
```

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`.

**Шаг 2: Перезапусти Frigate**

```bash
cd ~/frigate
docker compose restart
```

**Шаг 3: Создай пользователя**

Открой в браузере:

```
http://IP:5000/
```

Должно появиться окно создания администратора:
- **Username:** `admin` (или любой другой)
- **Password:** придумай надёжный пароль

**Шаг 4: Зайди на порт 8971**

Теперь открой (обрати внимание на **https://**):

```
https://IP:8971/
```

Браузер покажет предупреждение о сертификате — нажми "Продолжить" или "Accept Risk".

Войди с созданным логином/паролем.

---

### Разница между портами

| Порт | Протокол | Аутентификация | Использование |
|------|----------|----------------|---------------|
| **5000** | HTTP | Нет | Локальная сеть (дома) |
| **8971** | HTTPS | Да | Удалённый доступ (через VPN) |

**Рекомендация:**
- Для локальной сети используй порт 5000
- Для удалённого доступа через VPN используй порт 8971

---

### Проверка работы камер

Открой Frigate UI (`http://IP:5000` или `https://IP:8971`) и проверь:

✅ Все камеры отображаются  
✅ Видео показывается в реальном времени  
✅ Детекция объектов работает (помаши рукой перед камерой)  

Если камеры не работают, смотри раздел [Troubleshooting](#15--troubleshooting--решение-проблем).

---

# 8. 🏠 Установка Home Assistant

```bash
docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Europe/Moscow \
  -v /home/$USER/ha:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```

Открыть:  
`http://IP:8123`

---

# 9. 🔔 Уведомления в Telegram

## 9.1. Создать бота

Через @BotFather → получить TOKEN

## 9.2. Добавить в Home Assistant

`configuration.yaml`:

```yaml
telegram_bot:
  - platform: polling
    api_key: "ТОКЕН"
    allowed_chat_ids:
      - CHAT_ID

notify:
  - platform: telegram
    name: tg
    chat_id: CHAT_ID
```

---

# 10. 👤 Автоматизация уведомлений

```yaml
automation:
  - alias: "Frigate Person Alert"
    trigger:
      platform: mqtt
      topic: frigate/events
    condition:
      condition: template
      value_template: "{{ trigger.payload_json['type'] == 'person' }}"
    action:
      - service: notify.tg
        data:
          message: "Обнаружен человек на камере {{ trigger.payload_json['camera'] }}"
```

---

# 11. 🖥 Автовключение монитора (HDMI‑CEC)

```bash
sudo apt install cec-utils
```

Команда:

```bash
echo "on 0" | cec-client -s -d 1
```

Добавить в Home Assistant:

```yaml
shell_command:
  monitor_on: 'echo "on 0" | cec-client -s -d 1'
```

> ⚠️ Требует поддержки HDMI‑CEC на мониторе/телевизоре. Если её нет (например, Samsung UE39F5000AK и другие бюджетные модели без Anynet+) — используй BroadLink RM4C Mini (см. раздел 11.1).

---

## 11.1. Альтернатива: BroadLink RM4C Mini (без HDMI‑CEC)

Если телевизор не поддерживает HDMI‑CEC, можно управлять им через ИК-бластер BroadLink RM4C Mini. Он интегрируется в Home Assistant нативно, без HACS.

### Подключение устройства

1. Установи приложение **BroadLink** на телефон
2. Добавь RM4C Mini в приложение (подключит к Wi-Fi через Bluetooth)
3. Зафиксируй IP устройства в роутере (DHCP reservation) — HA будет обращаться по нему
4. В Home Assistant: **Settings → Integrations → Add Integration → BroadLink**  
   HA обнаружит устройство автоматически, либо введи IP вручную

### Обучение ИК-команде (Power)

В **Developer Tools → Actions** выполни:

```yaml
action: remote.learn_command
target:
  entity_id: remote.broadlink_rm4c_mini
data:
  device: samsung_tv
  command: power
```

После вызова направь пульт от телевизора на RM4C Mini и нажми кнопку **Power** — команда запишется.

### Использование в автоматизации

```yaml
action: remote.send_command
target:
  entity_id: remote.broadlink_rm4c_mini
data:
  device: samsung_tv
  command: power
```

### Добавить в Home Assistant (shell_command-стиль через сервис)

Или через `script` для удобного вызова из других автоматизаций:

```yaml
script:
  tv_power_toggle:
    alias: "TV Power Toggle"
    sequence:
      - action: remote.send_command
        target:
          entity_id: remote.broadlink_rm4c_mini
        data:
          device: samsung_tv
          command: power
```

> 💡 RM4C Mini — только ИК (без RF). Убедись, что устройство расположено с прямой видимостью на ИК-приёмник телевизора (обычно снизу по центру панели).

---

# 12. 🌍 Удалённый доступ через Xray VPN

## 12.1. Установка Xray client

```bash
bash <(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)
```

## 12.2. Конфиг клиента

`/usr/local/etc/xray/config.json`:

```json
{
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks"
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "SERVER_IP",
            "port": 443,
            "users": [
              {
                "id": "UUID",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls"
      }
    }
  ]
}
```

Запуск:

```bash
systemctl restart xray
systemctl enable xray
```

---

# 13. 🔐 Удалённый доступ

После подключения к Xray:

- **Frigate (authenticated):** `http://VPN_IP:8971`  
- **Frigate (internal):** `http://VPN_IP:5000`  
- **Home Assistant:** `http://VPN_IP:8123`  
- **Камеры:** `rtsp://VPN_IP/...`

> **Рекомендация:** Используй порт 8971 для доступа к Frigate — он требует аутентификацию.

---

# 14. ⚙️ Оптимизация производительности

## Выбор разрешения для детекции

Чем ниже разрешение для детекции, тем меньше нагрузка на CPU.

**Рекомендации:**

| Тип камеры | Разрешение детекции | Разрешение записи | FPS детекции |
|------------|---------------------|-------------------|--------------|
| IP камера (RTSP) | 640x360 (Sub Stream) | 1920x1080 (Main Stream) | 10-15 |
| USB камера | 640x480 | 640x480 | 10-15 |
| USB камера (качество) | 1280x720 | 1280x720 | 10 |

---

## FPS: детекция vs запись

**Параметры в config.yml:**

```yaml
cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://...
          input_args: ... -framerate 30  # Камера захватывает 30 fps
    detect:
      fps: 15  # Frigate анализирует только 15 fps
```

**Что это значит:**

- **`framerate 30`** — камера захватывает 30 кадров в секунду
- **`fps: 15`** — Frigate анализирует только 15 кадров (каждый второй)
- **Live stream** показывает 30 fps (плавно)
- **Запись** сохраняется в 30 fps
- **Детекция** работает на 15 fps (экономия CPU)

**Для детекции человека достаточно 10-15 fps.** Больше не нужно.

---

## Использование Google Coral TPU

Google Coral USB Accelerator ускоряет детекцию в 10-20 раз.

**Без Coral:**
- CPU: Intel N100
- Камеры: 3-4 камеры максимум
- Нагрузка CPU: 60-80%

**С Coral:**
- CPU: Intel N100
- Камеры: 10+ камер
- Нагрузка CPU: 10-20%

**Настройка:**

```yaml
detectors:
  coral:
    type: edgetpu
    device: usb
```

---

## Настройка shm_size

`shm_size` — это размер общей памяти для обработки видео.

**Рекомендации:**

| Количество камер | shm_size |
|------------------|----------|
| 1-2 камеры | 256mb |
| 3-4 камеры | 512mb |
| 5-8 камер | 1gb |
| 9+ камер | 2gb |

В `docker-compose.yml`:

```yaml
services:
  frigate:
    shm_size: "512mb"  # Измени на нужное значение
```

---

# 15. 🔧 Troubleshooting / Решение проблем

## 15.1. Ошибки авторизации (401 Unauthorized)

**Ошибка:**

```
[rtsp @ 0x...] method DESCRIBE failed: 401 Unauthorized
Error opening input file rtsp://user:pass@IP:554/...
```

**Причины:**

1. **Неправильный логин/пароль**
2. **Логин для веб-интерфейса ≠ логин для RTSP**
3. **Специальные символы в пароле не закодированы**

**Решение 1: Создать отдельного пользователя для RTSP**

Зайди в веб-интерфейс камеры:

```
Configuration → System → User Management → Add
```

Создай пользователя:
- Имя: `frigate`
- Пароль: `простой_пароль_без_спецсимволов` (например, `Frigate2026`)
- Права: **Operator**

Используй этого пользователя в config.yml:

```yaml
- path: rtsp://frigate:Frigate2026@10.0.0.21:554/Streaming/Channels/102
```

**Решение 2: URL-кодирование пароля**

Если пароль содержит спецсимволы, закодируй их:

| Символ | Код |
|--------|------|
| `!` | `%21` |
| `@` | `%40` |
| `#` | `%23` |
| `$` | `%24` |

Пример: `Pass!123` → `Pass%21123`

**Решение 3: Проверка с ffmpeg**

Проверь, работает ли RTSP:

```bash
ffmpeg -i 'rtsp://user:pass@IP:554/Streaming/Channels/102' -frames:v 1 -update 1 test.jpg
```

Если работает — используй те же учётные данные в Frigate.

---

## 15.2. Конфликты устройств (Device busy)

**Ошибка:**

```
[in#0 @ 0x...] Error opening input: Device or resource busy
Error opening input file /dev/video0
```

**Причина:** USB камера уже используется другой программой (mjpg-streamer, Home Assistant, VLC).

**Решение:**

**Шаг 1: Найди процесс**

```bash
sudo lsof /dev/video0
```

**Пример вывода:**

```
COMMAND     PID USER   FD   TYPE DEVICE
mjpg_stre 12345 root    4u   CHR   81,0  /dev/video0
```

**Шаг 2: Останови процесс**

```bash
sudo kill 12345
```

**Шаг 3: Отключи автозапуск (если это сервис)**

```bash
sudo systemctl stop mjpg-streamer
sudo systemctl disable mjpg-streamer
```

**Шаг 4: Перезапусти Frigate**

```bash
cd ~/frigate
docker compose restart
```

---

## 15.3. Проблемы с портами

### Порт 8971 показывает "400 Bad Request"

**Причина:** Порт 8971 работает только по HTTPS, а ты заходишь по HTTP.

**Решение:** Используй `https://` вместо `http://`:

```
https://10.0.0.3:8971/
```

Браузер покажет предупреждение о сертификате — нажми "Продолжить".

---

### Порт 5000 не открывается

**Проверь, что контейнер запущен:**

```bash
docker ps | grep frigate
```

Должно быть:

```
0.0.0.0:5000->5000/tcp
```

**Если порта нет:**

Проверь `docker-compose.yml`:

```yaml
ports:
  - "5000:5000"
```

Перезапусти:

```bash
docker compose down
docker compose up -d
```

---

## 15.4. Проверка логов

### Просмотр всех логов

```bash
docker compose logs frigate
```

### Просмотр последних 50 строк

```bash
docker compose logs frigate | tail -50
```

### Просмотр логов в реальном времени

```bash
docker compose logs -f frigate
```

Нажми `Ctrl+C` чтобы выйти.

### Поиск ошибок

```bash
docker compose logs frigate | grep -i error
```

### Логи конкретной камеры

```bash
docker compose logs frigate | grep camera_outside_1
```

---

# 16. ❓ FAQ / Частые вопросы

## Почему две камеры не работают с одним логином/паролем?

**Ответ:** Логин/пароль для **веб-интерфейса** камеры может отличаться от логина для **RTSP**. Некоторые камеры требуют создания отдельного пользователя для RTSP.

**Решение:** Создай пользователя `frigate` с правами **Operator** в настройках камеры.

---

## Как узнать разрешение камеры?

**Для IP камеры:**

```bash
ffmpeg -i rtsp://user:pass@IP:554/Streaming/Channels/102 -frames:v 1 test.jpg
```

Посмотри на вывод:

```
Stream #0:0: Video: h264, yuvj420p, 640x360, 25 fps
                                     ^^^^^^^^
                                     разрешение
```

**Для USB камеры:**

```bash
v4l2-ctl --list-formats-ext -d /dev/video0
```

---

## Можно ли использовать USB камеру и mjpg-streamer одновременно?

**Ответ:** Нет. USB камера может использоваться только одной программой одновременно.

**Решение:** Используй Frigate для детекции и записи, а для просмотра в Home Assistant подключи Frigate integration.

---

## Сколько FPS нужно для детекции человека?

**Ответ:** 10-15 FPS достаточно. Больше не улучшает качество детекции, но увеличивает нагрузку на CPU.

**Рекомендация:**
- Захват: 30 fps (плавное видео)
- Детекция: 10-15 fps (экономия CPU)

---

## Разница между Main Stream и Sub Stream?

**Main Stream:**
- Высокое разрешение (1920x1080)
- Для записи видео
- Большой битрейт

**Sub Stream:**
- Низкое разрешение (640x360)
- Для детекции объектов
- Маленький битрейт
- Меньше нагрузка на CPU

**В Frigate используются оба:**
- Sub Stream → детекция
- Main Stream → запись

---

## Нужен ли Google Coral TPU?

**Ответ:** Не обязательно, но очень рекомендуется.

**Без Coral:**
- 3-4 камеры максимум на Intel N100
- Высокая нагрузка CPU

**С Coral:**
- 10+ камер на Intel N100
- Низкая нагрузка CPU
- Быстрая детекция

**Цена:** ~60-80€

---

## Как настроить детекцию только когда меня нет дома?

**Ответ:** Используй автоматизации в Home Assistant.

**Пример:**

```yaml
automation:
  - alias: "Детекция когда нет дома"
    trigger:
      platform: state
      entity_id: person.your_name
      to: "not_home"
    action:
      - service: switch.turn_on
        entity_id: switch.frigate_camera_detect
```

Или используй зоны в Frigate (нарисуй зону детекции в UI).

---

# 17. 🎉 Готово

Ты получил:

- локальную детекцию человека  
- уведомления в Telegram  
- удалённый доступ через Xray  
- включение монитора  
- безопасную архитектуру без проброса портов  