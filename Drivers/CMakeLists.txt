# CMSIS Core
FetchContent_Declare(cmsis
  URL https://github.com/ARM-software/CMSIS_5/releases/download/5.9.0/ARM.CMSIS.5.9.0.pack
  URL_HASH MD5=633d69b410dc79065e17bc17a3296cb8
)
FetchContent_MakeAvailable(cmsis)

add_library(CMSIS_CORE INTERFACE)
target_include_directories(CMSIS_CORE INTERFACE
  ${cmsis_SOURCE_DIR}/CMSIS/Core/Include
)

# CMSIS Device
FetchContent_Declare(cmsis_device_h7
  URL https://github.com/STMicroelectronics/cmsis_device_h7/archive/refs/tags/v1.10.3.tar.gz
  URL_HASH MD5=f00ceaa34878335797b5604f14706b3c
)
FetchContent_MakeAvailable(cmsis_device_h7)

add_library(CMSIS_DEVICE_H7 STATIC)
target_include_directories(CMSIS_DEVICE_H7 PUBLIC
  ${cmsis_device_h7_SOURCE_DIR}/Include
)
target_sources(CMSIS_DEVICE_H7 PRIVATE
  ${MODULE_DIR}/sys/system_stm32h7xx.c
)
target_link_libraries(CMSIS_DEVICE_H7 PUBLIC
  CMSIS_CORE
  STM32H7XX_HAL_DRIVER
)

# STM32H7xx HAL Driver
FetchContent_Declare(stm32h7xx_hal_driver
  URL https://github.com/STMicroelectronics/stm32h7xx_hal_driver/archive/refs/tags/v1.11.2.tar.gz
  URL_HASH MD5=67402ec717e640e8431387ee020aacf9

  # NOTE: In-tree source patch until fixed upstream (volatile counters/loop unrolling)
  PATCH_COMMAND git apply --ignore-space-change --ignore-whitespace ${CMAKE_CURRENT_SOURCE_DIR}/stm32h7xx.patch
)
FetchContent_MakeAvailable(stm32h7xx_hal_driver)

file(GLOB STM32H7XX_HAL_DRIVER_SOURCES ${stm32h7xx_hal_driver_SOURCE_DIR}/Src/*.c) # All .c files
list(FILTER STM32H7XX_HAL_DRIVER_SOURCES EXCLUDE REGEX "template.c$") # unless they're templates

add_library(STM32H7XX_HAL_DRIVER STATIC ${STM32H7XX_HAL_DRIVER_SOURCES})

target_include_directories(STM32H7XX_HAL_DRIVER PUBLIC
  ${stm32h7xx_hal_driver_SOURCE_DIR}/Inc
  ${stm32h7xx_hal_driver_SOURCE_DIR}/Inc/Legacy
  ${MODULE_DIR}/sys # for stm32h7xx_hal_conf.h
)

# Weird Circular dependency (HAL -> CMSIS-Device -> HAL)
target_link_libraries(STM32H7XX_HAL_DRIVER PUBLIC CMSIS_DEVICE_H7)

target_compile_options(STM32H7XX_HAL_DRIVER PRIVATE
    -Wno-unused-but-set-variable
)

target_compile_definitions(STM32H7XX_HAL_DRIVER PRIVATE
  USE_HAL_DRIVER
  USE_FULL_LL_DRIVER
)
