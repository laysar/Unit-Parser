[العربية](README-AR.md) • **English**

Unit Parser

A lightweight Java utility for Android that parses, converts, and validates unit-based values through a unified and fast API.

It is designed for direct use in applications or as a foundation within libraries. It supports Android units, print units, physical length units, percentages, and degrees, along with unit policies, input constraints, and detailed results when needed.

com.laysar.Core.UnitParser

---

Installation

implementation("com.laysar:UnitParser:1.0.0")

Then:

import com.laysar.Core.UnitParser;

---

Quick Start

float Size = UnitParser.toPX(Context, "16dp");

More examples:

float Pixels = UnitParser.toPX(Context, "20sp");
float Ratio  = UnitParser.toPX(null, "64%");
float Angle  = UnitParser.toPX(null, "45°");
float Raw    = UnitParser.toPX(null, "12.5");

Input| Result
"12.5"| "12.5f"
"12px"| "12.0f"
"64%"| "0.64f"
"45°"| "45.0f"
"16dp"| Converted to "px" using display density
"20sp"| Converted to "px" using font scaling

---

Supported Units

Unit| Type| Behavior
No suffix| "UNITLESS"| Returned unchanged
"px"| "PIXEL"| Returned unchanged
"dp"| "DENSITY_INDEPENDENT_PIXEL"| Converted to "px"
"sp"| "SCALE_INDEPENDENT_PIXEL"| Converted to "px"
"pt"| "POINT"| Converted to "px"
"pc"| "PICA"| "1pc = 12pt"
"in"| "INCH"| Converted to "px"
"cm"| "CENTIMETER"| Converted to "px"
"mm"| "MILLIMETER"| Converted to "px"
"qmm"| "QUARTER_MILLIMETER"| "1qmm = 0.25mm"
"%"| "PERCENT"| Divided by "100"
"°"| "DEGREE"| Returned unchanged

Positive and negative values are supported, and results are not rounded.

---

Which Method Should I Use?

Need| Method
Value only using "Context"| "toPX(...)"
Value only using "Resources"| "toPXFromResources(...)"
Value + unit + success state| "toPXResult(...)"
Detailed result using "Resources"| "toPXResultFromResources(...)"
Unit policies and numeric constraints| "toPXWithConstraints(...)"
Constraints + detailed result| "toPXResultWithConstraints(...)"
Constraints using "Resources"| "toPXFromResourcesWithConstraints(...)"
Constraints + detailed result using "Resources"| "toPXResultFromResourcesWithConstraints(...)"

---

Using "Resources"

If you already have a "Resources" instance, you do not need to extract a "Context".

float Size = UnitParser.toPXFromResources(
Resources,
"16dp"
);

The same policy and constraint features are available:

UnitParser.toPXFromResources(Resources, RawValue);
UnitParser.toPXFromResources(Resources, RawValue, Policy);
UnitParser.toPXFromResources(Resources, RawValue, FirstPolicy, SecondPolicy);
UnitParser.toPXFromResourcesWithConstraints(Resources, RawValue, Constraints);

These values do not require "Context" or "Resources":

unitless
px
%
°

For example:

float Ratio = UnitParser.toPX(null, "64%");   // 0.64f
float Angle = UnitParser.toPX(null, "45°");   // 45.0f

---

Detailed Results with "UnitResult"

Use "UnitResult" when the numeric value alone is not enough.

UnitParser.UnitResult Result =
UnitParser.toPXResult(Context, "16dp");

if (Result.isValid()) {
float Value = Result.getValue();
int Unit = Result.getUnitType();
}

Method| Meaning
"getValue()"| Resulting value
"getUnitType()"| Original input unit
"isValid()"| Whether the operation succeeded
"isUnitResolved()"| Whether a supported unit was identified

Why is the detailed result useful?

Because "0.0f" is not always an error.

0% → Value = 0.0f, Valid = true

An invalid input may instead produce:

Value = 0.0f, Valid = false

Use "UnitResult" whenever you need to distinguish between these cases.

---

Unit Policies

Each use can restrict which supported units are accepted instead of always accepting every unit.

Allow Specific Units

UnitParser.UnitPolicy Policy =
UnitParser.fromAllowedUnits(
UnitParser.ANDROID_UNITS
);

float Value = UnitParser.toPX(
Context,
"16dp",
Policy
);

"ANDROID_UNITS" allows:

px
dp
sp

Every other supported unit is rejected.

---

Reject Specific Units

UnitParser.UnitPolicy Policy =
UnitParser.fromRejectedUnits(
UnitParser.UnitType.PERCENT
);

This policy allows every supported unit except "%".

---

Combine Groups and Individual Units

UnitParser.UnitPolicy Policy =
UnitParser.fromAllowedUnits(
UnitParser.ANDROID_UNITS,
UnitParser.PRINT_UNITS,
UnitParser.UnitType.PERCENT
);

Allowed units:

px
dp
sp
pt
pc
%

---

Built-in Unit Groups

Group| Units
"RAW_VALUE_ONLY"| Unitless values only
"ALL_UNITS"| All supported units
"ANDROID_UNITS"| "px", "dp", "sp"
"PRINT_UNITS"| "pt", "pc"
"PHYSICAL_LENGTH_UNITS"| "in", "cm", "mm", "qmm"

Individual units can also be selected directly:

