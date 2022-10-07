---
title: YubiKey 5C commit signing and SSH authentication
date: 2022-10-07T08:18:16+00:00
description: Setting Up the YubiKey 5C for Development with GPG Commit Signing and SSH Authentication
menu:
    sidebar:
        name: YubiKey 5C Setup
        identifier: yubikey-5c-setup
        weight: 5
author:
    name: Nick Ruffles
    image: /images/author/nick.png
tags: ["Security", "YubiKey", "SSH",  "Authentication", "GPG"]
categories: ["Security"]
hero: yubikey.jpeg
---

While being onboarded to my new job at [EngineerBetter](https://engineerbetter.com), I was handed a YubiKey and asked to
set it up with SSH authentication and commit signing. While I had heard of GPG and used it slightly before, I wasn't aware
that you could also use it in git to sign your commits in order to prove that you were the one to commit code. 

The setup for my YubiKey took me about a day, as there was issue after issue with the YubiKey, so in this guide I hope to 
show a relatively painless and secure setup for the YubiKey.

This blog will walk through how to set up `SSH authentication` and `commit signing` with `GPG keys` on the `YubiKey 5C` for `macOS`. 
This will be very useful for developers, however if you write any amount of code it can still be cool to set up. 
Storing your SSH key on the YubiKey allows you to use the same SSH key on multiple devices by simply plugging in the key 
and typing a password (really cool if you have multiple devices)

## Useful commands and information
```
Default PIN: 123456
Default Admin PIN: 12345678
```

If you have an experience anything similar to me, then you'll likely need to reset the YubiKey keys or GPG values a couple of times. You can reset the YubiKey using the `YubiKey Manager` by going to `Applications -> PIV -> Reset` or by entering the following command into the terminal

```bash
$ ykman openpgp reset # Reset the GPG keys
$ ykman piv reset     # Reset the PIN, Admin PIN and Management Key
```

## Setup for macOS

We need to install the `openssh` version of ssh, as the default Mac one does not have the features we need. The other dependencies are required for YubiKey cli.

```bash
$ brew install gnupg yubikey-personalization hopenpgp-tools ykman pinentry-mac openssh
$ export PATH=$(brew --prefix openssh)/bin:$PATH
```

Set up the GPG agent to prompt for a pin using `pinentry-mac`, this will allow us to enter the YubiKey pin when we first plug the YubiKey into our mac. We'll also tell GPG to support SSH, so that we can use our GPG key as an SSH key - More on this later.

```bash
$ cat <<EOF>> ~/.gnupg/gpg-agent.conf
# if on MacOS, we recommend you use pinentry-mac
# otherwise, look for 'pinentry' on your system (e.g. pinentry-gnome3 or pinentry-tty).
# If you don't set this, the default pinentry will be used
pinentry-program `which pinentry-mac`

# enables SSH support (ssh-agent)
enable-ssh-support
EOF
```

Run these commands to restart the GPG agent

```bash
$ gpg-connect-agent killagent /bye
$ gpg-connect-agent /bye
```

Set the retries attempt for the pin, as 3 retries is the default and this would lock us out of the YubiKey quite quickly if we forget the password or typo on occasion.

```bash
$ ykman piv set-pin-retries 10 10
```

## Generating the GPG key

Create the GPG keys on the YubiKey, this will be stored directly on the YubiKey (not the local file system). The YubiKey will need to be inserted in order to authenticate, sign or encrypt data.

```bash
$ gpg --edit-card
gpg> admin
gpg> generate
# Follow setup, make sure to set an email that is used in your Github account. 
# You can add multiple emails to your Github if 
# you decide to use a private Github account with a work email.
# Set the expiry date for the key to be in the region of 2 years
# You'll also be asked if you want to backup your encryption key, if you plan
# on using the YubiKey only for SSH/Commit signing then choose no. 
gpg> quit

# View the key
$ gpg --list-secret --keyid-format SHORT nick.ruffles@engineerbetter.com
sec> rsa2048/FEDECADE 2022-08-10 [SC] [expires: 2024-08-09]
 FEEDBEEFC0C0A7D867D34ADEADD0D0CAFEDECADE
 Card serial no. = 0006 19347066
uid  [ultimate] Nick Ruffles <nick.ruffles@engineerbetter.com>
ssb> rsa2048/F8D33758 2022-08-10 [A] [expires: 2024-08-09]
ssb> rsa2048/A809A024 2022-08-10 [E] [expires: 2024-08-09]
```

Make a note of the long value that you are shown, in this case `FEEDBEEFC0C0A7D867D34ADEADD0D0CAFEDECADE` as well as the short identifier after`rsa2048` on the line starting with `sec>`, in this case `FEDECADE`. You'll see these values being used in later commands.

Retrieve your SSH public key and your GPG public key from your YubiKey and place them [in your GitHub keys section](https://github.com/settings/keys)
```bash
# Lists the ssh key we just generated
$ ssh-add -L
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC9Tjj2J16yjs+Msb7+G2+HU5LxEjrMVT5zSdj+kOgcOPw6uZhVKasTsWfZDgpXnqebLBCEw5HQtgcKztCmcSO4t3lNHhtXKuy2fKHLfmZOt3R9xISXW0BKNhyC9zVjHXwK1MFLETGaQuUden7jvTRmVSWlp7mqm+CiBdGhKLt04SQzLa/S/UKcNT7Re79C6qfD/+7lvzL20Zo1fpt2aVh8dvWZymZAICtfQ4l/80BKsgZ+J50VXE63suboJcW1nxsRexWSX6lkAdUSPpx3nAhtuev+Qh5fUwDkMMjmwp+UGHr3xcmJn/WJZyuugBrkf0LaOqGh+AEx6ZFYYHRFHLyZ cardno:19 123 456
# Place this key in the SSH section of Github keys, should be denoted by a `cardno` prefix

# Using the long identifier we got from the above block
$ gpg --armor --export FEEDBEEFC0C0A7D867D34ADEADD0D0CAFEDECADE # Long identifier 
-----BEGIN PGP PUBLIC KEY BLOCK-----
mQENBGLzavsBCADPRKNmZHoID7Cn4DMmKreVFb1RoF4Ghez3EVwZbvQ3KWf56Sh6
[...snip...]
C/CCi48etdQebwvdv84UdeNoc/yIveis
-----END PGP PUBLIC KEY BLOCK-----
# Place this key in the GPG section of Github keys
```

Test the ssh authentication with GitHub, useful for pushing code to your git repo

```bash
$ ssh -T git@github.com
Hi TheDarthMole! You've successfully authenticated, but GitHub does not provide shell access.
```

## Setting up GPG for commit signing

We will also set up signing git commits with the GPG key, as this will allow git users to verify that the code was committed by you.

```bash
git config --global commit.gpgsign true
git config --global user.signingkey FEDECADE # Small identifier from earlier
git config --global user.name "Nick Ruffles"
git config --global user.email "nick.ruffles@engineerbetter.com"
```

Now to test that the GPG key is successfully signing the commits

```bash
$ mkdir /tmp/foo
$ git init
$ touch file
$ git add file
$ git commit -m "test"
$ git log --show-signature
commit 50afc824b6470002bf354b129155de291941b9d2 (HEAD -> main)
gpg: Signature made Wed 10 Aug 12:06:41 2022 BST
gpg: using RSA key 35AC1AB55DC65EC3E040BAB12F864F12CE8BC0A
gpg: Good signature from "Nick Ruffles <nick.ruffles@engineerbetter.com>" [ultimate]
Author: Nick Ruffles <nick.ruffles@engineerbetter.com>
Date: Wed Aug 10 12:06:41 2022 +0100

     test
```

If you get a similar output, the signing worked! You can now rest assured that code pushed from your computer is verified that it has come from your computer.

{{< img src="/posts/yubikey/commit_verified.png" align="center" title="Verified commit using GPG key" >}}

You can *optionally* go to GitHub and toggle `Vigilant mode` that will flag unsigned commits as unverified.

## Making our new GPG key public

Now that we have some working keys (Hopefully!!), we can put the public keys in an online repo so that other people can validate our commits

```bash
$ gpg --list-secret nick.ruffles@engineerbetter.com # Enter the email you used for your key

# Copy the key from the output and replace it in the following commands
$ gpg --keyserver keys.openpgp.org     --send-key 35AC1AB55DC65EC3E040BAB12F864F12CE8BC0A
$ gpg --keyserver keys.gnupg.net       --send-key 35AC1AB55DC65EC3E040BAB12F864F12CE8BC0A
$ gpg --keyserver pgp.mit.edu          --send-key 35AC1AB55DC65EC3E040BAB12F864F12CE8BC0A
$ gpg --keyserver keyserver.ubuntu.com --send-key 35AC1AB55DC65EC3E040BAB12F864F12CE8BC0A
```

You can view your keys URL by going to the keyserver website and searching for the id of your key.

## Changing the default credentials

You **SHOULD ABSOLUTELY** change the pins for your YubiKey. Anyone with the YubiKey and your `PIN` can mount and use your GPG credentials SO MAKE SURE YOU CHANGE THE DEFAULT VALUES!!!

You'll be entering the `PIN` when you use ssh or sign a commit, so make sure it's memorable. To change some identifiers and the pins (passwd) enter the following commands:

```bash
gpg --card-edit
# Toggle into admin mode, allowing us to edit name, sex, url and passwords
gpg> admin
gpg> name
gpg> sex
# The url for your public key, set this to use YubiKey with multiple devices
# e.g. keys.openpgp.org/vks/v1/by-fingerprint/35AC1AB55DC65EC3E040BAB12F864F12CE8BC0A
gpg> url
gpg> passwd
gpg> quit
```

Follow the instructions to change the `PIN` and the `Admin PIN`.¡

## Optional security improvements

Some optional security improvements that require touching the YubiKey for performing different operations

```bash
ykman openpgp keys set-touch aut on # Require touch for authentication
ykman openpgp keys set-touch enc on # Require touch for encryption
ykman openpgp keys set-touch sig on # Require touch for signature signing
```

## Using YubiKey with another machine

If you've successfully set up the YubiKey following this guide, then you should be able to simply repeat steps in [[00 - YubiKey Setup Guide#Setup for MacOS]] and [[00 - YubiKey Setup Guide#Setting up GPG for commit signing]] along with the following commands to retrieve the public keys from the YubiKey:

```bash
gpg --card-edit
> fetch
> quit
```

The `fetch` will download keys set in the `url` parameter we set earlier, so make sure you've set it beforehand!

The key should be all setup on your new machine! Easy right?

## Honorable mentions

This guide has been a collection of existing guides, google searches and trial-and-error for setting up the YubiKey, here are some of the resources I used in this guide:

- [Paddy Steed's SSH Auth guide](https://www.engineerbetter.com/blog/yubikey-ssh/)
- [Paddy Steed's Signed Git Commits guide ](https://www.engineerbetter.com/blog/yubikey-signed-commits/)
- [Bhavik Kumar's Yubikey 5 for Git guide](https://bhavik.io/2019/02/02/yubikey-git-ssh.html)
- [Webbmaffian's YubiKey on Mac guide](https://github.com/webbmaffian/yubikey-ssh)
- Stack Overflow
- Cups of Tea ☕️