cmake_minimum_required(VERSION 2.8.12)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake")
include(CheckIncludeFiles)

project(mt32emu_win32drv CXX)
set(mt32emu_win32drv_VERSION_MAJOR 1)
set(mt32emu_win32drv_VERSION_MINOR 7)
set(mt32emu_win32drv_VERSION_PATCH 0)
set(mt32emu_win32drv_VERSION "${mt32emu_win32drv_VERSION_MAJOR}.${mt32emu_win32drv_VERSION_MINOR}.${mt32emu_win32drv_VERSION_PATCH}")
string(TIMESTAMP mt32emu_win32drv_BUILD_DATE %m/%d/%Y)

if(NOT MSVC)
  # Building with MinGW is possible but currently isn't easy, thus not supported.
  message(FATAL_ERROR "Microsoft (R) C++ compiler required")
endif()

find_package(MT32EMU 2.5 REQUIRED)
set(EXT_LIBS winmm.lib ${MT32EMU_LIBRARIES})
include_directories(${MT32EMU_INCLUDE_DIRS})

CHECK_INCLUDE_FILES("windows.h;mmddk.h" MMDDK_FOUND)
if(NOT MMDDK_FOUND)
  message(FATAL_ERROR "Not found Multimedia Device Development Kit from Microsoft Windows Driver Kits")
endif()

add_definitions(-D_UNICODE -DUNICODE)

set(mt32emu_win32drv_SOURCES
  src/MidiSynth.cpp
  src/winmm_drv.cpp
)

configure_file("mt32emu.inf.in" "mt32emu.inf")

add_library(win32drv MODULE ${mt32emu_win32drv_SOURCES} src/winmm_drv.def)

target_link_libraries(win32drv ${EXT_LIBS})

if(CMAKE_CL_64)
  set(DLL_ARCH_SUFFIX _x64)
endif()

set_target_properties(win32drv PROPERTIES
  OUTPUT_NAME mt32emu${DLL_ARCH_SUFFIX}
  VERSION ${mt32emu_win32drv_VERSION}
)

install(TARGETS win32drv DESTINATION win32drv)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mt32emu.inf DESTINATION win32drv)
install(FILES README.md DESTINATION win32drv)
install(FILES
  COPYING.txt COPYING.LESSER.txt NEWS.txt
  DESTINATION share/doc/munt/mt32emu_win32drv
)

# Build a CPack driven installer package
set(CPACK_PACKAGE_GENERATOR ZIP)
set(CPACK_PACKAGE_VERSION_MAJOR "${mt32emu_win32drv_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${mt32emu_win32drv_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${mt32emu_win32drv_VERSION_PATCH}")
set(CPACK_PACKAGE_VENDOR "muntemu.org")
set(CPACK_PACKAGE_CONTACT "sergm@muntemu.org")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "MT32Emu Windows MME MIDI Driver")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING.LESSER.txt")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
# The default package file name has "-${CPACK_SYSTEM_NAME}" at the end, which doesn't match our previous releases
set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${mt32emu_win32drv_VERSION}-win32-amd64")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${CPACK_PACKAGE_FILE_NAME}")
set(CPACK_SOURCE_GENERATOR TGZ)
set(CPACK_SOURCE_STRIP_FILES TRUE)

include(CPack)
