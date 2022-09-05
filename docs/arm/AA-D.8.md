---
sidebar_position:  159
---

# D.8  Monotonic Time

{AI05-0299-1} [This subclause specifies a high-resolution, monotonic clock package.] 


#### Static Semantics

The following language-defined library package exists: 

```ada
{AI12-0241-1} {AI12-0302-1} package Ada.Real_Time
  with Nonblocking, Global =&gt in out synchronized is

```

```ada
  type Time is private;
  Time_First : constant Time;
  Time_Last : constant Time;
  Time_Unit : constant := implementation-defined-real-number;

```

```ada
  type Time_Span is private;
  Time_Span_First : constant Time_Span;
  Time_Span_Last : constant Time_Span;
  Time_Span_Zero : constant Time_Span;
  Time_Span_Unit : constant Time_Span;

```

```ada
  Tick : constant Time_Span;
  function Clock return Time;

```

```ada
  function "+" (Left : Time; Right : Time_Span) return Time;
  function "+" (Left : Time_Span; Right : Time) return Time;
  function "-" (Left : Time; Right : Time_Span) return Time;
  function "-" (Left : Time; Right : Time) return Time_Span;

```

```ada
  function "&lt" (Left, Right : Time) return Boolean;
  function "&lt="(Left, Right : Time) return Boolean;
  function "&gt" (Left, Right : Time) return Boolean;
  function "&gt="(Left, Right : Time) return Boolean;

```

```ada
  function "+" (Left, Right : Time_Span) return Time_Span;
  function "-" (Left, Right : Time_Span) return Time_Span;
  function "-" (Right : Time_Span) return Time_Span;
  function "*" (Left : Time_Span; Right : Integer) return Time_Span;
  function "*" (Left : Integer; Right : Time_Span) return Time_Span;
  function "/" (Left, Right : Time_Span) return Integer;
  function "/" (Left : Time_Span; Right : Integer) return Time_Span;

```

```ada
  function "abs"(Right : Time_Span) return Time_Span;

```

```ada
This paragraph was deleted.

```

```ada
  function "&lt" (Left, Right : Time_Span) return Boolean;
  function "&lt="(Left, Right : Time_Span) return Boolean;
  function "&gt" (Left, Right : Time_Span) return Boolean;
  function "&gt="(Left, Right : Time_Span) return Boolean;

```

```ada
  function To_Duration (TS : Time_Span) return Duration;
  function To_Time_Span (D : Duration) return Time_Span;

```

```ada
{AI95-00386-01}   function Nanoseconds  (NS : Integer) return Time_Span;
  function Microseconds (US : Integer) return Time_Span;
  function Milliseconds (MS : Integer) return Time_Span;
  function Seconds      (S  : Integer) return Time_Span;
  function Minutes      (M  : Integer) return Time_Span;

```

```ada
  type Seconds_Count is range implementation-defined;

```

```ada
  procedure Split(T : in Time; SC : out Seconds_Count; TS : out Time_Span);
  function Time_Of(SC : Seconds_Count; TS : Time_Span) return Time;

```

```ada
private
   ... -- not specified by the language
end Ada.Real_Time;

```

This paragraph was deleted.

