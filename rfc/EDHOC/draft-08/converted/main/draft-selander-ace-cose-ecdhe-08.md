Internet-Draft draft-selander-ace-cose-ecdhe-08 · Category: std · 05 March 2018

# Ephemeral Diffie-Hellman Over COSE (EDHOC)

Abstract

This document specifies Ephemeral Diffie-Hellman Over COSE (EDHOC), a compact, and lightweight authenticated Diffie-Hellman key exchange with ephemeral keys that can be used over any layer. EDHOC messages are encoded with CBOR and COSE, allowing reuse of existing libraries.

## 1 Introduction

Security at the application layer provides an attractive option for protecting Internet of Things (IoT) deployments, for example where transport layer security is not sufficient [I-D.hartke-core-e2e-security-reqs] or where the protocol needs to work on a variety of underlying protocols. IoT devices may be constrained in various ways, including memory, storage, processing capacity, and energy [RFC7228]. A method for protecting individual messages at the application layer suitable for constrained devices, is provided by CBOR Object Signing and Encryption (COSE) [RFC8152]), which builds on the Concise Binary Object Representation (CBOR) [RFC7049].

In order for a communication session to provide forward secrecy, the communicating parties can run an Elliptic Curve Diffie-Hellman (ECDH) key exchange protocol with ephemeral keys, from which shared key material can be derived. This document specifies Ephemeral Diffie-Hellman Over COSE (EDHOC), an authenticated ECDH protocol using CBOR and COSE objects. Authentication is based on credentials established out of band, e.g. from a trusted third party, such as an Authorization Server as specified by [I-D.ietf-ace-oauth-authz]. EDHOC supports authentication using pre-shared keys (PSK), raw public keys (RPK), and certificates (Cert). Note that this document focuses on authentication and key establishment: for integration with authorization of resource access, refer to [I-D.ietf-ace-oscore-profile]. This document also specifies the derivation of shared key material.

The ECDH exchange and the key derivation follow [SIGMA], NIST SP-800-56a [SP-800-56a], and HKDF [RFC5869]. CBOR [RFC7049] and COSE [RFC8152] are used to implement these standards.

### 1.1 Terminology

This document use the same informational CBOR Data Definition Language (CDDL) [I-D.ietf-cbor-cddl] grammar as COSE (see Section 1.3 of [RFC8152]). A vertical bar | denotes byte string concatenation.

### 1.2 Requirements Language

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [RFC2119]. These words may also appear in this document in lowercase, absent their normative meanings.

## 2 Protocol Overview

SIGMA (SIGn-and-MAc) is a family of theoretical protocols with a large number of variants [SIGMA]. Like IKEv2 and TLS 1.3, EDHOC is built on a variant of the SIGMA protocol which provide identity protection, and like TLS 1.3, EDHOC implements the SIGMA-I variant as Sign-then-MAC. The SIGMA-I protocol using an AEAD algorithm is shown in Figure 1.

```
Party U                                                 Party V
   |                          E_U                          |
   +------------------------------------------------------>|
   |                                                       |
   |         E_V, Enc(K_2; ID_V, Sig(V; E_U, E_V);)        |
   |<------------------------------------------------------+
   |                                                       |
   |           Enc(K_3; ID_U, Sig(U; E_V, E_U);)           |
   +------------------------------------------------------>|
   |                                                       |
```

Figure 1: AEAD variant of the SIGMA-I protocol

The parties exchanging messages are called “U” and “V”. They exchange identities and ephemeral public keys, compute the shared secret, and derive the keying material. The messages are signed, MACed, and encrypted.

- E_U and E_V are the ECDH ephemeral public keys of U and V, respectively.
- ID_U and ID_V are identifiers for the public keys of U and V, respectively.
- Sig(U; . ) and S(V; . ) denote signatures made with the private key of U and V, respectively.
- Enc(K; P; A) denotes AEAD encryption of plaintext P and additional authenticated data A using the key K derived from the shared secret. The AEAD MUST NOT be replaced by plain encryption, see Section 8.

As described in Appendix B of [SIGMA], in order to create a “full-fledged” protocol some additional protocol elements are needed. EDHOC adds:

- Explicit session identifiers S_U, S_V different from other concurrent session identifiers (EDHOC or other used protocol identifier) chosen by U and V, respectively.
- Explicit nonces N_U, N_V chosen freshly and anew with each session by U and V, respectively.
- Computationally independent keys derived from the ECDH shared secret and used for encryption of different messages.

EDHOC also makes the following additions:

- Negotiation of key derivation, encryption, and signature algorithms:
  - U proposes one or more algorithms of the following kinds:
    - HKDF
    - AEAD
    - Signature verification
    - Signature generation
  - V selects one algorithm of each kind
- Verification of common preferred ECDH curve:
  - U lists supported ECDH curves in order of preference
  - V verifies that the ECDH curve of the ephemeral key is the most preferred common curve
- Transport of opaque application defined data.

EDHOC is designed to encrypt and integrity protect as much information as possible, and all symmetric keys are derived using as much previous information as possible. EDHOC is furthermore designed to be as compact and lightweight as possible, in terms of message sizes, processing, and the ability to reuse already existing CBOR and COSE libraries. EDHOC does not put any requirement on the lower layers and can therefore be also be used e.g. in environments without IP.

This paper is organized as follows: Section 3 specifies general properties of EDHOC, including formatting of the ephemeral public keys and key derivation, Section 4 specifies EDHOC with asymmetric key authentication, Section 5 specifies EDHOC with symmetric key authentication, and Appendix A provides a wealth of test vectors to ease implementation and ensure interoperability.

