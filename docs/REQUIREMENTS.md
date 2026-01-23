# Requirements Document

## User Personas

### Persona 1: Sarah - The Busy Mother
- **Age:** 35-45
- **Description:** Working mother with 2-3 children, manages household meal planning
- **Goals:** 
  - Preserve grandmother's handwritten recipes digitally
  - Reduce daily "what's for dinner" stress
  - Keep track of what meals her family enjoys
  - Plan quick, healthy meals on busy weekdays
- **Pain Points:** 
  - Paper recipes are scattered, fading, and hard to search
  - Can't remember which meals the kids liked
  - Runs out of ideas for what to cook
  - Wastes time deciding what to make
- **Technical Proficiency:** Intermediate - comfortable with apps but not tech-savvy

### Persona 2: Emma - The Teenager/Family Member
- **Age:** 13-18
- **Description:** Child in the household who eats the meals
- **Goals:** 
  - Let mom know what meals she likes
  - Easy way to rate food without being asked
  - Access family recipes when living independently later
- **Pain Points:** 
  - Mom keeps asking "did you like it?"
  - Wants to give feedback but forgets
  - Doesn't want complicated systems
- **Technical Proficiency:** Advanced - smartphone native

---

## Functional Requirements

**Budget Constraint:** Low budget/hobby project - prioritize free or very low-cost services

### MVP Phase 1 (First Iteration) - Core Foundation

**Goal:** Get basic app working with 1 user in production

**Included Features:**
- Feature 7: Authentication (Microsoft OAuth)
- Feature 8: Recipe Management (CRUD operations)
- Feature 1: Photo Upload & OCR (basic text extraction)

---

#### Feature 7: User Authentication & Account Management (MVP Phase 1)
- **Description:** Microsoft OAuth authentication for secure login
- **User Story:** As a user, I want to sign in with my Microsoft account so that I don't need to manage another password
- **Acceptance Criteria:**
  - [ ] "Sign in with Microsoft" button
  - [ ] OAuth flow redirects to Microsoft login
  - [ ] User grants permission to access basic profile
  - [ ] App receives user info (name, email) from Microsoft
  - [ ] Session management with JWT tokens
  - [ ] User profile created automatically on first login
- **Priority:** High - Required for MVP
- **Estimated Effort:** Small
- **Benefits for Budget:**
  - FREE to implement (Microsoft OAuth is free)
  - No password management needed
  - No email verification needed (Microsoft already verified)
  - No password reset flows needed
  - More secure than custom auth
  - Users trust Microsoft login

---

#### Feature 8: Recipe Display & Management (MVP Phase 1)
- **Description:** Basic CRUD operations for recipes
- **User Story:** As a user, I want to manually add and view my recipes
- **Acceptance Criteria:**
  - [ ] Manually create recipe (title, ingredients, instructions)
  - [ ] List view of all user's recipes
  - [ ] View single recipe details
  - [ ] Edit recipe
  - [ ] Delete recipe (with confirmation)
  - [ ] Basic search by title
- **Priority:** High - Core functionality
- **Estimated Effort:** Small
- **Simplified for Budget:**
  - No advanced filtering yet
  - No sorting options initially
  - Simple text fields (no rich text editor)

---

#### Feature 1: Recipe Photo Upload & Basic OCR (MVP Phase 1)
- **Description:** Upload recipe photos and extract text using free OCR
- **User Story:** As a user, I want to photograph my recipe card so that I can preserve it digitally
- **Acceptance Criteria:**
  - [ ] User can upload photo from device
  - [ ] Image is stored (compressed)
  - [ ] Basic OCR extracts text from image
  - [ ] Extracted text shown for editing
  - [ ] User manually separates ingredients from instructions
  - [ ] Recipe saved with photo thumbnail
- **Priority:** High - Key differentiator
- **Estimated Effort:** Medium
- **Simplified for Budget:**
  - Use Tesseract.js (free, client-side OCR) initially
  - No AI auto-separation of ingredients/instructions
  - User manually organizes extracted text
  - Acceptable OCR accuracy (not perfect)
  - Process on client-side (no server OCR costs)

---

### MVP Phase 2 - Enhanced Features

**Add after Phase 1 is deployed and validated**

#### Feature 2: AI-Powered Mood-Based Recipe Suggestions (Phase 2)
- **Description:** Users describe their mood or situation in natural language and get intelligent recipe suggestions
- **User Story:** As a busy mother, I want to type "cold winter evening comfort food" so that I get relevant recipe suggestions without browsing
- **Priority:** Medium - Adds after basic recipe collection exists
- **Estimated Effort:** Large
- **Deferred Because:** Requires enough recipes in database first, and AI API costs

---

