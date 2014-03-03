# AngularJS Notes

## Table of contents

1. Modules
2. Module components
3. Scopes
4. Views
5. Directives
6. Controllers
7. Filters
8. HTTP 
9. Forms
10. Routing
11. Building

## Modules

A module is a logical collection of application components. They can contain services, controllers, filters, directives, constants and simple values.

Modules provide a mechanism for declaring how groups of app components should be configured and then bootstrapped. They also provide a way for modules to declare dependencies on other modules.

### Module lifecycle

Modules have two distinct phases in their lifecycle:

1. The configuration phase. Constants are set and module components are registered and configured.
2. The run phase. Module components are instantiated and injected with dependencies as needed.

### Module creation

Modules are created by defining the module name and an array of dependencies on other modules:

```javascript
angular.module("products",["resources.products","security.auth"]);
```

### Module retrieval 

Once created, modules can be retrieved later on by just passing in a single parameter, the module name:

```javascript
var mod = angular.module("products");
```

### Module dependencies

A module dependency implies that the configuration of the depended upon module should occur before that of the parent module, nothing more. The load/definition order of Angular modules is generally not important since at load time the component definition code is merely registered, not invoked.

### App module structure

Some apps will declare a single `"app"` module namespace and lump all the components together. Other apps will be slightly more granular and attempt to group components into a limited set of modules organised by type, i.e. a `"services"` module.

It is generally preferable however to create a highly granular set of modules - one module per site section, or even one module per file, and in each case explicitly define each module's limited set of dependencies. In this way the inter-dependency of modules is clear and the code is more testable.

Whichever approach taken it is common to declare an `"app"` module upon which top level dependencies are declared and site specific config/bootstrapping work is registered.

## Module components

These are the various decoupled components that are registered on the module.

Module components are registered during the configuration phase and instantiated and injected with dependencies later on during the run phase, therefore they can be registered in any order.

Each component is defined together with a unique name which is used to resolve dependency injections. Therefore module component names must be unique across the entire app.

A module component can be registered as follows:

```javascript
myModule.service("name",ConstructorFunction);
```

During component registration the constructor function or "recipe" for creating the component at a later date is passed in but it is not invoked. Components are instantiated by the framework later on lazily (on demand) during the run phase.

The registration syntax also supports method chaining. This is used when defining multiple components in one file. However it's generally a good idea to organise components into folders and individual files.

```javascript
angular.module("app")
    .service("apiService",ConstructorFunction)
    .controller("HomeCtrl",ConstructorFunction)
    .controller("HeaderCtrl",ConstructorFunction);
```

### Component dependency injection

A key feature of AngularJS is the way that module components are only instantiated as needed with their dependencies injected dynamically during the run phase. This makes writing testable code easier as modules and components can be created in isolation and injected with mock/stub dependencies during a test.

All dependency names are combined into one app wide namespace. Therefore components registered in one module are available for injection in any other module.

DI occurs when a component is created on demand by the framework. There are two main types of DI syntax.

#### Inferred syntax

This is where the framework infers the dependencies from the function parameter names. This is a terse syntax but it will fail once the code has been minified since this will typically rename function parameters and destroy the mappings:

```javascript
angular.module("app")
    .service("foo",function ($http,API,API_KEY) {
        // The AngularJS Core $http component and the custom constants
        // API and API_KEY will be injected here and available
        // for use in "foo" service.
        return function () {};
    });
```

#### Inline annotation syntax

In this syntax dependency names are hardcoded as strings into an ordered array along with the component function itself as the last element. This syntax is functionally equivalent to the inferred syntax above and resilient to minification although arguably more verbose and harder to maintain.

```javascript
angular.module("app")
    .service("foo",["$http","API","API_KEY",function ($http,API,API_KEY) {
        return function () {};
    }]);
```
		
### Constant components

Module constants can be set via the module `constant` method. These define immutable values that are then available for DI in both the configuration and run module phases.

```javascript
angular.module("app")
	.constant("API","/api/v1/")
	.constant("API_KEY","940e12e5eaeb4e1baf964e45fc946498")
	.constant("ERROR_MESSAGES",{
	    "500": "Oh no, something has broken",
	    "403": "You need to be logged in to do that"
	});
```

### Config components

This is code to be run only during the configuration phase. Components can be injected into a config block and configured at will. Constants (and "provider" services, more on these later) are the only sort of component that may be injected during config:

```javascript
angular.module("app")
    .config(function (API,API_KEY,apiService) {
        // Here the apiService provider is configured
        // with some constant values during the config phase
        apiService.base = API;
        apiService.key = API_KEY;
    );
```

