# Project Structure - Firebase Blog Platform

## Overview

This document outlines the complete project structure for the Firebase-based blog platform, organized to support scalable development, clear separation of concerns, and maintainable code architecture.

## Root Directory Structure

```
firebase-blog/
├── README.md                           # Project overview and setup
├── IMPLEMENTATION_PLAN.md              # Detailed implementation roadmap
├── TECHNICAL_SPECIFICATIONS.md         # Technical architecture specs
├── FEATURE_ANALYSIS.md                 # Ghost feature reverse engineering
├── PROJECT_STRUCTURE.md                # This file
├── package.json                        # Root package configuration
├── firebase.json                       # Firebase project configuration
├── .firebaserc                         # Firebase project aliases
├── .gitignore                          # Git ignore patterns
├── .env.example                        # Environment variables template
├── docker-compose.yml                  # Local development setup
├── scripts/                            # Build and deployment scripts
├── docs/                               # Additional documentation
├── functions/                          # Firebase Functions (Backend)
├── apps/                               # Frontend Applications
├── packages/                           # Shared Libraries
├── public/                             # Static Assets
├── firestore.rules                     # Firestore security rules
├── firestore.indexes.json              # Firestore indexes
├── storage.rules                       # Firebase Storage rules
└── tests/                              # Integration and E2E tests
```

## Backend Structure (Firebase Functions)

