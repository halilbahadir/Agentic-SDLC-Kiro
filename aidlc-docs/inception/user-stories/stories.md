# User Stories

## Story Organization: User Journey-Based

Stories are organized by user journeys, with each journey representing a meaningful workflow through the application.

---

## Journey 1: Account & Onboarding

### US-01: User Registration
**As** Alex or Sam, **I want to** create an account with my email and password **so that** my data is saved and accessible across sessions.

**Persona**: Both  
**Effort**: S  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am on the registration page, When I enter a valid email and password (min 8 chars), Then my account is created and I am logged in
- Given I enter an email already in use, When I submit the form, Then I see an error message "Email already registered"
- Given I enter an invalid email format, When I submit the form, Then I see a validation error
- Given registration succeeds, When I am redirected to the app, Then I see an onboarding prompt to set my profile

---

### US-02: User Login & Session Management
**As** Alex or Sam, **I want to** log in to my account **so that** I can access my saved data.

**Persona**: Both  
**Effort**: S  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I have an account, When I enter correct credentials, Then I am logged in and see my dashboard
- Given I enter incorrect credentials, When I submit, Then I see "Invalid email or password"
- Given I am logged in, When I close and reopen the app within 7 days, Then I remain logged in (token-based)
- Given I click "Forgot Password", When I enter my email, Then I receive a password reset link

---

### US-03: Profile Setup
**As** Alex or Sam, **I want to** set up my profile with dietary preferences and goals **so that** the app personalizes my experience.

**Persona**: Both  
**Effort**: M  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am setting up my profile, When I set a daily calorie target, Then it is saved and used for goal tracking
- Given I am setting up my profile, When I select dietary preferences (vegan, keto, etc.), Then recipe suggestions respect these preferences
- Given I am setting up my profile, When I set my cooking skill level (beginner/intermediate/advanced), Then recipes are filtered accordingly
- Given I complete profile setup, When I go to the dashboard, Then I see personalized content based on my settings

---

## Journey 2: Meal Logging (Photo-Based)

### US-04: Upload Meal Photo for AI Analysis
**As** Alex, **I want to** take or upload a photo of my meal **so that** the AI identifies the food items automatically.

**Persona**: Alex (Calorie Tracker)  
**Effort**: L  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am on the meal logging screen, When I tap "Add Photo", Then I can choose camera or gallery
- Given I upload a clear food photo, When the AI processes it, Then I see identified food items within 10 seconds
- Given the AI identifies foods, When results are displayed, Then each item shows name, estimated portion, and confidence score
- Given the photo is unclear or unrecognizable, When AI cannot identify items, Then I see a message prompting manual entry
- Given the image exceeds 10MB, When I try to upload, Then I see a size limit error

---

### US-05: Confirm or Correct AI Food Identification
**As** Alex, **I want to** review and correct the AI's food identification **so that** my calorie log is accurate.

**Persona**: Alex (Calorie Tracker)  
**Effort**: M  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given AI has identified food items, When I review the list, Then I can confirm each item or edit its name/portion
- Given an item is incorrectly identified, When I tap to edit, Then I can search for the correct food and update it
- Given the AI missed an item, When I tap "Add Item", Then I can manually add the missing food
- Given I have confirmed all items, When I tap "Save Meal", Then the meal is logged with corrected nutrition data

---

## Journey 3: Meal Logging (Manual Entry)

### US-06: Manual Food Entry with Search
**As** Alex, **I want to** manually search and add food items to my meal **so that** I can log meals without photos.

**Persona**: Alex (Calorie Tracker)  
**Effort**: M  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am on the meal logging screen, When I tap "Manual Entry", Then I see a search field with autocomplete
- Given I type a food name, When results appear, Then I see matching foods with calorie info per serving
- Given I select a food item, When I choose it, Then I can specify portion size (grams, cups, pieces, servings)
- Given I add multiple items, When I view the meal summary, Then I see total nutrition for all items combined

---

### US-07: Custom Food Item Creation
**As** Alex, **I want to** create custom food items with manual nutrition data **so that** I can log foods not in the database.

**Persona**: Alex (Calorie Tracker)  
**Effort**: S  
**Business Value**: Medium  
**MVP**: Yes

**Acceptance Criteria:**
- Given a food is not found in search, When I tap "Create Custom Food", Then I can enter name and nutrition values
- Given I enter custom food details, When I save it, Then it appears in my personal food database for future use
- Given I have custom foods saved, When I search for food, Then my custom items appear alongside database results

---

## Journey 4: Nutrition Tracking & Goals

### US-08: View Meal Nutrition Breakdown
**As** Alex, **I want to** see the full nutritional breakdown of each meal **so that** I understand what I'm consuming.

**Persona**: Alex (Calorie Tracker)  
**Effort**: M  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I have logged a meal, When I view meal details, Then I see calories, protein, carbs, fat, fiber
- Given I view meal details, When I expand nutrition, Then I see vitamins (A, C, D, B12) and minerals (iron, calcium, potassium, sodium)
- Given I have multiple meals logged today, When I view the daily summary, Then I see aggregated totals across all meals

