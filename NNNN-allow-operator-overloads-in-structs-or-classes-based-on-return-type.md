# Allows operator overloads in struct or classes based on return type

* Proposal: [SE-NNNN](NNNN-allow-operator-overloads-in-structs-or-classes-based-on-return-type.md)
* Authors: [Tommaso Piazza](https://github.com/blender)
* Review Manager: TBD
* Status: **Awaiting review**

## Introduction

Swift allows operator overloads to be defined as static functions on structs
or classes only if the type of the struct/class appears in one of the types
of the operands.

This proposal seeks to relax this requirement to allow overloads that include
the type of struct/class in the return type as well.

Swift-evolution thread: [Discussion thread topic for that proposal](https://lists.swift.org/pipermail/swift-evolution/)

## Motivation

As of current design it is impossible for classes and struct to define overloads
of operators unless one of the operands of the operator is of the same type as
the class/struct in question.

Consider the following  `•|` Cons operator

```
precedencegroup ConsPrecedence {
    associativity: right
    assignment: false
}

infix operator •| :ConsPrecedence
```

Consider now the following code

```
public struct NonEmptyArray<Element> {

    fileprivate var elements: Array<Element>

    fileprivate init(array: [Element]) {
        self.elements = array
    }
}

//Overload 1
public func •|<Element>(lhs: Element, rhs: [Element]) -> NonEmptyArray<Element> {
    return NonEmptyArray(array: rhs + [lhs])
}

//Overload 2
public func •|<Element>(lhs: Element, rhs:  NonEmptyArray<Element>) -> NonEmptyArray<Element> {
    return NonEmptyArray(array: [lhs] + rhs.elements)
}

//Overload 3
public func •|<Element>(lhs: NonEmptyArray<Element>, rhs: NonEmptyArray<Element>) -> NonEmptyArray<Element> {
    return NonEmptyArray(array: lhs.elements + rhs.elements)
}

//Usage
let arrayOfOneElement = 1 •| []
let arrayOrManyElements = 1 •| 2 •| []
```

Currently only overloads 2 and 3 can be written as static function on `NonEmptyArray`.

Overload 1 generates the following error message:

```
error: member operator '•|' must have at least one argument of type 'NonEmptyArray<Element>'
        public static func •|<Element>(lhs: Element, rhs: [Element]) -> NonEmptyArray<Element>
```

Because of this behavior properties and functions used in the overloads must be
declared in `NonEmptyArray` with access modifier `fileprivate`.

One way around the problem is to declare a static function with the desired
functionality like in the following snippet

```
public struct NonEmptyArray<Element> {

    private var elements: Array<Element>

    private init(array: [Element]) {
        self.elements = array
    }

    public static func cons<Element>(lhs: Element, rhs: [Element]) -> NonEmptyArray<Element> {
        return NonEmptyArray<Element>(array: rhs + [lhs])
    }
```

and in the implementation of the operator call the static function

```
//Overload 1
public func •|<Element>(lhs: Element, rhs: [Element]) -> NonEmptyArray<Element> {
    return NonEmptyArray.cons(array: rhs + [lhs])
}
```

The current design seems to impose and artificial limitation on type
signatures of functions that begin with a special set of characters
(the symbols chosen as operators).

## Proposed solution

The proposed solution is to relax the limitation on overloads definable as
static functions on structs/classes. The definable overloads should also include
those where the type of the struct/class appears as the return type.

## Detailed design

The definition of overloads as static function would also allow overloads where
there type os the struct/class appears as the return type.

```
public struct NonEmptyArray<Element> {

    private var elements: Array<Element>

    private init(array: [Element]) {
        self.elements = array
    }

    //Overload 1, currently not allowed
    public static func •|<Element>(lhs: Element, rhs: [Element]) -> NonEmptyArray<Element> {
        return NonEmptyArray<Element>(array: rhs + [lhs])
    }

    //Overload 2
    public static func •|<Element>(lhs: Element, rhs:  NonEmptyArray<Element>) -> NonEmptyArray<Element> {
        return NonEmptyArray<Element>(array: [lhs] + rhs.elements)
    }

    //Overload 3
    public static func •|<Element>(lhs: NonEmptyArray<Element>, rhs: NonEmptyArray<Element>) -> NonEmptyArray<Element> {
        return NonEmptyArray<Element>(array: lhs.elements + rhs.elements)
    }
}
```

The change would also allow functions and properties used in the overloads
to be declared with access modifier `private` instead of `fileprivate`.

## Impact on existing code

None

## Alternatives considered

Leave things as they are
