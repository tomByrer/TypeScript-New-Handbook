# Classes

[Background reading: Classes (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

TypeScript offers full support for the `class` keyword introduced in ES2015.
As with other JavaScript language features, TypeScript adds type annotations and other syntax to allow you to express relationships between classes and other types.

__toc__

## Class Members

Here's the most basic class - an empty one:

```ts
class Point {

}
```

This class isn't very useful yet, so let's start adding some members.

### Fields

A field declaration creates a public writeable property on a class:

```ts
// @strictPropertyInitialization: false
class Point {
    x: number;
    y: number;
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

As with other locations, the type annotation is optional, but will be an implict `any` if not specified.

Fields can also have *initializers*; these will run automatically when the class is instantiated:

```ts
class Point {
    x = 0;
    y = 0;
}

const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

Just like with `const`, `let`, and `var`, the initializer of a class property will be used to infer its type:

```ts
class Point {
    x = 0;
    y = 0;
}
//cut
const pt = new Point();
pt.x = "0";
```

#### `--strictPropertyInitialization`

The [[strictPropertyInitialization]] setting controls whether class fields need to be initialized in the constructor.

```ts
class BadGreeter {
  name: string;
}
```

```ts
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}
```

Note that the field needs to be initialized *in the constructor itself*.
TypeScript does not analyze methods you invoke from the constructor to detect initializations, because a derived class might override those methods and fail to initialize the members.

If you intend to definitely initialize a field through means other than the constructor (for example, maybe an external library is filling in part of your class for you), you can use the *definite assignment assertion operator*, `!`:

```ts
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}
```

### `readonly` {#readonly-class-properties}

Fields may be prefixed with the `readonly` modifier.
This prevents assignments to the field outside of the constructor.

```ts
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
  }
}
const g = new Greeter();
g.name = "also not ok";
```

### Constructors

[Background Reading: Constructor (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor)

Class constructors are very similar to functions.
You can add parameters with type annotations, default values, and overloads:

```ts
class Point {
  x: number;
  y: number;

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // TBD
  }
}
```

There are just a few differences between class constructor signatures and function signatures:

* Constructors can't have type parameters - these belong on the outer class declaration, which we'll learn about later
* Constructors can't have return type annotations - the class instance type is always what's returned

#### Super Calls

Just as in JavaScript, if you have a base class, you'll need to call `super();` in your constructor body before using any `this.` members:

```ts
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k);
    super();
  }
}
```

Forgetting to call `super` is an easy mistake to make in JavaScript, but TypeScript will tell you when it's necessary.

### Methods

>> [Background Reading: Method definitions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions)

A function property on a class is called a *method*.
Methods can use all the same type annotations as functions and constructors:

```ts
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

Other than the standard type annotations, TypeScript doesn't add anything else new to methods.

Note that inside a method body, it is still mandatory to access fields and other methods via `this.`.
An unqualified name in a method body will always refer to something in the enclosing scope:

```ts
let x: number = 0;

class C {
  x: string = "hello";

  m() {
    // This is trying to modify 'x' from line 1, not the class property
    x = "world";
  }
}
```

### Getters / Setters

Classes can also have *accessors*:

```ts
class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

> Note that a field-backed get/set pair with no extra logic is very rarely useful in JavaScript.
> It's fine to expose public fields if you don't need to add additional logic during the get/set operations.

TypeScript has some special inference rules for accessors:

* If no `set` exists, the property is automatically `readonly`
* The type of the setter parameter is inferred from the return type of the getter
* If the setter parameter has a type annotation, it must match the return type of the getter
* Getters and setters must have the same [[Member Visibility]]

It is not possible to have accessors with different types for getting and setting.

If you have a getter without a setter, the field is automatically `readonly`

### Index Signatures {#class-index-signatures}

Classes can declare index signatures; these work the same as [[Index Signatures]] for other object types:

```ts
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);
  check(s: string) {
    return this[s] as boolean;
  }
}
```

Because the index signature type needs to also capture the types of methods, it's not easy to usefully use these types.
Generally it's better to store indexed data in another place instead of on the class instance itself.

## Class Heritage

Like other langauges with object-oriented features, classes in JavaScript can inherit from base classes.

### `implements` Clauses

You can use an `implements` clause to check that a class satisfies a particular `interface`.
An error will be issued if a class fails to correctly implement it:

```ts
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log('ping!');
  }
}

class Ball implements Pingable {
  pong() {
    console.log('pong!');
  }
}
```

Classes may also implement multiple interfaces, e.g. `class C implements A, B {`.

#### Cautions

It's important to understand that an `implements` clause is only a check that the class can be treated as the interface type.
It doesn't change the type of the class or its methods *at all*.
A common source of error is to assume that an `implements` clause will change the class type - it doesn't!

```ts
// @noImplicitAny: false
interface Checkable {
  check(name: string): boolean;
}
class NameChecker implements Checkable {
  check(s) {
    // Notice no error here
    return s.toLowercse() === "ok";
           ^?
  }
}
```

In this example, we perhaps expected that `s`'s type would be influenced by the `name: string` parameter of `check`.
It is not - `implements` clauses don't change how the class body is checked or its type inferred.

Similarly, implementing an interface with an optional property doesn't create that property:

```ts
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
```

### `extends` Clauses

>> [Background Reading: extends keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends)

Classes may `extend` from a base class.
A derived class has all the properties and methods of its base class, and also define additional members.

```ts
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### Overriding Methods

>> [Background reading: super keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super)

A derived class can also override a base class field or property.
You can use the `super.` syntax to access base class methods.
Note that because JavaScript classes are a simple lookup object, there is no notion of a "super field".

TypeScript enforces that a derived class is always a subtype of its base class.

For example, here's a legal way to override a method:

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

It's important that a derived class follow its base class contract.
Remember that it's very common (and always legal!) to refer to a derived class instance through a base class reference:

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
declare const d: Base;
//cut
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet();
```

What if `Derived` didn't follow `Base`'s contract?

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  // Make this parameter required
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

If we compiled this code despite the error, this sample would then crash:

```ts
declare class Base  { greet(): void };
declare class Derived extends Base { }
//cut
const b: Base = new Derived();
// Crashes because "name" will be undefined
b.greet();
```

#### Initialization Order

The order that JavaScript classes initialize can be surprising in some cases.
Let's consider this code:

```ts
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + name);
  }
}

class Derived extends Base {
  name = "derived";
}

// Prints "base", not "derived"
const d = new Derived();
```

What happened here?

The order of class initialization, as defined by JavaScript, is:
 * The base class fields are initialized
 * The base class constructor runs
 * The derived class fields are initialized
 * The derived class constructor runs

This means that the base class constructor saw its own value for `name` during its own constructor, because the derived class field initializations hadn't run yet.

#### Inheriting Built-in Types

> Note: If you don't plan to inherit from built-in types like `Array`, `Error`, `Map`, etc., you may skip this section

In ES2015, constructors which return an object implicitly substitute the value of `this` for any callers of `super(...)`.
It is necessary for generated constructor code to capture any potential return value of `super(...)` and replace it with `this`.

As a result, subclassing `Error`, `Array`, and others may no longer work as expected.
This is due to the fact that constructor functions for `Error`, `Array`, and the like use ECMAScript 6's `new.target` to adjust the prototype chain;
however, there is no way to ensure a value for `new.target` when invoking a constructor in ECMAScript 5.
Other downlevel compilers generally have the same limitation by default.

For a subclass like the following:

```ts
class FooError extends Error {
    constructor(m: string) {
        super(m);
    }
    sayHello() {
        return "hello " + this.message;
    }
}
```

you may find that:

* methods may be `undefined` on objects returned by constructing these subclasses, so calling `sayHello` will result in an error.
* `instanceof` will be broken between instances of the subclass and their instances, so `(new FooError()) instanceof FooError` will return `false`.

As a recommendation, you can manually adjust the prototype immediately after any `super(...)` calls.

```ts
class FooError extends Error {
    constructor(m: string) {
        super(m);

        // Set the prototype explicitly.
        Object.setPrototypeOf(this, FooError.prototype);
    }

    sayHello() {
        return "hello " + this.message;
    }
}
```

However, any subclass of `FooError` will have to manually set the prototype as well.
For runtimes that don't support [`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf), you may instead be able to use [`__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto).

Unfortunately, [these workarounds will not work on Internet Explorer 10 and prior](https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx).
One can manually copy methods from the prototype onto the instance itself (i.e. `FooError.prototype` onto `this`), but the prototype chain itself cannot be fixed.

## Member Visibility

You can use TypeScript to control whether certain methods or properties are visible to code outside the class.

### `public`

The default visibility of class members is `public`.
A `public` member can be accessed by anywhere:

```ts
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```

Because `public` is already the default visibility modifier, you don't ever *need* to write it on a class member, but might choose to do so for style/readability reasons.

### `protected`

`protected` members are only visible to subclasses of the class they're declared in.

```ts
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
                            ^^^^^^^^^^^^^^
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
```

#### Exposure of `protected` members

Derived classes need to follow their base class contracts, but may choose to expose a more general type with more capabilities.
This includes making `protected` members `public`:

```ts
class Base {
  protected m = 10;
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```

Note that `Derived` was already able to freely read and write `m`, so this doesn't meaningfully alter the "security" of this situation.
The main thing to note here is that in the derived class, we need to be careful to repeat the `protected` modifier if this exposure isn't intentional.

#### Cross-hierarchy `protected` access

Different OOP languages disagree about whether it's legal to access a `protected` member through a base class reference:

```ts
class Base {
    protected x: number = 1;
}
class Derived1 extends Base {
    protected x: number = 5;
}
class Derived2 extends Base {
    f1(other: Derived2) {
        other.x = 10;
    }
    f2(other: Base) {
        other.x = 10;
    }
}
```

Java, for example, considers this to be legal.
On the other hand, C# and C++ chose that this code should be illegal.

TypeScript sides with C# and C++ here, because accessing `x` in `Derived2` should only be legal from `Derived2`'s subclasses, and `Derived1` isn't one of them.
Moreover, if accessing `x` through a `Derived2` reference is illegal (which it certainly should be!), then accessing it through a base class reference should never improve the situation.

See also [Why Can’t I Access A Protected Member From A Derived Class?](https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected-member-from-a-derived-class/) which explains more of C#'s reasoning.

### `private`

`private` is like `protected`, but doesn't allow access to the member even from subclasses:

```ts
class Base {
  private x = 0;
}
const b = new Base();
// Can't access from outside the class
console.log(b.x);
```

```ts
class Base {
  private x = 0;
}
//cut
class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x);
  }
}
```

Because `private` members aren't visible to derived classes, a derived class can't increase its visibility:


```ts
class Base {
  private x = 0;
}
class Derived extends Base {
  x = 1;
}
```

#### Cross-instance `private` access

Different OOP languages disagree about whether different instances of the same class may access each others' `private` members.
While languages like Java, C#, C++, Swift, and PHP allow this, Ruby does not.

TypeScript does allow cross-instance `private` access:

```ts
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
```

#### Caveats {#private-and-runtime-privacy}

Like other aspects of TypeScript's type system, `private` and `protected` are only enforced during type checking.
This means that JavaScript runtime constructs like `in` or simple property lookup can still access a `private` or `protected` member:

```ts
class MySafe {
  private secretKey = 12345;
}
```

```js
// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

If you need to protect values in your class from malicious actors, you should use mechanisms that offer hard runtime privacy, such as closures, weak maps, or [[private fields]].

## Static Members

>> [Background Reading: Static Members (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static)

Classes may have `static` members.
These members aren't associated with a particular instance of the class.
They can be accessed through the class constructor object itself:

```ts
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

Static members can also use the same `public`, `protected`, and `private` visibility modifiers:

```ts
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
```

Static members are also inherited:

```ts
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

### Special Static Names

It's generally not safe/possible to overwrite properties from the `Function` prototype.
Because classes are themselves functions that can be invoked with `new`, certain `static` names can't be used.
Function properties like `name`, `length`, and `call` aren't valid to define as `static` members:

```ts
class S {
  static name = "S!";
}
```

### Why No Static Classes?

TypeScript (and JavaScript) don't have a construct called `static class` the same way C# and Java do.

Those constructs *only* exist because those languages force all data and functions to be inside a class; because that restriction doesn't exist in TypeScript, there's no need for them.
A class with only a single instance is typically just represented as a normal *object* in JavaScript/TypeScript.

For example, we don't need a "static class" syntax in TypeScript because a regular object (or even top-level function) will do the job just as well:

```ts
// Unnecessary "static" class
class MyStaticClass {
    static doSomething() {

    }
}

// Preferred (alternative 1)
function doSomething() {

}

// Preferred (alternative 2)
const MyHelperObject = {
    dosomething() {

    }
}
```

## Generic Classes

Classes, much like interfaces, can be generic.
When a generic class is instantiated with `new`, its type parameters are inferred the same way as in a function call:

```ts
class Box<T> {
  contents: T;
  constructor(value: T) {
    this.contents = value;
  }
}

const b = new Box("hello!");
      ^?
```

Classes can use generic constraints and defaults the same way as interfaces.

### Type Parameters in Static Members

This code isn't legal, and it may not be obvious why:

```ts
class Box<T> {
  static defaultValue: T;
}
```

Remember that types are always fully erased!
At runtime, there's only *one* `Box.defaultValue` property slot.
This means that setting `Box<string>.defaultValue` (if that were possible) would *also* change `Box<number>.defaultValue` - not good.
The `static` members of a generic class can never refer to the class's type parameters.

## `this` at Runtime in Classes {#runtime-this-in-classes}

>> [Background Reading: `this` keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)

It's important to remember that TypeScript doesn't change the runtime behavior of JavaScript, and that JavaScript is somewhat famous for having some peculiar runtime behaviors.

JavaScript's handling of `this` is indeed unusual:

```ts
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName
};

// Prints "obj", not "MyClass"
console.log(obj.getName());
```

Long story short, by default, the value of `this` inside a function depends on *how the function was called*.
In this example, because the function was called through the `obj` reference, its value of `this` was `obj` rather than the class instance.

This is rarely what you want to happen!
TypeScript provides some ways to mitigate or prevent this kind of error.

### Arrow Functions

>> [Background Reading: Arrow functions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

If you have a function that will often be called in a way that loses its `this` context, it can make sense to use an arrow function property instead of a method definition:

```ts
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  }
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```

This has some trade-offs:
 * The `this` value is guaranteed to be correct at runtime, even for code not checked with TypeScript
 * This will use more memory, because each class instance will have its own copy of each function defined this way
 * You can't use `super.getName` in a derived class, because there's no entry in the prototype chain to fetch the base class method from

### `this` parameters

In a method or function definition, an initial parameter named `this` has special meaning in TypeScript.
These parameters are erased during compilation:

```ts
type SomeType = any
//cut
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}
```
```js
// JavaScript output
function fn(x) {
  /* ... */
}
```

TypeScript checks that calling a function with a `this` parameter is done so with a correct context.
Instead of using an arrow function, we can add a `this` parameter to method definitions to statically enforce that the method is called correctly:

```ts
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// OK
c.getName();

