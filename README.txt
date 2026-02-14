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

These rules apply to every SXX sheet and every block.

(A) TOTAL COLUMNS ARE FORMULA-ONLY (NEVER OVERWRITE)
- The Total columns are always formula-only and must never be typed/pasted/overwritten:
  Block 1 Total = K
  Block 2 Total = T
  Block 3 Total = AC
  Block 4 Total = AL
- If you want "Final Qty" or "Buffered Qty", do it by formula in the template (in K/T/AC/AL).
  Codex must NOT write numbers into Total columns.

(B) EXACT NAME + UOM LOCK (MASTER LIST IS KING)
- Description must be an exact item name from Materials.txt or Labour.txt.
  Do not rename, shorten, paraphrase, or invent new names.
- UOM must match the master list item exactly.
  Do NOT invent a new UOM and do NOT change UOM to fit a calculation.

(C) WASTAGE/BUFFER VALUE FORMAT (NO CONFUSION)
- Wastage/Buffer is stored as a fraction:
  0.10 means 10 percent
  0.05 means 5 percent
- Do NOT write 10 to mean 10 percent.
- If Buffer is blank, treat it as 0.

(D) NO TEXT IN NUMERIC CELLS (PENDING GOES TO NOTE ONLY)
- Never put text like "PENDING", "OK", "-", "TBC" inside numeric cells:
  Qty, Price/Unit, or Total columns.
- Put all pending/status/explanations in Note column only:
  Block 1 Note = L
  Block 2 Note = U
  Block 3 Note = AD
  Block 4 Note = AM

(E) UPDATE-ONLY (NO DELETE)
- Never delete old useful lines.
- If an old item becomes obsolete due to spec change:
  set Qty = 0 and write "OBSOLETE" with a reason in Note.
- Add new rows below existing ones when adding new items.

(F) NOTES AND FLAGS (USE THESE KEYWORDS ONLY)
Codex must use the following Note prefixes consistently:
- "CALC NOTE:" show what was calculated and the numbers/formula used.
- "ASSUMPTION:" a default used (allowed only if README or A:C defines the default).
- "INPUT NEEDED:" only when a required numeric input is missing and cannot be derived.
- "CONFLICT:" only when SXX A:C contradicts itself or is ambiguous.

Do NOT use "SPEC WARNING" as a general label. Use the four keywords above only.

(G) PACKAGING / CONVERSION RULE (NO DUMB QUESTIONS)
- If a conversion is needed (example: kg/sqm to BAG, L to SET, mm to LTH),
  Codex must parse pack size directly from the chosen master-list item name/description when present
  (examples: "25kg", "20kg", "5.8m", "1m").
- Codex must NOT ask for pack size if it is already present in the item description.
- "INPUT NEEDED" is allowed only if:
  1) conversion is required, AND
  2) pack size is not present in the item description, AND
  3) neither SXX A:C nor README provides the missing numeric value.

============================================================
4) BLOCK PURPOSES AND CALCULATION STANDARD
============================================================

This section defines what each block means and how Qty must behave.
Codex must follow these standards so that:
- Block 1 can be aggregated across S01..S06 (usage planning),
- Block 2 can price a single scope standalone (customer selects only S01),
- Labour and Rental/Service remain usage-based (man-hour / machine-hour planning).

------------------------------------------------------------
4.1 Block 1 (E:L) - MATERIAL USAGE (NO RM)
------------------------------------------------------------
Purpose:
- Stores usage quantities for aggregation across multiple scopes (S01..S100).
- This block is for consumption planning, not for minimum supplier purchase rounding.

Rules:
- Description: exact item name from Materials.txt (or Labour.txt if it is a service-type item).
- UOM: must match the chosen master list item.
- Qty (usage): decimal is allowed (example: 0.96 BAG is allowed as "usage equivalent").
- Price/Unit: keep blank (NO RM block).
- Wastage/Buffer: fraction (example 0.10).
- Total column K: formula-only (template decides what it shows).
  Recommended template meaning (example):
  - K = Qty * (1 + Buffer) to show buffered usage
  But Codex must never type/paste into K.

