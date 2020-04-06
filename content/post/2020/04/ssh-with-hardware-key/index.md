---
title: How to SSH with a hardware key
subtitle: Powered by the GPG gang
date: 2020-04-06
author: Antonin DUPONT, Duponin
tags:
  - SSH
  - GPG
---

There are many reasons why you want a hardware key.
From security to ease of configuration.
When you can't trust devices on which you are, hardware keys are a good choice.
Even if you trust devices on which you are, complexity to manage configuration goes exponential the more you have devices.

<!--more-->

GPG can be resumed with 3 features:

  - Encryption,
  - Signing,
  - Authentication.

While encryption and signing features are well known, authentication feature is curiously less.

If you're used to SSH and have password on your private keys (if you don't, you really should), you probably heard about SSH agent.
That agent is a handy program that runs in background.
It'll cache your private ssh key while it runs, avoiding you to type key's passphrase everytime you want to use it.
Sounds cool we agree? 

If you want to use GPG authentication feature, you have to replace SSH agent by GPG agent.
GPG agent does same thing as SSH agent but for GPG.
Aaaaaaand SSH too.

***mindblowing***

About hardware key, a big pro and con about it: once you inserted a key in, you never be able to remove it.
If you don't have backup or lost passcode… `¯\_(ツ)_/¯`

## Key preparation

For demonstration purpose a [Nitrokey Pro 2](https://shop.nitrokey.com/shop/product/nitrokey-pro-2-3) will be used.
It's an open-source and open-hardware key that's supports GPG keys.

Be sure you have drivers to use your key.

Let's find if hardware key is detected with `gpg2 --card-status`.
It does?
Good news.
Edit card configuration with `gpg2 --edit-card`.
Let's do a `factory-reset` in `admin` mode to be sure to start from a clean device.

Let's generate a GPG key on the card

```
gpg/card> generate
Make off-card backup of encryption key? (Y/n) n

Please note that the factory settings of the PINs are
   PIN = '123456'     Admin PIN = '12345678'
You should change them using the command --change-pin

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 42d
Key expires at Mon 18 May 2020 06:25:40 PM CEST
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Antonin DUPONT
Email address: duponin@locahlo.st
Comment: Duponin
You selected this USER-ID:
    "Antonin DUPONT (Duponin) <duponin@locahlo.st>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
gpg: key FB5586DB6E88C073 marked as ultimately trusted
gpg: revocation certificate stored as '~/.gnupg/openpgp-revocs.d/69A6EAD1FAD7BD5EC53820C1FB5586DB6E88C073.rev'
public and secret key created and signed.


gpg/card>
```

A `list` will show us information about hardware key and GPG keys:

```
gpg/card> list

[...]

Signature key ....: 69A6 EAD1 FAD7 BD5E C538  20C1 FB55 86DB 6E88 C073
      created ....: 2020-04-06 16:25:58
Encryption key....: 63F5 7147 3DB2 3960 B8B4  1F4F 2FEA 620C 7BDD 7F32
      created ....: 2020-04-06 16:25:58
Authentication key: 3A21 11D8 0729 35FE D73B  DD35 A247 F141 42B4 AFB8
      created ....: 2020-04-06 16:25:58
General key info..:
pub  rsa2048/FB5586DB6E88C073 2020-04-06 Antonin DUPONT (Duponin) <duponin@locahlo.st>
sec>  rsa2048/FB5586DB6E88C073  created: 2020-04-06  expires: 2020-05-18
                                card-no: **** ********
ssb>  rsa2048/A247F14142B4AFB8  created: 2020-04-06  expires: 2020-05-18
                                card-no: **** ********
ssb>  rsa2048/2FEA620C7BDD7F32  created: 2020-04-06  expires: 2020-05-18
                                card-no: **** ********

gpg/card>
```

To explain a bit what happened, GPG created on hardware key a private and generated 3 subkeys:

  - An encryption subkey,
  - A signing subkey,
  - An authentication subkey.

Even if these are useful, in that post we'll only use authetinaction subkey.
We'll cover two remaining subkeys in another post.

Our hardware key is now ready.

## GPG agent

We saw earlier that GPG agent can replace SSH agent.
It wasn't a lie.
It can be turned on with a single line of configuration:

```bash
$ echo "enable-ssh-support" >> ~/.gnupg/gpg-agent.conf
```

Your shell will need a small configuration too to be sure GPG agent is used instead of SSH agent.
Place following lines in your `~/.bashrc`/`~/.zshrc`:

```bash
export SSH_AGENT_PID=""
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
```

Once environment is set up, ssh public key is needed to authenticate with destination server.
To export it:

```bash
$ gpg2 --export-ssh-key 69A6EAD1FAD7BD5EC53820C1FB5586DB6E88C073
ssh-rsa AAAAB3NzaC1yc2EAAAA[...]ek5tcmV openpgp:0x42******
```

The output is your public ssh key.
You are free to distribute it but it's better in an `authorized_keys`.

To check if your SSH agent (which is GPG agent) is aware about your private/hardware key:

```bash
$ ssh-add -l
2048 SHA256:HqKODS3OAxtOBejGCoOyiGJsZYnJRq7E4GIWviRJUl8 cardno:000********* (RSA)
```

That command lists all SSH private keys known by the current SSH agent.
And our looks like it does!
If you have:

```bash
$ ssh-add -l
The agent has no identities.
```

Wait a little and try again.
If there is still nothing, try to disconnect and connect again your hardware key.
It should appears.

Now if you try to connect to server on which you add your new public key, a prompt should ask for your hardware key user password.
If it don't, it means you already unlocked it and GPG agent cached it.

Try connect to your servers on which you put generated pubkey, it should prompt you your hardware key user credentials.
Congratulations, you just logged with a hardware key!

## But what about new device?

To connect from a device is cool.
To connect from another device is better.
There's a way to avoid to be locked with a single device.
Secret key has to be exported.
If it was generated like I did, without a backup, you can store the export somewhere without being worried about.

>To backup `~/.gnupg` is a good idea to avoid bad surprises.
>A hardware key without keygrip reference is quite useless.

Following commands show how to export and import:

```bash
# Original device
$ gpg2 --export-secret-key --armor 69A6EAD1FAD7BD5EC53820C1FB5586DB6E88C073 > my_super_key.asc

# New device
$ gpg2 --import my_super_key.asc
$ gpg2 --card-status
$ gpg2 --edit-key 69A6EAD1FAD7BD5EC53820C1FB5586DB6E88C073
gpg> trust
gpg> 5
gpg> y
gpg> quit
```

To do a `--card-status` is mandatory to get the key available in keyring.
The five last commands are fix key trust.
Default GPG behavior is to not trust imported key.
Just be sure you don't miss needed drivers (such as `scdaemon`), config for your GPG agent and your shell.

Have fun!
