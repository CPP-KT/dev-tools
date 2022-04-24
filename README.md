# Dev-tools

## Dev environments: Ubuntu/Clang, Ubuntu/GNU

Про интеграцию с IDE можно сразу почитать [тут](https://www.jetbrains.com/help/clion/clion-toolchains-in-docker.html#build-and-run).

### Собираем образ

```
  $ cd clang-docker-env
  $ docker build --build-arg UID=$(id -u) -t clang-remote-dev .
```

Про аргумент `UID` можно почитать в гайде выше.

### Используем

- Заходим в CLion -> Settings -> Build,Execution... -> Toolchains
- Создаем новый docker тулчейн
  * image - выбираем наш image (`clang-remote-dev` или как он был назван при сборке)
  * build-tool - `/usr/bin/ninja`
- В настройках **каждой** cmake-конфигурации нужно указать `-DCMAKE_TOOLCHAIN_FILE=/vcpkg/scripts/buildsystems/vcpkg.cmake`