UnitParser.UnitType.PERCENT
UnitParser.UnitType.DEGREE
UnitParser.UnitType.PIXEL

---

Using Two Policies

Two policies can be applied to the same call.

UnitParser.UnitPolicy AndroidOnly =
UnitParser.fromAllowedUnits(
UnitParser.ANDROID_UNITS
);

UnitParser.UnitPolicy RejectPX =
UnitParser.fromRejectedUnits(
UnitParser.UnitType.PIXEL
);

float Value = UnitParser.toPX(
Context,
"16dp",
AndroidOnly,
RejectPX
);

Unit| Result
"dp"| Allowed
"sp"| Allowed
"px"| Rejected
"mm"| Rejected

A unit must be allowed by both policies.

---

Input Constraints

"InputConstraints" can define general numeric limits or custom limits for specific units.

UnitParser.InputConstraints Constraints =
new UnitParser.InputConstraints.Builder()
.setGeneralRange(0.0f, 1000.0f)
.setUnitRange(
UnitParser.UnitType.PERCENT,
0.0f,
100.0f
)
.setUnitRange(
UnitParser.UnitType.DEGREE,
-360.0f,
360.0f
)
.setUnitMaximum(
UnitParser.UnitType.DENSITY_INDEPENDENT_PIXEL,
600.0f
)
.toConstraints();

Then:

float Value = UnitParser.toPXWithConstraints(
Context,
"500dp",
Constraints
);

---

How Are Constraints Applied?

Constraints are evaluated against the numeric portion of the original input before conversion.

Input| Value Checked
"64%"| "64"
"16dp"| "16"
"4qmm"| "4"
"45°"| "45"

For example, if the "%" range is:

0 .. 100

the result is:

Input| Status
"0%"| Allowed
"64%"| Allowed
"100%"| Allowed
"120%"| Rejected
"-1%"| Rejected

Out-of-range values are rejected. They are not automatically clamped or modified.

---

"InputConstraints.Builder" API

General Limits

setGeneralMinimum(...)
setGeneralMaximum(...)
setGeneralRange(...)

unsetGeneralMinimum()
unsetGeneralMaximum()
unsetGeneralRange()

Unit-Specific Limits

setUnitMinimum(...)
setUnitMaximum(...)
setUnitRange(...)

unsetUnitMinimum(...)
unsetUnitMaximum()
unsetUnitRange(...)

Policies

setFirstUnitPolicy(...)
setSecondUnitPolicy(...)

Create the final constraints with:

toConstraints()

---

Combining Policies with Constraints

Both unit policies can be stored directly inside "InputConstraints".

UnitParser.InputConstraints Constraints =
new UnitParser.InputConstraints.Builder()
.setFirstUnitPolicy(
UnitParser.fromAllowedUnits(
UnitParser.ANDROID_UNITS
)
)
.setSecondUnitPolicy(
UnitParser.fromRejectedUnits(
UnitParser.UnitType.PIXEL
)
)
.setGeneralRange(0.0f, 1000.0f)
.toConstraints();

In this example:

Unit| Status
"dp"| Allowed
"sp"| Allowed
"px"| Rejected
Units outside "ANDROID_UNITS"| Rejected

---

Strict Input Parsing

The parser does not try to guess ambiguous input.

Accepted

16dp
-20sp
.5%
10.
45°

Rejected

10 dp
1e3dp
1_000dp
10dip
10vw
50%%
10%px
NaNpx
Infinitypx

---

Error Handling

Parsing and validation failures are designed not to crash the application.

When an error occurs:

1. A clear message is written to Logcat using the "UnitParser" tag.
2. Value-returning methods return "0.0f".
3. Methods returning "UnitResult" return an invalid result.

If "0.0f" can also be a valid value in your logic, use:

toPXResult(...)

instead of relying on the numeric value alone.

---

Quick Notes

- No automatic rounding.
- No clamping.
- "%" is normalized: "64% → 0.64f".
- "°" remains an angle value.
- "qmm" means quarter millimeter.
- Unit policies are evaluated before numeric constraints.
- Numeric constraints are evaluated before conversion.
- Values that do not require display metrics work without "Context" or "Resources".
- Both "Context" and "Resources" APIs are available.

---

Complete Example

UnitParser.UnitPolicy Allowed =
UnitParser.fromAllowedUnits(
UnitParser.ANDROID_UNITS,
UnitParser.UnitType.PERCENT
);

UnitParser.UnitPolicy Rejected =
UnitParser.fromRejectedUnits(
UnitParser.UnitType.PIXEL
);

UnitParser.InputConstraints Constraints =
new UnitParser.InputConstraints.Builder()
.setFirstUnitPolicy(Allowed)
.setSecondUnitPolicy(Rejected)
.setGeneralRange(0.0f, 1000.0f)
.setUnitRange(
UnitParser.UnitType.PERCENT,
0.0f,
100.0f
)
.setUnitMaximum(
UnitParser.UnitType.DENSITY_INDEPENDENT_PIXEL,
600.0f
)
.toConstraints();

UnitParser.UnitResult Result =
UnitParser.toPXResultWithConstraints(
Context,
"16dp",
Constraints
);

if (Result.isValid()) {
float Pixels = Result.getValue();
}

---

Requirements

- Android
- Java 8+
- Android API 26+