#### Feature 3: Recipe Filtering by Category (Phase 2)
- **Description:** Filter recipes by dietary categories (vegan, vegetarian, fish, meat, etc.)
- **User Story:** As a user, I want to filter by "vegetarian" so that I can quickly find meatless options
- **Priority:** Medium
- **Estimated Effort:** Medium
- **Deferred Because:** Need recipe collection first

---

#### Feature 4: Family Member Management & Rating System (Phase 2+)
- **Description:** Add family members and track their meal ratings using a heart-based system
- **Priority:** Medium
- **Deferred Because:** Nice-to-have after core recipe management works

---

#### Feature 5: Shareable Rating Links (Phase 2+)
- **Description:** Send email links to family members so they can rate meals themselves
- **Priority:** Medium
- **Deferred Because:** Requires email service costs

---

#### Feature 6: Family Account Linking (Phase 3)
- **Description:** Connect accounts with family members who also have FoodByTheBook accounts
- **User Story:** As a mother, I want to connect with my daughter's account so that we can share recipes and ratings
- **Acceptance Criteria:**
  - [ ] User can send connection request by entering family member's username
  - [ ] Cannot search/list users (privacy protection)
  - [ ] Connection requires approval from both parties
  - [ ] Connected accounts can share recipes (optional setting)
  - [ ] Connected accounts can see each other's ratings on shared recipes
  - [ ] User can remove connection at any time
- **Priority:** Low - Future enhancement
- **Estimated Effort:** Medium
- **Technical Requirements:**
  - User relationship system
  - Privacy controls
  - Approval workflow

---

### Future Features (Nice to Have)

---

#### Feature 6: Family Account Linking (Phase 3)
- [ ] Page load time < 3 seconds
- [ ] API response time < 500ms for 95th percentile
- [ ] Support [X] concurrent users
- [ ] Database query optimization

### Security
- [ ] User authentication (OAuth, JWT, etc.)
- [ ] Role-based access control (RBAC)
- [ ] Data encryption at rest and in transit
- [ ] HTTPS/SSL certificates
- [ ] Input validation and sanitization
- [ ] Protection against OWASP Top 10 vulnerabilities
- [ ] Regular security audits
- [ ] Secure password policies

### Scalability
- [ ] Horizontal scaling capability
- [ ] Load balancing
- [ ] Caching strategy
- [ ] Database optimization
- [ ] CDN for static assets

### Availability & Reliability
- [ ] 99.9% uptime target
- [ ] Automated backups (daily/weekly)
- [ ] Disaster recovery plan
- [ ] Health monitoring
- [ ] Error logging and alerting

### Usability
- [ ] Responsive design (mobile, tablet, desktop)
- [ ] Accessibility compliance (WCAG 2.1 Level AA)
- [ ] Intuitive navigation
- [ ] Clear error messages
- [ ] User onboarding flow

### Browser/Device Support
- [ ] Chrome (latest 2 versions)
- [ ] Firefox (latest 2 versions)
- [ ] Safari (latest 2 versions)
- [ ] Edge (latest 2 versions)
- [ ] iOS Safari (latest 2 versions)
- [ ] Android Chrome (latest 2 versions)

### Compliance & Legal
- [ ] GDPR compliance (if applicable)
- [ ] Privacy policy
- [ ] Terms of service
- [ ] Cookie consent
- [ ] Data retention policies

---

## API Requirements

### Endpoints Needed
[List the main API endpoints you'll need]

1. **[HTTP Method] /api/[endpoint]**
   - Purpose: [What it does]
   - Auth: Required/Optional
   - Parameters: [List parameters]
   - Response: [What it returns]

---

## Data Requirements

### Entities
[List main data entities/models]

1. **[Entity Name]**
   - Field 1: [type] - [description]
   - Field 2: [type] - [description]
   - Relationships: [to other entities]

---

## Integration Requirements
[Third-party services or APIs you need to integrate]

- [ ] **OCR Service** - Azure Computer Vision, Google Vision API, or Tesseract
- [ ] **AI/NLP Service** - OpenAI GPT-4, Azure OpenAI, or Claude for mood-based suggestions
- [ ] **Email Service** - SendGrid, Mailgun, AWS SES for rating links and notifications
- [ ] **Image Storage** - Azure Blob Storage, AWS S3, Cloudinary for recipe photos
- [ ] **Image Processing** - Sharp, ImageMagick for thumbnail generation and compression
- [ ] **Authentication** - Auth0, Firebase Auth, or custom JWT implementation
- [ ] **Analytics** - (Optional) Google Analytics, Mixpanel for usage tracking
- [ ] **Monitoring** - Sentry, Datadog for error tracking and performance

---

## Out of Scope
[What you explicitly WON'T build in this version]

- [Feature/capability]
- [Feature/capability]
