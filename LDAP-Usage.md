# LDAP Usage

* [Referenced website](https://newtoypia.blogspot.com/2016/11/ldap.html)

## 0 Setup

### 0.1 Install & configuration

* In Ubuntu 16.04

    ```bash
    apt-get install slapd ldap-utils
    ```

    ```bash
    dpkg-reconfigure slapd
    ```

* Install some packages

    * CRC32

        ```
        apt-get install libarchive-zip-perl
        ```

### 0.2 Example

* DNS domain name: `qhan.tw`
* Database backend to use: `MDB`
* Administrator password: `m13...`

## 1 Search

### 1.1 Simple search

* `-x` simple authentication.

* `-H` specifies the `slapd` server IP, the default port is 389 (e.g. `-H ldap://10.12.24.252:389/`).

* `-b` specifies the tree you want to search.

* Search all users in the domain `qhan.tw`.

    ```bash
    ldapsearch -x -b "dc=qhan,dc=tw"
    ```

    Result :

    ```
    # extended LDIF
    ...
    # qhan.tw
    dn: dc=qhan,dc=tw
    objectClass: top
    objectClass: dcObject
    objectClass: organization
    o: Qhan Corp
    dc: qhan
    
    # admin, qhan.tw
    dn: cn=admin,dc=qhan,dc=tw
    objectClass: simpleSecurityObject
    objectClass: organizationalRole
    cn: admin
    description: LDAP administrator
    
    # search result
    search: 2
    result: 0 Success
    
    # numResponses: 3
    # numEntries: 2
    ```

* Search a user named `admin` in the domain `qhan.tw`.

    ```bash
    ldapsearch -x -b "cn=admin,dc=qhan,dc=tw"
    ```

* To omit `-b`, you can write the base tree into `/etc/ldap/ldap.conf` (and we apply this to the rest of all commands).

    ```
    ...
    BASE	dc=qhan,dc=tw
    ```

### 1.2 Search with conditions

* Specifying `objectClass`.

    ```bash
    ldapsearch -x "objectClass=organizationalRole"
    ```

* Specifying `cn`.

    ```bash
    ldapsearch -x "cn=admin"
    ```

* Visit [this website](https://www.ibm.com/support/knowledgecenter/zh-tw/SSKTMJ_8.5.3/com.ibm.help.domino.admin85.doc/H_EXAMPLES_EXPORTING_THE_CONTENTS_OF_AN_LDAP_DIRECTORY_2804_STEPS.html) for more details about `ldapsearch`.

### 1.3 `slapcat`

* Using `slapcat` can check the whole thing with root access.

  ```
  sudo slapcat
  ```

## 2 Add & Delete Users

### 2.1 Add users

* The example file `users.ldif`.

    ```
    dn:    cn=user1,dc=qhan,dc=tw
    cn:    user1
    objectclass:    inetOrgPerson
    sn:    Lin
    givenName:    Liang Han
    
    dn:    cn=user2,dc=qhan,dc=tw
    cn:    user2
    objectclass:    inetOrgPerson
    sn:    Liang
    givenName:    Han
    ```

* `-D` specifies the commander (`dn`) (e.g. `cn=admin,dc=qhan,dc=tw`).

* `-W` prompt for bind password (e.g. `admin`'s password).

* ```bash
    ldapadd -x -D "cn=admin,dc=qhan,dc=tw" -W < users.ldif
    ```

    Result

    ```
    Enter LDAP Password: 
    adding new entry "cn=user1,dc=qhan,dc=tw"
    adding new entry "cn=user2,dc=qhan,dc=tw"
    ```

* `-w` will bind password input from command line, which is not recommanded.

* ```bash
    ldapadd -x -D "cn=admin,dc=qhan,dc=tw" -w "admin's password" < users.ldif
    ```

### 2.2 Delete users

* Specify `dc` to delete the user.

    ```bash
    ldapdelete -x -D "cn=admin,dc=qhan,dc=tw" -W "cn=user1,dc=qhan,dc=tw"
    ```

## 3 Passwords

### 3.1 Using password files

* Write the password into `admin.pwd`.

    ```bash
    echo -n "admin's password" > admin.pwd; chmod og-rwx admin.pwd
    ```

    And use it by `-y` flag.

    ```bash
    ldapadd -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd < users.ldif
    ```

### 3.2 change users passwords

#### 3.2.1 `ldappasswd`

* Same as 3.1, write `user1`'s password into `user1.pwd`.

    ```bash
    echo -n "user1's password" > user1.pwd; chmod og-rwx user1.pwd
    ```

    And change `user1`'s password by `ldappasswd` command and using `-T` flag to specify password file.

    ```bash
    ldappasswd -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd -T user1.pwd "cn=user1,dc=qhan,dc=tw"
    ```

* If only specify the commander (e.g. `admin`) without specifying target user (e.g. `user1`), it will change the commander's (e.g. `admin`) password.

* `-S` can replace `-T user1.pwd` and the system will ask you `user1`'s password with a prompt.

    ```bash
    ldappasswd -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd -S "cn=user1,dc=qhan,dc=tw"
    ```

* `-s` will bind `user1`'s password input from command line, which is not recommanded.

    ```bash
    ldappasswd -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd -s "user1's password" "cn=user1,dc=qhan,dc=tw"
    ```

#### 3.2.2 `ldapmodify`

* Write the changes of `user1` into `change_user1_password.ldif`.

    ```
    dn: cn=user1,dc=qhan,dc=tw
    changetype: modify
    replace: userPassword
    userPassword: "user1's password" or "user1's hashed password"
    ```

    * `"user1's password hashed password"` should be like this :

        ```
        {SSHA}onbhgl7e37bO/hjvtoyV8iitjk+Hm/1N
        ```

        And this line can be generated from this command :

        ```bash
        slappasswd -s "user1's password"
        ```

* Apply it.

    ```bash
    ldapmodify -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd < change_user1_password.ldif
    ```

### 3.3 Show passwords with base64 decoding

* After the commands in 3.2, lets show the results.

    ```bash
    ldapsearch -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd
    ```

    ```
    # extended LDIF
    ...
    # admin, qhan.tw
    dn: cn=admin,dc=qhan,dc=tw
    ...
    userPassword:: e1NTSEF9YWJyQVZRRmhweWNpbjY1VkRDZzhpc05pbml5elNUVlE=
    
    # user1, qhan.tw
    dn: cn=user1,dc=qhan,dc=tw
    ...
    userPassword:: e1NTSEF9b25iaGdsN2UzN2JPL2hqdnRveVY4aWl0amsrSG0vMU4=
    ...
    ```

* There are attributes `userPassword`, and are followed by two colons(`:`), which means the values are encoded to base64. So add this line into `~/.bashrc` :

    ```bash
    alias base64de='perl -MMIME::Base64 -MEncode=decode -n -00 -e '"'"'s/\n //g; s/(?<=:: )(\S+)/decode("UTF-8",decode_base64($1))/eg;print'"'"
    ```

    Show the result with base64 decoding again : 

    ```bash
    ldapsearch -x -D "cn=admin,dc=qhan,dc=tw" -y admin.pwd | base64de
    ```

    ```
    # extended LDIF
    ...
    # admin, qhan.tw
    dn: cn=admin,dc=qhan,dc=tw
    ...
    userPassword:: {SSHA}abrAVQFhpycin65VDCg8isNiniyzSTVQ
    
    # user1, qhan.tw
    dn: cn=user1,dc=qhan,dc=tw
    ...
    userPassword:: {SSHA}onbhgl7e37bO/hjvtoyV8iitjk+Hm/1N
    ...
    ```

### 3.4 Change `admin` priviledge password

* The passwords changed in [section 3](#3-Passwords) are written into the database (`/var/lib/ldap/data.mdb`)

* `admin` user has another password which is created when we reconfigure `slapd`

  ```
  dpkg-reconfigure slapd
  ```

* There is a way to change priviledge password without reconfiguring `slapd`

  1. Edit the config file `/etc/ldap/slapd.d/cn\=config/olcDatabase\=\{1\}mdb.ldif`, and replace the value of `olcRootPW` with the hashed new password.

     ```
     olcRootPW: {SSHA}9rWzRX1pTlju53gua106S3jXb7dp/yx0
     ```

  2. Create a backup (`backup.ldif`) of config file without first two lines.

     ```bash
     perl -ne 'print unless $.<=2' /etc/ldap/slapd.d/cn\=config/olcDatabase\=\{1\}mdb.ldif > backup.ldif
     ```

  3. Using CRC32 to calculate the checksum value of `backup.ldif`

     ```bash
     crc32 backup.ldif
     ```

     Result :

     ```
     494c62b2
     ```

     Overwrite the value of the second line of `/etc/ldap/slapd.d/cn\=config/olcDatabase\=\{1\}mdb.ldif`, after `# CRC32 `

     ```
     # CRC32 494c62b2
     ```

  4. Restart `slapd`

     ```bash
     service slapd restart
     ```

