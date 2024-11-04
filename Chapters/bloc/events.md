## Event handling

In graphical applications, whenever a user interacts with the application (BlElements),
an event is said to have been occurred. For example, clicking on a button, moving
the mouse, entering a character through keyboard, selecting an item from list, etc.
are the activities that causes an event to happen.

Bloc provides support to handle a wide variety of events. The class named
*BlEvent* is the base class for an event. An instance of any of its subclass
is an event. Some of them are are listed below.

- **Mouse Event**  − This is an input event that occurs when a mouse is clicked. It includes actions like mouse clicked, mouse pressed, mouse released, mouse moved, etc.
- **Key Event** − This is an input event that indicates the key stroke occurred on an element. Those events includes actions like key pressed, key released and key typed.
- **Drag Event** − This occurs when an Element is dragged by mouse. It includes actions like drag entered, drag dropped, drag entered target, drag exited target, drag over, etc.

Event Handling is the mechanism that controls the event and decides what should
happen, if an event occurs. This mechanism has the code which is known as an 
event handler that is executed when an event occurs.

Bloc provides handlers and filters to handle events. Every event has:

- Target − The element on which an event occurred.
- Source - Used in drag&drop to identify the source element.
- Type − Type of the occurred event

In this example, the background of the element will change when the mouse will
enter or leave the element.

```smalltalk
BlElement new 
  background: Color white; 
  border: (BlBorder paint: Color black width: 2); 
  size: 300 @ 200;
  addEventHandlerOn: BlMouseEnterEvent do: [ :anEvent | anEvent consume. anEvent currentTarget background: Color veryVeryLightGray];
  addEventHandlerOn: BlMouseLeaveEvent do: [ :anEvent | anEvent consume. anEvent currentTarget background: Color white ]; 
  openInNewSpace 
```

### About event bubbling

We should check `example_mouseEvent_descending_bubbling`
*This example exists only in Toplo for now, please load Toplo to your image to try it*

![event propagation](figures/EventPropagation.png)

![Windows nested in each others in Toplo.](figures/4windows.png)

#### Event Capturing Phase

This event travels to all elements in the dispatch chain (from top to bottom), 
starting from *BlSpace*. If any of these element has a handler for this event, 
it will be executed, until it reach the target element which can process it.

[Enzo : I would change the word 'handler' above for 'filter' because filters get the event before handlers and so before it reaches the target element]

#### Event Bubbling Phase

In the event bubbling phase, the event is travelled from the target element to 
BlSpace (bottom to top). If any of the element in the event dispatch chain has a 
handler registered for the event, it will be executed. When the event reaches 
the BlSpace element the process will be completed.

##### stop event propatation

You can stop the event propagation in an event handler by adding
`anEvent consume`

##### prevent event capture

Sometimes, you don't want your element to capture certain events. There is an option to
forbid mouse events for an element. You just send `#preventMouseEvents` to it.
`child2 := BlElement new size: 200 asPoint; position: 200 asPoint; border: (BlBorder paint: Color blue width: 2);preventMouseEvents.`

You can also prevent element and its children to capture event, with `#preventMeAndChildrenMouseEvents`
message, or apply it only to its children with `#preventChildrenMouseEvents`

For now, you can only prevent mouseEvents ,including dragStartEvent that needs a mouseDownEvent to be sent. However, if you start a drag, you can send `#preventMouseEvents` and still have events such has dragEvent and dragEndEvent, more about this case later. 

#### Event Handlers

Event handlers are those which contains application logic to process an event. 
An element can register more than one handler.

##### Simple case for BlElement.

1. use method: `BlElement>>addEventHandlerOn:do:`
2. anEventClass can be a subclass of `BlUIEvent`

**Note**
`addEventHandlerOn:do:` returns the new handler so that we can store to remove 
it in case. **Add a `#yourself` send after to return a BlElement.**

##### Complex case - reusing event handling logic with an event Handler

