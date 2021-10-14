# Angular

## Component interaction Patterns

### Intercept input property changes with a setter
Use an input property setter to intercept and act upon a value from the parent.

```js
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-name-child',
  template: '<h3>"{{name}}"</h3>'
})
export class NameChildComponent {
  @Input()
  get name(): string { return this._name; }
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }
  private _name = '';
}
```

### Parent interacts with child using local variable
A parent component cannot use data binding to read child properties or invoke child methods. Do both by creating a template reference 
variable for the child element and then reference that variable within the parent template as seen in the following example.

The following is a child CountdownTimerComponent that repeatedly counts down to zero and launches a rocket. The start and stop methods 
control the clock and a countdown status message displays in its own template.

```js
import { Component, OnDestroy } from '@angular/core';

@Component({
   selector: 'app-countdown-timer',
   template: '<p>{{message}}</p>'
})
export class CountdownTimerComponent implements OnDestroy {

   intervalId = 0;
   message = '';
   seconds = 11;

   ngOnDestroy() { this.clearTimer(); }

   start() { this.countDown(); }
   stop()  {
      this.clearTimer();
      this.message = `Holding at T-${this.seconds} seconds`;
   }

   private clearTimer() { clearInterval(this.intervalId); }

   private countDown() {
      this.clearTimer();
      this.intervalId = window.setInterval(() => {
         this.seconds -= 1;
         if (this.seconds === 0) {
            this.message = 'Blast off!';
         } else {
            if (this.seconds < 0) { this.seconds = 10; } // reset
            this.message = `T-${this.seconds} seconds and counting`;
         }
      }, 1000);
   }
}
```

The CountdownLocalVarParentComponent that hosts the timer component is as follows:

```js
import { Component } from '@angular/core';
import { CountdownTimerComponent } from './countdown-timer.component';

@Component({
   selector: 'app-countdown-parent-lv',
   template: `
    <h3>Countdown to Liftoff (via local variable)</h3>
    <button (click)="timer.start()">Start</button>
    <button (click)="timer.stop()">Stop</button>
    <div class="seconds">{{timer.seconds}}</div>
    <app-countdown-timer #timer></app-countdown-timer>
  `,
   styleUrls: ['../assets/demo.css']
})
export class CountdownLocalVarParentComponent { }
```
The parent component cannot data bind to the child's start and stop methods nor to its seconds property.

Place a local variable, #timer, on the tag <countdown-timer> representing the child component. That gives you a reference to the child 
component and the ability to access any of its properties or methods from within the parent template.

This example wires parent buttons to the child's start and stop and uses interpolation to display the child's seconds property.

### Parent calls an `@ViewChild()`
The local variable approach is straightforward. But it is limited because the parent-child wiring must be done entirely within the 
parent template. The parent component itself has no access to the child.

You can't use the local variable technique if the parent component's class relies on the child component's class. The parent-child 
relationship of the components is not established within each components respective class with the local variable technique. Because 
the class instances are not connected to one another, the parent class cannot access the child class properties and methods.

When the parent component class requires that kind of access, inject the child component into the parent as a ViewChild.

The following example illustrates this technique with the same Countdown Timer example. Neither its appearance nor its behavior changes.
The child CountdownTimerComponent is the same as well.

```js
import { AfterViewInit, ViewChild } from '@angular/core';
import { Component } from '@angular/core';
import { CountdownTimerComponent } from './countdown-timer.component';

@Component({
  selector: 'app-countdown-parent-vc',
  template: `
    <h3>Countdown to Liftoff (via ViewChild)</h3>
    <button (click)="start()">Start</button>
    <button (click)="stop()">Stop</button>
    <div class="seconds">{{ seconds() }}</div>
    <app-countdown-timer></app-countdown-timer>
  `,
  styleUrls: ['../assets/demo.css']
})
export class CountdownViewChildParentComponent implements AfterViewInit {

  @ViewChild(CountdownTimerComponent)
  private timerComponent!: CountdownTimerComponent;

  seconds() { return 0; }

  ngAfterViewInit() {
    // Redefine `seconds()` to get from the `CountdownTimerComponent.seconds` ...
    // but wait a tick first to avoid one-time devMode
    // unidirectional-data-flow-violation error
    setTimeout(() => this.seconds = () => this.timerComponent.seconds, 0);
  }

  start() { this.timerComponent.start(); }
  stop() { this.timerComponent.stop(); }
}
```
It takes a bit more work to get the child view into the parent component class.

