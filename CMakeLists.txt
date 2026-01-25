cmake_minimum_required(VERSION 3.14)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    if(DEFINED ENV{VITASDK})
        set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
    else()
        message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
    endif()
endif()

project(vitaGL C CXX ASM)

include("${VITASDK}/share/vita.cmake" REQUIRED)

set(VITAGL_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/source)

add_library(vitaGL STATIC
    ${VITAGL_SOURCE_DIR}/blending.c
    ${VITAGL_SOURCE_DIR}/buffers.c
    ${VITAGL_SOURCE_DIR}/custom_shaders.c
    ${VITAGL_SOURCE_DIR}/debug.cpp
    ${VITAGL_SOURCE_DIR}/display_lists.c
    ${VITAGL_SOURCE_DIR}/draw.c
    ${VITAGL_SOURCE_DIR}/egl.c
    ${VITAGL_SOURCE_DIR}/ffp.c
    ${VITAGL_SOURCE_DIR}/framebuffers.c
    ${VITAGL_SOURCE_DIR}/get_info.c
    ${VITAGL_SOURCE_DIR}/gxm.c
    ${VITAGL_SOURCE_DIR}/lookup.c
    ${VITAGL_SOURCE_DIR}/matrices.c
    ${VITAGL_SOURCE_DIR}/misc.c
    ${VITAGL_SOURCE_DIR}/tests.c
    ${VITAGL_SOURCE_DIR}/texture_callbacks.c
    ${VITAGL_SOURCE_DIR}/textures.c
    ${VITAGL_SOURCE_DIR}/utils/atitc_utils.c
    ${VITAGL_SOURCE_DIR}/utils/eac_utils.c
    ${VITAGL_SOURCE_DIR}/utils/etc1_utils.c
    ${VITAGL_SOURCE_DIR}/utils/etc_utils.c
    ${VITAGL_SOURCE_DIR}/utils/glsl_utils.c
    ${VITAGL_SOURCE_DIR}/utils/gpu_utils.c
    ${VITAGL_SOURCE_DIR}/utils/gxm_utils.c
    ${VITAGL_SOURCE_DIR}/utils/mem_utils.c
    ${VITAGL_SOURCE_DIR}/utils/preprocessor/expression.cpp
    ${VITAGL_SOURCE_DIR}/utils/preprocessor/preprocessor.cpp
    ${VITAGL_SOURCE_DIR}/utils/texture_swizzler.cpp
    ${VITAGL_SOURCE_DIR}/vgl.c
)

target_include_directories(vitaGL PUBLIC ${VITAGL_SOURCE_DIR})

set(CMAKE_C_FLAGS "-Wl,-q -g -O3 -ffast-math -mtune=cortex-a9 -mfpu=neon -Wno-incompatible-pointer-types -Wno-stringop-overflow")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -fexceptions -std=gnu++11 -Wno-write-strings")

# ======================================================================================================================
# Debug Flags
# ======================================================================================================================

option(HAVE_SHARK_LOG "Enable logging support in runtime shader compiler." OFF)
if (HAVE_SHARK_LOG)
    target_compile_definitions(vitaGL PRIVATE HAVE_SHARK_LOG)
endif()

set(LOG_ERRORS 0 CACHE STRING "0 = No logging, 1 = Errors will be logged with sceClibPrintf, 2 = Errors will be logged to ux0:data/vitaGL.log.")
if (LOG_ERRORS EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE LOG_ERRORS)
elseif (LOG_ERRORS EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE LOG_ERRORS FILE_LOG)
endif()

option(HAVE_PROFILING "Enable lightweighted profiler for CPU time spent in draw calls." OFF)
if (HAVE_PROFILING)
    target_compile_definitions(vitaGL PRIVATE HAVE_PROFILING)
endif()

