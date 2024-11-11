# Cinco Meta Language For Dummies
When defining a modelling language in cinco, you have to work with two textual languages, the Meta Graph Lanuage (MGL) `*.mgl` and the Meta Style Language (MSL) `.msl`.

## MGL

The mgl is where you define the construct in your graphical language and there are 4 types to know about `graphModel`, `container`, `node` & `edge`.

### `graphModel`
The `graphModel` element is the top level object, which contains all the elements that make up the language.
```
graphModel YourGraphModel {
	iconPath "path/to/icon.png"
	diagramExtension "flow" # the file extension for YourGraphModel
	
	containableElements(*) # replace * with the nodes which make up the graph.
}
```

`graphModel` element parameters:
- `containableElements(...)` - what `nodes`/`containers` the `graphModel` can contain.
- `diagramExtension "..."` - File extension for the graph model file
- `iconPath "..."` - relative path to the icon for the model file.


### `container`
The `container` element is essentially a node in the graph that can contain other nodes.

```
container myContainer {
	style myNodeStyle
	
	containableElements(Node1,Node2)
}
```

`container` element parameters:
- `style` - the `nodeStyle` from the MSL you wish to link to.
- `outgoingEdges(...)` - the type of edge that you connect from this node type, can also set constraints on how many edges etc..
- `incomingEdges(...)` - the type of edge that you connect to this node type with, can also set constraints on how many edges etc..
- `containableElements` - the `node` and/or `container` elements that this `container` can contain.
- `prime` - The entity from another graphmodel you wish for this container to reference - see **Prime References** below.

### `node`
The `node` element is an atomic element, i.e. it cannot contain other elements and is simply a node in the graph.

```
node Node1 {
	style Node1
	incomingEdges(DataFlow[0,1])
}
```

`node` element parameters:
- `style` - the `nodeStyle` from the MSL you wish to link to.
- `outgoingEdges(...)` - the type of edge that you connect from this node type, can also set constraints on how many edges etc..
- `incomingEdges(...)` - the type of edge that you connect to this node type with, can also set constraints on how many edges etc..
- `prime` - The entity from another graphmodel you wish for this container to reference - see **Prime References** below.

### `edge`
The `edge` element is used to define the edges which can be used to connect `node` and `container` elements.
```
edge DataFlow {
	style dataFlow
}
```

`edge` element parameters:
- `style` - the `edgeStyle` from the MSL you wish to link to.

### Style:
Styles in the msl can be defined to accept arguments. If you have arugments, they can be passed in like so, using a parenthesis, like you'd call a function in a programming langauge.
`style nodeStyleName(...)`

