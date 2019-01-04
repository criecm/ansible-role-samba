criecm.samba
=========

Install / configure samba standalone or in a NT4-style domain
(PDC,BDC or MEMBER)

Requirements
------------
ldap nss client installed (eg: `criecm.ldap_client`)

Role Variables
--------------

read: `variable_name` (default value) details

## Mandatory
* `smb_domain` () NT4 domain name
* `smb_join_user` () User needed to join domain
* `smb_join_passwd` () ... and his password

### shares
* `shares` ([])
  list of shares dict with:
  * `name` ('') MANDATORY
  name of share
  * `cifs` (False) MANDATORY
  share need this to be True to be defined
  * `smbparams` ({})
  dict of smb.conf additional parameters for the share
  * `path` ('') MANDATORY
  share's directory

## LDAP ((P)DC-only)
### Mandatory
* `smb_ldap_uri` ()
* `smb_ldap_suffix` ()
* `smb_ldap_admindn` () For samba use and add machine users
* `smb_ldap_adminpw` ()

### optional
* `upgrade (False)` do upgrade samba packages
* `smb_global_params` ({}) dict of samba parameters
```
smb_global_params:
  unix charset: 'utf8'
  map archive: 'No'
```
* `smb_ldap_readdn` (`smb_ldap_admindn`) read-only ldap user
* `smb_ldap_readpw` (`smb_ldap_adminpw`) and his passwd
* `smb_ldapr_uri` (`smb_ldap_uri`) LDAP replica if any
* `smb_ldap_user_suffix` ('ou=People')
* `smb_ldap_group_suffix` ('ou=Group')
* `smb_ldap_machine_suffix` ('ou=Machines')
* `smb_ldap_idmap_suffix` ('')
* `smb_ldap_scope` (sub)
* `x509_ca_file` () Used for ldaps x509 checks
* `x509_ldap_client_cert` ()
* `x509_ldap_client_key` ()

### optional - only for PDC (smbldap.conf.j2)
* `smb_ldap_pwdhash` (SSHA)
* `smb_crypt_salt_format` ('')
* `smbldap_loginshell` ('/bin/bash')
* `smbldap_userhome` ('/home/%U')
* `smbldap_defuser_gid` ('513')
* `smbldap_computer_gid` ('515')
* `smbldap_skeldir` ('/etc/skel')
* `smbldap_user_gecos` ('System User')

Dependencies
------------
* `criecm.ldap_client`

Example Playbook
----------------

    # BDC
    - hosts: bdc
      roles:
        - { role: criecm.samba }
      vars:
        shares:
          - name: "netlogon"
            path: "/shares/netlogon"
            cifs: True
            smbparms:
              comment: "Netlogon service"
              root preexec: "/my/script/mknetlogon %U %G %I"
        smb_domain: "MYDOM"
        smb_join_user: "automachines"
        smb_join_passwd: "his pass"
        smb_ldap_uri: "ldaps://ldap.my.domain"
        smb_ldap_suffix: "dc=organisation,dc=land"
        smb_ldap_admindn: "cn=admin,dc=organisation,dc=land
        smb_ldap_adminpw: "thisIsSecret"
    
    # member servers
    - hosts: servers
      roles:
        - { role: criecm.samba }
      vars:
        shares:
          - name: "myshare"
            path: "/shares/t"
            cifs: True
            smbparms:
              guest ok: "no"
              valid users: "me,him,her,us"
          - name: Profiles
            path: /shares/p
            cifs: True
            smbparms:
              browseable: "No"
              csc policy: "disable"
              root preexec: "'/bin/sh /root/mkprofile.sh %u %g'"
              profile acls: "Yes"
              read only: "No"
        smb_domain: "MYDOM"
        smb_join_user: "automachines"
        smb_join_passwd: "his pass"

License
-------

BSD

Author Information
------------------

https://github.com/criecm
