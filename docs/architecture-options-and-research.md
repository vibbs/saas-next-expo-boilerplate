Great. I’ll put together a complete technical documentation structure and research-based recommendations for your SaaS product boilerplate project using the specified stack. This will include:

* A detailed technical architecture overview
* Folder structure with explanations
* Best practices for shared packages (`app-core`)
* Evaluation of UI/component libraries like shadcn/ui for both Next.js and Expo
* Navigation handling strategy across web and mobile, including Solito evaluation
* Cross-platform animation strategy with Framer Motion or alternatives
* Reminder functionality implementation via Supabase or other services
* Deployment strategy including Vercel for Next.js and EAS/CI-CD for mobile
* Versioning, auto-deployment pipelines, and end-to-end testing setup (web and mobile)

I’ll get started on this and let you know as soon as it’s ready for review.


# SaaS Boilerplate: Technical Architecture Overview

**Overview:** This document provides a comprehensive technical overview of a SaaS product boilerplate. It covers the monorepo folder structure, technology stack, and design decisions for a Next.js web app and an Expo mobile app sharing a common core. We address cross-platform navigation, animations, UI libraries, scheduling, deployment, testing, and TypeScript best practices. The goal is a **solopreneur-friendly** architecture that follows industry best practices, ensuring easy reuse and smooth onboarding for future team members.

## Monorepo Project Structure

The boilerplate is organized as a monorepo with clearly separated **application** projects and shared **package** modules. Below is the high-level folder structure:

```plaintext
/ (root)
├── applications/
│   ├── next/               # Next.js 15 (App Router) web application
│   │   └── app/            # Next.js App Router directory
│   │       ├── (marketing)/    # Unprotected route group (landing pages, blog, etc.)
│   │       ├── (app)/          # Protected route group (dashboard, app pages)
│   │       └── ... (other routes and files)
│   └── expo/               # Expo React Native application (iOS & Android)
│       ├── android/            # Android native project (managed by Expo)
│       ├── ios/                # iOS native project (managed by Expo)
│       ├── App.tsx             # Expo entry point
│       └── ... (other Expo config files like app.json)
└── packages/
    └── app-core/           # Shared core modules
        ├── ui/                # Shared UI components (React/React Native)
        ├── database/          # Database & Supabase integration (schema, client)
        ├── auth/              # Authentication logic & UI (forms, workflows)
        ├── config/            # Configuration (env variables, security, tests)
        └── modules/           # Feature modules (e.g. home, user) with platform-agnostic logic
```

**Applications vs Packages:** The `applications` directory holds product-specific frontends (web and mobile), while `packages/app-core` contains reusable modules used by both apps. This separation enforces modularity—mobile and web can share code for business logic and UI where feasible, reducing duplication.

## Next.js Web Application (applications/next)

The web app is built with **Next.js 15** using the App Router. It uses **Route Groups** to partition pages into public vs. protected sections without affecting URL structure.

* **Unprotected Routes:** The `(marketing)` route group contains public-facing pages like the landing page, marketing pages, blog, docs, etc. These routes do not require authentication and have their own layout (e.g., a marketing site header/footer). Using Next.js Route Groups, we can organize these pages under `app/(marketing)` while keeping clean URLs.

* **Protected Routes:** The `(app)` route group contains application pages (dashboard, user settings, etc.) that require user login. This group can share an authenticated layout (e.g., sidebar navigation, protected header) and include logic to redirect to login if the user is not authenticated. Next.js 15 supports multiple root layouts, so the protected section can have its own layout and even its own `html`/`body` wrappers if needed.

**Routing & Layout:** With the App Router, each group has a `layout.tsx` to define UI chrome and enforce auth. For example, `app/(app)/layout.tsx` can include logic (running on the server) to check a Supabase auth session or cookie and redirect to `/login` if missing. This avoids flicker by securing pages server-side. Meanwhile, `app/(marketing)/layout.tsx` would not perform auth checks. The top-level `app/layout.tsx` can include shared providers (theme, context) that apply to the whole app.

**Auth in Next.js:** Next.js integrates well with Supabase Auth or other providers. We can use **Next.js Middleware** (`middleware.ts`) to protect routes as well, e.g., redirecting unknown users away from `/app/*` paths. Supabase provides helpers for Next.js (like `createServerComponentClient`) to retrieve session info inside server components. The auth UI (login/register forms) is implemented in the shared `auth` package (see **Shared Auth Module** below) but used within Next pages. For instance, the login page might be `app/(marketing)/login/page.tsx` which imports a `<LoginForm />` from `app-core/auth` and, on success, redirects to the protected dashboard.

## Expo Mobile Application (applications/expo)

The mobile app is built with **Expo (React Native)**, targeting iOS and Android from a single codebase. It lives in `applications/expo` and can leverage Expo’s managed workflow (no need to eject unless adding custom native modules).

**Navigation:** The Expo app uses **React Navigation** for screen routing (the de-facto standard in React Native). It may also integrate with a cross-platform navigation solution (see **Cross-Platform Navigation** below) to share route definitions with the Next.js app. Typically, the Expo app will have a stack or tab navigator for the main application screens (dashboard, etc.) and separate navigator for auth flow (login, onboarding). Using Expo’s CLI and libraries, we can also handle deep links and universal links such that clicking a link can open the app to the appropriate screen.

Expo SDK provides many libraries for native functionality (camera, notifications, etc.), which can be added without ejecting. The Expo app’s entry point (`App.tsx`) sets up global providers similar to Next (theme provider, Apollo/SWR for data fetching if used, Supabase client, etc.). It then conditionally renders either the authenticated navigator or the public auth screens based on user state.

**Using Shared Modules:** The Expo app imports code from `packages/app-core`. For example, screens in Expo can use shared UI components from `app-core/ui` and call shared business logic in `app-core/modules`. This means features like a `UserProfile` component or a `useUserData` hook can be written once and used on both platforms. The Expo project is configured with a monorepo-friendly setup (for instance, using Yarn/NPM workspaces and Metro config to include the `app-core` package). This ensures live reloading in development works across the monorepo.

