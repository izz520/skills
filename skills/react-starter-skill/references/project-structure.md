# Project Structure

## Root Directory

```
project-root/
├── .vscode/
│   └── settings.json
├── public/
├── src/
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig.json
├── tsconfig.node.json
├── lingui.config.ts
├── biome.json
├── vercel.json
└── components.json          # shadcn config
```

## src/ Directory

```
src/
├── main.tsx                 # Entry point
├── vite-env.d.ts
│
├── api/                     # API layer
│   ├── request.ts           # Axios instance & interceptors
│   └── user.ts              # User API example
│
├── components/
│   ├── Layout/
│   │   ├── index.tsx        # Main layout with Outlet
│   │   ├── Header.tsx
│   │   └── Footer.tsx
│   ├── ui/                  # shadcn components (auto-generated)
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   └── ...
│   ├── Provider/
│   │   ├── index.tsx        # Combined providers
│   │   ├── I18nProvider.tsx
│   │   ├── QueryProvider.tsx
│   │   └── ThemeProvider.tsx
│   └── [ComponentName]/     # Shared components (PascalCase)
│       └── index.tsx
│
├── pages/
│   ├── home/                # lowercase directory names
│   │   ├── index.tsx
│   │   ├── components/      # Page-specific components
│   │   └── hooks/           # Page-specific hooks
│   ├── about/
│   │   └── index.tsx
│   └── 404/
│       └── index.tsx
│
├── routes/                  # Route definitions
│   └── index.tsx
│
├── hooks/                   # Global hooks
│   └── usePrefersColor.ts   # System theme detection
│
├── stores/                  # Zustand stores
│   ├── user.ts
│   └── common.ts
│
├── locales/                 # Lingui translations
│   ├── en/
│   │   ├── messages.po
│   │   └── messages.ts      # Compiled (auto-generated)
│   ├── es/
│   │   ├── messages.po
│   │   └── messages.ts
│   ├── ja/
│   │   ├── messages.po
│   │   └── messages.ts
│   ├── ko/
│   │   ├── messages.po
│   │   └── messages.ts
│   ├── vi/
│   │   ├── messages.po
│   │   └── messages.ts
│   ├── tw/
│   │   ├── messages.po
│   │   └── messages.ts
│   ├── fr/
│   │   ├── messages.po
│   │   └── messages.ts
│   └── pt/
│       ├── messages.po
│       └── messages.ts
│
├── utils/                   # Utility functions (add as needed)
│
├── types/                   # TypeScript types
│   ├── response.ts
│   └── user.ts
│
├── constants/               # Constants
│   ├── languages.ts
│   └── config.ts
│
├── index.css               # Global Css
│
└── lib/                     # Third-party configs (add as needed)
```

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Pages | lowercase | `src/pages/home/` |
| Components | PascalCase | `src/components/UserCard/` |
| Hooks | camelCase with `use` prefix | `useMediaQuery.ts` |
| Stores | camelCase | `user.ts`, `common.ts` |
| Utils | camelCase | `format.ts` |
| Types | PascalCase for interfaces | `IResponse`, `IUser` |