In this Annex, real time is defined to be the physical time as observed in the external environment. The type Time is a time type as defined by 9.6; [values of this type may be used in a [delay_until_statement](./AA-9.6#S0267).] Values of this type represent segments of an ideal time line. The set of values of the type Time corresponds one-to-one with an implementation-defined range of mathematical integers. 

Discussion: Informally, real time is defined to be the International Atomic Time (TAI) which is monotonic and nondecreasing. We use it here for the purpose of discussing rate of change and monotonic behavior only. It does not imply anything about the absolute value of Real_Time.Clock, or about Real_Time.Time being synchronized with TAI. It is also used for real time in the metrics, for comparison purposes. 

Implementation Note: The specification of TAI as "real time" does not preclude the use of a simulated TAI clock for simulated execution environments. 

The Time value I represents the half-open real time interval that starts with E+I*Time_Unit and is limited by E+(I+1)*Time_Unit, where Time_Unit is an implementation-defined real number and E is an unspecified origin point, the epoch, that is the same for all values of the type Time. It is not specified by the language whether the time values are synchronized with any standard time reference. [For example, E can correspond to the time of system initialization or it can correspond to the epoch of some time standard.] 

Discussion: E itself does not have to be a proper time value.

This half-open interval I consists of all real numbers R such that E+I*Time_Unit &lt= R &lt E+(I+1)*Time_Unit. 

Values of the type Time_Span represent length of real time duration. The set of values of this type corresponds one-to-one with an implementation-defined range of mathematical integers. The Time_Span value corresponding to the integer I represents the real-time duration I*Time_Unit. 

Reason: The purpose of this type is similar to Standard.Duration; the idea is to have a type with a higher resolution. 

Discussion: We looked at many possible names for this type: Real_Time.Duration, Fine_Duration, Interval, Time_Interval_Length, Time_Measure, and more. Each of these names had some problems, and we've finally settled for Time_Span. 

Time_First and Time_Last are the smallest and largest values of the Time type, respectively. Similarly, Time_Span_First and Time_Span_Last are the smallest and largest values of the Time_Span type, respectively.

A value of type Seconds_Count represents an elapsed time, measured in seconds, since the epoch.


#### Dynamic Semantics

Time_Unit is the smallest amount of real time representable by the Time type; it is expressed in seconds. Time_Span_Unit is the difference between two successive values of the Time type. It is also the smallest positive value of type Time_Span. Time_Unit and Time_Span_Unit represent the same real time duration. A clock tick is a real time interval during which the clock value (as observed by calling the Clock function) remains constant. Tick is the average length of such intervals.

{AI95-00432-01} The function To_Duration converts the value TS to a value of type Duration. Similarly, the function To_Time_Span converts the value D to a value of type Time_Span. For To_Duration, the result is rounded to the nearest value of type Duration (away from zero if exactly halfway between two values). If the result is outside the range of Duration, Constraint_Error is raised. For To_Time_Span, the value of D is first rounded to the nearest integral multiple of Time_Unit, away from zero if exactly halfway between two multiples. If the rounded value is outside the range of Time_Span, Constraint_Error is raised. Otherwise, the value is converted to the type Time_Span.

To_Duration(Time_Span_Zero) returns 0.0, and To_Time_Span(0.0) returns Time_Span_Zero.

{AI95-00386-01} {AI95-00432-01} The functions Nanoseconds, Microseconds, Milliseconds, Seconds, and Minutes convert the input parameter to a value of the type Time_Span. NS, US, MS, S, and M are interpreted as a number of nanoseconds, microseconds, milliseconds, seconds, and minutes respectively. The input parameter is first converted to seconds and rounded to the nearest integral multiple of Time_Unit, away from zero if exactly halfway between two multiples. If the rounded value is outside the range of Time_Span, Constraint_Error is raised. Otherwise, the rounded value is converted to the type Time_Span. 

This paragraph was deleted.{AI95-00432-01} 

The effects of the operators on Time and Time_Span are as for the operators defined for integer types. 

Implementation Note: Though time values are modeled by integers, the types Time and Time_Span need not be implemented as integers. 

The function Clock returns the amount of time since the epoch.

The effects of the Split and Time_Of operations are defined as follows, treating values of type Time, Time_Span, and Seconds_Count as mathematical integers. The effect of Split(T,SC,TS) is to set SC and TS to values such that T*Time_Unit = SC*1.0 + TS*Time_Unit, and 0.0 &lt= TS*Time_Unit &lt 1.0. The value returned by Time_Of(SC,TS) is the value T such that T*Time_Unit = SC*1.0 + TS*Time_Unit. 


#### Implementation Requirements

The range of Time values shall be sufficient to uniquely represent the range of real times from program start-up to 50 years later. Tick shall be no greater than 1 millisecond. Time_Unit shall be less than or equal to 20 microseconds. 

Implementation Note: The required range and accuracy of Time are such that 32-bits worth of seconds and 32-bits worth of ticks in a second could be used as the representation. 

Time_Span_First shall be no greater than 3600 seconds, and Time_Span_Last shall be no less than 3600 seconds. 

Reason: This is equivalent to � one hour and there is still room for a two-microsecond resolution. 

A clock jump is the difference between two successive distinct values of the clock (as observed by calling the Clock function). There shall be no backward clock jumps.


#### Documentation Requirements

The implementation shall document the values of Time_First, Time_Last, Time_Span_First, Time_Span_Last, Time_Span_Unit, and Tick. 

Documentation Requirement: The values of Time_First, Time_Last, Time_Span_First, Time_Span_Last, Time_Span_Unit, and Tick for package Real_Time.

The implementation shall document the properties of the underlying time base used for the clock and for type Time, such as the range of values supported and any relevant aspects of the underlying hardware or operating system facilities used. 

Documentation Requirement: The properties of the underlying time base used in package Real_Time.

Discussion: If there is an underlying operating system, this might include information about which system call is used to implement the clock. Otherwise, it might include information about which hardware clock is used. 

The implementation shall document whether or not there is any synchronization with external time references, and if such synchronization exists, the sources of synchronization information, the frequency of synchronization, and the synchronization method applied. 

Documentation Requirement: Any synchronization of package Real_Time with external time references.

{AI05-0299-1} {AI12-0439-1} The implementation shall document any aspects of the external environment that can interfere with the clock behavior as defined in this subclause. 

Documentation Requirement: Any aspects of the external environment that can interfere with package Real_Time.

Discussion: For example, the implementation is allowed to rely on the time services of an underlying operating system, and this operating system clock can implement time zones or allow the clock to be reset by an operator. This dependence has to be documented. 


#### Metrics

{AI05-0299-1} For the purpose of the metrics defined in this subclause, real time is defined to be the International Atomic Time (TAI).

The implementation shall document the following metrics: 

An upper bound on the real-time duration of a clock tick. This is a value D such that if t1 and t2 are any real times such that t1 &lt t2 and Clockt1 = Clockt2 then t2  t1 &lt= D.

An upper bound on the size of a clock jump.

An upper bound on the drift rate of Clock with respect to real time. This is a real number D such that 

E*(1D) &lt= (Clockt+E  Clockt) &lt= E*(1+D)
        provided that: Clockt + E*(1+D) &lt= Time_Last.

where Clockt is the value of Clock at time t, and E is a real time duration not less than 24 hours. The value of E used for this metric shall be reported. 

Reason: This metric is intended to provide a measurement of the long term (cumulative) deviation; therefore, 24 hours is the lower bound on the measurement period. On some implementations, this is also the maximum period, since the language does not require that the range of the type Duration be more than 24 hours. On those implementations that support longer-range Duration, longer measurements should be performed. 

An upper bound on the execution time of a call to the Clock function, in processor clock cycles.

Upper bounds on the execution times of the operators of the types Time and Time_Span, in processor clock cycles. 

Implementation Note: A fast implementation of the Clock function involves repeated reading until you get the same value twice. It is highly improbable that more than three reads will be necessary. Arithmetic on time values should not be significantly slower than 64-bit arithmetic in the underlying machine instruction set. 

Documentation Requirement: The metrics for package Real_Time.


#### Implementation Permissions

{AI12-0444-1} Implementations targeted to machines with word size smaller than 32 bits may omit support for the full range and granularity of the Time and Time_Span types. 

Discussion: These requirements are based on machines with a word size of 32 bits.

Since the range and granularity are implementation defined, the supported values need to be documented. 


#### Implementation Advice

When appropriate, implementations should provide configuration mechanisms to change the value of Tick. 

Implementation Advice: When appropriate, mechanisms to change the value of Tick should be provided.

Reason: This is often needed when the compilation system was originally targeted to a particular processor with a particular interval timer, but the customer uses the same processor with a different interval timer. 

Discussion: Tick is a deferred constant and not a named number specifically for this purpose. 

Implementation Note: This can be achieved either by pre-run-time configuration tools, or by having Tick be initialized (in the package private part) by a function call residing in a board specific module. 

It is recommended that Calendar.Clock and Real_Time.Clock be implemented as transformations of the same time base. 

Implementation Advice: Calendar.Clock and Real_Time.Clock should be transformations of the same time base.

It is recommended that the "best" time base which exists in the underlying system be available to the application through Clock. "Best" may mean highest accuracy or largest range. 

Implementation Advice: The "best" time base which exists in the underlying system should be available to the application through Real_Time.Clock.

NOTE 1   {AI05-0299-1} The rules in this subclause do not imply that the implementation can protect the user from operator or installation errors that can result in the clock being set incorrectly.

NOTE 2   Time_Unit is the granularity of the Time type. In contrast, Tick represents the granularity of Real_Time.Clock. There is no requirement that these be the same.


#### Incompatibilities With Ada 95

{AI95-00386-01} {AI05-0005-1} Functions Seconds and Minutes are added to Real_Time. If Real_Time is referenced in a [use_clause](./AA-8.4#S0235), and an entity E with a [defining_identifier](./AA-3.1#S0022) of Seconds or Minutes is defined in a package that is also referenced in a [use_clause](./AA-8.4#S0235), the entity E may no longer be use-visible, resulting in errors. This should be rare and is easily fixed if it does occur. 


#### Wording Changes from Ada 95

{AI95-00432-01} Added wording explaining how and when many of these functions can raise Constraint_Error. While there always was an intent to raise Constraint_Error if the values did not fit, there never was any wording to that effect, and since Time_Span was a private type, the normal numeric type rules do not apply to it. 
