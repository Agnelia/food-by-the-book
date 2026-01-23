# Data Models

## Overview
This document defines the data models for the application, including entities, relationships, and constraints.

---

## Entities

### User
**Description:** Represents a user in the system

**Fields:**
| Field | Type | Required | Unique | Default | Description |
|-------|------|----------|--------|---------|-------------|
| id | UUID | Yes | Yes | auto | Primary key |
| email | String | Yes | Yes | - | User's email address |
| password_hash | String | Yes | No | - | Hashed password |
| username | String | Yes | Yes | - | Unique username for account linking |
| first_name | String | Yes | No | - | User's first name |
| last_name | String | Yes | No | - | User's last name |
| is_active | Boolean | Yes | No | true | Account active status |
| email_verified | Boolean | Yes | No | false | Email verification status |
| created_at | Timestamp | Yes | No | now() | Account creation date |
| updated_at | Timestamp | Yes | No | now() | Last update date |
| last_login | Timestamp | No | No | null | Last login timestamp |

**Indexes:**
- PRIMARY KEY (id)
- UNIQUE INDEX (email)
- UNIQUE INDEX (username)
- INDEX (created_at)

**Validation Rules:**
- email: Valid email format
- password: Min 8 characters, must include uppercase, lowercase, number, special char
- username: 3-20 characters, alphanumeric + underscore only
- first_name, last_name: 1-50 characters

---

### Recipe
**Description:** A digitized recipe with extracted text and original image

**Fields:**
| Field | Type | Required | Unique | Default | Description |
|-------|------|----------|--------|---------|-------------|
| id | UUID | Yes | Yes | auto | Primary key |
| user_id | UUID | Yes | No | - | Foreign key to User |
| title | String | Yes | No | - | Recipe name/title |
| ingredients | Text | Yes | No | - | List of ingredients (structured or plain text) |
| instructions | Text | Yes | No | - | Cooking instructions/steps |
| original_image_url | String | No | No | null | URL to full-size uploaded photo |
| thumbnail_url | String | No | No | null | URL to compressed thumbnail |
| servings | Integer | No | No | null | Number of servings |
| prep_time | Integer | No | No | null | Prep time in minutes |
| cook_time | Integer | No | No | null | Cook time in minutes |
| categories | JSON/Array | No | No | [] | Array of category tags (vegan, fish, etc.) |
| ocr_confidence | Float | No | No | null | OCR extraction confidence score |
| is_public | Boolean | Yes | No | false | Whether recipe is publicly visible |
| created_at | Timestamp | Yes | No | now() | Creation date |
| updated_at | Timestamp | Yes | No | now() | Last update date |

**Indexes:**
- PRIMARY KEY (id)
- FOREIGN KEY (user_id) REFERENCES users(id)
- INDEX (user_id, created_at)
- INDEX (categories) - for filtering
- FULLTEXT INDEX (title, ingredients) - for search

**Validation Rules:**
- title: 1-200 characters
- ingredients: max 5000 characters
- instructions: max 10000 characters
- servings: 1-100
- prep_time, cook_time: 0-999 minutes

---

### FamilyMember
**Description:** Represents a family member who can rate recipes

**Fields:**
| Field | Type | Required | Unique | Default | Description |
|-------|------|----------|--------|---------|-------------|
| id | UUID | Yes | Yes | auto | Primary key |
| user_id | UUID | Yes | No | - | Foreign key to User (the parent account) |
| name | String | Yes | No | - | Family member's name |
| email | String | No | No | null | Optional email for sending rating links |
| linked_user_id | UUID | No | No | null | If family member has own account, link to it |
| connection_approved | Boolean | No | No | null | Approval status for account linking |
| created_at | Timestamp | Yes | No | now() | Creation date |
| updated_at | Timestamp | Yes | No | now() | Last update date |

**Indexes:**
- PRIMARY KEY (id)
- FOREIGN KEY (user_id) REFERENCES users(id)
- FOREIGN KEY (linked_user_id) REFERENCES users(id)
- INDEX (user_id)
- INDEX (email)

**Validation Rules:**
- name: 1-50 characters
- email: Valid email format if provided

