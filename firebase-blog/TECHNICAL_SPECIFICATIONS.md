# Technical Specifications - Firebase Blog Platform

## Overview

This document provides detailed technical specifications for implementing a complete Ghost CMS equivalent using Firebase infrastructure. These specifications are based on reverse engineering the Ghost repository and adapting the features for Firebase's serverless architecture.

## System Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────────────┐
│                           Client Layer                               │
├─────────────────────┬───────────────────────┬──────────────────────┤
│   Public Website    │    Admin Dashboard    │   Member Portal      │
│  (Firebase Hosting) │    (React/Vue SPA)    │  (Firebase Hosting)  │
└─────────────────────┴───────────────────────┴──────────────────────┘
                                  │
┌─────────────────────────────────────────────────────────────────────┐
│                         API Gateway Layer                            │
├─────────────────────────────────────────────────────────────────────┤
│               Firebase Functions (Node.js Runtime)                   │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────────┐  │
│  │ Content API  │  │  Admin API   │  │  Authentication Service │  │
│  └──────────────┘  └──────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                  │
┌─────────────────────────────────────────────────────────────────────┐
│                         Data Layer                                   │
├────────────────┬────────────────┬──────────────┬───────────────────┤
│   Firestore    │ Firebase Storage│Firebase Auth │  Firebase Cache  │
│   Database     │  (Images/Media) │   (Users)    │   (CDN/Edge)    │
└────────────────┴────────────────┴──────────────┴───────────────────┘
                                  │
┌─────────────────────────────────────────────────────────────────────┐
│                    External Services Layer                           │
├────────────────┬────────────────┬──────────────┬───────────────────┤
│    SMTP2Go     │     Stripe      │   Algolia    │   Social APIs     │
│  (Email/News)  │   (Payments)    │   (Search)   │ (Twitter, etc.)   │
└────────────────┴────────────────┴──────────────┴───────────────────┘
```

## Data Architecture

### Firestore Database Schema

#### Collections Structure
```typescript
interface FirestoreSchema {
  posts: PostCollection;
  pages: PageCollection;
  users: UserCollection;
  members: MemberCollection;
  tags: TagCollection;
  settings: SettingsCollection;
  newsletters: NewsletterCollection;
  webhooks: WebhookCollection;
  images: ImageCollection;
  audit_logs: AuditLogCollection;
  comments: CommentCollection;
  subscriptions: SubscriptionCollection;
  tiers: TierCollection;
  invites: InviteCollection;
  sessions: SessionCollection;
}
```

#### Document Schemas

##### Post Document
```typescript
interface PostDocument {
  // Core fields
  id: string;                          // Auto-generated UUID
  slug: string;                        // URL-friendly identifier
  title: string;                       // Post title
  content: LexicalDocument;            // Lexical JSON structure
  html: string;                        // Rendered HTML (cached)
  excerpt: string;                     // Auto or custom excerpt
  
  // Media
  feature_image: string | null;        // Storage URL
  feature_image_alt: string | null;    
  feature_image_caption: string | null;
  
  // Publishing
  status: 'draft' | 'published' | 'scheduled';
  visibility: 'public' | 'members' | 'paid' | 'tiers';
  featured: boolean;
  published_at: Timestamp | null;
  created_at: Timestamp;
  updated_at: Timestamp;
  
  // Relations
  authors: string[];                   // User IDs
  primary_author: string;              // User ID
  tags: string[];                      // Tag IDs
  primary_tag: string | null;          // Tag ID
  
  // Access Control
  tiers: string[];                     // Tier IDs for access
  
  // SEO & Social
  meta_title: string | null;
  meta_description: string | null;
  og_image: string | null;
  og_title: string | null;
  og_description: string | null;
  twitter_image: string | null;
  twitter_title: string | null;
  twitter_description: string | null;
  canonical_url: string | null;
  
  // Technical
  custom_template: string | null;
  codeinjection_head: string | null;
  codeinjection_foot: string | null;
  reading_time: number;                // Calculated in minutes
  
  // Newsletter
  email_subject: string | null;
  send_email_when_published: boolean;
  newsletter_id: string | null;
  
