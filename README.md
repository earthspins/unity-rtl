# RTL – Right to Left Text for Unity

_Correctly render Arabic, Persian, Hebrew, Urdu, and Pashto text in Unity UI._

[![Unity Asset Store](https://img.shields.io/badge/Unity%20Asset%20Store-RTL%20Plugin-blue)](http://u3d.as/3kv)
[![Unity 6](https://img.shields.io/badge/Unity-6+-black)](https://unity.com)
[![.NET Standard 2.1](https://img.shields.io/badge/.NET-Standard%202.1-purple)](https://learn.microsoft.com/en-us/dotnet/standard/net-standard)

---

**RTL** is a Unity plugin that enables correct rendering of right-to-left languages in Unity UI. It processes your raw RTL text and produces a visually correct string that `UI.Text` and `TextMeshPro` can display properly, no engine changes required.

```csharp
// "New weapon unlocked. Are you ready to fight?"
string fixedText = RTLService.Convert("سلاح جدید باز شد. آماده نبرد هستید؟");
textComponent.text = fixedText;
```

## What It Does

RTL processes input strings before they are assigned to Unity's text component. It applies:

- **Bidirectional reordering** - characters and words are placed in correct visual RTL order
- **Contextual letter shaping** - Arabic and Persian letters take their correct initial, medial, final, or isolated forms
- **Mixed-direction support** - Persian or Arabic sentences containing English words or numbers render in proper logical order
- **RTL-aware word wrapping** - line breaks are inserted at correct boundaries for RTL scripts
- **Rich text preservation** - Unity tags like `<color>`, `<b>`, `<i>`, and `<size>` survive the conversion intact

The result is a correctly shaped, correctly ordered string that Unity renders normally.

**The plugin runs entirely locally and works offline, requiring no internet connection, external services, or third-party dependencies.**

## Supported Languages

- Arabic
- Persian (Farsi)
- Hebrew
- Urdu
- Pashto

---

## Why It Is Needed

Unity's built-in text components (`UI.Text` and `TextMeshPro`) render all text strictly left-to-right. They do not apply RTL language rules, so assigning Arabic, Persian, Hebrew, Urdu, or Pashto text directly produces broken output:

- Characters appear in **reversed order**
- **Letter shaping** is ignored, Arabic/Persian letters appear disconnected or in the wrong form
- **Mixed text** (e.g. a Persian sentence containing an English word) loses its logical order
- **Word wrapping** splits words at wrong boundaries
- **Rich text tags** get mangled when characters are reordered

Without RTL processing, text appears reversed and characters are disconnected:

| Language | Input                            | Without RTL (broken output)      |
| -------- | -------------------------------- | -------------------------------- |
| Persian  | `سلاح خود را انتخاب کنید`        | `ﺪﻴﻨﻛ ﺏﺎﺨﺘﻧﺍ ﺍﺭ ﺩﻮﺧ ﺡﻼﺳ`         |
| Arabic   | `تم هزيمة العدو! المستوى التالي` | `ﻲﻟﺎﺘﻟﺍ ﻯﻮﺘﺴﻤﻟﺍ !ﻭﺪﻌﻟﺍ ﺔﻤﻳﺰﻫ ﻢﺗ` |
| Hebrew   | `המשחק הסתיים! שחקן אחד ניצח`    | `חצינ דחא ןקחש !םייתסה קחשמה`    |
| Urdu     | `دشمن شکست کھا گیا! اگلا مرحلہ`  | `ﮧﻠﺣﺮﻣ ﻼﮔﺍ !ﺎﻴﮔ ﺎﮭﻛ ﺖﺴﻜﺷ ﻦﻤﺷﺩ`   |
| Pashto   | `دښمن مات شو! بل پړاو`           | `ﻭﺎﺮﭘ ﻞﺑ !ﻮﺷ ﺕﺎﻣ ﻦﻤښﺩ`           |

Any game or application targeting RTL-language audiences cannot simply assign text directly, it will be unreadable.

## Installation

1. Purchase and import the RTL plugin from the [Unity Asset Store](http://u3d.as/3kv)
2. The plugin DLL will be added to your project
3. Add `using RTL;` to any script that needs RTL conversion

---

## Usage

### Basic Conversion (No Word Wrap)

For single-line text or overflow format, use `RTLService.Convert()`:

```csharp
using RTL;
using UnityEngine.UI;

public class MyUI : MonoBehaviour
{
    public Text label;

    void Start()
    {
        // "Enemy defeated! You earned +50 points."
        string rtlText = "دشمن شکست خورد! +۵۰ امتیاز کسب کردید.";
        label.text = RTLService.Convert(rtlText);
    }
}
```

This also works with **TextMeshPro**:

```csharp
using RTL;
using TMPro;

public class MyUI : MonoBehaviour
{
    public TextMeshProUGUI label;

    void Start()
    {
        // "New weapon unlocked. Are you ready to fight?"
        string rtlText = "سلاح جدید باز شد. آماده نبرد هستید؟";
        label.text = RTLService.Convert(rtlText);
    }
}
```

#### Convert Parameters

```csharp
RTLService.Convert(
    string input,
    NumberFormat numberFormat = NumberFormat.Context,
    ConvertDirection convertDirection = ConvertDirection.Forward,
    bool richText = true,
    BaseDirection baseDirection = BaseDirection.Auto
);
```

| Parameter          | Required | Default                    | Description                                                                                                                                                                                                                                                                                                      |
| ------------------ | -------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input`            | Yes      | —                          | Raw RTL string to convert. This is the raw text as it would appear in your source or database. The plugin handles shaping and reordering.                                                                                                                                                                        |
| `numberFormat`     | No       | `NumberFormat.Context`     | Controls how digits are rendered in the output. `Context` keeps digits exactly as they appear in the input. `Arabic` forces Eastern Arabic numerals (٠١٢٣٤٥٦٧٨٩), `Farsi` forces Farsi numerals (۰۱۲۳۴۵۶۷۸۹), and `English` forces Western digits (0123456789).                                                  |
| `convertDirection` | No       | `ConvertDirection.Forward` | `Forward` produces the natural visual RTL reading order and is correct for the vast majority of use cases. Use `Backward` only for specific UI layouts that already reverse character order (e.g. custom text meshes).                                                                                           |
| `richText`         | No       | `true`                     | When `true`, Unity rich text tags such as `<color>`, `<b>`, `<i>`, and `<size>` are detected, extracted, and reattached after conversion so they survive the reordering step intact. Set to `false` when your input is plain text and angle-bracket characters should be treated as literals rather than markup. |
| `baseDirection`    | No       | `BaseDirection.Auto`       | Controls base paragraph direction for mixed LTR/RTL text. `Auto` detects direction from the first alphabetic character (recommended default for most UI strings). `ForceRTL` treats mixed content as RTL when any RTL text is present, which is useful for RTL-first paragraphs that start with an English word or token. |

> **Note:** If an unexpected error occurs internally, `Convert` returns an empty string `""` rather than throwing an exception. Always check that the result is non-empty before assigning it to a text component if a fallback is important.

#### BaseDirection In Mixed Text

Use `baseDirection` when your sentence mixes English and RTL text and you want to control overall block order.

- `BaseDirection.Auto` (default): best for general-purpose content. Direction follows the first alphabetic character, so LTR-starting mixed text keeps LTR block order.
- `BaseDirection.ForceRTL`: use this when your UI is RTL-first and mixed text should still read as an RTL paragraph, even if it starts with LTR content.

```csharp
// Example: force RTL paragraph behavior for mixed English/Arabic text.
label.text = RTLService.Convert(
    "Version 2.0 تم التفعيل بنجاح",
    baseDirection: BaseDirection.ForceRTL
);
```

---

### Word Wrap Conversion

When your text element has a fixed width and wraps, use `RTLService.ConvertWordWrap()` so line breaks land correctly for RTL text.

#### Option A: Using `UnityFontInfo`

Provide the font, size, and style so the plugin can measure exact glyph widths:

```csharp
using RTL;
using UnityEngine;
using UnityEngine.UI;

public class MyUI : MonoBehaviour
{
    public Text rtlLabel;
    public Font font;

    void UpdateText(string input)
    {
        UnityFontInfo fontInfo = new UnityFontInfo
        {
            Font = font,
            FontSize = rtlLabel.fontSize,
            FontStyle = rtlLabel.fontStyle,
        };

        float width = rtlLabel.rectTransform.rect.width;
        rtlLabel.text = RTLService.ConvertWordWrap(input, width, fontInfo);
    }
}
```

`UnityFontInfo` properties:

| Property      | Required | Default            | Type                    | Description                                                                                                  |
| ------------- | -------- | ------------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| `Font`        | Yes      | —                  | `UnityEngine.Font`      | The font asset that is assigned to your `Text` component.                                                    |
| `FontSize`    | Yes      | —                  | `int`                   | Font size in points. Must match the `fontSize` of your `Text` component for accurate line-break calculation. |
| `FontStyle`   | No       | `FontStyle.Normal` | `UnityEngine.FontStyle` | `Normal`, `Bold`, `Italic`, or `BoldAndItalic`. Affects glyph metrics; should match your component's style.  |
| `ScaleFactor` | No       | `1`                | `float`                 | Canvas scale factor. Pass `canvas.scaleFactor` to account for high-DPI or scaled canvases.                   |
| `LineSpacing` | No       | `1`                | `float`                 | Line spacing multiplier. Pass `text.lineSpacing` if your component uses a non-default value.                 |

#### Option B: Using `TextGenerationSettings`

For more precise control over how text is generated and wrapped, pass a `TextGenerationSettings` object and configure the layout parameters directly.

```csharp
using RTL;
using UnityEngine;
using UnityEngine.UI;

public class MyUI : MonoBehaviour
{
    public Text rtlLabel;

    void UpdateText(string input)
    {
        float width = rtlLabel.rectTransform.rect.width;

        var settings = new TextGenerationSettings
        {
            font = rtlLabel.font,
            fontSize = rtlLabel.fontSize,
            fontStyle = FontStyle.Normal,
            scaleFactor = rtlLabel.canvas.scaleFactor,
            lineSpacing = rtlLabel.lineSpacing,
            richText = true,
            horizontalOverflow = HorizontalWrapMode.Wrap,
            verticalOverflow = VerticalWrapMode.Overflow,
            generationExtents = new Vector2(width, rtlLabel.rectTransform.rect.height),
            generateOutOfBounds = true,
        };

        rtlLabel.text = RTLService.ConvertWordWrap(input, settings);
    }
}
```

#### ConvertWordWrap Parameters

Both overloads accept the same optional parameters after their font/settings arguments:

| Parameter          | Required | Default                    | Description                                                                                                                                                                                                |
| ------------------ | -------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input`            | Yes      | —                          | The raw input string to convert.                                                                                                                                                                           |
| `elementWidth`     | Yes      | —                          | Width of the text container in pixels. Used by the word-wrap algorithm to decide where to insert line breaks. Pass `rectTransform.rect.width`. [_Option A Only_]                                           |
| `fontInfo`         | No       | `null`                     | A `UnityFontInfo` object describing the font, size, style, scale factor, and line spacing used to measure glyph widths. When `null`, character widths fall back to a heuristic estimate. [_Option A Only_] |
| `settings`         | Yes      | —                          | A Unity `TextGenerationSettings` struct. Provides all layout parameters (font, size, extents, overflow mode, etc.) in one place for pixel-identical line-break matching with `UI.Text`. [_Option B Only_]  |
| `numberFormat`     | No       | `NumberFormat.Context`     | Controls how digits are rendered. `Context` keeps digits as-is; `Arabic` forces ٠١٢٣٤٥٦٧٨٩; `Farsi` forces ۰۱۲۳۴۵۶۷۸۹; `English` forces 0–9.                                                               |
| `convertDirection` | No       | `ConvertDirection.Forward` | `Forward` produces natural RTL visual order. Use `Backward` only for layouts that already invert character order.                                                                                          |
| `richText`         | No       | `true`                     | When `true`, Unity rich text tags (`<color>`, `<b>`, `<i>`, `<size>`, etc.) are preserved through the conversion. Set to `false` to treat the input as plain text with no markup.                          |
| `baseDirection`    | No       | `BaseDirection.Auto`       | Same behavior as `Convert`. Use `ForceRTL` to keep paragraph-level RTL block order in mixed LTR/RTL wrapped text, especially when lines begin with English tokens.                                            |

---

### Rich Text Example

Rich text tags are preserved through conversion:

```csharp
// "Warning: The enemy will attack in <red>30 seconds</red>. Equip your shield!"
string input = "هشدار: دشمن در <color=#ff0000>۳۰ ثانیه</color> دیگر حمله می‌کند. سپر خود را آماده کنید!";
label.text = RTLService.Convert(input);
```

This renders "۳۰ ثانیه" (30 seconds) in red while keeping the full sentence in correct RTL order.

---

### Number Formatting

Control how digits appear in converted text:

```csharp
// Preserve original digit style
label.text = RTLService.Convert(input, NumberFormat.Context);

// Force Eastern Arabic digits (٠١٢٣٤٥٦٧٨٩)
label.text = RTLService.Convert(input, NumberFormat.Arabic);

// Force Farsi digits (۰۱۲۳۴۵۶۷۸۹)
label.text = RTLService.Convert(input, NumberFormat.Farsi);

// Force Western digits (0123456789)
label.text = RTLService.Convert(input, NumberFormat.English);
```

---

## Language Examples

The plugin works the same way regardless of language. Pass any RTL text to `RTLService.Convert()` and it handles script-specific letter shaping and reordering automatically.

**Arabic**

```csharp
// "Enemy eliminated. Checkpoint reached."
label.text = RTLService.Convert("تم تصفية العدو. تم بلوغ نقطة التفتيش.");
```

**Hebrew**

```csharp
// "Mission complete. Collect your reward."
label.text = RTLService.Convert("המשימה הושלמה. אסוף את הפרס שלך.");
```

**Urdu**

```csharp
// "Level up! New power unlocked."
label.text = RTLService.Convert("لیول اپ! نئی طاقت فعال ہوگئی۔");
```

**Pashto**

```csharp
// "New quest has begun. Good luck, warrior."
label.text = RTLService.Convert("نوی دنده پیل شوه. بریالي اوسئ، جنګیالیه.");
```

**Persian (Farsi)**

```csharp
// "Choose your hero to enter the battle."
label.text = RTLService.Convert("قهرمان خود را برای ورود به نبرد انتخاب کنید.");
```

---

## Requirements

- **Unity 6** or later
- **.NET Standard 2.1**

## Support

For questions, issues, or feedback, please contact us at:  
[rtl@femoli.com](mailto:rtl@femoli.com)

## Unity Asset Store

This package is available for purchase on the Unity Asset Store:  
http://u3d.as/3kv

---

_©Femoli RTL - Right to Left (RTL) Text Plugin for Unity_