### Attributes:
All model elements can have attributes attached to them. Click [here](https://gitlab.com/scce/cinco-cloud/-/blob/main/cinco-cloud-archetype/editor/workspace/languages/attributes/attributes.mgl?ref_type=heads) for an extensive list of all the attributes and their annotations.

#### Attribute Annotations
- `@hidden` -  hidden from the user.
- `@readOnly` - the user can see but not edit
- `@color` - attribute is a color value.

### Modifiers: 
`abstract`: You can make an element abstract, like in OOP. It cannot be conretized but is useful if you have several different nodes that share some of the same properties.
```
abstract container YourAbstractContainer  {
	//your attributes, etc..
}
```
`extends`:  Denoting inheritance -> 
The container / node will inherit all attributes / parameters from the entity it is extending.
```
container YourConcreteContainer extends YourAbstractContainer {
	//your attributes, etc..
}
```

### MGL Annotations:
`@disable(...[create,delete,move,resize,select])`

The disable annotation, prevents the user from doing certain interactions with the annotated entity, such as moving it, etc..
The annotation can be applied to all your defined language elements `graphModel, container, node, edge`.

`@Hook(HookClassName,...HookMethodNames)`
A hook is how you handle any user interaction with the graphical modelling language. For example `postCreate` if you wish to perform some logic after an element is added to the canvas by the user.

`AbstractEdgeHook` for `edge` elements & `AbstractNodeHook` for `node` & `container` elements

```Typescript
export class HookClassName extends AbstractNodeHook {
    override CHANNEL_NAME: string | undefined = 'HookClassName [' + this.modelState.root.id + ']';

    override postCreate(node: Node): void {
    }

    override canSelect(node: Node, isSelected: boolean): boolean {
        // this.log('Triggered canSelect on node (' + node.id + ') - selected: ' + isSelected);

        return true;
    }


    override postMove(node: Node, oldPosition?: Point | undefined): void {

    }

    override postResize(node: Node, resizeBounds: ResizeBounds): void {
    }
}

LanguageFilesRegistry.register(HookClassName);
```



`@AppearanceProvider(MyAppearanceProvider)`
Appearance providers are used to update the style of a model in UI. Let's say for example when the user connect and edge to a node, you wish to change it's color, that would be done via an AppearanceProvider.


```
import { AppearanceProvider, LanguageFilesRegistry, ModelElement } from '@cinco-glsp/cinco-glsp-api';
import {Appearance,AppearanceUpdateRequestAction} from '@cinco-glsp/cinco-glsp-common';

export class MyAppearanceProvider extends AppearanceProvider {


    getAppearance(
        action: AppearanceUpdateRequestAction,
        ...args: unknown[]
    ): ApplyAppearanceUpdateAction[] | Promise<ApplyAppearanceUpdateAction[]> {

        // parse action
        const modelElementId: string = action.modelElementId;
        const element = this.getElement(modelElementId);
        

        return [];
    }
}
// register into app
LanguageFilesRegistry.register(MyAppearanceProvider);
```


`@CustomAction(YourCustomAction,'User Facing Name for Context Menu')`
When the user right-clicks on a specific element in your language, a context menu will appear. Any custom action you implement will appear in that context menu and when they click it will perform that action.

You create a class which inherits from the `CustomActionHandler` class from `'@cinco-glsp/cinco-glsp-api'` to implement the desired behaviour.

The boilerplate for a custom action is seen below:

```Typescript

import { Container, CustomActionHandler, LanguageFilesRegistry, Node } from '@cinco-glsp/cinco-glsp-api';
import { Action, CustomAction } from '@cinco-glsp/cinco-glsp-common';

export class YourCustomAction extends CustomActionHandler {
    // logging channel
    override CHANNEL_NAME: string | undefined = 'UpdateSIB [' + this.modelState.root.id + ']';
    
    override execute(action: CustomAction, ...args: any): Promise<Action[]> | Action[] {
        const csib = this.modelState.index.findNode(action.modelElementId) as Container

		// implement your logic here...

        return [];
    }
    
    override canExecute(action: CustomAction, ...args: unknown[]): boolean | Promise<boolean> {
        const artifact = this.modelState.index.findNode(action.modelElementId) as Container;

        return true;
    }
}

// adds the class to the registry so it is available for use.
LanguageFilesRegistry.register(YourCustomAction);
```

...

## MSL
The MSL is where you describe how the elements in your graphical lanugage should look, somewhat like what `css` is to `html`.

From the with-in the `.mgl` you link to it's accompanying MSL using:
`stylePath "yourgraphmodel.msl"`

### `appearance`

An appearance entity is essentially a container for some presets, so you don't have to explicitly define them everywhere. Here's how you define one:
```
appearance labelFont {
    foreground (0, 0, 0)
	font ("Sans",16)
}
```
### `nodeStyle`
A `nodeStyle` entity is what you use to define how a `node` or `container` should look.
You can define `shapes`, `images` and `text`.
`size` & `position` are properties of note here.

`nodeStyle` can accept arguments from the mgl, the number of arguments is defined with a number `myNodeStyle(3)` - myNodeStyle accepts 3 arguments. Then they are referenced using this notation - `"%1$s"` for the 1st positional argument, and so on.
```
nodeStyle myNodeStyle(3) {
	rectangle  {
		appearance default
		size (236,50)
		image {
			position (LEFT 8, MIDDLE)
		 	size (fix 24, fix 24)
		 	path ("%1$s")
		}
		text {
			appearance labelFontItalics // here you reference the appearance
			position (LEFT 40, TOP 7)
		 	value "%2$s"
		}
		text {
			appearance labelFont // here you reference the appearance
			position (LEFT 40, TOP 24)
		 	value "%3$s"
		}
	}
}
```

### `edgeStyle`

`edgeStyle` is essentially the same as `nodeStyle` but for edges.

```
edgeStyle controlFlow(1) {	
	appearance controlFlowAppearanceLine
	decorator {
		location (0.98) // at the end of the edge
		ARROW
		appearance controlFlowAppearanceArrowHead
	}
	decorator {
		movable
		text {
			appearance labelFontWhite
		 	value "%s"
		}
		location (0.5) // at the middle of the edge
	}
}
```

## Prime References

In the case you have two modelling languages and you wish to reference elements from `modelling language a` in `modelling language b`, you achieve it with what are known as prime references.

You import the mgl you wish to reference from into the current MGL.
`import "primelib.mgl" as primelib`

```
container primeExample {
	style primeExampleStyle
	
	prime primelib::Node as Node
}
```

You then set the prime parameter using the following `prime <namespace>::EntityName` syntax. This then makes the elements of type `Node` from "primelib.mgl" available in the your current MGL and you can access their attributes, etc..

# GLSP
Eclipse GLSP (Graphical Language Server Platform) is a client-server framework for building web-based diagram editors and is core for developing languages in cinco cloud.

One important thing to know is you interact with these glsp methods via actions.
**You create an action and then dispatch that action.** Which is then handled by the server asynchronously.

If class method you are overriding has return type `Promise<Action[]> | Action[]`, you simply need to return the actions() from the method `return [Action1,Action1]` or `return yourListOfActions`

If the method does not return an actions you can used the `actionDispatch` method to dispatch the action.
`this.actionDispatcher.dispatch(action)` - `Action` a single action object
`this.actionDispatcher.dispatchAll(actions)` - `Action[]` an array of action objects


The main `glsp` modules that I encountered were:
# `@eclipse-glsp/server` 
And the operations that I used were:
## `CreateNodeOperation` 
Create an instance of a node in the model on the canvas.
*Note*: if your node has attributes you wish to set, this will not work as `CreateNodeOperation` does not support passing attributes in.

If you need to create a node with attributes the `Node` class from the `@cinco-glsp/cinco-glsp-api` module supports that use case. Where you are directly editing the  model.

Here is a example of adding a port to a container.
```Typescript
// csib is a container sib 
let n = new Node() // create the node like any objects
n.type = 'mglname:nodename'; // set the node type
n.initializeProperties() // very important to do this.
// set where it is located with-in the parent
n.position = { x: 0, y: 0 } 
// set the size (in the case)
n.size = { width: csib.size.width, height: 20 }
// this is a work around
// the properties are stored in a map _attributes
// here I am directly setting them to my desired values
n['_attributes'] = {
	attr1 : 'value'
	...
}
// add the node to the container containments array
csib.containments.push(n)
```
**One thing to note with this approach:** This will not trigger a GUI update, so the model will have change (i.e. the node has been added to the container), but the GUI will not update automatically if you implement it with-in a method you implemented in a class that inherits from `AbstractNodeHook`.



# `'@cinco-glsp/cinco-glsp-common'`
## `ValidationResponseAction`
& also... `ValidationRequestAction, ValidationStatus, ValidationMessage`
If you are doing any form of static analysis of the model (graph) that the user is implementing you will encounter these.

`ValidationHandler` from the `@cinco-glsp/cinco-glsp-api` module is the class you inherit from to implement your analyser tool.

A `ValidationResponseAction[]` is what is returned from the `execute` method of your validation class and I've typically only returned an array with a single item. The main thing to note is that `ValidationResponseAction` contains an array of `ValidationMessage`. In my case I've only returned an array containing only 1 `ValidationResponseAction` and put all my `ValidationMessage`'s into it. 
The rationale behind this is, when something is correct I only want a single message displayed when things are ok (i.e. Dataflow is valid), rather than each a message for each individual thing that is correct.
![[Pasted image 20241108164759.png]]


# `'@cinco-glsp/server'`
## `DeleteElementOperation`
This works pretty much as expected. Just add in the array of ids, you wish to delete and dispatch the action.
