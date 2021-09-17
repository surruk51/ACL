#### Access Control List system ####

### Introduction ###

This ACL aims to achieve maximum flexibility with minimum complexity.
It uses just two tables. The first defines entities and the second
defines relationships between them.

NOTE This class has been developed and tested with PHP 7.2

## Entities ## 

Entities can be anything that 

- needs access to controlled items or 
- is a controlled item

Entities can store an arbitrary amount of information, fields being
created and dropped as required. The only essential part of an entity
is its name, which must be unique.

## Relationships ##

Relationships can either be one of ownership or of permission.

Ownership:  Entity B is a member of Entity A, 
Permission:: Entity A can do something with entity B.

It is possible to build up hierarchies to arbitray levels. For example

- People can be members of sections that are members of departments that
  are members of operating companies who are members of holding companies
- sub-components can be members of products which can be members of
  product goups

## Permissions ##

Permissions automatically flow through the hierarchy. For example if a 
sales department was given 'canWrite' permission to a product group, All
members of that sales department would automatically have 'canWrite'
permission to all products within that product group.

The permissions themselves are entirely arbitrary. A permission is 
merely a string that a program can use at run time to see if it is
permitted to carry out the function related to that string. New 
permissions can be invented as required. (But note that programs will
have these strings hard coded in them, so changing them later is not 
practical.

### The code ###

The class is presented as 
- an ACL_interface which lists the public methods, 
- an abstract class which implements the interface and all but the 
  database specific methods and finally
- an sqlite class which extends the abstract ACL class to use an 
  SQLite table to stare the actual values

The database access methods are declared as abstract methods in the
abstract class. 

## Interfacing different storage engines ##

The database access methods have been designed to be as easy as possible 
to implement on different database systems (e.g. LDAP) but have only been
tested with SQLite.

In SQL terms only the following functionality is required
SELECT UPDATE DELETE INSERT WHERE IN() LIMIT, all of which could
conceivably be implemented even in flat files if necessary.

### Using the interface to manage access to funtionality. ###

1. Instantiate the class. e.g. $acl = new ACL\ACL_sqlite($path_to_file);
2. Set the user who will be granted access if you plan to use the 
   $acl->I() method.
3. At the beginning of each function where it is needed, use the I
   method or the isItTruethat() method. to check permission.
   
   3.1 Using the I() method.
 
       First set the user name with one of these methods. set_user()
       does not check anything but simply sets the user. 
       authenticate checks that the supplied password matches the 
       password stored in the entity (hashed) and on success sets the
       user. The username is stored as a session variable so that the
       user only has to be set once per session. Ther eis no 
       provision to save it across sessions.
       
       From this point on, every time the program needs to check a 
       permission, include at the head of the function
       if($acl->I($canDoto, $entity) {
          ...rest of function code...
       }
       
   3.2 Using the isItTrueThat() method.
 
       This method is used when one person is acting as a surrogate for
       another, since the username is specified in the call. There is
       no need to call set_user or authenticate before using this
       call.
       
       if(isItTrueThat($username, $canDoTo, $entity)) {
          ...rest of function code...
       }

### Managing the security database. ###

Without a way of managing the database, the above methods are useless.

Therefore the following functions are provided

# create entity #

Used more often than you might think. For example you might need to
provide an entity for every product if you needed fine grained access
to the product table, or every customer if you wanted customers to be 
able to access their own records. 

Each entity has a name, a description, a type, and an unlimited
number of additional attributes. The data for the entity is supplied
as an array of name/value pairs. The name must be unique and to ensure
this, you should implement a naming convention which separates entities
into manageable groups (e.g. products, customers, staff and so on). 
Duplicate names will be rejected.

As far as attributes are concerned you can create as many as you need.
The only attribute that has special treatment is 'password'. The 
password itself is not saved. Instead a hash of the password is
stored. The authenticate method verifies the supplied password against 
this hash. This means that you can give a password to any entity you
wish. However this provision is intended for user entities when the user
logs in.

# update entity #

Any part of an entity, apart from its name, can be updated. The name is
supplied to identify the entity to be updated and an array of name/value
pairs supplies the data.

# delete entity #

An entity can be deleted provided that there are no relationships 
attached to it. That is, it must be a childless orphan. Attempts to 
delete other entities will be rejected.

# find entities #

An array of name/value pairs is supplied, and any entities that match 
ALL of the supplied name/values is returned as a list of entities.

The value part can be an array of possible values, an in this case 
entities matching any one of the values will be selected.

# read entity #

Returns just the specified (by its unique name) entity

# find_entity_column #

Is like read_entity, but you can specify the column to be returned.

# read_ChildrenAll #

Returns a list of all the entities that are members of / children of /
owned by the specified entity.

In addition to the entity details, details of the relationship are also
included. These are the immediate parent of the entity and their
relationship to it.

### Partner Classes ###

The Security classes provide a series of methods that simplify the 
implementation of HTML presentation layer. For example, 

- giving or revoking an entity's membership of another entity,
- creating new entities
- giving or revoking an entity's permissions
- creating a tree view of the ACL to a specified depth.

## Class Security_Presentation() ##

There is a presentation class which simplifies the laying out of the
information on the page: Security_Presentation(). This provides a
variety of methods yielding data in HTML or JSON formats.

## Class Security_Process() ##

There is a process class that is intended to help with executing 
requests that come back from the page: Security_Process(). These 
respond to $_REQUEST (POST AND GET) and $_FILE data in the received
packet.

