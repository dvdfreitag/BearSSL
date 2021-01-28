cmake_minimum_required(VERSION 3.6)

project(BearSSL LANGUAGES C ASM)

# Supported architectures
option(BEARSSL_ARCH_IS_X86_64 "BearSSL build targeting x86_64" OFF)
option(BEARSSL_ARCH_IS_X86 "BearSSL build targeting x86" OFF)
option(BEARSSL_ARCH_IS_POWER8 "BearSSL build targeting Power8" OFF)
option(BEARSSL_ARCH_IS_ARM "BearSSL build targeting arm-none-eabi" OFF)

# If the architecture isn't explicitly passed, attempt to identify it
if (NOT (BEARSSL_ARCH_IS_X86_64 OR BEARSSL_ARCH_IS_X86 OR BEARSSL_ARCH_IS_POWER8 OR BEARSSL_ARCH_IS_ARM))
	if(${CMAKE_C_COMPILER_TARGET} MATCHES "^(i?86-nt32.*|i?86.*|x86_64-x32.*|x32.*|x86_64.*x32)")
		set(BEARSSL_ARCH_IS_X86 ON)
	elseif(${CMAKE_C_COMPILER_TARGET} MATCHES "^(x86_64-nt64.*|^x86_64.*)")
		set(BEARSSL_ARCH_IS_X86_64 ON)
	elseif(${CMAKE_C_COMPILER_TARGET} MATCHES "^(powerpc64.*|ppc64.*)")
		set(BEARSSL_ARCH_IS_POWER8 ON)
	elseif(${CMAKE_C_COMPILER_TARGET} MATCHES "^arm.*-none-eabi")
		set(BEARSSL_ARCH_IS_ARM ON)
	else()
		message(FATAL_ERROR "Error: Unknown or unsupported architecture: ${CMAKE_C_COMPILER_TARGET}")
	endif()
endif()

# Get Native Path Separator
file(TO_NATIVE_PATH "/" separator)

if(BEARSSL_ARCH_IS_X86_64)
	include(${CMAKE_CURRENT_LIST_DIR}${separator}cmake${separator}x64.cmake)
endif()

if(BEARSSL_ARCH_IS_X86)
	include(${CMAKE_CURRENT_LIST_DIR}${separator}cmake${separator}x86.cmake)
endif()

if(BEARSSL_ARCH_IS_POWER8)
	include(${CMAKE_CURRENT_LIST_DIR}${separator}cmake${separator}power8.cmake)
endif()

if(BEARSSL_ARCH_IS_ARM)
	include(${CMAKE_CURRENT_LIST_DIR}${separator}cmake${separator}arm.cmake)
endif()

# Hash Functions
# Hash functions are used within the handshake, and also to support signature
#    generation and verification (in particular in certificates). TLS 1.0 and
#    1.1 requires both MD5 and SHA-1. TLS 1.2 uses either SHA-256 or SHA-384,
#    depending on the cipher suite.

option(BEARSSL_ENABLE_MD5					"MD5" OFF)
option(BEARSSL_ENABLE_SHA1					"SHA-1" OFF)
option(BEARSSL_ENABLE_SHA224				"SHA-224" OFF)
option(BEARSSL_ENABLE_SHA256				"SHA-256" OFF)
option(BEARSSL_ENABLE_SHA384				"SHA-384" OFF)
option(BEARSSL_ENABLE_SHA512				"SHA-512" OFF)

# Elliptic Curves
# Raw elliptic curve support is needed for ECDH and ECDHE cipher suites, and
#    for ECDSA signature generation or verification.

option(BEARSSL_ENABLE_EC_ALL_M15			"P-256, P-384, P-521, Curve25519, with m15 or i15 code" OFF)
option(BEARSSL_ENABLE_EC_ALL_M31			"P-256, P-384, P-521, Curve25519, with m31 or i31 code" OFF)
option(BEARSSL_ENABLE_EC_C25519_I15			"Curve25519, with i15 code" OFF)
option(BEARSSL_ENABLE_EC_C25519_I31			"Curve25519, with i31 code" OFF)
option(BEARSSL_ENABLE_EC_C25519_M15			"Curve25519, with m15 code" OFF)
option(BEARSSL_ENABLE_EC_C25519_M31			"Curve25519, with m31 code" OFF)
option(BEARSSL_ENABLE_EC_P256_M15			"P-256, with m15 code" OFF)
option(BEARSSL_ENABLE_EC_P256_M31			"P-256, with m31 code" OFF)
option(BEARSSL_ENABLE_EC_PRIME_I15			"P-256, P-384 and P-512, with i15 code" OFF)
option(BEARSSL_ENABLE_EC_PRIME_I31			"P-256, P-384 and P-512, with i31 code" OFF)

