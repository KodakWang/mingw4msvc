cmake_minimum_required(VERSION 3.20)

project(mingw4msvc VERSION 0.0.1 LANGUAGES CXX C)

set(PROJECT_SOURCES
    src/fwprintf.c
    src/mingw32_copy.c
    src/vfprintf.c
    src/vsnprintf.c
)

set(PROJECT_LIBS
    libiconv.a
    libintl.a
    libgcc.a
    libgcrypt.a
    libgpg-error.a
    libmingwex.a

    libavformat.a
    libavcodec.a
    libgsm.a
    libopenjp2.a
    libavutil.a
    #libgcc.a
    libz.a
    libmp3lame.a
    )

add_library(${PROJECT_NAME} STATIC ${PROJECT_SOURCES})
#target_link_directories(${PROJECT_NAME} PRIVATE libs/x86_64)
#target_link_options(${PROJECT_NAME} PUBLIC ${PROJECT_LIBS})

if(CMAKE_BUILD_TYPE AND (CMAKE_BUILD_TYPE STREQUAL "Debug"))
    set(OUT_NAME ${PROJECT_NAME}d.lib)
else()
    set(OUT_NAME ${PROJECT_NAME}.lib)
endif()

# merge all libs
cmake_path(GET CMAKE_C_COMPILER PARENT_PATH COMPILER_DIR)
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${COMPILER_DIR}/lib /out:${OUT_DIR}/${OUT_NAME} ${OUT_DIR}/${OUT_NAME} ${PROJECT_LIBS}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64)

#execute_process(COMMAND ${COMPILER_DIR}/lib /? WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64 OUTPUT_VARIABLE info)
#message(STATUS "info: ${info}")

# compress
file(MAKE_DIRECTORY libs/x86_64_txz)
file(GLOB_RECURSE _LIBS libs/x86_64/*)
foreach(item ${_LIBS})
    cmake_path(RELATIVE_PATH item BASE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64 OUTPUT_VARIABLE item2)
    #message(STATUS "_LIBS: ${item2}")
    
    cmake_path(GET item2 PARENT_PATH item_dir)
    if (item_dir)
        execute_process(COMMAND mkdir -p ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64_txz/${item_dir})
    endif()

    if (NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64_txz/${item2}.txz)
        cmake_path(GET item2 FILENAME item_name)
        execute_process(COMMAND tar --xz -cvf ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64_txz/${item2}.txz ${item_name}
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64/${item_dir})
    endif()
endforeach(item)

# decompress
if (NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64)
    file(MAKE_DIRECTORY libs/x86_64)
    file(GLOB_RECURSE _LIBS libs/x86_64_txz/*)
    foreach(item ${_LIBS})
        cmake_path(RELATIVE_PATH item BASE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64_txz OUTPUT_VARIABLE item2)
        #message(STATUS "_LIBS: ${item2}")
    
        cmake_path(GET item2 PARENT_PATH item_dir)
        if (item_dir)
            execute_process(COMMAND mkdir -p ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64/${item_dir})
        endif()

        execute_process(COMMAND tar --xz -xvf ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64_txz/${item2}
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/x86_64/${item_dir})
    endforeach(item)
endif()