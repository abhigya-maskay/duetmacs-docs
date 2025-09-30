---
tags: [navigation, guide, help]
aliases: [Help, Guide, How-to-Navigate]
---

# Navigation Guide

## Vault Organization

This Obsidian vault is organized with numbered folders for logical progression:

### Folder Structure
```
00-Start Here/             Entry point and overview
01-Architecture/           System design and decisions
02-Features/               Capabilities and inventory
03-Epics/                  Implementation details
04-Components/             Reusable component specs
05-UX Specifications/      User experience designs
06-Testing/                Quality assurance
07-Risk-Register/          Risk tracking by story
```

## Navigation Methods

### 1. WikiLinks
Use the WikiLink syntax like [ [Document Name] ] to link between documents. Obsidian will auto-complete as you type.

### 2. Tags
Click tags to find related content:
- `#epic/v1-foundation` - Current development work
- `#component/cli` - CLI-related documentation
- `#architecture` - System design documents
- `#testing` - Test-related content

### 3. Graph View
- Press `Ctrl+G` to see the knowledge graph
- [[00-Start-Here/README|README]] is the central hub node
- Clusters show related documentation

### 4. Quick Switcher
- Press `Ctrl+O` to quickly open any document
- Start typing to filter by name

### 5. Search
- Press `Ctrl+Shift+F` for full-text search
- Use operators: `file:`, `tag:`, `line:`, `section:`

## Document Structure

### Frontmatter
Each document includes:
```yaml
---
tags: [category, topic, subtopic]
aliases: [Alternative, Names]
---
```

### Navigation Footer
Documents include navigation breadcrumbs:
```
*Navigation: [[00-Start-Here/README|Home]] > `Parent` > Current*
```

## Key Documents

### Starting Points
1. [[00-Start-Here/README|README]] - Main hub with all links
2. [[Project Overview]] - What is DuetMacs?

### By Task
- **Understanding architecture**: [[System Overview]]
- **Checking progress**: [[Epic Roadmap]]
- **Finding components**: [[Component Map]]

### By Role
- **Developer**: Start with [[Epic Roadmap]]
- **Designer**: See [[05-UX-Specifications]]
- **Tester**: Review [[Test Strategy]]

## Tips for Obsidian

### Useful Shortcuts
- `Ctrl+E` - Toggle edit/preview mode
- `Ctrl+P` - Command palette
- `Ctrl+G` - Graph view
- `Ctrl+O` - Quick switcher
- `Ctrl+Shift+F` - Search vault
- `Alt+←/→` - Navigate back/forward

### Plugins to Consider
1. **Dataview** - Query documents like a database
2. **Kanban** - Task board for Project Bootstrap
3. **Excalidraw** - Architecture diagrams
4. **Templater** - Consistent document creation
5. **Git** - Version control integration

### Views Setup
Recommended workspace:
- Left: File explorer
- Center: Main document
- Right split: Graph view or outline
- Bottom: Search results

## Finding Information

### By Topic
- **Architecture**: [[01-Architecture]]
- **Features**: [[02-Features]]  
- **Epics**: [[03-Epics]]
- **Components**: [[04-Components]]
- **Testing**: [[06-Testing]]

### By Question
- "What can DuetMacs do?" → [[Feature Inventory]]
- "What's the architecture?" → [[System Overview]]
- "What's being built now?" → [[Epic Roadmap]]

## Contributing

When adding documentation:
1. Place in the appropriate numbered folder
2. Add frontmatter with tags and aliases
3. Link from relevant parent documents
4. Update [[00-Start-Here/README|README]] if it's a major addition
5. Include navigation footer

---
*Navigation: [[00-Start-Here/README|Home]] > Navigation Guide*
