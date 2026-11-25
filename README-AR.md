تقاعدت مكتبة [محلّل الوحدات](https://github.com/laysar/Unit-Parser)، ولم تعد قيد الصيانة، إلا أنها على درجة عالية جدًا من الاستقرار، مما يتيح للمشاريع الحالية مواصلة الاعتماد عليها لسنوات طويلة، بل إلى أجل غير مسمّى تقريبًا، ما دامت بيئة تشغيلها متوافقة معها

انتقل التطوير إلى مكتبة [وحدات](https://github.com/laysar/Wahdat)، التي ستصدر قريبًا جدًا، ويمكنك متابعة مستجدات تطويرها عبر [تلجرام](https://t.me/LaysarSiXTeeN)

**العربية** • [English](README.md)

# محلّل الوحدات

مكتبة Java خفيفة لأندرويد لتحليل القيم ذات الوحدات وتحويلها والتحقق منها بطريقة موحدة وسريعة.

صُممت للاستخدام المباشر داخل التطبيقات أو كأساس داخل المكتبات، وتدعم وحدات Android والطباعة والأطوال الفيزيائية، بالإضافة إلى النسب المئوية والدرجات، مع **سياسات للوحدات** و**قيود للإدخال** ونتائج تفصيلية عند الحاجة.

```text
com.laysar.Core.UnitParser
```

---

## التثبيت

```gradle
implementation("io.github.laysar:UnitParser:1.0.0")
```

ثم:

```java
import com.laysar.Core.UnitParser;
```

---

## بداية سريعة

```java
float Size = UnitParser.toPX(Context, "16dp");
```

أمثلة أخرى:

```java
float Pixels = UnitParser.toPX(Context, "20sp");
float Ratio  = UnitParser.toPX(null, "64%");
float Angle  = UnitParser.toPX(null, "45°");
float Raw    = UnitParser.toPX(null, "12.5");
```

| الإدخال | النتيجة |
| --- | ---: |
| `12.5` | `12.5f` |
| `12px` | `12.0f` |
| `64%` | `0.64f` |
| `45°` | `45.0f` |
| `16dp` | يتحول إلى `px` حسب كثافة الشاشة |
| `20sp` | يتحول إلى `px` حسب إعدادات الخط |

---

# الوحدات المدعومة

| الوحدة | النوع | السلوك |
| --- | --- | --- |
| بدون وحدة | `UNITLESS` | تعاد كما هي |
| `px` | `PIXEL` | تعاد كما هي |
| `dp` | `DENSITY_INDEPENDENT_PIXEL` | تتحول إلى `px` |
| `sp` | `SCALE_INDEPENDENT_PIXEL` | تتحول إلى `px` |
| `pt` | `POINT` | تتحول إلى `px` |
| `pc` | `PICA` | `1pc = 12pt` |
| `in` | `INCH` | تتحول إلى `px` |
| `cm` | `CENTIMETER` | تتحول إلى `px` |
| `mm` | `MILLIMETER` | تتحول إلى `px` |
| `qmm` | `QUARTER_MILLIMETER` | `1qmm = 0.25mm` |
| `%` | `PERCENT` | تقسم على `100` |
| `°` | `DEGREE` | تعاد كما هي |

القيم الموجبة والسالبة مدعومة، ولا يتم تقريب الناتج.

---

# أي دالة أستخدم؟

| المطلوب | الدالة |
| --- | --- |
| القيمة فقط باستخدام `Context` | `toPX(...)` |
| القيمة فقط باستخدام `Resources` | `toPXFromResources(...)` |
| القيمة + الوحدة + حالة النجاح | `toPXResult(...)` |
| نتيجة تفصيلية باستخدام `Resources` | `toPXResultFromResources(...)` |
| سياسات وحدود للإدخال | `toPXWithConstraints(...)` |
| قيود + نتيجة تفصيلية | `toPXResultWithConstraints(...)` |
| قيود باستخدام `Resources` | `toPXFromResourcesWithConstraints(...)` |
| قيود + نتيجة تفصيلية باستخدام `Resources` | `toPXResultFromResourcesWithConstraints(...)` |

---

# استخدام `Resources`

إذا كان لديك `Resources` مباشرة، فلا تحتاج إلى استخراج `Context`.

```java
float Size = UnitParser.toPXFromResources(
        Resources,
        "16dp"
);
```

وتتوفر نفس السياسات والقيود:

```java
UnitParser.toPXFromResources(Resources, RawValue);
UnitParser.toPXFromResources(Resources, RawValue, Policy);
UnitParser.toPXFromResources(Resources, RawValue, FirstPolicy, SecondPolicy);
UnitParser.toPXFromResourcesWithConstraints(Resources, RawValue, Constraints);
```

هذه القيم لا تحتاج إلى `Context` أو `Resources`:

```text
بدون وحدة
px
%
°
```

مثلًا:

```java
float Ratio = UnitParser.toPX(null, "64%");   // 0.64f
float Angle = UnitParser.toPX(null, "45°");   // 45.0f
```

---

# النتيجة التفصيلية `UnitResult`

استخدم `UnitResult` عندما لا تكفي القيمة وحدها.

```java
UnitParser.UnitResult Result =
        UnitParser.toPXResult(Context, "16dp");

if (Result.isValid()) {
    float Value = Result.getValue();
    int Unit = Result.getUnitType();
}
```

| الدالة | المعنى |
| --- | --- |
| `getValue()` | القيمة الناتجة |
| `getUnitType()` | وحدة الإدخال |
| `isValid()` | هل العملية نجحت؟ |
| `isUnitResolved()` | هل تم التعرف على الوحدة؟ |

## لماذا النتيجة التفصيلية مفيدة؟

لأن `0.0f` ليست دائمًا خطأ.

```text
0% → Value = 0.0f, Valid = true
```

بينما الإدخال غير الصالح قد ينتج:

```text
Value = 0.0f, Valid = false
```

إذا كنت تحتاج إلى التمييز بين الحالتين، استخدم `UnitResult`.

---

# سياسات الوحدات

يمكن تحديد الوحدات التي يقبلها كل استخدام بدل قبول جميع الوحدات دائمًا.

## السماح بوحدات محددة

```java
UnitParser.UnitPolicy Policy =
        UnitParser.fromAllowedUnits(
                UnitParser.ANDROID_UNITS
        );

float Value = UnitParser.toPX(
        Context,
        "16dp",
        Policy
);
```

`ANDROID_UNITS` تسمح بـ:

```text
px
dp
sp
```

وأي وحدة أخرى تكون مرفوضة.

---

## رفض وحدات محددة

```java
UnitParser.UnitPolicy Policy =
        UnitParser.fromRejectedUnits(
                UnitParser.UnitType.PERCENT
        );
```

هذه السياسة تسمح بجميع الوحدات المدعومة ما عدا `%`.

---

## دمج المجموعات والوحدات

```java
UnitParser.UnitPolicy Policy =
        UnitParser.fromAllowedUnits(
                UnitParser.ANDROID_UNITS,
                UnitParser.PRINT_UNITS,
                UnitParser.UnitType.PERCENT
        );
```

الوحدات المقبولة:

```text
px
dp
sp
pt
pc
%
```

---

# المجموعات الجاهزة

| المجموعة | الوحدات |
| --- | --- |
| `RAW_VALUE_ONLY` | بدون وحدة فقط |
| `ALL_UNITS` | جميع الوحدات المدعومة |
| `ANDROID_UNITS` | `px`, `dp`, `sp` |
| `PRINT_UNITS` | `pt`, `pc` |
| `PHYSICAL_LENGTH_UNITS` | `in`, `cm`, `mm`, `qmm` |

ويمكن اختيار وحدة منفردة مباشرة:

```java
UnitParser.UnitType.PERCENT
UnitParser.UnitType.DEGREE
UnitParser.UnitType.PIXEL
```

---

# استخدام سياستين

يمكن تطبيق سياستين على الاستدعاء نفسه.

```java
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
```

| الوحدة | النتيجة |
| --- | --- |
| `dp` | مقبولة |
| `sp` | مقبولة |
| `px` | مرفوضة |
| `mm` | مرفوضة |

يجب أن تسمح **السياستان معًا** بالوحدة.

---

# قيود الإدخال

تسمح `InputConstraints` بوضع حدود عامة أو حدود مخصصة لوحدات محددة.

```java
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
```

ثم:

```java
float Value = UnitParser.toPXWithConstraints(
        Context,
        "500dp",
        Constraints
);
```

---

## كيف تطبق الحدود؟

الحدود تطبق على **الرقم المكتوب قبل التحويل**.

| الإدخال | الرقم الذي يتم فحصه |
| --- | ---: |
| `64%` | `64` |
| `16dp` | `16` |
| `4qmm` | `4` |
| `45°` | `45` |

مثلًا إذا كان نطاق `%`:

```text
0 .. 100
```

فالنتيجة:

| الإدخال | الحالة |
| --- | --- |
| `0%` | مقبول |
| `64%` | مقبول |
| `100%` | مقبول |
| `120%` | مرفوض |
| `-1%` | مرفوض |

القيم الخارجة عن النطاق ترفض، ولا يتم قصها أو تعديلها تلقائيًا.

---

# واجهات `InputConstraints.Builder`

## الحدود العامة

```java
setGeneralMinimum(...)
setGeneralMaximum(...)
setGeneralRange(...)

unsetGeneralMinimum()
unsetGeneralMaximum()
unsetGeneralRange()
```

## حدود وحدة محددة

```java
setUnitMinimum(...)
setUnitMaximum(...)
setUnitRange(...)

unsetUnitMinimum()
unsetUnitMaximum()
unsetUnitRange(...)
```

## السياسات

```java
setFirstUnitPolicy(...)
setSecondUnitPolicy(...)
```

ثم إنشاء القيود النهائية:

```java
toConstraints()
```

---

# دمج السياسات مع القيود

يمكن تخزين سياستي الوحدات داخل `InputConstraints` نفسها.

```java
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
```

في هذا المثال:

| الوحدة | الحالة |
| --- | --- |
| `dp` | مقبولة |
| `sp` | مقبولة |
| `px` | مرفوضة |
| الوحدات خارج `ANDROID_UNITS` | مرفوضة |

---

# الإدخال الصارم

محلّل الوحدات لا يحاول تخمين المدخلات غير الواضحة.

### مقبول

```text
16dp
-20sp
.5%
10.
45°
```

### غير مقبول

```text
10 dp
1e3dp
1_000dp
10dip
10vw
50%%
10%px
NaNpx
Infinitypx
```

---

# معالجة الأخطاء

أخطاء التحليل والتحقق مصممة بحيث لا تؤدي إلى إسقاط التطبيق.

عند حدوث خطأ:

1. يتم تسجيل رسالة واضحة في Logcat تحت الوسم `UnitParser`.
2. الدوال التي تعيد قيمة ترجع `0.0f`.
3. الدوال التي تعيد `UnitResult` ترجع نتيجة غير صالحة.

إذا كانت `0.0f` قيمة صحيحة محتملة في منطقك، استخدم:

```java
toPXResult(...)
```

بدل الاعتماد على الرقم وحده.

---

# ملاحظات سريعة

- لا يوجد تقريب تلقائي.
- لا يوجد Clamping.
- `%` نسبة معيارية: `64% → 0.64f`.
- `°` تبقى قيمة زاوية كما هي.
- `qmm` تعني ربع مليمتر.
- السياسات تطبق قبل قيود الرقم.
- قيود الرقم تطبق قبل التحويل.
- القيم التي لا تحتاج إلى مقاييس تعمل بدون `Context` أو `Resources`.
- يمكن استخدام `Context` أو `Resources` حسب المتاح.

---

# مثال متكامل

```java
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
```

---

## المتطلبات

- Android
- Java 8+
- Android API 26+
