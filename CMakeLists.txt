cmake_minimum_required(VERSION 3.5)

project(vsgFramework)

include(ExternalProject)

set(DOWNLOADS_DIR ${PROJECT_BINARY_DIR}/downloads)
set(COMPONENTS_DIR ${PROJECT_BINARY_DIR})
set(BUILD_INSTALL_PREFIX ${PROJECT_BINARY_DIR} CACHE STRING "Install prefix ")
set(COMPONENT_CMAKE_optionS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${BUILD_INSTALL_PREFIX} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS})


# set compilation to use C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Change the default build type to Release
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

install(DIRECTORY ${INSTALL_PREFIX}/bin DESTINATION .)
install(DIRECTORY ${INSTALL_PREFIX}/lib DESTINATION .)
install(DIRECTORY ${INSTALL_PREFIX}/include DESTINATION .)

add_custom_target(clobber
    COMMAND git clean -d -f -x
)

find_package(Threads)

find_library(DL_LIBRARY dl)
if(NOT DL_LIBRARY)
    set(DL_LIBRARY "") # change from NOTFOUND to empty when passed to linker
endif()

##############################################################################################################
#
# VulkkanSDK
#
if (VULKAN_SDK)
    set(ENV{VULKAN_SDK} ${VULKAN_SDK})
endif()

find_package(Vulkan)

if (NOT VULKAN_FOUND)
    set(VulkanSDKVersion "1.2.176.1" CACHE STRING "Vulkan_SDK version to download")

    if (NOT EXISTS ${DOWNLOADS_DIR}/${VulkanSDKVersion})

        file(MAKE_DIRECTORY downloads)

        if (ANDROID)
            # TODO : Need to implement Andoid VulkanSDK download and unpack
        elseif (WIN32)
            # TODO : Need to implement Windows VulkanSDK download and unpack
        elseif (APPLE)
            # TODO : Need to implement MacOS VulkanSDK download and unpack
            # Possible something like:
            # execute_process(COMMAND wget --no-cookies -O ../vulkansdk-macos-${{env.VulkanSDKVersion}}.dmg https://sdk.lunarg.com/sdk/download/${{env.VulkanSDKVersion}}/mac/vulkansdk-macos-{{env.VulkanSDKVersion}}.dmg)
            # then mount the dmg in some fashion
        else()
            # linux
            execute_process(COMMAND wget --no-cookies -O ${DOWNLOADS_DIR}/vulkansdk-linux-x86_64-${VulkanSDKVersion}.tar.gz https://sdk.lunarg.com/sdk/download/${VulkanSDKVersion}/linux/vulkansdk-linux-x86_64-${VulkanSDKVersion}.tar.gz)
            execute_process(COMMAND tar -zxf ${DOWNLOADS_DIR}/vulkansdk-linux-x86_64-${VulkanSDKVersion}.tar.gz -C ${DOWNLOADS_DIR})
        endif()

    endif()

    if (EXISTS ${DOWNLOADS_DIR}/${VulkanSDKVersion})

        set(VULKAN_SDK ${DOWNLOADS_DIR}/${VulkanSDKVersion}/x86_64)
        set(ENV{VULKAN_SDK} ${VULKAN_SDK})
        set(EXTRA_CMAKE_ARGS_VULKAN_SDK -DVULKAN_SDK=${VULKAN_SDK})

        find_package(Vulkan)

    endif()
endif()

macro(ExternalCompnent component_name build_enalbed git_repository git_tag dependsOn1 dependsOn2 dependsOn3)
    message("ExternalCompnent ( component_name = " ${component_name} " build_enalbed = " ${build_enalbed} " git_repository = " ${git_repository} " git_tag = " ${git_tag} " depends on = " ${dependsOn1} " "  ${dependsOn1} " "  ${dependsOn2} " " ${dependsOn3})
    if (${build_enalbed})
        set(${component_name} "${component_name}")
        ExternalProject_Add(${component_name}
            PREFIX ${COMPONENTS_DIR} CMAKE_ARGS ${COMPONENT_CMAKE_optionS} GIT_PROGRESS ON
            GIT_REPOSITORY ${git_repository} GIT_TAG ${git_tag}
        )
        if (NOT ${dependsOn1} STREQUAL "")
            message("ExternalProject_Add_StepDependencies( " ${component_name} " build " ${dependencies})
            ExternalProject_Add_StepDependencies(${component_name} build ${dependsOn1} ${dependsOn2} ${dependsOn3})
        else()
            set(${component_name} "")
        endif()
    else()
        set(${component_name} "")
    endif()
    message("")
endmacro()

# optional components
option(glslang_BUILD "Build glslang" OFF)
option(VulkanSceneGraph_BUILD "Build VulkanSceneGraph" ON)
option(vsgGIS_BUILD "Build vsgGIS" ON)
option(assimp_BUILD "Build assimp" OFF)
option(vsgXchange_BUILD "Build vsgXchange" ON)
option(vsgImGui_BUILD "Build vsgImGui" OFF)
option(vsgExamples_BUILD "Build vsgExamples" OFF)

##############################################################################################################
#
# glslang
#
ExternalCompnent(glslang ${glslang_BUILD} "https://github.com/KhronosGroup/glslang.git" "master" "" "" "")
ExternalCompnent(assimp ${assimp_BUILD} "https://github.com/assimp/assimp.git" "master" "" "" "")
ExternalCompnent(VulkanSceneGraph ${VulkanSceneGraph_BUILD} "https://github.com/vsg-dev/VulkanSceneGraph.git" "master" "${glslang}" "" "")
ExternalCompnent(vsgGIS ${vsgGIS_BUILD} "https://github.com/vsg-dev/vsgGIS.git" "master" "${VulkanSceneGraph}" "" "")
ExternalCompnent(vsgXchange ${vsgXchange_BUILD} "https://github.com/vsg-dev/vsgXchange.git" "master" "${VulkanSceneGraph}" "${vsgGIS}" "${assimp}")
ExternalCompnent(vsgImGui ${vsgXchange_BUILD} "https://github.com/vsg-dev/vsgImGui.git" "master" "${VulkanSceneGraph}" "" "")
ExternalCompnent(vsgExamples ${vsgExamples_BUILD} "https://github.com/vsg-dev/vsgExamples.git" "master" "${VulkanSceneGraph}" "${vsgXchange}" "${vsgImGui}")
