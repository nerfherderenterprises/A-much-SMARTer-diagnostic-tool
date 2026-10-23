# drive-verdict

**Your hard drive is dying. You just can't tell, because the report is in hex.**

`drive-verdict` reads a hard drive's built-in health report and prints the full
technical report first, then a plain-English verdict at the end. One command,
two audiences: a technician reads the top and never opens a second terminal;
everyone else scrolls to the color and knows instantly whether to worry.

```
$ sudo drive-verdict

===== FULL TECHNICAL REPORT (/dev/sda) =====
smartctl 7.4 2023-08-01 r5530 [x86_64-linux-6.8.0-139-generic] (local build)
=== START OF INFORMATION SECTION ===
Model Family:     Seagate Barracuda 7200.14 (AF)
Device Model:     ST500DM002-1BD142
...
Vendor Specific SMART Attributes with Thresholds:
  ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
   5 Reallocated_Sector_Ct   0x0033   100   100   036    Pre-fail  Always       -       0
   9 Power_On_Hours          0x0032   064   064   000    Old_age   Always       -       32231
 197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
...
===== END OF REPORT — plain-English verdict below =====

VERDICT: healthy.
  No bad spots, nothing failing. This drive is fine.

  the details  (/dev/sda)
   how long it has been used : 32231 hours (~3 years)
   how many times on/off     : 3111
   temperature               : 27C

  the one rule:  as long as the top two lines stay at 0, the drive is not dying.
```

## Why this exists

Every drive ships with a self-monitoring system (SMART) that predicts failure
hours to weeks before it happens. Every drive also assumes you know what
`Reallocated_Sector_Ct` means. Most people — including paid technicians on a
busy day — do not.

So failing drives get ignored until the data is gone. This tool deletes the
retention problem: you don't need to remember what any field means, because the
answer is a color and a sentence. And it deletes the second-problem too: the
full raw report prints first, so a technician who wants the numbers has them in
the same run — no re-running `smartctl`, no second diagnostic.

The design rule it was built under: **the diagnostic should be readable by the
person who needs it even when the expert isn't answering.** Plain English isn't
a dumbed-down mode you switch into — it's the default, and the raw report rides
along for the people who want it.

## Install

```bash
git clone https://github.com/nerfherderenterprises/A-much-SMARTer-diagnostic-tool.git
cd A-much-SMARTer-diagnostic-tool
sudo apt install smartmontools          # or your distro's equivalent
sudo cp drive-verdict /usr/local/bin/
sudo chmod +x /usr/local/bin/drive-verdict
```

Or just grab the script and drop it anywhere on your `$PATH` — it's a single
self-contained Bash file with no dependencies beyond `smartctl`.

## Usage

```bash
sudo drive-verdict              # check the main drive (/dev/sda)
sudo drive-verdict /dev/sdb     # check a specific drive
```

Root is required because a drive's health report is protected. If you forget
`sudo`, the tool tells you the exact command to run instead of failing silently.

## The verdicts

| color  | means |
|--------|-------|
| 🟢 green  | **healthy** — no bad spots, nothing failing. Nothing to do. |
| 🟡 yellow | **tired but working** — no damage, but high mileage or running hot. Back it up soon. |
| 🔴 red    | **something is wrong on the disk surface** — back up **now**. |

The rule behind the colors, in full: drives quarantine bad sectors as they
appear, and the count of quarantined or waiting sectors is the single most
reliable early warning of death. As long as those two numbers stay at **0**, the
drive is not dying — no matter how old it looks. The moment they climb, start
moving data off.

## Scope (honest)

- **Checks:** traditional spinning hard drives (SATA/SAS) that expose SMART.
- **Doesn't check yet:** SSDs and NVMe drives (different health signals — SSD
  life is a wear-countdown, not a bad-spot count), and many USB enclosures,
  which block SMART passthrough entirely. If a USB drive returns no report, the
  tool says so plainly rather than guessing — the drive can still be perfectly
  fine.

Version 1 is a read-only health check. It will never write to, modify, or
"fix" your drive.

## License

MIT — see [LICENSE](LICENSE).