Instead of using addEventHandlerOn:do: you can also see users of `addEventHandler:`.
An event handler can manage multiple element at once, by overriding the method `eventsToHandle`.
This implies that you define a new subclass of `BlCustomEventHandler` for your handler.

This example is taken from `BlPullHandler` which demonstrate how you can drag around an
element (more on this at the end of this chapter)

```smalltalk
eventsToHandle
^ {	BlDragStartEvent. BlDragEvent. BlDragEndEvent }
```

You then add your event handler to your bloc element with method `addEventHandler:`.
This allows complete flexibility.

You can also declare dynamically event handler on specific event.

```smalltalk
BlEventHandler on: BlClickEvent do: [ :anEvent | self inform: 'Click!' ]
```

As a more general explanation, all UI related events can be controlled. Have a
look at `BlElementFlags` and `BlElementEventDispatcherActivatedEvents` and how
these classes are used.

```smalltalk
deco addEventHandler: (BlEventHandler on: BlMouseLeaveEvent
      do: [ :event | event currentTarget border: BlBorder empty ]).
```

#### event filters

You can also add event filter:

`addEventFilterOn:do:` returns the new filter so that we can store to remove it in case.
Event filters receive events before general event handlers. Their main goal is
to prevent some specific events from being handled by basic handlers. For that
custom filters should `consume` event to instantly stops propagation.
You can also use filters to do some actions before the event reaches your target element.
In this case, consuming the event is not recommanded

In the example below, the element will catch `BlMouseEnterEvent`. If you uncomment
`anEvent consume`, you'll only have the filtered version. If the event keep
propagating, both will be called, filter and then handler.

```smalltalk
addEventFilterOn:  BlMouseEnterEvent do: [ :anEvent | "anEvent consume". self inform: 'event filter'];
addEventHandlerOn:  BlMouseEnterEvent do: [ :anEvent | "anEvent consume". self inform: 'event handler' ];
```

#### event tips

##### Event inheritance.

 `BlPrimaryClickEvent`, `BlMiddleClickEvent` and `BlSecondaryClickEvent` are all subclasses of `BlClickEvent`. In Bloc logic, event handler will look for event that are from the defined event class, or inherit from its  parent (exact code is ` anEvent class == self eventClass or: [ anEvent class inheritsFrom: self eventClass ] `). For example, If you define handler for both `BlClickEvent` and `BlPrimaryClickEvent` on your element, and you left click on it, it will raise `BlPrimaryClickEvent`. Because `BlPrimaryClickEvent`inherit from `BlClickEvent`, both will be handled.

In the example below, 'click' will be raised, whatever the mouse button you use to click on your element. 
```smalltalk
elt := BlElement new extent: 200@200; border: (BlBorder paint: (Color black) width: 3 ); background: (BlBackground paint: Color blue).

elt addEventHandlerOn: BlClickEvent do: [  self inform: 'click' ].
elt addEventHandlerOn: BlPrimaryClickEvent do: [  self inform: 'Primary' ].
elt addEventHandlerOn: BlSecondaryClickEvent do: [ self inform: 'secondary' ].
elt addEventHandlerOn: BlMiddleClickEvent do: [ self inform: 'middle' ].

elt openInNewSpace 
```

A similar effect can be achieve with:
```smalltalk
elt := BlElement new extent: 200 @ 200; border: (BlBorder paint: Color black width: 3); background: (BlBackground paint: Color blue).
elt addEventHandlerOn: BlClickEvent do: [ :evt | evt primaryButtonPressed ifTrue: [ self inform: 'primary' ] ].
elt addEventHandlerOn: BlClickEvent do: [ :evt | evt secondaryButtonPressed ifTrue: [ self inform: 'secondary' ] ].
elt addEventHandlerOn: BlClickEvent do: [ :evt | evt middleButtonPressed ifTrue: [ self inform: 'middle' ] ].
elt openInNewSpace
```