Notes:
- Use "CALC NOTE:" to show the basis (areas, lengths, pack parsing, etc).
- Use "ASSUMPTION:" only if the assumption is defined in README or A:C.

------------------------------------------------------------
4.2 Block 2 (N:U) - MATERIAL STANDALONE / PRICING (RM)
------------------------------------------------------------
Purpose:
- Stores supplier purchase quantities and unit rates for standalone scope pricing.
- This block must work even if customer selects only one scope (example: only S01).

Rules:
- Description: exact item name from Materials.txt (purchase item).
- UOM: must match master list.
- Qty (purchase): must be rounded up to whole purchase units where applicable
  (BAG/SET/LTH/PCS etc), because supplier sells full units.
- Price/Unit: read from Materials.txt rate if available.
- Wastage/Buffer: optional, fraction (use only if you intentionally add purchase buffer).
- Total column T: formula-only (template computes RM).
  Recommended template meaning (example):
  - T = Qty * Price/Unit (and optionally apply Buffer by formula)
  But Codex must never type/paste into T.

Notes:
- Block 2 is the place to do CEILING to supplier pack units.
- Block 1 is NOT the place to do CEILING.

------------------------------------------------------------
4.3 Block 3 (W:AD) - LABOUR USAGE ONLY (PLANNING)
------------------------------------------------------------
Purpose:
- Labour is planned using usage units (man-hours / crew-hours), then aggregated.
- This template does NOT require a "labour standalone pricing" block.

Standard:
- Preferred UOM for labour planning is MH (man-hour).
- However, UOM must still follow the chosen master list item.
  If Labour.txt only provides DAY items, then:
  - use DAY as UOM, and
  - in Note write the assumed hours/day used for later conversion.

Rules:
- Description: exact labour item name from Labour.txt.
- Qty: usage quantity (decimal allowed).
- Price/Unit: normally blank unless your Labour.txt provides rates (optional).
- Buffer: fraction if needed.
- Total column AC: formula-only (template may show buffered qty or RM if rated later).
  Codex must never type/paste into AC.

Notes:
- If you want all labour in MH, ensure Labour.txt has MH-based items.
- Codex must not invent MH if the master list item is not MH.

------------------------------------------------------------
4.4 Block 4 (AF:AM) - RENTAL / SERVICE USAGE ONLY (PLANNING)
------------------------------------------------------------
Purpose:
- Rental and service are planned as usage (machine-hours, days, trips, lots), then aggregated.
- No standalone pricing block is required in this workflow.

Rules:
- Description: exact rental/service item name from Labour.txt.
- UOM: must match master list (HR/DAY/TRIP/LOT/etc).
- Qty: usage quantity (decimal allowed).
- Price/Unit: normally blank unless Labour.txt provides rates (optional).
- Buffer: fraction if needed.
- Total column AL: formula-only (template may show buffered qty or RM if rated later).
  Codex must never type/paste into AL.

------------------------------------------------------------
4.5 Usage vs Standalone - Conversion and Rounding Rule
------------------------------------------------------------
Codex must follow this conversion standard:

- First compute the physical requirement from SXX A:C (areas, lengths, counts).
- Apply Wastage/Buffer as a factor (fraction) consistently:
  - Either keep Buffer separate and let Total formula apply it, OR
  - If the template expects Qty already includes buffer, then Buffer must be 0.
  (Do not double-apply buffer.)

- For Block 1 (Usage):
  - Keep usage as-is (decimal allowed).
  - Do NOT round up to whole supplier packs per scope.

- For Block 2 (Standalone/Pricing):
  - Convert to purchase units and round up to whole packs per scope.

Packaging / pack size extraction:
- Pack size must be parsed from the exact item name when present (25kg, 20kg, 5.8m, 1m).
- Do not ask the user for pack size if it is visible in the master list item name.

------------------------------------------------------------
4.6 When to Ask vs When to Compute
------------------------------------------------------------
Codex must compute without asking questions when:
- SXX A:C provides the needed geometry and quantities, AND
- the chosen master list item provides its UOM and packaging size (if conversion is needed), AND
- README provides any needed global defaults (if you set them).

