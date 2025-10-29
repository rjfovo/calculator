cmake_minimum_required(VERSION 3.16)

set(PROJECT_NAME cutefish-calculator)
project(${PROJECT_NAME})

# 设置Qt6
find_package(Qt6 REQUIRED COMPONENTS Core Gui Quick QuickControls2 LinguistTools)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if (MSVC)
    add_compile_options("/utf-8")
endif()

set(SRCS
    main.cpp
    calcengine.cpp
    engine/constants.cpp
    engine/evaluator.cpp
    engine/functions.cpp
    engine/hmath.cpp
    engine/number.c
)

set(RESOURCES
    qml.qrc
)

# 使用标准的 add_executable
add_executable(${PROJECT_NAME} ${SRCS} ${RESOURCES})

# 使用 PRIVATE 关键字签名
target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt6::Core
    Qt6::Gui
    Qt6::Quick
    Qt6::QuickControls2
)

# 健壮的翻译文件处理
set(QM_FILES "")  # 初始化为空

# 检查翻译目录和文件是否存在
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/translations/)
    file(GLOB TS_FILES translations/*.ts)
    if(TS_FILES)
        message(STATUS "Found translation files: ${TS_FILES}")
        
        # 只生成 .qm 文件，不更新 .ts 文件
        # 使用 lrelease 直接编译现有的 .ts 文件
        foreach(TS_FILE ${TS_FILES})
            get_filename_component(TS_NAME ${TS_FILE} NAME_WE)
            set(QM_FILE "${CMAKE_CURRENT_BINARY_DIR}/${TS_NAME}.qm")
            add_custom_command(
                OUTPUT ${QM_FILE}
                COMMAND Qt6::lrelease
                ARGS ${TS_FILE} -qm ${QM_FILE}
                DEPENDS ${TS_FILE}
                COMMENT "Generating ${TS_NAME}.qm"
            )
            list(APPEND QM_FILES ${QM_FILE})
        endforeach()
        
        # 添加自定义目标确保翻译文件在构建时生成
        add_custom_target(generate_translations ALL 
            DEPENDS ${QM_FILES}
            COMMENT "Generating translation files"
        )
        add_dependencies(${PROJECT_NAME} generate_translations)
    else()
        message(WARNING "No .ts files found in translations directory")
    endif()
else()
    message(WARNING "Translations directory not found at ${CMAKE_CURRENT_SOURCE_DIR}/translations/")
endif()

# 安装可执行文件
install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION /usr/bin)

# 安装桌面文件
install(FILES
    cutefish-calculator.desktop
    DESTINATION /usr/share/applications/
)

# 只有在有翻译文件时才安装
if(QM_FILES)
    message(STATUS "Installing translation files: ${QM_FILES}")
    install(FILES ${QM_FILES} DESTINATION /usr/share/${PROJECT_NAME}/translations)
else()
    message(WARNING "No translation files to install")
endif()

# 可选：创建一个虚拟的翻译文件如果没有任何翻译文件存在
if(NOT QM_FILES AND NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/translations/)
    message(STATUS "Creating dummy translation setup to satisfy build system")
    add_custom_target(dummy_translations ALL
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/dummy_translations
        COMMAND echo "Creating dummy translation structure"
        COMMENT "Setting up dummy translations"
    )
    add_dependencies(${PROJECT_NAME} dummy_translations)
endif()