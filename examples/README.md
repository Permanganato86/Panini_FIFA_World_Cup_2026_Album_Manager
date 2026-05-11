# Example photos

This folder is for example photos demonstrating what good input looks like. They're useful for:

- New users seeing what a properly-shot page looks like before they shoot their own
- Documentation screenshots
- Testing the notebook without needing the physical album

## What to put here (after forking)

If you want to add examples to your fork:

1. `country_page_example.jpg` — a well-lit photo of one country page (slight blur or watermarking the stickers is fine if you want to anonymize)
2. `special_page_example.jpg` — a photo of an FWC or Coca-Cola page
3. `duplicates_example.jpg` — a photo of 10–20 sticker backs laid out flat

**Don't commit photos of your real, unaltered, complete album** — those are personal and likely subject to Panini's copyright on the artwork. If you publish examples, blur or pixelate the sticker contents and keep only the page layout / code areas visible.

## .gitignore note

The repo's `.gitignore` blocks all `.jpg`/`.jpeg`/`.png` files by default to prevent accidentally committing your personal album photos. There's an exception for `examples/*.jpg` etc., so files placed here CAN be committed if you want.
