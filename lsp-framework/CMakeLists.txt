cmake_minimum_required(VERSION 3.12)

project(lsp)

option(LSP_USE_SANITIZERS "Use address and undefined behavior sanitizers" ON)

set(CMAKE_CXX_STANDARD 20)

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	add_compile_options(/W4 /bigobj)
else()
	add_compile_options(-Wall -Wextra -pedantic)

	if(LSP_USE_SANITIZERS)
		add_compile_options(
			$<$<CONFIG:DEBUG>:-fsanitize=address,undefined>
			$<$<CONFIG:DEBUG>:-fno-omit-frame-pointer>
		)
		add_link_options($<$<CONFIG:DEBUG>:-fsanitize=address,undefined>)
	endif()

	if(WIN32 AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
		add_compile_options(-Wa,-mbig-obj)
	endif()
endif()

fips_begin_app(lspgen cmdline)
    fips_dir(lspgen)
        fips_files(lspgen.cpp)
    fips_dir(lsp/json)
        fips_files(json.cpp)
    fips_dir(lsp)
        fips_files(str.cpp)
    target_include_directories(lspgen PRIVATE .)
fips_end_app()

fips_begin_lib(lsp)
fips_dir(lsp)
fips_files(
        # lsp
        connection.h
        error.h
        fileuri.h
        messagebase.h
        messagedispatcher.h
        messagehandler.h
        nullable.h
        requesthandler.h
        serialization.h
        str.h
        util.h
        # io
        io/standardio.h
        # json
        json/json.h
        # jsonrpc
        jsonrpc/jsonrpc.h
        
        # lsp
        connection.cpp
        messagedispatcher.cpp
        requesthandler.cpp
        str.cpp
        fileuri.cpp
        # io
        io/standardio.cpp
        # json
        json/json.cpp
        # jsonrpc
        jsonrpc/jsonrpc.cpp
    )

set(LSP_GENERATED_FILES_DIR ${CMAKE_CURRENT_BINARY_DIR}/generated)
file(MAKE_DIRECTORY ${LSP_GENERATED_FILES_DIR}/lsp)

set(LSP_GENERATED_HEADERS
	${LSP_GENERATED_FILES_DIR}/lsp/types.h
	${LSP_GENERATED_FILES_DIR}/lsp/messages.h
)

set(LSP_GENERATED_SOURCES
	${LSP_GENERATED_FILES_DIR}/lsp/types.cpp
	${LSP_GENERATED_FILES_DIR}/lsp/messages.cpp
)

set(LSP_METAMODEL_JSON ${CMAKE_CURRENT_SOURCE_DIR}/lspgen/lspmetamodel.json)

add_custom_command(
	COMMAND lspgen ${LSP_METAMODEL_JSON}
	WORKING_DIRECTORY ${LSP_GENERATED_FILES_DIR}/lsp
	OUTPUT ${LSP_GENERATED_HEADERS} ${LSP_GENERATED_SOURCES}
	DEPENDS lspgen ${LSP_METAMODEL_JSON}
	COMMENT "Generating lsp types from meta model..."
)
target_include_directories(lsp PUBLIC . ${LSP_GENERATED_FILES_DIR})
target_sources(lsp PRIVATE ${LSP_GENERATED_HEADERS} ${LSP_GENERATED_SOURCES})

fips_end_lib()
