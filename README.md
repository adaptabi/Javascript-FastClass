Javascript-fastClass
====================
A faster and easier way to define Javascript Prototypal Inheritance: `classes` and `mixins`


## Performance tests
*  Among <a href="http://jsperf.com/js-inheritance-performance/36" target="_blank"><code>popular libraries define + usage</code></a> - 3 classes and 3 methods * 500 instances each
*  <a href="http://jsperf.com/js-inheritance-performance/35" target="_blank"><code>Fastest libraries define + usage</code></a> - 3 classes and 3 methods * 500 instances each
*  <a href="http://jsperf.com/js-inheritance-performance/34" target="_blank"><code>Fastest libraries define only</code></a> - 3 classes and 3 methods * 1 instances each
*  <a href="http://htmlpreview.github.io/?https://github.com/dotnetwise/Javascript-FastClass/blob/master/src/Scripts/Tests/Index.html" target="_blank">Unit tests</a> - so you can see it just works

## Syntactic sugar
```javascript
var Vehicle = Function.define(function(name)){//define the "base class"
    this.name = name;//constructor initializer
}, { //optionally specify  prototype members i.e. to be added on Vechicle.prototype
    draw: function() { console.log("Drawing a " + this.name + " vehicle."); }
});//optionally specify extra mixins i.e. , Wheels);

var Car = Vechicle.inheritWith(function(base, baseCtor)){//define the "derrived class"
    return { //optionally specify  prototype members
        constructor: function(name, color) { //optionally specify a custom construcor
            baseCtor.call(this, name);//ensure you are calling the baseCtor
            this.color = color;
        },
        draw: function() { //redefine the draw function on Car.prototype
            console.log("Drawing a "+this.color+" car."); 
            base.draw.call(this);//optionally call the base class' draw method
        }
    }
}, Engine);//optionally specify extra mixins whose prortotype members will be copied to Car.prototype

//Define the Engine mixin
function Engine() {
    //this constructor will be automatically called when creating any class using it 
    //(including derrived classes). i.e. new Car() in this example    
    this.powerSource = 'petrol';//these properties will be added to every Car instance
    this.cmc = 1400;// i.e. var c= new Car(); c.cmc = 1400
    this.horsePower = 140;
}.define({//optioanlly add custom methods on Engine.prototype mixin
    //these will be copied to Car.prototype in our example, baecause it references the Engine mixin
    isElectric: function() { return this.pwerSource === 'electric'; }
})

var toyota = new Car("toyota prius", "red");//creating a new Car
toyota.powerSouce; //petrol
toyota.powerSource = 'electric'; //changing the powerSource property 
toyota.isElectric();//returns true
toyota.draw(); 
//Drawing a red car. 
//Drawing a tyota prius vechicle.
```

## Why yet another library?
<div align="center">
<img src="../../wiki/images/NugetIcon.png"/>
</div>

Native javascript inheritance is a pin in the ass. Even if you understand it perfectly it still requires some hideous repetivie code.

There are a lot of libraries which aim to help you with such that, but the main question is:

What is <a href="http://jsperf.com/js-inheritance-performance/36" target="_blank"><code>the fastest</code></a> vs <a target="_blank" href="https://github.com/njoubert/inheritance.js/blob/master/INHERITANCE.md"><code>most convenient</code></a> (also known as: with the most `[sugar](http://en.wikipedia.org/wiki/Syntactic_sugar)`) to create <a href="http://msdn.microsoft.com/en-us/magazine/ff852808.aspx" target="_blank"><code>Prototypal Inheritance</code></a> with?

## When to use it?
You might want this library
* when you can't use a language that <a href="https://github.com/jashkenas/coffee-script/wiki/List-of-languages-that-compile-to-JS" target="_blank"><code>compiles</code></a> into javascript code.
* because it is fast 
* you love `writing less - doing more!`

e.g. <a href="https://developers.google.com/closure/" target="_blank">Google Closure</a>, <a href="http://www.typescriptlang.org/Playground/" target="_blank">TypeScript</a>, <a href="http://arcturo.github.com/library/coffeescript/03_classes.html" target="_blank">Coffee Script</a> etc.

