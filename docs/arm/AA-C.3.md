---
sidebar_position:  146
---

# C.3  Interrupt Support

{AI05-0299-1} [This subclause specifies the language-defined model for hardware interrupts in addition to mechanisms for handling interrupts.] 


#### Dynamic Semantics

[An interrupt represents a class of events that are detected by the hardware or the system software.] Interrupts are said to occur. An occurrence of an interrupt is separable into generation and delivery. Generation of an interrupt is the event in the underlying hardware or system that makes the interrupt available to the program. Delivery is the action that invokes part of the program as response to the interrupt occurrence. Between generation and delivery, the interrupt occurrence [(or interrupt)] is pending. Some or all interrupts may be blocked. When an interrupt is blocked, all occurrences of that interrupt are prevented from being delivered. Certain interrupts are reserved. The set of reserved interrupts is implementation defined. A reserved interrupt is either an interrupt for which user-defined handlers are not supported, or one which already has an attached handler by some other implementation-defined means. Program units can be connected to nonreserved interrupts. While connected, the program unit is said to be attached to that interrupt. The execution of that program unit, the interrupt handler, is invoked upon delivery of the interrupt occurrence. 

This paragraph was deleted.

To be honest: As an obsolescent feature, interrupts may be attached to task entries by an address clause. See J.7.1. 

While a handler is attached to an interrupt, it is called once for each delivered occurrence of that interrupt. While the handler executes, the corresponding interrupt is blocked.

While an interrupt is blocked, all occurrences of that interrupt are prevented from being delivered. Whether such occurrences remain pending or are lost is implementation defined.

Each interrupt has a default treatment which determines the system's response to an occurrence of that interrupt when no user-defined handler is attached. The set of possible default treatments is implementation defined, as is the method (if one exists) for configuring the default treatments for interrupts.

An interrupt is delivered to the handler (or default treatment) that is in effect for that interrupt at the time of delivery.

An exception propagated from a handler that is invoked by an interrupt has no effect.

[If the Ceiling_Locking policy (see D.3) is in effect, the interrupt handler executes with the active priority that is the ceiling priority of the corresponding protected object.]


#### Implementation Requirements

{AI12-0445-1} The implementation shall provide a mechanism to determine the minimum stack space that is necessary for each interrupt handler and to reserve that space for the execution of the handler. [This space should accommodate nested invocations of the handler where the system permits this.]

If the hardware or the underlying system holds pending interrupt occurrences, the implementation shall provide for later delivery of these occurrences to the program.

If the Ceiling_Locking policy is not in effect, the implementation shall provide means for the application to specify whether interrupts are to be blocked during protected actions.


#### Documentation Requirements

The implementation shall document the following items: 

Discussion: This information may be different for different forms of interrupt handlers. 

a)For each interrupt, which interrupts are blocked from delivery when a handler attached to that interrupt executes (either as a result of an interrupt delivery or of an ordinary call on a procedure of the corresponding protected object).

b)Any interrupts that cannot be blocked, and the effect of attaching handlers to such interrupts, if this is permitted.

c)Which run-time stack an interrupt handler uses when it executes as a result of an interrupt delivery; if this is configurable, what is the mechanism to do so; how to specify how much space to reserve on that stack.

d)Any implementation- or hardware-specific activity that happens before a user-defined interrupt handler gets control (e.g., reading device registers, acknowledging devices).

e)Any timing or other limitations imposed on the execution of interrupt handlers.

f)The state (blocked/unblocked) of the nonreserved interrupts when the program starts; if some interrupts are unblocked, what is the mechanism a program can use to protect itself before it can attach the corresponding handlers.

g)Whether the interrupted task is allowed to resume execution before the interrupt handler returns.

h)The treatment of interrupt occurrences that are generated while the interrupt is blocked; i.e., whether one or more occurrences are held for later delivery, or all are lost.

i)Whether predefined or implementation-defined exceptions are raised as a result of the occurrence of any interrupt, and the mapping between the machine interrupts (or traps) and the predefined exceptions.

