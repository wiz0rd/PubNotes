# Markdown Language Cheat Sheet

## Headers

```markdown
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

## Emphasis

```markdown
*italic* or _italic_
**bold** or __bold__
***bold italic*** or ___bold italic___
~~strikethrough~~
```

## Lists

### Unordered
```markdown
- Item 1
- Item 2
  - Subitem 2.1
  - Subitem 2.2
```

### Ordered
```markdown
1. First item
2. Second item
   1. Subitem 2.1
   2. Subitem 2.2
```

## Links

```markdown
[Link text](URL)
[Link with title](URL "Title")
[Reference-style link][reference]

[reference]: URL
```

## Images

```markdown
![Alt text](image-url)
![Alt text](image-url "Optional title")
```

## Code

Inline code: `code`

Code block:
```markdown
```language
code block
```
```

## Blockquotes

```markdown
> This is a blockquote
> > Nested blockquote
```

## Horizontal Rule

```markdown
---
***
___
```

## Tables

```markdown
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
| Cell 3   | Cell 4   |
```

## Task Lists

```markdown
- [x] Completed task
- [ ] Uncompleted task
```

## Footnotes

```markdown
Here's a sentence with a footnote. [^1]

[^1]: This is the footnote.
```

## Escaping Characters

Use a backslash to escape special characters:
```markdown
\* This is not italic \*
```

## HTML

Markdown supports inline HTML:
```markdown
<sup>Superscript</sup>
<sub>Subscript</sub>
```

## Definition Lists

```markdown
Term
: Definition
```

Remember, some Markdown features may depend on the specific Markdown processor or platform you're using.
