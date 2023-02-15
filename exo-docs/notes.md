## Terminology

- An *instance* is a virtual machine
- A *share* is a kind of virtual volume that typicall
consists of many physical volumes on various physical machines.

## Accessing an Instance Locally

- In Exosphere go the Instances card
- Find your instance and click on it. 
- In the new page, note the following info on the 
*Credentials* card:
  - Floating IP Address
  - Username
  - Passphrase
- Now do `ssh USERNAME@IPADDRESS` and paste in the passphrase



## Setting up a Share

A share, which we can think of as a virtual file
system, can be created in Jetstream2 (and soon
will be creatable on exosphere.) To be able
to access a share on a running instance,
two files must be correctly configured:

1. Your keyring file in `/etc/ceph`
2. The `/etc/fstab` fild

To set them up, ssh to your instance


### Keyring file

It is important that your keyring file name obey
certain conventions, e.g., 

```
/etc/ceph/ceph.client.RULE_NAME.keyring
```

```
[client.RULE_NAME]
  key = ACCESSKEY
```

```
[client.joeuser-testinstance-rule-1]
  key = AQ...bw==
```