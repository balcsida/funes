# Ingesting external sessions

`funes ingest [PATH|-] [--no-thinking]` imports sessions produced by external integrations. It
reads newline-delimited JSON from `PATH`, or standard input when the path is omitted or `-`.

Each line is one complete version 1 session:

```json
{"version":1,"harness":"opencode","session_id":"ses_example","cwd":"/work/project","turns":[{"turn_uuid":"msg_example","parent_uuid":null,"seq":0,"ts":"2026-09-17T12:00:00Z","role":"user","blocks":[{"block_type":"text","text":"Explain this project","tool_name":null,"tool_use_id":null}]}]}
```

Funes validates the entire input before changing memory. The input limit is 64 MiB; timestamps must
be RFC 3339; harness, role, and block-type names must match `[a-z][a-z0-9_-]*`; IDs must be nonempty;
and turn sequence numbers must increase within a session. Session and turn IDs are stored as
`<harness>:<id>`, so external IDs cannot collide with another harness.

Imports use the normal redaction, chunking, embedding, and append-only deduplication pipeline.
Re-sending the same complete session is a no-op. Send only stable, completed message content;
pending or streaming tool content should wait for a later complete-session import. `--no-thinking`
excludes thinking blocks just as it does for `funes index`.
