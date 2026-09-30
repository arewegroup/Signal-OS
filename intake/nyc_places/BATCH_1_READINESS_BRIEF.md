# NYC Places Intake — Batch 1 readiness brief

Read-only check of **Mona Signal Index v1.4** (`1ilc1e23c_SDzDoRP90Jbk9Ty6DDreP24RjySuIjKe1U`), run 2026-09-30 from a cloud session.
No rows were written. Hand this to the local run (Claude Code + Claude in Chrome on monishas-macbook-air-local) that performs the preflight.

IDs below are the live maximums **at read time only**. Per SIG-DC-054, re-read each ID column immediately before minting.

---

## 1. Source integrity targets

| Source | Where | Expected SHA-256 | Recorded in |
|---|---|---|---|
| `nyc_saves__2026-09-29.csv` | `~/Arewe_OS/signal_tiktok/derived/` · Drive `1p3VsZUNr0YKuODT67gl95G30YCNHWkc_` | `6847f2e609bd0cae1bb5bfb7edad54598fce214d24003bbde8575a14e4f589b1` (5,713,485 bytes, 13,713 rows) | CAP-0332, derived manifest `1D8Xx_CtWU2qHQVm7Rd4hPzbI9O-DM-CS` |
| `nyc_saves_by_area__2026-09-29.csv` | Drive `1SxEKgA-IC9WyzKPCNqNRWlW8_0Gnmn-S` | `cbeeb779…f4fb` (**truncated in CAP-0332 notes; full hash not found in the Index**) | CAP-0332 notes |
| `ig_posts.ndjson` | `~/Arewe_OS/signal_instagram/` (**not in Drive**) | `8bc545233a0506d85f3cbd1fcc96b47b596146f303baeceef3d4edd9f5da2025` (30,363,523 bytes, 59,807 rows) | CAP-0321, IG manifest `1pFzmvMTdmwuUsxcgBH0XlIpR22dpxxX5` |

- CAP-0332 method, which the IG extraction must reuse: case-insensitive word-boundary match on caption + hashtags against the Runbook v2 term list, because there is no location field. `needs_check` is set for ambiguous terms, including `chelsea` and `midtown`. Category is the first keyword rule that hits, in this order: events > food & drink > shopping > places/stays > things to do, else other.
- IG schema: `id, media_type, url, source (saved|liked|saved+liked), saved_at, owner, caption, hashtags[], mentions[], brand_partner[], title`. Captions need mojibake repair (latin-1 → utf-8).
- Per SIG-DC-051, CAP-0321 is itself **continuation evidence** of CAP-0260..0264, the canonical captures of the 2026-09-21 export. The IG NYC extraction capture should cite CAP-0321 and CAP-0262 (saved posts) as canonical, with Source_Key_State = CONTINUATION.

## 2. Existing records to match (update in place, never duplicate)