### Run components

This is code to be run after the configuration phase but before the module at large kicks into gear. This is where you can bootstrap your app if required. It is conceptually similar to an application's `main` function or primary entry point - the difference being that they are run on a per module basis and there can be multiple run blocks.

For example to expose the time at which the app started-up on the root view scope:

```javascript
angular.module("app") 
    .run(function ($rootScope) {
        $rootScope.appStartTime = new Date();
    });
```

### Service components

AngularJS services are singleton components. Only one instance is created by the framework and then supplied for every DI request. Service uses are typically:

- Persist and share data between components
- Provide an interface for loading and accessing that data sync/aysnc
- Containers for reusable chunks of app logic and functionality

Services can receive DI just like any other component and can be registered on a module in a number of ways.

#### module.service

This is the simplest pattern (although naming one of a set of module service creation methods `service` itself was a confusing naming mistake). It allows registration of a service via a constructor function:

```javascript
angular.module("app")
    .service("myService",function () {
        this.foo = function () {
        };
        this.bar = function () {
        };
    });
```

#### module.factory

This is a slightly more flexible pattern. Register an arbitrary object as a service. Object creation logic can be more complex and private fields can be simulated:

```javascript
angular.module("app")
    .factory("myServiceFactory",function () {
        var privateVal = "baz";
        return {
            foo: function () {
            },
            bar: function () {
            }
        }
    });
```

#### module.provider

The most complex pattern. It allows an arbitrary object to be registered just like the factory pattern but also allows that object to be configured during the configuration phase before it's used for DI. Usually overkill for most services, and most useful when a service needs to be re-used across applications with configurable changes to behaviour.

To register a provider service:

```javascript
angular.module("app")
    .provider("myServiceProvider",function () {
        var configurableVal = "foo";
        this.setConfigurableVal = function (val) {
            configurableVal = val;
        }
        this.$get = function () {
            return {
                // Provider service body is defined here as the return
                // value of the special $get method and has access to
                // any configured values
            }
        }
    });
```

And to configure it:

```javascript
angular.module("app")
    .config(function (myServiceProvider) {
        myServiceProvider.setConfigurableVal("foo");
    });
```

Note that providers injected during the configuration phase have not been fully instantiated via their `$get` method yet, only the configuration API is exposed to the config code.

## Scopes

Scopes are the application view model, that is, the model as it pertains to a section of the DOM. They provide a model data context for "scope-creating" directives (i.e. `ng-controller`) and view templates.

Each scope is an instance of the `Scope` class. The `Scope` class doesn't contain any special data model related functionality beyond some framework utility methods, and therefore such data models can be thought of as POJOs (plain old JavaScript objects) as least in terms of data access and assignment.

Any value assigned to a `$scope` instance will become available for evaluation in a view template and will be automatically watched for changes - this is the live two way data binding between template and model that Angular is well known for.

Example of automatic scope creation and injection in a controller:

```javascript
angular.controller("MyController",function ($scope) {
    // A new Scope instance has been instantiated and injected
    // ready for use in the controller and its view
    $scope.foo = "bar";
});
```

### Scope inheritance

All `$scope` objects inherit from the application's `$rootScope` instance via normal prototypical inheritance. Therefore properties defined in a parent scope are visible in all child scopes (unless a child scope redefines an identically named property). Since new scopes are created by directives/controllers they will tend to create a hierarchical, inheriting tree-like structure that partially reflects the DOM structure.

Note that inherited primitive properties such as the `string` and `number` types will prototypically propagate into child scopes by initial value only. That is, if their value is changed in the child scope then that change won't be reflected back into any parent scopes. Therefore if it is required that scope properties are fully shared throughout the scope inheritance hierarchy then define them as properties on an object reference, e.g. `obj:{prop:value}`.

### Scope events

Scopes can broadcast events down to child scopes, and emit events back up to parent scopes.

```javascript
$scope.$broadcast("event",data);
$scope.$emit("event",data);
$scope.$on("event",handlerFunction);
```

> Note: In general scope events should be used sparingly, the auto two-way data binding to model values can go a long way towards coordinating state changes across an application without the need for events. Events are best suited for notifying app actors of global async state changes that don't relate to state held in scope data models.

## Views

Views are live portions of the DOM, defined by valid HTML templates and associated with Angular code (directives, controllers etc.) via special attributes and expressions in the markup.

Views are glued to their respective controller via their shared `$scope` instance and the two way data binding that Angular sets up between the view's DOM and the model values.

