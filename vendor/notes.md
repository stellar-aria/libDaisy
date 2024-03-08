# Driver Notes

This can be used to keep track of any manual changes to the HAL or CMSIS files to make migrating to new versions easier in the future.
These are implemented in the file in this folder called `stm32h7xx.patch`

* In Drivers/STM32H7xx_HAL_Driver/Src/stm32h7xx_ll_spi.c added and `ifndef` around `assert_param` to prevent redeclaration warning
* In Drivers/STM32H7xx_HAL_Driver/Src/stm32h7xx_hal.c, `__HAL_RCC_AHB3_FORCE_RESET()` and its release (in `HAL_DeInit`) are commented out to avoid a system reset in the bootloader

# Middleware Notes

This can be used to keep track of any manual changes to Middleware files to make migrating to new versions easier in the future.

* Middlewares/ST/STM32_USB_Host_Library/Class/MSC/Src/usbh_msc.c
  * The msc class was modified to prevent dynamic allocation of the `MSC_HandleTypeDef` struct. It was also placed in uncached D2 ram to allow DMA transfers with D cache enabled.
  * modified again on 18 April 2022 to temporarily remove USBH_Free from usbh class -- this should be done for device classes, and/or we should just rework the system to work with malloc/free as designed. That change may require moving the heap out of DTCMRAM (default location within daisy linker) if the DMA needs access to the class data
