# study-on-canvas

[한국어](README.md) · [English](README.en.md)

- A Claude Code plugin that helps you learn by turning your conversation with an LLM into a visual HTML file as you go.

## Usage

### 1. Ask about the concept you want to learn, together with the plugin call.

- Calling the plugin directly with a slash command

```plain_text
/study-on-canvas:study-on-canvas I want to understand what Attention means in the paper "Attention Is All You Need", and how that idea led to today's generative AI.
```

- Calling it in natural language — mentioning "canvas", "캔버스", or "study note" triggers the plugin without a slash command.

```plain_text
Let's talk on a canvas. I want to understand what Attention means in the paper "Attention Is All You Need".
```

```plain_text
Turn on the canvas and explain transformers to me
```

You can also just say "let's talk on a canvas" with no topic. It will ask once what you want to study.

It does not trigger on plain questions that never mention the canvas, such as "explain transformers to me".




A Claude Code plugin that keeps a live HTML study note of your conversation while you learn new concepts with an LLM.

Every time one concept is explained, one to three follow-up concepts are stacked in the sidebar. Even if you dive into one and drift off course, the sibling concepts you have not covered stay on screen — so "what else was I going to look at?" never happens.

## What it does

- Start a session with `/study-on-canvas <topic>` and a single dark-themed HTML note is created and opened in your browser.
- Every concept explained adds one tab and one explanation panel to the note.
- At the same time, one to three follow-up concepts are stacked as sidebar cards. Never four or more — so you are not overwhelmed at once.
- Cards for concepts already covered are not deleted; they only turn yellow. They keep their place and become a map.
- Press the map button in the header and the parent–child relationships between cards are drawn as a tree (the concept map).
- Once an explanation ends, it stops there. It does not ask "what should we look at next?". You decide what to cover by looking at the sidebar.

The note is a single HTML file with no dependencies, so you can open, move, and share it as is. At mobile widths the sidebar turns into a bottom sheet.

## Installing the plugin

In Claude Code:

```
/plugin marketplace add winter-blanket/study-on-canvas
/plugin install study-on-canvas@study-on-canvas
```

## Usage

```
/study-on-canvas:study-on-canvas transformers
```

It asks once where to save. The default is `learning-notes/YYYY-MM-DD-<topic>.html` under the folder where the session started. After that, just talk.

```
> explain transformers        → tab created + 3 cards (Self-Attention / Positional Encoding / Multi-Head Attention)
> explain self-attention      → tab added, the first card turns yellow, 2 new cards
```

When the note is updated, just refresh it in the browser.

If you create the note inside a git repository it may get swept into a commit. If you would rather it did not, put `learning-notes/` in `.gitignore`.

## Language

The note UI defaults to Korean and also supports English. It is decided automatically from the language of the sentence that invoked the skill — ask in English and you get an English note. It does not change mid-session.

## Requirements

- Claude Code. It uses skill frontmatter such as `user-invocable` and the `AskUserQuestion` tool, so it does not work on other harnesses.
- It tries to open the browser automatically on macOS, Linux, and Windows (Git Bash/MSYS). If that fails it just tells you the file path, which does not affect how the skill works.

## Structure

```
.claude-plugin/
  plugin.json          plugin manifest
  marketplace.json     single-plugin marketplace definition
skills/study-on-canvas/
  SKILL.md             the skill itself — execution flow and HTML manipulation spec
  template/
    study-note.html    note template (single file, no external dependencies)
```

The template only holds placeholders such as `{{TITLE}}` and `{{DATE}}` plus markers like `<!-- TABS_INSERT -->`. The skill never rewrites the whole file; it only inserts between those markers.

## Customizing

- Colors: just edit the `:root` CSS variables in `study-note.html`. The theme is fixed to dark.
- UI wording: SKILL.md has a per-language dictionary in its "노트 언어" (note language) table. To add a language, add one more column to that table.
- Number of follow-up concepts: core principle 1 in SKILL.md.

## License

MIT