**Styling:** Styling in React Native is done via stylesheets or libraries like Tailwind via NativeWind, or styled-components. We maintain a consistent design system across web and mobile by using the same design tokens (colors, spacing, etc.) in both. Tools like **NativeWind** (Tailwind CSS in React Native) allow us to use Tailwind classes in Expo, aligning with Tailwind usage on the web. This helps achieve a consistent look and feel.

## Shared Core Packages (packages/app-core)

The `app-core` package houses reusable code that is platform-agnostic or abstracts platform differences. This is key for a **solopreneur** setup, enabling maximum reuse of code between Next.js and Expo.

### UI Component Library (packages/app-core/ui)

This folder contains shared UI components that can be used in both the Next.js and Expo apps. Examples might include buttons, form inputs, cards, etc., designed to look consistent on web and mobile. To achieve cross-platform compatibility, these components need to be carefully implemented:

* **React Native Web Approach:** One strategy is to implement components using React Native primitives (`View`, `Text`, etc.) and styles, and use **React Native Web** to render them on the web. This way, the same component code works on native and web. With this approach, our Next.js app can import components from `app-core/ui` and, under the hood, those use `react-native-web` to translate to `<div>` and `<span>` in the DOM.

* **shadcn/ui Compatibility:** The design language of the UI might be inspired by [**shadcn/ui**](https://ui.shadcn.com) (a popular library of Radix UI + Tailwind components for web). However, **shadcn/ui does not directly support React Native** – it uses web-specific primitives (e.g., `<div>`), Radix UI (which is web-only), and Tailwind CSS classes. Attempting to use it in Expo will fail unless the components are re-written for RN. Given this, we evaluate alternatives:

  * *Use separate implementations:* We could use shadcn/ui for the Next.js app (web) and a similar-looking component library for React Native. For example, [**React Native Reusables**](https://github.com/mrzachnugent/react-native-reusables) is a community project that ports shadcn-style components to React Native using NativeWind. This library aims to provide “universal shadcn/ui” components with accessibility and theming in mind (it effectively recreates Radix-like components for mobile). Early reports show promise, but it may require careful integration and may not yet cover all components.
  * *Use a cross-platform UI library:* [**Tamagui**](https://tamagui.dev/) is an alternative that supports both web and native. It provides a unified styling system (though not Tailwind-based) and pre-built components. Tamagui can achieve a consistent UI across platforms, but it uses its own style syntax (style props) instead of Tailwind utility classes. The design may differ from Radix UI's out-of-the-box look.
  * *Build custom shared components:* As a straightforward approach, we can create our own UI kit in `app-core/ui` with base components (buttons, inputs, etc.) using React Native primitives + NativeWind for styling. Then use those in Expo, and for Next.js either use them via React Native Web or provide thin wrapper components that apply Tailwind classes for web. This requires more upfront work but gives full control.

**Recommendation:** For maximum reuse, building a custom component library or using a cross-platform solution is ideal. If you want to leverage **Tailwind CSS consistently**, consider using **NativeWind** in React Native and normal Tailwind in Next.js, sharing the Tailwind config. The gist by Fernando Rojo (creator of Solito) highlights the gap in cross-platform component libraries: a Radix+Tailwind style system isn't natively available on React Native, and while Tamagui fills the gap, it doesn't use Tailwind utility classes. As a result, you might choose to use **shadcn/ui on web only**, and use a close equivalent on mobile (via NativeWind) to maintain a similar design. Over time, the community projects (like **react-native-reusables**) may mature to let you “copy-paste” shadcn components into your project, but be prepared for some platform-specific adjustments.

### Database Integration (packages/app-core/database)

This module is responsible for all data access, using **Supabase** as the backend (which provides a Postgres database, auth, and storage). Key aspects of `database` module include:

* **Database Schema & Types:** We can integrate Supabase by generating types from the Postgres schema (using Supabase CLI or the `supabase` JS client’s type generation). These TypeScript types are shared so that both the web and mobile app know the shape of the data (e.g., a `User` type, etc.). This ensures type-safe database queries across the app.

* **Supabase Client:** The module can export a configured Supabase client instance (or a function to create one) that is used by both Next.js and Expo. For Next.js, we might use the Supabase JS client for server-side calls (or via API routes), and for Expo, use the Supabase JS client which works in React Native as well. The `database` config can include the URL and public anon key for Supabase (taking them from environment variables). In Next.js, we keep these keys server-side (Next can use Edge Functions or server components to avoid exposing secrets), whereas in Expo, the anon key is embedded but rules in Supabase (RLS) protect data.

* **Auth Integration:** Supabase’s auth system is tied to the database. The `database` module may include SQL scripts or references to Supabase migration files that set up authentication schema (like the `auth.users` table and any custom profiles table). It also defines Row-Level Security (RLS) policies if needed (for multi-tenant data separation, etc.). Documentation or links to the Supabase schema should be included for transparency.

* **Data Access Abstractions:** We can create functions in this module that abstract direct queries. For example, a `getUserProfile(userId)` function that queries the `profiles` table. These functions use the Supabase client internally. By centralizing these, if we switch to a different backend or need caching, we can adjust here without affecting other code. Both the Next.js app (e.g. in a server action or API route) and the Expo app can import these helpers.

### Authentication Module (packages/app-core/auth)

All authentication logic and UI is consolidated here to avoid duplication. It includes:

* **Auth Workflows:** Functions for sign-up, login, logout, password reset, etc., using Supabase Auth under the hood. For example, `auth/signUp(email, password)` might call `supabase.auth.signUp()` and handle any error messages. Similarly, social login (OAuth) logic can be encapsulated here (though on native, web OAuth flows may require redirect handling via Expo AuthSession, etc.). By centralizing, both apps follow the same processes.

* **Auth UI Components:** Ready-made forms for authentication can be provided. For web (Next.js), we might have React components for `<LoginForm>` and `<RegisterForm>` styled with our UI library (shadcn or custom). For mobile, we can have analogous components using React Native UI. Ideally, these forms are built using the shared `ui` components so that they look consistent. If using React Native Web, we could even have one set of components that work cross-platform. For simplicity, one could also implement the forms separately if needed (given differences in keyboard handling on mobile, etc.), but keep them in the same module for consistency.

* **State Management:** The auth module might also manage user session state. For instance, it could export a React context provider that stores the current user and provides hooks like `useAuth()` to get the user and loading state. This provider can wrap the app in both Next.js (using a client-side context or simply rely on Next SSR for session) and Expo (using React context or libraries like Zustand for state). Supabase’s auth state changes (token refresh, etc.) should be handled here so that both frontends respond to logouts or token expiry gracefully.

Security best practices are followed: passwords never stored in plaintext (Supabase handles hashing), JWTs or session cookies are HttpOnly in web, etc. Also, since this is a boilerplate, the auth module can be easily swapped out for another provider if needed (e.g., Firebase Auth or NextAuth), by isolating changes within this package.

### Configuration Management (packages/app-core/config)

This module centralizes configuration for environment variables, security settings, and testing configurations:

* **Environment Variables:** Both Next.js and Expo need access to certain environment settings (Supabase URL, keys, etc.). Next.js uses environment files (`.env.local`) and can expose some variables to the browser (prefixed with `NEXT_PUBLIC_`). Expo uses an app config (`app.json` or `app.config.js`) where we can define extra config, or uses dotenv via babel plugin. In `app-core/config`, we can have a TypeScript file that reads from `process.env` (for Next) or from Expo Constants (for Expo) and exports a unified config object. This ensures the keys (like `SUPABASE_URL`) are defined in one place. For security, sensitive keys (like service role key for Supabase) would only be used on server (Next API or Supabase Edge Functions) and not included in the Expo app.

* **Security Config:** Any security-related constants or helpers can reside here. For example, content security policy (CSP) settings for Next.js, or in Expo, configurations for secure storage (to store tokens). If we have common crypto routines (like for encrypting local data or verifying something), they go here.

* **Feature Flags / Env-based Config:** We might toggle certain features based on environment (development vs production). This module can expose booleans or values for those, e.g., `__DEV__` flags or enabling/disabling mock data. Expo already provides a global `__DEV__` and Next has `process.env.NODE_ENV`, but a unified config can make it easier to adjust behavior in shared code.

* **Testing Setup:** Any configuration for testing frameworks can be here. For example, if using Jest for unit tests in packages, a base Jest config could live here (or at the root). Also any test utilities (like loading test env variables, setting up a test database schema or using Supabase’s in-memory or sandbox mode if possible) might be provided.

### Feature Modules (packages/app-core/modules)

The `modules` directory contains self-contained feature modules (for example, `home`, `user`, etc.). Each module encapsulates the domain logic and UI for a specific feature of the app, promoting separation of concerns. This structure makes it easy to onboard new developers by assigning them to a specific module without affecting others.

A feature module typically includes:

* **UI (Screens/Components):** For instance, a `user` module might export a `UserProfileScreen` (for mobile) and a `UserProfilePage` (for web) that both use a common `UserProfileView` component under the hood. The components in modules can utilize the shared UI library. In some cases, the UI is simple enough to be identical on web and mobile; in other cases, you might have slight variations and thus separate implementations, but kept in one module directory for coherence.

* **Business Logic:** Functions related to that feature, e.g., `updateUserProfile(data)` which calls the `database` module to update the DB and maybe triggers other side effects. These functions can be used by both the Next.js app (in an API route or server action) and the Expo app (directly calling, which then calls Supabase via the client). They ensure consistent behavior across platforms.

* **State Management:** If a feature needs local state management or context, that can be scoped to the module. For example, a `home` module might manage a feed of posts in state. This can be achieved with React context or Zustand/MobX store that is used in both apps. By colocating it, we ensure the web and mobile use the same state logic (reducing bugs where behavior diverges).

**Example:** A `notifications` module could hold code to register device tokens (for push notifications in Expo) and store notification preferences. The Next.js app might not use device tokens, but could use the same data structures to mark notifications as read, etc. The module would contain all logic for notifications so that enabling/disabling that feature is modular.

By structuring the app into modules, the boilerplate is scalable. A solo developer can build features in isolation, and later a team can work on separate modules without stepping on each other’s toes. This pattern also aligns with domain-driven design, making future refactoring or extraction (maybe into microservices) easier.

## Cross-Platform Navigation Strategy

One of the core challenges in a web + mobile monorepo is handling navigation. We have two main approaches: use a cross-platform abstraction like **Solito**, or implement separate navigation logic for each platform. Here’s a comparison:

| Navigation Approach            | Pros                                                                                                                                                                                                                                                                       | Cons                                                                                                                                                                                                                                                                                                |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Solito (Shared Navigation)** | - Allows sharing navigation code (routes, links) across web and native.<br>- Provides unified hooks (`useRouter`) and components (like `<Link>`) that work on both Next.js and React Navigation.<br>- Ensures consistent deep linking and route naming between apps.       | - Additional dependency and abstraction to learn.<br>- May not cover 100% of navigation use-cases; might require workarounds for complex native patterns.<br>- Slightly coupled to specific versions of Next and React Navigation (maintenance overhead when upgrading).                            |
| **Custom (Platform-Specific)** | - Full control of navigation on each platform (Next.js uses its App Router, Expo uses React Navigation directly).<br>- Can tailor user experience per platform (e.g., native modals vs web page routes) without abstraction constraints.<br>- Fewer external dependencies. | - Duplicate effort: must define routes and navigation logic twice (once for web, once for mobile).<br>- Increases risk of inconsistencies (e.g., a route exists on mobile but not implemented on web, or named differently).<br>- Deep linking between web and mobile requires manual coordination. |
| **Expo Router (Alternative)**  | - Expo Router offers file-based routing for React Native apps, similar to Next.js App Router, improving intuitiveness for web developers.<br>- Still uses React Navigation under the hood, but lets you define screens via filesystem.                                     | - Not directly compatible with Next.js routes — you’d still maintain a separate Next.js routing setup.<br>- Solito integration with Expo Router is possible but not as straightforward; you might lose some benefits of unified code.                                                               |

**Solito Evaluation:** **Solito** is a library by Fernando Rojo that “is the missing piece” to combine React Native and Next.js navigation. It provides a lightweight abstraction: on web it uses Next’s router, on native it uses React Navigation, but you call a common API. For example, you could navigate using `useRouter().push("/profile")` in shared code, and it will route correctly on both platforms. Solito also offers cross-platform link components and even animations integration via Moti (for transitions). This can greatly speed development as you write navigation code once. Solito works well for typical stack-and-tab navigation patterns. However, if your apps diverge in navigation structure (say the mobile app uses a lot of modals or a different screen flow), you might end up with custom handling anyway.

If used, Solito would typically reside in the `app-core` (or be an external dependency configured to know about your routes). You’d define navigation routes (maybe a route config or just use file systems and Solito infers). Protected routes can also be handled: for example, Solito’s docs provide patterns for auth (wrapping Next’s `<Link>` and React Navigation links with auth checks).

**Custom Navigation:** The alternative is to let Next.js and React Navigation do their own thing. In Next.js, you naturally use the App Router and `<Link>` components or `useRouter()`. In Expo, you set up a `StackNavigator` and define screens. You might still share route *names* or enums in a config file so that both apps use the same identifiers for screens where it makes sense (to avoid drift). This approach might be simpler to reason about in isolation and avoids any Solito-specific constraints. For a solopreneur, the trade-off is writing navigation twice, but it’s straightforward if the app is not too complex.

**Recommendation:** For this boilerplate, using **Solito** is recommended if the goal is maximum code sharing and you're comfortable with the abstraction. It has community adoption and handles a lot of cross-platform quirks. For example, Solito addresses React Navigation’s web quirks (like needing to use `Linking` or treat modals specially). However, keep in mind that if you encounter an edge case, you might need to dive into platform-specific navigation anyway. If you prefer minimal abstractions, go with separate navigation implementations but ensure to document routes clearly to keep web and mobile in sync.

## Animations and Transitions (Web & Mobile)

Animations enhance UX, but we need solutions that work in both React (web) and React Native. **Framer Motion** is a popular library on web, and there are analogous libraries for React Native.

* **Framer Motion (Web):** On the Next.js side, Framer Motion provides a declarative way to animate React components (e.g., fading in modals, smooth page transitions). It’s well-suited for the landing/marketing pages and even parts of the app (graphs, lists) on web. However, Framer Motion does not run on React Native directly (it’s built for DOM and SVG). For web-specific interactions, it’s a top choice due to its simplicity and power.

* **Moti (Cross-Platform):** [**Moti**](https://moti.fyi) is essentially the React Native version of Framer Motion, created by the Solito author. It follows the motto “write once, animate anywhere,” meaning you can reuse animation code on native and web (via RN Web). Moti is built on **React Native Reanimated** under the hood, which means animations run at 60 FPS on the UI thread for smooth performance. The API of Moti (components like `<MotiView>` and the `animate` props) will feel familiar to those who know Framer Motion. If we implement animations in shared components (inside `app-core/ui` or `modules`), we can use Moti so that those animations work on the Expo app. And thanks to React Native Web, if the Next.js app renders those same components, the Reanimated library can drive the animations on web as well (Reanimated 3 added better web support). In practice, this means a button press animation or a loading spinner component could be written once with Moti.

* **Reanimated & Gesture:** For more advanced interactions on mobile (and possibly web via RN Web), **React Native Reanimated** and **React Native Gesture Handler** can be used. For example, complex gesture-driven animations (like swipeable cards or pull-to-refresh) can be implemented in the shared code using those libraries. On web, Reanimated can handle some of these, or we might provide a web fallback (or use Framer Motion for an equivalent web gesture animation).

* **Framer Motion vs Moti separation:** We can also choose to use Framer Motion on the Next.js app where we are definitely in a web context (like page transition animations or hovering effects on the marketing site) and use Moti in the Expo app for screen transitions or component animations. They won’t conflict as they run in different environments. This does mean duplicating some animation logic if we want the exact same effect both on web and mobile, but we might accept slight differences per platform (web might have more intricate hover animations, mobile might have touch feedback animations).

* **Alternative Libraries:** Another cross-platform option is **React Native Animatable** or simply the Animated API built into React Native, but these are less powerful than Moti/Reanimated. Also, **Lottie** can be used for complex pre-made animations on both web and mobile (via lottie-web and lottie-react-native), which is great for things like animated illustrations.

**Recommendation:** Use **Moti for shared UI component animations**, so components in `app-core/ui` or screens in `app-core/modules` can have built-in animations that work in Expo and in Next (via RN web). For purely web pages (especially in the landing/marketing site where we might not use RN Web components), use **Framer Motion** to implement those rich animations. The APIs are similar enough that the mental model carries over. Document in the code where a certain animation is not cross-platform. Over time, you might consolidate around Moti if it covers all needs, since it indeed enables writing once and running anywhere for animations.

## Implementing Scheduled Jobs (Reminders & Cron)

The SaaS product likely needs to send reminders or perform scheduled tasks (e.g., daily summary emails, push notifications for upcoming deadlines, data backups). We have a few options for scheduling in this architecture:

* **Supabase Cron (pg\_cron):** Supabase introduced **Supabase Cron**, which is built on the Postgres `pg_cron` extension. This allows you to schedule SQL queries or function calls directly in the database on a cron schedule. For example, you could write a Postgres function to send a reminder (inserting a row into a notifications table, or calling an external webhook via `http_request` function) and schedule it to run every day at 9am. The advantage is that scheduling is managed inside Supabase – no separate server needed, and the job will run as long as the database is up. Supabase Cron integrates with Supabase’s dashboard, so you can monitor job runs. It’s a good option for tasks that involve database work (cleanups, periodic updates, etc.) or for triggering Supabase Edge Functions (serverless functions in Supabase) on a schedule. For instance, a reminder email could be handled by a Supabase Edge Function that sends emails, and pg\_cron triggers that function at the right time.

* **Vercel Cron for Next.js:** Since we are deploying the Next.js app to Vercel, we can use **Vercel’s Cron Jobs** feature for serverless functions. We could create an API route (or Edge Function) in Next.js (e.g., `pages/api/send-reminders.ts` or an App Router route handler) that sends out reminders (perhaps by calling Supabase or an email service). Then configure Vercel with a cron schedule (in `vercel.json`) to hit that endpoint on schedule. Vercel will ensure the function runs at that time, without us running a separate server. This approach is nice because all our code remains in the monorepo (no external scheduler), but note that Vercel cron jobs only run in production deployments and have some limitations (they rely on Vercel's infrastructure with EventBridge).

* **External Schedulers:** Alternatively, external services like **Upstash QStash**, **Cronhub**, or **GitHub Actions cron** could be used. Upstash QStash, for example, can send an HTTP request to a given URL at scheduled times, which could target our Next.js API route or even a Supabase function endpoint. This decouples scheduling from our infra and can be very reliable. Using GitHub Actions on a schedule to call a function is another lightweight approach (though GitHub Actions is more for CI, not ideal for app features).

* **In-App Scheduling:** For some reminder types (like local reminders or push notifications on mobile), the scheduling might happen on the client. For example, the Expo app could schedule local notifications using `Notifications.scheduleNotificationAsync` (if the reminder is specific to the user’s device and doesn’t need server verification). But for anything that requires central coordination (like emailing a user who may not have the app open), a backend cron is needed.

**Which to choose?** If the reminders involve database records (like sending a reminder email for all users who have item X due today), **Supabase Cron** is a great choice because it keeps the logic close to the data. You could write a SQL query or a small function that finds due items and inserts into a `notifications` table or sends emails. Supabase Cron will run it reliably and you can manage it in SQL or Supabase’s UI. On the other hand, if your reminder logic is better expressed in TypeScript (especially if reusing code from the app), using **Vercel Cron with a Next.js API route** might be easier – you can import your libraries, send emails via Node, etc., in an environment you're familiar with.

Since this boilerplate already includes Supabase, using **Supabase Cron** is an elegant solution for many scheduled tasks. It avoids introducing another third-party and leverages Postgres’s proven scheduler. However, one limitation is that heavy jobs could impact your DB performance. For very computational or external API heavy tasks, calling a serverless function might be better. Supabase Cron can trigger **Edge Functions** (which are essentially Deno functions) on schedule, providing the best of both worlds: scheduling in the DB, but executing in a separate runtime.

**Example:** To implement daily email reminders, you might create a Supabase Edge Function (`sendReminders`) that queries for all pending reminders and sends emails (using an email service API). Then schedule it with Supabase Cron for, say, 8:00 UTC daily. In contrast, if using Vercel, you’d create a Next.js API route for sending reminders and schedule it via Vercel Cron. Both achieve similar results. It’s acceptable to mix approaches if needed (Supabase Cron for DB-centric tasks, Vercel Cron for tasks that need a lot of Node library support).

## Deployment and CI/CD Strategy

We aim for a streamlined deployment process for both the web app and mobile apps, with automated builds, versioning, and multi-platform release processes.

### Web Deployment (Next.js on Vercel)

The Next.js application will be deployed on **Vercel**, which is the platform developed by the creators of Next.js and offers first-class support for it. Key points for Vercel deployment:

* **Continuous Deployment:** We connect the Git repository to Vercel. On every push to the main (or production) branch, Vercel will build and deploy the Next.js app. This includes building static pages, setting up serverless functions for any dynamic routes or API routes, and enabling edge functions as needed. Vercel’s integration means we get preview deployments for each PR (useful even for a solo dev to test features) and production deployment on merge.

* **Performance and Scalability:** Vercel automatically handles scaling the web app. If our SaaS grows, Vercel will scale out function invocations and static content via their CDN. We also benefit from features like ISR (Incremental Static Regeneration) and Edge Caching with zero configuration.

* **Environment Management:** We store environment variables (Supabase URL, etc.) in Vercel’s dashboard (which can have separate values for Preview and Production). This keeps secrets out of the repo. The `config` module in app-core will pick up these from `process.env` at build/runtime.

* **Custom Domain:** We can set up a custom domain for the SaaS (e.g., `mysaas.com`) on Vercel easily, including auto SSL.

* **Cron Jobs on Vercel:** As discussed, if we use Vercel’s cron, the configuration lives in `vercel.json` which is deployed alongside. This ensures scheduled tasks are active on production deploy.

In summary, **Vercel provides an optimized CI/CD for Next.js** – push your code and it’s live in seconds. This is ideal for a solo developer to automate deployments without maintaining infrastructure.

### Mobile Deployment (Expo EAS for iOS & Android)

For the Expo React Native app, we use **Expo Application Services (EAS)** to handle building and releasing the app:

* **EAS Build:** Expo’s cloud build service can compile the iOS and Android binaries. We configure build profiles in `eas.json` (e.g., `development`, `staging`, `production` with appropriate app identifiers). With a single command or via CI, we trigger EAS to build an `.apk/.aab` for Android and an `.ipa` for iOS. EAS manages credentials (keystore, provisioning profiles) securely, which is very helpful for solo developers not wanting to fiddle with Xcode and Android Studio.

* **EAS Submit:** After a successful build, EAS can automatically submit the binary to the Google Play Console and Apple App Store Connect. This can be part of the CI pipeline for release builds. We still go in and release to production on the stores manually (or set up staged releases as needed), but the uploading is automated.

* **Over-The-Air Updates:** One powerful Expo feature is **EAS Update**, which allows deploying JS/CSS asset updates to apps instantly (within the rules of app stores). We can set up a channel for production and push updates for minor changes that don’t require a rebuild (e.g., quick bug fix in UI). This is optional but useful; it means some changes can reach users without requiring them to download a new version from the store. It’s integrated into EAS workflows.

* **Versioning:** The `config` or `app.json` will have an app version. We should update the version for each store release (EAS can bump it automatically). We might use semantic versioning. Also, we maintain consistency that features in web and mobile are released together if needed (though mobile release may lag due to app store review).

* **CI/CD Integration:** We can connect the repository with EAS in multiple ways:

  * Use the **Expo GitHub Action** or Expo’s official GitHub App to trigger builds on push. For example, every time we tag a commit with `release-mobile-v1.x`, a GitHub Action can run `eas build --profile production --auto-submit`.
  * Alternatively, run EAS CLI on a CI service (like GitHub Actions or CircleCI) to kick off builds. Expo provides a managed GitHub App that simplifies this (just push and it builds, similar to Vercel’s style).

  By automating this, we avoid manual steps. After a successful build and submit, we get an email from Apple/Google when the app is processed or live.

* **Alternative CI:** If not using EAS, alternatives include using **fastlane** scripts with your own CI to build and upload, or services like **Microsoft App Center**, **Bitrise**, etc., which can detect a React Native project and build it. However, for an Expo-managed app, EAS is the path of least resistance because it knows the environment needed. App Center would require ejecting or at least handling Expo-specific things, which is more work.

Below is a summary comparison for deployment tools:

| Platform          | Preferred Deployment                | Features & Benefits                                                                                                                                                                                                                                   | Alternatives                                                                                                                                                                                                                                                             |
| ----------------- | ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Web (Next.js)** | **Vercel** (CI/CD integrated)       | - Automatic builds on git push (with preview and prod env).<br>- Serverless functions, Edge Functions support out-of-box.<br>- Global CDN for static assets and SSR responses.<br>- Easy env var management and custom domain setup.                  | - Self-host (Docker or Node server) on AWS, etc. (requires DevOps work).<br>- Netlify (supports Next.js, but Vercel often more seamless for latest Next features).<br>- AWS Amplify or CloudFront + Lambda for Next (more complex).                                      |
| **Mobile (Expo)** | **EAS (Expo Application Services)** | - Cloud builds for iOS/Android with managed credentials.<br>- One-click (or CI-driven) submissions to app stores.<br>- Supports OTA updates for JS assets (no rebuild needed for minor updates).<br>- Designed for Expo/React Native, minimal config. | - Manual build with Xcode/Android Studio (time-consuming and error-prone).<br>- Fastlane scripts on GitHub Actions (powerful but requires maintenance).<br>- Third-party CI like App Center or Bitrise (can build RN apps; good if using bare RN or custom native code). |

Using Vercel and EAS together covers our “full stack” deployment with minimal overhead, allowing a solo developer to achieve automated deployments across web and mobile. All critical builds and releases can be managed via Git, which means we can version control our deployment pipeline too (through config files like `vercel.json` and `eas.json`).

### Automatic Deployment Workflow

To tie it together, we can set up a **release workflow** such that:

1. Merging a PR to `main` triggers Vercel to deploy the web app (immediately live if tests pass).
2. For mobile, we might want to only trigger a build for certain tags or manual triggers (since releasing to app stores every commit is not practical). For example, we create a Git tag `v1.0.0` when ready to release mobile. A GitHub Action watches for tags and triggers `eas build --profile production --auto-submit` for both platforms. This produces build artifacts and sends to stores.
3. We monitor store releases and once available, announce or enable any needed server features (in case the mobile app depends on a backend change).
4. EAS Update can be integrated to push minor fixes outside of this cycle, but we must follow app store rules (no major changes via OTA).

By documenting this process in a README for the project, we ensure that when more team members join, they can follow the established pattern to release new versions.

## End-to-End Testing (Web & Mobile)

To ensure the application works as expected on both platforms, we set up end-to-end (E2E) testing pipelines for web and mobile. E2E tests simulate user interactions and validate the full stack (from UI down to backend).

### Web E2E Testing with Playwright

For the Next.js web app, **Playwright** is an excellent choice for E2E testing. Playwright can automate Chromium, Firefox, and WebKit browsers and is known for its reliability and speed. We can write tests in TypeScript that navigate through our Next.js application in a headless browser, click buttons, fill forms, and verify results on the page (text, URL changes, database effects via the UI, etc.).

* **Setup:** We add Playwright to the dev dependencies and perhaps use the Next.js testing guide for Playwright. We might create a test configuration that before each test, reseeds the database (to a known state) or uses a testing database. Supabase can provide a separate project or schema for test runs if needed.

* **Tests:** For example, a test could register a new user on the landing page, then log in, then create an item in the dashboard, and verify it appears. Playwright’s API allows waiting for elements, taking screenshots, etc. This ensures our critical paths (signup, usage, logout) work.

* **Running:** These tests can be run in CI (GitHub Actions) on pushes. Playwright has integrations to GitHub Actions that even upload videos of failures. This gives confidence that a new commit didn’t break core user flows on web.

* **Alternative:** One could also use Cypress for web E2E, which is another popular tool. However, Playwright has the advantage of supporting multiple browsers and tends to have less flakiness in our experience. It’s also maintained by Microsoft and keeps up with modern browser features.

### Mobile E2E Testing with Detox or Maestro

For the React Native app, traditional web E2E tools don’t apply (since it’s not running in a browser). We have two prominent options: **Detox** and **Maestro**.

* **Detox:** Detox is a gray-box testing framework for React Native (by Wix) that controls a simulator/emulator to interact with the app. It hooks into the app runtime to know when the app is idle, reducing flakiness. Tests are written in JavaScript/TypeScript (often using Jest) and they call Detox APIs to simulate taps, type text, etc. It’s quite powerful and has been around for a while. We can integrate Detox in our Expo app by configuring the native projects (Expo Development Builds are needed, since in a pure managed workflow Detox can’t instrument the app; EAS can build a dev client for testing). A sample Detox test might launch the app, tap the "Login" button, fill credentials, and verify that a certain screen appears. Detox tests run on CI with some setup (e.g., Android emulators or iOS simulators on a macOS runner).

* **Maestro:** Maestro is a newer tool from mobile.dev that takes a different approach – it allows writing **declarative test flows in YAML** and focuses on simplicity and resilience. You describe a flow (like "launch app", "tap 'Login'", "input text '[user@example.com](mailto:user@example.com)'", etc.) in a YAML file. Maestro runs this against a real app similarly by instrumentation, but it handles waiting and retries automatically. It’s designed to be less flaky out-of-the-box, with no need for manual `await` or sleeps. It’s also cross-platform (same script works on iOS and Android). For a solo developer, Maestro can be very quick to get started: you don’t need to write a test runner or code, just write steps. And you can use the Maestro Studio to visually pick elements. We can include the Maestro CLI as a dev dependency and create a suite of YAML test flows for key scenarios.

* **Comparing Detox vs Maestro:** Detox offers more programmatic control (you can write complex logic in tests, conditionals, etc., since it’s code). Maestro is more high-level and might be easier to maintain for straightforward flows. Detox requires a bit more setup (especially connecting to the React Native runtime); Maestro is somewhat simpler (just need the app binary and device). In terms of CI, both can be integrated. Maestro being a single binary can be run in CI easily; Detox might need the environment set up carefully. **Flakiness:** Maestro’s design prioritizes flakiness reduction (it waits for views to appear, handles partial matches, etc.), whereas with Detox you might need to add proper expectations to avoid flakiness (though Detox also waits for idle state). The table below outlines these differences:

| Tool        | Test Definition   | Pros                                                                                                                                                                                                                              | Cons                                                                                                                                                                                                                                                                 |
| ----------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Detox**   | Code (Jest, etc.) | - Write tests in JS/TS (flexible logic).<br>- Can leverage app internals (via Detox hooks) for fine control.<br>- Strong community usage, proven in many RN apps.                                                                 | - Setup is non-trivial (native build, config).<br>- Tests can be brittle if not written carefully (must await conditions).<br>- Only for mobile (cannot test web).                                                                                                   |
| **Maestro** | YAML scripts      | - Very simple syntax – quick to write and understand (even for non-devs).<br>- Built-in tolerance for delays/flakiness (auto-retries).<br>- Single tool for iOS & Android with same script.<br>- Can be run without writing code. | - Less flexibility (harder to do complex conditional logic or reuse code).<br>- Newer project (still evolving, fewer community examples than Detox).<br>- Requires maintaining separate YAML files rather than code (could be a pro or con depending on preference). |

Given our scenario, if our goal is to have **E2E tests that cover both web and mobile flows**, we might use **both**: Playwright for web, and Maestro for mobile, for example. Maestro even has the ability to automate a simulator and a real device; it could possibly automate some flows on a mobile web browser too, but primarily it’s for apps. Detox and Maestro are not mutually exclusive; but we’d likely pick one for mobile to avoid redundant effort. **For a solo dev**, Maestro might get you up and running faster with high-level tests. Detox might be something to introduce later if you need more complex checks.

We will also integrate these tests in CI. For instance, use GitHub Actions macOS runners to run iOS simulator tests, and Android on Linux runners (there are community actions for setting up AVD). For Maestro, the action would download the Maestro CLI and run the YAML flows on both platforms.

### Additional Testing

Beyond E2E, we should set up unit and integration tests for critical logic:

* **Unit Tests:** Use Jest (already common in RN and supported in Next) for pure functions, utilities in `app-core` (for example, test that our data transformation or validation functions work correctly).
* **Integration Tests:** Perhaps test some React components in isolation with React Testing Library (for web components) and a similar library for RN (like @testing-library/react-native) to ensure our shared UI behaves expectedly given props. This is less crucial than E2E for an MVP, but good for a growing codebase.

Having a solid test suite will also make it easier for others to join the project, as they can run tests to verify they didn’t break anything and see examples of how the app is supposed to behave.

## TypeScript Monorepo Best Practices

The entire codebase is **TypeScript-first**, meaning we use TypeScript for all modules (Next.js, React Native, Node scripts, etc.) to ensure type safety and developer efficiency.

**Monorepo TypeScript Configuration:**

* We can set up a root `tsconfig.json` that provides base config, and each package (and app) can extend it. In a monorepo, using **TypeScript Project References** is recommended to improve build times and enforce boundaries. Each package (like `app-core`) can have its own `tsconfig.json` where it references none (for standalone build), and the Next.js and Expo apps each have tsconfig that **references** the core package. This allows TypeScript to understand the cross-package imports without fuss and to type-check across project boundaries. It also helps tools like VSCode jump between packages easily.

* We will use path aliases to simplify imports. For example, `@core/ui` could point to `packages/app-core/ui/src` (if we have a src folder) or directly to that folder. This way, in code we import from `@core/ui` instead of relative paths. We then ensure our bundlers (Next and Metro for RN) understand these aliases (Next.js can use the `compilerOptions.paths` from tsconfig automatically, and Expo Metro can be configured with babel-plugin-module-resolver to mirror them).

**Consistent Types:**

* Types that are shared (like the shapes of data, or an `User` interface) live in the core packages (e.g., in `app-core/database` for DB types, or a separate `app-core/types` if it makes sense). This avoids duplicating interface definitions. Both Next and Expo can import these. For example, if Supabase generates types for tables (via `supabase/types.ts`), those reside in `database/` and are used everywhere. If we use a library like Zod for schema validation, we can define zod schemas in one place and derive TypeScript types from them, ensuring the web and mobile validate data consistently.

* We also use TypeScript for configuration files where possible (Next.js allows `next.config.mjs` but can be TS with a trick, Expo allows `app.config.ts`, etc.), so we get type checking on config as well.

**Linting and Formatting:** We will maintain a single ESLint configuration at the root that covers both TS and maybe React/React Native specifics. The config can extend recommended settings and include rules to prevent common mistakes (like using React Native’s deprecated features, or Next.js specific lint rules). We ensure Prettier or an equivalent is set up to format code on save/commit, so the coding style remains consistent across the monorepo. These tools also help new contributors by automatically flagging issues.

**Documentation and Onboarding:** We should include a **README** or docs for how to set up the dev environment (e.g., running `yarn dev` to start both Next and Expo concurrently, possibly with a tool like `turbo` or `nx` to run tasks in parallel). Instructions for how to add a new package or module, how to run tests, how to build, etc., should be included. Using TypeScript helps because newcomers can read function signatures and understand usage quickly, and catch mistakes at compile time.

**Ensuring type safety across network boundaries:**
If the web and mobile communicate with the backend (Supabase, or perhaps a custom API if added), we use generated types or SDKs so that there’s a single source of truth. For example, if we had a REST API, we might use OpenAPI and generate a client, but since we are using Supabase, the Supabase client already provides types if configured properly. Similarly, for any custom Edge Functions in Supabase, we define input/output schemas and reuse those types. This minimizes runtime errors.

**Example Type Safety:** If we have a module for “Tasks” with a function `completeTask(taskId: string)`, it will have a defined input and output type (maybe it returns a `Task` object updated). The Next.js API route for completing a task and the Expo screen that calls it (via Supabase or API) both use the same `completeTask` function from `app-core/modules/tasks`. Thus, a change in the task schema will throw a type error in both places if not adjusted, preventing inconsistent implementations.

Lastly, we treat **TypeScript as a self-documenting tool** – by reading the types and interfaces in `app-core`, a new developer can grasp what data is available and what functions do. We will strive to use descriptive names and JSDoc comments for functions, which editors can show on hover.

## Best Practices and Future-Proofing

In building this boilerplate, we adhere to industry best practices to ensure the project remains maintainable and scalable:

* **Separation of Concerns:** The division into `applications` and `packages` (core) is an application of separation of concerns at a high level. Within code, we further separate presentational components from business logic, use controllers or hooks to handle complex logic, and keep our code DRY (Don’t Repeat Yourself). This will make it easier for others to know where to add new code or find existing logic.

* **Scalability:** While initially for a solopreneur, the stack choices (Next.js, React Native, Supabase) are all scalable technologies used in production by many companies. This means down the line, if the product grows, a new team would not need to re-architect from scratch. We leverage serverless and managed services for scalability (Vercel, Supabase scaling, etc.).

* **Onboarding Developers:** We maintain good documentation within the repo. Each module in `app-core/modules` can have a README.md describing the module’s purpose and how to use it. We also keep an architectural README (possibly this document) in the root of the project, so any new developer or collaborator can quickly understand the structure and start contributing. Code is written in a clear style and we avoid overly clever abstractions that are hard to pick up.

* **Best-of-Breed Tools:** We chose tools that are widely adopted: Next.js for SSR web, React Native for mobile, Expo for ease of use, Supabase for backend (which uses Postgres, a rock-solid DB), Playwright/Detox for testing, etc. This means a new developer likely has familiarity with many of these, or there are plenty of learning resources available. For example, if someone hasn’t used Solito, they still understand React Navigation and Next.js separately, and can learn Solito quickly with its documentation.

* **Security:** We follow security best practices from day one. Examples:

  * All API calls require proper auth (we use Supabase’s RLS policies and checks in our Next.js API routes to ensure data privacy).
  * We store sensitive info securely (no secrets in git, use env vars; on mobile, store tokens in secure storage).
  * We use HTTPS everywhere (Vercel and the native apps connecting to Supabase via SSL).
  * Dependabot or similar can be enabled to keep libraries up to date with security patches.

* **Performance:** Next.js gives us good web performance out of the box (we will use image optimization, code-splitting via the App Router, etc.). The Expo app, thanks to React Native, runs at native speeds and we use performance best practices there too (avoid heavy computations on the JS thread, use FlatList for long lists, etc.). We also plan for offline or low-network scenarios (perhaps caching certain queries or using Expo’s offline storage). By addressing performance early (profiling critical paths), we ensure the app can handle a growing user base.

* **Future Extensibility:** Because we have a modular setup, adding a new platform (e.g., a desktop Electron app, or a future “Expo for Web” progressive web app) could reuse much of the `app-core`. The TypeScript monorepo could even be extended with a `/applications/desktop` or similar, without upheaval. Similarly, if one day the project needs a dedicated backend service beyond Supabase (say a Node microservice for some specialized task), we can integrate it by leveraging our existing libraries (or replacing parts of `app-core/database` to point to a new service). The key is our architecture is not tightly coupled to one vendor – Supabase is used because it’s convenient, but thanks to repository patterns in `database` and such, we could replace the data layer with minimal changes to the rest of the app.

In conclusion, this SaaS boilerplate provides a robust starting point with a **Next.js + Expo** dual application setup, shared TypeScript code, and thoughtful solutions to cross-platform challenges. By following this documentation, a developer can understand the moving parts and rationale, enabling confident development and the ability to leverage the full power of this modern stack.

**References:**

* Next.js Route Groups Documentation
* Solito (React Native + Next.js navigation) – Official Introduction
* Discussion on shadcn/ui (Radix + Tailwind) compatibility with React Native
* Moti (Cross-platform animation library) – “React Native version of Framer Motion”
* Supabase Cron announcement – scheduling jobs inside Postgres
* Vercel Cron Jobs guide – scheduling serverless functions via vercel.json
* Detox (React Native E2E) – Official GitHub description
* Maestro (Mobile UI testing) – Key features (YAML flows, flakiness tolerance)
