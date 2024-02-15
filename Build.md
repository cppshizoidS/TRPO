
# Гайд по сборке C/C++ для начинающих

Исходный C++ файл который вы пишете в .c и .cpp это всего лишь код, его нельзя запустить напрямую. Чтобы из .c/.cpp получить .exe/.dll(на windows) .out, .so(на linux) его необходимо скомпилировать.
## Нужные пакеты на linux

Сохраните это как setup_script.sh
потом выполните ```chmod +x setup_script.sh | ./setup_script.sh```
```bash
#!/bin/bash
# Проверка типа дистрибутива
if [ -x "$(command -v apt-get)" ]; then
    sudo apt-get update
    sudo apt-get install -y build-essential neofetch git clang clang-tools gcc cmake make ninja-build lld lldb valgrind clang-format

elif [ -x "$(command -v dnf)" ]; then
    sudo dnf install -y @development-tools neofetch git clang clang-tools-extra gcc cmake make ninja-build lld lldb valgrind


else
    echo "Не удалось определить дистрибутив и установщик пакетов."
    exit 1
fi

```
gcc или clang на выбор

## Процесс компиляции C++ программы:

### Шаг 1: Препроцессинг

Препроцессинг - это первый этап компиляции, где препроцессор обрабатывает исходный код до самой компиляции. На этом этапе выполняются следующие действия:

Обработка директив препроцессора: ```#include``` вставляет содержимое указанного файла прямо в код. Макросы объявленные при помощи ```#define``` подставляются во всех местах в коде где они были использованы. Так же на этом этапе удаляются комментарии.

```cpp
// Пример
#include  <iostream>  //подключение заголовочного файла ввода вывода в C++
// будет произведена замена PI на 3.14159 во всех частях кода
#define PI 3.14159

int main()  {
	std::cout <<"hello"  << PI;
	return  0;
}
```

Чтобы получить файл который получится после препроцессинга можно воспользоваться флагом -E в компиляторе

```gcc/clang -E``` 
```clang++ -E main.cpp -o main.ii```

