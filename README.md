# Voiceguard.ai
Voiceguard.ai is a AI powered real time detection and prevention of voice cloning impersonation attacks
VoiceGuard AI

Verify a voice before you trust it.

VoiceGuard is a full-stack web app for detecting AI-generated and deepfake voice clones, aimed at protecting people (especially in India, given the Hindi/English handling and Aadhaar/PAN/KYC awareness) from voice-cloning scams — the kind of call where someone impersonates a relative, bank official, or authority figure to pressure a victim into sending money or sharing sensitive information.

Core capabilities

Voice authenticity detection

Runs uploaded or recorded audio clips through a local deepfake-detection model (lab260/Spectra-AASIST3, an AASIST3 variant) and/or the Hive AI "AI-Generated & Deepfake Content Detection" API, producing a 0–100 risk score and a verdict (genuine / suspicious / AI-generated).
Supports enrolled voiceprints per customer (voiceprint.py), so a call can be compared against a known reference voice, not just judged in isolation.

Scam-pattern & transcript analysis

Transcribes calls (Google Cloud Speech-to-Text, English/Hindi with automatic fallback) and scans the text for a curated list of scam phrases — OTPs, "urgent transfer," "account suspended," lottery/prize language, KYC/Aadhaar/PAN references, remote-access tool names like AnyDesk/TeamViewer, etc.
Scans for PII exposure (card numbers, bank details) and boosts the risk score when scam keywords or PII are detected, with a floor/cap so keyword matches alone can push a call into "suspicious" territory even before the audio model runs.

Batch analysis

/api/batch-analyze processes up to 20 files at once through the same analysis pipeline as single-file uploads, with a results table and a downloadable CSV summary report.

Provenance & audit trail

Every analysis is logged with a content hash (SHA-256), source, format, duration, detection engine/model version, and a full timestamped event history (created, re-uploaded, re-analyzed, flagged, unflagged) — implemented client-side in localStorage plus a parallel backend record in SQLite, so the log survives independent of whether the backend copy is reachable.
A tamper-evident security audit log (/api/security/audit) with a verify endpoint to check log integrity.

Shareable verification reports

Generates an immutable, tokenized public report page (/verify/{token}) — a standalone HTML "Verified by VoiceGuard" badge page with risk score, classification, and a short (≤280 char) transcript excerpt — that can be copied, shared, or printed as proof a call was checked, without exposing raw audio or the full transcript. Tokens expire after a 90-day TTL and return a friendly 404 once gone.

Review workflow & analytics

A review-case queue (/api/review-cases) for flagged calls, with notes, so a human reviewer can triage suspicious detections.
An analytics dashboard (/analytics/summary) tracking risk trends over time, verdict distribution, and average risk scores.
Guided "safety sessions" (/api/safety-sessions) and a feedback endpoint for closing the loop on detection accuracy.
Architecture
Backend: Python/FastAPI (main.py, detector.py, db.py, voiceprint.py) with SQLite storage, optional Hive and Google STT integrations configured via backend/.env.
Frontend: vanilla HTML/CSS/JS (no framework) — index.html, script.js, analytics.html, review-queue.js, plus a three.min.js-driven background visual (bg-signal.js) — styled with a dark, security-dashboard aesthetic (risk rings, donut charts, trust badges).
No raw audio is ever persisted for reports or provenance — only metadata, hashes, and short transcript excerpts, by design.