```
functions/
├── package.json                        # Functions dependencies
├── tsconfig.json                       # TypeScript configuration
├── .eslintrc.js                        # ESLint configuration
├── src/
│   ├── index.ts                        # Main function exports
│   ├── config/
│   │   ├── firebase.ts                 # Firebase admin initialization
│   │   ├── environment.ts              # Environment configuration
│   │   ├── database.ts                 # Database connection setup
│   │   └── constants.ts                # Application constants
│   ├── api/
│   │   ├── index.ts                    # API router setup
│   │   ├── middleware/
│   │   │   ├── auth.ts                 # Authentication middleware
│   │   │   ├── validation.ts           # Request validation
│   │   │   ├── rateLimit.ts            # Rate limiting
│   │   │   ├── cors.ts                 # CORS configuration
│   │   │   ├── errorHandler.ts         # Error handling
│   │   │   └── logging.ts              # Request logging
│   │   ├── routes/
│   │   │   ├── content/                # Public content API
│   │   │   │   ├── posts.ts            # Posts endpoints
│   │   │   │   ├── pages.ts            # Pages endpoints
│   │   │   │   ├── tags.ts             # Tags endpoints
│   │   │   │   ├── authors.ts          # Authors endpoints
│   │   │   │   └── search.ts           # Search endpoints
│   │   │   ├── admin/                  # Admin API
│   │   │   │   ├── posts.ts            # Post management
│   │   │   │   ├── pages.ts            # Page management
│   │   │   │   ├── users.ts            # User management
│   │   │   │   ├── members.ts          # Member management
│   │   │   │   ├── tags.ts             # Tag management
│   │   │   │   ├── media.ts            # Media management
│   │   │   │   ├── settings.ts         # Settings management
│   │   │   │   ├── analytics.ts        # Analytics endpoints
│   │   │   │   ├── newsletters.ts      # Newsletter management
│   │   │   │   ├── webhooks.ts         # Webhook management
│   │   │   │   └── integrations.ts     # Integration management
│   │   │   ├── members/                # Member portal API
│   │   │   │   ├── auth.ts             # Member authentication
│   │   │   │   ├── profile.ts          # Profile management
│   │   │   │   ├── subscriptions.ts    # Subscription management
│   │   │   │   ├── payments.ts         # Payment management
│   │   │   │   └── preferences.ts      # Member preferences
│   │   │   ├── webhooks/               # Webhook handlers
│   │   │   │   ├── stripe.ts           # Stripe webhooks
│   │   │   │   ├── email.ts            # Email webhooks
│   │   │   │   └── external.ts         # External service webhooks
│   │   │   └── public/                 # Public endpoints
│   │   │       ├── site.ts             # Site information
│   │   │       ├── comments.ts         # Comments system
│   │   │       └── contact.ts          # Contact forms
│   │   └── utils/
│   │       ├── response.ts             # Response helpers
│   │       ├── pagination.ts           # Pagination utilities
│   │       └── filters.ts              # Query filtering
│   ├── services/
│   │   ├── auth/
│   │   │   ├── AuthService.ts          # Authentication service
│   │   │   ├── PermissionService.ts    # Permission management
│   │   │   ├── SessionService.ts       # Session management
│   │   │   └── TokenService.ts         # JWT token management
│   │   ├── content/
│   │   │   ├── PostService.ts          # Post operations
│   │   │   ├── PageService.ts          # Page operations
│   │   │   ├── TagService.ts           # Tag operations
│   │   │   ├── MediaService.ts         # Media operations
│   │   │   └── SearchService.ts        # Search operations
│   │   ├── members/
│   │   │   ├── MemberService.ts        # Member operations
│   │   │   ├── SubscriptionService.ts  # Subscription management
│   │   │   ├── PaymentService.ts       # Payment processing
│   │   │   └── TierService.ts          # Tier management
│   │   ├── email/
│   │   │   ├── EmailService.ts         # Email delivery
│   │   │   ├── NewsletterService.ts    # Newsletter management
│   │   │   ├── TemplateService.ts      # Email templates
│   │   │   └── AnalyticsService.ts     # Email analytics
│   │   ├── storage/
│   │   │   ├── StorageService.ts       # File storage
│   │   │   ├── ImageService.ts         # Image processing
│   │   │   └── CDNService.ts           # CDN management
│   │   ├── analytics/
│   │   │   ├── AnalyticsService.ts     # Analytics collection
│   │   │   ├── ReportingService.ts     # Report generation
│   │   │   └── MetricsService.ts       # Metrics calculation
│   │   ├── integrations/
│   │   │   ├── StripeService.ts        # Stripe integration
│   │   │   ├── AlgoliaService.ts       # Search integration
│   │   │   ├── Smtp2goService.ts       # Email integration
│   │   │   └── WebhookService.ts       # Webhook delivery
│   │   └── cache/
│   │       ├── CacheService.ts         # Caching abstraction
│   │       ├── MemoryCache.ts          # In-memory caching
│   │       └── FirestoreCache.ts       # Firestore caching
│   ├── models/
│   │   ├── base/
│   │   │   ├── BaseModel.ts            # Base model class
│   │   │   ├── BaseRepository.ts       # Base repository class
│   │   │   └── ValidationMixin.ts      # Validation mixin
│   │   ├── Post.ts                     # Post model
│   │   ├── Page.ts                     # Page model
│   │   ├── User.ts                     # User model
│   │   ├── Member.ts                   # Member model
│   │   ├── Tag.ts                      # Tag model
│   │   ├── Newsletter.ts               # Newsletter model
│   │   ├── Comment.ts                  # Comment model
│   │   ├── Subscription.ts             # Subscription model
│   │   ├── Tier.ts                     # Tier model
│   │   ├── Webhook.ts                  # Webhook model
│   │   └── Settings.ts                 # Settings model
│   ├── repositories/
│   │   ├── PostRepository.ts           # Post data access
│   │   ├── PageRepository.ts           # Page data access
│   │   ├── UserRepository.ts           # User data access
│   │   ├── MemberRepository.ts         # Member data access
│   │   ├── TagRepository.ts            # Tag data access
│   │   ├── NewsletterRepository.ts     # Newsletter data access
│   │   ├── CommentRepository.ts        # Comment data access
│   │   ├── SubscriptionRepository.ts   # Subscription data access
│   │   ├── TierRepository.ts           # Tier data access
│   │   ├── WebhookRepository.ts        # Webhook data access
│   │   └── SettingsRepository.ts       # Settings data access
│   ├── utils/
│   │   ├── validation/
│   │   │   ├── schemas.ts              # Joi validation schemas
│   │   │   ├── validators.ts           # Custom validators
│   │   │   └── sanitizers.ts           # Input sanitization
│   │   ├── security/
│   │   │   ├── encryption.ts           # Encryption utilities
│   │   │   ├── hashing.ts              # Password hashing
│   │   │   ├── sanitization.ts         # XSS prevention
│   │   │   └── csrf.ts                 # CSRF protection
│   │   ├── lexical/
│   │   │   ├── parser.ts               # Lexical JSON parser
│   │   │   ├── renderer.ts             # HTML renderer
│   │   │   ├── validator.ts            # Content validation
│   │   │   └── transformer.ts          # Content transformation
│   │   ├── email/
│   │   │   ├── templates.ts            # Email template helpers
│   │   │   ├── tracking.ts             # Email tracking utilities
│   │   │   └── delivery.ts             # Delivery optimization
│   │   ├── image/
│   │   │   ├── processor.ts            # Image processing
│   │   │   ├── optimizer.ts            # Image optimization
│   │   │   └── validator.ts            # Image validation
│   │   ├── slug.ts                     # URL slug generation
│   │   ├── pagination.ts               # Pagination helpers
│   │   ├── date.ts                     # Date utilities
│   │   ├── string.ts                   # String utilities
│   │   ├── crypto.ts                   # Cryptographic utilities
│   │   ├── logger.ts                   # Logging utilities
│   │   └── errors.ts                   # Error definitions
│   ├── types/
│   │   ├── api.ts                      # API type definitions
│   │   ├── models.ts                   # Model interfaces
│   │   ├── services.ts                 # Service interfaces
│   │   ├── auth.ts                     # Authentication types
│   │   ├── email.ts                    # Email types
│   │   ├── payment.ts                  # Payment types
│   │   ├── lexical.ts                  # Lexical types
│   │   └── common.ts                   # Common types
│   └── triggers/
│       ├── auth.ts                     # Auth triggers
│       ├── firestore.ts                # Firestore triggers
│       ├── storage.ts                  # Storage triggers
│       ├── pubsub.ts                   # Pub/Sub triggers
│       └── scheduled.ts                # Scheduled functions
└── test/
    ├── unit/                           # Unit tests
    ├── integration/                    # Integration tests
    └── fixtures/                       # Test data
```

