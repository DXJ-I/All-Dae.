# All Dae., Training, Nutrition & Recovery

A private, evidence-based fitness and health app for iPhone, built with SwiftUI (iOS 17+). Version 3.0 turns All Dae. from a food diary with an exercise add-on into a complete training app: structured programmes, a persistent set-by-set workout logger with progression and personal records, a daily recovery score, body composition tracking and on-device AI coaching, all without an account, a server or tracking.

## What's in the app

| Tab | Highlights |
|---|---|
| **Today** | Eat / Move / Hydrate rings, macro gauges, recovery score with daily check-in, today's programme session with one-tap start, steps vs goal with 7-day chart, sleep, meals and logged activity. |
| **Food** | Diary by meal with calorie budget and macro split bar, barcode scanning and Open Food Facts search, on-device photo estimates, **recipes & saved meals** (new UI), favourites, recents, copy previous day, water with custom amounts. |
| **Train** | Quick start or **Smart workout** (built from under-trained muscles, your equipment and today's recovery), **six programmes** (Foundations, Upper/Lower, PPL, Strength Base, Bodyweight, Mobility), user routines, weekly sets-per-muscle chart, recent PRs, a 128-exercise library with physically based 3D demos (planted feet, balanced centre of mass, real bar paths), history and the coach. |
| **Progress** | Overview (week-over-week review, streaks, achievements), Nutrition (averages, calories/protein charts, macro donut, adaptive maintenance), Training (workouts/week, weekly volume, muscle volume, acute:chronic load, e1RM sparklines), Body (weight trend with goal line, measurements, sleep & heart vs baseline). |
| **You** | Plan & targets, body & measurements, recovery, achievements, optional private cycle tracking, settings. |

### The workout logger
- **Persists across app launches**, the active workout is part of the diary, so closing the app never loses a session. A banner on Today and Train resumes it.
- **Previous performance on every set** (tap to copy), **double-progression pre-fill** (adds the smallest sensible jump once every set hits the top of the rep range at RPE ≤ 9).
- Set types (warm-up, working, drop, failure), per-set RPE, duration-based sets for planks and mobility, reps-only for bodyweight.
- **Rest timer** based on wall-clock time (accurate in the background) with a local notification, ±15 s and skip.
- **Live PR detection**, the tick turns into a trophy the moment a set beats your best estimated 1RM.
- Replace an exercise in place, per-exercise notes, rest per exercise, **plate calculator** (kg and lb).
- Finish with session RPE and notes, then a summary with records, muscles trained, weekly streak and "Save as routine".
- Loads display in kg or lb (from your profile); everything is stored in kilograms.

### Recovery score
A transparent 0–100 wellness indicator that uses only the signals you have: today's 20-second check-in (35 %), sleep (25 %), HRV vs your 30-day baseline (20 %), training load vs your 4-week average (15 %) and resting heart rate vs baseline (10 %). Missing signals are skipped and the rest re-weighted; the breakdown is always shown. Low recovery automatically trims volume and effort in Smart workouts.

### Siri & Shortcuts
"Log water in All Dae.", "How many calories do I have left in All Dae.", "Start a workout in All Dae." (App Intents in the main target, no extension needed).

## Project layout
```
AllDae/
  App/            Entry point, 5-tab shell + AppRouter, TrackerStore (state, persistence, backup validation)
  Core/           Foundation-only domain logic
                  Models · TrainingModels (sets, sessions, active workout, routines, programmes,
                  measurements, check-ins) · ExerciseLibrary (128 movements, stable IDs) ·
                  StrengthAnalytics (records, history, progression, weekly volume, plate calculator) ·
                  WellnessAnalytics (recovery, body composition, achievements) · TrendCalculator ·
                  NutritionCalculator · AppData/FileStore (schema v3, daily snapshots) · CSVExporter
  Services/       HealthKit (baselines, step history), Open Food Facts, Vision food analyser,
                  on-device coach & plan assistant (Foundation Models), reminders & rest timer, App Intents
  Features/       Today · Food · Train · Recovery · Body · Progress · Plan · Profile · Cycle · Settings · Onboarding · Exercise
  DesignSystem/   Theme tokens (adaptive light/dark), buttons, rings, bars, chips, cards, inputs
  Resources/      Assets (icon unchanged), PrivacyInfo.xcprivacy, entitlements
AllDaeTests/      XCTest unit tests (AllDaeTests, Wave3Tests)
AllDaeWatchWidgets/ Watch complications (WidgetKit)
AllDaeUITests/    XCUITest smoke tests, launched with -ui-testing
tools/            build_food_database.py (USDA FoodData Central to USDAFoods.json)
docs/             UPGRADE_3.0.md (audit + changes), history/ (earlier notes)
```

## Build & test
1. Open `AllDae.xcodeproj` in Xcode 16 or later. Xcode 26 enables the on-device Apple Intelligence coach; Xcode 27 (Swift 6.4, iOS 27 SDK) also compiles the Apple Private Cloud option. Older SDKs compile the built-in coach only.
2. Set your team. Bundle identifiers come from one project-level build setting, `APP_BUNDLE_ID` (currently `com.bakatel.alldae`): the widgets, Watch app, tests, App Group (`group.<id>`), iCloud container (`iCloud.<id>`) and the Watch's companion identifier all follow it. Change it under Project > Build Settings, not per target.
3. Widgets and the Live Activity live in the **AllDaeWidgetsExtension** target (iOS 18+). Both targets use the App Group `group.$(APP_BUNDLE_ID)`; with automatic signing Xcode registers it for your team.
4. The **AllDaeWatch** target (watchOS 10) is embedded in the iPhone app. The iPhone target also uses iCloud (CloudKit container `iCloud.$(APP_BUNDLE_ID)`) for optional sync and Push Notifications for sync updates.
5. Signing & Capabilities: **HealthKit** is already in the entitlements file. For Apple Private Cloud, request the Private Cloud Compute entitlement at developer.apple.com/private-cloud-compute (Small Business Program, under 2 million first-time downloads), then add the capability once Apple assigns it. Without it the app silently uses the on-device model.
6. Run the tests with **⌘U** (`AllDae` scheme → `AllDaeTests`). Add **AllDaeUITests** to the scheme's Test action to run the UI tests too.
7. HealthKit, barcode scanning and the camera need a physical iPhone.

## Bundled food data (USDA and UK CoFID)
1. Download **SR Legacy** and **Foundation Foods** (CSV) from https://fdc.nal.usda.gov/download-datasets. Both are public domain.
2. Optional, for British foods: download the **CoFID** Excel dataset (not the "old foods" file) from https://www.gov.uk/government/publications/composition-of-foods-integrated-dataset-cofid. It is Crown copyright under the Open Government Licence v3.0; the Settings > About sources list should credit it when included.
3. Run `python3 tools/build_food_database.py` from the project folder. It finds the downloads in ~/Downloads (zips, unzipped folders or the .xlsx). You can also pass names or paths, or add `--download` to try fetching SR Legacy.
4. It writes `AllDae/Resources/Foods/USDAFoods.json` (with household portions such as "1 medium" when USDA lists them) and, with CoFID, `UKFoods.json`. Rebuild the app. An empty or missing file turns that source off.

Search understands British words (courgette, aubergine, mince, porridge, crisps and others), ranks plain foods first and shows short names, with the full source name underneath.

## 3D athlete asset
Drop a licensed, rigged `Athlete.usdz` into `AllDae/Features/Exercise/Exercise3DAssets/` with animation clips named by exercise ID (for example `EX-001`) or movement pattern (`squat`). iOS 18 devices then use RealityKit; everything else keeps the procedural athlete.

## Translations
English (source), Polish (`pl`) and Brazilian Portuguese (`pt-BR`). Catalogs: `AllDae/Resources/Localizable.xcstrings` (app, widgets and Watch), `Exercises.xcstrings` (exercise library, read with `NSLocalizedString(_, tableName: "Exercises")`) and `InfoPlist.xcstrings` (permission prompts; the Watch has its own). After building, Xcode adds any newly found strings as "New"; translate them in the catalog editor. To test a language, edit the scheme > Run > Options > App Language.

## 4.0 setup notes
- **Location**: outdoor tracking uses When In Use location with background updates (the `location` background mode is in Info.plist).
- **CloudKit**: photo sync adds a `ProgressPhoto` record type in a `ProgressPhotos` zone. It is created in the Development environment on first sync; deploy the schema to Production in the CloudKit console before release.
- **Health**: the app now asks once more for workout routes and walking, running and cycling distance.

## 4.1 setup notes
- **Watch complications** are a new target, **AllDaeWatchWidgetsExtension**, embedded in the Watch app. It and the Watch app now use the App Group `group.$(APP_BUNDLE_ID)`; automatic signing registers it.
- **Challenges** need `CKSharingSupported` (in Info.plist) and add `Challenge` and `ChallengeEntry` record types. Deploy the CloudKit schema to Production before release, as for photos.
- **Snapshot tests**: run the tests once on one simulator (iPhone 16 suggested) to record references into `AllDaeTests/__Snapshots__`, then commit them. Set `SNAPSHOT_RECORD=1` in the scheme's test environment to re-record after an intentional design change.
- **Swift 6**: every target builds with complete concurrency checking, so remaining issues show as warnings. When the warning list is empty, set Swift Language Version to 6 under Build Settings for each target.

## 5.0 setup notes
- **Re-run the food script** to add household portions to `USDAFoods.json` (and CoFID if you downloaded it). Without it, generic foods still work, logged by weight.
- **Swift 6**: all targets stay in Swift 5 language mode with complete concurrency checking (5.0.1). Swift 6 mode adds runtime checks that stop the app when Apple frameworks call back on a background queue; switch only after testing every flow on a device.
- **Apple Private Cloud** needs Apple's managed Private Cloud Compute entitlement (request it at developer.apple.com/private-cloud-compute). Without it, creating the model stops the app, so the option stays disabled until you add the capability and set `AllDaePrivateCloudCompute` to YES in `AllDae/Resources/Info.plist`.
- **Household sharing** adds `HouseholdInfo`, `HouseholdMeal` and `HouseholdItem` record types in `Household-` zones. Deploy the CloudKit schema to Production before release, as for challenges. Invitations reuse the challenge scene delegate, which routes by zone name.
- **Apple Watch strength logging** sends routines in the Watch's application context and returns workouts with `transferUserInfo`, so a workout finished out of range arrives later.
- **Voice cues** use `AVSpeechSynthesizer` with the `.voicePrompt` audio mode; no extra permission or background mode is needed. Cues play while the app is open.
- **Snapshot tests**: two new references (`GenericFoodRows`, `PortionPicker`) are recorded on the first run.

## Releasing
See `docs/release/RELEASE_CHECKLIST.md`. Screenshots: `tools/screenshots.sh "iPhone 17 Pro Max"`. Metadata for App Store Connect is in `fastlane/metadata`. Translation review: `python3 tools/translation_review.py export`, send the CSVs, then `import`.

## Data & privacy
- The diary is one versioned JSON document in Application Support (`AllDae/diary.json`, schema v3) with atomic writes and iOS data protection. Every field decodes leniently, so 1.x and 2.x diaries open unchanged; unreadable files are quarantined, never overwritten. A daily snapshot of the last three days is kept locally as a safety net.
- No account, server, analytics, ads or tracking.
- **Water reminders** through the day (custom window and interval) show how much is left, pause once the goal is met, and offer "Log a glass" right on the notification. Food search sends only the search text or barcode to Open Food Facts.
- The coach and plan suggestions use **Apple Intelligence on this iPhone** by default. Users can choose **Apple Private Cloud** (Foundation Models on Private Cloud Compute, iOS 27) in Settings > Intelligence or from the coach. It has a 32K context and reasoning, falls back to on-device when offline or over the daily limit, and only the request is processed (Apple keeps nothing). Pain and injury questions always get fixed safety guidance.
- **App lock** with Face ID, Touch ID or Optic ID (passcode fallback) and a configurable delay; the diary is hidden in the app switcher; exports, backups, restores and erasing ask for authentication; exported files live in a private folder wiped every launch; water can only be logged from a notification once the iPhone is unlocked.
- Cycle data is never written to HealthKit, sent to the coach or used for targets.
- Progress photos stay on this iPhone (complete data protection, excluded from backups) unless you turn on iCloud photo sync. Habits, fasting, meal plans and outdoor routes are stored locally in `AllDae/features.json` and `AllDae/Routes/`.
- Crash and performance reports (MetricKit) stay on the iPhone and are shared only from Settings > About.
- CSV export (food, workouts, sessions, water, weight, measurements, check-ins), complete JSON backup/restore with validation, and delete-all.

## Methods
Mifflin–St Jeor; PAL 1.2–1.9; 7,700 kcal/kg with a 1 %/week cap and calorie floors; protein 1.2–1.8 g/kg of reference weight; SACN fibre; EFSA fluids; Compendium MET values and the ACSM equation; UK CMO 150-minute guideline; Brzycki 1RM (≤ 12 reps); hard-set volume with secondaries at 0.5; acute:chronic workload ratio; U.S. Navy circumference body-fat method; waist-to-height < 0.5. All outputs are general wellness estimates, not medical advice.

## Status
This environment has no Swift toolchain, so 3.0 was verified with a structural Swift checker (balanced delimiters, string/interpolation termination), a symbol cross-reference check, a `project.pbxproj` integrity check and Python replicas of the test maths, see `docs/UPGRADE_3.0.md`. Build in Xcode and run ⌘U first; any compiler diagnostics should be small and local.
