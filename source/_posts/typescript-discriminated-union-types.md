---
title: 'TypeScript: Precise domain modeling with discriminated unions'
url: 578.html
id: 578
categories:
  - TypeScript
  - Web
date: 2018-01-08 08:00:44
tags:
  - typescript
  - algebraic data types
---

In this post, we're going to look into an interesting feature of the TypeScript language. It's called **discriminated unions** and is also known as **algebraic data types**. The latter name comes from Functional Programming paradigm where such types are used very heavily.

Issues with enum types
----------------------

Let me start by showing you an example of a problem that can be solved with discriminated unions. You're working on an application which deals with the management of customers. There are two kinds of customers: individual and institutional. For each customer kind, you store different details: individual customers have a first and last name and a social security number. Companies have a company name and a tax identifier. You could model the above situation with the following types:

```typescript
    enum CustomerType {
        Individual,
        Institution
    }
    
    interface Customer {
        acquisitionDate: Date;
        type: CustomerType;
        firstName?: string;
        lastName?: string;
        socialSecurityNumber?: string;
        companyName?: string;
        companyTaxId?: number;
    }
```

Unfortunately, you have to make most of the fields optional. If you didn't, you would have to fill in all of the fields when creating an instance of `Customer`. However, you don't want to fill `companyTaxId` when creating an `Individual` customer. The problem with this solution is that it's now possible to create instances that don't make any sense in terms of business domain. For example, you can create an object with too little info:

```typescript
    const customer1: Customer = { 
        acquisitionDate: new Date(2016, 1, 1),
        type: CustomerType.Individual
    };
```

...or one that has too much data provided:

```typescript
    const customer2: Customer = { 
        acquisitionDate: new Date(2016, 1, 1),
        type: CustomerType.Individual,
        firstName: "John",
        lastName: "Green",
        companyName: "Acme",
        companyTaxId: 9243546
    };
```

Wouldn't it be nice if the type system could help us prevent such situations? Actually, this is what TypeScript is supposed to do, right?

Discriminated unions to the rescue
----------------------------------

With discriminated unions, you can model your domain with more precision. They are kind of like enum types but can hold additional data as well. Therefore, you can enforce that a specific customer type must have an exact set of fields. Let's see it in action.

```typescript
    interface IndividualCustomerType {
        kind: "individual";
        firstName: string;
        lastName: string;
        socialSecurityNumber: number;
    }
    
    interface InstitutionCustomerType {
        kind: "institutional";
        companyName: string;
        companyTaxId: number;
    }
    
    type CustomerType = IndividualCustomerType | InstitutionCustomerType;
    
    interface Customer {
        acquisitionDate: Date;
        type: CustomerType;
    }
```

We've defined two interfaces. Both of them have a `kind` property which is a **literal type**. Variable of literal type can only hold a single, specific value. Each interface contains only fields that are relevant to the given type of customer. Finally, we've defined `CustomerType` as a union of these two interfaces. Because they both have the `kind` field TypeScript recognizes them as discriminated union types and makes working with them easier. The biggest gain is that it's now impossible to create _illegal_ instances of `Customer`. For example, both of the following objects are fine:

```typescript
    const customer1: Customer = { 
        acquisitionDate: new Date(2016, 1, 1),
        type: {
            kind: "individual",
            firstName: "John",
            lastName: "Green",
            socialSecurityNumber: 423435
        }
    };
    
    const customer2: Customer = { 
        acquisitionDate: new Date(2016, 1, 1),
        type: {
            kind: "institutional",
            companyName: "Acme",
            companyTaxId: 124345454
        }
    };
```

...but TypeScript would fail to compile this one:

```typescript
    // fails to compile
    const customer3: Customer = { 
        acquisitionDate: new Date(2016, 1, 1),
        type: {
            kind: "institutional",
            companyName: "Acme",
            companyTaxId: 124345454,
            firstName: "John"
        }
    };
```

Working with discriminated unions
---------------------------------

Let's now see how to implement a function that takes a `Customer` object and prints the customer's name based on their type.

```typescript
    function printName(customer: Customer) {
        switch (customer.type.kind) {
            case "individual": return `${customer.type.firstName} ${customer.type.lastName}`;
            case "institutional": return customer.type.companyName;
        }
    }
```

As we can see, TypeScript is clever enough to know that inside `case "individual"` branch of the `switch` statement `customer.type` is actually an instance of `IndividualCustomerType`. For example, trying to access `companyName` field inside this branch would result in a compilation error. We would get the same behaviour inside an `if` statement branch. There is one more interesting mechanism called exhaustiveness checking. TypeScript is able to figure out that we have not covered all of the possible customer types! Of course, it would seem much more useful if we had tens of them and not just two.

```typescript
    // fails to compile
    function printName(customer: Customer) {
        switch (customer.type.kind) {
            case "individual": return `${customer.type.firstName} ${customer.type.lastName}`;
            // case "institutional": return customer.type.companyName;
            default: const exhaustiveCheck: never = customer.type;
        }
    }
```

This solution makes use of the `never` type. Since `case "institutional"` is not defined, control falls through to the `default` branch in which `customer.type` is inferred to be of type `InstitutionCustomerType` while being assigned to `never` type which of course results in an error.

Conclusion
----------

Discriminated union types are pretty cool. As I mentioned, the whole point of TypeScript is to help us catch mistakes that we would make without having type checking. Discriminated unions help us model the domain in more detail, therefore making _illegal_ instances impossible to create.

Disclaimer
----------

One could argue that the same thing could be achieved with inheritance (or interface extension in this case). And that's true. Solving this with inheritance would be an Object Oriented Programming approach while discriminated unions are specific to Functional Programming. I think this approach makes more sense in the context of web applications where we often fetch data from some REST API which doesn't support object inheritance. What's more, exhaustiveness checking is not possible to achieve with object inheritance. It's an example of the classical _composition versus inheritance_ dilemma.