---

### US-09: Daily Calorie & Macro Goal Tracking
**As** Alex, **I want to** see my progress toward daily calorie and macronutrient goals **so that** I stay on track.

**Persona**: Alex (Calorie Tracker)  
**Effort**: M  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I have set daily targets, When I view my dashboard, Then I see a visual progress indicator (ring/bar) for calories consumed vs. target
- Given I have logged meals today, When I check macros, Then I see protein/carbs/fat progress bars with remaining amounts
- Given I am approaching my daily limit (>90%), When I view the dashboard, Then I see a warning indicator
- Given I exceed my daily target, When I view the dashboard, Then the progress indicator shows overage clearly

---

### US-10: Weekly/Monthly Progress Charts
**As** Alex, **I want to** view my nutrition trends over time **so that** I can identify patterns and adjust my habits.

**Persona**: Alex (Calorie Tracker)  
**Effort**: L  
**Business Value**: Medium  
**MVP**: No (v2)

**Acceptance Criteria:**
- Given I have logged meals for multiple days, When I view weekly charts, Then I see a line chart of daily calorie intake vs. target
- Given I view monthly summary, When I check the chart, Then I see average daily intake, best/worst days, and streak count
- Given I want to compare macros over time, When I select a macro, Then the chart updates to show that nutrient's trend

---

## Journey 5: Ingredient Management

### US-11: Upload Fridge Photo for Ingredient Detection
**As** Sam, **I want to** take a photo of my fridge **so that** the AI identifies available ingredients automatically.

**Persona**: Sam (Home Cook)  
**Effort**: L  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am on the ingredients screen, When I tap "Scan Fridge", Then I can take or upload a photo
- Given I upload a fridge photo, When the AI processes it, Then I see a list of identified ingredients within 10 seconds
- Given ingredients are identified, When I review the list, Then I can confirm, remove, or add missing items
- Given some items are unrecognizable, When AI cannot identify them, Then they are flagged for manual entry

---

### US-12: Manual Ingredient List Management
**As** Sam, **I want to** manually add and remove ingredients from my pantry list **so that** I keep an accurate inventory.

**Persona**: Sam (Home Cook)  
**Effort**: S  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am on the ingredients screen, When I tap "Add Ingredient", Then I can search or type an ingredient name
- Given I have ingredients listed, When I tap an item, Then I can remove it or update its quantity
- Given I add an ingredient, When it is saved, Then it is categorized automatically (protein, vegetable, dairy, grain, etc.)
- Given I have a populated list, When I view it, Then ingredients are grouped by category

---

## Journey 6: Recipe Discovery

### US-13: Get AI Recipe Suggestions Based on Ingredients
**As** Sam, **I want to** get recipe suggestions based on my available ingredients **so that** I can cook without buying more food.

**Persona**: Sam (Home Cook)  
**Effort**: XL  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I have ingredients in my list, When I tap "Suggest Recipes", Then the AI generates 3-5 recipe suggestions within 15 seconds
- Given recipes are generated, When I view a suggestion, Then I see which of my ingredients are used and which extras are needed
- Given I have dietary preferences set, When recipes are generated, Then all suggestions respect my preferences
- Given I have a cooking time preference, When I filter recipes, Then only recipes within my time limit are shown

---

### US-14: View Recipe Details
**As** Sam, **I want to** view full recipe details including ingredients, steps, and nutrition **so that** I can follow the recipe.

**Persona**: Sam (Home Cook)  
**Effort**: M  
**Business Value**: High  
**MVP**: Yes

**Acceptance Criteria:**
- Given I select a recipe suggestion, When I view details, Then I see: name, description, ingredient list with quantities, step-by-step instructions
- Given I view recipe details, When I check nutrition, Then I see estimated calories, protein, carbs, fat per serving
- Given I view recipe details, When I check metadata, Then I see prep time, cook time, difficulty level, and serving size
- Given I like a recipe, When I tap "Save", Then it is added to my saved recipes for future reference

---

### US-15: Filter Recipes by Preferences
**As** Sam, **I want to** filter recipe suggestions by cooking time, skill level, and dietary needs **so that** I find recipes that fit my situation.

**Persona**: Sam (Home Cook)  
**Effort**: M  
**Business Value**: Medium  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am viewing recipe suggestions, When I set a time filter (<30min, 30-60min, >60min), Then only matching recipes are shown
- Given I set a skill level filter, When recipes refresh, Then only recipes at or below my skill level appear
- Given I have multiple filters active, When I view results, Then all filters are applied simultaneously
- Given no recipes match my filters, When I view results, Then I see a message suggesting I relax filters

---

## Journey 7: Meal History & Data Management

### US-16: View and Edit Meal History
**As** Alex, **I want to** view my past meals and edit entries **so that** I can correct mistakes and review my eating patterns.

**Persona**: Alex (Calorie Tracker)  
**Effort**: M  
**Business Value**: Medium  
**MVP**: Yes