# ECDSA
# ECDSA signature generation and verification requires an elliptic curve
#    implementation (that supports at least one of the NIST curves: P-256,
#    P-384 and P-521), and an extra piece of code that performs the big integer
#    computations specific to ECDSA (as opposed to ECDH).

option(BEARSSL_ENABLE_ECDSA_I15_SIGN		"ECDSA signature generation, with i15 code" OFF)
option(BEARSSL_ENABLE_ECDSA_I15_VRFY		"ECDSA signature verification, with i15 code" OFF)
option(BEARSSL_ENABLE_ECDSA_I31_SIGN		"ECDSA signature generation, with i31 code" OFF)
option(BEARSSL_ENABLE_ECDSA_I31_VRFY		"ECDSA signature verification, with i31 code" OFF)

# RSA Signatures
# For ECDHE_RSA cipher suites, the server needs to compute an RSA signature,
#    and the client needs to verify that signature. Signature verification is also
#    needed for X.509 certificate validation (validation of the server’s certificate
#    on the client).
# For certificate-based client authentication (except for full static ECDH), the client
#    needs to compute a signature, and the server verifies it.

option(BEARSSL_ENABLE_RSA_I15_PKCS1_SIGN	"RSA signature generation, with i15 code" OFF)
option(BEARSSL_ENABLE_RSA_I15_PKCS1_VRFY	"RSA signature verification, with i15 code" OFF)
option(BEARSSL_ENABLE_RSA_I31_PKCS1_SIGN	"RSA signature generation, with i31 code" OFF)
option(BEARSSL_ENABLE_RSA_I31_PKCS1_VRFY	"RSA signature verification, with i31 code" OFF)
option(BEARSSL_ENABLE_RSA_I32_PKCS1_SIGN	"RSA signature generation, with i32 code" OFF)
option(BEARSSL_ENABLE_RSA_I32_PKCS1_VRFY	"RSA signature verification, with i32 code" OFF)
option(BEARSSL_ENABLE_RSA_I62_PKCS1_SIGN	"RSA signature generation, with i62 code" OFF)
option(BEARSSL_ENABLE_RSA_I62_PKCS1_VRFY	"RSA signature verification, with i62 code" OFF)

# RSA Encryption
# RSA cipher suites (not ECDH_RSA or ECDHE_RSA) use asymmetric encryption for key exchange.
#    The client needs an RSA <em>encryption</em> engine, and the server requires an RSA
#    decryption engine.

option(BEARSSL_ENABLE_RSA_I15_PRIV			"RSA decryption (private), with i15 code" OFF)
option(BEARSSL_ENABLE_RSA_I15_PUB			"RSA encryption (public), with i15 code" OFF)
option(BEARSSL_ENABLE_RSA_I31_PRIV			"RSA decryption (private), with i31 code" OFF)
option(BEARSSL_ENABLE_RSA_I31_PUB			"RSA encryption (public), with i31 code" OFF)
option(BEARSSL_ENABLE_RSA_I32_PRIV			"RSA decryption (private), with i32 code" OFF)
option(BEARSSL_ENABLE_RSA_I32_PUB			"RSA encryption (public), with i32 code" OFF)
option(BEARSSL_ENABLE_RSA_I62_PRIV			"RSA decryption (private), with i62 code" OFF)
option(BEARSSL_ENABLE_RSA_I62_PUB			"RSA encryption (public), with i62 code" OFF)

# AES
# AES_CBC cipher suites need an AES/CBC encryption engine and an AES/CBC decryption engine.
#    These need not use the same implementation, although this is usually a good idea since
#    the encryption and decryption engines for a given implementation tend to share some code.

# AES_GCM cipher suites need an AES/CTR engine.
#    The “big” and “small” implementations use tables and thus are _not_ constant-time.

