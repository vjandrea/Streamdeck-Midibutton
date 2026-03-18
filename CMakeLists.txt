cmake_minimum_required(VERSION 3.15)
project(midibutton LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_definitions(
    -DASIO_STANDALONE
    -DWIN32_LEAN_AND_MEAN
    -DNOMINMAX
    -D_WIN32_WINNT=0x0601
    -DRTMIDI17_WINMM
)

set(SOURCES
    Sources/StreamDeckMidiButton.cpp
    Sources/Common/ESDConnectionManager.cpp
    Sources/Common/ESDLocalizer.cpp
    Sources/Common/ESDUtilitiesWindows.cpp
    Sources/Common/main.cpp
    Sources/base64.cpp
    Sources/include/rtmidi17/rtmidi17.cpp
)

add_executable(midibutton ${SOURCES})

target_include_directories(midibutton PRIVATE
    Sources
    Sources/Common
    Sources/Vendor/asio/include
    Sources/Vendor/websocketpp
    Sources/Vendor/json/src
    Sources/include
    Sources/macOS
)

target_link_libraries(midibutton PRIVATE ws2_32 winmm)
