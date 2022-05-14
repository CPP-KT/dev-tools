# Dev-tools
> :warning: **Работоспособность на чем-то кроме Ubuntu может быть нестабильной**, но мы постараемся помочь при возникновении проблем.

## Установка docker
### Linux
```
sudo apt install -y docker docker-compose bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

Проверить, что докер установился:
```
docker run hello-world
```

### macOS
Установите [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### Windows
Установите [Docker Desktop](https://docs.docker.com/desktop/windows/install/)


## Собираем или скачиваем образ

Собрать образ можно руками:
```
cd clang-docker-env
docker build --build-arg UID=$(id -u) -t clang-remote-dev .
```
Про то, зачем нужен аргумент `UID`, можно почитать ниже в гайде про интеграцию с Clion.


Можно ничего не собирать и скачать последнюю версию, загруженную нами в публичный Docker Hub:
```
docker pull lejabq/cpp-kt-clang:latest
```
И аналогично для тулчейна с gcc:
```
docker pull lejabq/cpp-kt-gcc:latest
```
Обратите внимание, что тогда образ локально у вас будет называться иначе. Все образы в системе можно посмотреть командой `docker images`.

### Используем

#### Clion:
Про интеграцию с IDE можно сразу почитать [тут](https://www.jetbrains.com/help/clion/clion-toolchains-in-docker.html#build-and-run).
- Заходим в CLion -> Settings -> Build,Execution... -> Toolchains
- Создаем новый docker тулчейн
  * image - выбираем наш image (`clang-remote-dev` или как он был назван при сборке/скачивании)
  * build-tool - `/usr/bin/ninja`
- В настройках **каждой** cmake-конфигурации нужно указать 
  - `-DCMAKE_TOOLCHAIN_FILE=/vcpkg/scripts/buildsystems/vcpkg.cmake`
  - [Только если это clang образ] `-DVCPKG_TARGET_TRIPLET=x64-linux-clang`

#### Ручной запуск:
Можно запустить шелл в контейнере и делать нужные вам действия прямо из него. \
Наверняка при этом вы хотите шарить какую-нибудь директорию вашей системы с файловой системой контейнера. \
Сделать это можно следующим образом, заменив пути и имя образа на нужные вам:
```
 docker run -t -v <path/to/local/directory>:/<directory_in_container_name> -i <image_name> /bin/bash
```
