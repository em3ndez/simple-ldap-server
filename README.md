# Simple LDAP Auth Server

[![MIT License](https://img.shields.io/apm/l/atomic-design-ui.svg?)](https://github.com/tterb/atomic-design-ui/blob/master/LICENSEs)
[![Version](https://badge.fury.io/gh/tterb%2FHyde.svg)](https://badge.fury.io/gh/tterb%2FHyde)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/Upekshe/simple-ldap-server/graphs/commit-activity)


Simple LDAP/LDAPS protocol (authentication and search) emulating user-store for **test environments**

```markdown
Dockerized Releases: https://hub.docker.com/r/upekshejay/simple-ldap-test-server
```

## Introduction

This project provides an easily configurable **`TEST ENV ONLY`** LDAP/LDAPS server designed to **mock an existing setup** for authentication and user store testing. It enables developers to simulate LDAP (or LDAPS) authentication mechanisms and directory structures during application development and testing.  

While the LDAP protocol supports various authentication mechanisms, this server focuses on two primary methods:  
- **Anonymous Authentication**  
- **Simple Authentication** (clear-text)  

### Purpose

When developing new systems, software solutions often need to integrate with an organization’s existing authentication infrastructure. Many organizations rely on Active Directory or OpenLDAP for authentication, and the standard integration approach involves the LDAP/LDAPS protocols.  

During development and testing stages—such as initial testing or User Acceptance Testing (UAT)—organizations typically prohibit direct access to their production LDAP services. To address this, a testing LDAP server is needed to:  
1. Simulate the LDAP protocol.  
2. Mimic the organizational unit (OU) structure of the target environment.  

By using this mock LDAP server, developers can replicate the behavior of an organization’s directory service, making it easier to test authentication mechanisms in isolation. On production deployment, the only required change is updating the LDAP server address to the actual service endpoint.  

### About This Project

This server, built using **LDAPJS**, is specifically designed to mock existing LDAP setups for development and testing purposes. It provides a lightweight and configurable solution for simulating directory services, ensuring seamless integration and robust testing.  

> **Note:** This project is strictly intended for **testing purposes** and should never be used in production environments.  

## How to use

Simply start the server to proceed, you can choose one of the following methods:  

### Using Node.js  

1. Navigate to the `simple-ldap-server` folder.  
2. Run:  
   ```bash
   npm start

### Using docker  

run `docker run -p 389:389 -p 636:636 --name simple-ldap-server upekshejay/simple-ldap-test-server`
    

Once started the server should prompt something similar. (This output is based on the default values included with the `/etc/store.json`)

```shell
[2022-02-19T12:08:12.973] [INFO] default - Added DC=mtr,DC=com
[2022-02-19T12:08:12.984] [INFO] default - Added OU=users,DC=mtr,DC=com
[2022-02-19T12:08:12.990] [INFO] default - Added CN=admin,OU=users,DC=mtr,DC=com
...
[2022-02-19T12:08:13.090] [INFO] default - LDAP server up at: ldaps://0.0.0.0:636
[2022-02-19T12:08:13.091] [INFO] default - LDAP Service initiation complete
```

Now the ldap server has exposeed its services.

Then check everything works as intended, lets use the tool `ldapserach`, execute following command in a shell

```shell
ldapsearch -x -H ldap://127.0.0.1:389 -b "CN=nimal,OU=users,DC=mtr,DC=com" -D "CN=admin,OU=users,DC=mtr,DC=com" -W
```
The above command search the ldap server for the user 'nimal' with the credentials of the 'admin' user. When this is executed it will ask for the password of admin user, enter the password "itachi". Now ldapsearch should print a prompt similar to below

```shell
# extended LDIF
#
# LDAPv3
# base <CN=nimal,OU=users,DC=mtr,DC=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# nimal, users, mtr.com
dn: cn=nimal,ou=users,dc=mtr,dc=com
sAMAccountName: nimal
uid: nimal
userprincipalname: nimal
mailnickname: nimal
groups: lime_users|IT
cn: nimal
objectClass: User

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

This indicates that server has responded with the search results including one user that matches the provided criteria

## Configurations

### System configurations

System level configurations are maintained inside the config folder

# JSON Configuration Field Descriptions  

## User Store  

| Field | Description |  
|-------|-------------|  
| **user-store** | Defines the configuration for loading users into the LDAP service. |  
| **user-store.mode** | Specifies the current running mode for the user store. Example: `user-file-store`. |  
| **user-store.modes** | Lists all the supported modes for the user store. |  
| **user-store.modes.user-file-store** | Defines the configuration for the `user-file-store` mode. This mode supports loading users from a JSON file to a single user group. |  
| **user-store.modes.user-file-store.location** | The file path for the JSON file containing user definitions. |  
| **user-store.modes.user-file-store.users-array-name** | The property name in the JSON file that holds the array of user objects. |  
| **user-store.modes.user-file-store.common-password** | A default password assigned to all users unless explicitly defined. |  
| **user-store.modes.user-file-store.user-group-ou** | The organizational unit (OU) that all users are linked to. Example: `ou=users`. |  
| **user-store.modes.user-file-store.root-entry** | The root entry for the directory structure, represented in Distinguished Name (DN) format. Example: `dc=mtr,dc=com`. |  
| **user-store.modes.user-file-store.attribute-mapping** | Defines how attributes in LDAP user records map to values in user objects or static values. |  
| **user-store.modes.user-file-store.optional** | Contains optional configurations for the user store. |  
| **user-store.modes.user-file-store.optional.search-permitted-users** | Lists additional users allowed to perform search operations. |  
| **user-store.modes.user-file-store.optional.search-permitted-users.dn** | The full DN of a user with search permissions. |  
| **user-store.modes.user-file-store.optional.search-permitted-users.password** | The password for a user with search permissions. |  
| **user-store.modes.structure-file-store** | Reserved for future use. This mode is not yet supported. |  

### Attribute Mapping  

| Field | Description |  
|-------|-------------|  
| **sAMAccountName** | Maps to the `id` field of the user object. |  
| **uid** | Maps to the `id` field of the user object. |  
| **userprincipalname** | Maps to the `id` field of the user object. |  
| **mailnickname** | Maps to the `id` field of the user object. |  
| **groups** | Specifies static group membership. Example: `lime_users|IT`. |  
| **cn** | Maps to the `id` field of the user object. |  
| **password** | Maps to the `credential` field of the user object. If null, the `common-password` is used. |  
| **objectClass** | Specifies a static value for the LDAP object class. Example: `User`. |  
| **permissions** | Maps to the `permissions` field of the user object. |  

---

## Anonymous Bind  

| Field | Description |  
|-------|-------------|  
| **anonymous-bind** | Configures whether anonymous binding to the LDAP server is enabled. |  
| **anonymous-bind.enabled** | A boolean value indicating whether anonymous binding is allowed. |  

---

## Protocols  

### LDAP Protocol  

| Field | Description |  
|-------|-------------|  
| **protocols.ldap** | Configuration for the LDAP protocol. |  
| **protocols.ldap.enabled** | A boolean value indicating whether the LDAP protocol is enabled. |  
| **protocols.ldap.port** | The port number used for the LDAP protocol. Default: `389`. |  

### LDAPS Protocol  

| Field | Description |  
|-------|-------------|  
| **protocols.ldaps** | Configuration for the LDAPS protocol. |  
| **protocols.ldaps.enabled** | A boolean value indicating whether the LDAPS protocol is enabled. |  
| **protocols.ldaps.port** | The port number used for the LDAPS protocol. Default: `636`. |  
| **protocols.ldaps.key-location** | The file path to the private key used for SSL/TLS. |  
| **protocols.ldaps.cert-location** | The file path to the certificate used for SSL/TLS. |  


``sample configuration and description``

```jsonc
{
    "user-store": { /* defines the way to load users to the ldap service */
        "mode": "user-file-store", /* current running mode */
        "modes": {
            /* 
            all the modes that are defines goes in here. One of these defined methods should 
            be used in 'user-store.mode'
            modes: 
                "user-file-store"
                "structure-file-store" NOT SUPPORTED YET
                "perf-store"           NOT SUPPORTED YET    
            */
            "user-file-store": {
                /* 
                defines the user file store, 
                which supports loading users from a file to a one specific user group
                */
                "location": "/app/etc/store.json", /* location of the file. File should be a json*/
                "users-array-name": "users", /* property name of the user array*/
                "common-password": "itachi", /* password for all the users */
                "user-group-ou": "ou=users", /* User group that all the defined users are linked to*/
                "root-entry": "dc=mtr,dc=com", /* DC of the structure */
                "attribute-mapping": {
                    /*
                        Attribute ,apping goes inside here.
                        Key is the id of the attribute in the LDAP user record, 
                        value is the mapping that should be used to retrieve the actual value (from the Object or direct)
                            Notation
                            If User object { "id": "sakura", "credentials": "asdf" }
                            and if we want to populate "uid" attribute from the "id" field of the object

                            ``
                            "uid": "obj.id"
                            ``

                            If want to populate "objectClass" from a static value "User"
                            ```
                            "objectClass": "User"
                            ```
                    */
                    "sAMAccountName": "obj.id",
                    "uid": "obj.id",
                    "userprincipalname": "obj.id",
                    "mailnickname": "obj.id",
                    "groups": "lime_users|IT",
                    "cn": "obj.id",
                    "password": "obj.credential", /* if this is null common passowrd will be used*/
                    "objectClass": "User",
                    "permissions": "obj.permissions"
                },
                "optional": {
                    /*
                        configurations inside this section are optional and it is not necessary to fill these
                    */
                    "search-permitted-users": [
                        /*
                            allow adding additional users with search permission
                        */
                        {
                            "dn": "cn=search_user,ou=users,dc=mtr,dc=com", /* complete DN of the user*/
                            "password": "sasuke" /* password for this specific user */
                        }
                    ]
                }
            },
            "structure-file-store": {}
        }
    },
    "anonymous-bind": {
        "enabled": true
    },
    "protocols": {
        "ldap": {
            "enabled": true,
            "port": 389
        },
        "ldaps": {
            "enabled": true,
            "port": 636,
            "key-location": "/app/cert/key.pem",
            "cert-location": "/app/cert/cert.pem"
        }
    }
}
```

### Store structure

There are multiple types of store structures each store has diffrent one

#### User store (user-file-store)

``sample configuration and description``

```jsonc
{
    "users": [
        /* 
        array that contains all the users
            simplest user entry should have a "id", for create an user with permissions
            add the property "permissions"
                currently allowed additional permissions are "search"
            an user entry can have any field and you can use that in the mapping inside the default json
        */
        {
            "id":"admin", /* id of the user */
            "permissions": ["search"]  /* permissions that are granted for the user */
            },
        {"id":"kamal"},
        {"id":"nimal"},
        {"id":"anil"},
        {"id":"supun"},
        {"id":"dasun"}
    ]
}
```

## How to use the Dockerized version

Simply start the contianer by running ``docker run``. Container will start with the preconfigured configurations. 

To run with a different configuration just run the container with mounted locations
