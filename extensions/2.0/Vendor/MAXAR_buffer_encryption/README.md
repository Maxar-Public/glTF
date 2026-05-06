<!-- omit in toc -->
# MAXAR_buffer_encryption
<!-- omit in toc -->
## Contributors

* Björn Blissing, [@bjornblissing](https://github.com/bjornblissing)
* Johan Borg, [@jo-borg](https://github.com/jo-borg)

<!-- omit in toc -->
## Status

**Version 1.0.0**, June 25, 2025

<!-- omit in toc -->
## Dependencies

Written against the glTF 2.0 spec.


## Contents

- [Overview](#overview)
- [Optional vs Required](#optional-vs-required)
- [Schema](#schema)
- [Usage](#usage)
- [Base64 encoding](#base64-encoding)

- [Examples](#examples)
- [Implementation Notes](#implementation-notes)

## Overview

The `MAXAR_buffer_encryption` extension describes encryption at the buffer level.

The presence of the extension at the buffer level indicates that specific ranges within the buffer are encrypted. The encryption key information can be provided through an external keyfile or by other application-specific means.

## Optional vs Required

This extension is required; it must be listed in the glTF JSON `extensionsUsed` and `extensionsRequired` arrays.

## Schema

The extension uses the following schema files:

* [buffer.MAXAR_buffer_encryption.schema.json](schema/buffer.MAXAR_buffer_encryption.schema.json) - Schema for the buffer extension
* [keyfile.schema.json](schema/keyfile.schema.json) - Schema for the external keyfile
* [encryptedRange.schema.json](schema/encryptedRange.schema.json) - Schema for encrypted range objects
* [encryptionKey.schema.json](schema/encryptionKey.schema.json) - Schema for encryption key objects
* [keyItem.schema.json](schema/keyItem.schema.json) - Schema for key item objects
* [userKey.schema.json](schema/userKey.schema.json) - Schema for user key objects

## Usage

To use this extension:

1. Add the extension to both `extensionsUsed` and `extensionsRequired` arrays in the glTF JSON
2. Add the extension object to each buffer that contains encrypted data
3. Reference the appropriate encryption key ID that corresponds to the key in the external keyfile

The buffer extension contains:
- A unique encryption key identifier that applies to the entire buffer
- An optional keyfile URI to specify the key source for this specific buffer (data URIs are not allowed for security reasons)
- The encryption algorithm used for this buffer's data (applies to all ranges)
- An array of encrypted ranges, each specifying which parts of the buffer are encrypted
- For each range: byte offset, byte length, initialization vector (IV), and authentication tag

**Important:** Only one encryption key is allowed per buffer. If multiple different keys are desired, multiple buffers must be employed. This design ensures clear key management and simplifies implementation.

**Note:** Key identifiers must be between 16 and 256 characters and uniquely identify the key within the system. The key ID should be generated to guarantee uniqueness (for example, using a UUID, a random string, a systematically generated id with namespace context, or a cryptographic hash of the key material). It must never contain the key itself or any reversible encoding of the key.

Relative URIs are resolved according to standard web URI resolution behavior as defined in RFC 3986. This means:
- For .gltf files: Relative URIs are resolved relative to the location of the .gltf file itself
- For .glb files: Since they are binary containers, relative URIs are resolved relative to the location of the .glb file

The keyfile contains an array of encryption keys, each with the necessary information to decrypt the content, including:
- A human-readable name for the user key
- A unique user key identifier
- The encryption algorithm used
- A unique encryption key identifier
- The encrypted secret key
- An initialization vector (IV) for encryption methods that require it
- An authentication tag for authenticated encryption modes

## Base64 encoding

All binary fields represented as strings in this extension use standard Base64 encoding (RFC 4648 §4) with the '+' and '/' alphabet. Padding is applied to Base64 encoded strings only when the final block contains fewer than three input bytes: with two bytes append "=", with one byte append "=="; with three bytes no padding is added. All Base64 encoded fields expect padding when needed.

### Keyfile Field Descriptions

#### userKey
Information about the user key used to encrypt the secret key.

##### userKey.name
A human-readable name for the user key to help identify its purpose or usage context.

##### userKey.id
A unique identifier for the user key used to encrypt the secret key. The user key itself is not stored in the dataset and must be managed separately by the viewing application. The key ID should be generated to guarantee uniqueness (for example, using a UUID, a random string, a systematically generated id with namespace context, or a cryptographic hash of the key material). It must never contain the key itself or any reversible encoding of the key.

#### encryptionKey
Information about the secret encryption key used to encrypt buffer data.

##### encryptionKey.id
A unique identifier for the encryption key. This provides an identifier for the secret key itself, separate from the user key ID. This ID is referenced by the `keyId` field in buffer ranges. The key ID should be generated to guarantee uniqueness (for example, using a UUID, a random string, a systematically generated id with namespace context, or a cryptographic hash of the key material). It must never contain the key itself or any reversible encoding of the key.

##### encryptionKey.data
The secret key encrypted by the user key. When decrypted, this key is used to decrypt buffer data.

##### encryptionKey.algorithm
The algorithm used to encrypt the secret key. This can be different from the algorithm used for buffer data encryption. Supported algorithms are:
- AES-256-GCM: AES with 256-bit key in Galois/Counter Mode
- AES-128-GCM: AES with 128-bit key in Galois/Counter Mode
- RSA-OAEP-SHA256-2048: RSA with OAEP padding and SHA-256 hash using 2048-bit key
- RSA-OAEP-SHA256-3072: RSA with OAEP padding and SHA-256 hash using 3072-bit key
- RSA-OAEP-SHA256-4096: RSA with OAEP padding and SHA-256 hash using 4096-bit key

##### encryptionKey.iv (Initialization Vector)
The initialization vector used for encryption of the secret key. This field is:
- **Required for**: AES-GCM modes (AES-256-GCM, AES-128-GCM)
- **Not required for**: RSA-OAEP modes (RSA-OAEP-SHA256-2048, RSA-OAEP-SHA256-3072, RSA-OAEP-SHA256-4096)
- **Format**: Base64-encoded binary data
- **Recommended length**: 12 bytes for AES-GCM modes

##### encryptionKey.authTag (Authentication Tag)
The authentication tag for verification of the encrypted secret key. Provides integrity verification and authenticity assurance for authenticated encryption modes. This field is:
- **Required for**: AES-GCM modes (AES-256-GCM, AES-128-GCM)
- **Not required for**: RSA-OAEP modes (RSA-OAEP-SHA256-2048, RSA-OAEP-SHA256-3072, RSA-OAEP-SHA256-4096)
- **Format**: Base64-encoded binary data
- **Recommended length**: 16 bytes for AES-GCM modes



### Buffer Extension Field Descriptions

#### keyId
A unique identifier for the encryption key used for this buffer. This must match a keyId defined in the external keyfile. The key ID should be generated to guarantee uniqueness (for example, using a UUID, a random string, a systematically generated id with namespace context, or a cryptographic hash of the key material). It must never contain the key itself or any reversible encoding of the key.

**Important:** Only one encryption key is allowed per buffer. All encrypted ranges within the buffer must use the same key. If multiple different keys are desired, multiple buffers must be employed.

#### keyfile (Optional)
An optional URI (or IRI) of the external key file for this specific buffer. If not specified, the application is responsible for providing the key information through other means. This allows individual buffers to reference specific keyfiles when needed.

#### algorithm
The encryption algorithm used for this buffer's data. Applies to all ranges. Supported algorithms are:
- AES-256-GCM: AES with 256-bit key in Galois/Counter Mode
- AES-128-GCM: AES with 128-bit key in Galois/Counter Mode


#### ranges
An array of encrypted ranges within the buffer. Each range specifies a portion of the buffer that is encrypted, allowing for partial encryption of buffers where some data remains unencrypted. Ranges must not overlap within a buffer.

Each range object contains:

##### byteOffset
The offset into the buffer in bytes where this encrypted range starts. Must be a non-negative integer.

##### byteLength
The length in bytes of this encrypted range. Must be a positive integer.

The sum of `byteOffset` and `byteLength` must not exceed the buffer's `byteLength` (i.e., `byteOffset + byteLength <= buffer.byteLength`).

##### iv (Initialization Vector)
The initialization vector used for encryption of this specific range. This field is:
- **Required for**: all buffer range encryption (AES-256-GCM, AES-128-GCM)
- **Format**: Base64-encoded binary data
- **Recommended length**: 12 bytes for AES-GCM modes

##### authTag (Authentication Tag)
The authentication tag for this specific encrypted range. Provides integrity verification and authenticity assurance. This field is:
- **Required for**: all buffer range encryption (AES-256-GCM, AES-128-GCM)
- **Format**: Base64-encoded binary data
- **Recommended length**: 16 bytes for AES-GCM modes


## Examples

### glTF JSON

```json
{
  "asset": {
    "version": "2.0"
  },
  "extensionsUsed": [
    "MAXAR_buffer_encryption"
  ],
  "extensionsRequired": [
    "MAXAR_buffer_encryption"
  ],
  "bufferViews": [
    {
      "buffer": 0,
      "byteOffset": 0,
      "byteLength": 102400
    },
    {
      "buffer": 0,
      "byteOffset": 102400,
      "byteLength": 51200
    }
  ],
  "buffers": [
    {
      "byteLength": 153600,
      "uri": "data.bin",
      "extensions": {
        "MAXAR_buffer_encryption": {
          "keyId": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
          "algorithm": "AES-256-GCM",
          "ranges": [{
            "byteOffset": 0,
            "byteLength": 153600,
            "iv": "cmFuZG9tSW5pdGlhbGl6YXRpb25WZWN0b3I=",
            "authTag": "YXV0aGVudGljYXRpb25UYWdFeGFtcGxl"
          }]
        }
      }
    },
    {
      "byteLength": 76800,
      "uri": "unencrypted_data.bin"
      // No extension - this buffer is not encrypted
    }
  ]
}
```

### Partial Encryption Example

This example shows a buffer where only part of the data is encrypted:

```json
{
  "asset": {
    "version": "2.0"
  },
  "extensionsUsed": [
    "MAXAR_buffer_encryption"
  ],
  "extensionsRequired": [
    "MAXAR_buffer_encryption"
  ],
  "bufferViews": [
    {
      "buffer": 0,
      "byteOffset": 0,
      "byteLength": 1920
      // This buffer view accesses encrypted data
    },
    {
      "buffer": 0,
      "byteOffset": 1920,
      "byteLength": 7690
      // This buffer view accesses unencrypted data
    }
  ],
  "buffers": [
    {
      "byteLength": 9610,
      "uri": "mixed_data.bin",
      "extensions": {
        "MAXAR_buffer_encryption": {
          "keyId": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
          "algorithm": "AES-256-GCM",
          "ranges": [{
            "byteOffset": 0,
            "byteLength": 1920,
            "iv": "e3E8JtN1eVqD",
            "authTag": "v3u1jXJ4qPLGvKz8N1Rw4C=="
          }]
        }
      }
    }
  ]
}
```

### Buffer-Specific Keyfile Example

This example shows a buffer that uses the optional keyfile field to link to an external keyfile. Since the keyfile field is optional, this approach enables sharing key references between multiple glTF files, in contrast to buffers without a keyfile that rely on the application to provide key information through other means:

```json
{
  "asset": {
    "version": "2.0"
  },
  "extensionsUsed": [
    "MAXAR_buffer_encryption"
  ],
  "extensionsRequired": [
    "MAXAR_buffer_encryption"
  ],
  "buffers": [
    {
      "byteLength": 9610,
      "uri": "special_data.bin",
      "extensions": {
        "MAXAR_buffer_encryption": {
          "keyId": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
          "keyfile": "encryption/special_key.json",
          "algorithm": "AES-256-GCM",
          "ranges": [{
            "byteOffset": 0,
            "byteLength": 9610,
            "iv": "e3E8JtN1eVqD",
            "authTag": "v3u1jXJ4qPLGvKz8N1Rw4C=="
          }]
        }
      }
    }
  ]
}
```

### Keyfile (special_key.json)

```json
{
  "keys": [
    {
      "userKey": {
        "name": "Project XYZ Encryption Key",
        "id": "b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6"
      },
      "encryptionKey": {
        "id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
        "data": "dGhpcyBpcyBhbiBleGFtcGxlIGVuY3J5cHRlZCBrZXk=",
        "algorithm": "AES-256-GCM",
        "iv": "cmFuZG9tSW5pdGlhbGl6YXRpb25WZWN0b3I=",
        "authTag": "YXV0aGVudGljYXRpb25UYWdFeGFtcGxl"
      }
    }
  ]
}
```

*Note: The `data` field content in this example is abbreviated for clarity. In a real implementation, the encrypted data for a 256-bit key would be significantly longer.*

### RSA Encryption Example

This example shows using RSA-OAEP-SHA256 for key encryption instead of AES-GCM:

```json
{
  "keys": [
    {
      "userKey": {
        "name": "RSA Public Key for Project ABC",
        "id": "b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6"
      },
      "encryptionKey": {
        "id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
        "data": "dGhpcyBpcyBhbiBleGFtcGxlIFJTQSBlbmNyeXB0ZWQgc2VjcmV0IGtleQ==",
        "algorithm": "RSA-OAEP-SHA256-2048"
      }
    }
  ]
}
```

*Note: When using RSA algorithms, the `iv` and `authTag` fields are not required since RSA-OAEP-SHA256 does not use initialization vectors or authentication tags. The `data` field content in this example is abbreviated for clarity.*

### Multiple Encryption Keys Example

This example shows how to use different encryption keys for different data by employing multiple buffers. Since only one encryption key is allowed per buffer, different buffers must be used when different keys are required:

```json
{
  "asset": {
    "version": "2.0"
  },
  "extensionsUsed": [
    "MAXAR_buffer_encryption"
  ],
  "extensionsRequired": [
    "MAXAR_buffer_encryption"
  ],
  "bufferViews": [
    {
      "buffer": 0,
      "byteOffset": 0,
      "byteLength": 76800
      // This buffer view accesses high-security encrypted data
    },
    {
      "buffer": 1,
      "byteOffset": 0,
      "byteLength": 76800
      // This buffer view accesses standard-security encrypted data
    }
  ],
  "buffers": [
    {
      "byteLength": 76800,
      "uri": "high_security_data.bin",
      "extensions": {
        "MAXAR_buffer_encryption": {
          "keyId": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
          "keyfile": "encryption/multi_key.json",
          "algorithm": "AES-256-GCM",
          "ranges": [
            {
              "byteOffset": 0,
              "byteLength": 76800,
              "iv": "aGlnaFNlY3VyaXR5SXY=",
              "authTag": "aGlnaFNlY3VyaXR5VGFn"
            }
          ]
        }
      }
    },
    {
      "byteLength": 76800,
      "uri": "standard_security_data.bin",
      "extensions": {
        "MAXAR_buffer_encryption": {
          "keyId": "f1e2d3c4b5a6978869504132a3b4c5d6",
          "keyfile": "encryption/multi_key.json",
          "algorithm": "AES-128-GCM",
          "ranges": [
            {
              "byteOffset": 0,
              "byteLength": 76800,
              "iv": "bG93U2VjdXJpdHlJdg==",
              "authTag": "bG93U2VjdXJpdHlUYWc="
            }
          ]
        }
      }
    }
  ]
}
```

### Multi-Key Keyfile (multi_key.json)

```json
{
  "keys": [
    {
      "userKey": {
        "name": "High Security Key",
        "id": "b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6"
      },
      "encryptionKey": {
        "id": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
        "data": "aGlnaFNlY3VyaXR5RW5jcnlwdGVkS2V5",
        "algorithm": "AES-256-GCM",
        "iv": "aGlnaFNlY3VyaXR5SXY=",
        "authTag": "aGlnaFNlY3VyaXR5VGFn"
      }
    },
    {
      "userKey": {
        "name": "Standard Security Key",
        "id": "c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7"
      },
      "encryptionKey": {
        "id": "f1e2d3c4b5a6978869504132a3b4c5d6",
        "data": "c3RhbmRhcmRTZWN1cml0eUVuY3J5cHRlZEtleQ==",
        "algorithm": "AES-128-GCM",
        "iv": "bG93U2VjdXJpdHlJdg==",
        "authTag": "bG93U2VjdXJpdHlUYWc="
      }
    }
  ]
}
```

*Note: The `data` field contents in this example are abbreviated for clarity. In a real implementation, the encrypted data would be longer, with the 256-bit key data being significantly longer than the 128-bit key data.*



## Key Management and Security Model

### User Key vs Secret Key

The `MAXAR_buffer_encryption` extension uses a two-tier key management system:

#### Secret Key (Data Encryption Key)
- **Purpose**: The actual key used to encrypt/decrypt the buffer content data
- **Storage**: Encrypted and stored in the `encryptionKey` field of the keyfile
- **Location**: Part of the dataset, distributed with the glTF content
- **Security**: Protected by encryption using the user key

#### User Key (Key Encryption Key)
- **Purpose**: Used to encrypt/decrypt the secret key stored in `encryptionKey`
- **Algorithm**: Can use either symmetric encryption (AES-256-GCM, AES-128-GCM) or asymmetric encryption (RSA-OAEP-SHA256 with 2048, 3072, or 4096-bit keys)
  - **Symmetric (AES)**: The same user key is used for both encryption and decryption of the secret key
  - **Asymmetric (RSA)**: Uses a public/private key pair. Can work in either direction:
    - **User-to-Producer**: User sends their public key to the data producer, who encrypts the secret key with it. User then decrypts using their private key
    - **Producer-to-User**: Data producer encrypts the secret key with their private key. User decrypts using the producer's public key
- **Storage**: **NOT stored anywhere in the dataset or keyfile**
- **Location**: Managed entirely by the viewing application
- **Security**: Responsibility of the application developer and end user

### Security Responsibilities

#### Dataset Provider Responsibilities:
1. Generate secure secret keys for data encryption
2. Encrypt buffer content using the secret keys
3. Encrypt the secret keys using user-provided keys
4. Store encrypted secret keys in keyfiles (when using the optional keyfile approach) or provide them through other application-specific means
5. Distribute the dataset with encrypted content and optional keyfiles

#### Viewing Application Responsibilities:
1. **Secure user key management**: Obtain, store, and protect user keys
2. **Key derivation**: Derive or obtain user keys through secure channels
3. **Secret key decryption**: Use user keys to decrypt `encryptionKey` values
4. **Memory protection**: Securely handle decrypted keys in memory
5. **Key lifecycle**: Manage user key rotation and expiration

#### End User Responsibilities:
1. Protect user keys from unauthorized access
2. Follow organizational security policies for key management
3. Ensure secure transmission of user keys to viewing applications

### Algorithm Selection

The extension supports different encryption algorithms for different purposes:

#### Buffer Data Encryption (AES-GCM only)
Buffer ranges are always encrypted using AES-GCM algorithms:
- **AES-256-GCM**: AES with 256-bit key in Galois/Counter Mode
- **AES-128-GCM**: AES with 128-bit key in Galois/Counter Mode
- **Requirements**: Always requires `iv` (initialization vector) and `authTag` (authentication tag) fields
- **Performance**: Optimized for high-performance bulk data encryption

#### Secret Key Encryption (AES-GCM or RSA-OAEP-SHA256)
Secret keys in the keyfile can be encrypted using either symmetric or asymmetric algorithms:

##### AES-GCM Algorithms (for secret key encryption)
- **Use case**: High-performance scenarios where the user key can be securely shared
- **Key sizes**: 128-bit and 256-bit options available
- **Requirements**: Requires `iv` (initialization vector) and `authTag` (authentication tag) fields
- **Key management**: Requires secure distribution of symmetric user keys

##### RSA-OAEP-SHA256 Algorithms (for secret key encryption)
- **Use case**: Scenarios requiring asymmetric cryptography for enhanced key distribution security
- **Key sizes**: 2048-bit, 3072-bit, and 4096-bit options available
- **Requirements**: Does not require `iv` or `authTag` fields
- **Key management**: Public keys can be distributed openly; only private keys need secure handling

### Security Benefits

This two-tier approach provides several security advantages:

1. **Dataset Distribution**: The dataset can be distributed through insecure channels since it does not contain the user keys needed for decryption
2. **Key Separation**: User keys are managed separately from the data, reducing exposure risk
3. **Access Control**: Only authorized users with valid user keys can decrypt the content
4. **Scalability**: Different user keys can be used for different users or organizations accessing the same dataset. Each user or organization would need their own keyfile where the secret keys are encrypted using their specific user keys.
5. **Granular Access Control**: Different encryption keys can be used for different buffers, allowing fine-grained access control where users may have access to some data but not others. Since only one key is allowed per buffer, organize data into separate buffers based on access control requirements.
6. **Algorithm Flexibility**: Choice between symmetric (AES) and asymmetric (RSA) encryption for user keys allows optimization for different security and performance requirements:
   - **Symmetric (AES)**: Faster performance, suitable when the same party encrypts and decrypts the data
   - **Asymmetric (RSA)**: Enhanced security for key distribution, supports bidirectional workflows:
     - Users can provide their public keys to producers for encryption, maintaining exclusive decryption access via their private keys
     - Producers can encrypt with their private keys, allowing users to decrypt with the producer's public key for basic authentication purposes

## Frequently Asked Questions

### How does `MAXAR_buffer_encryption` interact with compression extensions?

**Q: Can I use `MAXAR_buffer_encryption` with `KHR_draco_mesh_compression`?**
A: Yes, but the order of operations matters. Draco compression should be applied first, then the entire buffer containing the compressed data is encrypted. During decoding, decrypt the entire buffer first, then decompress individual Draco-compressed sections.

**Q: What about `KHR_mesh_quantization`?**
A: `KHR_mesh_quantization` works at the accessor level and does not affect buffer data directly. The order of operations matters: apply quantization first, then encrypt the relevant buffer ranges. During decoding, decrypt the encrypted ranges first, then the quantization parameters in the accessor definitions can be applied to the decrypted data.

**Q: Can I encrypt buffers containing `KHR_texture_basisu` compressed textures?**
A: Yes, but the order of operations matters. Basis Universal compression should be applied first, then encrypt the buffer ranges containing the compressed data. During decoding, decrypt the encrypted ranges first, then Basis Universal decoders can process the texture data.

**Q: How does this work with `EXT_meshopt_compression`?**
A: `MAXAR_buffer_encryption` is compatible with `EXT_meshopt_compression`. The order of operations matters: apply meshopt compression first, then encrypt the buffer ranges containing the compressed data. During decoding, decrypt the encrypted ranges first, then `EXT_meshopt_compression` can access the data and apply decompression.

### General Compression and Encryption Guidelines

1. **Order of operations**: Always compress first, then encrypt. This provides better compression ratios since encrypted data appears random and does not compress well.

2. **Performance considerations**: Encrypting entire buffers can impact loading performance. Consider encrypting only sensitive data to optimize performance while maintaining security.

3. **Selective encryption**: Consider using separate buffers for sensitive vs. non-sensitive data, encrypting only the buffers containing sensitive information. Alternatively, use ranges to encrypt only sensitive parts of a buffer while leaving other data unencrypted.

4. **Extension compatibility**: `MAXAR_buffer_encryption` is designed to work alongside other glTF extensions. The encryption is applied at the buffer level, so it does not interfere with higher-level extensions that reference the same data. However, decryption must happen before other extensions can process the encrypted data.

## Implementation Notes

### Authentication Tag Verification

**Critical Security Requirement:** A buffer view may never read data from a buffer where decryption has failed, i.e., failed the check against the `authTag` (authentication tag).

The authentication tag provides cryptographic verification that:
1. The encrypted data has not been tampered with
2. The correct decryption key was used
3. The data integrity is intact

Implementation requirements:
- **Before** allowing any buffer view to access decrypted data, the implementation **must** verify the authentication tag
- If authentication tag verification fails for any encrypted range, the entire buffer **must** be treated as inaccessible
- Buffer views that reference data from a buffer with failed authentication **must not** be processed
- The application should report a clear error indicating authentication failure rather than attempting to use potentially corrupted or tampered data

