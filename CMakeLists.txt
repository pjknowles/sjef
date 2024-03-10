cmake_minimum_required(VERSION 3.15)
project(sjef LANGUAGES CXX C)

include(FetchContent)
FetchContent_Declare(
        LibraryManager
        GIT_REPOSITORY https://github.com/molpro/library-manager.git
        GIT_TAG 0.12.3
)
FetchContent_MakeAvailable(LibraryManager)

LibraryManager_Project(DEFAULT_VERSION "$Format:%(describe:tags)$")
message(VERBOSE "PROJECT_VERSION=${PROJECT_VERSION}")
message(VERBOSE "PROJECT_VERSION_FULL=${PROJECT_VERSION_FULL}")

option(BUILD_TESTS "Whether to build tests or not" ON)
option(BUILD_PROGRAM "Whether to build sjef command-line program or not" ON)

project(sjef LANGUAGES CXX C VERSION ${PROJECT_VERSION})
set(SJEF_VERSION ${PROJECT_VERSION_FULL})

set(PROGRAM_FILENAME "sjef")
set(PROGRAM_NAME "sjef-program")

if (MSVC)
    add_compile_options(/EHsc /permissive-)
endif ()

set(CMAKE_POLICY_DEFAULT_CMP0048 NEW)
add_subdirectory(src)

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif ()

if (BUILD_PROGRAM)
    add_subdirectory(program)
endif ()

find_package(Doxygen)
if (DOXYGEN_FOUND)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
    add_custom_target(${PROJECT_NAME}-doc ALL
            DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/html/index.html
    )
    file(GLOB_RECURSE headers src/*.h */*.md *.md src/*.cpp)
    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/html/index.html
            COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
            DEPENDS ${headers} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            COMMENT "Generating API documentation with Doxygen" VERBATIM
    )
endif (DOXYGEN_FOUND)

add_custom_target(_version.py
        COMMAND echo "__version__ = \"${CMAKE_PROJECT_VERSION}\"" > _version.py
)

if (WIN32)
    if (CONDA_PROVIDES_RSYNC)
    else ()
        FetchContent_Declare(m2 URL https://repo.msys2.org/msys/x86_64/msys2-runtime-3.3-3.3.6-2-x86_64.pkg.tar.zst )
        FetchContent_Declare(bash URL http://repo.msys2.org/msys/x86_64/bash-5.2.026-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(libxxhash URL http://repo.msys2.org/msys/x86_64/libxxhash-0.8.1-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(rsync URL http://repo.msys2.org/msys/x86_64/rsync-3.2.7-2-x86_64.pkg.tar.zst)
        FetchContent_Declare(libzstd URL http://repo.msys2.org/msys/x86_64/libzstd-1.5.5-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(liblz4 URL http://repo.msys2.org/msys/x86_64/liblz4-1.9.4-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(libopenssl URL https://repo.msys2.org/msys/x86_64/libopenssl-3.1.1-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(libiconv URL https://repo.msys2.org/msys/x86_64/libiconv-1.17-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(openssh URL https://repo.msys2.org/msys/x86_64/openssh-9.6p1-1-x86_64.pkg.tar.zst )
        FetchContent_Declare(zlib URL https://repo.msys2.org/msys/x86_64/zlib-1.3.1-1-x86_64.pkg.tar.zst )
#        FetchContent_Declare(git URL https://repo.msys2.org/msys/x86_64/git-2.44.0-1-x86_64.pkg.tar.zst )
#        FetchContent_Declare(gss URL https://repo.msys2.org/mingw/mingw64/mingw-w64-x86_64-gss-1.0.4-1-any.pkg.tar.zst )
        foreach(c bash rsync m2 libzstd libxxhash liblz4 libopenssl libiconv)# openssh zlib)
            FetchContent_MakeAvailable(${c})
            install(DIRECTORY ${${c}_SOURCE_DIR}/usr/bin DESTINATION .)
        endforeach ()
#        foreach(c gss)
#            FetchContent_MakeAvailable(${c})
#            install(DIRECTORY ${${c}_SOURCE_DIR}/mingw64/bin DESTINATION .)
#        endforeach ()
    endif ()

    file(WRITE usr/bin/rsync "export MSYS_NO_PATHCONV=1 MSYS2_ARG_CONV_EXCL=\"*\"\nrsync.exe --rsh=\"/usr/bin/ssh -o UserKnownHostsFile=$HOMEPATH/.ssh/known_hosts -F $HOMEPATH/.ssh/config -i $HOMEPATH/.ssh/id_rsa -i $HOMEPATH/.ssh/id_dsa -i $HOMEPATH/.ssh/id_ed25519 -i $HOMEPATH/.ssh/id_ed25519_sk -i $HOMEPATH/.ssh/id_ecdsa -i $HOMEPATH/.ssh/id_ecdsa_sk\" \"$@\"")
    install(DIRECTORY usr/bin DESTINATION .)
endif ()
