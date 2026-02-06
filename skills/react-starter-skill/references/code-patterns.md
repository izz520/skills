# Code Patterns

## Entry Point (main.tsx)

```typescript
import { createRoot } from "react-dom/client";
import "./index.css";
import Providers from "./components/Provider";
import { routes } from "@/routes";
import { createBrowserRouter, RouterProvider } from "react-router";

const router = createBrowserRouter(routes);

createRoot(document.getElementById("root")!).render(
  <Providers>
    <RouterProvider router={router} />
  </Providers>
);
```


## Axios Request (api/request.ts)

```typescript
import axios, {
  type AxiosInstance,
  type AxiosPromise,
  type AxiosResponse,
  type InternalAxiosRequestConfig
} from "axios";
import { SUCCESS_CODE, UNAUTHORIZED_CODE } from "@/constants/config";
import { setLoginOut } from "@/stores/user";
import type { IResponse } from "@/types/response";

interface IRequestConfig<D = any> extends InternalAxiosRequestConfig<D> {
  isAuth?: boolean;
}

interface IHttp extends AxiosInstance {
  (config: IRequestConfig): AxiosPromise;
  get<T = any, R = IResponse<T>, D = any>(
    url: string,
    config?: IRequestConfig<D>
  ): Promise<R>;
  post<T = any, R = IResponse<T>, D = any>(
    url: string,
    data?: D,
    config?: IRequestConfig<D>
  ): Promise<R>;
}

const baseUrl = import.meta.env.VITE_API_HOST;

const request: IHttp = axios.create({
  baseURL: baseUrl,
  timeout: 35000
});

request.interceptors.request.use(
  (config: IRequestConfig) => {
    const userStore = JSON.parse(localStorage.getItem("user-store") || "{}");
    config.headers["Authorization"] = userStore?.state?.token || "";
    config.headers["Accept-Language"] = localStorage.getItem("lang") || "en";
    return config;
  },
  (err) => {
    console.error("http request error: ", err);
    return Promise.reject(err);
  }
);

request.interceptors.response.use(
  (res: AxiosResponse) => {
    if (res.data.code === SUCCESS_CODE) {
      return res.data;
    }
    if (res.data.code === UNAUTHORIZED_CODE) {
      setLoginOut();
    }
    return Promise.reject(res.data);
  },
  (error) => Promise.reject(error)
);

export default request;
```

## Response Type (types/response.ts)

```typescript
export interface IResponse<T = any> {
  code: number;
  message: string;
  data: T;
}
```

## Constants (constants/config.ts)

```typescript
export const SUCCESS_CODE = 0;
export const UNAUTHORIZED_CODE = 401;
```

## Constants (constants/languages.ts)

```typescript
export const baseLanguagesInfo = [
  {
    label: "English",
    value: "en"
  },
  {
    label: "Español",
    value: "es"
  },
  {
    label: "日本語",
    value: "ja"
  },
  {
    label: "한국어",
    value: "ko"
  },
  {
    label: "Tiếng Việt",
    value: "vi"
  },
  {
    label: "繁体中文",
    value: "tw"
  },
  {
    label: "Français",
    value: "fr"
  },
  {
    label: "Português",
    value: "pt"
  }
];

export const supportedLanguages = baseLanguagesInfo.map(
  (language) => language.value
);
```

## Zustand Store (stores/user.ts)

```typescript
import { createTrackedSelector } from "react-tracked";
import { create } from "zustand";
import { createJSONStorage, persist } from "zustand/middleware";

interface IUserStore {
  token: string;
  userInfo: { id: string; name: string } | null;
  setToken: (token: string) => void;
  setUserInfo: (info: { id: string; name: string } | null) => void;
  logout: () => void;
}

const store = create(
  persist<IUserStore>(
    (set) => ({
      token: "",
      userInfo: null,
      setToken: (token) => set({ token }),
      setUserInfo: (userInfo) => set({ userInfo }),
      logout: () => set({ token: "", userInfo: null })
    }),
    {
      name: "user-store",
      storage: createJSONStorage(() => localStorage)
    }
  )
);

export const useUserStore = createTrackedSelector(store);

// For use outside React components
export const setLoginOut = () => store.getState().logout();
```

