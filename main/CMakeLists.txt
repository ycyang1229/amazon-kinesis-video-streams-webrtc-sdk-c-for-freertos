set(LIB_SOURCE_DIR "${CMAKE_SOURCE_DIR}/main/lib")
set(LIBKVSPIC_NAME "amazon-kinesis-video-streams-pic")
set(LIBKVSPIC_INCLUDE_DIRS "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/common/include" 
                                "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/client/include"
                                "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/utils/include"
                                "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/mkvgen/include"
                                "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/view/include"
                                "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/heap/include"
                                "${LIB_SOURCE_DIR}/${LIBKVSPIC_NAME}/src/state/include")
set(LIBKVSCOMMONLWS_NAME "amazon-kinesis-video-streams-producer-c")
set(LIBKVSCOMMONLWS_INCLUDE_DIRS "${LIB_SOURCE_DIR}/${LIBKVSCOMMONLWS_NAME}/src/include")
set(LIBKVSWEBRTC_NAME "amazon-kinesis-video-streams-webrtc-sdk-c")
set(LIBKVSWEBRTC_INCLUDE_DIRS "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/include"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Crypto"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Ice"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/PeerConnection"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Rtcp"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Rtp"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Sctp"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Sdp"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Signaling"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Srtp"
                                "${LIB_SOURCE_DIR}/${LIBKVSWEBRTC_NAME}/src/source/Stun")

set(OPEN_SRC_INCLUDE_DIRS "${LIBKVSPIC_INCLUDE_DIRS}" "${LIBKVSCOMMONLWS_INCLUDE_DIRS}" "${LIBKVSWEBRTC_INCLUDE_DIRS}")
idf_component_register(SRCS "kvswebrtc_example_main.c" "Common.c" "kvsWebRTCClientMaster.c"
                    INCLUDE_DIRS "." "${OPEN_SRC_INCLUDE_DIRS}"
                    REQUIRES nvs_flash mbedtls lwip fatfs
                    EMBED_FILES "cert.pem")


idf_component_get_property(nvs_flash_lib nvs_flash COMPONENT_LIB)
idf_component_get_property(mbedtls_lib mbedtls COMPONENT_LIB)

add_definitions(-DKVS_PLAT_ESP_FREERTOS)
message("############################## start llhttp ##############################")
add_subdirectory(lib/libllhttp)
set_property(TARGET llhttp APPEND PROPERTY LINK_LIBRARIES
        ${nvs_flash_lib}
	${mbedtls_lib})
set_property(TARGET llhttp APPEND PROPERTY INTERFACE_LINK_LIBRARIES
        ${nvs_flash_lib}
	${mbedtls_lib})
target_link_libraries(${COMPONENT_LIB} PUBLIC llhttp)
set(KVS_LIBLLHTTP_INCLUDE_DIRS "${CMAKE_BINARY_DIR}/esp-idf/main/lib/libllhttp/include" "${LIB_SOURCE_DIR}/libllhttp/include")


message("############################## start wslay ##############################")

add_subdirectory(lib/private-wslay)
set_property(TARGET wslay APPEND PROPERTY LINK_LIBRARIES
        ${nvs_flash_lib}
	${mbedtls_lib})
set_property(TARGET wslay APPEND PROPERTY INTERFACE_LINK_LIBRARIES
        ${nvs_flash_lib}
	${mbedtls_lib})
target_link_libraries(${COMPONENT_LIB} PUBLIC wslay)
set(KVS_LIBWSLAY_INCLUDE_DIRS "${LIB_SOURCE_DIR}/private-wslay/lib/includes" "${CMAKE_BINARY_DIR}/esp-idf/main/lib/private-wslay/lib/includes")
message("KVS_LIBWSLAY_INCLUDE_DIRS:${KVS_LIBWSLAY_INCLUDE_DIRS}")

