# Таймлапсы Томска в Wplace.live

## [Доступно и в Telegram!](https://t.me/wpttimelapse)

Этот репозиторий содержит автоматически генерируемые таймлапсы на основе дампов из [этого репозитория](https://github.com/ugliestie/wplace-tomsk).
Видео сохранённых таймлапсов лежат в папке `timelapse/`.

### Как это работает
- **Ежедневно** запускается GitHub Actions, считает «вчера» по часовому поясу Томска (UTC+7), забирает дампы выборочно (sparse checkout) из `niklinque/wplace-tomsk/output/YYYMMDD` и генерирует квадратное видео 3000*3000 с меткой времени по центру снизу.
- **Автоматическая отправка в Telegram** через локальный Bot API сервер, поддерживающий файлы до 2GB (вместо стандартных 50MB).

### Запуск локально
1) Установить зависимости:
```bash
python -m venv .venv
. .venv/bin/activate  # Windows: .venv\\Scripts\\activate
pip install -r requirements.txt
```
2) Подготовить входные данные. Скрипт ожидает изображения формата `merged_tiles_*.png` в папке `output/YYYYMMDD/`.
   - Быстро забрать только нужный день из исходного репозитория можно так:
```bash
git clone --filter=blob:none --no-checkout https://github.com/niklinque/wplace-tomsk.git tmp_wplace
cd tmp_wplace
git sparse-checkout init --cone
git sparse-checkout set output/20250101  # подставьте нужную дату
git checkout main
cd ..
mkdir -p output
cp -r tmp_wplace/output/20250101 output/20250101
```
3) Запустить генерацию:
```bash
python create_timelapse.py --date 20250101
```
Результат появится в `timelapse/timelapse_20250101.mp4` и дубликат `timelapse/latest.mp4`.

### Не синхронизировать `timelapse/` локально
Чтобы к вам не подтягивались большие видео при `git pull`, используйте sparse checkout и исключите каталог `timelapse/` из рабочей копии:
```bash
git sparse-checkout init --no-cone
git sparse-checkout set "/*" "!/timelapse/"
```
Вернуть всё обратно:
```bash
git sparse-checkout disable
# или снова включить всё:
git sparse-checkout init --no-cone
git sparse-checkout set "/*"
```

### Структура
- `create_timelapse.py` — генерация квадратного видео 3000×3000, 9 FPS, белый фон, метка времени по центру снизу.
- `timelapse/timelapse_YYYYMMDD.mp4` — готовый таймлапс за день, `timelapse/latest.mp4` — копия последнего.
- `.github/workflows/generate-timelapse.yml` — ежедневный воркфлоу.
- `.github/workflows/generate-timelapse-artifat.yml` — воркфлоу для выгрузки в артефакт таймлапса из любого дня доступного в репозитории с дампами.
- `.github/workflows/telegram-upload.yml` — отправка больших видео в Telegram через локальный Bot API сервер.

