# GopherTrunk RF tune — mattypi (2026-09-16)

Live changes were applied on **mattypi** under `~/GopherTrunk/`. Backups were left next to the originals (`*.bak-pre-rf-tune-*`, `*.bak-pre-priority-*`).

## Problems observed

1. **Wideband ADC overload** (`wideband front end overloaded`, issue #749) with `gain: "auto"` on a hot MARCS-Cuyahoga simulcast.
2. **Voice-stick starvation** (`no voice device available for grant`) — only one dedicated voice SDR; most MARCS voice freqs sit **outside** the 774.3 ± 1.2 MHz wideband window, so `voice_taps` cannot cover them.
3. **Corrections chatter** (especially `LORHOPS`) was soaking the voice stick and recordings.

## Changes applied

### `config.yaml`

| Setting | Before | After |
|---|---|---|
| Wideband `00000001` gain | `"auto"` | `"254"` (25.4 dB, fixed; tenths of a dB) |
| Voice `00000002` gain | `"auto"` | `"297"` (29.7 dB, fixed) |
| `voice_hangtime_ms` | `3500` | `2000` |
| `scanner` | missing | `scan_mode: all` |

GopherTrunk gain is **tenths of a dB** (`254` = 25.4 dB). Do not set `"25"` thinking that is 25 dB.

### `talkgroups-marcs-live.csv`

- **Lockout=Y** on all `Tag=Corrections` / Lorain County State Corrections (39 TGs) — stops prison housing/ops from monopolizing the stick.
- **Priority=1** Fire Dispatch + EMS Dispatch
- **Priority=3** Law Dispatch
- **Priority=5** Fire-Tac / EMS-Tac / Emergency Ops

## Verification (~70s windows)

| Metric | Before (AGC) | After |
|---|---|---|
| Overload WARNs | frequent | **0** |
| Uncorrectable LDUs (sample) | mixed | **0** across quality lines |
| `LORHOPS` in recent WAVs | dominant | **absent** |
| Call devices | voice + wb taps | still both (good) |

`no voice device available` will still appear under busy MARCS — that needs a **3rd RTL-SDR** (or accepting missed concurrent calls).

## If overload returns

Step wideband gain down the R820T2 ladder: `254` → `229` → `207` → `200`. Do **not** raise gain. Optional: inline SMA attenuator.

## Rollback

```bash
# on mattypi
cd ~/GopherTrunk
cp -a config.yaml.bak-pre-rf-tune-YYYYMMDD-HHMMSS config.yaml
cp -a talkgroups-marcs-live.csv.bak-pre-priority-YYYYMMDD-HHMMSS talkgroups-marcs-live.csv
systemctl --user restart gophertrunk.service
```
