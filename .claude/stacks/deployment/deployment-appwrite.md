# Appwrite - Backend-as-a-Service Platform

**Purpose:** Complete backend platform with authentication, databases, storage, functions, and messaging

**Last Updated:** 2025-12-10

**Official Docs:** https://appwrite.io/docs

---

## What is Appwrite?

Appwrite is a self-hosted or cloud-hosted Backend-as-a-Service (BaaS) platform providing:
- **Authentication** - 9+ auth methods including OAuth, email/password, phone, magic URL
- **Databases** - Structured data storage with queries, relationships, and permissions
- **Storage** - File management with image transformations
- **Functions** - Serverless functions with multiple runtime support
- **Messaging** - Email, SMS, and push notifications
- **Realtime** - WebSocket-based event streaming
- **Sites** - Website deployment infrastructure

---

## When to Use Appwrite

### Use Appwrite When:
- Need a **complete backend solution** without building from scratch
- Want **self-hosting option** for data sovereignty
- Building apps requiring **multiple auth methods**
- Need **realtime updates** across clients
- Want **file storage with image transformations**
- Building **cross-platform apps** (web, mobile, desktop)

### Consider Alternatives When:
- Need **fine-grained SQL control** (use Postgres/Supabase)
- Building **simple static sites** (use Vercel/Netlify alone)
- Need **specific cloud provider integrations** (use Firebase/AWS)
- Require **complex serverless orchestration** (use AWS Lambda/Step Functions)

---

## Quick Start Guides

### TanStack Start

```bash
# 1. Create project
npm create @tanstack/start@latest my-app && cd my-app

# 2. Install SDK
npm install appwrite
```

```typescript
// src/utils/appwrite.ts
import { Client, Account, ID, Models } from 'appwrite'

export const client = new Client()
  .setEndpoint('https://cloud.appwrite.io/v1')
  .setProject('<PROJECT_ID>')

export const account = new Account(client)
export { ID }
export type { Models }
```

```typescript
// src/routes/index.tsx
import { account, ID } from '../utils/appwrite'
import { useState } from 'react'

export default function Home() {
  const [user, setUser] = useState(null)
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [name, setName] = useState('')

  async function login(email: string, password: string) {
    await account.createEmailPasswordSession(email, password)
    setUser(await account.get())
  }

  async function register(email: string, password: string, name: string) {
    await account.create(ID.unique(), email, password, name)
    await login(email, password)
  }

  async function logout() {
    await account.deleteSession('current')
    setUser(null)
  }

  // ... render form
}
```

Run with `npm run dev` → `localhost:3000`

---

### Next.js

```bash
# 1. Create project
npx create-next-app@latest my-app && cd my-app
# Use: TypeScript (optional), ESLint (yes), App Router (yes)

# 2. Install SDK
npm install appwrite
```

```javascript
// app/appwrite.js
import { Client, Account } from 'appwrite'

export const client = new Client()
  .setEndpoint('https://cloud.appwrite.io/v1')
  .setProject('<PROJECT_ID>')

export const account = new Account(client)
export { ID } from 'appwrite'
```

```javascript
// app/page.js
'use client'
import { useState } from 'react'
import { account, ID } from './appwrite'

export default function Home() {
  const [user, setUser] = useState(null)
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [name, setName] = useState('')

  async function login(email, password) {
    await account.createEmailPasswordSession(email, password)
    setUser(await account.get())
  }

  async function register(email, password, name) {
    await account.create(ID.unique(), email, password, name)
    await login(email, password)
  }

  async function logout() {
    await account.deleteSession('current')
    setUser(null)
  }

  // ... render form
}
```

Run with `npm run dev` → `localhost:3000`

---

### React Native (Expo)

```bash
# 1. Create project
npx create-expo-app my-app && cd my-app

# 2. Install SDK
npx expo install react-native-appwrite react-native-url-polyfill
```

**Console Setup:**
- Add Android platform: app name + package name (from `build.gradle`)
- Add iOS platform: app name + Bundle ID (from Xcode)

```javascript
// lib/appwrite.js
import { Client, Account, ID } from 'react-native-appwrite'

const client = new Client()
  .setEndpoint('https://cloud.appwrite.io/v1')
  .setProject('<PROJECT_ID>')
  .setPlatform('com.example.myapp') // Your bundle ID / package name

export const account = new Account(client)
export { ID }
```

