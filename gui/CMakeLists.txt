opentrack_boilerplate(opentrack NO-LIBRARY)

if(UNIX OR APPLE)
    target_include_directories(opentrack "${CMAKE_SOURCE_DIR}/qxt-mini")
    if(APPLE)
        set(qxt-plat mac)
    else()
        set(qxt-plat x11)
    endif()
    file(GLOB qxt-mini-c
        ${CMAKE_SOURCE_DIR}/qxt-mini/*.h
        ${CMAKE_SOURCE_DIR}/qxt-mini/qxtglobalshortcut.cpp
        ${CMAKE_SOURCE_DIR}/qxt-mini/plat/qxtglobalshortcut_${qxt-plat}.cpp
    )
    opentrack_qt(qxt-mini)
    add_library(opentrack-qxt-mini STATIC ${qxt-mini-all})
    target_link_libraries(opentrack-qxt-mini ${MY_QT_LIBS})
    if(NOT APPLE)
            target_link_libraries(opentrack-qxt-mini X11)
    endif()
endif()

if(WIN32)
    SET(SDK_CONSOLE_DEBUG FALSE CACHE BOOL "Console window visible at runtime")
endif()

if(WIN32 AND NOT SDK_CONSOLE_DEBUG)
    set(opentrack-win32-executable WIN32)
else()
    set(opentrack-win32-executable "")
endif()

if(UNIX OR APPLE)
    list(APPEND opentrack-c ${CMAKE_SOURCE_DIR}/qxt-mini/qxtglobalshortcut.h)
endif()
opentrack_qt(opentrack)
add_executable(opentrack ${opentrack-win32-executable} ${opentrack-all})
opentrack_compat(opentrack)
if(NOT WIN32)
    set_target_properties(opentrack PROPERTIES SUFFIX ".bin")
endif()

target_link_libraries(opentrack
    opentrack-api opentrack-version opentrack-pose-widget opentrack-spline-widget
    opentrack-tracker-pt opentrack-filter-accela
)

if(APPLE)
    SET_TARGET_PROPERTIES(opentrack-qxt-mini PROPERTIES LINK_FLAGS "-framework Carbon -framework CoreFoundation")
endif()
if(UNIX OR APPLE)
    target_link_libraries(opentrack opentrack-qxt-mini)
endif()
link_with_dinput8(opentrack)
target_link_libraries(opentrack ${MY_QT_LIBS})

if(APPLE)
    # for process detector
    target_link_libraries(opentrack proc)
endif()

if(LINUX)
    # for process detector
    target_link_libraries(opentrack procps)
endif()

install(TARGETS opentrack DESTINATION .)