---

### Rating
**Description:** Stores recipe ratings from family members

**Fields:**
| Field | Type | Required | Unique | Default | Description |
|-------|------|----------|--------|---------|-------------|
| id | UUID | Yes | Yes | auto | Primary key |
| recipe_id | UUID | Yes | No | - | Foreign key to Recipe |
| family_member_id | UUID | Yes | No | - | Foreign key to FamilyMember |
| hearts | Integer | Yes | No | - | Rating value (1-5 hearts) |
| comment | Text | No | No | null | Optional text feedback |
| rated_at | Timestamp | Yes | No | now() | When rating was given |

**Indexes:**
- PRIMARY KEY (id)
- FOREIGN KEY (recipe_id) REFERENCES recipes(id)
- FOREIGN KEY (family_member_id) REFERENCES family_members(id)
- UNIQUE INDEX (recipe_id, family_member_id) - One rating per person per recipe (or allow multiple with updated_at)
- INDEX (recipe_id) - For calculating average ratings

**Validation Rules:**
- hearts: 1-5 integer
- comment: max 500 characters

---

### RatingLink
**Description:** Temporary shareable links for rating recipes

**Fields:**
| Field | Type | Required | Unique | Default | Description |
|-------|------|----------|--------|---------|-------------|
| id | UUID | Yes | Yes | auto | Primary key |
| token | String | Yes | Yes | - | Unique shareable token |
| recipe_id | UUID | Yes | No | - | Foreign key to Recipe |
| family_member_id | UUID | Yes | No | - | Foreign key to FamilyMember |
| expires_at | Timestamp | Yes | No | - | Link expiration time |
| used_at | Timestamp | No | No | null | When link was used (if single-use) |
| created_at | Timestamp | Yes | No | now() | Creation date |

**Indexes:**
- PRIMARY KEY (id)
- UNIQUE INDEX (token)
- FOREIGN KEY (recipe_id) REFERENCES recipes(id)
- FOREIGN KEY (family_member_id) REFERENCES family_members(id)
- INDEX (expires_at) - For cleanup of expired links

**Validation Rules:**
- token: 32+ character random string
- expires_at: Must be future date

---

### UserConnection
**Description:** Approved connections between user accounts

**Fields:**
| Field | Type | Required | Unique | Default | Description |
|-------|------|----------|--------|---------|-------------|
| id | UUID | Yes | Yes | auto | Primary key |
| user_id_1 | UUID | Yes | No | - | First user in connection |
| user_id_2 | UUID | Yes | No | - | Second user in connection |
| status | Enum | Yes | No | pending | pending, approved, rejected |
| requested_by | UUID | Yes | No | - | User who initiated request |
| requested_at | Timestamp | Yes | No | now() | Request timestamp |
| responded_at | Timestamp | No | No | null | Response timestamp |
| share_recipes | Boolean | Yes | No | false | Whether to share recipes |

**Indexes:**
- PRIMARY KEY (id)
- FOREIGN KEY (user_id_1) REFERENCES users(id)
- FOREIGN KEY (user_id_2) REFERENCES users(id)
- UNIQUE INDEX (user_id_1, user_id_2) - Prevent duplicate connections
- INDEX (user_id_1, status)
- INDEX (user_id_2, status)

**Validation Rules:**
- user_id_1 != user_id_2
- status: 'pending', 'approved', 'rejected'

---

## Relationships

### User Relationships
```
User (1) ──── (Many) Recipe
  - Relationship type: One-to-Many
  - Foreign key: user_id in Recipe
  - Cascade: ON DELETE CASCADE (delete recipes when user deleted)

User (1) ──── (Many) FamilyMember
  - Relationship type: One-to-Many  
  - Foreign key: user_id in FamilyMember
  - Cascade: ON DELETE CASCADE

User (Many) ──── (Many) User (via UserConnection)
  - Relationship type: Many-to-Many (self-referencing)
  - Junction table: UserConnection
  - Represents: Family account connections
```

