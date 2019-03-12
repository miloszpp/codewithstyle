---
title: Comprehensive list of useful built-in types in TypeScript
date: 2019-03-09 19:55:34
tags:
---

[Advanced types](https://www.typescriptlang.org/docs/handbook/advanced-types.html) section of TypeScript docs mentions some very useful built-in types as examples of conditional types and mapped types. I was surprised to learn that there are more such types and some of them seem to be undocumented. This article contains a list of all such types.

The list is based on what I could find in `es5.d.ts` on [github](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts).

https://stackblitz.com/edit/typescript-builtins

## Types

### `Partial`

`Partial<T>` returns a type that has the same properties as `T` but all of them are optional. This is mostly useful when `strictNullChecks` flag is enabled.

`Partial` works on a single level - it doesn't affect nested objects.

This type is useful in many situtations. One that comes to my mind is when you need to type a function that lets you override default values of properties of some object.

```tyepscript
const defaultSettings: Settings = { /* ... */ };

function getSettings(custom: Partial<Settings>) {
  return { ...defaultSettings, ...custom };
}
```

[See implementation](https://github.com/Microsoft/TypeScript/blob/4ff71ecb98ccbd882feb1738b0c6f1cc93c2ea66/src/lib/es5.d.ts#L1404).

### `Required`

`Required<T>` removes optinality from `T`'s properties. Again, you'll most likely need it if you have `strictNullChecks` enabled (which you should 😉).

Similiarly to `Required`, `Partial` works on the top level only.

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

Once again, `Readonly` doesn't affect nested objects. It's quite easy to create a *deep* variant using recursive types:

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
}
```

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

// Regular dictionary (instead of writing { [key: string]: string })


DOCUMENTED
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};

DOCUMENTED
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;

// Define Omit with Pick and Exclude
// Mention types as sets

DOCUMENTED
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;

// get string properties from type

DOCUMENTED
/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;


/**
 * Obtain the parameters of a function type in a tuple
 */
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

// Wrapper functions (from reddit comment), dispatch props

/**
 * Obtain the parameters of a constructor function type in a tuple
 */
type ConstructorParameters<T extends new (...args: any) => any> = T extends new (...args: infer P) => any ? P : never;

DOCUMENTED
/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;

// action creators redux

DOCUMENTED
/**
 * Obtain the return type of a constructor function type
 */
type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;
