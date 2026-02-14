TRACKER / README FINAL - BOQ.xls WORKFLOW (S01-S100 + SUMMARY)
Version: FINAL
Last updated: 2026-02-14

GOAL
- BOQ.xls is the single source of truth for all scopes (general use, not project-specific).
- Sheets S01..S100 store scope data using a fixed template.
- SUMMARY auto-collects scope titles and subtotals.
- Update-only forever: never delete useful history; avoid double-count and avoid missing items.

FILES IN REPO (RECOMMENDED)
- BOQ.xls
- Materials.txt   (reference list for exact material names + UOM + rates if available)
- Labour.txt      (reference list for exact labour names + UOM + rates if available)
- README.txt      (this file)

============================================================
1) SHEET NAMING (STRICT)
============================================================
- Work sheets must be named exactly:
  S01, S02, ... S100  (uppercase S + 2 digits, no spaces)
- Summary sheet name:
  SUMMARY  (recommended uppercase; do not rename after formulas are set)

============================================================
2) TEMPLATE LAYOUT PER SXX (LOCKED STRUCTURE)
============================================================

(A) SPEC AREA (USER-OWNED, EDITABLE)
- Columns A:C only
  A = Spec
  B = Description
  C = Note
- A1: Scope title line (editable)
  Example: "S01 SPEC (HEADER - EDITABLE)"
- Row 2: headers
  A2 = "Spec"
  B2 = "Description"
  C2 = "Note"
- Rows A3..C?? are editable, no fixed end.
- SPEC is NOT used by SUMMARY calculations directly.
- SPEC defines the scope; if SPEC changes, related quantities/items may need update.

(B) SPACER COLUMNS (MUST STAY BLANK FOREVER)
- Column D  = spacer after SPEC
- Column M  = spacer after Block 1
- Column V  = spacer after Block 2
- Column AE = spacer after Block 3
- Do not write data or formulas in spacer columns.

(C) 4 DATA BLOCKS (CODEX WRITES HERE, FIXED POSITIONS)
Each block is exactly 8 columns wide with the same logical fields:
  1) Spec
  2) Description (Exact name from Materials.txt / Labour.txt)
  3) Qty
  4) UOM
  5) Price/Unit (RM)  (blank if NO RM block)
  6) Wastage/Buffer (%)
  7) Total (RM)       (FORMULA ONLY; must never be overwritten)
  8) Note             (all status/pending/remarks go here only)

Block positions (LOCKED):
- Block 1: E:L    Total column = K (FORMULA ONLY)
- Block 2: N:U    Total column = T (FORMULA ONLY)
- Block 3: W:AD   Total column = AC (FORMULA ONLY)
- Block 4: AF:AM  Total column = AL (FORMULA ONLY)

(D) BLOCK TITLES + HEADERS (MANDATORY)
Inside each block:
- Title row (1 full row): title text in the first column of that block only.
- Header row (next row): the 8 headers:
  Spec | Description (Exact name from Materials.txt/Labour.txt) | Qty | UOM | Price/Unit (RM) | Wastage/Buffer (%) | Total (RM) | Note
- Data rows follow below.
- Subtotal row: subtotal in the block Total column only (K / T / AC / AL), note in Note column.

============================================================
3) CRITICAL DATA RULES (APPLY TO ALL SXX)
============================================================
(A) TOTAL COLUMNS ARE FORMULA-ONLY
- Never type or paste into these columns:
  K (Block1), T (Block2), AC (Block3), AL (Block4)

(B) PENDING / STATUS TEXT RULE
- Never put text like "PENDING", "OK" inside numeric cells:
  Qty / Price/Unit / Total columns
- Put all pending/status text only in Note column:
  L (Block1), U (Block2), AD (Block3), AM (Block4)

(C) UPDATE-ONLY (NO DELETE)
- Never delete old useful lines.
- If an old item becomes obsolete due to spec change:
  set Qty = 0 and explain in Note (do not delete).
- Add new rows below when adding new items.

============================================================
4) BLOCK PURPOSES (GENERAL USE)
============================================================
Block 1 (E:L) - MATERIAL USAGE (NO RM)
- Stores calculated usage quantities (base qty + wastage).
- Price/Unit normally blank.
- Total column may be "final qty" or remain formula-based per your template choice.

Block 2 (N:U) - MATERIAL STANDALONE / PRICING (RM)
- Stores supplier purchase UOM + purchase qty + unit rate (RM).
- Total = per-line RM amount formula (do not overwrite Total column).
- Must work per-scope even if customer selects only one scope.

