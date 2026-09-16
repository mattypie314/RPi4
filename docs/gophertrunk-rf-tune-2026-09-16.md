# GopherTrunk RF tune — mattypi (2026-09-16)

Live changes were applied on **mattypi** under `~/GopherTrunk/`. Backups were left next to the originals (`*.bak-pre-rf-tune-*`, `*.bak-pre-priority-*`).

## Problems observed

1. **Wideband ADC overload** (`wideband front end overloaded`, issue #749) with `gain: "auto"` on a hot MARCS-Cuyahoga simulcast.
2. **Voice-stick starvation** (`no voice device available for grant`) — only one dedicated voice SDR; most MARCS voice freqs sit **outside** the 774.3 ± 1.2 MHz wideband window, so `voice_taps` cannot cover them.
3. **Corrections chatter** (especially `LORHOPS`) was soaking the voice stick and recordings.

## Changes applied

### Home location: North Ridgeville

- **NR Fire** is on **5-City** (P25 Phase 2) — TG `4270` N RIDGEVL FD
- **NR PD** is on **MARCS** — TG `29540` NRIDGEVL PD DISP
- Two systems do not fit one 2.4 MHz wideband window. Active config is **5-City primary** @ `771_000_000` Hz. Use `profiles/marcs.yaml` / the LAN switcher when you want NR PD live.

### `config.yaml`

| Setting | Before | After |
|---|---|---|
| Wideband center | `774_300_000` (MARCS) | `771_000_000` (5-City) |
| Wideband channels | MARCS CCs | `770.30625` + `770.60625` (5-City) |
| `sdr.sample_rate` | `2_400_000` | `1_800_000` (enough for this channel span; less Pi CPU) |
| Wideband `00000001` gain | `"auto"` | `"297"` (29.7 dB — locks 5-City; tenths of a dB) |
| Voice `00000002` gain | `"auto"` | `"297"` (29.7 dB, fixed) |
| `voice_hangtime_ms` | `3500` | `2000` |
| Active systems | 5-City + MARCS + GCRCN | **5-City only** (use switcher / `profiles/marcs.yaml` for NR PD) |
| Binary | v1.1.1 | **v1.1.5** (`/usr/local/bin/gophertrunk`) |
| `scanner` | missing | `scan_mode: all` |

Verified: `p25/phase2 cc locked` on **5-City @ 770.30625 MHz**.

GopherTrunk gain is **tenths of a dB** (`254` = 25.4 dB). Do not set `"25"` thinking that is 25 dB.

Same gain/hangtime patches applied to `profiles/5city.yaml` and `profiles/marcs.yaml`.

### Talkgroups

**`talkgroups-5city.csv` (Fire/EMS-first)**
- Priority **1**: NR Fire (`4270`, `4606`), all Fire/EMS Dispatch, LifeCare, county fire dispatch
- Priority **2**: Fire-Tac
- **Lockout=Y** on Law/PD/Schools/Interop so the voice stick stays on fire/EMS
- `scanner.scan_mode: list` (only `Scan=Y`)

**`talkgroups-marcs-live.csv`**
- **Lockout=Y** on Corrections (39 TGs)
- Lorain/NR priorities kept for when you switch to the MARCS profile

**Phone**
- OpenWebRX container **stopped** (`restart=no`) — both SDRs for GopherTrunk
- Landing page: `https://mattypi.tail5e4d38.ts.net/radio/`
- Console: `https://mattypi.tail5e4d38.ts.net:8443/`

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