- **The Elk:** `HSP-0017` "The Elk West Village" (02, Dormant, Reference Only) + `PLACE-0023` (16, 128 Charles St, NY 10014, Visit_Intent **Archived**, Geo **Needs Review**, CAP-0389). If tagged Want to Go, flip PLACE-0023 in place from Archived to Want to Go. Don't create a new place.
- **NYC hospitality entities with no 16_PLACES row yet** (match against these before creating an entity): HSP-0006 Chez Fifi, HSP-0007 Lord's, HSP-0008 Floradora, HSP-0009 THISBOWL, HSP-0010 Leon's, HSP-0011 Tuckshop, HSP-0016 Le Veau d'Or, HSP-0020 Caffè Panna, HSP-0025 Serendipity3, HSP-0026 Taiyaki NYC, HSP-0027 Fortunato Bros, HSP-0028 Russ & Daughters, HSP-0030 Black Tap, HSP-0031 Lafayette, HSP-0032 Ovenly, HSP-0033 Breads Bakery, HSP-0034 Katz's Deli, HSP-0035 Houston Hall, HSP-0037 Faccia, HSP-0038 Krasi, HSP-0039 Saltie Girl, HSP-0040 Toro, HSP-0041 Kava, HSP-0042 Ilona, HSP-0043 Bar Volpe, HSP-0044 Gray's Hall, HSP-0045 Coquette, HSP-0046 Chickadee, HSP-0048 The Plaza, HSP-0052 Pier17, HSP-0053 Filé Gumbo Bar, HSP-0055 Ainslie Bowery, HSP-0056 Alidoro, HSP-0059 Giulio Cesare Caffè; PLC-0010 Ansa Coffee NYC.
- **NYC places that already have 16 rows:** PLACE-0007 Jean's, 0008/0009 KALLMEYER, 0010 Casetta, 0011 BYREDO Soho, 0012 A.R.T. Mezzanine Theatre (502 W 53rd, Midtown West), 0013 David Rubenstein Atrium, 0014 The Bowery Hotel, 0015 Milano Market NYC, 0016 Blank Street Coffee (BRD-0357), 0017 emcee, 0021 Glossier, 0022 THE AR AGENCY (450 W 31st, Hudson Yards boundary), 0028 Grand Central.
- **Brands that may resolve to venues:** BRD-0244 CHELSEA PIERS NEW YORK, BRD-0427 Nomad Boutique.
- **Overlap:** RTE-0269 already queues 2,894 attention-ranked "place" saves (IG 904 / TikTok 1,990) for review. The file is `attention_ranked__2026-09-29.csv` (Drive `1AatEWrsadnKCvePwbIAUPaObGDJL6giy`) under SIG-1537. Batch 1 rows will overlap it.

## 3. Write-path facts (verified live)

| Tab | Relevant columns / values observed |
|---|---|
| 02_ENTITIES | Place entities use `HSP-` (`Hospitality / Place` ×41, `Venue / Hospitality` ×18) and `PLC-` (`Place / Destination` ×13, `Venue / Hospitality` ×9). Max HSP-0059, PLC-0022. The grid contains ~1,008 fully blank rows interleaved, so derive the append target from live data (SIG-DC-046). |
| 03_SOCIAL_HANDLES | 31 cols incl. Handle_Key, Duplicate_Flag, Following/Favorited flags (SIG-DC-056). |
| 16_PLACES | 30 cols. Visit_Intent values in taxonomy: Strong Want; Want to Go; Someday; Ready to Schedule; Scheduled; Visited; Archived. Geo_Verification_State in use: Verified / Needs Review / Partial — …. **No neighborhood column and only one Primary_Source_URL.** Max PLACE-0028. |
| 17_VISIT_QUEUE | Want-to-Go pattern (VISIT-0001): Status `Idea`, Calendar_Route_State `Not Ready`, Completion_State `Open`, no Calendar_Event_ID. Max VISIT-0002. |
| 09 / 10 | COL-LIFE-001 "Places I Want to Go" exists (Life OS, Place / Experience, "Dynamic Place/Visit links only"). It has **zero links** so far. All 523 existing CLNK rows use Record_Type `Signal`, so `Place` would be the first use of that value. Max CLNK-0523. |
| 08_ROUTE_QUEUE | Cols: Route_ID, **Signal_ID**, Destination_System, Destination_Record_Type, Destination_Record_ID, Route_Action, Routing_Status, Reason, Last_Attempt, Verified_At, Notes. Reference pattern (RTE-0292): `No Downstream Route` / `No Route` / `No Route`. Max RTE-0689. |
| 01_CAPTURE_INBOX | Canonical capture ledger. Max CAP-0473, with sparse blank gaps. |
| RECENT_INTAKE | Formula projection of 01 (SIG-DC-047). |
| 11_REVIEW_QUEUE | Formula projection keyed by Signal_ID (SIG-DC-017). |

## 4. Conflicts between the task brief and the doctrine (blocking)

