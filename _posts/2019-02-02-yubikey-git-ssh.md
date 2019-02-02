---
layout: post
title:  "Setting up Yubikey 5 for Git (GPG) and SSH"
date:   2019-02-02 10:00:00 +1200
---
I recently got a couple of Yubikey 5, the main reason is they are slowly getting popular for MFA, but they also support OpenPGP. OpenPGP lends itself well to having verified commits but also SSH, this post is a guide on setting up the key for this purpose.

The first thing to do is to download and install [gpg4win](https://www.gpg4win.org/download.html)

# Private GPG Key
## Keybase
I've been using [Keybase](https://keybase.io) for a while and trust them, so I used this as my starting point. There is a [Github Issue](https://github.com/keybase/keybase-issues/issues/1773) which describes how to export the key using the UI.

### Import the Key
Now that we have the private key from Keybase we are ready to import it. This can be done using the following command:

```
gpg --import private.key
```

When I imported my key I noticed that it my key only had signing, certify capabilities and a subkey with encrypt capabilities, but we will need more than this for SSH.

This was the output from listing the keys
```
pub   rsa4096 2017-12-13 [SC] [expires: 2033-12-09]
      SOMEVERYLONGKEYID
uid           [ultimate] Bhavik Kumar <contact@bhavik.io>
sub   rsa4096 2017-12-13 [E] [expires: 2033-12-09]
```

If the first line for you contains `[SCEA]` for the capabilities then you can skip this step, however if it does not then you will have to run the following command:

```
gpg --edit-key contact@bhavik.io
gpg> change-usage
```

Depending on the current allowed actions, toggle the appropriate capabilities until the current allowed actions reads: `Sign Certify Encrypt Authenticate`. Then press `q` to finish.

Then type `save` to complete the process.

## YubiKey 5 (Local Machine)
The other option is to generate the private key on your local machine using the following command:

```
gpg --expert --full-gen-key
```

For the options I will recommend the following:

1. For the kind of key select 8 for RSA (set your own capabilities)
2. Depend on the current allowed actions, toggle the appropriate capabilities until the current allowed actions reads: `Sign Certify Encrypt Authenticate`. Then press `q` to finish.
3. For the size of the key I would recommend 4096 bits.
4. For the key expiry its personal choice but I recommend `0 = key does not expire` - more on this later.
5. For the real name type both your first and last name
6. Enter your email address
7. You can leave comment blank
8. Select (O)kay
9. The key will be generated and will prompt for a [passphrase](https://xkcd.com/936/).

## Additional IDs
If you need more than one ID (email address) attached to your private key than this can be done by using the following command:

```
gpg --edit-key contact@bhavik.io
gpg> adduid
```

1. Enter in your first and last name for the Real name field.
2. Enter in the email address
3. The comment field is optional and can be left blank.
4. Select (O)kay
5. Type `save` to complete the process.

# Generate Private Subkeys
The next step is to generate the signing and authentication subkeys. This can be done with the following command:

```
gpg --expert --edit-key contact@bhavik.io
gpg> addkey
```

1. Select 8 for RSA (set your own capabilities)
2. Toggle the appropriate capabilities until the current allowed actions reads: `Authenticate`. Then press `q` to finish.
3. Specify the key size as 4096
4. This should be the same as the master key
5. Select `y` for the next couple of prompts to create the key
6. Enter the passphrase for the key.

Repeat the process except the capabilities on the key should read `Sign`.

Type `save` to complete the process.

# Export your Private Keys
Now we don't want to lose the master key, but we also don't want to keep the master key on our machine. Before we can export the private keys we need to know what our key ids are run the following command to get the output of all key ids.
```
gpg --keyid-format LONG -k
```

This is the example output from listing the keys
```
pub   rsa4096/MASTERKEYID 2017-12-13 [SC] [expires: 2033-12-09]
      SOMEVERYLONGKEYID
uid                 [ultimate] Bhavik Kumar <contact@bhavik.io>
sub   rsa4096/ENCRYPTKEYID 2017-12-13 [E] [expires: 2033-12-09]
sub   rsa4096/AUTHENTKEYID 2017-12-13 [A] [expires: 2033-12-09]
sub   rsa4096/SIGNINGKEYID 2017-12-13 [S] [expires: 2033-12-09]
```

Run the following command and then I recommend putting the output on a encrypted USB stick and storing it in a safe place. This will allow you to recover your private keys on to another Yubikey, just in case your current one fails.
```
gpg --export-secret-key --armor MASTERKEYID
```

It is also a good idea to export your public key using the following command:
```
gpg --armor --export contact@bhavik.io > public_gpg.asc
```

I recommend updating your Keybase public key with the exported one. Even if you don't use Keybase I recommend it uploading to a public location on the internet (E.g: Github gist). This will help when setting up your Yubikey on another machine and also allow people to send you encrypted messages.

# Importing your private keys on to your Yubikey
Insert your Yubikey 5 into your machine and run the following command:
```
gpg --edit-key contact@bhavik.io
gpg> toggle
```

The default pin is 123456 and the default admin pin is 12345678 for your Yubikey.

1. Type `keytocard` and select `y` to move your primary key
2. Type `key 1`
3. Type `keytocard` and then select `2`. This will move the encryption subkey to your Yubikey.
4. Type `key 1`
5. Type `key 2`
6. Type `keytocard` and then select `3`. This will move the authentication subkey to your Yubikey
7. Type `key 2`
8. Type `key 3`
9. Type `keytocard` and then select `1`. This will move the signing subkey to your Yubikey
10. Type `save` to complete the process

# Secure your Yubikey
The next step is to finalise the setup of your Yubikey. Using the following command:
```
gpg --card-edit
```

During this process you will be prompted for your Yubikeys admin pin.

1. Type `admin`
2. Type `name` and enter in your surname and firstname
3. Type `sex` and enter in either m, f or space.
4. Type `url` and enter in the url of your public key
5. Type `passwd`
6. Select `1`
7. When prompted enter in the default pin of `123456` and then enter a new pin which I recommend to be a passphrase of up to 127 characters.
8. Select `3`
9. When prompted enter in the default admin pin of `12345678` and then enter a new pin I recommend to be a passphrase of up to 127 characters. Do not use the same passphrase as the normal pin.
10. Select `q`
11. Type `quit` to exit

# Setting up GPG on another machine
If you are like me, you will have a couple of machines which you need to setup. Once GPG4Win is installed you can run the following commands and your Yubikey will work exactly like on the first machine that was setup.

This assumes you have your public key on the internet and setup the url on your Yubikey as well.
```
gpg --card-edit
fetch
quit
```

# Setup Git
Assuming you have GIT installed, the following commands can be run to setup Git to sign commits.

```
git config --global gpg.program "c:\Program Files (x86)\GnuPG\bin\gpg.exe"
git config --global commit.gpgsign true
git config --global user.signingkey SIGNINGKEYID
```

Now whenever you commit a change, your Yubikey will have to be plugged in and GPG will prompt for your Yubikey PIN to sign the commit.

If you are a GitHub user then you you will need to upload your public gpg keys using the following [guide](https://help.github.com/articles/adding-a-new-gpg-key-to-your-github-account/) before GitHub will verify your commits.

# Setup SSH Key
A little known fact is that you can use GPG to generate a public ssh key which you can use for Git or logging into machines. At the time of writing this, each developer uses their own SSH key to login to machines.

On windows I still prefer to use Windows native tools instead of MinGW, Cygwin or Git bash. Therefore I have Putty for SSH as I find it easier to use.

The first thing to do is tell Git to use [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). This can be done by setting the environment variable `GIT_SSH` to have a value of `C:\Program Files\PuTTY\plink.exe`

The next setup is to enable putty support in GPG. This can be done by adding the line `enable-putty-support` to `gpg-agent.conf` in `C:\Users\%USER%\AppData\Roaming\gnupg`.

For OpenSSH you will need to add the line `enable-ssh-support` instead.

Now run the following commands:
```
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
```

Run the following command to get a public ssh key:
```
gpg --export-ssh-key contact@bhavik.io > id_rsa.pub
```

Now you can upload this public key to GitHub and machines which you need to SSH to and your Yubikey will need to be plugged in and GPG will prompt for your PIN.