option(BEARSSL_ENABLE_AES_BIG_CBCDEC		"AES/CBC decryption, “big” (fast, uses tables)" OFF)
option(BEARSSL_ENABLE_AES_BIG_CBCENC		"AES/CBC encryption, “big” (fast, uses tables)" OFF)
option(BEARSSL_ENABLE_AES_BIG_CTR			"AES/CTR, “big” (fast, uses tables)" OFF)
option(BEARSSL_ENABLE_AES_SMALL_CBCDEC		"AES/CBC decryption, “small” (uses tables)" OFF)
option(BEARSSL_ENABLE_AES_SMALL_CBCENC		"AES/CBC encryption, “small” (uses tables)" OFF)
option(BEARSSL_ENABLE_AES_SMALL_CTR			"AES/CTR, “small” (uses tables)" OFF)
option(BEARSSL_ENABLE_AES_CT_CBCDEC			"AES/CBC decryption, “ct” (constant-time, for 32-bit architectures)" OFF)
option(BEARSSL_ENABLE_AES_CT_CBCENC			"AES/CBC encryption, “ct” (constant-time, for 32-bit architectures)" OFF)
option(BEARSSL_ENABLE_AES_CT_CTR			"AES/CTR, “ct” (constant-time, for 32-bit architectures)" OFF)
option(BEARSSL_ENABLE_AES_CT64_CBCDEC		"AES/CBC decryption, “ct64” (constant-time, for 64-bit architectures)" OFF)
option(BEARSSL_ENABLE_AES_CT64_CBCENC		"AES/CBC encryption, “ct64” (constant-time, for 64-bit architectures)" OFF)
option(BEARSSL_ENABLE_AES_CT64_CTR			"AES/CTR, “ct64” (constant-time, for 64-bit architectures)" OFF)
option(BEARSSL_ENABLE_AES_PWR8_CBCDEC		"AES/CBC decryption, “pwr8” (with POWER8 crypto opcodes)" OFF)
option(BEARSSL_ENABLE_AES_PWR8_CBCENC		"AES/CBC encryption, “pwr8” (with POWER8 crypto opcodes)" OFF)
option(BEARSSL_ENABLE_AES_PWR8_CTR			"AES/CTR, “pwr8” (with POWER8 crypto opcodes)" OFF)
option(BEARSSL_ENABLE_AES_X86NI_CBCDEC		"AES/CBC decryption, “x86ni” (with AES-NI opcodes)" OFF)
option(BEARSSL_ENABLE_AES_X86NI_CBCENC		"AES/CBC encryption, “x86ni” (with AES-NI opcodes)" OFF)
option(BEARSSL_ENABLE_AES_X86NI_CTR			"AES/CTR, “x86ni” (with AES-NI opcodes)" OFF)

# GHASH
# A GHASH implementation is needed for AES/GCM cipher suites.

option(BEARSSL_ENABLE_GHASH_CTMUL			"GHASH with 32→64 multiplications" OFF)
option(BEARSSL_ENABLE_GHASH_CTMUL32			"GHASH with 32→32 multiplications" OFF)
option(BEARSSL_ENABLE_GHASH_CTMUL64			"GHASH with 64→64 multiplications" OFF)
option(BEARSSL_ENABLE_GHASH_PCLMUL			"GHASH with AES-NI opcodes (pclmulqdq)" OFF)
option(BEARSSL_ENABLE_GHASH_PWR8			"GHASH with POWER8 crypto opcodes" OFF)

# 3DES
# 3DES cipher suites (“3DES_EDE_CBC”) need a 3DES/CBC encryption engine and a 3DES/CBC
#    decryption engine. The “tab” implementation is faster than “ct”; however, “ct” is
#    constant-time while “tab” is not.

option(BEARSSL_ENABLE_DES_TAB_CBCDEC		"3DES/CBC decryption, “tab” (uses tables)" OFF)
option(BEARSSL_ENABLE_DES_TAB_CBCENC		"3DES/CBC encryption, “tab” (uses tables)" OFF)
option(BEARSSL_ENABLE_DES_CT_CBCDEC			"3DES/CBC decryption, “ct” (constant-time)" OFF)
option(BEARSSL_ENABLE_DES_CT_CBCENC			"3DES/CBC encryption, “ct” (constant-time)" OFF)

# ChaCha20
# ChaCha20+Poly1305 cipher suites require a ChaCha20 implementation.

option(BEARSSL_ENABLE_CHACHA20_CT			"ChaCha20, “ct”" OFF)

# Poly1305
# ChaCha20+Poly1305 cipher suites require a Poly1305 implementation.

