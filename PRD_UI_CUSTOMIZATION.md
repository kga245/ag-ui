# Product Requirements Document: AG-UI Portal UI Customization

## Executive Summary

This PRD outlines the requirements for customizing the AG-UI Dojo application to serve as a branded agent portal for enterprise use. The current architecture provides a solid foundation with modular components, CSS variable-based theming, and Tailwind CSS styling.

---

## 1. Current Architecture Analysis

### 1.1 Technology Stack
- **Framework**: Next.js 15 (App Router)
- **Styling**: Tailwind CSS v4 with CSS variables
- **UI Components**: Shadcn/ui (Radix-based primitives)
- **Chat UI**: CopilotKit React UI
- **Theming**: next-themes (dark/light mode)
- **Icons**: Phosphor Icons, Lucide React
- **Fonts**: Plus Jakarta Sans (primary), Spline Sans Mono (code)

### 1.2 Component Inventory

#### Layout Components
| Component | File | Purpose | Customization Priority |
|-----------|------|---------|------------------------|
| MainLayout | `src/components/layout/main-layout.tsx` | Root layout with sidebar & content | HIGH |
| ViewerLayout | `src/components/layout/viewer-layout.tsx` | Content area wrapper | MEDIUM |
| Sidebar | `src/components/sidebar/sidebar.tsx` | Navigation, integration picker | HIGH |

#### Navigation & Selection
| Component | File | Purpose | Customization Priority |
|-----------|------|---------|------------------------|
| DemoList | `src/components/demo-list/demo-list.tsx` | Feature/agent list | HIGH |
| DropdownMenu | `src/components/ui/dropdown-menu.tsx` | Integration selector | MEDIUM |
| Tabs | `src/components/ui/tabs.tsx` | View switcher (Preview/Code/Docs) | MEDIUM |

#### Content Display
| Component | File | Purpose | Customization Priority |
|-----------|------|---------|------------------------|
| CodeViewer | `src/components/code-viewer/code-viewer.tsx` | Source code display | LOW |
| CodeEditor | `src/components/code-viewer/code-editor.tsx` | Monaco editor wrapper | LOW |
| FileTree | `src/components/file-tree/file-tree.tsx` | File navigation | LOW |
| Readme | `src/components/readme/readme.tsx` | Documentation viewer | LOW |
| MarkdownComponents | `src/components/ui/markdown-components.tsx` | MD rendering | MEDIUM |

#### Chat & Agent Interaction (CopilotKit)
| Component | Source | Purpose | Customization Priority |
|-----------|--------|---------|------------------------|
| CopilotChat | `@copilotkit/react-ui` | Main chat interface | CRITICAL |
| CopilotSidebar | `@copilotkit/react-ui` | Alternative chat layout | CRITICAL |
| Tool Renderers | Custom per feature | Tool result display | HIGH |

#### Atomic UI Elements
| Component | File | Purpose | Customization Priority |
|-----------|------|---------|------------------------|
| Button | `src/components/ui/button.tsx` | Action buttons | HIGH |
| Badge | `src/components/ui/badge.tsx` | Labels/tags | MEDIUM |
| ThemeToggle | `src/components/ui/theme-toggle.tsx` | Dark/light switch | MEDIUM |
| Carousel | `src/components/ui/carousel.tsx` | Content carousel | LOW |

---

## 2. Branding & Theming Customization

### 2.1 CSS Variables (Primary Customization Layer)

**Location**: `apps/dojo/src/app/globals.css`

#### Core Brand Colors
```css
:root {
  /* Primary brand colors - CUSTOMIZE THESE */
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);

  /* Accent/Secondary colors */
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);

  /* Background colors */
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
}
```

#### Custom Palette Colors
```css
@theme {
  /* Mint theme colors - can be replaced with brand colors */
  --color-palette-mint-400: #85e0ce;
  --color-palette-mint-800: #1b936f;

  /* Lilac accent colors */
  --color-palette-lilac-400: #bec2ff;

  /* Surface colors - for card/panel backgrounds */
  --color-palette-surface-container: #ffffff;
  --color-palette-surface-main: #dedee9;
}
```

