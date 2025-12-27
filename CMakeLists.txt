cmake_minimum_required(VERSION 3.20)
project(app VERSION 0.1.0 LANGUAGES C CXX)

set(APP_NAME app)
set(FRAMEWORK_SOURCE OpenGL)

# ============================================================
# APP Executable
# ============================================================
add_executable(${APP_NAME})

# ============================================================
# Deps
# ============================================================
add_subdirectory(thirdparty/cubism/Common)
add_subdirectory(thirdparty/cubism/Framework)
add_subdirectory(thirdparty/glfw)
add_subdirectory(thirdparty/glew/build/cmake)

# ============================================================
# DEMO SOURCE
# ============================================================
add_subdirectory(src)

# ============================================================
# Framework Render: OpenGL / GLEW  
# ============================================================
target_include_directories(Framework PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/glew/include
    ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/glfw/include
)

target_compile_definitions(Framework PUBLIC
    GLEW_STATIC
    CSM_TARGET_WIN_GL
)

# ============================================================
# stdout
# ============================================================
target_link_options(${APP_NAME} PRIVATE
    -static-libgcc
    -static-libstdc++
)

# ============================================================
# Includes
# ============================================================
target_include_directories(${APP_NAME} PUBLIC
    thirdparty/cubism/Common
    src
    thirdparty/stb
    thirdparty/cubism/Framework/src
    thirdparty/glew/include
    thirdparty/glfw/include
    thirdparty/cubism/Core/include
)

# ============================================================
# App Macro
# ============================================================
target_compile_definitions(${APP_NAME} PRIVATE
    WIN32
    _WINDOWS
    GLEW_STATIC
    CMAKE_INTDIR="Debug"
)

# ============================================================
# Search Libs
# ============================================================
target_link_directories(${APP_NAME} PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/cubism/Core/lib/windows/x86_64/145
    ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/cubism/Core/dll/windows/x86_64
)

# ============================================================
# Link the libaries for app
# ============================================================
target_link_libraries(${APP_NAME} PRIVATE
    # --- Live2D ---
    Framework
    Live2DCubismCore

    # --- OpenGL stack ---
    glfw
    opengl32
    glu32

    # --- GLEW ---
    glew

    # --- Win32 system libs ---
    kernel32
    user32
    gdi32
    winspool
    shell32
    ole32
    oleaut32
    uuid
    comdlg32
    advapi32
)

# ============================================================
# Copy runtime resources & DLLs for Live2D demo
# ============================================================

add_custom_command(TARGET ${APP_NAME} POST_BUILD

    # -------- Live2D resources --------
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/Resources
        $<TARGET_FILE_DIR:${APP_NAME}>/Resources

    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/FrameworkShaders
        $<TARGET_FILE_DIR:${APP_NAME}>/FrameworkShaders

    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/SampleShaders
        $<TARGET_FILE_DIR:${APP_NAME}>/SampleShaders


    # -------- Live2D Core DLL --------
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/cubism/Core/dll/windows/x86_64/Live2DCubismCore.dll
        $<TARGET_FILE_DIR:${APP_NAME}>


    # -------- GLEW runtime DLL --------
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:glew>
        $<TARGET_FILE_DIR:${APP_NAME}>
)
