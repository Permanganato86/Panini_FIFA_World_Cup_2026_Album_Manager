# 📖 Setup walkthrough

First time using this notebook? Follow these steps in order.

## 1. Get an OpenRouter API key

OpenRouter is a service that gives you access to many AI vision models (Anthropic Claude, Google Gemini, etc.) through one API.

1. Go to https://openrouter.ai/
2. Sign up (free, no credit card needed yet)
3. Go to https://openrouter.ai/keys
4. Click **Create Key**, give it a name like "Panini Album", copy the key (starts with `sk-or-...`)
5. **Save the key somewhere safe** — you'll paste it into the notebook every session

⚠️ **Keep your key private.** Anyone with it can spend your credit. The notebook never saves it to disk; it's prompted fresh each session.

## 2. Add credit

1. Go to https://openrouter.ai/credits
2. Add credit via card or crypto. **A few USD is enough for a full album run.**
3. Check current per-model pricing at https://openrouter.ai/models

## 3. Open the notebook in Colab

1. Click the **Open in Colab** badge in the [README](../README.md), OR go directly to:
   ```
   https://colab.research.google.com/github/Permanganato86/panini-wc26/blob/main/AlbumPanini2026_Manager.ipynb
   ```
2. In Colab, click **File → Save a copy in Drive**
3. Now you have your own editable copy in `My Drive → Colab Notebooks/`

## 4. Run cell 1️⃣ (Setup)

This cell:
- Installs Python packages (`openai`, `pandas`, `pillow-heif`, etc.)
- Mounts your Google Drive

When prompted, **allow Drive access**. The notebook needs to read photos from and write the inventory to your Drive.

## 5. Pick a model and paste your API key

In cell 2️⃣ you'll see a line setting the model:

```python
MODEL = "anthropic/claude-haiku-4.5"
```

**Pick whichever model fits your budget and accuracy needs.** Some options to consider:

| Model | Notes |
|---|---|
| `google/gemini-2.5-flash` | Fast and economical — used in the real-world test that achieved ~90% accuracy |
| `google/gemini-2.5-pro` | Stronger Gemini variant |
| `anthropic/claude-haiku-4.5` | Anthropic's fast model |
| `anthropic/claude-sonnet-4.5` | Anthropic's higher-end model |

Browse all available vision-capable models and their current pricing at https://openrouter.ai/models.

You can change `MODEL` and re-run the processing cells at any time — the `ProcessingLog` sheet records which model was used per photo, so you can A/B test models on your own album without losing the audit trail.

When you run the cell, a password-style prompt appears. **Paste your API key and press Enter.** The key is held in memory only — it disappears when the Colab runtime resets.

## 6. Run cell 3️⃣ (folder setup)

This creates the folder structure in your Drive:

```
MyDrive/PaniniAlbum2026/
├── album_photos/                       ← you'll put photos here
├── special_page_photos/
├── duplicate_photos/
├── processed_album/                    ← processed photos move here
├── processed_special_pages/
├── processed_duplicates/
├── backups/                            ← auto-created backups
└── logs/                               ← raw AI responses
```

## 7. Run cell 4️⃣ (create master inventory)

This creates `MyDrive/PaniniAlbum2026/album_inventory.xlsx` with all 994 stickers pre-populated:

- 960 country stickers (48 countries × 20)
- 20 FWC special stickers (Roll of Honour, emblems, ball, host emblems, World Cup history)
- 14 Coca-Cola stickers

FWC and Coca-Cola names + page numbers are **hardcoded**, so even before processing any photo, your inventory already has correct player names for those sections.

## 8. Take photos and upload them

See [taking_photos.md](taking_photos.md) for the photo-taking guide.

Upload them to the right folder in your Drive:

| Photo type | Drive folder |
|---|---|
| Country pages (20 stickers each) | `album_photos/` |
| FWC + Coca-Cola pages | `special_page_photos/` |
| Backs of duplicate stickers (≤28 per photo) | `duplicate_photos/` |

## 9. Run the processing cells

In order:
- **Cell 5️⃣ (preview rotation)** — optional sanity check. Verifies your photos are right-side up after the 180° auto-rotation.
- **Cell 6️⃣ (country pages)** — processes everything in `album_photos/`
- **Cell 7️⃣ (special pages)** — processes everything in `special_page_photos/`
- **Cell 8️⃣ (duplicates)** — processes everything in `duplicate_photos/`

Each cell:
1. Sends each photo to the AI
2. Validates the response
3. Updates the Excel atomically (with a backup first)
4. Moves the photo to `processed_*/` on success, or leaves it for retry on failure

## 10. Run reports

**Cell 9️⃣** generates the report sheets in the Excel:
- `Missing` — every sticker you don't have yet
- `Duplicates` — every sticker you have extras of
- `Stats` — totals and percentages
- `ByCountry` — per-country progress

**Cell 🔟** prints trade-ready lists you can copy-paste into WhatsApp / Telegram groups.

## 11. Manual cleanup (if needed)

If the AI got something wrong (see [troubleshooting.md](troubleshooting.md) for common cases), use cell 1️⃣1️⃣ helpers:

```python
mark_owned('ARG 1', False)
set_duplicates('FWC 10', 3)
set_player('ARG 20', 'Lionel Messi')
inspect_log('IMG_0733.HEIC', phase='album')
self_check()
```

---

## When you come back later

Subsequent sessions are simple:
1. Open the notebook from your Drive
2. Run cells 1️⃣ and 2️⃣ (paste key again — it's not saved)
3. Skip cell 4️⃣ (your inventory already exists)
4. Upload new photos and run the relevant processing cell
5. Run reports (cell 9️⃣) when done

Your progress is preserved in `album_inventory.xlsx`. Backups in `backups/` let you roll back any session.
