---
title: Comprehensive list of built-in utility types in TypeScript
date: 2019-03-09 19:55:34
icon: fas fa-check
image: /images/posts/checklist.jpg
tags:
    - typescript
    - advanced types
---

[Advanced types](https://www.typescriptlang.org/docs/handbook/advanced-types.html) section of TypeScript docs mentions some very useful built-in types as examples of conditional types and mapped types. 

I was surprised to learn that there are more such types and some of them seem to be undocumented. This article contains a list of all such types.

The list is based on what I could find in `es5.d.ts` on [github](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts).

## List of types

### `Partial`

`Partial<T>` returns a type that has the same properties as `T` but all of them are optional. This is mostly useful when `strictNullChecks` flag is enabled.

`Partial` works on a single level - it doesn't affect nested objects.

This type is useful in many situations. One that comes to my mind is when you need to type a function that lets you override default values of properties of some object.

```typescript
const defaultSettings: Settings = { /* ... */ };

function getSettings(custom: Partial<Settings>) {
  return { ...defaultSettings, ...custom };
}
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1404).

### `Required`

`Required<T>` removes optionality from `T`'s properties. Again, you'll most likely need it if you have `strictNullChecks` enabled (which you should 😉).

Similarly to `Required`, `Partial` works on the top level only.

The example is somehow symmetrical to the previous one. Here, we accept an object that has some optional properties. Then, we apply default values when a property is not present. The result is an object with no optional properties - `Required<Settings>`.

```typescript
function applySettings(settings: Settings) {
  const actualSettings: Required<Settings> = {
    width: settings.width || 100,
    height: settings.height || 200,
    title: settings.title || '',
  }
  // do something...
}
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1411).

### `Readonly`

This one you probably have heard of. `Readonly<T>` returns a type that has the same properties as `T` but they are all `readonly`. It is extremally useful for functional programming because it lets you ensure immutability at compile time. An obvious example would be to use it for Redux state.

Once again, `Readonly` doesn't affect nested objects.

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1418).

### `Pick`

`Pick` lets you create a type that only has selected properties of another type.

An example would be letting the caller override only a specific subset of some default properties.

```typescript
function updateSize(overrides: Pick<Settings, 'width' | 'height'>) {
  return { ...defaultSettings, ...overrides};
}
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1425).

### `Record`

`Record` lets you define a dictionary with keys belonging to a specific type.

JavaScript objects can be very naturally used as dictionaries. However, in TypeScript you usually work with objects using interfaces where the set of keys is predefined. You can work this around by writing something like:

```typescript
interface Options {
  [key: string]: string;
}
```

`Record` lets you do this in a more concise way: `type Options = Record<string, string>`.

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1432).

### `Exclude`

`Exclude` makes a lot of sense when you look at types in terms of sets of possible values. For example, `number` type can be looked at as a set containing all numerical numbers. `A | B` is called a union because its set of possible values is a sum of possible values of `A` and possible values of `B`.

`Exclude<T, U>` returns a type whose set of values is the same as the set of values of type `T` but with all `U` values removed. It is a bit like substruction, but defined on sets.

A good example of using `Exclude` is to define `Omit`. `Omit` takes a type and its key and returns a type without this key. 

```typescript
interface Settings {
  width: number;
  height: number;
  title: string;
}

type SizeSettings = Omit<Settings, 'title'>;

// type SizeSettings = {
//   width: number;
//   height: number;
// }
```

`Omit<T, K>` can be defined by picking all keys from `T` except `K`. First, we'll `Exclude` `K` from the set of keys of `T`. Next, we will use this set of keys to `Pick` from `T`.

```typescript
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1439).

### `Extract`

`Extract<T, U>` return those types included in `T` that are assignable to `U`. You can say that it returns a *common part* of `T` and `U`. However, the types don't have to be exactly the same - it suffices that a type from `T` is assignable to `U`.

For example, you can use `Extract` to filter out function types from a union type:

```typescript
type Functions = Extract<string | number | (() => void), Function>;  // () => void
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1444).

### `NonNullable`

`NonNullable<T>` removes `null` and `undefined` from the set of possible values of `T`.

It is mostly useful when working with `strictNullChecks` and optional properties and arguments. It has no effect on a type that is already not nullable.

```typescript
type Foo = NonNullable<string | null | undefined>; // string
```

You can find a good usage example of `NonNullable` in my [previous article](https://codewithstyle.info/Deep-property-access-in-TypeScript/).

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1449).

### `Parameters`

This useful type returns a tuple of types of parameters of given function.

```typescript
function fetchData(id: number, filter: string) {
}

type FetchDataParams = Parameters<typeof fetchData>; // [number, string]

type IdType = FetchDataParams[0]; // number
```

One interesting usage is typing wrapper functions without having to repeat the parameter list.

```typescript
function fetchDataLogged(...params: Parameters<typeof fetchData>) {
  console.log('calling fetchData');
  fetchData(...params);
}
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1454).

### `ConstructorParameters`

`ConstructorParameters` is exactly the same as `Parameters` but works with constructor functions.

```typescript
class Foo {
  constructor(a: string, b: number) {}
}

type FooConstructorParams = ConstructorParameters<typeof Foo>;
```

One caveat is that you have to remember about `typeof` in front of the class name.

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1459).

### `ReturnType`

The name is pretty self-explanatory - it returns a type returned by given function. I found this type really useful.

One example is in Redux where you define action creators and reducers. A reducer accepts a state object and an action object. You can use `ReturnType` of the action creator to type the action object.

```typescript
function fetchDataSuccess(data: string[]) {
  return {
    type: 'fetchDataSuccess',
    payload: data
  }
}

function reduceFetchDataSuccess(state: State, { payload }: ReturnType<typeof fetchDataSuccess>) {
}
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1464).

### `InstanceType`

`InstanceType` is an interesting one. You can say that it is complimentary to `typeof` operator. 

It accepts a type of a constructor function and returns an instance type of this function.

In TypeScript, `class C` defines two things:

* a constructor function `C` for creating new instances of class `C`
* an interface for objects of class `C` - the *instance type*

`typeof C` returns the type of the constructor function.

`InstanceType<typeof C>` takes the constructor function and returns type of the instances produced by this function: `C`.

```typescript
class C {
}

type CInstance = InstanceType<typeof C>;  // C
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1469).