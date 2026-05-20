# Requirements Document: Calorie Counter & Recipe Advisor

## Intent Analysis

| Attribute | Value |
|-----------|-------|
| **User Request** | Build an application to count calories per meal using image uploads and manual food lists with AI estimation, plus a recipe advisor component that suggests what to cook based on fridge photos or manual ingredient lists |
| **Request Type** | New Project (Greenfield) |
| **Scope Estimate** | Multiple Components (Calorie Tracker + Recipe Advisor + Shared AI/Data Layer) |
| **Complexity Estimate** | Complex (AI/ML integration, image processing, multi-component, full nutritional tracking, meal planning) |

---

## Functional Requirements

### FR-01: User Authentication & Account Management
- Users register and log in with email/password
- Password reset via email
- User profile with dietary preferences, goals, and cooking skill level
- Session management with secure token-based auth

### FR-02: Meal Image Upload & Analysis
- Users upload photos of their meals
- AI analyzes images to identify food items and estimate portions
- System returns identified foods with confidence scores
- Users can confirm, correct, or add items the AI missed
- Support common image formats (JPEG, PNG, HEIC)
- Maximum image size: 10MB

### FR-03: Manual Food Entry
- Users manually add food items by name
- Users specify portion size (grams, cups, pieces, servings)
- Autocomplete/search from food database
- Custom food item creation with manual nutrition data

### FR-04: AI Calorie & Nutrition Estimation
- Cloud-based AI service (AWS Bedrock) estimates calories from images
- Full nutritional breakdown per food item:
  - Calories (kcal)
  - Macronutrients: protein (g), carbohydrates (g), fat (g)
  - Fiber (g)
  - Key vitamins (A, C, D, B12)
  - Key minerals (iron, calcium, potassium, sodium)
- Aggregate nutrition totals per meal
- Confidence level indicator for AI estimates

### FR-05: Meal Logging & History
- Log meals by type: breakfast, lunch, dinner, snack
- Timestamp each meal entry
- View meal history by day, week, month
- Edit or delete past meal entries
- Add notes to meals

### FR-06: Daily/Weekly Goals & Progress Tracking
- Set daily calorie target
- Set macronutrient targets (protein, carbs, fat)
- Visual progress charts:
  - Daily intake vs. goal (bar/ring chart)
  - Weekly trend line chart
  - Monthly summary
- Streak tracking (consecutive days meeting goals)
- Notifications/alerts when approaching or exceeding daily limits

### FR-07: Refrigerator/Pantry Inventory
- Upload photos of refrigerator contents
- AI identifies available ingredients from photos
- Manual ingredient list entry (add/remove items)
- Ingredient categorization (proteins, vegetables, dairy, grains, etc.)
- Expiration date tracking (optional, manual entry)

### FR-08: AI Recipe Advisor
- Generate recipe suggestions based on:
  - Available ingredients (from fridge photos or manual list)
  - User's calorie/nutrition goals (remaining daily budget)
  - Dietary preferences (vegan, vegetarian, keto, gluten-free, etc.)
  - Cooking time preference (quick <30min, medium 30-60min, long >60min)
  - Skill level (beginner, intermediate, advanced)
- Each recipe includes:
  - Name and description
  - Ingredient list with quantities
  - Step-by-step instructions
  - Estimated nutrition breakdown
  - Prep time and cook time
  - Difficulty level
  - Serving size

### FR-09: Full Meal Planning
- Suggest complete daily meal plans (breakfast, lunch, dinner)
- Balance nutrition across meals to meet daily goals
- Consider available ingredients across all meals
- Allow swapping individual meals within a plan
- Save favorite meal plans for reuse
- Weekly meal plan generation

### FR-10: Data Storage & Sync
- Local storage for offline access (IndexedDB/localStorage)
- Cloud sync when online (AWS DynamoDB)
- Conflict resolution for multi-device edits (last-write-wins)
- Data export capability (CSV, JSON)

---

## Non-Functional Requirements

### NFR-01: Performance
- Image upload and AI analysis: < 10 seconds response time
- Page load time: < 2 seconds
- Recipe generation: < 15 seconds
- Smooth UI interactions (60fps animations)

### NFR-02: Scalability
- Support up to 10,000 concurrent users initially
- Serverless architecture (AWS Lambda) for auto-scaling
- Image storage in S3 with CDN distribution

### NFR-03: Availability
- 99.5% uptime target
- Graceful degradation when cloud services unavailable
- Offline mode for viewing cached data and manual entry

### NFR-04: Usability
- Responsive web design (mobile-first)
- Intuitive food logging (< 3 taps to log a meal)
- Accessible (WCAG 2.1 AA compliance target)
- Support for dark/light themes

### NFR-05: Data Privacy
- User data encrypted at rest (DynamoDB encryption)
- HTTPS for all communications
- Images stored privately per user (S3 bucket policies)
- User can delete all their data (right to erasure)

### NFR-06: AI Accuracy
- Calorie estimation within ±20% for common foods
- Ingredient identification accuracy > 80% for clear images
- Graceful handling of unrecognizable images (prompt manual entry)

---

## Technical Architecture Overview

| Layer | Technology |
|-------|-----------|
| **Frontend** | TypeScript, React, Vite |
| **Backend** | TypeScript, Node.js, AWS Lambda |
| **Database** | AWS DynamoDB (cloud), IndexedDB (local) |
| **AI/ML** | AWS Bedrock (Claude/Nova for image analysis and recipe generation) |
| **Storage** | AWS S3 (images) |
| **Auth** | AWS Cognito or custom JWT |
| **API** | REST API via API Gateway |
| **Hosting** | AWS CloudFront + S3 (frontend), Lambda (backend) |

---

## User Roles

| Role | Description |
|------|-------------|
| **User** | Primary user who logs meals, tracks calories, and gets recipe suggestions |
| **Admin** | (Future) Manage food database, monitor system health |

---

## Constraints

- Cloud AI requires internet connection for image analysis and recipe generation
- Offline mode limited to cached data viewing and manual entry
- AI estimates are approximations, not medical-grade measurements
- Recipe suggestions depend on quality of ingredient identification

---

## Success Criteria

1. Users can log a meal (photo or manual) and see full nutrition breakdown within 10 seconds
2. Daily/weekly progress charts accurately reflect logged data
3. Recipe advisor generates relevant suggestions based on available ingredients and goals
4. Meal plans balance nutrition across the day within ±10% of targets
5. Data syncs reliably between local and cloud storage

---

## Out of Scope (v1)

- Social features (sharing meals, community recipes)
- Barcode scanning for packaged foods
- Integration with fitness trackers/wearables
- Grocery shopping list generation
- Restaurant menu integration
- Admin dashboard
