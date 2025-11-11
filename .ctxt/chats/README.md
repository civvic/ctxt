# Chat Transcript Archive

## Purpose
Permanent archive of significant development sessions capturing:
- Architectural decisions and rationale
- Design discussions and alternatives
- User preferences and workflow patterns
- Problem-solving approaches

## Two-Tier Strategy

**Tier 1 (Automatic):** SpecStory → `.specstory/history/`
- Ephemeral, comprehensive, gitignored
- ctxt discovers via ChatHistorySource

**Tier 2 (Curated):** Manual → `.ctxt/chats/`
- Permanent, git-tracked
- Important sessions only

## Workflow

After significant sessions:
1. Export from Cursor → `YYYY-MM-DD_topic.md`
2. Optional: Copy SpecStory → `YYYY-MM-DD_topic-complete.md`
3. Git auto-commits

## Naming Convention
- `YYYY-MM-DD_topic.md` (Cursor export)
- `YYYY-MM-DD_HH-MMZ-topic-complete.md` (SpecStory copy)
