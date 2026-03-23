# mbscan -- Быстрый сканер шины Modbus RTU

Легковесная консольная утилита для сканирования шины Modbus RTU через последовательный порт. Открывает порт один раз и опрашивает диапазон адресов, отправляя запросы FC03 (Read Holding Registers).

## Возможности

- **Быстрый** -- сканирование 247 адресов за ~2.5 секунды на 115200 бод (таймаут 10мс)
- **Кроссплатформенный** -- Linux (x86_64, aarch64), Windows (x86_64), OpenWrt, Raspberry Pi, Armbian
- **Без зависимостей** -- чистый C, libmodbus не требуется
- **Свой CRC16** -- встроенная реализация Modbus CRC16
- **Настраиваемый** -- скорость, формат данных, диапазон адресов, таймаут, количество регистров

## Быстрый старт

### Linux

```bash
# Сканировать все адреса на /dev/ttyUSB0 (по умолчанию: 115200-8N1, таймаут 100мс)
mbscan -p /dev/ttyUSB0

# Быстрое сканирование с таймаутом 10мс
mbscan -p /dev/ttyUSB0 -o 10

# Сканировать диапазон, читать 4 регистра
mbscan -p /dev/ttyUSB0 -f 1 -t 30 -c 4

# 9600 бод, четность 8E1
mbscan -p /dev/ttyS1 -b 9600 -d 8E1
```

### Windows

```cmd
rem Сканировать COM3 с настройками по умолчанию
mbscan.exe -p COM3

rem Быстрое сканирование, таймаут 10мс
mbscan.exe -p COM3 -o 10

rem Адреса 1-30, 4 регистра, 9600 бод
mbscan.exe -p COM5 -b 9600 -d 8E1 -f 1 -t 30 -c 4

rem COM-порт с номером выше 9
mbscan.exe -p COM15
```

## Пример вывода

### Linux

```
mbscan: scanning /dev/ttyUSB0 115200-8N1, addresses 1-247, timeout 100ms
mbscan: reading 4 register(s) starting at 0

Found slave 125: [0]=125 [1]=1 [2]=830 [3]=794

mbscan: done. Found 1 device(s).
```

### Windows

```
mbscan: scanning COM3 115200-8N1, addresses 1-247, timeout 100ms
mbscan: reading 4 register(s) starting at 0

Found slave 125: [0]=125 [1]=1 [2]=830 [3]=794

mbscan: done. Found 1 device(s).
```

## Использование

```
mbscan -p PORT [опции]

Опции:
  -p PORT    Последовательный порт (обязательный)
             Linux: /dev/ttyUSB0, /dev/ttyS1
             Windows: COM3, COM10, \\.\COM15
  -b BAUD    Скорость (по умолчанию: 115200)
  -d PARAMS  Формат данных: 8N1, 8E1, 8O1, 7E1 и т.д. (по умолчанию: 8N1)
  -f FROM    Начальный адрес (по умолчанию: 1)
  -t TO      Конечный адрес (по умолчанию: 247)
  -o MS      Таймаут на адрес в мс (по умолчанию: 100)
  -r REG     Начальный регистр, с 0 (по умолчанию: 0)
  -c COUNT   Количество регистров для чтения (по умолчанию: 1)
  -v         Подробный вывод
  -h         Показать справку
```

## Готовые бинарники

Готовые бинарники доступны на странице [Releases](../../releases):

| Файл | Платформа | Примечания |
|---|---|---|
| `mbscan-linux-x86_64` | Linux x86_64 | Статический бинарник, без зависимостей |
| `mbscan-linux-aarch64` | Linux aarch64 (ARM64) | Статический бинарник |
| `mbscan.exe` | Windows x86_64 | Статический бинарник, без зависимостей |

## Сборка из исходников

### Linux -- нативная сборка

```bash
cd src
gcc -O2 -Wall -o mbscan mbscan.c
```

Статическая сборка (переносимая, без зависимостей):

```bash
gcc -O2 -Wall -static -o mbscan mbscan.c
```

### Linux ARM64 -- нативная сборка на устройстве

На Armbian, Raspberry Pi OS или любом ARM64 Linux с gcc:

```bash
# Установить gcc при необходимости
sudo apt install gcc

cd src
gcc -O2 -Wall -o mbscan mbscan.c
```

### Linux ARM64 -- кросс-компиляция с x86_64 хоста

```bash
sudo apt install gcc-aarch64-linux-gnu
aarch64-linux-gnu-gcc -O2 -Wall -static -o mbscan-linux-aarch64 src/mbscan.c
```

Флаг `-static` создает самодостаточный бинарник, который работает на любом aarch64 Linux (Armbian, Raspberry Pi OS, Ubuntu, OpenWrt и т.д.) без зависимостей.

### Windows -- кросс-компиляция из Linux

На Windows ничего устанавливать не нужно:

