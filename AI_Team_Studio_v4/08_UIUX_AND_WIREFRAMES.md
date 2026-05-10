# 18. UI/UX Governance & Wireframe Schema

> **Implementation Status (2026-03-27):** The wireframe system is fully implemented. The UX agent generates structured JSON wireframe artifacts using the `create_document` tool. 22 wireframe files were found in the sample project workspace. The platform UI itself is built with Tailwind CSS 4, dark theme with amber accent, Radix UI primitives, framer-motion transitions, and a drag-and-drop kanban board. All 12 platform pages are wired to real data.

## 18.1 Philosophy

Wireframes are **structured data**, not images. They are machine-readable JSON definitions that describe page layouts, components, fields, and states. The UX agent generates these as project artifacts stored in PostgreSQL before any UI coding begins.

## 18.2 Wireframe JSON Schema

```json
{
  "wireframe_id": "WF-001",
  "page_name": "User Dashboard",
  "route": "/dashboard",
  "layout_type": "sidebar-main",
  "owner": "UI/UX",
  "status": "approved",
  "sections": [
    {
      "section_id": "SEC-001",
      "name": "Sidebar Navigation",
      "position": "left",
      "components": [
        {
          "component_id": "COMP-001",
          "type": "nav-menu",
          "items": ["Dashboard", "Projects", "Settings", "Help"]
        }
      ]
    },
    {
      "section_id": "SEC-002",
      "name": "Main Content",
      "position": "center",
      "components": [
        {
          "component_id": "COMP-002",
          "type": "data-table",
          "columns": ["Name", "Status", "Last Updated", "Actions"],
          "actions": ["View", "Edit", "Delete"],
          "states": {
            "loading": { "display": "skeleton-loader" },
            "empty": { "display": "empty-state", "message": "No projects yet" },
            "error": { "display": "error-banner", "message": "Failed to load projects" },
            "populated": { "display": "data-rows" }
          }
        }
      ]
    }
  ],
  "fields": [
    {
      "field_id": "FLD-001",
      "label": "Search Projects",
      "type": "text-input",
      "placeholder": "Search by name...",
      "validation": { "required": false, "max_length": 100 }
    }
  ],
  "page_states": {
    "loading": "Show skeleton layout",
    "error": "Show error banner with retry button",
    "empty": "Show onboarding prompt",
    "authenticated": "Show full dashboard",
    "unauthenticated": "Redirect to login"
  }
}
```

## 18.3 Wireframe Governance Rules

- Wireframes **must be approved** before any UI coding begins — ✅ enforced via document approval state
- Every page must define: loading, error, and empty states — ✅ UX agent system prompt requires these
- Component inventory is controlled — no unauthorized component types
- UI/UX agent is the sole owner of wireframe files — ✅ authority guard enforces write permission
- Changes to approved wireframes require a formal Decision — ✅ create_decision tool required

## 18.4 Platform UI Implementation

The AI Team Studio platform UI itself is built to a production standard:

| Feature | Technology | Status |
|---------|-----------|--------|
| Framework | Next.js 16 (App Router, React 19, Turbopack) | ✅ |
| Styling | Tailwind CSS 4, dark theme, amber accent color | ✅ |
| Components | Radix UI primitives | ✅ |
| Animations | framer-motion | ✅ |
| State | Zustand stores | ✅ |
| Drag-and-drop kanban | @dnd-kit/core + @dnd-kit/sortable | ✅ |
| Command palette | cmdk | ✅ |
| Charts | Recharts | ✅ |
| 4-step project creation wizard | Idea → Audience → Priorities → Review | ✅ |
| Onboarding wizard | Welcome → Preferences → Done | ✅ |
| Platform pages | 12 pages, all wired to real PostgreSQL | ✅ |
| Marketing pages | Landing page, auth pages | ✅ |
| Decisions UI | "Your AI Team Recommends" banner + prominent approve CTA | ✅ |
| Board view | Drag-and-drop kanban + milestones view | ✅ |
| Agents page | Human-readable role descriptions + "What I Do" section | ✅ |