message("############################## start srtp ##############################")
set(KVS_PLAT_ESP_FREERTOS ON CACHE BOOL "Build for ESP FreeRTOS")
add_subdirectory(lib/libsrtp)
set_property(TARGET srtp2 APPEND PROPERTY LINK_LIBRARIES ${mbedtls_lib})
set_property(TARGET srtp2 APPEND PROPERTY INTERFACE_LINK_LIBRARIES ${mbedtls_lib})        
target_link_libraries(${COMPONENT_LIB} PUBLIC srtp2)
set(KVS_LIBSRTP_INCLUDE_DIRS "${LIB_SOURCE_DIR}/libsrtp/include")

###



message("############################## start pic ##############################")
add_subdirectory(lib/amazon-kinesis-video-streams-pic)
target_link_libraries(${COMPONENT_LIB} PUBLIC kvspicState kvspicUtils)

message("############################## start producer c ##############################")
## this needs to be fixed. #YC_TBD.
set(BUILD_DEPENDENCIES OFF CACHE BOOL "Whether or not to build depending libraries from source")
set(USE_OPENSSL OFF CACHE BOOL "Use openssl as crypto library")
set(USE_MBEDTLS ON CACHE BOOL "Use openssl as crypto library")
set(BUILD_COMMON_ONLY ON CACHE BOOL "Whether or not to build ProducerC libwebsockets common library")
set(BUILD_COMMON_LWS OFF CACHE BOOL "Whether or not to build ProducerC libwebsockets common library")
set(BUILD_COMMON_CURL OFF CACHE BOOL "Whether or not to build ProducerC curl common library")
set(BUILD_KVSPIC OFF CACHE BOOL "Whether or not to build the part of kvs pic")
set(KVS_PLAT_ESP_FREERTOS ON CACHE BOOL "Build for ESP FreeRTOS")
set(OPEN_SRC_INCLUDE_DIRS "${OPEN_SRC_INCLUDE_DIRS}")

add_subdirectory(lib/amazon-kinesis-video-streams-producer-c)

set_property(TARGET kvsCommon APPEND PROPERTY LINK_LIBRARIES kvspicUtils ${mbedtls_lib})
set_property(TARGET kvsCommon APPEND PROPERTY INTERFACE_LINK_LIBRARIES kvspicUtils ${mbedtls_lib})
target_link_libraries(${COMPONENT_LIB} PUBLIC kvsCommon)

message("############################## start webrtc ##############################")
set(BUILD_DEPENDENCIES OFF CACHE BOOL "Whether or not to build depending libraries from source")
set(BUILD_STATIC_LIBS ON CACHE BOOL "Build all libraries statically. (This includes third-party libraries.)")
set(BUILD_SAMPLE OFF CACHE BOOL "Build available samples")
set(ENABLE_DATA_CHANNEL OFF CACHE BOOL "Enable support for data channel")
set(OPEN_SRC_INCLUDE_DIRS "${OPEN_SRC_INCLUDE_DIRS}" "${KVS_LIBWSLAY_INCLUDE_DIRS} ${KVS_LIBSRTP_INCLUDE_DIRS} ${KVS_LIBLLHTTP_INCLUDE_DIRS}")

add_subdirectory(lib/amazon-kinesis-video-streams-webrtc-sdk-c)

set_property(TARGET kvsWebrtcSignalingClient APPEND PROPERTY LINK_LIBRARIES
        kvspicUtils
        wslay
        srtp2
        ${mbedtls_lib}
        kvsCommon
        llhttp)
set_property(TARGET kvsWebrtcSignalingClient APPEND PROPERTY INTERFACE_LINK_LIBRARIES
        kvspicUtils
        wslay
        srtp2
        ${mbedtls_lib}
        kvsCommon
        llhttp)
target_link_libraries(${COMPONENT_LIB} PUBLIC kvsWebrtcSignalingClient)

set_property(TARGET kvsWebrtcClient APPEND PROPERTY LINK_LIBRARIES
        kvspicUtils
        mbedtls
        srtp2
        ${mbedtls_lib}
        ${nvs_flash_lib})
set_property(TARGET kvsWebrtcClient APPEND PROPERTY INTERFACE_LINK_LIBRARIES
        kvspicUtils
        mbedtls
        srtp2
        ${mbedtls_lib}
        ${nvs_flash_lib})
target_link_libraries(${COMPONENT_LIB} PUBLIC kvsWebrtcClient)