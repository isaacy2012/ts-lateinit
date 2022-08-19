[![npm version](https://img.shields.io/npm/v/ts-lateinit.svg?logo=npm&style=for-the-badge)](https://www.npmjs.com/package/typedjson)
[![license](https://img.shields.io/npm/l/ts-lateinit?&style=for-the-badge&color=green)
](https://github.com/isaacy2012/ts-lateinit/blob/master/LICENSE)

Late initialization decorator for TypeScript. Comes with `@lateinit` and `@readonlyLateinit` variants for controlling 
the mutability of the property.

## Installation

TypedJSON is available from npm:

```
npm install ts-lateinit
```

## How to Use

TypeScript needs to run with the `experimentalDecorators` option enabled.

You must use the [definite assignment assertion](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-7.html#strict-class-initialization)
if the property type is not optional (i.e not `T | undefined`):
```typescript
class Person {
  @lateinit()
  name!: string;
  //  ^ definite assignment assertion
    
  @lateinit()
  nullableName: string | undefined;
  //  no definite assignment required
}
```

### Mutable `@lateinit` Property

```typescript
import { lateinit } from "ts-lateinit"

class Person {
  @lateinit()
  name!: string;
}

const person = new Person();
person.name = "Alice";
console.log(person.name); // prints "Alice"
person.name = "Bob";
console.log(person.name); // prints "Bob"

const anotherPerson = new Person();
console.log(anotherPerson.name); // throws a LateinitNotInitializedException
```

### Immutable `@readonlyLateinit` Property

```typescript
import { readonlyLateinit } from "ts-lateinit"

class Person {
    @readonlyLateinit()
    name!: string;
}

const person = new Person();
person.name = "Alice";
console.log(person.name); // prints "Alice"
person.name = "Bob"; // throws a ReadonlyLateinitAlreadyInitializedException
```

### Ignoring initial set to `undefined`

`@lateinit` and `@readonlyLateInit` accept an optional parameter `options` for which if `ignoreInitialUndefined` is `true`,
ts-lateinit will ignore the first "set" action on the property if it is `undefined`. 

This can be helpful when using reflection to instantiate a class, for example when using serialization/deserialization
libraries such as [TypedJson](https://www.npmjs.com/package/typedjson).

```typescript
import { readonlyLateinit } from "ts-lateinit"

class Person {
    @readonlyLateinit({ ignoreInitialUndefined: true })
    name!: string;
}

const person = new Person();
person.name = "Alice";
console.log(person.name); // prints "Alice"
person.name = "Bob"; // throws a ReadonlyLateinitAlreadyInitializedException
```

### Checking if a Property Has Been Initialized

To check whether the property has been initialized, use the `isInitialized(thisRef: any, propertyKey: string)` function.

```typescript
import { readonlyLateinit, isInitialized } from "ts-lateinit"

class Person {
    @readonlyLateinit()
    name!: string;
}

const person = new Person();
console.log(isInitialized(person, "name")); // prints "false"
person.name = "Alice";
console.log(isInitialized(person, "name")); // prints "true"
```

However, if you find you are using `isInitialized` a lot, you may wish to consider using a optional type instead 
to take advantage of TypeScripts optional chaining features such as `?` and `??`.

## Notes
ts-lateinit adds two extra properties for each property that is defined as lateinit, prefixed with `__ts-lateinit_SET__` 
and `__ts-lateinit_VAL__`, to keep track of whether the property was set and the value respectively. 

For example, for the above example with the property `name`, the two extra properties would be `__ts-lateinit_SET__name` 
and `__ts-lateinit_VAL__name`

Ensure that these properties are not overwritten. 

## License

ts-lateinit is licensed under the ISC License.