## Zustand Store (stores/common.ts)

```typescript
import { createTrackedSelector } from "react-tracked";
import { create } from "zustand";
import { createJSONStorage, persist } from "zustand/middleware";

interface ICommonStore {
  prefersColorScheme: "light" | "dark";
  themeMode: "system" | "light" | "dark";
  setThemeMode: (themeMode: "system" | "light" | "dark") => void;
  setPrefersColorScheme: (scheme: "light" | "dark") => void;
}

const store = create(
  persist<ICommonStore>(
    (set) => ({
      prefersColorScheme: window.matchMedia("(prefers-color-scheme: dark)")
        .matches
        ? "dark"
        : "light",
      themeMode: "system",
      setThemeMode: (themeMode) => set({ themeMode }),
      setPrefersColorScheme: (prefersColorScheme) => set({ prefersColorScheme })
    }),
    {
      name: "common-store",
      storage: createJSONStorage(() => localStorage)
    }
  )
);

export const useCommonStore = createTrackedSelector(store);
```

## Global Hook (hooks/usePrefersColor.ts)

```typescript
import { useEffect } from "react";
import { useCommonStore } from "@/stores/common";

const usePrefersColor = () => {
  const { setPrefersColorScheme } = useCommonStore();

  useEffect(() => {
    const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");

    // Set initial value
    setPrefersColorScheme(mediaQuery.matches ? "dark" : "light");

    const handleThemeChange = (e: MediaQueryListEvent) => {
      setPrefersColorScheme(e.matches ? "dark" : "light");
    };

    mediaQuery.addEventListener("change", handleThemeChange);

    return () => {
      mediaQuery.removeEventListener("change", handleThemeChange);
    };
  }, [setPrefersColorScheme]);
};

export default usePrefersColor;
```

## I18nProvider (components/Provider/I18nProvider.tsx)

```typescript
import { i18n } from "@lingui/core";
import { I18nProvider as LinguiProvider } from "@lingui/react";
import { messages as enMessages } from "@/locales/en/messages";
import { messages as esMessages } from "@/locales/es/messages";
import { messages as frMessages } from "@/locales/fr/messages";
import { messages as jaMessages } from "@/locales/ja/messages";
import { messages as koMessages } from "@/locales/ko/messages";
import { messages as ptMessages } from "@/locales/pt/messages";
import { messages as twMessages } from "@/locales/tw/messages";
import { messages as viMessages } from "@/locales/vi/messages";

i18n.load({
  en: enMessages,
  tw: twMessages,
  fr: frMessages,
  pt: ptMessages,
  es: esMessages,
  ja: jaMessages,
  ko: koMessages,
  vi: viMessages
});

const lang = localStorage.getItem("lang");

const I18nProvider = ({ children }: { children: React.ReactNode }) => {
  i18n.activate(lang || "en");
  return <LinguiProvider i18n={i18n}>{children}</LinguiProvider>;
};

export default I18nProvider;
```

## QueryProvider (components/Provider/QueryProvider.tsx)

```typescript
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
      gcTime: 1000 * 60 * 5,
      retry: 1
    }
  }
});

const QueryProvider = ({ children }: { children: React.ReactNode }) => (
  <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
);

export default QueryProvider;
```

## Combined Providers (components/Provider/index.tsx)

```typescript
import I18nProvider from "./I18nProvider";
import QueryProvider from "./QueryProvider";
import ThemeProvider from "./ThemeProvider";

const Providers = ({ children }: { children: React.ReactNode }) => (
  <I18nProvider>
    <QueryProvider>
      <ThemeProvider>{children}</ThemeProvider>
    </QueryProvider>
  </I18nProvider>
);

export default Providers;
```

## ThemeProvider (components/Provider/ThemeProvider.tsx)

