## Terminology

protocol requirement
: A statement describing a property, method, initializer, subscript or typealias that a type declared to conform to a given protocol must implement.

implementation
: A property, method, initializer, subscript or typealias that is capable of satisfying a given protocol requirement.    

witness
: The particular implementation used to satisfy a protocol requirement.

protocol conformance
: The set of witnesses used to satisfy the requirements of a given protocol.

A protocol specifies a set of requirements.  A protocol may be applied to any type that satisfies its protocol requirements.  

A protocol also may supply functionality, which may serve as default implementations of its own protocol requirements.  A protocol provide may provide additional functionality of an arbitrary nature.  

Most powerfully, a protocol may serve as the basis for an existential type bearing the same type name as the protocol, with the interface of the existential type being defined by the protocol.

## 1. Protocol Conformance
With respect to a type declared to conform to a protocol, a protocol conformance specifies, for each protocol requirement of the protocol, which property, method, initializer, subscript or typealias will be used to implement the protocol requirement if the protocol requirement is invoked on the type.  Understanding protocol conformance is key to obtaining predictable polymorphic behavior.  This guide explains the semantics of how Swift determines a protocol conformance.

### 1.1 The Witness
If a type is declared to conform to a protocol, the type must satisfy each of the protocol requirements of the protocol.  The type does so by having available at least one implementation--that is at least one property, method, initializer, subscript or typealias--for each protocol requirement of the protocol.  

With respect to the a type that is declared to conform to a protocol,
for each protocol requirement of the protocol, Swift determines the one and only implementation that will be used throughout a program to satisfy that protocol requirement for that type.  The implementation so determined is referred to as the witness for the protocol requirement.  

### 1.2 Model of a Protocol Conformance
With respect to the declaration that a type conforms to a protocol, the protocol conformance is a collection containing the witness for each protocol requirement of the protocol.
 
&#9724;   Metaphorically, a protocol conformance may be modeled as an immutable struct containing a declaration that a type conforms to a protocol and a dictionary of protocol requirement-witness pairs:
```
struct ConformanceDeclaration {
	let type: Type // must be a struct, enum or class
	let protocol: Type // must be a protocol
}

struct ProtocolConformance {
	let declaration: ConformanceDeclaration
	let witnesses: [ProtocolRequirement: Witness]
	
	init(/* the entire visible context */) {
		// See §1.4 of this guide
	}
}
```

### 1.3 Underpinnings of Protocol Conformance

#### 1.3.1 Singular
A protocol conformance is determined with respect to the pairing of a type and a protocol to which it conforms. A type may conform to a protocol in one and only one way.  There cannot be more than one protocol conformance for a type-protocol pairing.  

It is an error to twice declare the conformance of a type to a given protocol.  The same is true even if the conformances are conditional with disjoint conditions.

If a type conforms to multiple protocols, there will be a distinct protocol conformance for each type-protocol pair.

#### 1.3.2 Immutable
A protocol conformance is immutable.  It cannot be altered by operation of a program.  

#### 1.3.3 Invariable
With respect to a protocol conformance, during operation of a program, the witness for a particular protocol requirement will not vary, and does not depend upon how the type is used.

&#9724;   In complex arrangements, it may be perceived that the witness varies.  Sometimes, this appearance is due to a different protocol being used, with the protocols having the same requirement.  *See*, ____.  Other times, this appearance is due to complexity making it difficult to reason about which implementation is the witness for a protocol requirement.  *See*, ____. [^1]
[^1]: Could this also be due to overloading?

#### 1.3.5 Inferred Declaration
The declaration that a type conforms to a protocol is expressly stated. But the protocol conformance underpinning the declaration is neither declared nor guided by annotation.  Instead, Swift infers the protocol conformance for the declaration, through static analysis of all possibly available implementations of a protocol requirement.

#### 1.3.6 Possibly Available Implementations

**Potential for Multiple Implementations of a Requirement**

In a simple case, there may exist only one possible implementation for each protocol requirement.  In that case, determining the  protocol conformance  is a simple matter of matching each requirement with its implementation.  In more complex cases, there may be several  implementations  available for each  protocol requirement.  Implementations  may be scattered across multiple files, in the source code of other modules in a project, and in the binary files of imported SDKs.  Anticipating the  protocol conformance  that will result in a complex case requires careful attention to detail.

### Interaction with Class Inheritance

Where a class conforms to a protocol, the nature of class inheritance impacts the availability of implementations for purposes of protocol conformance.  For instance, a superclass may expose functionality that incidentally has the same signature as a protocol requirement.  Conversely, if a class is declared to conform to a protocol, the implementations in a subclass of the class are not available to satisfy the protocol requirements.

### Invocation
At point of invocation, which protocol's protocol requirement is being invoked?
Whether invoked at all--or concrete method on concrete type is called.


