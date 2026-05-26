# Agent Skills

A personal collection of high-quality, reusable skills for AI agents (Grok, Claude, Codex, and others).

## Skills

| Skill | Description |
|-------|-------------|
| [youtube-summarizer](./youtube-summarizer) | Takes any YouTube video URL and produces a structured, high-quality summary with TL;DR, key points, and timestamps. Uses yt-dlp (preferred) or youtube-transcript-api, with intelligent chunking for long videos. |
| [competitor-analysis](./competitor-analysis) | Given a company name (optionally with product/category focus or "vs X"), delivers a structured competitor & market analysis including key players, positioning, recent signals, differentiation, and risks/opportunities. Supports quick overviews and deep research. |

## Installation

### As a User Skill (available in all projects)

```bash
# Clone this repo (or just the skill you want)
git clone https://github.com/grivera82/agent-skills.git

# Copy the skill(s) you want into your user skills directory
cp -r agent-skills/youtube-summarizer ~/.grok/skills/
```

### As a Project Skill (only for a specific repo)

Copy the desired skill folder into your project's `.grok/skills/` directory:

```bash
mkdir -p .grok/skills
cp -r /path/to/agent-skills/youtube-summarizer .grok/skills/
```

After adding or updating skills, most agent tools will discover them automatically when placed in the correct directory for that tool.

## Requirements

Most skills in this collection have no external dependencies beyond what's already available in typical agent environments. When a skill benefits from a specific CLI tool (e.g. `yt-dlp` for youtube-summarizer), it is documented in the skill's README or SKILL.md.

## Adding New Skills

Each skill lives in its own top-level folder containing a `SKILL.md` file.

To add a new skill:

1. Create a new directory: `mkdir my-new-skill`
2. Write `my-new-skill/SKILL.md` following the standard agent skill format
3. Update this README with a short description
4. Commit and push

## License

MIT — feel free to use, modify, and share any skill in this collection.
