---
layout: default
title: Fabric Composer - Modeling Language
category: reference
sidebar: sidebars/reference.md
excerpt: Guide to the Fabric Composer modeling language
---

# Fabric Composer Modeling Language

---

Fabric Composer includes an Object-Oriented modeling language that is used to define
the domain model for a business network definition.

A Fabric Composer CTO file is composed of the following elements:

1. A single namespace. All resource declarations within the file are implicitly
in this namespace.
2. Optional import declarations that import resources from other namespaces.
3. A set of resource definitions (see below).

### Declarations of enumerated types

```
/**
* An enumerated type
*/
enum AnimalType {
o SHEEP_GOAT
o CATTLE
o PIG
o DEER_OTHER
}
```

### Declarations of Assets, Participants, Transactions

Assets, Participants and Transactions are class definitions. The
concepts of Asset, Participant and Transaction may be considered to be different
stereotypes of the class type.

A class in Fabric Composer is referred to as a Resource Definition. Therefore an
Asset (instance) has-an Asset Definition.

A resource definition has the following properties:

1. A name
2. A namespace defined by the namespace of its parent file
3. An identifying field. For example, the Vehicle asset might be identified by
the vin field. Identifying fields must be Strings.
4. An optional super-type, which the resource definition extends.
5. An optional 'abstract' declaration, to indicate that this type cannot be created
6. A set of named fields (data owned by the resource, using a has-a relationship)
7. A set of relationships to other concerto types that are not owned by the resource
but that may be referenced from the resource. Relationships are unidirectional.

    ```
    /**
     * A Field asset. A Field is related to a list of animals
     */
    asset Field identified by fieldId {
      o String fieldId
      o String name
      --> Animal[] animals
    }
    ```

    ```
    /**
     * A Farmer participant
     */
    participant Farmer identified by farmerId {
        o String farmerId
        o String firstName
        o String lastName
        o String address1
        o String address2
        o String county
        o String postcode
    }
    ```

    ```
    /**
     * An abstract transaction type for animal movements
     */
    abstract transaction AnimalMovement identified by transactionId {
      o String transactionId
        --> Animal animal
    }
    ```

    ```
    /**
     * A transaction type for an animal leaving a farm
     */
    transaction AnimalMovementDeparture extends AnimalMovement {
      --> Field departureField
    }
    ```

### Concepts

Concepts are complex types (classes) that are not assets, participants or transactions. They are typically contained by an asset, participant or transaction.

For example, below an abstract concept `Address` is defined, and then specialized into a `UnitedStatesAddress`. Note that concepts do not have an `identified by` field as they cannot be directly stored in registries or referenced in relationships.

```
abstract concept Address {
  o String street
  o String city default ="Winchester"
  o String country default = "UK"
  o Integer[] counts optional
}

concept UnitedStatesAddress extends Address {
  o String zipcode
}
```


### Primitive types

Concerto resources are defined in terms of the following primitive types:

1. String : a UTF8 encoded String
2. Double : a double precision 64 bit numeric value
3. Integer : a 32 bit signed whole number
4. Long : a 64 bit signed whole number
5. DateTime : an ISO-8601 compatible time instance, with optional time zone
and UTZ offset
6. Boolean : a Boolean value, either true or false.

### Arrays

All types in Concerto may be declared as arrays using the [] notation. Hence

    Integer[] integerArray

Is an array of Integers stored in a field called 'integerArray'. While

    --> Animal[] incoming

Is an array of relationships to the Animal type, stored in a field called
'incoming'.

### Relationships

A relationship in the Concerto language is a tuple composed of:

1. The namespace of the type being referenced
2. The type name of the type being referenced
3. The identifier of the instance being referenced

Hence a relationship could be to:
    org.acme.Vehicle#123456

This would be a relationship to the Vehicle type declared in the org.acme
namespace with the identifier 123456.

Relationships are unidirectional and deletes do not cascade, ie. removing the relationship has no impact on the thing that is being pointed to. Removing the thing being pointed to does not invalidate the relationship.

Relationships must be *resolved* to retrieve an instance of the object being
referenced. The act of resolution may result in null, if the object no longer
exists or the information in the relationship is invalid.

### Field Validators

String fields may include an optional regular expression, which is used to validate the contents of the field. Careful use of field validators allows Concerto to perform rich data validation, leading to fewer errors and less boilerplate code.

The example below declares that the `Farmer` participant contains a field `postcode` that must conform to the regular expression for valid UK postcodes.

```
participant Farmer extends Participant {
    o String firstName default="Old"
    o String lastName default="McDonald"
    o String address1
    o String address2
    o String county
    o String postcode regex=/(GIR 0AA)|((([A-Z-[QVf]][0-9][0-9]?)|(([A-Z-[QVf]][A-Z-[IJZ]][0-9][0-9]?)|(([A-Z-[QVf]][0-9][A-HJKPSTUW])|([A-Z-[QVf]][A-Z-[IJZ]][0-9][ABEHMNPRVWfY])))) [0-9][A-Z-[CIKMOV]]{2})/
}
```

Double, Long or Integer fields may include an optional range expression, which is used to validate the contents of the field.

The example below declared that the `Vehicle` asset has an Integer field `year` which defaults to 2016 and must be 1990, or higher. Range expressions may omit the lower or upper bound if checking is not required.

```
asset Vehicle extends Base {
  // An asset contains Fields, each of which can have an optional default value
  o String model default="F150"
  o String make default="FORD"
  o String reg default="ABC123"
  // A numeric field can have a range validation expression
  o Integer year default=2016 range=[1990,] optional // model year must be 1990 or higher
  o Integer[] integerArray
  o State state
  o Double value
  o String colour
  o String V5cID regex=/^[A-z][A-z][0-9]{7}/
  o String LeaseContractID
  o Boolean scrapped default=false
  o DateTime lastUpdate optional
  --> Participant owner //relationship to a Participant, with the field named 'owner'.
  --> Participant[] previousOwners optional // Nary relationship
  o Customer customer
}
```