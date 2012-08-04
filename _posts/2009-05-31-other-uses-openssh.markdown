---
layout: post
title: Other uses for OpenSSH
summary: |
    Many people use [OpenSSH][] to connect to a remote machine, but many don't
    know that you can actually do other interesting things with it, other than
    just opening a remote login shell. In this article I'll show you a few of
    them.
    
    We will see how to:
    
    * Copy files from one machine to another securely.
    
    * Forward ports from your machine through the remote machine, creating
      a secure tunnel for your unsecured applications.
    
    * Increase the security of your OpenSSH communications.
    
    [OpenSSH]: http://www.openssh.com/
---

Many people use [OpenSSH][] to connect to a remote machine, but many don't
know that you can actually do other interesting things with it, other than
just opening a remote login shell. In this article I'll show you a few of
them.

We will see how to:

* Copy files from one machine to another securely.

* Forward ports from your machine through the remote machine, creating
  a secure tunnel for your unsecured applications.

* Increase the security of your OpenSSH communications.

[OpenSSH]: http://www.openssh.com/


## Make sure the OpenSSH daemon is running.

OpenSSH is the most widely used implementation of SSH, and you'll probably
find it already installed in most UNIX like systems.

To be able to connect to the remote machine you'll need to have the `sshd`
daemon running in it. If this remote machine is actually physically remote
then it must have it already running, otherwise how could you do anything
at all?.

But if your remote machine is sitting right next you, or you are running
it as a virtual machine and you are experimenting with OpenSSH, then you
can find out if the daemon is running by executing the following command:

    $ ps -ef | grep sshd

You should see a line similar to this (the numbers will vary):

    root      2859     1  0 13:29 ?        00:00:00 /usr/sbin/sshd

If it is not already running consult your distribution documentation to
find out how to to start it. On an [Slackware][] system you can do that by
executing the following command as root:

    # /etc/rc.d/rc.sshd start

To have the `sshd` daemon run every time your Slackware system boots up,
you'll have to `chmod` the startup script so that it has execute
permission.

    # chmod 755 /etc/rc.d/rc.sshd

On a [Debian][] system you can start the daemon by executing the following
command:

    # /etc/init.d/ssh start

To find out how to make it start every time your Debian system boots up,
read the `update-rc.d` man page to figure out how.

    $ man update-rc.d

[Slackware]: http://slackware.com/
[Debian]: http://debian.org/


## 1. Copy files from one machine to another securely.

### 1.1. Using `scp` for simple transfers.

You can easily copy files from one machine to another through an encrypted
connection with OpenSSH, and this will prevent anyone else from spying
what you are sending over. The basic form is:

    mymachine:~$ scp somefile.tar gajon@remote.com:

That will copy the file `somefile.tar` to the remote machine `remote.com`
at the home folder of the user `gajon` (I'll be using my own user name in
all the examples here, but of course you should use your own instead).

It is important to put a colon (`:`) after the remote machine name,
otherwise `scp` will just rename your local file as `gajon@remote.com` in
the same local folder in your machine.

It's also possible to specify the path and final name that you want the
file to have in the remote server. For example, this will copy the file to
the folder `backups` that is located in the user's home.

    mymachine:~$ scp somefile.tar gajon@remote.com:backups/

This will copy the file to the same folder as the previous example, but
will save the file with a different name.

    mymachine:~$ scp somefile.tar gajon@remote.com:backups/back02.tar

You can also copy files from the remote machine to your local machine, you
just have to reverse the order of the arguments. For example, this will
copy the file `myapp.log` that is located in the `/var/log` folder of the
remote machine to your local machine, but with the name `remote.log`
instead.

    mymachine:~$ scp gajon@remote.com:/var/log/myapp.log remote.log

If you wanted to preserve the original name you could have used the
following command:

    mymachine:~$ scp gajon@remote.com:/var/log/myapp.log .

Notice that the remote path after the colon starts with a slash, that
indicates an absolute path.  If you don't start the path with a slash then
it's a relative path, relative to the home of the user in the remote
machine.


### 1.2 Using `sftp` for secure FTP transfers.

Many people know how to use FTP, but some may not realize that everything
they transfer during an FTP session, including their user names and
passwords, are sent unencrypted or protected in any form. This opens up
the possibility that someone with ill intentions might capture that
information.

`sftp` solves that problem and it is used the same way as `ftp`, so that
the user doesn't need to learn anything new, and most graphical FTP
clients can also use `sftp` -- for example, [FileZilla][].

