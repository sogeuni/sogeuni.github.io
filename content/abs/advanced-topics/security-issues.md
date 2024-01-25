---
title: 36.8. Security Issues
---


## 36.8.1. Infected Shell Scripts

A brief warning about script security is indicated. A shell script may contain a _worm_, _trojan_, or even a _virus_. For that reason, never run as _root_ a script (or permit it to be inserted into the system startup scripts in /etc/rc.d) unless you have obtained said script from a trusted source or you have carefully analyzed it to make certain it does nothing harmful.

Various researchers at Bell Labs and other sites, including M. Douglas McIlroy, Tom Duff, and Fred Cohen have investigated the implications of shell script viruses. They conclude that it is all too easy for even a novice, a "script kiddie," to write one. [^1]

Here is yet another reason to learn scripting. Being able to look at and understand scripts may protect your system from being compromised by a rogue script.

## 36.8.2. Hiding Shell Script Source

For security purposes, it may be necessary to render a script unreadable. If only there were a utility to create a stripped binary executable from a script. Francisco Rosales' [shc -- generic shell script compiler](http://www.datsi.fi.upm.es/~frosal/sources/) does exactly that.

Unfortunately, according to [an article](http://www.linuxjournal.com/article/8256) in the October, 2005 _Linux Journal_, the binary can, in at least some cases, be decrypted to recover the original script source. Still, this could be a useful method of keeping scripts secure from all but the most skilled hackers.

## 36.8.3. Writing Secure Shell Scripts

_Dan Stromberg_ suggests the following guidelines for writing (relatively) secure shell scripts.

- Don't put secret data in [[othertypesv#^ENVREF|environment variables]].
- Don't pass secret data in an external command's arguments (pass them in via a [[special-characters#^PIPEREF|pipe]] or [[io-redirection|redirection]] instead).
- Set your [[another-look-at-variables#^PATHREF|$PATH]] carefully. Don't just trust whatever path you inherit from the caller if your script is running as _root_. In fact, whenever you use an environment variable inherited from the caller, think about what could happen if the caller put something misleading in the variable, e.g., if the caller set [[another-look-at-variables#^HOMEDIRREF|$HOME]] to /etc.

[^1]: See Marius van Oers' article, [Unix Shell Scripting Malware](http://www.virusbtn.com/magazine/archives/200204/malshell.xml), and also the [[bibliography.md#^DENNINGREF|_Denning_ reference]] in the _bibliography_.