```typescript
import { useEffect } from "react";
import { useCommonStore } from "@/stores/common";
import usePrefersColor from "@/hooks/usePrefersColor";

const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  const { themeMode, prefersColorScheme } = useCommonStore();

  // Listen for system theme changes
  usePrefersColor();

  useEffect(() => {
    const root = document.documentElement;
    const isDark =
      themeMode === "dark" ||
      (themeMode === "system" && prefersColorScheme === "dark");

    root.classList.toggle("dark", isDark);
  }, [themeMode, prefersColorScheme]);

  return <>{children}</>;
};

export default ThemeProvider;
```

## Routes (routes/index.tsx)

```typescript
import type { RouteObject } from "react-router";
import Layout from "@/components/Layout";
import NotFound from "@/pages/404";
import Home from "@/pages/home";
import About from "@/pages/about";

export const routes: RouteObject[] = [
  {
    path: "/",
    element: <Layout />,
    children: [
      { path: "/", element: <Home /> },
      { path: "/about", element: <About /> },
      { path: "*", element: <NotFound /> }
    ]
  }
];
```

## Layout (components/Layout/index.tsx)

```typescript
import { Outlet } from "react-router";
import Header from "./Header";
import Footer from "./Footer";

const Layout = () => (
  <div className="flex min-h-screen flex-col">
    <Header />
    <main className="flex-1">
      <Outlet />
    </main>
    <Footer />
  </div>
);

export default Layout;
```

## Header (components/Layout/Header.tsx)

```typescript
import { Link } from "react-router";

const Header = () => (
  <header className="border-b border-slate-200 bg-white dark:border-slate-800 dark:bg-slate-900">
    <div className="mx-auto flex h-16 max-w-5xl items-center justify-between px-4">
      <Link to="/" className="text-xl font-bold text-slate-900 dark:text-white">
        React Starter
      </Link>
      <nav className="flex gap-4">
        <Link to="/" className="text-slate-600 hover:text-slate-900 dark:text-slate-400 dark:hover:text-white">
          Home
        </Link>
        <Link to="/about" className="text-slate-600 hover:text-slate-900 dark:text-slate-400 dark:hover:text-white">
          About
        </Link>
      </nav>
    </div>
  </header>
);

export default Header;
```

## Footer (components/Layout/Footer.tsx)

```typescript
const Footer = () => (
  <footer className="border-t border-slate-200 bg-white py-6 dark:border-slate-800 dark:bg-slate-900">
    <div className="mx-auto max-w-5xl px-4 text-center text-sm text-slate-500 dark:text-slate-400">
      © {new Date().getFullYear()} React Starter. All rights reserved.
    </div>
  </footer>
);

export default Footer;
```

## About Page (pages/about/index.tsx)

```typescript
import { Trans } from "@lingui/react/macro";

const About = () => (
  <div className="mx-auto max-w-5xl px-4 py-16">
    <h1 className="mb-4 text-3xl font-bold text-slate-900 dark:text-white">
      <Trans>About</Trans>
    </h1>
    <p className="text-slate-600 dark:text-slate-400">
      <Trans>This is a modern React starter template.</Trans>
    </p>
  </div>
);

export default About;
```

## 404 Page (pages/404/index.tsx)

```typescript
import { Trans } from "@lingui/react/macro";
import { Link } from "react-router";

const NotFound = () => (
  <div className="flex min-h-[60vh] flex-col items-center justify-center px-4">
    <h1 className="mb-2 text-6xl font-bold text-slate-900 dark:text-white">404</h1>
    <p className="mb-6 text-slate-600 dark:text-slate-400">
      <Trans>Page not found</Trans>
    </p>
    <Link
      to="/"
      className="rounded-lg bg-slate-900 px-4 py-2 text-white hover:bg-slate-800 dark:bg-white dark:text-slate-900 dark:hover:bg-slate-200"
    >
      <Trans>Go Home</Trans>
    </Link>
  </div>
);

export default NotFound;
```