First, you have to import references to the ViewChild decorator and the AfterViewInit lifecycle hook.

Next, inject the child CountdownTimerComponent into the private timerComponent property using the @ViewChild property decoration.

The #timer local variable is gone from the component metadata. Instead, bind the buttons to the parent component's own start and stop 
methods and present the ticking seconds in an interpolation around the parent component's seconds method.

These methods access the injected timer component directly.

The ngAfterViewInit() lifecycle hook is an important wrinkle. The timer component isn't available until after Angular displays the 
parent view. So it displays 0 seconds initially.

Then Angular calls the ngAfterViewInit lifecycle hook at which time it is too late to update the parent view's display of the countdown 
seconds. Angular's unidirectional data flow rule prevents updating the parent view's in the same cycle. The application must wait one 
turn before it can display the seconds.

Use setTimeout() to wait one tick and then revise the seconds() method so that it takes future values from the timer component.

## Architecture planning

### Architecture considerations
There are a series of steps we should consider when we want to start a -not only Angular- application. 

It could be a good idea to present to the team via powerpoints -or similar tools- a slide for every of the next points and discuss with 
the team:

1. App overview: what is the app for? what is the goal? business and strategic benefits? Etc.
2. App features: list all the features we know at the moment
3. Domain security: what about authentication and authorization? are we using roles? groups? How is going to the angular communicate 
   with the api? via tokens?
4. Domain rules: Rules like validation will run client side? server side? both? Etc.
5. Logging: What do we do with the errors we caught?
6. Services/Communication: How is the front communicating with the server? HttpClient or Axios? NodeJs? REST or GrapQL? http only? web 
   sockets? How between front-end components itself?
7. Data models: What model data are we expecting in the front? How are going to be the interfaces? Are we going to use interfaces? And 
   classes? When to use inheritance and composition? How are going to be exactly?
8. Feature components: How are going to structure our components? Which one will be presentational or functional? Which components are 
   going to be lazy loaded? Which structure are they go to have? Are we going to use grids systems? What about UI libraries?
9. Shared functionality: Do we have shared functionality? any third-parties? Custom components? Could we reuse components between "apps"?
10. Test: unit test? behaviour test? e2e test? Which framework are we going to use? Which policy?
11. Coding rules: are we going to follow any style guide? Lint? Prettier?
12. Accessibility
13. i18n
14. Environments
15. CI/CD
16. CDNs, containers, servers...

So, before start a project:

1. Talking and planning about the above points will help us a lot to define a better -but not final- architecture
2. Having a basic and simple architecture template that can be reused across projects is a good idea
3. Defining a style guide based on the official one will help also for PRs

## Organizing features and modules

### Features

1. Organize your code, that it's, your folders and files, in a feature-base approach
2. USe the Angular CLI to generate files as much as you can
3. Minimum one module per feature 
4. Avoid deeply nested folders: the below code don't follow the real app hierarchy but is far more easy to read: https://angular.io/guide/styleguide#flat

![](img-01.png)

### Features modules
1. Use two modules per feature: one for basic declarations and dependencies and other for routing
2. Import the routing one into the main one
3. Since you need to import the routing components to the routing-module, you can add to the exported class in the routing-module an 
   static attribute like "components" to be an array of all this components; in that way, you dont need to add more imports to the main 
   feature module. Here an example:

![](img-02.png)
![](img-03.png)

### Core and shared modules

#### Core and features modules
1. Core folder should contain singleton services shared throughout app, or at leasy things that should be created once (like a modal)
2. Services that are specific toa feature can go in the feature's folder
3. It should be imported only in the Root Module

Tool to prevent the core module to be re-imported:

![](img-04.png)

`@Optional` is a constructor parameter decorator that marks a dependency as optional.
`@SkipSelf` is a constructor parameter decorator that tells the DI framework that dependency resolution should start from the parent 
injector

Another "class based" solution for the same problem and a bit cleaner:

![](img-05.png)
![](img-06.png)

#### Shared modules
1. Reusable components, pipes, directives; all things that can be created more that once
2. It could be imported into many other modules (the ones that need it)

