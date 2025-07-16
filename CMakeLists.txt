cmake_minimum_required(VERSION 3.10)
project(Oscpack)

set(BUILD_EXAMPLES 0)
set(BUILD_TESTS 0)

# separate versions of NetworkingUtils.cpp and UdpSocket.cpp are provided for Win32 and POSIX
# the IpSystemTypePath selects the correct ones based on the current platform
if(WIN32)
    set(IpSystemTypePath ip/win32)
    set(LIBS ${LIBS} Ws2_32 winmm)
else(WIN32)
    set(IpSystemTypePath ip/posix)
endif(WIN32)

set(OSCPACK_HEADERS
    ip/IpEndpointName.h
    ip/NetworkingUtils.h
    ip/UdpSocket.h
    ip/PacketListener.h
    ip/TimerListener.h
    osc/OscTypes.h
    osc/OscHostEndianness.h
    osc/OscException.h
    osc/OscPacketListener.h
    osc/MessageMappingOscPacketListener.h
    osc/OscReceivedElements.h
    osc/OscPrintReceivedElements.h
    osc/OscOutboundPacketStream.h
)

set(OSCPACK_SOURCES
    ip/IpEndpointName.cpp
    ${IpSystemTypePath}/NetworkingUtils.cpp
    ${IpSystemTypePath}/UdpSocket.cpp
    osc/OscTypes.cpp 
    osc/OscReceivedElements.cpp
    osc/OscPrintReceivedElements.cpp
    osc/OscOutboundPacketStream.cpp
)

add_library(oscpack STATIC
  ${OSCPACK_HEADERS}
  ${OSCPACK_SOURCES}
)

target_include_directories(oscpack PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

target_compile_options(oscpack PRIVATE
    "$<$<CONFIG:DEBUG>:-O1>"
    "$<$<CONFIG:RELWITHDEBINFO>:-O2>"
    "$<$<CONFIG:RELEASE>:-Ofast>"
    "$<$<OR:$<CONFIG:RELWITHDEBINFO>,$<CONFIG:RELEASE>>:-Werror>"
)

if (BUILD_TESTS)
    ADD_EXECUTABLE(OscUnitTests tests/OscUnitTests.cpp)
    TARGET_LINK_LIBRARIES(OscUnitTests oscpack ${LIBS})

    ADD_EXECUTABLE(OscSendTests tests/OscSendTests.cpp)
    TARGET_LINK_LIBRARIES(OscSendTests oscpack ${LIBS})

    ADD_EXECUTABLE(OscReceiveTest tests/OscReceiveTest.cpp)
    TARGET_LINK_LIBRARIES(OscReceiveTest oscpack ${LIBS})
endif(BUILD_TESTS)

if (BUILD_EXAMPLES)
    ADD_EXECUTABLE(OscDump examples/OscDump.cpp)
    TARGET_LINK_LIBRARIES(OscDump oscpack ${LIBS})

    ADD_EXECUTABLE(SimpleReceive examples/SimpleReceive.cpp)
    TARGET_LINK_LIBRARIES(SimpleReceive oscpack ${LIBS})

    ADD_EXECUTABLE(SimpleSend examples/SimpleSend.cpp)
    TARGET_LINK_LIBRARIES(SimpleSend oscpack ${LIBS})
endif(BUILD_EXAMPLES)

if(MSVC)
  # Force to always compile with W4
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  # Update if necessary
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-long-long -pedantic")
endif()
