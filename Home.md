---
tags: [home]
---
# Thesis home

**Open:** [Claude Project](https://claude.ai/code/projects) · [Thesis repo in VS Code](vscode://file/Users/martyn/msc-thesis) · [Zotero](zotero://select/library) · [Thesis plan & open items](https://github.com/EfekTyndall/msc-thesis/blob/main/docs/thesis-plan.md)

## Recent days
Today's note and the "Next step" from the last session.

```base
filters:
  and:
    - file.inFolder("Daily")
views:
  - type: table
    name: Recent days
    order:
      - file.name
    sort:
      - property: file.name
        direction: DESC
    limit: 5
```

## New from Claude
Machine side (`Agent/`): read, then copy what you keep into your own notes.

```base
filters:
  and:
    - file.inFolder("Agent")
    - file.name != "README"
views:
  - type: table
    name: New from Claude
    order:
      - file.name
      - file.folder
      - file.mtime
    sort:
      - property: file.mtime
        direction: DESC
    limit: 10
```

## Chapters

```base
filters:
  and:
    - file.inFolder("Chapters")
views:
  - type: table
    name: Chapters
    order:
      - file.name
      - status
    sort:
      - property: file.name
        direction: ASC
```

## Decisions

```base
filters:
  and:
    - file.inFolder("Decisions")
views:
  - type: table
    name: Decisions
    order:
      - file.name
      - status
      - file.mtime
    sort:
      - property: file.name
        direction: ASC
```

## Recent literature

```base
filters:
  and:
    - file.inFolder("Literature")
views:
  - type: table
    name: Literature
    order:
      - file.name
      - title
      - year
    sort:
      - property: file.ctime
        direction: DESC
    limit: 10
```
