# Удалённый доступ к Home Assistant

Сравнение всех способов получить доступ к Home Assistant из интернета — от бесплатных до платных.

---

## 📋 Содержание

1. [Сравнительная таблица](#-сравнительная-таблица)
2. [VPN (WireGuard)](#1--vpn-wireguard--рекомендуется)
3. [VPN (Xray/V2Ray)](#2--vpn-xrayv2ray)
4. [Cloudflare Tunnel](#3--cloudflare-tunnel)
5. [Проброс портов + DynamicDNS](#4--проброс-портов--dynamicdns)
6. [Tailscale](#5--tailscale)
7. [Nabu Casa (Home Assistant Cloud)](#6--nabu-casa-home-assistant-cloud)
8. [Ngrok](#7--ngrok)
9. [Какой способ выбрать?](#-какой-способ-выбрать)

---

## 📊 Сравнительная таблица

| Способ | Цена | Сложность настройки | Безопасность | Скорость | Зависимость от облака |
|--------|------|---------------------|--------------|----------|----------------------|
| **VPN (WireGuard)** | ✅ Бесплатно | Средняя | ⭐⭐⭐⭐⭐ | Быстро | Нет |
| **VPN (Xray)** | ✅ Бесплатно | Сложная | ⭐⭐⭐⭐⭐ | Быстро | Нет |
| **Cloudflare Tunnel** | ✅ Бесплатно | Простая | ⭐⭐⭐⭐ | Средне | Да |
| **Проброс портов + DDNS** | ✅ Бесплатно | Средняя | ⭐⭐⭐ | Быстро | Нет |
| **Tailscale** | ✅ Бесплатно* | Очень простая | ⭐⭐⭐⭐⭐ | Быстро | Да |
| **Nabu Casa** | 💰 $6.50/мес | Очень простая | ⭐⭐⭐⭐ | Средне | Да |
| **Ngrok** | ✅/💰 Ограничено | Очень простая | ⭐⭐⭐ | Медленно | Да |

*Tailscale бесплатен до 100 устройств

---

## 1. 🔐 VPN (WireGuard) — Рекомендуется

**Лучший баланс безопасности, скорости и простоты.**

### Как работает

```
Телефон → WireGuard VPN → Домашняя сеть → Home Assistant (10.0.0.3:8123)
```

Ты подключаешься к домашней сети через VPN, как будто находишься дома.

### Плюсы

- ✅ Бесплатно
- ✅ Очень быстрый (современный протокол)
- ✅ Максимальная безопасность
- ✅ Доступ ко всей домашней сети (HA, камеры, NAS, принтеры)
- ✅ Работает на всех платформах (Android, iOS, Windows, Linux, macOS)
- ✅ Низкое потребление батареи

### Минусы

- ❌ Нужно настроить VPN-сервер дома
- ❌ Требуется открыть порт на роутере (обычно UDP 51820)
- ❌ Нужен статический IP или DynamicDNS

### Краткая инструкция

**На сервере (дома):**

```bash
# Установка WireGuard
sudo apt update
sudo apt install wireguard -y

# Генерация ключей
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key

# Конфигурация сервера
sudo nano /etc/wireguard/wg0.conf
```

```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = <server_private.key>

PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_public.key>
AllowedIPs = 10.8.0.2/32
```

```bash
# Запуск
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

**На роутере:**
- Открой порт UDP 51820 → IP сервера

**На телефоне/ноутбуке:**
- Установи WireGuard
- Импортируй конфиг клиента
- Подключись к VPN
- Открой `http://10.0.0.3:8123`

### Ссылки

- [Официальный сайт WireGuard](https://www.wireguard.com/)
- [Подробная инструкция](https://www.wireguard.com/quickstart/)

---

## 2. 🔐 VPN (Xray/V2Ray)

**Для продвинутых пользователей. Обходит блокировки, маскируется под обычный HTTPS-трафик.**

### Как работает

```
Телефон → Xray Client → Xray Server (дома) → Home Assistant
```

### Плюсы

- ✅ Бесплатно
- ✅ Максимальная безопасность
- ✅ Обходит блокировки VPN (маскируется под HTTPS)
- ✅ Гибкая настройка маршрутизации

### Минусы

- ❌ Сложная настройка
- ❌ Требуется домен и SSL-сертификат
- ❌ Меньше документации на русском

### Краткая инструкция

**На сервере:**

```bash
# Установка Xray
bash <(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)

# Генерация UUID
xray uuid

# Конфигурация
sudo nano /usr/local/etc/xray/config.json
```

```json
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "YOUR-UUID-HERE",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "certificates": [
            {
              "certificateFile": "/path/to/cert.pem",
              "keyFile": "/path/to/key.pem"
            }
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

```bash
sudo systemctl restart xray
sudo systemctl enable xray
```

**На клиенте:**
- Установи V2RayNG (Android) или V2RayX (iOS)
- Импортируй конфиг
- Подключись

### Ссылки

- [Xray GitHub](https://github.com/XTLS/Xray-core)
- [Документация](https://xtls.github.io/)

---

## 3. ☁️ Cloudflare Tunnel

**Простой и безопасный способ без открытия портов.**

### Как работает

```
Телефон → Cloudflare → Cloudflare Tunnel → Home Assistant (дома)
```

Cloudflare создаёт защищённый туннель от их серверов к твоему HA.

### Плюсы

- ✅ Бесплатно
- ✅ Не нужно открывать порты на роутере
- ✅ Автоматический SSL-сертификат
- ✅ Защита от DDoS
- ✅ Простая настройка

### Минусы

- ❌ Зависимость от Cloudflare
- ❌ Трафик идёт через их серверы
- ❌ Нужен домен (можно бесплатный через Cloudflare)

### Краткая инструкция

1. Зарегистрируйся на [Cloudflare](https://dash.cloudflare.com/)
2. Добавь домен (или используй бесплатный поддомен)
3. Установи `cloudflared` на сервере:

```bash
# Установка
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Авторизация
cloudflared tunnel login

# Создание туннеля
cloudflared tunnel create homeassistant

# Конфигурация
nano ~/.cloudflared/config.yml
```

```yaml
tunnel: homeassistant
credentials-file: /home/user/.cloudflared/<TUNNEL-ID>.json

ingress:
  - hostname: ha.yourdomain.com
    service: http://localhost:8123
  - service: http_status:404
```

```bash
# Запуск
cloudflared tunnel route dns homeassistant ha.yourdomain.com
cloudflared tunnel run homeassistant

# Автозапуск
sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
```

4. Открой `https://ha.yourdomain.com`

### Ссылки

- [Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/)
- [Документация](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)

---

## 4. 🌐 Проброс портов + DynamicDNS

**Классический способ. Прямой доступ через интернет.**

### Как работает

```
Телефон → yourhome.duckdns.org:8123 → Роутер (проброс порта) → Home Assistant
```

### Плюсы

- ✅ Бесплатно
- ✅ Прямое подключение (быстро)
- ✅ Полный контроль

### Минусы

- ❌ Нужно открыть порт на роутере (потенциальная уязвимость)
- ❌ Требуется SSL-сертификат
- ❌ Нужен DynamicDNS (если IP меняется)
- ❌ Менее безопасно, чем VPN

### Краткая инструкция

**1. Настройка DynamicDNS (DuckDNS):**

```bash
# Установка
mkdir ~/duckdns
cd ~/duckdns
nano duck.sh
```

```bash
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=YOURDOMAIN&token=YOUR-TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
```

```bash
chmod +x duck.sh
crontab -e
# Добавь строку:
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```

**2. Получение SSL-сертификата (Let's Encrypt):**

```bash
sudo apt install certbot -y
sudo certbot certonly --standalone -d yourhome.duckdns.org
```

**3. Настройка Home Assistant:**

Отредактируй `configuration.yaml`:

```yaml
http:
  ssl_certificate: /etc/letsencrypt/live/yourhome.duckdns.org/fullchain.pem
  ssl_key: /etc/letsencrypt/live/yourhome.duckdns.org/privkey.pem
```

**4. Проброс порта на роутере:**
- Открой порт 8123 (TCP) → IP сервера с HA

**5. Доступ:**
- `https://yourhome.duckdns.org:8123`

### Ссылки

- [DuckDNS](https://www.duckdns.org/)
- [Let's Encrypt](https://letsencrypt.org/)

---

## 5. 🌐 Tailscale

**Самый простой VPN. Настраивается за 5 минут.**

### Как работает

```
Телефон (Tailscale) ←→ Tailscale Cloud ←→ Сервер (Tailscale) → Home Assistant
```

Tailscale создаёт виртуальную сеть между твоими устройствами.

### Плюсы

- ✅ Бесплатно (до 100 устройств)
- ✅ Очень простая настройка (буквально 2 команды)
- ✅ Не нужно открывать порты
- ✅ Работает за NAT
- ✅ Автоматическое шифрование
- ✅ Приложения для всех платформ

### Минусы

- ❌ Зависимость от Tailscale (облачный сервис)
- ❌ Ограничение 100 устройств на бесплатном плане

### Краткая инструкция

**На сервере:**

```bash
# Установка
curl -fsSL https://tailscale.com/install.sh | sh

# Запуск
sudo tailscale up

# Получи IP
tailscale ip -4
```

**На телефоне/ноутбуке:**
1. Установи Tailscale из App Store / Google Play
2. Войди в тот же аккаунт
3. Включи VPN
4. Открой `http://<tailscale-ip>:8123`

### Ссылки

- [Tailscale](https://tailscale.com/)
- [Документация](https://tailscale.com/kb/)

---

## 6. 💰 Nabu Casa (Home Assistant Cloud)

**Официальное облачное решение от разработчиков Home Assistant.**

### Как работает

```
Телефон → Nabu Casa Cloud → Home Assistant (дома)
```

### Плюсы

- ✅ Настройка в 1 клик
- ✅ Автоматический SSL
- ✅ Не нужно открывать порты
- ✅ Интеграция с Alexa и Google Assistant
- ✅ Поддержка разработки HA

### Минусы

- ❌ Платно (см. цены ниже)
- ❌ Зависимость от облака

### Цены (2026)

| Регион | Месяц | Год |
|--------|-------|-----|
| 🇺🇸 США | $6.50 | $65 |
| 🇪🇺 ЕС | €7.50 | €75 |
| 🇬🇧 UK | £6.50 | £65 |
| 🇨🇦 Канада | $8.70 | $87 |
| 🌍 Другие | $6.50 | $65 |

> Годовая подписка = 10 месяцев (2 месяца бесплатно)

### Инструкция

1. В Home Assistant: **Settings → Home Assistant Cloud**
2. Нажми **Start 30-day free trial**
3. Зарегистрируйся
4. Включи **Remote Control**
5. Готово! Доступ через `https://xxxxx.ui.nabu.casa`

### Ссылки

- [Nabu Casa](https://www.nabucasa.com/)
- [Официальные цены](https://www.nabucasa.com/pricing/)

---

## 7. 🌐 Ngrok

**Быстрый способ для тестирования. Не рекомендуется для постоянного использования.**

### Как работает

```
Телефон → Ngrok Cloud → Ngrok Client (дома) → Home Assistant
```

### Плюсы

- ✅ Очень быстрая настройка (1 команда)
- ✅ Не нужно открывать порты
- ✅ Бесплатный план есть

### Минусы

- ❌ Бесплатный план: случайный URL, ограничения по трафику
- ❌ URL меняется при каждом перезапуске (бесплатный план)
- ❌ Платный план: $8/месяц за статический домен
- ❌ Не подходит для постоянного использования

### Краткая инструкция

```bash
# Установка
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update
sudo apt install ngrok

# Авторизация (получи токен на ngrok.com)
ngrok config add-authtoken YOUR_TOKEN

# Запуск
ngrok http 8123
```

Ngrok выдаст URL типа `https://abc123.ngrok.io` — открой его в браузере.

### Ссылки

- [Ngrok](https://ngrok.com/)

---

## 🤔 Какой способ выбрать?

### Для новичков:
**Tailscale** — проще не бывает, настраивается за 5 минут.

### Для продвинутых пользователей:
**WireGuard** — лучший баланс безопасности, скорости и контроля.

### Если не хочешь возиться с настройкой:
**Nabu Casa** — платно, но работает из коробки.

### Если нет статического IP и не хочешь открывать порты:
**Cloudflare Tunnel** — бесплатно и безопасно.

### Если нужна максимальная безопасность и обход блокировок:
**Xray/V2Ray** — сложно, но мощно.

### Для тестирования:
**Ngrok** — быстро поднять доступ на 5 минут.

---

## 🔒 Рекомендации по безопасности

Независимо от выбранного способа:

1. **Используй сильные пароли** в Home Assistant
2. **Включи двухфакторную аутентификацию** (Settings → Users → Security)
3. **Регулярно обновляй** Home Assistant и систему
4. **Ограничь доступ** к критичным устройствам (например, замкам)
5. **Мониторь логи** на подозрительную активность
6. **Используй отдельную сеть** для умных устройств (VLAN)

---

## 📚 Дополнительные ресурсы

- [Официальная документация HA по удалённому доступу](https://www.home-assistant.io/docs/configuration/remote/)
- [Форум Home Assistant](https://community.home-assistant.io/)
- [Reddit r/homeassistant](https://www.reddit.com/r/homeassistant/)

---

*Документ обновлён: май 2026*
