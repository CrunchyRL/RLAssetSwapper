# RL UPK Tools

A pair of Python scripts for inspecting and swapping Rocket League cosmetic assets (`.upk` packages).

- **`rl_upk_editor.py`** - Low-level UPK inspector and editor (GUI)
- **`rl_asset_swapper.py`** - High-level cosmetic asset swapper (GUI + CLI)

---

## Requirements

- Python 3.9+
- `cryptography` library

```bash
pip install cryptography
```

Both scripts use Python's built-in `tkinter` for their GUIs. On Windows this is included by default. On Linux you may need to install it separately (e.g. `sudo apt install python3-tk`).

The two scripts must be placed **in the same folder** - the asset swapper imports the UPK editor at runtime and will fail to start if it can't find it.

---

## Required Files

| File | Description |
|---|---|
| `items.json` | Item database mapping product names to their `.upk` asset packages |
| `keys.txt` | Decryption keys for Rocket League's encrypted `.upk` files |

Both files should be placed in the same directory as the scripts. The tools will look for them there automatically.

---

## ⚠️ Warning - Some Swaps Will Crash the Game

> **Swapping certain asset types is not yet fully supported and will cause Rocket League to crash on load.** This is a known limitation of the current version and is being worked on. Until a fix is released, avoid swapping the following:

> If a swap you attempt causes a crash, you should either validate game files, OR, you can select a directory with backup files (i play on steam, so i use my epic folder), that will be used when you hit the revert button.

---

## `rl_asset_swapper.py` - Asset Swapper

Takes a "donor" item's `.upk` file and repackages it so the game loads it in place of a "target" item. Both the main package and the thumbnail (`_T_SF.upk`) are handled automatically.

### GUI Mode (default)

```bash
python rl_asset_swapper.py
```

The window has four main sections:

**Top bar - file paths**

| Field | What to set |
|---|---|
| `items.json` | Path to your item database (auto-detected if in the same folder) |
| `keys.txt` | Path to your decryption keys (auto-detected if in the same folder) |
| Donor/input directory | The folder containing your source `.upk` files (e.g. your dumped `CookedPCConsole`) |
| Output directory | Where the swapped files will be written |
| Key/revert source dir | The folder containing the original, unmodified `.upk` files - used for correct re-encryption and for reverting swaps. Usually the same as the donor directory |

**Slot filter & options**

Select a slot (e.g. `Body`, `Wheel`, `Topper`) from the dropdown to filter the item lists. Options:

- **Also swap thumbnails/_T_SF** - also patches the inventory thumbnail package (recommended, on by default)
- **Preserve header offsets for shorter names** - keeps the package header size stable when a new name is shorter than the old one (recommended, on by default)
- **Overwrite + .bak** - overwrites existing output files and saves a `.bak` backup first

**Item lists**

The left list is the **target** - the item whose slot you want to replace visually. The right list is the **donor** - the item whose appearance will be used. Use the search boxes above each list to filter by name, ID, or package name.

The **Preview** panel below the lists shows exactly which name-table entries will be renamed before you commit.

**Running a swap**

1. Set all four directory/file paths in the top bar.
2. Choose a slot from the dropdown.
3. Select a target item (left list) and a donor item (right list).
4. Click **Swap**.

The Log panel will show the full operation output. If overwrite is enabled, a `.bak` copy of any replaced file is saved next to it.

**Reverting a swap**

Select the target item you want to revert and click **Revert selected target**. This copies the original file back from the key/revert source directory.

---

### CLI Mode

For scripting or automation, pass `--no-gui`, `--auto-swap`, or `--revert` to skip the GUI entirely.

```bash
# Swap a target item to look like a donor item
python rl_asset_swapper.py --no-gui --donor-dir  /path/to/CookedPCConsole --output-dir /path/to/output --slot Body --target "Octane" --donor  "Fennec"

# Revert a target item back to original
python rl_asset_swapper.py --revert  --donor-dir  /path/to/CookedPCConsole --output-dir /path/to/output --target "Octane"
```

**All CLI flags**