### Signature Matching

Protocol requirements and implementations are matched based on _____.  [check this]  Member names, argument labels, parameter order, parameter types, return types, etc., must be the same.  Parts that are not part of the signatures, such as parameter names, are disregarded for matching.

A type may satisfy the protocol requirements  of a protocol using functionality declared as part of the protocol itself, the type itself, a superclass from which the type inherits, or any other protocol to which the type conforms.

As there may be multiple independently-scoped contexts from which a given type may satisfy the protocol requirements of a given protocol,  multiple implementations  of a given protocol requirement may be available to the type.  The selection of which implementations available to a type will be used to satisfy the protocol requirements of a protocol is referred to as the  protocol conformance  for the type to the protocol.

A protocol conformance is the abstract concept of the behavior of a given protocol when conforming to a given type in the presence of a given set of other protocols, if any, to which the type also conforms.  All representations of that concept are implementation details internal to Swift.  Any difference in a protocol’s behavior from one context to the next is detectable only by the difference in state and/or side effects that result when the system chooses one implementation of a particular requirement or arbitrary functionality over another.

Fundamentally, a protocol is a collection of abstract behaviors, specified via  protocol requirements.  A protocol may include  default implementations  of its protocol requirements, which implementations may be concrete or generic in their operation.  Further, a protocol may include additional,  arbitrary functionality[define a term for this…].  A protocol’s protocol requirements, default implementations and arbitrary functionality, together, define a potential  range of behavior for the protocol.  The actual available behavior of a protocol within its potential range varies depending upon what type is conformed to the protocol and depending upon what other protocols are conformed to that type.  The actual available behavior is the protocol conformance.

A protocol conformance is the abstract representation of the behavior of a given protocol when conforming to a given type.  When a given type conforms a given protocol, the type forms a relationship with the protocol.  The behavior of the protocol in the context of that specific relationship is referred to as a  protocol conformance.  When a type conforms to multiple protocols, each type-protocol relationship in that arrangement constitutes a distinct protocol conformance.  The presence of any given protocol in an arrangement may affect the behavior of one or more other protocols in the arrangement such that a protocol conformance should be regarded as being specific to the arrangement.  Thus, a protocol conformance cannot safely be assumed to apply to a different arrangement of a type and protocols.

EXAMPLE


## Basic Rule of Protocol Conformance
The basic rule of protocol conformance is stated, as follows:
>For a given protocol requirement, the ***most specialized implementation*** that is  ***available to the protocol via the conforming type*** will be selected as the witness.

Protocol conformance involves matching each protocol requirement of a protocol to an implementation of the requirement.  For a given protocol requirement, multiple implementations may be present.  An implementation selected by protocol conformance is referred to as a  witness.  The  most specialized implementation of a protocol requirement that is  available to the protocol  will be selected as the witness for the protocol requirement.

Protocol conformance results in a set of requirement/witness pairings for the conformance of a given type to a given protocol:  one witness for each protocol requirement.  That set is referred to as a  protocol witness table.  The concept of the protocol witness table is an implementation detail, not visible to the user.  Nonetheless, the idea of the protocol witness table may help inform a user’s mental model of protocol conformance and what it produces.

Protocol conformance is performed at compile time.  Its results are not altered by conditions at run time.

**Associatedtype Requirements**

**Most Specialized Implementation**

A candidate implementation may be declared in any of the following:

1. the declaration or an extension of the subject type,

2. the declaration or an extension of a superclass of the subject type.  [check this]

3. an extension of another protocol lower in the subject protocol’s chain of protocol inheritance to which lower protocol the subject type also conforms,

4. an extension of the subject protocol, or

5. an extension of another protocol higher in the subject protocol’s chain of protocol inheritance.

The  most specialized implementation  is the candidate implementation closest in relationship to the subject type.  The listing, above, of possible sources of an implementation is listed in closeness of relationship to the subject type, beginning with the subject type itself.

EXAMPLES (simple and complex)

**Available to the Protocol**

With respect to a protocol, an implementation is  available to the protocol  if it visible within the scope of the subject type under all relevant conditions.  The scope of the subject type includes all protocols to which the type is declared to conform.  [check this] If an implementation is unconditionally declared, it is available under all conditions.  If an implementation is declared in an extension qualified by a generic where clause, the implementation is available only under the conditions stated in the generic where clause.  If the conditions applicable to a conditionally-declared implementation are not identical to the conditions under which the protocol conformance is stated to exist, the implementation is deemed unavailable.  The compiler does not attempt to reconcile one set of conditions with another to determine whether the conditions to a protocol conformance might be a subset of the conditions to the declaration of an implementation; instead, the compiler simply rejects the implementation.  [check this]

EXAMPLE