## 3 EDHOC Overview

EDHOC consists of three messages (message_1, message_2, message_3) that maps directly to the three messages in SIGMA-I, plus an EDHOC error message. All EDHOC messages consists of a CBOR array where the first element is an int specifying the message type (MSG_TYPE). After creating EDHOC message_3, Party U can derive the traffic key (master secret) and protected application data can therefore be sent in parallel with EDHOC message_3. The application data may be protected using the negotiated AEAD algorithm and the explicit session identifiers S_U and S_V. EDHOC may be used with the media type application/edhoc defined in Section 7.

```
Party U                                                 Party V
   |                                                       |
   | ------------------ EDHOC message_1 -----------------> |
   |                                                       |
   | <----------------- EDHOC message_2 ------------------ |
   |                                                       |
   | ------------------ EDHOC message_3 -----------------> |
   |                                                       |
   | <----------- Protected Application Data ------------> |
   |                                                       |
```

Figure 2: EDHOC message flow

The EDHOC message exchange may be authenticated using pre-shared keys (PSK), raw public keys (RPK), or certificates (Cert). EDHOC assumes the existence of mechanisms (certification authority, manual distribution, etc.) for binding identities with authentication keys (public or pre-shared). EDHOC with symmetric key authentication is very similar to EDHOC with asymmetric key authentication, the difference being that information is only MACed, not signed.

EDHOC also allows opaque application data (APP_1, APP_2, APP_3) to be sent in the respective messages. APP_1 is unprotected, APP_2 is protected (encrypted and integrity protected), and APP_3 is protected and mutually authenticated. When EDHOC is used with asymmetric key authentication APP_2 is sent to an unauthenticated party, but with symmetric key authentication APP_2 is mutually authenticated.

### 3.1 Formatting of the Ephemeral Public Keys

The ECDH ephemeral public key SHALL be formatted as a COSE_Key of type EC2 or OKP according to section 13.1 and 13.2 of [RFC8152]. The curve X25519 is mandatory to implement. For Elliptic Curve Keys of type EC2, compact representation and compact output as per [RFC6090] SHALL be used, i.e. the ‘y’ parameter SHALL NOT be present in the The COSE_Key object. COSE [RFC8152] always use compact output for Elliptic Curve Keys of type EC2.

### 3.2 Key Derivation

Key and IV derivation SHALL be done as specified in Section 11.1 of [RFC8152] with the following input:

- The PRF SHALL be the HKDF [RFC5869] in the ECDH-SS w/ HKDF negotiated during the message exchange (HKDF_V).
- The secret SHALL be the ECDH shared secret as defined in Section 12.4.1 of [RFC8152].
- The salt SHALL be the PSK when EDHOC is authenticated with symmetric keys and the empty string “” when EDHOC is authenticated with asymmetric keys.
- The fields in the context information COSE_KDF_Context SHALL have the following values:
  - AlgorithmID is an int or tstr as defined below
  - PartyUInfo = PartyVInfo = ( nil, nil, nil )
  - keyDataLength is a uint as defined below
  - protected SHALL be a zero length bstr
  - other is a bstr SHALL be aad_2, aad_3, or exchange_hash

where exchange_hash, in non-CDDL notation, is:

| exchange_hash = H( H( message_1 | message_2 ) | message_3 ) |
|---|---|---|

Table 1

where H() is the hash function in HKDF_V.

For message_i the key, called K_i, SHALL be derived using other = aad_i, where i = 2 or 3. The key SHALL be derived using AlgorithmID set to the integer value of the negotiated AEAD (AEAD_V), and keyDataLength equal to the key length of AEAD_V.

If the AEAD algorithm requires an IV, then IV_i for message_i SHALL be derived using other = aad_i, where i = 2 or 3. The IV SHALL be derived using AlgorithmID = “IV-GENERATION” as specified in section 12.1.2. of [RFC8152], and keyDataLength equal to the IV length of AEAD_V.

Application specific traffic keys and other data SHALL be derived using other = exchange_hash. AlgorithmID SHALL be a tstr defined by the application and SHALL be different for different data being derived (an example is given in Appendix C.2). keyDataLength is set to the length of the data being derived.

## 4 EDHOC Authenticated with Asymmetric Keys

### 4.1 Overview

EDHOC supports authentication with raw public keys (RPK) and certificates (Cert) with the requirements that:

- Party U SHALL be able to identify Party V’s public key using ID_V.
- Party V SHALL be able to identify Party U’s public key using ID_U.

ID_U and ID_V SHALL either contain the credential used for authentication (e.g. x5bag or x5chain) or uniquely identify the credential used for authentication (e.g. x5t), see [I-D.schaad-cose-x509]. Party U and V MAY retrieve the other party’s credential out of band. Optionally, ID_U and ID_V are complemented with the additional parameters HINT_ID_U and HINT_ID_V containing information about how to retrieve the credential of Party U and Party V, respectively (e.g. x5u), see [I-D.schaad-cose-x509].

Party U and Party V MAY use different type of credentials, e.g. one uses RPK and the other uses Cert. Party U and Party V MAY use different signature algorithms.

EDHOC with asymmetric key authentication is illustrated in Figure 3.