option(BEARSSL_ENABLE_POLY1305_CTMUL		"Poly1305, “ctmul” (with 32→64 multiplications)" OFF)
option(BEARSSL_ENABLE_POLY1305_CTMUL32		"Poly1305, “ctmul32” (with 32→32 multiplications)" OFF)
option(BEARSSL_ENABLE_POLY1305_CTMULQ		"Poly1305, “ctmulq” (with 64→128 multiplications)" OFF)
option(BEARSSL_ENABLE_POLY1305_I15			"Poly1305, “i15” (with i15 big integer code)" OFF)

# SSL Handshake
# SSL client and server share a common processing engine, but each has specific code for
#    handling handshake messages.

option(BEARSSL_ENABLE_SSL_CLIENT			"SSL client engine" OFF)
option(BEARSSL_ENABLE_SSL_SERVER			"SSL server engine" OFF)

# SSL Records
# The record processing code (encryption+MAC) depends on the type of encryption used.

option(BEARSSL_ENABLE_SSLREC_CBC			"SSL records with CBC+HMAC encryption/decryption" OFF)
option(BEARSSL_ENABLE_SSLREC_CHAPOL			"SSL records with ChaCha20+Poly1305 encryption/decryption" OFF)
option(BEARSSL_ENABLE_SSLREC_GCM			"SSL records with GCM encryption/decryption" OFF)

# PRF
# TLS 1.0 and 1.1 use a PRF function that relies on MD5 and SHA-1.
# TLS 1.2 uses a PRF that relies on either SHA-256 or SHA-384, depending on the cipher suite.

option(BEARSSL_ENABLE_PRF_TLS10				"PRF for TLS 1.0 and 1.1 (uses MD5 and SHA-1)" OFF)
option(BEARSSL_ENABLE_PRF_TLS12_SHA256		"PRF for TLS 1.2 with SHA-256" OFF)
option(BEARSSL_ENABLE_PRF_TLS12_SHA384		"PRF for TLS 1.2 with SHA-384" OFF)

# Certificate
# The server must send a certificate that contains its public key, and use the corresponding
#    private key. Similarly, when client certificates are used, the client must send a certificate
#    and use the corresponding private key. The elements below contain the code that offers a
#    single certificate chain and uses the private key.

option(BEARSSL_ENABLE_SSL_CCERT_EC			"SSL client certificate authentication (ECDH/ECDSA)" OFF)
option(BEARSSL_ENABLE_SSL_CCERT_RSA			"SSL client certificate authentication (RSA)" OFF)
option(BEARSSL_ENABLE_SSL_SCERT_EC			"SSL server certificate (ECDH/ECDSA)" OFF)
option(BEARSSL_ENABLE_SSL_SCERT_RSA			"SSL server certificate (RSA)" OFF)

# X.509 Validation
# X.509 validation is used to extract the public key sent by the peer. Clients need an X.509 engine;
#    if client certificates are used, then servers also need an X.509 engine.
# The “knownkey” engine ignores the received certificate chain, and returns an hardcoded public key.
# The “minimal” engine performs basic chain validation, which entails verifying signatures;
#    which signatures are supported depends on the configured hash functions and signature verification engines.

option(BEARSSL_ENABLE_X509_KNOWNKEY			"X.509 validation (pseudo: public key already known)" OFF)
option(BEARSSL_ENABLE_X509_MINIMAL			"X.509 validation (single chain processing)" OFF)

