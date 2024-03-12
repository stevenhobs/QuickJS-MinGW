include(ExternalProject)
if(CONFIG_WIN32)
ExternalProject_Add(
    dlfcn
    PREFIX ${CMAKE_BINARY_DIR}/fix-deps-dl
    GIT_REPOSITORY https://github.com/dlfcn-win32/dlfcn-win32.git
    GIT_TAG master
    UPDATE_DISCONNECTED 1
    CMAKE_ARGS -DBUILD_SHARED_LIBS=OFF -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}
)
endif()
