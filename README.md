Flux SOPS sameple
=================

This repository explain how to manage secret information with GitOps. Use PGP as key management program.

## Tools

- [Flux](https://github.com/fluxcd/flux): Continuous Deployment tool
- [SOPS](https://github.com/mozilla/sops): Encryption and Decryption tool
    - [GnuPG](https://gnupg.org/): GnuPG is a complete and free implementation of the OpenPGP standard

## Preparation

### Creating a GPG signing key

You can reference generating a key and importing a key to Flux container from [git-gpg](https://github.com/fluxcd/flux/blob/master/docs/references/git-gpg.md). You only have to specify `--git-gpg-key-import` option as flux arguments because only importing gpg key is enough.


You also are able to generate gpg key without prompt.

```txt:gpg-opts.txt
%echo Generating a basic OpenPGP key
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: Takeshi Takizawa
Name-Comment: Some comments
Name-Email: xxx@example.com
Expire-Date: 0
# Do a commit here, so that we can later print "done" :-)
%no-protection
%commit
%echo done
```

```sh
gpg --batch --full-generate-key gpg-opts.txt
```

More options see [Unattended key generation](https://www.gnupg.org/documentation/manuals/gnupg/Unattended-GPG-key-generation.html).

## Encryption

https://github.com/mozilla/sops#usage

## Decryption by Flux


### Creating `.flux.yaml` file

Before kubectl apply you need to decrypt file. So let's create `.flux.yaml` file to manage manifest generation steps.

```yaml:.flux.yaml
version: 1
patchUpdated:
  generators:
  - command: sops --decrypt --output secret.yaml secret.enc.yaml
  - command: rm html.secret.yaml
  patchFile: flux-patch.yaml
```

`sops` command is pre-installed in Flux container so all you have to do is run `sops --decrypt` command. For safety removing decrypted file is better. If you use `kustomize` to generate manifest, adding `kustomize build .` command after sops command please.

More detail see [Manifest generation through .flux.yaml configuration files](https://docs.fluxcd.io/en/1.18.0/references/fluxyaml-config-files.html).
