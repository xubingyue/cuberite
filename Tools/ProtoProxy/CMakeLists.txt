project (ProtoProxy)

include(../../SetFlags.cmake)

set_flags()
set_lib_flags()


# Set include paths to the used libraries:
include_directories(SYSTEM "../../lib")
include_directories(SYSTEM "../../lib/polarssl/include")
include_directories("../../src")

function(flatten_files arg1)
	set(res "")
	foreach(f ${${arg1}})
		get_filename_component(f ${f} ABSOLUTE)
		list(APPEND res ${f})
	endforeach()
	set(${arg1} "${res}" PARENT_SCOPE)
endfunction()

include(../../lib/polarssl.cmake)
add_subdirectory(../../lib/zlib ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/lib/zlib)

set_exe_flags()

# Include the shared files:
set(SHARED_SRC
	../../src/ByteBuffer.cpp
	../../src/StringUtils.cpp
	../../src/PolarSSL++/AesCfb128Decryptor.cpp
	../../src/PolarSSL++/AesCfb128Encryptor.cpp
	../../src/PolarSSL++/CryptoKey.cpp
	../../src/PolarSSL++/CtrDrbgContext.cpp
	../../src/PolarSSL++/EntropyContext.cpp
	../../src/PolarSSL++/RsaPrivateKey.cpp
	../../src/LoggerListeners.cpp
	../../src/Logger.cpp
)
set(SHARED_HDR
	../../src/ByteBuffer.h
	../../src/StringUtils.h
	../../src/PolarSSL++/AesCfb128Decryptor.h
	../../src/PolarSSL++/AesCfb128Encryptor.h
	../../src/PolarSSL++/CryptoKey.h
	../../src/PolarSSL++/CtrDrbgContext.h
	../../src/PolarSSL++/EntropyContext.h
	../../src/PolarSSL++/RsaPrivateKey.h
)
set(SHARED_OSS_SRC
	../../src/OSSupport/CriticalSection.cpp
	../../src/OSSupport/Event.cpp
	../../src/OSSupport/File.cpp
	../../src/OSSupport/IsThread.cpp
	../../src/OSSupport/StackTrace.cpp
)
set(SHARED_OSS_HDR
	../../src/OSSupport/CriticalSection.h
	../../src/OSSupport/Event.h
	../../src/OSSupport/File.h
	../../src/OSSupport/IsThread.h
	../../src/OSSupport/StackTrace.h
)

if(WIN32)
	list (APPEND SHARED_OSS_SRC ../../src/StackWalker.cpp)
	list (APPEND SHARED_OSS_HDR ../../src/StackWalker.h)
endif()

flatten_files(SHARED_SRC)
flatten_files(SHARED_HDR)
flatten_files(SHARED_OSS_SRC)
flatten_files(SHARED_OSS_HDR)
source_group("Shared" FILES ${SHARED_SRC} ${SHARED_HDR})
source_group("Shared\\OSSupport" FILES ${SHARED_OSS_SRC} ${SHARED_OSS_HDR})



# Include the main source files:
set(SOURCES
	Connection.cpp
	Globals.cpp
	ProtoProxy.cpp
	Server.cpp
)
set(HEADERS
	Connection.h
	Globals.h
	Server.h
)
source_group("" FILES ${SOURCES} ${HEADERS})

add_executable(ProtoProxy
	${SOURCES}
	${HEADERS}
	${SHARED_SRC}
	${SHARED_HDR}
	${SHARED_OSS_SRC}
	${SHARED_OSS_HDR}
)

target_link_libraries(ProtoProxy zlib mbedtls)