  // Analytics
  view_count: number;
  like_count: number;
  comment_count: number;
  
  // Internal
  version: number;                     // For optimistic locking
  deleted: boolean;
  deleted_at: Timestamp | null;
}
```

##### User Document (Staff)
```typescript
interface UserDocument {
  id: string;                          // Firebase Auth UID
  email: string;
  name: string;
  slug: string;
  profile_image: string | null;
  cover_image: string | null;
  bio: string | null;
  website: string | null;
  location: string | null;
  
  // Social
  facebook: string | null;
  twitter: string | null;
  
  // Permissions
  role: 'owner' | 'administrator' | 'editor' | 'author' | 'contributor';
  status: 'active' | 'invited' | 'suspended';
  
  // Security
  last_login: Timestamp | null;
  login_attempts: number;
  locked_until: Timestamp | null;
  two_factor_enabled: boolean;
  trusted_devices: DeviceFingerprint[];
  
  // Metadata
  created_at: Timestamp;
  updated_at: Timestamp;
  invited_by: string | null;           // User ID
}
```

##### Member Document
```typescript
interface MemberDocument {
  id: string;                          // Firebase Auth UID
  email: string;
  name: string | null;
  
  // Subscription
  status: 'free' | 'paid' | 'complimentary';
  stripe_customer_id: string | null;
  
  // Subscriptions (subcollection)
  subscriptions: {
    [key: string]: {
      tier_id: string;
      stripe_subscription_id: string;
      status: 'active' | 'canceled' | 'past_due';
      current_period_end: Timestamp;
      cancel_at_period_end: boolean;
    }
  };
  
  // Preferences
  subscribed_to_emails: boolean;
  newsletter_subscriptions: string[];   // Newsletter IDs
  
  // Metadata
  note: string | null;                 // Admin notes
  labels: string[];                    // Custom labels
  created_at: Timestamp;
  last_seen: Timestamp | null;
  
  // Geolocation (for analytics)
  geolocation: {
    country: string | null;
    region: string | null;
  };
  
  // Analytics
  total_posts_viewed: number;
  total_time_spent: number;           // in seconds
  last_post_viewed: string | null;
}
```

##### Tag Document
```typescript
interface TagDocument {
  id: string;
  name: string;
  slug: string;
  description: string | null;
  feature_image: string | null;
  visibility: 'public' | 'internal';
  
  // SEO
  meta_title: string | null;
  meta_description: string | null;
  og_image: string | null;
  og_title: string | null;
  og_description: string | null;
  twitter_image: string | null;
  twitter_title: string | null;
  twitter_description: string | null;
  
  // Analytics
  post_count: number;
  
  // Metadata
  created_at: Timestamp;
  updated_at: Timestamp;
}
```

##### Newsletter Document
```typescript
interface NewsletterDocument {
  id: string;
  name: string;
  description: string | null;
  sender_name: string;
  sender_email: string;
  sender_reply_to: string;
  
  // Design
  header_image: string | null;
  show_header_icon: boolean;
  show_header_title: boolean;
  title_font_category: string;
  title_alignment: 'left' | 'center';
  show_feature_image: boolean;
  body_font_category: string;
  footer_content: string | null;
  
  // Settings
  status: 'active' | 'archived';
  subscribe_on_signup: boolean;
  sort_order: number;
  
  // Analytics
  subscriber_count: number;
  open_rate: number;
  click_rate: number;
  
  // Metadata
  created_at: Timestamp;
  updated_at: Timestamp;
}
```

##### Comment Document
```typescript
interface CommentDocument {
  id: string;
  post_id: string;
  member_id: string | null;           // null for anonymous
  parent_id: string | null;           // for replies
  
  // Content
  html: string;
  status: 'published' | 'hidden' | 'deleted';
  
  // Member info (cached)
  member_name: string | null;
  member_email: string | null;
  member_avatar: string | null;
  
  // Analytics
  like_count: number;
  reply_count: number;
  
