if (DEFINED ENV{PICO_MBEDTLS_PATH} AND (NOT PICO_MBEDTLS_PATH))
    set(PICO_MBEDTLS_PATH $ENV{PICO_MBEDTLS_PATH})
    message("Using PICO_MBEDTLS_PATH from environment ('${PICO_MBEDTLS_PATH}')")
endif()

set(MBEDTLS_TEST_PATH "library/aes.c")
if (NOT PICO_MBEDTLS_PATH)
    set(PICO_MBEDTLS_PATH ${PROJECT_SOURCE_DIR}/lib/mbedtls)
elseif (NOT EXISTS ${PICO_MBEDTLS_PATH}/${MBEDTLS_TEST_PATH})
    message(WARNING "PICO_MBEDTLS_PATH specified but content not present.")
endif()

if (EXISTS ${PICO_MBEDTLS_PATH}/${MBEDTLS_TEST_PATH})
    message("mbedtls available at ${PICO_MBEDTLS_PATH}")

    pico_register_common_scope_var(PICO_MBEDTLS_PATH)

    set(src_crypto
        aes.c
        aes256ctr.c
        aesni.c
        arc4.c
        aria.c
        asn1parse.c
        asn1write.c
        base64.c
        bignum.c
        blowfish.c
        camellia.c
        ccm.c
        chacha20.c
        chachapoly.c
        cipher.c
        cipher_wrap.c
        cmac.c
        ctr_drbg.c
        des.c
        dhm.c
        dilithium.c
        dilithium_ntt.c
        dilithium_packing.c
        dilithium_poly.c
        dilithium_polyvec.c
        dilithium_reduce.c
        dilithium_rounding.c
        dilithium_symmetric-aes.c
        dilithium_symmetric-shake.c
        dilithium_fips202.c
        fips202.c
        ecdh.c
        ecdsa.c
        ecjpake.c
        ecp.c
        ecp_curves.c
        entropy.c
        entropy_poll.c
        error.c
        gcm.c
        havege.c
        hkdf.c
        hmac_drbg.c
        kyber_cbd.c
        kyber_indcpa.c
        kyber_kem.c
        kyber_ntt.c
        kyber_poly.c
        kyber_polyvec.c
        kyber_reduce.c
        kyber_verify.c
        kyber_symmetric-shake.c
        md.c
        md2.c
        md4.c
        md5.c
        md_wrap.c
        memory_buffer_alloc.c
        nist_kw.c
        oid.c
        padlock.c
        pem.c
        pk.c
        pk_wrap.c
        pkcs12.c
        pkcs5.c
        pkparse.c
        pkwrite.c
        platform.c
        platform_util.c
        poly1305.c
        ripemd160.c
        rsa.c
        rsa_internal.c
        saber_cbd.c
        saber_fips202.c
        saber_indcpa.c
        saber_kem.c
        saber_pack_unpack.c
        saber_poly.c
        saber_poly_mul.c
        saber_verify.c
        sha1.c
        sha256.c
        sha512.c
        shake256.c
        spx.c
        spx_fors.c
        #spx_haraka.c
        spx_hash_address.c
        #spx_hash_haraka.c
        #spx_hash_sha2.c
        spx_hash_sha256.c
        spx_hash_shake.c
        #spx_hash_shake256.c
        spx_merkle.c
        #spx_thash_haraka_robust.c
        #spx_thash_haraka_simple.c
        #spx_thash_sha2_robust.c
        #spx_thash_sha2_simple.c
        spx_thash_shake_robust.c
        #spx_thash_shake_simple.c
        spx_utils.c
        spx_utilsx1.c
        spx_wots.c
        spx_wotsx1.c
        threading.c
        timing.c
        version.c
        version_features.c
        xtea.c
    )
    list(TRANSFORM src_crypto PREPEND ${PICO_MBEDTLS_PATH}/library/)
    pico_add_library(pico_mbedtls_crypto NOFLAG)
    target_sources(pico_mbedtls_crypto INTERFACE ${src_crypto})

    set(src_x509
        certs.c
        pkcs11.c
        x509.c
        x509_create.c
        x509_crl.c
        x509_crt.c
        x509_csr.c
        x509write_crt.c
        x509write_csr.c
    )
    list(TRANSFORM src_x509 PREPEND ${PICO_MBEDTLS_PATH}/library/)
    pico_add_library(pico_mbedtls_x509 NOFLAG)
    target_sources(pico_mbedtls_x509 INTERFACE ${src_x509})

    set(src_tls
        debug.c
        net_sockets.c
        ssl_cache.c
        ssl_ciphersuites.c
        ssl_cli.c
        ssl_cookie.c
        ssl_srv.c
        ssl_ticket.c
        ssl_tls.c
    )
    list(TRANSFORM src_tls PREPEND ${PICO_MBEDTLS_PATH}/library/)
    pico_add_library(pico_mbedtls_tls NOFLAG)
    target_sources(pico_mbedtls_tls INTERFACE ${src_tls})

    pico_add_library(pico_mbedtls NOFLAG)
    pico_mirrored_target_link_libraries(pico_mbedtls INTERFACE pico_mbedtls_crypto pico_mbedtls_x509 pico_mbedtls_tls pico_rand)
    if (DEFINED PICO_MBEDTLS_CONFIG_FILE)
        target_compile_definitions(pico_mbedtls_headers INTERFACE MBEDTLS_CONFIG_FILE="${PICO_MBEDTLS_CONFIG_FILE}")
    else()
        target_compile_definitions(pico_mbedtls_headers INTERFACE MBEDTLS_CONFIG_FILE="mbedtls_config.h")
    endif()
    target_sources(pico_mbedtls INTERFACE ${CMAKE_CURRENT_LIST_DIR}/pico_mbedtls.c)
    target_include_directories(pico_mbedtls_headers INTERFACE ${PICO_MBEDTLS_PATH}/include/ ${PICO_MBEDTLS_PATH}/library/)

    function(suppress_mbedtls_warnings)
        set_source_files_properties(
            ${PICO_MBEDTLS_PATH}/library/ecdsa.c
            ${PICO_MBEDTLS_PATH}/library/ecp.c
            ${PICO_MBEDTLS_PATH}/library/ecp_curves.c
            ${PICO_MBEDTLS_PATH}/library/pk_wrap.c
            ${PICO_MBEDTLS_PATH}/library/pkparse.c
            ${PICO_MBEDTLS_PATH}/library/ssl_cli.c
            PROPERTIES
            COMPILE_OPTIONS "-Wno-cast-qual"
        )
        set_source_files_properties(
            ${PICO_MBEDTLS_PATH}/library/psa_crypto_client.c
            ${PICO_MBEDTLS_PATH}/library/psa_crypto_driver_wrappers.c
            PROPERTIES
            COMPILE_OPTIONS "-Wno-redundant-decls"
        )
        set_source_files_properties(
            ${PICO_MBEDTLS_PATH}/library/x509_crt.c
            PROPERTIES
            COMPILE_OPTIONS "-Wno-cast-qual;-Wno-null-dereference"
        )
        set_source_files_properties(
            ${PICO_MBEDTLS_PATH}/library/ssl_srv.c
            ${PICO_MBEDTLS_PATH}/library/ssl_tls.c
            PROPERTIES
            COMPILE_OPTIONS "-Wno-null-dereference"
        )
    endfunction()

    pico_promote_common_scope_vars()
endif()