Codex must use "INPUT NEEDED:" only when a required numeric value is truly missing AND cannot be derived:
- Missing area/length/count needed to compute Qty, OR
- Conversion requires pack size but item name does not contain it, OR
- A:C and README provide no default/coverage for a consumable rate.

Codex must use "CONFLICT:" only when SXX A:C contradicts itself (example: two different tile sizes).

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
Deliverable requirement (no hallucinations):
- If Codex edits via GitHub/PR mode, it must provide the actual PR URL.
  Do not claim "commit/PR created" without a real PR link.
- If Codex cannot create a PR, it must output manual paste TSV using the TWO-RANGE method
  for the required blocks (never touching Total columns).

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

## Core Rules: UOM, Packaging, and Conversions (MUST FOLLOW)

1) Exact Item Name
- Every row must use an exact item name from Materials.txt / Labour.txt.
- Do not rename, shorten, or paraphrase item names.

2) UOM Lock
- The UOM in BOQ must exactly match the UOM of the chosen master-list item.
- Never invent a new UOM and never change UOM to "fit" a calculation.

3) Packaging Parsing (No Questions)
- If a conversion is required (e.g., kg/sqm -> BAG, sqm -> SET, mm -> LTH),
  parse the pack size directly from the item description text (examples: "25kg", "20kg", "5.8m", "1m").
- Do NOT ask for pack size if it is already present inside the item description.
- Ask for INPUT NEEDED only when the pack size is not present AND conversion cannot be completed.

4) Usage vs Standalone
- Material Usage block = consumption for combined scopes (S01..S06). Decimal quantities are allowed in supplier UOM.
  Example: 0.96 BAG is allowed as usage.
- Standalone/Pricing block = customer selects only that scope. Quantities must be rounded up to purchase units
  (CEIL to whole BAG/SET/LTH/PCS as applicable).

5) Notes and Flags
- Use "CALC NOTE:" to explain formulas, assumptions already written in Sxx A:C, and conversion steps.
- Use "INPUT NEEDED:" only when a required numeric input is missing from Sxx A:C and cannot be derived.
- Do not use "SPEC WARNING" for normal calculations. Only use it when A:C conflicts or is ambiguous.

6) Do Not Rewrite Spec
- Sxx A:C is the spec. Do not rewrite spec text into blocks.
- Only compute quantities and write audit notes (CALC NOTE / INPUT NEEDED).

============================================================
8) HANDOVER (FOR NEW AI SESSION)
============================================================

Paste this at the start of a new chat:

HANDOVER:
- We use BOQ.xls with sheets S01..S100 + SUMMARY. Follow README FINAL strictly.
- SPEC is A:C only. Spacer columns D/M/V/AE must remain blank.
- There are 4 fixed 8-column blocks:
  Block 1 E:L (Total K formula-only)
  Block 2 N:U (Total T formula-only)
  Block 3 W:AD (Total AC formula-only)
  Block 4 AF:AM (Total AL formula-only)
- Never overwrite Total columns K/T/AC/AL. Manual paste must skip Totals using TWO-RANGE method.
- Exact item names must match Materials.txt / Labour.txt exactly. UOM must match master list (UOM lock).
- Wastage/Buffer is a fraction (0.10 = 10%). Never write 10 to mean 10%.
- Block meanings:
  Block 1 Material Usage = usage only (decimal allowed; no rounding to packs per scope).
  Block 2 Material Standalone/Pricing = purchase units (round up to packs per scope).
  Block 3 Labour = usage only (prefer MH if master list provides it).
  Block 4 Rental/Service = usage only.
- Notes must use these prefixes only:
  CALC NOTE, ASSUMPTION, INPUT NEEDED, CONFLICT. Do not use "SPEC WARNING".
- Update-only: never delete old useful rows. Obsolete -> Qty=0 + Note.
- Codex is the primary writer (fills E:AM blocks). GPT is checker/auditor and spec clarifier only.
- If using GitHub mode, only claim PR if a real PR link is provided; otherwise output TSV paste ranges.

END HANDOVER.