```javascript
// App.js
import { useState } from 'react'
import { account, ID } from './lib/appwrite'

export default function App() {
  const [user, setUser] = useState(null)

  async function login(email, password) {
    await account.createEmailPasswordSession(email, password)
    setUser(await account.get())
  }

  async function register(email, password, name) {
    await account.create(ID.unique(), email, password, name)
    await login(email, password)
  }

  async function logout() {
    await account.deleteSession('current')
    setUser(null)
  }

  // ... render UI
}
```

Run with `npx expo start`

---

### Android (Kotlin)

**1. Add dependency** in `app/build.gradle.kts`:
```kotlin
implementation("io.appwrite:sdk-for-android:8.1.0")
```

**2. Configure OAuth callback** in `AndroidManifest.xml`:
```xml
<activity android:name="io.appwrite.views.CallbackActivity" android:exported="true">
  <intent-filter android:label="android_web_auth">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="appwrite-callback-<PROJECT_ID>" />
  </intent-filter>
</activity>
```

**3. Create Appwrite singleton:**
```kotlin
// Appwrite.kt
object Appwrite {
    private const val ENDPOINT = "https://cloud.appwrite.io/v1"
    private const val PROJECT_ID = "<PROJECT_ID>"

    private lateinit var client: Client
    private lateinit var account: Account

    fun init(context: Context) {
        client = Client(context)
            .setEndpoint(ENDPOINT)
            .setProject(PROJECT_ID)
        account = Account(client)
    }

    suspend fun login(email: String, password: String): Session {
        return account.createEmailPasswordSession(email, password)
    }

    suspend fun register(email: String, password: String, name: String): User<Map<String, Any>> {
        return account.create(ID.unique(), email, password, name)
    }

    suspend fun logout() {
        account.deleteSession("current")
    }

    suspend fun getUser(): User<Map<String, Any>> {
        return account.get()
    }
}
```

---

### Apple (iOS/macOS - Swift)

**1. Add SDK via Swift Package Manager:**
- Package URL: `https://github.com/appwrite/sdk-for-apple`
- Version: 10.1.0 (Up to Next Major)

**2. Configure OAuth** in `Info.plist`:
```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>appwrite-callback-<PROJECT_ID></string>
    </array>
  </dict>
</array>
```

**3. Handle OAuth callback** (UIKit only) in `SceneDelegate.swift`:
```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    WebAuthComponent.handleIncomingCookie(from: url)
}
```

**4. Create Appwrite class:**
```swift
// Appwrite.swift
import Appwrite

class Appwrite {
    static let shared = Appwrite()

    let client = Client()
        .setEndpoint("https://cloud.appwrite.io/v1")
        .setProject("<PROJECT_ID>")

    lazy var account = Account(client)

    func register(email: String, password: String, name: String) async throws -> User<[String: AnyCodable]> {
        try await account.create(userId: ID.unique(), email: email, password: password, name: name)
    }

    func login(email: String, password: String) async throws -> Session {
        try await account.createEmailPasswordSession(email: email, password: password)
    }

    func logout() async throws {
        _ = try await account.deleteSession(sessionId: "current")
    }

    func getUser() async throws -> User<[String: AnyCodable]> {
        try await account.get()
    }
}
```

---

### Node.js (Server SDK)

```bash
# 1. Create project
mkdir my-app && cd my-app
npm init -y

# 2. Install SDK
npm install node-appwrite
```

**Generate API Key** in Console with scopes:
- `databases.write`, `collections.write`, `documents.read`, `documents.write`

```javascript
// app.js
const sdk = require('node-appwrite')

const client = new sdk.Client()
  .setEndpoint('https://cloud.appwrite.io/v1')
  .setProject('<PROJECT_ID>')
  .setKey('<API_KEY>')

const databases = new sdk.Databases(client)
const users = new sdk.Users(client)

// Create database and collection
async function setup() {
  await databases.create('main', 'Main Database')
  await databases.createCollection('main', 'todos', 'Todos')
  await databases.createStringAttribute('main', 'todos', 'title', 255, true)
  await databases.createBooleanAttribute('main', 'todos', 'completed', true)
}

// CRUD operations
async function createTodo(title) {
  return await databases.createDocument(
    'main',
    'todos',
    sdk.ID.unique(),
    { title, completed: false }
  )
}

async function listTodos() {
  return await databases.listDocuments('main', 'todos')
}

async function updateTodo(id, completed) {
  return await databases.updateDocument('main', 'todos', id, { completed })
}

async function deleteTodo(id) {
  await databases.deleteDocument('main', 'todos', id)
}

// TypeScript support
interface Todo {
  title: string
  completed: boolean
}

const todos = await databases.listDocuments<Todo>('main', 'todos')
```