// Error, would crash
const g = c.getName;
console.log(g());
```

This method takes the opposite trade-offs of the arrow function approach:
 * JavaScript callers might still use the class method incorrectly without realizing it
 * Only one function per class definition gets allocated, rather than one per class instance
 * Base method definitions can still be called via `super.`

## `this` Types

In classes, a special type called `this` refers *dynamically* to the type of the current class.
Let's see how this is useful:

```ts
class Box {
  contents: string = "";
  set(value: string) {
   ^?
    this.contents = value;
    return this;
  }
}
```

Here, TypeScript inferred the return type of `set` to be `this`, rather than `Box`.
Now let's make a subclass of `Box`:

```ts
class Box {
  contents: string = "";
  set(value: string) {
    this.contents = value;
    return this;
  }
}
//cut
class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}

const a = new ClearableBox();
const b = a.set("hello");
      ^?
```

You can also use `this` in a parameter type annotation:

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

This is different from writing `other: Box` -- if you have a derived class, its `sameAs` method will now only accept other instances of that same derived class:

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box  {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
```

## Parameter Properties

TypeScript offers special syntax for turning a constructor parameter into a class property with the same name and value.
These are called *parameter properties* and are created by prefixing a constructor argument with one of the visibility modifiers `public`, `private`, `protected`, or `readonly`.
The resulting field gets those modifier(s):