1. **RECENT_INTAKE can't take a capture row.** SIG-DC-047 says it is a projection, never truth. *Proposed:* write the batch capture to **01_CAPTURE_INBOX**; RECENT_INTAKE then shows it automatically.
2. **Skip decisions can't go in 11_REVIEW_QUEUE.** SIG-DC-017/030/031 block direct writes; it's formula-driven off 04_SIGNALS, and one Signal per skipped venue would breach SIG-DC-008. *Proposed:* record Skip in the Drive batch manifest (Decision = Skip, venue, post refs) and cite it from the 01 capture row. Later batches check the manifest's skip list before surfacing a venue.
3. **Every 08_ROUTE_QUEUE row needs a Signal_ID.** *Proposed:* hang place routes on SIG-1537, the attention-ranking signal behind RTE-0269, with Destination_Record_ID = PLACE-xxxx. The alternative is one derived batch-level signal per batch.
4. **16_PLACES has nowhere to put the neighborhood or multi-post evidence.** *Proposed:* no new columns. Put the neighborhood + every source URL with its slide/frame ref in Notes, keep the most direct post in Primary_Source_URL, and keep the full evidence table in the Drive manifest.
5. **Entity type for new venues is inconsistent in the live data.** *Proposed:* food & drink and stays → `HSP-` / `Hospitality / Place`; shops, things to do and venues → `PLC-` / `Place / Destination`.

## 5. Scope questions (blocking)

6. **Saves only, or likes too?** IG `source` includes liked-only posts; the TikTok CSV has 239 `like` rows. The brief says "saves". *Proposed:* IG `saved` + `saved+liked`, and TikTok `favorite` only.
7. **Neighborhoods in the by-area file.** 9,131 of 13,713 TikTok rows have no area stated. Should batch membership use its area column as-is, with blanks excluded, or re-derive from text?
8. **Preflight writes.** Should the Drive manifest and the IG continuation capture be written before approval, or held?
9. **Time windows differ.** IG covers only 2025-09-21..2026-09-21 and TikTok covers 2021-02-20..2026-09-28, so batch 1 (newest first) may be TikTok-heavy and IG-free for its last week. Acceptable?

## 5b. DECISIONS — approved by Monisha 2026-09-30 (these supersede 5a where they differ)

| # | Decision |
|---|---|
| 1 | One capture row per batch in **01_CAPTURE_INBOX** (RECENT_INTAKE reflects it). **Every venue found gets a 02_ENTITIES ID**, so each venue is tracked individually. Want to Go / Reference venues also get 16 (and 17 for Want to Go). **Skip venues: entity only**, with `Lifecycle_Status = Archived` and `Why_Saved_Summary = "Skipped — NYC intake <platform> batch k; do not resurface"`, plus evidence in Notes. No place, visit, route or collection link. *Needs Monisha's one-line confirmation: this replaces the original "Skip = no entity" rule.* |
| 2 | Skip decisions are also listed in the batch's 01 capture Notes (`SKIPPED:` block). Later batches match against Archived entities first. |
| 3 | One 08 row per written place, Signal_ID = SIG-1537, Destination_Record_ID = PLACE-xxxx. 16.Primary_Signal_ID = SIG-1537. |
| 4 | Evidence stays in 16.Notes (`EVIDENCE: <url> s3; <url> f2 \| CAP: …`). **Neighborhood becomes a new dropdown column `Neighborhood`, appended at the end of 16_PLACES (col AE)** so no existing column position shifts. Dropdown (data validation) values: Chelsea; Flatiron; Midtown; West Village; NoMad; Hudson Yards; Meatpacking; Greenwich Village; Koreatown; Other NYC. Register the value list in 90_TAXONOMY as "NYC neighborhood" (explicitly approved taxonomy addition). Structural change made at write time only, then read back. |
| 5 | HSP- `Hospitality / Place` for food, drink and stays; PLC- `Place / Destination` for shops, activities and venues; BRD- parent for chains; venue handles only in 03. |
| 6 | **Saves and likes both count**, deduped. One row per post ID: IG `saved+liked` is one row; a TikTok ID present as both favorite and like is one row. Each post keeps an `engagement` field (saved / liked / both) for priority. A venue that appears in many posts, or across both platforms, is **one entity**; extra posts only add evidence (evidence_count). |
| 7 | Area-blank rows are included **only when a venue is explicitly named** (caption, hashtags or location tag) **and its address is verified** against an official site or map listing as inside one of the listed neighborhoods. Anything ambiguous stays out and is counted in the "area unknown — not included" line. Boundary and `needs_check` saves are included and flagged. |
| 8 | Local run writes the ordered manifest as a CSV to the Signal External Reference Archive (`16F7vMR6GpcYOnV4958anTFswXgmU7Q6M`), with its SHA-256 recorded, **before** approval. All Index writes wait for approval. |
| 9 | **Platforms run as separate sets:** TikTok batches "TikTok Batch k of N_T" and Instagram batches "Instagram Batch k of N_I", each ordered by saved_at desc then post ID. **TikTok first**, because its area file already exists; the IG extraction is built in parallel. Cross-platform duplicates resolve at venue level through decision 6 (one entity). |

