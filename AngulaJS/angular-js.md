# `AngularJS`

## `The Major AngularJS Players`

### `$compile`

How does AngularJS know what to do with the application? How does it know that this is an HTML template that we need to bind it? 

*`$compile`* compiles DOM into a template function that chan then be used to link scope and the View together. 

*Compilation phase:*

1. At the lowest level there is a compilation cycle; so, when the page loads its loads your static DOM and you have the content ready event that's fired; an Angular is listening to that event.
2. Then Angular search for the ng-app directive.
3. Compile all your services, controllers, etc. than has been declared in the app.module.
4. Go through the DOM and look for directives and generates a template.
5. Goes back through and it links together: this template gets this scope; binds ir together and we have the view.

### `$digest`
*`$digest`* processes all of the watchers of the current scope.

### `$apply`
- *`$apply`* forces *`$digest`* cycle.
- *`$apply()`* is used to notify that something has happened outside of the AngularJS domain.

### `$scope`

 *`$scope`* is the **glue** between the Controller and the View. The Controller is responsible for constructing the model on *`$scope`* and providing commands for the View to act upon. Also *`$scope`* provides context.

 ### `Controllers`

 Controllers should...

 1. Not know anything about the view they control: they are simply data structures and methods. If you need to do DOM manipulation, that goes in a directives.
 2. Be small of focused: they should only care about the view that they control. 
 3. Not talk to other controllers.
 4. NOT own the domain model: in others words, the controller should not have information of data that the rest of the application is going to need.

### `View and Template`

- The View is AngularJS compiled DOM.
- The View is the product of $compile merging the HTML template with $scope.
- DOM is no longer the single source of thuth.

 ### `Model and Services`

- Services carry out common tasks specific to the web application.
- Services are consumed via the **AngularJS**Dependency Injection subsystem.
- Services are application singletons.
- Services are instantiated lazily.
- 

## `Introducing Data-Binding`
Without getting into the source (available at AngularJS.org¹⁶), Angular simply remembers the value that the model contains at any given time (in our example from hello world, the value of name).

When Angular thinks that the value could change, it will call $digest() on the value to check whether the value is “dirty.” Hence, when the Angular runtime is running, it will look for potential changes on the value. This process is dirty checking. Dirty checking is a relatively efficient approach to checking for changes on a model. Every time there could be a potential change, Angular will do a dirty check inside its event loop (discussed in depth in the under the hood chapter) to ensure everything is
consistent.

## `Change Detection`
Detect which parts of the data-model changes to reflex it into the user interface (view). 

Digest Cycle > Dirty Check > Re-Render.

## `UI-Router`

### Parameters
Three ways of declaring Parameters:

```javascript
.state('schools', {
    url: '/schools/:id'
    // ...
})
.state('classrooms', {
    url: '/classrooms/{id}' // Permite más cosas: p.e.: usar RegExp
    // ...
})
.state('activities', {
    // Para valores default
    url: '/activities' 
    params: {
        id: {
            value: 42
        }
    }
})
```

### Resolve Property

Permite especificar un mapeado de dependencias que queremos injectar en el state controller; generalmente se le prove de una función que devuelve una promesa que deberá ejecutarse con éxito antes de que el state change suceda. El resultado del resolved es inyectado en el controlador con el nombre especificado en el resolve attr.

## `Modules`

- Modules are not a namespacing mechanism for angular objects.
- The injector ($injector) is the name of the AngularJS object  that has a list of all the objects in your application. That means that exists only one injector per application.

## `Controllers`

 - A controller coordinate View and Model.
 - Dont let the controller do things that not supose to do.

### Bad practice:
```javascript
angular.module('app')
    .controller('myController', function($scope) {
        $scope.schedule = {
            classes: []
        };
        // The problem here is that register is not a piece of interaction between view and model, but business logic (a service).
        $scope.register = function(newClass) {
            $scope.schedule.classes.push({ class: newClass })
        }
    })
```

### Good practice:
```javascript
// Not to do:
angular.module('app')
    .controller('myController', function($scope, schedule) {
        $scope.schedule = schedule;
        $scope.register = function(newClass) {
            $scope.schedule.register(newClass)
        }
    })
```

