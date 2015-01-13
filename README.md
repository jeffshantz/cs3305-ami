CS 3305 AMI
===========

Builds an Ubuntu AMI for CS 3305 (Operating Systems) at Western University (http://www.csd.uwo.ca) from an official Ubuntu cloud image.

Building the AMI
----------------

Spin up an Ubuntu EC2 instance and clone the repository:

```bash
sudo apt-get update
sudo apt-get install git-core language-pack-en
git clone https://github.com/jeffshantz/cs3305-ami.git
cd cs3305-ami
```

Log into the AWS console and create an IAM user.  Attach a user policy with full access to EC2 and obtain the Access Key ID and Secret Access Key for the user.
Then, copy the file `aws.config.example` to `aws.config` and configure it appropriately:

```bash
cp aws.config.example aws.config
vim aws.config
```

Verify the configuration in the variables at the top of the `build-ami` script.  Additionally, edit the packages to install in the image, as appropriate.

```bash
vim build-ami
```

As it takes a moment for an IAM user policy to take effect, you should ensure that you can run a command like the following before running the `build-ami` script:

```bash
AWS_DEFAULT_REGION=us-east-1 aws ec2 describe-instances
```

Finally, build the AMI.

```
./build-ami
```

After a few minutes, if all goes according to plan, you should see the details of your AMI displayed by the script:

```
AMI: ami-dc2052b4 trusty us-east-1 amd64

ami id:       ami-99999999
aki id:       aki-99999999
region:       us-east-1 (us-east-1a)
architecture: x86_64 (amd64)
os:           Ubuntu 14.04 trusty
name:         cs3305-server-ubuntu-14.04-trusty-amd64-20150113-0205
description:  CS3305 AMI - Western University - Ubuntu 14.04 Trusty amd64 20150113-0205
EBS volume:   vol-99999999 (deleted)
EBS snapshot: snap-99999999
git:
gitolite:      ( branch)

ami_id=ami-99999999
snapshot_id=snap-99999999

Test the new AMI using something like:

  instance_id=$(aws ec2 run-instances \
    --region us-east-1 \
    --key $USER \
    --instance-type t1.micro \
    --image-id ami-99999999
    --output text \
    --query 'Instances[*].InstanceId' )
  echo instance_id=$instance_id
```

You should see the snapshot listed in the EC2 console under **Snapshots** and your AMI should be found under **AMIs**.  When you go to launch an instance using the wizard,
you should now be able to select your AMI from the **My AMIs** tab.

Users
-----

Three users are present in the image:

* `ubuntu` - The default user you might expect in an Ubuntu cloud image.  This user can access an instance based on the AMI using the keypair selected in the **Launch Instance** wizard.
* `marker` - A user allowing our TAs access to students' images.  The public key installed for this user is configured at the top of the script.
* `rescue` - A user that assists us when students inevitably screw up the permissions on their instances (more later).  The public key installed for this user is configured at the top of the script.

The `rescue` User
-----------------

From my experience teaching with Amazon EC2, there are a few students each semester who invariably run a command like `chmod -R 600 /` as `root` and end up locking themselves out of their instance, since they mess up the permissions on their `.ssh` directory.  To prevent this, I've added the `rescue` user.

This user's home directory is `/rescue` and has been made immutable using `chattr`.  Even if the student runs the aforementioned `chmod` command, he/she won't be able to lock out the `rescue` user.

```bash
root@ip-10-65-161-24:/# rm -rf rescue/
rm: cannot remove ‘rescue/.bashrc’: Permission denied
rm: cannot remove ‘rescue/.vim’: Permission denied
rm: cannot remove ‘rescue/.profile’: Permission denied
rm: cannot remove ‘rescue/.ssh/authorized_keys’: Permission denied
rm: cannot remove ‘rescue/.bash_logout’: Permission denied
rm: cannot remove ‘rescue/.vimrc’: Permission denied
root@ip-10-65-161-24:/# chmod -R 0400 /rescue/
chmod: changing permissions of ‘/rescue/’: Operation not permitted
chmod: changing permissions of ‘/rescue/.ssh’: Operation not permitted
chmod: changing permissions of ‘/rescue/.ssh/authorized_keys’: Operation not permitted
```

Why not just make the `marker` user's directory immutable?  This was done for user-friendliness.  Since the `rescue` user's home directory is immutable, it cannot write to it, which might be a pain when a TA is marking.

Author
------

This script was written by Jeff Shantz and was heavily adapted from alestic-git-build-ami: https://github.com/alestic/alestic-git/blob/master/bin/alestic-git-build-ami.

