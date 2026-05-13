# 📚 Frigate — Быстрая справка

Краткая шпаргалка по настройке и оптимизации Frigate.

---

## 🎯 Основные файлы

| Файл | Описание |
|------|----------|
| `observer-video-system.md` | Полное руководство по установке и настройке |
| `usb-camera-cpu-optimization.md` | Оптимизация USB камер (снижение CPU) |
| `frigate-retain-guide.md` | Настройка хранения данных |
| `frigate-automations-guide.md` | Автоматизации с Home Assistant |

---

## 🔧 Быстрые команды

### Управление Frigate

```bash
# Перезапустить
cd ~/frigate
docker compose restart

# Остановить
docker compose stop

# Запустить
docker compose up -d

# Посмотреть логи
docker compose logs -f frigate

# Посмотреть логи конкретной камеры
docker compose logs -f frigate | grep camera_name
```

---

### Проверка USB камеры

```bash
# Список устройств
ls -l /dev/video*

# Поддерживаемые форматы
v4l2-ctl --list-formats-ext -d /dev/video0

# Что использует камеру
sudo lsof /dev/video0

# Тест захвата кадра
ffmpeg -i /dev/video0 -frames:v 1 test.jpg
```

---

### Проверка RTSP потока (IP камеры)

```bash
# Тест захвата кадра
ffmpeg -i rtsp://user:pass@IP:554/Streaming/Channels/102 -frames:v 1 test.jpg

# Проверка разрешения потока
ffmpeg -i rtsp://user:pass@IP:554/Streaming/Channels/102 2>&1 | grep Video
```

---

## 📊 Оптимальные настройки

### IP камера (RTSP)

```yaml
camera_name:
  ffmpeg:
    inputs:
      # Sub Stream для детекции (экономия CPU)
      - path: rtsp://user:pass@IP:554/Streaming/Channels/102
        roles:
          - detect
      # Main Stream для записи (высокое качество)
      - path: rtsp://user:pass@IP:554/Streaming/Channels/101
        roles:
          - record
  detect:
    width: 640    # Разрешение Sub Stream
    height: 360
    fps: 10
  record:
    enabled: true
    continuous:
      days: 0
    motion:
      days: 0     # Не писать при любом движении
    detections:
      retain:
        days: 3   # Видео с объектами — 3 дня
    alerts:
      retain:
        days: 7   # Видео с объектами в зоне — 7 дней
```

---

### USB камера (экономия CPU)

```yaml
camera_name:
  ffmpeg:
    hwaccel_args: []
    inputs:
      - path: /dev/video0
        input_args: -f v4l2 -input_format mjpeg -video_size 640x480 -framerate 20
        roles:
          - detect
          - record
  detect:
    width: 640    # Должно совпадать с video_size!
    height: 480
    fps: 10
  record:
    enabled: true
    continuous:
      days: 0
    motion:
      days: 3
```

**Результат:** CPU ~8-10% (вместо 21%)

---

## ⚠️ Частые ошибки

### 1. Разрешение не совпадает

❌ **Неправильно:**
```yaml
input_args: ... -video_size 1280x720 ...
detect:
  width: 640
  height: 480
```

✅ **Правильно:**
```yaml
input_args: ... -video_size 640x480 ...
detect:
  width: 640
  height: 480
```

---

### 2. Ошибка 401 Unauthorized (RTSP)

**Причины:**
- Неправильный логин/пароль
- Специальные символы в пароле не закодированы

**Решение:**
- Создай отдельного пользователя для RTSP в камере
- Используй простой пароль без спецсимволов
- Или закодируй спецсимволы: `!` → `%21`, `@` → `%40`, `#` → `%23`

---

### 3. Device busy (USB камера)

**Причина:** Камера уже используется другой программой.

**Решение:**
```bash
# Найди процесс
sudo lsof /dev/video0

# Останови процесс
sudo kill <PID>

# Перезапусти Frigate
cd ~/frigate
docker compose restart
```

---

### 4. Порт 8971 не работает

**Причина:** Порт 8971 работает только по HTTPS.

**Решение:** Используй `https://` вместо `http://`:
```
https://IP:8971/
```

---

## 📊 Сравнение режимов записи

| Режим | Триггер | Место на диске (3 камеры, сутки) |
|-------|---------|----------------------------------|
| `continuous` | Всегда | ~50 GB |
| `motion` | Любое движение | ~15-30 GB |
| `detections` | Объект обнаружен | ~1-3 GB |
| `alerts` | Объект в зоне | ~0.1-0.5 GB |

**Рекомендация:** Используй `detections` или `alerts` для экономии места.

---

## 🔍 Отладка

### Проверка статуса Frigate

```bash
# Статус контейнера
docker ps | grep frigate

# Использование ресурсов
docker stats frigate

# Логи последних 50 строк
docker compose logs frigate | tail -50
```

---

### Проверка в Frigate UI

1. Открой `http://IP:5000` или `https://IP:8971`
2. System → Stats → Camera Processes
3. Смотри нагрузку CPU для каждой камеры

---

### Проверка в Home Assistant

1. Developer Tools → States
2. Найди `binary_sensor.camera_name_person_occupancy`
3. Проверь состояние (on/off)

---

## 💡 Полезные ссылки

- **Официальная документация:** https://docs.frigate.video
- **GitHub:** https://github.com/blakeblackshear/frigate
- **Форум:** https://github.com/blakeblackshear/frigate/discussions

---

## 📞 Поддержка

Если что-то не работает:

1. Проверь логи: `docker compose logs frigate`
2. Проверь конфигурацию: `nano ~/frigate/config/config.yml`
3. Перезапусти: `docker compose restart`
4. Смотри раздел "Troubleshooting" в `observer-video-system.md`
