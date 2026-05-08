# Создание кастомного Wake Word для ESPHome и Home Assistant

## Содержание
1. [Введение](#введение)
2. [Что такое Wake Word](#что-такое-wake-word)
3. [Доступные методы обучения](#доступные-методы-обучения)
4. [Метод 1: microWakeWord (для ESP32-S3)](#метод-1-microwakeword-для-esp32-s3)
5. [Метод 2: OpenWakeWord (для Home Assistant)](#метод-2-openwakeword-для-home-assistant)
6. [Интеграция с ESPHome](#интеграция-с-esphome)
7. [Советы по улучшению точности](#советы-по-улучшению-точности)
8. [Решение проблем](#решение-проблем)

---

## Введение

Wake Word (слово пробуждения) — это специальная фраза, которая активирует голосового ассистента. Вместо стандартных "Alexa", "OK Google" или "Hey Siri" вы можете создать свое собственное слово активации, например "Привет Джарвис", "Окей Дом" или любое другое.

Этот туториал покажет, как создать собственную модель wake word для использования с ESPHome и Home Assistant.

---

## Что такое Wake Word

**Wake Word Detection** — технология распознавания ключевого слова, которая:
- Постоянно прослушивает аудиопоток в фоновом режиме
- Использует минимум ресурсов устройства
- Активирует голосового ассистента при обнаружении заданной фразы
- Работает локально без отправки данных в облако

### Преимущества кастомного Wake Word:
- ✅ Полная конфиденциальность (работает локально)
- ✅ Персонализация (любое слово на вашем языке)
- ✅ Низкая задержка
- ✅ Работа без интернета
- ✅ Бесплатно и open-source

---

## Доступные методы обучения

### 1. **microWakeWord** 
- **Для:** ESP32-S3 (on-device detection)
- **Сложность:** Высокая (требует опыта с Python и ML)
- **Размер модели:** ~200-400 KB (TFLite)
- **Точность:** Высокая при правильной настройке
- **Ссылка:** [github.com/OHF-Voice/micro-wake-word](https://github.com/OHF-Voice/micro-wake-word)

### 2. **OpenWakeWord**
- **Для:** Home Assistant (серверная обработка)
- **Сложность:** Средняя
- **Размер модели:** ~200 KB (ONNX/TFLite)
- **Точность:** Очень высокая
- **Ссылка:** [github.com/dscripka/openWakeWord](https://github.com/dscripka/openWakeWord)

---

## Метод 1: microWakeWord (для ESP32-S3)

### Требования

**Аппаратные:**
- ESP32-S3 с минимум 8MB Flash (например, ESP32-S3-BOX-3)
- Микрофон
- Компьютер с GPU (рекомендуется NVIDIA для обучения)

**Программные:**
- Python 3.10
- TensorFlow 2.x
- Git
- Google Colab (альтернатива локальному обучению)

### Шаг 1: Подготовка окружения

#### Вариант A: Windows с NVIDIA GPU (РЕКОМЕНДУЕТСЯ для RTX 4070)

**Преимущества:**
- ✅ Быстрое обучение на вашей RTX 4070 (намного быстрее, чем Colab)
- ✅ Полный контроль над процессом
- ✅ Неограниченное время обучения
- ✅ Можно обучать несколько моделей подряд

**Требования:**
- Windows 10/11
- NVIDIA GPU (у вас RTX 4070 — отлично!)
- Python 3.10
- CUDA Toolkit
- Git

**Установка:**

1. **Установите Python 3.10:**
   - Скачайте с [python.org](https://www.python.org/downloads/)
   - ⚠️ Важно: Отметьте "Add Python to PATH" при установке

2. **Установите CUDA Toolkit 11.8:**
   ```powershell
   # Скачайте и установите CUDA 11.8 с официального сайта NVIDIA
   # https://developer.nvidia.com/cuda-11-8-0-download-archive
   # Выберите: Windows -> x86_64 -> 11 -> exe (local)
   ```

3. **Установите cuDNN:**
   ```powershell
   # Скачайте cuDNN 8.6 для CUDA 11.x
   # https://developer.nvidia.com/cudnn
   # Распакуйте и скопируйте файлы в папку CUDA
   ```

4. **Установите Git:**
   - Скачайте с [git-scm.com](https://git-scm.com/download/win)

5. **Клонируйте репозиторий:**
   ```powershell
   # Откройте PowerShell или Command Prompt
   cd C:\Users\ВашеИмя\Documents
   git clone https://github.com/kahrendt/microWakeWord.git
   cd microWakeWord
   ```

6. **Создайте виртуальное окружение:**
   ```powershell
   # Создаем venv
   python -m venv venv
   
   # Активируем (PowerShell)
   .\venv\Scripts\Activate.ps1
   
   # Или для CMD
   # venv\Scripts\activate.bat
   ```

7. **Установите зависимости:**
   ```powershell
   # Обновляем pip
   python -m pip install --upgrade pip
   
   # Устанавливаем TensorFlow с GPU поддержкой
   pip install tensorflow[and-cuda]==2.15.0
   
   # Устанавливаем остальные зависимости
   pip install -r requirements.txt
   
   # Дополнительные пакеты
   pip install jupyter notebook ipython
   ```

8. **Проверьте GPU:**
   ```powershell
   python -c "import tensorflow as tf; print('GPU доступен:', tf.config.list_physical_devices('GPU'))"
   ```
   
   Должно вывести что-то вроде:
   ```
   GPU доступен: [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
   ```

**Ожидаемая скорость обучения на RTX 4070:**
- Обучение модели: ~30-60 минут (вместо 2-4 часов на Colab)
- Генерация данных: ~10-15 минут
- Общее время: ~1-2 часа

#### Вариант B: WSL2 (Windows Subsystem for Linux)

Если хотите использовать Linux-окружение на Windows:

```powershell
# В PowerShell от администратора
wsl --install Ubuntu-22.04
wsl --update

# Перезагрузите компьютер

# Откройте Ubuntu из меню Пуск
# Внутри WSL:
sudo apt update
sudo apt install python3.10 python3.10-venv python3-pip git -y

# Клонируем репозиторий
git clone https://github.com/kahrendt/microWakeWord.git
cd microWakeWord

# Создаем виртуальное окружение
python3.10 -m venv venv
source venv/bin/activate

# Устанавливаем зависимости
pip install -r requirements.txt
```

**Для использования GPU в WSL2:**
```bash
# Установите NVIDIA CUDA для WSL2
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-11-8
```

#### Вариант C: Google Colab (если не хотите настраивать локально)

1. Откройте [Google Colab](https://colab.research.google.com/)
2. Создайте новый notebook
3. Включите GPU: `Runtime` → `Change runtime type` → `GPU`

**Ограничения Colab:**
- ⏱️ Ограничение времени сессии (~12 часов)
- 🐌 Медленнее, чем RTX 4070
- 📊 Ограниченная RAM и диск

### Шаг 2: Генерация обучающих данных

microWakeWord использует **синтетическую генерацию данных** через Piper TTS, что означает, что вам не нужно записывать сотни образцов самостоятельно.

#### Установка Piper Sample Generator:

```python
# В Colab или локально
!git clone https://github.com/rhasspy/piper-sample-generator
!pip install piper-tts piper-phonemize

# Скачиваем голосовую модель (например, для русского языка)
!wget -O piper-sample-generator/models/ru_RU-ruslan-medium.onnx \
  'https://huggingface.co/rhasspy/piper-voices/resolve/main/ru/ru_RU/ruslan/medium/ru_RU-ruslan-medium.onnx'

!wget -O piper-sample-generator/models/ru_RU-ruslan-medium.onnx.json \
  'https://huggingface.co/rhasspy/piper-voices/resolve/main/ru/ru_RU/ruslan/medium/ru_RU-ruslan-medium.onnx.json'
```

#### Тестирование произношения:

```python
import sys
sys.path.append("piper-sample-generator/")
import generate_samples
from IPython.display import Audio

# Ваше wake word
target_word = 'привет джарвис'  # Измените на свое

# Генерируем тестовый образец
generate_samples.generate_samples_onnx(
    text=target_word,
    max_samples=1,
    length_scales=[1.0],
    noise_scales=[0.7],
    noise_scale_ws=[0.7],
    output_dir='./',
    batch_size=1,
    file_names=["test_wake_word.wav"],
    model="piper-sample-generator/models/ru_RU-ruslan-medium.onnx"
)

# Прослушиваем результат
Audio("test_wake_word.wav", autoplay=True)
```

**Важно:** Если произношение неправильное:
- Попробуйте фонетическое написание: `"привет джарвис"` → `"при_вет джар_вис"`
- Пишите числа словами: `"2"` → `"два"`
- Избегайте спецсимволов и пунктуации

### Шаг 3: Настройка конфигурации обучения

Создайте файл конфигурации `config.yaml`:

```yaml
# Основные параметры
model_name: "privet_jarvis"  # Имя вашей модели
wake_word: "привет джарвис"  # Ваше wake word
language: "ru"

# Параметры генерации данных
positive_samples: 10000  # Количество положительных примеров
negative_samples: 50000  # Количество отрицательных примеров

# Параметры модели
model_size: "small"  # small, medium, large
target_false_accepts_per_hour: 0.5  # Целевое количество ложных срабатываний

# Параметры обучения
epochs: 30
batch_size: 512
learning_rate: 0.001

# Пути к данным
output_dir: "./models"
negative_data_dir: "./negative_data"  # Фоновые шумы
```

### Шаг 4: Загрузка негативных данных

Негативные данные — это фоновые звуки, музыка, речь (не wake word), которые помогают модели не срабатывать ложно.

```python
# Скачиваем предобработанные негативные данные с Hugging Face
from huggingface_hub import snapshot_download

# Ambient noise (фоновые шумы)
snapshot_download(
    repo_id="kahrendt/microwakeword_negative_data",
    repo_type="dataset",
    local_dir="./negative_data/ambient"
)

# Speech data (общая речь)
snapshot_download(
    repo_id="kahrendt/common_voice_negative_data",
    repo_type="dataset",
    local_dir="./negative_data/speech"
)
```

### Шаг 5: Обучение модели

#### На Windows с RTX 4070:

```powershell
# Убедитесь, что venv активирован
.\venv\Scripts\Activate.ps1

# Запустите Jupyter Notebook
jupyter notebook

# Откроется браузер с Jupyter
# Откройте basic_training_notebook.ipynb
```

Или используйте Python скрипт напрямую:

```python
# train_wake_word.py
from microwakeword import train
import tensorflow as tf

# Проверяем GPU
print("GPU доступен:", tf.config.list_physical_devices('GPU'))
print("TensorFlow версия:", tf.__version__)

# Загружаем конфигурацию
config = train.load_config("config.yaml")

# Генерируем обучающие данные
print("Генерация обучающих данных...")
train.generate_training_data(config)

# Обучаем модель
print("Начинаем обучение модели...")
model = train.train_model(config)

# Конвертируем в TFLite для ESP32
print("Конвертация в TFLite...")
train.convert_to_tflite(
    model,
    output_path=f"./models/{config['model_name']}.tflite",
    quantize=True  # Квантизация для уменьшения размера
)

print(f"Модель сохранена: ./models/{config['model_name']}.tflite")
```

Запустите скрипт:
```powershell
python train_wake_word.py
```

**Мониторинг GPU на Windows:**

Откройте второе окно PowerShell и запустите:
```powershell
# Мониторинг использования GPU
nvidia-smi -l 1
```

Вы увидите:
- Загрузку GPU (должна быть ~90-100% во время обучения)
- Использование памяти (RTX 4070 имеет 12GB VRAM — более чем достаточно)
- Температуру GPU

**Процесс обучения на RTX 4070:**
- ⚡ Генерация данных: ~10-15 минут
- ⚡ Обучение модели: ~30-60 минут (зависит от количества эпох)
- ⚡ Конвертация: ~1-2 минуты
- **Общее время: ~1-2 часа** (в 2-4 раза быстрее Colab!)

**Советы для оптимизации на RTX 4070:**

```yaml
# В config.yaml увеличьте batch_size для использования всей VRAM
training:
  batch_size: 1024  # Вместо 512 - RTX 4070 справится
  epochs: 30
  learning_rate: 0.001
  
  # Используйте mixed precision для ускорения
  use_mixed_precision: true
```

В Python коде:
```python
# Включаем mixed precision для ускорения на RTX 4070
from tensorflow.keras import mixed_precision
mixed_precision.set_global_policy('mixed_float16')

# Это ускорит обучение на ~30-40% без потери качества
```

### Шаг 6: Тестирование модели

```python
# Тестируем на валидационных данных
results = train.evaluate_model(
    model_path="./models/privet_jarvis.tflite",
    test_data_dir="./test_data"
)

print(f"Accuracy: {results['accuracy']:.2%}")
print(f"False Accepts/Hour: {results['false_accepts_per_hour']:.2f}")
print(f"False Rejects: {results['false_rejects']:.2%}")
```

**Целевые метрики:**
- Accuracy: > 95%
- False Accepts/Hour: < 1.0
- False Rejects: < 5%

### Шаг 7: Экспорт и развертывание модели

После успешного обучения у вас будет файл `.tflite`, который нужно загрузить на ваши устройства.

```powershell
# Проверьте файл модели
dir models\privet_jarvis.tflite
# Должен быть ~200-400 KB
```

**Важно:** После обучения модели на вашем ПК, вам нужно только **один раз** загрузить файл модели на ESP32 или сервер Home Assistant. После этого **ПК можно выключить** — модель будет работать независимо на устройствах.

#### Куда загружать модель:

**Вариант 1: Для ESP32-S3 (on-device wake word)**
- Модель загружается **в ESP32** при прошивке через ESPHome
- ESP32 работает автономно, ПК не нужен

**Вариант 2: Для Home Assistant (серверная обработка)**
- Модель загружается **на сервер Home Assistant**
- Сервер обрабатывает wake word, ПК не нужен

#### Способы развертывания:

**A. Через GitHub (рекомендуется для ESP32):**

```powershell
# 1. Создайте репозиторий на GitHub
# 2. Загрузите модель
git init
git add models/privet_jarvis.tflite
git commit -m "Add custom wake word model"
git remote add origin https://github.com/ваш-username/wake-word-models.git
git push -u origin main

# 3. Получите raw URL:
# https://github.com/ваш-username/wake-word-models/raw/main/models/privet_jarvis.tflite
```

В ESPHome конфигурации используйте URL:
```yaml
micro_wake_word:
  models:
    - model: privet_jarvis
      url: https://github.com/ваш-username/wake-word-models/raw/main/models/privet_jarvis.tflite
```

**B. Копирование на сервер Home Assistant:**

```powershell
# Скопируйте файл на сервер через SSH/SCP
scp models/privet_jarvis.tflite root@homeassistant.local:/config/esphome/models/

# Или через Samba/общую папку Windows
# Скопируйте в: \\homeassistant.local\config\esphome\models\
```

В ESPHome конфигурации используйте локальный путь:
```yaml
micro_wake_word:
  models:
    - model: privet_jarvis
      file: models/privet_jarvis.tflite
```

**C. Для OpenWakeWord в Home Assistant:**

```powershell
# Скопируйте модель в папку custom_wake_words
scp models/privet_jarvis.tflite root@homeassistant.local:/config/custom_wake_words/

# Или через Samba
# Скопируйте в: \\homeassistant.local\config\custom_wake_words\
```

#### Процесс развертывания (один раз):

```
┌─────────────────┐
│  ПК с RTX 4070  │  ← Обучение модели (1-2 часа)
│  (Windows)      │
└────────┬────────┘
         │ Генерирует privet_jarvis.tflite (~300 KB)
         ↓
┌────────────────────────────────────────┐
│  Загрузка модели (выберите способ):   │
│  • GitHub (URL)                        │
│  • Сервер HA (SSH/Samba)              │
│  • Локальная папка ESPHome            │
└────────┬───────────────────────────────┘
         ↓
┌─────────────────┐      ┌──────────────────┐
│   ESP32-S3      │  или │  Home Assistant  │
│   (on-device)   │      │  (сервер)        │
└─────────────────┘      └──────────────────┘
         ↓                        ↓
    Работает 24/7           Работает 24/7
    ПК НЕ НУЖЕН!           ПК НЕ НУЖЕН!
```

#### После развертывания:

✅ **ПК можно выключить** — модель уже на устройстве  
✅ **ESP32/HA работают автономно** — wake word обрабатывается локально  
✅ **Интернет не нужен** — всё работает офлайн  
✅ **Обновление модели** — только если хотите улучшить/изменить wake word  

---

## Метод 2: OpenWakeWord (для Home Assistant)

OpenWakeWord проще в обучении и работает на сервере Home Assistant, а не на ESP32.

### Шаг 1: Использование Google Colab

Самый простой способ — использовать готовый Colab notebook:

1. Откройте [OpenWakeWord Training Notebook](https://colab.research.google.com/github/dscripka/openWakeWord/blob/main/notebooks/automatic_model_training.ipynb)
2. Включите GPU: `Runtime` → `Change runtime type` → `GPU`

### Шаг 2: Модификация для русского языка

Замените код генерации данных на поддержку русского TTS:

```python
# Скачиваем русскую модель Piper
!wget -O models/ru_RU-ruslan-medium.onnx \
  'https://huggingface.co/rhasspy/piper-voices/resolve/main/ru/ru_RU/ruslan/medium/ru_RU-ruslan-medium.onnx'

!wget -O models/ru_RU-ruslan-medium.onnx.json \
  'https://huggingface.co/rhasspy/piper-voices/resolve/main/ru/ru_RU/ruslan/medium/ru_RU-ruslan-medium.onnx.json'

# Устанавливаем зависимости
!pip install piper-tts piper-phonemize
!pip install tensorflow==2.19.0
!pip install onnx==1.17.0
!pip install onnx2tf
!pip install onnxruntime==1.18.1
```

### Шаг 3: Генерация и обучение

```python
# Задаем wake word
target_wake_word = "окей дом"

# Запускаем автоматическое обучение
# Notebook автоматически:
# 1. Генерирует положительные примеры через TTS
# 2. Загружает негативные данные
# 3. Обучает модель
# 4. Конвертирует в ONNX и TFLite

# Следуйте инструкциям в notebook
```

### Шаг 4: Скачивание модели

После обучения скачайте файлы:
- `your_wake_word.onnx` — для OpenWakeWord в Home Assistant
- `your_wake_word.tflite` — для microWakeWord в ESPHome

---

## Интеграция с ESPHome

### Для microWakeWord (ESP32-S3)

#### Шаг 1: Загрузка модели

Есть два способа:

**Способ 1: Через GitHub (рекомендуется)**

1. Создайте репозиторий на GitHub
2. Загрузите файл `.tflite` в папку `models/`
3. Используйте raw URL в конфигурации ESPHome

**Способ 2: Локальный файл**

Поместите файл `.tflite` в папку с конфигурацией ESPHome.

#### Шаг 2: Конфигурация ESPHome

```yaml
# esp32-s3-voice-assistant.yaml

esphome:
  name: voice-assistant
  friendly_name: Voice Assistant

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_ESP32S3_DEFAULT_CPU_FREQ_240: "y"

# Микрофон (пример для I2S)
i2s_audio:
  - id: i2s_mic
    i2s_lrclk_pin: GPIO7
    i2s_bclk_pin: GPIO8

microphone:
  - platform: i2s_audio
    id: mic
    adc_type: external
    i2s_din_pin: GPIO9
    channel: left
    sample_rate: 16000
    bits_per_sample: 32bit

# Micro Wake Word
micro_wake_word:
  models:
    # Используем вашу кастомную модель
    - model: privet_jarvis
      # Локальный файл
      file: models/privet_jarvis.tflite
      # Или URL
      # url: https://github.com/your-repo/models/raw/main/privet_jarvis.tflite
      
  on_wake_word_detected:
    - voice_assistant.start:
        wake_word: !lambda return wake_word;

# Voice Assistant
voice_assistant:
  microphone: mic
  use_wake_word: true
  
  on_listening:
    - light.turn_on:
        id: led
        effect: pulse
        
  on_stt_end:
    - light.turn_off: led

# Индикатор (опционально)
light:
  - platform: status_led
    id: led
    pin: GPIO48
```

#### Шаг 3: Компиляция и загрузка

```bash
# Компилируем и загружаем
esphome run esp32-s3-voice-assistant.yaml
```

### Для OpenWakeWord (Home Assistant)

#### Шаг 1: Установка аддона

1. Откройте Home Assistant
2. `Settings` → `Add-ons` → `Add-on Store`
3. Найдите и установите **OpenWakeWord**

#### Шаг 2: Загрузка модели

1. Скопируйте файл `.tflite` или `.onnx` в:
   ```
   /config/custom_wake_words/your_wake_word.tflite
   ```

2. Перезапустите аддон OpenWakeWord

#### Шаг 3: Настройка Assist Pipeline

1. `Settings` → `Voice assistants` → `Add assistant`
2. Выберите:
   - **Wake word:** Ваша кастомная модель
   - **Speech-to-text:** Whisper
   - **Text-to-speech:** Piper
3. Сохраните

#### Шаг 4: Конфигурация ESP32 (без on-device wake word)

```yaml
# esp32-voice-satellite.yaml
esphome:
  name: voice-satellite

esp32:
  board: esp32dev

# Микрофон
microphone:
  - platform: i2s_audio
    id: mic
    # ... настройки микрофона

# Voice Assistant (wake word обрабатывается в HA)
voice_assistant:
  microphone: mic
  use_wake_word: true  # HA будет обрабатывать wake word
  
  on_listening:
    - logger.log: "Listening..."
```

---

## Советы по улучшению точности

### 1. Выбор Wake Word

**Хорошие wake words:**
- ✅ 2-4 слога: "Окей Дом", "Привет Джарвис"
- ✅ Уникальные звуки: избегайте общих слов
- ✅ Четкое произношение

**Плохие wake words:**
- ❌ Слишком короткие: "Дом", "Эй"
- ❌ Общие слова: "Привет", "Да", "Нет"
- ❌ Сложное произношение

### 2. Улучшение обучающих данных

```python
# Увеличьте разнообразие генерации
generate_samples.generate_samples_onnx(
    text=wake_word,
    max_samples=10000,
    # Варьируйте скорость речи
    length_scales=[0.8, 0.9, 1.0, 1.1, 1.2],
    # Варьируйте шум
    noise_scales=[0.5, 0.6, 0.7, 0.8],
    noise_scale_ws=[0.5, 0.6, 0.7, 0.8],
    # Используйте разные голоса
    model="models/ru_RU-ruslan-medium.onnx"
)
```

### 3. Настройка гиперпараметров

```yaml
# В config.yaml
training:
  # Увеличьте вес негативного класса для уменьшения ложных срабатываний
  negative_class_weight: 2.0
  positive_class_weight: 1.0
  
  # Настройте порог активации
  threshold: 0.7  # Выше = меньше ложных срабатываний, но больше пропусков
  
  # Используйте аугментацию
  spec_augment: true
  time_mask_max: 25
  freq_mask_max: 7
```

### 4. Добавление реальных записей

Для максимальной точности добавьте реальные записи:

```python
# Запишите себя, произносящего wake word 50-100 раз
# Сохраните в папку positive_samples/

# Добавьте в конфигурацию
config['real_positive_samples_dir'] = './positive_samples'
```

### 5. Тестирование в реальных условиях

```python
# Тестируйте модель с:
# - Фоновой музыкой
# - Разговорами
# - Шумом бытовых приборов
# - Разными расстояниями от микрофона

# Записывайте ложные срабатывания и добавляйте их в негативные данные
```

---

## Решение проблем

### Проблема 1: Модель не срабатывает

**Причины:**
- Порог активации слишком высокий
- Недостаточно обучающих данных
- Плохое качество микрофона

**Решение:**
```yaml
# В ESPHome понизьте порог
micro_wake_word:
  models:
    - model: your_model
      probability_cutoff: 0.5  # Понизьте с 0.7
      sliding_window_average_size: 10  # Увеличьте для стабильности
```

### Проблема 2: Слишком много ложных срабатываний

**Решение:**
```yaml
# Повысьте порог
micro_wake_word:
  models:
    - model: your_model
      probability_cutoff: 0.8  # Повысьте
      
# Или переобучите модель с большим negative_class_weight
```

### Проблема 3: Ошибка "Model too large"

**Решение:**
```python
# Используйте более агрессивную квантизацию
converter = tf.lite.TFLiteConverter.from_keras_model(model)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.target_spec.supported_types = [tf.int8]  # int8 вместо float16
tflite_model = converter.convert()
```

### Проблема 4: Низкая точность на русском языке

**Решение:**
1. Используйте лучшую модель Piper:
   ```python
   # Попробуйте разные голоса
   models = [
       "ru_RU-ruslan-medium",
       "ru_RU-dmitri-medium",
       "ru_RU-irina-medium"
   ]
   ```

2. Добавьте фонетические варианты:
   ```python
   wake_word_variants = [
       "привет джарвис",
       "при_вет джар_вис",
       "privet jarvis"
   ]
   ```

### Проблема 5: Модель не загружается в ESPHome

**Проверьте:**
```bash
# Размер файла
ls -lh your_model.tflite
# Должен быть < 500 KB для ESP32-S3

# Формат файла
file your_model.tflite
# Должно быть: "TensorFlow Lite model"

# Проверьте логи ESPHome
esphome logs your_device.yaml
```

---

## Полезные ссылки

### Официальные репозитории:
- [microWakeWord](https://github.com/OHF-Voice/micro-wake-word) — обучение моделей для ESP32-S3
- [microWakeWord (оригинал)](https://github.com/kahrendt/microWakeWord) — оригинальный репозиторий
- [ESPHome micro-wake-word-models](https://github.com/esphome/micro-wake-word-models) — готовые модели
- [OpenWakeWord](https://github.com/dscripka/openWakeWord) — альтернативный метод обучения
- [Piper TTS](https://github.com/rhasspy/piper) — генерация синтетических данных
- [Piper Sample Generator](https://github.com/rhasspy/piper-sample-generator) — генератор образцов

### Документация:
- [ESPHome Micro Wake Word](https://esphome.io/components/micro_wake_word.html)
- [Home Assistant Voice](https://www.home-assistant.io/voice_control/)
- [Year of the Voice - Chapter 6](https://www.home-assistant.io/blog/2024/02/21/voice-chapter-6/)

### Обучающие материалы:
- [Colab Notebook для OpenWakeWord (французский, адаптируется)](https://community.home-assistant.io/t/guide-train-a-custom-french-wake-word-for-home-assistant-with-openwakeword-colab/943111)
- [Picovoice Wake Word Guide](https://picovoice.ai/blog/complete-guide-to-wake-word/)
- [Smart Home Circle Tutorial](https://smarthomecircle.com/How-to-setup-on-device-wake-word-for-voice-assistant-home-assistant)

### Датасеты для обучения:
- [Hugging Face - microWakeWord Negative Data](https://huggingface.co/datasets/kahrendt/microwakeword_negative_data)
- [Common Voice](https://commonvoice.mozilla.org/) — для негативных примеров речи

### Сообщество:
- [Home Assistant Community Forum - Voice](https://community.home-assistant.io/c/voice-assistant/)
- [ESPHome Discord](https://discord.gg/KhAMKrd)

---

## Часто задаваемые вопросы (FAQ)

### Q: После обучения модели нужно держать ПК включенным?
**A:** Нет! После обучения вы:
1. Обучаете модель на ПК (1-2 часа с RTX 4070)
2. Получаете файл `.tflite` (~300 KB)
3. **Один раз** загружаете его на ESP32 или сервер Home Assistant
4. **Выключаете ПК** — модель работает на устройстве автономно 24/7

ПК нужен только для обучения, не для работы модели!

### Q: Куда загружать обученную модель?
**A:** Зависит от выбранного метода:
- **ESP32-S3** (on-device): Модель прошивается в ESP32 через ESPHome (через GitHub URL или локальный файл)
- **Home Assistant** (сервер): Модель копируется на сервер HA в папку `/config/custom_wake_words/`

После загрузки устройство работает независимо от вашего ПК.

### Q: Нужен ли мощный сервер для обучения?
**A:** Нет. Достаточно обычного ПК с NVIDIA GPU. RTX 4070 — отличный выбор. Можно даже без GPU, но будет медленнее (~8-12 часов).

### Q: Сколько времени занимает обучение на RTX 4070?
**A:** 
- Генерация данных: ~10-15 минут
- Обучение: ~30-60 минут
- Общее время: ~1-2 часа

### Q: Можно ли использовать русский язык?
**A:** Да! Используйте русские модели Piper TTS (например, `ru_RU-ruslan-medium.onnx`). Примеры в туториале.

### Q: Какой размер должна быть модель для ESP32-S3?
**A:** Оптимально 200-400 KB. Максимум ~500 KB. Используйте квантизацию для уменьшения размера.

### Q: Что лучше: microWakeWord или OpenWakeWord?
**A:**
- **microWakeWord** — для on-device detection на ESP32-S3 (быстрее, приватнее)
- **OpenWakeWord** — для обработки в Home Assistant (проще настроить)

### Q: Можно ли использовать несколько wake words?
**A:** Да! В ESPHome можно загрузить несколько моделей:
```yaml
micro_wake_word:
  models:
    - model: privet_jarvis
    - model: okay_dom
    - model: hey_assistant
```

### Q: Нужно ли записывать свой голос?
**A:** Нет! microWakeWord использует синтетическую генерацию через Piper TTS. Но добавление 50-100 реальных записей улучшит точность.

### Q: Работает ли без интернета?
**A:** Да! После обучения модель работает полностью локально без интернета.

### Q: Можно ли обучить модель для другого языка?
**A:** Да! Скачайте соответствующую модель Piper TTS с [Hugging Face](https://huggingface.co/rhasspy/piper-voices/tree/main). Доступны десятки языков.

---

## Заключение

Создание кастомного wake word — это сложный, но выполнимый процесс. Основные шаги:

1. ✅ Выберите метод (microWakeWord для ESP32-S3 или OpenWakeWord для HA)
2. ✅ Подготовьте окружение (ваш ПК с RTX 4070 — отличный выбор!)
3. ✅ Сгенерируйте обучающие данные через Piper TTS
4. ✅ Обучите модель (1-2 часа на RTX 4070)
5. ✅ Протестируйте и настройте параметры
6. ✅ **Один раз** загрузите модель на ESP32 или сервер HA
7. ✅ **Выключите ПК** — устройства работают автономно!

### Рабочий процесс:

```
Обучение (на ПК с RTX 4070):
┌──────────────────────────────────────┐
│ 1. Генерация данных      (~15 мин)  │
│ 2. Обучение модели       (~60 мин)  │
│ 3. Тестирование          (~15 мин)  │
│ 4. Экспорт .tflite       (~2 мин)   │
└──────────────────────────────────────┘
         ↓ Загрузка модели (один раз)
┌──────────────────────────────────────┐
│ ESP32-S3 или Home Assistant сервер   │
│ Работает 24/7 без вашего ПК!        │
└──────────────────────────────────────┘
```

**Важно помнить:**
- Обучение требует экспериментов с гиперпараметрами
- Качество модели зависит от разнообразия обучающих данных
- Тестируйте в реальных условиях использования
- Не бойтесь переобучать модель несколько раз
- **ПК нужен только для обучения, не для работы модели**
- После загрузки модели на устройство, оно работает полностью автономно

### Что происходит после развертывания:

| Компонент | Где работает | Нужен ли ПК? | Нужен ли интернет? |
|-----------|--------------|--------------|-------------------|
| **Обучение модели** | ПК с RTX 4070 | ✅ Да (1-2 часа) | ✅ Да (скачать данные) |
| **Wake Word Detection** | ESP32-S3 или HA | ❌ Нет | ❌ Нет |
| **Voice Assistant** | Home Assistant | ❌ Нет | ❌ Нет (если Whisper/Piper локально) |

Удачи в создании вашего персонального голосового ассистента! 🎤🏠