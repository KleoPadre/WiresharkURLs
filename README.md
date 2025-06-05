# PCAPNG HTTPS Domain Extractor

Инструмент для извлечения HTTPS доменов из файлов захвата сетевого трафика (PCAPNG) с использованием Python и Wireshark.

## 📋 Описание

Этот скрипт анализирует файлы PCAPNG (созданные Wireshark или другими сетевыми анализаторами) и извлекает из них только чистые доменные имена HTTPS-сайтов. Скрипт фильтрует результаты, убирая HTTP-адреса, IP-адреса, локальные адреса и пути после доменного имени.

### Что делает скрипт:

- ✅ Извлекает только HTTPS домены (HTTP игнорируются)
- ✅ Убирает пути после доменного имени (`example.com/path` → `example.com`)
- ✅ Исключает IP-адреса (`192.168.1.1`, `2001:db8::1`)
- ✅ Фильтрует локальные адреса (`localhost`, `192.168.x.x`, `10.x.x.x`)
- ✅ Удаляет дубликаты и сортирует результаты
- ✅ Автоматически именует выходные файлы
- ✅ Сохраняет результаты в отдельную папку

## 🔧 Требования

### Системные требования:

- Python 3.6+
- Wireshark (включает tshark)

### Python библиотеки:

- `pyshark` - для анализа PCAPNG файлов

## 📦 Установка

### 1. Установка Wireshark

**Windows:**

