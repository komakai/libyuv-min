# libyuv, following Chromium's BUILD.gn (libyuv, libyuv_internal, libyuv_neon,
# libyuv_sve and libyuv_sme). The LoongArch (lsx/lasx) targets are left out.
# Needs a `libjpeg` target (libjpeg_turbo-min) unless LIBYUV_DISABLE_JPEG is set.
cmake_minimum_required(VERSION 3.22)
project(libyuv C CXX)

option(LIBYUV_DISABLE_JPEG "Build without MJPEG support (libjpeg)" OFF)

if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64)")
  set(LIBYUV_CPU arm64)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^arm")
  set(LIBYUV_CPU arm)
else()
  set(LIBYUV_CPU other)
endif()

add_library(libyuv STATIC
  source/compare.cc
  source/compare_common.cc
  source/compare_gcc.cc
  source/compare_win.cc
  source/convert.cc
  source/convert_argb.cc
  source/convert_from.cc
  source/convert_from_argb.cc
  source/convert_jpeg.cc
  source/convert_to_argb.cc
  source/convert_to_i420.cc
  source/cpu_id.cc
  source/mjpeg_decoder.cc
  source/mjpeg_validate.cc
  source/planar_functions.cc
  source/rotate.cc
  source/rotate_any.cc
  source/rotate_argb.cc
  source/rotate_common.cc
  source/rotate_gcc.cc
  source/rotate_win.cc
  source/row_any.cc
  source/row_common.cc
  source/row_gcc.cc
  source/row_rvv.cc
  source/row_win.cc
  source/scale.cc
  source/scale_any.cc
  source/scale_argb.cc
  source/scale_common.cc
  source/scale_gcc.cc
  source/scale_rgb.cc
  source/scale_rvv.cc
  source/scale_uv.cc
  source/scale_win.cc
  source/video_common.cc
)
# libyuv_config (an all_dependent_config in gn), shared with the SIMD targets.
add_library(libyuv_config INTERFACE)
target_include_directories(libyuv_config INTERFACE include)
target_compile_definitions(libyuv_config INTERFACE
  CHROMIUM LIBYUV_DISABLE_LSX LIBYUV_DISABLE_LASX)
target_link_libraries(libyuv PUBLIC libyuv_config)
# Enables FMA vectorization for NEON.
target_compile_options(libyuv PRIVATE -ffp-contract=fast)
if(NOT LIBYUV_DISABLE_JPEG)
  target_compile_definitions(libyuv PRIVATE HAVE_JPEG)
  target_link_libraries(libyuv PRIVATE libjpeg)
endif()

if(LIBYUV_CPU STREQUAL "arm64" OR LIBYUV_CPU STREQUAL "arm")
  add_library(libyuv_neon OBJECT
    source/compare_neon.cc
    source/compare_neon64.cc
    source/rotate_neon.cc
    source/rotate_neon64.cc
    source/row_neon.cc
    source/row_neon64.cc
    source/scale_neon.cc
    source/scale_neon64.cc
  )
  if(LIBYUV_CPU STREQUAL "arm64")
    target_compile_options(libyuv_neon PRIVATE -march=armv8-a+dotprod+i8mm)
  endif()
  target_link_libraries(libyuv_neon PRIVATE libyuv_config)
  target_sources(libyuv PRIVATE $<TARGET_OBJECTS:libyuv_neon>)
else()
  target_compile_definitions(libyuv_config INTERFACE LIBYUV_DISABLE_NEON)
endif()

if(LIBYUV_CPU STREQUAL "arm64")
  add_library(libyuv_sve OBJECT source/row_sve.cc)
  target_compile_options(libyuv_sve PRIVATE -march=armv8.5-a+i8mm+sve2)
  target_link_libraries(libyuv_sve PRIVATE libyuv_config)
  target_sources(libyuv PRIVATE $<TARGET_OBJECTS:libyuv_sve>)
else()
  target_compile_definitions(libyuv_config INTERFACE LIBYUV_DISABLE_SVE)
endif()

# gn enables SME only for Linux and Android.
if(LIBYUV_CPU STREQUAL "arm64" AND (ANDROID OR CMAKE_SYSTEM_NAME STREQUAL "Linux"))
  add_library(libyuv_sme OBJECT
    source/rotate_sme.cc
    source/row_sme.cc
    source/scale_sme.cc
  )
  target_compile_options(libyuv_sme PRIVATE -march=armv8.5-a+i8mm+sme)
  target_link_libraries(libyuv_sme PRIVATE libyuv_config)
  target_sources(libyuv PRIVATE $<TARGET_OBJECTS:libyuv_sme>)
else()
  target_compile_definitions(libyuv_config INTERFACE LIBYUV_DISABLE_SME)
endif()