## Page Example (pages/home/index.tsx)

This example demonstrates all core features: Tailwind CSS, Zustand, React Router, and Lingui i18n.

```typescript
import { Trans } from "@lingui/react/macro";
import { useLingui } from "@lingui/react";
import { Link, useLocation } from "react-router";
import { useCommonStore } from "@/stores/common";

const LANGUAGES = [
  { code: "en", label: "English" },
  { code: "ja", label: "日本語" },
  { code: "ko", label: "한국어" },
  { code: "tw", label: "繁體中文" },
  { code: "vi", label: "Tiếng Việt" },
  { code: "fr", label: "Français" },
  { code: "es", label: "Español" },
  { code: "pt", label: "Português" }
];

const Home = () => {
  const { i18n } = useLingui();
  const { themeMode, setThemeMode } = useCommonStore();
  const location = useLocation();

  const handleLanguageChange = (lang: string) => {
    localStorage.setItem("lang", lang);
    i18n.activate(lang);
  };

  return (
    <div className="min-h-screen bg-gray-50/50 dark:bg-gray-900">
      <div className="mx-auto max-w-5xl px-4 py-16">
        {/* Header */}
        <div className="mb-16 text-center">
          <h1 className="mb-4 text-4xl font-extrabold tracking-tight text-slate-900 dark:text-white sm:text-5xl">
            <span className="bg-linear-to-r from-blue-600 to-indigo-600 bg-clip-text text-transparent">
              React Starter
            </span>
          </h1>
          <p className="mx-auto max-w-2xl text-lg text-slate-600 dark:text-slate-400">
            A modern, production-ready template powered by React 19, Vite 7, and Tailwind CSS v4.
            Everything you need to build faster.
          </p>
        </div>

        {/* Feature Cards */}
        <div className="grid gap-6 md:grid-cols-2 lg:gap-8">
          {/* 1. Tailwind CSS */}
          <section className="group relative overflow-hidden rounded-2xl border border-slate-200 bg-white p-6 shadow-sm transition-all hover:-translate-y-1 hover:shadow-md dark:border-slate-800 dark:bg-slate-800/50">
            <div className="mb-4 flex items-center gap-3">
              <span className="flex h-10 w-10 items-center justify-center rounded-full bg-blue-100/50 text-sm font-bold text-blue-600 dark:bg-blue-900/30 dark:text-blue-400">
                1
              </span>
              <h2 className="text-xl font-bold text-slate-900 dark:text-white">
                Tailwind CSS
              </h2>
            </div>
            <div className="space-y-4">
              <div className="rounded-xl bg-slate-50 p-4 transition-colors group-hover:bg-blue-50/50 dark:bg-slate-900 dark:group-hover:bg-slate-800">
                <p className="font-medium text-slate-700 dark:text-slate-300">
                  <Trans>Utility-first CSS framework</Trans>
                </p>
                <div className="mt-3 flex flex-wrap gap-2">
                  <div className="h-2 w-8 rounded-full bg-blue-400/20"></div>
                  <div className="h-2 w-12 rounded-full bg-blue-400/20"></div>
                  <div className="h-2 w-6 rounded-full bg-blue-400/20"></div>
                </div>
              </div>
              <div className="flex flex-wrap gap-2">
                {["v4 Engine", "Dark Mode", "Custom Theme"].map(tag => (
                  <span key={tag} className="inline-flex items-center rounded-full border border-blue-100 bg-blue-50 px-2.5 py-0.5 text-xs font-semibold text-blue-700 dark:border-blue-900 dark:bg-blue-900/10 dark:text-blue-400">
                    {tag}
                  </span>
                ))}
              </div>
            </div>
          </section>

          {/* 2. Zustand */}
          <section className="group relative overflow-hidden rounded-2xl border border-slate-200 bg-white p-6 shadow-sm transition-all hover:-translate-y-1 hover:shadow-md dark:border-slate-800 dark:bg-slate-800/50">
            <div className="mb-4 flex items-center gap-3">
              <span className="flex h-10 w-10 items-center justify-center rounded-full bg-amber-100/50 text-sm font-bold text-amber-600 dark:bg-amber-900/30 dark:text-amber-400">
                2
              </span>
              <h2 className="text-xl font-bold text-slate-900 dark:text-white">
                Zustand
              </h2>
            </div>
            <div className="space-y-4">
              <div className="flex items-center justify-between rounded-xl bg-slate-50 px-4 py-3 dark:bg-slate-900">
                <span className="text-sm text-slate-500 dark:text-slate-400">
                  <Trans>Theme:</Trans>
                </span>
                <span className="font-mono text-sm font-semibold text-amber-600 dark:text-amber-400">
                  {themeMode}
                </span>
              </div>
              <div className="flex gap-2">
                {(["light", "dark"] as const).map((mode) => (
                  <button
                    type="button"
                    key={mode}
                    onClick={() => setThemeMode(mode)}
                    className={`flex-1 rounded-lg border px-3 py-1.5 text-sm font-medium transition-all ${themeMode === mode
                      ? "border-amber-200 bg-amber-50 text-amber-700 dark:border-amber-800 dark:bg-amber-900/20 dark:text-amber-400"
                      : "border-transparent bg-slate-50 text-slate-600 hover:bg-slate-100 dark:bg-slate-900 dark:text-slate-400 dark:hover:bg-slate-800"
                      }`}
                  >
                    {mode.charAt(0).toUpperCase() + mode.slice(1)}
                  </button>
                ))}
              </div>
            </div>
          </section>

          {/* 3. React Router */}
          <section className="group relative overflow-hidden rounded-2xl border border-slate-200 bg-white p-6 shadow-sm transition-all hover:-translate-y-1 hover:shadow-md dark:border-slate-800 dark:bg-slate-800/50">
            <div className="mb-4 flex items-center gap-3">
              <span className="flex h-10 w-10 items-center justify-center rounded-full bg-rose-100/50 text-sm font-bold text-rose-600 dark:bg-rose-900/30 dark:text-rose-400">
                3
              </span>
              <h2 className="text-xl font-bold text-slate-900 dark:text-white">
                Router v7
              </h2>
            </div>
            <div className="space-y-4">
              <div className="flex items-center justify-between rounded-xl bg-slate-50 px-4 py-3 dark:bg-slate-900">
                <span className="text-sm text-slate-500 dark:text-slate-400">
                  <Trans>Path:</Trans>
                </span>
                <span className="font-mono text-sm font-medium text-slate-900 dark:text-white">
                  {location.pathname}
                </span>
              </div>
              <div className="flex gap-2">
                <Link
                  to="/about"
                  className="flex-1 rounded-lg bg-slate-900 px-3 py-2 text-center text-sm font-medium text-white transition-colors hover:bg-slate-800 dark:bg-white dark:text-slate-900 dark:hover:bg-slate-200"
                >
                  About
                </Link>
                <Link
                  to="/404"
                  className="flex-1 rounded-lg bg-slate-100 px-3 py-2 text-center text-sm font-medium text-slate-600 transition-colors hover:bg-slate-200 dark:bg-slate-800 dark:text-slate-300 dark:hover:bg-slate-700"
                >
                  404 Page
                </Link>
              </div>
            </div>
          </section>

          {/* 4. Lingui i18n */}
          <section className="group relative overflow-hidden rounded-2xl border border-slate-200 bg-white p-6 shadow-sm transition-all hover:-translate-y-1 hover:shadow-md dark:border-slate-800 dark:bg-slate-800/50">
            <div className="mb-4 flex items-center gap-3">
              <span className="flex h-10 w-10 items-center justify-center rounded-full bg-emerald-100/50 text-sm font-bold text-emerald-600 dark:bg-emerald-900/30 dark:text-emerald-400">
                4
              </span>
              <h2 className="text-xl font-bold text-slate-900 dark:text-white">
                LinguiJS
              </h2>
            </div>
            <div className="space-y-4">
              <div className="flex items-center justify-between rounded-xl bg-slate-50 px-4 py-3 dark:bg-slate-900">
                <span className="text-sm text-slate-500 dark:text-slate-400">
                  <Trans>Locale:</Trans>
                </span>
                <span className="font-mono text-sm font-bold text-emerald-600 dark:text-emerald-400">
                  {i18n.locale.toUpperCase()}
                </span>
              </div>
              <div className="flex flex-wrap gap-2">
                {LANGUAGES.map(({ code, label }) => (
                  <button
                    type="button"
                    key={code}
                    onClick={() => handleLanguageChange(code)}
                    className={`rounded-lg border px-2 py-1.5 text-xs font-medium transition-all ${i18n.locale === code
                      ? "border-emerald-200 bg-emerald-50 text-emerald-700 dark:border-emerald-800 dark:bg-emerald-900/20 dark:text-emerald-400"
                      : "border-transparent bg-slate-50 text-slate-600 hover:bg-slate-100 dark:bg-slate-900 dark:text-slate-400 dark:hover:bg-slate-800"
                      }`}
                  >
                    {label}
                  </button>
                ))}
              </div>
            </div>
          </section>
        </div>

        {/* Tech Stack Pills */}
        <div className="mt-16 text-center">
          <p className="mb-6 text-sm font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-500">
            <Trans>Powered by</Trans>
          </p>
          <div className="flex flex-wrap justify-center gap-3">
            {[
              "React 19", "TypeScript", "Vite 7", "Tailwind CSS v4",
              "React Router 7", "TanStack Query", "Zustand", "Lingui", "Axios"
            ].map((tech) => (
              <span
                key={tech}
                className="rounded-full bg-white px-4 py-2 text-sm font-medium text-slate-700 shadow-sm ring-1 ring-slate-900/5 dark:bg-slate-800 dark:text-slate-300 dark:ring-white/10"
              >
                {tech}
              </span>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
};

export default Home;
```