  // Metadata
  created_at: Timestamp;
  updated_at: Timestamp;
  edited_at: Timestamp | null;
}
```

### Lexical Document Format
```typescript
interface LexicalDocument {
  root: {
    children: LexicalNode[];
    direction: 'ltr' | 'rtl' | null;
    format: number;
    indent: number;
    type: 'root';
    version: number;
  };
}

interface LexicalNode {
  type: string;
  version: number;
  [key: string]: any;
}

// Specific node types
interface ParagraphNode extends LexicalNode {
  type: 'paragraph';
  children: LexicalNode[];
  format: number;
  indent: number;
  direction: 'ltr' | 'rtl' | null;
}

interface TextNode extends LexicalNode {
  type: 'text';
  text: string;
  format: number;
  style: string;
  mode: 'normal' | 'token' | 'segmented';
  detail: number;
}

interface ImageNode extends LexicalNode {
  type: 'image';
  src: string;
  alt: string;
  width?: number;
  height?: number;
  caption?: string;
}
```

## API Specifications

### Content API (Public)

#### Authentication
```typescript
// API Key Authentication Header
Headers: {
  'Authorization': 'Bearer {CONTENT_API_KEY}'
}
```

#### Endpoints

##### GET /api/content/posts
```typescript
interface GetPostsRequest {
  // Pagination
  page?: number;              // Default: 1
  limit?: number;             // Default: 15, Max: 100
  
  // Filtering
  filter?: string;            // NQL query string
  fields?: string;            // Comma-separated fields
  formats?: string;           // 'html' | 'plaintext' | 'lexical'
  
  // Includes
  include?: string;           // 'authors,tags,tiers'
  
  // Ordering
  order?: string;             // 'published_at desc'
}

interface GetPostsResponse {
  posts: PostDocument[];
  meta: {
    pagination: {
      page: number;
      limit: number;
      pages: number;
      total: number;
      next: number | null;
      prev: number | null;
    };
  };
}
```

##### GET /api/content/posts/:slug
```typescript
interface GetPostResponse {
  post: PostDocument;
}
```

##### GET /api/content/tags
```typescript
interface GetTagsResponse {
  tags: TagDocument[];
  meta: PaginationMeta;
}
```

##### GET /api/content/authors
```typescript
interface GetAuthorsResponse {
  authors: UserDocument[];
  meta: PaginationMeta;
}
```

### Admin API (Authenticated)

#### Authentication Flow
```typescript
// JWT Authentication
interface AdminAuthToken {
  uid: string;               // User ID
  email: string;
  role: string;
  exp: number;               // Expiration timestamp
  iat: number;               // Issued at timestamp
  device_id: string;         // Device fingerprint
}
```

#### Post Management Endpoints

##### POST /api/admin/posts
```typescript
interface CreatePostRequest {
  post: {
    title: string;
    content: LexicalDocument;
    status?: 'draft' | 'published';
    authors?: string[];
    tags?: string[];
    visibility?: 'public' | 'members' | 'paid' | 'tiers';
    featured?: boolean;
    feature_image?: string;
    excerpt?: string;
    meta_title?: string;
    meta_description?: string;
    og_image?: string;
    og_title?: string;
    og_description?: string;
    twitter_image?: string;
    twitter_title?: string;
    twitter_description?: string;
    canonical_url?: string;
    codeinjection_head?: string;
    codeinjection_foot?: string;
    custom_template?: string;
    published_at?: string;
    tiers?: string[];
  };
}

interface CreatePostResponse {
  post: PostDocument;
}
```

##### PUT /api/admin/posts/:id
```typescript
interface UpdatePostRequest {
  post: Partial<PostDocument>;
}

interface UpdatePostResponse {
  post: PostDocument;
}
```

##### DELETE /api/admin/posts/:id
```typescript
interface DeletePostResponse {
  success: boolean;
}
```

#### User Management Endpoints

##### POST /api/admin/users
```typescript
interface CreateUserRequest {
  user: {
    email: string;
    name: string;
    role: UserRole;
  };
}

interface CreateUserResponse {
  user: UserDocument;
  invite_link: string;
}
```

##### PUT /api/admin/users/:id
```typescript
interface UpdateUserRequest {
  user: Partial<UserDocument>;
}

