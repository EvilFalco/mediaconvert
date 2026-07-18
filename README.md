# mediaconvert

Утилита командной строки для пакетной конвертации изображений, видео и аудио в веб-форматы с сохранением структуры папок. Работает на macOS и Linux.

| Входной формат | Выходной формат | Движок |
|---|---|---|
| PNG, JPG / JPEG | WebP | Pillow |
| MP4 | WebM (VP9 + Opus) | ffmpeg |
| MP3 | OGG (Opus) | ffmpeg |

---

## Установка

### 1. Зависимости

**macOS:**
```bash
pip3 install Pillow transliterate unidecode
brew install ffmpeg
```

**Ubuntu / Debian:**
```bash
pip3 install Pillow transliterate unidecode
sudo apt install ffmpeg
```

**Windows (WSL):**
```bash
pip3 install Pillow transliterate unidecode
sudo apt install ffmpeg
```

### 2. Скрипт

Скопируй файл `mediaconvert` в директорию, которая есть в `$PATH`. Рекомендуется `~/.local/bin/` — она есть на большинстве систем и не требует прав администратора.

```bash
cp mediaconvert ~/.local/bin/mediaconvert
chmod +x ~/.local/bin/mediaconvert
```

### 3. PATH

Убедись, что `~/.local/bin` добавлена в `$PATH`. Проверить:

```bash
echo $PATH | tr ':' '\n' | grep local
```

Если пусто — добавь в `~/.zshrc` (macOS) или `~/.bashrc` (Linux):

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

### 4. Проверка

```bash
mediaconvert --help
```

---

## Быстрый старт

```bash
# Посмотреть что будет сделано — ничего не трогать
mediaconvert ./assets --dry-run

# Конвертировать папку (спросит подтверждение)
mediaconvert ./assets

# Конвертировать без вопросов, оригиналы оставить
mediaconvert ./assets --yes --keep-originals
```

---

## Синтаксис

```
mediaconvert [source] [опции]
```

### Аргументы

| Аргумент | Описание |
|---|---|
| `source` | Исходная директория (по умолчанию: текущая) |

### Опции

| Флаг | Сокращение | По умолчанию | Описание |
|---|---|---|---|
| `--output <dir>` | `-o` | — | Выходная директория. Если не задана — конвертация на месте |
| `--quality <n>` | `-q` | `85` | Качество WebP, 1–100 |
| `--crf <n>` | — | `33` | Качество VP9 для WebM, 0–63 (меньше = лучше) |
| `--audio-quality <n>` | `-aq` | `4` | Битрейт Opus, 0–10 (больше = лучше; 4 = 96kbps) |
| `--dry-run` | `-n` | — | Показать план, ничего не делать |
| `--keep-originals` | `-k` | — | Не удалять оригиналы после конвертации |
| `--yes` | `-y` | — | Пропустить запрос подтверждения |
| `--rename` | `-r` | — | Транслитерировать кириллицу → латиницу, пробелы → `_` в именах выходных файлов |
| `--workers <n>` | `-j` | все ядра CPU | Параллельных процессов для изображений |

---

## Примеры

### Базовые сценарии

```bash
# Текущая папка
mediaconvert

# Конкретная папка
mediaconvert ./assets

# Сухой прогон
mediaconvert ./assets --dry-run

# Конвертировать в другую папку (структура сохранится)
mediaconvert ./assets --output ./assets-web

# Пропустить подтверждение (удобно в скриптах)
mediaconvert ./assets --yes
```

### Управление качеством

```bash
# Изображения: максимальное качество
mediaconvert ./assets --quality 95

# Видео: высокое качество (меньше CRF = лучше)
mediaconvert ./assets --crf 24

# Аудио: высокое качество (192kbps)
mediaconvert ./assets --audio-quality 7

# Всё вместе
mediaconvert ./assets --quality 90 --crf 28 --audio-quality 6
```

**Рекомендуемые значения:**

| Тип | Параметр | Эконом | Баланс | Качество |
|---|---|---|---|---|
| Изображения | `--quality` | 70 | **85** | 95 |
| Видео | `--crf` | 40 | **33** | 24 |
| Аудио (Opus) | `--audio-quality` | 2 (64kbps) | **4 (96kbps)** | 7 (192kbps) |

### Переименование

