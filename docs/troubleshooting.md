# 🚑 Troubleshooting

Common issues and how to fix them.

## A photo got rejected

When a photo fails validation, it **stays in its original folder** (not moved to `processed_*/`). The error reason is logged in:

1. The cell's output (look for `🚫 REJECTED:` messages)
2. The `ProcessingLog` sheet in the Excel
3. The raw JSON file in `logs/`

### Common rejection reasons (country pages)

| Error | What it means | Fix |
|---|---|---|
| `expected exactly 20 stickers, got N` | AI didn't return 20 stickers | Re-shoot with better lighting. Try `claude-sonnet-4.5` if you keep getting this. |
| `expected 1 country, got [ARG, BRA]` | AI thought it saw 2 countries | Photo includes part of an adjacent page. Re-shoot one country at a time. |
| `missing slots [7]; duplicate slots [1]` | AI missed slot 7 and saw slot 1 twice | Usually a misread. Re-shoot or use sonnet. |
| `unknown codes in inventory: ['BHA1']` | AI invented a country code that doesn't exist | Misread country letters (BRA → BHA). Re-shoot. |
| `unparseable JSON` | AI's response wasn't valid JSON | Rare. Re-run the cell to retry. |

### Common rejection reasons (special pages)

| Error | Fix |
|---|---|
| `invalid special code: ARG1` | Country code leaked into special page processing. Make sure you uploaded the FWC/CC photo to `special_page_photos/`, NOT `album_photos/` |
| `duplicate codes in same photo: ['FWC0']` | AI saw the same sticker twice. Re-shoot. |
| `no stickers detected` | AI returned an empty list. Re-shoot. |

### Common rejection reasons (duplicates)

| Error | Fix |
|---|---|
| `REJECTED: N codes > 28 (likely hallucination)` | You have >28 stickers in the photo, OR the AI is hallucinating. Split into multiple photos with ≤28 each. |
| `REJECTED: N/M unknown codes (>25% threshold)` | Too many of the detected codes don't exist in the inventory. Likely a bad reading. Re-shoot or try a more accurate model. |
| `REJECTED: no readable codes detected` | AI returned no codes. Photo is too blurry/dark/glare-y. Re-shoot. |

## What if the AI marks a crest as filled when it isn't?

This is the most common ~10% error. Holographic stickers (slot 1 of every country) reflect light strangely, and the AI sometimes interprets the empty slot's reflection as a filled sticker.

**Fix manually in cell 1️⃣1️⃣:**

```python
mark_owned('ARG 1', False)
mark_owned('BRA 1', False)
```

After cleanup, re-run cell 9️⃣ to regenerate the reports.

## How do I know what the AI actually saw?

Every AI call's raw response is saved as a JSON file in `MyDrive/PaniniAlbum2026/logs/`. Use:

```python
inspect_log('IMG_0733.HEIC', phase='album')
```

Output looks like:
```
📄 Log: /content/drive/MyDrive/PaniniAlbum2026/logs/album_IMG_0733_20260511_193024_123456.json
   Model: anthropic/claude-haiku-4.5
   Timestamp: 2026-05-11T19:30:24.123456

   20 stickers detected:
     ARG1     - filled: True  - Crest
     ARG2     - filled: True  - Emiliano Martínez
     ...
```

Phases are: `'album'`, `'special'`, or `'duplicates'`.

## I messed something up. How do I roll back?

Every Excel write creates a backup first. The 100 most recent are kept.

```python
list_backups()
```

Shows:
```
📦 17 backups in /content/drive/MyDrive/PaniniAlbum2026/backups/ (newest first):
   • album_inventory_20260511_193024_123456_album.xlsx  (245.3 KB)
   • album_inventory_20260511_192758_654321_log.xlsx    (245.1 KB)
   ...
```

Restore one with:
```python
restore_backup('album_inventory_20260511_193024_123456_album.xlsx')
```

The current file is backed up first as `..._before_restore.xlsx` so even the restore is reversible.

## My photo rotation is wrong

After running cell 1️⃣ and 5️⃣, the **preview rotation** cell shows the first photo in `album_photos/` with the configured rotation. If it looks upside-down or sideways:

1. Open cell 5️⃣ (`Vision helpers`)
2. Change `ROTATION_ALBUM` to one of `0`, `90`, `180`, `270`
3. Re-run cells 5️⃣ and 6️⃣ (preview)
4. Repeat until the preview shows the page right-side up

If different photos in the same folder have different orientations, you can rotate one specific file in place:

```python
rotate_file('/content/drive/MyDrive/PaniniAlbum2026/album_photos/IMG_0731.HEIC', degrees=90)
```

## The inventory got corrupted (wrong totals, weird codes)

Run the integrity check:

```python
self_check()
```

Output looks like:
```
🔎 SELF-CHECK RESULTS
✅ Inventory is consistent: 994 stickers, all checks passed.
   Progress: 829/994 owned (83.4%)
   Duplicates: 697
```

If errors are reported, restore from a backup:

```python
list_backups()
restore_backup('album_inventory_20260511_180000_000000_album.xlsx')   # before the bad write
self_check()    # verify
```

## A photo wasn't processed

Check the `ProcessingLog` sheet in `album_inventory.xlsx`. Every photo attempt is logged with:
- `success` (True/False)
- `errors` (what went wrong)
- `log_json` (path to the raw response)

If you don't see your photo there, it might still be in the source folder waiting for the next run. The notebook never auto-processes photos in `processed_*/`.

## The same photo is being processed twice

This shouldn't happen because:
- Photos move to `processed_*/` after success (they're removed from the source folder)
- Duplicate photos are tracked by SHA-256 hash in `PhotoRegistry` — re-uploading the same photo under a different name is detected

If a duplicate photo is being re-processed anyway, check that:
1. The hash is in `PhotoRegistry`
2. The file has actually been moved out of `duplicate_photos/`

To force re-processing a duplicate photo: delete its row from `PhotoRegistry` and move it back to `duplicate_photos/`.

## "Run all" doesn't work as expected

Don't use Colab's "Run all" if you're not sure what's in your photo folders. Run cells one at a time, in order, and check the output of each before moving on. Each processing cell prints a summary at the end:
- ✅ how many stickers were added
- 🚫 how many photos were rejected (with reasons)
- ⚠️ codes not found in the inventory (typically AI misreads)

## I want to start fresh

To wipe everything and start over:

```python
create_initial_inventory(EXCEL_PATH, COUNTRIES, force=True)
```

This **deletes your progress** but creates a `..._before_force.xlsx` backup first. If the backup fails, the operation aborts — so you can't accidentally destroy data.

After this, also:
- Move photos back from `processed_*/` to the source folders if you want to re-process them
- Note that the `PhotoRegistry` is also reset, so previously-seen duplicate photos will be re-counted