The same example can also be written like this : 
```st
elt := BlElement new extent: 200 @ 200; border: (BlBorder paint: Color black width: 3); background: (BlBackground paint: Color blue).
elt addEventHandlerOn: BlClickEvent do: [ :evt |
	evt 
	ifPrimary: [ self inform: 'primary' ]
	secondary: [ self inform: 'secondary' ] 
	middle: [ self inform: 'middle' ] 
	other: [ self inform: 'other' ]].
elt openInNewSpace
```

In the first way, you tell explicitely which mouse button event you want to catch. In the second and the third, you have to filter it in the handler action.

##### Remove all eventHandlers from a BlElement?

```smalltalk
el removeEventHandlersSuchThat: [:e|true] 
```

or

```smalltalk
el eventDispatcher removeEventHandlers
```

##### using mouse wheels

In this snippet, we'll use the mouse wheel to scale up or down the element

```smalltalk
scaleFactor := 0.
surface addEventHandlerOn: BlMouseWheelEvent
        do: [ :anEvent | anEvent consumed: true.
              anEvent isScrollDown ifTrue:  [ scaleFactor := scaleFactor- 0.5 ].
              anEvent isScrollUp ifTrue:  [ scaleFactor := scaleFactor + 0.5 ].
              elt transformDo: [ :t | t scaleBy: scaleFactor].].
```

##### forbid mouse events on an element

There is an option to forbid mouse events for an element. You just send `preventMouseEvents` to it

```smalltalk
container := BlElement new size: 500 asPoint; border: (BlBorder paint: Color red width: 2).

"#addEventHandlerOn: do: returns the new event handler.  add a #yourself send after"
child1 := BlElement new size: 300 asPoint; background: Color lightGreen; position: 100 asPoint; addEventHandlerOn: BlClickEvent do: [ self inform: '1' ]; yourself .

"There is an option to forbid mouse events for an element. 
You just send #preventMouseEvents to it."
child2 := BlElement new size: 200 asPoint; position: 200 asPoint; border: (BlBorder paint: Color blue width: 2);preventMouseEvents.

container addChild: child1.
container addChild: child2.

container openInSpace.
```

### Keyboard event handling -  Combination from Bloc framework

Bloc come with its own keymapping framework.
BlShortcutWithAction would be the equivalent of KMKeymap.

Shortcut represents a keyboard shortcut that can be registered to any arbitrary BlElement.
Shortcut consist of an Action that is evaluated when a Shortcut is triggered and
BlKeyCombination that describes when shortcut should be triggered. A combination
is a logical formula expression that is composed of various key combinations
such as alternative, compulsory or single key. See subclasses of BlKeyCombination.
Additionally, shortcut may provide its optional textual description and name.

A shortcut can be added or removed from the element by using `BlElement>>#addShortcut:`
or `BlElement>>#removeShortcut:` methods. BlElement>>#shortcuts message can be
sent to an element in order to access a list of all registered shortcuts.

BlShortcutWithAction extend BlBasicShortcut with ability to specify a runtime
action that should be evaluated when shortcut is performed. In addition to that,
shortcuts with action allow users to customise the name and description of the shortcut.

```smalltalk
BlShortcutWithAction new
    combination: (BlKeyCombination builder alt; control; key: KeyboardKey C; build);
    action: [ flag := true ].
```

```smalltalk
addShortcut: (BlShortcutWithAction new
      combination: (BlKeyCombination builder shift; meta; key: BlKeyboardKey arrowLeft; build);
      action: [ :anEvent :aShortcut | self inform: 'Triggered ', aShortcut combination asString ]);
```

using low level event

```smalltalk
	space addEventHandlerOn: BlKeyDownEvent
			 do: [ :evt |
				 (evt key = KeyboardKey altLeft or: [
					  evt key = KeyboardKey altRight ]) ifTrue: [
					 self inform: 'source 1 alt key pressed' ] ].
```

Be careful, if you add a handler for keyboardEvents to a `BlElement`, you should send it the message `requestFocus` for it to receive these events.

```st
elt := BlElement new size: 100 asPoint; background: Color blue; position: 100 asPoint.
elt addEventHandlerOn: BlKeyDownEvent
			 do: [ :evt |
				 (evt key = KeyboardKey altLeft or: [
					  evt key = KeyboardKey altRight ]) ifTrue: [
					 self inform: 'source 1 alt key pressed' ] ].			
elt requestFocus.
elt openInNewSpace 
```

