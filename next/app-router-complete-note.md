# Next.js App Router — Complete Learning Notes

> **Focus:** Next.js App Router-এর routing structure, special files, Parallel Routes, Intercepting Routes এবং Private Folders।
>
> এই নোটগুলো Next.js **16.x App Router** মাথায় রেখে লেখা। `proxy.ts`-এর মতো version-sensitive convention-গুলোতে version note দেওয়া হয়েছে।

---

## সূচিপত্র

1. [App Router কীভাবে কাজ করে](#1-app-router-কীভাবে-কাজ-করে)
2. [Basic Routing Structure](#2-basic-routing-structure)
3. [Special Files — Quick Reference](#3-special-files--quick-reference)
4. [Page, Layout ও Template](#4-page-layout-ও-template)
5. [Loading UI](#5-loading-ui)
6. [Error Handling](#6-error-handling)
7. [Not Found Handling](#7-not-found-handling)
8. [Route Handler — `route.ts`](#8-route-handler--routets)
9. [Proxy — `proxy.ts`](#9-proxy--proxyts)
10. [Metadata ও SEO Files](#10-metadata-ও-seo-files)
11. [Instrumentation Files](#11-instrumentation-files)
12. [Parallel Routes](#12-parallel-routes)
13. [`default.tsx` কেন প্রয়োজন](#13-defaulttsx-কেন-প্রয়োজন)
14. [Intercepting Routes](#14-intercepting-routes)
15. [Private Folders](#15-private-folders)
16. [Other Folder Conventions](#16-other-folder-conventions)
17. [Production Project Structure](#17-production-project-structure)
18. [Production Best Practices](#18-production-best-practices)
19. [Practice Exercise](#19-practice-exercise)
20. [Official Documentation](#20-official-documentation)

---

# 1. App Router কীভাবে কাজ করে

Next.js App Router হলো file-system-based routing system। অর্থাৎ `app/` directory-র ভেতরে folder ও special file তৈরি করে application-এর route এবং UI structure define করা হয়।

উদাহরণ:

```text
src/
└── app/
    ├── page.tsx
    ├── about/
    │   └── page.tsx
    └── products/
        └── page.tsx
```

এখানে:

- `app/page.tsx` → `/`
- `app/about/page.tsx` → `/about`
- `app/products/page.tsx` → `/products`

**মনে রাখবে:** সাধারণ folder URL-এর segment তৈরি করে, কিন্তু route-এর UI তৈরি করার জন্য সাধারণত সেই segment-এ `page.tsx` বা `route.ts`-এর মতো উপযুক্ত special file প্রয়োজন।

App Router-এর মূল সুবিধা:

- Nested layouts
- Server Components ও Client Components
- Streaming এবং loading UI
- Error boundaries
- Route handlers
- Parallel ও Intercepting Routes
- File-based metadata conventions

---

# 2. Basic Routing Structure

## 2.1 Static Route

```text
app/
└── about/
    └── page.tsx
```

URL:

```text
/about
```

## 2.2 Dynamic Route

URL-এর কোনো অংশ dynamic হলে square bracket ব্যবহার করা হয়।

```text
app/
└── products/
    └── [id]/
        └── page.tsx
```

উদাহরণ URL:

```text
/products/101
/products/202
```

Next.js 15+ এবং 16-এ `params` Promise হিসেবে ব্যবহার করতে হয়:

```tsx
type Props = {
  params: Promise<{ id: string }>;
};

export default async function ProductPage({ params }: Props) {
  const { id } = await params;

  return <h1>Product ID: {id}</h1>;
}
```

## 2.3 Catch-all Route

```text
app/
└── docs/
    └── [...slug]/
        └── page.tsx
```

এটি এক বা একাধিক URL segment match করে।

উদাহরণ:

```text
/docs/getting-started
/docs/api/authentication
/docs/api/authentication/tokens
```

`params.slug` হবে string array।

## 2.4 Optional Catch-all Route

```text
app/
└── docs/
    └── [[...slug]]/
        └── page.tsx
```

এটি `/docs`-কেও match করে, slug না থাকলেও।

---

# 3. Special Files — Quick Reference

App Router-এ নির্দিষ্ট নামের file বিশেষ কাজ করে।

| File | কাজ |
|---|---|
| `page.tsx` | Route-এর UI render করে |
| `layout.tsx` | Shared, persistent layout |
| `template.tsx` | Navigation-এ নতুন component instance |
| `default.tsx` | Parallel Route slot-এর unmatched fallback |
| `loading.tsx` | Loading UI / Suspense fallback |
| `error.tsx` | Route segment-এর error boundary |
| `global-error.tsx` | Root-level error boundary |
| `not-found.tsx` | Not Found UI |
| `route.ts` | HTTP Route Handler / API endpoint |
| `forbidden.tsx` | Forbidden UI |
| `unauthorized.tsx` | Unauthorized UI |
| `proxy.ts` | Request interception / routing decisions |
| `sitemap.ts` | Sitemap generate করে |
| `robots.ts` | Robots instructions generate করে |
| `manifest.ts` | Web app manifest generate করে |
| `opengraph-image.*` | Open Graph share image |
| `twitter-image.*` | Twitter/X share image |
| `icon.*` | Site icon |
| `apple-icon.*` | Apple touch icon |
| `favicon.ico` | Browser favicon |
| `instrumentation.ts` | Server instrumentation |
| `instrumentation-client.ts` | Client instrumentation |

প্রতিটি special file-এর কাজ আলাদা। যেমন `default.tsx` loading fallback নয়; `error.tsx` 404 handler নয়; আর `route.ts` UI page নয়।

---

# 4. Page, Layout ও Template

## 4.1 `page.tsx`

`page.tsx` একটি route-এর মূল UI তৈরি করে।

```text
app/
└── dashboard/
    └── page.tsx
```

```tsx
export default function DashboardPage() {
  return <h1>Dashboard</h1>;
}
```

এটি `/dashboard` route-এর UI।

## 4.2 `layout.tsx`

একই route segment-এর একাধিক page-এ shared UI রাখার জন্য layout ব্যবহার করা হয়।

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── analytics/
    │   └── page.tsx
    └── users/
        └── page.tsx
```

`layout.tsx`:

```tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <aside>Dashboard Sidebar</aside>
      <main>{children}</main>
    </div>
  );
}
```

এখন `/dashboard`, `/dashboard/analytics` এবং `/dashboard/users` একই dashboard layout ব্যবহার করবে।

### Layout-এর গুরুত্বপূর্ণ বৈশিষ্ট্য

- Shared UI reuse হয়।
- Navigation-এর সময় layout সাধারণত mounted থাকে।
- Layout-এর state সাধারণত ধরে রাখা হয়।
- Root layout-এ `<html>` ও `<body>` থাকতে হয়।
- Nested layout parent layout-এর ভেতরে render হয়।

Navbar, sidebar এবং persistent dashboard shell-এর জন্য `layout.tsx` উপযুক্ত।

## 4.3 `template.tsx`

`template.tsx` দেখতে layout-এর মতো, কিন্তু navigation-এর সময় নতুন component instance তৈরি হয়। ফলে এর local state reset হয় এবং effects আবার চালু হতে পারে।

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── template.tsx
    └── page.tsx
```

```tsx
export default function DashboardTemplate({
  children,
}: {
  children: React.ReactNode;
}) {
  return <div>{children}</div>;
}
```

### Layout বনাম Template

| বিষয় | Layout | Template |
|---|---|---|
| Navigation-এর সময় | সাধারণত state ধরে রাখে | নতুন instance তৈরি করে |
| Local state | Persist করতে পারে | Reset হয় |
| Effects | সাধারণত একই instance থাকে | আবার mount হতে পারে |
| Use case | Navbar, sidebar, shared shell | Route transition বা fresh state দরকার হলে |

**Rule of thumb:** সাধারণ shared layout-এর জন্য `layout.tsx` ব্যবহার করবে। বিশেষভাবে নতুন instance দরকার হলেই `template.tsx` ব্যবহার করবে।

---

# 5. Loading UI — `loading.tsx`

কোনো route segment-এর content load বা stream হওয়ার সময় loading UI দেখাতে `loading.tsx` ব্যবহার হয়।

```text
app/
└── dashboard/
    ├── loading.tsx
    └── page.tsx
```

```tsx
export default function Loading() {
  return <p>Dashboard is loading...</p>;
}
```

Next.js এটিকে ওই segment-এর content-এর চারপাশে Suspense fallback হিসেবে ব্যবহার করে।

### কখন কাজে লাগে?

- Slow data fetching
- Route transition-এর সময় loading feedback
- বড় dashboard page-এর content stream হওয়া
- Nested route-এর loading state

### গুরুত্বপূর্ণ পার্থক্য

`loading.tsx` মানেই browser-এ HTML/JavaScript load হওয়ার আগের global splash screen নয়। এটি Next.js-এর route rendering এবং Suspense lifecycle-এর অংশ। পুরো application-এর JavaScript browser-এ না পৌঁছালে React-ভিত্তিক loading UI render করতে পারবে না।

---

# 6. Error Handling

## 6.1 `error.tsx`

একটি route segment-এর rendering বা descendant component-এ uncaught error হলে fallback UI দেখায়।

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── error.tsx
    └── page.tsx
```

`error.tsx` অবশ্যই Client Component হতে হবে।

```tsx
"use client";

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <section>
      <h2>Something went wrong.</h2>
      <p>{error.message}</p>

      <button onClick={() => reset()}>
        Try again
      </button>
    </section>
  );
}
```

- `error` → error-এর তথ্য।
- `reset()` → error boundary-এর content আবার render করার চেষ্টা করে।

### গুরুত্বপূর্ণ: নিজের layout-এর error

একটি segment-এর `error.tsx` সেই segment-এর `layout.tsx`-এর error ধরতে পারে না, কারণ error boundary layout-এর ভেতরের content-কে wrap করে। ওই layout-এর error ধরতে parent segment-এর error boundary দরকার।

## 6.2 `global-error.tsx`

Root layout-এ error হলে `global-error.tsx` ব্যবহার করা হয়।

```text
app/
├── layout.tsx
├── global-error.tsx
└── page.tsx
```

এটি root layout-কে replace করে। তাই এখানে নিজের `<html>` এবং `<body>` দিতে হয়।

```tsx
"use client";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h1>Application Error</h1>
        <button onClick={() => reset()}>
          Try again
        </button>
      </body>
    </html>
  );
}
```

## 6.3 Production best practices

- User-facing UI-তে sensitive stack trace দেখাবে না।
- Error details server-side monitoring-এ log করবে।
- `reset()` সব ধরনের permanent error fix করে না; এটি retry attempt।
- Authentication এবং authorization error-এর জন্য শুধু generic error boundary-এর ওপর নির্ভর করবে না।

---

# 7. Not Found Handling — `not-found.tsx`

কোনো resource পাওয়া না গেলে custom 404 UI দেখাতে `not-found.tsx` ব্যবহার হয়।

```text
app/
├── not-found.tsx
└── products/
    └── [id]/
        ├── page.tsx
        └── not-found.tsx
```

Global not-found:

```tsx
export default function NotFound() {
  return <h1>Page not found</h1>;
}
```

Product page:

```tsx
import { notFound } from "next/navigation";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = await getProduct(id);

  if (!product) {
    notFound();
  }

  return <h1>{product.name}</h1>;
}
```

`notFound()` call করলে নিকটবর্তী উপযুক্ত `not-found.tsx` render হয়।

### `not-found.tsx` বনাম `error.tsx`

- Resource বা route পাওয়া যায়নি → `not-found.tsx`
- Rendering বা application logic-এ unexpected error → `error.tsx`

---

# 8. Route Handler — `route.ts`

`route.ts` দিয়ে Next.js App Router-এর মধ্যে HTTP endpoint তৈরি করা যায়।

এটি UI page নয়; HTTP request receive করে response return করে।

```text
app/
└── api/
    └── users/
        └── route.ts
```

```tsx
import { NextRequest, NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({
    message: "Users fetched successfully",
  });
}

export async function POST(request: NextRequest) {
  const body = await request.json();

  return NextResponse.json(
    {
      message: "User created",
      data: body,
    },
    { status: 201 }
  );
}
```

এখন:

- `GET /api/users`
- `POST /api/users`

Route Handler-এ `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD` এবং `OPTIONS` export করা যায়।

### `page.tsx` বনাম `route.ts`

| `page.tsx` | `route.ts` |
|---|---|
| UI render করে | HTTP response দেয় |
| Browser page-এর জন্য | API/webhook/HTTP endpoint-এর জন্য |
| React component export করে | HTTP method function export করে |

একই route segment-এ `page.tsx` এবং `route.ts` একসঙ্গে রাখা যায় না।

### Production use cases

- REST API
- Webhook endpoint
- Third-party service callback
- File download
- Custom HTTP response

### Security

Route Handler-এ প্রতিটি protected request-এর জন্য authentication ও authorization যাচাই করতে হবে। শুধু frontend button hide করলেই API secure হয় না।

---

# 9. Proxy — `proxy.ts`

Next.js 16-এ `middleware.ts` convention-এর নাম পরিবর্তন করে `proxy.ts` করা হয়েছে।

এটি request-এর response তৈরি হওয়ার আগে request inspect করে routing decision নিতে পারে।

Project structure:

```text
src/
├── app/
│   └── dashboard/
│       └── page.tsx
└── proxy.ts
```

Example:

```tsx
import { NextRequest, NextResponse } from "next/server";

export function proxy(request: NextRequest) {
  const token = request.cookies.get("token");

  if (
    !token &&
    request.nextUrl.pathname.startsWith("/dashboard")
  ) {
    return NextResponse.redirect(
      new URL("/login", request.url)
    );
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

### Proxy-এর কাজ

- Redirect
- Rewrite
- Request/response header নিয়ে কাজ
- Cookie inspect
- নির্দিষ্ট path-এ request filtering

### গুরুত্বপূর্ণ Security Warning

Proxy-তে cookie/token check করলেই পুরো application secure হয়ে যায় না।

Production application-এ:

- Server Component-এ protected data access-এর আগে authorization যাচাই করবে।
- Server Action-এ authorization যাচাই করবে।
- Route Handler-এ authorization যাচাই করবে।
- Database query-তেও user-এর permission enforce করবে।

Proxy হলো request-level convenience/optimization layer; এটি সম্পূর্ণ authorization system-এর বিকল্প নয়।

---

# 10. Metadata ও SEO Files

Next.js metadata API দিয়ে page title, description, social sharing image ইত্যাদি define করা যায়।

## 10.1 `metadata` object

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "My Store",
  description: "Explore our products",
};

export default function HomePage() {
  return <h1>Welcome to My Store</h1>;
}
```

এটি Server Component-এ export করতে হয়।

## 10.2 `generateMetadata()`

Dynamic data অনুযায়ী metadata তৈরি করতে:

```tsx
import type { Metadata } from "next";

type Props = {
  params: Promise<{ id: string }>;
};

export async function generateMetadata({
  params,
}: Props): Promise<Metadata> {
  const { id } = await params;
  const product = await getProduct(id);

  return {
    title: product.name,
    description: product.description,
  };
}
```

## 10.3 Metadata file conventions

```text
app/
├── favicon.ico
├── icon.png
├── apple-icon.png
├── opengraph-image.png
├── twitter-image.png
├── sitemap.ts
├── robots.ts
└── manifest.ts
```

| File | কাজ |
|---|---|
| `favicon.ico` | Browser tab icon |
| `icon.png` | Site icon |
| `apple-icon.png` | Apple touch icon |
| `opengraph-image.png` | Open Graph sharing image |
| `twitter-image.png` | Twitter/X sharing image |
| `sitemap.ts` | Sitemap generate |
| `robots.ts` | Crawling rules |
| `manifest.ts` | PWA manifest |

### Sitemap example

```tsx
import type { MetadataRoute } from "next";

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: "https://example.com",
      lastModified: new Date(),
    },
    {
      url: "https://example.com/about",
      lastModified: new Date(),
    },
  ];
}
```

এটি `/sitemap.xml` তৈরি করে।

### Robots example

```tsx
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: "*",
      allow: "/",
      disallow: "/admin/",
    },
    sitemap: "https://example.com/sitemap.xml",
  };
}
```

**Note:** `robots.ts` crawler-কে নির্দেশ দেয়; এটি authentication বা access control নয়। Sensitive route secure করতে server-side authorization লাগবে।

---

# 11. Instrumentation Files

Observability, monitoring এবং telemetry setup-এর জন্য:

| File | কাজ |
|---|---|
| `instrumentation.ts` | Server instrumentation |
| `instrumentation-client.ts` | Client-side instrumentation |

Project root অথবা `src/`-এ রাখা যায়।

Production-এ এগুলো দিয়ে error monitoring, tracing, OpenTelemetry integration এবং performance tracking setup করা যায়।

এগুলো সাধারণ page component নয়।

---

# 12. Parallel Routes

## 12.1 Parallel Routes কী?

Parallel Routes দিয়ে একই layout-এর মধ্যে একাধিক independent route slot render করা যায়।

ধরো একটি SaaS dashboard-এ একই সময়ে দেখাতে হবে:

- Main dashboard content
- Analytics panel
- Notifications panel

প্রতিটি panel-কে আলাদা slot হিসেবে manage করতে Parallel Routes ব্যবহার করা যায়।

**Important:** Parallel Routes মানে একাধিক API request একসঙ্গে করা নয়। এটি routing এবং UI composition-এর feature।

## 12.2 Folder convention

`@` দিয়ে শুরু হওয়া folder হলো Parallel Route slot।

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── @analytics/
    │   └── page.tsx
    └── @notifications/
        └── page.tsx
```

এখানে:

- `@analytics` → analytics slot
- `@notifications` → notifications slot
- `@` folder URL segment তৈরি করে না।

তাই URL:

```text
/dashboard
```

`/dashboard/analytics` নয়।

## 12.3 Layout-এ slot receive করা

```tsx
type DashboardLayoutProps = {
  children: React.ReactNode;
  analytics: React.ReactNode;
  notifications: React.ReactNode;
};

export default function DashboardLayout({
  children,
  analytics,
  notifications,
}: DashboardLayoutProps) {
  return (
    <div className="grid grid-cols-3 gap-4">
      <main className="col-span-2">
        {children}
      </main>

      <aside>
        {analytics}
        {notifications}
      </aside>
    </div>
  );
}
```

`children` আসলে implicit `@children` slot-এর মতো কাজ করে।

## 12.4 Slot page examples

`@analytics/page.tsx`:

```tsx
export default function AnalyticsPage() {
  return (
    <section>
      <h2>Analytics</h2>
      <p>Total revenue: $12,000</p>
    </section>
  );
}
```

`@notifications/page.tsx`:

```tsx
export default function NotificationsPage() {
  return (
    <section>
      <h2>Notifications</h2>
      <p>You have 3 new notifications.</p>
    </section>
  );
}
```

## 12.5 Production use cases

- SaaS dashboard: overview + analytics + activity feed
- Admin panel: users + metrics + alerts
- Collaboration app: workspace + messages + notifications
- Multi-panel application যেখানে slot-specific navigation দরকার

### কখন ব্যবহার করবে না?

শুধু UI-কে column বা component-এ ভাগ করতে Parallel Routes দরকার নেই। সাধারণ React component দিয়েও করা যায়।

Parallel Routes ব্যবহার করবে যখন আলাদা slot-এর independent routing state, navigation বা fallback দরকার।

---

# 13. `default.tsx` কেন প্রয়োজন

Parallel Routes-এর সঙ্গে `default.tsx`-এর সম্পর্ক বোঝা জরুরি।

ধরো:

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── @analytics/
    │   └── page.tsx
    └── @notifications/
        └── page.tsx
```

Next.js soft navigation-এর সময় parallel slot-এর আগের active state ধরে রাখতে পারে।

কিন্তু browser refresh বা hard navigation হলে URL থেকে প্রতিটি slot-এর আগের active state সবসময় পুনরুদ্ধার করা সম্ভব হয় না।

তখন Next.js unmatched slot-এর জন্য `default.tsx` খোঁজে।

## 13.1 Example

```text
dashboard/
├── layout.tsx
├── page.tsx
├── @analytics/
│   ├── page.tsx
│   └── default.tsx
└── @notifications/
    ├── page.tsx
    └── default.tsx
```

`@analytics/default.tsx`:

```tsx
export default function DefaultAnalytics() {
  return null;
}
```

`@notifications/default.tsx`:

```tsx
export default function DefaultNotifications() {
  return null;
}
```

`null` return করলে ওই slot-এ কিছু render হবে না।

## 13.2 `default.tsx` বনাম `loading.tsx`

| `default.tsx` | `loading.tsx` |
|---|---|
| Unmatched parallel slot-এর fallback | Loading UI |
| Route state restore না হলে কাজে লাগে | Content load/stream হওয়ার সময় কাজে লাগে |
| `null` return করতে পারে | সাধারণত spinner/skeleton দেখায় |

**মনে রাখবে:** `default.tsx` কোনো homepage বা generic default configuration নয়। এটি file-system routing-এর নির্দিষ্ট fallback convention।

---

# 14. Intercepting Routes

## 14.1 Intercepting Routes কী?

একটি route-এ navigation করার সময় সেটিকে অন্য route context-এর মধ্যে render করতে Intercepting Routes ব্যবহার করা যায়।

Classic industry example: Instagram-style photo modal।

ধরো:

1. User `/feed`-এ আছে।
2. Feed থেকে photo click করল।
3. URL হলো `/photo/123`।
4. কিন্তু full page না দেখিয়ে feed-এর ওপর modal দেখানো হলো।
5. User সরাসরি `/photo/123` open করলে full photo page দেখানো হলো।

**একই URL, কিন্তু navigation context আলাদা বলে UI আলাদা।**

## 14.2 সাধারণ routing-এর সমস্যা

```text
app/
├── feed/
│   └── page.tsx
└── photo/
    └── [id]/
        └── page.tsx
```

এখানে Feed থেকে `/photo/123`-এ গেলে সাধারণত full photo page-ই দেখাবে। Modal experience তৈরি হবে না।

## 14.3 Intercepting Routes + Parallel Routes

```text
app/
├── layout.tsx
├── feed/
│   └── page.tsx
├── photo/
│   └── [id]/
│       └── page.tsx
└── @modal/
    ├── default.tsx
    └── (.)photo/
        └── [id]/
            └── page.tsx
```

এখানে:

- `photo/[id]/page.tsx` → Direct URL-এর full photo page
- `@modal` → Parallel Route slot
- `(.)photo/[id]/page.tsx` → Matching navigation intercept করে modal UI দেখায়
- `@modal/default.tsx` → Modal slot-এর default fallback

Root layout:

```tsx
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}
```

Modal route:

```tsx
export default async function PhotoModal({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/70">
      <div className="rounded-xl bg-white p-8">
        <h2>Photo Modal</h2>
        <p>Photo ID: {id}</p>
      </div>
    </div>
  );
}
```

Full page:

```tsx
export default async function PhotoPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return (
    <main>
      <h1>Full Photo Page</h1>
      <p>Photo ID: {id}</p>
    </main>
  );
}
```

Feed link:

```tsx
import Link from "next/link";

export default function FeedPage() {
  return (
    <Link href="/photo/123">
      Open Photo
    </Link>
  );
}
```

## 14.4 Navigation-এর ফলাফল

### Feed থেকে click — soft navigation

- URL: `/photo/123`
- Feed-এর content থাকে।
- `@modal` slot-এ photo modal render হয়।

### Direct URL বা refresh — hard navigation

- URL: `/photo/123`
- Full photo page render হয়।
- `@modal` slot তার default fallback পায়।

## 14.5 Intercepting convention

| Convention | Meaning |
|---|---|
| `(.)photo` | একই route level |
| `(..)photo` | এক level উপরের route |
| `(..)(..)photo` | দুই level উপরের route |
| `(...)photo` | App root থেকে route |

**Important:** এগুলো filesystem-এর folder depth গুনে না; URL route segment-এর relative level বোঝায়। `@modal`-এর মতো Parallel Route slot-কে URL segment হিসেবে গণনা করা হয় না।

## 14.6 Industry use cases

- Instagram/Facebook photo modal
- E-commerce product quick view
- Gallery lightbox
- Social post preview
- Dashboard details modal

### কখন ব্যবহার করবে না?

যদি modal-এর জন্য URL পরিবর্তন বা direct-link support প্রয়োজন না হয়, সাধারণ React modal যথেষ্ট হতে পারে।

---

# 15. Private Folders

## 15.1 `_components` কী?

App Router-এ `_` দিয়ে শুরু হওয়া folder হলো private folder। এটি routing system থেকে বাদ থাকে।

উদাহরণ:

```text
app/
└── dashboard/
    ├── page.tsx
    └── _components/
        ├── DashboardHeader.tsx
        ├── Sidebar.tsx
        └── StatsCard.tsx
```

`_components` কোনো route তৈরি করে না।

এর ভেতরের component import করে ব্যবহার করা যায়:

```tsx
import DashboardHeader from "./_components/DashboardHeader";

export default function DashboardPage() {
  return (
    <>
      <DashboardHeader />
      <h1>Dashboard</h1>
    </>
  );
}
```

## 15.2 কেন দরকার?

বড় application-এ route-এর নিজস্ব component, hook, helper এবং internal implementation একই জায়গায় গুছিয়ে রাখা যায়।

উদাহরণ:

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── analytics/
    │   └── page.tsx
    ├── users/
    │   └── page.tsx
    │
    ├── _components/
    │   ├── DashboardSidebar.tsx
    │   ├── DashboardHeader.tsx
    │   └── StatsCard.tsx
    │
    ├── _hooks/
    │   └── useDashboardFilters.ts
    │
    └── _lib/
        └── formatRevenue.ts
```

Routes:

- `/dashboard`
- `/dashboard/analytics`
- `/dashboard/users`

Private implementation:

- `_components`
- `_hooks`
- `_lib`

## 15.3 `_components` কি বাধ্যতামূলক?

না। Private folder optional convention।

Globally reusable component-এর জন্য project-level `components/` folder রাখা যায়:

```text
src/
├── app/
│   ├── dashboard/
│   └── products/
│
├── components/
│   ├── ui/
│   ├── shared/
│   └── forms/
│
└── lib/
```

### Recommended organization

- Globally reusable UI → `src/components/`
- নির্দিষ্ট route-এর private component → `app/dashboard/_components/`
- Route-specific hooks → `app/dashboard/_hooks/`
- Shared utilities → `src/lib/`
- Feature-specific utilities → feature-এর private folder

**Important:** `_components` JavaScript-এর private access modifier নয়। অন্য file থেকেও import করা যায়। এটি শুধু Next.js routing system থেকে folder-কে বাদ দেয়।

---

# 16. Other Folder Conventions

## 16.1 Route Groups — `(auth)`

Parentheses দিয়ে folder তৈরি করলে সেটি URL segment তৈরি করে না।

```text
app/
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
└── (marketing)/
    ├── about/
    │   └── page.tsx
    └── pricing/
        └── page.tsx
```

URLs:

- `/login`
- `/register`
- `/about`
- `/pricing`

Use cases:

- Route organize করা
- আলাদা layout group তৈরি করা
- Auth pages এবং marketing pages আলাদা রাখা

## 16.2 Dynamic Segment — `[id]`

```text
products/[id]/page.tsx
```

একটি dynamic URL segment।

## 16.3 Catch-all — `[...slug]`

এক বা একাধিক segment match করে।

## 16.4 Optional Catch-all — `[[...slug]]`

Zero বা একাধিক segment match করে।

## 16.5 Parallel Slot — `@modal`

একই layout-এ independent slot তৈরি করে। URL segment নয়।

## 16.6 Intercepting Segment — `(.)`, `(..)`, `(...)`

Navigation context অনুযায়ী route intercept করে।

## 16.7 Private Folder — `_components`

Routing থেকে folder বাদ দেয়।

---

# 17. Production Project Structure

একটি SaaS dashboard-এর জন্য সম্ভাব্য structure:

```text
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── global-error.tsx
│   ├── not-found.tsx
│   ├── robots.ts
│   ├── sitemap.ts
│   │
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   │
│   ├── dashboard/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   │
│   │   ├── @analytics/
│   │   │   ├── page.tsx
│   │   │   └── default.tsx
│   │   │
│   │   ├── @notifications/
│   │   │   ├── page.tsx
│   │   │   └── default.tsx
│   │   │
│   │   ├── analytics/
│   │   │   └── page.tsx
│   │   │
│   │   ├── users/
│   │   │   └── page.tsx
│   │   │
│   │   ├── _components/
│   │   │   ├── DashboardHeader.tsx
│   │   │   └── DashboardSidebar.tsx
│   │   │
│   │   └── _lib/
│   │       └── formatMetrics.ts
│   │
│   ├── products/
│   │   ├── page.tsx
│   │   └── [id]/
│   │       └── page.tsx
│   │
│   └── api/
│       └── users/
│           └── route.ts
│
├── components/
│   ├── ui/
│   └── shared/
│
├── lib/
│   ├── db.ts
│   └── auth.ts
│
└── proxy.ts
```

এটি একটি illustrative structure—সব project-এ প্রতিটি convention লাগবে না। Project-এর complexity এবং feature অনুযায়ী structure ছোট বা বড় করবে।

---

# 18. Production Best Practices

1. **প্রতিটি special file প্রয়োজন অনুযায়ী ব্যবহার করবে।** শুধু convention আছে বলে file তৈরি করবে না।
2. **Layout-এ persistent UI রাখবে।** Navbar, sidebar এবং shared shell-এর জন্য।
3. **Template শুধু প্রয়োজন হলে ব্যবহার করবে।** প্রতিটি route navigation-এ fresh instance দরকার হলে।
4. **Loading ও error state design করবে।** Slow network এবং failure production-এ স্বাভাবিক ঘটনা।
5. **Route Handler-এ security enforce করবে।** Client-side UI hide করা security নয়।
6. **Proxy-কে complete authorization layer ভাববে না।** Protected data access-এর জায়গায় permission check করবে।
7. **Parallel Routes অকারণে ব্যবহার করবে না।** Independent slot navigation দরকার হলে ব্যবহার করবে।
8. **Parallel slot-এর `default.tsx` বিবেচনা করবে।** Hard navigation ও unmatched slot-এর fallback ঠিক রাখবে।
9. **Intercepting Routes শুধু প্রয়োজনীয় UX-এ ব্যবহার করবে।** URL shareability এবং modal navigation দরকার হলে।
10. **Private folder দিয়ে route-specific code organize করবে।** Shared component global location-এ রাখবে।
11. **Metadata ও SEO setup করবে।** Title, description, sitemap, robots এবং social image।
12. **Sensitive information client-এ পাঠাবে না।** Server-side authorization এবং data filtering ঠিক রাখবে।

---

# 19. Practice Exercise

## Exercise 1 — Route Structure

একটি e-commerce app-এর জন্য নিচের routes তৈরি করার folder structure লেখো:

- `/`
- `/products`
- `/products/123`
- `/login`
- `/register`
- `/dashboard`
- `/dashboard/users`

শর্ত:
- Login ও register একই route group-এ থাকবে।
- Product ID dynamic হবে।
- Dashboard-এর নিজস্ব layout থাকবে।

## Exercise 2 — Parallel Routes

Dashboard-এ একই সঙ্গে দেখাতে হবে:

- Main dashboard
- Analytics
- Notifications

শর্ত:
- `@analytics` ও `@notifications` slot থাকবে।
- Layout-এ slot props receive করবে।
- প্রতিটি slot-এর `default.tsx` থাকবে।

## Exercise 3 — Intercepting Routes

Product listing থেকে product click করলে modal খুলবে।

কিন্তু `/products/123` direct URL-এ open করলে full product page দেখাবে।

শর্ত:
- `@modal` slot ব্যবহার করবে।
- `(.)` convention ব্যবহার করবে।
- Modal slot-এর `default.tsx` থাকবে।

## Exercise 4 — Private Folder

Dashboard-এর জন্য নিচের code organize করো:

- DashboardHeader
- DashboardSidebar
- useDashboardFilters
- formatCurrency

কোনগুলো `_components`, `_hooks`, `_lib`-এ যাবে এবং কোনগুলো global shared folder-এ যাবে, যুক্তিসহ ব্যাখ্যা করো।

---

# 20. Official Documentation

Next.js-এর official documentation থেকে version-specific behaviour মিলিয়ে নেবে।

- [App Router](https://nextjs.org/docs/app)
- [File-system conventions](https://nextjs.org/docs/app/api-reference/file-conventions)
- [Project structure](https://nextjs.org/docs/app/getting-started/project-structure)
- [Layouts and Pages](https://nextjs.org/docs/app/getting-started/layouts-and-pages)
- [Loading UI and Streaming](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)
- [Error Handling](https://nextjs.org/docs/app/getting-started/error-handling)
- [Parallel Routes](https://nextjs.org/docs/app/building-your-application/routing/parallel-routes)
- [Intercepting Routes](https://nextjs.org/docs/app/building-your-application/routing/intercepting-routes)
- [Route Handlers](https://nextjs.org/docs/app/getting-started/route-handlers)
- [Proxy](https://nextjs.org/docs/app/api-reference/file-conventions/proxy)
- [Metadata](https://nextjs.org/docs/app/getting-started/metadata-and-og-images)

---

## Final Mental Model

App Router শেখার সময় এই mapping মাথায় রাখবে:

- **`page.tsx`** → কোন UI দেখাবে?
- **`layout.tsx`** → কোন UI shared থাকবে?
- **`template.tsx`** → কখন নতুন component instance দরকার?
- **`loading.tsx`** → content আসা পর্যন্ত কী দেখাবে?
- **`error.tsx`** → rendering error হলে কী দেখাবে?
- **`not-found.tsx`** → resource না পেলে কী দেখাবে?
- **`route.ts`** → HTTP request-এর response কী হবে?
- **`proxy.ts`** → request আসার সময় কী routing decision নিতে হবে?
- **`@slot`** → একই layout-এ কোন independent UI slot render হবে?
- **`default.tsx`** → Parallel slot-এর matching UI না থাকলে কী হবে?
- **`(.)route`** → navigation-এর context অনুযায়ী কোন route intercept হবে?
- **`_folder`** → কোন implementation folder routing-এর বাইরে থাকবে?

এগুলো বুঝে গেলে Next.js App Router-এর routing architecture অনেক পরিষ্কার হয়ে যাবে।