```ts
class A {
  constructor (public readonly x: number, protected y: number, private z: number) {
    // No body necessary
  }
}
const a = new A(1, 2, 3);
console.log(a.x);
              ^?
console.log(a.z);
```

## Class Expressions

>> [Background reading: Class expressions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class)

Class expressions are very similar to class declarations.
The only real difference is that class expressions don't need a name, though we can refer to them via whatever identifier they ended up bound to:

```ts
const someClass = class<T> {
  content: T;
  constructor(value: T) {
    this.content = value;
  }
}

const m = new someClass("Hello, world");
      ^?
```

## `abstract` Classes and Members

Classes, methods, and fields in TypeScript may be *abstract*.

An *abstract method* or *abstract field* is one that hasn't had an implementation provided.
These members must exist inside an *abstract class*, which cannot be directly instantiated.

The role of abstract classes is to serve as a base class for subclasses which do implement all the abstract members.
When a class doesn't have any abstract members, it is said to be *concrete*.

Let's look at an example

```ts
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
```

We can't instantiate `Base` with `new` because it's abstract.
Instead, we need to make a derived class and implement the abstract members:

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
//cut
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

Notice that if we forget to implement the base class's abstract members, we'll get an error:
```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
//cut
class Derived extends Base {
  // forgot to do anything
}
```

