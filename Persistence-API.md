A Multi-Index container abstraction of persistent data on EOSIO blockchains

## Table of Contents

- [Overview](#overview)
   * [The Need for Persistence Services](#need-for-persistence-services)
- [The EOSIO Multi-Index API](#eosio-multi-index-api)
   * [EOSIO Multi-Index Iterators](#eosio-multi-index-iterators)
- [Putting It All Together](#putting-it-all-together)
   * [How to Create Your EOSIO Multi-Index Table](#how-to-create-your-own-table)
   * [How to Use Your EOSIO Multi-Index Table](#how-to-use-your-own-table)
- [Vehicle Maintenance Tracker Example](#vehicle-maintenance-tracker-example)
- [C++ API Reference](#cpp-api-reference)
   * [eosio::multi_index](#multi-index-api)
   * [eosio::indexed_by](#indexed-by-api)
   * [eosio::multi_index::index](#index-api)

## Overview

EOSIO provides a set of services and interfaces that enable contract developers to
persist state across action, and consequently transaction, boundaries.  Without persistence, state that is
generated during the processing of actions and transactions will be lost when processing goes out of scope.  The
persistence components include:

1. Services to persist state in a database
2. Enhanced query capabilities to find and retrieve database content
3. C++ APIs to these services, intended for use by contract developers
4. C APIs for access to core services, of interest to library and system developers

This document covers the first three topics.

<a name="need-for-persistence-services"></a>
### The Need for Persistence Services

Actions perform the work of EOSIO contracts. Actions operate within an environment known as the action context.  As illustrated in the diagram below, 
an action context provides several things necessary for the execution of the action.
One of those things is the action's working memory. This is where the action maintains its working
state. Before processing an action,
EOSIO sets up a clean working memory for the action. Variables that might have been set when another action executed
are not available within the new action's context. The only way to pass state among actions is to persist it to and retrieve
it from the EOSIO database.

![Actions-Transactions-Mutable-Immutable-Data](assets/action-apply-context-diagram.png)

<a name="eosio-multi-index-api"></a>
## The EOSIO Multi-Index API

The EOSIO Multi-Index API provides a C++ interface to the EOSIO database.  The EOSIO Multi-Index API a is patterned after 
[Boost Multi-Index Containers](https://www.boost.org/doc/libs/1_66_0/libs/multi_index/doc/index.html).  This API provides a
model for object storage with rich retrieval capabilities, enabling the use of multiple indices with different sorting
and access semantics. The Multi-Index API is provided by the [`eosio::multi_index`](https://github.com/EOSIO/eos/blob/master/contracts/eosiolib/multi_index.hpp) 
C++ class found in the `contracts/eosiolib` folder of the [`EOSIO/eos` GitHub repository](https://github.com/EOSIO/eos/).
This class enables a contract written in C++ to read and modify persistent state in the EOSIO database.

The Multi-Index container interface `eosio::multi_index` provides a homogeneous container of an arbitrary 
C++ type (and it does not need to be a plain-old data type or be fixed-size) that is kept sorted in multiple indices by 
keys of various types that are derived from the objects. It can be compared to a traditional database table with rows, 
columns, and indices. It can also be easily compared to [Boost Multi-index Containers](https://www.boost.org/doc/libs/1_66_0/libs/multi_index/doc/index.html).
In fact many of the member function signatures of `eosio::multi_index` are modeled after `boost::multi_index`, although 
there are important differences.

`eosio::multi_index` can be conceptually viewed as tables in a conventional database in which the rows are the 
individual objects in the container, the columns are the member properties of the objects in the container,
and the indices provide fast lookup of an object by a key compatible with an object member property.

Traditional database tables allow the index to be a user-defined function over some number of columns of the table.
`eosio::multi_index` similarly allows the index to be any user-defined function (provided as a member function of 
the `class`/`struct` of the element type) but with its return value restricted to one of a limited set of supported key 
types.

Traditional database tables typically have a single unique primary key that allows 
unambiguously identifying a particular row in the table and also provides the standard sort order for the rows in the 
table. `eosio::multi_index` supports a similar semantic, but the primary key of the object in the `eosio::multi_index` 
container must be a unique unsigned 64-bit integer. The objects in the `eosio::multi_index` container are sorted by the
primary key index in ascending order of the unsigned 64-bit integer primary key.

<a name="eosio-multi-index-iterators"></a>
### EOSIO Multi-Index Iterators

A key differentiator of the EOSIO persistence services over other blockchain infrastructures is its
Multi-Index iterators. Unlike some other blockchains that only provide a key-value store, EOSIO Multi-Index tables allow a 
contract developer to keep a collection of objects sorted by a variety of different key types, which could be derived 
from the data within the object.  This enables rich retrieval capabilities.  Up to 16 secondary indices can be
defined, each having its own way of ordering and retrieving table contents.

The EOSIO Multi-Index iterators follow a pattern that is common to C++ iterators.  All iterators are bi-directional
`const`, either `const_iterator` or `const_reverse_iterator`.  The iterators can be dereferenced to provide access to an
object in the Multi-Index table. 

<a name="putting-it-all-together"></a>
## Putting It All Together

<a name="how-to-create-your-own-table"></a>
### How to Create Your EOSIO Multi-Index Table

Here is a summary of the steps to create your own persistent data using EOSIO Multi-Index tables.

- Define your object(s) using C++ `class` or `struct`.  Each object will be in its own Multi-Index table.
- Define a `const` member function in the `class`/`struct` called `primary_key` that returns the `uint64_t`
primary key value of your object.
- Determine the secondary indices. Up to 16 additional indices are supported. A secondary index
supports several key types, listed below.
    - `idx64` - Primitive 64-bit unsigned integer key
    - `idx128` - Primitive 128-bit unsigned integer key
    - `idx256` - 256-bit fixed-size lexicographical key
    - `idx_double` - Double precision floating point key
    - `idx_long_double` - Quadruple precision floating point key
- Define a key extractor for each secondary index. The key extractor is a function used to obtain the keys
from the elements of the Multi-Index table. See [Multi-Index Constructor](#multi-index-constructor) and
[indexed_by](#indexed-by-api) sections below.

<a name="how-to-use-your-own-table"></a>
### How to Use Your EOSIO Multi-Index Table

- Instantiate your Multi-Index table.
- Insert (`emplace`) into, and subsequently `modify` or `erase` objects in your table as required by your contract.
- Locate and traverse objects in your table using `get`, `find` and iterator operations.

<a name="vehicle-maintenance-tracker-example"></a>
## Vehicle Maintenance Tracker Example

In the following example, we will look at how to use the EOSIO Multi-Index API to implement a simple vehicle maintenance tracker.
The tracker will keep a ledger of vehicle maintenance activities.

The target users of the tracker are service mechanics and their customers.  A service mechanic will add records
to the ledger as service jobs are done.  They can use this information to track service history and notify customers when
service is due. Customers can also track their service history.
They can also periodically update their vehicle mileage to enable their mechanic to better determine when service is needed.

If we were implementing the full contract for this example, we might have actions such as:

- Create a new service record, which can only be done by the mechanic
- Update mileage, which can be done by the mechanic for any vehicle or a customer for its vehicle
- Various types of reporting actions

For the purposes of this example, we will focus on the aspects of storage and retrieval, and not the contract actions.

We actually need two tables for our application.  The first table will contain individual service transactions created
by the mechanic. The same customer can have many records in this table, representing each time the vehicle was serviced.
Our second table will contain the current state for a customer. Each customer will have one entry in this table.  Note
that for simplicity, our design here limits one vehicle per customer.

### Vehicle Maintenance Tracker `service` Table
We'll call the table that contains individual service transactions the `service` table. This table
will be used to create service record reports. The records of this table will contain the following properties: 
- Primary key - the Customer ID can't be the primary key, since a customer can have many records.  In fact, we don't
need the primary key directly, so we can let the system generate one for us.
- Customer ID - this will correspond to the account name (a uint64_t value) of the customer
- Date of Service - the date when the service was performed
- Odometer - the vehicle's odometer reading at date of service

We want to be able to search this table by customer,
so we will create a secondary index on the customer property.  The diagram below illustrates the `service` table and
its secondary index, `bycustomer`.

![Service Table Example](assets/service-table-example.png)

We will declare a struct `service_rec` with these properties to represent our service record object:

```
struct service_rec {
    uint64_t        pkey;           // opaque, will use available_primary_key()
    account_name    customer;       // will create a secondary index on this
    uint32_t        service_date;
    uint32_t        odometer;
    
    auto            primary_key()const { return pkey; }
    account_name    get_customer()const { return customer; }

    EOSLIB_SERIALIZE( service_rec, (pkey)(customer)(service_date)(odometer) )
};
```
To instantiate our `service` table (we'll call our instance `service_table`), we write the following in our contract,
where `mechanic` is the account name for the mechanic.
```
using service_table_type = multi_index<service, service_rec,
    indexed_by< N(bycustomer), const_mem_fun<service_rec, account_name, &service_rec::get_customer> >
>;
service_table_type service_table( current_receiver(), mechanic );
```
The two parameters passed to the constructor establish access privileges to the table. The first parameter (the
`code` parameter, see [below](#multi-index-constructor)) determines whether the action is accessing the contract's
own persistent state (i.e., `code == current_receiver())`, in which case the action context has both
read and write access to the table, or if it is accessing some other contract's persistent state and is,
therefore, read-only.

To add records to `service_table`, we can use C++ similar to the following (assuming corresponding local variables for
the service record content). Variable `customer_name` contains the human-readable string name of the customer,
which we must convert to the `account_name` encoding of the customer stored in the table.
```
service_table.emplace(mechanic, [&]( auto& s_rec ) {
    s_rec.pkey = service_table.available_primary_key();
    s_rec.customer = eosio::chain::string_to_name(customer_name);
    s_rec.service_date = service_date;
    s_rec.odometer = odometer;
});
```
To find all service records for a customer, we first get the `bycustomer` secondary index,
```
auto customer_index = service_table.template get_index<N(bycustomer)>();
```
then we can `find` the desired customer using the secondary index.
```
account_name customer_acct = eosio::chain::string_to_name(customer_name);
auto cust_itr = customer_index.find(customer_acct);
```
We can iterate through the secondary index to get all `service` records for the customer.

```
while (cust_itr != service_table.end() && cust_itr->customer == customer_acct) {
    // code to process customer record...
    ...
    cust_itr++;
} 
```

### Vehicle Maintenance Tracker `customer` Table
Our second table will contain the current state for a customer. We'll call this the `customer` table. This state consists
of the following properties:

- Customer ID - this will be unique for the `customer` table, so we can use it as our primary key _(note that for simplicity,
this example limits to one vehicle per customer)_;
- Last Service Date - the last time the vehicle was in for routine service
- Odometer at Last Service Date - the odometer reading when the vehicle was last in for service
- Miles Since Last Service - this will be calculated each time the customer or mechanic updates the latest odometer
value (e.g., voluntarily by the customer, and each time the mechanic performs a service)

We will index the `customer` table on the Last Service Date and on the Miles Since Last Service.  The diagram below
illustrates the `customer` table and its two secondary indices, `bydate` and `bymiles`. Note in the example that Sue has
volunteered her latest odometer reading, presumably using an interface provided by the application. The corresponding Miles
Since Last Service is calculated whenever the customer enters miles, and this value is indexed. This enables the mechanic
to identify customers needing service by date or by miles, at least for customers who choose to participate.  Examples
of using these values are given below.

![Customer Table Example](assets/customer-table-example.png)

We will declare a struct `customer_rec` with these properties to represent our customer record object:
```
struct customer_rec {
    account_name    customer_id             // primary key
    uint64_t        last_service_date;      // will create a secondary index on this
    unit32_t        odometer_at_last_service;
    uint32_t        latest_odometer;
    
    auto            primary_key()const { return customer_id; }
    uint64_t        get_last_service_date()const { return last_service_date; }
    uint64_t        get_miles_since_service()const {
                        return latest_odometer > odometer_at-last_service ? latest_odometer - odometer_at_last_service : 0;
                    }

    EOSLIB_SERIALIZE( customer_rec, (customer_id)(last_service_date)(odometer_at_last_service)(latest_odometer) )
};
```
To instantiate our `customer` table (we'll call it `customer_table`), we write the following in our contract,
where `mechanic` is the account name for the mechanic.
```
using customer_table_type = multi_index<customer, customer_rec,
   indexed_by< N(bydate), const_mem_fun<customer_rec, uint64_t, &customer_rec::get_last_service_date> >,
   indexed_by< N(bymiles), const_mem_fun<customer_rec, uint64_t, &customer_rec::get_miles_since_service> >
>;
customer_table_type customer_table( current_receiver(), mechanic );
```
To add records to `customer_table`, we can use C++ similar to the following (assuming corresponding local variables for
the customer record content). Variable `customer_name` contains the human-readable string name of the customer, which
we must convert to the `account_name` encoding of the customer stored in the table.
```
customer_table.emplace(mechanic, [&]( auto& c_rec ) {
    c_rec.customer_id = eosio::chain::string_to_name(customer_name);
    c_rec.last_service_date = current_date;
    c_rec.odometer_at_last_service = current_odometer;
    c_rec.latest_odometer = current_odometer;
});
```
We can `find` a customer using the primary key, `customer_id`.
```
account_name customer_id = eosio::chain::string_to_name(customer_name);
auto cust_itr = customer_table.find(customer_id);
```
To update the `latest_odometer` property, use something similar to the following.
```
account_name customer_id = eosio::chain::string_to_name(customer_name);
customer_table.modify(customer_table.get(customer_id), mechanic, [&]( auto& c_rec) {
    c_rec.latest_odometer = customer_provided_odometer_reading;     // customer provided input
});
```
Parameter `mechanic` is the payer for the update action.  Our example allows the mechanic or
customer to update the `latest_odometer` property, but we don't want the customer paying just to
volunteer their latest odometer reading.

Note that the above requires calculating the `miles_since_service` property.  The example assumes the
variable `customer_odometer_at_last_service` holds the odometer reading from the customer's last service.
We can obtain the most recent service record for the customer by using the `bycustomer` secondary
index of the `service` table. See the `service` table example above for using this index.

To get the secondary indices for `customer_table`, use something similar to the following.
```
auto last_service_date_index = customer_table.template get_index<N(bydate)>();
auto miles_since_service_index = customer_table.template get_index<N(bymiles)>();
```
We can iterate backwards through the `bydate` secondary index to get all customers who are due for service, e.g., because their
last service is more than 3 months ago.
```
auto overdue_date_itr = last_service_date_index.upper_bound(three_months_ago);  // date for three months earlier than today
// code to iterate the customer table backwards from three months ago
...
```
We can iterate forward through the `bymiles` secondary index to get all customers who are due for service, e.g.,
because their miles since last service is greater than 3000. 
```
auto over_miles_itr = last_service_date_index.lower_bound(3000);  // all records 3000 or more miles
// code to iterate the customer table forward for greater than 3000 miles
...
```

<a name="cpp-api-reference"></a>
# C++ API Reference

<a name="multi-index-api"></a>
## eosio::multi_index

This section documents the `eosio::multi_index` C++ API.  This interface is, effectively, an adaptation of the
Boost Multi-Index container library.  It makes direct use of the `boost/multi_index/const_mem_fun` class template for key 
extraction.  It also uses the Boost Hana library for metaprogramming.  Please refer to the Boost documentation 
at [www.boost.org](https://www.boost.org) for additional conceptual information and detail.

In the descriptions below, the following aliases will be used for common template declarations.

Alias | Description
----- | -----
_object_type_ | The type of an object in the Multi-Index table
_secondary_index_ | The type of the appropriate Secondary Index of the Multi-Index table

<a name="multi-index-constructor"></a>
## Constructor

Method | Description
----- | -----
`multi_index( uint64_t code, uint64_t scope )` | Constructs an instance of a Multi-Index table
_**Parameters**_ | 
`code` | account that owns table
`scope` | scope identifier within the `code` hierarchy
_**Postcondition**_ | A new empty instance of a Multi-Index table is created
 | `code` and `scope` member properties are initialized
 | each secondary index table initialized

_**Note:**_ The `eosio::multi_index` template has template parameters `<uint64_t TableName, typename T, typename... Indices>`, where:
 - `TableName` is the name of the table, maximum 12 characters long, characters in the name from the set of lowercase letters, digits 1 to 5, and the "." (period) character;
 - `T` is the object type (i.e., row definition);
 - `Indices` is a list of up to 16 secondary indices.
    - Each must be a default constructable class or struct
    - Each must have a function call operator that takes a const reference to the table object type and returns either a 
    secondary key type or a reference to a secondary key type
    - It is recommended to use the eosio::const_mem_fun template, which is a type alias to the boost::multi_index::const_mem_fun.  See the documentation for the Boost const_mem_fun key extractor for more details.

## Copy and Assignment

Copy constructors and assignment operators are not supported.

## Table Manipulation

### emplace

Method | Description
----- | -----
`const_iterator emplace( unit64_t payer, Lambda&& constructor )` | Adds a new object (i.e., row) to the table.
_**Parameters**_ | 
`payer` | account name of the payer for the Storage usage of the new object
`constructor` | lambda function that does an in-place initialization of the object to be created in the table
_**Returns**_ | a primary key Iterator to the newly created object
_**Precondition**_ | 
 | `payer` is a valid account that authorized the current action (and thus allowed to be billed for storage usage)
_**Postcondition**_ | 
 | A new object is created in the Multi-Index table, with a unique primary key (as specified in the object).  The object is serialized and written to the table. If the table does not exist, it is created.
 | Secondary indices are updated to refer to the newly added object. If the secondary index tables do not exist, they are created.
 | The payer is charged for the storage usage of the new object and, if the table (and secondary index tables) must be created, for the overhead of the table creation.
_**Exceptions**_ | The current receiver is not the account that owns the table (the `code` of `multi_index`).

### get

Method | Description
----- | -----
`const object_type& get( uint64_t primary )const` | Retrieves an existing object from a table using its primary key.
_**Parameters**_ | 
`primary` | primary key value of the object
_**Returns**_ | A constant reference to the object containing the specified primary key.
_**Precondition**_ | None
_**Postcondition**_ | None
_**Exceptions**_ | No object matches the given key

### find

Method | Description
----- | -----
`const_iterator find( uint64_t primary )const` | Search for an existing object in a table using its primary key.
_**Parameters**_ | 
`primary` | primary key value of the object
_**Returns**_ | 
 | An iterator to the found object which has a primary key equal to `primary`, _**OR**_
 | The `end` iterator of the referenced table if an object with primary key `primary` is not found
_**Precondition**_ | None
_**Postcondition**_ | None
_**Exceptions**_ | None

### modify

Method | Description
----- | -----
`void modify( const_iterator itr, uint64_t payer, Lambda&& updater )` | Modifies an existing object in a table.
`void modify( const object_type &obj, uint64_t payer, Lambda&& updater )` | 
_**Parameters**_ | 
`itr` | an iterator pointing to the object to be updated
`obj` | a reference to the object to be updated
`payer` | account name of the payer for the Storage usage of the updated row; a value of `0` indicates that the payer of the modified row is the same as the existing payer.
`updater` | lambda function that updates the target object
_**Precondition**_ | 
 | `itr` points to, or `obj` is an existing object in the table.
 | `payer` is a valid account that authorized the current action (and thus allowed to be billed for storage usage)
_**Postcondition**_ | 
 | The modified object is serialized, then replaces the existing object in the table.
 | Secondary indices are updated; the primary key of the updated object is not changed.
 | The `payer` is charged for the storage usage of the updated object.
 | If `payer` is the same as the existing payer, `payer` only pays for the usage difference between existing and updated object (and is refunded if this difference is negative).
 | If `payer` is different from the existing payer, the existing payer is refunded for the storage usage of the existing object.
_**Exceptions**_ |
 | If called with an invalid precondition, execution is aborted.
 | The current receiver is not the account that owns the table (the `code` of `multi_index`).

### erase

Method | Description
----- | -----
`const_iterator erase( const_iterator itr )` | Remove an existing object from a table using its primary key.
`void erase( const object_type& obj )` | 
_**Parameters**_ | 
`itr` | an iterator pointing to the object to be removed
`obj` | a reference to the object to be removed
_**Returns**_ | For the signature with `const_iterator`, returns a pointer to the object following the removed object.
_**Precondition**_ | None
_**Postcondition**_ | 
 | The object is removed from the table and all associated storage is reclaimed.
 | Secondary indices associated with the table are updated.
 | The existing payer for storage usage of the object is refunded for the table and secondary indices usage of the removed object, and if the table and indices are removed, for the associated overhead.
_**Exceptions**_ | 
 | The object to be removed is not in the table.
 | The action is not authorized to modify the table.
 | The given iterator is invalid.

## Member Access

### get_code

Method | Description
----- | -----
`uint64_t get_code() const` | Returns the `code` member property.
_**Returns**_ | account name of the Code that owns the Primary Table

### get_scope

Method | Description
----- | -----
`uint64_t get_scope() const` | Returns the `scope` member property.
_**Returns**_ | scope id of the Scope within the Code of the Current Receiver under which the desired Primary Table instance can be found

## Utilities

### available_primary_key

Method | Description
----- | -----
`uint64_t available_primary_key()const` | Returns an available (unused) primary key value, intended to be used in tables in which the primary keys of the table are strictly intended to be auto-incrementing, and thus will never be set to custom values by the contract.  Violating this expectation could result in the table appearing to be full due to inability to allocate an available primary key. 


## Iterators

The Multi-Index iterators follow a pattern that is common to C++ iterators.  All iterators are bi-directional
`const`, either `const_iterator` or `const_reverse_iterator`.  The iterators can be dereferenced to provide access to an
object in the Multi-Index table. 

### begin and cbegin

Method | Description
----- | -----
`const_iterator begin()const` | Returns an iterator pointing to the `object_type` with the lowest primary key value in the Multi-Index table.
`const_iterator cbegin()const` | 


### end and cend

Method | Description
----- | -----
`const_iterator end()const` | Returns an iterator pointing to the virtual row representing just past the last row of the table (assuming one exists); cannot be dereferenced; can be advanced backward but not forward.
`const_iterator cend()const` | 

### rbegin and crbegin

Method | Description
----- | -----
`const_iterator rbegin()const` | Returns a reverse iterator pointing to the `object_type` with the highest primary key value in the Multi-Index table.
`const_iterator crbegin()const` | 

### rend and crend

Method | Description
----- | -----
`const_iterator rend()const` | Returns an iterator pointing to the virtual row representing just past the last row of the table (assuming one exists); cannot be dereferenced; can be advanced forward but not backward.
`const_iterator crend()const` | 

### lower_bound

Method | Description
----- | -----
`const_iterator lower_bound( uint64_t primary )const` | Searches for the `object_type` with the lowest primary key that is greater than or equal to a given primary key.
_**Parameters**_ | 
`primary` | Primary key that establishes the target value for the lower bound search
_**Returns**_ | 
 | an iterator pointing to the `object_type` that has the lowest primary key that is greater than or equal to `primary`, _**OR**_
 | the `end` iterator if an object could not be found, including if the table does not exist

### upper_bound

Method | Description
----- | -----
`const_iterator upper_bound( uint64_t primary )const` | Searches for the `object_type` with the lowest primary key that is greater than a given primary key.
_**Parameters**_ | 
`primary` | Primary key that establishes the target value for the upper bound search
_**Returns**_ | 
 | an iterator pointing to the `object_type` that has the lowest primary key that is greater than `primary`, _**OR**_
 | the `end` iterator if an object could not be found, including if the table does not exist

### get_index

Method | Description
----- | -----
`secondary_index get_index<IndexName>()` | Returns an appropriately typed Secondary Index.
`secondary_index get_index<IndexName>() const` | 
_**Parameters**_ | 
`IndexName` | the ID of the desired secondary index
_**Returns**_ | An index of the appropriate type, i.e., one of the following:
`idx64` | Primitive 64-bit unsigned integer key
`idx128` | Primitive 128-bit unsigned integer key
`idx256` | 256-bit fixed-size lexicographical key
`idx_double` | Double precision floating point key
`idx_long_double` | Quadruple precision floating point key


### iterator_to

Method | Description
----- | -----
`const_iterator iterator_to( const object_type& obj )const` | Returns an iterator to the given object in a Multi-Index table.
_**Parameters**_ |
`obj` | a reference to the desired object
_**Returns**_ | An iterator to the given object


<a name="indexed-by-api"></a>
### indexed_by

The `indexed_by` struct is used to instantiate the indices for the Multi-Index table. In EOSIO, up to 16 secondary indices
can be specified.
```
template<uint64_t IndexName, typename Extractor>
struct indexed_by {
   enum constants { index_name   = IndexName };
   typedef Extractor secondary_extractor_type;
};
```

_**Parameters**_

- `IndexName` is the name of the index.  The name must be provided as an EOSIO base32 encoded 64-bit integer and must
conform to the EOSIO naming requirements of a maximum of 13 characters,
the first twelve from the lowercase characters a-z, digits 0-5, and ".", and if there is a 13th character, it is restricted
to lowercase characters a-p and ".".
- `Extractor` is a function call operator that takes a const reference to the table object type and returns either a 
secondary key type or a reference to a secondary key type. It is recommended to use the `eosio::const_mem_fun` template,
which is a type alias to the `boost::multi_index::const_mem_fun`.  See the documentation for the Boost `const_mem_fun` key
extractor for more details.

*Example*

      multi_index<mytable, record,
         indexed_by< N(bysecondary), const_mem_fun<record, uint128_t, &record::get_secondary> >
      > table( code, scope);

*In this example, the Multi-Index table is named `mytable`, has object (row) type `record`, is indexed by a secondary index
named `bysecondary`, whose name is converted to the required encoded format using
the `N` macro. A `const_mem_fun` key extractor is specified for the `record` object class; the key type is `uint128_t`, 
obtained using the `record::get_secondary` function.*

<a name="index-api"></a>
## The eosio::multi_index::index API (Secondary Indices)
An EOSIO Multi-Index table can have up to 16 secondary indices.  An index 
can be retrieved from the `multi_index` object using the `get_index` method, and can then be iterated using a common
C++ iterator pattern.

The `index` API described here operates in many ways the same as the `multi_index` described above, but it also
differs in some significant ways.
- An `index` is intended to provide alternate means to access the Multi-Index Table.
- The content of a secondary index table is basically a key mapping to a primary key.  That mapping is transparent to the user.
- Some operations do not apply to secondary indices, most notably, one does not `emplace` content using a secondary index.
Objects can be modified and erased using a secondary index.
- Operations for retrieving primary content using a secondary table are virtually the same, differing primarily in the 
key values that are used (you cannot pass an object to a secondary index to find it in the table). The key values will
be one of the supported secondary index types:
    - `idx64` - Primitive 64-bit unsigned integer key
    - `idx128` - Primitive 128-bit unsigned integer key
    - `idx256` - 256-bit fixed-size lexicographical key (does not support arithmetic operations)
    - `idx_double` - Double precision floating point key
    - `idx_long_double` - Long Double (quadruple) precision floating point key
- Secondary indices have `code` and `scope` member properties, the same as the corresponding properties
on the Multi-Index table.

Alias | Description
----- | -----
_object_type_ | The typename of an object in the Multi-Index table
_secondary_index_ | An appropriately typed reference to a Secondary Index

## Constructor

A secondary index is created as part of the construction of the Multi-Index table.  Direct construction of `index` is
not supported.

## Copy and Assignment

Copy constructors and assignment operators are not supported.

## Table Manipulation

### get

Method | Description
----- | -----
`const object_type& get( secondary_key_type&& secondary )const` | Retrieves an existing object from a table using its secondary key. Multiple signatures are available.
`const object_type& get( const secondary_key_type& secondary )const` | 
_**Parameters**_ | 
`secondary` | secondary key value of the object
_**Returns**_ | A constant reference to the object containing the specified secondary key.
_**Precondition**_ | None
_**Postcondition**_ | None
_**Exceptions**_ | No object matches the given key

### find

Method | Description
----- | -----
`const_iterator find( secondary_key_type&& secondary )const` | Search for an existing object in a table using a secondary key.
`const_iterator find( const secondary_key_type&& secondary )const` | 
_**Parameters**_ | 
`secondary` | secondary key value of the object
_**Returns**_ | 
 | An iterator to the found object which has a secondary key equal to `secondary`, _**OR**_
 | The `end` iterator if an object with secondary key `secondary` is not found
_**Precondition**_ | None
_**Postcondition**_ | None
_**Exceptions**_ | None

### modify

Method | Description
----- | -----
`void modify( const_iterator itr, uint64_t payer, Lambda&& updater )` | Modifies an existing object in a table.
_**Parameters**_ | 
 `itr` | an iterator pointing to the object to be updated
`payer` | account name of the payer for the Storage usage of the updated row; a value of `0` indicates that the payer of the modified row is the same as the existing payer.
`updater` | lambda function that updates the target object
_**Precondition**_ | 
 | `itr` points to an existing object in the table.
 | `payer` is a valid account that authorized the current action (and thus allowed to be billed for storage usage)
_**Postcondition**_ | 
 | The modified object is serialized, then replaces the existing object in the table.
 | Secondary indices are updated; the primary key of the updated object is not changed.
 | The `payer` is charged for the storage usage of the updated object.
 | If `payer` is the same as the existing payer, `payer` only pays for the usage difference between existing and updated object (and is refunded if this difference is negative).
 | If `payer` is different from the existing payer, the existing payer is refunded for the storage usage of the existing object.
_**Exceptions**_ |
 | If called with an invalid precondition, execution is aborted.
 | The current receiver is not the account that owns the table (the `code` of `multi_index`).

### erase

Method | Description
----- | -----
`const_iterator erase( const_iterator itr )` | Remove an existing object from a table using its secondary key.
_**Parameters**_ | 
`itr` | an iterator pointing to the object to be removed
_**Returns**_ | For the signature with `const_iterator`, returns a pointer to the object following the removed object.
_**Precondition**_ | None
_**Postcondition**_ | 
 | The object is removed from the table and all associated storage is reclaimed.
 | Secondary indices associated with the table are updated.
 | The existing payer for storage usage of the object is refunded for the table and secondary indices usage of the removed object, and if the table and indices are removed, for the associated overhead.
_**Exceptions**_ | 
 | The object to be removed is not in the table.
 | The action is not authorized to modify the table.
 | The given iterator is invalid.

## Member Access

### get_code

Method | Description
----- | -----
`uint64_t get_code() const` | Returns the `code` member property.
_**Returns**_ | account name of the Code that owns the Secondary Index Table

### get_scope

Method | Description
----- | -----
`uint64_t get_scope() const` | Returns the `scope` member property.
_**Returns**_ | scope id of the Scope within the Code of the Current Receiver under which the desired Secondary Index Table instance can be found

### name

Method | Description
----- | -----
`constexpr static uint64_t name()` | Returns the `index_table_name` member constant.
_**Returns**_ | the table name of the Secondary Index

### number

Method | Description
----- | -----
`constexpr static uint64_t number()` | Returns the `index_number` member constant.
_**Returns**_ | the number of the Secondary Index

## Iterators

The EOSIO secondary index iterators follow a pattern that is common to C++ iterators.

### const_iterator

Method | Description
----- | -----
`struct const_iterator : public std::iterator<std::bidirectional_iterator_tag, const object_type> ` | A bi-directional iterator to objects in a Multi-Index table.
_**Parameters**_ | 
`object_type` | the typename of the object in the table

### const_reverse_iterator

Method | Description
----- | -----
`typedef std::reverse_iterator<const_iterator> const_reverse_iterator` | A reverse bi-directional iterator to objects in a Multi-Index table.

### begin and cbegin

Method | Description
----- | -----
`const_iterator begin()const` | Returns an iterator pointing to the lowest secondary key value in the secondary index table.
`const_iterator cbegin()const` | 

### end and cend

Method | Description
----- | -----
`const_iterator end()const` | Returns an iterator pointing to the virtual row representing just past the last row of the table (assuming one exists); cannot be dereferenced; can be advanced backward but not forward.
`const_iterator cend()const` | 

### rbegin and crbegin

Method | Description
----- | -----
`const_iterator rbegin()const` | Returns a reverse iterator pointing to the highest secondary key value in the secondary index table.
`const_iterator crbegin()const` | 

### rend and crend

Method | Description
----- | -----
`const_iterator rend()const` | Returns a reverse iterator pointing to the virtual row representing just before the first row of the table (assuming one exists); cannot be dereferenced; can be advanced forward but not backward.
`const_iterator crend()const` | 

### lower_bound

Method | Description
----- | -----
`const_iterator lower_bound( secondary_key_type&& secondary )const` | Searches for the `object_type` with the lowest secondary key that is greater than or equal to a given secondary key.
`const_iterator lower_bound( const secondary_key_type&& secondary )const` | 
_**Parameters**_ | 
`secondary` | Secondary key that establishes the target value for the lower bound search
_**Returns**_ |
 | an iterator pointing to the object in the secondary index that has the lowest secondary key that is greater than or equal to `secondary`, _**OR**_
 | the `end` iterator if an object could not be found, including if the table does not exist

### upper_bound

Method | Description
----- | -----
`const_iterator upper_bound( secondary_key_type&& secondary )const` | Searches for the `object_type` with the lowest secondary key that is greater than a given secondary key.
`const_iterator upper_bound( const secondary_key_type&& secondary )const` | 
_**Parameters**_ | 
`secondary` | Secondary key that establishes the target value for the upper bound search
_**Returns**_ | 
 | an iterator pointing to the object in the secondary index that has the lowest secondary key that is greater than `secondary`, _**OR**_
 | the `end` iterator if an object could not be found, including if the table does not exist

### iterator_to

Method | Description
----- | -----
`const_iterator iterator_to( const object_type& obj )const` | Returns a secondary index iterator to the given object in the Multi-Index table.
_**Parameters**_ | 
`obj` | a const reference to the desired object
_**Returns**_ | An iterator to the given object

## Utilities

### extract_secondary_key

Method | Description
----- | -----
`static auto extract_secondary_key(const object_type& obj)` | Returns the secondary key value for an object from a secondary index.
_**Parameters**_ | 
`obj` | a const reference to the desired object
_**Returns**_ | A key value for the given object, for that index
