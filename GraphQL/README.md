# GraphQL APIs - دليل شامل

## المحتويات
1. [ما هو GraphQL](#ما-هو-graphql)
2. [GraphQL vs REST](#graphql-vs-rest)
3. [المفاهيم الأساسية](#المفاهيم-الأساسية)
4. [Schema و Types](#schema-و-types)
5. [Queries](#queries)
6. [Mutations](#mutations)
7. [Subscriptions](#subscriptions)
8. [Resolvers](#resolvers)
9. [أمثلة عملية](#أمثلة-عملية)
10. [Best Practices](#best-practices)
11. [أدوات GraphQL](#أدوات-graphql)
12. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو GraphQL

**GraphQL** هو لغة استعلام (Query Language) ووقت تشغيل (Runtime) لـ APIs تم تطويره بواسطة Facebook في 2012 وأصبح Open Source في 2015.

### المزايا الرئيسية
- **Single Endpoint**: نقطة واحدة فقط لجميع الطلبات
- **Client-Specified Queries**: العميل يحدد البيانات التي يريدها بالضبط
- **No Over-fetching**: لا تحصل على بيانات زائدة
- **No Under-fetching**: تحصل على جميع البيانات المطلوبة في طلب واحد
- **Strong Typing**: نظام Types قوي
- **Introspection**: استكشاف Schema تلقائياً
- **Real-time Updates**: دعم Subscriptions

### متى تستخدم GraphQL؟
- ✅ تطبيقات معقدة مع بيانات متداخلة
- ✅ تطبيقات موبايل (تقليل عدد الطلبات)
- ✅ عندما تحتاج مرونة في البيانات
- ✅ عند وجود عملاء متعددين بمتطلبات مختلفة
- ✅ عندما تريد تقليل Bandwidth

### متى تستخدم REST؟
- ✅ APIs بسيطة
- ✅ عمليات CRUD أساسية
- ✅ عندما تحتاج HTTP Caching
- ✅ File uploads/downloads
- ✅ فريق ليس لديه خبرة في GraphQL

---

## GraphQL vs REST

| الميزة | GraphQL | REST |
|--------|---------|------|
| **Endpoints** | واحد فقط (`/graphql`) | متعدد (`/users`, `/posts`) |
| **Data Fetching** | العميل يحدد البيانات | Server يحدد البيانات |
| **Over-fetching** | لا يحدث | يحدث غالباً |
| **Under-fetching** | لا يحدث | يحدث (N+1 problem) |
| **Versioning** | لا حاجة له | `/v1/users`, `/v2/users` |
| **Caching** | معقد (يحتاج Apollo Client) | بسيط (HTTP Cache) |
| **Learning Curve** | أعلى | أقل |
| **Documentation** | تلقائي (Introspection) | يدوي (Swagger) |
| **Real-time** | مدمج (Subscriptions) | يحتاج WebSockets منفصل |
| **Error Handling** | دائماً 200 OK | HTTP Status Codes |

### مثال مقارنة

**REST:**
```bash
# طلب 1: جلب User
GET /users/1
Response: {
  "id": 1,
  "name": "Ahmed",
  "email": "ahmed@example.com",
  "bio": "...",
  "avatar": "...",
  "created_at": "..."
}

# طلب 2: جلب Posts للـ User
GET /users/1/posts
Response: [{
  "id": 101,
  "title": "GraphQL Guide",
  "content": "...",
  "created_at": "...",
  "author_id": 1
}]

# طلب 3: جلب Comments لكل Post
GET /posts/101/comments
```

**GraphQL:**
```graphql
# طلب واحد فقط
query {
  user(id: 1) {
    name
    email
    posts {
      title
      comments {
        content
        author {
          name
        }
      }
    }
  }
}

# Response يحتوي على كل شيء
{
  "data": {
    "user": {
      "name": "Ahmed",
      "email": "ahmed@example.com",
      "posts": [{
        "title": "GraphQL Guide",
        "comments": [{
          "content": "Great post!",
          "author": {
            "name": "Sara"
          }
        }]
      }]
    }
  }
}
```

---

## المفاهيم الأساسية

### 1. Schema
تعريف هيكل API بالكامل باستخدام **GraphQL Schema Definition Language (SDL)**.

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  published: Boolean!
}

type Query {
  user(id: ID!): User
  users: [User!]!
  post(id: ID!): Post
  posts: [Post!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String, email: String): User!
  deleteUser(id: ID!): Boolean!
}
```

### 2. Types

#### Scalar Types
أنواع بيانات أساسية:
- **Int**: عدد صحيح 32-bit
- **Float**: عدد عشري
- **String**: نص UTF-8
- **Boolean**: true/false
- **ID**: معرّف فريد

#### Object Types
أنواع مخصصة:
```graphql
type User {
  id: ID!
  name: String!
}
```

#### Enum Types
قيم ثابتة:
```graphql
enum Role {
  ADMIN
  USER
  GUEST
}

type User {
  id: ID!
  name: String!
  role: Role!
}
```

#### Input Types
للإدخال في Mutations:
```graphql
input CreateUserInput {
  name: String!
  email: String!
  age: Int
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

#### List Types
قوائم:
```graphql
type User {
  posts: [Post!]!  # قائمة Posts (non-null)
}
```

#### Non-Null Types
استخدام `!` يعني القيمة مطلوبة:
```graphql
type User {
  id: ID!        # مطلوب
  name: String!  # مطلوب
  age: Int       # اختياري (nullable)
}
```

### 3. Query
قراءة البيانات (مثل GET في REST).

### 4. Mutation
تعديل البيانات (مثل POST, PUT, DELETE في REST).

### 5. Subscription
تلقي تحديثات في الوقت الفعلي (Real-time).

### 6. Resolvers
دوال تنفذ منطق جلب البيانات لكل Field.

---

## Schema و Types

### Schema كامل - مثال متقدم

```graphql
# Scalar Types مخصصة
scalar DateTime
scalar Email
scalar URL

# Enums
enum UserRole {
  ADMIN
  MODERATOR
  USER
  GUEST
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

# Object Types
type User {
  id: ID!
  name: String!
  email: Email!
  avatar: URL
  role: UserRole!
  bio: String
  createdAt: DateTime!
  updatedAt: DateTime!
  posts(
    limit: Int = 10
    offset: Int = 0
    status: PostStatus
  ): [Post!]!
  followers: [User!]!
  following: [User!]!
  followerCount: Int!
  followingCount: Int!
}

type Post {
  id: ID!
  title: String!
  content: String!
  excerpt: String
  status: PostStatus!
  author: User!
  comments: [Comment!]!
  likes: [Like!]!
  likeCount: Int!
  createdAt: DateTime!
  updatedAt: DateTime!
  publishedAt: DateTime
  tags: [Tag!]!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  post: Post!
  parent: Comment
  replies: [Comment!]!
  likes: [Like!]!
  likeCount: Int!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Like {
  id: ID!
  user: User!
  post: Post
  comment: Comment
  createdAt: DateTime!
}

type Tag {
  id: ID!
  name: String!
  slug: String!
  posts: [Post!]!
}

# Input Types
input CreateUserInput {
  name: String!
  email: Email!
  password: String!
  avatar: URL
  bio: String
}

input UpdateUserInput {
  name: String
  email: Email
  avatar: URL
  bio: String
}

input CreatePostInput {
  title: String!
  content: String!
  excerpt: String
  status: PostStatus = DRAFT
  tags: [ID!]
}

input UpdatePostInput {
  title: String
  content: String
  excerpt: String
  status: PostStatus
  tags: [ID!]
}

input CreateCommentInput {
  content: String!
  postId: ID!
  parentId: ID
}

input PaginationInput {
  limit: Int = 10
  offset: Int = 0
}

input FilterInput {
  search: String
  status: PostStatus
  authorId: ID
  tagIds: [ID!]
}

# Query Type
type Query {
  # User Queries
  me: User
  user(id: ID!): User
  users(
    pagination: PaginationInput
    search: String
    role: UserRole
  ): UserConnection!

  # Post Queries
  post(id: ID!): Post
  posts(
    pagination: PaginationInput
    filter: FilterInput
  ): PostConnection!

  # Comment Queries
  comment(id: ID!): Comment
  comments(postId: ID!, pagination: PaginationInput): CommentConnection!

  # Tag Queries
  tag(id: ID!): Tag
  tags(search: String): [Tag!]!

  # Search
  search(query: String!, limit: Int = 10): SearchResult!
}

# Mutation Type
type Mutation {
  # Auth Mutations
  register(input: CreateUserInput!): AuthPayload!
  login(email: Email!, password: String!): AuthPayload!
  logout: Boolean!

  # User Mutations
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  followUser(userId: ID!): User!
  unfollowUser(userId: ID!): User!

  # Post Mutations
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post!
  deletePost(id: ID!): Boolean!
  publishPost(id: ID!): Post!
  likePost(postId: ID!): Like!
  unlikePost(postId: ID!): Boolean!

  # Comment Mutations
  createComment(input: CreateCommentInput!): Comment!
  updateComment(id: ID!, content: String!): Comment!
  deleteComment(id: ID!): Boolean!
  likeComment(commentId: ID!): Like!
  unlikeComment(commentId: ID!): Boolean!

  # Tag Mutations
  createTag(name: String!): Tag!
  deleteTag(id: ID!): Boolean!
}

# Subscription Type
type Subscription {
  postCreated: Post!
  postUpdated(postId: ID!): Post!
  commentAdded(postId: ID!): Comment!
  userFollowed(userId: ID!): User!
  likeAdded(postId: ID!): Like!
}

# Connection Types (للـ Pagination)
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type CommentConnection {
  edges: [CommentEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type CommentEdge {
  node: Comment!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Union Types
union SearchResult = User | Post | Tag

# Auth Payload
type AuthPayload {
  token: String!
  user: User!
}
```

---

## Queries

### Basic Queries

```graphql
# جلب User واحد
query GetUser {
  user(id: "1") {
    id
    name
    email
  }
}

# جلب جميع Users
query GetAllUsers {
  users {
    id
    name
    email
  }
}

# جلب User مع Posts
query GetUserWithPosts {
  user(id: "1") {
    id
    name
    posts {
      id
      title
      content
    }
  }
}
```

### Queries مع Arguments

```graphql
query GetUserPosts {
  user(id: "1") {
    name
    posts(limit: 5, offset: 0, status: PUBLISHED) {
      id
      title
      status
      createdAt
    }
  }
}
```

### Queries مع Variables

```graphql
query GetUser($userId: ID!, $postsLimit: Int = 10) {
  user(id: $userId) {
    id
    name
    email
    posts(limit: $postsLimit) {
      id
      title
    }
  }
}

# Variables
{
  "userId": "1",
  "postsLimit": 5
}
```

### Query Aliases

```graphql
query {
  user1: user(id: "1") {
    name
    email
  }
  user2: user(id: "2") {
    name
    email
  }
}

# Response
{
  "data": {
    "user1": {
      "name": "Ahmed",
      "email": "ahmed@example.com"
    },
    "user2": {
      "name": "Sara",
      "email": "sara@example.com"
    }
  }
}
```

### Fragments

```graphql
fragment UserBasicInfo on User {
  id
  name
  email
  avatar
}

fragment PostInfo on Post {
  id
  title
  excerpt
  createdAt
}

query GetUserWithPosts {
  user(id: "1") {
    ...UserBasicInfo
    posts {
      ...PostInfo
      author {
        ...UserBasicInfo
      }
    }
  }
}
```

### Inline Fragments (للـ Union Types)

```graphql
query Search {
  search(query: "graphql") {
    ... on User {
      id
      name
      email
    }
    ... on Post {
      id
      title
      content
    }
    ... on Tag {
      id
      name
      slug
    }
  }
}
```

### Nested Queries

```graphql
query GetComplexData {
  user(id: "1") {
    name
    posts {
      title
      comments {
        content
        author {
          name
          avatar
        }
        replies {
          content
          author {
            name
          }
        }
      }
      likes {
        user {
          name
        }
      }
    }
  }
}
```

---

## Mutations

### Basic Mutations

```graphql
# إنشاء User
mutation CreateUser {
  createUser(
    input: {
      name: "Ahmed"
      email: "ahmed@example.com"
      password: "password123"
    }
  ) {
    id
    name
    email
    createdAt
  }
}

# تحديث User
mutation UpdateUser {
  updateUser(
    id: "1"
    input: {
      name: "Ahmed Ali"
      bio: "Software Engineer"
    }
  ) {
    id
    name
    bio
    updatedAt
  }
}

# حذف User
mutation DeleteUser {
  deleteUser(id: "1")
}
```

### Mutations مع Variables

```graphql
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    id
    title
    content
    status
    author {
      id
      name
    }
    createdAt
  }
}

# Variables
{
  "input": {
    "title": "GraphQL Guide",
    "content": "Complete guide to GraphQL...",
    "status": "PUBLISHED",
    "tags": ["1", "2"]
  }
}
```

### Multiple Mutations

```graphql
mutation CreatePostAndComment {
  createPost(
    input: {
      title: "New Post"
      content: "Content here..."
    }
  ) {
    id
    title
  }

  createComment(
    input: {
      content: "First comment!"
      postId: "1"
    }
  ) {
    id
    content
  }
}
```

### Optimistic UI Update

```graphql
mutation LikePost($postId: ID!) {
  likePost(postId: $postId) {
    id
    user {
      id
      name
    }
    createdAt
  }
}
```

---

## Subscriptions

### Basic Subscriptions

```graphql
subscription OnPostCreated {
  postCreated {
    id
    title
    content
    author {
      id
      name
    }
    createdAt
  }
}
```

### Subscriptions مع Arguments

```graphql
subscription OnCommentAdded($postId: ID!) {
  commentAdded(postId: $postId) {
    id
    content
    author {
      name
      avatar
    }
    createdAt
  }
}

# Variables
{
  "postId": "1"
}
```

### Multiple Subscriptions

```graphql
subscription OnPostUpdates($postId: ID!) {
  postUpdated(postId: $postId) {
    id
    title
    content
    updatedAt
  }

  commentAdded(postId: $postId) {
    id
    content
    author {
      name
    }
  }

  likeAdded(postId: $postId) {
    id
    user {
      name
    }
  }
}
```

---

## Resolvers

### Node.js مع Apollo Server

```javascript
const { ApolloServer, gql } = require('apollo-server');

// Schema Definition
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
    comments: [Comment!]!
  }

  type Comment {
    id: ID!
    content: String!
    author: User!
    post: Post!
  }

  type Query {
    user(id: ID!): User
    users: [User!]!
    post(id: ID!): Post
    posts: [Post!]!
  }

  type Mutation {
    createUser(name: String!, email: String!): User!
    createPost(title: String!, content: String!, authorId: ID!): Post!
    createComment(content: String!, postId: ID!, authorId: ID!): Comment!
  }

  type Subscription {
    postCreated: Post!
    commentAdded(postId: ID!): Comment!
  }
`;

// Resolvers
const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      // جلب User من Database
      return await context.db.User.findById(id);
    },

    users: async (parent, args, context) => {
      return await context.db.User.findAll();
    },

    post: async (parent, { id }, context) => {
      return await context.db.Post.findById(id);
    },

    posts: async (parent, args, context) => {
      return await context.db.Post.findAll();
    }
  },

  Mutation: {
    createUser: async (parent, { name, email }, context) => {
      // التحقق من Authentication
      if (!context.user) {
        throw new Error('Not authenticated');
      }

      // التحقق من البيانات
      if (!email.includes('@')) {
        throw new Error('Invalid email');
      }

      // إنشاء User
      const user = await context.db.User.create({
        name,
        email
      });

      return user;
    },

    createPost: async (parent, { title, content, authorId }, context) => {
      // التحقق من Authorization
      if (context.user.id !== authorId) {
        throw new Error('Not authorized');
      }

      const post = await context.db.Post.create({
        title,
        content,
        authorId
      });

      // إطلاق Subscription
      context.pubsub.publish('POST_CREATED', {
        postCreated: post
      });

      return post;
    },

    createComment: async (parent, { content, postId, authorId }, context) => {
      const comment = await context.db.Comment.create({
        content,
        postId,
        authorId
      });

      // إطلاق Subscription
      context.pubsub.publish('COMMENT_ADDED', {
        commentAdded: comment,
        postId: postId
      });

      return comment;
    }
  },

  Subscription: {
    postCreated: {
      subscribe: (parent, args, context) => {
        return context.pubsub.asyncIterator(['POST_CREATED']);
      }
    },

    commentAdded: {
      subscribe: (parent, { postId }, context) => {
        return context.pubsub.asyncIterator([`COMMENT_ADDED_${postId}`]);
      }
    }
  },

  // Field Resolvers
  User: {
    posts: async (parent, args, context) => {
      // parent هو User object
      return await context.db.Post.findByAuthorId(parent.id);
    }
  },

  Post: {
    author: async (parent, args, context) => {
      // parent هو Post object
      return await context.db.User.findById(parent.authorId);
    },

    comments: async (parent, args, context) => {
      return await context.db.Comment.findByPostId(parent.id);
    }
  },

  Comment: {
    author: async (parent, args, context) => {
      return await context.db.User.findById(parent.authorId);
    },

    post: async (parent, args, context) => {
      return await context.db.Post.findById(parent.postId);
    }
  }
};

// Context Function
const context = ({ req }) => {
  // استخراج Token من Headers
  const token = req.headers.authorization || '';

  // التحقق من Token
  const user = getUserFromToken(token);

  return {
    user,
    db: database,
    pubsub: pubsub
  };
};

// Server
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context
});

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

### DataLoader (حل N+1 Problem)

```javascript
const DataLoader = require('dataloader');

// Batch Function
const batchUsers = async (ids) => {
  const users = await db.User.findAll({
    where: { id: ids }
  });

  // ترتيب النتائج حسب ترتيب IDs
  const userMap = {};
  users.forEach(user => {
    userMap[user.id] = user;
  });

  return ids.map(id => userMap[id]);
};

// Context مع DataLoader
const context = ({ req }) => {
  return {
    user: getUserFromToken(req.headers.authorization),
    db: database,
    loaders: {
      user: new DataLoader(batchUsers)
    }
  };
};

// Resolver يستخدم DataLoader
const resolvers = {
  Post: {
    author: async (parent, args, context) => {
      // DataLoader يجمع الطلبات ويرسلها دفعة واحدة
      return await context.loaders.user.load(parent.authorId);
    }
  }
};
```

---

## أمثلة عملية

### مثال 1: Authentication & Authorization

```javascript
// Schema
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    role: Role!
  }

  enum Role {
    ADMIN
    USER
  }

  type AuthPayload {
    token: String!
    user: User!
  }

  type Query {
    me: User
  }

  type Mutation {
    register(name: String!, email: String!, password: String!): AuthPayload!
    login(email: String!, password: String!): AuthPayload!
  }
`;

// Resolvers
const resolvers = {
  Query: {
    me: (parent, args, context) => {
      // التحقق من Authentication
      if (!context.user) {
        throw new Error('Not authenticated');
      }
      return context.user;
    }
  },

  Mutation: {
    register: async (parent, { name, email, password }, context) => {
      // تشفير Password
      const hashedPassword = await bcrypt.hash(password, 10);

      // إنشاء User
      const user = await context.db.User.create({
        name,
        email,
        password: hashedPassword,
        role: 'USER'
      });

      // إنشاء JWT Token
      const token = jwt.sign(
        { userId: user.id, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      );

      return { token, user };
    },

    login: async (parent, { email, password }, context) => {
      // البحث عن User
      const user = await context.db.User.findOne({ email });

      if (!user) {
        throw new Error('Invalid credentials');
      }

      // التحقق من Password
      const valid = await bcrypt.compare(password, user.password);

      if (!valid) {
        throw new Error('Invalid credentials');
      }

      // إنشاء Token
      const token = jwt.sign(
        { userId: user.id, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      );

      return { token, user };
    }
  }
};

// Context مع Authentication
const context = ({ req }) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  let user = null;
  if (token) {
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      user = { id: decoded.userId, role: decoded.role };
    } catch (err) {
      // Token غير صالح
    }
  }

  return { user, db: database };
};

// Authorization Directive
const { SchemaDirectiveVisitor } = require('apollo-server');

class AuthDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field) {
    const { resolve = defaultFieldResolver } = field;
    const { role } = this.args;

    field.resolve = async function(...args) {
      const context = args[2];

      if (!context.user) {
        throw new Error('Not authenticated');
      }

      if (role && context.user.role !== role) {
        throw new Error('Not authorized');
      }

      return resolve.apply(this, args);
    };
  }
}

// استخدام Directive
const typeDefs = gql`
  directive @auth(role: Role) on FIELD_DEFINITION

  type Query {
    me: User @auth
    users: [User!]! @auth(role: ADMIN)
  }
`;

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context,
  schemaDirectives: {
    auth: AuthDirective
  }
});
```

### مثال 2: Pagination (Cursor-based)

```javascript
const typeDefs = gql`
  type Post {
    id: ID!
    title: String!
    content: String!
  }

  type PostEdge {
    node: Post!
    cursor: String!
  }

  type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
  }

  type PostConnection {
    edges: [PostEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
  }

  type Query {
    posts(
      first: Int
      after: String
      last: Int
      before: String
    ): PostConnection!
  }
`;

const resolvers = {
  Query: {
    posts: async (parent, { first, after, last, before }, context) => {
      const limit = first || last || 10;
      let posts;

      if (after) {
        // جلب Posts بعد Cursor معين
        posts = await context.db.Post.findAll({
          where: { id: { $gt: after } },
          limit: limit + 1,
          order: [['id', 'ASC']]
        });
      } else if (before) {
        // جلب Posts قبل Cursor معين
        posts = await context.db.Post.findAll({
          where: { id: { $lt: before } },
          limit: limit + 1,
          order: [['id', 'DESC']]
        });
        posts.reverse();
      } else {
        posts = await context.db.Post.findAll({
          limit: limit + 1,
          order: [['id', 'ASC']]
        });
      }

      const hasNextPage = posts.length > limit;
      const hasPreviousPage = !!after;

      if (hasNextPage) {
        posts.pop();
      }

      const edges = posts.map(post => ({
        node: post,
        cursor: post.id
      }));

      const totalCount = await context.db.Post.count();

      return {
        edges,
        pageInfo: {
          hasNextPage,
          hasPreviousPage,
          startCursor: edges[0]?.cursor,
          endCursor: edges[edges.length - 1]?.cursor
        },
        totalCount
      };
    }
  }
};
```

**استخدام:**
```graphql
query GetPosts {
  posts(first: 10) {
    edges {
      node {
        id
        title
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}

# الصفحة التالية
query GetNextPage {
  posts(first: 10, after: "10") {
    edges {
      node {
        id
        title
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### مثال 3: File Upload

```javascript
const { GraphQLUpload } = require('graphql-upload');
const fs = require('fs');
const path = require('path');

const typeDefs = gql`
  scalar Upload

  type File {
    filename: String!
    mimetype: String!
    encoding: String!
    url: String!
  }

  type Mutation {
    uploadFile(file: Upload!): File!
    uploadFiles(files: [Upload!]!): [File!]!
  }
`;

const resolvers = {
  Upload: GraphQLUpload,

  Mutation: {
    uploadFile: async (parent, { file }) => {
      const { createReadStream, filename, mimetype, encoding } = await file;

      // حفظ الملف
      const stream = createReadStream();
      const filepath = path.join(__dirname, 'uploads', filename);

      await new Promise((resolve, reject) => {
        stream
          .pipe(fs.createWriteStream(filepath))
          .on('finish', resolve)
          .on('error', reject);
      });

      return {
        filename,
        mimetype,
        encoding,
        url: `/uploads/${filename}`
      };
    },

    uploadFiles: async (parent, { files }) => {
      const uploadedFiles = [];

      for (const file of files) {
        const { createReadStream, filename, mimetype, encoding } = await file;

        const stream = createReadStream();
        const filepath = path.join(__dirname, 'uploads', filename);

        await new Promise((resolve, reject) => {
          stream
            .pipe(fs.createWriteStream(filepath))
            .on('finish', resolve)
            .on('error', reject);
        });

        uploadedFiles.push({
          filename,
          mimetype,
          encoding,
          url: `/uploads/${filename}`
        });
      }

      return uploadedFiles;
    }
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  uploads: {
    maxFileSize: 10000000, // 10 MB
    maxFiles: 5
  }
});
```

**استخدام من Client:**
```javascript
import { gql, useMutation } from '@apollo/client';

const UPLOAD_FILE = gql`
  mutation UploadFile($file: Upload!) {
    uploadFile(file: $file) {
      filename
      url
    }
  }
`;

function FileUpload() {
  const [uploadFile] = useMutation(UPLOAD_FILE);

  const handleFileChange = (e) => {
    const file = e.target.files[0];

    uploadFile({
      variables: { file }
    }).then(response => {
      console.log('Uploaded:', response.data.uploadFile);
    });
  };

  return <input type="file" onChange={handleFileChange} />;
}
```

### مثال 4: Real-time Chat مع Subscriptions

```javascript
const { PubSub } = require('graphql-subscriptions');
const pubsub = new PubSub();

const typeDefs = gql`
  type Message {
    id: ID!
    content: String!
    author: User!
    room: String!
    createdAt: String!
  }

  type Query {
    messages(room: String!): [Message!]!
  }

  type Mutation {
    sendMessage(content: String!, room: String!): Message!
  }

  type Subscription {
    messageAdded(room: String!): Message!
    userTyping(room: String!): User!
  }
`;

const resolvers = {
  Query: {
    messages: async (parent, { room }, context) => {
      return await context.db.Message.findAll({
        where: { room },
        order: [['createdAt', 'DESC']],
        limit: 50
      });
    }
  },

  Mutation: {
    sendMessage: async (parent, { content, room }, context) => {
      if (!context.user) {
        throw new Error('Not authenticated');
      }

      const message = await context.db.Message.create({
        content,
        room,
        authorId: context.user.id,
        createdAt: new Date().toISOString()
      });

      // نشر Subscription
      pubsub.publish(`MESSAGE_ADDED_${room}`, {
        messageAdded: message
      });

      return message;
    }
  },

  Subscription: {
    messageAdded: {
      subscribe: (parent, { room }) => {
        return pubsub.asyncIterator([`MESSAGE_ADDED_${room}`]);
      }
    },

    userTyping: {
      subscribe: (parent, { room }) => {
        return pubsub.asyncIterator([`USER_TYPING_${room}`]);
      }
    }
  },

  Message: {
    author: async (parent, args, context) => {
      return await context.loaders.user.load(parent.authorId);
    }
  }
};
```

**Client (React + Apollo):**
```javascript
import { gql, useQuery, useMutation, useSubscription } from '@apollo/client';

const GET_MESSAGES = gql`
  query GetMessages($room: String!) {
    messages(room: $room) {
      id
      content
      author {
        id
        name
      }
      createdAt
    }
  }
`;

const SEND_MESSAGE = gql`
  mutation SendMessage($content: String!, $room: String!) {
    sendMessage(content: $content, room: $room) {
      id
      content
      createdAt
    }
  }
`;

const MESSAGE_ADDED = gql`
  subscription OnMessageAdded($room: String!) {
    messageAdded(room: $room) {
      id
      content
      author {
        id
        name
      }
      createdAt
    }
  }
`;

function Chat({ room }) {
  const { data } = useQuery(GET_MESSAGES, { variables: { room } });
  const [sendMessage] = useMutation(SEND_MESSAGE);

  // Subscription للرسائل الجديدة
  useSubscription(MESSAGE_ADDED, {
    variables: { room },
    onSubscriptionData: ({ client, subscriptionData }) => {
      // تحديث Cache
      const newMessage = subscriptionData.data.messageAdded;
      const existingMessages = client.readQuery({
        query: GET_MESSAGES,
        variables: { room }
      });

      client.writeQuery({
        query: GET_MESSAGES,
        variables: { room },
        data: {
          messages: [newMessage, ...existingMessages.messages]
        }
      });
    }
  });

  return (
    <div>
      {data?.messages.map(msg => (
        <div key={msg.id}>
          <strong>{msg.author.name}:</strong> {msg.content}
        </div>
      ))}

      <form onSubmit={(e) => {
        e.preventDefault();
        sendMessage({
          variables: {
            content: e.target.message.value,
            room
          }
        });
        e.target.reset();
      }}>
        <input name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

---

## Best Practices

### 1. Schema Design

#### ✅ استخدم أسماء واضحة
```graphql
# جيد
type User {
  id: ID!
  fullName: String!
  emailAddress: String!
}

# سيء
type User {
  id: ID!
  fn: String!
  email: String!
}
```

#### ✅ استخدم Enums للقيم الثابتة
```graphql
enum UserRole {
  ADMIN
  USER
  GUEST
}
```

#### ✅ استخدم Input Types للـ Mutations
```graphql
input CreateUserInput {
  name: String!
  email: String!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

#### ✅ استخدم Connections للـ Pagination
```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}
```

### 2. Performance

#### ✅ استخدم DataLoader لحل N+1 Problem
```javascript
const userLoader = new DataLoader(async (ids) => {
  const users = await db.User.findAll({ where: { id: ids } });
  return ids.map(id => users.find(u => u.id === id));
});
```

#### ✅ استخدم Query Depth Limiting
```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)]
});
```

#### ✅ استخدم Query Complexity Analysis
```javascript
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    createComplexityLimitRule(1000)
  ]
});
```

#### ✅ Caching
```javascript
// Redis Cache
const Redis = require('ioredis');
const redis = new Redis();

const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      // محاولة جلب من Cache
      const cached = await redis.get(`user:${id}`);
      if (cached) {
        return JSON.parse(cached);
      }

      // جلب من Database
      const user = await context.db.User.findById(id);

      // حفظ في Cache
      await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);

      return user;
    }
  }
};
```

### 3. Security

#### ✅ Authentication & Authorization
```javascript
// Middleware للـ Authentication
const context = ({ req }) => {
  const token = req.headers.authorization;
  const user = verifyToken(token);

  return { user, db };
};

// Authorization في Resolvers
const resolvers = {
  Query: {
    sensitiveData: (parent, args, context) => {
      if (!context.user || context.user.role !== 'ADMIN') {
        throw new Error('Not authorized');
      }
      return getSensitiveData();
    }
  }
};
```

#### ✅ Rate Limiting
```javascript
const { RateLimiterMemory } = require('rate-limiter-flexible');

const rateLimiter = new RateLimiterMemory({
  points: 100, // عدد الطلبات
  duration: 60 // في الثانية
});

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async ({ req }) => {
    try {
      await rateLimiter.consume(req.ip);
    } catch (err) {
      throw new Error('Too many requests');
    }

    return { db };
  }
});
```

#### ✅ Input Validation
```javascript
const Joi = require('joi');

const createUserSchema = Joi.object({
  name: Joi.string().min(3).max(50).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required()
});

const resolvers = {
  Mutation: {
    createUser: async (parent, args) => {
      const { error } = createUserSchema.validate(args);
      if (error) {
        throw new Error(error.details[0].message);
      }

      // إنشاء User
    }
  }
};
```

#### ✅ Query Cost Analysis
```javascript
const costAnalysis = require('graphql-cost-analysis').default;

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    costAnalysis({
      maximumCost: 1000,
      defaultCost: 1,
      onComplete: (costs) => {
        console.log('Query cost:', costs);
      }
    })
  ]
});
```

### 4. Error Handling

#### ✅ Custom Errors
```javascript
class AuthenticationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'AuthenticationError';
    this.extensions = {
      code: 'UNAUTHENTICATED'
    };
  }
}

class AuthorizationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'AuthorizationError';
    this.extensions = {
      code: 'FORBIDDEN'
    };
  }
}

// استخدام
const resolvers = {
  Query: {
    me: (parent, args, context) => {
      if (!context.user) {
        throw new AuthenticationError('You must be logged in');
      }
      return context.user;
    }
  }
};
```

#### ✅ Error Formatting
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (err) => {
    // إخفاء Internal Errors
    if (err.message.startsWith('Database')) {
      return new Error('Internal server error');
    }

    // Log الأخطاء
    console.error(err);

    return err;
  }
});
```

### 5. Testing

#### Unit Testing Resolvers
```javascript
const { createTestClient } = require('apollo-server-testing');
const { ApolloServer } = require('apollo-server');

describe('User Resolvers', () => {
  it('should return user by id', async () => {
    const server = new ApolloServer({
      typeDefs,
      resolvers,
      context: () => ({
        db: mockDatabase,
        user: { id: '1', role: 'USER' }
      })
    });

    const { query } = createTestClient(server);

    const GET_USER = gql`
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          name
          email
        }
      }
    `;

    const res = await query({
      query: GET_USER,
      variables: { id: '1' }
    });

    expect(res.data.user).toEqual({
      id: '1',
      name: 'Test User',
      email: 'test@example.com'
    });
  });
});
```

---

## أدوات GraphQL

### 1. Apollo Server (Node.js)
أشهر Server Implementation لـ GraphQL.

```bash
npm install apollo-server graphql
```

### 2. Apollo Client (React)
Client library للتفاعل مع GraphQL APIs.

```bash
npm install @apollo/client graphql
```

### 3. GraphQL Yoga
Server بسيط وسريع.

```bash
npm install graphql-yoga
```

### 4. Prisma
ORM حديث يدعم GraphQL.

```bash
npm install @prisma/client
```

### 5. Hasura
GraphQL Engine يوفر GraphQL API تلقائياً من Database.

### 6. AWS AppSync
Managed GraphQL service من AWS.

### 7. GraphQL Code Generator
توليد Types و Hooks تلقائياً.

```bash
npm install @graphql-codegen/cli
```

### 8. GraphQL Playground
IDE لاختبار GraphQL APIs (مثل Postman).

### 9. GraphiQL
IDE آخر لاختبار APIs.

### 10. Apollo Studio
منصة لمراقبة وإدارة GraphQL APIs.

---

## أسئلة المقابلات

### أسئلة أساسية

**1. ما هو GraphQL ولماذا نستخدمه؟**
- **الإجابة**: GraphQL هو Query Language لـ APIs يسمح للعميل بطلب البيانات المحددة التي يحتاجها فقط. المزايا:
  - No Over-fetching/Under-fetching
  - Single Endpoint
  - Strong Typing
  - Real-time Updates (Subscriptions)

**2. ما الفرق بين GraphQL و REST؟**
- **الإجابة**:
  - **Endpoints**: GraphQL endpoint واحد، REST endpoints متعددة
  - **Data Fetching**: GraphQL العميل يحدد البيانات، REST Server يحدد
  - **Versioning**: GraphQL لا يحتاج، REST يحتاج `/v1`, `/v2`
  - **Caching**: REST بسيط، GraphQL معقد

**3. ما هي الـ Types الأساسية في GraphQL؟**
- **الإجابة**:
  - **Scalar**: Int, Float, String, Boolean, ID
  - **Object**: Types مخصصة
  - **Enum**: قيم ثابتة
  - **Input**: للإدخال في Mutations
  - **List**: قوائم

**4. ما الفرق بين Query و Mutation؟**
- **الإجابة**:
  - **Query**: لقراءة البيانات (مثل GET)
  - **Mutation**: لتعديل البيانات (مثل POST, PUT, DELETE)
  - Queries تُنفذ بالتوازي، Mutations تُنفذ بالتسلسل

**5. ما هو Schema في GraphQL؟**
- **الإجابة**: Schema هو تعريف هيكل API بالكامل، يحتوي على:
  - Types
  - Queries
  - Mutations
  - Subscriptions
  - مكتوب بـ SDL (Schema Definition Language)

### أسئلة متقدمة

**6. ما هو Resolver في GraphQL؟**
- **الإجابة**: Resolver هو دالة تنفذ منطق جلب البيانات لكل Field في Schema. يستقبل 4 parameters:
  - `parent`: البيانات من Resolver السابق
  - `args`: Arguments المُمررة
  - `context`: بيانات مشتركة (user, db, loaders)
  - `info`: معلومات عن Query

**7. ما هي N+1 Problem وكيف تحلها؟**
- **الإجابة**: N+1 Problem يحدث عندما:
  - Query 1 لجلب N items
  - N Queries لجلب بيانات مرتبطة لكل item

  **الحل**: استخدام DataLoader:
  - يجمع الطلبات
  - يرسلها دفعة واحدة
  - يقلل عدد Database queries

**8. ما هي Subscriptions في GraphQL؟**
- **الإجابة**: Subscriptions تسمح بـ Real-time Updates:
  - تستخدم WebSockets
  - Client يشترك في Event معين
  - Server يرسل تحديثات عند حدوث Event
  - مثال: Chat, Notifications, Live Updates

**9. كيف تنفذ Authentication في GraphQL؟**
- **الإجابة**:
  - استخراج Token من Headers
  - التحقق من Token في Context function
  - إضافة User إلى Context
  - فحص `context.user` في Resolvers

**10. كيف تنفذ Pagination في GraphQL؟**
- **الإجابة**: طريقتان:
  - **Offset-based**: `posts(limit: 10, offset: 20)`
  - **Cursor-based**: `posts(first: 10, after: "cursor")`
    - أفضل للبيانات الديناميكية
    - يستخدم Relay Connection Specification

**11. ما هو Fragment في GraphQL؟**
- **الإجابة**: Fragment هو مجموعة Fields قابلة لإعادة الاستخدام:
```graphql
fragment UserInfo on User {
  id
  name
  email
}

query {
  user(id: "1") {
    ...UserInfo
  }
}
```

**12. ما هو Introspection في GraphQL؟**
- **الإجابة**: Introspection يسمح باستكشاف Schema:
  - Client يسأل عن Types المتاحة
  - يستخدم في GraphQL Playground
  - يجب تعطيله في Production لأسباب أمنية

**13. كيف تتعامل مع File Upload في GraphQL؟**
- **الإجابة**:
  - استخدام `graphql-upload` package
  - تعريف `Upload` Scalar
  - معالجة File stream في Resolver
  - أو استخدام Pre-signed URLs (S3)

**14. ما هي Directives في GraphQL؟**
- **الإجابة**: Directives تغير سلوك Execution:
  - `@include(if: Boolean)`: شرط لإظهار Field
  - `@skip(if: Boolean)`: شرط لتخطي Field
  - Custom Directives للـ Authentication/Authorization

**15. كيف تحسن أداء GraphQL API؟**
- **الإجابة**:
  - DataLoader لحل N+1 Problem
  - Query Depth Limiting
  - Query Complexity Analysis
  - Caching (Redis)
  - Batching & Deduplication
  - Persistent Queries

**16. ما هو Union Type في GraphQL؟**
- **الإجابة**: Union يسمح بإرجاع أنواع مختلفة:
```graphql
union SearchResult = User | Post | Comment

query {
  search(query: "test") {
    ... on User { name }
    ... on Post { title }
    ... on Comment { content }
  }
}
```

**17. ما الفرق بين Interface و Union في GraphQL؟**
- **الإجابة**:
  - **Interface**: Types تشترك في Fields مشتركة
  - **Union**: Types مختلفة تماماً بدون Fields مشتركة

**18. كيف تتعامل مع Errors في GraphQL؟**
- **الإجابة**:
  - رمي Error في Resolver
  - استخدام Custom Error Classes
  - Error Formatting في Server config
  - Errors تُرجع في `errors` array مع `data: null`

**19. ما هو Apollo Federation؟**
- **الإجابة**: Apollo Federation يسمح بـ:
  - تقسيم Schema إلى Microservices
  - كل Service يدير جزء من Schema
  - Gateway يوحد جميع Services
  - مناسب لـ Microservices Architecture

**20. كيف تنفذ Rate Limiting في GraphQL؟**
- **الإجابة**:
  - Query Complexity Analysis
  - Rate Limiting بـ IP/User
  - Throttling في Context middleware
  - Cost-based Rate Limiting

### أسئلة Best Practices

**21. ما هي Best Practices لتصميم Schema؟**
- **الإجابة**:
  - أسماء واضحة ومعبرة
  - استخدام Enums للقيم الثابتة
  - Input Types للـ Mutations
  - Nullable Fields فقط عند الحاجة
  - Pagination للـ Lists
  - Documentation في Schema

**22. متى تستخدم GraphQL بدلاً من REST؟**
- **الإجابة**: استخدم GraphQL عندما:
  - تطبيق معقد مع بيانات متداخلة
  - عملاء متعددين (Web, Mobile, etc.)
  - تحتاج تقليل Bandwidth
  - تحتاج Real-time Updates
  - تريد Schema قوي ومُوثق

**23. كيف تتعامل مع Caching في GraphQL؟**
- **الإجابة**:
  - Server-side: Redis/Memcached
  - Client-side: Apollo Client Cache
  - HTTP Caching محدود (دائماً POST)
  - Persisted Queries للـ GET
  - Cache Invalidation Strategies

**24. ما هو DataLoader وكيف يعمل؟**
- **الإجابة**: DataLoader هو Batching & Caching layer:
  - يجمع الطلبات في نفس Event Loop
  - يرسلها دفعة واحدة
  - يخزن النتائج مؤقتاً
  - يحل N+1 Problem

**25. كيف تختبر GraphQL API؟**
- **الإجابة**:
  - Unit Tests للـ Resolvers
  - Integration Tests للـ Queries/Mutations
  - Schema Validation Tests
  - Authorization Tests
  - Performance Tests
  - استخدام Apollo Server Testing

---

## الخلاصة

GraphQL هو تقنية قوية لبناء APIs حديثة:
- ✅ Single Endpoint مع مرونة كبيرة
- ✅ No Over-fetching/Under-fetching
- ✅ Strong Typing & Introspection
- ✅ Real-time Updates (Subscriptions)
- ✅ Developer Experience ممتاز
- ✅ Documentation تلقائي
- ✅ مناسب للتطبيقات المعقدة

**متى تستخدم GraphQL:**
- تطبيقات معقدة مع بيانات متداخلة
- تطبيقات موبايل (تقليل Bandwidth)
- عملاء متعددين بمتطلبات مختلفة
- عندما تحتاج Real-time Updates
- Microservices Architecture (مع Federation)

**متى تستخدم REST:**
- APIs بسيطة
- CRUD أساسي
- عندما تحتاج HTTP Caching
- File uploads/downloads كثيرة
- فريق بدون خبرة في GraphQL

**الأدوات الأساسية:**
- **Server**: Apollo Server, GraphQL Yoga
- **Client**: Apollo Client, urql, Relay
- **ORM**: Prisma, TypeORM
- **Testing**: Apollo Server Testing, Jest
- **Monitoring**: Apollo Studio, GraphQL Inspector