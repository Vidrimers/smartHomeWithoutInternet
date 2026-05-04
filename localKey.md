# Как получить Local Key для Tuya устройств

Local Key необходим для локального управления **Wi-Fi устройствами Tuya** через LocalTuya без интернета.

> ⚠️ **Важно:** Эта инструкция только для **Wi-Fi устройств Tuya**. Zigbee устройства подключаются через Zigbee2MQTT и не требуют Local Key.

---

## Предварительные требования

1. Установлен **tinytuya**: `pipx install tinytuya`
2. Создан проект на **Tuya IoT Platform** (iot.tuya.com)
3. Устройства добавлены в приложение **Smart Life** или **HIPER IoT**

---

## Шаг 1: Создать проект на Tuya IoT Platform

### 1.1 Определить правильный датацентр

Твой аккаунт в приложении привязан к определённому датацентру. Нужно создать проект на **том же датацентре**.

**Как узнать датацентр:**
- Зайди на https://myaccount.tuya.com/account
- Посмотри какая страна указана в профиле
- Сопоставь с таблицей:

| Страна/Регион | Датацентр |
|---|---|
| Россия, Европа | Central Europe (`eu`) |
| Китай | China (`cn`) |
| США (запад) | US Western (`us`) |
| США (восток) | US Eastern (`us-e`) |
| Индия | India (`in`) |

### 1.2 Создать проект

1. Зайди на https://iot.tuya.com
2. **Cloud → Development → Create Cloud Project**
3. Заполни:
   - **Project Name:** `Home Assistant`
   - **Industry:** Smart Home
   - **Development Method:** Smart Home
   - **Data Center:** выбери правильный (например, Central Europe)
4. Нажми **Create**

### 1.3 Активировать API сервисы

1. Открой созданный проект
2. Вкладка **Service API**
3. Нажми **Subscribe** (бесплатно) на:
   - **IoT Core**
   - **Authorization Token Management**
   - **Smart Home Basic Service**

### 1.4 Получить credentials

1. Вкладка **Overview**
2. Скопируй:
   - **Access ID/Client ID** (например: `4xh93m95nkffd7qkwnuq`)
   - **Access Secret/Client Secret** (нажми "показать" и скопируй)

### 1.5 Привязать аккаунт приложения

1. Вкладка **Devices → Link Tuya App Account**
2. **Важно:** В правом верхнем углу выбери тот же датацентр что и при создании проекта
3. Отсканируй QR-код через приложение **Smart Life** (или HIPER IoT)
4. Выбери **Automatic Link** — все устройства привяжутся автоматически

**Если QR-код показывает "нет доступа" или "expired":**
- Проверь что датацентр в правом верхнем углу совпадает с регионом аккаунта
- Попробуй другие датацентры (Central Europe, Western Europe, China)

---

## Шаг 2: Получить Local Key через Python скрипт

После успешной привязки аккаунта используй этот скрипт:

```bash
cat > /tmp/get_local_keys.py << 'EOF'
import sys
sys.path.insert(0, '/home/ТВОЙ_ПОЛЬЗОВАТЕЛЬ/.local/share/pipx/venvs/tinytuya/lib/python3.12/site-packages')

import tinytuya

# Credentials из Tuya IoT Platform
cloud = tinytuya.Cloud(
    apiRegion="eu",  # Измени на свой регион: eu, cn, us, us-e, in
    apiKey="ТВОЙ_CLIENT_ID",
    apiSecret="ТВОЙ_CLIENT_SECRET"
)

# Получить список всех устройств с Local Key
devices = cloud.getdevices()

# Вывести информацию
for device in devices:
    print(f"\nУстройство: {device['name']}")
    print(f"  Device ID: {device['id']}")
    print(f"  Local Key: {device['key']}")
    print(f"  IP: (узнай через роутер)")
    print(f"  MAC: {device.get('mac', 'N/A')}")
    print(f"  Модель: {device['product_name']}")
EOF

# Замени ТВОЙ_ПОЛЬЗОВАТЕЛЬ, CLIENT_ID, CLIENT_SECRET и регион
python3 /tmp/get_local_keys.py
```

**Пример вывода:**
```
Устройство: Люстра
  Device ID: bf51b43d6bb0e7316elyjp
  Local Key: p&c_K.`WO';=s!mR
  IP: (узнай через роутер)
  MAC: 38:2c:e5:00:3a:1a
  Модель: WiFi 5001
```

---

## Шаг 3: Узнать IP адрес устройства

Local Key получен, но нужен ещё IP адрес устройства в локальной сети.

### Способ 1: Через роутер

1. Зайди в веб-интерфейс роутера
2. Найди список подключённых устройств
3. Найди устройство по MAC-адресу (из скрипта выше)
4. Скопируй IP адрес

### Способ 2: Через nmap

```bash
sudo nmap -sn 192.168.1.0/24 | grep -B 2 "Tuya\|Espressif"
```

---

## Шаг 4: Добавить устройство в LocalTuya

Теперь у тебя есть всё необходимое:
- ✅ Device ID
- ✅ Local Key
- ✅ IP адрес

### В Home Assistant:

1. **Settings → Devices & Services → LocalTuya → Configure**
2. **Add or Edit a device → "..." (manual entry)**
3. Введи:
   - **Friendly name:** название устройства
   - **Host:** IP адрес (например `192.168.1.78`)
   - **Device ID:** из скрипта
   - **Local Key:** из скрипта (вводи точно, включая спецсимволы!)
   - **Protocol Version:** `3.3` (если не работает — попробуй `3.1`, `3.2`, `3.4`)
   - **Manual DPS:** `1,20,21,22,23,24,25,26` (для RGB ламп)

4. Нажми **Submit**

5. Выбери тип сущности:
   - **light** — для лампочек и люстр
   - **switch** — для розеток и выключателей
   - **cover** — для штор и жалюзи

6. Настрой DP (data points) — LocalTuya покажет доступные DP с текущими значениями

---

## Типичные проблемы

### Ошибка "permission deny" (код 1106)

**Причина:** Аккаунт не привязан к проекту или неправильный датацентр.

**Решение:**
1. Проверь что QR-код отсканирован успешно
2. Убедись что датацентр в правом верхнем углу совпадает с регионом аккаунта
3. Попробуй другие датацентры

### Ошибка "sign invalid" (код 1004)

**Причина:** Неправильные Client ID или Secret.

**Решение:**
1. Проверь что скопировал правильные credentials из вкладки Overview
2. Убедись что нет лишних пробелов в начале/конце

### QR-код показывает "expired" сразу после сканирования

**Причина:** Несовпадение датацентров.

**Решение:**
1. В правом верхнем углу страницы "Link Tuya App Account" выбери другой датацентр
2. Попробуй все доступные: Central Europe, Western Europe, China, US

### Устройство не подключается в LocalTuya

**Причины:**
1. Неправильный Local Key (проверь спецсимволы)
2. Неправильный Protocol Version (попробуй 3.1, 3.2, 3.3, 3.4)
3. Устройство работает только через облако (редко)

---

## Полезные ссылки

- [Tuya IoT Platform](https://iot.tuya.com)
- [LocalTuya GitHub](https://github.com/rospogrigio/localtuya)
- [LocalTuya документация (русский)](./localtuya-ru.md)
- [tinytuya документация](https://github.com/jasonacox/tinytuya)

---

## Примечания

- Local Key **меняется** при перепривязке устройства к другому аккаунту
- После получения Local Key устройство работает **полностью локально** без интернета
- Если заблокируешь устройству доступ в интернет — блокируй также DNS (иначе зависнет в "зомби-режиме")