## Using Lingui

```typescript
// In components - use Trans macro
import { Trans } from "@lingui/react/macro";
<Trans>Hello World</Trans>

// With variables
import { Trans } from "@lingui/react/macro";
<Trans>Hello {name}</Trans>

// In hooks/logic - use useLingui
import { msg } from "@lingui/core/macro";
import { useLingui } from "@lingui/react";
const { _ } = useLingui();
const message = _(msg`Hello World`);

// Switch language
import { useLingui } from "@lingui/react";
const { i18n } = useLingui();
localStorage.setItem("lang", "ja");
i18n.activate("ja");

// Extract & compile translations
// 1. pnpm i18n:extract  - Extracts all translatable strings to .po files
// 2. Edit src/locales/{lang}.po - Add translations: msgid (key) → msgstr (translated text)
// 3. pnpm i18n:compile  - Compiles .po files to runtime format
```

## User Types (types/user.ts)

```typescript
export interface ILoginReq {
  email: string;
  password: string;
}

export interface ILoginInfo {
  token: string;
  user: IUser;
}

export interface IUser {
  id: string;
  name: string;
  email: string;
}
```

## API Usage Example (api/user.ts)

```typescript
import request from "@/api/request";
import type { ILoginReq, ILoginInfo, IUser } from "@/types/user";

export const login = (data: ILoginReq) =>
  request.post<ILoginInfo>("/user/login", data);

export const getUser = (id: string) =>
  request.get<IUser>(`/user/${id}`);

export const updateUser = (id: string, data: Partial<IUser>) =>
  request.post<IUser>(`/user/${id}`, data);
```

## React Query Usage

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getUser, updateUser } from "@/api/user";

// Query
const useUser = (id: string) =>
  useQuery({
    queryKey: ["user", id],
    queryFn: () => getUser(id)
  });

// Mutation
const useUpdateUser = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<IUser> }) =>
      updateUser(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ["user", id] });
    }
  });
};
```

## vercel.json
```
{
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}

```