### 2.2 Typography

**Font Configuration**: `apps/dojo/tailwind.config.ts`

```typescript
fontFamily: {
  sans: ['Plus Jakarta Sans', ...],  // Replace with brand font
  mono: ['Spline Sans Mono', ...],   // Replace with brand mono font
}
```

**Typography Styles**: `apps/dojo/src/styles/typography.css`

### 2.3 Logo & Branding

**Header Title**: `apps/dojo/src/components/sidebar/sidebar.tsx:103-107`
```tsx
<h1 className="text-lg font-light">
  {getTitleForCurrentDomain() || "AG-UI Interactive Dojo"}
</h1>
```

**Domain Configuration**: `apps/dojo/src/utils/domain-config.ts`
- Supports custom titles per domain
- Configure via `NEXT_PUBLIC_CUSTOM_DOMAIN_TITLE` env var

---

## 3. Customization Requirements

### 3.1 P0 - Critical (Brand Identity)

| Requirement | File(s) to Modify | Description |
|-------------|-------------------|-------------|
| **Company Logo** | `sidebar.tsx`, `public/` | Add logo image/SVG to header |
| **Brand Colors** | `globals.css` | Replace mint/lilac with brand colors |
| **Application Title** | `sidebar.tsx`, `env.ts` | Custom portal name |
| **Primary Font** | `tailwind.config.ts`, `layout.tsx` | Brand typography |
| **Chat UI Styling** | Feature `style.css` files | CopilotKit component theming |

### 3.2 P1 - High Priority (User Experience)

| Requirement | File(s) to Modify | Description |
|-------------|-------------------|-------------|
| **Integration Naming** | `menu.ts` | Rename integrations to match internal terms |
| **Feature Labels** | `config.ts` | Custom feature names/descriptions |
| **Navigation Structure** | `sidebar.tsx` | Reorganize or simplify navigation |
| **Mobile Layout** | `main-layout.tsx` | Mobile-specific branding |
| **Empty States** | Various pages | Custom messaging for no-content states |

### 3.3 P2 - Medium Priority (Polish)

| Requirement | File(s) to Modify | Description |
|-------------|-------------------|-------------|
| **Loading States** | Various components | Branded spinners/skeletons |
| **Error Messages** | API routes, components | Consistent error styling |
| **Icons** | Import statements | Custom icon set |
| **Shadows/Elevation** | `globals.css` | Brand-specific depth |
| **Animations** | `tailwind.config.ts` | Custom transitions |

### 3.4 P3 - Low Priority (Nice-to-have)

| Requirement | File(s) to Modify | Description |
|-------------|-------------------|-------------|
| **Code Syntax Theme** | Monaco config | Match brand colors in code viewer |
| **Markdown Styling** | `markdown-components.tsx` | Documentation theming |
| **Carousel Indicators** | `carousel.tsx` | Brand-styled navigation dots |

---

## 4. CopilotKit Chat UI Customization

### 4.1 Styling Override Points

**Global CopilotKit Styles**: Each feature imports `@copilotkit/react-ui/styles.css`

**Per-Feature Custom CSS**: `src/app/[integrationId]/feature/*/style.css`

Example customization targets:
```css
/* Chat container */
.copilotKitChat { }

/* Message bubbles */
.copilotKitMessage { }

/* User messages */
.copilotKitUserMessage { }

/* Assistant messages */
.copilotKitAssistantMessage { }

/* Input area */
.copilotKitInput { }

/* Send button */
.copilotKitSendButton { }
```

### 4.2 Component Props Customization

**CopilotChat Component**: Supports props for:
- `instructions` - System prompt
- `labels` - UI text strings
- `icons` - Custom icons
- `className` - Container styling

---

## 5. Implementation Roadmap

### Phase 1: Foundation (Week 1-2)
- [ ] Create brand color palette in CSS variables
- [ ] Replace fonts and configure typography
- [ ] Add company logo to sidebar
- [ ] Update application title
- [ ] Configure environment variables for domain

### Phase 2: Core UI (Week 3-4)
- [ ] Restyle sidebar and navigation
- [ ] Customize integration dropdown
- [ ] Update feature cards/list styling
- [ ] Apply brand colors to buttons and actions
- [ ] Style tab components

