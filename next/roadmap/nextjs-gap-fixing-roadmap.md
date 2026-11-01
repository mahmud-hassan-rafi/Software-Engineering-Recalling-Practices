# Next.js Gap-Fixing Roadmap

**Goal:** Build strong Next.js fundamentals and production-level problem-solving skills.

**Duration:** 4 Weeks
**Stack:** Next.js 16+ | App Router | TypeScript

Progress: 0 / 29 topics

---

## Week 1 — Rendering Fundamentals

**Goal:** Understand how Next.js renders pages and when rendering happens.

- [ ] Build Time vs Request Time
- [ ] Static vs Dynamic Rendering
- [ ] Prerendering
- [ ] `generateStaticParams()`
- [ ] `connection()`

### Practical Tasks

- [ ] Create a static route.
- [ ] Create a dynamic route.
- [ ] Test direct URL access and refresh.
- [ ] Compare build-time and request-time rendering.
- [ ] Explain the rendering lifecycle without notes.

**Week 1 Review**

- [ ] I understand the mental models.
- [ ] I can implement these concepts independently.
- [ ] I can debug common rendering errors.

---

## Week 2 — Caching Architecture

**Goal:** Understand Next.js caching and data-fetching strategies.

- [ ] `use cache`
- [ ] `cacheLife`
- [ ] `cacheTag`
- [ ] `revalidateTag`
- [ ] Cache Invalidation
- [ ] Database Fetching + Caching

### Practical Tasks

- [ ] Cache a data-fetching function.
- [ ] Experiment with different cache lifetimes.
- [ ] Implement tag-based cache invalidation.
- [ ] Fetch MongoDB data using a reusable data-access function.
- [ ] Test stale and refreshed data.
- [ ] Explain when caching should and should not be used.

**Week 2 Review**

- [ ] I understand cache keys and cache lifetime.
- [ ] I can implement caching independently.
- [ ] I can choose an appropriate caching strategy.

---

## Week 3 — Advanced Routing

**Goal:** Understand Next.js navigation and route composition.

- [ ] Parallel Routes
- [ ] Intercepting Routes
- [ ] Soft Navigation vs Hard Navigation
- [ ] Browser History
- [ ] Modal Routing
- [ ] Nested Layouts

### Practical Tasks

- [ ] Create a parallel route.
- [ ] Create an intercepting route.
- [ ] Build a modal with URL-based navigation.
- [ ] Test direct navigation vs client-side navigation.
- [ ] Test browser Back and Forward.
- [ ] Navigate from modal to canonical full page.
- [ ] Implement proper fallback routes.

**Week 3 Review**

- [ ] I understand the route tree.
- [ ] I can explain soft and hard navigation.
- [ ] I can build modal routing independently.

---

## Week 4 — Production Problem Solving

**Goal:** Develop independent debugging and architectural decision-making skills.

- [ ] Metadata Generation
- [ ] Loading and Error Boundaries
- [ ] Suspense Boundaries
- [ ] Not Found Handling
- [ ] Debugging Next.js Errors
- [ ] Architecture Decisions

### Practical Tasks

- [ ] Implement dynamic metadata.
- [ ] Implement loading and error states.
- [ ] Handle invalid dynamic routes.
- [ ] Debug a Suspense-related error.
- [ ] Investigate a Blocking Route error.
- [ ] Read official documentation before choosing a fix.
- [ ] Explain why one architectural approach is better than another.

**Week 4 Review**

- [ ] I can investigate errors independently.
- [ ] I can use official documentation effectively.
- [ ] I can justify my architecture decisions.

---

## Final Assessment

- [ ] I can build a Next.js App Router project without following a tutorial.
- [ ] I understand rendering and caching strategies.
- [ ] I can implement advanced routing patterns.
- [ ] I can debug framework-level errors.
- [ ] I can explain my technical decisions in an interview.

## Learning Rule

For every topic, follow this process:

1. Understand the mental model.
2. Read the official documentation.
3. Implement a small practical example.
4. Test edge cases and failure scenarios.
5. Write personal notes in Markdown.
6. Explain the concept without looking at notes.

**Important:** A topic is complete only when I can implement and explain it independently—not merely when I have watched a tutorial.