j)On a multi-processor, the rules governing the delivery of an interrupt to a particular processor. 

Documentation Requirement: The treatment of interrupts.


#### Implementation Permissions

{AI95-00434-01} If the underlying system or hardware does not allow interrupts to be blocked, then no blocking is required [as part of the execution of subprograms of a protected object for which one of its subprograms is an interrupt handler].

In a multi-processor with more than one interrupt subsystem, it is implementation defined whether (and how) interrupt sources from separate subsystems share the same Interrupt_Id type (see C.3.2). In particular, the meaning of a blocked or pending interrupt may then be applicable to one processor only. 

Discussion: This issue is tightly related to the issue of scheduling on a multi-processor. In a sense, if a particular interrupt source is not available to all processors, the system is not truly homogeneous.

One way to approach this problem is to assign sub-ranges within Interrupt_Id to each interrupt subsystem, such that "similar" interrupt sources (e.g. a timer) in different subsystems get a distinct id. 

Implementations are allowed to impose timing or other limitations on the execution of interrupt handlers. 

Reason: These limitations are often necessary to ensure proper behavior of the implementation. 

{AI95-00434-01} {AI05-0299-1} Other forms of handlers are allowed to be supported, in which case the rules of this subclause should be adhered to.

The active priority of the execution of an interrupt handler is allowed to vary from one occurrence of the same interrupt to another.


#### Implementation Advice

{AI95-00434-01} If the Ceiling_Locking policy is not in effect, the implementation should provide means for the application to specify which interrupts are to be blocked during protected actions, if the underlying system allows for finer-grained control of interrupt blocking. 

Implementation Advice: If the Ceiling_Locking policy is not in effect and the target system allows for finer-grained control of interrupt blocking, a means for the application to specify which interrupts are to be blocked during protected actions should be provided.

NOTE 1   The default treatment for an interrupt can be to keep the interrupt pending or to deliver it to an implementation-defined handler. Examples of actions that an implementation-defined handler is allowed to perform include aborting the partition, ignoring (i.e., discarding occurrences of) the interrupt, or queuing one or more occurrences of the interrupt for possible later delivery when a user-defined handler is attached to that interrupt.

NOTE 2   It is a bounded error to call Task_Identification.Current_Task (see C.7.1) from an interrupt handler.

NOTE 3   The rule that an exception propagated from an interrupt handler has no effect is modeled after the rule about exceptions propagated out of task bodies.


## C.3.1  Protected Procedure Handlers

Paragraphs 1 through 6 were moved to Annex J, "Obsolescent Features". 


#### Static Semantics

{AI05-0229-1} For a parameterless protected procedure, the following language-defined representation aspects may be specified: 

Interrupt_Handler The type of aspect Interrupt_Handler is Boolean. If directly specified, the aspect_definition shall be a static expression. [This aspect is never inherited;] if not directly specified, the aspect is False.

Aspect Description for Interrupt_Handler: Protected procedure may be attached to interrupts.