[Результат на godbolt](https://godbolt.org/z/h6417eh51)

### Шаг 2: Компиляция
Компиляция - это второй этап, на котором препроцессированный код преобразуется в объектные файлы.
Компиляция включает в себя преобразование препроцессированного кода на языке программирования в набор инструкций ассемблера. Этот этап создает объектные файлы (.o или .obj), содержащие машинный код, который представляет собой набор инструкций для конкретной архитектуры.(В примерах используется g++ но вы можете использовать clang++ просто введя вместо g++ - clang++)

Пример использования компилятора GCC для компиляции исходного файла:

```bash 
g++ -c main.cpp -o main.o
``` 

В данном примере:

-   `g++` - это вызов компилятора GCC для языка C++.
-   `-c` - флаг, указывающий компилятору создать только объектный файл.
-   `main.cpp` - имя исходного файла.
-   `-o main.o` - опция, указывающая имя выходного объектного файла.

### Шаг 3: Линковка

Линковка - это процесс объединения нескольких объектных файлов в один исполняемый файл. На этом этапе также происходит разрешение ссылок между различными частями программы.

Пример использования компилятора GCC для сборки объектных файлов в исполняемый файл:

`g++ main.o -o hello` 

В данном примере:

-   `g++` - вызов компилятора GCC для языка C++.
-   `main.o` - объектный файл, который нужно включить в сборку.
-   `-o hello` - опция, указывающая имя выходного исполняемого файла.

Теперь у вас есть исполняемый файл `hello`, который можно запустить:
```shell
./hello
```

Данные инструкции будут удобны если у нас 1 файл который нужно компилировать, но если файлов больше и/или код использует библиотеки то удобнее использовать системы сборки например make*.

*  make не является  системой сборки только для C/C++ это утилита автоматизирующая процесс преобразования файлов из одной формы в другую. Но чаще всего используется как билд система для C/C++.

## Использование [make](https://www.gnu.org/software/make/manual/make.html) для сборки проектов
### Написание Makefile

`Makefile` - это текстовый файл, содержащий инструкции для утилиты `make`. Этот файл определяет зависимости между файлами и указывает, какие команды нужно выполнить для сборки проекта.

Пример простого `Makefile`:

```makefile
CC = g++
CFLAGS = -Wall

all: hello

hello: main.o
	$(CC) $(CFLAGS) main.o -o hello
main.o: main.cpp
	$(CC) $(CFLAGS) -c main.cpp
clean:
	rm -rf *.o hello
```

В данном примере:

-   `CC` - это переменная, содержащая имя компилятора (в данном случае `g++`).
-   `CFLAGS` - флаги компиляции (в данном случае `-Wall` для вывода предупреждений).
-   `all` - цель по умолчанию, которая собирает исполняемый файл `hello`.
-   `hello` - цель для сборки исполняемого файла.
-   `main.o` - цель для сборки объектного файла `main.o`.
-   `clean` - цель для удаления временных файлов.

### Запуск утилиты make

1.  Создайте файл с именем `Makefile` в корневой директории проекта.
2.  Добавьте необходимые инструкции в `Makefile`.
3.  Откройте терминал и перейдите в директорию проекта.
4.  Выполните команду `make` для сборки проекта.

### Запуск исполняемого файла

После успешной сборки проекта выполните следующую команду в терминале для запуска исполняемого файла:
`./hello`


## Сборка проектов использующих [cmake](https://cmake.org/) 
Часто вы можете увидеть проекты в корне которых есть файл CMakeLists.txt это файл описания генерации файлов сборки при помощи cmake
Использование cmake для проектов содержащих 4+ файлов вместо make имхо уже необходимо для большего удобства
Для того чтобы собрать такие проекты нужно перейти в директорую с проектом `cd ProjectDir`
Там создать и перейти в папку build `mkdir build && cd build`
Выполнить ```cmake ..```
После будет выполнена генерация файлов систем сборки таких как `make` или `ninja`
Чтобы сгенерировать файлы сборки для определенной системы сборки например для `ninja` нужно выполнить `cmake ..-G=Ninja`
Чтобы сгенерировать файлы сборки для утилиты `make` нужно выполнить `cmake .. -G="Unix Makefiles"`
Чтобы начать сборку проекта нужно выполнить `ninja -j$(nproc)` или `make -j$(nproc)` для ninja[^1]
Чтобы запустить исполняемый файл нужно выполнить `./<название_исполняемого_файла>`
Название исполняемого файла можно узнать посмотрев в файл CMakeLists.txt на строчку ```project()```

[^1]: Лично я предпочитаю ninja потому что он быстрее собирает


## Простейший CMakeLists.txt для сборки проекта на C++
```cmake
cmake_minimum_required(VERSION 3.20)
project(<Название проекта>)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_executable(<Название проекта> main.cpp)
```

## Как использовать [clang-format](https://clang.llvm.org/docs/ClangFormat.html) - форматтер кода
```sh
clang-format -style=google -dump-config > .clang-format
clang-format -i *.cpp #если форматируете руками, не в редакторе кода
```

Обычно форматировать код в ручную не нужно этим занимается [vscode](https://marketplace.visualstudio.com/items?itemName=xaver.clang-format) и все популярные ide по сохранению файла

## CMake и статические библиотеки

Бывает так что вам нужно какую то часть вашего кода использовать как статическую библиотеку.
Чтобы это было удобно сделать можно создать примерно такую структуру файлов:
```
|---src
|    \
|     CMakeLists.txt
|      main.cpp
|---mystaticlib
|	\
|	 CMakeLists.txt
|	  MyStaticLibrary.cpp
|	  MyStaticLibrary.hpp
|
|---CMakeLists.txt
```

`src` содержит `main.cpp` в код которого будет включаться статически линкующуюся библиотека из поддиректории mystaticlib

`CMakeLists.txt` в находящийся в корне будет выглядеть примерно так:

```cmake
cmake_minimum_required (VERSION 3.20)
project (Example)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # это нужно чтобы ланг сервер в vscode мог лучше понять написаный вами код

add_subdirectory (mystaticlib)
add_subdirectory (src)
```
`CMakeLists.txt` в находящийся в папке `mystatilib` будет выглядеть примерно так:
```cmake
cmake_minimum_required (VERSION 3.20)
 
project(mystatilib)
 
set(SOURCE_FILES "MyStaticLibrary.cpp")
set(HEADER_FILES "MyStaticLibrary.h")
 
# Мы объявляем проект как статическую библиотеку и добавляем в нее все файлы исходного кода.
add_library(MyStaticLibrary STATIC ${HEADER_FILES} ${SOURCE_FILES})
```
`CMakeLists.txt` в находящийся в папке `src` будет выглядеть примерно так:
```cmake
cmake_minimum_required (VERSION 3.20)
 
project(exampleStatic)
 
set(SOURCE_FILES "main.cpp")
 
add_executable (exampleStatic ${SOURCE_FILES})
 
# Подключаем библиотеку, указываем откуда брать заголовочные файлы
include_directories("../MyStaticLibrary")
# А также указываем зависимость от статической библиотеки
target_link_libraries(exampleStatic mystatilib)
```

Документация по [cmake](https://cmake.org/documentation/)