## `Services`

- Handle non-view logic.
- Communicate with Server.
- Hold data & state.

Hay 5 formas diferentes de crear servicios en AngularJS:

1. provider()
2. factory()
3. service()
4. value()
5. constant()

### provider()
 
#### Using $provide.provider()

- Call the "provider" function on the $provide service.
- Provider must define a "$get" function.
- Service is the object returned from the $get function.
- The benefit of the provider is that is configurable during the module configuration phase.

```javascript
var app = angular.module('app', []);

app.provider('books', function() {
        var includeVersionInTitle = false;
        this.setIncludeVersionInTitle = function(value) {
            includeVersionInTitle = value;
        }
    
        this.$get = function() {
            var appNmae = 'Book Logger';
            var version = '1.0';

            if (includeVersionInTitle) {
                appName += ' ' + version;
            }

            return {
                appName: appName
            }
        }
    });

// Angular automaticamente crea el objeto booksProvider (nombre + Provider)
app.config(function(booksProvider) { 
    booksProvider.setIncludeVersionInTitle(true);
})
```
#### Using $provide.factory()

- Simpler version of provider when additional configuration is unnecesary.
- Registers a service facotory function that will return a service instance.

```javascript
function factory(name, factoryFn, enforce) {
    return provider(name, {
        $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn;
    });
}
```

#### Using $provide.service()

- Calls factory function which calls provider function.
- Treats function it is passed as a constructor.
- Executes constructor function with "new" operator.

```javascript
function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
        return $injector.instantiate(constructor);
    }]);
}
```

#### Value Services:

- Shorthand for factory with no parameters.
- Cannot be injected into a module configuration function.
- Can be overridden by and AngularJS decorator.

```javascript
angular.module('app')
/* Los servicios creados con value son simplemente wrapper alrededor de la factory function. Podemos usar value en vez de factory si no necesitamos injectarlo en un modulo config de angularjs. Recordar: solamente se pueden injectar aquellos que parten como provider*/
       .value('bookService', {
           retrieveBooks: retrieveBooks
       })

function retrieveBooks() {
    return ['book1', 'book2'];
}
```

#### Constant Services:
- Simply registers service with injector, no factory/provider calls.
- Can be injected into a module configuration function.
- Cannot be overridden by and AngularJS decorator.
```javascript
angular.module('app')
        // We can pass also a function or string, just a value that won´t change.
       .constant('constants', {
           API_TITLE: 'Book Logger',
           APP_DESCRIPTION: 'Track which books you read.'
       })
```

### Dependency annotations
1. Inform inejctor what services should be injected into a component.
2. Use to support minimization.
3. Three techniques available:
    1. Implicity from function parameter names.
    2. Using inline array annotation.
    3. Using $inject property annotation. 

#### Example using inline array annotation
```javascript
angular.module('app')
       .controller('BooksController', ['books', 'dataService', BooksController]);

function BooksController(books, dataService) {
           // Controller code...
       }
```
#### Example using $inject property annotation
```javascript
angular.module('app')
       .factory('dataService', dataService);

function dataService(logger) {
    logger.show('hola que tal');
    return {
        getData: function() {
            return ['data1', 'data2'];
        }
    }
}

dataService.$inject = ['logger'];
```

### Built-in Services

#### Prommises and the $q Service

In javascript, promises are objects which represent the pending result of an asynchronous operation.

The $q Service provide an api for working with promises and an api for the so-called deferred objects that return promises to the calling code, and signal them with the results when the asynchronous operation is complete.

The client/Server flow of a promise is easy: 

| Client                                          |     | Service                                          |
| :---------------------------------------------: |:---:| :----------------------------------------------: |
| Initiate asynchronous call to service           | >>> | Create deferred object                           |
| Use promise API to configure callback functions | <<< | Return promise to client                         | 
|                                                 |     | Perform work...                                  |
| Execute callback functions                      | <<< | User deferred API to signal client with results  |
|                                                 |     |                                                  |

##### Using $q Service