```
Party U                                                          Party V
|                      S_U, N_U, E_U, ALG_1, APP_1                     |
+--------------------------------------------------------------------->|
|                               message_1                              |
|                                                                      |
|S_U, S_V, N_V, E_V, ALG_2, Enc(K_2; Sig(V; ID_V, aad_2, APP_2); aad_2)|
|<---------------------------------------------------------------------+
|                               message_2                              |
|                                                                      |
|           S_V, Enc(K_3; Sig(U; ID_U, aad_3, APP_3); aad_3)           |
+--------------------------------------------------------------------->|
|                               message_3                              |
```

Figure 3: EDHOC with asymmetric key authentication.

#### 4.1.1 Mandatory to Implement Algorithms

For EDHOC authenticated with asymmetric keys, the COSE algorithms ECDH-SS + HKDF-256, AES-CCM-64-64-128, and EdDSA are mandatory to implement.

### 4.2 EDHOC Message 1

#### 4.2.1 Formatting of Message 1

message_1 SHALL be a CBOR array as defined below

```
message_1 = [
  MSG_TYPE : int,
  S_U : bstr,
  N_U : bstr,
  E_U : serialized_COSE_Key,
  ECDH-Curves_U : alg_array,
  HKDFs_U : alg_array,
  AEADs_U : alg_array,
  SIGs_V : alg_array,
  SIGs_U : alg_array,
  ? APP_1 : bstr
]

serialized_COSE_Key = bstr .cbor COSE_Key

alg_array = [ + alg : int / tstr ]
```

where:

- MSG_TYPE = 1
- S_U - variable length session identifier
- N_U - 64-bit random nonce
- E_U - the ephemeral public key of Party U
- ECDH-Curves_U - EC curves for ECDH which Party U supports, in the order of decreasing preference
- HKDFs_U - supported ECDH-SS w/ HKDF algorithms
- AEADs_U - supported AEAD algorithms
- SIGs_V - signature algorithms, with which Party U supports verification
- SIGs_U - signature algorithms, with which Party U supports signing
- APP_1 - bstr containing opaque application data

#### 4.2.2 Party U Processing of Message 1

Party U SHALL compose message_1 as follows:

- Determine which ECDH curve to use with Party V. If U previously received from Party V an error message to message_1 with diagnostic payload identifying an ECDH curve in ECDH-Curves_U, then U SHALL retrieve an ephemeral from that curve. Otherwise the first curve in ECDH-Curves_U MUST be used. The content of ECDH-Curves_U SHALL be fixed, and SHALL NOT be changed based on previous error messages.
- Retrieve an ephemeral ECDH key pair generated as specified in Section 5 of [SP-800-56a] and format the ephemeral public key E_U as a COSE_key as specified in Section 3.1.
- Generate the pseudo-random nonce N_U.
- Choose a session identifier S_U which is not in use and store it for the length of the protocol. The session identifier SHOULD be different from other concurrent session identifiers used by Party U. The session identifier MAY be used with the protocol for which EDHOC establishes traffic keys/master secret, in which case S_U SHALL be different from the concurrently used session identifiers of that protocol.
- Format message_1 as specified in Section 4.2.1.

#### 4.2.3 Party V Processing of Message 1

Party V SHALL process message_1 as follows:

- Verify (OPTIONAL) that N_U has not been received before.
- Verify that at least one of each kind of the proposed algorithms are supported.
- Verify that the ECDH curve used in E_U is supported, and that no prior curve in ECDH-Curves_U is supported.
- For elliptic curves, that E_U is a valid point by verifying that there is a solution to the curve definition for the given parameter ‘x’.

If any verification step fails, Party V MUST send an EDHOC error message back, formatted as defined in Section 6.1, and the protocol MUST be discontinued. If V does not support the ECDH curve used in E_U, but supports another ECDH curves in ECDH-Curves_U, then the error message MUST include the following diagnostic payload describing the first supported ECDH curve in ECDH-Curves_U:

```
ERR_MSG = "Curve not supported; X"

where X is the first curve in ECDH-Curves_U that V supports,
encoded as in Table 22 of {{RFC8152}}.
```

- Pass APP_1 to the application.

### 4.3 EDHOC Message 2

#### 4.3.1 Formatting of Message 2

message_2 SHALL be a CBOR array as defined below

```
message_2 = [
  data_2,
  COSE_ENC_2 : COSE_Encrypt0
]

data_2 = (
  MSG_TYPE : int,
  S_U : bstr,
  S_V : bstr,
  N_V : bstr,
  E_V : serialized_COSE_Key,
  HKDF_V : int / tstr,
  AEAD_V : int / tstr,
  SIG_V : int / tstr,
  SIG_U : int / tstr
)

aad_2 : bstr
```

where aad_2, in non-CDDL notation, is:

```
aad_2 = H( message_1 | [ data_2 ] )
```

where:

- MSG_TYPE = 2
- S_V - variable length session identifier
- N_V - 64-bit random nonce
- E_V - the ephemeral public key of Party V
- HKDF_V - a single chosen algorithm from HKDFs_U
- AEAD_V - a single chosen algorithm from AEADs_U
- SIG_V - a single chosen algorithm from SIGs_V with which Party V signs
- SIG_U - a single chosen algorithm from SIGs_U with which Party U signs
- COSE_ENC_2 has the following fields and values:
  - external_aad = aad_2
  - plaintext = [ COSE_SIG_V, ? APP_2 ]
- COSE_SIG_V is a COSE_Sign1 object with the following fields and values:
  - protected = { abc : ID_V, ? xyz : HINT_ID_V }
  - detached payload = aad_2, ? APP_2
