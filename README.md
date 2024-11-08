# GLSP
Eclipse GLSP (Graphical Language Server Platform) is a client-server framework for building web-based diagram editors and is core for developing languages in cinco cloud.

One important thing to know is you interact with these glsp via actions.
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
