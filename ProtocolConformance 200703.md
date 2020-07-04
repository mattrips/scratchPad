# 1 Protocol Conformance

>***Witness*** - Given a concrete type `T` declared to conform to
>protocol `P` and a requirement *m* of `P`, the member of `T` 
>used to perform *m* is referred to as the *witness* for *m* of `T: P`.

When a named concrete type `T` is conformed to a protocol `P`, a set of
implementations--*witnesses*--is determined, one for each requirement of `P`.  
That set of witnesses is referred to as the conformance for ["of" or "for"?] `T: P`.  
This document specifies how Swift determines a conformance.

## 1.1 Creation of a Conformance
The substance of a conformance is not declared.  It is created in response to a named concrete type being conformed to a protocol.  The set of witnesses that is the conformance inferred from the context.[^2]  
[^2]: [What about tuples and [SE-0283\](https://github.com/apple/swift-evolution/blob/master/proposals/0283-tuples-are-equatable-comparable-hashable.md)?]

A concrete type is declared to conform to a protocol in one of two ways.  A non-generic concrete type is directly declared to conform to a protocol, while a concretization of a generic type is indirectly declared to conform to a protocol based on the pattern established in the declaration of the corresponding generic type.

A concretization of a generic type is a specialized version of the generic type formed by replacing the generic type's type parameters with concrete type arguments.  A generic type may specify that its concretizations will conform, or conditionally may conform, to one or more protocols, which establishes a pattern for protocol conformance.  When a concretization is formed, its protocol conformances are determined by that pattern.


 

results from the instantiation of a type arguments  

The specialized version of the generic `Dictionary` type, `Dictionary<String,  Int>` is formed by replacing the generic parameters `Key:  Hashable` and `Value` with the concrete type arguments `String` and `Int`. Each type argument must satisfy all the constraints of the generic parameter it replaces, including any additional requirements specified in a generic `where` clause. In the example above, the `Key` type parameter is constrained to conform to the `Hashable` protocol and therefore `String` must also conform to the `Hashable` protocol.

follows the protocol conformance   

When a generic 


and, implicitly, to any protocols refined by the protocol
That declaration may occur in one of two ways.  For a non-generic concrete type, that declaration occurs in single step, with an express statement of the conformance in the declaration or an extension of the type. 



 stated on the concrete type.  For a concretization of a generic type, that declaration occurs
[^1]: Should we address the possible implicit conformance of tuple types to `Equatable`, `Hashable` and `Comparable`?

A non-generic concrete type `X` may be declared to conform to a protocol `P` by stating `X: P` on the declaration or an extension of `X`.  That declaration directly results in the conformance for `X: P` being determined.

By contrast, if a generic type `Y<T>` is declared to conform to a protocol `P` (by stating either `Y<T>: P` on the declaration of `Y<T>` or `Y: P` on an extension of `Y<T>`), `Y<T>` itself does not conform to `P`, and no conformance is determined.  Rather, the declaration establishes the pattern by which concretizations of `Y<T>` may conform to `P`.  If and when a concretization of `Y<T>`, with the generic argument for parameter `T` specified as concrete type `C`, `Y: C` satisfies the conditions, if any, of the `Y<T>` pattern for conforming to `P` is instantiated, the conformance of `Y<C>: P` is determined.  





The syntactic declaration, `T: P`, that type `T` conforms to protocol `P` is separate and distinct from the determination of the conformance associated with the 


For a type `T` to conform to a protocol `P`, `T` must be declared to conform to
`P`, and `T` must have at least one unconditionally accessible implementation
for each protocol requirement of `P`.  A distinct set of protocol witnesses is
established for the conformance `T: P`.

A declaration that `T` conforms to `P` further constitutes, with respect to each
protocol `o`*<sub>i</sub>* from which `P` directly or indirectly inherits, a
declaration that `T` conforms to `o`*<sub>i</sub>* so long as `T` has not
already been declared to conform to `o`*<sub>i</sub>*.  Thus, for the
declaration of `T: P` to be valid, the declarations of each `T: o`*<sub>i</sub>*
must satisfy the requirement that `T` have at least one unconditionally
accessible implementation for each protocol requirement of
`o`*<sub>i</sub>*. For each conformance `T: o`*<sub>i</sub>*, a distinct set of
protocol witnesses is established.

In a given scope, a type can conform to a protocol in only one way.  A type `T`
cannot be declared to conform to a protocol `P` if, within the visible scope,
another declaration exists of `T: P`.  This rule holds true even where competing
declarations are conditional with disjoint conditions.

## 1.2 Protocol Witness

Given the declaration of conformance `T: P`, a **protocol requirement** *m* is a
statement in the declaration of `P` that `T` (or any other type seeking to
conform to `P`) must have a member satisfying the requirements of *m*.  A member
of `T` that satisfies the requirements of *m* is referred to as an
*implementation* of *m*.

Given `T: P` and a protocol requirement *m* of `P`, one and only one of `T`'s
implementations of *m* will actually be used as the implementation of *m*.  Such
implementation of *m* is referred to as the *protocol witness* for the *m*
requirement of the conformance `T: P`.

The protocol witness for *m* is the implementation that is the *most
specialized* of the *unconditionally accessible* implementations of *m* on `T`,
as determined in the scope in which the declaration `T: P` is stated.  If `T`
has only one unconditionally accessible implementation of *m*, that
implementation will be the protocol witness.  If `T` has more than one
unconditionally accessible implementation of *m*, the most specialized of those
implementations will be the protocol witness.

An implementation is not *declared* to be a protocol witness.  The identity of
the protocol witness for a protocol requirement is inferred from the entirety of
the scope, including all declarations made within the scope and those imported
into the scope.  Careful engineering is required in order to achieve the
intended witness for a given requirement.

## 1.3 Unconditionally Accessible Implementations

A type's possible implementation of a protocol requirement is available to serve
as the protocol witness for the requirement only if the implementation is an
*unconditionally accessible* member of the type in the scope in which the
protocol conformance is declared.  Given a declaration of `T: P` and an
implementation of a protocol requirement *m* of `P`, the implementation is
*unconditionally accessible* if and only if (i) `T` satisfies the conditions, if
any, to which the declaration of the implementation is subject, and (ii) per the
rules of access control, the implementation is visible in the scope in which `T:
P` is declared.

With respect to generic types, clause (i) of this rule is not fully implemented.
A generic type instantiated with its generic arguments specified as concrete
types may be referred to as a *concretization* of the generic type.  A
concretization is a type separate and apart from its generic type.  Swift's
existing implementation of protocols does not include the ability for a
concretization to develop its own, specialized conformance relationship with a
protocol.  Instead, a concretization uses the protocol conformances of its
generic type, without specialization for implementations available via the
concretization.  Thus, although a concretization may satisfy the conditions of a
generic where clause that would make an implementation unconditionally
accessible, the implementation subject to such where clause nevertheless is
unavailable to serve as a protocol witness for the shared conformance.
  * Although specialized implementations available to a concretization are not
    available to serve as protocol witnesses and cannot be accessed via the
    interface of a protocol, they remain members of the concretization.
    Accordingly, as members, they may be accessed directly on the concretization
    following the rules applicable to resolution of overloads.
  * Notwithstanding the general inability of concretizations to take advantage
    of specialized implementations, the `Collection` family of protocols in the
    Standard Library uses a private attribute to gain some access to specialized
    implementations for concretizations of generic types conforming to
    `BidirectionalCollection` and `RandomAccessCollection`.


## 1.4 Most Specialized Implementation
Among a type's unconditionally accessible implementations of a protocol
requirement, the most specialized implementation will serve as the protocol
witness for the requirement.  The relative specialization between two
implementations, `i`*<sub>1</sub>* and `i`*<sub>2</sub>*, is determined as
follows:

If `i`*<sub>1</sub>* is declared in an extension of a protocol and
`i`*<sub>2</sub>* is declared on T (whether in the declaration and/or an
extension), then `i`*<sub>2</sub>* is more specialized.

If `i`*<sub>1</sub>* is declared in an extension of protocol p1 and
`i`*<sub>2</sub>* is declared in an extension of protocol p2, then (i) if p2
inherits from p1, `i`*<sub>2</sub>* is more specialized, (ii) if p1 inherits
from p2, `i`*<sub>1</sub>* is more specialized, and (iii) otherwise,
`i`*<sub>1</sub>* and `i`*<sub>2</sub>* present an ambiguity (if there is no
other implementation that is more specialized than both `i`*<sub>1</sub>* and
`i`*<sub>2</sub>*, an error will be raised at compile time).

If `i`*<sub>1</sub>* and `i`*<sub>2</sub>* are both declared on T (whether in
the declaration and/or an extension) or are both declared in extensions of the
same protocol, then (i) if the declaration of `i`*<sub>1</sub>* is more
constrained than the declaration of `i`*<sub>2</sub>*, `i`*<sub>1</sub>* is more
specialized, (ii) if the declaration of `i`*<sub>2</sub>* is more constrained
than the declaration of `i`*<sub>1</sub>*, `i`*<sub>2</sub>* is more
specialized, and (iii) otherwise, `i`*<sub>1</sub>* and `i`*<sub>2</sub>*
present an ambiguity (if there is no other implementation that is more
specialized than both `i`*<sub>1</sub>* and `i`*<sub>2</sub>*, an error will be
raised at compile time).[^1]

[^1]: Given the way conditional declarations work or don’t work, I’m not sure
      these declared-on-same-type situations could arise in a meaningful
      way. Thoughts?

## 1.5 Set of Protocol Witnesses

Given a declaration that a type conforms to a protocol, the protocol witness for
the conformance is the set consisting of the protocol witness for each declared
requirement of the protocol.  Inherited requirements of a protocol are
irrelevant to determination of a protocol witness set.

There is only one protocol witness set for a protocol conformance declaration.
Such set is immutable, and is not subject to replacement.

If a protocol has no declared requirements, the protocol witness set for
conformances to the protocol is empty.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUwMTA1MDI4MCwxNDMxMTM2MTQzLC04MT
kzODA1NjMsMTM1MzU3Mjg4NCwxMzI1NTg2NjUyLDE2OTg0Mjgx
NSw4NDU3Mjk0NTksLTE2Nzg2NDAwNDQsLTMxNTU1ODE4MCwtMT
UwMzIzMTI5NywxMzU5Mzg0MjI4XX0=
-->