# 1 Protocol Conformance

>Given `t: p` and a protocol requirement `m` of `p`, the protocol witness for `m` is the implementation that is the most specialized of the unconditionally accessible implementations of `m` on `t`, as determined in the scope in which the declaration `t: p` is stated.

When a type is declared to conform to a protocol, a set of implementations--*protocol witnesses*--is determined, one for each protocol requirement of the protocol.  Understanding how a set of protocol witnesses is determined is key to obtaining predictable polymorphic behavior.  This guide explains the semantics of how Swift determines the *protocol witness set* for a protocol conformance.

## 1.1 Declaration of Protocol Conformance
For a type `t` to conform to a protocol `p`, `t` must be declared to conform to `p`, and `t` must have at least one unconditionally accessible implementation for each protocol requirement of `p`.  A distinct set of protocol witnesses is established for the conformance `t: p`.

A declaration that `t` conforms to `p` further constitutes, with respect to each protocol `o`*<sub>i</sub>* from which `p` directly or indirectly inherits, a declaration that `t` conforms to `o`*<sub>i</sub>* so long as `t` has not already been declared to conform to `o`*<sub>i</sub>*.   Thus, for the declaration of `t: p` to be valid, the declarations of each `t: o`*<sub>i</sub>* must satisfy the requirement that `t`  have at least one unconditionally accessible implementation for each protocol requirement of  `o`*<sub>i</sub>*. For each conformance `t: o`*<sub>i</sub>*, a distinct set of protocol witnesses is established.

In a given scope, a type can conform to a protocol in only one way.  A type `t` cannot be declared to conform to a protocol `p` if, within the visible scope, another declaration exists of `t: p`.  This rule holds true even where competing declarations are conditional with disjoint conditions.  

## 1.2 Protocol Witness
Given the declaration of conformance `t: p`, a *protocol requirement* `m` is a statement in the declaration of `p` that `t` (or any other type seeking to conform to `p`) must have a member satisfying the requirements of `m`.  A member of `t` that satisfies the requirements of `m` is referred to as an *implementation* of `m`.  

Given `t: p` and a protocol requirement `m` of `p`, one and only one of `t`'s  implementations of `m` will actually be used as the implementation of `m`.  Such implementation of `m` is referred to as the *protocol witness* for the `m` requirement of the conformance `t: p`.

The protocol witness for `m` is the implementation that is the *most specialized* of the *unconditionally accessible* implementations of `m` on `t`, as determined in the scope in which the declaration `t: p` is stated.  If `t` has only one unconditionally accessible implementation of `m`,  that implementation will be the protocol witness.  If `t` has more than one unconditionally accessible implementation of `m`,  the most specialized of those implementations will be the protocol witness.

An implementation is not *declared* to be a protocol witness.  The identity of the protocol witness for a protocol requirement is inferred from the entirety of the scope, including all declarations made within the scope and those imported into the scope.  Careful engineering is required in order to achieve the intended witness for a given requirement.

## 1.3 Unconditionally Accessible Implementations
A type's possible implementation of a protocol requirement is available to serve as the protocol witness for the requirement only if the implementation is an *unconditionally accessible* member of the type in the scope in which the protocol conformance is declared.  Given a declaration of `t: p` and an implementation of a protocol requirement `m` of `p`, the implementation is *unconditionally accessible* if and only if (i) `t` satisfies the conditions, if any, to which the declaration of the implementation is subject, and (ii) per the rules of access control, the implementation is visible in the scope in which `t: p` is declared.

With respect to generic types, clause (i) of this rule is not fully implemented.  A generic type instantiated with its generic arguments specified as concrete types may be referred to as a *concretization* of the generic type.  A concretization is a type separate and apart from its generic type.  Swift's existing implementation of protocols does not include the ability for a concretization to develop its own, specialized conformance relationship with a protocol.  Instead, a concretization uses the protocol conformances of its generic type, without specialization for implementations available via the concretization.  Thus, although a concretization may satisfy the conditions of a generic where clause that would make an implementation unconditionally accessible, the implementation subject to such where clause nevertheless is unavailable to serve as a protocol witness for the shared conformance.  
  * Although specialized implementations available to a concretization are not available to serve as protocol witnesses and cannot be accessed via the interface of a protocol, they remain members of the concretization.  Accordingly, as members, they may be accessed directly on the concretization following the rules applicable to resolution of overloads.
  * Notwithstanding the general inability of concretizations to take advantage of specialized implementations, the `Collection` family of protocols in the Standard Library uses a private attribute to gain some access to specialized implementations for concretizations of generic types conforming to `BidirectionalCollection` and `RandomAccessCollection`.
  