In this example we get the file `back02.tar` from the remote machine and
save it in our local computer.

    mymachine:~$ sftp gajon@remote.com
    Connecting to remote.com...
    gajon@remote.com's password:
    sftp> cd backups
    sftp> get back02.tar
    sftp> bye

When you log on to a remote machine with the `sftp` client the prompt will
change to `sftp>`. The commands are almost the same as with a traditional
`ftp`, you can enter `?` to see a list of available commands.

[FileZilla]: http://filezilla-project.org/


## 2. Forwarding ports.

### 2.1. From your local machine to a remote machine.

Port forwarding is useful for creating a secure data tunnel for an
application that doesn't support encryption or other form of secure
communication out of the box. This is also useful to go through a firewall
and skip any of its restrictions.

As an example, lets suppose you want to access a POP3 server. You can
redirect one of your local ports, lets say `9999`, to the port `110`
(POP3) of the remote machine.

    mymachine:~$ ssh -L 9999:localhost:110 gajon@remote.com

Next you need to configure your email client to use port `9999`. OpenSSH
will take all traffic on port `9999` in your local machine and send it
encrypted to the remote machine, where the data will continue through port
`110` to its original destination.

Notice that the remote machine will send the data to the final destination
using port `110`, **AND** this data will **NOT** be encrypted anymore from
that point on. You must be aware of that, if your remote machine and the
destination machine (POP3 server) are the same then it's OK, otherwise
there's still a risk.


### 2.2. From the remote machine to your local machine.

It's also possible to do a forwarding in the other direction, from the
remote machine to your local machine. An example of when this could be
useful is when you have a machine in your office that sits behind a
firewall that blocks all inbound connections, but permits all outbound
connections, and you want to be able to connect to your office machine
from your home and open an `ssh` session.

From the office machine you can establish the tunnel to the home machine:

    office:~$ ssh -R 2222:localhost:22 gajon@home

This tunnel will listen on port `2222` of the home machine and will
encrypt data and send it to port `22` of the machine at the office. Once
you are at home you can connect to the office machine from home with
OpenSSH, using this tunnel that you established from the office machine. 

    home:~$ ssh -p 2222 gajon@office

Notice that we are indicating to `ssh` that we want to connect with the
port `2222` (`-p` option), this is because that's the port that your local
machine is listening to.


### 2.3. Dynamic port forwarding.

Dynamic port forwarding lets you forward any port on your local machine
through a secure tunnel to a remote machine, by creating a SOCKS server.

Let's say that you have a restrictive firewall at the office that blocks
web pages that have not been authorized. Imagine you need to find a
solution to a programming problem you are having but can't do that because
the firewall is blocking all websites that contain anything similar to a
forum, and we all know that forums are where we always find solutions to a
technical, undocumented problem.

Well, that has happened to me, and the solution is a SOCKS server that can
proxy our data through a remote machine; and have it encrypted as an added
bonus!.

You can establish a dynamic port forwarding tunnel as:

    mymachine:~$ ssh -D 12345 gajon@remote.com

With that command, you establish a connection to the remote machine, but
OpenSSH will listen to any SOCKS communications on your local machine at
port `12345`.

To be able to use this tunnel, your application that needs to connect must
know how to use the SOCKS protocol (either SOCKS4 or SOCKS5). On Firefox
it's easy to do that, we go to *Preferences* and in the *Advanced* group
select the *Network* tab, click on the *Settings* button and select
*Manual proxy settings* and in the *SOCKS Server* field put `localhost`,
and *port* `12345`.

And that's it, when Firefox needs to load a website, it will communicate
with the SOCKS server in your local machine with port `12345`; OpenSSH
will encrypt your data and send it through a secure channel to the remote
machine, where the data will continue its way to its destination and
original port.

But there are many applications that don't implement a SOCKS protocol, one
example is Opera. There are tools that can wrap a non SOCKS application
and transparently send all its connections through a SOCKS server. One
such application that I have used is `tsocks`.


## 3. Increase the security of your OpenSSH communications.

There are several things you can do to increase the security of your
OpenSSH communications, and it's a good idea to do them.


### 3.1. Disable root logins.

The most useful and easiest way to increase the security of your OpenSSH
server is to disable `root` logins. To do this you must edit the
`sshd_config` file in your server.

    remote:~# vim /etc/ssh/sshd_config

Search for the `PermitRootLogin` attribute and change it to `no`. Then
restart your OpenSSH server. On Slackware you use the following line:

    remote:~# /etc/rc.d/rc.ssh restart

