CylValve Add-On Instruction - README
Revision 1.5  |  Studio 5000 Logix Designer v35  |  Ladder logic only
======================================================================

PURPOSE
-------
Controls an air cylinder through a single or double solenoid valve.
Confirms position with two proxes, runs a watchdog timer on each move,
raises digital alarms on timeout, and records stroke times.

Required = must be assigned a tag when the AOI is placed in ladder.
Visible  = shown on the instruction block.


INPUT PARAMETERS
----------------
Cmd_Extend        BOOL  Required
    Extend request from the main program. Must be HELD ON to keep the
    cylinder extended. If Cmd_Retract is also on, neither solenoid fires.

Cmd_Retract       BOOL  Required
    Retract request from the main program. Must be HELD ON to keep the
    cylinder retracted. If Cmd_Extend is also on, neither solenoid fires.

Inp_ProgReset     BOOL  Required
    Wired to the ProgReset operand of both ALMD alarms. Resets a latched
    alarm once the fault condition has cleared. Typically tied to a
    reset pushbutton or an HMI reset bit.

Inp_ExtendedPrx   BOOL  Required
    Extended-position prox input.

Inp_RetractedPrx  BOOL  Required
    Retracted-position prox input.

Cfg_ExtendTime    DINT  Optional (hidden)  Default 5000
    Extend watchdog time in ms. Values of 0 or less are ignored and the
    5000 ms default is used.

Cfg_RetractTime   DINT  Optional (hidden)  Default 5000
    Retract watchdog time in ms. Values of 0 or less are ignored and the
    5000 ms default is used.


OUTPUT PARAMETERS
-----------------
Out_ExtendSol     BOOL  Required
    Extend solenoid output. On while Cmd_Extend is held (and Cmd_Retract
    is off). For a single-solenoid spring-return valve, this is the only
    output used.

Out_RetractSol    BOOL  Required
    Retract solenoid output for double-solenoid valves. On while
    Cmd_Retract is held (and Cmd_Extend is off). On a single-solenoid
    valve, assign it to an unused/spare BOOL tag.

Sts_Extended      BOOL  Visible
    Extended prox on AND retracted prox off.

Sts_Retracted     BOOL  Visible
    Retracted prox on AND extended prox off.

Sts_ExtendFault   BOOL  Visible
    Extend watchdog timed out. Clears when the extended prox is made or
    the extend request drops. (The alarm holds until reset - see below.)

Sts_RetractFault  BOOL  Visible
    Retract watchdog timed out. Clears when the retracted prox is made or
    the retract request drops.

Sts_ExtendLast    DINT  Visible
    Time of the most recent extend, in ms.

Sts_ExtendAvg     REAL  Visible
    Average of the last 5 extend times, in ms. Averages fewer samples
    until 5 extends have been recorded.

Sts_RetractLast   DINT  Visible
    Time of the most recent retract, in ms.

Sts_RetractAvg    REAL  Visible
    Average of the last 5 retract times, in ms.

Sts_Extending     BOOL  Hidden
    Extend solenoid on, extended position not yet confirmed.

Sts_Retracting    BOOL  Hidden
    Retract solenoid on, retracted position not yet confirmed.

Sts_ProxFault     BOOL  Hidden
    Both proxes on at the same time (status only, no alarm).

Hidden outputs are read from the instance tag, e.g. MyCyl.Sts_ProxFault


INOUT PARAMETERS
----------------
InOut parameters pass a reference to a tag you create outside the AOI.
The AOI reads and writes that tag directly.

Alm_ExtendFail    ALARM_DIGITAL  Required
    Digital alarm for "cylinder failed to extend within watchdog time".
    Create an ALARM_DIGITAL tag for each cylinder and assign it here.

Alm_RetractFail   ALARM_DIGITAL  Required
    Digital alarm for "cylinder failed to retract within watchdog time".
    Create an ALARM_DIGITAL tag for each cylinder and assign it here.

Alarm tag setup (in each ALARM_DIGITAL tag's properties):
  - Check LATCHED. Without it, Inp_ProgReset has no effect and the alarm
    clears as soon as the cylinder reaches position.
  - Enter the alarm message text and severity on the tag.
  - If ACKNOWLEDGE REQUIRED is checked, the operator must acknowledge
    the alarm as well as reset it.


INTERNAL (LOCAL) TAGS - for reference while monitoring the AOI logic
-------------------------------------------------------------------
ExtendWD / RetractWD            TIMER     Watchdog timers (TON)
ExtendStroke / RetractStroke    TIMER     Stroke time measurement (RTO)
ExtendTimes / RetractTimes      DINT[5]   Last 5 times in ms, [0] newest
ExtendCount / RetractCount      DINT      Samples recorded (0-5)
ExtendLogged / RetractLogged    BOOL      Seal-in, records each move once
ExtendSum / RetractSum          REAL      Sum used for the average


NOTES
-----
- No latch/unlatch instructions and no one-shots are used. One-time
  recording of each move is done with a seal-in rung.
- Retract times are only recorded while Cmd_Retract is held. On a
  single-solenoid valve, a spring return from dropping Cmd_Extend is not
  timed.
- Times are accurate to the scan/period time of the task the AOI runs in.
