# START HERE — Instructions for AI

You are given a repository codedump split into multiple volume files.

## Goal
Answer questions by opening the minimum necessary content.

## Format notes
- Use `==== FILE_INDEX ====` first (lines starting with `ENTRY`).
- Then jump to `----- FILE BEGIN -----` with matching `path="..."`.
- For big files, prefer `--- CHUNK BEGIN ---` blocks.


## How to navigate this dump
1) Open `Index.txt` (master index) and use it to locate the right volume.
2) Pick the relevant volume file.
3) Use the per-volume index section to locate the path.
4) Read the exact file content (or required chunks only).
5) Expand cautiously (imports / calls / routes), 1–2 hops unless needed.

## Rules
- Do NOT try to read the entire dump.
- Prefer docs/diagrams/indices if present.
- When answering, cite file paths and the volume filename.

## Files
- Instructions (this): `00_START_HERE.instructions.md`
- Master index: `Index.txt`
- Volumes:
- docs_20260227_111307_01_ROOT.txt  —  ROOT FILES
- docs_20260227_111307_02_20-nodes.txt  —  FOLDER: 20-nodes
- docs_20260227_111307_03_30-artifacts.txt  —  FOLDER: 30-artifacts
- docs_20260227_111307_04_10-system.txt  —  FOLDER: 10-system
- docs_20260227_111307_05_50-operations.txt  —  FOLDER: 50-operations
- docs_20260227_111307_06_90-reference.txt  —  FOLDER: 90-reference
- docs_20260227_111307_07_40-integration.txt  —  FOLDER: 40-integration
- docs_20260227_111307_99_OTHERS.txt  —  OTHERS (Misc Folders)