###### Service:
```javascript
angular.module('app')
       .factory('dataService', ['$q', '$timeout', dataService]);

function dataService($q, $timeout) {
    return {
        getData: getData
    }

    function getData() {
        var data = ['data1', 'data2'];
        // Call the defer method and assign its value to a variable
        var deferred = $q.defer();
        // Simulate an async operation
        $timeout(function() {
            var success = true;
            if (success) {
                // The API allows to notify the caller while the work is being perform:
                deferred.notify('Getting Data...');
                deferred.resolve(data);
            } else {
                deferred.reject('Error retrieving the data...');
            }
        }, 1000);
        // Return a promise to the caller
        return deferred.promise;
    }
}
```
###### Caller:
```javascript
angular.module('app')
       .controller('DataController', ['dataService', '$log', DataController]);

function DataController(dataService, $log) {
    dataService.getData()
        .then(successHandler, null, notificationHandler);
        .catch(errorHandler)
        .finally(function() {
            $log.warn('Get Data completed');
        })

    function successHandler(data) {
        $log.log(data);
    }

    function errorHandler(reason) {
        $log.error(reason)
    }

    function notificationHandler(notification) {
        $log.log('Promise notification: ', notifiction);
    }
}
```

###### Using Promise All:
```javascript
angular.module('app')
       .controller('DataController', ['dataService', '$q', '$log', DataController]);

function DataController(dataService, $q, $log) {
    var vm = this;

    var dataPromise = dataService.getData();
    var extraDataPromise = dataService.getExtraData();

    // Solo se ejecutará cuando todas las promesas se resuelvan.
    $q.all([dataPromise, extraDataPromise])
        .then(getAllDataSuccess)
        .catch(getAllDataError)

    function getAllDataSuccess(dataArray) {
        vm.data = dataArray[0];
        vm.extraData = dataArray[1];
    }

    function getAllDataError(reason) {
        $log.error(reason)
    }
}
```
### Caching (Cache)

- $cacheFactory creates cache objects.
- Add and remove items to and from cache objects.
- Cache objects requested at any time from $cacheFactory.

#### Using $cacheFactory

- Call as a function to create new cache objects.

```javascript
// Create an  cache object with the name 'bookLoggerCache':
var dataCache = $cacheFactory('bookLoggerCache');
```

- Call get() method to retrieve existing cache objects.

```javascript
// Create an  cache object with the name 'bookLoggerCache':
var dataCache = $cacheFactory.get('bookLoggerCache');
```

#### Using Cache Objects
- put(key, value)
- get(key)
- remove(key)
- removeAll()
- destroy()
- info()

Example using cache with data:
```javascript
var deferred = $q.defer();
var dataCache = $cacheFactory('bookLoggerCache');

if (!dataCache) {
    dataCache = $cacheFactory('bookLoggerCache');
}

var summaryFromCache = dataCache.get('summary');

if (summaryFromCache) {
    console.log('Returning summary from cache');
    deferred.resolve(summaryFromCache);
} else {
    // get data using http request...
    // var summaryData = http.get()...
    // And then, before resolve the deferred:
    dataCache.put('summary', summaryData);
    deferred.resolve(summaryData);
}
```

### Service Decorators

#### Decorator Pattern

In object-oriented programming, the decorator pattern is a design pattern that allows behavior to be added to an individual object, either statically or dynamically...

#### Using $provide to decorate a service
```javascript
app.config('$provide', function($provide) {
    $provide.decorator('$log', logDecorator);
});

// $delegate contains the service to be decorated
// In this case $delegate refers to $log service
function logDecorator($delegate) {
    // Now we override the log function of the $log service
    function log(message) {
        $delegate.log(message.toUpperCase());
    }

    // ..

    return {
        log: log,
        // ..
    }
}
```

## `Directives`

At a high level, directives are markers on a DOM element (such as an attribute, element name, comment or CSS class) that tell AngularJS's HTML compiler ($compile) to attach a specified behavior to that DOM element (e.g. via event listeners), or even to transform the DOM element and its children.