- abc - any COSE map label that can identify a public key, see Section 4.1
- ID_V - identifier for the public key of Party V
- xyz - any COSE map label for information about how to retrieve the credential of Party V, see Section 4.1
- HINT_ID_V - information about how to retrieve the credential of Party V
- APP_2 - bstr containing opaque application data
- H() - the hash function in HKDF_V

#### 4.3.2 Party V Processing of Message 2

Party V SHALL compose message_2 as follows:

- Retrieve an ephemeral ECDH key pair generated as specified in Section 5 of [SP-800-56a] using same curve as used in E_U. Format the ephemeral public key E_V as a COSE_key as specified in Section 3.1.
- Generate the pseudo-random nonce N_V.
- Choose a session identifier S_V which is not in use and store it for the length of the protocol. The session identifier SHOULD be different from other relevant concurrent session identifiers used by Party V. The session identifier MAY be used with the protocol for which EDHOC establishes traffic keys/master secret, in which case S_V SHALL be different from the concurrently used session identifiers of that protocol.
- Select HKDF_V, AEAD_V, SIG_V, and SIG_U from the algorithms proposed in HKDFs_U, AEADs_U, SIGs_V, and SIGs_U.
- Format message_2 as specified in Section 4.3.1:
  - COSE_Sign1 is computed as defined in section 4.4 of [RFC8152], using algorithm SIG_V and the private key of Party V.
  - COSE_Encrypt0 is computed as defined in section 5.3 of [RFC8152], with AEAD_V, K_2, and IV_2. The AEAD algorithm MUST NOT be replaced by plain encryption, see Section 8.

#### 4.3.3 Party U Processing of Message 2

Party U SHALL process message_2 as follows:

- Use the session identifier S_U to retrieve the protocol state.
- Verify that HKDF_V, AEAD_V, SIG_V, and SIG_U were proposed in HKDFs_U, AEADs_U, SIGs_V, and SIGs_U.
- Verify (OPTIONAL) that N_V has not been received before.
- For elliptic curves, validate that E_V is a valid point by verifying that there is a solution to the curve definition for the given parameter ‘x’.
- Verify message_2 as specified in Section 4.3.1:
  - COSE_Encrypt0 is decrypted defined in section 5.3 of [RFC8152], with AEAD_V, K_2, and IV_2.
  - COSE_Sign1 is verified as defined in section 4.4 of [RFC8152], using algorithm SIG_V and the public key of Party V.

If any verification step fails, Party U MUST send an EDHOC error message back, formatted as defined in Section 6.1, and the protocol MUST be discontinued.

- Pass APP_2 to the application.

### 4.4 EDHOC Message 3

#### 4.4.1 Formatting of Message 3

message_3 SHALL be a CBOR array as defined below

```
message_3 = [
  data_3,
  COSE_ENC_3 : COSE_Encrypt0
]

data_3 = (
  MSG_TYPE : int,
  S_V : bstr
)

aad_3 : bstr
```

where aad_3, in non-CDDL notation, is:

```
aad_3 = H( H( message_1 | message_2 ) | [ data_3 ] )
```

where:

- MSG_TYPE = 3
- COSE_ENC_3 has the following fields and values:
  - external_aad = aad_3
  - plaintext = [ COSE_SIG_U, ? APP_3 ]
- COSE_SIG_U is a COSE_Sign1 object with the following fields and values:
  - protected = { abc : ID_U, ? xyz : HINT_ID_U }
  - detached payload = aad_3, ? APP_3
- abc - any COSE map label that can identify a public key, see Section 4.1
- ID_U - identifier for the public key of Party U
- xyz - any COSE map label for information about how to retrieve the credential of Party U, see Section 4.1
- HINT_ID_V - information about how to retrieve the credential of Party U
- APP_3 - bstr containing opaque application data

#### 4.4.2 Party U Processing of Message 3

Party U SHALL compose message_3 as follows:

- Format message_3 as specified in Section 4.4.1:
  - COSE_Sign1 is computed as defined in section 4.4 of [RFC8152], using algorithm SIG_U and the private key of Party U.
  - COSE_Encrypt0 is computed as defined in section 5.3 of [RFC8152], with AEAD_V, K_3, and IV_3. The AEAD algorithm MUST NOT be replaced by plain encryption, see Section 8.

#### 4.4.3 Party V Processing of Message 3

Party V SHALL process message_3 as follows:

- Use the session identifier S_V to retrieve the protocol state.
- Verify message_3 as specified in Section 4.4.1:
  - COSE_Encrypt0 is decrypted as defined in section 5.3 of [RFC8152], with AEAD_V, K_3, and IV_3.
  - COSE_Sign1 is verified as defined in section 4.4 of [RFC8152], using algorithm SIG_U and the public key of Party U.

If any verification step fails, Party V MUST send an EDHOC error message back, formatted as defined in Section 6.1, and the protocol MUST be discontinued.

- Pass APP_3 to the application.

## 5 EDHOC Authenticated with Symmetric Keys

### 5.1 Overview

EDHOC supports authentication with pre-shared keys. Party U and V are assumed to have a pre-shared uniformly random key (PSK) with the requirement that:

- Party V SHALL be able to identify the PSK using KID.

KID may optionally contain information about how to retrieve the PSK.

EDHOC with symmetric key authentication is illustrated in Figure 4.

```
Party U                                                       Party V
|                S_U, N_U, E_U, ALG_1, KID, APP_1                   |
+------------------------------------------------------------------>|
|                             message_1                             |
|                                                                   |
|         S_U, S_V, N_V, E_V, ALG_2, Enc(K_2; APP_2; aad_2)         |
|<------------------------------------------------------------------+
|                             message_2                             |
|                                                                   |
|                    S_V, Enc(K_3; APP_3; aad_3)                    |
+------------------------------------------------------------------>|
|                             message_3                             |
```

