---
title: "Commit Object(s)"
url: /refguide/committing-objects/
weight: 30
tags: ["studio pro"]
---

{{% alert color="warning" %}}
This activity can be used in both **Microflows** and **Nanoflows**.
{{% /alert %}}

## 1 Introduction

The **Commit** activity works on one or more objects. For persistable entities, committing an object stores it in the database. Committing non-persistable entities stores the current attribute values and association values in memory, this allows a rollback to revert to those values. See also [Persistability](/refguide/persistability/). External objects can't be committed. To store changed values of external objects, use the [Send External Object](/refguide/send-external-object/) activity.

{{% alert color="info" %}}
A Mendix commit does not always behave like a database commit. See [How Commits Work](#how-commits-work), below, for more information.
{{% /alert %}}

## 2 Properties

An example of commit object(s) properties is represented in the image below:

{{< figure src="/attachments/refguide/modeling/application-logic/microflows-and-nanoflows/activities/object-activities/committing-objects/commit-properties.png" alt="commit object(s) properties" >}}

There are two sets of properties for this activity, those in the dialog box on the left, and those in the properties pane on the right.

The commit object(s) properties pane consists of the following sections:

* [Action](#action)
* [Common](#common)

## 3 Action Section{#action}

The Action section of the properties pane shows the action associated with this activity.

You can open a dialog box to configure this action by clicking the ellipsis (**…**) next to the action.

You can also open the dialog box by double-clicking the activity in the microflow or right-clicking the activity and selecting **Properties**.

### 3.1 Object or List

The object or list of objects that you want to commit.

### 3.2 With Events

{{% alert color="info" %}}
This property is for microflows only.
{{% /alert %}}

Indicates whether or not to execute the commit event handlers of the objects.

Default: *Yes*

#### 3.2.1 Events in Nanoflows

Nanoflows do not have this property.

If the commit object(s) action is used in an online app, it sends a commit request to the Mendix Runtime and always runs the events.

If the commit object(s) action is used in an offline app, the changes are committed to the offline database, and event handlers are run when the offline app synchronizes.

### 3.3 Refresh in Client{#refresh-in-client}

This setting defines how changes are reflected in the pages presented to the end-user.

Default: *No*

{{% alert color="info" %}}
To make pages of a Mendix app efficient, many widgets display values from an attribute of an object which is cached on the page. Attributes in widgets which use cached data are *always* reflected in the client when they are updated or deleted irrespective of the value of **Refresh in client**.

If a widget is only updated when a [data source](/refguide/data-sources/) is loaded, then changes will only be seen when **Refresh in client** is set to *Yes*.

When testing your app, ensure that the desired data is being displayed by the widgets you have chosen.
{{% /alert %}}

{{% alert color="warning" %}}
When committing a large number of objects, we recommend that you do not enable 'Refresh in client' because it can slow things down.
{{% /alert %}}

#### 3.3.1 Microflow is Called from the Client in an Online App

If **Refresh in client** is set to *No*, the change is not reflected in the client.

If set to *Yes*, the object is refreshed across the client, which includes reloading the relevant [data sources](/refguide/data-sources/).

#### 3.3.2 Microflow is Called in an Offline, Native, or Hybrid App

When inside a microflow that is called from an offline, native, or hybrid app, the **Refresh in client** option is ignored and functions as if it was set to **No**.

For more information, see the [Microflows](/refguide/mobile/using-mobile-capabilities/offlinefirst-data/best-practices/#microflows) section of *Offline-First Data*.

#### 3.3.3 Action is in a Nanoflow

When inside a [nanoflow](/refguide/nanoflows/), the object is refreshed across the client as if **Refresh in client** was set to *Yes*.

## 4 Common Section{#common}

{{% snippet file="/static/_includes/refguide/microflow-common-section-link.md" %}}

## 5 How Commits Work{#how-commits-work}

### 5.1 Committing Objects

When you commit an object, the current value is saved. This means that you cannot rollback to the previous values of the object using the **Rollback** action of a microflow.

However, a Mendix **Commit** is not the same as a database **Commit**. For an object of a persistable entity, the saved value is not committed to the database until the microflow and any microflows from which it is called, completes. This means that errors in a microflow *can* initiate a rollback. If a microflow action errors and has **Error handling** set to *Rollback* or *Custom with rollback*, the value of the object *will* be rolled back to the value it had at the start of the microflow. See [Error Event](/refguide/error-event/#errors-in-microflows) for more information.

Mendix mimics this behavior for *non-persistable* entities. Committing a non persistable entity means you cannot use a **Rollback** action to go back to the previous values, although rollback error handling in a microflow *will* roll back to the original values.

### 5.2 Autocommit and Associated Objects

When an object is committed through a default Save button, a commit activity, or web services, it will always trigger the commit events. The platform will also evaluate all associated objects. To guarantee data consistency, the platform may also autocommit associated objects.

An autocommit is an automatic commit from the platform, which is done to keep the domain model in sync. If your application ends up having autocommitted objects, then you will have a modeling error. Since an association is also a member of an object, the association will be stored in the database as well. This means that if you create an order line inside an order and the order line is the parent of the association, when you commit the order line, the order will be autocommitted.

{{% alert color="warning" %}}
An autocommit is not the same as an explicit commit!

If a rollback is triggered for any reason (for example, if the user session is terminated by the user closing the browser), then autocommitted objects will be deleted from the database. See [Persistability](/refguide/persistability/) for more information about how Mendix handles persistable objects.
{{% /alert %}}

If you end up with autocommitted objects, it is always because of a modeling error. At some point in time, an association was set to a new object, the associated object was committed, and all of its associations were committed as well to keep all the data consistent.

During commit the the following will occur:

* Events: For *explicitly committed* objects all before and after events are executed, and if any before-rollback event returns false, an exception can be thrown
	* If an exception occurs during an event, all the applied changes are reverted with the default error handling behavior
	* Changes made prior to the commit will be kept
		{{% alert color="warning" %}}Before and after events are not executed for autocommitted objects.{{% /alert %}}
* Database: there is an insert or update query executed both for explicitly committed objects and auto committed objects
	* Depending on the object state, the platform will do an insert for objects with the state **Instantiated** and an update for all other states
* Result: an object with the state Instantiated will be inserted into the database, and an object with any other state will be updated

{{< figure src="/attachments/refguide/modeling/application-logic/microflows-and-nanoflows/activities/object-activities/committing-objects/18582172.png" >}}