## Frontend Applications Structure

```
apps/
├── admin/                              # Admin Dashboard
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   ├── index.html
│   ├── src/
│   │   ├── main.tsx                    # App entry point
│   │   ├── App.tsx                     # Root component
│   │   ├── components/
│   │   │   ├── common/                 # Shared components
│   │   │   │   ├── Layout.tsx          # Main layout
│   │   │   │   ├── Sidebar.tsx         # Navigation sidebar
│   │   │   │   ├── Header.tsx          # Top header
│   │   │   │   ├── Modal.tsx           # Modal component
│   │   │   │   ├── Button.tsx          # Button component
│   │   │   │   ├── Input.tsx           # Input component
│   │   │   │   ├── Table.tsx           # Table component
│   │   │   │   ├── Pagination.tsx      # Pagination component
│   │   │   │   └── Loading.tsx         # Loading states
│   │   │   ├── editor/                 # Content editor
│   │   │   │   ├── LexicalEditor.tsx   # Main editor
│   │   │   │   ├── Toolbar.tsx         # Editor toolbar
│   │   │   │   ├── plugins/            # Editor plugins
│   │   │   │   └── nodes/              # Custom nodes
│   │   │   ├── media/                  # Media components
│   │   │   │   ├── MediaLibrary.tsx    # Media browser
│   │   │   │   ├── ImageUpload.tsx     # Image uploader
│   │   │   │   └── MediaCard.tsx       # Media item card
│   │   │   ├── posts/                  # Post management
│   │   │   │   ├── PostList.tsx        # Posts listing
│   │   │   │   ├── PostEditor.tsx      # Post editor
│   │   │   │   ├── PostSettings.tsx    # Post settings
│   │   │   │   └── PostPreview.tsx     # Post preview
│   │   │   ├── members/                # Member management
│   │   │   │   ├── MemberList.tsx      # Members listing
│   │   │   │   ├── MemberDetail.tsx    # Member details
│   │   │   │   └── MemberImport.tsx    # Member import
│   │   │   ├── analytics/              # Analytics dashboard
│   │   │   │   ├── Dashboard.tsx       # Main dashboard
│   │   │   │   ├── Charts.tsx          # Chart components
│   │   │   │   └── Metrics.tsx         # Metrics display
│   │   │   ├── settings/               # Settings panels
│   │   │   │   ├── GeneralSettings.tsx # General settings
│   │   │   │   ├── EmailSettings.tsx   # Email configuration
│   │   │   │   ├── PaymentSettings.tsx # Payment setup
│   │   │   │   └── IntegrationSettings.tsx # Integrations
│   │   │   └── newsletters/            # Newsletter management
│   │   │       ├── NewsletterList.tsx  # Newsletter listing
│   │   │       ├── NewsletterEditor.tsx # Newsletter editor
│   │   │       └── NewsletterStats.tsx # Newsletter analytics
│   │   ├── pages/                      # Page components
│   │   │   ├── Dashboard.tsx           # Dashboard page
│   │   │   ├── Posts.tsx               # Posts page
│   │   │   ├── Members.tsx             # Members page
│   │   │   ├── Analytics.tsx           # Analytics page
│   │   │   ├── Settings.tsx            # Settings page
│   │   │   ├── Login.tsx               # Login page
│   │   │   └── Profile.tsx             # User profile
│   │   ├── hooks/                      # Custom React hooks
│   │   │   ├── useAuth.ts              # Authentication hook
│   │   │   ├── useApi.ts               # API hook
│   │   │   ├── usePagination.ts        # Pagination hook
│   │   │   ├── useLocalStorage.ts      # Local storage hook
│   │   │   └── useDebounce.ts          # Debounce hook
│   │   ├── services/                   # API services
│   │   │   ├── api.ts                  # API client
│   │   │   ├── auth.ts                 # Auth service
│   │   │   ├── posts.ts                # Posts API
│   │   │   ├── members.ts              # Members API
│   │   │   ├── media.ts                # Media API
│   │   │   └── analytics.ts            # Analytics API
│   │   ├── store/                      # State management
│   │   │   ├── index.ts                # Store setup
│   │   │   ├── authSlice.ts            # Auth state
│   │   │   ├── postsSlice.ts           # Posts state
│   │   │   ├── membersSlice.ts         # Members state
│   │   │   └── uiSlice.ts              # UI state
│   │   ├── utils/                      # Utility functions
│   │   │   ├── constants.ts            # App constants
│   │   │   ├── helpers.ts              # Helper functions
│   │   │   ├── validation.ts           # Form validation
│   │   │   └── formatting.ts           # Data formatting
│   │   ├── styles/                     # Styling
│   │   │   ├── globals.css             # Global styles
│   │   │   └── components.css          # Component styles
│   │   └── types/                      # TypeScript types
│   │       ├── api.ts                  # API types
│   │       ├── auth.ts                 # Auth types
│   │       └── common.ts               # Common types
│   └── public/                         # Static assets
├── portal/                             # Member Portal
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   ├── index.html
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── components/
│   │   │   ├── auth/                   # Authentication
│   │   │   ├── profile/                # Profile management
│   │   │   ├── subscription/           # Subscription management
│   │   │   ├── billing/                # Billing management
│   │   │   └── common/                 # Shared components
│   │   ├── pages/
│   │   │   ├── Login.tsx
│   │   │   ├── Signup.tsx
│   │   │   ├── Profile.tsx
│   │   │   ├── Subscription.tsx
│   │   │   └── Billing.tsx
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── store/
│   │   ├── utils/
│   │   ├── styles/
│   │   └── types/
│   └── public/
├── website/                            # Public Website
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.js                  # Next.js configuration
│   ├── tailwind.config.js
│   ├── src/
│   │   ├── pages/                      # Next.js pages
│   │   │   ├── index.tsx               # Homepage
│   │   │   ├── posts/
│   │   │   │   ├── [slug].tsx          # Post page
│   │   │   │   └── index.tsx           # Posts listing
│   │   │   ├── tags/
│   │   │   │   └── [slug].tsx          # Tag page
│   │   │   ├── authors/
│   │   │   │   └── [slug].tsx          # Author page
│   │   │   ├── search.tsx              # Search page
│   │   │   └── _app.tsx                # App wrapper
│   │   ├── components/
│   │   │   ├── layout/                 # Layout components
│   │   │   ├── post/                   # Post components
│   │   │   ├── navigation/             # Navigation
│   │   │   ├── search/                 # Search components
│   │   │   └── common/                 # Shared components
│   │   ├── lib/                        # Utilities
│   │   ├── styles/                     # Styling
│   │   └── types/                      # Types
│   └── public/                         # Static assets
├── comments/                           # Comments System
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── components/
│   │   │   ├── CommentForm.tsx
│   │   │   ├── CommentList.tsx
│   │   │   ├── Comment.tsx
│   │   │   └── CommentThread.tsx
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── utils/
│   │   └── types/
│   └── public/
└── search/                             # Search Interface
    ├── package.json
    ├── tsconfig.json
    ├── vite.config.ts
    ├── src/
    │   ├── main.tsx
    │   ├── App.tsx
    │   ├── components/
    │   │   ├── SearchBox.tsx
    │   │   ├── SearchResults.tsx
    │   │   ├── SearchFilters.tsx
    │   │   └── SearchSuggestions.tsx
    │   ├── hooks/
    │   ├── services/
    │   ├── utils/
    │   └── types/
    └── public/
```

