---
layout: post
title: Remote Cloud Hosted Password Managers/Vaults
---
We've come a long way in usability of these tools since my first exposure using
[Password Safe](https://pwsafe.org/). Services such as Bitwarden, LastPass,
1Password, and so on have made it extremely easy to get on board with using
a single master password to decrypt a database of passwords (that are ideally
random, long and unique), clients working across multiple devices and OS, vault
storage in the cloud. They also generally come with 2FA options, sounds super
secure.

I've been using these types of services for a while (LastPass originally then
migrating to Bitwarden a few years ago, Bitwarden is great), they really do
make life easier and solve the password re-use problem. But I got curious
recently as to how these services are supposedly secure, after researching how
my current provider (Bitwarden) works I thought it useful to document here in
simple language what's going on.

I'll assume these providers all use a similar scheme, they tend to use flashy
marketing saying it's all awesome, listing accreditations and then direct you
to white-papers for the details. I read through the paper for Bitwarden and I
can see why it's probably not shouted far and wide because it appears not
particularly secure at a glance, but it is pretty good, though it does have
pitfalls that should be made clearer.

Encryption keys
---------------
### Master Password (`$MASTER_PW`)
This is your (changeable) password to login to the cloud service and decrypt
the vault to retrieve your passwords. This theoretically never leaves your
local machine.

### The Vault (`$VAULT_KEY`)
When your account is created a symmetric key is generated locally, generally in
your browser. This key will then be used to encrypt and decrypt all the data
stored in your vault.

Storage
-------
### `$VAULT_KEY` -> `$ENC_VAULTKEY`
The symmetric vault key is encrypted and stored using symmetric encryption
keyed with `$MASTER_PW`.

### Vault
The vault is stored in the "cloud" encrypted with `$VAULT_KEY`.

Access
------
When you log in to a service you send a hash of `$LOGIN+$MASTER_PW` to the
remote service, if you have enabled 2FA the service may require this to
authenticate you (note: this 2FA has **nothing** to do with your vault), if you
have successfully authenticated you will receive back a copy of the encrypted
vault and `$ENC_VAULTKEY`.

The local client will then decrypt `$ENC_VAULTKEY` to further decrypt the vault
itself.

Master Password Change
----------------------
When you change your master password `$MASTER_PW` and `$ENC_VAULTKEY` will
change, however `$VAULT_KEY` will not.

The One Ring
------------
In general there is one original key, the `$VAULT_KEY` that if compromised is
not good. If the `$VAULT_KEY` is compromised the vault can be decrypted, even
after a `$MASTER_PW` change. So in summary the service stores your vault and
`$ENC_VAULTKEY`, theoretically locking access to both of those behind potential
2FA and `$MASTER_PW`.

It still is a pretty good system but pitfalls should be obvious after reading
this, be careful where you unlock the vault.