> Note: Importantly views are declarative. They are focussed on describing *what* should happen rather *how* it should happen. The data model manipulation and mutation logic for UI behaviour should live in the view's controller, and any DOM manipulation code should live in directives.

### View expressions

Expressions are simple JavaScript-like code snippets inside HTML templates that operate within the data context provided by the current scope. They can access scope values, call scope methods or perform simple operations.

When possible expressions are automatically re-evaluated and re-rendered when the related scope value changes.

Render a scope value in a template:

```html
<span>{{user.firstname}}</span>
```

Render the result of a scope method call in a template:

```html
<span>Remaining issues: {{remainingIssues()}}
```

Perform a simple operation:

```html
<span>{{2+2}}</span>
```

## Directives

Directives are reusable blocks of Angular code/behaviour that can be attached to the live DOM via markers in the HTML. When Angular "compiles" the template HTML it will attempt to match directives with DOM elements based on these markers.

Directives are commonly used to attach specific behaviour to a DOM element or transform a DOM element's (and potentially its children's) structure.

Angular provides are large library of useful directives as part of the core framework, in addition you can define more application specific custom directives yourself.

> Note: Directive markers can be placed in HTML using a number of methods (comments, tag names, classes and attributes). The recommended approach however is to use attributes in a dash delimited format.

### Simple core directive examples

#### `ngClick`

Evaluates an expression when an element is clicked.

```html
<button ng-click="onBtnClicked()">Click me</button>
```

#### `ngRepeat`

Repeats the current element based on an expression.

```html
<li ng-repeat="user in users">
    {{user.name}}
</li>
```

#### `ngClass`

Adds class names to an element if an expression in an object hash of potential class names evaluates to true.

```html
<span ng-class="{'warning':shouldWarn()}"></span>
```

### Custom directives

Custom directives can be defined by returning a directive definition object from a factory function. The options object instructs the Angular compiler how to link a HTML template or DOM portion together with a scope in order to create a functioning directive.

```javascript
angular.module("app")
    .directive("myDirective",function () {
        return {
            // Directive definition options
        };
    });
```