## Shared Packages Structure

```
packages/
├── shared/                             # Shared utilities
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── types/                      # Shared TypeScript types
│   │   │   ├── api.ts
│   │   │   ├── models.ts
│   │   │   ├── auth.ts
│   │   │   └── common.ts
│   │   ├── utils/                      # Shared utilities
│   │   │   ├── validation.ts
│   │   │   ├── formatting.ts
│   │   │   ├── constants.ts
│   │   │   └── helpers.ts
│   │   ├── components/                 # Shared React components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   └── Loading.tsx
│   │   └── hooks/                      # Shared React hooks
│   │       ├── useApi.ts
│   │       ├── useAuth.ts
│   │       └── useLocalStorage.ts
│   └── dist/                           # Built package
├── ui/                                 # UI Component Library
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── src/
│   │   ├── components/
│   │   │   ├── forms/
│   │   │   ├── navigation/
│   │   │   ├── feedback/
│   │   │   ├── layout/
│   │   │   └── data-display/
│   │   ├── styles/
│   │   ├── themes/
│   │   └── utils/
│   └── dist/
└── api-client/                         # API Client Library
    ├── package.json
    ├── tsconfig.json
    ├── src/
    │   ├── client.ts                   # Main API client
    │   ├── endpoints/
    │   │   ├── posts.ts
    │   │   ├── members.ts
    │   │   ├── auth.ts
    │   │   └── media.ts
    │   ├── types/
    │   └── utils/
    └── dist/
```

