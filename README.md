Vault Experiment
================
Note: This is not secure by any means. This is meant to show a simple configuration of Hashicorp vault and is only slightly different than dev mode at this time of writing, differing only in that it stores secrets to disk. This is meant for experimentation and nothing more. 

This instance of Vault can be accessed from the host machine using the name 'vault1.vagrant.local'.

## Prereqs
You must have the `vagrant-hostmanager` plugin installed:
```
vagrant plugin install vagrant-hostmanager
```

## Starting the instance
```
vagrant up
```

## Accessing Vault from the host
Linux
```
export VAULT_ADDR=http://vault1.vagrant.local:8200
```

Windows (Powershell)
```
$env:VAULT_ADDR='http://vault1.vagrant.local:8200'
```

## Initialize vault
This will need to be done prior to following the Getting Started documentation linked in the next section, as the documentation assumes you are running in dev mode which auto-inits and unseals your vault for you.
```
vault init # take note of the unseal keys
vault unseal KEY # note that you will need to run `vault unseal` with as many keys as the threshold dictates. By default this is 3 keys/
```

## Vault Docs
If you have never used Vault before, you will probably want to read through the [Getting Started](https://www.vaultproject.io/intro/getting-started/first-secret.html) documentation

## Changing Vault server configuration
### Server configs
Edit .json or .hcl files in `files/config`. A basic .hcl configuration file already exists there by default.

### Vault server environment variables
Edit `files/vaultenv` using the pattern `VARIABLE=VALUE`.

### Updating the configuration after making changes
Re-provision the vagrant machine to update configurations after you change them in the source:
```
vagrant provision
```
You probably will need to restart the vault service yourself after this. As this is mostly for experimentation and proof-of-concept, not a lot of effort when into the reprovisioning code.


