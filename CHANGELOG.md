# Changelog

All notable changes to this project. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versions follow [Semantic Versioning](https://semver.org/). Each version is also a
[GitHub Release](https://github.com/bablobanov/hermes-agent-telegram-checklist/releases).

## [Unreleased]

- Repository renamed to `bablobanov/hermes-agent-telegram-checklist` to match the product name
  Hermes Agent; GitHub redirects the old address. No code change.

## [1.1.1] - 2026-09-25

### Security
- A per-topic allowlist entry (`-100...:33`) is now enforced on reads, not only on writes:
  `get` refuses a checklist that lives outside the allowed topics, and `list-topics` fetches only
  the allowed topics by id (`messages.GetForumTopicsByID`) instead of listing the whole forum.
  Before, both commands checked only the chat, so a topic-restricted configuration still let an
  agent read any checklist and every topic title of that chat.
- The topic refusal on `get` / `append` / `toggle` no longer names the topic the message is
  actually in (`message N is outside the allowed topics of chat C`): probing message ids one at
  a time no longer maps a chat's messages to topics.

### Changed
- `list-topics` on a per-topic allowlisted chat warns about allowed ids the chat does not have.
- 150 offline tests (six new: read and list gates, refusal wording).

The JSON contract is unchanged except for the wording of that one refusal.

## [1.1.0] - 2026-09-24

### Changed
- `SKILL.md` frontmatter follows the Hermes skill authoring standards (57-character `description`,
  `author`, `license`, `platforms`, `metadata.hermes.related_skills`).
- Documented the runtime fallback for large sourced lists (`TODO_ITEMS_TOO_MUCH`: rebuild as
  batches of at most 20 shorter items).
- `plan.json` accepts `media: rich_media`.
- Tests: `--dry-run` preserves the shared flags; the `rich_media` case. 144 offline tests.

## [1.0.1] - 2026-09-24

- Repository transfer to `bablobanov/hermes-telegram-checklist`; no code change.

## [1.0.0] - 2026-07-21

- Initial release: `plan`, `create` (direct and from `plan.json`), `get`, `append`, `toggle`,
  `list-topics`; chat / topic allowlist; offline plan validation; post-write verification.