## Configuration Files

```
firebase-blog/
├── firebase.json                       # Firebase configuration
├── .firebaserc                         # Firebase project settings
├── firestore.rules                     # Firestore security rules
├── firestore.indexes.json              # Firestore indexes
├── storage.rules                       # Storage security rules
├── package.json                        # Root package.json
├── tsconfig.json                       # Root TypeScript config
├── .eslintrc.js                        # ESLint configuration
├── .prettierrc                         # Prettier configuration
├── .gitignore                          # Git ignore patterns
├── .env.example                        # Environment variables template
├── docker-compose.yml                  # Local development setup
└── .github/
    └── workflows/
        ├── ci.yml                      # Continuous integration
        ├── deploy.yml                  # Deployment workflow
        └── test.yml                    # Testing workflow
```

## Documentation Structure

```
docs/
├── api/                                # API documentation
│   ├── content-api.md
│   ├── admin-api.md
│   ├── members-api.md
│   └── webhooks.md
├── guides/                             # User guides
│   ├── getting-started.md
│   ├── content-management.md
│   ├── member-management.md
│   ├── email-setup.md
│   └── payment-setup.md
├── development/                        # Developer documentation
│   ├── setup.md
│   ├── architecture.md
│   ├── testing.md
│   ├── deployment.md
│   └── contributing.md
├── migration/                          # Migration guides
│   ├── from-ghost.md
│   ├── from-wordpress.md
│   └── data-import.md
└── troubleshooting/                    # Troubleshooting guides
    ├── common-issues.md
    ├── performance.md
    └── security.md
```