interface UpdateUserResponse {
  user: UserDocument;
}
```

#### Member Management Endpoints

##### GET /api/admin/members
```typescript
interface GetMembersRequest {
  page?: number;
  limit?: number;
  filter?: string;
  search?: string;
  order?: string;
}

interface GetMembersResponse {
  members: MemberDocument[];
  meta: PaginationMeta;
}
```

##### POST /api/admin/members
```typescript
interface CreateMemberRequest {
  member: {
    email: string;
    name?: string;
    note?: string;
    labels?: string[];
    subscribed_to_emails?: boolean;
  };
}

interface CreateMemberResponse {
  member: MemberDocument;
}
```

### Members API (Member Portal)

#### Authentication
```typescript
// Member JWT Token
interface MemberAuthToken {
  uid: string;
  email: string;
  subscription_status: string;
  exp: number;
  iat: number;
}
```

#### Endpoints

##### POST /api/members/auth/signin
```typescript
interface MemberSigninRequest {
  email: string;
  password: string;
}

interface MemberSigninResponse {
  member: MemberDocument;
  token: string;
}
```

##### GET /api/members/me
```typescript
interface GetMemberResponse {
  member: MemberDocument;
}
```

##### PUT /api/members/me
```typescript
interface UpdateMemberRequest {
  member: {
    name?: string;
    subscribed_to_emails?: boolean;
  };
}

interface UpdateMemberResponse {
  member: MemberDocument;
}
```

##### GET /api/members/subscriptions
```typescript
interface GetSubscriptionsResponse {
  subscriptions: SubscriptionDocument[];
}
```

## Security Architecture

### Firebase Security Rules

#### Firestore Security Rules
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Public read for published content
    match /posts/{postId} {
      allow read: if resource.data.status == 'published' 
        && resource.data.visibility == 'public'
        && resource.data.published_at <= request.time;
      
      allow read: if resource.data.visibility == 'members' 
        && request.auth != null
        && exists(/databases/$(database)/documents/members/$(request.auth.uid));
      
      allow read: if resource.data.visibility == 'paid'
        && request.auth != null
        && exists(/databases/$(database)/documents/members/$(request.auth.uid))
        && get(/databases/$(database)/documents/members/$(request.auth.uid)).data.status in ['paid', 'complimentary'];
      
      allow write: if request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['owner', 'administrator', 'editor', 'author'];
    }
    
    // Staff user access
    match /users/{userId} {
      allow read: if request.auth.uid == userId;
      allow write: if request.auth.uid == userId 
        && request.resource.data.role == resource.data.role; // Prevent role escalation
    }
    
    // Member access
    match /members/{memberId} {
      allow read, write: if request.auth.uid == memberId;
    }
    
    // Comments
    match /comments/{commentId} {
      allow read: if resource.data.status == 'published';
      allow create: if request.auth != null;
      allow update, delete: if request.auth.uid == resource.data.member_id
        || get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['owner', 'administrator', 'editor'];
    }
  }
}
```

#### Firebase Storage Security Rules
```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    // Images
    match /images/{imageId} {
      allow read: if true;
      allow write: if request.auth != null
        && get(/firestore/databases/(default)/documents/users/$(request.auth.uid)).data.role in ['owner', 'administrator', 'editor', 'author'];
    }
    
    // Member avatars
    match /avatars/{memberId} {
      allow read: if true;
      allow write: if request.auth.uid == memberId;
    }
  }
}
```

### Input Validation Service
```typescript
import * as Joi from 'joi';

class ValidationService {
  private static postSchema = Joi.object({
    title: Joi.string().required().max(255),
    content: Joi.object().required(),
    slug: Joi.string().pattern(/^[a-z0-9\-]+$/).max(191),
    status: Joi.string().valid('draft', 'published', 'scheduled'),
    visibility: Joi.string().valid('public', 'members', 'paid', 'tiers'),
    tags: Joi.array().items(Joi.string()),
    authors: Joi.array().items(Joi.string()).min(1),
    featured: Joi.boolean(),
    excerpt: Joi.string().max(500),
    meta_title: Joi.string().max(300),
    meta_description: Joi.string().max(500),
    published_at: Joi.date().iso(),
    tiers: Joi.array().items(Joi.string())
  });
  
  private static memberSchema = Joi.object({
    email: Joi.string().email().required(),
    name: Joi.string().max(191),
    note: Joi.string().max(500),
    labels: Joi.array().items(Joi.string()),
    subscribed_to_emails: Joi.boolean()
  });
  
  static validatePost(data: any): ValidationResult {
    return this.postSchema.validate(data, { 
      abortEarly: false,
      stripUnknown: true 
    });
  }
  
  static validateMember(data: any): ValidationResult {
    return this.memberSchema.validate(data, {
      abortEarly: false,
      stripUnknown: true
    });
  }
}
```