set(HAVE_DEBUGGER 0 CACHE STRING "0 = None, 1 = Enable lightweighted on screen debugger interface, 2 = Enable lightweighted on screen debugger interface with extra information (devkit only).")
if (HAVE_DEBUGGER EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE HAVE_DEBUG_INTERFACE)
elseif (HAVE_DEBUGGER EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE
        HAVE_DEVKIT HAVE_RAZOR HAVE_RAZOR_INTERFACE HAVE_DEBUG_INTERFACE HAVE_LIGHT_RAZOR
    )
endif()

set(HAVE_RAZOR 0 CACHE STRING "0 = None, 1 = Enable debugging features through Razor debugger (retail and devkit compatible), 2 = Enable debugging features through Razor debugger (retail and devkit compatible) with ImGui interface.")
if (HAVE_RAZOR EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE HAVE_RAZOR)
elseif (HAVE_RAZOR EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE HAVE_RAZOR HAVE_RAZOR_INTERFACE HAVE_DEBUG_INTERFACE)
endif()

set(HAVE_DEVKIT 0 CACHE STRING "0 = None, 1 = Enable extra debugging features through Razor debugger available only for devkit users, 2 = Enable extra debugging features through Razor debugger available only for devkit users with ImGui interface.")
if (HAVE_DEVKIT EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE HAVE_DEVKIT HAVE_RAZOR)
elseif (HAVE_DEVKIT EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE HAVE_DEVKIT HAVE_RAZOR HAVE_RAZOR_INTERFACE HAVE_DEBUG_INTERFACE)
endif()

option(HAVE_CPU_TRACER "Make the library insert a Razor CPU sync at every buffer swap for better frame timelines in the profiler." OFF)
if (HAVE_CPU_TRACER)
    target_compile_definitions(vitaGL PRIVATE HAVE_CPU_TRACER)
endif()

option(DEBUG_GLSL_TRANSLATOR "Enable logging of GLSL translator input and output prior shader compilation process." OFF)
if (DEBUG_GLSL_TRANSLATOR)
    target_compile_definitions(vitaGL PRIVATE DEBUG_GLSL_TRANSLATOR)
endif()

option(DEBUG_GLSL_PREPROCESSOR "Enable logging of GLSL preprocessor input and output prior shader compilation process." OFF)
if (DEBUG_GLSL_PREPROCESSOR)
    target_compile_definitions(vitaGL PRIVATE DEBUG_GLSL_PREPROCESSOR)
endif()

option(DEBUG_GC "Enable sanity checks for the internal garbage collector." OFF)
if (DEBUG_GC)
    target_compile_definitions(vitaGL PRIVATE DEBUG_GC)
endif()

# ======================================================================================================================
# Compatibility Flags
# ======================================================================================================================

option(HAVE_CUSTOM_HEAP "Replace sceClib heap implementation with custom one (Less efficient but safer)." OFF)
if (HAVE_CUSTOM_HEAP)
    target_compile_definitions(vitaGL PRIVATE HAVE_CUSTOM_HEAP)
endif()

set(HAVE_GLSL_SUPPORT 0 CACHE STRING "0 = None, 1 = Enable experimental GLSL to CG auto translation for shader sources with preprocessor pass (Recommended), 2 = Without preprocessor pass.")
if (HAVE_GLSL_SUPPORT EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE HAVE_GLSL_TRANSLATOR HAVE_GLSL_PREPROCESSOR)
elseif (HAVE_GLSL_SUPPORT EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE HAVE_GLSL_TRANSLATOR)
endif()

option(HAVE_FFP_SHADER_SUPPORT "Enable support for GLSL 1.20 legacy built-in ffp uniform bindings (eg. gl_ModelViewProjectionMatrix). Causes the shader pipeline to be slightly slower." OFF)
if (HAVE_FFP_SHADER_SUPPORT)
    target_compile_definitions(vitaGL PRIVATE HAVE_FFP_SHADER_SUPPORT)
endif()

option(SOFTFP_ABI "Compile the library in soft floating point compatibility mode." OFF)
if (SOFTFP_ABI)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfloat-abi=softfp")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfloat-abi=softfp")
    target_compile_definitions(vitaGL PRIVATE HAVE_SOFTFP_ABI)
