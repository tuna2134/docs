# メールサーバ構築

## 1.1. LDAPをインストール
```sh
apt-get update
apt-get install -y slapd ldap-utils
```

## 1.2. LDAPの設定をいじる
```
sudo dpkg-reconfigure slapd
```

## 1.3. ユーザーの追加のconfig
`add_content.ldif`:
```
dn: ou=People,dc=example,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=example,dc=com
objectClass: organizationalUnit
ou: Groups

dn: uid=noreply,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: shadowAccount
sn: noreply
givenName: noreply
uid: noreply
cn: Noreply System
displayName: Noreply
userPassword: {CRYPT}x
mail: noreply@example.com
```

## 1.4. ユーザーを追加
```
ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_content.ldif
```

## 2.1 Postfixのインストール
```sh
apt-get install -y postfix postfix-ldap
```

## 2.2. postfixの設定
`main.cf`:
```
compatibility_level = 3.6

myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain

# See /usr/share/postfix/main.cf.dist for a commented, more complete version

smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

mydestination = $myhostname, localhost.$mydomain, localhost
relayhost = 
mynetworks = 0.0.0.0/0
recipient_delimiter = +

home_mailbox = Maildir/

virtual_alias_maps = ldap:/etc/postfix/ldap_alias.cf
virtual_mailbox_maps = ldap:/etc/postfix/ldap_mailbox.cf
virtual_mailbox_domains = example.com
virtual_mailbox_base = /home/vmail

virtual_mailbox_limit = 512000000
virtual_minimum_uid = 5000
virtual_transport = virtual
virtual_uid_maps = static:5000
virtual_gid_maps = static:5000
local_transport = virtual
local_recipient_maps = $virtual_mailbox_maps

smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
smtpd_sasl_security_options = noanonymous
smtpd_sasl_tls_security_options = $smtpd_sasl_security_options
smtpd_tls_security_level = may
smtpd_tls_auth_only = yes
smtpd_tls_received_header = yes
smtpd_sasl_local_domain = $mydomain
broken_sasl_auth_clients = yes

smtp_tls_security_level = may
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_tls_loglevel = 1
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem

smtpd_milters = inet:127.0.0.1:8891
non_smtpd_milters = $smtpd_milters
milter_default_action = accept
```

## 2.3. PostfixのLDAP連携
`{ADMIN_PASSWORD}` - LDAPのadminパスワード

`ldap_alias.cf`:
```
server_host = ldap://localhost:389
search_base = ou=people,dc=example,dc=com
bind = yes
bind_dn = cn=admin,dc=example,dc=com
bind_pw = {ADMIN_PASSWORD}
scope = one
query_filter = (&(objectClass=inetOrgPerson)(mail=%s))
result_attribute = mail
version = 3
```

`ldap_mailbox.ldif`:
```
server_host = ldap://localhost:389
search_base = ou=people,dc=example,dc=com
bind = yes
bind_dn = cn=admin,dc=example,dc=com
bind_pw = {ADMIN_PASSWORD}
scope = one
query_filter = (&(objectClass=inetOrgPerson)(mail=%s))
result_attribute = mail
result_format = %u/Maildir/
version = 3
```

## 3.1. dovecotのインストール
```
apt-get install -y dovecot dovecot-ldap
```

## 3.2. LDAPの接続設定
`dovecot-ldap.conf.ext`:
```
uris = ldap://localhost:389
dn = cn=admin,dc=example,dc=com

dnpass = ADMIN_PASSWORD

auth_bind = no

ldap_version = 3

base = ou=people,dc=example,dc=com

user_attrs = mail=user,userPassword=password
user_filter = (&(objectClass=inetOrgPerson)(mail=%u))

pass_attrs = mail=user,userPassword=password
pass_filter = (&(objectClass=inetOrgPerson)(mail=%u))

iterate_attrs = mail=user
iterate_filter = (objectClass=inetOrgPerson)

default_pass_scheme = MD5
```

## 3.3. mailの設定
`10-mail.conf`:
```
mail_home = /home/vmail/%n
mail_location = maildir:~/Maildir

namespace inbox {
  inbox = yes
}

mail_uid = vmail
mail_gid = vmail

mail_privileged_group = mail
```

## 3.4. masterの設定
`10-master.conf`:
```
service imap-login {
  inet_listener imap {
    port = 0
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }
}

service pop3-login {
  inet_listener pop3 {
    port = 0
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}

service submission-login {
  inet_listener submission {
    #port = 587
  }
  inet_listener submissions {
    #port = 465
  }
}

service lmtp {
  unix_listener lmtp {
    #mode = 0666
  }

  #inet_listener lmtp {
    # Avoid making LMTP visible for the entire internet
    #address =
    #port = 
  #}
}

service imap {
  #process_limit = 1024
}

service pop3 {
  #process_limit = 1024
}

service submission {
  #process_limit = 1024
}

service auth {
  unix_listener auth-userdb {
    mode = 0666
    user = postの設定
    group = postfix
  }

  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }

  # Auth process is run as this user.
  #user = $default_internal_user
}

service auth-worker {
  #user = root
}

service dict {
  unix_listener dict {
    #mode = 0600
    #user = 
    #group = 
  }
}
```

## 3.5. LDAPの設定
`10-ldap.conf`:
```
# Authentication for LDAP users. Included from 10-auth.conf.
#
# <doc/wiki/AuthDatabase.LDAP.txt>

passdb {
  driver = ldap

  # Path for LDAP configuration file, see example-config/dovecot-ldap.conf.ext
  args = /etc/dovecot/dovecot-ldap.conf.ext
}

# "prefetch" user database means that the passdb already provided the
# needed information and there's no need to do a separate userdb lookup.
# <doc/wiki/UserDatabase.Prefetch.txt>
#userdb {
#  driver = prefetch
#}

userdb {
  driver = ldap
  args = /etc/dovecot/dovecot-ldap.conf.ext
  
  # Default fields can be used to specify defaults that LDAP may override
  default_fields = /home/vmail/%d/Maildir
}

# If you don't have any user-specific settings, you can avoid the userdb LDAP
# lookup by using userdb static instead of userdb ldap, for example:
# <doc/wiki/UserDatabase.Static.txt>
#userdb {
  #driver = static
  #args = uid=vmail gid=vmail home=/var/vmail/%u
#}
```