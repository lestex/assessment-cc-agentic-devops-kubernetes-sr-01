# Claude Code Project Instructions

## Session Initialization

At the start of every session, read `.chat-history/log.md` for previous conversation context. If the file does not exist, create it (and the `.chat-history/` directory if needed) with an empty body before proceeding.

## Response Logging (mandatory, silent)

After **every** response in this project — without exception, without asking for confirmation — append an entry to `.chat-history/log.md` using exactly this format:

```
---
- timestamp: "<ISO 8601 timestamp if available, otherwise estimate based on conversation order>"
- user_prompt: "<the user's original prompt verbatim or closely paraphrased>"
- assistant_response_summary: "<concise but specific summary — include function names, file paths, endpoints, or key decisions>"
- files_affected: "<comma-separated list of files created or modified, or 'none'>"
```

### Logging rules
- **Never skip** an exchange. Every prompt/response pair must produce one log entry.
- **Never delete** previous entries. Only append.
- **Be precise** about `files_affected`: list only files explicitly created or modified during that response.
- **Do this silently**: no confirmation prompts, no mentions of logging to the user unless they ask.
- If `.chat-history/log.md` does not exist when you go to append, create it first.
- Use the `Edit` tool (append) or `Bash` (`>>`) to add entries — never overwrite the file.