### Custom library
https://angular.io/guide/creating-libraries

## Organizing components
### Container and presentation component patterns
#### Container components
1. Container components retrieves state: manage state, interacts with a service/store, etc.
2. Container components use presentation components as children and pass the data to them

#### Presentation components
1. Presentation components presents/render state in UI

When using presentation components we received data from a container component. From the container component "point of view" we don't 
want the child presentation component change that state or data; oc, the user can change the data so we can inform of this via an event 
emitter, but the idea is that the component can't change the data by itself, because that's the job of the container/parent component. 
Oc, the presentation component can "initilize" data, but don't change it after that.

To avoid this in a "force" way, we can use the "on push" strategy in the presentation component:

```js
@Component({
   // ...
   changeDetection: ChangeDetectionStrategy.OnPush
})
```

Using this option, if any @Input variable inside the presentation component change because of some internal logic, then the component is 
not going to re-render. In a more technical exaplanation, OnPush cause change detection only when:

1. An input property reference change
2. An output/EventEmitter or DOM event fires
3. Async pipes receives an event
4. Change detection is manually invoked via ChangeDetectorRef

Benefits of use it:

1. This will protect us to accidently overwrite code from our state
2. Performance (component isn't checked until OnPush conditions are met)

### Child routes
1. Use child routes to dynamically load components based on the url

### Reference vs. Value types
It's important to know that a presentation component with a OnPush strategy will only change if the input reference change, that means 
that if you pass a non-primitive value, like object or an array, it will no re-render.

To avoid that problem we can implement "cloning" techniques.

#### Cloning 
Cloning is the process of making an exact copy of an object.

We can have tree solutions, from basic to more robust one:

1. JSON.parse
2. Custom solution
3. Inmutable.js

### Component Inheritance
Since a component in angular is a class with a decorator, that means you can use the inheritance pattern for components if you need.

You just need to create a component and the "extends" with it the ones you want, and the call "super()" in the constructor.

You can use this pattern when you have components that uses the same kind of inputs, methods, etc, so you can group this "shared" 
fncioality in a component and extendes the other ones.

## Components communication
![](img-07.png)

### Event bus service
![](08.png)
![](09.png)
![](10.png)

### Observable service
![](11.png)
![](12.png)
![](13.png)
![](14.png)

We can see that the Observable service not only notifies the change, but also it is the "producer" of that change of data.
![](15.png)

### Unsubscribing from observables

#### NgOnDestroy
![](16.png)

#### Auto unsubscribe decorator
![](17.png)

#### Using Subsink
![](18.png)

## Functions vs. pipes
- Functions call made from a template are invoked every time a change occurs (no cachin).

Example:
```js
{{ addTax(product.price) | currency }}
```

### Replacing functions with pipes
- A pure pipe returns the same result given the same inputs
- Only called when inputs to the pipe are changed

Refactored example with a pipe
```js
{{ product.price | addTax | currency }}
```

### Using a memo decorator
- `memo-decorator` is a package from NPM. Just install it and add the decorator `@memo` to a Pipe declaration. It will cache the inputs 
  pass with the data returned to avoid to call the function body when it has already a value for the same inputs.

## Security considerations
![](20.png)
![](21.png)
![](22.png)
![](23.png)
![](24.png)
![](25.png)

## Services
Do limit logic in a component to only that requires for the view. All other logic should be delegated to services, since service must 
have all the funcionality not related to the UI.

### Usar provideIn
Es mejor usar la configuración `provideIn` del decorator `@Injectable` ya que no solo nos ahorramos añadir el servicio al modulo en el 
array de providers, sino que tambien, angular en tiempo de compilacion determinara si ese servicio es usado o no, y si no lo es, 
directamente no lo meterá en el código final, teniendo asi un bundle size más pequeño.

### Dependency Injection

#### Providers
A dependency provider configures an injector with a DI token, with that injector uses to provide the runtime version of a dependency value.

![](32.png)

A short syntax of the same (an the most used):

![](31.png)

#### Injectors

![](33.png)

If for any reason, a dependency that is provider in a root level, we want to have a different "instance" inside an specific component,
then we can add this to de decorator class:

![](34.png)
![](35.png)

#### Handling errors in async services with RxJs

![](36.png)
![](37.png)

#### Built-in Angular services


 