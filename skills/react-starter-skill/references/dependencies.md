# Dependencies

## Core Dependencies

```bash
pnpm add react@latest react-dom@latest react-router@latest
```

## State Management

```bash
pnpm add @tanstack/react-query@latest zustand@latest react-tracked@latest
```

## Styling

```bash
pnpm add tailwindcss@latest @tailwindcss/vite@latest
```

## i18n (Lingui)

```bash
pnpm add @lingui/core@latest @lingui/react@latest
pnpm add -D @lingui/cli@latest @lingui/vite-plugin@latest @lingui/babel-plugin-lingui-macro@latest
```

## HTTP Client

```bash
pnpm add axios@latest
```

## Icons

```bash
pnpm add lucide-react@latest
```

## shadcn/ui

```bash
pnpm dlx shadcn@latest init
```

Then add components as needed:
```bash
pnpm dlx shadcn@latest add button card dialog
```

## Dev Dependencies

```bash
pnpm add -D typescript@latest @types/react@latest @types/react-dom@latest
pnpm add -D vite@latest @vitejs/plugin-react@latest
pnpm add -D @biomejs/biome@latest
```

## Full Install Command

```bash
# Core
pnpm add react@latest react-dom@latest react-router@latest

# State & Data
pnpm add @tanstack/react-query@latest zustand@latest react-tracked@latest axios@latest

# Styling & Icons
pnpm add tailwindcss@latest @tailwindcss/vite@latest lucide-react@latest

# i18n
pnpm add @lingui/core@latest @lingui/react@latest
pnpm add -D @lingui/cli@latest @lingui/vite-plugin@latest @lingui/babel-plugin-lingui-macro@latest

# Dev
pnpm add -D typescript@latest @types/react@latest @types/react-dom@latest vite@latest @vitejs/plugin-react@latest
pnpm add -D @biomejs/biome@latest

# shadcn
pnpm dlx shadcn@latest init
```
