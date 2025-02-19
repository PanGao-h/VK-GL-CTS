# Platform port library

# Target file may define TCUTIL_PLATFORM_SRCS
if (NOT DEFINED TCUTIL_PLATFORM_SRCS)
	if (DE_OS_IS_WIN32)
		set(TCUTIL_PLATFORM_SRCS
			win32/tcuWin32Platform.hpp
			win32/tcuWin32Platform.cpp
			win32/tcuWGLContextFactory.hpp
			win32/tcuWGLContextFactory.cpp
			win32/tcuWGL.hpp
			win32/tcuWGL.cpp
			win32/tcuWin32API.h
			win32/tcuWin32Window.cpp
			win32/tcuWin32Window.hpp
			win32/tcuWin32EGLNativeDisplayFactory.hpp
			win32/tcuWin32EGLNativeDisplayFactory.cpp
			win32/tcuWin32VulkanPlatform.hpp
			win32/tcuWin32VulkanPlatform.cpp
			)

	elseif ((DE_OS_IS_UNIX OR DE_OS_IS_OSX) AND (DEQP_USE_X11 OR DEQP_USE_WAYLAND OR DEQP_USE_HEADLESS))

		set(TCUTIL_PLATFORM_SRCS
			lnx/tcuLnx.cpp
			lnx/tcuLnx.cpp
			lnx/tcuLnxPlatform.cpp
			lnx/tcuLnxPlatform.hpp
			lnx/tcuLnxEglPlatform.hpp
			lnx/tcuLnxEglPlatform.cpp
			lnx/tcuLnxVulkanPlatform.hpp
			lnx/tcuLnxVulkanPlatform.cpp
		)

		include_directories(lnx)

		if (DEQP_USE_X11)
			add_definitions(-DDEQP_SUPPORT_X11=1)
			include_directories(lnx/X11)

			message(STATUS "DEQP_SUPPORT_GLX        = ${DEQP_SUPPORT_GLX}")

			if (DEQP_SUPPORT_GLX)
				add_definitions(-DDEQP_SUPPORT_GLX=1)
			endif ()

			set(TCUTIL_PLATFORM_SRCS
				${TCUTIL_PLATFORM_SRCS}
				lnx/X11/tcuLnxX11.cpp
				lnx/X11/tcuLnxX11.hpp
				lnx/X11/tcuLnxX11EglDisplayFactory.hpp
				lnx/X11/tcuLnxX11EglDisplayFactory.cpp
				)

			if (DEQP_USE_XCB)
				set(TCUTIL_PLATFORM_SRCS
					${TCUTIL_PLATFORM_SRCS}
					lnx/X11/tcuLnxX11Xcb.hpp
					lnx/X11/tcuLnxX11Xcb.cpp
					)
			endif()

			if (DEQP_SUPPORT_GLX)
				set(TCUTIL_PLATFORM_SRCS
					${TCUTIL_PLATFORM_SRCS}
					lnx/X11/tcuLnxX11GlxPlatform.hpp
					lnx/X11/tcuLnxX11GlxPlatform.cpp
				)
			endif()
		endif()

		if (DEQP_USE_WAYLAND)
			add_definitions(-DDEQP_SUPPORT_WAYLAND=1)
			include_directories(lnx/wayland)

			set(TCUTIL_PLATFORM_SRCS
				${TCUTIL_PLATFORM_SRCS}
				lnx/wayland/tcuLnxWayland.hpp
				lnx/wayland/tcuLnxWayland.cpp
				lnx/wayland/tcuLnxWaylandEglDisplayFactory.cpp
				lnx/wayland/tcuLnxWaylandEglDisplayFactory.hpp
				)
			include_directories(wayland)
		endif()

		if (DEQP_USE_HEADLESS)
			add_definitions(-DDEQP_SUPPORT_HEADLESS=1)
		endif()

	elseif (DE_OS_IS_UNIX AND DEQP_USE_SURFACELESS)
		set(TCUTIL_PLATFORM_SRCS
			surfaceless/tcuSurfacelessPlatform.hpp
			surfaceless/tcuSurfacelessPlatform.cpp
			)

	elseif (DE_OS_IS_ANDROID)
		set(TCUTIL_PLATFORM_SRCS
			android/tcuAndroidExecService.cpp
			android/tcuAndroidExecService.hpp
			)

		if (DE_ANDROID_API GREATER 8)
			# Add NativeActivity code
			set(TCUTIL_PLATFORM_SRCS
				${TCUTIL_PLATFORM_SRCS}
				android/tcuAndroidAssets.cpp
				android/tcuAndroidAssets.hpp
				android/tcuAndroidNativeActivity.cpp
				android/tcuAndroidNativeActivity.hpp
				android/tcuAndroidPlatform.cpp
				android/tcuAndroidPlatform.hpp
				android/tcuAndroidRenderActivity.cpp
				android/tcuAndroidRenderActivity.hpp
				android/tcuAndroidTestActivity.cpp
				android/tcuAndroidTestActivity.hpp
				android/tcuAndroidUtil.cpp
				android/tcuAndroidUtil.hpp
				android/tcuAndroidWindow.cpp
				android/tcuAndroidWindow.hpp
				)
		endif ()

	elseif (DE_OS_IS_IOS)
		set(TCUTIL_PLATFORM_SRCS
			ios/tcuIOSApp.mm
			ios/tcuIOSApp.h
			ios/tcuIOSPlatform.mm
			ios/tcuIOSPlatform.hh
			)

	elseif (DE_OS_IS_OSX)
		set(TCUTIL_PLATFORM_SRCS
			osx/tcuOSXPlatform.cpp
			osx/tcuOSXPlatform.hpp
			osx/tcuOSXVulkanPlatform.cpp
			osx/tcuOSXVulkanPlatform.hpp
			osx/tcuOSXMetalView.mm
			osx/tcuOSXMetalView.hpp
			)
		list(APPEND TCUTIL_PLATFORM_LIBS ${QUARTZCORE_LIBRARY} ${COCOA_LIBRARY})

	else ()
		set(TCUTIL_PLATFORM_SRCS
			vanilla/tcuVanillaPlatform.cpp
			)

	endif ()