### XSS Prevention
```typescript
import * as DOMPurify from 'isomorphic-dompurify';

class SecurityService {
  static sanitizeHTML(html: string): string {
    return DOMPurify.sanitize(html, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'blockquote', 
                     'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
                     'pre', 'code', 'img', 'figure', 'figcaption', 'iframe'],
      ALLOWED_ATTR: ['href', 'src', 'alt', 'title', 'class', 'id', 'width', 'height'],
      ALLOWED_URI_REGEXP: /^(?:(?:(?:f|ht)tps?|mailto|tel|callto|cid|xmpp):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i
    });
  }
  
  static escapeString(str: string): string {
    const map: Record<string, string> = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#39;',
      '/': '&#x2F;'
    };
    return str.replace(/[&<>"'\/]/g, (s) => map[s]);
  }
}
```

### Rate Limiting Implementation
```typescript
import * as rateLimit from 'express-rate-limit';

const createRateLimiter = (windowMs: number, max: number, message: string) => {
  return rateLimit({
    windowMs,
    max,
    keyGenerator: (req) => req.ip,
    handler: (req, res) => {
      res.status(429).json({
        error: message,
        retry_after: req.rateLimit.resetTime
      });
    }
  });
};

// Different rate limits for different endpoints
const loginLimiter = createRateLimiter(
  60 * 60 * 1000,  // 1 hour
  5,               // 5 attempts
  'Too many login attempts'
);

const apiLimiter = createRateLimiter(
  15 * 60 * 1000,  // 15 minutes
  100,             // 100 requests
  'Too many API requests'
);

const contentLimiter = createRateLimiter(
  60 * 1000,       // 1 minute
  60,              // 60 requests
  'Too many content requests'
);
```

## Performance & Scalability

### Caching Strategy

#### Multi-Layer Cache Architecture
```typescript
enum CacheLayer {
  EDGE = 'firebase-hosting-cache',      // 24 hours
  CDN = 'firebase-cdn',                 // 1 hour  
  FUNCTION_MEMORY = 'in-memory',        // 5 minutes
  FIRESTORE = 'firestore-cache'         // 30 minutes
}

class CacheService {
  private memoryCache = new Map<string, CachedItem>();
  
  async get(key: string, layers: CacheLayer[]): Promise<any> {
    // Check memory cache first
    if (layers.includes(CacheLayer.FUNCTION_MEMORY)) {
      const memItem = this.memoryCache.get(key);
      if (memItem && memItem.expires > Date.now()) {
        return memItem.value;
      }
    }
    
    // Check Firestore cache
    if (layers.includes(CacheLayer.FIRESTORE)) {
      const doc = await db.collection('cache').doc(key).get();
      if (doc.exists) {
        const data = doc.data();
        if (data.expires > Date.now()) {
          // Populate memory cache
          this.memoryCache.set(key, data);
          return data.value;
        }
      }
    }
    
    return null;
  }
  
  async set(key: string, value: any, ttl: number, layers: CacheLayer[]): Promise<void> {
    const expires = Date.now() + ttl * 1000;
    const cacheItem = { value, expires };
    
    if (layers.includes(CacheLayer.FUNCTION_MEMORY)) {
      this.memoryCache.set(key, cacheItem);
    }
    
    if (layers.includes(CacheLayer.FIRESTORE)) {
      await db.collection('cache').doc(key).set(cacheItem);
    }
  }
  
  async invalidate(pattern: string): Promise<void> {
    // Invalidate memory cache
    for (const key of this.memoryCache.keys()) {
      if (key.includes(pattern)) {
        this.memoryCache.delete(key);
      }
    }
    
    // Invalidate Firestore cache
    const snapshot = await db.collection('cache')
      .where('key', '>=', pattern)
      .where('key', '<', pattern + '\uf8ff')
      .get();
    
    const batch = db.batch();
    snapshot.docs.forEach(doc => batch.delete(doc.ref));
    await batch.commit();
  }
}
```