### Abstract Construct Signatures

Sometimes you want to accept some class constructor function that produces an instance of a class which derives from some abstract class.

For example, you might want to write this code:

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
class Derived extends Base { getName() { return "" } }
//cut
function greet(ctor: typeof Base) {
  const instance = new ctor();
  instance.printName();
}
```

TypeScript is correctly telling you that you're trying to instantiate an abstract class.
After all, given the definition of `greet`, it's perfectly legal to write this code, which would end up constructing an abstract class:

```ts
declare const greet: any, Base: any;
//cut
// Bad!
greet(Base);
```

Instead, you want to write a function that accepts something with a construct signature:

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
class Derived extends Base { getName() { return "" } }
//cut
function greet(ctor: new() => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base);
```

Now TypeScript correctly tells you about which class constructor functions can be invoked - `Derived` can because it's concrete, but `Base` cannot.

## Relationships Between Classes

In most cases, classes in TypeScript are compared structurally, the same as other types.

For example, these two classes can be used in place of each other because they're identical:

```ts
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

// OK
const p: Point1 = new Point2();
```

Similarly, subtype relationships between classes exist even if there's no explicit inheritance:

```ts
// @strict: false
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

// OK
const p: Person = new Employee();
```

This sounds straightforward, but there are a few cases that seem stranger than others.

Empty classes have no members.
In a structural type system, a type with no members is generally a supertype of anything else.
So if you write an empty class (don't!), anything can be used in place of it:

```ts
class Empty { }

function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}

// All OK!
fn(window);
fn({ });
fn(fn);
```
