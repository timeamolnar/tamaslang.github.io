---
layout: post
title: Interfaces of Rest Devtools
---

### Common interfaces
Some specific purpose interfaces are defined as part of the development tools. <br/>
These interfaces are serving the goal to help the usage of a common approach for general problems like database entity identifying, versioning.

### Identification:
Most of the DB classes needs to be identifiable by some type of id. <br/>
For this a generic IdentifiableById interface presents that inherits from the Identifiable marker interface.
![placeholder]({{ site.url }}/assets/devtools_identifiable.png "Identifiable")

```java
/**
 * Interface to mark objects that are identifiable by an ID of any type.
 */
public interface IdentifiableById<T> extends Identifiable{
    T getId();
    void setId(T id);
}
```

### Tracking created and modified dates and authors
Another common requirement is to track the dates and authors of created and modified events for certain entities.<br/>
The interfaces define standard names for the date fields: 'created&#95;on' and 'modified&#95;on'<br/>
and for the author fields : 'created&#95;by' and 'modified&#95;by'.

The type for the date fields are Jodatime's DateTime.<br/>
For the author fields the type is generic, a Long id might fit most of the time but going on with a String type like email address is also possible.
![placeholder]({{ site.url }}/assets/devtools_created_modified.png "Tracking Created and Modified dates")

### Soft delete entities
There might be a requirement to soft delete entities, so when a delete event occurs
these entities should still present in the database but should not be accessible for most of the queries except the ones which
are specifically interested in the deleted entities.<br/>
For this the SoftDeletable interface defines a deleted flag and suggests the 'deleted' field name to use in the database.

![placeholder]({{ site.url }}/assets/devtools_softdeletable.png "Soft deletable")

```java
/**
 * Interface to support soft deletion
 */
public interface SoftDeletable {

    String DELETED_FIELD_NAME = "deleted";

    boolean isDeleted();

    void setDeleted(boolean deleted);

}
```

### Versioning
Versioning might also present as a requirement for certain entities to avoid update conflicts. <br/>
The Versionable interface defines a version Integer field and suggests the 'version' field name to map it in the database.

![placeholder]({{ site.url }}/assets/devtools_versionable.png "Versionable")

```java
/**
 * Interface for versionable objects
 */
public interface Versionable {

    String VERSION_FIELD_NAME = "version";

    public Integer getVersion();

    public void setVersion(Integer version);
    
}
```

### Summary
These interfaces might ease up and standardise the solutions for the above mentioned common requirements. <br/>
Using them however is not mandatory when the the rest-devtools library is included.
