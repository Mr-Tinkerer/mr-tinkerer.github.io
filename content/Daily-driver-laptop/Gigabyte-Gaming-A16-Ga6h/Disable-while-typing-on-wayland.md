---
title: Fixing Disable-While-Typing (DWT) on Wayland
description: How I restored touchpad disable-while-typing behavior on this laptop's keyboard/touchpad hardware under Niri/KDE on Wayland.
tags:
  - Linux
  - Wayland
  - Touchpad
  - libinput
---
# Fixing Disable-While-Typing (DWT) on This Laptop

> **TL;DR:** My touchpad kept moving while I typed because libinput didn't realize my keyboard and touchpad were the same physical laptop. I fixed this by manually telling `udev` and `libinput` "these devices belong together," and the trick was making sure I'd told it about **every** keyboard event node that can actually emit a keystroke — including the input-remapper virtual one I use for a hardware macro key.

---

## 1. What "Disable-While-Typing" actually is

When I type, `libinput` (the layer that turns raw hardware signals into mouse/keyboard events for KDE or Niri) briefly mutes the touchpad — about 500ms after each keystroke — so my palms don't cause stray clicks or cursor jumps. This is DWT.

For DWT to activate, **three conditions must all be true at the same time**, for the keyboard device that is actually sending the keystroke:

1. **Same Group.** libinput assigns every input device a `Group` number. The keyboard and the touchpad must share the *same* group number, or libinput assumes they're unrelated devices (e.g. a desktop keyboard sitting next to a separate laptop).
2. **Both tagged "internal."** Both devices need a tag telling libinput they're physically built into the laptop, not an external/USB/Bluetooth accessory.
3. **No phantom external mouse.** The compositor (KWin or Niri) watches for anything tagged as an external pointer device. If it thinks a mouse is plugged in, it can disable touchpad gestures/behavior outright, DWT included.

If even one of these three is wrong **for the specific device node generating my keystrokes**, DWT silently does nothing — no error, no warning, nothing in the logs. It just doesn't work.

---

## 2. Why this breaks on this laptop specifically

This laptop has two complications that most guides online don't cover:

### 2a. The keyboard isn't one device — it's several
Running `libinput list-devices` shows *multiple* entries all named things like `GIGABYTE USB-HID Keyboard`, each with a different `/dev/input/eventN` number, but the same USB hardware ID (`usb:0414:8104`). This is normal — modern keyboard controllers split themselves into several logical interfaces (one for regular keys, one for media/volume keys, one for a "wireless radio control" toggle, etc).

**The trap:** only ONE of these nodes is the one that actually emits real letter/number keystrokes at any given moment. Which one it is can depend on what's intercepting input (see below), and it can change across reboots, kernel updates, or driver changes. If a udev/libinput config only tags *some* of these nodes as "internal" and grouped, and the active one isn't among them, DWT fails completely — even though everything else looks correctly configured.

### 2b. The Microsoft Copilot key needs a remapper
This laptop has a hardware Copilot key that doesn't send a normal keypress. At the firmware level it fires a macro — `Left Meta` + `Left Shift` + `F23` — all within milliseconds. To turn that into a clean, usable `Meta`/`Super` key, I need a remapping tool sitting in the input path to catch that macro and rewrite it.

