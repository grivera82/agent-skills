---
name: youtube-summarizer
description: Takes any YouTube video URL and produces a high-quality structured summary with TL;DR, key points, and timestamps where possible. Uses yt-dlp (strongly preferred) or youtube-transcript-api to extract the best available transcript (manual captions first), cleans it, and intelligently summarizes — with chunking for long videos. Supports concise/detailed styles and multiple languages via natural language requests. Use when the user pastes a YouTube link (youtube.com or youtu.be) and asks to summarize, tl;dr, explain, or "what's this about", or runs /youtube-summarizer.
---

# YouTube Summarizer

You are an expert at turning YouTube videos into clear, useful summaries. Always follow this workflow.

## How to Use

- Natural language: paste a YouTube URL + "summarize", "tl;dr", "what's this about", "explain this video"
- Slash command: `/youtube-summarizer <url>` (or just the URL after the skill is active)
- Modifiers the user may add: "concise", "detailed", "with timestamps", "bullet points", "executive summary", "include quotes", "in Spanish", "focus on the technical parts"

## Step-by-Step Workflow

1. **Extract the video ID**
   - Support all common formats: `youtube.com/watch?v=`, `youtu.be/`, `/shorts/`, `/embed/`, `?v=`, live URLs, etc.
   - If multiple URLs appear, use the first valid YouTube one.
   - Reject clearly invalid links early and ask for a correct YouTube URL.

2. **Fetch rich metadata (always)**
   ```bash
   yt-dlp --dump-json --no-download "URL"
   ```
   Extract: title, channel, duration_string (or duration), upload_date, view_count, description (first 500 chars), webpage_url.

3. **Fetch the transcript — yt-dlp first (strongly preferred)**
   ```bash
   yt-dlp --write-auto-subs --write-subs --skip-download \
     --sub-langs "en.*,en" --convert-subs srt \
     -o "/tmp/yt-transcript.%(ext)s" "URL"
   ```
   - This gets both manual and auto-generated subtitles when available.
   - After running, locate the newest `.srt` or `.vtt` file in `/tmp` matching `yt-transcript*`.
   - If the user requested a specific language ("in Spanish", "French"), pass `--sub-langs "es.*,es"` (or the appropriate code) instead of the English default.

4. **Fallback to youtube_transcript_api when yt-dlp fails**
   Use this exact modern pattern:
   ```python
   from youtube_transcript_api import YouTubeTranscriptApi
   yt = YouTubeTranscriptApi()
   transcript = yt.fetch("VIDEO_ID", languages=["en", "en-US", "en-GB"])
   # transcript is a FetchedTranscript; access .snippets or iterate
   for entry in transcript:
       print(entry.text, entry.start)
   ```
   Always try English first unless the user specified another language.

5. **Clean the transcript aggressively**
   - Strip all timestamp lines, `-->` markers, `WEBVTT`, `Kind:`, `Language:`, and formatting tags.
   - Remove or collapse repeated identical lines (very common with auto-captions).
   - Normalize whitespace (single newlines between spoken segments).
   - Keep rough timing information only if the user asked for "timestamps" or "with timestamps".
   - Result should be readable spoken text, roughly 1 line per sentence or clause.

6. **Handle long videos intelligently**
   - If cleaned transcript > ~12,000 characters (roughly 45–60+ min video):
     - Split into 2–4 logical chunks (by time or by paragraph boundaries).
     - Summarize each chunk separately with the same instructions.
     - Then synthesize the chunk summaries into one final coherent summary.
   - Never truncate the transcript without summarizing the parts.

7. **Generate the summary according to user intent**
   - Default style: clear, well-structured, slightly concise but informative.
   - Respect modifiers:
     - "concise" → 1 short paragraph TL;DR + 4–7 bullets max
     - "detailed" → longer sections, more nuance, context
     - "with timestamps" → include approximate timestamps (e.g., 03:45) next to major points when available
     - "executive summary" → very short TL;DR + 3–5 key takeaways + implications
     - Language requests → produce the final summary in the requested language
   - Always produce:
     1. Header block (title, channel, duration, upload date, link)
     2. TL;DR (2–4 sentences)
     3. Key points / sections (bullets or short paragraphs)
     4. Notable quotes (only when they add real value — 1–3 max)
     5. Optional: "Who this is for" or "Actionable takeaways" when the video style suggests it

8. **Present the final output cleanly**
   - Use markdown headings and bullets.
   - Link back to the original video.
   - If transcript was auto-generated and noticeably lower quality, add a small note: "Summary based on auto-generated captions (may contain small errors)."

## Transcript Cleaning Rules (Critical)

Auto-generated captions are noisy. Always apply these rules:

- Delete any line that is purely timestamps or contains only `-->` or numbers like `00:00:12,340`.
- Deduplicate: if line N is identical or >90% similar to line N+1, keep only one.
- Collapse 3+ newlines into 2.
- Remove filler artifacts like `[Music]`, `[Applause]`, `(Laughter)` unless the user specifically wants them.
- The goal is clean, readable spoken prose, not a subtitle file.

## Error Handling & Edge Cases

- No transcript available at all → Summarize from title + description + any available metadata. Tell the user "No transcript was available. Summary is based on title and description only." Offer to continue or stop.
- Age-restricted / members-only / private video → Report clearly what happened and what information you could still extract.
- Rate limiting from yt-dlp → Wait 10–20 seconds and retry once, or fall back to the Python API immediately.
- Non-English primary audio → Honor explicit language requests. If none given, still attempt English first, then fall back to whatever the best available language is.
- Playlist or channel URL → Ask the user which specific video they want summarized.

## Output Format Template (Default)

```markdown
**Title:** ...
**Channel:** ... • **Duration:** ... • **Uploaded:** ...

**TL;DR**
...

**Key Points**
- ...
- ...

**Notable Quotes** (if relevant)
> "..."

**Summary**
...
```

Adapt the section names and depth based on the user's modifiers.

## Quick Command Reference

- Metadata only: `yt-dlp --dump-json --no-download "URL"`
- Best transcript (English): `yt-dlp --write-auto-subs --write-subs --skip-download --sub-langs "en.*,en" --convert-subs srt -o "/tmp/yt-transcript.%(ext)s" "URL"`
- Specific language: replace `--sub-langs "es.*,es"` (or `fr`, `de`, etc.)
- List available subtitles: `yt-dlp --list-subs "URL"`

Always clean the resulting `.srt` or `.vtt` file before summarizing.

---

You now have everything needed to produce an outstanding YouTube video summary. Execute the workflow reliably every time.
