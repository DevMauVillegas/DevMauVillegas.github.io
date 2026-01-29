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

# Command Pattern

Hello! This is Game Programming Patterns 101 and this video is about the Command Pattern.

Going straight to the point, there’s two definitions we can use:
1. The original one, from the gang of four original book, where almost all known patterns were first explained formally.
2. And the one offered by wikipedia because this is the internet.

### Book Definition
As the book defines: Encapsulate a request as an object thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations.


### Wiki definition
And as wikipedia defines: .. An object is used to encapsulate all information needed to perform an action or trigger an event at a later time.

Both definitions mean taking some concept and turning it into a piece of data, an object in our case, that you can stick in a variable, pass to a function, store or whatever.

To picture it better, we can make an example.

```cpp
void InputHandler::handleInput()
{
	if (isPressed(BUTTON_X)) jump();
	else if (isPressed(BUTTON_Y)) fireGun();
	else if (isPressed(BUTTON_A)) swapWeapon();
	else if (isPressed(BUTTON_B)) lurchIneffectively();
}
```

Imagine we are handling our inputs like this. Not too flexible since the mapping of buttons and actions is hardcoded and once the action was made, it disappear, no record, no nothing.

```cpp
class Command {
public:
	virtual ~Command() { }
	virtual void execute() = 0;
};
```

To make it better, we can add something in between the handler function and the ultimate action.

Introducing the core concept of the command pattern. An empty abstract class that’s gonna be used to create our commands inheriting from it. Like, for example, `Jump()`


```cpp
class JumpCommand : public Command {
public:
	virtual void execute() { jump(); }
}
```

The JumpCommand class inherits from the base one, Command.

Something we’re going to have in all commands is the `execute()` function. In each derived class, we’re going to implement the action we really need in there.

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

In our Input Handler class that we saw before, we can store a Pointer to a command for each button we have, then, the buttons can be mapped to different command clases, maybe to nothing, or even be set and changed later by the player.

```cpp
void InputHandler::handleInput() {
	if (isPressed(BUTTON_X)) buttonX->execute();
	else if (isPressed(BUTTON_Y)) buttonY->execute();
	else if (isPressed(BUTTON_A)) buttonA->execute();
	else if (isPressed(BUTTON_B)) buttonB->execute();
}
```

All we need to do after is to access those pointers to call each `execute()` function in the input handler. Since all of them are commands in their base class, we can let the inheritance do the trick.

The same result can be obtained by using interfaces, but since the parent class is just empty with minimal implementation, it would be enough.

So far, we managed to improve the code by making possible exchanging the behavior of the buttons; but there’s more useful things we can do with this pattern.

A common case, is to implement a Redo operation. To make the commands undo or redo, we need to define another operation in our command base class or interface.

```cpp
class Command {
public:
	virtual ~Command() { }
	virtual void execute() = 0;
	virtual void undo() = 0;
};
```

This new function reverses the state changed by the corresponding `execute()` function. In order to use the `undo()` function, we need to save the needed information inside our child command class. Information like the original position if moved, a potion name if it was used, and things like that.

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

This example considers a class where we have values to store the position before the execution. To let the player undo a move, we keep around the last command executed.
When the player bangs `ctrl+Z` (or whatever) we call that command’s `undo().`

After all this, supporting multiple levels of `undo()` and `redo()` is not much more conplicated. Instead of remembering the last command, we can keep a list of them. To `redo()` or `undo()` we just do the same than before but moving across the list instead.

With this little example we can have a pretty much operational implementation following the Command Pattern. But as a final thought: Notice that some commands are just stateless junks of pure behavior. In cases like this, having more than one instance of that class can waste memory. Flywheight pattern addresses that and we are going to review it in the following posts.
