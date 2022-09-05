---
sidebar_position:  149
---

# C.6  Shared Variable Control

{AI05-0229-1} {AI05-0299-1} [This subclause defines representation aspects that control the use of shared variables.] 

Paragraphs 2 through 6 were moved to Annex J, "Obsolescent Features". 


#### Static Semantics

{AI05-0229-1} {AI12-0282-1} For an [object_declaration](./AA-3.3#S0032), a [component_declaration](./AA-3.8#S0070), a [full_type_declaration](./AA-3.2#S0024), or a [formal_complete_type_declaration](./AA-12.5#S0321), the following representation aspects may be specified:

AtomicThe type of aspect Atomic is Boolean.

Aspect Description for Atomic: Declare that a type, object, or component is atomic.

IndependentThe type of aspect Independent is Boolean.

Aspect Description for Independent: Declare that a type, object, or component is independently addressable.

VolatileThe type of aspect Volatile is Boolean.

Aspect Description for Volatile: Declare that a type, object, or component is volatile.

{AI12-0363-1} Full_Access_OnlyThe type of aspect Full_Access_Only is Boolean.

Aspect Description for Full_Access_Only: Declare that a volatile type, object, or component is full access.

{AI05-0229-1} {AI12-0282-1} For a [full_type_declaration](./AA-3.2#S0024) of an array type, an [object_declaration](./AA-3.3#S0032) for an object of an anonymous array type, or the [formal_complete_type_declaration](./AA-12.5#S0321) of a formal array type, the following representation aspects may be specified:

Atomic_ComponentsThe type of aspect Atomic_Components is Boolean.

Aspect Description for Atomic_Components: Declare that the components of an array type or object are atomic.

Volatile_ComponentsThe type of aspect Volatile_Components is Boolean.

Aspect Description for Volatile_Components: Declare that the components of an array type or object are volatile.

{AI05-0229-1} {AI12-0282-1} For a [full_type_declaration](./AA-3.2#S0024) of a composite type, an [object_declaration](./AA-3.3#S0032) for an object of an anonymous composite type, or the [formal_complete_type_declaration](./AA-12.5#S0321) of a formal composite type, the following representation aspect may be specified:

Independent_ComponentsThe type of aspect Independent_Components is Boolean.

Aspect Description for Independent_Components: Declare that the components of an array or record type, or an array object, are independently addressable.

{AI05-0229-1} {AI12-0363-1} If any of these aspects are directly specified, the [aspect_definition](./AA-13.1#S0348) shall be a static expression. If not specified for a type (including by inheritance), the Atomic, Atomic_Components, and Full_Access_Only aspects are False. If any of these aspects are specified True for a type, then the corresponding aspect is True for all objects of the type. If the Atomic aspect is specified True, then the aspects Volatile, Independent, and Volatile_Component (if defined) are True; if the Atomic_Components aspect is specified True, then the aspects Volatile, Volatile_Components, and Independent_Components are True. If the Volatile aspect is specified True, then the Volatile_Components aspect (if defined) is True, and vice versa. When not determined by one of the other aspects, or for an object by its type, the Volatile, Volatile_Components, Independent, and Independent_Components aspects are False.

Ramification: {AI12-0363-1} Aspects Volatile and Volatile_Components (when defined) are equivalent. We provide the Volatile_Components aspect only to give symmetry with Atomic_Components and Independent_Components aspects. 

{AI95-00272-01} {AI05-0229-1} An atomic type is one for which the aspect Atomic is True. An atomic object (including a component) is one for which the aspect Atomic is True, or a component of an array for which the aspect Atomic_Components is True for the associated type, or any object of an atomic type, other than objects obtained by evaluating a slice. 

Ramification: {AI95-00272-01} A slice of an atomic array object is not itself atomic. That's necessary as executing a read or write of a dynamic number of components in a single instruction is not possible on many targets. 

{AI05-0229-1} A volatile type is one for which the aspect Volatile is True. A volatile object (including a component) is one for which the aspect Volatile is True, or a component of an array for which the aspect Volatile_Components is True for the associated type, or any object of a volatile type. In addition, every atomic type or object is also defined to be volatile. Finally, if an object is volatile, then so are all of its subcomponents [(the same does not apply to atomic)].

{AI05-0009-1} {AI05-0229-1} {AI12-0001-1} When True, the aspects Independent and Independent_Components specify as independently addressable the named object or component(s), or in the case of a type, all objects or components of that type. All atomic objects and aliased objects are considered to be specified as independently addressable.

Ramification: If the compiler cannot guarantee that an object (including a component) for which aspect Independent or aspect Independent_Components is True is independently addressable from any other nonoverlapping object, then the aspect specification must be rejected.

Similarly, an atomic object (including atomic components) is always independently addressable from any other nonoverlapping object. Any representation item which would prevent this from being true should be rejected, notwithstanding what this Standard says elsewhere (specifically, in the Recommended Level of Support). 

{AI12-0363-1} The Full_Access_Only aspect shall not be specified unless the associated type or object is volatile [(or atomic)]. A full access type is any atomic type, or a volatile type for which the aspect Full_Access_Only is True. A full access object (including a component) is any atomic object, or a volatile object for which the aspect Full_Access_Only is True for the object[ or its type]. A Full_Access_Only aspect is illegal if any subcomponent of the object or type is a full access object or is of a generic formal type.

Ramification: {AI12-0363-1} This last rule breaks privacy, but that is considered OK for representation clauses when there is no clear alternative. Note that atomic objects may be nested, so long as the outer atomic object does not have the Full_Access_Only aspect True. 

Reason: {AI12-0363-1} We disallow subcomponents of a generic formal type in a Full_Access_Only object or type as the actual to a formal type can be a full access type. We could have had a less restrictive rule, but such a use is unlikely as full access only objects are intended to be used to access memory-mapped devices with access restrictions, and those will need a concrete mapping not possible for generic formal types. 

Paragraph 9 was moved to Annex J, "Obsolescent Features". 


#### Legality Rules

{AI05-0229-1} If aspect Independent_Components is specified for a [full_type_declaration](./AA-3.2#S0024), the declaration shall be that of an array or record type.

{AI05-0229-1} {AI12-0001-1} It is illegal to specify either of the aspects Atomic or Atomic_Components to have the value True for an object or type if the implementation cannot support the indivisible and independent reads and updates required by the aspect (see below).

{AI12-0001-1} It is illegal to specify the Size attribute of an atomic object, the Component_Size attribute for an array type with atomic components, or the layout attributes of an atomic component, in a way that prevents the implementation from performing the required indivisible and independent reads and updates.

{AI05-0142-4} {AI05-0218-1} {AI12-0282-1} {AI12-0363-1} If an atomic object is passed as a parameter, then the formal parameter shall either have an atomic type or allow pass by copy. If an atomic object is used as an actual for a generic formal object of mode in out, then the type of the generic formal object shall be atomic. If the [prefix](./AA-4.1#S0093) of an [attribute_reference](./AA-4.1#S0100) for an Access attribute denotes an atomic object [(including a component)], then the designated type of the resulting access type shall be atomic. Corresponding rules apply to volatile objects and to full access objects.

Ramification: {AI05-0142-4} A formal parameter allows pass by copy if it is not aliased and it is of a type that allows pass by copy (that is, is not a by-reference type). 

{AI12-0128-1} {AI12-0363-1} If a nonatomic subcomponent of a full access object is passed as an actual parameter in a call then the formal parameter shall allow pass by copy (and, at run time, the parameter shall be passed by copy). A nonatomic subcomponent of a full access object shall not be used as an actual for a generic formal of mode in out. The [prefix](./AA-4.1#S0093) of an [attribute_reference](./AA-4.1#S0100) for an Access attribute shall not denote a nonatomic subcomponent of a full access object.

{AI05-0218-1} {AI12-0282-1} {AI12-0363-1} If the Atomic, Atomic_Components, Volatile, Volatile_Components, Independent, Independent_Components, or Full_Access_Only aspect is True for a generic formal type, then that aspect shall be True for the actual type. If an atomic type is used as an actual for a generic formal derived type, then the ancestor of the formal type shall be atomic. A corresponding rule applies to volatile types and similarly to full access types.

{AI12-0363-1} If a type with volatile components is used as an actual for a generic formal array type, then the components of the formal type shall be volatile. Furthermore, if the actual type has atomic components and the formal array type has aliased components, then the components of the formal array type shall also be atomic. A corresponding rule applies when the actual type has volatile full access components.

Reason: {AI12-0363-1} The limitations on formal array types are separate for volatile and atomic because of the fact that only volatility is carried down to all subcomponents of a volatile object, while atomicity is not. The goal of both limitations is that we don't want 'Access for an access type to produce a value that designates an object whose atomicity and volatility don't agree with that of the designated type of the access type. The above rules ensure that the generic "sees" the relevant volatility and atomicity. 

{AI05-0229-1} If an aspect Volatile, Volatile_Components, Atomic, or Atomic_Components is directly specified to have the value True for a stand-alone constant object, then the aspect Import shall also be specified as True for it. 

Ramification: Hence, no initialization expression is allowed for such a constant. Note that a constant that is atomic or volatile because of its type is allowed. 

Reason: Stand-alone constants that are explicitly specified as Atomic or Volatile only make sense if they are being manipulated outside the Ada program. From the Ada perspective the object is read-only. Nevertheless, if imported and atomic or volatile, the implementation should presume it might be altered externally. For an imported stand-alone constant that is not atomic or volatile, the implementation can assume that it will not be altered. 

To be honest: {AI05-0218-1} Volatile_Components and Atomic_Components actually are aspects of the anonymous array type; this rule only applies when the aspect is specified directly on the constant object and not when the (named) array type has the aspect. 

{AI05-0009-1} {AI05-0229-1} It is illegal to specify the aspect Independent or Independent_Components as True for a component, object or type if the implementation cannot provide the independent addressability required by the aspect (see 9.10).

{AI05-0009-1} {AI05-0229-1} It is illegal to specify a representation aspect for a component, object or type for which the aspect Independent or Independent_Components is True, in a way that prevents the implementation from providing the independent addressability required by the aspect.

Paragraph 14 was moved to Annex J, "Obsolescent Features". 


#### Dynamic Semantics

For an atomic object (including an atomic component) all reads and updates of the object as a whole are indivisible.

{AI05-0117-1} {AI05-0275-1} All tasks of the program (on all processors) that read or update volatile variables see the same order of updates to the variables. A use of an atomic variable or other mechanism may be necessary to avoid erroneous execution and to ensure that access to nonatomic volatile variables is sequential (see 9.10).

Implementation Note: {AI05-0117-1} {AI05-0275-1} To ensure this, on a multiprocessor, any read or update of an atomic object may require the use of an appropriate memory barrier. 

Discussion: {AI05-0275-1} From 9.10 it follows that (in non-erroneous programs) accesses to variables, including those shared by multiple tasks, are always sequential. This guarantees that no task will ever see partial updates of any variable. For volatile variables (including atomic variables), the above rule additionally specifies that all tasks see the same order of updates.

{AI05-0275-1} If for a shared variable X, a read of X occurs sequentially after an update of X, then the read will return the updated value if X is volatile or atomic, but may or or may not return the updated value if X is nonvolatile. For nonvolatile accesses, a signaling action is needed in order to share the updated value.

{AI05-0275-1} Because accesses to the same atomic variable by different tasks establish a sequential order between the actions of those tasks, implementations may be required to emit memory barriers around such updates or use atomic instructions that imply such barriers. 

Two actions are sequential (see 9.10) if each is the read or update of the same atomic object.

If a type is atomic or volatile and it is not a by-copy type, then the type is defined to be a by-reference type. If any subcomponent of a type is atomic or volatile, then the type is defined to be a by-reference type.

If an actual parameter is atomic or volatile, and the corresponding formal parameter is not, then the parameter is passed by copy. 

Implementation Note: Note that in the case where such a parameter is normally passed by reference, a copy of the actual will have to be produced at the call-site, and a pointer to the copy passed to the formal parameter. If the actual is atomic, any copying has to use indivisible read on the way in, and indivisible write on the way out. 

Reason: It has to be known at compile time whether an atomic or a volatile parameter is to be passed by copy or by reference. For some types, it is unspecified whether parameters are passed by copy or by reference. The above rules further specify the parameter passing rules involving atomic and volatile types and objects. 

{AI12-0128-1} {AI12-0347-1} All reads of or writes to any nonatomic subcomponent of a full access object are performed by reading and/or writing all of the nearest enclosing full access object.

Implementation Note: {AI12-0005-1} {AI12-0128-1} } For example, if a 32-bit record object has four nonatomic components, each occupying one byte, then an assignment to one of those components might normally be implemented on some target machines via some sort of store_byte instruction; if the record object is an atomic full access object then instead a 32-bit read-modify-write must be performed. That read-modify-write need not be atomic, although the read and the write must each separately be atomic. Note that it doesn't matter whether the store_byte instruction would have executed atomically. This rule is needed in some cases for memory-mapped device registers. 

Discussion: The atomic reads and writes associated with accesses to nonatomic components of a full access object that is atomic are normal atomic operations - all of the rules that apply to other atomic operations apply to these as well. In particular, these atomic reads and writes are sequential if they apply to the same object. 


#### Implementation Requirements

{AI12-0128-1} {AI12-0439-1} The external effect of a program (see ) is defined to include each read and update of a volatile or atomic object. The implementation shall not generate any memory reads or updates of atomic or volatile objects other than those specified by the program. However, there may be target-dependent cases where reading or writing a volatile but nonatomic object (typically a component) necessarily involves reading and/or writing neighboring storage, and that neighboring storage can overlap a volatile object. 

Discussion: The presumption is that volatile or atomic objects might reside in an "active" part of the address space where each read has a potential side effect, and at the very least might deliver a different value.

The rule above and the definition of external effect are intended to prevent (at least) the following incorrect optimizations, where V is a volatile variable: 

X:= V; Y:=V; cannot be allowed to be translated as Y:=V; X:=V;

Deleting redundant loads: X:= V; X:= V; shall read the value of V from memory twice.

Deleting redundant stores: V:= X; V:= X; shall write into V twice.

Extra stores: V:= X+Y; should not translate to something like V:= X; V:= V+Y;

Extra loads: X:= V; Y:= X+Z; X:=X+B; should not translate to something like Y:= V+Z; X:= V+B;

Reordering of loads from volatile variables: X:= V1; Y:= V2; (whether or not V1 = V2) should not translate to Y:= V2; X:= V1;

Reordering of stores to volatile variables: V1:= X; V2:= X; should not translate to V2:=X; V1:= X; 

{AI12-0128-1} The part about "target-dependent cases" is intended to let compilers use a read-modify-write operation when a volatile component has a size that cannot be directly read or written with the available machine instructions. (For instance, writing a Boolean component with size 1 in a volatile packed array of Boolean requires a pre-read on most existing machines.) 

This paragraph was deleted.{AI05-0229-1} {AI12-0001-1} 

This paragraph was deleted.{AI05-0009-1} {AI12-0001-1} 


#### Implementation Permissions

{AI12-0282-1} Within the body of an instance of a generic unit that has a formal type T that is not atomic and an actual type that is atomic, if an object O of type T is declared and explicitly specified as atomic, the implementation may introduce an additional copy on passing O to a subprogram with a parameter of type T that is normally passed by reference. A corresponding permission applies to volatile parameter passing. 


#### Implementation Advice

{AI95-00259-01} {AI12-0128-1} A load or store of a volatile object whose size is a multiple of System.Storage_Unit and whose alignment is nonzero, should be implemented by accessing exactly the bits of the object and no others, except in the case of a volatile but nonatomic subcomponent of an atomic object. 

Implementation Advice: A load or store of a volatile object whose size is a multiple of System.Storage_Unit and whose alignment is nonzero, should be implemented by accessing exactly the bits of the object and no others.

Reason: {AI12-0128-1} Since any object can be a volatile object, including packed array components and bit-mapped record components, we require the above only when it is reasonable to assume that the machine can avoid accessing bits outside of the object. The exception is needed so this advice doesn't conflict with other rules of this subclauase. 

Ramification: This implies that the load or store of a volatile object that meets the above requirement should not be combined with that of any other object, nor should it access any bits not belonging to any other object. This means that the suitability of the implementation for memory-mapped I/O can be determined from its documentation, as any cases where the implementation does not follow Implementation Advice must be documented. 

{AI95-00259-01} A load or store of an atomic object should, where possible, be implemented by a single load or store instruction. 

Implementation Advice: A load or store of an atomic object should be implemented by a single load or store instruction.

NOTE 1   An imported volatile or atomic constant behaves as a constant (i.e. read-only) with respect to other parts of the Ada program, but can still be modified by an "external source".

NOTE 2   {AI12-0001-1} Specifying the Pack aspect cannot override the effect of specifying an Atomic or Atomic_Components aspect.

NOTE 3   {AI12-0128-1} {AI12-0440-1} When mapping an Ada object to a memory-mapped hardware register, the Ada object can be declared atomic to ensure that the compiler will read and write exactly the bits of the register as specified in the source code and no others.

Discussion: This is especially important for a write-only hardware register, as in such a case a read-modify-write cycle will not work. The only time the language guarantees such a cycle will not happen is when writing an entire atomic object. If one wants to write individual components of a write-only hardware register (assuming the hardware supports that), those also need to be declared atomic. 


#### Incompatibilities With Ada 83

Pragma Atomic replaces Ada 83's pragma Shared. The name "Shared" was confusing, because the pragma was not used to mark variables as shared. 


#### Wording Changes from Ada 95

{AI95-00259-01} Added Implementation Advice to clarify the meaning of Atomic and Volatile in machine terms. The documentation that this advice applies will make the use of Ada implementations more predictable for low-level (such as device register) programming.

{AI95-00272-01} Added wording to clarify that a slice of an object of an atomic type is not atomic, just like a component of an atomic type is not (necessarily) atomic. 


#### Incompatibilities With Ada 2005

{AI05-0218-1} Correction: Plugged a hole involving volatile components of formal types when the formal type's component has a nonvolatile type. This was done by making certain actual types illegal for formal derived and formal array types; these types were allowed for Ada 95 and Ada 2005. 


#### Extensions to Ada 2005

{AI05-0009-1} {AI05-0229-1} Aspects Independent and Independent_Components are new; they eliminate ambiguity about independent addressability.

{AI05-0229-1} Aspects Atomic, Atomic_Components, Volatile, and Volatile_Components are new; [pragma](./AA-2.8#S0019)s Atomic, Atomic_Components, Volatile, and Volatile_Components are now obsolescent. 


#### Wording Changes from Ada 2005

{AI05-0117-1} {AI05-0275-1} Revised the definition of volatile to eliminate overspecification and simply focus on the root requirement (that all tasks see the same view of volatile objects). This is not an inconsistency; "memory" arguably includes on-chip caches so long as those are kept consistent. Moreover, it is difficult to imagine a program that could tell the difference.

{AI05-0142-4} Added wording to take explicitly aliased parameters (see 6.1) into account when determining the legality of parameter passing of volatile and atomic objects. 


#### Inconsistencies With Ada 2012

{AI12-0128-1}  Required that nonatomic components of a atomic object use a read-modify-write cycle, as well as clarifying that such a cycle is allowed for volatile objects that aren't exactly the size of a machine scalar. That can introduce a read-modify-write cycle where one was not used previously (for instance, if the components are exactly byte-sized on most machines); it is thought that this will more often fix bugs with access to hardware than it will cause them. If this is an issue, declaring the components themselves as atomic will restore the previous behavior. 


#### Extensions to Ada 2012

{AI12-0282-1} These aspects now can be specified for generic formal types.

{AI12-0363-1} Aspect Full_Access_Only is new; it can be used to guarantee access to a complete device register for any operation even when the register is mapped to a number of components. 


#### Wording Changes from Ada 2012

{AI12-0001-1} Corrigendum: Clarified that aliased objects are considered to be specified as independently addressable, and also eliminated an unnecessary rule. 


## C.6.1  The Package System.Atomic_Operations

{AI12-0234-1} The language-defined package System.Atomic_Operations is the parent of a set of child units that provide facilities for manipulating objects of atomic types and for supporting lock-free synchronization. The subprograms of this subsystem are Intrinsic subprograms (see 6.3.1) in order to provide convenient access to machine operations that can provide these capabilities if they are available in the target environment. 


#### Static Semantics

{AI12-0234-1} The library package System.Atomic_Operations has the following declaration:

```ada
package System.Atomic_Operations
   with Pure, Nonblocking is
end System.Atomic_Operations;

```

{AI12-0234-1} System.Atomic_Operations serves as the parent of other language-defined library units that manipulate atomic objects; its declaration is empty.

{AI12-0234-1} A call to a subprogram is said to be lock-free  if the subprogram is guaranteed to return from the call while keeping the processor of the logical thread of control busy for the duration of the call.

{AI12-0234-1} In each child package, a function Is_Lock_Free(...) is provided to check whether the operations of the child package can all be provided lock-free for a given object. Is_Lock_Free returns True if operations defined in the child package are lock-free when applied to the object denoted by Item, and Is_Lock_Free returns False otherwise.


#### Extensions to Ada 2012

{AI12-0234-1} This package is new. 


## C.6.2  The Package System.Atomic_Operations.Exchange

{AI12-0234-1} The language-defined generic package System.Atomic_Operations.Exchange provides the following operations:

To atomically compare the value of two atomic objects, and update the first atomic object with a desired value if both objects were found to be equal, or otherwise update the second object with the value of the first object.

To atomically update the value of an atomic object, and then return the value that the atomic object had just prior to the update.


#### Static Semantics

{AI12-0234-1} The generic library package System.Atomic_Operations.Exchange has the following declaration:

```ada
generic
   type Atomic_Type is private with Atomic;
package System.Atomic_Operations.Exchange
   with Pure, Nonblocking is

```

```ada
   function Atomic_Exchange (Item  : aliased in out Atomic_Type;
                             Value : Atomic_Type) return Atomic_Type
      with Convention =&gt Intrinsic;

```

```ada
   function Atomic_Compare_And_Exchange 
     (Item    : aliased in out Atomic_Type;
      Prior   : aliased in out Atomic_Type;
      Desired : Atomic_Type) return Boolean
      with Convention =&gt Intrinsic;

```

```ada
   function Is_Lock_Free (Item : aliased Atomic_Type) return Boolean
      with Convention =&gt Intrinsic;

```

```ada
end System.Atomic_Operations.Exchange;

```

{AI12-0234-1} Atomic_Exchange atomically assigns the value of Value to Item, and returns the previous value of Item.

{AI12-0234-1} Atomic_Compare_And_Exchange first evaluates the value of Prior. Atomic_Compare_And_Exchange then performs the following steps as part of a single indivisible operation:

evaluates the value of Item;

compares the value of Item with the value of Prior;

if equal, assigns Item the value of Desired;

otherwise, makes no change to the value of Item.

{AI12-0234-1} After these steps, if the value of Item and Prior did not match, Prior is assigned the original value of Item, and the function returns False. Otherwise, Prior is unaffected and the function returns True.


#### Examples

{AI12-0234-1} Example of a spin lock using Atomic_Exchange:

```ada
type Atomic_Boolean is new Boolean with Atomic;
package Exchange is new
   Atomic_Operations.Exchange (Atomic_Type =&gt Atomic_Boolean);

```

```ada
Lock : aliased Atomic_Boolean := False;

```

```ada
...

```

```ada
begin -- Some critical section, trying to get the lock:

```

```ada
   -- Obtain the lock
   while Exchange.Atomic_Exchange (Item =&gt Lock, Value =&gt True) loop
      null;
   end loop;

```

```ada
   ... -- Do stuff

```

```ada
   Lock := False; -- Release the lock
end;

```


#### Extensions to Ada 2012

{AI12-0234-1} This package is new. 


## C.6.3  The Package System.Atomic_Operations.Test_and_Set

{AI12-0321-1} The language-defined package System.Atomic_Operations.Test_And_Set provides an operation to atomically set and clear an atomic flag object. 


#### Static Semantics

{AI12-0321-1} The library package System.Atomic_Operations.Test_And_Set has the following declaration:

```ada
package System.Atomic_Operations.Test_And_Set
   with Pure, Nonblocking is

```

```ada
   type Test_And_Set_Flag is mod implementation-defined
      with Atomic, Default_Value =&gt 0, Size =&gt implementation-defined;

```

```ada
   function Atomic_Test_And_Set
     (Item : aliased in out Test_And_Set_Flag) return Boolean
      with Convention =&gt Intrinsic;

```

```ada
   procedure Atomic_Clear
     (Item : aliased in out Test_And_Set_Flag)
      with Convention =&gt Intrinsic;

```

```ada
   function Is_Lock_Free
     (Item : aliased Test_And_Set_Flag) return Boolean
      with Convention =&gt Intrinsic;

```

```ada
end System.Atomic_Operations.Test_And_Set;

```

Implementation defined: The modulus and size of Test_and_Set_Flag.

{AI12-0321-1} Test_And_Set_Flag represents the state of an atomic flag object. An atomic flag object can either be considered to be set or cleared.

{AI12-0321-1} Atomic_Test_And_Set performs an atomic test-and-set operation on Item. Item is set to some implementation-defined nonzero value. The function returns True if the previous contents were nonzero, and otherwise returns False.

Implementation defined: The value used to represent the set value for Atomic_Test_and_Set.

{AI12-0321-1} Atomic_Clear performs an atomic clear operation on Item. After the operation, Item contains 0. This call should be used in conjunction with Atomic_Test_And_Set.


#### Extensions to Ada 2012

{AI12-0321-1} This package is new. 


## C.6.4  The Package System.Atomic_Operations.Integer_Arithmetic

{AI12-0321-1} {AI12-0364-1} The language-defined generic package System.Atomic_Operations.Integer_Arithmetic provides operations to perform arithmetic atomically on objects of integer types. 


#### Static Semantics

{AI12-0321-1} {AI12-0364-1} The generic library package System.Atomic_Operations.Integer_Arithmetic has the following declaration:

```ada
generic
   type Atomic_Type is range &lt&gt with Atomic;
package System.Atomic_Operations.Integer_Arithmetic
   with Pure, Nonblocking is

```

```ada
   procedure Atomic_Add (Item  : aliased in out Atomic_Type;
                         Value : Atomic_Type)
      with Convention =&gt Intrinsic;

```

```ada
   procedure Atomic_Subtract (Item  : aliased in out Atomic_Type;
                              Value : Atomic_Type)
      with Convention =&gt Intrinsic;

```

```ada
   function Atomic_Fetch_And_Add
     (Item  : aliased in out Atomic_Type;
      Value : Atomic_Type) return Atomic_Type
      with Convention =&gt Intrinsic;

```

```ada
   function Atomic_Fetch_And_Subtract
     (Item  : aliased in out Atomic_Type;
      Value : Atomic_Type) return Atomic_Type
      with Convention =&gt Intrinsic;

```

```ada
   function Is_Lock_Free (Item : aliased Atomic_Type) return Boolean
      with Convention =&gt Intrinsic;

```

```ada
end System.Atomic_Operations.Integer_Arithmetic;

```

{AI12-0321-1} The operations of this package are defined as follows:

```ada
procedure Atomic_Add (Item  : aliased in out Atomic_Type;
                      Value : Atomic_Type)
   with Convention =&gt Intrinsic;

```

{AI12-0321-1} Atomically performs: Item := Item + Value;

```ada
procedure Atomic_Subtract (Item  : aliased in out Atomic_Type;
                           Value : Atomic_Type)
   with Convention =&gt Intrinsic;

```

{AI12-0321-1} Atomically performs: Item := Item - Value;

```ada
function Atomic_Fetch_And_Add
  (Item  : aliased in out Atomic_Type;
   Value : Atomic_Type) return Atomic_Type
   with Convention =&gt Intrinsic;

```

{AI12-0321-1} Atomically performs: Tmp := Item; Item := Item + Value; return Tmp;

```ada
function Atomic_Fetch_And_Subtract
  (Item  : aliased in out Atomic_Type;
   Value : Atomic_Type) return Atomic_Type
   with Convention =&gt Intrinsic;

```

{AI12-0321-1} Atomically performs: Tmp := Item; Item := Item - Value; return Tmp;


#### Extensions to Ada 2012

{AI12-0321-1} This package is new. 


## C.6.5  The Package System.Atomic_Operations.Modular_Arithmetic

{AI12-0364-1} The language-defined generic package System.Atomic_Operations.Modular_Arithmetic provides operations to perform arithmetic atomically on objects of modular types. 


#### Static Semantics

{AI12-0364-1} The generic library package System.Atomic_Operations.Modular_Arithmetic has the following declaration:

```ada
generic
   type Atomic_Type is mod &lt&gt with Atomic;
package System.Atomic_Operations.Modular_Arithmetic
   with Pure, Nonblocking is

```

```ada
   procedure Atomic_Add (Item  : aliased in out Atomic_Type;
                         Value : Atomic_Type)
      with Convention =&gt Intrinsic;

```

```ada
   procedure Atomic_Subtract (Item  : aliased in out Atomic_Type;
                              Value : Atomic_Type)
      with Convention =&gt Intrinsic;

```

```ada
   function Atomic_Fetch_And_Add
     (Item  : aliased in out Atomic_Type;
      Value : Atomic_Type) return Atomic_Type
      with Convention =&gt Intrinsic;

```

```ada
   function Atomic_Fetch_And_Subtract
     (Item  : aliased in out Atomic_Type;
      Value : Atomic_Type) return Atomic_Type
      with Convention =&gt Intrinsic;

```

```ada
   function Is_Lock_Free (Item : aliased Atomic_Type) return Boolean
      with Convention =&gt Intrinsic;

```

```ada
end System.Atomic_Operations.Modular_Arithmetic;

```

{AI12-0364-1} The operations of this package are defined as follows:

```ada
procedure Atomic_Add (Item  : aliased in out Atomic_Type;
                      Value : Atomic_Type)
   with Convention =&gt Intrinsic;

```

{AI12-0364-1} Atomically performs: Item := Item + Value;

```ada
procedure Atomic_Subtract (Item  : aliased in out Atomic_Type;
                           Value : Atomic_Type)
   with Convention =&gt Intrinsic;

```

{AI12-0364-1} Atomically performs: Item := Item - Value;

```ada
function Atomic_Fetch_And_Add
  (Item  : aliased in out Atomic_Type;
   Value : Atomic_Type) return Atomic_Type
   with Convention =&gt Intrinsic;

```

{AI12-0364-1} Atomically performs: Tmp := Item; Item := Item + Value; return Tmp;

```ada
function Atomic_Fetch_And_Subtract
  (Item  : aliased in out Atomic_Type;
   Value : Atomic_Type) return Atomic_Type
   with Convention =&gt Intrinsic;

```

{AI12-0364-1} Atomically performs: Tmp := Item; Item := Item - Value; return Tmp;


#### Extensions to Ada 2012

{AI12-0364-1} This package is new. 
