add_executable(freertos_ampelsteuerung
  main.cpp 
  )

# Add pico_stdlib library which aggregates commonly used features
target_link_libraries(freertos_ampelsteuerung pico_stdlib)
pico_enable_stdio_usb(freertos_ampelsteuerung 1)
pico_enable_stdio_uart(freertos_ampelsteuerung 0)

# create map/bin/hex/uf2 file in addition to ELF.
pico_add_extra_outputs(freertos_ampelsteuerung)

target_link_libraries(freertos_ampelsteuerung FreeRTOS pico-cpp)