*js*
```javascript
var app = angular.module('app', []);

// Use camelcase sintax
app.directive('myDirective', function() {
    returrn {
        restrict: 'E',
        replace: true,
        template: '<div>This is my directive</div>'
    }
})
```

*html* 
```html
<body>
    <div ng-contorller="MyController">
    <!-- Use snake-case -->
        <my-directive></my-directive>
    </div>
</body>
```
### Purposes of Directives

1. Widgets (reusable view with its own logic: ad-section, etc.)
2. DOM Events (click events, etc.)
3. Functionality (like ngShow, etc.)

### Why a **DSL** (*Domain-specific language*)?

> People find DSLs valuable because a well-designed DSL can be much easier to program with than a traditional library. This improves programmer productivity, which is always valuable. In particular it may also improve communication with domain experts, **which is an important toll for tackling one of the hardest in software development.** 
(**Martin Fowler**)

### Directive Definition Object

Most simple directives consist of three things (or less). Having a firm grasp on these three things will make advanced directives much more approachable:

1. Link
2. Controller
3. DDO (*Directive Definition Object*)

The Directive Definition Object (DDO) tells the compiler how directive is supposed to be assembled. Common properties are:

 - The **link** function
 - The **controller** function
 - **restrict**
 - **template**
 - **templateUrl**

*Example*

```html
<div animate-down></div>
<div animate="down"></div>
```

*First option*
```js
app.directive('animateDown', function() {
    var linker = function(scope, element, attrs) {
        var down = function() {
            $(this).animate({
                top: '+=150'
            });        
        }
        // Element is basicly a jQuery Element
        element.on('clikc', down);
    }

    return {
        restrict: 'A',
        link: linker
    }
})
```

*Second option*
```js
app.directive('animate', function() {
    var linker = function(scope, element, attrs) {
        scope.down = function() {
            $(this).animate({
                top: '+=150'
            });  
        }
        var direction = attrs['animate'];
        element.on('clikc', scope[direction]);
    }

    return {
        restrict: 'A',
        link: linker
    }
})
```

### Isolated Scope
- Isolated scope does not prototypically from its parent. This prevents your directive from accidentally modifyinh data in the parent scope. This in a sense defines the API for the directive.
  
#### Attribut isaloted scope

- Defined with an **`'@'`** symbol.
- Binds a local scope property to the value of a DOM attribute.
- The binding is uni-directional from parent to directive.
- Te result is always a string because the DOM attributs are strings.

```js

app.directive('myDirective', function() {
    return {
        restrict: 'E',
        scope: {
            value: '@'
        }
    }
})
```

#### Binding isaloted scope

- Defined with an **`'='`** symbol.
- Bi-directional binding between parent and directive.
- You can define the binding as optional via **`'=?'`**.
- Optimal for dealing with objects and collections.

```js

app.directive('myDirective', function() {
    return {
        restrict: 'E',
        scope: {
            name: '='
        }
    }
})
```

#### Expression isaloted scope

- Defined with an **`'&'`** symbol.
- Allows you to execute an expression on the parent scope.
- To pass variables from child to parent expressions you must use an object  map.

```html
<div ng-app="phoneApp">
  <div ng-controller="AppCtrl">
    <div phone dial="callHome('called home!')"></div>
  </div>
</div>
```

```js
var app = angular.module('phoneApp', []);

app.controller("AppCtrl", function ($scope) {
  $scope.callHome = function (message) {
    alert(message);
  };
});

app.directive("phone", function () {
  return {
    scope: {
      dial: "&"
    },
    template: '<div class="button" ng-click="dial()">Call home!</div>',
  };
});
```

### Transclusion

- Compiles the contents of the element and makes it available to the directive.
- The transcluded contents are bound to the parent scope.
- The directive contents are bound to an isolated scope.
- The two scopes are siblings.
- This is good for decorating an existing element with a directive without destroying it.

### Link and Controller

When building a directive with any decent amount of functionality, you often have the option of adding that functionality to your directive through one of two means:

1. The first is to put the functionality into the link function of the directive.
2. The other option is to create a controller and put that functionality into the contorller.