On a Debian system you use the following line:

    remote:~# /etc/init.d/sshd restart


### 3.2. Change the port where OpenSSH listens.

There are many people with bad intentions that use automated tools that
try to breach OpenSSH servers. If you have a server on the Internet, you
may have noticed in your logs that there are lots and lots of attempts to
login to your server using many different user names. What they are trying
to do is to get to one server by applying a so called *dictionary attack*,
that is, attempting a huge combination of common user names and passwords,
with the hope that one of those combinations will actually succeed.

A very easy way to stop that is to simply change the port of your OpenSSH
daemon. If the attacker cannot find a server at the default port `22` he
will probably just move to the next server that he can find. These kind of
attackers don't bother checking every port of the machine, that would take
too much time, instead what they do is try all the ip addresses they can. 

To change the port number of your server, edit the `sshd_config` file
again and change the `Port` attribute. For example:

    Port 30234

Restart the OpenSSH daemon, and now to connect you have to specify the
port to use. You can use the `-p` switch:

    mymachine:~$ ssh -p 30234 gajon@remote.com

But having to type the `-p` switch and the port is tedious, so instead of
that you can specify the port in a `config` file that your `ssh` client
can use. This `config` file is usually found in the `$HOME/.ssh/` folder.

Open up the `$HOME/.ssh/config` file in your editor and put these lines
there:

    Host remote.com
      Port 30234

Make sure the `Port` line below the `Host` is indented.

This way you don't have to specify the port on the command line.

    mymachine:~$ ssh gajon@remote.com

You could also add the user name in the `config` file:

    Host remote.com
      User gajon
      Port 30234

And then you can just type:

    mymachine:~$ ssh remote.com


### 3.3. Specify which groups can connect.

You can tell OpenSSH that only those users belonging to a certain group
can log in. For example, lets create a group `sshers`, and add your user
to that group, you do that with `root`. In this example I'll be adding the
user `gajon` to the group.

We have to do this on the remote server, and the first thing to do is
create the group:

    remote:~# groupadd sshers

Then check what groups the user is already a member of:

    remote:~# groups gajon
    gajon : users wheel audio video cdrom games

Then make it a member of the `sshers` group:

    remote:~# usermod -G wheel,audio,video,cdrom,games,sshers gajon
    remote:~# groups gajon
    gajon : users wheel audio video cdrom games sshers

Notice that the first group listed with `groups gajon` is `users`, that's
the primary group of the user, the rest are the secondary groups. You
modify the list of secondary groups with `usermod -G`, separating each
group name with a coma. See `man usermod` for more details.

OK, now that you have your new group and your user is part of that group,
let's configure the OpenSSH daemon so that only members of that group can
connect at the remote server. Edit the `sshd_config` file in the server:

    remote:~# vim /etc/ssh/sshd_config

And add the `AllowGroups` property to to file:

    # Only members of group sshers will be able to connect
    AllowGroups sshers

And that's it, any user that is not part of the `sshers` group will not be
able to log in.

### 3.4. Use Public-Key Authentication.

It is not a good idea to type the password of your remote user while
connecting to the remote machine.  The problem is that if someones guesses
your password, or looks over your shoulder while you are typing it, or
somehow it is compromised, that person that has your password can easily
connect to the remote machine from anywhere.

To avoid this danger, you could instead authenticate by setting up a so
called *"Public-Key Cryptography"* authentication.

In Public-Key Cryptography, each party has a pair of keys, one key is
known as the *"private key"* while the other is known as the *"public
key"*. The idea is that when you want to communicate with another party
you use their *public key* and only that who has the corresponding
*private key* can decrypt the message.

This topic of Public-Key Cryptography is quite interesting, if you want to
read more you can read the [Wikipedia page][wpk] about it.

The OpenSSH daemon can be configured to authenticate users with a
Public-Key Cryptography method. The advantage of this scheme is that you
will have your pair of keys, one public and one private, and if someone
managed to obtain your user password it won't matter, because no one will
be able to log in to the remote system without the proper private key
needed to establish the authentication.

Also, for an attacker to be able to log in into your system, not only
he'll need to get your private key, but also the password you use to
decrypt your private key, which is only inside your head.

Let's create your key pair in your local machine:

    mymachine:~$ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/gajon/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/gajon/.ssh/id_rsa.
    Your public key has been saved in /home/gajon/.ssh/id_rsa.pub.
    The key fingerprint is:
    fd:6b:f8:82:85:40:e2:8b:1b:31:47:c6:94:25:47:87 gajon@mymachine