The directive definition object can take many options which are detailed [here](http://docs.angularjs.org/api/ng/service/$compile), but the below is a brief discussion about some of the more important ones.

#### `scope`

This value defines how directive scope data (that is, attribute values set in the HTML template and/or values in the matched `$scope`) are exposed to the directive.

The default value is `false`/`null`, which means the directive shares the current scope.

A value of `true` creates a new scope for the directive that prototypically inherits from the current scope.

An object hash `{}` can define various patterns for setting up data binding between the current scope and an otherwise new/isolated directive scope. Useful when creating reusable directives that should be encapsulated from the surrounding data context.

Patterns available in the scope object hash:

##### `@` Attribute string expression binder

Binds a local property to the string result of an expression in an attribute the outer scope. Live changes are propagated into local scope. Example:

```html
<div my-directive my-attr="{{username}}"></div>
```

```javasctipt
scope: {
    localName: "@myAttr"
}
```

##### `=` Two way scope property binder

Two way binding between a local scope property and an outer scope property as defined by an attribute. Any change to the property in one scope will be reflected in the other.

```html
<div my-directive my-attr="username"></div>
```

```javascript
scope: {
    localUsername: "=myAttr"
}
```

##### `&` Expression invoker

Provides a mechanism for executing expressions in the context of the outer scope. Arguments can be passed via an object hash.

```html
<div my-directive my-attr="increment(count)"></div>
```

```javascript
scope: {
    increment: "&myAttr"
}
// Then later on in the directive:
$scope.increment({count:1});
```

##### `?` Optional flag

Adding the optional `?` flag will avoid errors if the property doesn't exist in the parent scope, for example `=?myAttr`.

> Note: by omitting the attribute name in the scope option object hash value string we indicate that the target attribute name is the same as the local isolate scope property name, e.g. `scope: { prop: "=" }` would bind to an attribute `prop` in the markup, such as `<div my-directive prop="username"></div>`.

#### `restrict`

Defines what sort of declaration style when encountered in the markup should be used to invoke the directive. Takes the form of a string containing a subset of the letters `EACM` - the presence of a letter allows the associated style. The default value is `A`.

For example, a directive named `myDirective` could be declared in the markup using the following styles:

- Attribute `<div my-directive></div>` `(A)`
- Element `<my-directive></my-directive>` `(E)`
- Class `<div class="my-directive"></div>` `!deprecated` `(C)`
- Comment `<!-- directive:my-directive -->` `!deprecated` `(M)`

> Note: Favour the default `A` attribute style of declaration, especially when decorating an existing element with extra behaviour. The `E` element name style can be considered when creating custom elements unique to your app.

#### `template`/`templateUrl`

Replaces, or appends to, the current element with the HTML in the template, either specified as a HTML string in the case of `template` or as a path to an external template in the case of `templateUrl`. All attributes and classes are migrated from the original element onto the new one.

#### `link`

Define a function with a signature `fn (scope,elem,attrs,ctrl)`. Used to attach DOM event listeners and manipulate the DOM based on UI interaction and changes in the scope model. Most directive logic goes here.

#### `controller`

Define a controller constructor function. The controller can be shared amoung directives thereby facilitating inter-directive communication.

### Custom directive examples

#### Template expansion

A simple use for custom directives can template expansion. Conceptually similar to partials this sort of directive can be used to avoid repeating often used chunks of HTML.

Directive definition:

```javascript
angular.module("app")
    .directive("customer-details",function () {
        return {
            templateUrl: "/templates/customer-details.html"
        }
    });
```

The HTML:

```html
<div customer-details></div>
```

The template:

```html
<p>Name: {{customer.name}}</p>
<p>Address: {{customer.address}}</p>
```

#### Scope isolation

It's often good practice to isolate directive scopes and reduce their coupling to the surrounding HTML structure and data context. We need to map properties in the outer scope onto local properties in a isolated scope inside the directive.

```javascript
angular.module("app")
    .controller("MyCtrl",function ($scope) {
           $scope.personA = "Jack";
           $scope.personB = "Jill";
    })
    .directive("myDirective",function () {
        return {
            template: "<p>{{localPerson}}</p>",
            scope: {
                localPerson: "="
            }
        }
    })
```

The HTML:

```html
<div my-ctrl>
	<div my-directive local-person="personA"></div>
	<div my-directive local-person="personB"></div>
</div>
```

Here we are mapping value of the `local-person` attribute to the `localPerson` value in the directive's isolated scope. The value of the attribute is a property name expression that resolves to a value in the parent controller's scope.

In this way two way data binding can be set up between a parent scope context and an encapsulated isolated directive scope. Note that the actual detail of the mapping is defined declaratively in the template.

#### DOM interaction

Typically when manipulating the directive's DOM and attaching DOM event listeners we use the `link` function option. The `link` function takes the following parameters:

- `scope` The Angular scope instance for the directive
- `element` The element matched by the directive, wrapped in jqLite
- `attrs` Object hash of normalised element attribute names and values

The `link` function is executed after Angular has processed the template and wired up the DOM, so its safe to transform the DOM and work with the scope.

```javascript
angular.module("app")
    .directive("myDirective",function () {
        return {
            link: function ($scope,element,attrs) {
                // Manipulate the DOM:
                element.text("foobar");
                // Watch an attribute expression for changes:
                $scope.$watch(attrs.fooAttribute,function (value) {});
                // Listen to a DOM event:
                element.on("mousedown",function (e) {});
                // Respond to the element being destroyed:
                element.on("$destroy",function () {});
            }
        }
    });
```

#### Transcluding directives

> "In computer science *transclusion* is the insertion of a document into another document at a particular point." - Wikipedia

In terms of Angular directives, transclusion describes the process of a directive wrapping the *contents* of the matched element with the result of its own compiled template. In other words, the matched element's contents is extracted and inserted into the directive's template at a particular point.

Consider a use case where arbitrary content should be wrapped in a HTML structure designed to turn it into an alert-style popup.

The original markup:
    
```html
<div alert-popup>
	<p>Alert! Something happened you should know about.</p>
</div>
```

The directive:

```javascript
angular.module("app")
    .directive("alertPopup",function () {
        return {
            transclude: true,
            template: "<div class='alert' ng-transclude></div>"
        }
    });
```

The resulting processed DOM:

```html    
<div alert-popup>
	<div class="alert" ng-transclude>
		<p>Alert! Something happened you should know about.</p>
	</div>
</div>
```

> Note: the `ng-transclude` directive in the template defines the position at which the transclusion should be inserted.

#### Inter-directive cooperation

Directives can communicate with other directives in the same scope. When directives depend on one another we can use the `require` option to enforce their presence. An example of a cooperating set of directives would be a tab-based navigation managing a set of content panes.

## Controllers

Controllers are constructor functions for JavaScript classes that are used to augment a view's scope with behaviour related to its particular data model.

Controllers are implemented by first preparing them for DI as a module component:

```javascript
angular.module("app")
    .controller("MyCtrl",function ($scope) {
        $scope.salutation = "Hello";
        $scope.salutationTarget = "world";
        $scope.num = 2;
        $scope.double = function (value) {
            return value*2;
        }
    });
```

And then referencing them in a view via the `ng-controller` directive:

```html
<div ng-controller="MyCtrl">
	<p>{{salutation}} {{salutationTarget}}</p>
	<p>{{num}} doubled is {{double(num)}}</p>
</div>
```

### Controller dos and don'ts

- **DO** make them slim. They should only contain simple, testable logic related only to the model data context for a single view.
- **DO** inject controllers with services if they need access to any shared state or functionality.
- **DON'T** manipulate the DOM in a controller, use the automatic data binding or core Angular directives library instead, or if you need to go further custom directives.
- **DON'T** filter output directly, use Angular filters.
- **DON'T** format input, use Angular form controls.

## Filters

Create data transformation "pipelines" either in template expressions or other app components. A set of basic filters are provided by the framework but additional custom ones can be defined.

### In views

For example in a template expression to limit a string to 80 chars and then transform it to all lowercase:

```html
{{myString | limitTo:80 | lowercase}}
```

Filters can also be applied to arrays, for example when reducing the elements outputted by the `ng-repeat` directive. In the code below only model items that contain an `active` property equal to `true` will be rendered.

```html
<li ng-repeat="item in items | filter{active:true}">
	{{item.firstName}} {{item.lastName}}
</li>
```

### In controllers

When using a filter directly in a controller then its DI name is formed from the pattern `<filter-name>Filter`, for example to inject a filter called `bar`:

```javascript
angular.module("foo")
	.controller("FooCtrl",function ($scope,$barFilter) {
		$scope.filteredValue = $barFilter(valueToFilter);
	});
```

As a rule its better to define complex filtering logic in a controller since this can be tested in isolation away from the DOM.

### Custom filters

Custom filters can also be defined via the `filter` method. The below is a simple custom filter for paginating an array:

```javascript
angular.module("app")
	.filter("paginate",function () {
		return function (array,page,length) {
			var start = page*length;
			return array.slice(start,start+length);
		};
});
```

### Filter dos and don'ts

- **DO** make them fast and efficient, they are typically "hot" code paths
- **DO** make them idempotent. Calling them multiple times shouldn't have side effects and they shouldn't modify passed-in data directly
- **DON'T** return HTML markup from filters, they should just operate abstractly on data

## Angular promises

Promises ...

## HTTP

The `$http` service is the core Angular utility for communicating with a backend. 

By default all `$http` request data objects are transformed into JSON and all responses are transformed into objects, if possible.

### Requests

Requests can be made with the HTTP method shortcuts:

```javascript
$http.get("/api/v1/endpoint");
```

Or by passing in a configuration object:

```javascript
$http({
    method: "GET",
    url: "/api/v1/endpoint",
    params: {key:"value"},
    data: {}
});
```

### Responses

Responses are exposed by Angular's `$q` promise implementation:

```javascript
$http.get("/api/v1/endpoint")
    .then(onSuccess,onError,onNotify)
    .done(onSuccess,onError,onNotify)
    .catch(onError)
    .progresa(onNotify)
    .finally(onSuccessOrError)
```

The `$http` service also provides the `success` and `error` methods on top of the above promise API:

```javascript
$http.get("/api/v1/endpoint")
    .success(function (data,status,headers,config) {
    })
    .error(function (data,status,headers,config) {
    });
```

### App wide HTTP setup

Any global error handling, authentication, pre or post-processing of requests or responses can be done via interceptors:

An example to globally detect a `403` response:

```javascript
angular.module("http.interceptors.auth")
    .factory("http403Interceptor",function ($q) {
        return {
            "responseError": function (res) {
                if ( res.status === 403 ) {
                    // Display error?
                    // Redirect to login?
                    // Etc
                }
                $q.reject(res);
            }
        }
    });
```

```javascript
angular.module("http.interceptors.auth")
    .config(function ($httpProvider) {
        $httpProvider.interceptors.push("http403Interceptor")
    });
```

### HTTP resources

When working with RESTful APIs we can abstract some of our HTTP code using the `$resource` service. This enables us to quickly define a clientside representation of a backend resource and then interact with it with a handy API.

Example here ...

### Custom REST adapter


## Templates

## Forms

## Routing

## Build

- AngularJS has an in-built module system that isolates app modules from the global scope and handles dependency resolution and DI automatically so solutions like RequireJS may not be strictly needed. AngularJS app files can just be concatenated together when built, no special processing or ordering of code blocks is required.
- If you use the "inferred" DI style then AngularJS code will break when minified. Either use the "inline annotation" DI style or a tool like `ngmin` in your build process to get around this.