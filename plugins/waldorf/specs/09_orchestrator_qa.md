# Orchestrator QA observations

Checks run against isolated `/private/tmp/xuvi-orchestrator-qa` at port 5184 during second implementation pass; this list tracks findings until fixed, not final status.

- `/api/ai/status`: configured true from `.env`.
- `/api/timetable`: 40 entries, 1350 instructional + 750 routine = 2100 minutes.
- `/api/grades`: 41 full source records.
- POST `/api/requirements`: 500 “9 values for 10 columns” (index.ts initial correction around line 33).
- POST `/api/requirements/bulk-from-pages`: 500 “8 values for 9 columns”.
- POST `/api/matrix/generate`: same 500. Add successful endpoint regression tests, not just invalid-body checks.
- SQL errors currently return Express HTML stack rather than JSON; global error handler needed, no internal paths/secrets in responses.
- Image endpoint returns WebP bytes despite no requested format; sniff content/type before saving instead of blindly labeling `.png`.
- Backup JSON is globally limited to 8MB while a single allowed media file can be 25MB (base64 bigger). Need route-specific realistic backup limit and explicit failure messaging.
- Restore commits DB before writing/removing media; filesystem failure can leave partially restored state. Stage media and validate all records before swap, preserve recovery copy/rollback.
- Attempt snapshot references images that scene edit/delete may remove because fileCleanup checks current scenes/attempt audio only. Include snapshot media references or store immutable copies. Retelling conditions must remain reviewable after source edits/deletes.
- AI scene route truncates sourceText at 24,000 chars while claiming coverage; process entire story within model limits or explicit chapter batches, never silently drop ending.
- `/api/coverage` counts saved mappings independently of `requirementId` links in timetable; merely changing lesson count does not update mapping validity. Need actual affected-requirement indicator and honest scheduled vs evidenced states.
- Active grade/year/week/student filters should govern output documents; default type 2 shouldn't export all 41 detailed programs when user wants one grade.
- DOCX visual QA: type 5 rendered successfully to four pages, Vietnamese glyphs correct, but table rows split across pages (page 2 starts mid-lesson) and 8 equal-width portrait columns are cramped. Use landscape for wide tables, proportional column widths, repeated header and `cantSplit` on rows; remove the duplicated 40-line “thẻ dạy gọn” from full weekly report and provide an actual selected-day teaching card. Use Vietnamese local date/time, readable spacing and user labels instead of `hoc/sinh-hoat`.
- App `/api/ai/transcribe` succeeded with the synthesized 8.64s Vietnamese WAV (upload -> saved path -> remote transcription). It returned transcript and turn timestamps. One repeated word demonstrates why editable transcript remains necessary.
- App `/api/stories/1/ai-scenes` succeeded live via Wonder: returned structured scenes with chapters/essential details for Hạt Mầm Ánh Sáng. This is verified through the application and saved in local DB.
- Fixed & verified: semantic AI retelling feedback (`POST /api/attempts/:id/ai-feedback` + UI button in `StoryDetail`, remote disclosure confirmation, uses FROZEN source/scenes/audience and corrected transcript; structured `strengths`/`omissions`/`adaptations`/`ageFit`/`nextStep`, `overallScore: null`, quotes evidence from actual inputs and distinguishes audio delivery not analyzed, with audio replay right alongside feedback; tested in unit, fake-provider integration, and live Meta AI check).
- Fixed & verified: Document exports (types 1–6) use human-readable Vietnamese labels and formatted dates (no raw JSON in type 4, no raw internal slugs in types 3/5/6), proportional DOCX table column widths with header shading and `cantSplit`, landscape `@page` print CSS for wide tables, and UI filter controls for `gradeId`, `day` (selected-day teaching card), and `studentId`.
- Fixed & verified: `client/public/images/garden-watercolor.webp` is displayed in the Dashboard hero, Story library header, and Scene 1 of Story 1 (`Hạt Mầm Ánh Sáng`), and all 28 seeded stories in `data/xuvi.db` have full non-truncated chapter-preserving scenes.

Independent Meta live checks and safe generated assets are documented in `08_meta_live_check.md`. Full provider text/image/voice successful; application-route testing remains separate.