**Acceptance Criteria:**
- Given I have logged meals, When I view history, Then I see meals grouped by day with meal type labels (breakfast, lunch, dinner, snack)
- Given I view a past meal, When I tap it, Then I see full details and can edit or delete it
- Given I edit a past meal, When I save changes, Then daily totals and progress charts update accordingly
- Given I want to find a specific meal, When I scroll through history, Then meals are ordered chronologically (newest first)

---

### US-17: Offline Access & Cloud Sync
**As** Alex or Sam, **I want to** use the app offline and have my data sync when I'm back online **so that** I never lose my entries.

**Persona**: Both  
**Effort**: L  
**Business Value**: Medium  
**MVP**: Yes

**Acceptance Criteria:**
- Given I am offline, When I log a meal manually, Then it is saved locally and synced when I reconnect
- Given I am offline, When I view my history, Then I see all previously cached data
- Given I am offline, When I try to use AI features (photo analysis, recipe generation), Then I see a message that internet is required
- Given I come back online, When the app syncs, Then local changes are pushed to the cloud without data loss
- Given I use the app on two devices, When both sync, Then the most recent changes are preserved (last-write-wins)

---

## Journey 8: Full Meal Planning (v2)

### US-18: Generate Daily Meal Plan
**As** Sam, **I want to** get a complete daily meal plan (breakfast, lunch, dinner) **so that** my nutrition is balanced across the day.

**Persona**: Sam (Home Cook)  
**Effort**: XL  
**Business Value**: High  
**MVP**: No (v2)

**Acceptance Criteria:**
- Given I request a meal plan, When the AI generates it, Then I see breakfast, lunch, and dinner suggestions
- Given a meal plan is generated, When I view nutrition totals, Then the combined meals meet my daily calorie and macro targets (±10%)
- Given a meal plan uses my ingredients, When I review it, Then it prioritizes items I already have
- Given I don't like a suggested meal, When I tap "Swap", Then an alternative is generated for that slot

---

### US-19: Weekly Meal Plan Generation
**As** Sam, **I want to** generate a weekly meal plan **so that** I can plan ahead and reduce daily decision fatigue.

**Persona**: Sam (Home Cook)  
**Effort**: XL  
**Business Value**: Medium  
**MVP**: No (v2)

**Acceptance Criteria:**
- Given I request a weekly plan, When the AI generates it, Then I see 7 days of breakfast/lunch/dinner suggestions
- Given a weekly plan is generated, When I view it, Then there is variety across days (no repeated meals within 3 days)
- Given I save a weekly plan, When I view it later, Then I can access it from my saved plans
- Given I want to reuse a plan, When I select a saved plan, Then I can apply it to the current week

---

### US-20: Save Favorite Meal Plans
**As** Sam, **I want to** save and reuse meal plans I liked **so that** I don't have to regenerate them.

**Persona**: Sam (Home Cook)  
**Effort**: S  
**Business Value**: Low  
**MVP**: No (v2)

**Acceptance Criteria:**
- Given I have a meal plan displayed, When I tap "Save Plan", Then it is stored in my saved plans list
- Given I have saved plans, When I view the list, Then I see plan name, date created, and calorie summary
- Given I select a saved plan, When I tap "Use This Plan", Then it becomes my active plan for the selected period

---

## Story Summary

| ID | Story | Persona | Effort | Value | MVP |
|----|-------|---------|--------|-------|-----|
| US-01 | User Registration | Both | S | High | ✅ |
| US-02 | User Login & Session | Both | S | High | ✅ |
| US-03 | Profile Setup | Both | M | High | ✅ |
| US-04 | Upload Meal Photo | Alex | L | High | ✅ |
| US-05 | Confirm/Correct AI ID | Alex | M | High | ✅ |
| US-06 | Manual Food Entry | Alex | M | High | ✅ |
| US-07 | Custom Food Item | Alex | S | Medium | ✅ |
| US-08 | Nutrition Breakdown | Alex | M | High | ✅ |
| US-09 | Daily Goal Tracking | Alex | M | High | ✅ |
| US-10 | Weekly/Monthly Charts | Alex | L | Medium | ❌ v2 |
| US-11 | Fridge Photo Scan | Sam | L | High | ✅ |
| US-12 | Manual Ingredient List | Sam | S | High | ✅ |
| US-13 | AI Recipe Suggestions | Sam | XL | High | ✅ |
| US-14 | Recipe Details View | Sam | M | High | ✅ |
| US-15 | Recipe Filters | Sam | M | Medium | ✅ |
| US-16 | Meal History & Edit | Alex | M | Medium | ✅ |
| US-17 | Offline & Cloud Sync | Both | L | Medium | ✅ |
| US-18 | Daily Meal Plan | Sam | XL | High | ❌ v2 |
| US-19 | Weekly Meal Plan | Sam | XL | Medium | ❌ v2 |
| US-20 | Save Favorite Plans | Sam | S | Low | ❌ v2 |

**Total Stories**: 20  
**MVP Stories**: 16  
**v2 Stories**: 4  
