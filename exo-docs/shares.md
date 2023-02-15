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

First, we mount the share:

```
$ sudo mount /mydata
```

Now let's write something to a file on the share:

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

Finally, we unmont the share:

```
$ sudo umount /mydata
```


## Script to set up a share on an instance

```
" #!/bin/bash
# File: setup-share

# Draft 1 of a script for setting up shares on an instance
# In this draft, the four parameters local_share_name, rule_name
# path, and access_key are entered manually.

# We should automate input of parameters to the greatest extent possible

# Define the filename
# Eventually it will be /etc/fstab
fstab='fake-fstab'

# Define the ceph directory
# Eventually it will be /etc/ceph
ceph_dir='fake-ceph'

# Infor for user
echo
echo "  DIRECTIONS"
echo "  You will need four pieces of information to setup a share on your instance:"
echo
echo "  1.  The name of your access rule, defined in Jetstream2"
echo "  2.  The access key. This is part of your access rule."
echo "  3.  The path to your share."
echo "      Click on your share in Jetstream2 to get the details page."
echo "      It is there."
echo "  4.  The name you give to your share on this instance.  Up to you."
echo

# Get the needed inputs
read -p "Enter a local name for your share, e.g., 'myshare':" local_share_name
read -p "Enter the name of your access rule, e.g, 'joe-sharename-rule-1':" rule_name
read -p "Enter the path for your share:" path
read -p "Enter your access key:" access_key


# construct the new line for /etc/fstab that will enable mounting of the share
new_fstab_line="$path $local_share_name ceph name=$rule_name,x-systemd.device-timeout=30,x-systemd.mount-timeout=30,noatime,_netdev,rw 0   2"

ceph_data="[client.$rule_name]\n    key = $access_key"
keyring_file_name="ceph.client.$rule_name.keyring"

# append line to fstab
echo $new_fstab_line >> $fstab
echo Your data has been written to $fstab

# create keyring file
echo $ceph_data >> $ceph_dir/$keyring_file_name
echo Your key ring file has been created: $ceph_dir/$keyring_file_name"
```

## Listing Shares

### Request



### Response example

```
{
    "shares": [
        {
            "id": "d94a8548-2079-4be0-b21c-0a887acd31ca",
            "links": [
                {
                    "href": "http://172.18.198.54:8786/v1/16e1ab15c35a457e9c2b2aa189f544e1/shares/d94a8548-2079-4be0-b21c-0a887acd31ca",
                    "rel": "self"
                },
                {
                    "href": "http://172.18.198.54:8786/16e1ab15c35a457e9c2b2aa189f544e1/shares/d94a8548-2079-4be0-b21c-0a887acd31ca",
                    "rel": "bookmark"
                }
            ],
            "name": "My_share"
        },
        {
            "id": "406ea93b-32e9-4907-a117-148b3945749f",
            "links": [
                {
                    "href": "http://172.18.198.54:8786/v1/16e1ab15c35a457e9c2b2aa189f544e1/shares/406ea93b-32e9-4907-a117-148b3945749f",
                    "rel": "self"
                },
                {
                    "href": "http://172.18.198.54:8786/16e1ab15c35a457e9c2b2aa189f544e1/shares/406ea93b-32e9-4907-a117-148b3945749f",
                    "rel": "bookmark"
                }
            ],
            "name": "Share1"
        }
    ],
    "count": 10
}
```

### Modules changed


```
	modified:   src/LocalStorage/LocalStorage.elm
	modified:   src/OpenStack/Shares.elm (NEW)
	modified:   src/OpenStack/Types.elm
	modified:   src/Page/ProjectOverview.elm
	modified:   src/Page/ShareList.elm
	modified:   src/Route.elm
	modified:   src/State/State.elm
	modified:   src/State/ViewState.elm
	modified:   src/Types/Defaults.elm
	modified:   src/Types/HelperTypes.elm
	modified:   src/Types/OuterMsg.elm
	modified:   src/Types/Project.elm
	modified:   src/Types/View.elm
	modified:   src/View/Breadcrumb.elm
	modified:   src/View/PageTitle.elm
	modified:   src/View/View.elm
--------------------------------------------
31 modules
```