There is no right or wrong solution here, but the two options can be quite different in implementation.

#### Using the `link` function:

- The **link** function is where DOM manipulation happens. It comes with **scope**, **element** and **attrs**.
- **scope** is the same **$scope** in the controller function.
- **element** is a jQuery instance of the DOM element the directive is declared on.
- **attrs** is a list of attributes declared on the element. 

```javascript
var app = angular.module('app', []);

app.controller('scheduleCtrl', function($scope) {
    $scope.classList = [
        { id: 1, name: 'Biology' },
        { id: 2, name: 'Chemistry' },
        { id: 3, name: 'History' },
    ];
});

app.controller('scScheduleCtrl', function($scope) {
    $scope.viewClassDetails = function(classToView) {
        console.log('Viewing details for ', classToView.name);
    }
});

app.directive('scSchedule', function() { 
   return { 
        restrict: 'E',
        replace: true,
        // controller: 'scScheduleCtrl',
        link: function(scope) {
            $scope.viewClassDetails = function(classToView) {
                console.log('Viewing details for ', classToView.name);
                }
            },
        template: `
            <div>
                <h3>Your Schedule</h3>
                <div ng-repeat="class in classList">
                    <div>{{ class.name }}</div>
                    <div>
                        <button ng-click="viewClassDetails(class)">
                            Details
                        </button>
                    </div>
                </div>
            </div>
        `
   }
});

```

In the next example we will build a directive that displays the current time. Once a second, it updates the DOM to reflect the current time.

Directives that want to modify the DOM typically use the `link` option to register DOM listeners as well as update the DOM. It is executed after the template has been cloned and is where directive logic will be put.

`link` takes a function with the following signature, `function link(scope, element, attrs, controller, transcludeFn) { ... }`, where:

- `scope` is an AngularJS scope object.
- `element` is the jqLite-wrapped element that this directive matches.
- `attrs` is a hash object with key-value pairs of normalized attribute names and their corresponding attribute values.
- `controller` is the directive's required controller instance(s) or its own controller (if any). The exact value depends on the directive's require property.
- `transcludeFn` is a transclude linking function pre-bound to the correct transclusion scope.

In our `link` function, we want to update the displayed time once a second, or whenever a user changes the time formatting string that our directive binds to. We will use the `$interval` service to call a handler on a regular basis. This is easier than using `$timeout` but also works better with end-to-end testing, where we want to ensure that all $timeouts have completed before completing the test. We also want to remove the `$interval` if the directive is deleted so we don't introduce a memory leak.

*script.js*
```javascript
angular.module('docsTimeDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.format = 'M/d/yy h:mm:ss a';
}])
.directive('myCurrentTime', ['$interval', 'dateFilter', function($interval, dateFilter) {

  function link(scope, element, attrs) {
    var format,
        timeoutId;

    function updateTime() {
      element.text(dateFilter(new Date(), format));
    }

    scope.$watch(attrs.myCurrentTime, function(value) {
      format = value;
      updateTime();
    });

    element.on('$destroy', function() {
      $interval.cancel(timeoutId);
    });

    // start the UI update process; save the timeoutId for canceling
    timeoutId = $interval(function() {
      updateTime(); // update DOM
    }, 1000);
  }

  return {
    link: link
  };
}]);
```

*index.html*
```html
<div ng-controller="Controller">
  Date format: <input ng-model="format"> <hr/>
  Current time is: <span my-current-time="format"></span>
</div>
```

### Requering Controllers in Directives

```javascript
return {
    restrict: 'E',
    require: 'nameOfSiblingController' || '^nameOfParentController'
}
```

When you require another directive's controller, then your `link` function will get a fourth parameter, which will be that controller. If you want to get several controllers from several other directives, then instead of a string, you can supply an array of strings and that fourth parameter in your link function will be an array of controllers in the same order.

*Example:*