```bash
# Транслитерировать кириллицу и заменить пробелы на _
mediaconvert ./assets --rename
```

Применяется только к **выходным** файлам — оригиналы не переименовываются.

```
Фото лета 2024.jpg  →  foto_leta_2024.webp
Привет мир.png      →  privet_mir.webp
normal_file.jpg     →  normal_file.webp   ← латиница не трогается
```

Транслитерация: `transliterate` (русский стандарт) + `unidecode` как фоллбэк для прочего Unicode.

Установка зависимостей для `--rename`:
```bash
pip3 install transliterate unidecode
```

**Конфликты имён.** Если два разных исходника дают одинаковое имя после транслитерации (например, `мир.png` и `mir.jpg` — оба → `mir.webp`), второй файл сохраняется с суффиксом `_(conflict)`:

```
мир.png  →  mir.webp
mir.jpg  →  mir_(conflict).webp
```

Предупреждение выводится дважды — до начала конвертации и в итоговой сводке. Файлы не теряются, но требуют ручной проверки.

### Параллелизм (только изображения)

```bash
# Ограничить 4 ядрами, чтобы не перегружать машину
mediaconvert ./assets --workers 4
```

Видео и аудио обрабатываются последовательно: ffmpeg сам использует все доступные ядра для каждого файла.

---

## Как это выглядит в терминале

```
  Images  (PNG/JPG → WebP):   218 to convert  [52 large, max 2 at a time]
  Videos  (MP4    → WebM):      6 to convert
  Audio   (MP3    → OGG):      34 to convert,   5 already done (will skip)

Proceed? [y/N] y

Converting 218 image(s)...
  [  1/218   0%] OK    banner.png       42KB →  18KB  (43%)
  [  2/218   1%] OK    photo.jpg         8KB →   1KB  (11%)
  [  3/218   1%] OK    atlas.png    187837KB → 161921KB  (86%)  [large, lossless fallback]
  ...

Converting 6 video(s)  CRF=33...
  [1/6  17%] intro.mp4  [████████████░░░░░░░░]  60%
```

Для видео и аудио строка обновляется на месте — виден живой прогресс-бар с процентом. По завершении заменяется итогом:

```
  [1/6  17%] OK    intro.mp4    124MB → 38MB  (31%)
```

---

## Поведение при повторном запуске

Если рядом с оригиналом уже лежит файл с нужным расширением (`.webp`, `.webm`, `.ogg`) — он не конвертируется повторно, а помечается как `SKIP`. Оригинал при этом всё равно удаляется в конце, если не передан `--keep-originals`.

---

## Поведение при удалении оригиналов

Удаление происходит **только если все файлы сконвертированы без ошибок** — каждый выходной файл проверяется на существование и ненулевой размер. При любой ошибке оригиналы не трогаются, выводится список проблемных файлов.

---

## Особенности для изображений

**Большие файлы (>20 МБ на диске)** обрабатываются не более чем двумя воркерами одновременно — чтобы не исчерпать память.

**Изображения шире 16 383 пикселей** автоматически уменьшаются до этого предела с сохранением пропорций — это жёсткий лимит формата WebP.

**PNG с прозрачностью** (RGBA) конвертируются корректно, альфа-канал сохраняется в WebP.

**Lossless fallback:** если очень большое изображение не помещается в VP8-партицию, скрипт автоматически переключается на lossless WebP. Такой файл помечается `[lossless fallback]`.

---

## Устранение проблем

**`mediaconvert: command not found`**
```bash
# Проверить PATH
echo $PATH | tr ':' '\n' | grep local

# Добавить ~/.local/bin если не добавлено
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

**`ModuleNotFoundError: No module named 'PIL'`**
```bash
pip3 install Pillow
```

**`ffmpeg: command not found`**
```bash
brew install ffmpeg        # macOS
sudo apt install ffmpeg    # Ubuntu/Debian
```

**`ffprobe: command not found`**
`ffprobe` входит в стандартную поставку ffmpeg. Если он не найден — скорее всего ffmpeg установлен нестандартным способом. Переустановите через `brew` или `apt`.

**Размер файла вырос после конвертации**
Бывает на коротких аудио/видео или уже хорошо сжатых исходниках. Используйте `--keep-originals`, сравните вручную и при необходимости скорректируйте параметры качества.