| Flag | Default | Description |
|---|---|---|
| `--items` | `items.json` | Path to item database |
| `--keys` | `keys.txt` | Path to decryption keys (auto-searched if omitted) |
| `--donor-dir` | *(required)* | Source `.upk` directory |
| `--output-dir` | *(required)* | Output directory |
| `--key-source-dir` | Same as `--donor-dir` | Directory with originals for re-encryption/revert |
| `--slot` | *(none)* | Filter items by slot |
| `--target` | *(required)* | Target item - name, ID, or package stem |
| `--donor` | *(required unless `--revert`)* | Donor item - name, ID, or package stem |
| `--include-thumbnails` / `--no-thumbnails` | On | Also swap the `_T_SF` thumbnail package |
| `--preserve-header-offsets` / `--no-preserve-header-offsets` | On | Keep header size stable for shorter names |
| `--overwrite` / `--no-overwrite` | On | Overwrite existing outputs (saves `.bak`) |
| `--no-gui` | Off | Run headlessly (requires `--target` and `--donor`) |
| `--auto-swap` | Off | Alias for `--no-gui` |
| `--revert` | Off | Copy original back instead of swapping |

---

## `rl_upk_editor.py` - UPK Editor

A low-level inspector and editor for individual `.upk` files. Useful for examining package internals, making precise edits, and saving re-encrypted packages that the game will accept.

### Launching

```bash
python rl_upk_editor.py
```

### Loading a Package

Use **File → Open UPK** to load any `.upk` file. Encrypted Rocket League packages are automatically decrypted using `keys.txt` (searched in the script directory and next to the loaded file). Decrypted working copies are saved to an `AssetSwapper_Decrypted/` subfolder next to the script.

### Inspector Tabs

Once a package is loaded, four tabs are available:

**Summary** - package header metadata: file version, counts and offsets for the name/export/import tables, compression flags, and the package GUID.

**Names** - the full name table. Select any entry to edit it in the field at the bottom of the tab and click **Rename** (or press Enter) to apply. Renaming rebuilds all header offsets automatically.

**Exports** - all exported objects. Selecting an entry shows its properties, raw hex data, and (where applicable) decoded property tags. Additional actions available when an export is selected:

- **Rename Export FName** - change the FName of the selected export
- **Replace Export From Donor** - overwrite this export's serial data with an export from a different `.upk`
- **Set DLLBind** - inject or change a DLLBind DLL name on a UClass export (used by mod loaders such as BakkesMod and CodeRed)

**Imports** - all imported object references, shown with their resolved package and class paths.

### Editing and Saving

All edits are made in memory and must be explicitly saved. Two save options are available under the **File** menu:

- **Save Decrypted UPK** - writes the raw decrypted bytes. Use this for packages that were not originally encrypted, or for external tools that work with plain UPKs.
- **Save Re-Encrypted UPK** - re-compresses and re-encrypts the package using the original file's key, producing a file the game will load. **Always use this when modifying Rocket League packages for in-game use.**

### Advanced Operations (Edit menu)

- **Import Donor Names** - append name-table entries from a second `.upk` into the loaded package.
- **Import Donor Exports as Imports** - re-expose a donor package's exports as import references in the current package, so the engine will load the donor file as a dependency at runtime.
- **Verify Package** - run a consistency check on all header invariants and display a colour-coded report. Useful for diagnosing crashes caused by invalid offsets after editing.

---

## Typical Workflow

1. Locate and backup `CookedPCConsole` from Rocket League (or obtain `.upk` files another way).
2. Place `items.json` and `keys.txt` next to both scripts.
3. Run `rl_asset_swapper.py`, point **Donor/input directory** and **Key/revert source dir** at your backed up files, and set the **Output directory** to your real Rocket League `CookedPCConsole`.
4. Choose a slot, pick your target and donor items, and click **Swap**.
5. Copy the output `.upk` file(s) into your game's `CookedPCConsole` folder (overwrite or add via a mod loader).
6. Launch the game.

To undo a swap, either click **Revert selected target** in the swapper, or manually delete/replace the output file with the original.

---

## Tips

- The **target** item is the slot that will change visually in-game. The **donor** item is the one whose model/texture/effect will be used.
- Target and donor must be from the **same slot** (e.g. you cannot swap a Body onto a Wheel slot).
- If a swap produces a package that fails to load, open the output file in `rl_upk_editor.py` and run **Verify Package** to see which header invariant was violated.
- The `.bak` files created by overwrite mode make it easy to restore a previous swap without needing the original dump.
