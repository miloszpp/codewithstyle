---
title: TypeScript generic types
date: 2019-10-20 18:35:50
tags:
---

In my previous article I tackled the topic of generic functions. This time, I'd like to discuss generic types.

Let's start with a classic example of a generic class. Let's say you want to implement a data structure that stores a collection of strings and can return a random element from it. The implementation could look as follows.

```typescript
class StringBag {
    constructor(private items: string[] = []) {}

    add(item: string) {
        this.items.push(item);
    }

    getRandom(): string {
        return this.items[Math.random() * this.items.length];
    }    
}
```

Later, you realize that you need a similiar data structure, but one that can hold numbers instead of strings. The implementations is identical - the only thing that differs are types.

```typescript
class NumberBag {
    constructor(private items: number[] = []) {}

    add(item: number) {
        this.items.push(item);
    }

    getRandom(): number {
        return this.items[Math.random() * this.items.length];
    }    
}
```

You're a good programmer and you don't want to violate the DRY principle, so you come up with the following solution.

```typescript
class Bag {
    constructor(private items: any[] = []) {}

    add(item: any) {
        this.items.push(item);
    }

    getRandom(): any {
        return this.items[Math.random() * this.items.length];
    }    
}
```

Unfortunately, it's not type-safe! You're not getting the benefits of TypeScript with this approach. You loose all the knowledge of the shape of the items stored in the `Bag`.

Generic types were designed specifically to address 