Two keys have been generated, one private (`id_rsa`), and the other public
(`id_rsa.pub`). The password that you entered is used to encrypt your
private key.

Notice that the files were placed in your `$HOME/.ssh/` folder in your
computer. Make sure that's the case, you don't want them scattered on your
file system.

Next you have to copy your public key to the `$HOME/.ssh/authorized_keys`
file in the remote server. The `authorized_keys` file can contain the
information of several public keys, so to avoid overwriting an existing
`authorized_keys` in your remote server, you are going to append the
contents of your public key to the `authorized_keys` file. And if that
file did not exist, it will be created anyway.

    mymachine:~$ cat .ssh/id_rsa.pub | ssh gajon@remote.com "cat >> .ssh/authorized_keys"

Now to connect to the remote machine using Public-Key Cryptography
authentication, you need to use the `-i` switch and specify the
**private** key to use:

    mymachine:~$ ssh -i ~/.ssh/id_rsa gajon@remote.com
    Enter passphrase for key '/home/gajon/.ssh/id_rsa':
    remote.com:~$

Notice that your `ssh` client is asking you for your private key password,
which is NOT the same password for your system user, but the password you
entered when you created this pair of keys.

After you have seen that public key authentication works, it is a good
idea to disable simple password logins on your remote machine, so that the
only way of connecting to it is with the use of Public-Key Cryptography.
To do that you'll need to set the `PasswordAuthentication` property with
the value of `no`, in the `sshd_config` file.

    remote.com:~# vim /etc/ssh/sshd_config

Disable tunnelled clear text passwords:

    PasswordAuthentication no

Restart your `sshd` daemon, and now you will only be able to log in to the
remote server if the user in the remote server has the public key info in
the `.ssh/authorized_keys` file, AND you have the corresponding private
key in your own machine, and of course you know the password to that
private key.

**BE VERY CAREFUL**, after you make the above change and restart the
daemon, **do not** log out of your session just yet, open up another
terminal and try to make a second connection to your remote machine. If
you can't, try to figure it out in the other session that you still have
open. I'm telling you this because if you disable password authentication,
and the public key authentication fails, you could easily lock yourself
out of the remote machine, which would be really bad.

Now, it is annoying having to specify the private key to use with the `-i`
switch every time you want to connect to the remote machine. So instead of
that, you can configure what private key to use in the `$HOME/.ssh/config`
file in your local machine.

Retaking the example we saw in section 3.2, you add the `IdentityFile`
setting which specifies a private key to use when connecting to the host.

    Host remote.com
      User gajon
      IdentityFile ~/.ssh/id_rsa
      Port 30234

This way it's not necessary to specify the private key on the command
line, nor the user or the port (if it were different from the default
`22`):

    mymachine:~$ ssh remote.com
    Enter passphrase for key '/home/gajon/.ssh/id_rsa':
    remote.com:~$

A final note; you can name your keys any way you want. If you have more
than one remote machine you should give more meaningful names to your key
pairs, so that you can more easily manage them. For example, if we had
three servers, *"breakpoint"*, *"lucretia"* and *"she-wolf"*, we could
create a `$HOME/.ssh/config` file with the following entries:

    Host breakpoint
      HostName 11.22.33.44
      User gajon
      IdentityFile ~/.ssh/id_rsa_breakpoint
      Port 30234

    Host lucretia
      HostName lucretia.example.com
      User johnny
      IdentityFile ~/.ssh/id_rsa_lucretia

    Host she-wolf.remote.com
      User admin
      IdentityFile ~/.ssh/id_rsa_shewolf

Notice that I added another setting, `HostName`, which specifies the
actual host to connect to. It could be an IP address or a fully qualified
name of the server. That way the `Host` setting is actually an alias that
you can freely define. With that example we can easily connect to the
*"breakpoint"* server, with an IP address of `11.22.33.44`.

    mymachine:~$ ssh breakpoint


[wpk]: http://en.wikipedia.org/wiki/Public-key_cryptography "Public-Key Cryptography - Wikipedia page"


## And that's all for now.

I hope these tips are useful to you, but remember that we have just seen a
few uses of OpenSSH, and that there are a lot more. For example, in a next
article I'll write how to use Public-Key authentication to easily issue
commands to remote servers and automate tasks.


<!-- vim: set tw=74 sw=4 ts=4 et spell filetype=mkd: -->
