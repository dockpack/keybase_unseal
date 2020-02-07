Keybase Unseal
==============

This Ansible role for initializing and unsealing Hashicorp Vault creates one
encrypted json file for your team Keybase team in KBFS. Default is a macOS path.

```
vault_credentials: '/keybase/team/{{ keybase_team }}/vault.json'
```

You can share the passphrase on KBFS, or on your jumphost, but for encryption
this environment variable should be defined `ANSIBLE_VAULT_PASSWORD_FILE`.

```
export ANSIBLE_VAULT_PASSWORD_FILE="/Volumes/Keybase/team/dockpack/vault.pass"
```

To work with the initial root token in your ENV:
```
export VAULT_TOKEN=$(ansible-vault view "/Volumes/Keybase/team/dockpack/vault.json"| jq -r .root_token)
```

Replacing the initial root token after set-up is a best practice.


Requirements
------------

Create users on https://keybase.io and add define `keybase_team:`. Let the
users install the Keybase.app on multiple personal devices.

Python libraries for the ansible control host are listed in requirements.txt.

All trusted users should store their public PGP key on keybase if you want
**Shamir Secret Sharing**.



Role Variables
--------------

`keybase_team:` the name of the Keybase team responsible for Vault operations.

If `auto_unseal: true`, then the Keybase team is the basis of shared trust, and
that is great for automation, so it is the default.

If `auto_unseal: false`, then Shamir Secret Sharing is enforced because Hashicorp
Vault will be initialized with encrypted unseal keys in the output. Each trusted
user can only decrypt their own key.

`key_threshold:` number of keys are needed to unseal the Vault, with `auto_unseal: false`
this is a manual process done in the UI or using the CLI.

`kbt:` is a set of trusted users in the Keybase team. It is a set to avoid duplicates.

`key_shares:` is the number of generated unseal keys, by default the length of `kbt:`.

Dependencies
------------


```yaml
---
  - src: brianshumate.vault
  - src: brianshumate.consul
  - src: leonallen22.ansible_role_keybase
```

Example Playbook
----------------

A full example of a Vault Clusted with a Consul backend is available:

[https://github.com/dockpack/vault_dojo](https://github.com/dockpack/vault_dojo)

License
-------

BSD

Author Information
------------------

https://github.com/bbaassssiiee
https://github.com/dockpack
