# Cake Shop — Detailed Diagrams

---

## 1. System Architecture Diagram (High-Level)

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                              INTERNET / BROWSER                              ║
╠═══════════════════════════════════════════════════════════════════════════════╣
║                                                                               ║
║    ┌──────────────────────────┐        ┌──────────────────────────┐           ║
║    │   CUSTOMER (Public)      │        │   ADMIN (Protected)      │           ║
║    │                          │        │                          │           ║
║    │  • Browse cakes          │        │  • Login with email/pw   │           ║
║    │  • Read blog updates     │        │  • Manage all sections   │           ║
║    │  • View shop info        │        │  • Upload media          │           ║
║    │  • Read custom pages     │        │  • Create custom pages   │           ║
║    │  • Contact / directions  │        │  • Site settings         │           ║
║    └────────────┬─────────────┘        └────────────┬─────────────┘           ║
║                 │                                    │                         ║
║                 │         HTTPS Requests             │                         ║
║                 └──────────────┬──────────────────────┘                        ║
║                                │                                              ║
╚════════════════════════════════╪══════════════════════════════════════════════╝
                                 │
                                 ▼
╔════════════════════════════════════════════════════════════════════════════════╗
║                         VERCEL EDGE NETWORK                                   ║
║                                                                               ║
║  ┌─────────────────────────────────────────────────────────────────────────┐   ║
║  │                        MIDDLEWARE LAYER                                  │   ║
║  │                                                                         │   ║
║  │   Every request passes through here first                               │   ║
║  │                                                                         │   ║
║  │   ┌─────────────┐    ┌──────────────────┐    ┌───────────────────┐      │   ║
║  │   │ Is /admin/* │───▶│ Has valid JWT?   │───▶│ Role == admin?   │      │   ║
║  │   │ or          │ NO │                  │ NO │                  │      │   ║
║  │   │ /api/admin? │──┐ │ Redirect to     │──┐ │ Redirect to /   │      │   ║
║  │   └─────────────┘  │ │ /admin/login    │  │ └───────┬─────────┘      │   ║
║  │                     │ └──────────────────┘  │         │ YES            │   ║
║  │        Pass through │                       │         ▼                │   ║
║  │             ▼       │                       │   ┌────────────┐        │   ║
║  │      Public routes  │                       │   │ Allow      │        │   ║
║  │      served         │                       │   │ Access     │        │   ║
║  │                     │                       │   └────────────┘        │   ║
║  └─────────────────────┴───────────────────────┴─────────────────────────┘   ║
║                                                                               ║
║  ┌─────────────────────────────────────────────────────────────────────────┐   ║
║  │                      NEXT.JS 15 APP ROUTER                              │   ║
║  │                                                                         │   ║
║  │   ┌─────────────────────┐       ┌─────────────────────────┐             │   ║
║  │   │  SERVER COMPONENTS  │       │  CLIENT COMPONENTS      │             │   ║
║  │   │                     │       │                         │             │   ║
║  │   │  • Page rendering   │       │  • Hero carousel        │             │   ║
║  │   │  • Data fetching    │       │  • Admin forms          │             │   ║
║  │   │  • Metadata         │       │  • Media uploader       │             │   ║
║  │   │  • Layout           │       │  • Rich text editor     │             │   ║
║  │   │  • SEO              │       │  • Animations           │             │   ║
║  │   └──────────┬──────────┘       │  • Toasts / Modals      │             │   ║
║  │              │                  └─────────────────────────┘             │   ║
║  │              │                                                          │   ║
║  │   ┌──────────▼──────────────────────────────────────────────────────┐   │   ║
║  │   │  SERVER ACTIONS + API ROUTE HANDLERS                            │   │   ║
║  │   │                                                                 │   │   ║
║  │   │  Server Actions (preferred for forms):                          │   │   ║
║  │   │    updateHero(), updateAbout(), createFavorite(), etc.           │   │   ║
║  │   │                                                                 │   │   ║
║  │   │  Route Handlers (for REST API / media):                         │   │   ║
║  │   │    /api/admin/*    — CRUD endpoints (auth required)             │   │   ║
║  │   │    /api/public/*   — Read-only public endpoints                 │   │   ║
║  │   └──────────┬──────────────────────────────────────────────────────┘   │   ║
║  │              │                                                          │   ║
║  │   ┌──────────▼──────────────────────────────────────────────────────┐   │   ║
║  │   │  DATA ACCESS LAYER                                              │   │   ║
║  │   │                                                                 │   │   ║
║  │   │  Mongoose ODM  ←→  Zod Validation  ←→  Business Logic           │   │   ║
║  │   └──────────┬──────────────────────────────────────────────────────┘   │   ║
║  └──────────────┼──────────────────────────────────────────────────────────┘   ║
╚═════════════════╪═════════════════════════════════════════════════════════════╝
                  │
       ┌──────────┴──────────────────────────────────┐
       │                                              │
       ▼                                              ▼
╔══════════════════════════╗         ╔══════════════════════════╗
║     MONGODB ATLAS        ║         ║     CLOUDINARY CDN       ║
║                          ║         ║                          ║
║  ┌────────────────────┐  ║         ║  ┌────────────────────┐  ║
║  │  Collections:      │  ║         ║  │  Folders:          │  ║
║  │                    │  ║         ║  │                    │  ║
║  │  users             │  ║         ║  │  /cakeshop/hero    │  ║
║  │  headers           │  ║         ║  │  /cakeshop/cakes   │  ║
║  │  footers           │  ║         ║  │  /cakeshop/about   │  ║
║  │  heroes            │  ║         ║  │  /cakeshop/updates │  ║
║  │  abouts            │  ║         ║  │  /cakeshop/pages   │  ║
║  │  favorites         │  ║         ║  │  /cakeshop/gallery │  ║
║  │  updates           │  ║         ║  │                    │  ║
║  │  visits            │  ║         ║  │  Auto-generates:   │  ║
║  │  custompages       │  ║         ║  │  • WebP/AVIF       │  ║
║  │  media             │  ║         ║  │  • Thumbnails      │  ║
║  │  sitesettings      │  ║         ║  │  • Responsive      │  ║
║  └────────────────────┘  ║         ║  └────────────────────┘  ║
║                          ║         ║                          ║
╚══════════════════════════╝         ╚══════════════════════════╝
```

---

## 2. Authentication & Authorization Flow

```
                           ┌─────────────────────────────┐
                           │     Admin visits /admin      │
                           └──────────────┬──────────────┘
                                          │
                                          ▼
                           ┌─────────────────────────────┐
                           │   Middleware intercepts      │
                           │   the request                │
                           └──────────────┬──────────────┘
                                          │
                                ┌─────────┴─────────┐
                                │  Has JWT cookie?   │
                                └─────────┬─────────┘
                                          │
                              ┌───────────┴───────────┐
                              │                       │
                          YES ▼                   NO  ▼
                    ┌─────────────┐          ┌─────────────────┐
                    │ Decode JWT  │          │ Redirect to     │
                    │ & verify    │          │ /admin/login    │
                    └──────┬──────┘          └────────┬────────┘
                           │                          │
                    ┌──────┴──────┐                   ▼
                    │ role ==     │          ┌─────────────────┐
                    │ "admin" or  │          │  LOGIN PAGE     │
                    │ "superadmin"│          │                 │
                    └──────┬──────┘          │  ┌───────────┐  │
                           │                 │  │ Email     │  │
                  ┌────────┴────────┐        │  │ Password  │  │
                  │                 │        │  │ [Login]   │  │
              YES ▼             NO ▼        │  └─────┬─────┘  │
        ┌────────────┐    ┌──────────┐      └────────┼────────┘
        │ ACCESS     │    │ Redirect │               │
        │ GRANTED    │    │ to /     │               ▼
        │            │    │ (home)   │      ┌─────────────────┐
        │ Show admin │    └──────────┘      │ NextAuth        │
        │ dashboard  │                      │ Credentials     │
        └────────────┘                      │ Provider        │
                                            └────────┬────────┘
                                                     │
                                                     ▼
                                            ┌─────────────────┐
                                            │ Find user in    │
                                            │ MongoDB by      │
                                            │ email           │
                                            └────────┬────────┘
                                                     │
                                            ┌────────┴────────┐
                                            │  User found?    │
                                            └────────┬────────┘
                                                     │
                                          ┌──────────┴──────────┐
                                          │                     │
                                      YES ▼                 NO  ▼
                                ┌──────────────┐      ┌──────────────┐
                                │ bcrypt.compare│      │ Show error   │
                                │ password     │      │ "Invalid     │
                                └──────┬───────┘      │  credentials"│
                                       │              └──────────────┘
                              ┌────────┴────────┐
                              │  Match?         │
                              └────────┬────────┘
                                       │
                            ┌──────────┴──────────┐
                            │                     │
                        YES ▼                 NO  ▼
                  ┌──────────────┐       ┌──────────────┐
                  │ Create JWT   │       │ Show error   │
                  │ with:        │       │ "Invalid     │
                  │ • user id    │       │  credentials"│
                  │ • email      │       └──────────────┘
                  │ • role       │
                  │              │
                  │ Set HTTP-only│
                  │ cookie       │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ Redirect to  │
                  │ /admin       │
                  │ (dashboard)  │
                  └──────────────┘
```

---

## 3. Request Lifecycle — Public Page

```
Customer visits "/"
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│                    NEXT.JS SERVER                             │
│                                                               │
│   1. app/(public)/layout.tsx   (Server Component)            │
│      │                                                        │
│      ├── Fetch Header data from MongoDB ──────────────────┐   │
│      ├── Fetch Footer data from MongoDB ──────────────┐   │   │
│      │                                                │   │   │
│      │   Cache Strategy:                              │   │   │
│      │   fetch() with { next: { tags: ["header"] } }  │   │   │
│      │   Revalidated when admin updates header        │   │   │
│      │                                                │   │   │
│   2. app/(public)/page.tsx   (Server Component)       │   │   │
│      │                                                │   │   │
│      ├── Fetch Hero slides ───────────────────────┐   │   │   │
│      ├── Fetch Favorites (isFeatured: true) ──┐   │   │   │   │
│      ├── Fetch Latest Updates (limit: 4) ──┐  │   │   │   │   │
│      ├── Fetch Visit section data ──────┐  │  │   │   │   │   │
│      │                                  │  │  │   │   │   │   │
│      │   ┌──────────────────────────────┴──┴──┴───┴───┴───┘   │
│      │   │                                                     │
│      │   │         MongoDB (parallel queries)                  │
│      │   │                                                     │
│      │   │  db.heroes.findOne()                                │
│      │   │  db.favorites.find({isFeatured:true}).sort({order}) │
│      │   │  db.updates.find({isPublished:true}).limit(4)       │
│      │   │  db.visits.findOne()                                │
│      │   │  db.headers.findOne()                               │
│      │   │  db.footers.findOne()                               │
│      │   │                                                     │
│      │   └─────────────────────────────────────────────────────│
│      │                                                         │
│   3. RENDER (Server-side HTML)                                │
│      │                                                         │
│      ├── <Header />           ← Server Component              │
│      ├── <HeroSection />      ← Client Component (carousel)   │
│      ├── <FavoritesSection /> ← Server + Client (animations)  │
│      ├── <LatestUpdates />    ← Server Component              │
│      ├── <VisitSection />     ← Server + Client (map)         │
│      └── <Footer />           ← Server Component              │
│                                                               │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │  HTML + JS sent │
                   │  to browser     │
                   │                 │
                   │  • Full HTML    │
                   │    (SEO ready)  │
                   │  • Minimal JS   │
                   │    (hydration   │
                   │     for client  │
                   │     components) │
                   └─────────────────┘
```

---

## 4. Admin Content Update Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        ADMIN PANEL FLOW                                 │
└─────────────────────────────────────────────────────────────────────────┘

Admin clicks "Hero" in sidebar
        │
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  app/(admin)/admin/hero/page.tsx                                        │
│                                                                         │
│  SERVER: Fetch current hero data from MongoDB                           │
│  ┌──────────────────────────────────────────────────────────────┐       │
│  │  const heroData = await Hero.findOne({});                    │       │
│  └──────────────────────────────────────────────────────────────┘       │
│                                                                         │
│  Pass data to CLIENT component:                                         │
│  ┌──────────────────────────────────────────────────────────────┐       │
│  │  <HeroEditor initialData={heroData} />                       │       │
│  └──────────────────────────────────────────────────────────────┘       │
│                                                                         │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  CLIENT: HeroEditor Component                                           │
│                                                                         │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  FORM (React Hook Form + Zod)                                  │     │
│  │                                                                │     │
│  │  Slide 1:                                                      │     │
│  │  ┌─────────────────┐  Title:    [Handcrafted with Love    ]   │     │
│  │  │                 │  Subtitle: [Premium cakes for every..]   │     │
│  │  │  [Upload Image] │  CTA Text: [Explore Our Cakes       ]   │     │
│  │  │                 │  CTA Link: [/menu                    ]   │     │
│  │  │  current.jpg    │  Overlay:  [===●=====] 0.4               │     │
│  │  └─────────────────┘  Active:   [✓]                           │     │
│  │                                                                │     │
│  │  [+ Add Slide]                                                 │     │
│  │                                                                │     │
│  │  Autoplay Speed: [5000] ms                                     │     │
│  │                                                                │     │
│  │  ┌──────────────┐  ┌──────────────┐                           │     │
│  │  │   Preview    │  │  Save Changes│                           │     │
│  │  └──────────────┘  └──────┬───────┘                           │     │
│  └───────────────────────────┼────────────────────────────────────┘     │
│                              │                                          │
└──────────────────────────────┼──────────────────────────────────────────┘
                               │
                    Admin clicks "Save Changes"
                               │
                               ▼
                  ┌────────────────────────┐
                  │  CLIENT-SIDE           │
                  │  Zod Validation        │
                  │                        │
                  │  heroSchema.parse(data)│
                  └────────────┬───────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                VALID ▼              INVALID ▼
          ┌──────────────┐       ┌─────────────────┐
          │ Call Server  │       │ Show inline      │
          │ Action       │       │ error messages   │
          └──────┬───────┘       │ under fields     │
                 │               └─────────────────┘
                 ▼
          ┌──────────────────────────────────────────┐
          │  SERVER ACTION: updateHero()              │
          │                                          │
          │  1. Verify auth session                  │
          │     const session = await auth();        │
          │     if (!session) throw Unauthorized;    │
          │                                          │
          │  2. Server-side Zod validation           │
          │     heroSchema.parse(data);              │
          │                                          │
          │  3. Update MongoDB                       │
          │     await Hero.findOneAndUpdate(          │
          │       {}, data, { upsert: true }         │
          │     );                                   │
          │                                          │
          │  4. Revalidate cache                     │
          │     revalidateTag("hero");               │
          │                                          │
          │  5. Return { success: true }             │
          └──────────────┬───────────────────────────┘
                         │
                         ▼
          ┌──────────────────────────────┐
          │  CLIENT receives response    │
          │                              │
          │  ┌────────────────────────┐  │
          │  │  ✓ Hero section        │  │
          │  │    updated             │  │
          │  │    successfully!       │  │
          │  │         (toast)        │  │
          │  └────────────────────────┘  │
          │                              │
          │  Public site now shows       │
          │  updated hero immediately    │
          │  (cache was revalidated)     │
          └──────────────────────────────┘
```

---

## 5. Database Relationship Diagram (ERD)

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                        MongoDB Collections (ERD)                            ║
╚══════════════════════════════════════════════════════════════════════════════╝

 ┌─────────────────────┐
 │       USERS          │
 ├─────────────────────┤
 │ _id: ObjectId  [PK] │
 │ name: String        │
 │ email: String [UQ]  │──────────────────────────────────────────┐
 │ password: String    │                                          │
 │ role: String        │                                          │
 │ avatar: String      │                                          │
 │ isActive: Boolean   │                                          │
 │ lastLogin: Date     │                                          │
 │ createdAt: Date     │                                          │
 │ updatedAt: Date     │                                          │
 └─────────────────────┘                                          │
                                                                   │
  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ SINGLETON COLLECTIONS ─ ─ ─ ─│─ ─ ─ ─ ─ ┐
  │                                   (one document each)          │           │
  │                                                                │
  │  ┌─────────────────────┐   ┌─────────────────────┐            │           │
  │  │      HEADERS         │   │       HEROES         │            │
  │  ├─────────────────────┤   ├─────────────────────┤            │           │
     │ _id: ObjectId  [PK] │   │ _id: ObjectId  [PK] │            │
  │  │ logo: {             │   │ slides: [{          │            │           │
  │  │   imageUrl, altText,│   │   title, subtitle,  │            │
  │  │   linkTo            │   │   backgroundImage,  │            │           │
  │  │ }                   │   │   ctaText, ctaLink, │
  │  │ navigation: [{      │   │   overlayOpacity,   │            │           │
  │  │   label, href,      │   │   order, isActive   │
  │  │   order, isVisible  │   │ }]                  │            │           │
  │  │ }]                  │   │ autoplaySpeed: Num  │
  │  │ ctaButton: {        │   │ updatedAt: Date     │            │           │
  │  │   text, href,       │   └─────────────────────┘
  │  │   isVisible         │                                      │           │
  │  │ }                   │   ┌─────────────────────┐
  │  │ isSticky: Boolean   │   │       ABOUTS         │            │           │
  │  │ updatedAt: Date     │   ├─────────────────────┤
  │  └─────────────────────┘   │ _id: ObjectId  [PK] │            │           │
  │                             │ sectionTitle: Str   │
  │                             │ heading: String     │            │           │
  │  ┌─────────────────────┐   │ description: String │
  │  │       VISITS         │   │ images: [{url,alt}] │            │           │
  │  ├─────────────────────┤   │ stats: [{label,     │
  │  │ _id: ObjectId  [PK] │   │   value, icon}]     │            │           │
  │  │ sectionTitle: Str   │   │ teamMembers: [{     │
  │  │ heading: String     │   │   name, role,       │            │           │
  │  │ description: String │   │   image, bio}]      │
  │  │ address: {          │   │ isVisible: Boolean  │            │           │
  │  │   street, city,     │   │ updatedAt: Date     │
  │  │   state, zip,       │   └─────────────────────┘            │           │
  │  │   country           │
  │  │ }                   │   ┌─────────────────────┐            │           │
  │  │ phone: String       │   │      FOOTERS         │
  │  │ email: String       │   ├─────────────────────┤            │           │
  │  │ businessHours: [{   │   │ _id: ObjectId  [PK] │
  │  │   day, openTime,    │   │ logo: {imageUrl,    │            │           │
  │  │   closeTime,        │   │   altText}          │
  │  │   isClosed          │   │ description: String │            │           │
  │  │ }]                  │   │ sections: [{        │
  │  │ mapEmbedUrl: String │   │   title, links:[{   │            │           │
  │  │ images: [{url,alt}] │   │     label, href,    │
  │  │ isVisible: Boolean  │   │     isExternal}],   │            │           │
  │  │ updatedAt: Date     │   │   order             │
  │  └─────────────────────┘   │ }]                  │            │           │
  │                             │ socialLinks: [{     │
  │  ┌─────────────────────┐   │   platform, url,    │            │           │
  │  │    SITESETTINGS      │   │   icon}]            │
  │  ├─────────────────────┤   │ copyrightText: Str  │            │           │
  │  │ _id: ObjectId  [PK] │   │ newsletterEnabled   │
  │  │ siteName: String    │   │ updatedAt: Date     │            │           │
  │  │ tagline: String     │   └─────────────────────┘
  │  │ favicon: String     │                                      │           │
  │  │ seo: {defaultTitle, │
  │  │   defaultDesc,      │                                      │           │
  │  │   ogImage}          │
  │  │ theme: {primary,    │                                      │           │
  │  │   secondary, accent,│
  │  │   fontHeading,      │                                      │           │
  │  │   fontBody}         │
  │  │ maintenance: {      │                                      │           │
  │  │   isEnabled, msg}   │
  │  │ updatedAt: Date     │                                      │           │
  │  └─────────────────────┘
  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘

  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ MULTI-DOC COLLECTIONS ─ ─ ─ ─ ─ ─ ─ ─ ┐
  │                                   (many documents)                         │
  │                                                                            │
  │  ┌─────────────────────┐   ┌─────────────────────┐                        │
  │  │     FAVORITES        │   │      UPDATES         │                        │
  │  ├─────────────────────┤   ├─────────────────────┤                        │
     │ _id: ObjectId  [PK] │   │ _id: ObjectId  [PK] │
  │  │ name: String        │   │ title: String       │                        │
  │  │ slug: String   [UQ] │   │ slug: String   [UQ] │
  │  │ description: String │   │ excerpt: String     │                        │
  │  │ price: Number       │   │ content: String     │
  │  │ images: [{url,alt}] │   │ coverImage: String  │                        │
  │  │ category: String    │   │ author: ObjectId ───┼───── ref: Users
  │  │ tags: [String]      │   │ category: String    │                        │
  │  │ isFeatured: Boolean │   │ tags: [String]      │
  │  │ isAvailable: Boolean│   │ isPublished: Boolean│                        │
  │  │ rating: Number      │   │ publishedAt: Date   │
  │  │ order: Number       │   │ createdAt: Date     │                        │
  │  │ createdAt: Date     │   │ updatedAt: Date     │
  │  │ updatedAt: Date     │   └─────────────────────┘                        │
  │  └─────────────────────┘
  │                             ┌─────────────────────┐                        │
  │  ┌─────────────────────┐   │       MEDIA          │
  │  │    CUSTOMPAGES       │   ├─────────────────────┤                        │
  │  ├─────────────────────┤   │ _id: ObjectId  [PK] │
  │  │ _id: ObjectId  [PK] │   │ filename: String    │                        │
  │  │ title: String       │   │ url: String         │
  │  │ slug: String   [UQ] │   │ thumbnailUrl: String│                        │
  │  │ content: String     │   │ mimeType: String    │
  │  │ metaTitle: String   │   │ size: Number        │                        │
  │  │ metaDescription: Str│   │ width: Number       │
  │  │ coverImage: String  │   │ height: Number      │                        │
  │  │ isPublished: Boolean│   │ alt: String         │
  │  │ publishedAt: Date   │   │ folder: String      │                        │
  │  │ author: ObjectId ───┼── │ uploadedBy: ObjId ──┼───── ref: Users
  │  │ createdAt: Date     │   │ createdAt: Date     │                        │
  │  │ updatedAt: Date     │   └─────────────────────┘
  │  └─────────────────────┘                                                   │
  │                                                                            │
  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┘
```

---

## 6. Component Architecture Diagram

```
                            app/layout.tsx (Root)
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
        app/(public)/layout.tsx          app/(admin)/layout.tsx
        ┌───────────────────┐            ┌───────────────────┐
        │ <Header />  [SC]  │            │ <Sidebar />  [CC] │
        │ {children}        │            │ <Topbar />   [CC] │
        │ <Footer />  [SC]  │            │ {children}        │
        └───────┬───────────┘            └───────┬───────────┘
                │                                │
     ┌──────────┴──────────┐          ┌──────────┴──────────────────┐
     │                     │          │          │          │        │
  page.tsx            [slug]/      /admin     /admin/     /admin/  ...
  (Home)              page.tsx   page.tsx   hero/       favorites/
     │                             │       page.tsx    page.tsx
     │                             │          │           │
     ├── <HeroSection />  [CC]     │   <HeroEditor/> <FavTable/>
     │    └── <Carousel />         │     [CC]          [CC]
     │                             │      │             │
     ├── <FavoritesSection/> [SC]  │   ┌──┴──┐       ┌──┴──┐
     │    └── <CakeCard /> [SC]    │   │Form │       │CRUD │
     │        └── hover anim [CC]  │   │Image│       │Table│
     │                             │   │RTE  │       │Modal│
     ├── <LatestUpdates /> [SC]    │   └─────┘       └─────┘
     │    └── <UpdateCard /> [SC]  │
     │                             │
     ├── <VisitSection />  [SC]    │
     │    └── <Map />  [CC]        │
     │                             │
     └── <AnimatedSection /> [CC]  │
          (wraps each section)     │

[SC] = Server Component (default)
[CC] = Client Component ("use client")
```

---

## 7. Caching & Revalidation Strategy

```
┌──────────────────────────────────────────────────────────────────────┐
│                    CACHING ARCHITECTURE                               │
└──────────────────────────────────────────────────────────────────────┘

  ADMIN ACTION                    CACHE LAYER                PUBLIC VIEW
  ─────────────                   ───────────                ───────────

  Admin updates     ──▶   revalidateTag("hero")    ──▶   Homepage re-renders
  Hero section             Cache invalidated              with new hero data
                           │
                           ▼
                    ┌──────────────────────┐
                    │  Next.js Data Cache   │
                    │                      │
                    │  Tags:               │
                    │  ┌────────────────┐  │
                    │  │ "header"       │──┼──▶  Header component
                    │  │ "footer"       │──┼──▶  Footer component
                    │  │ "hero"         │──┼──▶  Hero section
                    │  │ "about"        │──┼──▶  About section/page
                    │  │ "favorites"    │──┼──▶  Favorites section
                    │  │ "updates"      │──┼──▶  Latest Updates
                    │  │ "visit"        │──┼──▶  Visit section
                    │  │ "pages"        │──┼──▶  Custom pages
                    │  │ "settings"     │──┼──▶  Metadata, theme
                    │  └────────────────┘  │
                    │                      │
                    │  When tag is          │
                    │  revalidated:         │
                    │  • Cached data purged │
                    │  • Next request       │
                    │    fetches fresh data │
                    │  • New response       │
                    │    cached with tag    │
                    └──────────────────────┘


  FLOW EXAMPLE:

  1. First visit to "/"
     │
     ├── Cache MISS → fetch Hero from MongoDB → cache with tag "hero"
     ├── Cache MISS → fetch Favorites from MongoDB → cache with tag "favorites"
     └── ... (all sections cached)

  2. Second visit to "/"
     │
     ├── Cache HIT → serve Hero from cache (fast!)
     ├── Cache HIT → serve Favorites from cache
     └── ... (all served from cache, no DB queries)

  3. Admin updates Hero
     │
     ├── Server Action: updateHero()
     ├── MongoDB updated
     ├── revalidateTag("hero")  ←── Only hero cache purged
     └── Other caches untouched (favorites, footer, etc.)

  4. Next visit to "/"
     │
     ├── Cache MISS → fetch Hero from MongoDB (fresh data!) → re-cache
     ├── Cache HIT → Favorites still cached
     └── ... (only hero was re-fetched)
```

---

## 8. Media Upload Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                      MEDIA UPLOAD PIPELINE                          │
└─────────────────────────────────────────────────────────────────────┘

  ADMIN BROWSER                     SERVER                    CLOUD
  ─────────────                     ──────                    ─────

  ┌───────────────┐
  │ 1. Select     │
  │    File       │
  │ ┌───────────┐ │
  │ │ cake.jpg  │ │
  │ │ 4.2 MB    │ │
  │ │ 3000x2000 │ │
  │ └───────────┘ │
  └───────┬───────┘
          │
          ▼
  ┌───────────────┐
  │ 2. Client     │
  │    Validation  │
  │               │
  │ ✓ Type: jpg   │
  │ ✓ Size: <10MB │
  │ ✓ Preview OK  │
  └───────┬───────┘
          │
          ▼
  ┌───────────────┐        ┌───────────────────┐
  │ 3. Upload     │───────▶│ 4. API Route      │
  │    Progress   │        │    /api/admin/media│
  │               │        │                   │
  │ [████████░░]  │        │ • Verify auth     │
  │    80%        │        │ • Validate file   │
  └───────────────┘        │ • Generate name   │
                           └─────────┬─────────┘
                                     │
                                     ▼
                           ┌───────────────────┐      ┌──────────────────┐
                           │ 5. Upload to      │─────▶│ 6. CLOUDINARY    │
                           │    Cloudinary      │      │                  │
                           │                   │      │ Auto-generates:  │
                           │ cloudinary.       │      │                  │
                           │  uploader.upload( │      │ • Original       │
                           │    file,          │      │   /cakeshop/     │
                           │    {folder:       │      │   cake_abc123    │
                           │     "cakeshop/    │      │                  │
                           │      cakes"}      │      │ • WebP version   │
                           │  )               │      │   (auto-format)  │
                           └───────────────────┘      │                  │
                                                      │ • Thumbnail      │
                                                      │   (c_thumb,      │
                                                      │    w_300,h_300)  │
                                                      │                  │
                                                      │ Returns:         │
                                                      │ • secure_url     │
                                                      │ • width, height  │
                                                      │ • format, bytes  │
                                                      └────────┬─────────┘
                                                               │
                                     ┌─────────────────────────┘
                                     │
                                     ▼
                           ┌───────────────────┐
                           │ 7. Save metadata  │
                           │    to MongoDB     │
                           │                   │
                           │ Media.create({    │
                           │   filename,       │
                           │   url,            │
                           │   thumbnailUrl,   │
                           │   mimeType,       │
                           │   size,           │
                           │   width, height,  │
                           │   folder,         │
                           │   uploadedBy      │
                           │ })                │
                           └─────────┬─────────┘
                                     │
                                     ▼
  ┌───────────────┐        ┌───────────────────┐
  │ 8. Show       │◀───────│ Return URL +      │
  │    success    │        │ metadata to       │
  │               │        │ client            │
  │ ┌───────────┐ │        └───────────────────┘
  │ │ cake.jpg  │ │
  │ │ ✓ Uploaded│ │
  │ └───────────┘ │
  └───────────────┘
```

---

## 9. Public Website Page Flow (User Journey)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        CUSTOMER JOURNEY MAP                              │
└──────────────────────────────────────────────────────────────────────────┘


    ┌─────────┐
    │ Customer│
    │ arrives │
    └────┬────┘
         │
         ▼
    ┌─────────────────────────────────────────────┐
    │              HOME PAGE  "/"                  │
    │                                              │
    │  ┌────────────────────────────────────────┐  │
    │  │           HERO CAROUSEL                │  │
    │  │    "Handcrafted with Love"             │  │
    │  │    [Explore Our Cakes] ────────────────┼──┼──── → /menu
    │  └────────────────────────────────────────┘  │
    │              ↓ scroll                        │
    │  ┌────────────────────────────────────────┐  │
    │  │         OUR FAVORITES                  │  │
    │  │    [Cake] [Cake] [Cake] [Cake]         │  │
    │  │     Click any cake ────────────────────┼──┼──── → /menu#cake-slug
    │  │    [View All] ────────────────────────┼──┼──── → /menu
    │  └────────────────────────────────────────┘  │
    │              ↓ scroll                        │
    │  ┌────────────────────────────────────────┐  │
    │  │        LATEST UPDATES                  │  │
    │  │    [Blog Post]  [Blog Post]            │  │
    │  │     Click post ───────────────────────┼──┼──── → /updates/slug
    │  └────────────────────────────────────────┘  │
    │              ↓ scroll                        │
    │  ┌────────────────────────────────────────┐  │
    │  │       VISIT THE CAKE SHOP              │  │
    │  │    [Map]  Address  Hours               │  │
    │  │    [Get Directions] ──────────────────┼──┼──── → Google Maps
    │  └────────────────────────────────────────┘  │
    │                                              │
    └──────────────────────────────────────────────┘

    Navigation (available on every page):

    ┌──────┐  ┌───────┐  ┌──────┐  ┌─────────┐  ┌─────────┐  ┌────────┐
    │ Home │  │ About │  │ Menu │  │ Gallery │  │ Contact │  │ [slug] │
    │  /   │  │/about │  │/menu │  │/gallery │  │/contact │  │/[slug] │
    └──┬───┘  └───┬───┘  └──┬───┘  └────┬────┘  └────┬────┘  └───┬────┘
       │          │         │           │             │            │
       │          │         │           │             │            │
       ▼          ▼         ▼           ▼             ▼            ▼
    Home       Our Story  Full Cake   Photo       Contact     Dynamic
    (all       + Team     Catalog     Gallery     Form +      Custom
    sections)  + Stats    + Filter    + Lightbox  Map +       Pages
                          + Search               Hours       (CMS)
```

---

## 10. Deployment Pipeline

```
┌──────────────────────────────────────────────────────────────────────────┐
│                       DEPLOYMENT PIPELINE                                │
└──────────────────────────────────────────────────────────────────────────┘


  DEVELOPMENT                    STAGING                     PRODUCTION
  ───────────                    ───────                     ──────────

  ┌─────────────┐
  │ Developer   │
  │ pushes to   │
  │ feature     │
  │ branch      │
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐    ┌─────────────┐
  │ GitHub      │───▶│ Vercel      │
  │ Repository  │    │ Preview     │
  │             │    │ Deployment  │
  │ PR created  │    │             │
  └──────┬──────┘    │ https://    │
         │           │ pr-123.     │
         │           │ vercel.app  │
         │           └──────┬──────┘
         │                  │
         │           ┌──────▼──────┐
         │           │ Preview     │
         │           │ Environment │
         │           │             │
         │           │ MongoDB:    │
         │           │ dev-cluster │
         │           │             │
         │           │ Cloudinary: │
         │           │ dev-folder  │
         │           └─────────────┘
         │
         ▼
  ┌─────────────┐
  │ Merge to    │
  │ main branch │
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐
  │ GitHub      │───▶│ Vercel      │───▶│ PRODUCTION          │
  │ main branch │    │ Production  │    │                     │
  │             │    │ Build       │    │ https://             │
  │             │    │             │    │ sweetdelights.com    │
  │             │    │ • Build     │    │                     │
  │             │    │ • Optimize  │    │ MongoDB:             │
  │             │    │ • Deploy    │    │ production-cluster   │
  │             │    │             │    │                     │
  │             │    │ ~60 seconds │    │ Cloudinary:          │
  │             │    │             │    │ production-folder    │
  │             │    └─────────────┘    │                     │
  │             │                       │ Vercel Edge Network  │
  │             │                       │ (Global CDN)         │
  │             │                       └─────────────────────┘
  └─────────────┘


  ENVIRONMENT VARIABLES (per environment):

  ┌────────────────────────────────────────────────────────────────┐
  │  Development (.env.local)                                      │
  │  MONGODB_URI = mongodb://localhost:27017/cakeshop-dev          │
  │  NEXTAUTH_URL = http://localhost:3000                          │
  ├────────────────────────────────────────────────────────────────┤
  │  Preview (Vercel env vars)                                     │
  │  MONGODB_URI = mongodb+srv://...cakeshop-staging              │
  │  NEXTAUTH_URL = https://pr-xxx.vercel.app                     │
  ├────────────────────────────────────────────────────────────────┤
  │  Production (Vercel env vars)                                  │
  │  MONGODB_URI = mongodb+srv://...cakeshop-prod                 │
  │  NEXTAUTH_URL = https://sweetdelights.com                     │
  └────────────────────────────────────────────────────────────────┘
```

---

*These diagrams provide a complete visual reference for the Cake Shop architecture. Use them alongside the main architecture document (`CAKE-SHOP-ARCHITECTURE.md`) for implementation guidance.*
