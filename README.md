# vsgFramework
Framework collects all the main VulkanSceneGraph related projects together under one repository/directory structure with build support for each component. vsgFramework can be used as a template repository for projects that wish to use it as a base for their own projects.

vsgFramework uses CMake's ExternalProject facility to automatically clones external projects in the src directory, then configure, build and install the external project headers, libraries and executables in local include, lib and bin directories.

You can then build your own applications against these local include, lib and bin directories by setting system paths to vsgFramework directory, or install them in system directories, or user defined location by setting the CMAKE_INSTALL_PREFIX variable.

You can toggle on/off the use of external projects via ccmake/CMakeSetup.

To checkout:

    git clone https://github.com/vsg-dev/vsgFramework.git

To build:

    cmake .
    make -j 8

To install headers, libraries and excutables in system or user defined location

    sudo make install

To cross compile for Windows (64bit) on openSUSE 15.3:

    sudo zypper ar https://download.opensuse.org/repositories/home:/rhabacker:/branches:/games:/mingw64/openSUSE_Leap_15.3/home:rhabacker:branches:games:mingw64.repo
    sudo zypper install mingw64-vulkan-devel
    mkdir vsgFramework-build
    cd vsgFramework-build
    CXXFLAGS=-Wl,--enable-stdcall-fixup mingw32-cmake ../vsgFramework -DBUILD_assimp=OFF -DCONFIGURE_COMMAND=/usr/bin/mingw32-cmake \
        -DVulkanSceneGraph_CMAKE_OPTIONS=-DGLSLANG_ResourceLimits_maxDualSourceDrawBuffersEXT_EXITCODE=0;-DGLSLANG_ResourceLimits_maxDualSourceDrawBuffersEXT_EXITCODE__TRYRUN_OUTPUT="
        -DCOMPONENTS_CMAKE_OPTIONS=-DVulkan_LIBRARY=/usr/x86_64-w64-mingw32/sys-root/mingw/bin/vulkan-1.dll
    make -j 8
