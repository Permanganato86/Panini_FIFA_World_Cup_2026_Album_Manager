# 📚 Panini FIFA World Cup 2026 Album Manager

> AI-powered inventory tracker for your Panini sticker album. Built for Google Colab.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Permanganato86/Panini_FIFA_World_Cup_2026_Album_Manager/blob/main/AlbumPanini2026_Manager.ipynb)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Take photos of your album pages → an AI vision model reads them → your inventory is updated automatically in a Google Sheets-compatible Excel file. Generates trade lists, tracks duplicates, and audits its own work.

---

## ⚡ Quick start

1. Click the **Open in Colab** badge above.
2. In Colab, click **File → Save a copy in Drive** to make it your own. *(Without this, you cannot save changes.)*
3. Get an OpenRouter API key at https://openrouter.ai/keys and load a few USD of credit.
4. Run cells in order. Cell 1️⃣ asks Drive permission, cell 2️⃣ asks for your API key, cell 4️⃣ creates the master inventory.
5. Upload photos to the right folder in your Drive, re-run the matching cell, profit.

> ⚠️ **Before processing photos:** check the rotation setting. The notebook defaults to `ROTATION_ALBUM = 180` because the test photos came out upside down (phone held over a flat album). **If your photos are already right-side up, change this to `0` in cell 5️⃣.** Details in [docs/taking_photos.md](docs/taking_photos.md#about-rotation--important-read-this).

📖 **Full setup walkthrough:** [docs/setup.md](docs/setup.md)

---

## 🎯 What this does

| Feature | Description |
|---|---|
| 🤖 **AI vision** | Reads your album pages via OpenRouter (Gemini, Claude, etc.) |
| 📊 **Excel master inventory** | All 994 stickers tracked: owned, duplicates, player names, page numbers |
| 🔁 **Trade lists** | Copy-paste-ready WhatsApp/Telegram lists of duplicates and missing |
| 🛡️ **Auto-backups** | Every Excel write creates a timestamped backup. Roll back any time. |
| 🚦 **Strict validation** | Rejects bad photos (incomplete pages, AI hallucinations, >28 duplicates) rather than corrupting your data |
| 🔍 **Audit trail** | Every AI call's raw response saved as JSON. A `ProcessingLog` sheet tracks every run. |
| 🔐 **Hash-based dedup** | The same duplicate photo can't be processed twice, even renamed |
| 🌟 **Holographic detection** | The prompt knows FWC stickers are holographic and uses that as the primary filled/empty signal |
| 📱 **HEIC native** | iPhone photos work directly, no conversion needed |
| 🔄 **Auto-rotation** | Photos taken phone-over-album come upside-down — handled automatically |

---

## 📈 Expected accuracy

**Around 90%** of stickers are detected correctly on the first pass. In a real-world test of a full 994-sticker album using `google/gemini-2.5-flash`:

- ✅ 48 country pages processed, 0 rejected by validation
- ✅ All special pages (FWC + Coca-Cola) processed
- ⚠️ ~5 false positives on **crest detection** (slot 1): the AI sometimes calls a crest "filled" when the holographic foil reflects light off an empty slot
- ⚠️ 1 misread on a page

**Use [Manual corrections](#-manual-corrections) (cell 1️⃣1️⃣) to fix individual mistakes:**

```python
mark_owned('ARG 1', False)              # un-mark a wrongly-detected crest
set_duplicates('FWC 10', 3)             # adjust duplicate count
set_player('ARG 20', 'Lionel Messi')    # fix a player name
```

**The time saved versus entering 994 stickers manually is enormous** — even with ~10% manual cleanup, you go from hours of typing to minutes of corrections.

**Tips to push accuracy higher:**
- Try different OpenRouter models for the country-page pass — see the comparison at https://openrouter.ai/models
- Take photos with even lighting, minimal glare on holographic stickers
- After processing, run cell 9️⃣ and review the `ByCountry` sheet — countries at 100% are reliable, countries with weird gaps deserve a second look

---

## 💰 Cost

Costs depend entirely on the model you choose. Check current pricing at https://openrouter.ai/models. Generally, a full album processing run uses ~80 API calls (48 countries + ~5 special pages + ~30 duplicate photos), so even with mid-tier vision models you're looking at a small fraction of a dollar to a few dollars total.

The `ProcessingLog` sheet records which model was used per photo, so you can experiment freely without losing the audit trail.

---

## 📂 What gets created in your Drive

```
MyDrive/PaniniAlbum2026/
├── album_inventory.xlsx                ← master inventory
├── album_photos/                       ← upload country pages here
├── special_page_photos/                ← upload FWC + Coca-Cola pages here
├── duplicate_photos/                   ← upload back-of-duplicates here
├── processed_album/                    ← moved here after successful processing
├── processed_special_pages/
├── processed_duplicates/
├── backups/                            ← auto-rotated, 100 most recent kept
└── logs/                               ← raw AI responses per photo
```

---

## 🛡️ Safety features (in plain English)

- The AI **cannot un-mark stickers you already own** — only manual `mark_owned(code, False)` can do that.
- Photos are **moved to `processed_*/` only after** the Excel write succeeded. Failures leave the photo for retry.
- Every Excel write creates a **timestamped backup** before touching the file. The last 100 backups are kept.
- Country pages with <20 stickers, duplicate slots, mixed countries, or unknown codes are **rejected**, not partially applied.
- Duplicate photos with >28 codes detected (AI hallucination) are **rejected entirely** — no guessing which 28 were real.
- Duplicate photos with >25% unknown codes are **rejected** as likely bad reads.
- The string `"false"` is parsed correctly as `False` — fixes a Python footgun that could silently mark empty slots as owned.

---

## 📋 Requirements

- Google account (for Colab + Drive)
- OpenRouter account with a few USD of credit ([sign up](https://openrouter.ai))
- Your Panini FIFA World Cup 2026 album + a phone

That's it. No local setup, no Python install, no dependencies on your machine.

---

## 📚 More docs

- 📖 [Setup walkthrough](docs/setup.md) — getting your API key, mounting Drive, picking a model
- 📸 [How to take good photos](docs/taking_photos.md) — lighting, rotation, the 28-duplicate rule
- 🚑 [Troubleshooting](docs/troubleshooting.md) — what to do when a photo gets rejected

---

## ⚖️ License

Licensed under the **Apache License, Version 2.0** — see [LICENSE](LICENSE) and [NOTICE](NOTICE). Free to use, modify, fork, share, including for commercial purposes. Includes explicit patent grant from contributors. No warranty.

---

## 🙋 Contributing

Found a bug, want to add a feature, or have a better prompt? Open an issue or send a pull request. This is a personal project but community improvements are welcome.

**Some good first contributions:**
- Verify page numbers for countries against your physical album and submit corrections
- Translate `docs/` to your language
- Improve prompts for edge cases (poor lighting, weird angles)
- Add support for other Panini albums (the structure is generic enough)
