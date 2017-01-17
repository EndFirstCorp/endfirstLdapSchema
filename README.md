# LDAP Schema for EndFirst

## IANA Private Enterprise Number

EndFirst LLC has registered the [OID 1.3.6.1.4.1.47049](http://oid-info.com/get/1.3.6.1.4.1.47049) with IANA for its LDAP schema. We have defined our schema in accordance with the recommendations found in the [OpenLDAP Admin guide](http://www.openldap.org/doc/admin24/schema.html)

- **1.3.6.1.4.1.47049.1**: LDAP objects
- **1.3.6.1.4.1.47049.1.1.x**: LDAP attributes
- **1.3.6.1.4.1.47049.1.2.x**: LDAP object classes 

### userHash Field

endfirst userHash is used to maintain referential integrity between LDAP,
Postgres and Redis databases. It is a surrogate key for the Postgres UserId

- **OID**: 1.3.6.1.4.1.47049.1.1.1
- **AttributeType**: userHash

### EndFirst objectClass

- **OID**: 1.3.6.1.4.1.47049.1.2.1
- **ObjectClass**: endfirst

## Usage

A user can be extended with the auxillary `objectClass: endfirst` and the attribute `userHash` will then be required in order to enable referencing across databases.

## Installation

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
    objectClass: account
    objectClass: endfirst
    dn: uid=bogus@endfirst.com,ou=Users,dc=endfirstlocal
    uid: bogus@endfirst.com
    userHash: VAflWnGDt4VIg_0jNpQV7Z0PIEi79vNPcbpAZUO-6AM=

