---
tags: [tool, recon, smb]
---
# smbclient

> Access Windows network shares over the SMB protocol.

## Usage
```bash
smbclient -L //<target>/ -N                     # list shares (anonymous)
smbclient //<target>/share -N                   # enter a share
smbclient //<target>/share -N -c 'put file.lnk' # upload a file (one-shot)
```

## Flags
| Flag          | Meaning                          |
| ------------- | -------------------------------- |
| `-L`          | List – list shares               |
| `-N`          | No password (anonymous)          |
| `-U user`     | authenticate as a user           |
| `-c '...'`    | run a command directly           |

## Inside a share
`ls`, `get <file>`, `put <file>`, `cd <folder>`

=> Technique: SMB Enumeration
