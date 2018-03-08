# Setup an SFTP Server w/ Group Access on Ubuntu

This is a short guide to setup a chrooted SFTP user group and server on Ubuntu.
It has been tested on Ubuntu 16.04 but should work with 14 and 18.

## Create a New User and New SFTP Group

Let's start by logging in as root,

```
% sudo -s
```

Add a user,

```
% adduser <username>
```

You will be prompted for a password. This will allow access to the server via
password authentication.

Create an SFTP group,

```
% groupadd sftpusers
```

Now let's add our user to the group,

```
% usermod -a -G sftpusers <username>
```

## Create Directories and Lock Down File System

Create directories to where file sharing will occur. We want to prevent our SFTP
users from being able to access the rest of our file system so we will restrict
the user to a single folder.

I like to create a root /sftp folder and have that be the chrooted directory.

```
% mkdir /sftp
```

Most of the time we need an incoming and an outgoing folder for files.

```
% mkdir /sftp/shared
% mkdir /sftp/shared/incoming
% mkdir /sftp/shared/outgoing
```

If this SFTP will be multipurpose, you may want to add additional folders in the
incoming and outgoing to folders, fore example `products` and `orders`.

First we will restrict the writing to `sftp` to only the folder owner,

```
% chmod 755 /sftp
```

Let's make sure root owns the `sftp` folder,

```
% chown root:root /sftp
```

Now we need to give our SFTP users access to the shared folder and subfolders
through our group.

```
% chown root:sftpusers -R /sftp/shared
```

We still need to allow our SFTP users to write to the incoming folders,

```
% chown <username>:sftpusers -R /sftp/shared/incoming/
```

## SSH Config and Locking Down the SFTP Users

Lets edit `/etc/ssh/sshd_config`,

> Pay close attention to the file name and the `sshd_config`.

```
% vim /etc/ssh/sshd_config
```

Find the section with `Subsystem sftp /var/lib/openssh/sftp-server` and comment
it out, i.e. `#Subsystem sftp /var/lib/openssh/sftp-server`.

Add the line below the commented out line,

```
Subsystem sftp internal-sftp
```

Head to the bottom of the file and add the following lines,

```
Match Group sftpusers
ChrootDirectory /sftp
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
ForceCommand internal-sftp
PasswordAuthentication yes
```

Save and exit.

* *Match User*: Tells the SSH server to only apply the following settings to the one
user
* *ChrootDirectory*: This tells the server what directory our user is allowed to
ONLY work within this directory
* *X11Forwading*, *AllowTCPForwarding*, *AllowAgentForwarding*: Prohibits the user from
port forwarding, tunneling and X11 forwarding for the user. These are all
security things.
* *ForceCommand internal-sftp*: Forces the SSH server to the run the SFTP program
upon access which disables shell access.
* *PasswordAuthentication*: Allows for the user to login with a typed password. You
can remove this is you would rather use a security key which is by far safer.

Restart the SSH Server

```
% /etc/init.d/ssh restart
```

That's it, you should now be able to sftp to the server using the username and
password you created.

```
% sftp username@server.ip.address
/> password:

sftp >
```
