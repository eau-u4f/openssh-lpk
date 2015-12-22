## Description ##

The lpk patch allows you to lookup ssh public keys over LDAP helping central authentication of multiple servers. This patch is an alternative to other authentication system working in a similar way (Kerberos, SecurID, etc...), except the fact that it's based on OpenSSH and its public key code.

We are currently working on phasing out the old code and providing an abstraction layer for publick key lookup that would move LDAP code out of OpenSSH and provide a Nice and Clean abstraction layer that allows any kind of plugins to look up the key.

The original openssh-lpk project (once hosted at the OpenDarwin project) was originally created by [Eric Auge](mailto:eric.auge+NOSPAM+@gmail.com)
was maintained by [Eric Auge](mailto:eric.auge+NOSPAM+@gmail.com) and [Andrea Barisani](mailto:andrea+NOSPAM+@inversepath.com).

InversePath is now leaving from the development of the LDAP Public Key. The project is
now maintained by its original creator [Eric Auge](mailto:eric.auge+NOSPAM+@gmail.com).

## Documentation ##

What do you need:

- obviously a patched OpenSSH :)
- an accessible LDAP server somewhere on the network with the provided schema (OpenLDAP schema | Sun schema). Please note that lpk does not support LDAP over SSL. To use lpk you must either use standard ldap (not recommended) or LDAP + TLS. ldaps:// URLs will not work.
- a 'posixAccount' user entry with 'ldapPublicKey' objectclass and 'sshPublicKey' attribute

Multiple 'sshPublicKey' in a user entry are allowed, as well as multiple 'memberUid' attributes in a group entry.

Example:

```
dn: uid=foo,ou=users,dc=example,dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: posixAccount
objectclass: ldapPublicKey
description: John Doe Account
userPassword: {crypt}0LXhFAsrBWEEQ
cn: John Doe
sn: John Doe
uid: foo
uidNumber: 1034
gidNumber: 100
homeDirectory: /home/foo
sshPublicKey: ssh-dss AAAAB3...
sshPublicKey: ssh-dss AAAAM5...
```

Additionally Group entries can be defined:

- attached to the 'posixGroup' objectclass
- with a 'cn' groupname attribute
- with multiple 'memberUid' attributes filled with usernames allowed in this group

Example:

```
dn: cn=unix,ou=groups,dc=example,dc=com
objectclass: top
objectclass: posixGroup
description: Unix group
cn: unix
gidNumber: 1002
memberUid: foo
memberUid: bar
```

The patch applies to OpenSSH code, this is an example on how to apply it:

cd openssh-4.6p1
patch -Np1 -i /tmp/openssh-lpk-4.6p1-0.3.9.patch
./configure <your usual options ...> --with-ldap
make

With lpk code enabled sshd adds LDAP public key lookup to its authentication methods. A ldapsearch is performed to get the public key directly from the LDAP instead of reading it from the filesystem (usually in $HOME/.ssh/authorized\_keys).

If groups are enabled, it will also check if the user that wants to login is in the group of the server he is trying to log into. If it fails, it falls back to the other enabled authentication methods.

Several additional tokens are added to sshd\_config:

```
UseLPK yes
LpkServers         ldap://10.1.7.1 ldap://10.1.7.2
LpkUserDN          ou=users,dc=example,dc=com
LpkGroupDN         ou=groups,dc=example,dc=com
LpkBindDN          cn=Manager,dc=example,dc=com
LpkBindPw          somepasswordifneeded
LpkServerGroup     somegroupname
LpkForceTLS        yes
LpkSearchTimelimit 3
LpkBindTimelimit   3
```


but we recommend using existing ldap.conf syntax from nss\_ldap and pam\_ldap with the following configuration:

```
UseLPK yes
LpkLdapConf /etc/ldap.conf
```

Explaining how to configure/manage your LDAP directory is beyond our scope, but you can check the following presentation for a full overview on how to sensibly centralize users/accounts and much more with LDAP:

[FOSDEM 2006 LDAP presentation](http://dev.inversepath.com/openssh-lpk/ldap_fosdem_2006.pdf) (PDF file)

## Requirements ##

The latest patch applies to [OpenSSH](http://openssh.org/) version 4.6.

The latest contributed patch applies to [OpenSSH](http://openssh.org/) version 5.1p1

The [OpenLDAP](http://openldap.org/) client library.