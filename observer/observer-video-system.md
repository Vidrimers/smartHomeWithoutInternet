# Observer-video-system.md

# 📹 Система видеонаблюдения с детекцией человека, уведомлениями и удалённым доступом

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

- Ubuntu Server 22.04 LTS

---

# 4. 🐳 Установка Docker + Docker Compose

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo usermod -aG docker $USER
```

Перезагрузка.

---

# 5. 📦 Установка Frigate

```bash
mkdir -p ~/frigate/config
mkdir -p ~/frigate/media
```

Создать `docker-compose.yml`:

```yaml
version: "3.9"
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
    ports:
      - "5000:5000"
      - "8554:8554"
      - "8555:8555/tcp"
      - "8555:8555/udp"
```

---

# 6. 🎥 Настройка камер в Frigate

Создать `config/config.yml`:

```yaml
mqtt:
  enabled: False

detectors:
  cpu1:
    type: cpu

cameras:
  front:
    ffmpeg:
      inputs:
        - path: rtsp://USER:PASS@IP:554/stream1
          roles:
            - detect
            - record
    detect:
      width: 1280
      height: 720
      enabled: True
    objects:
      track:
        - person
```

---

# 7. ▶ Запуск Frigate

```bash
cd ~/frigate
docker-compose up -d
```

Открыть:  
`http://IP:5000`

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

- Frigate: `http://VPN_IP:5000`  
- Home Assistant: `http://VPN_IP:8123`  
- Камеры: `rtsp://VPN_IP/...`

---

# 14. 🎉 Готово

Ты получил:

- локальную детекцию человека  
- уведомления в Telegram  
- удалённый доступ через Xray  
- включение монитора  
- безопасную архитектуру без проброса портов  