Run with `node app.js`

---

## Authentication

### Supported Methods

| Method | Description |
|--------|-------------|
| Email/Password | Argon2 hashed credentials |
| Phone (SMS) | Passwordless via mobile |
| Magic URL | Email link authentication |
| Email OTP | Time-based codes via email |
| OAuth 2 | 30+ providers (Google, GitHub, etc.) |
| Anonymous | Guest accounts, convertible later |
| JWT | Token-based delegation |
| SSR | Server-side rendering auth |
| Custom Token | Biometric/passkey support |

### Email/Password Authentication

```typescript
import { account, ID } from '@/lib/appwrite'

// Register
async function register(email: string, password: string, name: string) {
  await account.create(ID.unique(), email, password, name)
  await login(email, password)
}

// Login
async function login(email: string, password: string) {
  await account.createEmailPasswordSession(email, password)
}

// Logout
async function logout() {
  await account.deleteSession('current')
}

// Get current user
async function getCurrentUser() {
  try {
    return await account.get()
  } catch {
    return null
  }
}
```

### OAuth Authentication

```typescript
import { account } from '@/lib/appwrite'
import { OAuthProvider } from 'appwrite'

// Redirect to OAuth provider
function loginWithGoogle() {
  account.createOAuth2Session(
    OAuthProvider.Google,
    'http://localhost:3000/dashboard', // Success URL
    'http://localhost:3000/login'      // Failure URL
  )
}

function loginWithGitHub() {
  account.createOAuth2Session(
    OAuthProvider.Github,
    'http://localhost:3000/dashboard',
    'http://localhost:3000/login'
  )
}
```

### Multi-Factor Authentication

```typescript
// Enable MFA
await account.updateMFA(true)

// Create MFA challenge
const challenge = await account.createMfaChallenge('totp')

// Verify MFA
await account.updateMfaChallenge(challenge.$id, otp)
```

---

## Databases

### Structure

- **Databases** contain **Collections**
- **Collections** contain **Documents**
- **Attributes** define document schema
- **Indexes** optimize queries

### Create Database & Collection

Via Console or Server SDK:

```typescript
// Server-side only
import { Client, Databases, Permission, Role } from 'node-appwrite'

const client = new Client()
  .setEndpoint('https://cloud.appwrite.io/v1')
  .setProject('<PROJECT_ID>')
  .setKey('<API_KEY>')

const databases = new Databases(client)

// Create database
await databases.create('main', 'Main Database')

// Create collection
await databases.createCollection(
  'main',
  'posts',
  'Posts',
  [
    Permission.read(Role.any()),
    Permission.create(Role.users()),
    Permission.update(Role.users()),
    Permission.delete(Role.users())
  ]
)

// Add attributes
await databases.createStringAttribute('main', 'posts', 'title', 255, true)
await databases.createStringAttribute('main', 'posts', 'content', 10000, true)
await databases.createStringAttribute('main', 'posts', 'authorId', 36, true)
await databases.createDatetimeAttribute('main', 'posts', 'createdAt', true)
```

### CRUD Operations

```typescript
import { databases, ID } from '@/lib/appwrite'
import { Query } from 'appwrite'

const DATABASE_ID = 'main'
const COLLECTION_ID = 'posts'

// Create document
async function createPost(title: string, content: string, authorId: string) {
  return await databases.createDocument(
    DATABASE_ID,
    COLLECTION_ID,
    ID.unique(),
    {
      title,
      content,
      authorId,
      createdAt: new Date().toISOString()
    }
  )
}

// Read document
async function getPost(postId: string) {
  return await databases.getDocument(DATABASE_ID, COLLECTION_ID, postId)
}

// List documents with queries
async function getPosts(authorId?: string) {
  const queries = [
    Query.orderDesc('createdAt'),
    Query.limit(10)
  ]

  if (authorId) {
    queries.push(Query.equal('authorId', authorId))
  }

  return await databases.listDocuments(DATABASE_ID, COLLECTION_ID, queries)
}

// Update document
async function updatePost(postId: string, data: Partial<Post>) {
  return await databases.updateDocument(
    DATABASE_ID,
    COLLECTION_ID,
    postId,
    data
  )
}

// Delete document
async function deletePost(postId: string) {
  await databases.deleteDocument(DATABASE_ID, COLLECTION_ID, postId)
}
```