Block 3 (W:AD) - LABOUR (NO RM by default)
- Stores labour items in days/hours (pick one standard).
- Price/Unit optional (only if you later decide to rate labour).
- Total column can be buffered days/hours.

Block 4 (AF:AM) - RENTAL / SERVICE (NO RM by default)
- Stores rental/service quantities + duration or lot basis.
- Price/Unit optional if later rated.
- Total column can be buffered qty/duration.

============================================================
5) ROLE SPLIT (FINAL - NO CONFLICT)
============================================================
(A) CODEX = PRIMARY WRITER (DOES THE WORK)
Codex must:
- Read BOQ.xls and ONLY work on specified sheet(s) SXX.
- Read Materials.txt and Labour.txt.
- Expand SPEC into complete lists:
  - materials usage (Block 1)
  - materials standalone/pricing (Block 2) if rates/UOM exist
  - labour (Block 3)
  - rental/service (Block 4)
- Map "Description" to EXACT names from Materials.txt / Labour.txt.
- Choose correct UOM and compute Qty + Wastage/Buffer.
- Update-only: do not delete old useful lines.
- Never overwrite Total columns (K/T/AC/AL).
- Put pending status only in Note columns.
If inputs are missing:
- Return ONE "INPUTS NEEDED" list only (no repeated questions every run).

(B) GPT (CHATGPT) = CHECKER / AUDITOR (NOT PRIMARY WRITER)
GPT must:
- Review Codex output for compliance:
  - correct block columns (no drift)
  - Total columns untouched
  - no text in numeric cells
  - exact name mapping matches Materials/Labour list
  - update-only respected (no deletions, obsolete -> Qty=0)
- Help user rewrite SPEC (A:C) more clearly for Codex when inputs are missing.
- Provide correction notes (what to fix, where, why).

IMPORTANT:
- GPT should NOT be the one filling E:AM as the main workflow in this plan.

============================================================
6) SUMMARY SHEET (AUTO S01-S100)
============================================================
SUMMARY recommended columns:
A = SheetName (S01..S100)
B = Scope Title (reads SXX!A1)
C = Material Standalone Subtotal (RM)
D = Labour Subtotal (days or RM)
E = Rental/Service Subtotal (qty or RM)
F = Grand Total (RM) = C + D + E (only if units are RM; otherwise keep separate)

A column list:
- A3 = S01
- A4 = S02
- ...
- A102 = S100

Formulas (start at row 3, fill down):
B3:
=IFERROR(INDIRECT("'"&TRIM($A3)&"'!$A$1"),"")

C3/D3/E3:
- You MUST standardize fixed subtotal cell positions in each SXX.
- Example fixed cells (edit to your final template):
  C3 (Material standalone subtotal):
  =IFERROR(INDIRECT("'"&TRIM($A3)&"'!$T$40"),0)
  D3 (Labour subtotal):
  =IFERROR(INDIRECT("'"&TRIM($A3)&"'!$AC$40"),0)
  E3 (Rental/service subtotal):
  =IFERROR(INDIRECT("'"&TRIM($A3)&"'!$AL$40"),0)

F3:
=SUM(C3:E3)

If row 3 shows blank:
- Most common causes:
  - formula references A2 instead of A3
  - A3 sheet name has leading/trailing spaces (use TRIM)
  - sheet name typo (must match exact S01)
  - missing quotes in INDIRECT

============================================================
7) SAFE MANUAL PASTE METHOD (ONLY IF YOU MUST PASTE)
============================================================
Because Total columns are formula-only, manual paste must SKIP them.

Per block, use TWO-RANGE paste:
- Range A (6 cols): Spec, Description, Qty, UOM, Price/Unit, Wastage/Buffer
- Range B (1 col): Note

Block 1 (E:L) skip K:
- Paste Range A into E:J
- Paste Range B into L

Block 2 (N:U) skip T:
- Paste Range A into N:S
- Paste Range B into U

Block 3 (W:AD) skip AC:
- Paste Range A into W:AB
- Paste Range B into AD

Block 4 (AF:AM) skip AL:
- Paste Range A into AF:AK
- Paste Range B into AM

Never paste into K/T/AC/AL.

============================================================
8) HANDOVER (FOR NEW AI SESSION)
============================================================
When starting a new chat, paste this:
- "We are using BOQ.xls template S01..S100 + SUMMARY. Follow README FINAL role split:
   Codex is primary writer (maps exact names + fills E:AM blocks update-only),
   GPT is checker/auditor only.
   Do not overwrite Total columns K/T/AC/AL.
   No text in numeric columns; pending only in Note columns.
   Spacer columns D/M/V/AE must stay blank.
   Work only on specified sheet(s)."

END OF README FINAL