Figure 4: EDHOC with symmetric key authentication.

#### 5.1.1 Mandatory to Implement Algorithms

For EDHOC authenticated with symmetric keys, the COSE algorithms ECDH-SS + HKDF-256 and AES-CCM-64-64-128 are mandatory to implement.

### 5.2 EDHOC Message 1

#### 5.2.1 Formatting of Message 1

message_1 SHALL be a CBOR array as defined below

```
message_1 = [
  data_1
]

data_1 = (
  MSG_TYPE : int,
  S_U : bstr,
  N_U : bstr,
  E_U : serialized_COSE_Key,
  ECDH-Curves_U : alg_array,
  HKDFs_U : alg_array,
  AEADs_U : alg_array,
  KID : bstr,
  ? APP_1 : bstr
)

serialized_COSE_Key = bstr .cbor COSE_Key

alg_array = [ + alg : int / tstr ]
```

where:

- MSG_TYPE = 4
- S_U - variable length session identifier
- N_U - 64-bit random nonce
- E_U - the ephemeral public key of Party U
- ECDH-Curves_U - EC curves for ECDH which Party U supports, in the order of decreasing preference
- HKDFs_U - supported ECDH-SS w/ HKDF algorithms
- AEADs_U - supported AEAD algorithms
- KID - identifier of the pre-shared key
- APP_1 - bstr containing opaque application data

#### 5.2.2 Party U Processing of Message 1

Party U SHALL compose message_1 as follows:

- Determine which ECDH curve to use with Party V. If U previously received from Party V an error message to message_1 with diagnostic payload identifying an ECDH curve in ECDH-Curves_U, then U SHALL retrieve an ephemeral from that curve. Otherwise the first curve in ECDH-Curves_U MUST be used.
- Retrieve an ephemeral ECDH key pair generated as specified in Section 5 of [SP-800-56a] and format the ephemeral public key E_U as a COSE_key as specified in Section 3.1.
- Generate the pseudo-random nonce N_U.
- Choose a session identifier S_U which is not in use and store it for the length of the protocol. The session identifier SHOULD be different from other relevant concurrent session identifiers used by Party U. The session identifier MAY be used with the protocol for which EDHOC establishes traffic keys/master secret, in which case S_U SHALL be different from the concurrently used session identifiers of that protocol.
- Format message_1 as specified in Section 5.2.1.

#### 5.2.3 Party V Processing of Message 1

Party V SHALL process message_1 as follows:

- Verify (OPTIONAL) that N_U has not been received before.
- Verify that at least one of each kind of the proposed algorithms are supported.
- Verify that the ECDH curve used in E_U is supported, and that no prior curve in ECDH-Curves_U is supported.
- For elliptic curves, validate that E_U is a valid point by verifying that there is a solution to the curve definition for the given parameter ‘x’.

If any verification step fails, Party V MUST send an EDHOC error message back, formatted as defined in Section 6.1, and the protocol MUST be discontinued. If V does not support the ECDH curve used in E_U, but supports another ECDH curves in ECDH-Curves_U, then the error message SHOULD include a diagnostic payload describing the first supported ECDH curve in ECDH-Curves_U.

- Pass APP_1 to the application.

### 5.3 EDHOC Message 2

#### 5.3.1 Formatting of Message 2

message_2 SHALL be a CBOR array as defined below

```
message_2 = [
  data_2,
  COSE_ENC_2 : COSE_Encrypt0
]

data_2 = (
  MSG_TYPE : int,
  S_U : bstr,
  S_V : bstr,
  N_V : bstr,
  E_V : serialized_COSE_Key,
  HKDF_V : int / tstr,
  AEAD_V : int / tstr
)

aad_2 : bstr
```

where aad_2, in non-CDDL notation, is:

```
aad_2 = H( message_1 | [ data_2 ] )
```

where:

- MSG_TYPE = 5
- S_V - variable length session identifier
- N_V - 64-bit random nonce
- E_V - the ephemeral public key of Party V
- HKDF_V - an single chosen algorithm from HKDFs_U
- AEAD_V - an single chosen algorithm from AEADs_U
- COSE_ENC_2 has the following fields and values:
  - external_aad = aad_2
  - plaintext = ? APP_2
- APP_2 - bstr containing opaque application data
- H() - the hash function in HKDF_V

#### 5.3.2 Party V Processing of Message 2

Party V SHALL compose message_2 as follows:

- Retrieve an ephemeral ECDH key pair generated as specified in Section 5 of [SP-800-56a] using same curve as used in E_U. Format the ephemeral public key E_V as a COSE_key as specified in Section 3.1.
- Generate the pseudo-random nonce N_V.
- Choose a session identifier S_V which is not in use and store it for the length of the protocol. The session identifier SHOULD be different from other relevant concurrent session identifiers used by Party V. The session identifier MAY be used with the protocol for which EDHOC establishes traffic keys/master secret, in which case S_V SHALL be different from the concurrently used session identifiers of that protocol.
- Select HKDF_V and AEAD_V from the algorithms proposed in HKDFs_U and AEADs_U.
- Format message_2 as specified in Section 5.3.1 where COSE_Encrypt0 is computed as defined in section 5.3 of [RFC8152], with AEAD_V, K_2, and IV_2.

#### 5.3.3 Party U Processing of Message 2