endif()

option(STORE_DEPTH_STENCIL "Make all framebuffers depth/stencil surfaces to be load/stored on memory. Makes the rendering slower but more compliant with OpenGL standards." OFF)
if (STORE_DEPTH_STENCIL)
    target_compile_definitions(vitaGL PRIVATE STORE_DEPTH_STENCIL)
endif()

option(HAVE_HIGH_FFP_TEXUNITS "Enable support for more than 2 texunits for fixed function pipeline at the cost of some performance loss." OFF)
if (HAVE_HIGH_FFP_TEXUNITS)
    target_compile_definitions(vitaGL PRIVATE HAVE_HIGH_FFP_TEXUNITS)
endif()

option(HAVE_DISPLAY_LISTS "Enable support for display lists at the cost of some performance loss." OFF)
if (HAVE_DISPLAY_LISTS)
    target_compile_definitions(vitaGL PRIVATE HAVE_DLISTS)
endif()

option(SAFE_ETC1 "Disable hardware support for ETC1 textures. Makes ETC1 textures usage less efficient but allows for proper debugging in Razor." OFF)
if (SAFE_ETC1)
    target_compile_definitions(vitaGL PRIVATE DISABLE_HW_ETC1)
endif()

option(SAFE_DRAW "Make some optimizations in the drawing pipeline less efficient but can solve some glitches." OFF)
if (SAFE_DRAW)
    target_compile_definitions(vitaGL PRIVATE STRICT_DRAW_COMPLIANCE)
endif()

option(SAFE_UNIFORMS "Make some optimizations in the shaders pipeline less efficient but makes uniform location indexing for basic type arrays compliant." OFF)
if (SAFE_UNIFORMS)
    target_compile_definitions(vitaGL PRIVATE STRICT_UNIFORMS_COMPLIANCE)
endif()

option(UNPURE_TEXFORMATS "Enable support for texture dimensions different than 2D (tex2D is still required in shader code)" OFF)
if (UNPURE_TEXFORMATS)
    target_compile_definitions(vitaGL PRIVATE HAVE_UNPURE_TEXFORMATS)
endif()

option(HAVE_VITA3K_SUPPORT "Disable several features in order to make vitaGL compatible with Vita3K. Requires vitaShaRK compiled with https://github.com/Rinnegatamante/vitaShaRK/blob/master/source/vitashark.c#L24 uncommented." OFF)
if (HAVE_VITA3K_SUPPORT)
    target_compile_definitions(vitaGL PRIVATE HAVE_VITA3K_SUPPORT DISABLE_HW_ETC1)
endif()

# ======================================================================================================================
# Hack Flags
# ======================================================================================================================

option(NO_TEX_COMBINER "Disable texture combiner support (GL_COMBINE) for faster fixed function pipeline code execution." OFF)
if (NO_TEX_COMBINER)
    target_compile_definitions(vitaGL PRIVATE DISABLE_TEXTURE_COMBINER)
endif()

option(NO_DEBUG "Disable most of the error handling features (Faster CPU code execution but code may be non compliant to all OpenGL standards)." OFF)
if (NO_DEBUG)
    target_compile_definitions(vitaGL PRIVATE SKIP_ERROR_HANDLING)
endif()

option(BUFFERS_SPEEDHACK "Enable faster vertex buffer copying. May cause crashes." OFF)
if (BUFFERS_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE BUFFERS_SPEEDHACK)
endif()

set(DRAW_SPEEDHACK 0 CACHE STRING "0 = none, 1 = Enable faster code for draw calls. May cause crashes., 2 = Enable faster code for draw calls only for large vertex data draws. May cause crashes.")
if (DRAW_SPEEDHACK EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE DRAW_SPEEDHACK)
elseif (DRAW_SPEEDHACK EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE SAFER_DRAW_SPEEDHACK)
endif()

