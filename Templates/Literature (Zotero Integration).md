---
citekey: {{citekey}}
title: "{{title}}"
year: {{date | format("YYYY")}}
tags: [literature]
---
# {{title}}
{{authors}} · [Zotero]({{desktopURI}})

## Key facts (with page)

## Relevance to thesis

## Annotations
{% for annotation in annotations %}
- {{annotation.annotatedText}} (p. {{annotation.page}}){% if annotation.comment %} — {{annotation.comment}}{% endif %}
{% endfor %}
