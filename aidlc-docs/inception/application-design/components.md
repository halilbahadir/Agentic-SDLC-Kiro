# Application Components

## Component Overview

The system is organized into feature-based components, each with clear boundaries and responsibilities.

---

## Frontend Components

### FC-01: Auth Module
**Purpose**: Handle user registration, login, session management, and profile setup  
**Responsibilities**:
- Registration form with email/password validation
- Login form with error handling
- Password reset flow
- Session token management (Cognito tokens)
- Profile setup wizard (calorie goals, dietary preferences, skill level)
- Protected route guards

### FC-02: Meal Logging Module
**Purpose**: Enable users to log meals via photo upload or manual entry  
**Responsibilities**:
- Camera/gallery image capture and upload
- AI analysis result display with confidence scores
- Food item confirmation/correction UI
- Manual food search with autocomplete
- Portion size selection (grams, cups, pieces, servings)
- Custom food item creation form
- Meal type selection (breakfast, lunch, dinner, snack)

### FC-03: Nutrition Dashboard Module
**Purpose**: Display nutrition data, daily progress, and goal tracking  
**Responsibilities**:
- Daily calorie progress ring/bar chart
- Macronutrient progress bars (protein, carbs, fat)
- Full nutrition breakdown display (vitamins, minerals)
- Daily/weekly summary views
- Goal exceeded/approaching warnings
- Meal history list with edit/delete

### FC-04: Ingredient Management Module
**Purpose**: Manage available ingredients via fridge photos or manual entry  
**Responsibilities**:
- Fridge photo capture and upload
- AI ingredient identification result display
- Manual ingredient add/remove/update
- Ingredient categorization display (protein, vegetable, dairy, grain)
- Ingredient list with quantity tracking

### FC-05: Recipe Advisor Module
**Purpose**: Display AI-generated recipe suggestions and details  
**Responsibilities**:
- Recipe suggestion request with filters (time, skill, dietary)
- Recipe card list display
- Full recipe detail view (ingredients, steps, nutrition, metadata)
- Recipe save/favorite functionality
- Filter controls (cooking time, skill level, dietary preferences)

### FC-06: Offline Sync Module
**Purpose**: Handle offline data access and synchronization  
**Responsibilities**:
- PouchDB local database management
- Sync status indicator
- Offline mode detection and UI feedback
- Mutation queue for offline changes
- Conflict resolution (last-write-wins)
- Cache management for viewed data

### FC-07: Shared UI Components
**Purpose**: Reusable UI elements across all modules  
**Responsibilities**:
- Navigation bar and routing
- Loading states and error boundaries
- Image upload component
- Search/autocomplete component
- Chart components (ring, bar, line)
- Toast notifications
- Theme management (dark/light)

---

## Backend Components (Lambda Functions)

### BC-01: Auth Service
**Purpose**: Handle authentication flows via AWS Cognito  
**Responsibilities**:
- User registration (Cognito User Pool)
- Login and token refresh
- Password reset initiation and confirmation
- User profile CRUD (DynamoDB)
- Token validation middleware

### BC-02: Meal Service
**Purpose**: Handle meal logging, nutrition calculation, and history  
**Responsibilities**:
- Meal CRUD operations
- Nutrition data aggregation per meal
- Daily/weekly nutrition summaries
- Meal history queries with pagination
- Food database search
- Custom food item management

### BC-03: AI Analysis Service
**Purpose**: Orchestrate AI-powered image analysis via AWS Bedrock  
**Responsibilities**:
- Meal image analysis (identify foods, estimate portions)
- Fridge image analysis (identify ingredients)
- Nutrition estimation from identified foods
- Confidence scoring for AI results
- Fallback handling for unrecognizable images

### BC-04: Recipe Service
**Purpose**: Generate and manage AI-powered recipe suggestions  
**Responsibilities**:
- Recipe generation via Bedrock (based on ingredients, goals, preferences)
- Recipe filtering (time, skill, dietary)
- Saved recipe management
- Nutrition calculation for generated recipes
- Recipe detail formatting

### BC-05: Ingredient Service
**Purpose**: Manage user ingredient inventory  
**Responsibilities**:
- Ingredient CRUD operations
- Ingredient categorization
- Ingredient search and autocomplete
- Inventory queries for recipe generation

### BC-06: Image Service
**Purpose**: Handle image upload and storage  
**Responsibilities**:
- Pre-signed URL generation for S3 uploads
- Image metadata storage
- Image retrieval URLs
- Image cleanup (orphaned images)

---

## Infrastructure Components

### IC-01: API Gateway
**Purpose**: Route HTTP requests to appropriate Lambda functions  
**Responsibilities**:
- REST API routing
- Request validation
- CORS configuration
- Rate limiting
- Cognito authorizer integration

### IC-02: DynamoDB Tables
**Purpose**: Persistent data storage  
**Responsibilities**:
- User profiles and preferences
- Meal logs and nutrition data
- Ingredient inventories
- Saved recipes
- Custom food items

### IC-03: S3 Buckets
**Purpose**: Image and static asset storage  
**Responsibilities**:
- Meal images (private per user)
- Fridge images (private per user)
- Frontend static assets (CloudFront origin)

### IC-04: CloudFront Distribution
**Purpose**: CDN for frontend and image delivery  
**Responsibilities**:
- Frontend SPA hosting
- Image delivery with caching
- HTTPS termination

### IC-05: Cognito User Pool
**Purpose**: Managed authentication  
**Responsibilities**:
- User registration and verification
- Login and token issuance
- Password reset
- Token refresh
