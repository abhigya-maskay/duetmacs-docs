---
tags: [navigation, guide, help]
aliases: [Help, Guide, How-to-Navigate]
---

# Navigation Guide

## Documentation Structure

This vault contains DuetMacs project documentation organized in numbered folders:

```
00-Start Here/             Entry point and project overview
01-Architecture/           System design and technical decisions
02-Features/               Feature inventory and specifications
03-Epics/                  Epic-level implementation planning
04-Components/             Component specifications and interfaces
05-UX Specifications/      User experience designs
06-Testing/                Test strategies and documentation
```

## Finding Information

### Quick Access Points

#### Essential Documents
- [[00-Start-Here/README|Main README]] - Central hub with all documentation links
- [[Project Overview]] - High-level DuetMacs description
- [[01-Architecture/System Overview|System Overview]] - Technical architecture
- [[02-Features/Feature Inventory|Feature Inventory]] - Complete feature list
- [[03-Epics/Epic Roadmap|Epic Roadmap]] - Development timeline

#### Information by Topic
- **Architecture Decisions**: `01-Architecture/ADRs/`
- **Feature Specifications**: `02-Features/`
- **Epic Details**: `03-Epics/V1-Foundation/`
- **Component Documentation**: `04-Components/`
- **UX Specifications**: `05-UX-Specifications/`
- **Test Documentation**: `06-Testing/`
- **Risk Analysis**: See risk-register.md in each story folder

### Search Strategies

#### Using Obsidian Search
- `Ctrl+Shift+F` - Full-text search across all documents
- Search operators:
  - `file:` - Search by filename
  - `tag:` - Search by tag
  - `line:` - Search within specific lines
  - `section:` - Search within sections

#### Using Tags
Common tags for finding related content:
- `#architecture` - Architecture and design documents
- `#component/*` - Component-specific documentation
- `#epic/*` - Epic-related content
- `#story/*` - Story documentation
- `#testing` - Test-related documents
- `#risk` - Risk assessments

#### Using Links
- WikiLinks connect related documents
- Type `[[` to see available documents
- Backlinks show which documents reference the current one

### Navigation Tools

#### Obsidian Features
- `Ctrl+O` - Quick switcher for file access
- `Ctrl+G` - Graph view to visualize connections
- `Alt+←/→` - Navigate back/forward through history
- `Ctrl+E` - Toggle between edit and preview modes

#### Document Metadata
All documents include frontmatter:
```yaml
---
tags: [category, subcategory]
aliases: [Alternative Names]
---
```

### Common Queries

| Looking for... | Find it in... |
|---------------|--------------|
| What DuetMacs does | [[Project Overview]] |
| Technical design | [[01-Architecture/System Overview\|System Overview]] |
| Available features | [[02-Features/Feature Inventory\|Feature Inventory]] |
| Current development | [[03-Epics/Epic Roadmap\|Epic Roadmap]] |
| Component details | [[04-Components/Component Map\|Component Map]] |
| Test coverage | [[06-Testing/Test Strategy\|Test Strategy]] |
| Risk assessments | See risk-register.md in each story folder |

### Document Organization

#### Hierarchical Structure
- Numbered folders provide logical grouping
- README files in each folder provide section overviews
- Sub-folders contain detailed specifications

#### Cross-References
- Documents link to related content
- Navigation footers show document hierarchy
- Backlinks reveal document relationships

---
*Navigation: [[00-Start-Here/README|Home]] > Navigation Guide*
