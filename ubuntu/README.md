# Ubuntu

My primary linux system is Ubuntu 22.04.03 LTS (jammy).  The following
information explains how to set up the system after a fresh install.

## Initial Settings

1. On first login, connect to Ubuntu, Google, and Microsoft accounts.
1. Change the user-specific group to `users`.
1. Change the following line in `/etc/sudoers`:
```
%sudo	ALL=(ALL:ALL) NOPASSWD:ALL
```
1. Start firefox and log in to my firefox account.  This should install add-ons
   and favorites registered with the account.
1. On firefox, log in to the Lastpass password vault.  Lastpass should already
   be installed by the previous step.  If it is not install then install it as
   a firefox add-on.

## Emacs

1. Since this is the first install, update the package library and then
   install emacs:
``` bash
apt-get update
apt-get install emacs
```
1. Copy the file [`emacs.conf`](emacs.conf) to `~/.emacs`.
1. Copy the emacs turd cleaner [`clean`](clean) to `~/bin`.
1. Set emacs as the default editor by adding the following line to `~/.profile`:
``` bash
export EDITOR=/usr/bin/emacs
```

## Markdown

### Viewer

Almost all of the markdown viewers for linux are out-of-date and produce
warnings and errors if they work at all.  The best solution is to use the
GitHub markdown viewer add-on for firefox.  This add-on should have been
installed upon the first login to firefox.  Otherwise, install the add-on
manually.

### Configuration

1. Make the MIME packages directory:
``` bash
mkdir -p ~/.local/share/mime/packages
```
1. Copy the file [`text-markdown.xml`](text-markdown.xml) to
`~/.local/share/mime/packages/text-markdown.xml`.
1. Update the MIME database:
``` bash
update-mime-database ~/.local/share/mime
```

### emacs

1. Install (package-install) the `markdown-mode` package for emacs.  This will
   add two modes: `markdown-mode` for standard markdown and `gfm-mode` for
   git-flavored markdown.
1. Add the following line to `~\.emacs` so that GFM mode is the default for
   `README.md` files:
``` emacs-lisp
(add-to-list 'auto-mode-alist '("\\.md\\'" . gfm-mode))
```

## Git

### Installation

Install Git and all of its associated GUIs and tools:
``` bash
apt-get install git
apt-get install gitk
apt-get install git-gui
```

### Global Configuration

The following commands will write global settings to `~/.gitconfig`:
``` bash
git config --global user.name "Jeffery Cavallaro"
git config --global user.email "jeffery@cavcom.com"
git config --global core.editor "/usr/bin/emacs"
```

## Brasero

This is the standard tool for cd/dvd rom write and copy operations:
``` bash
apt-get install brasero
```

## Codecs
Most of the needed codecs for A/V are in proprietary packages and must be
installed separately:
``` bash
apt-get install ubuntu-restricted-extras
```

Accept all license agreements during the installation.

## Development

Install the compilers and debuggers:
``` bash
apt-get install build-essential
```

## ssh

If not installed, install and enable ssh support:
``` bash
# apt-get install ssh
# systemctl enable ssh
```

## samba

The Ubuntu systems serves as a samba server for our windows boxes.  Setup is
as follows:

1. Install samba:
``` bash
apt-get install samba
```
1. Stop the server.
``` bash
systemctl stop smbd
```
1. Copy the [smb config](smb.conf) to `/etc/smb.conf`.
1. Make the samba data directory and make it owned by the selected samba user.

``` bash
mkdir /data/samba
chown ${SAMBA_USER}:users /data/samba
```
1. Make a samba password.  This is the username and password that will need to
   be specified during authentication on the windows side.
``` bash
smbpasswd -a ${SAMBA_USER}
```
1. Restart and enable the server.
``` bash
systemctl start smbd
systemctl enable smbd
```

# VSCode

I have left JetBrains behind and have embraced vscode.  This is best installed
from the snap store:
``` bash
snap install --classic code
```

# Python

The version of Python 3 that comes with Jammy is sufficient (3.10).  However,
a venv should be created and the version of pip upgraded:

1. Install packages:
``` bash
apt-get install python3-pip
apt-get install python3-venv
pip3 install --upgrade pip
```

# TeXLive

This is the LaTeX distribution that I use for all of my math papers.

1. Download `install-tl-unx.tar.gz` from the TUG website.
1. Unpack the downloaded archive.
1. Run the installer: `install-tl`.
1. Use option `D` to change the install directory to `/opt`.
1. Use option `O` to make letter size paper the default.
1. Use option `I` to start the installation.
1. Update PATH.

## Docker

Docker uses the following ports.  Note that the encrypted and unencrypted port
numbers are used only if the client and server are communicating over a TCP
socket as opposed to a domain socket (`/var/run/docker.sock`).

| PORT | USE |
|---|---|
| 2375 | Unencrypted |
| 2376 | Encrypted |
| 2377 | Swarm |

Installation is as follows:

1. Install curl:
``` bash
apt-get install curl
```
1. Get the docker GPG key:
``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker.gpg
```
1. Set up the repository:
``` bash
echo \
  "deb [arch="$(dpkg --print-architecture)" \
signed-by=/usr/share/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" \
  | tee /etc/apt/sources.list.d/docker.list > /dev/null
```
1. Install docker:
``` bash
apt-get update
apt-get install \
    docker-ce docker-ce-cli containerd.io docker-buildx-plugin \
    docker-compose-plugin
```
1. Add users to the `docker` group so that they can access the domain socket
   used for docker client/server communication.
1. Test the installation:
``` bash
docker run hello-world
```

## Virtualbox

The use of VirtualBox is under review due to its need to invasively patch the
kernel and because it is an Oracle product.  A shift to vanilla KVM is being
contemplated.
``` bash
# wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
# sudo add-apt-repository "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib"
# apt-get update
# apt-get install build-essential
# apt-get install virtualbox
```

If EFI secure boot is enabled then the system will walk through the steps to
install a new MOK.

## Thunderbird

|NAME|VALUE|
|----|-----|
|Username|jeffery@cavcom.com|
|IMAP Server|netsol-imap-oxcs.hostingplatform.com|
|IMAP Port|993|
|IMAP Connection Security|SSL/TLS|
|IMAP Authentication Method|Normal Password|
|SMTP Server|netsol-imap-oxcs.hostingplatform.com|
|SMTP Port|465|
|SMTP Connection Security|SSL/TLS|
|SMTP Authentication Method|Normal Password|
