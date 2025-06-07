cmake_minimum_required(VERSION 3.4)

# Включаем поддержку C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Основной проект
project(banking_app)

# Версия приложения
set(BANKING_VERSION_MAJOR 0)
set(BANKING_VERSION_MINOR 1)
set(BANKING_VERSION_PATCH 0)
set(BANKING_VERSION_TWEAK 0)
set(BANKING_VERSION "${BANKING_VERSION_MAJOR}.${BANKING_VERSION_MINOR}.${BANKING_VERSION_PATCH}.${BANKING_VERSION_TWEAK}")
set(CPACK_PACKAGE_VERSION "${BANKING_VERSION}")
set(BANKING_VERSION_STRING "v${BANKING_VERSION}")

# Добавляем пути к модулям CMake
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/third-party/CMake-codecov/cmake" ${CMAKE_MODULE_PATH})

# Подключаем исходники банковского приложения
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/banking)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/banking)

# Тестирование
if (BUILD_TESTS)
  # Подключаем Google Test
  add_subdirectory(third-party/gtest)
  enable_testing()

  include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
  include_directories(${gmock_SOURCE_DIR}/include ${gmock_SOURCE_DIR})

  # Тестовые файлы
  file(GLOB ACCOUNT_TEST_SOURCES tests/*.cpp)
  add_executable(check ${ACCOUNT_TEST_SOURCES})
  target_link_libraries(check gtest transaction gtest_main account gmock)
  add_test(NAME check COMMAND check)

  # Покрытие кода (если включено)
  if (BUILD_COVERAGE)
    set(ENABLE_COVERAGE ON CACHE BOOL "Enable coverage build." FORCE)
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    find_package(codecov)
    add_coverage(check)
    list(APPEND LCOV_REMOVE_PATTERNS "'${PROJECT_SOURCE_DIR}/tests/*'")
    coverage_evaluate()
  endif ()
endif ()

# Настройки CPack — генерация пакетов
set(CPACK_PACKAGE_NAME "banking_app")
set(CPACK_PACKAGE_DESCRIPTION "Banking application built with CMake and CPack")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A simple banking application for lab work")
set(CPACK_PACKAGE_VENDOR "YourCompany")
set(CPACK_PACKAGE_CONTACT "your.email@example.com")
set(CPACK_PACKAGE_VERSION_MAJOR ${BANKING_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${BANKING_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${BANKING_VERSION_TWEAK})

# Лицензия и группа для RPM
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Applications/Engineering")
set(CPACK_RPM_PACKAGE_ARCHITECTURE x86_64)

# Форматы пакетов: можно оставить только нужные
set(CPACK_GENERATOR "TGZ;DEB;RPM")

# Убираем или создаём ChangeLog.md, чтобы не было ошибок
# set(CPACK_RPM_CHANGELOG_FILE "${PROJECT_SOURCE_DIR}/ChangeLog.md")  # раскомментировать, если файл существует

# Включаем CPack
include(CPack)
