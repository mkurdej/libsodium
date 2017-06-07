include_directories(
    ${LIBSODIUM_SOURCE_INCLUDE_DIR}
    ${LIBSODIUM_SOURCE_INCLUDE_DIR}/sodium
    ${LIBSODIUM_BINARY_INCLUDE_DIR}
    ${LIBSODIUM_BINARY_INCLUDE_DIR}/sodium
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../quirks
)

add_definitions("-DTEST_SRCDIR=\"${CMAKE_CURRENT_SOURCE_DIR}\"")

macro(PROJECT_ADD_TEST TNAME)
    add_executable(${TNAME}
        cmptest.h
        ${TNAME}.c
    )

    target_link_libraries(${TNAME}
        libsodium-static
    )

    add_test(
        NAME ${TNAME}
        COMMAND ${TNAME}
    )
endmacro()

project_add_test(aead_aes256gcm)
project_add_test(aead_chacha20poly1305)
project_add_test(aead_xchacha20poly1305)
project_add_test(auth)
project_add_test(auth2)
project_add_test(auth3)
project_add_test(auth5)
project_add_test(auth6)
project_add_test(auth7)
project_add_test(box)
project_add_test(box2)
project_add_test(box7)
project_add_test(box8)
project_add_test(box_easy)
project_add_test(box_easy2)
project_add_test(box_seal)
project_add_test(box_seed)
project_add_test(chacha20)
project_add_test(core1)
project_add_test(core2)
project_add_test(core3)
project_add_test(core4)
project_add_test(core5)
project_add_test(core6)
project_add_test(ed25519_convert)
project_add_test(generichash)
project_add_test(generichash2)
project_add_test(generichash3)
project_add_test(hash)
project_add_test(hash3)
project_add_test(kdf)
project_add_test(keygen)
project_add_test(kx)
project_add_test(onetimeauth)
project_add_test(onetimeauth2)
project_add_test(onetimeauth7)
project_add_test(pwhash)
project_add_test(randombytes)
project_add_test(scalarmult)
project_add_test(scalarmult2)
project_add_test(scalarmult5)
project_add_test(scalarmult6)
project_add_test(scalarmult7)
project_add_test(secretbox)
project_add_test(secretbox2)
project_add_test(secretbox7)
project_add_test(secretbox8)
project_add_test(secretbox_easy)
project_add_test(secretbox_easy2)
project_add_test(shorthash)
project_add_test(sign)
project_add_test(sodium_core)
project_add_test(sodium_utils)
project_add_test(sodium_version)
project_add_test(stream)
project_add_test(stream2)
project_add_test(stream3)
project_add_test(stream4)
project_add_test(verify1)
