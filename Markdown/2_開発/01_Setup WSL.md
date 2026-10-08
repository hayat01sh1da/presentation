[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/setup-wsl)

<img src="https://hackmd.io/_uploads/B1jBtFHOA.png" alt="WSL on Windows" />

## 1. Environment

Ubuntu 20.04.5 LTS

## 2. Install Package

Update package manager

```bash
$ sudo apt update
```

* Upgrade installed packages

```bash
$ sudo apt upgrade -y
```

## 3. Install Git

Only if Git is not installed yet after you run `sudo apt update` and `sudo apt upgrade`.

### 3-1. Default Package

You will install an older version than the latest one in this way.

Install Git.

```bash
$ sudo apt install git
```

Check the installed Git version.

```bash
$ git --version
```

### 3-2. Install from a Source

You will install the latest version of Git although this way takes more time and cannot be managed by Apt.

Install Git.

```bash
$ sudo apt install make libssl-dev libghc-zlib-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip
```

Check the installed Git version.

```bash
$ git --version
```

## 4. Git Settings

Assign username and email.

```bash
$ git config --global user.name "[username]"
$ git config --global user.email "[email]"
```

Omit `--global` to assign username and email in each project.

Check the configuration.

```bash
$ git config --list
```

```bash
user.name=[username]
user.email=[email]
```

## 5. SSH Settings

Generate a SSH key.

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/[username_ubuntu]/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/[username_ubuntu]/.ssh/id_rsa.
Your public key has been saved in /home/[username_ubuntu]/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:hxbYsu6UHql6qA4tIIA7BnbVktsubFbEV6ewBQw+hmM [username_ubuntu]@DESKTOP-QDJ9F6V
The key's randomart image is:
+---[RSA 2048]----+
|     hogehoge    |
|      foobar     |
|     hogehoge    |
|      foobar     |
|     hogehoge    |
|      foobar     |
|     hogehoge    |
|      foobar     |
|     hogehoge    |
+----[SHA256]-----+
```

Create a `config` file and take down hostname, domain, username and path to the secret key.

```bash
vim ~/.ssh/config
```

```bash
Host github
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
```

Or

```bash
Host gitlab
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_rsa
```

***

I faced some troubles here.

1. Ubuntu refers to `~/.ssh/id_rsa` while Windows detects `/mnt/c/Users/[username_windows]/.ssh/id_rsa`, so I had to manage both because I wanted to handle Git commands in IDE on the host OS, Windows.
2. When I put a symbolic file of `~/.ssh/` on Ubuntu on Windows, Windows could not read it because of permission, so I failed to use Git in your editor.
3. When I put a symbolic file of `/mnt/c/Users/[username_windows]/.ssh/id_rsa` on Windows to Ubuntu, Ubuntu warned that permission was too loose.

After all, I placed the same `~/.ssh` on both Ubuntu and Windows, but I doubt that this is cool.  
Somebody, please tell me a better solution.

## 6. GitHub Settings

### 6-2. Register the Public Key to GitHub.

Print and copy the string in `~/.ssh/id_rsa.pub` to clipboard.

```bash
$ cat ~/.ssh/id_rsa.pub
ssh-rsa HOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobarHOGEHOGEFOOBARhogehogefoobar [username_ubuntu]@DESKTOP-XXXXXXX
```

* Click `Setting`

<img src="https://user-images.githubusercontent.com/37478830/194862850-a1cdaaf3-6dec-4a6a-a117-767ab4979c48.png" alt="GitHub -Click Setting-" />

* Click `SSH and GPG keys`

<img src="https://user-images.githubusercontent.com/37478830/194862852-45627f50-ae7c-4f3e-b8dd-a55213feb006.png" alt="GitHub -Click SSH and GPG keys-" />

* Click `New SSH key`

<img src="https://user-images.githubusercontent.com/37478830/194862854-dfaded2d-2306-4b39-a574-b3d77bfaf490.png" alt="GitHub -Click New SSH key-" />

* Label an arbitrary name, paste the string of the public key to register.

<img src="https://user-images.githubusercontent.com/37478830/194862859-1359e74b-ac97-4e41-8b48-ddd323ca6ef6.png" alt="GitHub -Key Registration-" />

* Check if the key is registered successfully.

<img src="https://user-images.githubusercontent.com/37478830/194862863-a8bdf3fb-fe4c-4d70-9a10-a55c1a9ae17b.png" alt="GitHub -Confirm Key-" />

### 6-2. Check Connection

* Check the connection with GitHub / GitLab.

```bash
$ ssh github
PTY allocation request failed on channel 0
Hi oasis-forever! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

## 7. Reference

* [How To Gitインストール on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-18-04) - Last Accessed: 12 September 2019
* [Florian Brinkmann](https://florianbrinkmann.com/en/ssh-key-and-the-windows-subsystem-for-linux-3436/) - Last Accessed: 12 September 2019
* [WSLでもgitでsshを使いたい](https://qiita.com/angel_katayoku/items/9d95241af81dc4466ee1) - Last Accessed: 12 September 2019
* [今日からはじめるGitHub 〜 初心者がGitをインストールして、プルリクできるようになるまでを解説](https://employment.en-japan.com/engineerhub/entry/en/2017/01/31/110000) - Last Accessed: 12 September 2019