```bash
sudo apt install gcc-mingw-w64-x86-64
x86_64-w64-mingw32-gcc -O2 -Wall -static -o mbscan.exe src/mbscan.c
```

Полученный `mbscan.exe` работает на любой Windows x86_64 без зависимостей.

### Windows -- нативная сборка с MinGW/MSYS2

Установите [MSYS2](https://www.msys2.org/), затем:

```bash
pacman -S mingw-w64-x86_64-gcc
gcc -O2 -Wall -o mbscan.exe mbscan.c
```

### Windows -- нативная сборка с MSVC

Откройте "Developer Command Prompt for VS" и выполните:

```cmd
cl src\mbscan.c /Fe:mbscan.exe /O2
```

### Пакет OpenWrt

Скопируйте каталог `mbscan` в дерево пакетов OpenWrt и соберите:

```bash
cp -r mbscan /path/to/openwrt/package/
cd /path/to/openwrt
echo "CONFIG_PACKAGE_mbscan=y" >> .config
make package/mbscan/compile -j$(nproc)
```

Собранный пакет `.ipk` / `.apk` будет в `bin/packages/*/base/`.

### Кросс-компиляция для aarch64 через тулчейн OpenWrt

```bash
/path/to/openwrt/staging_dir/toolchain-aarch64_generic_gcc-*/bin/aarch64-openwrt-linux-gcc \
  -O2 -Wall -static -o mbscan-linux-aarch64 src/mbscan.c
```

## Как это работает

1. Открывает последовательный порт, настраивает скорость, четность, биты данных/стоп-биты
2. Для каждого адреса в диапазоне:
   - Очищает буфер порта от старых данных
   - Отправляет 8-байтный запрос Modbus RTU FC03 с CRC16
   - Ждет ответ с настраиваемым таймаутом
   - Проверяет ответ: CRC, адрес, код функции
   - Выводит найденные устройства с значениями регистров
3. Соблюдает межкадровую паузу Modbus (3.5 символьных интервала) между запросами

## Производительность

| Таймаут | Полное сканирование (1-247) | Примечания |
|---|---|---|
| 10 мс | ~2.5 сек | Минимум, может пропустить устройства (см. ниже) |
| 20 мс | ~5 сек | Минимальный рекомендуемый |
| 50 мс | ~12 сек | Рекомендуется для большинства случаев |
| 100 мс | ~25 сек | По умолчанию, надежный |
| 200 мс | ~50 сек | Длинные линии RS-485 |

> **Про короткие таймауты:** При `-o 10` некоторые устройства могут быть не обнаружены -- 10мс может не хватить slave-устройству для формирования и отправки ответа, особенно на Windows, где задержки USB-Serial драйверов выше, чем на Linux. Минимальный рекомендуемый таймаут -- **20мс**. Если устройство не найдено, попробуйте увеличить таймаут прежде чем проверять подключение или адрес.

## Заметки по Windows

- Имена COM-портов передаются как есть: `-p COM3`. Префикс `\\.\` добавляется автоматически, поэтому порты с номером выше 9 (COM10+) работают без специального синтаксиса.
- Номер COM-порта можно найти в Диспетчере устройств -> Порты (COM и LPT).
- Популярные USB-Serial адаптеры (CH341, CP2102, FTDI, PL2303) устанавливают драйверы автоматически на Windows 10/11.

## Интеграция

`mbscan` используется как бэкенд для вкладки **Scan Bus** в [luci-app-mbpoll](https://github.com/lab240/luci-app-mbpoll) -- веб-интерфейсе LuCI для опроса Modbus-устройств и сканирования шины на OpenWrt.

## Платы NAPI

Проект разрабатывается и поддерживается командой [NAPI Lab](https://github.com/napilab) и тестируется в первую очередь на промышленных одноплатных компьютерах NAPI на базе SoC Rockchip.

Если вам нужна надежная аппаратная платформа для работы `mbscan` в продакшене, ознакомьтесь с линейкой плат NAPI:

[github.com/napilab/napi-boards](https://github.com/napilab/napi-boards)

- **NAPI2** -- RK3568J, RS-485 на плате, Armbian
- **NAPI-C** -- RK3308, компактный, промышленного класса

## Железо

Протестировано на:

- [Платы NAPI](https://github.com/napilab/napi-boards) -- промышленные IoT-шлюзы (RK3308, RK3568J, aarch64, OpenWrt/Armbian)
- Linux x86_64 с USB-RS485 адаптерами (CH341, CP2102, FTDI)
- Windows 10/11 x86_64 с USB-RS485 адаптерами

## Авторы

- **Дмитрий Новиков** ([@dmnovikov](https://github.com/dmnovikov)) -- [NAPI Lab](https://github.com/napilab)
- **Claude** (Anthropic) -- ИИ-ассистент и соавтор -- архитектура, код, документация

## Лицензия

MIT