# Obsidian Markdown Reference

A complete reference of Obsidian-specific markdown syntax, formatting, plugins, and features.

---

## 1. Headings

```markdown
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
```

---

## 2. Basic Formatting

```markdown
**bold**  
*italic*  
~~strikethrough~~  
`inline code`

```python
# code block
print("Hello, Obsidian!")
```

---

## 3. Internal and External Links

```markdown
[[Note Name]]                        # Internal link
[[Note Name|Custom Label]]           # Link with alias
[[Note Name#Section Title]]          # Link to header
[[Note Name^block-id]]               # Link to block

[Google](https://www.google.com)     # External link
```

---

## 4. Embeds (Transclusions)

```markdown
![[Note Name]]                       # Embed entire note
![[Note Name#Section]]              # Embed specific section
![[Note Name^block-id]]             # Embed block
![[image.png]]                       # Embed image
```

---

## 5. Block References

Add a block ID:

```markdown
This is a paragraph. ^my-block
```

Reference it:

```markdown
[[Note Name^my-block]]
![[Note Name^my-block]]
```

---

## 6. Tags

```markdown
#python
#ml/supervised
#project/ai
```

Tags support hierarchy and are searchable.

---

## 7. YAML Frontmatter

```yaml
---
title: Tuple Notes
tags: [python, stage1]
book: Data Science from Scratch
created: 2025-07-17
---
```

Used for metadata and plugins like Dataview or Templater.

---

## 8. Callouts (Admonitions)

```markdown
> [!note] Note Title
> This is a standard note.

> [!tip] Tip Title
> Useful advice here.

> [!warning] Warning Title
> Be cautious.

> [!info] Info Title
> General information.

> [!question] Question Title
> Pose a reflective or quiz-like question.
```

---

## 9. Task Lists

```markdown
- [ ] Unchecked task
- [x] Completed task
```

Tasks can be searched, filtered, or queried.

---

## 10. Footnotes

```markdown
This sentence has a footnote.[^1]

[^1]: This is the footnote content.
```

---

## 11. Daily Notes

- Enable via Settings → Core Plugins → Daily Notes
- Files named as `YYYY-MM-DD.md`
- Useful with the Calendar plugin

---

## 12. Search Syntax

Examples for use in the Obsidian search bar:

```markdown
tag:#python                        # Find notes tagged #python
path:"01-python-foundations"       # Notes in folder
file:lists                         # File name includes "lists"
section:Functions                  # Section name
line:tuple                         # Match line content
-task:""                           # All unchecked tasks
```

---

## 13. Dataview Plugin

### Table Example

```markdown
table file.name, tags
from "01-python-foundations"
where contains(tags, "#python")
```

### List Example

```markdown
list from "notes"
where contains(file.tags, "#review")
```

### Task Query Example

```markdown
task from "notes"
where !completed
```

---

## 14. Templater Plugin

```markdown
<%*
const today = new Date().toISOString().slice(0, 10);
tR += `# Notes for ${today}`;
%>
```

- Use `<% %>` or `<%* %>` for scripts
- Allows reusable templates and automation

---

## 15. Canvas (Visual Note Maps)

- Create `.canvas` files
- Drag notes as nodes
- Link with arrows
- Good for brainstorming or mind maps

---

## 16. Tables

```markdown
| Feature   | Description        |
|-----------|--------------------|
| List      | Ordered/Unordered  |
| Tuple     | Immutable Sequence |
```

Advanced Tables plugin helps edit tables.

---

## 17. Graph View

- Shows all notes as a connected graph
- Hover to see linked notes
- Filter by tag, path, or link type

---

## 18. Backlinks Pane

- Shows which other notes link to the current one
- Access via right sidebar

---

## 19. File Embeds and Linking

```markdown
[[01-python-foundations/notes]]
![[01-python-foundations/notes]]
```

Use relative paths to link/organize notes from folders.

---

## 20. Vault Structuring Tips

- Organize by stages or topics (e.g., `01-python-foundations`)
- Put one `notes.md` per stage if preferred
- Keep `README.md`, `weekly-learning-plan.md`, and `obsidian-markdown-reference.md` in root
- Use consistent note titles and tags

---

## 21. Useful Core Plugins

- Daily Notes
- Templates
- Backlinks
- Tag Pane
- Page Preview

Enable them from Settings → Core Plugins

---

## 22. Recommended Community Plugins

| Plugin          | Purpose                          |
|-----------------|----------------------------------|
| Templater       | Dynamic templates and scripts    |
| Calendar        | Visual daily note access         |
| Obsidian Git    | Sync vault with GitHub           |
| Dataview        | Query notes like a database      |
| Advanced Tables | Table editing and formatting     |
| QuickAdd        | Custom capture templates         |

Install from Settings → Community Plugins → Browse

---

## 23. Keyboard Shortcuts (Mac)

| Action                  | Shortcut          |
|-------------------------|-------------------|
| Toggle preview/edit     | Cmd + E           |
| Bold                    | Cmd + B           |
| Italic                  | Cmd + I           |
| Open quick switcher     | Cmd + O           |
| Search vault            | Cmd + Shift + F   |
| Backlinks pane          | Cmd + Shift + B   |

Customize shortcuts under Settings → Hotkeys.

---

## 24. Tips for AI Learning Notes

- Use one `notes.md` per stage
- Add headings per topic (e.g., `## Lists`, `## Tuples`)
- Use callouts for personal takeaways or pitfalls
- Include code blocks and outputs
- Use tags like `#concept`, `#syntax`, `#error`, `#tip`

---

End of file.
