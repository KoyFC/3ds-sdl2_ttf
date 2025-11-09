# 3DS SDL2_ttf (v2.24.0)

A tutorial to port **SDL2_ttf 2.24.0** to the **Nintendo 3DS** using the **devkitPro** toolchain.
This library allows rendering of TrueType (TTF) fonts through SDL2 and FreeType on 3DS.

---

## Acknowledgements

Special thanks to all the repositories and projects that made this possible:

- [**libsdl-org/SDL_ttf**](https://github.com/libsdl-org/SDL_ttf): the original SDL_ttf library.
- [**devkitPro**](https://devkitpro.org/): for the official Nintendo 3DS toolchain and portlibs.
<!-- - [**gradylink/3ds-sdl2-packages**](https://github.com/gradylink/3ds-sdl2-packages): for the initial SDL2 3DS package used as a base when rebuilding dependencies. -->

---

## Table of Contents

1. [Requirements](#requirements)

   - [Base tools](#base-tools)
   - [devkitPro](#devkitpro)
   - [Required dependencies](#required-dependencies)

2. [Installing SDL2](#installing-sdl2)
3. [Building SDL2_ttf 2.24.0](#building-sdl2_ttf-2240)

   - [Build automatically via makepkg](#option-1-build-automatically-via-makepkg)
   - [Install the generated package manually](#option-2-install-the-generated-package-manually)
   - [Manual compilation](#manual-compilation)

4. [Installing manually](#installing-manually)
5. [Usage in a Makefile](#usage-in-a-makefile)
6. [Example usage](#example-usage)

---

## Requirements

Ensure you have the following installed:

### Base tools

#### MSYS2 (Windows)

```bash
pacman -S base-devel git cmake
```

#### Linux (Debian/Ubuntu)

```bash
sudo apt install build-essential git cmake
```

#### macOS (Homebrew)

```bash
brew install cmake git
```

### devkitPro

You will need **devkitPro** installed to build and link this library. Follow the official guide here:
[https://devkitpro.org/wiki/Getting_Started](https://devkitpro.org/wiki/Getting_Started)

Then execute:

```bash
export DEVKITARM=$DEVKITPRO/devkitARM
export PATH=$DEVKITARM/bin:$PATH
```

### Required dependencies

Install the portlibs:

```bash
(dkp-)pacman -S 3ds-freetype 3ds-libpng 3ds-zlib
```

---

## Installing SDL2

To build SDL2 for 3DS, clone the official repository and compile it using devkitPro’s 3DS toolchain:

```bash
git clone https://github.com/libsdl-org/SDL.git
cd SDL
git checkout SDL2
mkdir build && cd build
cmake -DCMAKE_TOOLCHAIN_FILE=$DEVKITPRO/cmake/3DS.cmake ..
make -j4
make install
```

This installs SDL2 into `/opt/devkitpro/portlibs/3ds/`.

---

## Building SDL2_ttf

<!-- > **WARNING:** If the SDL2_ttf build fails because SDL2 cannot be found as a dependency (which is very likely, since it would mean you had already done this step), install the prebuilt SDL2 package from [gradylink/3ds-sdl2-packages/releases](https://github.com/gradylink/3ds-sdl2-packages/releases) using: -->

### Manual compilation

```bash
git clone https://github.com/libsdl-org/SDL_ttf.git
cd SDL_ttf
git checkout release-2.24.0
rm -rf Xcode
mkdir build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE="$DEVKITPRO/cmake/3DS.cmake" -DCMAKE_BUILD_TYPE=Release
cmake --build . -- -j4
```

The resulting static library will be located in `build/libSDL2_ttf.a`.

If cmake was able to link the library properly displaying the following text, ignore the errors and follow through with the installation in the next section

```bash
[100%] Linking C executable [file].elf
```

You can also use `cmake --install . --prefix "$DEVKITPRO/portlibs/3ds"` and skip the next section.

---

## Installing the library

Navigate to the build directory if you're not already in it.

#### Linux/macOS

```bash
sudo cp libSDL2_ttf.a /opt/devkitpro/portlibs/3ds/lib/
sudo cp ../SDL_ttf.h /opt/devkitpro/portlibs/3ds/include/SDL2/
```

#### Windows (MSYS2)

```bash
cp libSDL2_ttf.a /opt/devkitpro/portlibs/3ds/lib/
cp ../SDL_ttf.h /opt/devkitpro/portlibs/3ds/include/SDL2/
```

Verify that files exist:

```
/opt/devkitpro/portlibs/3ds/lib/libSDL2_ttf.a
/opt/devkitpro/portlibs/3ds/include/SDL2/SDL_ttf.h
```

---

## Usage in a Makefile

```make
CFLAGS  += $(shell sdl2-config --cflags)
LIBS    += $(shell sdl2-config --libs) -lSDL2_ttf -lfreetype -lbz2 -lpng -lz -lcitro2d -lcitro3d -lctru -lm
```

> The order matters: `-lSDL2_ttf` must come **before** `-lfreetype`.

---

## Example usage

```c
#include <SDL2/SDL.h>
#include <SDL2/SDL_ttf.h>

int main(int argc, char* args[]) {
    SDL_Init(SDL_INIT_VIDEO);
    TTF_Init();

    TTF_Font* font = TTF_OpenFont("romfs:/fonts/font.ttf", 16);
    SDL_Color color = {255, 255, 255, 255};
    SDL_Surface* textSurface = TTF_RenderText_Solid(font, "Hello 3DS!", color);

    SDL_FreeSurface(textSurface);
    TTF_CloseFont(font);
    TTF_Quit();
    SDL_Quit();
    return 0;
}
```
