---
sidebar_position:  20
---

# 3.4  Derived Types and Classes

{AI95-00401-01} {AI95-00419-01} A [derived_type_definition](./AA-3.4#S0035) defines a derived type (and its first subtype) whose characteristics are derived from those of a parent type, and possibly from progenitor types. 

Glossary entry: A derived type is a type defined in terms of one or more other types given in a derived type definition. The first of those types is the parent type of the derived type and any others are progenitor types. Each class containing the parent type or a progenitor type also contains the derived type. The derived type inherits properties such as components and primitive operations from the parent and progenitors. A type together with the types derived from it (directly or indirectly) form a derivation class.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[derived type], Def=[a type defined in terms of a parent type and zero or more progenitor types given in a derived type definition], Note1=[A derived type inherits properties such as components and primitive operations from its parent and progenitors.], Note2=[A type together with the types derived from it (directly or indirectly) form a derivation class.]

{AI95-00442-01} A class of types is a set of types that is closed under derivation; that is, if the parent or a progenitor type of a derived type belongs to a class, then so does the derived type. By saying that a particular group of types forms a class, we are saying that all derivatives of a type in the set inherit the characteristics that define that set. The more general term category of types is used for a set of types whose defining characteristics are not necessarily inherited by derivatives; for example, limited, abstract, and interface are all categories of types, but not classes of types.

Ramification: A class of types is also a category of types. 


#### Syntax

{AI95-00251-01} {AI95-00419-01} derived_type_definition<a id="S0035"></a> ::= 
    [abstract] [limited] new parent_[subtype_indication](./AA-3.2#S0027) [[and [interface_list](./AA-3.9#S0078)] [record_extension_part](./AA-3.9#S0075)]


#### Legality Rules

{AI95-00251-01} {AI95-00401-01} {AI95-00419-01} The parent_[subtype_indication](./AA-3.2#S0027) defines the parent subtype; its type is the parent type. The [interface_list](./AA-3.9#S0078) defines the progenitor types (see 3.9.4). A derived type has one parent type and zero or more progenitor types.

Glossary entry: The parent of a derived type is the first type given in the definition of the derived type. The parent can be almost any kind of type, including an interface type.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[parent of a derived type], Def=[the first ancestor type given in the definition of the derived type], Note1=[The parent can be almost any kind of type, including an interface type.]

A type shall be completely defined (see 3.11.1) prior to being specified as the parent type in a [derived_type_definition](./AA-3.4#S0035) - [the [full_type_declaration](./AA-3.2#S0024)s for the parent type and any of its subcomponents have to precede the [derived_type_definition](./AA-3.4#S0035).] 

Discussion: This restriction does not apply to the ancestor type of a private extension - see 7.3; such a type need not be completely defined prior to the [private_extension_declaration](./AA-7.3#S0233). However, the restriction does apply to record extensions, so the ancestor type will have to be completely defined prior to the [full_type_declaration](./AA-3.2#S0024) corresponding to the [private_extension_declaration](./AA-7.3#S0233). 

Reason: We originally hoped we could relax this restriction. However, we found it too complex to specify the rules for a type derived from an incompletely defined limited type that subsequently became nonlimited. 

{AI95-00401-01} If there is a [record_extension_part](./AA-3.9#S0075), the derived type is called a record extension of the parent type. A [record_extension_part](./AA-3.9#S0075) shall be provided if and only if the parent type is a tagged type. [An [interface_list](./AA-3.9#S0078) shall be provided only if the parent type is a tagged type.] 

Proof: {AI95-00401-01} The syntax only allows an [interface_list](./AA-3.9#S0078) to appear with a [record_extension_part](./AA-3.9#S0075), and a [record_extension_part](./AA-3.9#S0075) can only be provided if the parent type is a tagged type. We give the last sentence anyway for completeness. 

Implementation Note: We allow a record extension to inherit discriminants; an early version of Ada 9X did not. If the parent subtype is unconstrained, it can be implemented as though its discriminants were repeated in a new [known_discriminant_part](./AA-3.7#S0061) and then used to constrain the old ones one-for-one. However, in an extension aggregate, the discriminants in this case do not appear in the component association list. 

Ramification: {AI95-00114-01} This rule needs to be rechecked in the visible part of an instance of a generic unit because of the "only if" part of the rule. For example: 

```ada
generic
   type T is private;
package P is
   type Der is new T;
end P;

```

```ada
package I is new P (Some_Tagged_Type); -- illegal

```

{AI95-00114-01} The instantiation is illegal because a tagged type is being extended in the visible part without a [record_extension_part](./AA-3.9#S0075). Note that this is legal in the private part or body of an instance, both to avoid a contract model violation, and because no code that can see that the type is actually tagged can also see the derived type declaration.

No recheck is needed for derived types with a [record_extension_part](./AA-3.9#S0075), as that has to be derived from something that is known to be tagged (otherwise the template is illegal). 

{AI95-00419-01} {AI05-0096-1} If the reserved word limited appears in a [derived_type_definition](./AA-3.4#S0035), the parent type shall be a limited type. If the parent type is a tagged formal type, then in addition to the places where Legality Rules normally apply (see 12.3), this rule applies also in the private part of an instance of a generic unit. 

Reason: We allow limited because we don't inherit limitedness from interfaces, so we must have a way to derive a limited type from interfaces. The word limited has to be legal when the parent could be an interface, and that includes generic formal abstract types. Since we have to allow it in this case, we might as well allow it everywhere as documentation, to make it explicit that the type is limited.

However, we do not want to allow limited when the parent is nonlimited: limitedness cannot change in a derivation tree.

If the parent type is an untagged limited formal type with an actual type that is nonlimited, we allow derivation as a limited type in the private part or body as no place could have visibility on the resulting type where it was known to be nonlimited (outside of the instance). (See the previous paragraph's annotations for an explanation of this.) However, if the parent type is a tagged limited formal type with an actual type that is nonlimited, it would be possible to pass a value of the limited type extension to a class-wide type of the parent, which would be nonlimited. That's too weird to allow (even though all of the extension components would have to be nonlimited because the rules of 3.9.1 are rechecked), so we have a special rule to prevent that in the private part (type extension from a formal type is illegal in a generic package body). 


#### Static Semantics

The first subtype of the derived type is unconstrained if a [known_discriminant_part](./AA-3.7#S0061) is provided in the declaration of the derived type, or if the parent subtype is unconstrained. Otherwise, the constraint of the first subtype corresponds to that of the parent subtype in the following sense: it is the same as that of the parent subtype except that for a range constraint (implicit or explicit), the value of each bound of its range is replaced by the corresponding value of the derived type. 

Discussion: A [digits_constraint](./AA-3.5#S0050) in a [subtype_indication](./AA-3.2#S0027) for a decimal fixed point subtype always imposes a range constraint, implicitly if there is no explicit one given. See 3.5.9, "Fixed Point Types". 

{AI95-00231-01} The first subtype of the derived type excludes null (see 3.10) if and only if the parent subtype excludes null.

{AI05-0110-1} The characteristics and implicitly declared primitive subprograms of the derived type are defined as follows: 

Ramification: {AI05-0110-1} The characteristics of a type do not include its primitive subprograms (primitive subprograms include predefined operators). The rules governing availability/visibility and inheritance of characteristics are separate from those for primitive subprograms. 

{AI95-00251-01} {AI95-00401-01} {AI95-00442-01} [If the parent type or a progenitor type belongs to a class of types, then the derived type also belongs to that class.] The following sets of types, as well as any higher-level sets composed from them, are classes in this sense[, and hence the characteristics defining these classes are inherited by derived types from their parent or progenitor types]: signed integer, modular integer, ordinary fixed, decimal fixed, floating point, enumeration, boolean, character, access-to-constant, general access-to-variable, pool-specific access-to-variable, access-to-subprogram, array, string, non-array composite, nonlimited, untagged record, tagged, task, protected, and synchronized tagged. 

Discussion: This is inherent in our notion of a "class" of types. It is not mentioned in the initial definition of "class" since at that point type derivation has not been defined. In any case, this rule ensures that every class of types is closed under derivation. 

If the parent type is an elementary type or an array type, then the set of possible values of the derived type is a copy of the set of possible values of the parent type. For a scalar type, the base range of the derived type is the same as that of the parent type. 

Discussion: The base range of a type defined by an [integer_type_definition](./AA-3.5#S0041) or a [real_type_definition](./AA-3.5#S0044) is determined by the _definition, and is not necessarily the same as that of the corresponding root numeric type from which the newly defined type is implicitly derived. Treating numerics types as implicitly derived from one of the two root numeric types is simply to link them into a type hierarchy; such an implicit derivation does not follow all the rules given here for an explicit [derived_type_definition](./AA-3.4#S0035). 

If the parent type is a composite type other than an array type, then the components, protected subprograms, and entries that are declared for the derived type are as follows: 

The discriminants specified by a new [known_discriminant_part](./AA-3.7#S0061), if there is one; otherwise, each discriminant of the parent type (implicitly declared in the same order with the same specifications) - in the latter case, the discriminants are said to be inherited, or if unknown in the parent, are also unknown in the derived type;

Each nondiscriminant component, entry, and protected subprogram of the parent type, implicitly declared in the same order with the same declarations; these components, entries, and protected subprograms are said to be inherited; 

Ramification: The profiles of entries and protected subprograms do not change upon type derivation, although the type of the "implicit" parameter identified by the [prefix](./AA-4.1#S0093) of the [name](./AA-4.1#S0091) in a call does.

To be honest: Any name in the parent [type_declaration](./AA-3.2#S0023) that denotes the current instance of the type is replaced with a name denoting the current instance of the derived type, converted to the parent type.

Each component declared in a [record_extension_part](./AA-3.9#S0075), if any. 

Declarations of components, protected subprograms, and entries, whether implicit or explicit, occur immediately within the declarative region of the type, in the order indicated above, following the parent [subtype_indication](./AA-3.2#S0027). 

Discussion: The order of declarations within the region matters for [record_aggregate](./AA-4.3#S0107)s and [extension_aggregate](./AA-4.3#S0111)s. 

Ramification: In most cases, these things are implicitly declared immediately following the parent [subtype_indication](./AA-3.2#S0027). However, 7.3.1, "Private Operations" defines some cases in which they are implicitly declared later, and some cases in which the are not declared at all. 

Discussion: The place of the implicit declarations of inherited components matters for visibility - they are not visible in the [known_discriminant_part](./AA-3.7#S0061) nor in the parent [subtype_indication](./AA-3.2#S0027), but are usually visible within the [record_extension_part](./AA-3.9#S0075), if any (although there are restrictions on their use). Note that a discriminant specified in a new [known_discriminant_part](./AA-3.7#S0061) is not considered "inherited" even if it has the same name and subtype as a discriminant of the parent type. 

This paragraph was deleted.{AI95-00419-01} 

[For each predefined operator of the parent type, there is a corresponding predefined operator of the derived type.] 

Proof: This is a ramification of the fact that each class that includes the parent type also includes the derived type, and the fact that the set of predefined operators that is defined for a type, as described in 4.5, is determined by the classes to which it belongs. 

Reason: Predefined operators are handled separately because they follow a slightly different rule than user-defined primitive subprograms. In particular the systematic replacement described below does not apply fully to the relational operators for Boolean and the exponentiation operator for Integer. The relational operators for a type derived from Boolean still return Standard.Boolean. The exponentiation operator for a type derived from Integer still expects Standard.Integer for the right operand. In addition, predefined operators "reemerge" when a type is the actual type corresponding to a generic formal type, so they need to be well defined even if hidden by user-defined primitive subprograms. 

{AI95-00401-01} For each user-defined primitive subprogram (other than a user-defined equality operator - see below) of the parent type or of a progenitor type that already exists at the place of the [derived_type_definition](./AA-3.4#S0035), there exists a corresponding inherited primitive subprogram of the derived type with the same defining name. Primitive user-defined equality operators of the parent type and any progenitor types are also inherited by the derived type, except when the derived type is a nonlimited record extension, and the inherited operator would have a profile that is type conformant with the profile of the corresponding predefined equality operator; in this case, the user-defined equality operator is not inherited, but is rather incorporated into the implementation of the predefined equality operator of the record extension (see 4.5.2). 

Ramification: We say "...already exists..." rather than "is visible" or "has been declared" because there are certain operations that are declared later, but still exist at the place of the [derived_type_definition](./AA-3.4#S0035), and there are operations that are never declared, but still exist. These cases are explained in 7.3.1.

Note that nonprivate extensions can appear only after the last primitive subprogram of the parent - the freezing rules ensure this. 

Reason: A special case is made for the equality operators on nonlimited record extensions because their predefined equality operators are already defined in terms of the primitive equality operator of their parent type (and of the tagged components of the extension part). Inheriting the parent's equality operator as is would be undesirable, because it would ignore any components of the extension part. On the other hand, if the parent type is limited, then any user-defined equality operator is inherited as is, since there is no predefined equality operator to take its place. 

Ramification: {AI95-00114-01} Because user-defined equality operators are not inherited by nonlimited record extensions, the formal parameter names of = and /= revert to Left and Right, even if different formal parameter names were used in the user-defined equality operators of the parent type. 

Discussion: {AI95-00401-01} This rule only describes what operations are inherited; the rules that describe what happens when there are conflicting inherited subprograms are found in 8.3. 

{AI95-00401-01} {AI05-0164-1} {AI05-0240-1} The profile of an inherited subprogram (including an inherited enumeration literal) is obtained from the profile of the corresponding (user-defined) primitive subprogram of the parent or progenitor type, after systematic replacement of each subtype of its profile (see 6.1) that is of the parent or progenitor type, other than those subtypes found in the designated profile of an [access_definition](./AA-3.10#S0084), with a corresponding subtype of the derived type. For a given subtype of the parent or progenitor type, the corresponding subtype of the derived type is defined as follows: 

If the declaration of the derived type has neither a [known_discriminant_part](./AA-3.7#S0061) nor a [record_extension_part](./AA-3.9#S0075), then the corresponding subtype has a constraint that corresponds (as defined above for the first subtype of the derived type) to that of the given subtype.

If the derived type is a record extension, then the corresponding subtype is the first subtype of the derived type.

If the derived type has a new [known_discriminant_part](./AA-3.7#S0061) but is not a record extension, then the corresponding subtype is constrained to those values that when converted to the parent type belong to the given subtype (see 4.6). 

Reason: An inherited subprogram of an untagged type has an Intrinsic calling convention, which precludes the use of the Access attribute. We preclude 'Access because correctly performing all required constraint checks on an indirect call to such an inherited subprogram was felt to impose an undesirable implementation burden.

{AI05-0164-1} Note that the exception to substitution of the parent or progenitor type applies only in the profiles of anonymous access-to-subprogram types. The exception is necessary to avoid calling an access-to-subprogram with types and/or constraints different than expected by the actual routine. 

{AI95-00401-01} The same formal parameters have [default_expression](./AA-3.7#S0063)s in the profile of the inherited subprogram. [Any type mismatch due to the systematic replacement of the parent or progenitor type by the derived type is handled as part of the normal type conversion associated with parameter passing - see 6.4.1.] 

Reason: {AI95-00401-01} We don't introduce the type conversion explicitly here since conversions to record extensions or on access parameters are not generally legal. Furthermore, any type conversion would just be "undone" since the subprogram of the parent or progenitor is ultimately being called anyway. (Null procedures can be inherited from a progenitor without being overridden, so it is possible to call subprograms of an interface.) 

{AI95-00401-01} If a primitive subprogram of the parent or progenitor type is visible at the place of the [derived_type_definition](./AA-3.4#S0035), then the corresponding inherited subprogram is implicitly declared immediately after the [derived_type_definition](./AA-3.4#S0035). Otherwise, the inherited subprogram is implicitly declared later or not at all, as explained in 7.3.1.

A derived type can also be defined by a [private_extension_declaration](./AA-7.3#S0233) (see 7.3) or a [formal_derived_type_definition](./AA-12.5#S0325) (see 12.5.1). Such a derived type is a partial view of the corresponding full or actual type.

All numeric types are derived types, in that they are implicitly derived from a corresponding root numeric type (see 3.5.4 and 3.5.6).


#### Dynamic Semantics

The elaboration of a [derived_type_definition](./AA-3.4#S0035) creates the derived type and its first subtype, and consists of the elaboration of the [subtype_indication](./AA-3.2#S0027) and the [record_extension_part](./AA-3.9#S0075), if any. If the [subtype_indication](./AA-3.2#S0027) depends on a discriminant, then only those expressions that do not depend on a discriminant are evaluated. 

Discussion: {AI95-00251-01} We don't mention the [interface_list](./AA-3.9#S0078), because it does not need elaboration (see 3.9.4). This is consistent with the handling of [discriminant_part](./AA-3.7#S0059)s, which aren't elaborated either. 

{AI95-00391-01} {AI95-00401-01} For the execution of a call on an inherited subprogram, a call on the corresponding primitive subprogram of the parent or progenitor type is performed; the normal conversion of each actual parameter to the subtype of the corresponding formal parameter (see 6.4.1) performs any necessary type conversion as well. If the result type of the inherited subprogram is the derived type, the result of calling the subprogram of the parent or progenitor is converted to the derived type, or in the case of a null extension, extended to the derived type using the equivalent of an [extension_aggregate](./AA-4.3#S0111) with the original result as the [ancestor_part](./AA-4.3#S0112) and null record as the [record_component_association_list](./AA-4.3#S0108). 

Discussion: {AI95-00391-01} If an inherited function returns the derived type, and the type is a nonnull record extension, then the inherited function shall be overridden, unless the type is abstract (in which case the function is abstract, and (unless overridden) cannot be called except via a dispatching call). See 3.9.3. 

NOTE 1   Classes are closed under derivation - any class that contains a type also contains its derivatives. Operations available for a given class of types are available for the derived types in that class.

NOTE 2   Evaluating an inherited enumeration literal is equivalent to evaluating the corresponding enumeration literal of the parent type, and then converting the result to the derived type. This follows from their equivalence to parameterless functions. 

NOTE 3   A generic subprogram is not a subprogram, and hence cannot be a primitive subprogram and cannot be inherited by a derived type. On the other hand, an instance of a generic subprogram can be a primitive subprogram, and hence can be inherited.

NOTE 4   If the parent type is an access type, then the parent and the derived type share the same storage pool; there is a null access value for the derived type and it is the implicit initial value for the type. See 3.10.

NOTE 5   If the parent type is a boolean type, the predefined relational operators of the derived type deliver a result of the predefined type Boolean (see 4.5.2). If the parent type is an integer type, the right operand of the predefined exponentiation operator is of the predefined type Integer (see 4.5.6).

NOTE 6   Any discriminants of the parent type are either all inherited, or completely replaced with a new set of discriminants.

NOTE 7   {AI12-0442-1} For an inherited subprogram, the subtype of a formal parameter of the derived type can be such that it has no value in common with the first subtype of the derived type. 

Proof: This happens when the parent subtype is constrained to a range that does not overlap with the range of a subtype of the parent type that appears in the profile of some primitive subprogram of the parent type. For example: 

```ada
type T1 is range 1..100;
subtype S1 is T1 range 1..10;
procedure P(X : in S1);  -- P is a primitive subprogram
type T2 is new T1 range 11..20;
-- implicitly declared:
-- procedure P(X : in T2'Base range 1..10);
--      X cannot be in T2'First .. T2'Last

```

NOTE 8   If the reserved word abstract is given in the declaration of a type, the type is abstract (see 3.9.3).

NOTE 9   {AI95-00251-01} {AI95-00401-01} An interface type that has a progenitor type "is derived from" that type. A [derived_type_definition](./AA-3.4#S0035), however, never defines an interface type.

NOTE 10   {AI95-00345-01} It is illegal for the parent type of a [derived_type_definition](./AA-3.4#S0035) to be a synchronized tagged type. 

Proof: {AI05-0299-1} 3.9.1 prohibits record extensions whose parent type is a synchronized tagged type, and this subclause requires tagged types to have a record extension. Thus there are no legal derivations. Note that a synchronized interface can be used as a progenitor in an [interface_type_definition](./AA-3.9#S0077) as well as in task and protected types, but we do not allow concrete extensions of any synchronized tagged type. 


#### Examples

Examples of derived type declarations: 

```ada
type Local_Coordinate is new Coordinate;   --  two different types
type Midweek is new Day range Tue .. Thu;  --  see 3.5.1
type Counter is new Positive;              --  same range as Positive 

```

```ada
type Special_Key is new Key_Manager.Key;   --  see 7.3.1
  -- the inherited subprograms have the following specifications: 
  --         procedure Get_Key(K : out Special_Key);
  --         function "&lt"(X,Y : Special_Key) return Boolean;

```


#### Inconsistencies With Ada 83

When deriving from a (nonprivate, nonderived) type in the same visible part in which it is defined, if a predefined operator had been overridden prior to the derivation, the derived type will inherit the user-defined operator rather than the predefined operator. The work-around (if the new behavior is not the desired behavior) is to move the definition of the derived type prior to the overriding of any predefined operators.


#### Incompatibilities With Ada 83

When deriving from a (nonprivate, nonderived) type in the same visible part in which it is defined, a primitive subprogram of the parent type declared before the derived type will be inherited by the derived type. This can cause upward incompatibilities in cases like this: 

```ada
   package P is
      type T is (A, B, C, D);
      function F( X : T := A ) return Integer;
      type NT is new T;
      -- inherits F as
      -- function F( X : NT := A ) return Integer;
      -- in Ada 95 only
      ...
   end P;
   ...
   use P;  -- Only one declaration of F from P is use-visible in
           -- Ada 83;  two declarations of F are use-visible in
           -- Ada 95.
begin
   ...
   if F &gt 1 then ... -- legal in Ada 83, ambiguous in Ada 95

```


#### Extensions to Ada 83

The syntax for a [derived_type_definition](./AA-3.4#S0035) is amended to include an optional [record_extension_part](./AA-3.9#S0075) (see 3.9.1).

A derived type may override the discriminants of the parent by giving a new [discriminant_part](./AA-3.7#S0059).

The parent type in a [derived_type_definition](./AA-3.4#S0035) may be a derived type defined in the same visible part.

When deriving from a type in the same visible part in which it is defined, the primitive subprograms declared prior to the derivation are inherited as primitive subprograms of the derived type. See 3.2.3. 


#### Wording Changes from Ada 83

We now talk about the classes to which a type belongs, rather than a single class.


#### Extensions to Ada 95

{AI05-0190-1} {AI95-00251-01} {AI95-00401-01} A derived type may inherit from multiple (interface) progenitors, as well as the parent type - see 3.9.4, "Interface Types".

{AI95-00419-01} A derived type may specify that it is a limited type. This is required for interface ancestors (from which limitedness is not inherited), but it is generally useful as documentation of limitedness. 


#### Wording Changes from Ada 95

{AI95-00391-01} Defined the result of functions for null extensions (which we no longer require to be overridden - see 3.9.3).

{AI95-00442-01} Defined the term "category of types" and used it in wording elsewhere; also specified the language-defined categories that form classes of types (this was never normatively specified in Ada 95). 


#### Incompatibilities With Ada 2005

{AI05-0096-1} Correction: Added a (re)check that limited type extensions never are derived from nonlimited types in generic private parts. This is disallowed as it would make it possible to pass a limited object to a nonlimited class-wide type, which could then be copied. This is only possible using Ada 2005 syntax, so examples in existing programs should be rare. 


#### Wording Changes from Ada 2005

{AI05-0110-1} Correction: Added wording to clarify that the characteristics of derived types are formally defined here. (This is the only place in the Reference Manual that actually spells out what sorts of things are actually characteristics, which is rather important.)

{AI05-0164-1} Correction: Added wording to ensure that anonymous access-to-subprogram types don't get modified on derivation. 


## 3.4.1  Derivation Classes

In addition to the various language-defined classes of types, types can be grouped into derivation classes. 


#### Static Semantics

{AI95-00251-01} {AI95-00401-01} A derived type is derived from its parent type directly; it is derived indirectly from any type from which its parent type is derived. A derived type, interface type, type extension, task type, protected type, or formal derived type is also derived from every ancestor of each of its progenitor types, if any. The derivation class of types for a type T (also called the class rooted at T) is the set consisting of T (the root type of the class) and all types derived from T (directly or indirectly) plus any associated universal or class-wide types (defined below). 

Discussion: Note that the definition of "derived from" is a recursive definition. We don't define a root type for all interesting language-defined classes, though presumably we could. 

To be honest: By the class-wide type "associated" with a type T, we mean the type T'Class. Similarly, the universal type associated with root_integer, root_real, and root_fixed are universal_integer, universal_real, and universal_fixed, respectively. 

{AI95-00230-01} {AI12-0437-1} Every type is one of a specific type, a class-wide type, or a universal type. A specific type is one defined by a [type_declaration](./AA-3.2#S0023), a [formal_type_declaration](./AA-12.5#S0320), or a full type definition embedded in another construct. Class-wide and universal types are implicitly defined, to act as representatives for an entire class of types, as follows: 

To be honest: The root types root_integer, root_real, and root_fixed are also specific types. They are declared in the specification of package Standard. 

Class-wide types Class-wide types are defined for [(and belong to)] each derivation class rooted at a tagged type (see 3.9). Given a subtype S of a tagged type T, S'Class is the [subtype_mark](./AA-3.2#S0028) for a corresponding subtype of the tagged class-wide type T'Class. Such types are called "class-wide" because when a formal parameter is defined to be of a class-wide type T'Class, an actual parameter of any type in the derivation class rooted at T is acceptable (see 8.6).

The set of values for a class-wide type T'Class is the discriminated union of the set of values of each specific type in the derivation class rooted at T (the tag acts as the implicit discriminant - see 3.9). Class-wide types have no primitive subprograms of their own. However, as explained in 3.9.2, operands of a class-wide type T'Class can be used as part of a dispatching call on a primitive subprogram of the type T. The only components [(including discriminants)] of T'Class that are visible are those of T. If S is a first subtype, then S'Class is a first subtype. 

Reason: We want S'Class to be a first subtype when S is, so that an [attribute_definition_clause](./AA-13.3#S0349) like "for S'Class'Output use ...;" will be legal. 

{AI95-00230-01} {AI12-0445-1} Universal types Universal types are defined for [(and belong to)] the integer, real, fixed point, and access classes, and are referred to in this document as respectively, universal_integer, universal_real, universal_fixed, and universal_access. These are analogous to class-wide types for these language-defined elementary classes. As with class-wide types, if a formal parameter is of a universal type, then an actual parameter of any type in the corresponding class is acceptable. In addition, a value of a universal type (including an integer or real [numeric_literal](./AA-2.4#S0006), or the literal null) is "universal" in that it is acceptable where some particular type in the class is expected (see 8.6).

The set of values of a universal type is the undiscriminated union of the set of values possible for any definable type in the associated class. Like class-wide types, universal types have no primitive subprograms of their own. However, their "universality" allows them to be used as operands with the primitive subprograms of any type in the corresponding class. 

Discussion: A class-wide type is only class-wide in one direction, from specific to class-wide, whereas a universal type is class-wide (universal) in both directions, from specific to universal and back.

{AI95-00230-01} We considered defining class-wide or perhaps universal types for all derivation classes, not just tagged classes and these four elementary classes. However, this was felt to overly weaken the strong-typing model in some situations. Tagged types preserve strong type distinctions thanks to the run-time tag. Class-wide or universal types for untagged types would weaken the compile-time type distinctions without providing a compensating run-time-checkable distinction.

We considered defining standard names for the universal numeric types so they could be used in formal parameter specifications. However, this was felt to impose an undue implementation burden for some implementations. 

To be honest: Formally, the set of values of a universal type is actually a copy of the undiscriminated union of the values of the types in its class. This is because we want each value to have exactly one type, with explicit or implicit conversion needed to go between types. An alternative, consistent model would be to associate a class, rather than a particular type, with a value, even though any given expression would have a particular type. In that case, implicit type conversions would not generally need to change the value, although an associated subtype conversion might need to. 

The integer and real numeric classes each have a specific root type in addition to their universal type, named respectively root_integer and root_real.

{AI12-0325-1} A class-wide or universal type is said to cover all of the types in its class. In addition, universal_integer covers a type that has a specified Integer_Literal aspect, while universal_real covers a type that has a specified Real_Literal aspect (see 4.2.1). A specific type covers only itself.

{AI95-00230-01} {AI95-00251-01} A specific type T2 is defined to be a descendant of a type T1 if T2 is the same as T1, or if T2 is derived (directly or indirectly) from T1. A class-wide type T2'Class is defined to be a descendant of type T1 if T2 is a descendant of T1. Similarly, the numeric universal types are defined to be descendants of the root types of their classes. If a type T2 is a descendant of a type T1, then T1 is called an ancestor of T2. An ultimate ancestor of a type is an ancestor of that type that is not itself a descendant of any other type. Every untagged type has a unique ultimate ancestor. 

Ramification: A specific type is a descendant of itself. Class-wide types are considered descendants of the corresponding specific type, and do not have any descendants of their own.

A specific type is an ancestor of itself. The root of a derivation class is an ancestor of all types in the class, including any class-wide types in the class. 

Discussion: The terms root, parent, ancestor, and ultimate ancestor are all related. For example: 

{AI95-00251-01} Each type has at most one parent, and one or more ancestor types; each untagged type has exactly one ultimate ancestor. In Ada 83, the term "parent type" was sometimes used more generally to include any ancestor type (e.g. RM83-9.4(14)). In Ada 95, we restrict parent to mean the immediate ancestor.

A class of types has at most one root type; a derivation class has exactly one root type.

The root of a class is an ancestor of all of the types in the class (including itself).

The type root_integer is the root of the integer class, and is the ultimate ancestor of all integer types. A similar statement applies to root_real. 

Glossary entry: An ancestor of a type is the type itself or, in the case of a type derived from other types, its parent type or one of its progenitor types or one of their ancestors. Note that ancestor and descendant are inverse relationships.

Glossary entry: A type is a descendant of itself, its parent and progenitor types, and their ancestors. Note that descendant and ancestor are inverse relationships.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[ancestor of a type], Def=[the type itself or, in the case of a type derived from other types, its parent type or one of its progenitor types or one of their ancestors], Note1=[Ancestor and descendant are inverse relationships.]

Version=[5],Kind=(AddedNormal),Group=[T],Term=[descendant of a type], Def=[the type itself or a type derived (directly or indirectly) from it], Note1=[Descendant and ancestor are inverse relationships.]

An inherited component [(including an inherited discriminant)] of a derived type is inherited from a given ancestor of the type if the corresponding component was inherited by each derived type in the chain of derivations going back to the given ancestor.

NOTE   Because operands of a universal type are acceptable to the predefined operators of any type in their class, ambiguity can result. For universal_integer and universal_real, this potential ambiguity is resolved by giving a preference (see 8.6) to the predefined operators of the corresponding root types (root_integer and root_real, respectively). Hence, in an apparently ambiguous expression like 

1 + 4 &lt 7

where each of the literals is of type universal_integer, the predefined operators of root_integer will be preferred over those of other specific integer types, thereby resolving the ambiguity. 

Ramification: Except for this preference, a root numeric type is essentially like any other specific type in the associated numeric class. In particular, the result of a predefined operator of a root numeric type is not "universal" (implicitly convertible) even if both operands were. 


#### Wording Changes from Ada 95

{AI95-00230-01} Updated the wording to define the universal_access type. This was defined to make null for anonymous access types sensible.

{AI95-00251-01} {AI95-00401-01} The definitions of ancestors and descendants were updated to allow multiple ancestors (necessary to support interfaces). 


#### Wording Changes from Ada 2012

{AI12-0325-1} Updated the wording to say that universal types cover the types with the appropriate user-defined literal. 