option(INDICES_DRAW_SPEEDHACK "Enable faster code for handling index buffers for draw calls. May cause crashes." OFF)
if (INDICES_DRAW_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE INDICES_DRAW_SPEEDHACK)
endif()

option(INDICES_SPEEDHACK "Produce faster draw code but disable support for instanced draws and make 32 bit (GL_UNSIGNED_INT) indexed draws potentially cause glitches." OFF)
if (INDICES_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE INDICES_SPEEDHACK)
endif()

option(MATH_SPEEDHACK "Enable faster code for matrix math calls. May cause glitches." OFF)
if (MATH_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE MATH_SPEEDHACK)
endif()

option(TEXTURES_SPEEDHACK "Make glTexSubImage2D/glTexSubImage1D non fully OpenGL compliant but makes rendering pipeline slightly faster. Incompatible with HAVE_TEXTURE_CACHE=1." OFF)
if (TEXTURES_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE TEXTURES_SPEEDHACK)
endif()

option(SAMPLERS_SPEEDHACK "Enable faster code for samplers resolution during shaders usage. May cause glitches." OFF)
if (SAMPLERS_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE SAMPLERS_SPEEDHACK)
endif()

option(PRIMITIVES_SPEEDHACK "Make draw calls more efficient but GL_LINES and GL_POINTS primitives usage may cause glitches." OFF)
if (PRIMITIVES_SPEEDHACK)
    target_compile_definitions(vitaGL PRIVATE PRIMITIVES_SPEEDHACK)
endif()

option(DEPTH_STENCIL_HACK "Make depth and stencil buffers have no memory costs but can cause crashes in some circumstances." OFF)
if (DEPTH_STENCIL_HACK)
    target_compile_definitions(vitaGL PRIVATE DEPTH_STENCIL_HACK)
endif()

# ======================================================================================================================
# Misc Flags
# ======================================================================================================================

option(HAVE_TEXTURE_CACHE "Add file caching for textures not used since a lot of time, acting like a sort of swap implementation to increase effective available memory. (Experimental)" OFF)
if (HAVE_TEXTURE_CACHE)
    target_compile_definitions(vitaGL PRIVATE HAVE_TEX_CACHE)
endif()

option(NO_DMAC "Disable sceDmacMemcpy usage. In some rare instances, it can improve framerate." OFF)
if (NO_DMAC)
    target_compile_definitions(vitaGL PRIVATE DISABLE_DMAC)
endif()

option(HAVE_UNFLIPPED_FBOS "Framebuffers objects won't be internally flipped to match OpenGL standards." OFF)
if (HAVE_UNFLIPPED_FBOS)
    target_compile_definitions(vitaGL PRIVATE HAVE_UNFLIPPED_FBOS)
endif()

option(HAVE_WVP_ON_GPU "Move calculation of the wvp in fixed function pipeline codepath to the GPU. Reduces CPU workload and increases GPU one." OFF)
if (HAVE_WVP_ON_GPU)
    target_compile_definitions(vitaGL PRIVATE HAVE_WVP_ON_GPU)
endif()

set(SHARED_RENDERTARGETS 0 CACHE STRING "0 = none, 1 = Make small framebuffers objects use shared rendertargets instead of dedicated ones., 2 = Make small framebuffers objects use shared rendertargets instead of dedicated ones and adds a mechanism for recycling older rendertargets.")
if (SHARED_RENDERTARGETS EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE HAVE_SHARED_RENDERTARGETS)
elseif (SHARED_RENDERTARGETS EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE HAVE_SHARED_RENDERTARGETS RECYCLE_RENDERTARGETS)
endif()

set(CIRCULAR_VERTEX_POOL 0 CACHE STRING "0 = none, 1 = Make temporary data buffers being handled with a circular pool, 2 = Make temporary data buffers being handled with a circular pool with fallback to regular allocation if the pool gets overrun.")
if (CIRCULAR_VERTEX_POOL EQUAL 1)
    target_compile_definitions(vitaGL PRIVATE HAVE_CIRCULAR_VERTEX_POOL)
