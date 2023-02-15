# Working with Shares

The notes below show how to set up a share
using a combination of [Exosphere](https://exosphere.app),
[Jetstream2](https://js2.jetstream-cloud.org/), and your 
local terminal.  Eventually this somewhat convoluted
process will be reduced to something quite simple on Exosphere.

## Terminology

- An *instance* is a virtual machine
- A *share* is a kind of virtual volume that typically
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
system, can be created in [Jetstream2](https://js2.jetstream-cloud.org/) (and soon
will be creatable on exosphere.)  To set up share,
navigate to the *Shares* page in Jetstream2.  Click on *+ Create Share*
and fill in the resulting form:

- Share Name
- Size
- Share protocol: `CephFS`
- Share type: your only choice is `cephfsnativetype`

The other fields are optional.  However, **do not** check the 
box *Make visible to users from all projects.**

Now you must add an access rule.  To do this, click on the 
triangular drop-down icon to the right of *Edit Share*
which you will find on the right-hand side of the row
in which your instance appears.  Select *Manage Rules,*
then click on the new button *Add Rule* which you will 
find on the right, above the row in which your share 
appears.  Fill in the resulting form as follows:

- Access Type: `cephx`
- Access Level: choose the default `read-write` unless you need `read-only`.
- Acces To: Enter what will be your rule name.  It should have the form
`USERNAME-SHARENAME-rule-N`, where `N = 1, 2, 3 ...`

Leave the other fields blank, then click the *Add* button. 
Now look at the row in which your rule appears.  In the
column *Access Key*, there will be a long string of letters
ending with `==`.  You will need this access key later,
as well as the rule name. 

## Accessing a Share

To be able to access a share on a running instance,
two files must be correctly configured:

1. Your keyring file in `/etc/ceph`
2. The `/etc/fstab` fild

To set them up, ssh to your instance


### Keyring file

It is important that your keyring file name obey
certain conventions, namely as in this model.

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


### fstab file

You must add a line of the following form to your `/etc/fstab` file:

```
PATH LOCAL_SHARE_NAME ceph name=RULE_NAME,x-systemd.device-timeout=30,x-systemd.mount-timeout=30,noatime,_netdev,rw 0   2
```

You already know about the `RULE_NAME`. For the `LOCAL_SHARE_NAME`, do 
something like `mkdir /mydata` in your instance. Then use `mydata` as the 
`LOCAL_SHARE_NAME`.  For the `PATH`, go to Jetstream2, click on the
name of your share in the mult-row display of shares so as to get
the share detail.  You will notice an item **Path**.  Copy long
string of numbers and letters you see there.

## Using the share

Let's write something to a file on the share:

```
$ sudo cat /mydata/foo.txt
This is a tests
^D
```

Then we have

```
$ ls /mydata
foo.txt

$ cat /mydata/foo.txt
This is a test
```



