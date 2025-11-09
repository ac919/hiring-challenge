# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Overview
- This repo is a standalone data-analysis challenge (see `README.md` and `mission_challenge.md`).
- Goal: extract the Security Code of the longest "Completed" Mars mission from `space_missions.log`.
- There is no build, lint, or test framework here. Workflows revolve around one-off shell/awk commands.

## Common commands
All commands assume zsh/bash on macOS/Linux and run from the repo root.

- Quick answer (prints only the Security Code):
  ```sh
  CODE=$(awk -F'|' '
    BEGIN { max=-1 }
    /^[[:space:]]*#/ { next }                         # skip comments
    NF>=8 {                                           # require all fields
      for (i=1;i<=NF;i++) gsub(/^[ \t]+|[ \t]+$/, "", $i)  # trim spaces
      if ($3=="Mars" && $4=="Completed" && $6 ~ /^[0-9]+(\.[0-9]+)?$/) {
        d=$6+0
        if (d>max) { max=d; code=$8 }
      }
    }
    END { if (code!="") print code; else { print "No match" > "/dev/stderr"; exit 1 } }
  ' space_missions.log) && echo "$CODE"
  ```

- Show the top 5 candidate missions (duration, security code, mission id) for verification:
  ```sh
  awk -F'|' '
    /^[[:space:]]*#/ { next }
    NF>=8 {
      for (i=1;i<=NF;i++) gsub(/^[ \t]+|[ \t]+$/, "", $i)
      if ($3=="Mars" && $4=="Completed" && $6 ~ /^[0-9]+(\.[0-9]+)?$/) {
        printf "%d\t%s\t%s\n", $6+0, $8, $2
      }
    }
  ' space_missions.log | sort -nr | head -5
  ```

- Print the full, normalized row of the longest matching mission (handy for sanity-checking):
  ```sh
  awk -F'|' '
    BEGIN { max=-1 }
    /^[[:space:]]*#/ { next }
    NF>=8 {
      for (i=1;i<=NF;i++) gsub(/^[ \t]+|[ \t]+$/, "", $i)
      if ($3=="Mars" && $4=="Completed" && $6 ~ /^[0-9]+(\.[0-9]+)?$/) {
        d=$6+0
        if (d>max) { max=d; rec=$0 }
      }
    }
    END { if (max>=0) print rec; else { print "No match" > "/dev/stderr"; exit 1 } }
  ' space_missions.log
  ```

- Optional: validate the security code format (ABC-123-XYZ style) after computing `CODE`:
  ```sh
  [[ "$CODE" =~ ^[A-Z0-9]{3}-[0-9]{3}-[A-Z0-9]{3}$ ]] && echo OK || { echo "Unexpected format: $CODE"; exit 1; }
  ```

- Convenience function to re-run quickly in your shell session:
  ```sh
  mars_code() {
    awk -F'|' '
      BEGIN { max=-1 }
      /^[[:space:]]*#/ { next }
      NF>=8 {
        for (i=1;i<=NF;i++) gsub(/^[ \t]+|[ \t]+$/, "", $i)
        if ($3=="Mars" && $4=="Completed" && $6 ~ /^[0-9]+(\.[0-9]+)?$/) {
          d=$6+0
          if (d>max) { max=d; code=$8 }
        }
      }
      END { if (code!="") print code; else { print "No match" > "/dev/stderr"; exit 1 } }
    ' space_missions.log
  }
  ```

## Architecture and repository structure (big picture)
- `space_missions.log`: single large, pipe-delimited data source. Nuances:
  - Lines may start with `#` (comments) and should be ignored.
  - Pipe-separated fields may contain inconsistent surrounding whitespace; trim before comparing.
  - Field schema (from `mission_challenge.md`):
    Date | Mission ID | Destination | Status | Crew Size | Duration (days) | Success Rate | Security Code
- `mission_challenge.md`: authoritative problem statement and data schema.
- `README.md`: entry point to the challenge and submission guidance. It notes sharing the command you used (Warp Shared Block link) along with the answer.

## Notes
- No CLAUDE, Cursor, or Copilot rule files are present.
- There is no build/lint/test tooling in this repo; prefer one-off shell/awk workflows captured as Warp Blocks for reproducibility.
