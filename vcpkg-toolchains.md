## Вступление

Под "**тулчейном**" стоит понимать компиляторы (для C и C++), какую-то (не обязательно поставляемую с этим компилятором стандартную библиотеку), а также набор 
утилит, поставляемых с компилятором. 
Под "**тулчейном по умолчанию**" давайте понимать тот тулчейн, который выбирает cmake, когда ему не передают никаких параметров, вроде `CMAKE_CXX_COMPILER` 
или `CMAKE_TOOLCHAIN_FILE`. Скорее всего, это что-то вроде `/usr/bin/c++`.

## Проблема

Вам захотелось собрать свой проект под другим компилятором и/или стандартной библиотекой. VCPKG с тулчейном не по умолчанию дружит плохо: скорее всего, 
 без хаков вроде (`CC='/usr/bin/clang' CXX='/usr/bin/clang++ -stdlib=libc++'`) вы не сможете заставить vcpkg собирать зависимости под нужным тулчейном. 
 Хочется универсального красивого решения, не так ли?
 
## Причины 

#### CMAKE

Если выставляется переменная `CMAKE_TOOLCHAIN_FILE` (а это наш случай!), то всякие настройки тулчейна, вроде `CMAKE_CXX_COMPILER`, попросту игнорируются. 
Используется тулчейн по умолчанию.

#### VCPKG

У нас следующий набор слоёв (от вызывающего к вызываемому):
- `cmake`, который вы запускаете;
- `vcpkg` - бинарь, который запускается в случае необходимости установки пакетов в manifest mode, в обычном режиме запускается пользователем;
- `cmake`, который запускается при установке пакета (для его конфигурации и сборки).

И это приводит нас к проблеме: между запусками `cmake` нет передачи тулчейна. Это является проблемой в обе стороны. 
- (Вверх) То есть даже если вы делаете [триплет-файл](https://vcpkg.readthedocs.io/en/latest/users/triplets/), в котором укажете 
`VCPKG_CHAINLOAD_TOOLCHAIN_FILE`, то верхний `cmake` этот тулчейн-файл не увидит и будет собирать ваш код тулчейном по умолчанию.
- (Вниз) Если вы укажете `VCPKG_CHAINLOAD_TOOLCHAIN_FILE` при конфигурации вашего проекта (и не укажете в триплете), то зависимости будут 
собираться тулчейном по умолчанию.

## Решение

Следующие шаги нужно повторить для каждого стороннего тулчейна. Я покажу конфигурацию на примере clang-тулчейна с `libc++` для linux.

#### Тулчейн-файл
Внутри `$VCPKG_HOME/triplets` создаётся `clang-toolchain.cmake`:
```
set(CMAKE_C_COMPILER /usr/bin/clang)
set(CMAKE_CXX_COMPILER /usr/bin/clang++)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
```
Можно написать через `find_program`, но давайте не усложнять примеры. Если хотите вдохновения, то [вот](https://github.com/Neumann-A/my-vcpkg-triplets/blob/master/x64-windows-llvm.toolchain-no-lto.cmake)
отличный репозиторий.

#### Триплет
Внутри `$VCPKG_HOME/triplets` создаётся `x64-linux-clang.cmake`:
```
set(VCPKG_TARGET_ARCHITECTURE x64)
set(VCPKG_CRT_LINKAGE dynamic)
set(VCPKG_LIBRARY_LINKAGE static)

set(VCPKG_CMAKE_SYSTEM_NAME Linux)

set(VCPKG_CHAINLOAD_TOOLCHAIN_FILE "${CMAKE_CURRENT_LIST_DIR}/clang-toolchain.cmake")
```

#### Используем в CLion

Создаётся новая cmake-конфигурация с переменными:
* `-DCMAKE_TOOLCHAIN_FILE=/Your/Path/To/vcpkg/scripts/buildsystems/vcpkg.cmake`;
* `-DVCPKG_TARGET_TRIPLET=x64-linux-clang` - выставляем только что созданный триплет;
* `-DVCPKG_CHAINLOAD_TOOLCHAIN_FILE=/Your/Path/To/vcpkg/triplets/clang-toolchain.cmake` - выставляем наш тулчейн.
* [Если надо, можно использовать с пресетами] `--preset SanitizedDebug` 