set(BEARSSL_CHECK_ALGOS
	AES_BIG_CBCDEC
	AES_BIG_CBCENC
	AES_BIG_CTR
	AES_CT_CBCDEC
	AES_CT_CBCENC
	AES_CT_CTR
	AES_CT64_CBCDEC
	AES_CT64_CBCENC
	AES_CT64_CTR
	AES_PWR8_CBCDEC
	AES_PWR8_CBCENC
	AES_PWR8_CTR
	AES_SMALL_CBCDEC
	AES_SMALL_CBCENC
	AES_SMALL_CTR
	AES_X86NI_CBCDEC
	AES_X86NI_CBCENC
	AES_X86NI_CTR
	CHACHA20_CT
	DES_CT_CBCDEC
	DES_CT_CBCENC
	DES_TAB_CBCDEC
	DES_TAB_CBCENC
	EC_ALL_M15
	EC_ALL_M31
	EC_C25519_I15
	EC_C25519_I31
	EC_C25519_M15
	EC_C25519_M31
	EC_P256_M15
	EC_P256_M31
	EC_PRIME_I15
	EC_PRIME_I31
	ECDSA_I15_SIGN
	ECDSA_I15_VRFY
	ECDSA_I31_SIGN
	ECDSA_I31_VRFY
	GHASH_CTMUL
	GHASH_CTMUL32
	GHASH_CTMUL64
	GHASH_PCLMUL
	GHASH_PWR8
	MD5
	POLY1305_CTMUL
	POLY1305_CTMUL32
	POLY1305_CTMULQ
	POLY1305_I15
	RSA_I15_PKCS1_SIGN
	RSA_I15_PKCS1_VRFY
	RSA_I15_PRIV
	RSA_I15_PUB
	RSA_I31_PKCS1_SIGN
	RSA_I31_PKCS1_VRFY
	RSA_I31_PRIV
	RSA_I31_PUB
	RSA_I32_PKCS1_SIGN
	RSA_I32_PKCS1_VRFY
	RSA_I32_PRIV
	RSA_I32_PUB
	RSA_I62_PKCS1_SIGN
	RSA_I62_PKCS1_VRFY
	RSA_I62_PRIV
	RSA_I62_PUB
	SHA1
	SHA224
	SHA256
	SHA384
	SHA512
	SSL_CLIENT
	SSL_CCERT_EC
	SSL_CCERT_RSA
	SSL_SERVER
	SSL_SCERT_EC
	SSL_SCERT_RSA
	SSLREC_CBC
	SSLREC_CHAPOL
	SSLREC_GCM
	PRF_TLS10
	PRF_TLS12_SHA256
	PRF_TLS12_SHA384
	X509_KNOWNKEY
	X509_MINIMAL
	CACHE INTERNAL ""
)

