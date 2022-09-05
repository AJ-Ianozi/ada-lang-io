---
sidebar_position:  170
---

# E.2  Categorization of Library Units

[Library units can be categorized according to the role they play in a distributed program. Certain restrictions are associated with each category to ensure that the semantics of a distributed program remain close to the semantics for a nondistributed program.]

{AI05-0243-1} {AI12-0417-1} A categorization aspect restricts the declarations, child units, or semantic dependences of the library unit to which it applies. A categorized library unit is a library unit that has a categorization aspect that is True.

{AI05-0243-1} {AI12-0417-1} The aspects Shared_Passive, Remote_Types, and Remote_Call_Interface are categorization aspects. In addition, for the purposes of this Annex, the aspect Pure (see 10.2.1) is considered a categorization aspect.

{8652/0078} {AI95-00048-01} {AI05-0243-1} [ A library package or generic library package is called a shared passive library unit if the Shared_Passive aspect of the unit is True.  A library package or generic library package is called a remote types library unit if the Remote_Types aspect of the unit is True.  A library unit is called a remote call interface if the Remote_Call_Interface aspect of the unit is True.] A normal library unit is one for which no categorization aspect is True. 

Proof: {AI05-0243-1} {AI05-0299-1} These terms (other than "normal library unit") are really defined in the following subclauses. 

Ramification: {8652/0078} {AI95-00048-01} A library subprogram can be a remote call interface, but it cannot be a remote types or shared passive library unit. 

{AI05-0206-1} {AI05-0243-1} {AI05-0269-1} {AI05-0299-1} [The various categories of library units and the associated restrictions are described in this and the following subclauses. The categories are related hierarchically in that the library units of one category can depend semantically only on library units of that category or an earlier one in the hierarchy, except that the body of a remote types or remote call interface library unit is unrestricted, the declaration of a remote types or remote call interface library unit may depend on preelaborated normal library units that are mentioned only in private with clauses, and all categories can depend on limited views.

{AI05-0243-1} {AI05-0269-1} The overall hierarchy (including declared pure) is as follows, with a lower-numbered category being "earlier in the hierarchy" in the sense of the previous paragraph: 

a)Declared Pure

b)Shared Passive

c)Remote Types

d)Remote Call Interface

e)Normal (no restrictions) 

Paragraphs 7 through 11 were deleted. 

Declared pure and shared passive library units are preelaborated. The declaration of a remote types or remote call interface library unit is required to be preelaborable. ]

Paragraph 13 and 14 were deleted. 


#### Wording Changes from Ada 95

{8652/0078} {AI95-00048-01} Corrigendum: Clarified that a library subprogram can be a remote call interface unit.

{8652/0079} {AI95-00208-01} Corrigendum: Removed the requirement that types be represented the same in all partitions, because it prevents the definition of heterogeneous distributed systems and goes much further than required. 


#### Wording Changes from Ada 2005

{AI05-0206-1} {AI05-0299-1} We now allow private withs of preelaborated units in Remote Types and Remote Call Interface units; this is documented as an extension in the subclauses where this is defined normatively.

{AI05-0243-1} {AI05-0299-1} We have introduced categorization aspects; these are documented as extensions in the subclauses where they actually are defined. 


#### Wording Changes from Ada 2012

{AI12-0417-1} Categorization pragmas are now obsolescent. 


## E.2.1  Shared Passive Library Units

[A shared passive library unit is used for managing global data shared between active partitions. The restrictions on shared passive library units prevent the data or tasks of one active partition from being accessible to another active partition through references implicit in objects declared in the shared passive library unit.] 


#### Language Design Principles

The restrictions governing a shared passive library unit are designed to ensure that objects and subprograms declared in the package can be used safely from multiple active partitions, even though the active partitions live in different address spaces, and have separate run-time systems. 

Paragraphs 2 and 3 were moved to Annex J, "Obsolescent Features". 


#### Legality Rules

{AI05-0243-1} {AI12-0417-1} When the library unit aspect (see 13.1.1) Shared_Passive of a library unit is True, the library unit is a shared passive library unit. The following restrictions apply to such a library unit:

Aspect Description for Shared_Passive: A given package is used to represent shared memory in a distributed system.

[it shall be preelaborable (see 10.2.1);] 

