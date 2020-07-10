# scratchPad

## 1. Protocol Conformance
With respect to a type declared to conform to a protocol, a protocol conformance specifies, for each protocol requirement of the protocol, which property, method, initializer, subscript or typealias will be used to implement the protocol requirement when invoked on the type.  Understanding protocol conformance is key to obtaining predictable polymorphic behavior.  This guide explains the semantics of how Swift determines a protocol conformance.

&#9724; protocol requirement
: A statement in a protocol declaration describing a property, method, initializer, subscript or typealias that a type conforming to the protocol must implement.

&#9724; implementation
: A property, method, initializer, subscript or typealias that is capable of satisfying a given protocol requirement.   

<img src="https://render.githubusercontent.com/render/math?math=e^{i \pi} = -1">