## What is it? 
FastClass is a very tiny library (~1KB minified and gzipped) that helps you quickly derrive your `classes` so to speak. 
It comes in two flavours:
* [`Function.prototype.fastClass(creator)`](#fastclass-flavour) - sets the `Base.prototype` to the `creator` function and then calls `new creator(this.prototype, this)`
```javascript
function(base, baseCtor) { this.somePrototypeMethod1 =  ...; this.somePrototypeMethod2 =  ...; } }
```

* [`Function.prototype.inheritWith(creator)`](#inheritwith-flavour) - **recommended** returns a `plain object` containing the members of the new prototype, including the constructor itself

It makes usage of __proto__ on all new browsers (which makes it blazing fast) except `Internet Explorer` and maybe other ancient browsers where it fallbacks to `for (var key in obj)` statement.

Note `__proto__` will become standard in <a href="http://javascript.spec.whatwg.org/#object.prototype.__proto__" target="_blank">ECMAScript 6</a>
```javascript
function(base, baseCtor) { return { somePrototypeMethod1: ..., somePrototypeMethod2: ...} }
```
whereas `baseCtor` is the function we want to inherit and base is it's prototype *(`baseCtor.prototype` that is).*


## How to use it?

### Defining the base "class"
```javascript
var Figure = function(name) {
    this.name = name;
    Function.initMixins(this);// since this is a top function, we need to call initMixins to make sure they will be initialized.
}
Figure.prototype.draw = function() { console.log("figure " + name); }
```

### `.define` sugar

You can define the first `class' constructor function` (same as above but with sugar syntax) as following:

```javascript
var A = function(name) { 
    this.name = name; 
    Function.initMixins(this);// since this is a top function, we need to call initMixins to make sure they will be initialized.
}.define({
    draw: function() {
        console.log("figure "+ this.name);
    }
}/*, add any mixins here */) 
```

The `define` function copies all the members of the returned object to the `function.prorortpe` *(`A.prototype`)* and returns it *(`A`)*

### `Function.define` even better sugar - **recommended**
This way we don't need to call `Function.initMixins(this)` as it will be automatically called for us
```javascript
var A = Function.define(function(name){
    this.name = name;
}, {
    draw: function() {//method added on function(name) {}.prototype
        console.log("figure " + this.name);
    }
}
```

Alternatively you can pass an object with the costructor function specified (similar syntax to `.inheritWith`)
```javascript
var A = Function.define({
    constructor: function(name) {//the constructor itself can be even missing. If so we will add one for you!
        this.name = name;
    },
    draw: function() {//method added on constructor.prototype
        console.log("figure " + this.name);
    }
}
```
All of the above methods are doing the same thing. You decide which one better suits you.

## Inheritance

A classical example to use inheritance when you have a base class called `Figure` and a derrived class called `Square`.

#### `.fastClass` flvaour

To define the `derrived class` Square:
```javascript
var Square = Figure.fastClass(function(base, baseCtor) {
    this.constructor = function(name, length) { 
      this.length = length;
      baseCtor.call(this, name);//calls the bacse ctor
    }
    this.draw = function() {//redefines the draw function
      console.log("square with length " + this.length);
      base.draw.call(this);//calls the base class' draw function
    }
})
```

#### `.inheritWith` flavour 

To define the `derrived class` Square:
```javascript
var Square = Figure.inheritWith(function(base, baseCtor) {
    return { 
        constructor:  function(name, length) { 
            this.length = length;
            baseCtor.call(this, name);//calls the bacse ctor
        },
        draw: function() {//redefines the draw function
            console.log("square with length " + this.length);
            base.draw.call(this);//calls the base class' draw function
        }
    }
})
```

As you can see in both cases the definition is pretty simple and very similar. 

However the `.inheritWith` flavour comes with about 5-15% <a href="http://jsperf.com/js-inheritance-performance/34" target="_blank"><code>performance boost</code></a> depending on the actual browser and number of members.
That is because we are simply setting `__proto__` with the BaseClass.prototype for those browsers who support it (all nowdays browsers except `IE<=10`)

### The constructor
In both cases we the `constructor` is the function that is returned as the `derrived class' constructor`. 
The `constructor` is optional and we should only add it when we have some custom code as both functions will add it for us otherswise.

#### Usage

Whichever flavour you choose the code usage is the same. Firstly you need to instantiate the Constructor with the `new` operator:
```javascript
var figure = new Figure("generic");
figure.draw();
//figure generic
var square = new Square("10cm square", 10);
figure.draw(); 
//square with length 10
//figure 10cm square
```

## Mixins support
<a href="http://www.joezimjs.com/javascript/javascript-mixins-functional-inheritance/" target="_blank">Mixins</a> are some grouped functionalities that you can add to a `class` without inheritance

You can <a href="http://programmers.stackexchange.com/questions/123342/inheritance-vs-mixins-in-dynamic-languages" target="_blank">learn more</a> about `mixins` vs `inheritance` on <a href="http://programmers.stackexchange.com/questions/123342/inheritance-vs-mixins-in-dynamic-languages" target="_blank">this post</a>.

We can define mixins as 
* an `object` containing functions
* a `function` which will be executed for every instance of the class that is using it
    * The `prototype` of the function will be automatically copied to the `class.prototype` that are using it

### Another Example

```javascript
// Animal base class
function Animal() {
    // Private
    function private1(){}
    function private2(){}

    // Privileged - on instance
    this.privileged1 = function(){}
    this.privileged2 = function(){}
    Function.initMixins(this);
}.define({ 
    // Public - on prototype
    method1: function() { console.log("Animal::method1"); }
});
```

Alternatively the above can be defined as:
```javascript
var Animal = Function.define(function(){
    // Private 
    function private1() {}
    function private2() {}
     // Privileged - on instance
    this.privileged1 = function(){}
    this.privileged2 = function(){}
    }, {
        method1: function() { console.log("Animal::method1"); }
    };
});
```

Or even simpler as an object providing the `constructor`:
```javascript
var Animal = Function.define({
    constructor: function(){ // the constructor is optional
        // Private 
        function private1() {}
        function private2() {}
         // Privileged - on instance
        this.privileged1 = function(){}
        this.privileged2 = function(){}
    },
    method1: function() { console.log("Animal::method1"); }
});
```

The `function Animal` is the `function` given as first parameter. 
It acts as the constructor, which is invoked when an instance is created:

```javascript
var animal = new Animal(); // Create a new Animal instance
```

### Spicing it up with inheritance and mixins

We can typically define a `move` action which is a set of methods grouped in our `Move mixin`

#### Defining the mixin
```javascript
function Move() {//define the constructor of the mixin. This will be automatically called for every instance in the constructor of the class that is using this mixin
    this.position = { x: 0, y: 0};
}.define({//define mixin's prototype. These will be copied to the prototype of the classes that will use this `mixin`
   moveTo: function(x, y) {
       this.position.x = x;
       this.position.y = y;
   },
   resetPosition: function() {
       this.position.x = 0; 
       this.position.y = 0;
   },
   logPosition: function() {
       console.log("Position: ", this.position);
   }
});
```

### Adding Inheritance and using the `Move` mixin

#### Usage using `.inheritWith` flavour

`[[constructor]].inheritWith( function(base, baseCtor) { return {...} }, mixins )` - that function should `return` methods for the derrived prototype

#### Example
```javascript
// Extend the Animal class with inheritWith flavor
var Dog = Animal.inheritWith(function(base, baseCtor) {
    //derrived class containing some private method(s)
    function someOtherPrivateMethod() { }
    
    return {
        // Override base class `method1`
        method1: function(){
            someOtherPrivateMethod.call(this);
            base.method1.call(this);//calling a methd from the base class: Animal.prtotype.method1
            console.log('Dog::method1');
        },
        scare: function(){
            console.log('Dog::I scare you');
        },
        //some new public method(s)
        method2: function() { }
    }
}, Move);//specify any extra mixins to be added to the Dog's prototype
```

Create an instance of `Dog`:

```javascript
var husky = new Dog();
husky.method1(); 
// Animal::method1
// Dog::method1

husky.scare();
// Dog::I scare you'


/// testing the mixin:
husky.logPosition(); // Call the method of the mixin. 
// Position: {x: 0, y: 0}

husky.moveTo(10, 20);  
husky.logPosition();
// Position: {x: 10, y: 20}

husky.resetPosition();
husky.logPosition();
// Position: {x: 0, y: 0}
```

#### Same as above using `.fastClass` flavour

`[[constructor]].fastClass( function(base, baseCtor) { this.m = function {...} ... } )` - that function `populates` the derrived prototype
Every class definition has access to the parent's prototype via the first argument passed into the function. The second argument is the `base Class` itself (its constructor i.e. `Animal` in our case):

```javascript
// Same as above but Extend the Animal class using fastClass flavor
var Dog = Animal.fastClass(function(base, baseCtor) {
    //private functions
    function someOtherPrivateMethod() {};
    
    //since we don't set this.constructor, automatically will be added one for us which will call the baseCtor with all the provided arguments
    //this is importat so we can set the new prototype into it
    
    // Override base class `method1`
    this.method1 = function(){
        someOtherPrivateMethod.call(this);
        // Call the parent function
        base.method1.call(this);
        console.log('Dog::method1');
    };
    this.scare = function(){
        console.log('Dog::I scare you');
    };
    //some more public functions
    this.method2 = function() {}; 
}, Move);
```
### Static members
You can sometime extend a constructor or a function with some static methods
In order to do so you need to call `.defineStatic({...})`

```javascript
var F = function() {}.defineStatic({
    loadData: function() {...}
}).
typeof F.loadData // returns function
F.loadData() //call the static function
```

This can be easily mixed with `Function.define`
```javascript
var F = Function.define({
    constructor: function() {}
).defineStatic({//make sure you are adding it here and not on the constructor function itself. 
//that is because Function.define will automatically declare a function in order 
//to ensure Function.initMixins gets automatically called

//add any static members here
    loadData: function() {...}
};
typeof F.loadData // returns function
F.loadData() //call the static function
```

### Mixin `abstract` methods
The `Function.initMixins` will always check for duplicate members that are already defined in the object i.e. the class defines a member whith the same name, or another mixin defines it.
If such collision is found an exception will be thorwn when using the debug version of the library. Using the `.min` file there is no error and the member will be overriden.

However in order to prevent this exception you can define a function as abstract with `.defineStatic(obj)`:
```javascript
var Animal = Function.define({    
    method1: function() { 
        throw new Exception("You need to implement method1 on your class's or mixin's prototype")
    }.defineStatic({abstract: true});//make this function abstract so it can be overriden by mixins
});
```

### Abstract methods sugar
```javascript
var f = Function.abstract();
f();//will trigger a failed assert with message "Not implemented"
var g = Function.abstract("Custom not implemented message");
g();//will trigger a failed assert with message "Custom not implemented message"
f.abstract === true;//true
var h = Function.abstract(function() { /* will be called before the assert "Not implemented" will be thrown */});
h();//will call the given function and then will trigger a failed assert with message "Not implemented"
var j = Function.abstract("Custom not implemented message", function() { /* will be called before the assert "Not implemented" will be thrown */});
j();//will call the given function and then will trigger a failed assert with message "Custom not implemented message"
```

## Where to get it from?
Beside GitHub, you can download it as a <a href="http://nuget.org/packages/Javascript-FastClass/" target="_blank"><code>Nuget package</code></a> in Visual Studio from<a href="http://nuget.org/packages/Javascript-FastClass/" target="_blank"><code>here</code></a>
```javascript
Install-Package Javascript-FastClass
```

## What's next?
* [Private methods support!](../../wiki/Private-Methods)
* [Self test](../../wiki/Self-Test)

Do you have a better & faster way? Share it! We would love to seeing creativity in action!

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/ced79a6263a52ce6aed7515d0cd0b0f3 "githalytics.com")](http://githalytics.com/dotnetwise/Javascript-FastClass)