### Recipe Relationships
```
Recipe (1) ──── (Many) Rating
  - Relationship type: One-to-Many
  - Foreign key: recipe_id in Rating
  - Cascade: ON DELETE CASCADE (delete ratings when recipe deleted)

Recipe (1) ──── (Many) RatingLink
  - Relationship type: One-to-Many
  - Foreign key: recipe_id in RatingLink  
  - Cascade: ON DELETE CASCADE
```

### FamilyMember Relationships
```
FamilyMember (1) ──── (Many) Rating
  - Relationship type: One-to-Many
  - Foreign key: family_member_id in Rating
  - Cascade: ON DELETE CASCADE

FamilyMember (1) ──── (Many) RatingLink
  - Relationship type: One-to-Many
  - Foreign key: family_member_id in RatingLink
  - Cascade: ON DELETE CASCADE

FamilyMember (0..1) ──── (1) User (linked_user_id)
  - Relationship type: Optional One-to-One
  - Foreign key: linked_user_id in FamilyMember
  - Cascade: ON DELETE SET NULL (preserve family member if linked user deleted)
```

---

## Enums

### RatingStatus (UserConnection)
```typescript
enum ConnectionStatus {
  PENDING = 'pending',
  APPROVED = 'approved',
  REJECTED = 'rejected'
}
```

### RecipeCategory
```typescript
enum RecipeCategory {
  VEGAN = 'vegan',
  VEGETARIAN = 'vegetarian',
  FISH = 'fish',
  MEAT = 'meat',
  POULTRY = 'poultry',
  GLUTEN_FREE = 'gluten_free',
  DAIRY_FREE = 'dairy_free',
  LOW_CARB = 'low_carb',
  DESSERT = 'dessert',
  BREAKFAST = 'breakfast',
  LUNCH = 'lunch',
  DINNER = 'dinner',
  SNACK = 'snack'
}
```

---

## Database Schema (SQL)

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  role VARCHAR(20) NOT NULL DEFAULT 'user',
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  last_login TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Add your tables below
```

---

## TypeScript Types/Interfaces

```typescript
// User interface
interface User {
  id: string;
  email: string;
  password_hash: string;
  first_name: string;
  last_name: string;
  role: UserRole;
  is_active: boolean;
  created_at: Date;
  updated_at: Date;
  last_login?: Date;
}

// User creation DTO
interface CreateUserDto {
  email: string;
  password: string;
  first_name: string;
  last_name: string;
  role?: UserRole;
}

// User response DTO (without sensitive data)
interface UserResponseDto {
  id: string;
  email: string;
  first_name: string;
  last_name: string;
  role: UserRole;
  created_at: Date;
}

// Add your interfaces below
```

---

## Sample Data

### User
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "user@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "role": "user",
  "is_active": true,
  "created_at": "2026-01-23T10:00:00Z",
  "updated_at": "2026-01-23T10:00:00Z"
}
```

### [Add Sample Data for Your Entities]

---

## Migration Strategy

### Initial Migration
```sql
-- migrations/001_create_users_table.sql
CREATE TABLE users (
  -- schema here
);
```

### Migration Checklist
- [ ] Write migration up script
- [ ] Write migration down script (rollback)
- [ ] Test migration on dev database
- [ ] Add seed data if needed
- [ ] Document migration in changelog

---

## Data Access Patterns

### Common Queries

#### Get User by Email
```sql
SELECT * FROM users WHERE email = $1;
```

#### Get Active Users
```sql
SELECT * FROM users WHERE is_active = true ORDER BY created_at DESC;
```

### [Add Your Common Queries]

---

## Caching Strategy

### Cached Entities
[List what should be cached]
- User sessions: 1 hour TTL
- [Entity]: [Duration] TTL

### Cache Keys
```
user:{id}
user:email:{email}
[Add your cache keys]
```

---

## Data Validation

### Validation Rules
```typescript
const userValidationSchema = {
  email: {
    type: 'string',
    format: 'email',
    required: true
  },
  password: {
    type: 'string',
    minLength: 8,
    pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])',
    required: true
  },
  first_name: {
    type: 'string',
    minLength: 1,
    maxLength: 50,
    required: true
  }
  // Add more fields
};
```

---

## Notes & Decisions

### [Date] - [Decision]
[Document important decisions about data modeling]