```javascript
var app = angular.module('app', []);

app.value('scFollowedInstructors', []);

app.controller('scInstructorsCtrl', function($scope, scFollowedInstructors) {
    $scope.followedInstCount = scFollowedInstructors.length;
    
    this.followInstructor = function(instructor) {
        $scope.followedInstCount.push(instructor);
        $scope.followedInstCount = scFollowedInstructors.length;
    }
});

app.directive('scInstructors', function() { 
    return { 
        restrict: 'E',
        replace: true,
        controller: 'scInstructorsCtrl',
        template: `
            <div>
                <h3>Instructors ({{followedInstCount}} followed)</h3>
                <div ng-repeat="instructor in instructorList">
                    <h4>{{ instructor.name }}</h4>
                    <sc-follow-instructor instructor-to-follow="instructor" />
                </div>
            </div>
        `
   }
});

app.directive('scFollowInstructor', function() { 
    return { 
        restrict: 'E',
        replace: true,
        template: `
            <button ng-click="followInstructor()">
                Follow
            </button>`,
        scope : {
            instructorToFollow: '='
        },
        // Ponemos el nombre de la directiva padre, no del controlador de la directiva.
        require: '^scInstructors',
        link: function(scope, elmeent, attrs, ctrl) {
            scope.followInstructor = function() {
                ctrl.followInstructor(scope.instructor);
                // Ocultamos el button para que no s epueda añadir de nuevo:
                element.css('display', 'none');
            }
        }
   }
});
```

That was a good example of using require to get a handle on another directive's contorller. We can use this to get access to siblings controllers in case we have two directives that should have enhanced functionality if they are both on the same element or many other uses.

### Avoiding FOUC in Views

FUOC or Flash of Unstyled Content (in this case we are talking about Uncompiled Content, noy unstyled exactly, that means: when angular get to run and show the bindings: {{ }}).

What we can do to avoid showing that curly braces is:

- `ngCloak`
- `ngBind`
- Waiting Image

#### ngCloak

1. To make ngCloak work you must go to your `style.css` and put: 

```
[ng\:cloak], [ng-cloak], [data-ng-cloak], [x-ng-cloak], .ng-cloak, .x-ng-cloak {
    display: none !important;
}
```

2. Then, add the `ngCloak` directive to any html element; that will hide that element until Angular has a chance to run.

```html
<body ng-cloak>
    ...
</body>
```

#### ngBind

The `ngBind` directive lets you do the same thing that the curly braces do when binding to an element. Just add `<div ng-bind="myVariable">&nbsp</div>`; the `&nbsp` is to set the height of the div element with a default content.

#### Image loading

*Example:*
```html
<div>
    <img src="spinner.gif" ng-hide="true" style="height: 400px" />
    <div class="content" ng-cloak>
        ...
    </div>
</div>
```

#### Extra: Solution for async data

This time the best option is using the image spinner also. The same solution but using a variable `loading`:

*Example:*
```html
<div>
    <img src="spinner.gif" ng-show="!loading" style="height: 400px" />
    <div class="content" ng-show="!loading" ng-cloak>
        ...
    </div>
</div>
```

## Scope

Angular always starts out with a Root Scope. Within that Root Scope, all scopes within your Angular application exist within a hierarchy. 

All scopes belong to exactly one elmement. When you create a scope, you are creating it on an elmement in your DOM. All child elments of that element either use that scope or a child scope.

### Creating Scopes

Scopes are created by directives. Controllers set up the scopes, but directives are the reason that scopes are actually created.

## `Communicating between Components`

There are three ways:

1. Inherited Scope
2. Events
3. Services

### Inherited Scopes

Inherited scopeslet us communicate between components because anything defined on a higher-level scope is available to all the inherited scopes.

Imagine you have a parent scope and two child scopes. You wish for some action taken on Child Scope 1 to have an effect on something that belongs to Child Scope 2 (Parent Scope > Child Scope 1 + Child Scope 2). Well, with inherited scopes, if there is an object on the parent scope then both child scopes can see and make calls on that object and respond to changes in that object.

### Events

- `boadcast`: Emit events to the child scopes.
- `emit`: Emit events to the parent scopes.
- `on`: Listen to emitted events.

## `Communicating with the server`

### Using $http