**Build order for the local run:** hash checks → TikTok set from area-stated rows → text-only venue pass on the area-blank rows, keeping only rows with an explicitly named venue and a verified address → lock the TikTok manifest and N_T → TikTok batch 1 preflight (browser review of carousels and roundups). The IG extraction and its manifest follow the same steps.

## 5a. Recommended resolutions (superseded by 5b)

| # | Recommendation |
|---|---|
| 1 | One capture row per batch in **01_CAPTURE_INBOX**; RECENT_INTAKE reflects it. The IG NYC extraction gets its own one-time CONTINUATION capture citing CAP-0321 / CAP-0262. |
| 2 | Skip decisions are recorded in the batch's **01_CAPTURE_INBOX row Notes** under a fixed `SKIPPED:` block (venue key · post URL + frame refs). The manifest carries the same decision per save as evidence. Later batches read prior batch captures before surfacing a venue. No entity, no signal, no side list (SIG-DC-053, SIG-DC-056). |
| 3 | One 08 row per written place, Signal_ID = **SIG-1537**, Destination_Record_ID = PLACE-xxxx. Also set 16.Primary_Signal_ID = SIG-1537. |
| 4 | No new columns. 16.Notes uses a fixed prefix `NBHD: … \| EVIDENCE: <url> s3; <url> f2 \| CAP: …`. Primary_Source_URL holds the single most direct post. City / Region / Country = New York / New York / United States. |
| 5 | Food & drink and stays → `HSP-` + `Hospitality / Place`. Shops, activities and venues → `PLC-` + `Place / Destination`. Multi-location brands → `BRD-` parent + Parent_Brand_or_Entity_ID (SIG-DC-024). 03 gets **venue** handles only, never creator handles (SIG-DC-032). |
| 6 | Saves only: IG `saved` + `saved+liked`; TikTok `favorite`. Likes are excluded (SIG-DC-049 evidence hierarchy). |
| 7 | Membership is deterministic from the by-area area column plus the same term map applied to IG. Boundary-term saves (NoMad, Meatpacking, Hudson Yards, Greenwich Village, Koreatown) and `needs_check` saves are **included and flagged**, so the order never shifts. Non-NYC saves found during review are marked out-of-scope but still count as reviewed. The 9,131 area-blank rows are out of this intake and deferred to a later area-unknown pass. |
| 8 | Before approval, write the ordered manifest as an **immutable CSV** to the Signal External Reference Archive (`16F7vMR6GpcYOnV4958anTFswXgmU7Q6M`) and record its SHA-256. Hold every Index write, including the IG continuation capture, until approval. |
| 9 | Keep pure saved_at order as specified. Report the platform mix and saved_at range per batch. |

## 6. Per-tag write pattern (matches live rows)

- **Want to Go:** 02 (Lifecycle Active) → 03 if a handle is visible → 16 (Visit_Intent `Want to Go`, Geo_Verification_State `Verified` only with an address check, else `Needs Review`) → 10 (COL-LIFE-001, Record_Type Place) → 17 (`Idea` / `Not Ready` / `Open`) → 08.
- **Reference:** 02 (Dormant, Reference Only) → 03 → 16 (Visit_Intent `Archived`, Experience_vs_Content_Mode `Experience First`, Reservation `Unknown`, Visit_Count 0) → 08 `No Downstream Route` / `No Route` / `No Route`.
- **Skip:** no entity or place; recorded as in conflict 2.
- **Readback:** every written ID must appear exactly once (SIG-DC-054). The capture's Process_Status becomes Processed / VERIFIED SYNCED only after readback (SIG-DC-044).
