---
title: "Patterns: Observer"
description: "Overview and explanation about the Observer pattern oriented to videogames"
sidebar:
  nav: "projects"
categories:
  - Game Programming Patterns
  - Learning Content
  - blog
tags:
  - GameProgramming Patterns
  - Educational
  - Learning Content
---

# Observer Pattern

The Observer pattern is one of the original Gang of Four design patterns, and it is both widely known and widely used.

### Original Programming Patterns definition

> "Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically."

### Wikipedia definition

> "The observer pattern is a design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and automatically notifies them of any state changes by calling one of their methods."

Wikipedia section is a little summarized to reduce text bloating.


So, in a nutshell, this pattern covers the need to send notifications between different objects without coupling them. One object broadcasts a notification, and any listener that cares can receive it and act on that information.

Consider that listeners subscribe only when they actually need that information.

---

Say we're adding achievements to our game; what we'd like is to have all the code concerned with the achievements nicely lumped in one place instead of spreading unlocking checks across all the code base.

That's what the Observer Pattern is for. It lets one piece of code announce that something happened without actually caring who receives the notification.

```cpp
// This example updates the down acceleration of an entity.
void Physics::updateEntity(Entity& entity) {
    bool wasOnSurface = entity.isOnSurface();
    entity.accelerate(GRAVITY);
    entity.update();
    if (wasOnSurface && !entity.isOnSurface()) {
        // Entity was on the ground, but after update is not anymore.
        // We notify that in case anyone cares
        notify(entity, EVENT_START_FALL);
    }
}
```

Above we see how the notification is being sent and under which conditions. It is true that we will need to add a little bit of code into the ```Physics``` section of the game in order to notify, but at least it is decoupled from anyone actually hearing about it.

Let's take a look at the part where someone receives this information.

```cpp
class Observer {
public:
    virtual ~Observer() {}
    virtual void onNotify(const Entity& entity, Event event) = 0;
};
```

Any concrete class implementing this will be an observer.

```cpp
class Achievements : public Observer {
public:
    // onNotify signature is up to you. This example fits the purpose, but other implementations of observers might need different parameters. 
    virtual void onNotify(const Entity& entity, Event event) {
        switch(event) {
        case EVENT_START_FALL:
        if (entity.isHero && bHeroIsOnBridge) {
            unlock(ACHIEVEMENT_FELL_OFF_BRIDGE);
        }
        break;
        // Other events
        }
    }

private:
    void unlock(Achievement achievement) {
        // Unlock if not already unlocked
    }

    bool bHeroIsOnBridge;
};

```

The notification method is invoked by the object being observed. In the original Gang of 4 book, that object is called "The Subject"

```cpp
class Subject {
private:
    Observer* observers_[MAX_OBSERVERS];
    int numObservers_;

private:
    void notify(const Entity& entity, Event event) {
        for (int i = 0; i < numObservers_; ++i) {
            observers_[i]->onNotify(entity, event);
        }
    }

public:
    void addObserver(Observer* newObserver) {
        // Adding observer to the array
        ++numObservers_;
    }

    void removeObserver(Observer* oldObserver) {
        // Removing the observer
        --numObservers_;
    }
}
```

Now, returning to the first ```updateEntity``` example at the top of the page, the ```Physics``` class declaration will look something like:

```cpp
class Physics : public Subject {
public:
    void updateEntity(Entity& entity);
}
```

Then, when the Physics class does something important, it can call the notify function that will explore the observers list and let them know what happened. It's up to the observers to do something with this information.

It is relevant to note that inheriting the ```Subject``` class is not strictly necessary. In this case the ```Physics``` class can hold an instance of the ```Subject``` class to avoid multiple inheritance.

---

### Possible problems with the pattern

#### It's slow and painful

"Events", "Messages" and "Data Binding" might be systems with higher performance overhead than simpler operations. They involve things like queueing or performing dynamic memory allocations.

But no. With this pattern, that is just not the case. Sending notifications is just walking linearly through an array to call the observer's notify function. It could be worrisome if the function is virtual, but in practice it doesn't need to be.

Yes, it might be slower than embedded calls inside the code, but the tradeoff of clean and organized code is higher than an object access even in critically performative sections.

#### It performs too many dynamic allocations

Many game developers have moved into garbage collected languages. Even in C++ environments like Unreal Engine; most of the memory can be managed by the engine. So dynamic allocation isn't the spooky operation it used to be. 

But even in new tech and specially in performance critical software like videogames, memory allocation still matters and it needs to be managed carefully. 

Dynamic allocation takes time, and so does reclaiming that memory and managing fragmentation over time. In real implementations of this pattern the list of observers is most likely to be a list that grows and shrinks over time.

So there's two things we can do to manage this better. Making the observers a linked list and allocating the nodes in a pool at the beginning of the game.

To implement this in our example, first we get rid of the array in the ```Subject``` class and replace it with a pointer to the head of the list of observers.

```cpp
class Subject {
    Subject() : head_(NULL) {}

private:
    Observer* head_;
}
```

Then we extend the ```Observer``` class with a pointer to the next observer in the list.

```cpp
class Observer {
    friend class Subject;
    Observer() : next_(NULL) {}

private:
    Observer* next_;
}
```
We're also making ```Subject``` a friend class so we can easily access observers to add and remove.

Finally, managing observers is no different from managing a normal linked list, like in the computer science classes many of us took.

```cpp
void Subject::addObserver(Observer* newObserver) {
    newObserver->next_ = head_;
    head_ = newObserver;
}
```
---

An improvement to the list implementation can be having, instead of a list of ```Observer```, a list of ```Node``` objects that hold a pointer to an observer.   

![Pattern Picture](/assets/images/Patterns/observer-nodes.png)

Since multiple nodes can all point to the same observer, that means an observer can be in more than one subject's list at the same time. 

The way you avoid dynamic allocation here is simple: since all of those nodes are the same size and type, you pre-allocate an object pool. That gives you a fixed-sized pile of list nodes to work with, and you can use and reuse them as you need without having to hit an actual memory allocator.

---

#### Destroying subjects and observers

It's an observer's job to unregister itself from any subjects when it gets deleted. The observer does know which subjects it's observing, so it's usually just a matter of adding a ```removeObserver()``` call to its destructor.

This is an important safety point: the order in which subjects and observers are destroyed matters. If an observer is deleted while still registered, the subject may later try to notify a dangling pointer. If a subject is destroyed first, observers must be able to handle the fact that their subject is gone.

A common safe strategy is:
- Observers unregister from subjects during their own destruction.
- Subjects either remove themselves from observers or send a final "dying breath" notification before they are destroyed.

If you don't want to leave observers hanging when a subject gives up the ghost, that's easy to fix. Just have the subject send the final "dying breath" notification right before it gets destroyed. That way, any observer can receive that and take whatever action it thinks is appropriate.

Even better is to make observers automatically unregister themselves from every subject when they get destroyed. If you implement the logic for that once in your base ```Observer``` class, everyone using it doesn't have to remember to do it.

A robust implementation also includes a way for observers to detect invalid subjects, or for subjects to clear their observer list during teardown. That removes the need for callers to remember the exact destruction order and makes the pattern safer in larger systems.