Party U SHALL process message_2 as follows:

- Use the session identifier S_U to retrieve the protocol state.
- For elliptic curves, validate that E_V is a valid point by verifying that there is a solution to the curve definition for the given parameter ‘x’.
- Verify message_2 as specified in Section 5.3.1 where COSE_Encrypt0 is decrypted defined in section 5.3 of [RFC8152], with AEAD_V, K_2, and IV_2.

If any verification step fails, Party U MUST send an EDHOC error message back, formatted as defined in Section 6.1, and the protocol MUST be discontinued.

- Pass APP_2 to the application.

### 5.4 EDHOC Message 3

#### 5.4.1 Formatting of Message 3

message_3 SHALL be a CBOR array as defined below

```
message_3 = [
  data_3,
  COSE_ENC_3 : COSE_Encrypt0
]

data_3 = (
  MSG_TYPE : int,
  S_V : bstr
)

aad_3 : bstr
```

where aad_3, in non-CDDL notation, is:

```
aad_3 = H( H( message_1 | message_2 ) | [ data_3 ] )
```

where:

- MSG_TYPE = 6
- COSE_ENC_3 has the following fields and values:
  - external_aad = aad_3
  - plaintext = ? APP_3
- APP_3 - bstr containing opaque application data

#### 5.4.2 Party U Processing of Message 3

Party U SHALL compose message_3 as follows:

- Format message_3 as specified in Section 5.4.1 where COSE_Encrypt0 is computed as defined in section 5.3 of [RFC8152], with AEAD_V, K_3, and IV_3.

#### 5.4.3 Party V Processing of Message 3

Party V SHALL process message_3 as follows:

- Use the session identifier S_V to retrieve the protocol state.
- Verify message_3 as specified in Section 5.4.1 where COSE_Encrypt0 is decrypted and verified as defined in section 5.3 of [RFC8152], with AEAD_V, K_3, and IV_3.

If any verification step fails, Party V MUST send an EDHOC error message back, formatted as defined in Section 6.1, and the protocol MUST be discontinued.

- Pass APP_3 to the application.

## 6 Error Handling

### 6.1 Error Message Format

This section defines a message format for an EDHOC error message, used during the protocol. This is an error on EDHOC level and is independent of the lower layers used. An advantage of using such a construction is to avoid issues created by usage of cross protocol proxies (e.g. UDP to TCP).

error SHALL be a CBOR array as defined below

```
error = [
  MSG_TYPE : int,
  ? ERR_MSG : tstr
]
```

where:

- MSG_TYPE = 0
- ERR_MSG is an optional text string containing the diagnostic payload, defined in the same way as in Section 5.5.2 of [RFC7252].

## 7 IANA Considerations

### 7.1 The Well-Known URI Registry

IANA has added the well-known URI ‘edhoc’ in the Well-Known URIs registry.

URI suffix: edhoc

Change controller: IETF

Specification document(s): [[this document]]

Related information: None

### 7.2 Media Types Registry

IANA has added the media type ‘application/edhoc’ to the Media Types registry:

```
    Type name: application

    Subtype name: edhoc

    Required parameters: N/A

    Optional parameters: N/A

    Encoding considerations: binary

    Security considerations: See Section 7 of this document.

    Interoperability considerations: N/A

    Published specification: [[this document]] (this document)

    Applications that use this media type: To be identified

    Fragment identifier considerations: N/A

    Additional information:

    * Magic number(s): N/A

    * File extension(s): N/A

    * Macintosh file type code(s): N/A

    Person & email address to contact for further information:
       Göran Selander <goran.selander@ericsson.com>

    Intended usage: COMMON

    Restrictions on usage: N/A

    Author: Göran Selander <goran.selander@ericsson.com>

    Change Controller: IESG
```

## 8 Security Considerations

EDHOC builds on the SIGMA-I family of theoretical protocols that provides perfect forward secrecy and identity protection with a minimal number of messages. The encryption algorithm of the SIGMA-I protocol provides identity protection, but the security of the protocol requires the MAC to cover the identity of the signer. Hence the message authenticating functionality of the authenticated encryption in EDHOC is critical: authenticated encryption MUST NOT be replaced by plain encryption only, even if authentication is provided at another level or through a different mechanism.

EDHOC adds an explicit message type and expands the message authentication coverage to additional elements such as algorithms, application data, and previous messages. EDHOC uses the same Sign-then-MAC approach as TLS 1.3.

EDHOC does not include negotiation of parameters related to the ephemeral key, but it enables Party V to verify that the ECDH curve used in the protocol is the most preferred curve by U which is supported by both U and V.

Party U and V must make sure that unprotected data and metadata do not reveal any sensitive information. This also applies for encrypted data sent to an unauthenticated party. In particular, it applies to APP_1 and APP_2 in the asymmetric case, and APP_1 and KID in the symmetric case. The communicating parties may therefore anonymize KID.

Using the same KID or unprotected application data in several EDHOC sessions allows passive eavesdroppers to correlate the different sessions. Another consideration is that the list of supported algorithms may be used to identify the application.

Party U and V are allowed to select the session identifiers S_U and S_V, respectively, for the other party to use in the ongoing EDHOC protocol as well as in a subsequent traffic protection protocol (e.g. OSCORE [I-D.ietf-core-object-security]). The choice of session identifier is not security critical but intended to simplify the retrieval of the right security context in combination with using short identifiers. If the wrong session identifier of the other party is used in a protocol message it will result in the receiving party not being able to retrieve a security context (which will terminate the protocol) or retrieving the wrong security context (which also terminates the protocol as the message cannot be verified).