## Testing Structure

```
tests/
├── unit/                               # Unit tests
│   ├── functions/
│   │   ├── services/
│   │   ├── models/
│   │   └── utils/
│   └── apps/
│       ├── admin/
│       ├── portal/
│       └── website/
├── integration/                        # Integration tests
│   ├── api/
│   ├── auth/
│   ├── payments/
│   └── email/
├── e2e/                               # End-to-end tests
│   ├── admin-dashboard/
│   ├── member-portal/
│   ├── public-website/
│   └── comments-system/
├── fixtures/                          # Test data
│   ├── posts.json
│   ├── users.json
│   ├── members.json
│   └── settings.json
├── helpers/                           # Test utilities
│   ├── setup.ts
│   ├── teardown.ts
│   ├── factories.ts
│   └── mocks.ts
└── config/
    ├── jest.config.js
    ├── playwright.config.ts
    └── test-env.ts
```

## Scripts Structure

```
scripts/
├── build/                             # Build scripts
│   ├── build-functions.sh
│   ├── build-apps.sh
│   └── build-all.sh
├── deploy/                            # Deployment scripts
│   ├── deploy-staging.sh
│   ├── deploy-production.sh
│   └── deploy-functions.sh
├── dev/                               # Development scripts
│   ├── start-dev.sh
│   ├── start-emulators.sh
│   └── seed-data.sh
├── test/                              # Testing scripts
│   ├── run-unit-tests.sh
│   ├── run-integration-tests.sh
│   └── run-e2e-tests.sh
├── migration/                         # Migration scripts
│   ├── migrate-from-ghost.js
│   ├── migrate-from-wordpress.js
│   └── import-data.js
└── maintenance/                       # Maintenance scripts
    ├── backup-data.sh
    ├── cleanup-storage.sh
    └── update-indexes.sh
```

This comprehensive project structure provides:

1. **Clear Separation of Concerns**: Backend, frontend, and shared code are properly organized
2. **Scalable Architecture**: Modular structure supports growth and team collaboration
3. **Type Safety**: TypeScript throughout with shared type definitions
4. **Testing Strategy**: Comprehensive testing structure for all components
5. **Documentation**: Complete documentation for users and developers
6. **DevOps Integration**: CI/CD and deployment automation support
7. **Maintainability**: Consistent patterns and clear organization
8. **Flexibility**: Modular packages allow for independent development and deployment

The structure follows modern best practices for full-stack TypeScript applications and provides a solid foundation for implementing the complete Ghost CMS feature set using Firebase.