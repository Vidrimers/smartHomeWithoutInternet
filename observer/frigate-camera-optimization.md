# ⚙️ Оптимизация производительности

## 1.1. Выбор разрешения для детекции

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

## 1.2. Аппаратное ускорение AMD GPU (VA-API + OpenVINO)

Если у тебя мини-ПК с AMD CPU (например, Ryzen 4300U / 5500U и подобные), встроенный GPU Radeon Vega можно использовать для декодирования видеопотоков. Это снижает нагрузку на CPU и освобождает ресурсы для других задач (HA, Whisper, Piper).

### Что это даёт

| | Без ускорения | С VA-API + OpenVINO |
|---|---|---|
| Декодирование видео (2 IP-камеры) | ~15% CPU | ~2% CPU (GPU) |
| Детекция объектов | ~20% CPU | ~12% CPU |
| Итого | ~35%+ | ~15% |

- **VA-API** — аппаратное декодирование H.264 потоков с IP-камер через GPU
- **OpenVINO** — оптимизированный детектор объектов от Intel, работает быстрее стандартного `cpu` детектора на x86 процессорах (в том числе AMD)
- USB-камеры аппаратное ускорение **не поддерживают** — для них всё остаётся как есть (`hwaccel_args: []`)

> ⚠️ Этот раздел актуален для **AMD GPU (Radeon Vega/GCN)**. Для Intel GPU используется другой драйвер (`iHD` вместо `radeonsi`).

---

### Шаг 1: Изменить docker-compose.yml

```bash
nano ~/frigate/docker-compose.yml
```

Добавить секцию `devices` и переменную окружения `LIBVA_DRIVER_NAME`:

```yaml
services:
  frigate:
    container_name: frigate
    image: ghcr.io/blakeblackshear/frigate:stable
    privileged: true
    restart: unless-stopped
    shm_size: "512mb"
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128   # AMD GPU render node
    volumes:
      - ./config:/config
      - ./media:/media/frigate
      - /etc/localtime:/etc/localtime:ro
      - /dev/bus/usb:/dev/bus/usb
      - /dev/video0:/dev/video0  # USB камера (если есть)
    ports:
      - "8971:8971"
      - "5000:5000"
      - "8554:8554"
      - "8555:8555/tcp"
      - "8555:8555/udp"
    environment:
      FRIGATE_RTSP_PASSWORD: "твой_пароль"
      LIBVA_DRIVER_NAME: "radeonsi"               # Драйвер Mesa для AMD
```

> **Почему `renderD128`, а не `card0`:** Для VA-API в Docker нужен именно render-узел, а не основное устройство карты. Это частая ошибка при настройке.

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`.

---

### Шаг 2: Изменить config.yml

```bash
nano ~/frigate/config/config.yml
```

#### 2.1. Добавить глобальный размер модели

В самое начало `config.yml` (перед `mqtt:`) добавить:

```yaml
model:
  width: 300   # Размер входного изображения для ssdlite_mobilenet_v2
  height: 300
```

> **Важно:** ssdlite_mobilenet_v2 ожидает 300x300. Без этого Frigate подаёт 320x320 и детектор падает с ошибкой `ValueError: could not broadcast input array`.

#### 2.2. Заменить детектор

Вместо стандартного `cpu` использовать OpenVINO:

```yaml
# Было:
detectors:
  cpu1:
    type: cpu

# Стало:
detectors:
  ov:
    type: openvino
    device: CPU  # OpenVINO быстрее стандартного cpu детектора на x86/AMD
    model_path: /openvino-model/ssdlite_mobilenet_v2.xml  # встроенная модель в контейнере
    model:
      model_type: ssd
```

> **Почему `model_path` на уровне детектора, а не `model.path`:** В Frigate 0.17 путь к модели детектора указывается через `model_path` в блоке детектора. Поле `model.path` зарезервировано для Frigate+ моделей и игнорируется для встроенных детекторов.

#### 2.3. Добавить hwaccel_args для IP-камер

Для каждой IP-камеры добавить строку `hwaccel_args: preset-vaapi` в секцию `ffmpeg:`:

```yaml
cameras:
  camera_outside_1:
    ffmpeg:
      hwaccel_args: preset-vaapi   # ← добавить эту строку
      inputs:
        - path: rtsp://...
          roles:
            - detect
        - path: rtsp://...
          roles:
            - record
    # ... остальные настройки без изменений

  camera_outside_2:
    ffmpeg:
      hwaccel_args: preset-vaapi   # ← добавить эту строку
      inputs:
        # ...
```

> **USB-камера:** для неё `hwaccel_args: []` — оставить без изменений, GPU для MJPEG не используется.

`preset-vaapi` — это встроенный пресет Frigate, который автоматически подставляет все нужные аргументы ffmpeg для VA-API.

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`.

---

### Шаг 3: Перезапустить Frigate

```bash
cd ~/frigate
docker compose down
docker compose up -d
```

---

### Шаг 4: Проверить что всё работает

#### Проверка логов

```bash
docker logs frigate 2>&1 | grep -i vaapi
```

Не должно быть строк с `failed` или `error`. Нормальный вывод выглядит примерно так:

```
[info] vaapi: using /dev/dri/renderD128
```

#### Проверка внутри контейнера

```bash
# Зайти внутрь контейнера
docker exec -it frigate /bin/bash

# Проверить что устройство проброшено
ls /dev/dri/
# Должен быть renderD128

# Проверить что VA-API видит GPU
vainfo
```

Если `vainfo` вывел список профилей (`VAProfileH264...`, `VAProfileHEVC...`) — ускорение работает.

#### Проверка нагрузки

Открой Frigate UI → **System → Stats** и сравни нагрузку CPU до и после.

---

### Troubleshooting VA-API

**`vainfo` выдаёт ошибку `libva: /dev/dri/renderD128: permission denied`**

```bash
# Добавь пользователя в группу render и video
sudo usermod -aG render,video $USER
# Перезагрузись
sudo reboot
```

**В логах Frigate: `Failed to initialize VAAPI`**

Проверь что устройство существует на хосте:
```bash
ls -la /dev/dri/
```

Если `renderD128` нет — возможно нужно установить драйверы Mesa:
```bash
sudo apt install mesa-va-drivers libva-drm2
```

**OpenVINO детектор не запускается**

Проверь логи:
```bash
docker logs frigate 2>&1 | grep -i openvino
```

Если ошибка — можно вернуться к стандартному детектору:
```yaml
detectors:
  cpu1:
    type: cpu
```