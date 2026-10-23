# Web Development Prompts — Production-Grade AI Prompt Templates

> Curated prompts for frontend, full-stack, and performance-critical web development. React, Next.js, Angular, accessibility, and more. Based on the [CRTSE framework](best-practices.md).

---

## React / Next.js

### Data table component with server-side pagination
```
Act as a senior React developer (TypeScript, Next.js 15, React 19 specialist).

I'm building an admin dashboard that displays a large dataset (10K+ user records) in a sortable, filterable, paginated table.

Task: Implement the `UserTable` component with server-side pagination, sorting, and filtering.

Requirements:
- Use React 19 Server Components for the initial render (data fetched on server)
- Client component handles sorting/filtering/pagination via URL search params
- Pagination: page size selector (10, 25, 50, 100), prev/next buttons, showing "1-25 of 10,342"
- Sorting: click column header to toggle asc/desc; sort indicator icon
- Filtering: text input that filters by name/email (debounced 300ms)
- All state changes update the URL so the table state is shareable/bookmarkable
- Skeleton loading state while data fetches (not a spinner)
- TypeScript: every prop, state variable, and API response fully typed

Do NOT:
- Use `any` or `as` type assertions
- Fetch data in `useEffect` (use React Query or RSC data fetching)
- Store filter/sort state in component-local useState without URL sync
- Use `index` as the React key

Our existing table pattern:
```tsx
// app/admin/users/page.tsx
export default async function UsersPage({ searchParams }: { 
  searchParams: Promise<{ page?: string; sort?: string; filter?: string }> 
}) {
  const params = await searchParams;
  const data = await fetchUsers(params);
  return <UserTable initialData={data} />;
}
```

Output: the server component (page.tsx), client component (UserTable.tsx), and the fetchUsers server action.
```

### Form with validation (React Hook Form + Zod)
```
Act as a senior React developer.

I'm building a multi-step onboarding form for a SaaS product.

Task: Implement the "Company Details" step of the form:
- Company name (required, 2-100 chars)
- Website URL (optional, must be valid URL if provided)
- Industry (select: Technology, Healthcare, Finance, Education, Other)
- Employee count (number, min 1, max 1,000,000)
- A "Back" and "Next" button (next disabled until form is valid)

Requirements:
- React Hook Form v7 with @hookform/resolvers for Zod validation
- All Zod schemas defined in a separate file (reusable across wizard steps)
- Accessible: all inputs have labels, error messages linked via aria-describedby, keyboard navigable
- Form state preserved across wizard step navigation (use a context or Zustand store)
- Server-side re-validation on Next button click (dry run against API, show server errors inline)
- Optimistic UI: show "Saving..." on the Next button while validating

Do NOT:
- Use uncontrolled inputs with refs — use React Hook Form's register
- Show validation errors before the field has been touched (use mode: 'onTouched')
- Allow form submission if invalid (Next should be disabled)

Output: the CompanyDetailsStep component, the Zod schema, and the form context provider.
```

### Accessible modal dialog
```
Act as a senior frontend accessibility engineer.

I need a reusable Modal component that meets WCAG 2.2 AA.

Task: Build an accessible Modal with:
- Focus trap (Tab/Shift+Tab cycles within modal, can't tab to background)
- Escape key closes the modal
- Clicking the backdrop closes the modal
- Focus returns to the trigger element on close
- Announced as a dialog to screen readers (role="dialog", aria-modal, aria-labelledby)
- Body scroll locked while modal is open
- Animated enter/exit (CSS transitions, respects prefers-reduced-motion)
- Render via React Portal (createPortal to document.body)

Requirements:
- All props typed (isOpen, onClose, title, children, size: 'sm' | 'md' | 'lg' | 'full')
- The component must be reusable — no hardcoded content or styles specific to one use case
- Include a visually hidden "Close" button for screen readers
- Renders nothing (null) when isOpen is false

Do NOT:
- Use `display: none` to hide (prevents animation exit)
- Trap focus without an escape mechanism
- Use z-index battles (single z-index layer managed by the component)

Output: the Modal component with its CSS module.
```

---

## Angular

### Signal-based service with rxResource
```
Act as a senior Angular engineer (Angular 19+, Signals, standalone components).

I'm building a real-time dashboard with Angular 19 standalone components and RxJS.

Task: Implement an `OrderStreamService` that:
- Opens a WebSocket connection to `/ws/orders` on initialization
- Exposes a `Signal<Order[]>` of all active orders (updated in real-time)
- Exposes a `Signal<OrderStats>` with total count, total value, count by status
- Reconnects automatically with exponential backoff (1s, 2s, 4s, max 30s)
- Shows connection status as a `Signal<ConnectionStatus>` (connected/disconnected/reconnecting)

Requirements:
- Use Angular's `rxResource` for initial data load + real-time updates via a `Subject`
- All Signals read-only from outside the service (private writable signals)
- Connection status is a computed signal derived from the WebSocket state
- Inject the service as a singleton (providedIn: 'root')
- Clean up WebSocket and subscriptions in `ngOnDestroy`

Do NOT:
- Subscribe inside components — components only read Signals
- Use `setInterval` for reconnection checking
- Throw from the WebSocket error handler (handle gracefully)

Output: the complete OrderStreamService.
```

---

## Performance & Core Web Vitals

### Optimize for LCP and INP
```
Act as a senior web performance engineer.

Our Next.js e-commerce product page has poor Core Web Vitals: LCP 4.2s, INP 380ms. The hero product image is a 2.4MB PNG blocking LCP, and the add-to-cart handler has a synchronous bundle that blocks INP.

Task: Optimize this page to pass Core Web Vitals (LCP < 2.5s, INP < 200ms).

Requirements:
- Replace the hero image with next/image using WebP/AVIF with responsive sizes, lazy loading, and priority on the LCP image
- Add a blur placeholder (blurDataURL) for the hero image
- Defer non-critical JavaScript (analytics, chat widget, recommendations carousel) — inject after idle callback
- Break the add-to-cart handler into a minimal synchronous response (optimistic UI update) + async validation
- Audit and remove unused JavaScript (show what tree-shaking misses)
- Add resource hints: preconnect to the CDN, prefetch the checkout page

Do NOT:
- Suggest a full rewrite
- Use `setTimeout(func, 0)` as a code-splitting hack
- Disable SSR or hydration

Output: the specific code changes to page.tsx, next.config.js, and the image/script components — with before/after LCP and INP estimates.
```