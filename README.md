# 389ds-on-kubernetes

### デプロイ手順

以下の手順に従ってデプロイを行う。

#### namespaceの作成
```bash
$ alias k=kubectl
$ k create ns 389ds
```

#### Serviceを作成
```bash
$ k apply -f svc-internal.yaml -n 389ds
$ k apply -f svc-external.yaml -n 389ds
$ k get svc -n 389ds
```

#### Directory Managerのパスワード作成(管理者アカウント的な)
```bash
k create secret generic dirsrv-dm-password --from-literal=dm-password='Secret123' -n 389ds
```

#### 389dsのデプロイ
```bash
k apply -f dirsrv-statefulset.yaml -n 389ds
k get po -n 389ds
```
### 動作確認
現時点だとldapsearchかけてもエントリー情報がないのでバックエンドとsuffixを作成。
```bash
$ k exec -it dirsrv-0 -n 389ds -- dsconf localhost backend create --suffix dc=example,dc=com --be-name userroot --create-suffix --create-entries

```

今回設定したコンテナイメージにはldap関連コマンドがデフォルトで入っていないため、ldap-clientを起動し、コンテナ内で`ldapsearch`コマンドを実施。
```bash
$ k run ldap-client -it --image=quay.io/389ds/clients:latest -n 389ds
```

```bash
[root@ldap-client /]# ldapsearch -xLLL -H ldap://dirsrv-internal-svc.389ds.svc.cluster.local:3389 -D "cn=Directory Manager" -w Secret123
dn: dc=example,dc=com
objectClass: top
objectClass: domain
dc: example
description: dc=example,dc=com

dn: ou=groups,dc=example,dc=com
objectClass: top
objectClass: organizationalunit
ou: groups

dn: ou=people,dc=example,dc=com
objectClass: top
objectClass: organizationalunit
ou: people

dn: ou=permissions,dc=example,dc=com
objectClass: top
objectClass: organizationalunit
ou: permissions

dn: ou=services,dc=example,dc=com
objectClass: top
objectClass: organizationalunit
ou: services

dn: uid=demo_user,ou=people,dc=example,dc=com
objectClass: top
objectClass: nsPerson
objectClass: nsAccount
objectClass: nsOrgPerson
objectClass: posixAccount
uid: demo_user
cn: Demo User
displayName: Demo User
legalName: Demo User Name
uidNumber: 99998
gidNumber: 99998
homeDirectory: /var/empty
loginShell: /bin/false

dn: cn=demo_group,ou=Groups,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: posixGroup
objectClass: nsMemberOf
cn: demo_group
gidNumber: 99999

dn: cn=group_admin,ou=permissions,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: nsMemberOf
cn: group_admin

dn: cn=group_modify,ou=permissions,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: nsMemberOf
cn: group_modify

dn: cn=user_admin,ou=permissions,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: nsMemberOf
cn: user_admin

dn: cn=user_modify,ou=permissions,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: nsMemberOf
cn: user_modify

dn: cn=user_passwd_reset,ou=permissions,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: nsMemberOf
cn: user_passwd_reset

dn: cn=user_private_read,ou=permissions,dc=example,dc=com
objectClass: top
objectClass: groupOfNames
objectClass: nsMemberOf
cn: user_private_read
```
エントリー情報の取得確認完了。
