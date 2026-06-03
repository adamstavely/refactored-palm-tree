# Interop brand assets

The mark is two interlocking rounded-square links rotated 22.5 degrees. It reads as connected capabilities held in one place, which is the registry's job.

## Color

| Token | Hex | Use |
|---|---|---|
| Brand blue (primary) | `#3b5bff` | Primary ring, primary mark color, icon tile background |
| Brand blue light (secondary) | `#8a9dff` | Secondary ring, the second tone in the mark |
| Ink | `#0e1330` | Wordmark on light backgrounds |
| White | `#ffffff` | Wordmark and mark knockout on the brand tile |
| Knockout light | `#c9d3ff` | Secondary ring inside the brand-blue tile |
| Tile light | `#eaeeff` | Light icon tile background |

The mark color (`#3b5bff` + `#8a9dff`) holds up on both light and dark backgrounds, so the mark itself does not need a dark variant. Only the wordmark color swaps: ink on light, white on dark.

## Files

| File | What it is |
|---|---|
| `interop-mark.svg` | Standalone mark, transparent background |
| `interop-lockup-horizontal.svg` | Mark + wordmark, for light backgrounds |
| `interop-lockup-horizontal-dark.svg` | Mark + wordmark, white text for dark backgrounds |
| `interop-lockup-stacked.svg` | Mark above wordmark, for light backgrounds |
| `interop-icon-app.svg` | App icon / favicon, brand-blue tile with white mark |
| `interop-icon-app-light.svg` | App icon, light tile with blue mark |

## Notes

The wordmark uses a system sans stack so it renders without a font install. For production, convert the wordmark text to outlines (or set a licensed brand font) so it renders identically everywhere. The mark contains no text and is already pure vector, so it is safe as-is.

Clear space: keep padding around the mark equal to one ring's corner radius. Do not recolor the mark outside the tokens above, do not add a third tone, and do not change the 22.5 degree angle.

Minimum sizes: mark down to 16px, full horizontal lockup down to about 96px wide before the wordmark gets tight.