[input-remapper](https://github.com/sezanzeb/input-remapper) is the tool I currently use for this. It works by grabbing the physical keyboard's events and re-emitting them through a new *virtual* device (visible as `input-remapper keyboard` in `libinput list-devices`). The problem: libinput now sees keystrokes coming from this virtual device, not the physical one — so the virtual device also needs to be grouped and tagged correctly, on top of the physical node.

> **Note on keyd:** an earlier attempt used `keyd` instead of `input-remapper` for the same purpose. If `keyd virtual keyboard` / `keyd virtual pointer` ever show up in `libinput list-devices` while input-remapper is supposed to be the active remapper, that means keyd's daemon is still running in the background (a service left enabled after switching tools). Two remappers can both grab the keyboard and cause confusing, inconsistent behavior. Fix: `sudo systemctl disable --now keyd` and reboot, then confirm the keyd virtual devices are gone from `libinput list-devices`.

### 2c. The phantom mouse
The keyboard controller also exposes a sub-device called `GIGABYTE USB-HID Keyboard Mouse` that has pointer capabilities, purely because of how the USB HID descriptor is built — there's no actual second mouse. Without intervention, KDE/Niri can see this and think "there's an external mouse plugged in," which can suppress touchpad DWT behavior compositor-wide (condition 3 above).

---

## 3. The fix

I use two config files. One tells `udev` (the kernel's device manager) to group the right devices together. The other tells `libinput` directly which devices count as "internal."

### File 1 — `/etc/udev/rules.d/99-libinput-grouping.rules`

```
# 1. Group the physical touchpad with the legacy AT keyboard interface
ACTION=="add|change", SUBSYSTEM=="input", KERNEL=="event*", ATTRS{name}=="ELAN0A05:01 04F3:3337 Touchpad", ENV{LIBINPUT_DEVICE_GROUP}="gigabyte_laptop_unified"
ACTION=="add|change", SUBSYSTEM=="input", KERNEL=="event*", ATTRS{name}=="AT Translated Set 2 keyboard", ENV{LIBINPUT_DEVICE_GROUP}="gigabyte_laptop_unified"

# 2. Group the physical GIGABYTE USB-HID Keyboard interface(s)
#    (this is the node that actually emits real keystrokes on this hardware)
ACTION=="add|change", SUBSYSTEM=="input", KERNEL=="event*", ATTRS{name}=="GIGABYTE USB-HID Keyboard", ENV{LIBINPUT_DEVICE_GROUP}="gigabyte_laptop_unified"

# 3. Group the input-remapper virtual keyboard (for when input-remapper is running)
ACTION=="add|change", SUBSYSTEM=="input", KERNEL=="event*", ATTRS{name}=="*input-remapper*", ENV{LIBINPUT_DEVICE_GROUP}="gigabyte_laptop_unified"

# 4. Strip the phantom mouse tags from the Gigabyte keyboard's pointer sub-node
ACTION=="add|change", SUBSYSTEM=="input", KERNEL=="event*", ATTRS{name}=="GIGABYTE USB-HID Keyboard Mouse", ENV{ID_INPUT_MOUSE}="", ENV{ID_INPUT_POINTER}=""
```

**What each rule does, in plain language:**

| Rule | Matches | Effect |
|---|---|---|
| 1 | Touchpad + legacy "AT Translated" keyboard interface | Puts them in the same group |
| 2 | The physical `GIGABYTE USB-HID Keyboard` interface(s) | Puts the *actual real keyboard hardware* in the same group — this is the one that's easy to miss |
| 3 | Anything with "input-remapper" in its name (wildcard) | Catches the virtual keyboard input-remapper creates, regardless of its exact name or event number |
| 4 | The specific phantom mouse sub-node | Removes its "I am a mouse" tags so the compositor stops thinking an external mouse is connected |

### File 2 — `/etc/libinput/local-overrides.quirks`

```
[Virtual Keyboard Internal Tag]
MatchName=*input-remapper*
AttrKeyboardIntegration=internal

[Touchpad Internal Tag]
MatchName=ELAN0A05:01 04F3:3337 Touchpad
AttrKeyboardIntegration=internal

[Physical HID Keyboard Internal Tag]
MatchName=GIGABYTE USB-HID Keyboard
AttrKeyboardIntegration=internal
```

Each `[Header]` block is one rule. `MatchName` finds the device (wildcards with `*` allowed), and `AttrKeyboardIntegration=internal` tells libinput "treat this as physically built into the laptop, not an external accessory."

### Apply the changes

```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Then **reboot**. In testing, a reboot was required for the new group assignments to take full effect — `udevadm trigger` alone wasn't always enough to fix devices that were already enumerated.

---

## 4. How to verify it's actually fixed

I don't just trust that "it looks right" — I confirm using these three checks, in order.

### Check 1: Are the right devices grouped together?
```bash
sudo libinput list-devices
```
Find the entries for the touchpad, `AT Translated Set 2 keyboard`, `GIGABYTE USB-HID Keyboard`, and (if running) `input-remapper keyboard`. **They must all show the same `Group:` number.** If even one is different, DWT will not work for keystrokes coming from that device.

Also confirm the touchpad's own line says:
```
Disable-w-typing:        enabled
```

### Check 2: Are the internal tags actually applied?
```bash
sudo libinput quirks list /dev/input/eventN
```
(replace `eventN` with the actual node — find it from the list-devices output above). I should see:
```
AttrKeyboardIntegration=internal
```
Run this for **both** the touchpad's event node and the keyboard's event node.

### Check 3: Watch raw events live (the most reliable test)
```bash
sudo libinput debug-events
```
With this running, type on the keyboard and immediately try to move the touchpad. Watch the output:

- Look at which `eventN` the `KEYBOARD_KEY` lines are coming from. Note its group from the `DEVICE_ADDED` line near the top of the output.
- If `POINTER_MOTION` events from the touchpad **stop appearing** for about half a second after each keypress, DWT is working at the libinput level.
- **This is the test that catches the "wrong keyboard node" problem.** If the keyboard events are coming from a node whose `DEVICE_ADDED` line shows a different group than the touchpad, that's the culprit — no guessing required.

If `debug-events` shows correct suppression but the cursor *still* visibly moves on screen in KDE or Niri, the bug has moved one layer up, into the compositor itself — see the troubleshooting table below.

---

## 5. Troubleshooting / common failure patterns

| Symptom | Likely cause | Fix |
|---|---|---|
| DWT fails for **all** keys, no exceptions | The actual keyboard event node isn't grouped with the touchpad | Run `debug-events`, find which node sends keystrokes, add a udev rule + quirks entry for it |
| DWT fails only for **remapped** keys (e.g. Copilot key) | input-remapper's virtual device isn't grouped/tagged | Confirm `input-remapper keyboard` shows the same group as the touchpad in `list-devices` |
| `libinput list-devices` shows `keyd virtual keyboard` AND `input-remapper keyboard` at once | Two remappers running simultaneously (leftover keyd service) | `sudo systemctl disable --now keyd`, reboot, recheck |
| `GIGABYTE USB-HID Keyboard Mouse` shows `ID_INPUT_MOUSE=1` in `udevadm info` | Phantom mouse rule isn't matching or isn't loaded | Re-check rule 4 spelling exactly matches the device name; reload rules |
| Everything *looks* correctly grouped and tagged, but DWT still fails | The wrong physical node was configured — there are multiple `GIGABYTE USB-HID Keyboard` entries with the same name; only the active one needs grouping, but grouping all of them by name is safe and simpler | Match by `ATTRS{name}` (not a specific event number) so all current and future nodes with that name are covered automatically |
| Settings toggle in KDE System Settings does nothing | KDE's toggle only writes to its own libinput config layer; it can't fix grouping/tagging problems underneath | Ignore the GUI toggle for diagnosis — fix at the udev/quirks level, GUI toggle becomes irrelevant once the layers below are correct |
| A typo in a udev rule (e.g. `KERNEL=="problemsevent*"` instead of `KERNEL=="event*"`) | Rule silently never matches anything — `udev` doesn't warn about rules that just never fire | Always re-view the rules file after editing; test with `udevadm test /sys/class/input/eventN` to see which rules actually apply to a given device |

---

## 6. What could break this again in the future

This setup is fragile in a specific way: it depends on **device names and groupings that aren't guaranteed to stay the same** across updates. Here's what to watch for:

### Kernel updates
A kernel update can change how the keyboard controller's USB HID descriptor is parsed, which can change:
- Which `eventN` number gets assigned to which sub-interface (not a problem, since these rules match by **name**, not number)
- In rarer cases, the **name** udev assigns to a device, if the kernel's HID parsing logic changes how it labels endpoints

**If this happens:** run `libinput list-devices` after any kernel update and confirm the device names being matched on (`GIGABYTE USB-HID Keyboard`, `ELAN0A05:01 04F3:3337 Touchpad`, `AT Translated Set 2 keyboard`) still appear exactly as written. If a name changes even slightly, the udev rule silently stops matching — no error, it just stops working.

### input-remapper updates
An update could change:
- The name pattern of the virtual device it creates (currently anything containing `input-remapper` — covered by the wildcard match, so minor naming tweaks are safe)
- How it grabs the physical device (e.g. switching which physical node it intercepts) — if this changes, the *real* keystrokes might start coming from a different physical node than before, recreating the exact bug fixed in section 3

**If this happens:** re-run Check 3 (`debug-events`) above. It will immediately show which node is now live.

### libinput updates
libinput periodically changes default heuristics for automatic device grouping and quirks loading. A libinput update could:
- Change the default group assignment algorithm, potentially undoing the *effect* of group overrides if the override mechanism's syntax changes
- Ship new built-in quirks that conflict with or shadow `/etc/libinput/local-overrides.quirks`

**If this happens:** run `libinput quirks validate` (no arguments) after any libinput update — this validates the full merged quirks database, not just this file, and will catch syntax incompatibilities. Also re-run Check 1 and Check 2 from section 4.

### Switching remappers again (input-remapper ↔ keyd ↔ something else)
If I ever switch remapping tools again:
1. Fully disable the old one: `sudo systemctl disable --now <old-tool>`
2. Reboot
3. Confirm with `libinput list-devices` that the old tool's virtual devices are completely gone
4. Add a new udev rule + quirks entry matching the new tool's virtual device name
5. Run all three checks in section 4 again

### General rule of thumb
**Any time DWT mysteriously breaks after a system update, the fix is almost always: re-run `libinput list-devices`, find what's actually emitting keystrokes via `libinput debug-events`, and confirm it shares a group with the touchpad.** The three rules in section 1 haven't changed in years — what changes is *which device* needs them applied.

---

## 7. Command reference

| Command | What it tells me |
|---|---|
| `sudo libinput list-devices` | All input devices, their group numbers, and capabilities. My main diagnostic tool. |
| `sudo libinput debug-events` | Live raw event stream. Type and move the touchpad while it's running to see exactly what libinput sees, in real time. The most reliable way to find the *actual* active keyboard node. |
| `sudo libinput quirks list /dev/input/eventN` | Shows which quirks (like `AttrKeyboardIntegration=internal`) are currently active on a specific device. |
| `libinput quirks validate` | Checks quirks file(s) for syntax errors against the full merged quirks database. Run with no arguments for the complete check. |
| `udevadm info /dev/input/eventN \| grep ID_INPUT` | Shows whether a device is currently tagged as a mouse/pointer/keyboard by udev. Use this to confirm the phantom-mouse strip worked. |
| `udevadm test /sys/class/input/eventN` | Shows exactly which udev rules match (and don't match) a given device — useful for catching silent typos in rule files. |
| `sudo udevadm control --reload-rules && sudo udevadm trigger` | Reloads udev rules and re-runs them against currently connected hardware. Follow with a reboot if devices were already active. |
| `sudo systemctl disable --now keyd` | Fully stops and disables the keyd service (use if switching away from keyd to avoid leftover virtual devices). |

---

## 8. Quick checklist for "DWT broke again"

- [ ] `libinput list-devices` — do the touchpad + all real/virtual keyboard nodes show the **same Group number**?
- [ ] `libinput quirks list` on both touchpad and keyboard nodes — does `AttrKeyboardIntegration=internal` show up on both?
- [ ] `udevadm info` on the phantom mouse node — confirm `ID_INPUT_MOUSE` is empty/absent
- [ ] `libinput debug-events` — type, and confirm which event node is *actually* firing `KEYBOARD_KEY`, then confirm that exact node's group matches the touchpad
- [ ] Only one remapping tool running (`systemctl status input-remapper` / `keyd` — confirm only the intended one is active)
- [ ] Re-view both config files for typos (`KERNEL==`, `ATTRS{name}==`, exact device names) — udev does not warn on non-matching rules

---

## 9. Lessons learned

The original version of the udev rules file only grouped the touchpad with the legacy `AT Translated Set 2 keyboard` node and the input-remapper virtual node — it never grouped the physical `GIGABYTE USB-HID Keyboard` node (rule 2 above), which turned out to be the node actually emitting real keystrokes on this hardware. Because that node was left in its own group, DWT failed 100% of the time for every key, even though the touchpad itself and every other part of the configuration was correct. The lesson: when multiple device nodes share similar names, don't assume the "obvious" one (the legacy AT-translated node) is the one generating live input — verify with `libinput debug-events` and group by name pattern rather than guessing which single node matters.

A separate, earlier draft of this fix also had a plain typo — `KERNEL=="problemsevent*"` instead of `KERNEL=="event*"` — which silently prevented that rule from ever matching anything. udev does not warn about rules that never fire, so a rule with a typo looks identical to a correctly-applied one until it's tested against real hardware with `udevadm test`.
