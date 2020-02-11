Keybase Unseal
==============

This Ansible role for initializing and unsealing Hashicorp Vault creates one
encrypted json file for your team Keybase team in KBFS. Default is a macOS path.

You can create a Keybase account and a team with sub-team on:
[https://keybase.io](https://keybase.io)

Requirements
------------

Create users on [https://keybase.io](https://keybase.io) and add define
`keybase_team:` in group_vars. Let the users install the Keybase.app on
multiple personal devices. Let them add proof for their online identities, so
they have [Skin in the game](https://en.wikipedia.org/wiki/Skin_in_the_Game_(book)).

Python libraries for the ansible control host are listed in requirements.txt.
The best experience is on a Mac due to the integration of Keybase, GPG Suite,
Apple Keychain, Ansible, etc. This was made on a Mac.

All trusted users should publish their public PGP key on Keybase if you want
[Shamir Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing).

Fun fact: **Shamir** is the S in RSA, he is a well-known cryptographer.

Secret-Zero & Shamir
====================
Keybase KBFS creates an encrypted shared filesystem for your team. This is used
to store 2 critical files: `vault.json` and `vault.pass`. The json file is the
output of `vault operator init ...` with PGP encrypted unseal keys.

```
vault_credentials: '/keybase/team/{{ keybase_team }}/vault.json'
```

The `vault.pass` file is stored there for remote work.

```
export KEYBASE_TEAM=dockpack.vault
export ANSIBLE_VAULT_PASSWORD_FILE="/keybase/team/$KEYBASE_TEAM/vault.pass"
```

You can store the `vault.pass` file on KBFS, it encrypts a temporary root token,
but each unseal key is encrypted with one member's PGP key,
(as long as **shamir: true** that is). Replacing the initial root token after
set-up is a best practice.

For decryption of `vault.json` with `ansible-vault` this environment
variable should be defined: `ANSIBLE_VAULT_PASSWORD_FILE`. Your team can agree
on a secret-zero out-of-band.

To work with the initial root token in your ENV:
```
export VAULT_TOKEN=$(ansible-vault view "/keybase/team/$KEYBASE_TEAM/vault.json"| jq -r .root_token)
```

Role Variables
--------------

`keybase_team:` the name of the Keybase team responsible for Vault operations.

If `shamir: true`, then Shamir Secret Sharing is enforced because Hashicorp
Vault will be initialized with encrypted unseal keys in the output. Each
trusted user can only decrypt their own key.

If `shamir: false`, then the Keybase team is the basis of shared trust, great
for automation, but less secure.

`key_threshold:` number of keys are needed to unseal the Vault, with
`shamir: true` this means that `key_threshold` number of members of the
`keybase_team` should run the playbook with `--tags unseal`.

`kbt:` is a set of trusted users in the Keybase team, avoid duplicates.
`export KBT_INDEX=<n>` to your index in this list.

`key_shares:` is the number of generated unseal keys, by default the length of `kbt:`.

Dependencies
------------
I use this role in a larger project, where I depend on these fine roles to
install Consul and Vault, among others.

```yaml
---
  - src: brianshumate.vault
  - src: brianshumate.consul
  - src: leonallen22.ansible_role_keybase
```

Example Playbook
----------------

A full example of a Vault Cluster with a Consul backend and a client and a
bastion is available:

[https://github.com/dockpack/vault_dojo](https://github.com/dockpack/vault_dojo)

License
-------

BSD

Author Information
------------------
© Copyright 2020 Bas Meijer
https://github.com/bbaassssiiee
https://github.com/dockpack
