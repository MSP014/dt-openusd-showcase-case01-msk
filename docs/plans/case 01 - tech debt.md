# Case 01 Technical Debt

*No critical technical debt identified during initial audit by Higgins.*

## [ADR] USD Composition Strategy

- **Status:** Deferred
- **Severity:** Low (Architectural)
- **Description:** ADR 004 (USD Composition) was skipped during initial standardization to focus on baseline stability.
- **Task:** Formalize the USD layering and referencing strategy Once the Omniverse Kit environment is fully stable.

## [PIPELINE] Asset Hydration & Bootstrap

- **Status:** Open
- **Severity:** Low (Process)
- **Description:** Per ADR 005 revision, heavy assets (textures, caches) will be distributed via a simple ZIP archive rather than a custom `bootstrap.py` script to avoid unnecessary infrastructure overhead.
- **Tasks to Resolve:**
  - Pack initial heavy assets into a `case01_assets.zip` archive.
  - Upload the archive to external storage (e.g., Google Drive / OneDrive).
  - Add the download link and explicit manual extraction instructions to the `README.md` under a "Heavy Assets Setup" section.

## [ENVIRONMENT] Pip Security Lock (CVE-2026-1703)

- **Status:** Open
- **Severity:** High (Security)
- **Description:** Environment `case01-env` is running `pip 25.3`, which contains CVE-2026-1703. Cannot upgrade to `pip 26.0+` because it breaks `pip-tools`.
- **Tasks to Resolve:**
  1. Do **NOT** upgrade pip until `pip-tools` releases a fix.
  2. If `pip-audit` flags this, verify pip is ignored or risk is accepted.
  3. On session start: if date >= **Next Check Date**, run `pip index versions pip-tools` to check for `pip 26+` compatibility.
- **Check Log:**
  - **2026-02-26**: `pip-tools` latest is 7.5.3. No fix yet. `pip` upgrade blocked.
- **Next Check Date:** 2026-03-14
