# Encrypt/decrypt files using Docker and OpenSSL

> Just for fun, play with Docker and OpenSSL

![Banner](./banner.svg)

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Encrypt a file](#encrypt-a-file)
- [Decrypt a file](#decrypt-a-file)
  - [Decrypt on the console, don't write a file](#decrypt-on-the-console-dont-write-a-file)
- [A DOS-compatible version](#a-dos-compatible-version)
- [License](#license)


## Introduction

Using a [Docker Alpine/OpenSSL](https://hub.docker.com/r/alpine/openssl) image, it's so easy to encrypt/decrypt files using OpenSSL.

This repository comes with two Linux scripts (can be easily translated to Windows): `encrypt.sh` and `decrypt.sh`.

The idea behind this repository is to provide a way to encrypt files for, for instance, before sending them on the internet. Let's imagine you're writing markdown content with sensitive information. You wish to use all the features of Github.com but don't want your secrets are stored there in plain text.

So, before committing the files on Github (`git push`), we'll encrypt them and still keep them as text files; encrypted ones.

On the other side, we will retrieve files (`git pull`) and we can easily decrypt them on our computer; using our tremendous password only known by us.

This repository show a way to achieve this, very easily.

## Prerequisites

The only thing to have on your machine is Docker: [https://www.docker.com/products/personal/](https://www.docker.com/products/personal/). It's free for personal use.

## Encrypt a file

Here is the content of the `encrypt.sh` file. For the example, the script will encrypt the file `secrets.md`, supplied in this repo.

```bash
#!/usr/bin/env bash

(
  MY_PASSWORD="ThisIsMyLongPasswordNobodyWillBeAbleToCrackIt" &&
  docker run --rm -it -v $(pwd):/data -w /data -u $(id -u):$(id -g) alpine/openssl enc -aes-256-cbc -salt -pbkdf2 -a -in /data/secrets.md -out /data/secrets_encrypted.md -k ${MY_PASSWORD}
)
```

For the sample, `MY_PASSWORD` is defined at a variable on the operating system. You can use any technique you want, of course, to provide the password.

In a real world, we'll not store the password in the script but use techniques like:

* Using a `.env` file in our repository, that file will be ignored in `.gitignore`
* Use an OS variable defined on our machine, not in the script, 
* Use a `cat my_password.txt` instruction for providing the password as entry (stdin) to our command, 
* ... (be creative)

The options used are:

| Option         | Description                                             |
| -------------- | ------------------------------------------------------- |
| `enc`          | Encoding with Ciphers                                   |
| `-aes-256-cbc` | The encryption cipher to be used                        |
| `-salt`        | Adds strength to the encryption                         |
| `-pbkdf2`      | Generate a PBKDF2 key derivation of a supplied password |
| `-a`           | Encrypted data should be in Base64 and not binary       |
| `-in`          | Specifies the input file                                |
| `-out`         | Specifies the output file                               |
| `-k`           | Provide the password to use for the encryption          |

By running that command (or by running `encrypt.sh`), you'll encrypt the file called `secrets.md` and get a newer one called `secrets_encrypted.md`.

Here is the content:

```text
U2FsdGVkX18jyyHAiaDcwolgvrCmB9SutNFhOFosDZvYA+t/8F5PWsxU+YIb0xLj
/0swl1Mvh9XBcg3FwpQn5CGm5ltb3zKiExPO8WoTuYOmlJj2PN5eLJv3GWVVJ8/t
q31xBBAlbI0k+a3pWiETl1qEmh4hwc4jeC5NOByYSAojiIdCNF0W5+VVkUlBeKGb
sv8tpDWEb/dgHrfFPtZD5MqeNQw71/ndORZC1ZDIT/Ju6O7a6rd9ph0aQuPz49PU
SzDUePUgn9wbR0tZvNM1JA1LkN1kDaguJ940TdKns+Q=
```

## Decrypt a file

Here is the content of the `decrypt.sh` file.

```bash
#!/usr/bin/env bash

(
  MY_PASSWORD="ThisIsMyLongPasswordNobodyWillBeAbleToCrackIt" &&
  docker run --rm -it -v $(pwd):/data -w /data -u $(id -u):$(id -g) alpine/openssl enc -aes-256-cbc -salt -pbkdf2 -a -d -in /data/secrets_encrypted.md -out /data/secrets_decrypted.md -k ${MY_PASSWORD}
)
```

The options used are:

| Option         | Description                                             |
| -------------- | ------------------------------------------------------- |
| `enc`          | Encoding with Ciphers                                   |
| `-aes-256-cbc` | The encryption cipher to be used                        |
| `-salt`        | Adds strength to the encryption                         |
| `-pbkdf2`      | Generate a PBKDF2 key derivation of a supplied password |
| `-a`           | Encrypted data should be in Base64 and not binary       |
| `-d`           | Decrypt action                                          |
| `-in`          | Specifies the input file                                |
| `-out`         | Specifies the output file                               |
| `-k`           | Provide the password to use for the decryption          |

By running that command (or by running `decrypt.sh`), you'll decrypt the file `secrets_encrypted.md` and get a newer one called `secrets_decrypted.md`.

### Decrypt on the console, don't write a file

Edit the `decrypt.sh` file and search for `-out /data/secrets_decrypted.md`. Remove that part.

Now, when you'll run `decrypt.sh` the decrypted content will be displayed on the console only, nothing will be written on the disk. Your secrets are safe.

## A DOS-compatible version

The encryption script, `encrypt.cmd`, will ask you for a password (since the `-k` parameter is not part of the instruction). If the encrypted file has been created, the original one will be removed from your disk.

```text
@echo off

cls

docker run --rm -it -v %CD%:/data -w /data alpine/openssl enc -aes-256-cbc -salt -pbkdf2 -a -in /data/secrets.md -out /data/secrets.encrypted

# Put this part in comment if you want to keep the original, unencrypted, file.
IF EXIST secrets.encrypted (
  del secrets.md
)
```

The decryption script, `decrypt.cmd` will ask you for the password and will display the decrypted content on the console (since the `-out` parameter is not part of the instruction).

```text
@echo off

cls

docker run --rm -it -v C:\temp:/data -w /data alpine/openssl enc -aes-256-cbc -salt -pbkdf2 -a -d -in /data/secrets.encrypted
```

## License

[MIT](LICENSE)