#### Keymap at system platform level

`KeyboardKey` class is used when a key on the keyboard is pressed.

It's used ultimately by BlKeyCombinationBuilder to build keyboard shortcut
in bloc. It's also used to convert key from event by BlOSWindowEventHandler.

### Drag and Drop

Explore BlBaseDragEvent and subclasses.

Full drag&drop example

2 hints:
- to allow drop event to reach other element, you have to sent `preventMeAndChildrenMouseEvents` to dragged element.
- To allow dragged element to stay on top of the other, you must 1. remove it `removeFromParent` and 2. add it to the top most element. `space root addChild:` 
  
```smalltalk
| source1 source2 target space offset |
space := BlSpace new.
source1 := BlElement new size: 100 @ 100; background: Color red; border: (BlBorder paint: Color gray width: 2); position: 0@0.
source2 := BlElement new size: 100 @ 100; background: Color purple; border: (BlBorder paint: Color gray width: 2); position: 150@0.
target := BlElement new size: 100 @ 100; background: Color blue; border: (BlBorder paint: Color gray width: 2); position: 500 @ 500.

space root addChildren: { source1. source2. target. }.

source1 addEventHandlerOn: BlDragStartEvent do: [ :event | event consumed: true. self inform: 'source1 BlStartDragEvent'. 
	offset := event position - source1 position. 
	source1 removeFromParent.
	source1 preventMeAndChildrenMouseEvents].
source1 addEventHandlerOn: BlDragEndEvent do: [ :event | event consumed: true. self inform: 'source1 BlDragEndEvent'. 
	source1 allowMeAndChildrenMouseEvents ].
source1 addEventHandlerOn: BlDragEvent do: [ :event | event consumed: true. "self inform:  'source1 BlDragEvent'."
	source1 position: event position - offset.
	source1 hasParent ifFalse: [space root addChild: source1]].

source2 addEventHandlerOn: BlDragStartEvent do: [ :event | event consumed: true. self inform: 'source2 BlStartDragEvent'. 
	offset := event position - source2 position.
	source2  removeFromParent.
	source2 preventMeAndChildrenMouseEvents].
source2 addEventHandlerOn: BlDragEndEvent do: [ :event | event consumed: true. self inform: 'source2 BlDragEndEvent'. 
	source2 allowMeAndChildrenMouseEvents ].
source2 addEventHandlerOn: BlDragEvent do: [ :event | event consumed: true. "self inform:  'source BlDragEvent'."
	source2 position: event position - offset.
	source2 hasParent ifFalse: [space root addChild: source2] ].

target addEventHandlerOn: BlDropEvent do: [ :event | event consumed: true. self inform: 'target BlDropEvent'.
	event gestureSource background paint color = (Color red)
			ifTrue: [ self inform: 'drop accepted' ]
			ifFalse: [ self inform: 'drop rejected'. event gestureSource position: 100 @ 400] ].
target addEventHandlerOn: BlDragEnterEvent do: [ :event | event consumed: true. self inform: 'target BlDragEnterEvent' ].
target addEventHandlerOn: BlDragLeaveEvent do: [ :event | event consumed: true. self inform: 'target BlDragLeaveEvent' ].

space show
```

You can catch modifier while dragging your element. This will catch the 'Alt' key
during a drag event.

```smalltalk
elt addEventHandlerOn: BlDragEvent do: [ :event | event modifiers isAlt ifTrue: [self inform: 'alt']].
```

#### BlPullHandler

If you just need to drag an element around, use `BlPullHandler` which is
a specialized event handler.

```smalltalk
parent := BlElement new background: Color lightGreen; size: 600 asPoint.
elt := BlElement new background: Color red; size: 100 asPoint.
parent addChild: elt.

elt addEventHandler: BlPullHandler new disallowOutOfBounds.

parent openInSpace.
```

