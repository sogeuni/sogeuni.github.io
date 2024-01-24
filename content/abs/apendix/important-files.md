---
title: Appendix H. Important Files
---


**startup files**

These files contain the aliases and [[othertypesv#^ENVREF|environmental variables]] made available to Bash running as a user shell and to all Bash scripts invoked after system initialization.

/etc/profile

Systemwide defaults, mostly setting the environment (all Bourne-type shells, not just Bash [^1])

/etc/bashrc

systemwide functions and [[../advanced-topics/aliases#^ALIASREF|aliases]] for Bash

$HOME/.bash_profile

user-specific Bash environmental default settings, found in each user's home directory (the local counterpart to /etc/profile)

$HOME/.bashrc

user-specific Bash init file, found in each user's home directory (the local counterpart to /etc/bashrc). Only interactive shells and user scripts read this file. See [[sample-bashrc-and-bash-profile-files.html|Appendix M]] for a sample .bashrc file.

**logout file**

$HOME/.bash_logout

user-specific instruction file, found in each user's home directory. Upon exit from a login (Bash) shell, the commands in this file execute.

**data files**

/etc/passwd

A listing of all the user accounts on the system, their identities, their home directories, the groups they belong to, and their default shell. Note that the user passwords are _not_ stored in this file, [^2] but in /etc/shadow in encrypted form.

**system configuration files**

/etc/sysconfig/hwconf

Listing and description of attached hardware devices. This information is in text form and can be extracted and parsed.

```bash
bash$ grep -A 5 AUDIO /etc/sysconfig/hwconf	      
class: AUDIO
 bus: PCI
 detached: 0
 driver: snd-intel8x0
 desc: "Intel Corporation 82801CA/CAM AC'97 Audio Controller"
 vendorId: 8086
 
```

> [!note]
> This file is present on Red Hat and Fedora Core installations, but may be missing from other distros.

[^1]: This does not apply to **csh**, **tcsh**, and other shells not related to or descended from the classic Bourne shell (**sh**).

[^2]: In older versions of UNIX, passwords _were_ stored in /etc/passwd, and that explains the name of the file.
