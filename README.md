# MIPI DCS HAL for HAGL Graphics Library

HAL for HAGL graphics library for display drivers supporting the [MIPI Display Command Set](https://www.mipi.org/specifications/display-command-set) standard. This covers most displays currently used by hobbyists. Tested with ST7789V (Sipeed M1 Dock Suit).

[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

## Usage

To use with a K210 Standalon SDK project you include this HAL and the [HAGL graphics library](https://github.com/tuupola/hagl) itself. For example applications see [Graphics Speed Tests](https://github.com/tuupola/k210_gfx.git).

```
$ mkdir external
$ cd external
$ git submodule add https://github.com/tuupola/hagl_k210_mipi.git hagl_hal
$ git submodule add https://github.com/tuupola/hagl.git
```

Then in your `CMakeLists.txt` include both libraries in the build.

```
add_subdirectory(external/hagl)
add_subdirectory(external/hagl_hal)
target_link_libraries(firmware hagl hagl_hal)

```

By default the HAL uses single buffering. The buffer is the GRAM of the display driver chip. You can enable double buffering with the following.

```
target_compile_definitions(firmware PRIVATE
  HAGL_HAL_USE_DOUBLE_BUFFER
)
```

By default flushing from back buffer to front buffer is a locking operation. You can avoid that by enabling DMA. This is faster but will cause screen tearing unless you handle vertical syncing in the software.

**HEADS UP!** DMA support is currently untested and actually seems to be slower than not using DMA.

```
target_compile_definitions(firmware PRIVATE
  HAGL_HAL_USE_DOUBLE_BUFFER
  HAGL_HAL_USE_DMA
)
```

Alternatively you can also use triple buffering. This is the fastest and will not have screen tearing with DMA. Downside is that it uses lot of memory.

**HEADS UP!** DMA support is currently untested and actually seems to be slower than not using DMA.

```
target_compile_definitions(firmware PRIVATE
  HAGL_HAL_USE_TRIPLE_BUFFER
  HAGL_HAL_USE_DMA
)
```

The default config can be found in `hagl_hal.h`. Defaults are ok for [Sipeed M1 Dock Suit](https://www.seeedstudio.com/Sipeed-M1-dock-suit-M1-dock-2-4-inch-LCD-OV2640-K210-Dev-Board-1st-RV64-AI-board-for-Edge-Computing.html) in vertical mode.

## Configuration

You can override any of the default settings setting in `CMakeLists.txt`. You only need to override a value if default is not ok. Below example shows all the possible overridable values. These are also the default values.

```
target_compile_definitions(firmware PRIVATE
  MIPI_DISPLAY_SPI_CLOCK_SPEED_HZ=65000000
  MIPI_DISPLAY_PIN_CS=36
  MIPI_DISPLAY_PIN_DC=38
  MIPI_DISPLAY_PIN_RST=37
  MIPI_DISPLAY_PIN_BL=-1
  MIPI_DISPLAY_PIN_CLK=39
  MIPI_DISPLAY_PIN_MOSI=-1
  MIPI_DISPLAY_PIN_MISO=-1
  MIPI_DISPLAY_PIXEL_FORMAT=MIPI_DCS_PIXEL_FORMAT_16BIT
  MIPI_DISPLAY_ADDRESS_MODE=MIPI_DCS_ADDRESS_MODE_RGB
  MIPI_DISPLAY_WIDTH=240
  MIPI_DISPLAY_HEIGHT=320
  MIPI_DISPLAY_OFFSET_X=0
  MIPI_DISPLAY_OFFSET_Y=0
  MIPI_DISPLAY_INVERT=0
)
```

`MIPI_DISPLAY_ADDRESS_MODE` controls the orientation and the RGB order of the display. The value is a bit field which can consist of the following flags defined in `mipi_dcs.h`.

```
#define MIPI_DCS_ADDRESS_MODE_MIRROR_Y      0x80
#define MIPI_DCS_ADDRESS_MODE_MIRROR_X      0x40
#define MIPI_DCS_ADDRESS_MODE_SWAP_XY       0x20
#define MIPI_DCS_ADDRESS_MODE_BGR           0x08
#define MIPI_DCS_ADDRESS_MODE_RGB           0x00
#define MIPI_DCS_ADDRESS_MODE_FLIP_X        0x02
#define MIPI_DCS_ADDRESS_MODE_FLIP_Y        0x01
```

You should `OR` together the flags you want to use. For example if you have a 240x320 pixel display on vertical mode.

```
target_compile_definitions(firmware PRIVATE
  MIPI_DISPLAY_WIDTH=240
  MIPI_DISPLAY_HEIGHT=320
)
```

You can change it to 320x240 pixel vertical mode by swapping the X and Y coordinates. You will also need to mirror Y axis.

```
target_compile_definitions(firmware PRIVATE
  MIPI_DISPLAY_ADDRESS_MODE=MIPI_DCS_ADDRESS_MODE_SWAP_XY|MIPI_DCS_ADDRESS_MODE_MIRROR_Y
  MIPI_DISPLAY_WIDTH=320
  MIPI_DISPLAY_HEIGHT=240
)
```

You can `OR` together as many flags as you want. Not all combinations make sense but any display orientation can be achieved with correct combination of the flags. When in doubt just try different combinations.

## Common problems

If red and blue are mixed but green is ok change to BGR mode.

```
target_compile_definitions(firmware PRIVATE
  MIPI_DISPLAY_ADDRESS_MODE=MIPI_DCS_ADDRESS_MODE_BGR
)
```

If display seems inverted turn on the inversion.

```
target_compile_definitions(firmware PRIVATE
  MIPI_DISPLAY_INVERT
)
```

If image is not center on screen adjust X and Y offsets as needed.

```
target_compile_definitions(firmware PRIVATE
  MIPI_DISPLAY_OFFSET_X=25
  MIPI_DISPLAY_OFFSET_Y=30
)
```

## Speed

Below testing was done with Sipeed M1 Dock Suit with 320x240x16 display clocked at 65MHz. Double buffering display refresh rate was set to 30 frames per second. Number represents operations per seconsd ie. bigger number is better.

|                               | Single | Double    | Double DMA | Triple DMA |
|-------------------------------|--------|-----------|------------|------------|
| hagl_put_pixel()              |  87373 |    860130 |            |            |
| hagl_draw_line()              |    739 |     20137 |            |            |
| hagl_draw_vline()             |  35914 |     78627 |            |            |
| hagl_draw_hline()             |  34542 |    165891 |            |            |
| hagl_draw_circle()            |    978 |     29084 |            |            |
| hagl_fill_circle()            |   1293 |     10360 |            |            |
| hagl_draw_ellipse()           |    551 |     14371 |            |            |
| hagl_fill_ellipse()           |    541 |      3610 |            |            |
| hagl_draw_triangle()          |    246 |      6930 |            |            |
| hagl_fill_triangle()          |    376 |      1992 |            |            |
| hagl_draw_rectangle()         |   9216 |     28119 |            |            |
| hagl_fill_rectangle()         |    437 |      2187 |            |            |
| hagl_draw_rounded_rectangle() |   2558 |     24276 |            |            |
| hagl_fill_rounded_rectangle() |    398 |      2073 |            |            |
| hagl_fill_polygon()           |    231 |      1166 |            |            |
| hagl_put_char()               |        |           |            |            |

## License

The MIT License (MIT). Please see [LICENSE](LICENSE) for more information.