elseif (CIRCULAR_VERTEX_POOL EQUAL 2)
    target_compile_definitions(vitaGL PRIVATE HAVE_CIRCULAR_VERTEX_POOL HAVE_FAILSAFE_CIRCULAR_VERTEX_POOL)
endif()

option(USE_SCRATCH_MEMORY "Make GL_DYNAMIC and GL_STREAM vbos be configurable to use circular pool instead of regular allocations. Needs CIRCULAR_VERTEX_POOL." OFF)
if (USE_SCRATCH_MEMORY)
    target_compile_definitions(vitaGL PRIVATE HAVE_SCRATCH_MEMORY)
endif()

option(HAVE_PTHREAD "Use pthread instead of sceKernel for starting garbage collector thread." OFF)
if (HAVE_PTHREAD)
    target_compile_definitions(vitaGL PRIVATE HAVE_PTHREAD)
endif()

option(SINGLE_THREADED_GC "Make the garbage collector run on main thread." OFF)
if (SINGLE_THREADED_GC)
    target_compile_definitions(vitaGL PRIVATE HAVE_SINGLE_THREADED_GC)
endif()

option(PHYCONT_ON_DEMAND "Make the physically contiguous RAM be handled with separate memblocks instead of an heap." OFF)
if (PHYCONT_ON_DEMAND)
    target_compile_definitions(vitaGL PRIVATE PHYCONT_ON_DEMAND)
endif()

option(UNPURE_TEXTURES "Make legal to upload textures without base level." OFF)
if (UNPURE_TEXTURES)
    target_compile_definitions(vitaGL PRIVATE HAVE_UNPURE_TEXTURES)
endif()

option(UNPURE_TEXCOORDS "Make legal to use multitexturing with fixed-function pipeline with GL_TEXTURE0 disabled." OFF)
if (UNPURE_TEXCOORDS)
    target_compile_definitions(vitaGL PRIVATE HAVE_UNPURE_TEXCOORDS)
endif()

option(DISABLE_FFP_MULTITEXTURE "Disable multitexture processing during draw calls performed with fixed function pipeline." OFF)
if (DISABLE_FFP_MULTITEXTURE)
    target_compile_definitions(vitaGL PRIVATE DISABLE_FFP_MULTITEXTURE)
endif()

option(HAVE_WRAPPED_ALLOCATORS "Allow usage of vgl allocators inside wrapped allocators." OFF)
if (HAVE_WRAPPED_ALLOCATORS)
    target_compile_definitions(vitaGL PRIVATE HAVE_WRAPPED_ALLOCATORS)
endif()

option(HAVE_SHADER_CACHE "Enable fast automatic file caching (based on XH3 xxHash algorithm) for application provided shaders." OFF)
if (HAVE_SHADER_CACHE)
    target_compile_definitions(vitaGL PRIVATE HAVE_SHADER_CACHE)
endif()

option(NO_CLIB "Disable sceClib functions usage for easier debugging at the cost of slightly slower CPU code." OFF)
if (NO_CLIB)
    target_compile_definitions(vitaGL PRIVATE DISABLE_CLIB)
endif()

option(DISABLE_W_CLAMPING "Disable W clamping during viewport calculation. Might fix some glitches." OFF)
if (DISABLE_W_CLAMPING)
    target_compile_definitions(vitaGL PRIVATE DISABLE_W_CLAMPING)
endif()

option(NO_TILE_CLIPPER "Disable early tile clipping for scissor testing. Slightly reduces CPU workload but increases GPU workload." OFF)
if (NO_TILE_CLIPPER)
    target_compile_definitions(vitaGL PRIVATE DISABLE_TILE_CLIPPER)
endif()

install(TARGETS vitaGL ARCHIVE DESTINATION lib)

install(FILES ${VITAGL_SOURCE_DIR}/vitaGL.h DESTINATION include)
