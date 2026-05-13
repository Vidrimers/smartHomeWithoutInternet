# 🏠 Smart Home без интернета на Home Assistant

Полностью локальная система умного дома, работающая независимо от облака. Никаких сервисов Google, Яндекса или других провайдеров — всё работает внутри домашней сети.

## 📋 Что здесь?

### 🏠 Основная система

- **[smart-home-without-internet.md](smart-home-without-internet.md)** — Подробное руководство по сборке умного дома:
  - Выбор и установка сервера (мини-ПК или Raspberry Pi)
  - Настройка Zigbee-координатора для связи с устройствами
  - Выбор умных устройств (лампочки, датчики и т.д.)
  - Локальное голосовое управление через Whisper + Home Assistant
  - Управление ИК-устройствами через Broadlink RM4 Mini
  - Пошаговый порядок настройки

- **[ha-dashboards-guide.md](ha-dashboards-guide.md)** — Создание дашбордов в Home Assistant:
  - Настройка интерфейса
  - Карточки и виджеты
  - Организация устройств

- **[remote-access-ha.md](remote-access-ha.md)** — Удалённый доступ к Home Assistant:
  - Настройка безопасного доступа
  - VPN и другие методы
  - Рекомендации по безопасности

---

### 🎤 Голосовое управление

- **[esp32-s3-box-3b-voice-assistant.md](esp32-s3-box-3b-voice-assistant.md)** — Настройка ESP32-S3-BOX-3B:
  - Голосовой ассистент на базе ESP32
  - Интеграция с Home Assistant
  - Локальное распознавание речи

- **[esp32-s3-box-3b-custom-display.md](esp32-s3-box-3b-custom-display.md)** — Кастомизация дисплея ESP32-S3-BOX-3B:
  - Настройка интерфейса
  - Отображение информации
  - Управление через экран

- **[WakeWord-custom.md](WakeWord-custom.md)** — Создание кастомных wake words:
  - Обучение собственных слов активации
  - Настройка чувствительности
  - Интеграция с голосовым ассистентом

---

### 🖥️ Дисплеи и мониторы

- **[turing-smart-screen-guide.md](turing-smart-screen-guide.md)** — Turing Smart Screen + Home Assistant:
  - Установка на Ubuntu/Linux
  - Интеграция с Home Assistant через IoTuring
  - Создание кастомных тем
  - Отображение данных HA на экране
  - Автозапуск и troubleshooting

---

### 📹 Видеонаблюдение (Frigate)

- **[observer/1. observer-video-system.md](observer/1.%20observer-video-system.md)** — Система видеонаблюдения:
  - Детекция человека через Frigate
  - Интеграция с Home Assistant
  - Удалённый доступ через Xray VPN
  - Рекомендуемое оборудование (камеры, PoE-коммутаторы и т.д.)
  - Аппаратное ускорение AMD GPU (VA-API + OpenVINO)

- **[observer/2. observer-camera-cable-guide.md](observer/2.%20observer-camera-cable-guide.md)** — Практическое руководство по монтажу:
  - Выбор и прокладка уличных кабелей для PoE-камер
  - Правильное сверление отверстий в стене
  - Защита кабелей от влаги и мороза

- **[observer/3. frigate-usb-camera-tutorial.md](observer/3.%20frigate-usb-camera-tutorial.md)** — Подключение USB веб-камеры:
  - Настройка USB камеры в Frigate
  - Выбор разрешения и FPS
  - Интеграция с детекцией объектов

- **[observer/4. frigate-usb-camera-cpu-optimization.md](observer/4.%20frigate-usb-camera-cpu-optimization.md)** — Оптимизация USB камер:
  - Почему USB камеры используют больше CPU
  - Как снизить нагрузку с 21% до 8-10%
  - Оптимальные настройки разрешения и FPS
  - Сравнение вариантов конфигурации

- **[observer/5. frigate-camera-optimization.md](observer/5.%20frigate-camera-optimization.md)** — Общая оптимизация камер:
  - Настройка зон детекции
  - Оптимизация нагрузки на CPU и GPU
  - Тонкая настройка детектора

- **[observer/6. frigate-retain-guide.md](observer/6.%20frigate-retain-guide.md)** — Настройка хранения данных Frigate:
  - Объяснение параметров конфигурации
  - Режимы записи (continuous, motion, detections, alerts)
  - Оптимизация места на диске

- **[observer/7. frigate-automations-guide.md](observer/7.%20frigate-automations-guide.md)** — Автоматизации Frigate + Home Assistant:
  - Настройка MQTT для интеграции
  - Примеры автоматизаций (уведомления, включение света и т.д.)
  - Использование зон детекции
  - Советы по оптимизации

- **[observer/8. FRIGATE-QUICK-REFERENCE.md](observer/8.%20FRIGATE-QUICK-REFERENCE.md)** — 📚 Быстрая справка по Frigate:
  - Основные команды
  - Оптимальные настройки
  - Частые ошибки и их решения
  - Отладка

---

### 🔌 Интеграции устройств

- **[localtuya-ru.md](localtuya-ru.md)** — Интеграция Tuya устройств локально:
  - Настройка LocalTuya
  - Подключение устройств без облака
  - Получение localKey

- **[localKey.md](localKey.md)** — Получение localKey для Tuya устройств:
  - Пошаговая инструкция
  - Необходимые инструменты
  - Troubleshooting

## 🎯 Основные компоненты

### Сервер (мозг системы)

- **Рекомендуется**: мини-ПК на Intel N100 (Beelink Mini S12 Pro, GMKtec G3 и т.д.)
- **Альтернатива**: Raspberry Pi 5
- **ПО**: Home Assistant OS

### Сеть умных устройств

- **Координатор**: Sonoff Zigbee 3.0 USB Dongle Plus
- **Программа**: Zigbee2MQTT или ZHA
- **Устройства**: IKEA TRADFRI, Philips Hue и др.

### Видеонаблюдение

- **Софт**: Frigate (детекция объектов)
- **Камеры**: IP-камеры с поддержкой RTSP или PoE
- **Интеграция**: Home Assistant + уведомления в Telegram

### Голосовое управление

- **Распознавание речи**: Whisper (локально, без облака)
- **Обработка команд**: Home Assistant Assist
- **Управление**: Zigbee и ИК-устройства

## 🚀 Быстрый старт

1. Подготовьте сервер (мини-ПК или Raspberry Pi)
2. Установите Home Assistant OS
3. Подключите Zigbee-координатор
4. Добавляйте умные устройства
5. Настройте голосовое управление

## 💡 Особенности

✅ **100% локальная работа** — никакой зависимости от интернета  
✅ **Приватность** — все данные остаются в вашей сети  
✅ **Низкий расход электроэнергии** — 10-15W для мини-ПК  
✅ **Масштабируемость** — легко добавлять новые устройства  
✅ **Open Source** — Home Assistant, Zigbee2MQTT, Frigate

## 📖 Документация

Каждый файл в репозитории содержит подробные инструкции с примерами, ценами и рекомендациями.

## 💰 Смета расходов

Полная смета всех материалов и оборудования:  
📊 [Google Sheets — Расходы](https://docs.google.com/spreadsheets/d/17kcrknVjOhv5eCSQ_UyaT82vD06NXWZFNAjI5jBwDTI/edit?gid=1015011691#gid=1015011691)

---

**Версия:** 1.0  
**Язык:** Русский  
**Лицензия:** MIT