1. Скачайте Wireshark с [официального сайта](https://www.wireshark.org/download.html)
2. Установите `.exe` файл
3. Убедитесь, что выбрана опция установки `tshark`

**Linux (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install wireshark tshark
```

**Linux (CentOS/RHEL):**

```bash
sudo yum install wireshark wireshark-cli
# или для новых версий:
sudo dnf install wireshark wireshark-cli
```

**macOS:**

```bash
# Через Homebrew
brew install wireshark
```

### 2. Проверка установки tshark

```bash
tshark --version
```

### 3. Установка Python зависимостей

**Глобальная установка:**

```bash
pip install pyshark
```

**Или создание виртуального окружения (рекомендуется):**

```bash
# Создание виртуального окружения
python -m venv venv

# Активация виртуального окружения
# Linux/macOS:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Установка зависимостей
pip install pyshark
```

## 📥 Получение `.pcapng` файла на роутере Keenetic

Чтобы получить `.pcapng` файл с роутера Keenetic для анализа в Wireshark:

1. Перейдите в веб-интерфейс Keenetic
2. Откройте раздел **Управление → Диагностика → Захват сетевых пакетов**

<img width="1375" alt="image" src="https://github.com/user-attachments/assets/47d8722b-604d-422c-a10e-8fbd70d47482" />


3. Создайте правило, нажав кнопку **Добавить правило захвата**, выберите подключение, через которое хотите собрать адреса и выберете, куда сохранять файл
4. В выпадающем списке **Подключение** выберите нужное (например, `PPPoE0`), через которое хотите собрать адреса и выберете
5. Нужно выбрать флешку как местос сохрарения в поле **Место хранения**, так как файл можетбыть больше 100мб
6. Нажмите **Сохранить**

<img width="502" alt="image" src="https://github.com/user-attachments/assets/b66b4270-cd07-4059-aae7-159bbb46e7bb" />


7. Нажмите кнопку **Запустить**
8. Открывайте сайты, с которых хотите собрать адреса (на компьютере, телевизове, телефоне)
9. Через некоторое время нажмите **Остановить**
10. Скачайте полученный `.pcapng` файл на компьютер

Теперь вы можете использовать этот файл с данным проектом для анализа URL и доменов.

## 🚀 Использование

### Основной скрипт

```bash
python main.py <путь_к_pcapng_файлу>
```

**Примеры:**

```bash
# Простой пример
python main.py capture.pcapng

# Файл с пробелами в имени
python main.py "capture-PPPoE0-Jun 5 16-16-32.pcapng"

# Полный путь к файлу
python main.py /path/to/network-capture.pcapng
```

### Диагностический скрипт

Для анализа содержимого PCAPNG файла и поиска конкретных доменов:

```bash
python diagnostic.py <путь_к_pcapng_файлу> [домены_для_поиска]
```

**Примеры:**

```bash
# Поиск YouTube-связанных доменов (по умолчанию)
python diagnostic.py capture.pcapng

# Поиск конкретных доменов
python diagnostic.py capture.pcapng google.com facebook.com

# Поиск доменов содержащих подстроки
python diagnostic.py capture.pcapng googlevideo ytimg ggpht
```

## 📁 Структура проекта

```
pcapng-extractor/
├── main.py              # Основной скрипт для извлечения доменов
├── diagnostic.py        # Диагностический скрипт для анализа
├── results/            # Папка с результатами (создается автоматически)
│   └── urls-*.txt      # Файлы с извлеченными доменами
├── venv/               # Виртуальное окружение (опционально)
└── README.md           # Этот файл
```

## 📄 Формат выходных файлов

### Именование файлов

Выходные файлы автоматически именуются по шаблону:

```
results/urls-{исходное_имя_файла}.txt
```

**Примеры:**

- `capture.pcapng` → `results/urls-capture.txt`
- `network-trace-2024.pcapng` → `results/urls-network-trace-2024.txt`

### Содержимое файла

```
HTTPS домены из файла: capture.pcapng
Всего найдено уникальных доменов: 25
==================================================

facebook.com
google.com
instagram.com
stackoverflow.com
youtube.com
```

## 🔍 Принцип работы

Скрипт анализирует три основных источника доменов в сетевом трафике:

### 1. HTTP заголовки

- `Host` заголовки из HTTP запросов
- `Referer` заголовки
- Но сохраняет только если это HTTPS трафик

### 2. TLS/SSL SNI (Server Name Indication)

- Доменные имена из TLS handshake
- Наиболее надежный источник для HTTPS трафика

### 3. DNS запросы

- Запросы к DNS серверам
- Показывает какие домены резолвились

## ⚙️ Настройки фильтрации

### Исключаются автоматически:

- **HTTP адреса** (`http://example.com`)
- **IP-адреса** (`192.168.1.1`, `2001:db8::1`)
- **Локальные адреса**:
  - `localhost`
  - `127.x.x.x`
  - `192.168.x.x`
  - `10.x.x.x`
  - `172.16.x.x` - `172.31.x.x`
- **Пути и параметры** (`example.com/path?param=value`)
- **Порты** (`example.com:8080`)

### Сохраняются только:

- ✅ Валидные доменные имена
- ✅ HTTPS трафик
- ✅ Внешние (не локальные) адреса

## 🐛 Диагностика проблем

### Если домены не найдены:

1. **Используйте диагностический скрипт:**

   ```bash
   python diagnostic.py your-file.pcapng domain-to-find.com
   ```

2. **Проверьте содержимое файла в Wireshark:**

   - Откройте файл в Wireshark
   - Используйте фильтры: `http.host`, `tls.handshake.extensions_server_name`, `dns.qry.name`

3. **Проверьте права доступа:**
   ```bash
   # Linux/macOS - может потребоваться добавить пользователя в группу wireshark
   sudo usermod -a -G wireshark $USER
   ```

### Частые проблемы:

**ModuleNotFoundError: No module named 'pyshark'**

```bash
# Убедитесь, что активировано виртуальное окружение
source venv/bin/activate
pip install pyshark
```

**tshark not found**

```bash
# Проверьте установку Wireshark
tshark --version
# Если не найден, переустановите Wireshark
```

**Файл не найден**

```bash
# Используйте кавычки для файлов с пробелами
python main.py "file with spaces.pcapng"
# Или полный путь
python main.py /full/path/to/file.pcapng
```

## 📊 Примеры использования

### Анализ домашнего трафика

```bash
# Захват трафика на роутере Keenetic
# Экспорт в PCAPNG через веб-интерфейс
python main.py home-traffic-2024-06-05.pcapng
```

### Анализ трафика приложения

```bash
# Захват трафика конкретного приложения
python main.py app-traffic.pcapng

# Поиск конкретных доменов
python diagnostic.py app-traffic.pcapng api.service.com
```

### Мониторинг YouTube трафика

```bash
# Диагностика YouTube доменов
python diagnostic.py youtube-session.pcapng googlevideo ytimg ggpht
```

## 🤝 Вклад в проект

1. Форкните репозиторий
2. Создайте ветку для новой функции (`git checkout -b feature/amazing-feature`)
3. Зафиксируйте изменения (`git commit -m 'Add amazing feature'`)
4. Отправьте в ветку (`git push origin feature/amazing-feature`)
5. Откройте Pull Request

## 📝 Лицензия

Этот проект распространяется под лицензией MIT. См. файл `LICENSE` для подробностей.

## 🔗 Полезные ссылки

- [Wireshark Download](https://www.wireshark.org/download.html)
- [PyShark Documentation](https://kiminewt.github.io/pyshark/)
- [PCAPNG Format Specification](https://tools.ietf.org/html/draft-tuexen-opsawg-pcapng-02)
- [Keenetic Router Manual](https://help.keenetic.com/)

## ❓ FAQ

**Q: Почему скрипт находит не все домены?**
A: Используйте диагностический скрипт для анализа. Некоторые домены могут передаваться через зашифрованные соединения или нестандартные протоколы.

**Q: Можно ли анализировать файлы .pcap?**
A: Да, PyShark поддерживает оба формата - .pcap и .pcapng.

**Q: Как ускорить обработку больших файлов?**
A: Рассмотрите возможность разбиения больших файлов на части или фильтрации трафика в Wireshark перед экспортом.

**Q: Безопасно ли использовать скрипт?**
A: Скрипт только читает файлы захвата и не выполняет сетевых подключений. Однако будьте осторожны с файлами из недоверенных источников.
