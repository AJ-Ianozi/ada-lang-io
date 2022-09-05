---
sidebar_position:  77
---

# 9.6  Delay Statements, Duration, and Time

[ A [delay_statement](./AA-9.6#S0266) is used to block further execution until a specified expiration time is reached. The expiration time can be specified either as a particular point in time (in a [delay_until_statement](./AA-9.6#S0267)), or in seconds from the current time (in a [delay_relative_statement](./AA-9.6#S0268)). The language-defined package Calendar provides definitions for a type Time and associated operations, including a function Clock that returns the current time. ]


#### Syntax

delay_statement<a id="S0266"></a> ::= [delay_until_statement](./AA-9.6#S0267) | [delay_relative_statement](./AA-9.6#S0268)

delay_until_statement<a id="S0267"></a> ::= delay until delay_[expression](./AA-4.4#S0132);

delay_relative_statement<a id="S0268"></a> ::= delay delay_[expression](./AA-4.4#S0132);


#### Name Resolution Rules

The expected type for the delay_[expression](./AA-4.4#S0132) in a [delay_relative_statement](./AA-9.6#S0268) is the predefined type Duration. The delay_[expression](./AA-4.4#S0132) in a [delay_until_statement](./AA-9.6#S0267) is expected to be of any nonlimited type.


#### Legality Rules

{AI05-0092-1} There can be multiple time bases, each with a corresponding clock, and a corresponding time type. The type of the delay_[expression](./AA-4.4#S0132) in a [delay_until_statement](./AA-9.6#S0267) shall be a time type - either the type Time defined in the language-defined package Calendar (see below), the type Time in the package Real_Time (see D.8), or some other implementation-defined time type. 

Implementation defined: Any implementation-defined time types.


#### Static Semantics

[There is a predefined fixed point type named Duration, declared in the visible part of package Standard;] a value of type Duration is used to represent the length of an interval of time, expressed in seconds. [The type Duration is not specific to a particular time base, but can be used with any time base.]

{AI05-0092-1} A value of the type Time in package Calendar, or of some other time type, represents a time as reported by a corresponding clock.

The following language-defined library package exists: 

```ada
{AI12-0241-1} {AI12-0302-1} 
package Ada.Calendar 
  with Nonblocking, Global =&gt in out synchronized is
  type Time is private;

```

```ada
{AI95-00351-01}   subtype Year_Number  is Integer range 1901 .. 2399;
  subtype Month_Number is Integer range 1 .. 12;
  subtype Day_Number   is Integer range 1 .. 31;
  subtype Day_Duration is Duration range 0.0 .. 86_400.0;

```

Reason: {AI95-00351-01} A range of 500 years was chosen, as that only requires one extra bit for the year as compared to Ada 95. This was done to minimize disruptions with existing implementations. (One implementor reports that their time values represent nanoseconds, and this year range requires 63.77 bits to represent.) 

```ada
  function Clock return Time;

```

```ada
  function Year   (Date : Time) return Year_Number;
  function Month  (Date : Time) return Month_Number;
  function Day    (Date : Time) return Day_Number;
  function Seconds(Date : Time) return Day_Duration;

```

```ada
  procedure Split (Date  : in Time;
                   Year    : out Year_Number;
                   Month   : out Month_Number;
                   Day     : out Day_Number;
                   Seconds : out Day_Duration);

```

```ada
  function Time_Of(Year  : Year_Number;
                   Month   : Month_Number;
                   Day     : Day_Number;
                   Seconds : Day_Duration := 0.0)
    return Time;

```

```ada
  function "+" (Left : Time;   Right : Duration) return Time;
  function "+" (Left : Duration; Right : Time) return Time;
  function "-" (Left : Time;   Right : Duration) return Time;
  function "-" (Left : Time;   Right : Time) return Duration;

```

```ada
  function "&lt" (Left, Right : Time) return Boolean;
  function "&lt="(Left, Right : Time) return Boolean;
  function "&gt" (Left, Right : Time) return Boolean;
  function "&gt="(Left, Right : Time) return Boolean;

```

```ada
  Time_Error : exception;

```

```ada
private
   ... -- not specified by the language
end Ada.Calendar;

```


#### Dynamic Semantics

For the execution of a [delay_statement](./AA-9.6#S0266), the delay_[expression](./AA-4.4#S0132) is first evaluated. For a [delay_until_statement](./AA-9.6#S0267), the expiration time for the delay is the value of the delay_[expression](./AA-4.4#S0132), in the time base associated with the type of the [expression](./AA-4.4#S0132). For a [delay_relative_statement](./AA-9.6#S0268), the expiration time is defined as the current time, in the time base associated with relative delays, plus the value of the delay_[expression](./AA-4.4#S0132) converted to the type Duration, and then rounded up to the next clock tick. The time base associated with relative delays is as defined in D.9, "Delay Accuracy" or is implementation defined. 

Implementation defined: The time base associated with relative delays.

Ramification: Rounding up to the next clock tick means that the reading of the delay-relative clock when the delay expires should be no less than the current reading of the delay-relative clock plus the specified duration. 

The task executing a [delay_statement](./AA-9.6#S0266) is blocked until the expiration time is reached, at which point it becomes ready again. If the expiration time has already passed, the task is not blocked. 

Discussion: For a [delay_relative_statement](./AA-9.6#S0268), this case corresponds to when the value of the delay_[expression](./AA-4.4#S0132) is zero or negative.

Even though the task is not blocked, it might be put back on the end of its ready queue. See D.2, "Priority Scheduling". 

{AI05-0092-1} If an attempt is made to cancel the [delay_statement](./AA-9.6#S0266) [(as part of an [asynchronous_select](./AA-9.7#S0280) or abort - see 9.7.4 and 9.8)], the statement is cancelled if the expiration time has not yet passed, thereby completing the [delay_statement](./AA-9.6#S0266). 

Reason: This is worded this way so that in an [asynchronous_select](./AA-9.7#S0280) where the [triggering_statement](./AA-9.7#S0282) is a [delay_statement](./AA-9.6#S0266), an attempt to cancel the delay when the [abortable_part](./AA-9.7#S0283) completes is ignored if the expiration time has already passed, in which case the optional statements of the [triggering_alternative](./AA-9.7#S0281) are executed. 

The time base associated with the type Time of package Calendar is implementation defined. The function Clock of package Calendar returns a value representing the current time for this time base. [The implementation-defined value of the named number System.Tick (see 13.7) is an approximation of the length of the real-time interval during which the value of Calendar.Clock remains constant.] 

Implementation defined: The time base of the type Calendar.Time.

{AI95-00351-01} The functions Year, Month, Day, and Seconds return the corresponding values for a given value of the type Time, as appropriate to an implementation-defined time zone; the procedure Split returns all four corresponding values. Conversely, the function Time_Of combines a year number, a month number, a day number, and a duration, into a value of type Time. The operators "+" and "" for addition and subtraction of times and durations, and the relational operators for times, have the conventional meaning. 

Implementation defined: The time zone used for package Calendar operations.

Ramification: {AI05-0119-1} The behavior of these values and subprograms if the time zone changes is also implementation-defined. In particular, the changes associated with summer time adjustments (like Daylight Savings Time in the United States) should be treated as a change in the implementation-defined time zone. The language does not specify whether the time zone information is stored in values of type Time; therefore the results of binary operators are unspecified when the operands are the two values with different effective time zones. In particular, the results of "-" may differ from the "real" result by the difference in the time zone adjustment. Similarly, the result of UTC_Time_Offset (see 9.6.1) may or may not reflect a time zone adjustment. 

If Time_Of is called with a seconds value of 86_400.0, the value returned is equal to the value of Time_Of for the next day with a seconds value of 0.0. The value returned by the function Seconds or through the Seconds parameter of the procedure Split is always less than 86_400.0.

{8652/0030} {AI95-00113-01} The exception Time_Error is raised by the function Time_Of if the actual parameters do not form a proper date. This exception is also raised by the operators "+" and "" if the result is not representable in the type Time or Duration, as appropriate. This exception is also raised by the functions Year, Month, Day, and Seconds and the procedure Split if the year number of the given date is outside of the range of the subtype Year_Number. 

To be honest: {8652/0106} {AI95-00160-01} By "proper date" above we mean that the given year has a month with the given day. For example, February 29th is a proper date only for a leap year. We do not mean to include the Seconds in this notion; in particular, we do not mean to require implementations to check for the "missing hour" that occurs when Daylight Savings Time starts in the spring. 

Reason: {8652/0030} {AI95-00113-01} {AI95-00351-01} We allow Year and Split to raise Time_Error because the arithmetic operators are allowed (but not required) to produce times that are outside the range of years from 1901 to 2399. This is similar to the way integer operators may return values outside the base range of their type so long as the value is mathematically correct. We allow the functions Month, Day and Seconds to raise Time_Error so that they can be implemented in terms of Split. 


#### Implementation Requirements

The implementation of the type Duration shall allow representation of time intervals (both positive and negative) up to at least 86400 seconds (one day); Duration'Small shall not be greater than twenty milliseconds. The implementation of the type Time shall allow representation of all dates with year numbers in the range of Year_Number[; it may allow representation of other dates as well (both earlier and later).] 


#### Implementation Permissions

{AI05-0092-1} An implementation may define additional time types.

An implementation may raise Time_Error if the value of a delay_[expression](./AA-4.4#S0132) in a [delay_until_statement](./AA-9.6#S0267) of a [select_statement](./AA-9.7#S0269) represents a time more than 90 days past the current time. The actual limit, if any, is implementation-defined. 

Implementation defined: Any limit on [delay_until_statement](./AA-9.6#S0267)s of [select_statement](./AA-9.7#S0269)s.

Implementation Note: This allows an implementation to implement [select_statement](./AA-9.7#S0269) timeouts using a representation that does not support the full range of a time type. In particular 90 days of seconds can be represented in 23 bits, allowing a signed 24-bit representation for the seconds part of a timeout. There is no similar restriction allowed for stand-alone [delay_until_statement](./AA-9.6#S0267)s, as these can be implemented internally using a loop if necessary to accommodate a long delay. 


#### Implementation Advice

Whenever possible in an implementation, the value of Duration'Small should be no greater than 100 microseconds. 

Implementation Note: This can be satisfied using a 32-bit 2's complement representation with a small of 2.0**(14) - that is, 61 microseconds - and a range of � 2.0**17 - that is, 131_072.0. 

Implementation Advice: The value of Duration'Small should be no greater than 100 microseconds.

{AI12-0444-1} The time base for [delay_relative_statement](./AA-9.6#S0268)s should be monotonic; it can be different than the time base as used for Calendar.Clock. 

Implementation Advice: The time base for [delay_relative_statement](./AA-9.6#S0268)s should be monotonic.

NOTE 1   A [delay_relative_statement](./AA-9.6#S0268) with a negative value of the delay_[expression](./AA-4.4#S0132) is equivalent to one with a zero value.

NOTE 2   {AI12-0440-1} A [delay_statement](./AA-9.6#S0266) can be executed by the environment task; consequently [delay_statement](./AA-9.6#S0266)s can be executed as part of the elaboration of a [library_item](./AA-10.1#S0287) or the execution of the main subprogram. Such statements delay the environment task (see 10.2).

NOTE 3   A [delay_statement](./AA-9.6#S0266) is an abort completion point and a potentially blocking operation, even if the task is not actually blocked.

NOTE 4   There is no necessary relationship between System.Tick (the resolution of the clock of package Calendar) and Duration'Small (the small of type Duration). 

Ramification: The inaccuracy of the [delay_statement](./AA-9.6#S0266) has no relation to System.Tick. In particular, it is possible that the clock used for the [delay_statement](./AA-9.6#S0266) is less accurate than Calendar.Clock.

We considered making Tick a run-time-determined quantity, to allow for easier configurability. However, this would not be upward compatible, and the desired configurability can be achieved using functionality defined in Annex D, "Real-Time Systems". 

NOTE 5   Additional requirements associated with [delay_statement](./AA-9.6#S0266)s are given in D.9, "Delay Accuracy".


#### Examples

Example of a relative delay statement: 

```ada
delay 3.0;  -- delay 3.0 seconds

```

Example of a periodic task: 

```ada
declare
   use Ada.Calendar;
   Next_Time : Time := Clock + Period;
                      -- Period is a global constant of type Duration
begin
   loop               -- repeated every Period seconds
      delay until Next_Time;
      ... -- perform some actions
      Next_Time := Next_Time + Period;
   end loop;
end;

```


#### Inconsistencies With Ada 83

For programs that raise Time_Error on "+" or "" in Ada 83,the exception might be deferred until a call on Split or Year_Number, or might not be raised at all (if the offending time is never Split after being calculated). This should not affect typical programs, since they deal only with times corresponding to the relatively recent past or near future. 


#### Extensions to Ada 83

The syntax rule for [delay_statement](./AA-9.6#S0266) is modified to allow [delay_until_statement](./AA-9.6#S0267)s.

{AI95-00351-01} The type Time may represent dates with year numbers outside of Year_Number. Therefore, the operations "+" and "" need only raise Time_Error if the result is not representable in Time (or Duration); also, Split or Year will now raise Time_Error if the year number is outside of Year_Number. This change is intended to simplify the implementation of "+" and "" (allowing them to depend on overflow for detecting when to raise Time_Error) and to allow local time zone information to be considered at the time of Split rather than Clock (depending on the implementation approach). For example, in a POSIX environment, it is natural for the type Time to be based on GMT, and the results of procedure Split (and the functions Year, Month, Day, and Seconds) to depend on local time zone information. In other environments, it is more natural for the type Time to be based on the local time zone, with the results of Year, Month, Day, and Seconds being pure functions of their input.

This paragraph was deleted.{AI95-00351-01} 


#### Inconsistencies With Ada 95

{AI95-00351-01} The upper bound of Year_Number has been changed to avoid a year 2100 problem. A program which expects years past 2099 to raise Constraint_Error will fail in Ada 2005. We don't expect there to be many programs which are depending on an exception to be raised. A program that uses Year_Number'Last as a magic number may also fail if values of Time are stored outside of the program. Note that the lower bound of Year_Number wasn't changed, because it is not unusual to use that value in a constant to represent an unknown time. 


#### Wording Changes from Ada 95

{8652/0002} {AI95-00171-01} Corrigendum: Clarified that Month, Day, and Seconds can raise Time_Error. 


## 9.6.1  Formatting, Time Zones, and other operations for Time


#### Static Semantics

{AI95-00351-01} {AI95-00427-01} The following language-defined library packages exist:

```ada
{AI12-0241-1} {AI12-0302-1} package Ada.Calendar.Time_Zones 
   with Nonblocking, Global =&gt in out synchronized is

```

```ada
   -- Time zone manipulation:

```

```ada
   type Time_Offset is range -28*60 .. 28*60;

```

Reason: {AI12-0005-1} We want to be able to specify the difference between any two arbitrary time zones. You might think that 1440 (24 hours) would be enough, but there are places (like Tonga, which is UTC+13hr) which are more than 12 hours different than UTC. Combined with summer time (known as daylight saving time in some parts of the world)  which switches opposite in the northern and southern hemispheres  and even greater differences are possible. We know of cases of a 26 hours difference, so we err on the safe side by selecting 28 hours as the limit. 

```ada
   Unknown_Zone_Error : exception;

```

```ada
{AI12-0336-1}    function Local_Time_Offset (Date : Time := Clock) return Time_Offset;

```

```ada
{AI12-0336-1}    function UTC_Time_Offset (Date : Time := Clock) return Time_Offset
      renames Local_Time_Offset;

```

```ada
end Ada.Calendar.Time_Zones;

```

```ada
{AI12-0241-1} {AI12-0302-1} 
package Ada.Calendar.Arithmetic 
   with Nonblocking, Global =&gt in out synchronized is

```

```ada
   -- Arithmetic on days:

```

```ada
   type Day_Count is range
     -366*(1+Year_Number'Last - Year_Number'First)
     ..
     366*(1+Year_Number'Last - Year_Number'First);

```

```ada
   subtype Leap_Seconds_Count is Integer range -2047 .. 2047;

```

Reason: The maximum number of leap seconds is likely to be much less than this, but we don't want to reach the limit too soon if the earth's behavior suddenly changes. We believe that the maximum number is 1612, based on the current rules, but that number is too weird to use here. 

```ada
   procedure Difference (Left, Right : in Time;
                         Days : out Day_Count;
                         Seconds : out Duration;
                         Leap_Seconds : out Leap_Seconds_Count);

```

```ada
   function "+" (Left : Time; Right : Day_Count) return Time;
   function "+" (Left : Day_Count; Right : Time) return Time;
   function "-" (Left : Time; Right : Day_Count) return Time;
   function "-" (Left, Right : Time) return Day_Count;

```

```ada
end Ada.Calendar.Arithmetic;

```

```ada
{AI12-0241-1} {AI12-0302-1} 
with Ada.Calendar.Time_Zones;
package Ada.Calendar.Formatting 
   with Nonblocking, Global =&gt in out synchronized is

```

```ada
   -- Day of the week:

```

```ada
   type Day_Name is (Monday, Tuesday, Wednesday, Thursday,
       Friday, Saturday, Sunday);

```

```ada
   function Day_of_Week (Date : Time) return Day_Name;

```

```ada
   -- Hours:Minutes:Seconds access:

```

```ada
   subtype Hour_Number         is Natural range 0 .. 23;
   subtype Minute_Number       is Natural range 0 .. 59;
   subtype Second_Number       is Natural range 0 .. 59;
   subtype Second_Duration     is Day_Duration range 0.0 .. 1.0;

```

```ada
   function Year       (Date : Time;
                        Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Year_Number;

```

```ada
   function Month      (Date : Time;
                        Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Month_Number;

```

```ada
   function Day        (Date : Time;
                        Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Day_Number;

```

```ada
   function Hour       (Date : Time;
                        Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Hour_Number;

```

```ada
   function Minute     (Date : Time;
                        Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Minute_Number;

```

```ada
   function Second     (Date : Time)
                           return Second_Number;

```

```ada
   function Sub_Second (Date : Time)
                           return Second_Duration;

```

```ada
   function Seconds_Of (Hour   :  Hour_Number;
                        Minute : Minute_Number;
                        Second : Second_Number := 0;
                        Sub_Second : Second_Duration := 0.0)
                           return Day_Duration;

```

```ada
   procedure Split (Seconds    : in Day_Duration;
                    Hour       : out Hour_Number;
                    Minute     : out Minute_Number;
                    Second     : out Second_Number;
                    Sub_Second : out Second_Duration);

```

```ada
   function Time_Of (Year       : Year_Number;
                     Month      : Month_Number;
                     Day        : Day_Number;
                     Hour       : Hour_Number;
                     Minute     : Minute_Number;
                     Second     : Second_Number;
                     Sub_Second : Second_Duration := 0.0;
                     Leap_Second: Boolean := False;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Time;

```

```ada
   function Time_Of (Year       : Year_Number;
                     Month      : Month_Number;
                     Day        : Day_Number;
                     Seconds    : Day_Duration := 0.0;
                     Leap_Second: Boolean := False;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                           return Time;

```

```ada
   procedure Split (Date       : in Time;
                    Year       : out Year_Number;
                    Month      : out Month_Number;
                    Day        : out Day_Number;
                    Hour       : out Hour_Number;
                    Minute     : out Minute_Number;
                    Second     : out Second_Number;
                    Sub_Second : out Second_Duration;
                    Time_Zone  : in Time_Zones.Time_Offset := 0);

```

```ada
   procedure Split (Date       : in Time;
                    Year       : out Year_Number;
                    Month      : out Month_Number;
                    Day        : out Day_Number;
                    Hour       : out Hour_Number;
                    Minute     : out Minute_Number;
                    Second     : out Second_Number;
                    Sub_Second : out Second_Duration;
                    Leap_Second: out Boolean;
                    Time_Zone  : in Time_Zones.Time_Offset := 0);

```

```ada
   procedure Split (Date       : in Time;
                    Year       : out Year_Number;
                    Month      : out Month_Number;
                    Day        : out Day_Number;
                    Seconds    : out Day_Duration;
                    Leap_Second: out Boolean;
                    Time_Zone  : in Time_Zones.Time_Offset := 0);

```

```ada
   -- Simple image and value:
   function Image (Date : Time;
                   Include_Time_Fraction : Boolean := False;
                   Time_Zone  : Time_Zones.Time_Offset := 0) return String;

```

```ada
{AI12-0336-1} {AI12-0347-1}    function Local_Image (Date : Time;
                         Include_Time_Fraction : Boolean := False)
      return String is
      (Image (Date, Include_Time_Fraction,
              Time_Zones.Local_Time_Offset (Date)));

```

```ada
   function Value (Date : String;
                   Time_Zone  : Time_Zones.Time_Offset := 0) return Time;

```

```ada
   function Image (Elapsed_Time : Duration;
                   Include_Time_Fraction : Boolean := False) return String;

```

```ada
   function Value (Elapsed_Time : String) return Duration;

```

```ada
end Ada.Calendar.Formatting;

```

{AI95-00351-01} {AI12-0336-1} Type Time_Offset represents for a given locality at a given moment  the number of minutes the local time is, at that moment, ahead (+) or behind (-) Coordinated Universal Time (abbreviated UTC).[ The Time_Offset for UTC is zero].

```ada
{AI12-0336-1} function Local_Time_Offset (Date : Time := Clock) return Time_Offset;

```

{AI95-00351-01} {AI05-0119-1} {AI05-0269-1} {AI12-0336-1} Returns, as a number of minutes, the Time_Offset of the implementation-defined time zone of Calendar, at the time Date. If the time zone of the Calendar implementation is unknown, then Unknown_Zone_Error is raised. 

Ramification: {AI05-0119-1} In North America, the result will be negative; in Europe, the result will be zero or positive. 

Discussion: {AI12-0336-1} The Date parameter is needed to take into account time differences caused by daylight-savings time and other time changes.

Other time zones can be supported with a child package. We don't define one because of the lack of agreement on the definition of a time zone.

The accuracy of this routine is not specified; the intent is that the facilities of the underlying target operating system are used to implement it. 

```ada
procedure Difference (Left, Right : in Time;
                      Days : out Day_Count;
                      Seconds : out Duration;
                      Leap_Seconds : out Leap_Seconds_Count);

```

{AI95-00351-01} {AI95-00427-01} Returns the difference between Left and Right. Days is the number of days of difference, Seconds is the remainder seconds of difference excluding leap seconds, and Leap_Seconds is the number of leap seconds. If Left &lt Right, then Seconds &lt= 0.0, Days &lt= 0, and Leap_Seconds &lt= 0. Otherwise, all values are nonnegative. The absolute value of Seconds is always less than 86_400.0. For the returned values, if Days = 0, then Seconds + Duration(Leap_Seconds) = Calendar."" (Left, Right). 

Discussion: Leap_Seconds, if any, are not included in Seconds. However, Leap_Seconds should be included in calculations using the operators defined in Calendar, as is specified for "" above. 

```ada
function "+" (Left : Time; Right : Day_Count) return Time;
function "+" (Left : Day_Count; Right : Time) return Time;

```

{AI95-00351-01} Adds a number of days to a time value. Time_Error is raised if the result is not representable as a value of type Time.

```ada
function "-" (Left : Time; Right : Day_Count) return Time;

```

{AI95-00351-01} Subtracts a number of days from a time value. Time_Error is raised if the result is not representable as a value of type Time.

```ada
function "-" (Left, Right : Time) return Day_Count;

```

{AI95-00351-01} Subtracts two time values, and returns the number of days between them. This is the same value that Difference would return in Days.

```ada
function Day_of_Week (Date : Time) return Day_Name;

```

{AI95-00351-01} Returns the day of the week for Time. This is based on the Year, Month, and Day values of Time.

```ada
function Year       (Date : Time;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                        return Year_Number;

```

{AI95-00427-01} Returns the year for Date, as appropriate for the specified time zone offset.

```ada
function Month      (Date : Time;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                        return Month_Number;

```

{AI95-00427-01} Returns the month for Date, as appropriate for the specified time zone offset.

```ada
function Day        (Date : Time;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                        return Day_Number;

```

{AI95-00427-01} Returns the day number for Date, as appropriate for the specified time zone offset.

```ada
function Hour       (Date : Time;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                        return Hour_Number;

```

{AI95-00351-01} Returns the hour for Date, as appropriate for the specified time zone offset.

```ada
function Minute     (Date : Time;
                     Time_Zone  : Time_Zones.Time_Offset := 0)
                        return Minute_Number;

```

{AI95-00351-01} Returns the minute within the hour for Date, as appropriate for the specified time zone offset.

```ada
function Second     (Date : Time)
                        return Second_Number;

```

{AI95-00351-01} {AI95-00427-01} Returns the second within the hour and minute for Date.

```ada
function Sub_Second (Date : Time)
                        return Second_Duration;

```

{AI95-00351-01} {AI95-00427-01} Returns the fraction of second for Date (this has the same accuracy as Day_Duration). The value returned is always less than 1.0.

```ada
function Seconds_Of (Hour   : Hour_Number;
                     Minute : Minute_Number;
                     Second : Second_Number := 0;
                     Sub_Second : Second_Duration := 0.0)
                        return Day_Duration;

```

{AI95-00351-01} {AI95-00427-01} Returns a Day_Duration value for the combination of the given Hour, Minute, Second, and Sub_Second. This value can be used in Calendar.Time_Of as well as the argument to Calendar."+" and Calendar."". If Seconds_Of is called with a Sub_Second value of 1.0, the value returned is equal to the value of Seconds_Of for the next second with a Sub_Second value of 0.0.

```ada
procedure Split (Seconds    : in Day_Duration;
                 Hour       : out Hour_Number;
                 Minute     : out Minute_Number;
                 Second     : out Second_Number;
                 Sub_Second : out Second_Duration);

```

{AI95-00351-01} {AI95-00427-01} {AI05-0238-1} Splits Seconds into Hour, Minute, Second and Sub_Second in such a way that the resulting values all belong to their respective subtypes. The value returned in the Sub_Second parameter is always less than 1.0. If Seconds = 86400.0, Split propagates Time_Error.

Ramification: There is only one way to do the split which meets all of the requirements. 

Reason: {AI05-0238-1} If Seconds = 86400.0, one of the returned values would have to be out of its defined range (either Sub_Second = 1.0 or Hour = 24 with the other value being 0). This doesn't seem worth breaking the invariants. 

```ada
function Time_Of (Year       : Year_Number;
                  Month      : Month_Number;
                  Day        : Day_Number;
                  Hour       : Hour_Number;
                  Minute     : Minute_Number;
                  Second     : Second_Number;
                  Sub_Second : Second_Duration := 0.0;
                  Leap_Second: Boolean := False;
                  Time_Zone  : Time_Zones.Time_Offset := 0)
                          return Time;

```

{AI95-00351-01} {AI95-00427-01} If Leap_Second is False, returns a Time built from the date and time values, relative to the specified time zone offset. If Leap_Second is True, returns the Time that represents the time within the leap second that is one second later than the time specified by the other parameters. Time_Error is raised if the parameters do not form a proper date or time. If Time_Of is called with a Sub_Second value of 1.0, the value returned is equal to the value of Time_Of for the next second with a Sub_Second value of 0.0. 

Discussion: Time_Error should be raised if Leap_Second is True, and the date and time values do not represent the second before a leap second. A leap second always occurs at midnight UTC, and is 23:59:60 UTC in ISO notation. So, if the time zone is UTC and Leap_Second is True, if any of Hour /= 23, Minute /= 59, or Second /= 59, then Time_Error should be raised. However, we do not say that, because other time zones will have different values where a leap second is allowed. 

```ada
function Time_Of (Year       : Year_Number;
                  Month      : Month_Number;
                  Day        : Day_Number;
                  Seconds    : Day_Duration := 0.0;
                  Leap_Second: Boolean := False;
                  Time_Zone  : Time_Zones.Time_Offset := 0)
                          return Time;

```

{AI95-00351-01} {AI95-00427-01} If Leap_Second is False, returns a Time built from the date and time values, relative to the specified time zone offset. If Leap_Second is True, returns the Time that represents the time within the leap second that is one second later than the time specified by the other parameters. Time_Error is raised if the parameters do not form a proper date or time. If Time_Of is called with a Seconds value of 86_400.0, the value returned is equal to the value of Time_Of for the next day with a Seconds value of 0.0.

```ada
procedure Split (Date       : in Time;
                 Year       : out Year_Number;
                 Month      : out Month_Number;
                 Day        : out Day_Number;
                 Hour       : out Hour_Number;
                 Minute     : out Minute_Number;
                 Second     : out Second_Number;
                 Sub_Second : out Second_Duration;
                 Leap_Second: out Boolean;
                 Time_Zone  : in Time_Zones.Time_Offset := 0);

```

{AI95-00351-01} {AI95-00427-01} If Date does not represent a time within a leap second, splits Date into its constituent parts (Year, Month, Day, Hour, Minute, Second, Sub_Second), relative to the specified time zone offset, and sets Leap_Second to False. If Date represents a time within a leap second, set the constituent parts to values corresponding to a time one second earlier than that given by Date, relative to the specified time zone offset, and sets Leap_Seconds to True. The value returned in the Sub_Second parameter is always less than 1.0.

```ada
procedure Split (Date       : in Time;
                 Year       : out Year_Number;
                 Month      : out Month_Number;
                 Day        : out Day_Number;
                 Hour       : out Hour_Number;
                 Minute     : out Minute_Number;
                 Second     : out Second_Number;
                 Sub_Second : out Second_Duration;
                 Time_Zone  : in Time_Zones.Time_Offset := 0);

```

{AI95-00351-01} {AI95-00427-01} Splits Date into its constituent parts (Year, Month, Day, Hour, Minute, Second, Sub_Second), relative to the specified time zone offset. The value returned in the Sub_Second parameter is always less than 1.0.

```ada
procedure Split (Date       : in Time;
                 Year       : out Year_Number;
                 Month      : out Month_Number;
                 Day        : out Day_Number;
                 Seconds    : out Day_Duration;
                 Leap_Second: out Boolean;
                 Time_Zone  : in Time_Zones.Time_Offset := 0);

```

{AI95-00351-01} {AI95-00427-01} If Date does not represent a time within a leap second, splits Date into its constituent parts (Year, Month, Day, Seconds), relative to the specified time zone offset, and sets Leap_Second to False. If Date represents a time within a leap second, set the constituent parts to values corresponding to a time one second earlier than that given by Date, relative to the specified time zone offset, and sets Leap_Seconds to True. The value returned in the Seconds parameter is always less than 86_400.0.

```ada
function Image (Date : Time;
                Include_Time_Fraction : Boolean := False;
                Time_Zone  : Time_Zones.Time_Offset := 0) return String;

```

{AI95-00351-01} Returns a string form of the Date relative to the given Time_Zone. The format is "Year-Month-Day Hour:Minute:Second", where the Year is a 4-digit value, and all others are 2-digit values, of the functions defined in Calendar and Calendar.Formatting, including a leading zero, if needed. The separators between the values are a minus, another minus, a colon, and a single space between the Day and Hour. If Include_Time_Fraction is True, the integer part of Sub_Seconds*100 is suffixed to the string as a point followed by a 2-digit value. 

Discussion: The Image provides a string in ISO 8601 format, the international standard time format. Alternative representations allowed in ISO 8601 are not supported here.

ISO 8601 allows 24:00:00 for midnight; and a seconds value of 60 for leap seconds. These are not allowed here (the routines mentioned above cannot produce those results). 

Ramification: The fractional part is truncated, not rounded. It would be quite hard to define the result with proper rounding, as it can change all of the values of the image. Values can be rounded up by adding an appropriate constant (0.5 if Include_Time_Fraction is False, 0.005 otherwise) to the time before taking the image. 

```ada
function Value (Date : String;
                Time_Zone  : Time_Zones.Time_Offset := 0) return Time;

```

{AI95-00351-01} Returns a Time value for the image given as Date, relative to the given time zone. Constraint_Error is raised if the string is not formatted as described for Image, or the function cannot interpret the given string as a Time value.

Discussion: {AI05-0005-1} The intent is that the implementation enforce the same range rules on the string as the appropriate function Time_Of, except for the hour, so "cannot interpret the given string as a Time value" happens when one of the values is out of the required range. For example, "2005-08-31 24:00:00" should raise Constraint_Error (the hour is out of range). 

```ada
function Image (Elapsed_Time : Duration;
                Include_Time_Fraction : Boolean := False) return String;

```

{AI95-00351-01} {AI12-0445-1} Returns a string form of the Elapsed_Time. The format is "Hour:Minute:Second", where all values are 2-digit values, including a leading zero, if necessary. The separators between the values are colons. If Include_Time_Fraction is True, the integer part of Sub_Seconds*100 is suffixed to the string as a point followed by a 2-digit value. If Elapsed_Time &lt 0.0, the result is Image (abs Elapsed_Time, Include_Time_Fraction) prefixed with a minus sign. If abs Elapsed_Time represents 100 hours or more, the result is implementation-defined. 

Implementation defined: The result of Calendar.Formatting.Image if its argument represents more than 100 hours.

Implementation Note: This cannot be implemented (directly) by calling Calendar.Formatting.Split, since it may be out of the range of Day_Duration, and thus the number of hours may be out of the range of Hour_Number.

If a Duration value can represent more then 100 hours, the implementation will need to define a format for the return of Image. 

```ada
function Value (Elapsed_Time : String) return Duration;

```

{AI95-00351-01} Returns a Duration value for the image given as Elapsed_Time. Constraint_Error is raised if the string is not formatted as described for Image, or the function cannot interpret the given string as a Duration value. 

Discussion: The intent is that the implementation enforce the same range rules on the string as the appropriate function Time_Of, except for the hour, so "cannot interpret the given string as a Time value" happens when one of the values is out of the required range. For example, "10:23:60" should raise Constraint_Error (the seconds value is out of range). 


#### Implementation Advice

{AI95-00351-01} An implementation should support leap seconds if the target system supports them. If leap seconds are not supported, Difference should return zero for Leap_Seconds, Split should return False for Leap_Second, and Time_Of should raise Time_Error if Leap_Second is True. 

Implementation Advice: Leap seconds should be supported if the target system supports them. Otherwise, operations in Calendar.Formatting should return results consistent with no leap seconds.

Discussion: An implementation can always support leap seconds when the target system does not; indeed, this isn't particularly hard (all that is required is a table of when leap seconds were inserted). As such, leap second support isn't "impossible or impractical" in the sense of . However, for some purposes, it may be important to follow the target system's lack of leap second support (if the target is a GPS satellite, which does not use leap seconds, leap second support would be a handicap to work around). Thus, this Implementation Advice should be read as giving permission to not support leap seconds on target systems that don't support leap seconds. Implementers should use the needs of their customers to determine whether or not support leap seconds on such targets. 

NOTE 1   {AI95-00351-01} {AI12-0336-1} {AI12-0442-1} The implementation-defined time zone of package Calendar can be the local time zone. Local_Time_Offset always returns the difference relative to the implementation-defined time zone of package Calendar. If Local_Time_Offset does not raise Unknown_Zone_Error, UTC time can be safely calculated (within the accuracy of the underlying time-base).

Discussion: {AI95-00351-01} The time in the time zone known as Greenwich Mean Time (GMT) is generally very close to UTC time; for most purposes they can be treated the same. GMT is the time based on the rotation of the Earth; UTC is the time based on atomic clocks, with leap seconds periodically inserted to realign with GMT (because most human activities depend on the rotation of the Earth). At any point in time, there will be a sub-second difference between GMT and UTC. 

NOTE 2   {AI95-00351-01} {AI12-0336-1} Calling Split on the results of subtracting Duration(Local_Time_Offset*60) from Clock provides the components (hours, minutes, and so on) of the UTC time. In the United States, for example, Local_Time_Offset will generally be negative. 

Discussion: {AI12-0336-1} This is an illustration to help specify the value of Local_Time_Offset. A user should pass zero as the Time_Zone parameter of Split, rather than trying to make the above calculation. 


#### Extensions to Ada 95

{AI95-00351-01} {AI95-00428-01} Packages Calendar.Time_Zones, Calendar.Arithmetic, and Calendar.Formatting are new. 


#### Inconsistencies With Ada 2005

{AI05-0238-1} Correction: Defined that Split for Seconds raises Time_Error for a value of exactly 86400.0, rather than breaking some invariant or raising some other exception. Ada 2005 left this unspecified; a program that depended on what some implementation does might break, but such a program is not portable anyway. 


#### Wording Changes from Ada 2005

{AI05-0119-1} Correction: Clarified the sign of UTC_Time_Offset. 


#### Inconsistencies With Ada 2012

{AI12-0336-1} Correction: Changed the definition of Time_Offset to be compatible with most compilers. Also added Local_Time_Offset and Local_Image to better describe the intent. A program that expects the Ada 2012 definition of Time_Offset would get incorrect answers. However, most compilers tested use the revised definition, so the likelihood of a program breaking is quite low. Additionally, the new definitions could cause a use clause conflict; see the introduction of Annex A for more on this topic. 
