# 📸 How to take good photos

The accuracy of this tool depends mostly on the quality of your photos. Spend 30 seconds reading this — it will save you hours of manual corrections.

## General rules (all photo types)

✅ **Do:**
- Take photos with even, indirect lighting
- Hold the phone parallel to the page (not at an angle)
- Get the entire content in frame, with a small border
- Use HEIC or JPG (any modern phone format works)

❌ **Don't:**
- Shoot with the sun or a lamp directly above (creates glare on holographic stickers)
- Shoot at an angle — perspective distortion confuses the AI
- Zoom in / crop tightly — the AI works better when it sees the whole page
- Apply filters or edit the photos before uploading

## Album page photos

For country pages and special FWC/Coca-Cola pages, take photos like this:

- **One country page per photo** (NOT both sides of a spread)
- The album lying **flat on a table**
- Phone hovering directly above, screen facing down at the page
- The whole page fits in the frame with maybe 1 cm of border around it

📁 Upload to: `MyDrive/PaniniAlbum2026/album_photos/` (countries) or `MyDrive/PaniniAlbum2026/special_page_photos/` (FWC, Coca-Cola)

### About rotation — IMPORTANT, read this

The notebook ships with `ROTATION_ALBUM = 180` because the photos used in development came out **upside down** (when you hold a phone over a flat album and shoot down, the top of the album ends up at the bottom of the image — this is the most common case).

**You need to decide what value to use based on YOUR photos:**

| Your photos look like... | Set `ROTATION_ALBUM` to |
|---|---|
| Upside down (album top is at the bottom of the photo) | `180` ← default, no change needed |
| Already right-side up (album top is at the top of the photo) | `0` ← change this in cell 5️⃣ |
| Rotated 90° clockwise | `270` |
| Rotated 90° counter-clockwise | `90` |

**How to decide:**

1. Open one of your photos in your phone's gallery or Google Drive preview
2. If you can read the country name and player names normally without rotating your head → your photos are right-side up → **change `ROTATION_ALBUM` to `0` in cell 5️⃣**
3. If the text is upside down → keep the default `180`

**How to verify before processing everything:**

After running cell 5️⃣ (vision helpers), the notebook has a **preview cell** that shows you exactly how the first photo will look after rotation is applied. Run it. If the preview shows the page right-side up, you're good. If not, change `ROTATION_ALBUM` and re-run the preview until it looks correct.

⚠️ Getting rotation wrong won't damage your data — modern vision models can read sideways text — but accuracy drops. Spend 30 seconds verifying before processing all 48 country pages.

## Duplicate sticker photos

For the BACKS of duplicate stickers (the side with the printed code):

✅ **Do:**
- Lay stickers **flat on a table**, **no overlapping**, **no stacking**
- **Maximum 28 stickers per photo** — this is a hard limit, photos with more get rejected
- Make sure the white code rectangle is clearly visible on each sticker
- If you have 3 copies of the same sticker, lay them out separately — the AI counts each one

❌ **Don't:**
- Stack stickers on top of each other
- Take a photo of more than 28 at once (split them into multiple photos)
- Place them so close together that the codes touch or overlap

📁 Upload to: `MyDrive/PaniniAlbum2026/duplicate_photos/`

### Why the 28 limit?

If the AI returns more than 28 codes for a photo (when you physically only have ≤28 stickers there), it's hallucinating. Since we can't tell which detected codes are real and which are invented, the notebook **rejects the whole photo**. Better to keep your inventory clean than to add fake duplicates that will haunt your trade lists.

## How many photos do I need?

For a full album:

| Photo type | Quantity | Notes |
|---|---|---|
| Country pages | **48** | One per country |
| Special pages | **~3–5** | Cover FWC sections + Coca-Cola pages |
| Duplicates | **varies** | Split your duplicates into batches of 28 or fewer |

Total: roughly 55–80 photos for a complete first pass.

## Verifying rotation before processing

After uploading, run **cell 5️⃣ (Preview rotation)**. It shows you the first photo from `album_photos/` after rotation is applied. If text on the page reads normally (top of country name at the top of the image), you're good. If it's upside-down or sideways, adjust `ROTATION_ALBUM` in cell 5️⃣ and re-run the preview.

## Photo file types accepted

- ✅ `.HEIC` / `.heic` (iPhone default)
- ✅ `.JPG` / `.jpg` / `.JPEG` / `.jpeg`
- ✅ `.PNG` / `.png`
- ✅ `.webp`

iPhone photos work directly — no conversion needed.

## Photo size

iPhone photos are typically ~4000 pixels on the long side. The notebook automatically downsizes them to 2200 px before sending to the AI. This:

- Reduces API cost (smaller image = fewer tokens)
- Speeds up processing
- Doesn't hurt accuracy — sticker codes are still very readable at this size

You don't need to resize anything before uploading.
