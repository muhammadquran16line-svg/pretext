# canvas measureText() returns incorrect (inflated) width for emoji at small font sizes

## Summary

`CanvasRenderingContext2D.measureText()` returns widths for emoji characters that are larger than what the DOM renders. The discrepancy is 2-5px at font sizes below 24px and disappears at >=24px. DOM always renders emoji at exactly the font size (width = fontSize). The issue affects all emoji, all font families, and both `<canvas>` and `OffscreenCanvas`.

Safari does not have this issue — canvas and DOM agree on emoji widths at all sizes.

## Steps to reproduce

1. Open the attached `emoji-measuretext-repro.html`
2. Observe the table comparing canvas `measureText('😀').width` vs `span.getBoundingClientRect().width`

## Expected result

Canvas `measureText()` and DOM `getBoundingClientRect()` return the same width for the same emoji at the same font size. (This is what Safari does.)

## Actual result (Chrome on macOS, DPR=2)

```
fontSize  canvas  DOM     diff
  10px     13.0    11.0   +2.0
  12px     15.0    12.0   +3.0
  14px     18.0    14.0   +4.0
  15px     19.0    15.0   +4.0
  16px     20.0    16.0   +4.0
  18px     21.0    18.0   +3.0
  20px     22.0    20.0   +2.0
  24px     24.0    24.0   +0.0
  28px     28.0    28.0   +0.0
  32px     32.0    32.0   +0.0
```

## Additional observations

- Same result with `OffscreenCanvas`
- Same result with DPR emulated at 1x (DevTools device mode)
- Same diff for all 74 emoji tested (basic, ZWJ sequences, flags, skin tones, keycaps)
- Same diff across all font families (sans-serif, serif, monospace, named fonts)
- The DOM value always equals the font size (emoji width = fontSize)
- HarfBuzz and opentype.js (reading the Apple Color Emoji font directly) also give width = fontSize, matching DOM
- Firefox has the same class of bug but with different diff values (+1 to +5, converges at 28px instead of 24px)
- Safari canvas matches DOM perfectly at all sizes

## Impact

This breaks canvas-based text measurement libraries that need to predict line breaks matching DOM rendering. Text blocks containing emoji get measured as wider than they actually render, causing premature line breaks and incorrect height calculations.

## Version

Chrome 134 on macOS 15.3 (Apple Silicon)
