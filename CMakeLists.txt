add_library(hagl_hal INTERFACE)

target_include_directories(hagl_hal INTERFACE ${CMAKE_CURRENT_LIST_DIR}/include)

target_sources(hagl_hal INTERFACE
  ${CMAKE_CURRENT_LIST_DIR}/mipi_display.c
  ${CMAKE_CURRENT_LIST_DIR}/hagl_hal_single.c
  ${CMAKE_CURRENT_LIST_DIR}/hagl_hal_double.c
  ${CMAKE_CURRENT_LIST_DIR}/hagl_hal_triple.c
)