Party U and V must make sure that unprotected data does not trigger any harmful actions. In particular, this applies to APP_1 in the asymmetric case, and APP_1 and KID in the symmetric case. Party V should be aware that replays of EDHOC message_1 cannot be detected unless previous nonces are stored.

The availability of a secure pseudorandom number generator and truly random seeds are essential for the security of EDHOC. If no true random number generator is available, a truly random seed must be provided from an external source. If ECDSA is supported, “deterministic ECDSA” as specified in RFC6979 is RECOMMENDED.

Nonces MUST NOT be reused, both parties MUST generate fresh random nonces.

Ephemeral keys SHOULD NOT be reused, both parties SHOULD generate fresh random ephemeral key pairs. Party V MAY reuse the ephemeral key to limit the effect of certain DoS attacks. For example, to reduce processing costs in the case of repeated uncompleted protocol runs, party V MAY pre-compute its ephemeral key E_V and reuse it for a small number of concurrent EDHOC executions, for example until a number of EDHOC protocol instances has been successfully completed, which triggers party V to pre-compute a new ephemeral key E_V to use with subsequent protocol runs.

The referenced processing instructions in [SP-800-56a] must be complied with, including deleting the intermediate computed values along with any ephemeral ECDH secrets after the key derivation is completed.

Party U and V are responsible for verifying the integrity of certificates. The selection of trusted CAs should be done very carefully and certificate revocation should be supported.

The choice of key length used in the different algorithms needs to be harmonized, so that a sufficient security level is maintained for certificates, EDHOC, and the protection of application data. Party U and V should enforce a minimum security level.

Note that, depending on the application, the keys established through the EDHOC protocol will need to be renewed, in which case the communicating parties need to run the protocol again.

Implementations should provide countermeasures to side-channel attacks such as timing attacks.

## 9 Acknowledgments

The authors want to thank Dan Harkins, Ilari Liusvaara, Jim Schaad and Ludwig Seitz for reviewing intermediate versions of the draft and contributing concrete proposals incorporated in this version. We are especially indebted to Jim Schaad for his continuous reviewing and implementation of different versions of the draft.

TODO: This section should be after Appendices and before Authors’ addresses according to RFC7322.

## 10 References

### 10.1 Normative References

[RFC2119] S. Bradner, "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997, <https://www.rfc-editor.org/info/rfc2119>.

[RFC6090] D. McGrew, K. Igoe, M. Salter, "Fundamental Elliptic Curve Cryptography Algorithms", RFC 6090, DOI 10.17487/RFC6090, February 2011, <https://www.rfc-editor.org/info/rfc6090>.

[RFC7049] C. Bormann, P. Hoffman, "Concise Binary Object Representation (CBOR)", RFC 7049, DOI 10.17487/RFC7049, October 2013, <https://www.rfc-editor.org/info/rfc7049>.

[RFC8152] J. Schaad, "CBOR Object Signing and Encryption (COSE)", RFC 8152, DOI 10.17487/RFC8152, July 2017, <https://www.rfc-editor.org/info/rfc8152>.

[I-D.schaad-cose-x509] Jim Schaad (August Cellars), "CBOR Object Signing and Encryption (COSE): Headers for carrying and referencing X.509 certificates", Internet-Draft draft-schaad-cose-x509-03, December 2018, <https://datatracker.ietf.org/doc/html/draft-schaad-cose-x509-03>.

[SP-800-56a] E. Barker, L. Chen, A. Roginsky, M. Smid, "Recommendation for Pair-Wise Key Establishment Schemes Using Discrete Logarithm Cryptography", NIST Special Publication 800-56A Revision 2, May 2013, <http://dx.doi.org/10.6028/NIST.SP.800-56Ar2>.

[SIGMA] H. Krawczyk, "SIGMA - The 'SIGn-and-MAc' Approach to Authenticated Diffie-Hellman and Its Use in the IKE-Protocols (Long version)", June 2003, <http://webee.technion.ac.il/~hugo/sigma-pdf.pdf>.

### 10.2 Informative References

[I-D.hartke-core-e2e-security-reqs] Göran Selander (Ericsson AB), Francesca Palombini (Ericsson AB), Klaus Hartke (Universitaet Bremen TZI), "Requirements for CoAP End-To-End Security", Internet-Draft draft-hartke-core-e2e-security-reqs-03, July 2017, <https://datatracker.ietf.org/doc/html/draft-hartke-core-e2e-security-reqs-03>.

[I-D.ietf-ace-oauth-authz] Ludwig Seitz (Combitech), Göran Selander (Ericsson), Erik Wahlstroem, Samuel Erdtman (Spotify AB), Hannes Tschofenig (Arm Ltd.), "Authentication and Authorization for Constrained Environments Using the OAuth 2.0 Framework (ACE-OAuth)", Internet-Draft draft-ietf-ace-oauth-authz-46, November 2021, <https://datatracker.ietf.org/doc/html/draft-ietf-ace-oauth-authz-46>.

[I-D.ietf-core-object-security] Göran Selander (Ericsson AB), John Preuß Mattsson (Ericsson AB), Francesca Palombini (Ericsson AB), Ludwig Seitz (RISE SICS), "Object Security for Constrained RESTful Environments (OSCORE)", Internet-Draft draft-ietf-core-object-security-16, March 2019, <https://datatracker.ietf.org/doc/html/draft-ietf-core-object-security-16>.