set(BEARSSL_ALGO_FEATURES_EC_ALL_M15			"ec_p256" "ec_p384" "ec_p521" "ec_c25519" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_ALL_M31			"ec_p256" "ec_p384" "ec_p521" "ec_c25519" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_C25519_I15			"ec_c25519" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_C25519_I31			"ec_c25519" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_C25519_M15			"ec_c25519" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_C25519_M31			"ec_c25519" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_P256_M15			"ec_p256" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_P256_M31			"ec_p256" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_PRIME_I15			"ec_p256" "ec_p384" "ec_p521" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_EC_PRIME_I31			"ec_p256" "ec_p384" "ec_p521" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_ECDSA_I15_SIGN		"ecdsa_sign" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_ECDSA_I15_VRFY		"ecdsa_vrfy" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_ECDSA_I31_SIGN		"ecdsa_sign" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_ECDSA_I31_VRFY		"ecdsa_vrfy" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_MD5					"md5" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SHA1					"sha1" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SHA224				"sha224" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SHA256				"sha256" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SHA384				"sha384" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SHA512				"sha512" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_GHASH_CTMUL			"ghash" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_GHASH_CTMUL32			"ghash" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_GHASH_CTMUL64			"ghash" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_GHASH_PCLMUL			"ghash" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_GHASH_PWR8			"ghash" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I15_PKCS1_SIGN	"rsa_sign" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I15_PKCS1_VRFY	"rsa_vrfy" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I15_PRIV			"rsa_priv" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I15_PUB			"rsa_pub" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I31_PKCS1_SIGN	"rsa_sign" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I31_PKCS1_VRFY	"rsa_vrfy" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I31_PRIV			"rsa_priv" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I31_PUB			"rsa_pub" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I32_PKCS1_SIGN	"rsa_sign" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I32_PKCS1_VRFY	"rsa_vrfy" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I32_PRIV			"rsa_priv" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I32_PUB			"rsa_pub" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I62_PKCS1_SIGN	"rsa_sign" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I62_PKCS1_VRFY	"rsa_vrfy" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I62_PRIV			"rsa_priv" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_RSA_I62_PUB			"rsa_pub" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_BIG_CBCDEC		"aes_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_BIG_CBCENC		"aes_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_BIG_CTR			"aes_ctr" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_CT_CBCDEC			"aes_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_CT_CBCENC			"aes_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_CT_CTR			"aes_ctr" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_CT64_CBCDEC		"aes_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_CT64_CBCENC		"aes_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_CT64_CTR			"aes_ctr" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_PWR8_CBCDEC		"aes_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_PWR8_CBCENC		"aes_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_PWR8_CTR			"aes_ctr" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_SMALL_CBCDEC		"aes_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_SMALL_CBCENC		"aes_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_SMALL_CTR			"aes_ctr" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_X86NI_CBCDEC		"aes_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_X86NI_CBCENC		"aes_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_AES_X86NI_CTR			"aes_ctr" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_DES_TAB_CBCDEC		"des_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_DES_TAB_CBCENC		"des_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_DES_CT_CBCDEC			"des_cbcdec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_DES_CT_CBCENC			"des_cbcenc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_CHACHA20_CT			"chacha20" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_POLY1305_CTMUL		"poly1305" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_POLY1305_CTMUL32		"poly1305" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_POLY1305_CTMULQ		"poly1305" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_POLY1305_I15			"poly1305" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSL_CLIENT			"ssl_client" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSL_SERVER			"ssl_server" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSLREC_CBC			"sslrec_cbc" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSLREC_CHAPOL			"sslrec_chapol" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSLREC_GCM			"sslrec_gcm" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_PRF_TLS10				"prf_tls10" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_PRF_TLS12_SHA256		"prf_tls12_sha256" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_PRF_TLS12_SHA384		"prf_tls12_sha384" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSL_CCERT_EC			"ssl_ccert_ec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSL_CCERT_RSA			"ssl_ccert_rsa" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSL_SCERT_EC			"ssl_scert_ec" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_SSL_SCERT_RSA			"ssl_scert_rsa" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_X509_KNOWNKEY			"x509" CACHE INTERNAL "")
set(BEARSSL_ALGO_FEATURES_X509_MINIMAL			"x509" CACHE INTERNAL "")

set(BEARSSL_CIPHERSUITES
	"0024TLS_RSA_WITH_3DES_EDE_CBC_SHA"
	"0124TLS_RSA_WITH_AES_128_CBC_SHA"
	"0224TLS_RSA_WITH_AES_256_CBC_SHA"
	"0144TLS_RSA_WITH_AES_128_CBC_SHA256"
	"0244TLS_RSA_WITH_AES_256_CBC_SHA256"
	"0304TLS_RSA_WITH_AES_128_GCM_SHA256"
	"0405TLS_RSA_WITH_AES_256_GCM_SHA384"
	"4024TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA"
	"4124TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA"
	"4224TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA"
	"2024TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA"
	"2124TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA"
	"2224TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA"
	"3024TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA"
	"3124TLS_ECDH_RSA_WITH_AES_128_CBC_SHA"
	"3224TLS_ECDH_RSA_WITH_AES_256_CBC_SHA"
	"1024TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA"
	"1124TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA"
	"1224TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA"
	"2144TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256"
	"2255TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384"
	"4144TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256"
	"4255TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384"
	"1144TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256"
	"1255TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384"
	"3144TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256"
	"3255TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384"
	"2304TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256"
	"2405TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384"
	"4304TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256"
	"4405TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384"
	"1304TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
	"1405TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
	"3304TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256"
	"3405TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384"
	"1504TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256"
	"2504TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256"
	CACHE INTERNAL ""
)

set(BEARSSL_CIPHERSUITES_CLIENT_TLS10_ENABLED "" CACHE INTERNAL "")
set(BEARSSL_CIPHERSUITES_CLIENT_TLS12_ENABLED "" CACHE INTERNAL "")
set(BEARSSL_CIPHERSUITES_SERVER_TLS10_ENABLED "" CACHE INTERNAL "")
set(BEARSSL_CIPHERSUITES_SERVER_TLS12_ENABLED "" CACHE INTERNAL "")

set(BEARSSL_FEATURES "" CACHE INTERNAL "")

function(check_impl algorithm)
	if(${BEARSSL_ENABLE_${algorithm}})
		if(${BEARSSL_ARCH_${algorithm}_ENABLED})
			set(BEARSSL_FEATURES
				"${BEARSSL_FEATURES}"
				"${BEARSSL_ALGO_FEATURES_${algorithm}}"
				PARENT_SCOPE
			)
		else()
			message(FATAL_ERROR "Architecture has unsupported feature: " ${algorithm})
		endif()
	endif()
endfunction()

foreach(algorithm ${BEARSSL_CHECK_ALGOS})
	check_impl("${algorithm}")
endforeach()

list(REMOVE_DUPLICATES BEARSSL_FEATURES)

list(FIND BEARSSL_FEATURES "ec_p256" ec_p256)
list(FIND BEARSSL_FEATURES "ec_p384" ec_p384)
list(FIND BEARSSL_FEATURES "ec_p521" ec_p521)

if(ec_p256 OR ec_p384 OR ec_p521)
	list(APPEND BEARSSL_FEATURES "ec_nist")
else()
	list(REMOVE_ITEM BEARSSL_FEATURES "ecdsa_sign")
	list(REMOVE_ITEM BEARSSL_FEATURES "ecdsa_vrfy")
endif()

list(FIND BEARSSL_FEATURES "ec_nist" ec_nist)
list(FIND BEARSSL_FEATURES "ec_c25519" ec_c25519)

if(ec_nist OR ec_c22519)
	list(APPEND BEARSSL_FEATURES "ecdhe")
endif()

foreach(ciphersuite ${BEARSSL_CIPHERSUITES})
	string(REGEX MATCH "(.)(.)(.)(.)(.+)" match ${ciphersuite})

	set(a ${CMAKE_MATCH_1})
	set(b ${CMAKE_MATCH_2})
	set(c ${CMAKE_MATCH_3})
	set(d ${CMAKE_MATCH_4})

	set(suite ${CMAKE_MATCH_5})

	set(CLIENT_TLS10 ON)
	set(CLIENT_TLS12 ON)
	set(SERVER_TLS10 ON)
	set(SERVER_TLS12 ON)

	if(${a} STREQUAL "0")							# RSA
		if(NOT "rsa_priv" IN_LIST BEARSSL_FEATURES OR NOT "ssl_scert_rsa" IN_LIST BEARSSL_FEATURES)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		endif()

		if(NOT "rsa_pub" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
		endif()
	elseif(${a} STREQUAL "1")						# ECDHE-RSA
		if(NOT "ecdhe" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		else()
			if(NOT "rsa_sign" IN_LIST BEARSSL_FEATURES OR NOT "ssl_scert_rsa" IN_LIST BEARSSL_FEATURES)
				set(SERVER_TLS10 OFF)
				set(SERVER_TLS12 OFF)
			endif()

			if(NOT "rsa_vrfy" IN_LIST BEARSSL_FEATURES)
				set(CLIENT_TLS10 OFF)
				set(CLIENT_TLS12 OFF)
			endif()
		endif()
	elseif(${a} STREQUAL "2")						# ECDHE-ECDSA
		if(NOT "ecdhe" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		else()
			if(NOT "ecdsa_sign" IN_LIST BEARSSL_FEATURES OR NOT "ssl_scert_ec" IN_LIST BEARSSL_FEATURES)
				set(SERVER_TLS10 OFF)
				set(SERVER_TLS12 OFF)
			endif()

			if(NOT "ecdsa_vrfy" IN_LIST BEARSSL_FEATURES)
				set(CLIENT_TLS10 OFF)
				set(CLIENT_TLS12 OFF)
			endif()
		endif()

	elseif(${a} STREQUAL "3" OR ${a} STREQUAL "4")	# ECDH-RSA / ECDH-ECDSA
		if(NOT "ssl_scert_ec" IN_LIST BEARSSL_FEATURES)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		endif()

		if(NOT "ecdhe" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
		endif()
	endif()

	if(${b} STREQUAL "0")							# 3DES/CBC
		if(NOT "des_cbcdec" IN_LIST BEARSSL_FEATURES OR NOT "des_cbcenc" IN_LIST BEARSSL_FEATURES OR NOT "sslrec_cbc" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	elseif(${b} STREQUAL "1" OR ${b} STREQUAL "2")	# AES-(128|256)/CBC
		if (NOT "aes_cbcdec" IN_LIST BEARSSL_FEATURES OR NOT "aes_cbcenc" IN_LIST BEARSSL_FEATURES OR NOT "sslrec_cbc" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	elseif(${b} STREQUAL "3" OR ${b} STREQUAL "4")	# AES-(128|256)/GCM
		set(CLIENT_TLS10 OFF)
		set(SERVER_TLS10 OFF)

		if(NOT "aes_ctr" IN_LIST BEARSSL_FEATURES OR NOT "ghash" IN_LIST BEARSSL_FEATURES OR NOT "sslrec_gcm" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	elseif(${b} STREQUAL "5")						# ChaCha20/Poly1305
		set(CLIENT_TLS10 OFF)
		set(SERVER_TLS10 OFF)

		if(NOT "chacha20" IN_LIST BEARSSL_FEATURES OR NOT "poly1305" IN_LIST BEARSSL_FEATURES OR NOT "sslrec_chapol" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	endif()

	if(${c} STREQUAL "0")							# no MAC (AEAD cipher suite)
		set(CLIENT_TLS10 OFF)
		set(SERVER_TLS10 OFF)
	elseif(${c} STREQUAL "1")						# HMAC/SHA-1
		if(NOT "sha1" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS10 OFF)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS10 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	elseif(${c} STREQUAL "4")						# HMAC/SHA-256
		set(CLIENT_TLS10 OFF)
		set(SERVER_TLS10 OFF)

		if(NOT "sha256" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	elseif(${c} STREQUAL "5")						# HMAC/SHA-384
		set(CLIENT_TLS10 OFF)
		set(SERVER_TLS10 OFF)

		if(NOT "sha384" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	endif()

	if(${d} STREQUAL "4")							# TLS 1.2 SHA-256
		if(NOT "prf_tls12_sha256" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	elseif(${d} STREQUAL "5")						# TLS 1.2 SHA-384
		if(NOT "prf_tls12_sha384" IN_LIST BEARSSL_FEATURES)
			set(CLIENT_TLS12 OFF)
			set(SERVER_TLS12 OFF)
		endif()
	endif()

	if(${CLIENT_TLS10})
		list(APPEND BEARSSL_CIPHERSUITES_CLIENT_TLS10_ENABLED ${suite})
	endif()

	if(${CLIENT_TLS12})
		list(APPEND BEARSSL_CIPHERSUITES_CLIENT_TLS12_ENABLED ${suite})
	endif()

	if(${SERVER_TLS10})
		list(APPEND BEARSSL_CIPHERSUITES_SERVER_TLS10_ENABLED ${suite})
	endif()

	if(${SERVER_TLS12})
		list(APPEND BEARSSL_CIPHERSUITES_SERVER_TLS12_ENABLED ${suite})
	endif()
endforeach()

message(STATUS "Supported cipher suites as SSL client:")

message(STATUS "    - TLS 1.0 and 1.1:")

if(BEARSSL_CIPHERSUITES_CLIENT_TLS10_ENABLED)
	foreach(ciphersuite ${BEARSSL_CIPHERSUITES_CLIENT_TLS10_ENABLED})
		message(STATUS "        - " ${ciphersuite})
	endforeach()
else()
	message(STATUS "        - not supported")
endif()

message(STATUS "    - TLS 1.2:")

if(BEARSSL_CIPHERSUITES_CLIENT_TLS12_ENABLED)
	foreach(ciphersuite ${BEARSSL_CIPHERSUITES_CLIENT_TLS12_ENABLED})
		message(STATUS "        - " ${ciphersuite})
	endforeach()
else()
	message(STATUS "        - not supported")
endif()

message(STATUS "Supported cipher suites as SSL server:")

message(STATUS "    - TLS 1.0 and 1.1:")

if(BEARSSL_CIPHERSUITES_SERVER_TLS10_ENABLED)
	foreach(ciphersuite ${BEARSSL_CIPHERSUITES_SERVER_TLS10_ENABLED})
		message(STATUS "        - " ${ciphersuite})
	endforeach()
else()
	message(STATUS "        - not supported")
endif()

message(STATUS "    - TLS 1.2:")

if(BEARSSL_CIPHERSUITES_SERVER_TLS12_ENABLED)
	foreach(ciphersuite ${BEARSSL_CIPHERSUITES_SERVER_TLS12_ENABLED})
		message(STATUS "        - " ${ciphersuite})
	endforeach()
else()
	message(STATUS "        - not supported")
endif()

# Get Absolute Path to source and include directories
set(BEARSSL_SRCDIR ${CMAKE_CURRENT_LIST_DIR}${separator}src)
set(BEARSSL_INCDIR ${CMAKE_CURRENT_LIST_DIR}${separator}inc)
# Get all C source files
file(GLOB_RECURSE BEARSSL_SRCS ${BEARSSL_SRCDIR}${separator}*.c)
# Get all public headers
file(GLOB BEARSSL_INCLUDES ${BEARSSL_INCDIR}${separator}*.h)

add_library(bearssl STATIC ${BEARSSL_SRCS})

target_include_directories(bearssl PUBLIC ${BEARSSL_INCDIR})
target_include_directories(bearssl PRIVATE ${BEARSSL_SRCDIR})

set_target_properties(bearssl PROPERTIES PUBLIC_HEADER "${BEARSSL_INCLUDES}")

install(TARGETS bearssl
	ARCHIVE
		DESTINATION lib
	PUBLIC_HEADER
		DESTINATION include${separator}BearSSL
)
