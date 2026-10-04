# Perform keeps levels (Digitakt II)

A mod for Digitakt II OS 1.17, with one site of its own.

- **Stock** (the manual, 10.10): in PERFORM KIT mode, changes to the kit are
  not saved, and the kit stays loaded when you change patterns. When you
  leave PERFORM KIT mode, the pattern's own kit loads again, so every
  change is lost, track levels included.
- **With this mod:** the track levels you leave PERFORM KIT mode with stay.
  They go into the pattern's kit (the pattern active when you leave) as if
  you had set them outside PERFORM KIT mode. Every other parameter still
  goes back to the pattern's kit, as stock. Only the 16 track levels are
  kept (LEVEL/DATA, and the INTERNAL MIXER pages' track levels). The
  pattern volume, the send FX and everything else in the kit go back.

How it works:
- **The stock toggle.** [FUNC] + [PRESET] runs 0x40064aec. Turned on, it
  copies the pattern's kit into a spare kit, slot 0x80 (0x40035e12), and
  plays that one, so every tweak goes there. Turned off, it selects the
  pattern's kit (`d2`, 0-127) again (0x40036cba).
- **The hook.** The off path starts at 0x40064ba6 with
  `move.l $98(a2),-(sp) ; jsr (a3)`, which gets the kit collection. This
  mod's site replaces those two instructions with a `jsr` to `pl_off`
  (`levels.s`). `pl_off` does their work and calls `pl_keep(kits, d2)`
  (`levels.c`).
- **The copy.** `pl_keep` copies the 16 level words (kit + 0x14, track 1
  first, the value in the high byte) from slot 0x80 into the pattern's kit.
  If any of them changed, it then calls the collection's +0x5c(kits, slot,
  0), the call the firmware makes after each of its own kit copies. The
  select that follows loads the kit with those levels.

```bash
python -m elekloader.sdk.build mods/core-dt2 --stock Digitakt_II_OS1.17.syx
python -m elekloader.sdk.build examples/perform-levels --stock Digitakt_II_OS1.17.syx
python -m elekloader.patch --stock Digitakt_II_OS1.17.syx \
    --mod mods/core-dt2/out/core-1.0.elemod \
    --mod examples/perform-levels/out/perform-levels-1.0.elemod --out levels.syx --version PL01
```

It links with [perform-direct](../perform-direct), which moves PERFORM to
[PRESET] alone. Both are released in
[toonst/digitakt2-perform](https://github.com/toonst/digitakt2-perform),
and listed in the web page's mod shop.

Checked in digikit's emulator (4 Oct 2026). A cold boot of core with this
mod passes every stage, as stock does. The emulator applies no encoder
turn to a kit, on stock either: it shows the value but writes nothing. So
each run turned PERFORM KIT on, then wrote the perform kit (slot 0x80)
directly: track 1's level 106 -> 32, track 16's 110 -> 127, and the first
four letters of track 1's sound name as a non-level marker. Then it turned
PERFORM KIT off.
- **Stock:** the pattern's kit keeps 106 and 110, and LEVEL/DATA shows 106
  again.
- **With this mod:** the pattern's kit has 32 and 127, LEVEL/DATA shows 32,
  and the sound name went back, as every other parameter does.

Neither run has a DSP, so what you hear was not checked there. Confirmed
on a unit (5 Oct 2026).
