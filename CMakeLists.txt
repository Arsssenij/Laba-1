# Устанавливаем минимально поддерживаемую версию CMake
cmake_minimum_required(VERSION 3.10)

# Устанавливаем переменную с номером патча и версию проекта
set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

# Инициализируем проект с названием и версией
project(helloworld VERSION ${PROJECT_VERSION})

# Ищем библиотеку Boost с компонентом unit_test_framework
find_package(Boost COMPONENTS unit_test_framework REQUIRED)

# Генерируем файл версии из шаблона version.h.in
configure_file(version.h.in version.h)

# Создаем исполняемый файл helloworld_cli на основе main.cpp
add_executable(helloworld_cli main.cpp)

# Создаем библиотеку helloworld на основе lib.cpp
add_library(helloworld lib.cpp)

# Создаем исполняемый файл для тестов на основе test_version.cpp
add_executable(test_version test_version.cpp)

# Устанавливаем стандарт C++14 для всех целей и включаем строгие требования
set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

# Указываем путь к сгенерированному заголовочному файлу
target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)

# Настройки для тестов: использование динамической линковки Boost
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

# Связываем исполняемый файл helloworld_cli с библиотекой helloworld
target_link_libraries(helloworld_cli PRIVATE
    helloworld
)

# Связываем тестовый файл с библиотекой Boost и helloworld
target_link_libraries(test_version
    ${Boost_LIBRARIES}
    helloworld
)

# Устанавливаем флаги компилятора в зависимости от системы:
if (MSVC)
    # Для Visual Studio: максимальные предупреждения (/W4)
    target_compile_options(helloworld_cli PRIVATE
        /W4
    )
    target_compile_options(helloworld PRIVATE
        /W4
    )
    target_compile_options(test_version PRIVATE
        /W4
    )
else ()
    # Для GCC/Clang: включаем все предупреждения и педантичность
    target_compile_options(helloworld_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()

# Устанавливаем helloworld_cli в каталог bin при инсталляции
install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

# Включаем CPack для создания установочных пакетов
include(CPack)

# Настраиваем генерацию пакетов в зависимости от операционной системы:
if(APPLE)
    # Для macOS используем productbuild для создания .pkg
    set(CPACK_GENERATOR "productbuild")
    set(CPACK_PACKAGE_FILE_NAME "helloworld-${CMAKE_PROJECT_VERSION}-macOS")
    set(CPACK_PRODUCTBUILD_IDENTITY "Developer ID Installer")  # Указываем идентификатор разработчика для подписывания
elseif(UNIX)
    # Для Linux используем генерацию DEB пакетов
    set(CPACK_GENERATOR "DEB")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")  # Указываем имя мейнтейнера пакета
    set(CPACK_PACKAGE_FILE_NAME "helloworld-${CMAKE_PROJECT_VERSION}-Linux")
endif()

# Общие настройки пакета: версия и контактная информация
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_NAME "helloworld")

# Добавляем контактные данные для пакета
set(CPACK_PACKAGE_CONTACT example@example.com)

# Включаем поддержку модульного тестирования
enable_testing()

# Добавляем тест test_version, который будет выполнен при запуске тестов
add_test(test_version test_version)
