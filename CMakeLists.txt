#
# project: dcss
#
cmake_minimum_required(VERSION 3.21)
project(dcss-webasm)

set(CMAKE_CXX_STANDARD 11)
set(CRAWL_SOURCE crawl/crawl-ref/source)
set(CRAWL_CONTRIB ${CRAWL_SOURCE}/contrib)
set(BUILD_SHARED_LIBS false)

# include the fips main cmake file
get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
include("${FIPS_ROOT_DIR}/cmake/fips.cmake")

fips_setup()

add_compile_options(-w)
fips_begin_app(dcss windowed)

# zlib
set(ZLIB_SOURCE_ROOT ../zlib)
add_subdirectory(${ZLIB_SOURCE_ROOT} zlib EXCLUDE_FROM_ALL)
add_dependencies(dcss zlibstatic)
target_link_libraries(dcss zlibstatic)
target_include_directories(dcss PRIVATE ${ZLIB_SOURCE_ROOT} ${CMAKE_CURRENT_BINARY_DIR}/zlib NO_RECURSE)

# lua
set(LUA_SOURCE_ROOT ${CRAWL_CONTRIB}/lua/src)
file(GLOB LUA_FILES ${LUA_SOURCE_ROOT}/*.c)
add_library(lua STATIC ${LUA_FILES})
add_dependencies(dcss lua)
target_link_libraries(dcss lua)
target_include_directories(dcss PRIVATE ${LUA_SOURCE_ROOT} NO_RECURSE)

# sqlite
set(SQLITE_SOURCE_ROOT ${CRAWL_CONTRIB}/sqlite)
file(GLOB SQLITE_FILES ${SQLITE_SOURCE_ROOT}/*.c)
add_library(sqlite STATIC ${SQLITE_FILES})
add_dependencies(dcss sqlite)
target_link_libraries(dcss sqlite)
target_include_directories(dcss PRIVATE ${SQLITE_SOURCE_ROOT} NO_RECURSE)

set(RLTILES_SOURCE_ROOT ${CRAWL_SOURCE}/rltiles)
target_include_directories(dcss PRIVATE ${CRAWL_SOURCE} ${RLTILES_SOURCE_ROOT} NO_RECURSE)
target_compile_options(dcss PRIVATE -fexceptions -frtti -sUSE_SDL=2 -sUSE_SDL_IMAGE=2)
target_compile_definitions(dcss PRIVATE
    MONOSPACED_FONT=\"contrib/fonts/DejaVuSansMono.ttf\"
    PROPORTIONAL_FONT=\"contrib/fonts/DejaVuSans.ttf\"
    USE_GL
    USE_GLES
    USE_SDL
    USE_TILE
    USE_TILE_LOCAL
)
fips_src(${CRAWL_SOURCE} EXCEPT libunix.cc NO_RECURSE)
fips_src(${RLTILES_SOURCE_ROOT} NO_RECURSE)