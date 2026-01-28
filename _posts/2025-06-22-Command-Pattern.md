---
title: "Patterns: Command"
description: "Overview and explanation about the Command pattern oriented to videogames"
categories:
  - Game Programming Patterns
  - Learning Content
  - blog
tags:
  - GameProgramming Patterns
  - Educational
  - Learning Content
---

## Book Definition

> Encapsulate a request as an object thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations.
> 

## Wiki definition

> .. An object is used to encapsulate all information needed to perform an action or trigger an event at a later time.
> 

---

Both terms mean taking some concept and turning it into a piece of data (an object) that you can stick in a variable, pass to a function, etc.

Commands are an object-oriented replacement for callbacks.

## Main Code

```cpp
class Command {
	public:
		virtual ~Command() { }
		virtual void execute() = 0;
};
```

Then we create as many subclasses as we need to represent the actions.

```cpp
class JumpCommand : public Command {
	public:
		virtual void execute() { jump(); }
}
```

In our input handler we store a pointer to a command for each needed

```cpp
class InputHandler {
	public:
		void handleInput();
	private:
		Command* buttonX_;
		Command* buttonY_;
}
```

The `InputHandler` class is going to need methods to bind the buttons to each specific command

Then, we can just execute.

```cpp
void InputHandler::handleInput() {
	if (isPressed(BUTTON_X)) buttonX_->execute();
	else if (isPressed(BUTTON_X)) buttonX_->execute();
	else if (isPressed(BUTTON_X)) buttonX_->execute();
	else if (isPressed(BUTTON_X)) buttonX_->execute();
}
```

---

So far we managed to improve the code by making posible the exchange of buttons behavior. But there is more offered by the command pattern.

To make commands undoable, we define another operation that each command class needs to implement.

```cpp
public:
	virtual void undo();
```

This function reverses the game state changed by the corresponding `execute()` method. In order to use the `undo()` function properly, we need to save the game state changed, like a position if moved, a potion name if used, etc.

To let the player undo a move, we keep around the last command executed. When they bang on `Ctrl-Z`, (or whatever) we call those command’s `undo()` method.

---

Supporting multiple levels of undo isn’t much harder. Instead of remembering the last command, we keep a list of them. A linked list where you can move forward or backwards depending if the player pressed `undo()` or `redo()`.

<aside>
💡 Some commands are stateless chinks of pure behavior. In cases like this,k having more than one instance of that class wastes memory.  Flywheight pattern addreses this.

</aside>

---

## Video Script

> Hello! This is Game Programming Patterns 101 and this video is about the Command Pattern.
> 

`[ Display the thumbnail ]`

> Going straight to the point, there’s two definitions we can use:
1. The original one, from the gang of four original book, where almost all known patterns were first explained formally.
2. And the one offered by wikipedia because this is the internet.
> 

`[ Show logos ]`

## Book Definition

> As the book defines: quote. Encapsulate a request as an object thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations. end quote
> 

## Wiki definition

> And as wikipedia defines: quote .. An object is used to encapsulate all information needed to perform an action or trigger an event at a later time. end quote
> 

`[ Read Both ]`

> **Ok, since these aren’t the clearest definitions ever, let’s talk about it.**
> 
- `[ Show bullet 1 ]`
Both terms mean taking some concept and turning it into a piece of data (an object) that you can stick in a variable, pass to a function, etc.

> Both definitions mean taking some concept and turning it into a piece of data, an object in our case, that you can stick in a variable, pass to a function, store or whatever.
> 

> To picture it better, we can make an example.
> 

`[ Display this code ]`

```cpp
void InputHandler::handleInput()
{
	if (isPressed(BUTTON_X)) jump();
	else if (isPressed(BUTTON_Y)) fireGun();
	else if (isPressed(BUTTON_A)) swapWeapon();
	else if (isPressed(BUTTON_B)) lurchIneffectively();
}
```

> Imagine we are handling our inputs like this. Not too flexible since the mapping of buttons and actions is hardcoded and once the action was made, it disappear, no record, no nothing.
> 

`[Display the class command]`

```cpp
class Command {
	public:
		virtual ~Command() { }
		virtual void execute() = 0;
};
```

> To make it better, we can add something in between the handler function and the ultimate action.
> 

> Introducing the core concept of the command pattern. An empty abstract class that’s gonna be used to create our commands inheriting from it. Like, for example, `Jump()`
> 

`[Display the Jump() command]`

```cpp
class JumpCommand : public Command {
	public:
		virtual void execute() { jump(); }
}
```

> The JumpCommand class inherits from the base one, Command.
> 

> Something we’re going to have in all commands is the `execute()` function. In each derived class, we’re going to implement the action we really need in there.
> 

`[Display the Input Handler Class]`

```cpp
class InputHandler {
	public:
		void handleInput();
	private:
		Command* buttonX;
		Command* buttonY;
		...
}
```

> In our Input Handler class that we saw before, we can store a Pointer to a command for each button we have, then, the buttons can be mapped to different command clases, maybe to nothing, or even be set and changed later by the player.
> 

`[Display the handleInput with the commands]`

```cpp
void InputHandler::handleInput() {
	if (isPressed(BUTTON_X)) buttonX->execute();
	else if (isPressed(BUTTON_Y)) buttonY->execute();
	else if (isPressed(BUTTON_A)) buttonA->execute();
	else if (isPressed(BUTTON_B)) buttonB->execute();
}
```

