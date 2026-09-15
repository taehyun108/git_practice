---
name: drum-practice-audio-extractor
description: Implement, review, and verify the Windows drum-practice audio separation tool defined in the repository PRD. Use for changes involving Demucs stems, drum/no-drums MP3 generation, local or authorized YouTube inputs, batch execution, or Windows packaging.
---

# Drum Practice Audio Extractor

Use this skill when working on the drum-practice audio separation project. Before changing behavior, read the repository root `PRD.md`; treat it as the product contract and keep unresolved items there unresolved unless the user decides them.

## Product invariants

- The supported product is a Windows Python application that can run on CPU and use CUDA when available.
- Inputs are local `.mp3`, `.wav`, `.flac`, and `.m4a` files under `input/`, plus URLs explicitly supplied by the user in `youtube_urls.txt` or `--url`.
- YouTube processing is limited to content the user owns or is authorized to download. Do not add login, cookie extraction, DRM, paywall, age, region, or access-control bypasses.
- Use Demucs four-stem separation: `vocals`, `drums`, `bass`, `other`.
- Generate exactly two final MP3 products per song: the `drums` stem and `vocals + bass + other`.
- Keep intermediate downloads and separation artifacts under `temp/`; clean them after success and best-effort after failure.
- Sanitize Windows-invalid filename characters and keep outputs under `output/<song>/`.
- A completed pair is skipped by default; `--force` explicitly requests replacement.
- A failure for one song must be logged and must not stop the remaining batch.

## Implementation guidance

1. Separate input discovery, authorized downloading, stem separation, audio mixing/encoding, orchestration, and logging so each boundary can be tested without a real model or network download.
2. Resolve paths relative to the project root, not the caller's current directory. Preserve Unicode filenames and use safe subprocess argument lists rather than shell interpolation.
3. Detect FFmpeg and the selected PyTorch device before expensive processing. Make forced-device behavior explicit and keep the fallback policy aligned with the current PRD decision.
4. Write outputs atomically where practical: encode to a temporary destination, validate that it exists, then move it into the song output directory. Avoid leaving a misleading completed pair after a partial failure.
5. Make model output path handling tolerant of Demucs version-specific folder layouts, but fail with a diagnostic that includes the expected stem names and searched paths.
6. Keep console progress concise and useful for a non-developer: current song, current phase, skip/error/completion, and final output paths.
7. Record timestamp, song, input type, success/failure, duration, and error details in `logs/app.log` without logging secrets or unauthorized authentication material.

## Verification workflow

Run lightweight checks first:

- Python syntax and import checks.
- Unit tests for filename sanitization, input discovery, output naming, duplicate detection, CLI parsing, logging, and temporary-file cleanup.
- Mocked tests for FFmpeg, yt-dlp, Demucs output discovery, device selection, and per-song error isolation.
- A dry or fixture-based end-to-end test that does not require downloading copyrighted content.

When tools are available, separately report whether FFmpeg, Demucs, PyTorch, and CUDA were actually detected. Do not claim a real audio separation run unless a permitted fixture or user-provided audio was processed successfully.

## Scope discipline

Do not add a GUI, cloud service, audio player, automatic song search, or commercial-content downloader unless the user changes the PRD. When a behavior is not specified, identify the exact open question and preserve a configurable seam instead of silently inventing product policy.