endif ()

include_directories(${PROJECT_BINARY_DIR}/external/vulkancts/framework/vulkan)

add_library(tcutil-platform STATIC ${TCUTIL_PLATFORM_SRCS})


# Add vkutil to the deps before tcutil so that it picks up the c++11 dependencies
target_link_libraries(tcutil-platform vkutil)

target_link_libraries(tcutil-platform tcutil ${TCUTIL_PLATFORM_LIBS})

# Always link to glutil as some platforms such as Win32 always support GL
target_link_libraries(tcutil-platform glutil)

# Always link to eglutil
target_link_libraries(tcutil-platform eglutil)

# X11 libraries
if (DEQP_USE_X11)
	find_package(X11 REQUIRED)
	target_link_libraries(tcutil-platform ${X11_LIBRARIES})
	if (DEQP_USE_XCB)
		find_package(XCB REQUIRED)
		target_link_libraries(tcutil-platform ${XCB_LIBRARIES})
		target_compile_definitions(tcutil-platform PUBLIC DEQP_SUPPORT_XCB=1)
	endif ()
	if (DEQP_SUPPORT_GLX)
	  # GLX functions don't currently have wrappers, so link directly to libGL.
	  target_link_libraries(tcutil-platform GL)
	endif ()
	if (DEQP_USE_WAYLAND)
		find_package(Wayland REQUIRED)
		target_link_libraries(tcutil-platform ${WAYLAND_LIBRARIES})
		target_compile_definitions(tcutil-platform PUBLIC DEQP_SUPPORT_WAYLAND=1)
	endif ()
endif ()

if (DE_OS_IS_WIN32)
	target_link_libraries(tcutil-platform "version")
endif()

# Add similar library for Vulkan SC ( uses same code as vkutil, but with different INL files )
add_library(tcutil-sc-platform STATIC ${TCUTIL_PLATFORM_SRCS})
target_include_directories(tcutil-sc-platform BEFORE PUBLIC ${CMAKE_SOURCE_DIR}/external/vulkancts/framework/vulkan/generated/vulkansc)

# Add vkscutil to the deps before tcutil so that it picks up the c++11 dependencies
target_link_libraries(tcutil-sc-platform vkscutil)

target_link_libraries(tcutil-sc-platform tcutil ${TCUTIL_PLATFORM_LIBS})

# Always link to glutil as some platforms such as Win32 always support GL
target_link_libraries(tcutil-sc-platform glutil)

# Always link to eglutil
target_link_libraries(tcutil-sc-platform eglutil)

# X11 libraries
if (DEQP_USE_X11)
	find_package(X11 REQUIRED)
	target_link_libraries(tcutil-sc-platform ${X11_LIBRARIES})
	if (DEQP_USE_XCB)
		find_package(XCB REQUIRED)
		target_link_libraries(tcutil-sc-platform ${XCB_LIBRARIES})
		target_compile_definitions(tcutil-sc-platform PUBLIC DEQP_SUPPORT_XCB=1)
	endif ()
	if (DEQP_SUPPORT_GLX)
	  # GLX functions don't currently have wrappers, so link directly to libGL.
	  target_link_libraries(tcutil-sc-platform GL)
	endif ()
	if (DEQP_USE_WAYLAND)
		find_package(Wayland REQUIRED)
		target_link_libraries(tcutil-sc-platform ${WAYLAND_LIBRARIES})
		target_compile_definitions(tcutil-sc-platform PUBLIC DEQP_SUPPORT_WAYLAND=1)
	endif ()
endif ()

if (DE_OS_IS_WIN32)
	target_link_libraries(tcutil-sc-platform "version")
endif()