#### Cache Headers Configuration
```typescript
// Firebase Hosting Configuration (firebase.json)
{
  "hosting": {
    "headers": [
      {
        "source": "/api/content/**",
        "headers": [{
          "key": "Cache-Control",
          "value": "public, max-age=300, s-maxage=600"
        }]
      },
      {
        "source": "/images/**",
        "headers": [{
          "key": "Cache-Control",
          "value": "public, max-age=31536000"
        }]
      },
      {
        "source": "**/*.@(js|css)",
        "headers": [{
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }]
      }
    ]
  }
}
```

### Database Optimization

#### Composite Indexes
```javascript
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "visibility", "order": "ASCENDING" },
        { "fieldPath": "published_at", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "tags", "arrayConfig": "CONTAINS" },
        { "fieldPath": "published_at", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "authors", "arrayConfig": "CONTAINS" },
        { "fieldPath": "published_at", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "members",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "created_at", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "comments",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "post_id", "order": "ASCENDING" },
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "created_at", "order": "ASCENDING" }
      ]
    }
  ]
}
```

## Email & Newsletter System

### Email Service Architecture
```typescript
interface EmailService {
  sendTransactional(email: TransactionalEmail): Promise<void>;
  sendNewsletter(newsletter: Newsletter): Promise<void>;
  trackOpen(emailId: string, memberId: string): Promise<void>;
  trackClick(emailId: string, memberId: string, url: string): Promise<void>;
}

class Smtp2goEmailService implements EmailService {
  private smtp2go: Smtp2go;
  
  constructor(config: Smtp2goConfig) {
    this.smtp2go = new Smtp2go({
      apiKey: config.apiKey,
      domain: config.domain
    });
  }
  
  async sendNewsletter(newsletter: Newsletter): Promise<void> {
    // Batch members into groups of 1000
    const batches = chunk(newsletter.recipients, 1000);
    
    for (const batch of batches) {
      await this.smtp2go.messages.create({
        from: newsletter.from,
        to: batch.map(r => r.email),
        subject: newsletter.subject,
        html: this.renderTemplate(newsletter.template, newsletter.data),
        'o:tag': ['newsletter', newsletter.id],
        'o:tracking': true,
        'o:tracking-clicks': true,
        'o:tracking-opens': true
      });
    }
  }
  
  async sendTransactional(email: TransactionalEmail): Promise<void> {
    await this.smtp2go.messages.create({
      from: email.from,
      to: email.to,
      subject: email.subject,
      html: email.html,
      'o:tag': ['transactional', email.type]
    });
  }
  
  async trackOpen(emailId: string, memberId: string): Promise<void> {
    await db.collection('email_events').add({
      email_id: emailId,
      member_id: memberId,
      event_type: 'opened',
      timestamp: Timestamp.now()
    });
  }
  
  async trackClick(emailId: string, memberId: string, url: string): Promise<void> {
    await db.collection('email_events').add({
      email_id: emailId,
      member_id: memberId,
      event_type: 'clicked',
      url: url,
      timestamp: Timestamp.now()
    });
  }
}
```

