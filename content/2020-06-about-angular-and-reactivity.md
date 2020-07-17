# About Angular and Reactivity

While working with Vue for several years, I got used to several things. I immediately noticed that after switching to Angular for a new project. One of those things was how reactivity works. This is what I learned.

## Data Binding
The data binding part is pretty similar. There are template expressions {{}} to display data, props [] to pass data to a child component, event listener () to react to child events and a combination of props and events, called two way data binding [()].

## Change Detection
This is where it gets interesting. Angular works using change detectors, those little things who (when called) decide if a component got changed or not. Every component gets one of these on startup.

### Component isChanged
A component isChanged if a value of a bound variable (used in a template expression) changed its value.

### Trigger change detection
In default mode a change detection is triggered for browser events, setTimeout/Interval, ajax requests and websockets. So basically most of the things that change data. But keep in mind that there can be more things that change data but do not trigger an event, like events from indexedDB.

Once the change detection got triggered, it will run every change detector for every component, starting at the root node. Then DOM updates will be rendered in a batch.

## Default change detection
As default this just works, most of the time. It also works for changes within deeply nested objects, if those object properties are bound to template expressions. Modifying object properties will not work for

## OnPush change detection
This mode can be activated for every component individually. You will have more control about how change detection works. This can improve the performance of components as you do not have to run every change detector of every component for every event. But you will also have to take care of some things yourself, so chose wisely.

### Trigger onPush change detection
OnPush change detection will be triggered for each component independently in the following cases:

 - The **inputs** of this component changes. When having an object as input, its value needs to change. Changing an object property is not enaugh, you need to pass a new object. This is why immutable.js can help to avoid bugs.

 - Something within the component fires an **event**. This one is simple, change detection will still be triggered for events, but only events of this component.

 - A bound **observable** within this component fires an event. OnPush change detection will also notice if an subscribed observable gets a new update. But only if this observable got subscribe using the | async syntax within a template expression.

 You can also mark a component with onPush change detection as changed manually using `ChangeDetectorRef.markForCheck()`. This will check your component for changes even if non of the above happened.

### Rendering
When using immutable.js or another way to generate unique object references, Angular will have a harder time to render your DOM in some cases. When looping over objects, Angular keeps track of every object reference and its according DOM node. This way Angular knows which DOM nodes to update when the list changes. Using immutable objects may break those references and require the DOM to rerender all the time. You can avoid this by using a unique object ID in loops using `trackBy: trackByFunction`.

## Manual Change Detection
 If you want to take over the change detection completely for a component, you can inject a reference in your constructor: `constructor(private ref: ChangeDetectorRef){}` and disable it using `ref.detach()`. From now on you have to call `this.ref.detectChanged()` to detect changes within your component.

Another way to disable change detection for a specific part of you code would be to inject `private ngZone: NgZone` and run your code within this wrapper `this.ngZone.runOutsideAngular(()=>{ /*your code*/ })`.

## Debugging Change Detection
When running into change detection problems you will almost always debug whats happening first. One easy way to do this is to add a debug function within template expressions {{ check() }}. Everytime the change detection runs and checks your template expressions, your function will be called. You can `console.log('change detection running')` to get notified.
