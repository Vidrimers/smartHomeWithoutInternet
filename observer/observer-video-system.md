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
7. [▶ Запуск Frigate](#7--запуск-frigate)
8. [🏠 Установка Home Assistant](#8--установка-home-assistant)
9. [🔔 Уведомления в Telegram](#9--уведомления-в-telegram)
   - [9.1. Создать бота](#91-создать-бота)
   - [9.2. Добавить в Home Assistant](#92-добавить-в-home-assistant)
10. [👤 Автоматизация уведомлений](#10--автоматизация-уведомлений)
11. [🖥 Автовключение монитора (HDMI‑CEC)](#11--автовключение-монитора-hdmicec)
12. [🌍 Удалённый доступ через Xray VPN](#12--удалённый-доступ-через-xray-vpn)
    - [12.1. Установка Xray client](#121-установка-xray-client)
    - [12.2. Конфиг клиента](#122-конфиг-клиента)
13. [🔐 Удалённый доступ](#13--удалённый-доступ)
14. [🎉 Готово](#14--готово)

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

## Подготовка камер (на примере HiWatch DS-I200)

Перед настройкой Frigate нужно правильно настроить камеры.

### Шаг 1: Включить RTSP в камере

Зайди в веб-интерфейс камеры:
- Камера 1: `http://10.0.0.21`
- Камера 2: `http://10.0.0.22`

Логин/пароль: обычно `admin/admin` или тот, что установил при первой настройке.

**Путь в меню:**
```
Configuration → Network → Advanced Settings → RTSP
```

**Проверь:**
- ✅ RTSP включен (Enable RTSP)
- ✅ Порт: `554` (по умолчанию)

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

### Шаг 5: Проверка RTSP потока

Перед настройкой Frigate проверь, что потоки работают:

```bash
# Установи ffmpeg (если ещё не установлен)
sudo apt install ffmpeg

# Проверь поток камеры 1
ffmpeg -i rtsp://frigate:password@10.0.0.21:554/Streaming/Channels/102 -frames:v 1 test1.jpg

# Проверь поток камеры 2
ffmpeg -i rtsp://frigate:password@10.0.0.22:554/Streaming/Channels/102 -frames:v 1 test2.jpg
```

Если команды выполнились без ошибок и создались файлы `test1.jpg` и `test2.jpg` — всё работает!

Посмотри изображения:

```bash
ls -lh test*.jpg
```

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
      width: 704
      height: 576
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
      width: 704
      height: 576
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

> **Важно:** Замени `frigate:password` на свои учётные данные и измени названия камер (`front_door`, `backyard`) на свои.

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

# 7. ▶ Запуск Frigate

```bash
cd ~/frigate
docker compose up -d
```

> **Примечание:** Используй `docker compose` (с пробелом), а не `docker-compose` (с дефисом). Новая версия Docker Compose — это плагин.

Проверка логов:

```bash
docker compose logs -f frigate
```

Открыть UI:  
- **Authenticated (рекомендуется):** `http://IP:8971`
- **Internal (без пароля):** `http://IP:5000`

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

# 14. 🎉 Готово

Ты получил:

- локальную детекцию человека  
- уведомления в Telegram  
- удалённый доступ через Xray  
- включение монитора  
- безопасную архитектуру без проброса портов  