### Newsletter Template System
```typescript
interface NewsletterTemplate {
  id: string;
  name: string;
  html: string;
  css: string;
  preheader: string;
  
  // Personalization tokens
  tokens: {
    unsubscribe_url: string;
    member_name: string;
    member_email: string;
    post_title: string;
    post_content: string;
    post_url: string;
    site_title: string;
    site_url: string;
  };
}

class TemplateRenderer {
  render(template: NewsletterTemplate, data: any): string {
    let html = template.html;
    
    // Replace tokens
    Object.entries(template.tokens).forEach(([token, value]) => {
      const regex = new RegExp(`{{${token}}}`, 'g');
      html = html.replace(regex, data[token] || '');
    });
    
    // Process conditional blocks
    html = this.processConditionals(html, data);
    
    // Process loops
    html = this.processLoops(html, data);
    
    return html;
  }
  
  private processConditionals(html: string, data: any): string {
    const conditionalRegex = /{{#if\s+(\w+)}}(.*?){{\/if}}/gs;
    return html.replace(conditionalRegex, (match, condition, content) => {
      return data[condition] ? content : '';
    });
  }
  
  private processLoops(html: string, data: any): string {
    const loopRegex = /{{#each\s+(\w+)}}(.*?){{\/each}}/gs;
    return html.replace(loopRegex, (match, arrayName, content) => {
      const array = data[arrayName] || [];
      return array.map((item: any) => {
        let itemContent = content;
        Object.entries(item).forEach(([key, value]) => {
          const regex = new RegExp(`{{${key}}}`, 'g');
          itemContent = itemContent.replace(regex, value as string);
        });
        return itemContent;
      }).join('');
    });
  }
}
```

## Image Processing & Storage

### Image Upload Pipeline
```typescript
class ImageProcessingService {
  async processUpload(file: Express.Multer.File): Promise<ProcessedImage> {
    // Validate file
    this.validateImage(file);
    
    // Generate unique filename
    const filename = this.generateFilename(file);
    
    // Process image variants
    const variants = await Promise.all([
      this.createVariant(file.buffer, { width: 2000, quality: 80 }), // Original
      this.createVariant(file.buffer, { width: 1000, quality: 80 }), // Large
      this.createVariant(file.buffer, { width: 600, quality: 80 }),  // Medium
      this.createVariant(file.buffer, { width: 300, quality: 80 })   // Thumbnail
    ]);
    
    // Upload to Firebase Storage
    const urls = await Promise.all(
      variants.map((variant, index) => 
        this.uploadToStorage(variant, `${filename}_${this.getSizeSuffix(index)}.webp`)
      )
    );
    
    // Store metadata in Firestore
    await this.storeImageMetadata({
      id: filename,
      original_url: urls[0],
      variants: {
        large: urls[1],
        medium: urls[2],
        thumbnail: urls[3]
      },
      size: file.size,
      mimetype: file.mimetype,
      uploaded_at: Timestamp.now()
    });
    
    return { id: filename, urls };
  }
  
  private async createVariant(buffer: Buffer, options: ImageOptions): Promise<Buffer> {
    return sharp(buffer)
      .resize(options.width, null, { 
        withoutEnlargement: true,
        fit: 'inside' 
      })
      .webp({ quality: options.quality })
      .toBuffer();
  }
  
  private validateImage(file: Express.Multer.File): void {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
    if (!allowedTypes.includes(file.mimetype)) {
      throw new Error('Invalid file type');
    }
    
    const maxSize = 10 * 1024 * 1024; // 10MB
    if (file.size > maxSize) {
      throw new Error('File too large');
    }
  }
  
  private generateFilename(file: Express.Multer.File): string {
    const timestamp = Date.now();
    const random = Math.random().toString(36).substring(2);
    const ext = path.extname(file.originalname);
    return `${timestamp}_${random}${ext}`;
  }
  
  private async uploadToStorage(buffer: Buffer, path: string): Promise<string> {
    const bucket = admin.storage().bucket();
    const file = bucket.file(`images/${path}`);
    
    await file.save(buffer, {
      metadata: {
        contentType: 'image/webp',
        cacheControl: 'public, max-age=31536000'
      }
    });
    
    await file.makePublic();
    return `https://storage.googleapis.com/${bucket.name}/images/${path}`;
  }
}
```

## Search Implementation

### Search Architecture Using Algolia
```typescript
interface SearchService {
  indexPost(post: PostDocument): Promise<void>;
  search(query: string, options: SearchOptions): Promise<SearchResults>;
  deletePost(postId: string): Promise<void>;
}

class AlgoliaSearchService implements SearchService {
  private index: SearchIndex;
  
