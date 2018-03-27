cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

project(tgvoip)

set(tgvoip_src_loc ..)
set(official_build_target CACHE STRING "?")
set(linux_path_opus_include /usr/include/opus CACHE STRING "?")
set(linux_path_ssl_include /usr/include/openssl-1.0 CACHE STRING "?")
set(POSIX ON CACHE BOOL "?")

file(GLOB tgvoip_SRCS
    "../*.cpp"
    "../audio/*.cpp"
    )

if (UNIX)
    file(GLOB __files
        "../os/linux/*.cpp")
    list(APPEND tgvoip_SRCS ${__files})
endif()

if (POSIX)
    file(GLOB_RECURSE __files
        "../os/posix/*.cpp"
        "../webrtc_dsp/webrtc/*.cc"
        "../webrtc_dsp/webrtc/*.c"
        )
    list(REMOVE_ITEM __files
        ../webrtc_dsp/webrtc/modules/audio_processing/utility/ooura_fft_tables_neon.cc
        ../webrtc_dsp/webrtc/modules/audio_processing/ns/nsx_core_neon.c
        ../webrtc_dsp/webrtc/modules/audio_processing/aecm/aecm_core_neon.cc
        ../webrtc_dsp/webrtc/modules/audio_processing/aec/aec_core_neon.cc
        ../webrtc_dsp/webrtc/common_audio/signal_processing/spl_sqrt_floor_arm.S
        ../webrtc_dsp/webrtc/common_audio/signal_processing/min_max_operations_neon.c
        ../webrtc_dsp/webrtc/common_audio/signal_processing/filter_ar_fast_q12_armv7.S
        ../webrtc_dsp/webrtc/common_audio/signal_processing/downsample_fast_neon.c
        ../webrtc_dsp/webrtc/common_audio/signal_processing/cross_correlation_neon.c
        ../webrtc_dsp/webrtc/common_audio/signal_processing/complex_bit_reverse_arm.S)

    list(APPEND tgvoip_SRCS ${__files})
endif()

add_library(tgvoip SHARED ${tgvoip_SRCS})

target_link_libraries(tgvoip
    PRIVATE
    opus
    dl
    ssl
    "-L/usr/lib/openssl-1.0 -lcrypto"
    )

target_compile_definitions(tgvoip
    PRIVATE
    WEBRTC_APM_DEBUG_DUMP=0
    TGVOIP_USE_DESKTOP_DSP)

target_include_directories(tgvoip
    PRIVATE
    ${tgvoip_src_loc}/webrtc_dsp
    ${linux_path_opus_include}
    ${linux_path_ssl_include})

target_compile_features(tgvoip
    PRIVATE
    cxx_std_11)

if (UNIX)
    target_compile_definitions(tgvoip
        PRIVATE
        WEBRTC_POSIX)
    target_compile_options(tgvoip
        PRIVATE
        -msse2)
endif()

file(GLOB tgvoip_test_SRCS
    *.cpp)

add_executable(tgvoip_test ${tgvoip_test_SRCS})
target_include_directories(tgvoip_test
    PRIVATE
    ..)

find_package(GTest)

target_link_libraries(tgvoip_test
    tgvoip
    GTest::Main
    GTest::GTest)