### Phase 3: Chat Experience (Week 5-6)
- [ ] Create custom CopilotKit theme CSS
- [ ] Style message bubbles with brand colors
- [ ] Customize input area and send button
- [ ] Add company avatar/icon for assistant
- [ ] Style tool result renderers

### Phase 4: Polish & QA (Week 7-8)
- [ ] Responsive design verification
- [ ] Dark mode consistency check
- [ ] Accessibility audit (WCAG 2.1)
- [ ] Performance optimization
- [ ] Cross-browser testing

---

## 6. Files Requiring Modification

### High Impact Files
1. `apps/dojo/src/app/globals.css` - Color palette, fonts, spacing
2. `apps/dojo/tailwind.config.ts` - Tailwind theme extensions
3. `apps/dojo/src/components/sidebar/sidebar.tsx` - Logo, title, navigation
4. `apps/dojo/src/menu.ts` - Integration names and features
5. `apps/dojo/src/config.ts` - Feature configurations
6. `apps/dojo/src/app/layout.tsx` - Root layout and fonts
7. All `style.css` files in feature directories - CopilotKit overrides

### Configuration Files
- `apps/dojo/next.config.ts` - Build configuration
- `apps/dojo/src/env.ts` - Environment variables
- `apps/dojo/public/` - Static assets (logo, favicon)

---

## 7. Testing Requirements

### Visual Regression Testing
- Screenshot comparison for all major views
- Mobile responsive breakpoints
- Dark/light mode variants

### Accessibility Testing
- Color contrast ratios (WCAG AA minimum)
- Keyboard navigation
- Screen reader compatibility
- Focus indicators

### Cross-Browser Testing
- Chrome, Firefox, Safari, Edge
- Mobile browsers (iOS Safari, Chrome Mobile)

---

## 8. Recommendations

### Immediate Actions
1. **Create a brand theme file** - Centralize all brand colors/fonts
2. **Fork CopilotKit styles** - Don't override, replace entirely
3. **Remove unused features** - Simplify to only needed integrations
4. **Custom loading component** - Branded skeleton screens

### Future Enhancements
1. **White-label configuration** - JSON-based theme configuration
2. **Multi-tenant support** - Different brands per organization
3. **Custom chat avatars** - User and assistant personas
4. **Analytics dashboard** - Usage metrics for agents

---

## 9. Success Metrics

- Brand recognition score (user survey)
- Time to complete customization vs estimate
- Accessibility compliance percentage
- Performance metrics (Lighthouse scores maintained)
- Developer satisfaction with customization process

---

## Appendix A: Complete Component List

```
apps/dojo/src/components/
├── code-viewer/
│   ├── code-editor.tsx
│   └── code-viewer.tsx
├── demo-list/
│   └── demo-list.tsx
├── file-tree/
│   ├── file-tree-nav.tsx
│   └── file-tree.tsx
├── layout/
│   ├── main-layout.tsx
│   └── viewer-layout.tsx
├── readme/
│   └── readme.tsx
├── sidebar/
│   └── sidebar.tsx
├── ui/
│   ├── badge.tsx
│   ├── button.tsx
│   ├── carousel.tsx
│   ├── dropdown-menu.tsx
│   ├── markdown-components.tsx
│   ├── mdx-components.tsx
│   ├── tabs.tsx
│   └── theme-toggle.tsx
├── theme-provider.tsx
└── theme-wrapper.tsx
```

## Appendix B: Key File Paths

| Purpose | Path |
|---------|------|
| Global Styles | `apps/dojo/src/app/globals.css` |
| Tailwind Config | `apps/dojo/tailwind.config.ts` |
| Typography | `apps/dojo/src/styles/typography.css` |
| Integration Config | `apps/dojo/src/menu.ts` |
| Feature Config | `apps/dojo/src/config.ts` |
| Environment Variables | `apps/dojo/src/env.ts` |
| Root Layout | `apps/dojo/src/app/layout.tsx` |
| Chat Feature Styles | `apps/dojo/src/app/[integrationId]/feature/*/style.css` |