Attach_Handler The aspect Attach_Handler is an [expression](./AA-4.4#S0132), which shall be of type Interrupts.Interrupt_Id. [This aspect is never inherited.]

Aspect Description for Attach_Handler: Protected procedure is attached to an interrupt.


#### Legality Rules

{AI95-00434-01} {AI05-0033-1} {AI05-0229-1} If either the Attach_Handler or Interrupt_Handler aspect are specified for a protected procedure, the corresponding [protected_type_declaration](./AA-9.4#S0249) or [single_protected_declaration](./AA-9.4#S0250) shall be a library-level declaration and shall not be declared within a generic body. In addition to the places where Legality Rules normally apply (see 12.3), this rule also applies in the private part of an instance of a generic unit.

Discussion: In the case of a [protected_type_declaration](./AA-9.4#S0249), an [object_declaration](./AA-3.3#S0032) of an object of that type need not be at library level.

{AI05-0033-1} {AI05-0229-1} We cannot allow these aspects in protected declarations in a generic body, because legality rules are not checked for instance bodies, and these should not be allowed if the instance is not at the library level. The protected types can be declared in the private part if this is desired. Note that while the 'Access to use the handler would provide the check in the case of Interrupt_Handler, there is no other check for Attach_Handler. Since these aspects are so similar, we want the rules to be the same. 

This paragraph was deleted.{AI95-00253-01} {AI95-00303-01} {AI05-0033-1} 


#### Dynamic Semantics

{AI05-0229-1} If the Interrupt_Handler aspect of a protected procedure is True, then the procedure may be attached dynamically, as a handler, to interrupts (see C.3.2). [Such procedures are allowed to be attached to multiple interrupts.]

{AI05-0229-1} The [expression](./AA-4.4#S0132) specified for the Attach_Handler aspect of a protected procedure P is evaluated as part of the creation of the protected object that contains P. The value of the [expression](./AA-4.4#S0132) identifies an interrupt. As part of the initialization of that object, P (the handler procedure) is attached to the identified interrupt. A check is made that the corresponding interrupt is not reserved. Program_Error is raised if the check fails, and the existing treatment for the interrupt is not affected.

{AI95-00434-01} {AI05-0229-1} If the Ceiling_Locking policy (see D.3) is in effect, then upon the initialization of a protected object that contains a protected procedure for which either the Attach_Handler aspect is specified or the  Interrupt_Handler aspect is True, a check is made that the initial ceiling priority of the object is in the range of System.Interrupt_Priority. If the check fails, Program_Error is raised.

{8652/0068} {AI95-00121-01} {AI05-0229-1} When a protected object is finalized, for any of its procedures that are attached to interrupts, the handler is detached. If the handler was attached by a procedure in the Interrupts package or if no user handler was previously attached to the interrupt, the default treatment is restored. If the Attach_Handler aspect was specified and the most recently attached handler for the same interrupt is the same as the one that was attached at the time the protected object was initialized, the previous handler is restored. 

Discussion: {8652/0068} {AI95-00121-01} {AI95-00303-01} {AI05-0229-1} If all protected objects for interrupt handlers are declared at the library level, the finalization discussed above occurs only as part of the finalization of all library-level packages in a partition. However, objects of a protected type containing procedures with an Attach_Handler aspect specified need not be at the library level. Thus, an implementation needs to be able to restore handlers during the execution of the program. (An object with an Interrupt_Handler aspect also need not be at the library level, but such a handler cannot be attached to an interrupt using the Interrupts package.) 

When a handler is attached to an interrupt, the interrupt is blocked [(subject to the Implementation Permission in C.3)] during the execution of every protected action on the protected object containing the handler.

{AI12-0252-1} If restriction No_Dynamic_Attachment is in effect, then a check is made that the interrupt identified by an Attach_Handler aspect does not appear in any previously elaborated Attach_Handler aspect; Program_Error is raised if this check fails.

Reason: When No_Dynamic_Attachment is in effect, it is not possible to call any previous handler. Therefore, the only possibility is to replace the original handler, rendering it ineffective. Having an unused handler in a program is likely a bug, so we require an exception to be raised. 


#### Erroneous Execution

If the Ceiling_Locking policy (see D.3) is in effect and an interrupt is delivered to a handler, and the interrupt hardware priority is higher than the ceiling priority of the corresponding protected object, the execution of the program is erroneous.

{8652/0068} {AI95-00121-01} {AI05-0229-1} If the handlers for a given interrupt attached via aspect Attach_Handler are not attached and detached in a stack-like (LIFO) order, program execution is erroneous. In particular, when a protected object is finalized, the execution is erroneous if any of the procedures of the protected object are attached to interrupts via aspect Attach_Handler and the most recently attached handler for the same interrupt is not the same as the one that was attached at the time the protected object was initialized. 

Discussion: {8652/0068} {AI95-00121-01} {AI05-0229-1} This simplifies implementation of the Attach_Handler aspect by not requiring a check that the current handler is the same as the one attached by the initialization of a protected object. 


#### Metrics

The following metric shall be documented by the implementation: 

{AI95-00434-01} The worst-case overhead for an interrupt handler that is a parameterless protected procedure, in clock cycles. This is the execution time not directly attributable to the handler procedure or the interrupted execution. It is estimated as C  (A+B), where A is how long it takes to complete a given sequence of instructions without any interrupt, B is how long it takes to complete a normal call to a given protected procedure, and C is how long it takes to complete the same sequence of instructions when it is interrupted by one execution of the same procedure called via an interrupt. 

Implementation Note: The instruction sequence and interrupt handler used to measure interrupt handling overhead should be chosen so as to maximize the execution time cost due to cache misses. For example, if the processor has cache memory and the activity of an interrupt handler could invalidate the contents of cache memory, the handler should be written such that it invalidates all of the cache memory. 

Documentation Requirement: The metrics for interrupt handlers.


#### Implementation Permissions

{AI05-0229-1} When the aspects Attach_Handler or Interrupt_Handler are specified for a protected procedure, the implementation is allowed to impose implementation-defined restrictions on the corresponding [protected_type_declaration](./AA-9.4#S0249) and [protected_body](./AA-9.4#S0254). 

Ramification: The restrictions may be on the constructs that are allowed within them, and on ordinary calls (i.e. not via interrupts) on protected operations in these protected objects. 

Implementation defined: Any restrictions on a protected procedure or its containing type when an aspect Attach_handler or Interrupt_Handler is specified.

An implementation may use a different mechanism for invoking a protected procedure in response to a hardware interrupt than is used for a call to that protected procedure from a task. 

Discussion: This is despite the fact that the priority of an interrupt handler (see D.1) is modeled after a hardware task calling the handler. 

{AI05-0229-1} Notwithstanding what this subclause says elsewhere, the Attach_Handler and Interrupt_Handler aspects are allowed to be used for other, implementation defined, forms of interrupt handlers. 

Ramification: {AI05-0229-1} For example, if an implementation wishes to allow interrupt handlers to have parameters, it is allowed to do so via these aspects; it need not invent implementation-defined aspects for the purpose. 

Implementation defined: Any other forms of interrupt handler supported by the Attach_Handler and Interrupt_Handler aspects.


#### Implementation Advice

Whenever possible, the implementation should allow interrupt handlers to be called directly by the hardware. 

Implementation Advice: Interrupt handlers should be called directly by the hardware.

Whenever practical, the implementation should detect violations of any implementation-defined restrictions before run time. 

Implementation Advice: Violations of any implementation-defined restrictions on interrupt handlers should be detected before run time.

NOTE 1   {AI05-0229-1} {AI12-0440-1} The Attach_Handler aspect can provide static attachment of handlers to interrupts if the implementation supports preelaboration of protected objects. (See C.4.)

NOTE 2   {AI95-00434-01} {AI12-0442-1} For a protected object that has a (protected) procedure attached to an interrupt, the correct ceiling priority is at least as high as the highest processor priority at which that interrupt will ever be delivered.

NOTE 3   Protected procedures can also be attached dynamically to interrupts via operations declared in the predefined package Interrupts.

NOTE 4   An example of a possible implementation-defined restriction is disallowing the use of the standard storage pools within the body of a protected procedure that is an interrupt handler.


#### Incompatibilities With Ada 95

{AI95-00253-01} Amendment Correction: Corrected the wording so that the rules for the use of Attach_Handler and Interrupt_Handler are identical. This means that uses of pragma Interrupt_Handler outside of the target protected type or single protected object are now illegal. 


#### Wording Changes from Ada 95

{8652/0068} {AI95-00121-01} Corrigendum: Clarified the meaning of "the previous handler" when finalizing protected objects containing interrupt handlers.

{AI95-00303-01} Dropped the requirement that an object of a type containing an Interrupt_Handler pragma must be declared at the library level. This was a generic contract model violation. This change is not an extension, as an attempt to attach such a handler with a routine in package Interrupts will fail an accessibility check anyway. Moreover, implementations can retain the rule as an implementation-defined restriction on the use of the type, as permitted by the Implementation Permissions above. 


#### Extensions to Ada 2005

{AI05-0229-1} Aspects Interrupt_Handler and Attach_Handler are new; [pragma](./AA-2.8#S0019)s Interrupt_Handler and Attach_Handler are now obsolescent. 


#### Wording Changes from Ada 2005

{AI05-0033-1} Correction: Added missing generic contract wording for the aspects Attach_Handler and Interrupt_Handler. 


#### Inconsistencies With Ada 2012

{AI12-0252-1} Correction: Program_Error might be raised if two handlers are attached when restriction No_Dynamic_Attachment is in effect (including as part of the Ravenscar profile). Original Ada 2012 would have just ignored the first handler. This is almost certainly a bug rather than an intended result. 


## C.3.2  The Package Interrupts


#### Static Semantics

The following language-defined packages exist: 

```ada
{AI05-0167-1} {AI12-0241-1} {AI12-0302-1} with System;
with System.Multiprocessors;
package Ada.Interrupts
   with Nonblocking, Global =&gt in out synchronized is
   type Interrupt_Id is implementation-defined;
   type Parameterless_Handler is
      access protected procedure
      with Nonblocking =&gt False;

```

```ada
This paragraph was deleted.

```

```ada
   function Is_Reserved (Interrupt : Interrupt_Id)
      return Boolean;

```

```ada
   function Is_Attached (Interrupt : Interrupt_Id)
      return Boolean;

```

```ada
   function Current_Handler (Interrupt : Interrupt_Id)
      return Parameterless_Handler;

```

```ada
   procedure Attach_Handler
      (New_Handler : in Parameterless_Handler;
       Interrupt   : in Interrupt_Id);

```

```ada
   procedure Exchange_Handler
      (Old_Handler : out Parameterless_Handler;
       New_Handler : in Parameterless_Handler;
       Interrupt   : in Interrupt_Id);

```

```ada
   procedure Detach_Handler
      (Interrupt : in Interrupt_Id);

```

```ada
   function Reference (Interrupt : Interrupt_Id)
      return System.Address;

```

```ada
{AI05-0167-1}    function Get_CPU (Interrupt : Interrupt_Id)
      return System.Multiprocessors.CPU_Range;

```

```ada
private
   ... -- not specified by the language
end Ada.Interrupts;

```

```ada
{AI12-0302-1} package Ada.Interrupts.Names 
   with Nonblocking, Global =&gt null is
   implementation-defined : constant Interrupt_Id :=
     implementation-defined;
      . . .
   implementation-defined : constant Interrupt_Id :=
     implementation-defined;
end Ada.Interrupts.Names;

```


#### Dynamic Semantics

The Interrupt_Id type is an implementation-defined discrete type used to identify interrupts.

The Is_Reserved function returns True if and only if the specified interrupt is reserved.

The Is_Attached function returns True if and only if a user-specified interrupt handler is attached to the interrupt.

{8652/0069} {AI95-00166-01} The Current_Handler function returns a value that represents the attached handler of the interrupt. If no user-defined handler is attached to the interrupt, Current_Handler returns null.

{AI05-0229-1} The Attach_Handler procedure attaches the specified handler to the interrupt, overriding any existing treatment (including a user handler) in effect for that interrupt. If New_Handler is null, the default treatment is restored. If New_Handler designates a protected procedure for which the aspect Interrupt_Handler is False, Program_Error is raised. In this case, the operation does not modify the existing interrupt treatment.

{8652/0069} {AI95-00166-01} The Exchange_Handler procedure operates in the same manner as Attach_Handler with the addition that the value returned in Old_Handler designates the previous treatment for the specified interrupt. If the previous treatment is not a user-defined handler, null is returned. 

Ramification: Calling Attach_Handler or Exchange_Handler with this value for New_Handler restores the previous handler.

{8652/0069} {AI95-00166-01} If the application uses only parameterless procedures as handlers (other types of handlers may be provided by the implementation, but are not required by the standard), then if Old_Handler is not null, it may be called to execute the previous handler. This provides a way to cascade application interrupt handlers. However, the default handler cannot be cascaded this way (Old_Handler must be null for the default handler). 

The Detach_Handler procedure restores the default treatment for the specified interrupt.

For all operations defined in this package that take a parameter of type Interrupt_Id, with the exception of Is_Reserved and Reference, a check is made that the specified interrupt is not reserved. Program_Error is raised if this check fails.

{AI05-0229-1} If, by using the Attach_Handler, Detach_Handler, or Exchange_Handler procedures, an attempt is made to detach a handler that was attached statically (using the aspect Attach_Handler), the handler is not detached and Program_Error is raised.

{AI95-00434-01} The Reference function returns a value of type System.Address that can be used to attach a task entry via an address clause (see J.7.1) to the interrupt specified by Interrupt. This function raises Program_Error if attaching task entries to interrupts (or to this particular interrupt) is not supported.

{AI05-0153-3} The function Get_CPU returns the processor on which the handler for Interrupt is executed. If the handler can execute on more than one processor the value System.Multiprocessors.Not_A_Specific_CPU is returned.


#### Implementation Requirements

At no time during attachment or exchange of handlers shall the current handler of the corresponding interrupt be undefined.


#### Documentation Requirements

{AI95-00434-01} {AI05-0229-1} {AI12-0320-1} {AI12-0444-1} The implementation shall document, when the Ceiling_Locking policy (see D.3) is in effect, the default ceiling priority assigned to a protected object that contains a protected procedure that specifies either the Attach_Handler or Interrupt_Handler aspects, but does not specify the Interrupt_Priority aspect. [This default can be different for different interrupts.] 

Documentation Requirement: If the Ceiling_Locking policy is in effect, the default ceiling priority for a protected object that specifies an interrupt handler aspect.


#### Implementation Advice

If implementation-defined forms of interrupt handler procedures are supported, such as protected procedures with parameters, then for each such form of a handler, a type analogous to Parameterless_Handler should be specified in a child package of Interrupts, with the same operations as in the predefined package Interrupts.

Implementation Advice: If implementation-defined forms of interrupt handler procedures are supported, then for each such form of a handler, a type analogous to Parameterless_Handler should be specified in a child package of Interrupts, with the same operations as in the predefined package Interrupts.

NOTE 1   The package Interrupts.Names contains implementation-defined names (and constant values) for the interrupts that are supported by the implementation.


#### Examples

Example of interrupt handlers: 

```ada
{AI05-0229-1} {AI12-0178-1} Device_Priority : constant
  array (Ada.Interrupts.Interrupt_Id range 1..5) of
    System.Interrupt_Priority := ( ... );
protected type Device_Interface
  (Int_Id : Ada.Interrupts.Interrupt_Id) 
     with Interrupt_Priority =&gt Device_Priority(Int_Id) is
  procedure Handler
     with Attach_Handler =&gt Int_Id;
  ...
end Device_Interface;
  ...
Device_1_Driver : Device_Interface(1);
  ...
Device_5_Driver : Device_Interface(5);
  ...

```


#### Wording Changes from Ada 95

{8652/0069} {AI95-00166-01} Corrigendum: Clarified that the value returned by Current_Handler and Exchange_Handler for the default treatment is null. 


#### Incompatibilities With Ada 2005

{AI05-0167-1} Function Get_CPU is added to Interrupts. If Interrupts is referenced in a [use_clause](./AA-8.4#S0235), and an entity E with a [defining_identifier](./AA-3.1#S0022) of Get_CPU is defined in a package that is also referenced in a [use_clause](./AA-8.4#S0235), the entity E may no longer be use-visible, resulting in errors. This should be rare and is easily fixed if it does occur. 
