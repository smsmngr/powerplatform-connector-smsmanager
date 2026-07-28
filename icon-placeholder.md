# icon.png (placeholder)

This connector needs a real `icon.png` before it can be imported or submitted for
certification. It is **not** included here because a binary image cannot be safely
fabricated as text.

## Requirements

Microsoft Power Platform custom connector icon requirements:

- **Format:** PNG
- **Aspect ratio:** 1:1 (square)
- **Recommended size:** 230 x 230 px (minimum 100 x 100 px)
- **Max file size:** 1 MB
- **Background color:** SmsManager crimson `#A81943` (this must match the
  `iconBrandColor` in `apiProperties.json`)
- **Foreground:** the white SmsManager logo mark, centered, with comfortable padding

## How to produce it

1. Export the SmsManager logo mark as white on a transparent background.
2. Place it centered on a solid `#A81943` square canvas (230 x 230 px).
3. Export as `icon.png` into this directory.
4. Confirm `settings.json` references `"icon": "icon.png"`.

Once `icon.png` exists, delete this placeholder file.
