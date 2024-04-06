#
# project: dcss
#
cmake_minimum_required(VERSION 3.21)
project(dcss-webasm)

set(CMAKE_CXX_STANDARD 11)
set(CRAWL_SOURCE crawl/crawl-ref/source)
set(CRAWL_CONTRIB ${CRAWL_SOURCE}/contrib)
set(BUILD_SHARED_LIBS false)
set(CMAKE_CXX_FLAGS "       \
    ${CMAKE_CXX_FLAGS}      \
    -fexceptions            \
    -frtti                  \
    -sMAX_WEBGL_VERSION=2   \
    -sUSE_GLFW=3            \
    --use-port=freetype     \
    --use-port=sdl2         \
    --use-port=sdl2_image   \
    --use-port=sqlite3      \
    --use-port=zlib"
)

# include the fips main cmake file
get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
include("${FIPS_ROOT_DIR}/cmake/fips.cmake")

fips_setup()

add_compile_options(-w)
fips_begin_app(dcss windowed)

# lua
set(LUA_SOURCE_ROOT ${CRAWL_CONTRIB}/lua/src)
file(GLOB LUA_FILES ${LUA_SOURCE_ROOT}/*.c)
add_library(lua STATIC ${LUA_FILES})
add_dependencies(dcss lua)
target_link_libraries(dcss lua)
target_include_directories(dcss PRIVATE ${LUA_SOURCE_ROOT} NO_RECURSE)

set(RLTILES_SOURCE_ROOT ${CRAWL_SOURCE}/rltiles)
target_include_directories(dcss PRIVATE ${CRAWL_SOURCE} ${RLTILES_SOURCE_ROOT} NO_RECURSE)
target_link_options(dcss PRIVATE
    -O2 # this is needed because using asyncify with lesser optimizations causes crashes at runtime, see https://github.com/emscripten-core/emscripten/issues/19346
)
target_compile_definitions(dcss PRIVATE
    ASSERTS
    CLUA_BINDINGS
    MONOSPACED_FONT=\"./fonts/DejaVuSansMono.ttf\"
    PROPORTIONAL_FONT=\"./fonts/DejaVuSans.ttf\"
    TOUCH_UI
    USE_FT
    USE_GL
    USE_GLES
    USE_SDL
    USE_TILE
    USE_TILE_LOCAL
    WIZARD
)
fips_src(${CRAWL_SOURCE} EXCEPT libunix.cc NO_RECURSE)
fips_src(${RLTILES_SOURCE_ROOT} NO_RECURSE)
fips_src(${CRAWL_SOURCE}/prebuilt NO_RECURSE)

file(COPY ${CRAWL_SOURCE}/dat/ DESTINATION dat/)
file(COPY ${CRAWL_SOURCE}/../docs/ DESTINATION docs/)
file(COPY ${CRAWL_SOURCE}/../settings/ DESTINATION settings/)
file(COPY ${CRAWL_CONTRIB}/fonts/ DESTINATION fonts/)

target_link_libraries(dcss
    -sLEGACY_GL_EMULATION=1  # FIXME
    -sFILESYSTEM
    -sGL_UNSAFE_OPTS=0
    -sSDL2_IMAGE_FORMATS='["png"]'

    "--preload-file dat"
    "--preload-file docs"
    "--preload-file settings"
    "--preload-file fonts"

    -sASYNCIFY
    -sASYNCIFY_STACK_SIZE=16777216  # 16 Mb
)

message("Build type: ${CMAKE_BUILD_TYPE}")
if(CMAKE_BUILD_TYPE STREQUAL Debug)
    target_compile_options(dcss PRIVATE
        -g3
    )
    target_link_options(dcss PRIVATE
        -gsource-map
        -sDISABLE_EXCEPTION_CATCHING=0
    )
else()
    target_compile_options(dcss PRIVATE
        -g0
    )
    target_link_options(dcss PRIVATE
        -g0
)
endif()