*Returning the $http request:*
```javascript
app.factory('dataService', function($http) {
    return {
        getData: function() {
            return $http.get('url...');
        }
    }
});

app.controller('dataController', function(dataService) {
   var vm = this;

   dataService.getData()
    .success(function(data) {
        vm.data = data;
    });
})

```

*A better way:*
```javascript
app.factory('dataService', function($http, $q) {
    return {
        getData: function() {
            var deferred = $q.defer();

            $http.get('url...').success(function(data) {
                deferred.resolve(data);
            });

            return deferred.promise;
        }
    }
});

app.controller('dataController', function(dataService) {
    var vm = this;
    // Angular is resolving this promise for us and binding to the data that's resolved with that promise.
    vm.data = dataService.getData();
});
```

*Using $resource*

```javascript
app.factory('dataService', function($http, $resource) {
    return {
        getData: function() {
            return $resource('url', undefined, {
                get: {
                    method: 'GET',
                    isArray: true
                }
            }).get().$promise;
        }
    }
});

app.controller('dataController', function(dataService) {
    var vm = this;
    // Angular is resolving this promise for us and binding to the data that's resolved with that promise.
    vm.data = dataService.getData();
});
```

- `$http`: Returns an Object with 5 methods:
    - `then(config)`: data is in `config.data`.
    - `catch`
    - `finally`
    - `success(data, status, headers, config)`
    - `error`
- `$q`: Returns an Object with only 3 methods:
    - `then(data)`: Unlike the `then` function in the `$http`service, this `then` method gives us direct the data as a first parameter.
    - `catch`
    - `finally`
- `$resource`: Returns an Array that has the elements in it plus 2 adittional objetcs:
    - `$promise`:
        - `then`
        - `catch`
        - `finally`
    - `$resolved`

### Restangular

`Restangular`is a singleton service that has a bunch og methods on it. It doesn't really work with URLs; instead, you give the RESTful fragments of your URL and it will construct the URL for you.

```javascript
var app = angular.module('app', ['restangular']);

app.config(function(RestangularProvider) {
    RestangularProvider.setBaseUrl('/data');
});

app.factory('usersService', function(Restangular) {
    // URL created: /data/users
    return Restangular.all('users');
});

app.controller('usersController', function(usersService) {
    var vm = this.

    // We can request for all users
    usersService.getList()
        .then(function(users) {
            vm.users = users;
        });
    
    // Or We can pass the id of the user we want
    usersService.get(1)
        .then(function(user) {
            vm.currentUser = user;
        });
    
    // We can also try to get a list of items inside a specific item:
    usersService.one(1)
        .all('califications')
        .getList()
        .then(function(califications) {
            vm.califications = califications;
        });
})
```

## `Models`

### What is a Model?
There are a few features that a model may contain that we may want to have:

1. `Represents Data and State`: For Angular, a model is the Javascript objects that contain the data and the state of our application. These models don't have to be attached to any scope. Remember that the scope is not the model: it's simply a place where we can attach our model, but the scope itself is not the model.
2. `Business Logic`: We want our models to contain the business logic that controls the models themselves.
3. `Caching`: We want our models to be cached client side so that on different views we don't have to request the same data from the server multiple times.
4. `Change notification`: 

### Creating Models with `Restangular`

`Restangular`, unlike *`$resource`*, doesn't give a handle to the constructor function for the objects that it creates. Instead, `Restangular` has an extend model function that allows you to add methods to models as they are created.

```javascript
var app = angular.module('app', ['restangular']);

app.factory('users', function(Restangular) {
    // First we pass the name of the model and second a function that receives the model.
    Resangular.extendModel('users'. function(model) {
        model.authenticate = function(name, password) {
            var isValid = this.name === name && this.password === password;
            return isValid;
        }
        return model;
    });
    return Restangular.all('users');
});

app.factory('courses', function(Restangular) {
    Restangular.extendModel('courses', function(model) {
        model.register = function() {
            this.registered = true;
            this.post();
        };
        model.unRegister = function() {
            this.registered = false;
            this.post();
        };
        return model;
    })
    return Restangular.all('courses');
});
```

That's a way to encapsulate logic with our models using `Restangular`;

