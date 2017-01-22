# LDAP Schema for EndFirst

This is the LDAP schema used by EndFirst to provide LDAP authentication and directory services to our suite of applications. In order to ensure interoperability with other objectClasses such as inetOrgPerson, account, etc., we have not defined attributes that exist in other standard schemas. The upside, of course, is interoperability. The downside is that the endfirst schema is dependent upon attributes found in the standard cosine schema, and will not work without this being installed first.

## IANA Private Enterprise Number

EndFirst LLC has registered the [OID 1.3.6.1.4.1.47049](http://oid-info.com/get/1.3.6.1.4.1.47049) with IANA for its LDAP schema. We have defined our schema in accordance with the recommendations found in the [OpenLDAP Admin guide](http://www.openldap.org/doc/admin24/schema.html)

- **1.3.6.1.4.1.47049.1**: LDAP objects
- **1.3.6.1.4.1.47049.1.1.x**: LDAP attributes
- **1.3.6.1.4.1.47049.1.2.x**: LDAP object classes 

### fileQuota Attribute

fileQuota maintains the user quota for the file server

- **OID**: 1.3.6.1.4.1.47049.1.1.1
- **DataType**: IA5String
- **Size**: 50, single-value

### mailQuota Attribute

mailQuota maintains the user quota for the mail server

- **OID**: 1.3.6.1.4.1.47049.1.1.2
- **DataType**: IA5String
- **Size**: 50, single-value

### proxy Attribute

proxy field is used by Dovecot to determing how to proxy access to mail for the user. This is used together with the host field (part of the built-in cosine schema) to handle all proxying aspects.

- **OID**: 1.3.6.1.4.1.47049.1.1.3
- **DataType**: IA5String
- **Size**: 50, single-value

### mailFolder Attribute

mailFolder is an override to the default user mail folder. This can be used when migrating users to different file shares, etc.

- **OID**: 1.3.6.1.4.1.47049.1.1.4
- **DataType**: IA5String
- **Size**: 255, single-value

### dbUserID Attribute

This is the userid assigned in the Postgres database for the user. Having this here helps maintain integrity between different databases

- **OID**: 1.3.6.1.4.1.47049.1.5
- **DataType**: Integer, single-value

### endfirstAccount objectClass

This objectClass grants users login access, but grants no access to endfirst services.

- **OID**: 1.3.6.1.4.1.47049.1.2.1
- **MUST**: uid, dbUserId, cn, userPassword

### endFirstSubscriber objectClass

Adds additional fields for users who are subscribed to the email, calendar and file-sharing service

NOTE: endFirstSubscriber relies on the standard cosine schema that ships with OpenLDAP. In order to use the endfirstSubscriber objectClass, this schema must be installed first.

- **OID**: 1.3.6.1.4.1.47049.1.2.2
- **MAY**: fileQuota, mailQuota
- **MUST**: host, proxy, mailFolder, gidNumber, uidNumber

## Usage

Verified accounts are given the endfirstAccount objectClass which grants them access to the system. 

Subscribers are also given the endfirstSubscriber objectClass. These additional attributes make it possible to administer these services on their account.

## Installation

**NOTE:** The endfirst schema relies on the standard cosine schema that ships with OpenLDAP. In order to use the endfirstSubscriber objectClass, this schema must be installed first.

To install the schema in OpenLDAP using OLC (cn=config), use the `ldapadd` command:

    root# ldapadd -Y EXTERNAL -H ldapi:/// -f endfirst.ldif
   
And verify that the schema is correctly loaded:

    root# ldapsearch -H ldapi:// -Y EXTERNAL -LLL -b cn=config "(cn={*}endfirst)"
    ...
    # {5}endfirst, schema, config
    dn: cn={5}endfirst,cn=schema,cn=config
    objectClass: olcSchemaConfig
    cn: {5}endfirst
    olcAttributeTypes: {0}( 1.3.6.1.4.1.47049.1.1.1 NAME 'userHash' DESC 'EndFirst
     User Hash' EQUALITY caseExactMatch SINGLE-VALUE 
     SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 )
    olcObjectClasses: {0}( 1.3.6.1.4.1.47049.1.2.1 NAME 'endfirst' DESC 'endfirst
      LDAP Schema' AUXILIARY MUST ( userHash ) )

If your LDAP server does not use OLC (cn=config), then add the schema `endfirst.schema` in the schema directory, and update your configuration accordingly.


## Example

    root# ldapsearch -H ldapi:// -Y EXTERNAL -LLL -b ou=Users,dc=endfirstlocal "(objectclass=endfirst)" 
    ...
    objectClass: endfirstUser
    objectClass: endfirst
    dn: uid=bogus@endfirst.com,ou=Users,dc=endfirstlocal
    uid: bogus@endfirst.com
    cn: Bogus User
    dbUserId: 1234
    userPassword: secret-SHA512-Hash-iterated-to-make-hard-to-crack

