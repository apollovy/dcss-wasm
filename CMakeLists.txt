#
# project: dcss
#
cmake_minimum_required(VERSION 3.21)
project(dcss-webasm)

set(CMAKE_CXX_STANDARD 11)
set(CRAWL_SOURCE crawl/crawl-ref/source)
set(CRAWL_CONTRIB ${CRAWL_SOURCE}/contrib)
set(BUILD_SHARED_LIBS false)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fexceptions -s USE_WEBGL2=1 -s USE_GLFW=3 --use-port=freetype")
set(SDL2_LIBRARIES "-sUSE_SDL=2")
set(SDL2_IMAGE_LIBRARIES "-sUSE_SDL_IMAGE=2")
set(DISABLE_EXCEPTION_CATCHING 0)

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
    MONOSPACED_FONT=\"./fonts/DejaVuSansMono.ttf\"
    PROPORTIONAL_FONT=\"./fonts/DejaVuSans.ttf\"
    USE_FT
    USE_GL
    USE_GLES
    USE_SDL
    USE_TILE
    USE_TILE_LOCAL
)
# xcode settings: ASSERTS=1 CLUA_BINDINGS=1 DCSS_IOS=1 ENABLE_NLS=1 MONOSPACED_FONT=\"./fonts/DejaVuSansMono.ttf\" PROPORTIONAL_FONT=\"./fonts/DejaVuSans.ttf\" TOUCH_UI=1 USE_FT=1 USE_GL=1 USE_GLES=1 USE_SDL=1 USE_TILE=1 USE_TILE_LOCAL=1 HAVE_STAT=1 WIZARD=1 USE_SOUND=1
fips_src(${CRAWL_SOURCE} EXCEPT libunix.cc NO_RECURSE)
fips_src(${RLTILES_SOURCE_ROOT} NO_RECURSE)
fips_src(${CRAWL_SOURCE}/prebuilt NO_RECURSE)

install(DIRECTORY ${CRAWL_SOURCE}/dat/ DESTINATION dat/)
install(DIRECTORY ${CRAWL_SOURCE}/../docs/ DESTINATION docs/)
install(DIRECTORY ${CRAWL_SOURCE}/../settings/ DESTINATION settings/)
install(DIRECTORY ${CRAWL_CONTRIB}/fonts/ DESTINATION fonts/)

target_link_libraries(dcss
    ${SDL2_LIBRARIES}
    ${SDL2_IMAGE_LIBRARIES}
    "-s LEGACY_GL_EMULATION=1 -sFILESYSTEM -sGL_UNSAFE_OPTS=0 -sSDL2_IMAGE_FORMATS='[\"png\"]'"
    "--preload-file dat"
    "--preload-file docs"
    "--preload-file settings"
    "--preload-file fonts"
)
