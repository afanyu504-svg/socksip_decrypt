# socksip_decrypt

Decrypt **SocksIP / SocksTunnel** `.sip` config files into readable JSON.

`socksip_decrypt.py` is a single-file Python tool that reverses the multi-layer encryption used by the SocksIP Android VPN app for its exported `.sip` config files. It outputs every field of the embedded `SerSocksIP` object as plain JSON, including the decrypted password.

## Features

- Supports **VER2**, **VER5** and **VER7** file formats (auto-detected)
- Full 9-layer decryption
- Parses the Java-serialized config object
- Decrypts the embedded password field (OpenSSL `Salted__` blob)
- Typed JSON output (`bool` / `int` / `str`)

## Requirements

- Python 3.8+
- [pycryptodome](https://pypi.org/project/pycryptodome/)

```bash
pip install pycryptodome
```

## Usage

```bash
python socksip_decrypt.py config.sip output.json
python socksip_decrypt.py config.sip            # defaults to config.sip.json
```

## How it works

The `.sip` file is a base64-encoded onion. The only difference between versions is the key-tail length `T`:

| Magic | `T` |
|---|---|
| `VER2` | 32 |
| `VER5` | 52 |
| `VER7` | 47 |

```
base64
  -> AES/ECB/PKCS5
  -> strip "VERx" magic
  -> AES-256 key recovery (last T bytes, [16:T] reversed)
  -> CFB stream
  -> Salsa20
  -> CAST5 CFB
  -> PBKDF2-SHA1 XOR
  -> AES-256 CFB
  -> Java serialization (SerSocksIP)
  -> OpenSSL "Salted__" blob (password field)
```


## Notes

- The password field is only decrypted when it is an OpenSSL `Salted__` blob; plaintext values pass through unchanged.
