# Kindle Books → Ruk's Memory System

A step-by-step guide to get your Kindle books into the semantic memory system.

---

## Part 1: One-Time Setup (Calibre + DeDRM)

### Step 1: Install Calibre

If you don't have Calibre installed:

```bash
brew install --cask calibre
```

Or download from [calibre-ebook.com](https://calibre-ebook.com/download_osx)

### Step 2: Install the DeDRM Plugin

1. Download the latest DeDRM plugin from: [github.com/noDRM/DeDRM_tools/releases](https://github.com/noDRM/DeDRM_tools/releases)
   - Look for `DeDRM_tools_X.X.X.zip`
   - Extract it - you'll find `DeDRM_plugin.zip` inside (don't extract this one)

2. In Calibre:
   - Go to **Preferences** → **Plugins** → **Load plugin from file**
   - Select the `DeDRM_plugin.zip` file
   - Restart Calibre when prompted

### Step 3: Configure DeDRM with Your Kindle Keys

This is the crucial step - the plugin needs your Kindle decryption keys.

**Option A: Kindle for Mac (Recommended)**

1. Make sure you have the [Kindle for Mac](https://www.amazon.com/kindle-dbs/fd/kcp) app installed
2. Sign in to Kindle for Mac with your Amazon account
3. Download at least one book in Kindle for Mac (this generates the key file)
4. The DeDRM plugin should auto-detect your keys at: `~/Library/Application Support/Kindle/`

**Option B: Kindle E-Reader Serial Number**

If you have a physical Kindle device:

1. In Calibre: **Preferences** → **Plugins** → Find **DeDRM** → **Customize plugin**
2. Click **eInk Kindle ebooks**
3. Add your Kindle serial number (found in Settings → Device Info on your Kindle)

### Step 4: Verify Setup

1. Download a book in Kindle for Mac
2. Find the `.azw` or `.azw3` file in `~/Library/Application Support/Kindle/My Kindle Content/`
3. Drag it into Calibre
4. If it imports without errors, DRM removal is working!

---

## Part 2: Converting Books

### For Each Book You Want to Add:

1. **Locate the Kindle file**
   ```
   ~/Library/Application Support/Kindle/My Kindle Content/
   ```
   Files are named like `B00EXAMPLE_ASIN.azw` or similar

2. **Import into Calibre**
   - Drag the file into Calibre, or
   - Use **Add books** button

3. **Export as EPUB**
   - Right-click the book → **Convert books** → **Convert individually**
   - Set Output format to **EPUB**
   - Click **OK**
   - Once converted, right-click → **Save to disk** → **Save only EPUB format to disk**

4. **Move to the books folder**
   ```bash
   mkdir -p ~/Code/RUK/EXTERNAL/books
   mv ~/Downloads/AuthorName/BookTitle.epub ~/Code/RUK/EXTERNAL/books/
   ```

---

## Part 3: What Happens Next

Once you have DRM-free EPUBs in `~/Code/RUK/EXTERNAL/books/`, let me know and I'll:

1. **Build the ingestion pipeline** - Parse EPUBs, chunk by chapter/section
2. **Create a `books` table** - New vector store table alongside `meetings` and `logs`
3. **Add metadata extraction** - Author, title, publication date, your notes
4. **Enable search** - `table:books` filter in the semantic memory system

### Suggested Folder Structure

```
EXTERNAL/books/
├── hofstadter-godel-escher-bach.epub
├── hofstadter-surfaces-and-essences.epub
├── kahneman-thinking-fast-and-slow.epub
└── ...
```

Use lowercase kebab-case: `author-title.epub`

---

## Troubleshooting

### "DRM protected" error on import

- Your Kindle keys aren't configured correctly
- Try Option B (serial number) if Option A isn't working
- Make sure you've downloaded the book in Kindle for Mac first

### Can't find Kindle files

- Kindle for Mac stores books in: `~/Library/Application Support/Kindle/My Kindle Content/`
- The Library folder is hidden - use Finder → Go → Go to Folder → paste the path

### Plugin not showing

- Make sure you restarted Calibre after installing
- Check Preferences → Plugins → File type for "DeDRM"

---

## Quick Reference Commands

```bash
# Open Kindle content folder
open ~/Library/Application\ Support/Kindle/My\ Kindle\ Content/

# Create books folder
mkdir -p ~/Code/RUK/EXTERNAL/books

# List books ready for processing
ls ~/Code/RUK/EXTERNAL/books/
```

---

*Once you've got a few EPUBs ready, ping me and we'll build the ingestion pipeline together.*