**Interdependent Protocol Conformance**

The protocol conformance of a type to a protocol is determined with the provisional assumption that all declarations of the type conforming to other protocols are valid, and so any additional functionality provided by those other protocols is assumed to be available to the subject protocol.  Thus, if the requirement of one protocol is satisfied by additional functionality of another protocol, and vice versa, the protocol conformance will be valid as to both protocols.

**Protocol Conformance for a Generic Type**

If a type is generic, the protocol conformance is performed with respect to the type in its general form.  If an implementation of a protocol requirement is declared in an extension of a generic type that is constrained by a type parameter of the generic type, that implementation is disregarded for purposes of protocol conformance.  [check this]

EXAMPLE

**Conditional Conformance**

If a type is declared to conditionally conform to a protocol, the protocol conformance is performed with the conditionality taken into account.

If an extension of a protocol in a protocol hierarchy declares an implementation for a requirement of a protocol higher in the hierarchy, that implementation will be available to satisfy the higher protocol’s requirement only when a the type conforms to the lower protocol.  If the type’s conformance to the lower protocol is conditional and the conformance to the higher protocol is unconditional or differently-conditioned, then, per the rules of implementation availability discussed, above, the implementation declared on the lower protocol will be regarded as unavailable for protocol conformance for the higher protocol.  An implication of such unavailability is that, even though the specialized behavior is present in the hierarchy, it will not be available to the conditionally conforming type—not even when the condition to the conformance to the lower protocol is satisfied.

EXAMPLE

Where a type is declared to conform to multiple protocols that exist in an inheritance relationship, protocol conformance is performed beginning with the protocol lowest in the hierarchy (i.e. the most specialized).  The protocol conformance is determined for all protocol requirements declared by such protocol, if any.  Note that the restatement of a requirement of a protocol higher in the hierarchy is meaningless, and so is disregarded (but does not generate a warning or error).  Then, the protocol conformance is determined for the protocol from which the lowest protocol directly inherits, with that protocol conformance being determined for all protocol requirements declared by such protocol, if any.  The process proceeds until protocol conformance for the highest protocol in the hierarchy is complete.  [check this]  [all of this is a giant implementation detail; not relevant, here]

If multiple branches of a protocol hierarchy are involved, the process will begin with the lowest descending branch, and then proceed to the next lowest descending branch, and so on.  [check this] once the protocol conformance of a protocol in the hierarchy is determined while processing a branch, that same protocol conformance will be used in all subsequent branches.  It will not be redetermined.  In that fashion, a type will conform to any given protocol in one, and only one, way.  Thus, there will only be one protocol conformance for the combination of a given type and protocol.  [annotation re: multi-module issue] [annotation re: specializations unavailable due to conditionality]

If a type conforms directly to multiple protocols or a protocol inherits directly from multiple protocols, the order in which protocol conformance is performed is as follows: ____.

Once determined, it is encoded into a program’s binary representation.  A  protocol conformance  is not directly inspectable via tooling.

Conditionally available …

One and only one

Module scope

Static dispatch of non-required implementations….

Invocation ...  whether the invocation is directly on an instance of the type or through an existential container holding an instance of the type.   


Another key factor to understand is whether a given call actually invokes the protocol system, which factor is discussed at ____, below.

(or a set of protocols in an inheritance relationship)

Concrete invocations…

Shadowing...

However, it should be noted that such an implementation nevertheless will be available on an instance of the concrete type, and if the name of the implementation is called on such instance, the call will be dispatched to the implementation declared in the constrained extension of the generic type; such implementation is said to shadow the protocol requirement.  When a shadow implementation is called, the protocol system effectively is not involved in the dispatching, although, from the user’s perspective, the net effect is the same as if the shadow implementation had been selected via protocol conformance.

Protocol inheritance from a class

A type cannot re-conform to a protocol in varying ways inside the same module

Annotations:

[copy more from thread]

[discuss subtleties and surprises]

[discuss Collection hierarchy’s special behavior]

[describe implementation departures from intended system]

[describe known bugs]

[no implicit override]

[no override at all]

[At the implementation level, that set of pairings is called a protocol witness table.]

[annotate re: leakage of imported protocols beyond the file in which the import occurs]

[test harness to ensure desired protocol conformance]
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyNzM0MDY4MywtMTM1NzE3NjQzLDc5MD
Y3ODc5MSwxNTA3NTA4MDg2LDExNzg5NzU5ODksODY4NzEzMzMx
LC0zMTkwOTA4MDUsMTM0MTQxNDUzNiwyMDgyMDkxNTk3LDIxND
Y2NjQ0NDksLTEyMDQyNzU0MjMsLTExMTcxMjQyNjksMTgxNzgz
ODE2MywtMTEzODg1NTIyMF19
-->