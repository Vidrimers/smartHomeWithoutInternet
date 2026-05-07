# Подключение веб-камеры к Home Assistant через mjpg-streamer

> **Окружение:** Ubuntu/Debian, Home Assistant (любой тип установки), USB веб-камера  
> **IP сервера с камерой:** `10.0.0.3`  
> **Порт стрима:** `8090`

---

## Содержание

1. [Проверка камеры в системе](#1-проверка-камеры-в-системе)
2. [Установка v4l-utils](#2-установка-v4l-utils)
3. [Вариант A — установка через snap](#3-вариант-a--установка-через-snap)
4. [Вариант B — сборка из исходников (рекомендуется)](#4-вариант-b--сборка-из-исходников-рекомендуется)
5. [Запуск стрима](#5-запуск-стрима)
6. [Проверка стрима в браузере](#6-проверка-стрима-в-браузере)
7. [Настройка firewall](#7-настройка-firewall)
8. [Автозапуск через systemd](#8-автозапуск-через-systemd)
9. [Добавление камеры в Home Assistant](#9-добавление-камеры-в-home-assistant)
10. [Добавление карточки на дашборд](#10-добавление-карточки-на-дашборд)
11. [Смена порта](#11-смена-порта)
12. [Удаление mjpg-streamer](#12-удаление-mjpg-streamer)
13. [Устранение неполадок](#13-устранение-неполадок)

---

## 1. Проверка камеры в системе

Подключи USB-камеру и убедись, что система её видит:

```bash
lsusb
```

Камера должна появиться в списке USB-устройств. Пример вывода:
```
Bus 001 Device 003: ID 046d:0825 Logitech, Inc. Webcam C270
```

Проверь, что создалось видеоустройство:

```bash
ls /dev/video*
```

Ожидаемый вывод:
```
/dev/video0
```

> Если устройств несколько (`video0`, `video1`, ...), камера обычно на `video0`.  
> Уточнить можно после установки v4l-utils (следующий шаг).

---

## 2. Установка v4l-utils

`v4l-utils` нужен **в обоих вариантах** — snap и сборка из исходников.  
Он позволяет диагностировать камеру и проверить поддерживаемые форматы.

```bash
sudo apt update
sudo apt install v4l-utils -y
```

### Проверка установки

Список всех видеоустройств:

```bash
v4l2-ctl --list-devices
```

Пример вывода:
```
UVC Camera (046d:0825) (usb-0000:00:14.0-2):
    /dev/video0
    /dev/video1
```

Проверка поддерживаемых форматов камеры:

```bash
v4l2-ctl --device=/dev/video0 --list-formats-ext
```

Пример вывода:
```
ioctl: VIDIOC_ENUM_FMT
    Type: Video Capture

    [0]: 'MJPG' (Motion-JPEG, compressed)
        Size: Discrete 1280x720
            Interval: Discrete 0.033s (30.000 fps)
        Size: Discrete 640x480
            Interval: Discrete 0.033s (30.000 fps)
    [1]: 'YUYV' (YUYV 4:2:2)
        Size: Discrete 640x480
```

> Если в списке есть `MJPG` — отлично, mjpg-streamer будет работать с меньшей нагрузкой на CPU.  
> Если только `YUYV` — придётся использовать флаг `-y` при запуске (конвертация на лету).

---

## 3. Вариант A — установка через snap

> **Минусы snap-версии:**
> - Нет встроенного веб-интерфейса (`http://10.0.0.3:8090/` даёт `404: Not Found`)
> - Работают только прямые URL потока и снапшота
> - Изолированная среда требует дополнительной настройки прав

```bash
sudo snap install mjpg-streamer
```

### Предоставление доступа к камере

```bash
sudo snap connect mjpg-streamer:camera
```

### Проверка установки

```bash
snap list mjpg-streamer
```

Пример вывода:
```
Name          Version  Rev  Tracking       Publisher  Notes
mjpg-streamer 1.0.0    10   latest/stable  ogra       -
```

### Запуск (snap)

```bash
mjpg-streamer \
  -i "input_uvc.so -d /dev/video0 -r 1280x720 -f 15" \
  -o "output_http.so -p 8090"
```

> Флаг `-w` (папка с веб-файлами) в snap-версии не работает — не указывай его.

---

## 4. Вариант B — сборка из исходников (рекомендуется)

Это полноценная версия с веб-интерфейсом. Репозиторий
[jacksonliam/mjpg-streamer](https://github.com/jacksonliam/mjpg-streamer)
актуален — последняя активность в январе 2025 года.

### Установка зависимостей

```bash
sudo apt update
sudo apt install cmake libjpeg8-dev gcc g++ git -y
```

### Клонирование и сборка

```bash
cd ~
git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
make
sudo make install
```

Пример успешного вывода `make`:
```
[ 25%] Building C object ...
[ 50%] Building C object ...
[100%] Linking C shared library input_uvc.so
[100%] Built target input_uvc
```

### Проверка установки

```bash
mjpg_streamer --version
```

Пример вывода:
```
MJPG Streamer Version: git rev: 310b29f
```

### Где находятся файлы после установки

| Файл | Путь |
|---|---|
| Бинарник | `/usr/local/bin/mjpg_streamer` |
| Плагины | `/usr/local/lib/mjpg-streamer/` |
| Веб-интерфейс | `/usr/local/share/mjpg-streamer/www/` |

### Запуск (исходники)

```bash
mjpg_streamer \
  -i "input_uvc.so -d /dev/video0 -r 1280x720 -f 15" \
  -o "output_http.so -p 8090 -w /usr/local/share/mjpg-streamer/www"
```

> Обрати внимание: бинарник называется `mjpg_streamer` (с подчёркиванием), а не `mjpg-streamer`.

---

## 5. Запуск стрима

### Параметры input_uvc.so

| Параметр | Описание | Пример |
|---|---|---|
| `-d` | Устройство | `-d /dev/video0` |
| `-r` | Разрешение | `-r 1280x720` или `-r 640x480` |
| `-f` | FPS | `-f 15` |
| `-q` | Качество JPEG (1–100) | `-q 80` |
| `-y` | Использовать YUYV вместо MJPG | если камера не поддерживает MJPG |

### Если камера не поддерживает MJPG

Добавь флаг `-y`:

```bash
# Snap-версия
mjpg-streamer \
  -i "input_uvc.so -d /dev/video0 -r 640x480 -f 10 -y" \
  -o "output_http.so -p 8090"

# Сборка из исходников
mjpg_streamer \
  -i "input_uvc.so -d /dev/video0 -r 640x480 -f 10 -y" \
  -o "output_http.so -p 8090 -w /usr/local/share/mjpg-streamer/www"
```

> При использовании `-y` рекомендуется снизить FPS до 10–5, так как CPU нагружается сильнее.

---

## 6. Проверка стрима в браузере

| URL | Snap (A) | Исходники (B) |
|---|---|---|
| `http://10.0.0.3:8090/` (веб-интерфейс) | ❌ 404 | ✅ Работает |
| `http://10.0.0.3:8090/?action=stream` | ✅ | ✅ |
| `http://10.0.0.3:8090/?action=snapshot` | ✅ | ✅ |

**Рабочие URL для обоих вариантов:**

```
http://10.0.0.3:8090/?action=stream    ← живой MJPEG поток
http://10.0.0.3:8090/?action=snapshot  ← один кадр (JPEG)
```

> Если снапшот открывается в браузере — стрим работает, можно идти дальше.

---

## 7. Настройка firewall

Если используешь `ufw`, открой порт 8090:

```bash
sudo ufw allow 8090/tcp
sudo ufw reload
```

Проверка:

```bash
sudo ufw status
```

Ожидаемый вывод (среди правил):
```
8090/tcp                   ALLOW       Anywhere
```

---

## 8. Автозапуск через systemd

### Создание файла сервиса

```bash
sudo nano /etc/systemd/system/mjpg-streamer.service
```

**Для Варианта A (snap):**

```ini
[Unit]
Description=MJPG Streamer - Webcam stream for Home Assistant
After=network.target

[Service]
Type=simple
User=root
ExecStart=/snap/bin/mjpg-streamer \
  -i "input_uvc.so -d /dev/video0 -r 1280x720 -f 15" \
  -o "output_http.so -p 8090"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**Для Варианта B (исходники):**

```ini
[Unit]
Description=MJPG Streamer - Webcam stream for Home Assistant
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/mjpg_streamer \
  -i "input_uvc.so -d /dev/video0 -r 1280x720 -f 15" \
  -o "output_http.so -p 8090 -w /usr/local/share/mjpg-streamer/www"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Активация сервиса

```bash
sudo systemctl daemon-reload
sudo systemctl enable mjpg-streamer
sudo systemctl start mjpg-streamer
```

### Проверка статуса

```bash
sudo systemctl status mjpg-streamer
```

Ожидаемый вывод:
```
● mjpg-streamer.service - MJPG Streamer - Webcam stream for Home Assistant
     Loaded: loaded (/etc/systemd/system/mjpg-streamer.service; enabled)
     Active: active (running) since ...
```

Просмотр логов в реальном времени:

```bash
sudo journalctl -u mjpg-streamer -f
```

---

## 9. Добавление камеры в Home Assistant

Нужно отредактировать файл `configuration.yaml`. Есть три способа — выбери удобный.

---

### Способ 1 — File Editor (через веб-интерфейс HA)

Самый простой способ, не нужен терминал.

1. **Settings → Add-ons → Add-on Store**
2. Найди **File Editor** и установи
3. Запусти File Editor и открой `configuration.yaml`
4. Добавь в конец файла:

```yaml
camera:
  - platform: mjpeg
    name: "Веб-камера"
    still_image_url: "http://10.0.0.3:8090/?action=snapshot"
    mjpeg_url: "http://10.0.0.3:8090/?action=stream"
```

5. Сохрани файл

> Если аддоны недоступны — ты используешь HA Container (Docker). Переходи к Способу 2 или 3.

---

### Способ 2 — через терминал на сервере (Docker)

Найди папку с конфигом HA — ту, что примонтирована в контейнер:

```bash
docker inspect homeassistant | grep -A5 Mounts
```

Пример вывода:
```json
"Mounts": [
    {
        "Source": "/home/user/homeassistant",
        "Destination": "/config",
    }
]
```

Значит конфиг лежит по пути `/home/user/homeassistant/configuration.yaml`. Открой его:

```bash
nano /home/user/homeassistant/configuration.yaml
```

Добавь в конец файла:

```yaml
camera:
  - platform: mjpeg
    name: "Веб-камера"
    still_image_url: "http://10.0.0.3:8090/?action=snapshot"
    mjpeg_url: "http://10.0.0.3:8090/?action=stream"
```

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`.

---

### Способ 3 — внутри контейнера Docker

Если не знаешь где лежит папка с конфигом — можно зайти прямо внутрь контейнера:

```bash
docker exec -it homeassistant bash
nano /config/configuration.yaml
```

Добавь в конец файла:

```yaml
camera:
  - platform: mjpeg
    name: "Веб-камера"
    still_image_url: "http://10.0.0.3:8090/?action=snapshot"
    mjpeg_url: "http://10.0.0.3:8090/?action=stream"
```

Сохрани: `Ctrl+O`, `Enter`, `Ctrl+X`. Выйди из контейнера:

```bash
exit
```

> Не используй `localhost` или `127.0.0.1` — HA в Docker не найдёт стрим по этому адресу, всегда используй реальный IP `10.0.0.3`.

---

### Перезапуск Home Assistant

Через интерфейс: **Settings → System → Restart**

Или через терминал:

```bash
docker restart homeassistant
```

### Проверка в HA

Перейди в **Settings → Devices & Services → Entities** и найди `camera.veb_kamera`.

Или проверь через **Developer Tools → States** — в списке должна появиться сущность с префиксом `camera.`.

---

## 10. Добавление карточки на дашборд

1. Открой нужный дашборд
2. Нажми **Edit → Add Card**
3. Выбери **Picture Entity** или **Picture Glance**
4. В поле Entity выбери `camera.veb_kamera`
5. Сохрани

Или добавь вручную через YAML редактор карточки:

```yaml
type: picture-glance
title: Камера
entities: []
camera_image: camera.veb_kamera
```

---

## 11. Смена порта

Если нужно изменить порт (например с `8090` на `8080`), поменяй его в трёх местах.

### Шаг 1 — обнови файл сервиса

```bash
sudo nano /etc/systemd/system/mjpg-streamer.service
```

Найди строку с `-p 8090` и замени на нужный порт:

```ini
# Было
-o "output_http.so -p 8090"

# Стало
-o "output_http.so -p 8080"
```

Перезапусти сервис:

```bash
sudo systemctl daemon-reload
sudo systemctl restart mjpg-streamer
```

### Шаг 2 — обнови firewall

```bash
sudo ufw allow 8080/tcp
sudo ufw delete allow 8090/tcp  # закрой старый порт, если не нужен
sudo ufw reload
```

### Шаг 3 — обнови configuration.yaml в Home Assistant

```yaml
camera:
  - platform: mjpeg
    name: "Веб-камера"
    still_image_url: "http://10.0.0.3:8080/?action=snapshot"
    mjpeg_url: "http://10.0.0.3:8080/?action=stream"
```

Перезапусти HA: **Settings → System → Restart**

---

## 12. Удаление mjpg-streamer

### Шаг 1 — остановить сервис (для обоих вариантов)

```bash
sudo systemctl stop mjpg-streamer
sudo systemctl disable mjpg-streamer
sudo rm /etc/systemd/system/mjpg-streamer.service
sudo systemctl daemon-reload
```

---

### Удаление Варианта A (snap)

```bash
sudo snap remove mjpg-streamer
```

Проверка:

```bash
snap list | grep mjpg
# Вывод должен быть пустым
```

---

### Удаление Варианта B (сборка из исходников)

```bash
# Перейди в папку сборки
cd ~/mjpg-streamer/mjpg-streamer-experimental

# Удали установленные файлы
sudo make uninstall
```

Если `make uninstall` не сработал — удали вручную:

```bash
sudo rm /usr/local/bin/mjpg_streamer
sudo rm -rf /usr/local/lib/mjpg-streamer/
sudo rm -rf /usr/local/share/mjpg-streamer/
```

Удали папку с исходниками:

```bash
rm -rf ~/mjpg-streamer
```

Проверка:

```bash
which mjpg_streamer
# Вывод должен быть пустым
```

---

### Удаление v4l-utils (опционально)

```bash
sudo apt remove v4l-utils -y
sudo apt autoremove -y
```

---

## 13. Устранение неполадок

### Камера не найдена (`/dev/video0` нет)

```bash
# Проверь подключение
lsusb

# Перезагрузи модуль
sudo modprobe uvcvideo

# Проверь логи ядра
dmesg | grep -i video | tail -20
```

### mjpg-streamer не запускается

```bash
# Проверь, не занят ли порт
sudo lsof -i :8090

# Проверь доступ к устройству
ls -la /dev/video0

# Добавь пользователя в группу video
sudo usermod -aG video $USER
newgrp video
```

### HA не видит стрим (ошибка соединения)

```bash
# С машины где запущен HA проверь доступность
curl -I http://10.0.0.3:8090/?action=snapshot

# Проверь firewall
sudo ufw status
sudo ufw allow 8090/tcp
```

### Чёрный экран или зависший кадр

```bash
# Перезапусти сервис
sudo systemctl restart mjpg-streamer

# Попробуй меньшее разрешение
sudo nano /etc/systemd/system/mjpg-streamer.service
# Измени -r 1280x720 на -r 640x480
sudo systemctl daemon-reload
sudo systemctl restart mjpg-streamer
```

### Высокая нагрузка на CPU

- Уменьши FPS: `-f 10` или `-f 5`
- Уменьши разрешение: `-r 640x480`
- Убедись, что камера поддерживает MJPG (без флага `-y`)

### Ошибка при сборке из исходников

```bash
# Если ошибка "libjpeg not found"
sudo apt install libjpeg8-dev libjpeg-dev -y

# Пересобери
cd ~/mjpg-streamer/mjpg-streamer-experimental
make distclean
make
sudo make install
```

---

## Итоговая схема

```
USB-камера
    ↓
/dev/video0
    ↓
mjpg-streamer / mjpg_streamer (порт 8090)
    ↓
http://10.0.0.3:8090/?action=stream    (поток)
http://10.0.0.3:8090/?action=snapshot  (снапшот)
    ↓
Home Assistant → configuration.yaml (camera: mjpeg)
    ↓
Карточка на дашборде
```

---

## Сравнение вариантов

| | Snap (Вариант A) | Из исходников (Вариант B) |
|---|---|---|
| Установка | Одна команда | Нужна сборка (~5 мин) |
| Веб-интерфейс | ❌ 404 | ✅ |
| Поток / снапшот | ✅ | ✅ |
| Обновление | Автоматически | Вручную (`git pull` + `make`) |
| Удаление | `snap remove` | `make uninstall` |
| Изоляция | Да (snap sandbox) | Нет |

---

*Туториал проверен на Ubuntu 22.04 + Home Assistant 2024.x*  
*Репозиторий jacksonliam/mjpg-streamer актуален — последняя активность в январе 2025 г.*