### Query Operations

```typescript
import { Query } from 'appwrite'

// Comparison
Query.equal('status', 'published')
Query.notEqual('status', 'draft')
Query.lessThan('price', 100)
Query.greaterThanEqual('rating', 4)

// Search
Query.search('title', 'appwrite')
Query.contains('tags', ['typescript', 'react'])

// Ordering & Pagination
Query.orderAsc('createdAt')
Query.orderDesc('price')
Query.limit(25)
Query.offset(50)
Query.cursorAfter(lastDocumentId)

// Combining queries
const results = await databases.listDocuments(
  DATABASE_ID,
  COLLECTION_ID,
  [
    Query.equal('status', 'published'),
    Query.greaterThan('views', 100),
    Query.orderDesc('createdAt'),
    Query.limit(10)
  ]
)
```

---

## Storage

### Bucket Setup

Create buckets via Console with permissions.

### File Operations

```typescript
import { storage, ID } from '@/lib/appwrite'

const BUCKET_ID = 'uploads'

// Upload file
async function uploadFile(file: File) {
  return await storage.createFile(BUCKET_ID, ID.unique(), file)
}

// Get file URL
function getFileUrl(fileId: string) {
  return storage.getFileView(BUCKET_ID, fileId)
}

// Get file preview (with transformations)
function getFilePreview(
  fileId: string,
  width?: number,
  height?: number
) {
  return storage.getFilePreview(
    BUCKET_ID,
    fileId,
    width,   // Width in pixels
    height,  // Height in pixels
    'center', // Gravity
    90       // Quality (0-100)
  )
}

// Download file
async function downloadFile(fileId: string) {
  return await storage.getFileDownload(BUCKET_ID, fileId)
}

// Delete file
async function deleteFile(fileId: string) {
  await storage.deleteFile(BUCKET_ID, fileId)
}

// List files
async function listFiles() {
  return await storage.listFiles(BUCKET_ID)
}
```

### Image Transformations

```typescript
// Resize image
storage.getFilePreview(BUCKET_ID, fileId, 300, 200)

// Crop with gravity
storage.getFilePreview(BUCKET_ID, fileId, 300, 200, 'top-left')

// Quality and format
storage.getFilePreview(
  BUCKET_ID,
  fileId,
  undefined, // width
  undefined, // height
  undefined, // gravity
  80,        // quality
  undefined, // border width
  undefined, // border color
  undefined, // border radius
  undefined, // opacity
  undefined, // rotation
  undefined, // background
  'webp'     // output format
)
```

---

## Functions

### Supported Runtimes

- Node.js (16, 18, 20)
- Python (3.9, 3.10, 3.11)
- PHP (8.0, 8.1, 8.2)
- Ruby (3.1, 3.2)
- Dart (2.17, 3.0)
- Deno
- Swift
- Kotlin
- .NET
- Go

### Creating a Function

```javascript
// functions/hello/src/main.js
import { Client } from 'node-appwrite'

export default async ({ req, res, log, error }) => {
  const client = new Client()
    .setEndpoint(process.env.APPWRITE_FUNCTION_API_ENDPOINT)
    .setProject(process.env.APPWRITE_FUNCTION_PROJECT_ID)
    .setKey(req.headers['x-appwrite-key'] ?? '')

  log('Function executed!')

  if (req.method === 'GET') {
    return res.json({
      message: 'Hello from Appwrite Functions!',
      timestamp: new Date().toISOString()
    })
  }

  if (req.method === 'POST') {
    const body = JSON.parse(req.body)
    return res.json({
      received: body
    })
  }

  return res.json({ error: 'Method not allowed' }, 405)
}
```

### Function Triggers

- **HTTP** - Direct URL invocation
- **Schedule** - Cron-based execution
- **Events** - Database/auth/storage events

### Deploy via CLI

```bash
# Install CLI
npm install -g appwrite-cli

# Login
appwrite login

# Initialize function
appwrite init function

# Deploy
appwrite deploy function
```

---

## Realtime

### Subscribe to Changes

