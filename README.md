## Laboratory work IV

## Tasks

- [x] 1. Авторизоваться на сервисе **Travis CI** с использованием **GitHub** аккаунта
- [x] 2. Создать публичный репозиторий с названием **lab04** на сервисе **GitHub**
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Включить интеграцию сервиса **Travis CI** с созданным репозиторием
- [x] 5. Получить токен для **Travis CLI** с правами **repo** и **user**
- [x] 6. Получить фрагмент вставки значка сервиса **Travis CI** в формате **Markdown**
- [x] 7. Выполнить инструкцию учебного материала
- [x] 8. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
[![Build Status](https://travis-ci.com/4Rjee/lab04.svg?branch=master)](https://travis-ci.com/4Rjee/lab04)

```sh
$ export GITHUB_USERNAME=4Rjee
$ export GITHUB_TOKEN=****************************************
```

```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

```sh
# установка rvm
$ curl -sSL https://get.rvm.io | bash -s -- --ignore-dotfiles
$ echo "source $HOME/.rvm/scripts/rvm" >> scripts/activate
$ . scripts/activate
$ rvm autolibs disable
$ rvm install ruby-2.4.2
$ rvm use 2.4.2 --default
$ gem install travis
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab03 projects/lab04
$ cd projects/lab04
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab04
```

```sh
# выбираем язык
$ cat > .travis.yml <<EOF
language: cpp
EOF
```

```sh
# прописываем последовательность команд, выполняемую при сборке
$ cat >> .travis.yml <<EOF
script:
- cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
- cmake --build _build
- cmake --build _build --target install
EOF
```

```sh
# зависимости
$ cat >> .travis.yml <<EOF
addons:
  apt:
    sources:
      - george-edison55-precise-backports
    packages:
      - cmake
      - cmake-data
EOF
```

```sh
# добавляем токен для работы с travis
$ travis login --github-token ${GITHUB_TOKEN}
```

```sh
# проверка файла .travis.yml
$ travis lint
```

```sh
$ git add .travis.yml
$ git add README.md
$ git commit -m"added CI"
$ git push origin master
```

```sh
$ travis lint
# список аккаунтов
$ travis accounts
# синхронизация
$ travis sync
# список репозиториев
$ travis repos
# активируем проект в Travis CI
$ travis enable
# показывает последние действия
$ travis whatsup
# показывает последний билд каждой ветки
$ travis branches
# история билдов
$ travis history
# информация о последнем билде
$ travis show
```

## Report

```sh
$ popd
$ export LAB_NUMBER=04
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework
[![Build Status](https://travis-ci.com/4Rjee/lab04.svg?branch=master)](https://travis-ci.com/4Rjee/lab04)

Вы продолжаете проходить стажировку в "Formatter Inc." (см [подробности](https://github.com/tp-labs/lab03#Homework)).

В прошлый раз ваше задание заключалось в настройке автоматизированной системы **CMake**.

Сейчас вам требуется настроить систему непрерывной интеграции для библиотек и приложений, с которыми вы работали в [прошлый раз](https://github.com/tp-labs/lab03#Homework). Настройте сборочные процедуры на различных платформах:
* используйте [TravisCI](https://travis-ci.com/) для сборки на операционной системе **Linux** с использованием компиляторов **gcc** и **clang**;
* используйте [AppVeyor](https://www.appveyor.com/) для сборки на операционной системе **Windows**.

Настройка git-репозитория **hw04** для работы
```sh
% git clone https://github.com/4Rjee/hw03 projects/hw04
% cd projects/hw04
% git remote remove origin
% git remote add origin https://github.com/4Rjee/hw04
```
Cоздание единого `CMakeLists.txt`, по которому будет осуществляться сборка при помощи **TravisCI**
```sh
% cat >> CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.10)
project(formatter)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(formatter STATIC formatter_lib/formatter.cpp)
include_directories(formatter_lib)
add_library(formatter_ex STATIC formatter_ex_lib/formatter_ex.cpp)
include_directories(formatter_ex_lib)
add_executable(hello_world hello_world_application/hello_world.cpp)
target_link_libraries(hello_world formatter formatter_ex)
add_library(solver_lib STATIC solver_lib/solver.cpp)
include_directories(solver_lib)
add_executable(solver solver_application/equation.cpp)
target_link_libraries(solver formatter formatter_ex solver_lib)
EOF
```
Создание файла `.travis.yml`
```sh
# Установка языка программирования
# операционной системы и компиляторов
% cat > .travis.yml <<EOF
language: cpp
os:
  - osx
EOF
```
Скрипт, который выполняет команды `cmake` для сборки проектов в каждой директории
```sh
% cat >> script <<EOF
cmake formatter_lib/CMakeLists.txt -Bformatter_lib/_build -DCMAKE_CURRENT_SOURCE_DIR=$PWD
cmake --build formatter_lib/_build
cmake formatter_ex_lib/CMakeLists.txt -Bformatter_ex_lib/_build -DCMAKE_CURRENT_SOURCE_DIR=$PWD
cmake --build formatter_ex_lib/_build
cmake hello_world_application/CMakeLists.txt -Bhello_world_application/_build -DCMAKE_CURRENT_SOURCE_DIR=$PWD
cmake --build hello_world_application/_build
cmake solver_application/CMakeLists.txt -Bsolver_application/_build -DCMAKE_CURRENT_SOURCE_DIR=$PWD
cmake --build solver_application/_build
EOF
```
Установка пользовательского сценария запуска с помощью **CMake**
```sh
# Установка настроек СMake, запуск и исполнение
% cat >> .travis.yml <<EOF
jobs:
  include:
  - name: "all projects"
    script:
    - cmake -H. -B_build
    - cmake --build _build
  - name: "each CMakeLists.txt"
    script:
    - source ./script
EOF
```
Установка дополнительных исполняемых файлов и пакетов
```sh
# Дополнения конфигурирации источников и пакетов
$ cat >> .travis.yml <<EOF
addons:
  apt:
    sources:
      - george-edison55-precise-backports
    packages:
      - cmake
      - cmake-data
EOF
```
Проверка `.travis.yml` на ошибки
```sh
% travis lint
Hooray, .travis.yml looks valid :)
```

## Report
Создание отчета по ЛР № 4
```sh
$ popd
$ export LAB_NUMBER=04
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Links

- [Travis Client](https://github.com/travis-ci/travis.rb)
- [AppVeyour](https://www.appveyor.com/)
- [GitLab CI](https://about.gitlab.com/gitlab-ci/)

```
Copyright (c) 2015-2020 The ISC Authors
```
