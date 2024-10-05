cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VESRION})

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

configure_file(version.h.in version.h)

add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
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

# Установки для CPack
include(CPack)

# Проверяем операционную систему
if(APPLE)
    # Для macOS используем productbuild для создания .pkg
    set(CPACK_GENERATOR "productbuild")
    set(CPACK_PACKAGE_FILE_NAME "helloworld-${CMAKE_PROJECT_VERSION}-macOS")
    set(CPACK_PRODUCTBUILD_IDENTITY "Developer ID Installer")  # Указать, если используешь код-подпись
elseif(UNIX)
    # Для Linux используем DEB пакеты
    set(CPACK_GENERATOR "DEB")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")  # Установи твоё имя
    set(CPACK_PACKAGE_FILE_NAME "helloworld-${CMAKE_PROJECT_VERSION}-Linux")
endif()

# Общие настройки для всех ОС
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

set(CPACK_PACKAGE_NAME "helloworld")

# Добавляем CPack
include(CPack)

# Добавляем контактные данные для пакета
set(CPACK_PACKAGE_CONTACT example@example.com)

# Включаем поддержку модульного тестирования
enable_testing()

# Добавляем тест test_version, который будет выполнен при запуске тестов
add_test(test_version test_version)