```typescript
import { client } from '@/lib/appwrite'

// Subscribe to collection changes
const unsubscribe = client.subscribe(
  `databases.main.collections.posts.documents`,
  (response) => {
    if (response.events.includes('databases.*.collections.*.documents.*.create')) {
      console.log('New document:', response.payload)
    }
    if (response.events.includes('databases.*.collections.*.documents.*.update')) {
      console.log('Updated document:', response.payload)
    }
    if (response.events.includes('databases.*.collections.*.documents.*.delete')) {
      console.log('Deleted document:', response.payload)
    }
  }
)

// Subscribe to specific document
client.subscribe(
  `databases.main.collections.posts.documents.${documentId}`,
  (response) => {
    console.log('Document changed:', response.payload)
  }
)

// Subscribe to file uploads
client.subscribe(
  `buckets.${BUCKET_ID}.files`,
  (response) => {
    console.log('File event:', response.payload)
  }
)

// Cleanup
unsubscribe()
```

---

## Permissions

### Permission Types

```typescript
import { Permission, Role } from 'appwrite'

// Anyone (public)
Permission.read(Role.any())

// All logged-in users
Permission.read(Role.users())
Permission.write(Role.users())

// Specific user
Permission.read(Role.user('user123'))
Permission.update(Role.user('user123'))
Permission.delete(Role.user('user123'))

// Team members
Permission.read(Role.team('team123'))
Permission.write(Role.team('team123', 'admin'))

// Guests only
Permission.read(Role.guests())
```

### Document-Level Permissions

```typescript
// Create with permissions
await databases.createDocument(
  DATABASE_ID,
  COLLECTION_ID,
  ID.unique(),
  { title: 'My Post' },
  [
    Permission.read(Role.any()),
    Permission.update(Role.user(userId)),
    Permission.delete(Role.user(userId))
  ]
)
```

---

## Server-Side (TanStack Start / Next.js)

### Server SDK Setup

```typescript
// lib/appwrite-server.ts
import { Client, Account, Databases, Users } from 'node-appwrite'

export function createAdminClient() {
  const client = new Client()
    .setEndpoint(process.env.APPWRITE_ENDPOINT!)
    .setProject(process.env.APPWRITE_PROJECT_ID!)
    .setKey(process.env.APPWRITE_API_KEY!)

  return {
    account: new Account(client),
    databases: new Databases(client),
    users: new Users(client)
  }
}

export function createSessionClient(session: string) {
  const client = new Client()
    .setEndpoint(process.env.APPWRITE_ENDPOINT!)
    .setProject(process.env.APPWRITE_PROJECT_ID!)
    .setSession(session)

  return {
    account: new Account(client),
    databases: new Databases(client)
  }
}
```

### SSR Authentication

```typescript
// Server function
import { createSessionClient } from '@/lib/appwrite-server'
import { cookies } from 'next/headers'

async function getUser() {
  const session = cookies().get('appwrite-session')?.value

  if (!session) return null

  try {
    const { account } = createSessionClient(session)
    return await account.get()
  } catch {
    return null
  }
}
```

---

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
NEXT_PUBLIC_APPWRITE_PROJECT_ID=your_project_id

# Server-only
APPWRITE_API_KEY=your_api_key
```

---

## Self-Hosting

### Docker Compose

```bash
# Install
docker run -it --rm \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  --volume "$(pwd)"/appwrite:/usr/src/code/appwrite:rw \
  --entrypoint="install" \
  appwrite/appwrite:1.5.7

# Start
cd appwrite && docker compose up -d
```

### Configuration

Edit `.env` file for:
- Domain settings
- SMTP configuration
- Storage limits
- Function runtimes

---

## Best Practices

### DO:
- Use **server SDK** for sensitive operations
- Set **proper permissions** on collections and buckets
- Enable **MFA** for admin accounts
- Use **indexes** for frequently queried attributes
- Implement **realtime** for collaborative features

### DON'T:
- Expose **API keys** in client code
- Use `Role.any()` for **write permissions**
- Store **sensitive data** without encryption
- Skip **input validation** before database writes

---

## Resources

- **Official Docs:** https://appwrite.io/docs
- **Quick Starts:** https://appwrite.io/docs/quick-starts
- **SDKs:** https://appwrite.io/docs/sdks
- **API Reference:** https://appwrite.io/docs/references
- **Discord:** https://appwrite.io/discord
- **GitHub:** https://github.com/appwrite/appwrite

---

## Related Stack Modules

- [auth-better-auth.md](../auth/auth-better-auth.md) - Alternative auth solution
- [database-supabase.md](../database/database-supabase.md) - Alternative BaaS
- [database-neon.md](../database/database-neon.md) - Serverless Postgres
- [deployment-vercel.md](./deployment-vercel.md) - Frontend deployment

---

**Last Updated:** 2025-12-10
**Maintained By:** Claude Memory System
**Status:** Active