Ramification: It cannot contain library-level declarations of protected objects with entries, nor of task objects. Task objects are disallowed because passive partitions don't have any threads of control of their own, nor any run-time system of their own. Protected objects with entries are disallowed because an entry queue contains references to calling tasks, and that would require in effect a pointer from a passive partition back to a task in some active partition. 

{AI05-0243-1} it shall depend semantically only upon declared pure or shared passive [library_item](./AA-10.1#S0287)s; 

Reason: Shared passive packages cannot depend semantically upon remote types packages because the values of an access type declared in a remote types package refer to the local heap of the active partition including the remote types package. 

Ramification: {AI05-0243-1} We say [library_item](./AA-10.1#S0287) here, so that limited views are allowed; those are not library units, but they are [library_item](./AA-10.1#S0287). 

{8652/0080} {AI95-00003-01} {AI12-0038-1} {AI12-0320-1} it shall not contain a library-level declaration:

{AI12-0320-1} of an access type that designates a class-wide type; 

Reason: These kinds of access types are disallowed because the object designated by an access value of such a type could contain an implicit reference back to the active partition on whose behalf the designated object was created. 

{AI12-0320-1} of a type with a part that is of a task type;

{AI12-0320-1} of a type with a part that is of a protected type with [entry_declaration](./AA-9.5#S0257)s; or

{AI12-0038-1} {AI12-0320-1} that contains a name that denotes a type declared within a declared-pure package, if that type has a part that is of an access type; for the purposes of this rule, the parts considered include those of the full views of any private types or private extensions.

Reason: This rule breaks privacy by looking into the full views of private types. Avoiding privacy breakage here would have required disallowing the use in a shared passive package of any private type declared in a declared-pure package, which would have been severely incompatible. 

Notwithstanding the definition of accessibility given in 3.10.2, the declaration of a library unit P1 is not accessible from within the declarative region of a shared passive library unit P2, unless the shared passive library unit P2 depends semantically on P1. 

Discussion: We considered a more complex rule, but dropped it. This is the simplest rule that recognizes that a shared passive package may outlive some other library package, unless it depends semantically on that package. In a nondistributed program, all library packages are presumed to have the same lifetime.

{AI12-0417-1} Implementations may define additional aspects or pragmas that force two library packages to be in the same partition, or to have the same lifetime, which would allow this rule to be relaxed in the presence of such aspects or pragmas. 


#### Static Semantics

A shared passive library unit is preelaborated.


#### Post-Compilation Rules

A shared passive library unit shall be assigned to at most one partition within a given program.

Notwithstanding the rule given in 10.2, a compilation unit in a given partition does not need (in the sense of 10.2) the shared passive library units on which it depends semantically to be included in that same partition; they will typically reside in separate passive partitions.

To be honest: {AI12-0359-1} This rule is necessary so that the shared data of a shared passive partition is not duplicated in each partition. It does not imply a particular implementation; in particular, code can be replicated in each active partition. 

Implementation Note: {AI12-0359-1} One possible implementation for a shared passive partition is as a shared library that is mapped into the address space of of each active partition. In such case, both the code and the data of the passive partition would be mapped into the active partitions and directly called/accessed from each active partition. For instance, on Microsoft Windows a DLL has the correct semantics.

Alternatively, the shared data can be represented as a file or persistent memory, with the shared code being replicated in each active partition. Code replication is an as-if optimization; it should be impossible to tell where the code lives since no elaboration is necessary. 


#### Wording Changes from Ada 95

{8652/0080} {AI95-00003-01} Corrigendum: Corrected the wording to allow access types in blocks in shared passive generic packages. 


#### Extensions to Ada 2005

{AI05-0243-1} {AI12-0417-1} Shared_Passive is now a categorization aspect, so it can be specified by an [aspect_specification](./AA-13.1#S0346) 


#### Incompatibilities With Ada 2012

{AI12-0038-1} Corrigendum: Uses of access types declared in declared-pure units are not allowed in library-level shared passive packages. These were allowed by Ada 2005 and Ada 2012, but it is unlikely that they work properly, as active partitions could disappear before the shared-passive partition. As such, the new errors are more likely to catch bugs than to cause them. 


#### Wording Changes from Ada 2012

{AI12-0417-1} The pragma Shared_Passive is now obsolescent. 


## E.2.2  Remote Types Library Units

[A remote types library unit supports the definition of types intended for use in communication between active partitions.] 


#### Language Design Principles

The restrictions governing a remote types package are similar to those for a declared pure package. However, the restrictions are relaxed deliberately to allow such a package to contain declarations that violate the stateless property of pure packages, though it is presumed that any state-dependent properties are essentially invisible outside the package. 

Paragraphs 2 and 3 were moved to Annex J, "Obsolescent Features". 


#### Legality Rules

{AI05-0243-1} {AI12-0417-1} When the library unit aspect (see 13.1.1) Remote_Types of a library unit is True, the library unit is a remote types library unit. The following restrictions apply to such a library unit:

Aspect Description for Remote_Types: Types in a given package may be used in remote procedure calls.

[it shall be preelaborable;]

{AI05-0206-1} {AI05-0243-1} it shall depend semantically only on declared pure [library_item](./AA-10.1#S0287)s, shared passive library units, other remote types library units, or preelaborated normal library units that are mentioned only in private with clauses;

Ramification: {AI05-0243-1} We say declared pure [library_item](./AA-10.1#S0287) here, so that (all) limited views are allowed; those are not library units, but they are declared pure [library_item](./AA-10.1#S0287)s. 

it shall not contain the declaration of any variable within the visible part of the library unit; 

Reason: This is essentially a "methodological" restriction. A separate copy of a remote types package is included in each partition that references it, just like a normal package. Nevertheless, a remote types package is thought of as an "essentially pure" package for defining types to be used for interpartition communication, and it could be misleading to declare visible objects when no remote data access is actually being provided. 

{AI95-00240-01} {AI95-00366-01} the full view of each type declared in the visible part of the library unit that has any available stream attributes shall support external streaming (see 13.13.2). 

Reason: This is to prevent the use of the predefined Read and Write attributes of an access type as part of the Read and Write attributes of a visible type. 

Ramification: {AI95-00366-01} Types that do not have available stream attributes are excluded from this rule; that means that attributes do not need to be specified for most limited types. It is only necessary to specify attributes for nonlimited types that have a part that is of any access type, and for extensions of limited types with available stream attributes where the [record_extension_part](./AA-3.9#S0075) includes a subcomponent of an access type, where the access type does not have specified attributes. 

{8652/0082} {AI95-00164-01} {AI05-0060-1} A named access type declared in the visible part of a remote types or remote call interface library unit is called a remote access type. Such a type shall be:

{8652/0082} {AI95-00164-01} an access-to-subprogram type, or

{8652/0082} {AI95-00164-01} {AI05-0060-1} a general access type that designates a class-wide limited private type, a class-wide limited interface type, or a class-wide private extension all of whose ancestors are either private extensions, limited interface types, or limited private types. 

{8652/0081} {AI95-00004-01} A type that is derived from a remote access type is also a remote access type.

{AI12-0283-1} A remote access-to-subprogram type shall not be nonblocking (see 9.5).

Reason: All calls on remote subprograms are considered potentially blocking, so they cannot statically be allowed in nonblocking code. 

Ramification: The type declaration of a remote type is illegal if the Nonblocking aspect is True, either implicitly by inheritance or by explicit specification. 

The following restrictions apply to the use of a remote access-to-subprogram type: 

{AI95-00431-01} A value of a remote access-to-subprogram type shall be converted only to or from another (subtype-conformant) remote access-to-subprogram type;

The [prefix](./AA-4.1#S0093) of an Access [attribute_reference](./AA-4.1#S0100) that yields a value of a remote access-to-subprogram type shall statically denote a (subtype-conformant) remote subprogram. 

The following restrictions apply to the use of a remote access-to-class-wide type: 

{8652/0083} {AI95-00047-01} {AI95-00240-01} {AI95-00366-01} {AI05-0060-1} {AI05-0101-1} The primitive subprograms of the corresponding specific type shall only have access parameters if they are controlling formal parameters. The primitive functions of the corresponding specific type shall only have an access result if it is a controlling access result. Each noncontrolling formal parameter and noncontrolling result type shall support external streaming (see 13.13.2);

{AI05-0060-1} {AI05-0215-1} {AI05-0269-1} The corresponding specific type shall not have a primitive procedure with the Synchronization aspect specified unless the [synchronization_kind](./AA-9.5#S0256) is Optional (see 9.5);

A value of a remote access-to-class-wide type shall be explicitly converted only to another remote access-to-class-wide type;

{AI12-0034-1} A value of a remote access-to-class-wide type shall be dereferenced (or implicitly converted to an anonymous access type) only as part of a dispatching call to a primitive operation of the designated type where the value designates a controlling operand of the call (see E.4, "Remote Subprogram Calls"); 

Ramification: {AI12-0034-1} Stream attributes of the designated type are not primitive operations of the designated type, and thus remote calls to them are prohibited by this rule. This is good, as the access parameter of a stream attribute does not have external streaming, and thus cannot be a parameter of a remote call. 

{AI05-0101-1} A controlling access result value for a primitive function with any controlling operands of the corresponding specific type shall either be explicitly converted to a remote access-to-class-wide type or be part of a dispatching call where the value designates a controlling operand of the call;

{AI95-00366-01} {AI12-0085-1} The Storage_Pool attribute is not defined for a remote access-to-class-wide type; the expected type for an [allocator](./AA-4.8#S0164) shall not be a remote access-to-class-wide type. A remote access-to-class-wide type shall not be an actual parameter for a generic formal access type. The Storage_Size attribute of a remote access-to-class-wide type yields 0.  The Storage_Pool and Storage_Size aspects shall not be specified for a remote access-to-class-wide type. 

Reason: {AI05-0005-1} All of these restrictions are because there is no storage pool associated with a remote access-to-class-wide type. The Storage_Size is defined to be 0 so that there is no conflict with the rules for pure units. 

Ramification: {AI12-0085-1} The prohibition against specifying the Storage_Size aspect for an access-to-class-wide type applies to any method of doing that, including via either an [aspect_specification](./AA-13.1#S0346) or an [attribute_definition_clause](./AA-13.3#S0349). 


#### Erroneous Execution

{AI12-0076-1} Execution is erroneous if some operation (other than the initialization or finalization of the object) modifies the value of a constant object declared in the visible part of a remote types package.

Discussion: {AI12-0005-1} This could be accomplished via a self-referencing pointer or via squirrelling away a writable pointer to a controlled object. 

NOTE 1   {AI12-0442-1} A remote types library unit is not necessarily pure, and the types it defines can include levels of indirection implemented by using access types. User-specified Read and Write attributes (see 13.13.2) provide for sending values of such a type between active partitions, with Write marshalling the representation, and Read unmarshalling any levels of indirection.

NOTE 2   {AI05-0060-1} The value of a remote access-to-class-wide limited interface can designate an object of a nonlimited type derived from the interface.

NOTE 3   {AI05-0060-1} {AI12-0440-1} A remote access type can designate a class-wide synchronized, protected, or task interface type. 

Proof: Synchronized, protected, and task interfaces are all considered limited interfaces, see 3.9.4. 


#### Incompatibilities With Ada 95

{AI95-00240-01} {AI05-0248-1} Amendment Correction: The wording was changed from "user-specified" to "available" read and write attributes. (This was then further changed, see below.) This means that an access type with the attributes specified in the private part would originally have been sufficient to allow the access type to be used in a remote type, but that is no longer allowed. Similarly, the attributes of a remote type that has access components have to be specified in the visible part. These changes were made so that the rules were consistent with the rules introduced for the Corrigendum for stream attributes; moreover, legality should not depend on the contents of the private part. 


#### Extensions to Ada 95

{AI95-00366-01} {AI05-0005-1} Remote types that cannot be streamed (that is, have no available stream attributes) do not require the specification of stream attributes. This is necessary so that most extensions of Limited_Controlled do not need stream attributes defined (otherwise there would be a significant incompatibility, as Limited_Controlled would need stream attributes, and then all extensions of it also would need stream attributes). 


#### Wording Changes from Ada 95

{8652/0081} {AI95-00004-01} Corrigendum: Added missing wording so that a type derived from a remote access type is also a remote access type.

{8652/0083} {AI95-00047-01} Corrigendum: Clarified that user-defined Read and Write attributes are required for the primitive subprograms corresponding to a remote access-to-class-wide type.

{8652/0082} {AI95-00164-01} Corrigendum: Added missing wording so that a remote access type can designate an appropriate private extension.

{AI95-00366-01} Changed the wording to use the newly defined term type that supports external streaming, so that various issues with access types in pure units and implicitly declared attributes for type extensions are properly handled.

{AI95-00366-01} Defined Storage_Size to be 0 for remote access-to-class-wide types, rather than having it undefined. This eliminates issues with pure units requiring a defined storage size.

{AI95-00431-01} Corrected the wording so that a value of a local access-to-subprogram type cannot be converted to a remote access-to-subprogram type, as intended (and required by the ACATS). 


#### Incompatibilities With Ada 2005

{AI05-0101-1} {AI12-0005-1} {AI12-0005-1} Correction: Added rules about the returning of remote access-to-class-wide types; this had been missed in the past. While programs that returned unstreamable types from RCI functions were legal, it is not clear what they could have done (as the results could not be marshalled). Similarly, RCI functions that return remote controlling access types could try to save those values, but it is unlikely that a compiler would know how to do that usefully. Thus, it seems unlikely that any real programs will be impacted by these changes. 


#### Extensions to Ada 2005

{AI05-0060-1} Correction: Clarified that anonymous access types are never remote access types (and can be used in remote types units subject to the normal restrictions). Added wording to allow limited class-wide interfaces to be designated by remote access types.

{AI05-0206-1} Added wording to allow private withs of preelaborated normal units in the specification of a remote types unit.

{AI05-0243-1} {AI12-0417-1} Remote_Types is now a categorization aspect, so it can be specified by an [aspect_specification](./AA-13.1#S0346) 


#### Wording Changes from Ada 2012

{AI12-0034-1} Corrigendum: Clarified that dispatching remote stream attribute calls are prohibited. We don't document this as an incompatibility, as the stream parameter cannot be marshalled for a remote call (it doesn't have external streaming), so it's impossible that any working program depends on this functionality.

{AI12-0076-1} Corrigendum: Explicitly stated that modifying a visible constant in a remote types package is erroneous. We don't document this as inconsistent as implementations certainly can still do whatever they were previously doing (no change is required); moreover, this case (and many more) were erroneous in Ada 2005 and before, so we're just restoring the previous semantics.

{AI12-0085-1} Corrigendum: Clarified that specifying the Storage_Pool or Storage_Size aspect for an access-to-class-wide type is not allowed. The intent is clear, and no implementation has ever allowed specifying the aspects (the attributes already cannot be specified), so we don't document this as an incompatibility.

{AI12-0283-1} Added a rule to ensure that potentially blocking remote calls are not considered nonblocking.

{AI12-0417-1} The pragma Remote_Types is now obsolescent.


## E.2.3  Remote Call Interface Library Units

[A remote call interface library unit can be used as an interface for remote procedure calls (RPCs) (or remote function calls) between active partitions.] 


#### Language Design Principles

The restrictions governing a remote call interface library unit are intended to ensure that the values of the actual parameters in a remote call can be meaningfully sent between two active partitions. 

Paragraphs 2 through 6 were moved to Annex J, "Obsolescent Features". 


#### Legality Rules

{8652/0078} {AI95-00048-01} {AI05-0243-1} {AI12-0417-1} When the library unit aspect (see 13.1.1) Remote_Call_Interface of a library unit is True, the library unit is a remote call interface (RCI). A subprogram declared in the visible part of such a library unit, or declared by such a library unit, is called a remote subprogram.

Aspect Description for Remote_Call_Interface: Subprograms in a given package may be used in remote procedure calls.

{AI05-0206-1} {AI05-0243-1} The declaration of an RCI library unit shall be preelaborable (see 10.2.1), and shall depend semantically only upon declared pure [library_item](./AA-10.1#S0287)s, shared passive library units, remote types library units, other remote call interface library units, or preelaborated normal library units that are mentioned only in private with clauses.

Ramification: {AI05-0243-1} We say declared pure [library_item](./AA-10.1#S0287) here, so that (all) limited views are allowed; those are not library units, but they are declared pure [library_item](./AA-10.1#S0287)s. 

{8652/0078} {AI95-00048-01} In addition, the following restrictions apply to an RCI library unit: 

{8652/0078} {AI95-00048-01} its visible part shall not contain the declaration of a variable; 

Reason: {8652/0078} {AI95-00048-01} Remote call interface units do not provide remote data access. A shared passive package has to be used for that. 

{8652/0078} {AI95-00048-01} its visible part shall not contain the declaration of a limited type; 

Reason: {AI95-00240-01} {AI95-00366-01} We disallow the declaration of task and protected types, since calling an entry or a protected subprogram implicitly passes an object of a limited type (the target task or protected object). We disallow other limited types since we require that such types have available Read and Write attributes, but we certainly don't want the Read and Write attributes themselves to involve remote calls (thereby defeating their purpose of marshalling the value for remote calls). 

{8652/0078} {AI95-00048-01} its visible part shall not contain a nested [generic_declaration](./AA-12.1#S0310); 

Reason: This is disallowed because the body of the nested generic would presumably have access to data inside the body of the RCI package, and if instantiated in a different partition, remote data access might result, which is not supported. 

{8652/0078} {AI95-00048-01} {AI05-0229-1} it shall not be, nor shall its visible part contain, the declaration of a subprogram for which aspect Inline is True;

{AI12-0283-1} it shall not be, nor shall its visible part contain, the declaration of a subprogram that is nonblocking (see 9.5);

Reason: All such subprograms are remote subprograms, and all calls on remote subprograms are considered potentially blocking, so they cannot statically be allowed in nonblocking code. 

Ramification: The type declaration of a remote subprogram is illegal if the Nonblocking aspect is True, either implicitly by inheritance or by explicit specification. 

{8652/0078} {AI95-00048-01} {AI95-00240-01} {AI95-00366-01} {AI05-0101-1} it shall not be, nor shall its visible part contain, a subprogram (or access-to-subprogram) declaration whose profile has a parameter or result of a type that does not support external streaming (see 13.13.2);

Ramification: {AI05-0101-1} No anonymous access types support external streaming, so they are never allowed as parameters or results of RCI subprograms. 

any public child of the library unit shall be a remote call interface library unit. 

Reason: {AI12-0417-1} No restrictions apply to the private part of an RCI package, and since a public child can "see" the private part of its parent, such a child must itself have a Remote_Call_Interface aspect, and be assigned to the same partition (see below). 

Discussion: {AI12-0417-1} We considered making the public child of an RCI package implicitly RCI, but it seemed better to require an explicit aspect to avoid any confusion.

Note that there is no need for a private child to be an RCI package, since it can only be seen from the body of its parent or its siblings, all of which are required to be in the same active partition. 

{AI12-0002-1} Specification of a stream-oriented attribute is illegal in the specification of a remote call interface library unit. In addition to the places where Legality Rules normally apply (see 12.3), this rule applies also in the private part of an instance of a generic unit.

Ramification: This applies whether the stream-oriented attribute is specified with an [attribute_definition_clause](./AA-13.3#S0349) or with an [aspect_specification](./AA-13.1#S0346). 

Reason: Stream-oriented attributes are intended to be used locally (to marshall values), while subprograms in a remote call interface are intended to be used remotely. We resolve the conflict by banning the local use routines. The type should be included in a remote types package and imported into the remote call interface package. 

{AI05-0229-1} {AI12-0417-1} All_Calls_Remote is a library unit aspect. If the All_Calls_Remote aspect of a library unit is True, the library unit shall be a remote call interface.

Aspect Description for All_Calls_Remote: All indirect or dispatching remote subprogram calls, and all direct remote subprogram calls, should use the Partition Communication Subsystem.


#### Post-Compilation Rules

A remote call interface library unit shall be assigned to at most one partition of a given program. A remote call interface library unit whose parent is also an RCI library unit shall be assigned only to the same partition as its parent. 

Implementation Note: {8652/0078} {AI95-00048-01} The declaration of an RCI unit, with a calling-stub body, is automatically included in all active partitions with compilation units that depend on it. However the whole RCI library unit, including its (non-stub) body, will only be in one of the active partitions. 

Notwithstanding the rule given in 10.2, a compilation unit in a given partition that semantically depends on the declaration of an RCI library unit, needs (in the sense of 10.2) only the declaration of the RCI library unit, not the body, to be included in that same partition. [Therefore, the body of an RCI library unit is included only in the partition to which the RCI library unit is explicitly assigned.]


#### Implementation Requirements

{8652/0078} {AI95-00048-01} {AI05-0229-1} {AI12-0031-1} If aspect All_Calls_Remote is True for a given RCI library unit, then the implementation shall route any of the following calls through the Partition Communication Subsystem (PCS); see E.5: 

{AI12-0031-1} A direct call to a subprogram of the RCI unit from outside the declarative region of the unit;

{AI12-0031-1} An indirect call through a remote access-to-subprogram value that designates a subprogram of the RCI unit;

{AI12-0031-1} A dispatching call with a controlling operand designated by a remote access-to-class-wide value whose tag identifies a type declared in the RCI unit. 

Discussion: {8652/0078} {AI95-00048-01} {AI05-0229-1} {AI12-0031-1} {AI12-0005-1} When this aspect is False (or not used), it is presumed that most implementations will not make calls through the PCS if the call originates in the same partition as that of the RCI unit. When this aspect is True, all indirect or dispatching remote subprogram calls to the RCI unit, and all direct calls from outside the subsystem rooted at the RCI unit, are treated like calls from outside the partition, ensuring that the PCS is involved in all such calls (for debugging, redundancy, etc.). 

Reason: {AI12-0031-1} {AI12-0005-1} There is no point in forcing local direct calls (including calls from children) to go through the PCS, since on the target system these calls are always local, and all the units are in the same active partition. 


#### Implementation Permissions

{AI05-0243-1} {AI12-0417-1} {AI12-0444-1} An implementation may omit support for the Remote_Call_Interface aspect or the All_Calls_Remote aspect. [Explicit message-based communication between active partitions can be supported as an alternative to RPC.] 

Ramification: {AI12-0417-1} Of course, it is pointless to support the All_Calls_Remote aspect if the Remote_Call_Interface aspect (or some approximate equivalent) is not supported. 


#### Incompatibilities With Ada 95

{AI95-00240-01} {AI05-0248-1} Amendment Correction: The wording was changed from "user-specified" to "available" read and write attributes. (This was then further changed, see below.) This means that a type with the attributes specified in the private part would originally have been allowed as a formal parameter of an RCI subprogram, but that is no longer allowed. This change was made so that the rules were consistent with the rules introduced for the Corrigendum for stream attributes; moreover, legality should not depend on the contents of the private part. 


#### Wording Changes from Ada 95

{8652/0078} {AI95-00048-01} Corrigendum: Changed the wording to allow a library subprogram to be a remote call interface unit.

{AI95-00366-01} Changed the wording to use the newly defined term type that supports external streaming, so that various issues with access types in pure units and implicitly declared attributes for type extensions are properly handled. 


#### Incompatibilities With Ada 2005

{AI05-0101-1} Correction: Added a rule to ensure that function results are streamable; this was missing in previous versions of Ada. While programs that returned unstreamable types from RCI functions were legal, it is not clear what they could have done (as the results could not be marshalled). Thus, it seems unlikely that any real programs will be impacted by this change. 


#### Extensions to Ada 2005

{AI05-0206-1} Added wording to allow private withs of preelaborated normal units in the specification of a remote call interface unit.

{AI05-0229-1} {AI12-0417-1} All_Calls_Remote is now a representation aspect, so it can be specified by an [aspect_specification](./AA-13.1#S0346)

{AI05-0243-1} {AI12-0417-1} Remote_Call_Interface is now a categorization aspect, so it can be specified by an [aspect_specification](./AA-13.1#S0346) 


#### Inconsistencies With Ada 2012

{AI12-0031-1} Corrigendum: Redefined when indirect and dispatching remote calls have to be remote for a unit for which the aspect All_Calls_Remote is True. With the new rules, a local target called indirectly or via dispatching will be routed through the PCS, while that was not necessarily true in earlier Ada. If a program depended on local targets not being routed through the PCS even when All_Calls_Remote is used, then it might behave differently or fail in Ada 2012 as corrected with this refinement. This is highly unlikely as the PCS is going to be able to communicate with any partition of the program, including the local partition. 


#### Incompatibilities With Ada 2012

{AI12-0002-1} Correction: Specifying user-defined stream-oriented attributes is now illegal in an RCI unit. If a program contains such attributes, they and their associated type will need to be moved to a remote types package (which is then imported into the RCI unit). 


#### Wording Changes from Ada 2012

{AI12-0283-1} Added a rule to ensure that potentially blocking remote calls are not considered nonblocking.

{AI12-0417-1} The pragmas Remote_Call_Interface and All_Calls_Remote are now obsolescent. 