## 1.4 Most Specialized Implementation
Among a type's unconditionally accessible implementations of a protocol requirement, the most specialized will serve as the protocol witness for the requirement.  The relative specialization between two implementations, imp1 and imp2, is determined as follows:
If imp1 is declared in an extension of a protocol and imp2 is declared on t (whether in the declaration and/or an extension), then imp2 is more specialized.  
If imp1 is declared in an extension of protocol p1 and imp2 is declared in an extension of protocol p2, then (a) if p2 inherits from p1, imp2 is more specialized, (b) if p1 inherits from p2, imp1 is more specialized, and (c) otherwise, imp1 and m2 present an ambiguity (if there is no other implementation that is more specialized than both imp1 and imp2, an error will be raised at compile time).  
If imp1 and imp2 are both declared on t (whether in the declaration and/or an extension) or are both declared in extensions of the same protocol,[^2] then (i) if the declaration of imp1 is more constrained than the declaration of imp2, imp1 is more specialized, (ii) if the declaration of m2 is more constrained than the declaration of imp1, imp2 is more specialized, and (iii) otherwise, imp1 and imp2 present an ambiguity (if there is no other implementation that is more specialized than both imp1 and imp2, an error will be raised at compile time). 

[^2 ]: Given the way conditional declarations work or don’t work, I’m not sure these declared-on-same-type situations could arise in a meaningful way. Thoughts?







en multiple implementations of the same protocol requirement, the degree of specialization of an implementation is based on the declaration of the implementation, as follows, from most specialized to least specialized: [check this]
  1. conditionally declared in an extension of the type;
  2. unconditionally declared in the declaration or an extension of the type;
  3.  conditionally declared in an extension of a protocol to which the type conforms; and
  4. unconditionally declared in an extension of a protocol to which the type conforms. 

  * If multiple protocols in a chain of inheritance of protocols provide declarations of an implementation, the implementation on a more refined protocol is more specialized than an implementation on a less refined protocol. [check this]
  * Where multiple protocols outside of a single chain of inheritance provide implementations, an ambiguity error occurs. [check this]

## 1.5 Protocol Witness Set

The set of requirements declared within the declaration of a protocol.  

Given a declaration that a type conforms to a protocol, the protocol witness set is the set consisting of the protocol witness for each declared requirement of the protocol.  [A protocol witness set also is referred to as a protocol conformance.  I am suggesting this term to avoid ambiguity between that usage of the term protocol conformance and similar usage of the term to refer to the more general notion that a type is declared to conform to a protocol or that a type has implementations capable of conforming to a protocol.  I also am suggesting that the termprotocol witness set presents a much better conceptual picture of what the conformance actually is.]
  * There is only one protocol witness set for a protocol conformance declaration.  Such set is immutable, and is not subject to replacement.
  * If a protocol has no declared requirements, the protocol witness set for conformances to the protocol is empty.
  * Inherited requirements of a protocol are irrelevant to determination of a protocol witness set.
  


<!--stackedit_data:
eyJwcm9wZXJ0aWVzIjoiZXh0ZW5zaW9uczpcbiAgcHJlc2V0Oi
AnJ1xuIiwiaGlzdG9yeSI6Wy0xMzk3NDk4OTQ3LC01MTg0OTM5
MTAsMTQxNzk1ODA2MywzNTAxMjI0NjksMTMxMTMwNzM4OSw4MD
A5MjgyMTAsLTExMjI1NzkyNDAsMTU2MzA5NTMyMSwtMjE0MzQ1
Nzc4Miw1NzMyNTA5MzYsLTExMTE0MDM2NiwtMTk5Nzk3NzI4Mi
wtMTcwNDMzMDYyMCw2NjAxNTcwODEsMTg2MDE0NTU1Niw2NjA0
MjgxMjksLTY0NjM5MDQxOSwtMjkwNzU4NDMxLDE3NDM1MDk5Mj
QsMTIxMzUwMTQ5Ml19
-->