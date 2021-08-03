OPTION(BUILD_IMGUI_DEMO "Build ImGUI Demo APP" OFF)
OPTION(LIBMINC_MINC1_SUPPORT           "Support minc1 file format, requires NETCDF" OFF)

SET(OpenGL_GL_PREFERENCE "LEGACY")
find_package(OpenGL REQUIRED)
FIND_PACKAGE(ZLIB REQUIRED)
SET(HDF5_NO_FIND_PACKAGE_CONFIG_FILE ON)
FIND_PACKAGE(HDF5 REQUIRED COMPONENTS C )


SET(LIBMINC_EXTERNALLY_CONFIGURED  ON)
SET(LIBMINC_BUILD_SHARED_LIBS     OFF)

add_subdirectory(external/libminc)

SET(LIBMINC_DIR "${CMAKE_CURRENT_BINARY_DIR}/external/libminc")

add_subdirectory(external/minc2-simple)

include_directories(external/minc2-simple/src 
                    external/libminc/libsrc2 
                    external/libminc/libcommon 
                    external/imgui
                    ${HDF5_INCLUDE_DIRS} )

add_subdirectory(external/glfw)

add_library(imgui
            external/imgui/imconfig.h
            external/imgui/imgui.cpp
            external/imgui/imgui_demo.cpp
            external/imgui/imgui_draw.cpp
            external/imgui/imgui.h
            external/imgui/imgui_internal.h
            external/imgui/imgui_tables.cpp
            external/imgui/imgui_widgets.cpp
            external/imgui/imstb_rectpack.h
            external/imgui/imstb_textedit.h
            external/imgui/imstb_truetype.h

            external/imgui/backends/imgui_impl_glfw.h
            external/imgui/backends/imgui_impl_glfw.cpp

            external/imgui/backends/imgui_impl_opengl2.h
            external/imgui/backends/imgui_impl_opengl2.cpp

)

target_include_directories(imgui PUBLIC imgui)
target_include_directories(imgui PUBLIC . .. external/imgui/examples)
target_compile_definitions(imgui PUBLIC -DIMGUI_IMPL_OPENGL_LOADER_GLAD)
target_link_libraries(imgui PUBLIC glfw)

add_executable(viewer viewer.cpp)

target_include_directories(viewer PUBLIC external/imgui external/imgui/backends )
target_include_directories(viewer PUBLIC external/glfw/include)

target_link_libraries(viewer PUBLIC minc2-simple imgui glfw ${LIBMINC_LIBRARIES})

if(TARGET OpenGL::GL)
    target_link_libraries(viewer PUBLIC OpenGL::GL)
else()
    target_link_libraries(viewer PUBLIC ${OPENGL_gl_LIBRARY})
endif()


IF(BUILD_IMGUI_DEMO)
    add_executable(imgui_demo external/imgui/imgui_demo.cpp imgui_demo_main.cpp)
    target_include_directories(imgui_demo PUBLIC imgui external/imgui/backends )
    target_include_directories(imgui_demo PUBLIC glfw/include)

    target_link_libraries(imgui_demo PUBLIC imgui glfw)

    if(TARGET OpenGL::GL)
        target_link_libraries(imgui_demo PUBLIC OpenGL::GL)
    else()
        target_link_libraries(imgui_demo PUBLIC ${OPENGL_gl_LIBRARY})
    endif()

ENDIF()