> All we need to do after is to access those pointers to call each `execute()` function in the input handler. Since all of them are commands in their base class, we can let the magic of inheritance do the trick.
> 

`[Display “Undo and redo” text in screen]`

> So far, we managed to improve the code by making possible exchanging the behavior of the buttons; but there’s more useful things we can do with this pattern.
> 

> A common case, is to implement a Redo operation. To make the commands undo or redo, we need to define another operation in our command base class.
> 

`[Display the Command with undo()]`

```cpp
class Command {
	public:
		virtual ~Command() { }
		virtual void execute() = 0;
		virtual void undo() = 0;
};
```

> This new function reverses the state changed by the corresponding `execute()` function. In order to use the `undo()` function, we need to save the needed information inside our child command class. Information like the original position if moved, a potion name if it was used, and things like that.
> 

```cpp
class MoveUnitCommand : public Command {
		Unit* ThisUnit;
		int Pos_X, Pos_Y;
		int Prev_X, Prev_Y;
		
	public:
		MoveUnitCommand(Unit* NewUnit, int X, int Y) :
			ThisUnit(NewUnit), Pos_X(X), Pos_Y(Y), Prev_X(0), Prev_Y(0)
			
		virtual void Execte();
		virtual void Undo();
}
```

`[Display the execute() with position]`

```cpp
virtual void execute() {
	// Remember the unit's position before the move so we can restore it.
	Prev_X = ThisUnit->GetPosition().X
	Prev_Y = ThisUnit->GetPosition().Y;
	ThisUnit->moveTo(Pos_X, Pos_Y);
}

virtual void undo() {
	ThisUnit->moveTo(Prev_X, Prev_Y);
}
```

> This example considers a class where we have values to store the position before the execution. To let the player undo a move, we keep around the last command executed.
When the player bangs `ctrl+Z` (or whatever) we call that command’s `undo().`
> 

`[Display commands list graphic]`

![IMG_0497.jpeg](IMG_0497.jpeg)

> After all this, supporting multiple levels of `undo()` and `redo()` is not much more harder. Instead of remembering the last command, we can keep a list of them. To `redo()` or `undo()` we just do the same than before but moving across the list instead.
> 

`[Display thunbnail]`

> With this little example we can have a pretty much operational implementation following the Command Pattern. But as a final thought: Notice that some commands are just stateless junks of pure behavior. In cases like this, having more than one instance of that class can waste memory. Flywheight pattern addresses that and we are going to review it in the following videos.
> 

> Stay tuned for that! Like subscribe an everything. BIE!
> 

# Chat GPT script improvement

Title: Game Programming Patterns 101: Understanding the Command Pattern

Introduction:
Hello, fellow game developers! Welcome back to Game Programming Patterns 101. In this video, we're going to dive into the Command design pattern. Before we begin, let's explore two different definitions of the Command pattern and understand how it can benefit our game development.

Definition of Command Pattern:
The Command pattern involves encapsulating a request as an object, allowing us to parameterize clients with different requests, queue or log requests, and support undoable operations. Simply put, it transforms actions into objects, making them more versatile and easier to manage.

Example of the Command Pattern:
Let's start by looking at a practical example. Imagine we have an InputHandler class responsible for handling user inputs. Currently, our code looks like this:

```cpp
cppCopy code
void InputHandler::handleInput()
{
    if (isPressed(BUTTON_X)) jump();
    else if (isPressed(BUTTON_Y)) fireGun();
    else if (isPressed(BUTTON_A)) swapWeapon();
    else if (isPressed(BUTTON_B)) lurchIneffectively();
}
```

Challenges without the Command Pattern:
This approach is inflexible and does not support undo and redo functionalities for user inputs. To overcome these challenges, we'll introduce the Command pattern.

Introducing the Command Class:
The core of the Command pattern is the Command class, an abstract class with a pure virtual function called **`execute()`**. This class acts as the base for all our specific commands.

```cpp
cppCopy code
class Command
{
public:
    virtual ~Command() { }
    virtual void execute() = 0;
};
```

Creating Specific Commands:
Let's create specific commands like JumpCommand that inherit from the Command class and implement their respective **`execute()`** functions.

```cpp
cppCopy code
class JumpCommand : public Command
{
public:
    virtual void execute() override { jump(); }
};
```

Using the Command Pattern:
In the InputHandler class, we store pointers to different commands for each button. This way, we can easily map buttons to specific command classes or allow players to customize their controls.

```cpp
cppCopy code
class InputHandler
{
public:
    void handleInput();

private:
    Command* buttonX_;
    Command* buttonY_;
};
```

Implementing Undo and Redo:
To support undo and redo functionalities, we need to extend the Command class with an additional **`undo()`** function. This function will reverse the state changes made by the corresponding **`execute()`** function.

```cpp
cppCopy code
class Command
{
public:
    virtual ~Command() { }
    virtual void execute() = 0;
    virtual void undo() { }
};
```

Conclusion:
With the Command pattern, we've improved the code by making it more flexible and enabling undo and redo for user inputs. Remember, the Command pattern is incredibly valuable for complex systems with multiple levels of undo and redo functionalities.

Outro:
In the next video, we'll explore the Flyweight pattern, which addresses memory efficiency concerns when dealing with stateless behavior objects. Stay tuned for more exciting insights into Game Programming Patterns!

BIE (Bye in Excitement)!
