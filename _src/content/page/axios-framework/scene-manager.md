---
title: "Scene Manager"
date: 2018-06-22T14:36:52+02:00
subtitle: "Simple, a manager for all our scenes"
tags: 
    # - example
bigimg: 
    # - {src: "path", desc: "description"}
comments: true
---
A `Scene` is a level or a section. A certain moment in the program where a certain group of objects are loaded. The main menu is a scene. The level is a scene. `Scenes` are managers of objects.
<!--more-->
 
## Documentation
<!-- > Github files: [Header](https://github.com/antjowie/Axios-framework/blob/master/include/Axios/SceneManager.h) and [Source](https://github.com/antjowie/Axios-framework/blob/master/src/Axios/SceneManager.cpp) -->
> Github files yet to be added

## Design choices
Jump to a certain section:

- [Runtime data containers](#runtime-data-containers)
- [Object updating](#object-updating)
- [Object reference handling](#object-reference-handling)
- [Event handling](#event-handling)

#### Runtime data containers
The runtime data containers (one/two frame data container and dynamic data container) are used by objects. Because they only apply to objects are they not included in the `Assets Manager`.  
The runtime data containers are kept in the `Scene Manager`. objects can access these containers via the reference they have to the `Scene`. The `Scene` holds a reference to the `Scene Manager` to push new `Scenes` and access the runtime data containers.  
The user can specify the number of bytes the runtime containers can dynamically allocate. The **new** keyword is expansive, and during runtime object may need to dynamically allocate some memory. In that case, the runtime data containers are used. 

#### The difference between the two containers
There are two data containers (one/two frame data container and dynamic data container). The difference is small. The one/two frame data container contains data that will live for two frames. Think about event data of internal calculations needed for the next frame. The data will be destructed automatically, so pointers become invalid very easily.  
The dynamic data container contains data that will live for the whole lifetime of the `Scene`. They can be manually deallocated and are deallocated when the `Scene` ends. This container contains data that has to last longer than the next frame, like bullets from a gun or a flying arrow.  

#### Object updating
On paper it sounds easy, just call a virtual update function with the time and let an object do its thing. While this can work for simple games, things tend to get rough when objects depend on each other (or have references to each other) see the following examples.  
When updating objects. The order can be very important. For example. Imagine a player position that is being updated by the car he is inside of. If the player is updated before the car, the player's position will be wrong because, at that moment in the game, the car hasn't moved yet. To help with this problem a simple priority system can be used. It goes by the name of buckets.  
If we let an object update itself, it gets a lot of freedom. It will probably call all of its `Components` ([read more about objects and `Components` here] Yet to make the page). And so will the next object, and the next one. This causes the CPU to jump around doing the same calculations. It is far more efficient for a CPU to do an update of the same component at one time. This process is called `Batched Updates`.  
At last, virtual functions can be expansive. Some objects don't even need to update every frame. It also means that every class that has to be updated or is dependent on subsystems in some way be an object. It is way better to make callback function hooks to the subsystems. This way, CPU cycles won't be wasted iterating over classes that don't even need to be updated.

So how do we update?
I'll write about it soon 

#### Object reference handling
Objects are dependent on one another, so they need to be able to reference each other. A simple way could be to request a reference to an object. But what will happen if the object is destroyed and we try to access it?  
This is why all references are wrapped in a simple checker class. It works as follows. 

There are two classes. ObjectReferenceManager(ORM) and ObjectReference(OR).  
The ORM can be seen as a database containing all the references and the OR can be seen as a key to get the reference.  

Every object gets assigned a unique ID. This ID is not visible for the developer. The OR class contains two values, the unique ID of the object, and the index in the ORM database. When an object wants to construct a reference to an object. It passes the object into the constructor of an OR instance.  
When a user wants to get the object, it passes the OR into the ORM. The OR checks the index, if the unique ID is still the same, the OR will return a **pointer** to the object, else it will return a **nullptr**.

With this design, the object can check if the references object still exists. 

#### Event handling
I'll write about this soon