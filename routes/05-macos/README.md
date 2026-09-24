# Установка Geant4 на macOS

Этот маршрут предназначен для установки Geant4 из исходного кода на macOS.

Маршрут проверен на:

- MacBook Pro с процессором Intel `x86_64`;
- macOS Ventura 13.7.8;
- Apple Clang 14.0.3;
- CMake 3.31.12;
- Qt 5.15.19;
- Geant4 11.4.2.

В результате будет собран Geant4 с поддержкой Qt5 и OpenGL, после чего установка проверяется на стандартном примере `B1`.

## 1. Открытие Terminal

Откройте поиск Spotlight:

```text
Command + Space
```

Введите:

```text
Terminal
```

и нажмите `Enter`.

## 2. Проверка системы

Выполните:

```bash
uname -m
sw_vers
xcode-select -p
clang --version
```

На проверенной машине архитектура:

```text
x86_64
```

а команда:

```bash
xcode-select -p
```

возвращает:

```text
/Library/Developer/CommandLineTools
```

Это означает, что Command Line Tools уже установлены.

Если они отсутствуют, установите их командой:

```bash
xcode-select --install
```

## 3. Установка MacPorts

На проверенной Intel-машине зависимости устанавливались через MacPorts.

Скачайте установочный `.pkg` для своей версии macOS с официального сайта MacPorts:

<https://www.macports.org/install.php>

Установите пакет обычным способом.

После установки полностью закройте Terminal, откройте его снова и проверьте:

```bash
port version
```

Пример успешного вывода:

```text
Version: 2.12.6
```

## 4. Установка CMake и Qt5

Обновите MacPorts:

```bash
sudo port selfupdate
```

Установите CMake и Qt5:

```bash
sudo port install cmake qt5
```

Установка Qt может занять заметное время и установить большое количество зависимостей.

Проверьте CMake:

```bash
cmake --version
```

Проверьте Qt5:

```bash
/opt/local/libexec/qt5/bin/qmake --version
```

На проверенной машине:

```text
cmake version 3.31.12

QMake version 3.1
Using Qt version 5.15.19 in /opt/local/libexec/qt5/lib
```

## 5. Скачивание исходников Geant4

Создайте рабочую директорию:

```bash
mkdir -p ~/software/Geant4
cd ~/software/Geant4
```

Скачайте Geant4 11.4.2:

```bash
curl -L https://github.com/Geant4/geant4/archive/refs/tags/v11.4.2.tar.gz -o geant4-v11.4.2.tar.gz
```

Распакуйте архив:

```bash
tar -xzf geant4-v11.4.2.tar.gz
```

Проверьте содержимое:

```bash
ls
```

Должны появиться:

```text
geant4-11.4.2
geant4-v11.4.2.tar.gz
```

## 6. Конфигурация Geant4

Исходный код, сборку и установленную версию будем хранить отдельно.

Создайте папку сборки:

```bash
cd ~/software/Geant4
mkdir -p build
cd build
```

Запустите CMake:

```bash
cmake ../geant4-11.4.2 \
  -DCMAKE_INSTALL_PREFIX="$HOME/software/Geant4/geant4-11.4.2-install" \
  -DGEANT4_INSTALL_DATA=ON \
  -DGEANT4_USE_QT=ON \
  -DGEANT4_USE_QT_QT5=ON \
  -DCMAKE_PREFIX_PATH=/opt/local/libexec/qt5
```

Здесь:

- `CMAKE_INSTALL_PREFIX` задаёт папку, куда будет установлен готовый Geant4;
- `GEANT4_INSTALL_DATA=ON` включает загрузку наборов физических данных;
- `GEANT4_USE_QT=ON` включает графический интерфейс;
- `GEANT4_USE_QT_QT5=ON` явно выбирает Qt5;
- `CMAKE_PREFIX_PATH` указывает расположение Qt5 из MacPorts.

> [!IMPORTANT]
> Для Geant4 11.4 при использовании Qt5 необходимо явно указать `GEANT4_USE_QT_QT5=ON`. Без этого CMake может начать искать Qt6 и завершиться ошибкой.

При успешной конфигурации в конце будет примерно:

```text
GEANT4_USE_QT: Build Geant4 with Qt5 support
-- Configuring done
-- Generating done
```

## 7. Сборка Geant4

Запустите сборку с использованием всех доступных потоков процессора:

```bash
cmake --build . --parallel $(sysctl -n hw.ncpu)
```

Сборка может занять продолжительное время.

При успешном завершении последние строки будут выглядеть примерно так:

```text
[100%] Built target G4physicslists
```

## 8. Установка Geant4

После успешной сборки выполните:

```bash
cmake --install .
```

Geant4 установится в:

```text
~/software/Geant4/geant4-11.4.2-install
```

Проверьте содержимое:

```bash
ls ~/software/Geant4/geant4-11.4.2-install
```

Должны быть каталоги:

```text
bin
include
lib
share
```

Проверьте установочные скрипты:

```bash
ls ~/software/Geant4/geant4-11.4.2-install/bin
```

Должны присутствовать:

```text
geant4-config
geant4.csh
geant4.sh
```

## 9. Подключение Geant4 к zsh

Чтобы окружение Geant4 загружалось при каждом запуске Terminal, добавьте его в `~/.zshrc`:

```bash
echo 'source "$HOME/software/Geant4/geant4-11.4.2-install/bin/geant4.sh"' >> ~/.zshrc
```

Примените изменения:

```bash
source ~/.zshrc
```

Проверьте версию:

```bash
geant4-config --version
```

Ожидаемый результат:

```text
11.4.2
```

## 10. Проверка на стандартном примере B1

Перейдите в каталог стандартного примера:

```bash
cd ~/software/Geant4/geant4-11.4.2/examples/basic/B1
```

Создайте отдельную папку сборки:

```bash
mkdir -p build
cd build
```

Сконфигурируйте пример:

```bash
cmake .. \
  -DGeant4_DIR="$HOME/software/Geant4/geant4-11.4.2-install/lib/Geant4-11.4.2"
```

При успешной конфигурации CMake должен найти установленный Geant4:

```text
-- Found Geant4: ... (found version "11.4.2")
-- Configuring done
-- Generating done
```

Соберите пример:

```bash
cmake --build . --parallel $(sysctl -n hw.ncpu)
```

Запустите:

```bash
./exampleB1
```

Должно открыться графическое окно Geant4 с Qt-интерфейсом и визуализацией геометрии примера `B1`.

Если окно открылось и геометрия отображается, значит корректно работают:

- Geant4;
- Qt5;
- OpenGL;
- CMake;
- Apple Clang;
- наборы физических данных Geant4.

## 11. Итоговая структура

После установки рабочая директория выглядит примерно так:

```text
~/software/Geant4/
├── geant4-11.4.2/
├── geant4-v11.4.2.tar.gz
├── build/
└── geant4-11.4.2-install/
    ├── bin/
    ├── include/
    ├── lib/
    └── share/
```

То есть отдельно хранятся:

```text
исходный код
↓
build
↓
установленная версия
```

На этом установка и проверка Geant4 11.4.2 на macOS завершены.