  constructor(config: AlgoliaConfig) {
    const client = algoliasearch(config.appId, config.apiKey);
    this.index = client.initIndex(config.indexName);
  }
  
  async indexPost(post: PostDocument): Promise<void> {
    const searchObject = {
      objectID: post.id,
      title: post.title,
      excerpt: post.excerpt,
      content: this.extractTextFromLexical(post.content),
      authors: await this.getAuthorNames(post.authors),
      tags: await this.getTagNames(post.tags),
      published_at: post.published_at?.toMillis(),
      url: `/posts/${post.slug}`,
      visibility: post.visibility,
      _tags: [...post.tags, ...post.authors] // For filtering
    };
    
    await this.index.saveObject(searchObject);
  }
  
  async search(query: string, options: SearchOptions): Promise<SearchResults> {
    const searchParams = {
      query,
      hitsPerPage: options.limit || 20,
      page: options.page || 0,
      attributesToRetrieve: ['title', 'excerpt', 'url', 'published_at', 'authors', 'tags'],
      attributesToHighlight: ['title', 'excerpt'],
      filters: this.buildFilters(options),
      facets: ['tags', 'authors'],
      maxValuesPerFacet: 10
    };
    
    return await this.index.search(searchParams);
  }
  
  async deletePost(postId: string): Promise<void> {
    await this.index.deleteObject(postId);
  }
  
  private extractTextFromLexical(content: LexicalDocument): string {
    // Extract plain text from Lexical document
    const extractText = (node: any): string => {
      if (node.type === 'text') {
        return node.text;
      }
      if (node.children) {
        return node.children.map(extractText).join(' ');
      }
      return '';
    };
    
    return content.root.children.map(extractText).join(' ');
  }
  
  private buildFilters(options: SearchOptions): string {
    const filters = [];
    
    if (options.tags) {
      filters.push(`tags:${options.tags.join(' OR tags:')}`);
    }
    
    if (options.authors) {
      filters.push(`authors:${options.authors.join(' OR authors:')}`);
    }
    
    if (options.visibility) {
      filters.push(`visibility:${options.visibility}`);
    }
    
    return filters.join(' AND ');
  }
}
```

## Deployment Architecture

### Firebase Configuration
```json
// firebase.json
{
  "hosting": {
    "public": "public",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "/api/**",
        "function": "api"
      },
      {
        "source": "/admin/**",
        "destination": "/admin/index.html"
      },
      {
        "source": "/portal/**",
        "destination": "/portal/index.html"
      },
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(jpg|jpeg|gif|png|webp)",
        "headers": [{
          "key": "Cache-Control",
          "value": "public, max-age=31536000"
        }]
      },
      {
        "source": "**/*.@(js|css)",
        "headers": [{
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }]
      }
    ]
  },
  "functions": {
    "runtime": "nodejs18",
    "region": "us-central1",
    "memory": "512MB",
    "maxInstances": 100,
    "minInstances": 1,
    "source": "functions"
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "storage": {
    "rules": "storage.rules"
  }
}
```

### Environment Configuration
```typescript
// Environment Variables (Firebase Functions Config)
interface EnvironmentConfig {
  // Core
  site: {
    url: string;
    title: string;
    description: string;
  };
  
  // Authentication
  auth: {
    jwt_secret: string;
    session_secret: string;
    content_api_key: string;
    admin_api_key: string;
  };
  
  // Email
  smtp2go: {
    api_key: string;
    domain: string;
    from_address: string;
  };
  
  // Payment
  stripe: {
    secret_key: string;
    webhook_secret: string;
    public_key: string;
  };
  
  // Search
  algolia: {
    app_id: string;
    api_key: string;
    index_name: string;
  };
  
  // Storage
  storage: {
    bucket: string;
    cdn_url: string;
  };
  
  // External APIs
  social: {
    twitter_api_key: string;
    facebook_app_id: string;
  };
}
```

This comprehensive technical specification provides the foundation for implementing a complete Ghost CMS equivalent using Firebase infrastructure. Each section includes detailed interfaces, implementation patterns, and configuration examples that maintain feature parity with Ghost while leveraging Firebase's serverless capabilities.