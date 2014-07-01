CMAKE_MINIMUM_REQUIRED(VERSION 2.6 FATAL_ERROR)
CMAKE_POLICY(VERSION 2.6)

set(CMAKE_C_FLAGS "-Wno-unsequenced -Wno-unused-result -std=gnu99")

FIND_PACKAGE(Torch REQUIRED)
FIND_PACKAGE(JPEG QUIET)
FIND_PACKAGE(PNG QUIET)

SET(src ppm.c)
ADD_TORCH_PACKAGE(ppm "${src}" "${luasrc}" "Image Processing")
TARGET_LINK_LIBRARIES(ppm luaT TH)

IF (APPLE) 
    # Apple platform: our fmemopen.c provides fmemopen
    SET(FMEMOPEN_FOUND TRUE)
ELSE (APPLE)
    # Non-Apple: linux has fmemopen, windows doesn't, check it the hard way
    INCLUDE (CheckCSourceCompiles)
    CHECK_C_SOURCE_COMPILES("
#include <stdio.h>

int main() {
    char * buf = NULL;
    fmemopen(buf, 0, \"r\");
    return 0;
}
" FMEMOPEN_FOUND)
ENDIF (APPLE)

IF (FMEMOPEN_FOUND)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DFMEMOPEN_FOUND")
ENDIF(FMEMOPEN_FOUND)

if (JPEG_FOUND)
    SET(src jpeg.c fmemopen.c)
    include_directories (${JPEG_INCLUDE_DIR})
    ADD_TORCH_PACKAGE(jpeg "${src}" "${luasrc}" "Image Processing")
    TARGET_LINK_LIBRARIES(jpeg luaT TH ${JPEG_LIBRARIES})
else (JPEG_FOUND)
    message ("WARNING: Could not find JPEG libraries, JPEG wrapper will not be installed")
endif (JPEG_FOUND)

if (PNG_FOUND)
    SET(src png.c)
    include_directories (${PNG_INCLUDE_DIR})
    ADD_TORCH_PACKAGE(png "${src}" "${luasrc}" "Image Processing")
    TARGET_LINK_LIBRARIES(png luaT TH ${PNG_LIBRARIES})

    SET(src libcompress.c)
    include_directories (${PNG_INCLUDE_DIR})
    ADD_TORCH_PACKAGE(compress "${src}" "${luasrc}" "Image Processing")
    TARGET_LINK_LIBRARIES(compress luaT TH ${PNG_LIBRARIES})
else (PNG_FOUND)
    message ("WARNING: Could not find PNG libraries, PNG wrapper will not be installed")
endif (PNG_FOUND)

SET(src image.c)
SET(luasrc init.lua lena.jpg lena.png lena_full.jpg lena_full.png win.ui)

ADD_TORCH_PACKAGE(image "${src}" "${luasrc}" "Image Processing")
TARGET_LINK_LIBRARIES(image luaT TH)