[I-D.ietf-ace-oscore-profile] Francesca Palombini (Ericsson AB), Ludwig Seitz (Combitech), Göran Selander (Ericsson AB), Martin Gunnarsson (RISE), "The Object Security for Constrained RESTful Environments (OSCORE) Profile of the Authentication and Authorization for Constrained Environments (ACE) Framework", Internet-Draft draft-ietf-ace-oscore-profile-19, May 2021, <https://datatracker.ietf.org/doc/html/draft-ietf-ace-oscore-profile-19>.

[I-D.ietf-cbor-cddl] Henk Birkholz (Fraunhofer SIT), Christoph Vigano (Universität Bremen), Carsten Bormann (Universitaet Bremen TZI), "Concise Data Definition Language (CDDL): A Notational Convention to Express Concise Binary Object Representation (CBOR) and JSON Data Structures", Internet-Draft draft-ietf-cbor-cddl-08, March 2019, <https://datatracker.ietf.org/doc/html/draft-ietf-cbor-cddl-08>.

[I-D.ietf-core-resource-directory] Christian Amsüss, Zach Shelby (ARM), Michael Koster (SmartThings), Carsten Bormann (Universitaet Bremen TZI), Peter Van der Stok (consultant), "Constrained RESTful Environments (CoRE) Resource Directory", Internet-Draft draft-ietf-core-resource-directory-28, March 2021, <https://datatracker.ietf.org/doc/html/draft-ietf-core-resource-directory-28>.

[RFC7228] C. Bormann, M. Ersue, A. Keranen, "Terminology for Constrained-Node Networks", RFC 7228, DOI 10.17487/RFC7228, May 2014, <https://www.rfc-editor.org/info/rfc7228>.

[RFC7252] Z. Shelby, K. Hartke, C. Bormann, "The Constrained Application Protocol (CoAP)", RFC 7252, DOI 10.17487/RFC7252, June 2014, <https://www.rfc-editor.org/info/rfc7252>.

[RFC5869] H. Krawczyk, P. Eronen, "HMAC-based Extract-and-Expand Key Derivation Function (HKDF)", RFC 5869, DOI 10.17487/RFC5869, May 2010, <https://www.rfc-editor.org/info/rfc5869>.

## A Appendix A. Test Vectors

TODO: This section needs to be updated.

## B Appendix B. PSK Chaining

An application using EDHOC with symmetric keys may have a security policy to change the PSK as a result of successfully completing the EDHOC protocol. In this case, the old PSK SHALL be replaced with a new PSK derived using other = exchange_hash, AlgorithmID = “EDHOC PSK Chaining” and keyDataLength equal to the key length of AEAD_V, see Section 3.2.

## C Appendix C. EDHOC with CoAP and OSCORE

### C.1 Transferring EDHOC in CoAP

EDHOC can be transferred as an exchange of CoAP [RFC7252] messages, with the CoAP client as party U and the CoAP server as party V. By default EDHOC is sent to the Uri-Path: “/.well-known/edhoc”, but an application may define its own path that can be discovered e.g. using resource directory [I-D.ietf-core-resource-directory].

In practice, EDHOC message_1 is sent in the payload of a POST request from the client to the server’s resource for EDHOC. EDHOC message_2 or the EDHOC error message is sent from the server to the client in the payload of a 2.04 Changed response. EDHOC message_3 or the EDHOC error message is sent from the client to the server’s resource in the payload of a POST request. If needed, an EDHOC error message is sent from the server to the client in the payload of a 2.04 Changed response

An example of successful EDHOC exchange using CoAP is shown in Figure 5.

```
Client    Server
  |          |
  +--------->| Header: POST (Code=0.02)
  |   POST   | Uri-Path: "/.well-known/edhoc"
  |          | Content-Type: application/edhoc
  |          | Payload: EDHOC message_1
  |          |
  |<---------+ Header: 2.04 Changed
  |   2.04   | Content-Type: application/edhoc
  |          | Payload: EDHOC message_2
  |          |
  +--------->| Header: POST (Code=0.02)
  |   POST   | Uri-Path: "/.well-known/edhoc"
  |          | Content-Type: application/edhoc
  |          | Payload: EDHOC message_3
  |          |
  |<---------+ Header: 2.04 Changed
  |   2.04   |
  |          |
```

Figure 5: Transferring EDHOC in CoAP

### C.2 Deriving an OSCORE context from EDHOC

When EDHOC is use to derive parameters for OSCORE [I-D.ietf-core-object-security], the parties must make sure that the EDHOC session identifiers are unique Recipient IDs in OSCORE. In case that the CoAP client is party U and the CoAP server is party V:

- The AEAD Algorithm is AEAD_V, as defined in this document
- The Key Derivation Function (KDF) is HKDF_V, as defined in this document
- The Client’s Sender ID is S_V, as defined in this document
- The Server’s Sender ID is S_U, as defined in this document
- The Master Secret is derived as specified in Section 3.2 of this document, with other = exchange_hash, AlgorithmID = “EDHOC OSCORE Master Secret” and keyDataLength equal to the key length of AEAD_V.
- The Master Salt is derived as specified in Section 3.2 of this document, with other = exchange_hash, AlgorithmID = “EDHOC OSCORE Master Salt” and keyDataLength equal to 64 bits.

## 11 Authors' Addresses

Göran Selander · Ericsson AB · Email: goran.selander@ericsson.com

John Mattsson · Ericsson AB · Email: john.mattsson@ericsson.com

Francesca Palombini · Ericsson AB · Email: francesca.palombini@ericsson.com
