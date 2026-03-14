# Cake Shop Website — Full Architecture Document

> **Tech Stack:** Next.js 15 (App Router) | MongoDB (Mongoose) | NextAuth.js
> **Goal:** A premium, creative cake shop website with a fully manageable admin panel

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Folder Structure](#2-folder-structure)
3. [Technology Stack & Libraries](#3-technology-stack--libraries)
4. [Architecture Diagram](#4-architecture-diagram)
5. [Routing Strategy](#5-routing-strategy)
6. [Data Models (MongoDB / Mongoose)](#6-data-models-mongodb--mongoose)
7. [Authentication & Authorization](#7-authentication--authorization)
8. [Admin Panel — Content Management](#8-admin-panel--content-management)
9. [Public Website — Pages & Sections](#9-public-website--pages--sections)
10. [API Design (Route Handlers)](#10-api-design-route-handlers)
11. [Image & Media Management](#11-image--media-management)
12. [UI/UX Design Guidelines](#12-uiux-design-guidelines)
13. [Performance & SEO Strategy](#13-performance--seo-strategy)
14. [Deployment Strategy](#14-deployment-strategy)
15. [Future Feature Suggestions](#15-future-feature-suggestions)

---

## 1. Project Overview

A **premium cake shop website** that serves two audiences:

| Audience | Experience |
|----------|-----------|
| **Customers** | Browse cakes, view about/story, see latest updates, explore favorites, find shop location, read custom pages |
| **Admins** | Full CMS to manage every section — hero, about, favorites, updates, header, footer, custom pages |

### Core Principles

- **Content-first:** Every visible section is admin-manageable — zero hardcoded content
- **Performance:** Server Components by default, Client Components only where interactivity is needed
- **Security:** Admin routes fully protected, role-based access
- **Premium Feel:** Smooth animations, elegant typography, rich imagery

---

## 2. Folder Structure

```
cake-shop/
├── app/
│   ├── (public)/                    # Public route group
│   │   ├── layout.tsx               # Public layout (header + footer from DB)
│   │   ├── page.tsx                 # Home page (Hero, Favorites, Updates, Visit)
│   │   ├── about/
│   │   │   └── page.tsx             # About / Our Story page
│   │   ├── menu/
│   │   │   └── page.tsx             # Full cake menu / catalog
│   │   ├── gallery/
│   │   │   └── page.tsx             # Cake gallery
│   │   ├── contact/
│   │   │   └── page.tsx             # Contact & location
│   │   └── [slug]/
│   │       └── page.tsx             # Dynamic custom pages
│   │
│   ├── (admin)/                     # Admin route group
│   │   ├── layout.tsx               # Admin layout (sidebar + topbar)
│   │   ├── admin/
│   │   │   ├── page.tsx             # Admin dashboard
│   │   │   ├── header/
│   │   │   │   └── page.tsx         # Manage header (logo, nav links)
│   │   │   ├── footer/
│   │   │   │   └── page.tsx         # Manage footer (links, social, text)
│   │   │   ├── hero/
│   │   │   │   └── page.tsx         # Manage hero section
│   │   │   ├── about/
│   │   │   │   └── page.tsx         # Manage about section
│   │   │   ├── favorites/
│   │   │   │   └── page.tsx         # Manage "Our Favorites" items
│   │   │   ├── updates/
│   │   │   │   └── page.tsx         # Manage latest updates / blog
│   │   │   ├── visit/
│   │   │   │   └── page.tsx         # Manage "Visit the Cake Shop" section
│   │   │   ├── pages/
│   │   │   │   ├── page.tsx         # List all custom pages
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx     # Create new custom page
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx     # Edit custom page
│   │   │   ├── media/
│   │   │   │   └── page.tsx         # Media library
│   │   │   └── settings/
│   │   │       └── page.tsx         # Site-wide settings (SEO, colors)
│   │   └── middleware.ts            # (handled at root level)
│   │
│   ├── api/
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   │       └── route.ts         # NextAuth API route
│   │   ├── admin/
│   │   │   ├── header/
│   │   │   │   └── route.ts         # CRUD for header
│   │   │   ├── footer/
│   │   │   │   └── route.ts         # CRUD for footer
│   │   │   ├── hero/
│   │   │   │   └── route.ts         # CRUD for hero
│   │   │   ├── about/
│   │   │   │   └── route.ts         # CRUD for about
│   │   │   ├── favorites/
│   │   │   │   └── route.ts         # CRUD for favorites
│   │   │   ├── updates/
│   │   │   │   └── route.ts         # CRUD for updates
│   │   │   ├── visit/
│   │   │   │   └── route.ts         # CRUD for visit section
│   │   │   ├── pages/
│   │   │   │   └── route.ts         # CRUD for custom pages
│   │   │   ├── media/
│   │   │   │   └── route.ts         # Upload / delete media
│   │   │   └── settings/
│   │   │       └── route.ts         # Site settings
│   │   └── public/
│   │       ├── content/
│   │       │   └── route.ts         # Fetch all public content
│   │       └── pages/
│   │           └── [slug]/
│   │               └── route.ts     # Fetch single custom page
│   │
│   ├── layout.tsx                   # Root layout
│   ├── loading.tsx                  # Global loading UI
│   ├── not-found.tsx                # 404 page
│   └── error.tsx                    # Error boundary
│
├── components/
│   ├── public/                      # Public-facing components
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── HeroSection.tsx
│   │   ├── AboutSection.tsx
│   │   ├── FavoritesSection.tsx
│   │   ├── LatestUpdates.tsx
│   │   ├── VisitSection.tsx
│   │   ├── CakeCard.tsx
│   │   └── AnimatedSection.tsx      # Scroll-triggered animations
│   │
│   ├── admin/                       # Admin panel components
│   │   ├── Sidebar.tsx
│   │   ├── Topbar.tsx
│   │   ├── RichTextEditor.tsx       # WYSIWYG editor
│   │   ├── ImageUploader.tsx
│   │   ├── MediaPicker.tsx
│   │   ├── ContentForm.tsx          # Reusable form component
│   │   ├── DataTable.tsx
│   │   └── DashboardStats.tsx
│   │
│   └── ui/                          # Shared UI primitives
│       ├── Button.tsx
│       ├── Input.tsx
│       ├── Modal.tsx
│       ├── Toast.tsx
│       ├── Skeleton.tsx
│       └── Badge.tsx
│
├── lib/
│   ├── db.ts                        # MongoDB connection (singleton)
│   ├── auth.ts                      # NextAuth configuration
│   ├── models/                      # Mongoose models
│   │   ├── User.ts
│   │   ├── Header.ts
│   │   ├── Footer.ts
│   │   ├── Hero.ts
│   │   ├── About.ts
│   │   ├── Favorite.ts
│   │   ├── Update.ts
│   │   ├── Visit.ts
│   │   ├── CustomPage.ts
│   │   ├── Media.ts
│   │   └── SiteSettings.ts
│   ├── actions/                     # Server Actions
│   │   ├── header.actions.ts
│   │   ├── footer.actions.ts
│   │   ├── hero.actions.ts
│   │   ├── about.actions.ts
│   │   ├── favorites.actions.ts
│   │   ├── updates.actions.ts
│   │   ├── visit.actions.ts
│   │   ├── pages.actions.ts
│   │   └── media.actions.ts
│   ├── validations/                 # Zod schemas
│   │   ├── header.schema.ts
│   │   ├── hero.schema.ts
│   │   └── ...
│   └── utils.ts                     # Helper functions
│
├── hooks/                           # Custom React hooks
│   ├── useToast.ts
│   ├── useMediaUpload.ts
│   └── useDebounce.ts
│
├── styles/
│   └── globals.css                  # Tailwind + custom CSS variables
│
├── public/
│   ├── fonts/                       # Premium custom fonts
│   └── images/                      # Static fallback images
│
├── middleware.ts                     # Root middleware (auth guard)
├── next.config.ts
├── tailwind.config.ts
├── .env.local
├── .env.example
└── package.json
```

---

## 3. Technology Stack & Libraries

### Core

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Framework | **Next.js 15** (App Router) | SSR, SSG, API routes, Server Components |
| Database | **MongoDB Atlas** | Document-based storage for CMS content |
| ODM | **Mongoose 8** | Schema validation, queries, middleware |
| Auth | **NextAuth.js v5** (Auth.js) | Admin authentication with credentials/OAuth |
| Styling | **Tailwind CSS 4** | Utility-first styling |
| Animations | **Framer Motion** | Scroll animations, page transitions |

### Admin Panel

| Library | Purpose |
|---------|---------|
| **Tiptap** or **React Quill** | Rich text editor for content |
| **React Hook Form** | Performant form handling |
| **Zod** | Schema validation (shared client/server) |
| **Uploadthing** or **Cloudinary** | Image/media uploads |
| **Sonner** | Toast notifications |
| **Lucide React** | Icon library |

### UI Enhancement

| Library | Purpose |
|---------|---------|
| **Shadcn/ui** | Pre-built accessible components |
| **Embla Carousel** | Smooth carousels for cakes/gallery |
| **next/image** | Optimized image loading |
| **next-themes** | Light/dark mode support |

---

## 4. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                         │
├────────────────────────────┬────────────────────────────────────┤
│     Public Website         │         Admin Panel                │
│  (Server Components +      │   (Client Components +            │
│   Client interactivity)    │    Server Actions)                 │
└────────────┬───────────────┴──────────────┬─────────────────────┘
             │                              │
             │         Next.js 15           │
             │        App Router            │
             │                              │
┌────────────▼──────────────────────────────▼─────────────────────┐
│                                                                  │
│                     MIDDLEWARE LAYER                              │
│           (Authentication + Route Protection)                    │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    SERVER LAYER                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐      │
│  │ Server       │  │ Route        │  │ Server            │      │
│  │ Components   │  │ Handlers     │  │ Actions           │      │
│  │ (Data fetch) │  │ (REST API)   │  │ (Form mutations)  │      │
│  └──────┬───────┘  └──────┬───────┘  └────────┬──────────┘      │
│         │                 │                    │                  │
│         └─────────────────┼────────────────────┘                 │
│                           │                                      │
│  ┌────────────────────────▼─────────────────────────────────┐    │
│  │              DATA ACCESS LAYER                            │    │
│  │  Mongoose Models + Validation (Zod) + Business Logic      │    │
│  └────────────────────────┬─────────────────────────────────┘    │
│                           │                                      │
└───────────────────────────┼──────────────────────────────────────┘
                            │
               ┌────────────▼────────────┐
               │     MongoDB Atlas       │
               │  ┌────────────────────┐ │
               │  │  Collections:      │ │
               │  │  - users           │ │
               │  │  - headers         │ │
               │  │  - footers         │ │
               │  │  - heroes          │ │
               │  │  - abouts          │ │
               │  │  - favorites       │ │
               │  │  - updates         │ │
               │  │  - visits          │ │
               │  │  - custompages     │ │
               │  │  - media           │ │
               │  │  - sitesettings    │ │
               │  └────────────────────┘ │
               └─────────────────────────┘

               ┌─────────────────────────┐
               │   Cloudinary / S3       │
               │   (Image Storage)       │
               └─────────────────────────┘
```

---

## 5. Routing Strategy

### Public Routes (No Auth Required)

| Route | Page | Data Source |
|-------|------|------------|
| `/` | Home | Hero, Favorites, Updates, Visit |
| `/about` | About / Our Story | About collection |
| `/menu` | Cake Menu / Catalog | Favorites + categories |
| `/gallery` | Photo Gallery | Media collection |
| `/contact` | Contact & Location | Visit collection |
| `/[slug]` | Custom Pages | CustomPages collection |

### Admin Routes (Auth Required — Admin Role Only)

| Route | Page | Manages |
|-------|------|---------|
| `/admin` | Dashboard | Overview stats |
| `/admin/header` | Header Manager | Logo, navigation links |
| `/admin/footer` | Footer Manager | Footer links, social, copyright |
| `/admin/hero` | Hero Manager | Hero images, text, CTA |
| `/admin/about` | About Manager | Story, team, images |
| `/admin/favorites` | Favorites Manager | Featured cakes CRUD |
| `/admin/updates` | Updates Manager | Blog / news CRUD |
| `/admin/visit` | Visit Manager | Address, hours, map embed |
| `/admin/pages` | Custom Pages | Create/edit/delete pages |
| `/admin/pages/new` | New Custom Page | Rich text page builder |
| `/admin/pages/[id]` | Edit Custom Page | Edit existing page |
| `/admin/media` | Media Library | Upload, browse, delete images |
| `/admin/settings` | Site Settings | SEO defaults, theme colors |

### Auth Routes

| Route | Page |
|-------|------|
| `/admin/login` | Admin login page |

---

## 6. Data Models (MongoDB / Mongoose)

### User

```typescript
// lib/models/User.ts
{
  name:          String,          // "Admin Name"
  email:         String (unique), // "admin@cakeshop.com"
  password:      String,          // bcrypt hashed
  role:          String,          // "admin" | "superadmin"
  avatar:        String,          // URL
  isActive:      Boolean,         // default: true
  lastLogin:     Date,
  createdAt:     Date,
  updatedAt:     Date
}
```

### Header

```typescript
// lib/models/Header.ts
{
  logo: {
    imageUrl:    String,          // Logo image URL
    altText:     String,          // "Sweet Delights Bakery"
    linkTo:      String           // "/" (home)
  },
  navigation: [{
    label:       String,          // "Menu"
    href:        String,          // "/menu"
    order:       Number,          // Sort order
    isVisible:   Boolean          // Toggle visibility
  }],
  ctaButton: {
    text:        String,          // "Order Now"
    href:        String,          // "/contact"
    isVisible:   Boolean
  },
  isSticky:      Boolean,         // Sticky header toggle
  updatedAt:     Date
}
```

### Hero

```typescript
// lib/models/Hero.ts
{
  slides: [{
    title:       String,          // "Handcrafted with Love"
    subtitle:    String,          // "Premium cakes for every occasion"
    backgroundImage: String,      // URL
    ctaText:     String,          // "Explore Our Cakes"
    ctaLink:     String,          // "/menu"
    overlayOpacity: Number,       // 0.0 - 1.0
    order:       Number,
    isActive:    Boolean
  }],
  autoplaySpeed: Number,          // milliseconds (default: 5000)
  updatedAt:     Date
}
```

### About

```typescript
// lib/models/About.ts
{
  sectionTitle:  String,          // "Our Story"
  heading:       String,          // "Baking Happiness Since 2010"
  description:   String,          // Rich text / HTML
  images: [{
    url:         String,
    alt:         String,
    order:       Number
  }],
  stats: [{                       // e.g., "500+ Cakes Sold"
    label:       String,
    value:       String,
    icon:        String
  }],
  teamMembers: [{
    name:        String,
    role:        String,
    image:       String,
    bio:         String
  }],
  isVisible:     Boolean,
  updatedAt:     Date
}
```

### Favorite (Our Favorites)

```typescript
// lib/models/Favorite.ts
{
  name:          String,          // "Red Velvet Dream"
  slug:          String (unique), // "red-velvet-dream"
  description:   String,          // Short description
  price:         Number,          // 45.99
  images: [{
    url:         String,
    alt:         String
  }],
  category:      String,          // "Wedding" | "Birthday" | "Custom"
  tags:          [String],        // ["chocolate", "premium"]
  isFeatured:    Boolean,         // Show on homepage
  isAvailable:   Boolean,
  rating:        Number,          // Average rating
  order:         Number,          // Display order
  createdAt:     Date,
  updatedAt:     Date
}
```

### Update (Latest Updates / Blog)

```typescript
// lib/models/Update.ts
{
  title:         String,          // "New Summer Collection!"
  slug:          String (unique),
  excerpt:       String,          // Short preview text
  content:       String,          // Rich text / HTML
  coverImage:    String,          // URL
  author:        ObjectId (ref: User),
  category:      String,          // "News" | "Recipe" | "Event"
  tags:          [String],
  isPublished:   Boolean,
  publishedAt:   Date,
  createdAt:     Date,
  updatedAt:     Date
}
```

### Visit (Visit the Cake Shop)

```typescript
// lib/models/Visit.ts
{
  sectionTitle:  String,          // "Visit Us"
  heading:       String,          // "Come Experience the Magic"
  description:   String,
  address: {
    street:      String,
    city:        String,
    state:       String,
    zipCode:     String,
    country:     String
  },
  phone:         String,
  email:         String,
  businessHours: [{
    day:         String,          // "Monday"
    openTime:    String,          // "09:00"
    closeTime:   String,          // "18:00"
    isClosed:    Boolean
  }],
  mapEmbedUrl:   String,          // Google Maps embed URL
  images: [{
    url:         String,
    alt:         String
  }],
  isVisible:     Boolean,
  updatedAt:     Date
}
```

### Footer

```typescript
// lib/models/Footer.ts
{
  logo: {
    imageUrl:    String,
    altText:     String
  },
  description:   String,          // Short brand description
  sections: [{
    title:       String,          // "Quick Links"
    links: [{
      label:     String,
      href:      String,
      isExternal: Boolean
    }],
    order:       Number
  }],
  socialLinks: [{
    platform:    String,          // "instagram" | "facebook" | "twitter"
    url:         String,
    icon:        String
  }],
  copyrightText: String,          // "© 2026 Sweet Delights"
  newsletterEnabled: Boolean,
  updatedAt:     Date
}
```

### CustomPage

```typescript
// lib/models/CustomPage.ts
{
  title:         String,          // "Catering Services"
  slug:          String (unique), // "catering-services"
  content:       String,          // Rich text / HTML
  metaTitle:     String,          // SEO title
  metaDescription: String,        // SEO description
  coverImage:    String,
  isPublished:   Boolean,
  publishedAt:   Date,
  author:        ObjectId (ref: User),
  createdAt:     Date,
  updatedAt:     Date
}
```

### Media

```typescript
// lib/models/Media.ts
{
  filename:      String,          // "red-velvet-cake.jpg"
  url:           String,          // Cloudinary / S3 URL
  thumbnailUrl:  String,          // Thumbnail version
  mimeType:      String,          // "image/jpeg"
  size:          Number,          // bytes
  width:         Number,
  height:        Number,
  alt:           String,          // Alt text
  folder:        String,          // "cakes" | "hero" | "about"
  uploadedBy:    ObjectId (ref: User),
  createdAt:     Date
}
```

### SiteSettings

```typescript
// lib/models/SiteSettings.ts
{
  siteName:      String,          // "Sweet Delights Bakery"
  tagline:       String,          // "Handcrafted Cakes & Pastries"
  favicon:       String,          // URL
  seo: {
    defaultTitle:       String,
    defaultDescription: String,
    ogImage:            String
  },
  theme: {
    primaryColor:    String,      // "#D4A574" (warm gold)
    secondaryColor:  String,      // "#8B4513" (chocolate brown)
    accentColor:     String,      // "#F5E6D3" (cream)
    fontHeading:     String,      // "Playfair Display"
    fontBody:        String       // "Lato"
  },
  maintenance: {
    isEnabled:   Boolean,
    message:     String
  },
  updatedAt:     Date
}
```

---

## 7. Authentication & Authorization

### Strategy: NextAuth.js v5 + Middleware

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Login      │────▶│  NextAuth    │────▶│  MongoDB     │
│   Page       │     │  Credentials │     │  User Check  │
└──────────────┘     │  Provider    │     └──────┬───────┘
                     └──────────────┘            │
                                                 ▼
                     ┌──────────────┐     ┌──────────────┐
                     │  JWT Token   │◀────│  Password    │
                     │  (role incl.)│     │  Verify      │
                     └──────┬───────┘     └──────────────┘
                            │
                            ▼
                     ┌──────────────┐
                     │  Middleware   │
                     │  Check Role  │
                     └──────────────┘
```

### Middleware (root `middleware.ts`)

```typescript
// middleware.ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export default auth((req) => {
  const { pathname } = req.nextUrl;
  const isAdminRoute = pathname.startsWith("/admin");
  const isApiAdminRoute = pathname.startsWith("/api/admin");
  const isLoginPage = pathname === "/admin/login";

  // Allow login page
  if (isLoginPage) return NextResponse.next();

  // Protect admin routes
  if (isAdminRoute || isApiAdminRoute) {
    if (!req.auth) {
      return NextResponse.redirect(new URL("/admin/login", req.url));
    }
    if (req.auth.user.role !== "admin" && req.auth.user.role !== "superadmin") {
      return NextResponse.redirect(new URL("/", req.url));
    }
  }

  return NextResponse.next();
});

export const config = {
  matcher: ["/admin/:path*", "/api/admin/:path*"],
};
```

### Auth Configuration

```typescript
// lib/auth.ts
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import bcrypt from "bcryptjs";
import { User } from "./models/User";
import { connectDB } from "./db";

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    Credentials({
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        await connectDB();
        const user = await User.findOne({ email: credentials.email });
        if (!user || !user.isActive) return null;

        const isValid = await bcrypt.compare(credentials.password, user.password);
        if (!isValid) return null;

        return { id: user._id, name: user.name, email: user.email, role: user.role };
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) token.role = user.role;
      return token;
    },
    async session({ session, token }) {
      session.user.role = token.role;
      return session;
    },
  },
  pages: {
    signIn: "/admin/login",
  },
});
```

### Security Layers

| Layer | Protection |
|-------|-----------|
| **Middleware** | Redirects unauthenticated users away from `/admin/*` |
| **API Route Handlers** | Verify session + role before any mutation |
| **Server Actions** | Verify session inside each action |
| **UI** | Conditionally render admin UI (defense in depth) |

---

## 8. Admin Panel — Content Management

### Admin Dashboard Layout

```
┌─────────────────────────────────────────────────────────┐
│  Topbar  [Sweet Delights Admin]        [Admin ▼] [🔔]  │
├────────────┬────────────────────────────────────────────┤
│            │                                            │
│  Sidebar   │            Main Content Area               │
│            │                                            │
│  Dashboard │   ┌──────────────────────────────────┐     │
│  ─────────  │   │  Dashboard Stats                 │     │
│  Header    │   │  ┌────┐ ┌────┐ ┌────┐ ┌────┐    │     │
│  Hero      │   │  │Pages│ │Media│ │Posts│ │Visits│  │     │
│  About     │   │  │ 12  │ │ 48  │ │ 23 │ │1.2K │  │     │
│  Favorites │   │  └────┘ └────┘ └────┘ └────┘    │     │
│  Updates   │   └──────────────────────────────────┘     │
│  Visit     │                                            │
│  ─────────  │   ┌──────────────────────────────────┐     │
│  Pages     │   │  Recent Activity                  │     │
│  Media     │   │  • Hero updated — 2 hours ago     │     │
│  ─────────  │   │  • New cake added — 5 hours ago   │     │
│  Footer    │   │  • Page published — 1 day ago     │     │
│  Settings  │   └──────────────────────────────────┘     │
│            │                                            │
└────────────┴────────────────────────────────────────────┘
```

### Content Management Flow

```
Admin opens section
        │
        ▼
┌─────────────────┐     ┌──────────────────┐
│  Fetch current  │────▶│  Pre-fill form   │
│  data (Server)  │     │  with data       │
└─────────────────┘     └────────┬─────────┘
                                 │
                                 ▼
                        ┌──────────────────┐
                        │  Admin edits     │
                        │  content         │
                        └────────┬─────────┘
                                 │
                                 ▼
                        ┌──────────────────┐
                        │  Client-side     │
                        │  Zod validation  │
                        └────────┬─────────┘
                                 │
                          ┌──────┴──────┐
                          ▼             ▼
                     ┌────────┐   ┌──────────┐
                     │ Valid  │   │ Invalid  │
                     └───┬────┘   │ Show     │
                         │        │ errors   │
                         ▼        └──────────┘
                  ┌──────────────┐
                  │ Server Action│
                  │ / API call   │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ Server-side  │
                  │ Zod + Auth   │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ MongoDB      │
                  │ Update       │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ Revalidate   │
                  │ Cache +      │
                  │ Toast        │
                  └──────────────┘
```

### Admin Features Per Section

| Section | Admin Capabilities |
|---------|-------------------|
| **Header** | Upload logo, add/reorder/hide nav links, toggle sticky, edit CTA button |
| **Hero** | Add/edit/remove slides, upload backgrounds, set overlay, autoplay speed |
| **About** | Edit story text (rich editor), manage team members, update stats |
| **Favorites** | CRUD cakes, set featured, manage categories, drag-to-reorder |
| **Updates** | CRUD blog posts, rich text editor, schedule publishing |
| **Visit** | Edit address, business hours, phone, email, map embed |
| **Footer** | Edit sections/links, social links, copyright text, newsletter toggle |
| **Pages** | Create/edit/delete custom pages with slug, rich text, SEO fields |
| **Media** | Upload images, browse library, delete, set alt text, organize by folder |
| **Settings** | Site name, SEO defaults, theme colors, maintenance mode |

---

## 9. Public Website — Pages & Sections

### Home Page Layout

```
┌─────────────────────────────────────────────────────────┐
│                    HEADER                                │
│  [Logo]    Home  About  Menu  Gallery  Contact  [Order] │
├─────────────────────────────────────────────────────────┤
│                                                          │
│                  HERO SECTION                            │
│           ┌──────────────────────────┐                   │
│           │                          │                   │
│           │   "Handcrafted with      │                   │
│           │        Love"             │                   │
│           │                          │                   │
│           │   Premium cakes for      │                   │
│           │   every occasion         │                   │
│           │                          │                   │
│           │   [ Explore Our Cakes ]  │                   │
│           │                          │                   │
│           │     ● ○ ○  (slides)      │                   │
│           └──────────────────────────┘                   │
│                                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│              ✦ OUR FAVORITES ✦                           │
│                                                          │
│    ┌─────────┐  ┌─────────┐  ┌─────────┐               │
│    │  Cake   │  │  Cake   │  │  Cake   │               │
│    │  Image  │  │  Image  │  │  Image  │               │
│    │         │  │         │  │         │               │
│    │ Red     │  │ Choco   │  │ Vanilla │               │
│    │ Velvet  │  │ Truffle │  │ Dream   │               │
│    │ $45.99  │  │ $38.99  │  │ $42.99  │               │
│    └─────────┘  └─────────┘  └─────────┘               │
│                                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│              LATEST UPDATES                              │
│                                                          │
│    ┌──────────────────┐  ┌──────────────────┐           │
│    │  Cover Image     │  │  Cover Image     │           │
│    │                  │  │                  │           │
│    │  New Summer      │  │  Wedding Cake    │           │
│    │  Collection!     │  │  Trends 2026     │           │
│    │  Mar 10, 2026    │  │  Mar 5, 2026     │           │
│    └──────────────────┘  └──────────────────┘           │
│                                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│            VISIT THE CAKE SHOP                           │
│                                                          │
│    ┌────────────────┐    Address: 123 Baker St           │
│    │                │    Phone: (555) 123-4567           │
│    │   Google Map   │    Hours: Mon-Sat 9AM-6PM         │
│    │   Embed        │                                    │
│    │                │    [ Get Directions ]              │
│    └────────────────┘                                    │
│                                                          │
├─────────────────────────────────────────────────────────┤
│                    FOOTER                                │
│  [Logo]  Quick Links  |  Social  |  Newsletter          │
│          About           Instagram   [Email      ]      │
│          Menu            Facebook    [Subscribe  ]      │
│          Contact         Twitter                        │
│                                                          │
│         © 2026 Sweet Delights Bakery                    │
└─────────────────────────────────────────────────────────┘
```

### Data Fetching Strategy

| Section | Rendering | Revalidation |
|---------|-----------|-------------|
| Header | Server Component | `revalidateTag("header")` |
| Hero | Server Component + Client carousel | `revalidateTag("hero")` |
| About | Server Component | `revalidateTag("about")` |
| Favorites | Server Component | `revalidateTag("favorites")` |
| Updates | Server Component | `revalidateTag("updates")` |
| Visit | Server Component | `revalidateTag("visit")` |
| Footer | Server Component | `revalidateTag("footer")` |
| Custom Pages | Dynamic SSR | `revalidateTag("pages")` |

All public pages use **Server Components** for initial data fetch. Interactive elements (carousel, animations, forms) are wrapped in small Client Components.

---

## 10. API Design (Route Handlers)

### Admin API Endpoints

All admin endpoints require authentication and admin role.

```
# Header
GET    /api/admin/header          → Fetch header config
PUT    /api/admin/header          → Update header config

# Hero
GET    /api/admin/hero            → Fetch hero config
PUT    /api/admin/hero            → Update hero config

# About
GET    /api/admin/about           → Fetch about content
PUT    /api/admin/about           → Update about content

# Favorites (CRUD)
GET    /api/admin/favorites       → List all favorites
POST   /api/admin/favorites       → Create new favorite
PUT    /api/admin/favorites/[id]  → Update favorite
DELETE /api/admin/favorites/[id]  → Delete favorite
PATCH  /api/admin/favorites/reorder → Reorder favorites

# Updates (CRUD)
GET    /api/admin/updates         → List all updates
POST   /api/admin/updates         → Create new update
PUT    /api/admin/updates/[id]    → Update post
DELETE /api/admin/updates/[id]    → Delete post

# Visit
GET    /api/admin/visit           → Fetch visit config
PUT    /api/admin/visit           → Update visit config

# Footer
GET    /api/admin/footer          → Fetch footer config
PUT    /api/admin/footer          → Update footer config

# Custom Pages (CRUD)
GET    /api/admin/pages           → List all custom pages
POST   /api/admin/pages           → Create new page
PUT    /api/admin/pages/[id]      → Update page
DELETE /api/admin/pages/[id]      → Delete page

# Media
GET    /api/admin/media           → List media (paginated)
POST   /api/admin/media           → Upload media
DELETE /api/admin/media/[id]      → Delete media

# Settings
GET    /api/admin/settings        → Fetch site settings
PUT    /api/admin/settings        → Update site settings
```

### Public API Endpoints

```
GET    /api/public/content        → Fetch all homepage content (batched)
GET    /api/public/pages/[slug]   → Fetch single custom page
```

### Preferred Approach: Server Actions

For the admin panel, **Server Actions** are preferred over Route Handlers for form submissions:

```typescript
// lib/actions/hero.actions.ts
"use server";

import { auth } from "@/lib/auth";
import { Hero } from "@/lib/models/Hero";
import { heroSchema } from "@/lib/validations/hero.schema";
import { revalidateTag } from "next/cache";

export async function updateHero(formData: FormData) {
  const session = await auth();
  if (!session || !["admin", "superadmin"].includes(session.user.role)) {
    throw new Error("Unauthorized");
  }

  const data = heroSchema.parse(Object.fromEntries(formData));
  await Hero.findOneAndUpdate({}, data, { upsert: true });

  revalidateTag("hero");
  return { success: true };
}
```

---

## 11. Image & Media Management

### Upload Flow

```
Admin selects image
        │
        ▼
┌──────────────────┐
│ Client-side      │
│ • Preview        │
│ • Validate type  │
│ • Validate size  │
│ • Compress       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Upload to        │
│ Cloudinary / S3  │
│ via API route    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Save metadata    │
│ to MongoDB       │
│ (Media model)    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Return URL +     │
│ thumbnail URL    │
└──────────────────┘
```

### Recommended: Cloudinary Integration

| Feature | Benefit |
|---------|---------|
| Auto-optimization | WebP/AVIF conversion, responsive sizes |
| Transformations | Crop, resize, blur on-the-fly via URL params |
| CDN delivery | Global edge caching |
| Free tier | 25GB storage, 25GB bandwidth/month |

### Media Picker Component

When editing any section, admins can either:
1. **Upload new** — opens file picker, uploads to cloud, saves to Media
2. **Choose existing** — opens Media Library modal with search/filter

---

## 12. UI/UX Design Guidelines

### Color Palette (Premium Bakery)

```css
:root {
  --color-primary:    #D4A574;   /* Warm Gold — main brand color */
  --color-secondary:  #8B4513;   /* Chocolate Brown — headings, accents */
  --color-accent:     #F5E6D3;   /* Cream — backgrounds, cards */
  --color-dark:       #2C1810;   /* Dark Espresso — text */
  --color-light:      #FFF8F0;   /* Off-White — page background */
  --color-rose:       #E8B4B8;   /* Soft Rose — highlights */
}
```

### Typography

| Use | Font | Style |
|-----|------|-------|
| Headings | **Playfair Display** | Serif, elegant, premium feel |
| Body | **Lato** or **Inter** | Clean sans-serif, readable |
| Accent | **Great Vibes** | Script font for decorative text |

### Animation Strategy (Framer Motion)

```typescript
// components/public/AnimatedSection.tsx
"use client";
import { motion } from "framer-motion";

const fadeInUp = {
  hidden: { opacity: 0, y: 40 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.6, ease: "easeOut" } }
};

export function AnimatedSection({ children }) {
  return (
    <motion.div
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true, margin: "-100px" }}
      variants={fadeInUp}
    >
      {children}
    </motion.div>
  );
}
```

### Key UX Patterns

| Pattern | Implementation |
|---------|---------------|
| **Smooth scrolling** | CSS `scroll-behavior: smooth` + section anchors |
| **Parallax hero** | Subtle background parallax on hero images |
| **Hover effects** | Cake cards lift + shadow on hover |
| **Loading states** | Skeleton placeholders matching content layout |
| **Page transitions** | Fade transitions between routes |
| **Image reveal** | Images fade in as they enter viewport |
| **Sticky header** | Header becomes compact + adds backdrop blur on scroll |

---

## 13. Performance & SEO Strategy

### Performance

| Technique | Implementation |
|-----------|---------------|
| **Server Components** | Default for all data-fetching pages |
| **Image optimization** | `next/image` with Cloudinary loader |
| **Font optimization** | `next/font` for Google Fonts (zero layout shift) |
| **Code splitting** | Dynamic imports for heavy components (editor, maps) |
| **ISR** | Tag-based revalidation for content updates |
| **Bundle analysis** | `@next/bundle-analyzer` in dev |

### SEO

```typescript
// app/(public)/layout.tsx
import { Metadata } from "next";

export async function generateMetadata(): Promise<Metadata> {
  const settings = await fetchSiteSettings(); // cached

  return {
    title: {
      default: settings.seo.defaultTitle,
      template: `%s | ${settings.siteName}`,
    },
    description: settings.seo.defaultDescription,
    openGraph: {
      images: [settings.seo.ogImage],
      siteName: settings.siteName,
    },
  };
}
```

| SEO Feature | Implementation |
|-------------|---------------|
| **Dynamic metadata** | Per-page title, description, OG image |
| **Sitemap** | `app/sitemap.ts` auto-generates from pages + custom pages |
| **Robots** | `app/robots.ts` with admin routes disallowed |
| **Structured data** | JSON-LD for LocalBusiness, BreadcrumbList |
| **Canonical URLs** | Auto-set via metadata API |

---

## 14. Deployment Strategy

### Recommended: Vercel

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│   GitHub     │────▶│   Vercel     │────▶│  Production  │
│   Push       │     │   Build      │     │  Deploy      │
└─────────────┘     └──────────────┘     └──────────────┘
                                                │
                    ┌──────────────┐             │
                    │  MongoDB     │◀────────────┘
                    │  Atlas       │
                    └──────────────┘
                    ┌──────────────┐
                    │  Cloudinary  │◀──── (image CDN)
                    └──────────────┘
```

### Environment Variables

```env
# .env.local
MONGODB_URI=mongodb+srv://...
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=https://yourdomain.com

CLOUDINARY_CLOUD_NAME=your-cloud
CLOUDINARY_API_KEY=your-key
CLOUDINARY_API_SECRET=your-secret

# Optional
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```

### Initial Admin Seeding

```typescript
// scripts/seed-admin.ts
import bcrypt from "bcryptjs";
import { connectDB } from "../lib/db";
import { User } from "../lib/models/User";

async function seedAdmin() {
  await connectDB();
  const exists = await User.findOne({ role: "superadmin" });
  if (exists) return console.log("Admin already exists");

  await User.create({
    name: "Super Admin",
    email: "admin@cakeshop.com",
    password: await bcrypt.hash("ChangeMe123!", 12),
    role: "superadmin",
    isActive: true,
  });
  console.log("Admin user created!");
}

seedAdmin();
```

---

## 15. Future Feature Suggestions

### Phase 2 — E-Commerce & Orders

| Feature | Description |
|---------|-------------|
| **Online Ordering** | Add to cart, checkout flow with Stripe/Razorpay integration |
| **Custom Cake Builder** | Interactive cake customizer (layers, flavors, toppings, message) |
| **Order Tracking** | Real-time order status updates for customers |
| **Pricing Tiers** | Size-based pricing (6", 8", 10") with automatic calculation |
| **Coupon System** | Discount codes, seasonal promotions, first-order discounts |

### Phase 3 — Customer Engagement

| Feature | Description |
|---------|-------------|
| **Customer Reviews** | Star ratings + photo reviews on cakes |
| **Wishlist / Favorites** | Customers can save their favorite cakes |
| **Newsletter** | Email subscription with Mailchimp/Resend integration |
| **Live Chat** | Tawk.to or custom chat widget for cake inquiries |
| **Social Media Feed** | Instagram feed embed showing latest cake photos |
| **Loyalty Program** | Points system — earn on orders, redeem for discounts |

### Phase 4 — Advanced CMS

| Feature | Description |
|---------|-------------|
| **Multi-language Support** | i18n for header, content, and pages |
| **Drag-and-Drop Page Builder** | Visual page builder for custom pages |
| **Content Versioning** | History/undo for all content changes |
| **Scheduled Publishing** | Set publish/unpublish dates for updates and pages |
| **Role-Based Access** | Editor, Manager, Super Admin with granular permissions |
| **Activity Log** | Track all admin actions (who changed what, when) |

### Phase 5 — Analytics & Growth

| Feature | Description |
|---------|-------------|
| **Analytics Dashboard** | Page views, popular cakes, conversion tracking |
| **A/B Testing** | Test hero variants, CTA text, layouts |
| **SEO Analyzer** | Built-in SEO score for each page |
| **PWA Support** | Installable app with offline browsing |
| **Push Notifications** | Notify customers about new cakes, offers |
| **AI Recommendations** | "You might also like" based on browsing history |

### Phase 6 — Operations

| Feature | Description |
|---------|-------------|
| **Inventory Management** | Track ingredients, availability, stock levels |
| **Delivery Zones** | Define delivery areas with radius-based pricing |
| **Calendar Integration** | Manage bookings, delivery slots, event orders |
| **Multi-Branch Support** | Manage multiple shop locations from one admin |
| **Automated Backups** | Scheduled MongoDB backups with one-click restore |

---

## Quick Start Checklist

```
[ ] Initialize Next.js 15 project with TypeScript
[ ] Set up Tailwind CSS 4 + custom theme
[ ] Configure MongoDB connection (lib/db.ts)
[ ] Define all Mongoose models
[ ] Set up NextAuth.js v5 with credentials provider
[ ] Create middleware for admin route protection
[ ] Seed initial admin user
[ ] Build admin layout (sidebar + topbar)
[ ] Build admin CRUD pages for each section
[ ] Build public layout (header + footer from DB)
[ ] Build homepage sections (Hero, Favorites, Updates, Visit)
[ ] Set up Cloudinary for media uploads
[ ] Add Framer Motion animations
[ ] Configure SEO metadata
[ ] Deploy to Vercel + connect MongoDB Atlas
[ ] Test admin flow end-to-end
[ ] Test public site performance (Lighthouse)
```

---

*Architecture designed for **Sweet Delights Cake Shop** — a premium, admin-manageable cake shop website built with Next.js 15 and MongoDB.*
