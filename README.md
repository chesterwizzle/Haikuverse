# Haiku Bot Advanced - Backlog and Known Bugs

This document lists known issues, areas for improvement, planned future features, and provides a record of completed efforts.

-------------------
## Bug

*   **Incompatibility - Just_Audio:** (bug) Flutter package just_audio.dart does not work with Windows-native apps. It only works with Android, iOS, and Chrome. It does not recognize the plug-ins.
*   **Logbook Tab - Comment Persistence:** (bug) Comments submitted but not approved should persist when the user navigates away and then returns to the Logbook Tab.

-------------------
## High Priority (Security, Stability, and Core Features)

### Monetization
*   **Play Console - Configure Subscriptions:** Define subscription products (e.g., monthly pass), including price and SKU ID, in the Google Play Console.
*   **Subscription Screen - Add Purchase Package:** Integrate the official `in_app_purchase` package into the Flutter app's `pubspec.yaml`.
*   **Subscription Screen - Display Products:** Build a new screen that uses the `in_app_purchase` package to fetch and display the configured products from the Play Store.
*   **Subscription Screen - Initiate Purchase Flow:** When a user taps a product, use the package to launch the native Google Play purchase pop-up and listen for the result.
*   **Backend - Create Validation Function:** Develop a Cloud Function that receives a `purchaseToken` from the app after a successful purchase.
*   **Backend - Verify with Google API:** In the function, use the Google Play Developer API to securely validate the `purchaseToken` to confirm it's legitimate.
*   **Backend - Grant Entitlement:** If the token is valid, have the Cloud Function update the user's document in Firestore with an entitlement flag (e.g., `subscription_active: true`).
*   **Enforce Subscription - App Initialization:** Create a check within `main.dart` that listens to the entitlement flag from Firestore. If the check fails, navigate the user to the subscription screen.
*   **Enforce Subscription - User Sign-in:** Create a check within `auth_screen.dart` that reads the entitlement flag from Firestore. If the check fails, navigate to subscription screen after EULA acceptance screen.

-------------------
## Medium Priority (Functionality and Enhancements)

### Play Store Launch
*   **Play Store - App Release:** Generate a signed Android App Bundle (.aab) for production upload.
*   **Play Store - Listing Setup:** Create all required store assets (icon, graphics, screenshots) and write listing text.
*   **Play Store - Closed Testing:** Set up a closed testing track in the Play Console for the initial group of testers.

-------------------
## Low Priority (Refinements and Future Features)

*   **Rate Limiting - Cloud Functions:** Implement rate limiting within the Cloud Function to prevent excessive API calls from individual users or in aggregate.

-------------------
## Bright Idea Parking Lot (Deferred)

*   **Haikuverse - Inter-Constellation Warping:** Implement animated transitions to adjacent constellations when a semantic link is activated.
*   **Haikuverse - Recenter Constellation:** Add a button to reset constellation pan and zoom to default.
*   **Settings Tab - Refactor for Service Layer:** Refactor settings_tab.dart  to use the service layer rather than direct calls to the Cloud Run functions for Gemini API.
*   **Customization Tab - Community Recognition:** Colorful badges for community contributions and feedback. AI generated with simple animation effects.
*   **Customization Tab - Travel Log:** Provide a postcard feature crafting for downloadable digital mementos about a user's journey through constellations.
*   **Customization Tab - User Nameplate Customization:** Stylized nameplates for display on owned constellations. New themes released every month!
*   **Customization Tab - Voice Customization:** Text-to-speech voice packs. Use marketplace previews for demonstration. New themes released every month! 
*   **Cloud Functions - Cold Start Performance:** Refactor index.js to use lazy initialization for heavy clients (e.g., VertexAI, Storage, TTS) to improve cold start times and deployment reliability.
*   **Automated Testing - Service Layer:** Implement unit tests for `GeminiService`, `VertexAIService`, `FirestoreService`, and `StorageService`, verifying their internal logic.
*   **Automated Testing - main.dart:** Add widget testing for _MyAppState logic (specifically _toggleFavorite, _removeFavorite, _associateImageUrlToHaiku) to verify interactions.
*   **Automated Testing - Cloud Functions:** Develop unit/integration tests for new/modified backend Cloud Functions (e.g., theme recommendations, travel advisor).
*   **Automated Testing - Integration Tests:** Implement end-to-end integration tests to validate critical user flows across multiple components. Focus on simulating realistic user journeys.
*   **Social Metrics:** Metrics, metrics, metrics on social interactions help build community-wide reports, activity highlights, and records of individual achievements.
*   **Gen AI Music Background:** Use Generative AI to overlay text-to-speech on top of generated music or ambient sounds, creating a richer, multi-sensory user experience.
*   **Custom Claims/Admin Users:**  Implement different user roles (e.g., regular users and administrators)using Firebase custom claims to manage user permissions and access levels.
*   **Comprehensive User Guide:** Develop detailed documentation and a user guide for end-users, explaining the app's features, setup, and usage instructions for a wider audience.
*   **More Granular Error Handling in UI:** For certain critical errors, switch from Snackbar to more prominent UI elements like error dialogs or dedicated error screens to guide the user.
*   **User-Configurable Hyperparameter Tuning:** Expose user settings for Gemini model parameters (temperature, top-p, etc.), enabling personalized control.
*   **Multilingual Support:** Extend language support beyond English to enable haiku generation and UI localization in multiple languages, broadening the app's reach and accessibility.
*   **Internationalization:** Use a localization library like `flutter_localizations`. This will enable the app to be easily translated into multiple languages that are localized.
*   **Technical Documentation:**  Create comprehensive technical documentation targeted at developers, detailing the app's architecture, code structure, API integrations, and testing strategies.
*   **Reduce GCP Attack Surface:** Implement IAM restrictions at the GCP Cloud Run level to further limit access and reduce the attack surface, particularly for production deployments.
*   **Premium Monetization Strategy:** Incentivize monetization strategy to entice a subscription model (e.g., free Haikuverse view access, but content creation will require a subscription!).
*   **Repository Pattern:** A stricter implementation of the Repository Pattern would have FirestoreService only handle data access, while a separate CloudFunctionService could handle the HTTP triggers.
*   **Move Favorites Handling to Service Layer:** Moved the favorites handling methods in `main.dart` to `firestore_serivce.dart`.
*   **Refactor for Riverpod:** Riverpod offers compile-time safety, reduced boilerplate, and enhanced performance characteristics, which can be beneficial for very large and complex applications.
*   **Performance Profiling and Optimization:** Regularly use Flutter's performance profiling tools to identify potential bottlenecks (widget rebuilds, network latency, data processing).
*   **Dartdoc Comments:** Create comprehensive in-code documentation that explains the purpose, functionality, and usage of classes, methods, and functions directly within the code itself.
*   **Theming Options / UI Tools:** Explore more advanced theming options and consider providing UI tools for users to customize colors.
*   **Error Handling - Utility Function:** Develop a reusable utility function or class to standardize the display of SnackBar error messages or more complex error UI elements.
*   **Multi-Environment Firebase Backend:** Create separate Firebase projects (e.g., dev, staging, prod). Configure core services (Firestore, Auth, Functions) in each.
*   **Configure Flutter App Flavors:** Implement Flutter Flavors in the user and admin app codebases to easily switch between connecting to the Dev, Staging, and Prod Firebase environments.
*   **Build Admin App:** Create a separate Admin app connected to the common Firebase project and GCP back-end for managing community content and data maintenance functions.
*   **Admin Notification Feature:** Create an SMS text reporting capability to monitor system health and moderate community features when needed.
*   **Vector Database Maintenance:** Develop and test the batch Firebase Function (or Cloud Run job) for re-indexing existing Firestore data into Vertex AI Vector Search using the Admin app.
*   **Performance Tuning - Image Generation/Loading:** Use caching, image compression, and progressive image loading to further improve the user experience.
*   **Manage Tab - Constellation Name:** Add capability for user to create their own constellation name, run it through a Cloud Run sanitizer function, and then publish on success.
*   **FavoritesScreen - Group By Constellation:** Group favorite haikus by constellation. Implement roll-up headers with haiku counts for each constellation (e.g., drill-down deck UI).
*   **Export Capability:** Add support to download printable haiku greeting cards. 
*   **Parse Haiku Lines:** Parse the favorite haiku into three separate lines so the image slideshow can transition in synch with the text-to-speech reading of each line.
*   **User Dictation:** Add the capability for the user to dictate an entry into the prompt window. Add an enable microphone icon at the bottom right of the text window.
*   **Additional Sign-In Methods:** Add support for alternative Firebase Authentication providers, such as Facebook Login, and Sign-in with Apple.

-------------------
## Reach Goal - Advanced Semantic Discovery with GNN-Powered Fable Index:

*   **Periodic GNN Processing:** Construct graph from discoverable constellations (fable embeddings = node features, semanticNeighborIds = edges) for GNN to refine fable index.
*   **Graph-Aware Embeddings:** Utilize a GNN architecture (e.g., GCN, GraphSAGE via PyTorch Geometric/DGL) in a self-supervised manner to generate new, richer embeddings for each constellation.
*   **Dedicated GNN Vector Search Index (VS_Fables_GNNEmbed):** Populate a new Vertex AI Vector Search index with these GNN-generated embeddings.
*   **Explore Tab - Enhanced Launch Gate:** Modify the "Explore Tab" launch gate to query this VS_Fables_GNNEmbed index when a user enters a prompt for their desired "adventure type."
*   **Preserve Incremental Knowledge Graph:** Continue using the existing VS_Fables_TextEmbed index and the updateSemanticNeighbors function for incrementally building and updating constellation documents.

-------------------
## Completed Tasks

*   **(complete) Haiku Data Class:** Created the `Haiku` data class.
*   **(complete) Shared Preferences Memory Leak:** (bug) Fixed the potential memory leak.
*   **(complete) NicknameProvider:** Implemented the `NicknameProvider` and integrated it correctly.
*   **(complete) HomeScreen - Consolidate _generateContent:** Refactored the `_generateContent` method.
*   **(complete) favorites_screen.dart:** Removed the unused `model` parameter.
*   **(complete) ConstellationsScreen - Index Management**: Added a Provider to correctly hand the index logic.
*   **(complete) ConstellationsScreen - Layout:** Implemented the three-column layout with correct sizing and alignment.
*   **(complete) ConstellationsScreen - "x of y" Text:** Display the current haiku number and total favorites.
*   **(complete) Constellations Screen Arrows:** Implemented arrows to control the cycling of haikus.
*   **(complete) Authentication:** Implement user authentication (using Firebase Authentication) to secure access to the application and user data.
*   **(complete) Services Directory:** Move REST API calls into this directory for modularity and re-use.
*   **(complete) Sign-in/Sign-out:** Need to properly set initial route and follow-on routes when user signs-in, signs-out, and signs-in again.
*   **(complete) Refactor Provider Calls:** Change Provider.of<T>(context, listen: false) to context.read<T>() and Provider.of<T>(context) to context.watch<T>.
*   **(complete) Sign-out Button Width:** The sign-out button on the preferences screen is not wide enough.
*   **(complete) Scrolling Underlays Header:** (bug) The scrolling list of haikus in the favorites screen incorrectly scrolls under the List header.
*   **(complete) Unfavorite Button Broken:** (bug) The unfavorite button on the home screen does not work, and the favorites list creates duplicate entries every time it is pressed.
*   **(complete) Email Provider:** Implement an email Provider that automatically updates the preferences screen with the email of the currently signed-in user rather than a manual entry.
*   **(complete) StarsScreen - Name Change:** Change the name of the Constellations screen to the Stars (Image Pairing) Screen.
*   **(complete) Stars Icon - New:** Implement an "image pairing" icon as a fifth screen in the bottom navigation bar for the Stars screen.
*   **(complete) ConstellationsScreen - New:** Add a fifth screen that has three tabs (i.e., Constellation Selection, Shared Profile, and Community Achievements).
*   **(complete) theme_provider.dart:** Load by name instead of index.
*   **(complete) Seasonal Themes:** Add Autumn Theme and Beach Theme to the Theme Provider.
*   **(complete) Cloud Migration - API:** Migrate sensitive logic (API key handling, haiku generation) to Google Cloud Functions. This is essential for security and scalability.
*   **(complete) Cloud Migration - Clean-up:** Remove .env and dotenv functionality. Remove gemini "model" parameter and clean-up associated logic.
*   **(complete) Refactor Favorite Button:** Modify _toggleFavorite and _removeFavorite in main.dart to use Haiku.id for favorite management and update duplicate checks with haiku.text.
*   **(complete) Input Sanitization - Cloud Function:** Sanitize prompt input within the Cloud Function before sending it to Vertex AI. This is a crucial security measure to prevent prompt injection attacks.
*   **(complete) Deploy Cloud Functions:** Set-up VS Code to deploy Firebase Cloud Functions from VS Code. Maintain CM of the folders directory in GitHub.
*   **(complete) Cloud Functions - Authentication:** Enable user authentication in Cloud Run using Firebase authentication and Cloud Run functions.
*   **(complete) Password Reset:** Implement a "Forgot Password" flow, allowing users to reset their passwords via email. This is a standard Firebase Authentication feature.
*   **(complete) Email Verification:** Require users to verify their email addresses after signing up.
*   **(complete) Shrinkable Textboxes:** Implement a shrinkable textbox for the haiku textboxes on the stars_screen so there is no bottom overflow on a smaller phones.
*   **(complete) SignOutProvider - BuildContext:** Moved BuildContext from the signOut Provider to the Preferences and Verification screens for significantly easier unit testing.
*   **(complete) Redundant firebase_options.dart:** (bug) Fix this right meow!!
*   **(complete) Automated Testing - Unit Tests:** Isolate each unit (like NicknameProvider's logic, Haiku's toJson/fromJson, or a utility function) and check if it behaves exactly as expected.
*   **(complete) Automated Testing - Widget Tests:** Verify the UI renders correctly given a specific state, basic interactions work, and UI interactions correctly trigger state changes or callbacks. 
*   **(complete) Back-end - Image Generation:** Implement image generation logic using Imagen 3 using Firebase Cloud Function(s) and Vertex AI.
*   **(complete) StarsScreen - Image Generation:** Implement the image generation functionality (using a Firebase Cloud Function that interacts with Imagen 3).
*   **(complete) StarsScreen - Loading Indicators:** Add loading indicators to `StarsScreen` (for image generation) where asynchronous loading of Generative data occurs.
*   **(complete) StarsScreen - Save Slots:** Add three generated image save slots for each favorite Haiku in the favorite list, and have the option to overwrite slots.
*   **(complete) StarsScreen - Pop-up Confirmation:** Add a window that pops up with the image to be overwritten and a confirmation button whenever a save slot is selected.
*   **(complete) Shared Preferences - Current Image:** (bug) The most recently saved image for a specific favorite Haiku does not display when navigating back to stars_screen or when restarting the app.
*   **(complete) Shared Preferences - Save-slot Independent:** Regardless of which save slot an image overwrites, the stars_screen must display only the most recent image (saved or unsaved).
*   **(complete) Deleted Haikus Persist Indefinetely:** (bug) The entry for that deleted haiku's ID will remain forever in the CurrentImageProvider's map inside SharedPreferences.
*   **(complete) Firebase Storage - Image Disposal:** When unsaved images become inaccessible upon navigating away from stars_screen, they are incorrectly orphaned in Firebase storage.
*   **(complete) StarsScreen Indexing:** (bug) Adding a new haiku to the favorites list incorrectly resets the current haiku index in Stars Screen to 0 instead of the most recently viewed.
*   **(complete) PreferencesScreen - Input Validation:** Change the validator in `PreferencesScreen` to use an "else if" tree to provide the user with the reason why their input is invalid.
*   **(complete) Automated Testing - current_image_provider:** Add widget testing for curent_image_provider.dart.
*   **(complete) Automated Testing - Overwrite Confirmation Modal:** Add widget testing for overwrite_confirmation_modal.dart with the Cloud Function mock for image generation.
*   **(complete) Automated Testing - Image Slots Screen:** Add widget testing for image_slots_screen.dart with the Cloud Function mock for image generation.
*   **(complete) Automated Testing - FavoritesScreen:** Add widget testing for favorites_screen.dart, verifying UI rendering based on input list and interaction with the onFavoriteRemoved callback.
*   **(complete) Automated Testing - HomeScreen:** Add widget testing for home_screen.dart with dependency injections for haiku generation. The "generate haiku" button states cannot be directly tested.
*   **(complete) Generating Disables Save Button** (bug) Generating image improperly clears the save icon for all other unsaved images. Updated from string to map and added "true" to Navigator.pop(). 
*   **(complete) Local Branch for Troubleshooting:** Used git checkout -b local-troubleshooting to work through a tenacious bug using a branch. Use git add lib/main.dart to move it back to main.
*   **(complete) Fatal Range Error** (bug) Unfavoriting from home_screen always crashes the app if there is an image paired to the current haiku, when there is two or more haikus on favorites list.
*   **(complete) Restarting App Disables Save Button** (bug) Restarting the App improperly clears the save icon for all unsaved images, so they are abandoned in Firebase Storage.
*   **(complete) Saved Image Improperly Shows Saved Icon:** (bug) After an image is saved, it incorrectly shows a save icon. When the generate image button is pressed, it clears the condition.
*   **(complete) Implement Firebase Storage Security Rules:**  Configure Firebase Storage Security Rules to require user authentication for image access, enhancing data privacy and security. ("@.@) --> d(^_^d)
*   **(complete) Automated Testing - Refactoring Updates:** Updated automated testing for changes made to haiku.dart, user_profile.dart, current_image_provider, preferences_screen, and verificaitons_screen.
*   **(complete) Firestore Data Structure:** Create a top-level Firestore collection named users.
*   **(complete) Firestore Service Layer:** Implement a dedicated service class (e.g., services/firestore_service.dart) to encapsulate all interactions with Firestore related to user data and favorites.
*   **(complete) User-Specific Keys - Users:** Modify SharedPreferences keys to be user-specific. Instead of just 'userProfile', use a key like 'userProfile_{uid}' for caching.
*   **(complete) Device-Independent User Profiles:** Utilize Firebase Authentication's user UID as the unique identifier to enable cross-device profile access and prepare for future community features.
*   **(complete) Unique Filenames for Storage:** Generate unique (UUID) filenames for profile images uploaded to Firebase Storage.
*   **(complete) Handling New Users:** Provide a smooth onboarding experience and ensures every authenticated user has a corresponding Firestore document.
*   **(complete) Firestore Security Rules:** Ensure users can access, modify, and delete only their own data, preventing unauthorized access.
*   **(complete) Cloud Function - Profile Image Sanitization:** Sanitize uploaded profile images using Cloud Vision SafeSearch via a Cloud Function trigger.
*   **(complete) User-Specific Cache Keys - Themes:** Update `SharedPreferences` keys for theme (theme_{uid}) to be user-specific.
*   **(complete) Firestore Defaults for New Users":** (bug) Corrected default nickname initialization for new users. 
*   **(complete) Navigate to HomeScreen on Sign-in:** (bug) Implemented auth state listener to reset navigation index on login.
*   **(complete) Firestore Persistence for User Switching:** (bug) Added provider reload calls after auth events to fix state persistence issues between users.
*   **(complete) Automated Testing - Refactored Providers:** Refactor user_profile, current_image_provider, nickname_provider, and theme_provider tests for migration of User profiles to Firestore.
*   **(complete) Automated Testing - Refactored Screens:** Refactor preferences_screen, auth_screen, and verification_screen tests for migration of User profiles to Firestore.
*   **(complete) Refactor Favorites UI Logic:** Extract favorites management UI logic from MyApp and put it into firestore_service for coupling with onSnapshot.
*   **(complete) User-Specific Cache Keys - Favorites:** Update `SharedPreferences` keys for favorites (favorites_{uid}) to be user-specific.
*   **(complete) Firestore Favorites Sync:** Implement storing/retrieving user favorites in Firestore (/users/{uid}/favorites) for multi-device sync.
*   **(complete) Firestore as Single Source of Truth:** Refactor state management (e.g., favorites, profiles) to primarily use Firestore instead of SharedPreferences caching. d(O_O")
*   **(complete) Automated Testing - Refactored Favorites:** Refactor user_profile, favorites_screen, and home_screen automated tests for migration of User favorites data to Firestore. 
*   **(complete) StarsScreen - Haiku Text Alignment:** Vetically align center the haiku text inside the fitted box.
*   **(complete) Nickname in Firestore not Updating:** (bug) The `authorNickname` field in Firestore database is not updating when the profile_screen save button is pressed.
*   **(complete) Add VertexAIService:** Add vertex_ai_service.dart to encapsulate direct Firebase Vertex AI (Imagen) SDK calls in StarsScreen. Enables Dependency Injection for mocking image generation.
*   **(complete) Add StorageService:** Add storage_service.dart to encapsulate direct Firebase Storage SDK calls in StarsScreen. Dependecy Injection avoids complex platform channel mocking during testing.
*   **(complete) Automated Testing - StarsScreen:** Add widget testing for stars_screen.dart under the new service layer architecture to mock image generation, cloud storage, and Firebase authentication.
*   **(complete) Cloud Function - Nickname Sanitization:** Validate/sanitize user nicknames on profile save using a Gemini safety check Cloud Function.
*   **(complete) Nickname in Firestore Favorites not Updating:** (bug) The `authorNickname` field in Firestore Favorites docuements are not updating when the profile_screen save button is pressed.
*   **(complete) FirestoreService - Batch Write Nickname:** New method in firestore_service.dart to batch write user ID and nickname changes throughout the favorites subcollection.
*   **(complete) User Profile - Special Characters:** Change RegExp in the validator for the nickname TextFormField in preferences_screen.dart to include spaces, hyphens, and apostrophes.
*   **(complete) Input Validation - SantizeNickname Function:** Limit special characters user can use for Nickname to prevent prompt injection attacks into the sanitizenickname cloud function.
*   **(complete) Unique User Nickname:** Do not allow a user to duplicate the nickname of another user. Implement a duplicate check on the back-end in the SanitizeNickname cloud function.
*   **(complete) Initial Login - Default Nickname for StarsScreen:** (bug) Upon initial sign-in to app, favorites on StarsScreen all have the default nickname. Now stars_screen reads from the Haiku object.
*   **(complete) Automated Testing - StarsScreen:** (bug) Update stars_screen_test.dart for the refactor of nickname_provider.dart to load nicknames from Firestore rather than `SharedPreferences` cache.
*   **(complete) Set-up Haikuverse Data Structures:** Build the core backend data structures required to support Haikuverse social features.
*   **(complete) Set-up Haikuverse Community Storage:** Build the core backend storage infrastructure required to support Haikuverse social features.
*   **(complete) Set-up Haikuverse Community Database:** Build the core backend datatbase infrastructure required to support Haikuverse social features.
*   **(complete) Incremental Indexing:** Firestore onCreate triggered Firebase Function calls Vertex AI Embedding API, and upserts the resulting vector into the appropriate Vertex AI Vector Search index.
*   **(complete) Vector Search Querying:** Create the Callable Firebase Function that embeds a query, performs a similarity search using Vertex AI Vector Search, and returns relevant constellations.
*   **(complete) Publishing Recommendations:**  Use Vertex AI Vector Search (create index, define dimensionality) to identify constellations related to the semantics of the haiku package to be published.
*   **(complete) ConstellationsScreen - Publish Tab:** Display haiku experience packages for each haiku on the favorites list and the ability to scroll using left and right arrows.
*   **(complete) ConstellationsScreen - Publish Tab:** Create a slideshow that displays each of the three saved pictures in the generated haiku. Add an icon button to play the slideshow.
*   **(complete) ConstellationsScreen - Publish Tab:** Add a button to publish a haiku experience package to the Haikuverse back-end.
*   **(complete) ConstellationsScreen - Publish Tab:** Add an icon button to pop-up a bottom modal screen with constellation picker that lists recommended constellations to publish to.
*   **(complete) ConstellationsScreen - Publish Tab:** Allow the user to select a constellation from an existing subscription or to add a new constellation.
*   **(complete) Left/Right Arrow Indexing:** (bug) The left/right arrows in stars_screen and preview_star are causing run-time crashes. Modified _favoritesSubscription in main.dart to update indexes. 
*   **(complete) User NickName Missing from New Constellaton:** (bug) User nickname is incorrectly set to default "New Poet" when constellation_detail_modal is accessed for a new constellation.
*   **(complete) Empty Constellations not Deleted from Firestore:** (bug) Constellations decremented to star count zero are incorrectly remaining in the Firestore collection for Constellations.
*   **(complete) ConstellationsScreen - Publish Tab:** Implement the Mine slicer logic so that the list is filtered by constellations the user has an existing realationship with.
*   **(complete) Automated Testing - Publish Tab:** Add widget and unit testing for publish_tab.dart and subordinate models, screens, and widgets.
*   **(complete) Automated Testing - Image Picker:** Add widget testing for `profile_image_picker.dart. Simulated interactions with a mocked ImagePickerPlatform.
*   **(complete) FavoritesScreen - Voice Pairing:** Change top app bar title to Voice Pairing to reflect enhanced funcitonality of this page.
*   **(complete) Cloud Run Function - Text-to-Voice:** Utilize Google Cloud Text-to-Speech API to create an audio file for any haiku in the favorites list. Manage Firebase storage and Firestore database.
*   **(complete) Firebase Storage - Audio Files:** Store audio files in Firebase storage and pass URLs to Firestore database for on-demand use. Ensure haiku creator has CRUD rights to manage storage.
*   **(complete) Firestore Database - Audio Metadata:** Manage audio file metadata and allow public read when a star is published. Ensure haiku creator has CRUD rights to manage storage.
*   **(complete) FavoritesScreen - Text-to-Voice Button:** Add an icon button to each tile to access the read control modal screen and modify the paired voice for each haiku in the favorites list.
*   **(complete) FavoritesScreen - Text-to-Voice Modal:** Add a modal screen to select voice style and cadence to read the haiku. Add a play button and done button to return to `FavoritesScreen`. 
*   **(complete) FavoritesScreen - Audio Playback:** Add a big round icon button to the text-to-voice modal screen generate a live preview and save the audio file (or a playback if file already exists).
*   **(complete) Firebase Storage - Audio File Clean-up:** Add capability delete an audio file from storage when a haiku is removed from the favorites list by any method.
*   **(complete) Automated Testing - TTS Integration:** Add widget testing for favorites_screen.dart and audio_toolkit_modal.dart for the integration of Cloud Text-to-Speech. 
*   **(complete) Manage Tab - Constellation Scrolling:** Implement a scrollable card list of constellations the user owns.
*   **(complete) Manage Tab - Constellation Fables:** Add the cability for a constellation owner to use Gemini to generate a short story for each constellation that dynamically reflects star composition.
*   **(complete) Manage Tab - Constellation Images:** Add the cability for a constellation owner to use Imagen 3 to generate an image for each constellation that reflects star composition.
*   **(complete) Fable Index:** Create an index in Vertex AI Vector Search for determining semantic similarity of a fable to other fables. 
*   **(complete) Manage Tab - Knowledge-Graph Building:** Create a knowledge graph for relationships between discoverable constellations using semantics (implemented in Firestore /constellations).
*   **(complete) Cloud Run Functions- Memory Configuration:** Update generateconstellationrecommendations, publishstar, deletepublishedstar, and suggestconstellationnames to V2 functions.
*   **(complete) Automated Testing - Manage Tab:** Add widget and unit testing for manage_tab.dart and subordinate models, screens, and widgets.
*   **(complete) Explore Tab - UI Structure:** Implemented two main vertical sections with 50/50 flex scaling (Top: Display, Middle: Toolkits).
*   **(complete) Explore Tab - Launch Gate Display:** Shows selected constellation's fable image (fitted, left) and fable text (scrollable, right).
*   **(complete) Explore Tab - Static Graph Preview:** Renders `MiniConstellationGraphView` in bottom-half of top section, updating with selection.
*   **(complete) Explore Tab - Mode Toggle:** Implemented centered `ToggleButtons` for "Discover", "Known", "News", and "Launch" toolkits.
*   **(complete) Explore Tab - "Discover" Toolkit:** Added `TextField` (min 3 lines, clearable), "Find Adventures" button, and `Slider` for (mock) results.
*   **(complete) Explore Tab - "Known" Toolkit:** Displays selected constellation's name, owner avatar/nickname, star count; includes `Slider` for (mock) results.
*   **(complete) Explore Tab - "News" Toolkit:** Basic stub UI implemented for future news content.
*   **(complete) Explore Tab - "Launch" Toolkit:** Displays contextual comment area and "Launch into Haikuverse!" button.
*   **(complete) Explore Tab - State Persistence:** "Discover" prompt, results, slider & "Known" list, slider persist across mode toggles.
*   **(complete) Explore Tab - Display Sync Logic:** Graph/fable area correctly syncs with active slider when toggling modes.
*   **(complete) Explore Tab - Launch Button Logic:** Button enables with selection and is stubbed (logs ID, shows snackbar).
*   **(complete) Explore Tab - Discover Toolkit:** Added Cloud Function getthemebasedconstellationrecommendations() to embed user prompt, query `VS_Fables_TextEmbed`, return top N discoverable constellations.
*   **(complete) Explore Tab - Discover Toolkit:** Added FirestoreService method getThemeBasedConstellationRecommendations() to call theme query Cloud Function.
*   **(complete) Explore Tab - Discover Toolkit:** Replace mock constellations for Discover toolkit with real data, loading, and errors.
*   **(complete) Explore Tab - Known Toolkit:** Added FirestoreService method for user's discoverable constellations.
*   **(complete) Explore Tab - Known Toolkit:** Replaced mock constellations for Known toolkit with real data, loading, and errors.
*   **(complete) Explore Tab - Launch Toolkit:** Add Cloud Function for Gemini to generate travel advice based on constellation data & user prompt (if any).
*   **(complete) Explore Tab - Launch Toolkit:** Implement Firestore caching for travel advisor comments based on input hashes.
*   **(complete) Haikuverse - Navigation Screen:** Create dedicated screen, triggered by Explore Tab's "Launch" button, for 2D Haikuverse exploration.
*   **(complete) Haikuverse - Graph Visualization Core:** Implement a heuristic  procedural generation state machine for dynamic, interactive constellation layout.
*   **(complete) Haikuverse - Intra-Constellation Navigation ("Zorching"):** Enable smooth animated panning, zooming within the current constellation's detailed view.
*   **(complete) Haikuverse - Star Focusing:** Implement animation controllers to smoothly centers and zooms in on a star when tapped, and reverse the pan/zoom when exiting the pop-up screen.
*   **(complete) Haikuverse - CustomPaint Styling:** Implement rich vector art style for constellations, stars (if shown), and links using `CustomPaint`.
*   **(complete) Haikuverse - Pulse Star on Focus-out:**Add single pulse effect when focus-out (rewind) animation completes and the star is back in its original constellation position.
*   **(complete) Haikuverse - Star Detail Display:** On tapping a star node, display the haiku pop-up window.
*   **(complete) Haikuverse - Data Loading:** Fetch selected constellation and its stars from Firestore to populate the 2D vector-art constellation rendered in the Haikuverse Navigation screen.
*   **(complete) Star Popup - Experience Tab:** Create the unbiased, art-focused initial view of the selected star with a synchronized audio file and image slideshow playback (with a pause feature). 
*   **(complete) Increased Allocations:** Increase maximum number of haikus on the favorites list to 99.
*   **(complete) Constellation Lifecycle & Ownership:** Ensured constellations are deleted when empty and ownership transfers correctly if the original owner unpublishes all their stars while others remain.
*   **(complete) Orphans in Published Stars Collection:** (bug) Unfavoriting a star needs to remove it from `published_stars` collection, decrement the star counter, and clear local cache.
*   **(complete) Explorer Tab - Manager ID:** Updated the Known Toolkit in `explorery_tab.dart` to display the manager ID rather then the creator ID.    
*   **(complete) Publish Tab - Manager ID:** Updated `constellation_detail_modal.dart` and `constellation_info_display.dart` to display the manager ID rather then the creator ID.
*   **(complete) Manage Tab - Constellation Connections:** Restrict connections for constellations to neighbors with high semantic similarity (random between 1 and 5 neighbors on save). \m/ (>.<) \m/
*   **(complete) Publish Tab - Data Validation:** Implemented client-side validation of knowledge graph edges each time a constellation node is selected for view in the mini_constellation_graph_view.
*   **(complete) Manage Tab - New Neighbors Notification:** Modified the "successful save" snackbar message in constellations_customization screen to display the constellations new, randomized neighbors.
*   **(complete) Star Popup - Like/Unlike Star:** Allow users to like/unlike a star without knowing who the creator is via the like button on the Experience Tab of the Star Popup window.
*   **(complete) Star Popup - Star Udpates:** Used cache-busting and audio_toolkit_provider.dart to synchronize star_detail_popup with user changes to published stars using `FavoritesScreen` and `StarsScreen.`
*   **(complete) Automated Testing - Audio Player Provider:** Add unit testing for audio_player_provider.dart.
*   **(complete) Update to Firebase SDK:** Refactored vertex_ai_service.dart, stars_screen.dart, constellation_customization_screen.dart to migrate from firebase_vertxai to firebase_ai.
*   **(complete) Update to Firebase SDK:** Refactored stars_screen_test.dart and constellation_customization_screen_test.dart to migrate from firebase_vertxai to firebase_ai.
*   **(complete) Manage Tab - Conditional Plural Case:** (bug) For a count of 1 star, use the singular case "star" and for more than one star, use the plural case "stars" in constellation tiles.
*   **(complete) Explore Tab - Dynamic Graph Preview:** Implement animation controller(s) for constellations in `MiniConstellationGraphView` for users to visually navigate to adjacent constellations.
*   **(complete) Audio Modal - Provider State Detection:** (bug) audio_toolkit_modal and star_detail_popup have play buttons that are not properly tracking audio_provider state. 
*   **(complete) Automated Testing - Audio Modal:** Updated automated testing for the refactoring of audio_toolkit_modal.dart and audio_player_provider.dart.
*   **(complete) HomeScreen - Refactor for Service Layer:** Refactor home_screen.dart  to use the service layer rather than direct calls to the Cloud Run functions for Gemini API.
*   **(complete) HomeScreen - Star Naming Toolkit:** Add a Cloud Run function for Gemini to suggest star names based on the haiku text. 
*   **(complete) HomeScreen - Button Theme Color:** Generate button on home_screen needs to match the theme color of the buttons on all the other screens.
*   **(complete) HomeScreen - Fixed Textbox Size:** Change the constrained box for the generated haiku into a fitted box so that there is always only three lines of text. Center align the text.   
*   **(complete) Haikuverse - Star Labels:** Refactor haikuverse_navigation_screen.dart so that the constellation nodes display star names rather than the first line of Haiku text. 
*   **(complete) Star Popup - Star Names:** Refactor star_detail_popup.dart to display the star names on the haiku experience tab.
*   **(complete) Automated Testing - HomeScreen Update:** Update widget and unit testing for `home_screen.dart` after refactoring for Star Naming and Notifications toolkits.
*   **(complete) Automated Testing - Haikuverse Navigation:** Add tests for the new `haikuverse_navigation_screen.dart` and `constellation_view_painter.dart`.
*   **(complete) Star Popup - Unpausable if Image Only:** (bug) In the absence of an audio file, pushing the "Play Full Experience" button incorrectly shows "Resume Experience" instead of "Pause Experience". 
*   **(complete) Star Popup - Logbook Tab:** Display social context (creator and metadata), existing comments, and allow new comment submission.
*   **(complete) Star Popup - Viewer Comments:** Allow users to enter comment(s) about the star in the Logbook tab. After a harm-check, push the publish request to `HomeSecreen` notifications area. 
*   **(complete) Notifications Screen:** Add a scrolling list of user-specific messages such as moderation of user comments the Logbooks for owned stars. Accessible from the alarm bell icon in the top app bar.
*   **(complete) Controlled Social Posts:** Implement social posts for stars that are sanitized by Gemini. Allow the star owner to aprove the post before it is made public (or report abuse).
*   **(complete) Community Moderation Tool:** Implement private moderation_queue in Firestore for administrators to adjudicate user reports of abuse and review system reports of failed harm checks. 
*   **(complete) New Firestore Collection - Public Profiles:** Moved from a single-user data model to a secure, multi-user architecture with a clear separation between private and public data.
*   **(complete) Poet Tab - Follow/Unfollow Button:** Integrated a dynamic UI button to follow or unfollow poets.
*   **(complete) Poet Tab - Conditional UI:** The follow button is now correctly hidden when a user views their own profile.
*   **(complete) Cloud Functions - Follow/Unfollow Logic:** Deployed HTTP functions for secure follow/unfollow actions.
*   **(complete) Firestore - Atomic Follows:** Used batch writes for atomic updates to follower/following lists and public follower counts.
*   **(complete) Cloud Functions - Follow Notifications:** The backend now creates a notification for the poet when they gain a new follower.
*   **(complete) Poet Tab - Display Achievements:** Displayed earned achievements dynamically on the Poet tab from public profile data.
*   **(complete) Cloud Functions - Idempotent Achievements:** Refactored backend logic to award achievements idempotently.
*   **(complete) Cloud Functions - Achievement Notifications:** Backend now creates a notification when an achievement is awarded.
*   **(complete) Notifications Screen - Refactor:** Rebuilt the screen to handle multiple, distinct notification types.
*   **(complete) Notifications Screen - Dismiss Action:** Implemented the ability for users to dismiss notifications.
*   **(complete) Cloud Functions - Data Sync:** Synced public likeCount and constellationId back to the private user favorites document. d(╬ò_ó)d
*   **(complete) Achievements - Constellations:** Add idempotent achievements for first, fifth and tenth public constellations.
*   **(complete) Poet Tab - Scrollable Acheivements:** Add a scrollable list to the Poet achievements area.
*   **(complete) Launch Toolkit - Ghost Constellations:** (bug) When unpublishing the last star from a constellation, that constellation incorrectly persists in the explore tab launch toolkit.
*   **(complete) Poet Tab - Zero Followers:** Hide followers count when the user has 0 followers.
*   **(complete) Automated Testing - Public Data Models:** Add unit testing for `comment.dart`, `notification.dart`, and `public_profile.dart`.
*   **(complete) Image Slots Screen - Live Previews:** Refactored `image_slots_screen.dart` to show a preview of the image when licking on an overwrite slot. Also updated automated testing for the refactor.
*   **(complete) Automated Testing - Notifications Screen:** Add widget and unit testing for `notifications_screen.dart`.
*   **(complete) Automated Testing - Star Popup Screen:** Add widget and unit testing for `star_detail_popoup.dart` and `star_slideshow.dart`.
*   **(complete) Zeitgeist Engine:** Develop the backend Cloud Function to analyze activity, extract the top 5 themes, and rank the top 5 candidate constellations for each.
*   **(complete) Zeitgeist Map Widget:** Build the new CustomPaint widget to display the floating theme words, handling the "pop" animation and "Focus Chevron" indicator on tap.
*   **(complete) Zeitgeist Data Integration:** Refactor the Explore Tab and FirestoreService to fetch and stream the live Zeitgeist data from the /app_meta/zeitgeist document.
*   **(complete) Zeitgeist Connection Logic:** Implement the "Waterfall Check" and connect the ZeitgeistMap tap events to the MiniConstellationGraphView to display the result.
*   **(complete) Zeitgeist State Pesistence:** (bug) The Zeitgeist map does not maintain persistence when navigating to other toolkits, tabs, and screens and then back. Udpated persistence of all toolkits.
*   **(complete) Explore Tab - Latency:** (bug) Occasional latency when switching between Discover toolkit and Known toolkit. Fixed when `explore_tab.dart` was refactored for persistence of all toolkits.
*   **(complete) Haikuverse - Procedural Override:** For three stars or less, bypass the heuristic algorithm to make one connection (only) from each of the stars on the first ring to the center star.
*   **(complete) Known Toolkit - Get Followed Constellations:** Implement a Cloud Run function getFollowedConstellations()  get the constellations owned by poets a user follows.
*   **(complete) Known Toolkit - Data Layer:** Add methods to firestore_service.dart to get the Private, Public, and Followed lists of constellations. 
*   **(complete) Known Toolkit - Scrollable Tiles:** Refactor Known Toolkit to create a list of scrollable constellation tiles for Private, Public, and Followed constellations.
*   **(complete) Known Toolkit - Filtering Logic:** Add the slicer logic for filtering Private, Public, and Followed constellations.
*   **(complete) Constellation Customization - Stale Snackbar:** (bug) Refactored `constellation_customization_screen.dart` so the neighbors snackbar uses a StreamBuilder to overcome a backend race condition.
*   **(complete) Publish Tab - Star Capacity Limit:** Added a cap of 10 to the number of stars that can be published to a constellation to prevent the navigation screen from becoming cluttered. 
*   **(complete) Publish Tab - Deduped RAG Recommendations:** Added deduplication logic to cloud run function generateconstellationrecommendations() so it always returns five constellations with less than 10 stars.
*   **(complete) Capped Constellation Icon:** Use a conditional symbols.star_shine for constellations that reach 10 stars.
*   **(complete) Constellation Customization - Stale Snackbar:** (bug) Fixed neighbors snackbar by implementing a synchronous RPC-style backend function. Removed unreliable Firestore onDocumentWritten trigger. 
*   **(complete) Automated Testing - Constellation Customization:** Refactored `constellations_customization_screen_test.dart`, `constellation_test.dart`, and `manage_tab_test.dart` for the new StreamBuilder logic.
*   **(complete) Automated Testing - Zeitgeist Map:** Add widget and unit testing for `zeitgeist.dart` and `zeitgeist_map.dart`.  
*   **(complete) Automated Testing - Explore Tab:** Add widget and unit testing for `explore_tab.dart` and `mini_constellation_graph_view.dart`.
*   **(complete) Automated Testing - ConstellationsScreen:** Add widget testing for `constellations_screen.dart` to verify tab rendering and placeholder content are rendered correctly.
*   **(complete) Publish Tab - Constellation Details:** Add fable image and text to constellation_detail_modal.dart so the user can review them before publishing. Also updated automated testing.
*   **(complete) Publish Tab - Audio Ready Indicator:** Added a trailing icon after the Star Name to indicate when an audio file is paired to the star.
*   **(complete) Publish Tab - Star Name at Top:** Added the Star Name above the slideshow preview.
*   **(complete) Fable Storage Clean-up:** (bug) Needed to clean-up fable images that the user discards in storage whenever the save button, visualize button, or exit button is pressed. Also updated automated testing.
*   **(complete) Manage Tab - Constellation Counter:** Add a banner at the top of the manage tab that shows the current number of constellations owned. Also updated automated testing.
*   **(complete) Manage Tab - Constellation Sharing Rules:** Ensure owners can publish to thier own private constellations, but other users can't see private constellations as publish recommendations. 
*   **(complete) Backend - Private Constellation Discoverability:** Updated getFollowedConstellations() and getThemeBasedConstellationRecommendations() with a dual-flag system (isPublic vs. isDiscoverable).
*   **(complete) Constellation Detail Modal - Private Constellations:** (bug) Constellation detail modal showed "Managed by: Unknown User". Fixed generateConstellationRecommendations() to denormalize owner's profile data.
*   **(complete) Backend - Corrected Stale Creator Data on Creation:** (bug) Moved the public profile read inside the Firestore transaction in the publishStar function to guarantee atomicity. Added "data healing" logic.
*   **(complete) Vector Search - Fixed Fable Indexing for Discoverability** (bug) Addressed a critical bug in saveConstellationCustomizations() where private, discoverable constellations were not appearing in searches.
*   **(complete) Automated Testing - Explore Tab:** Added a test to explicitly verify the interactions between all three filter buttons, ensuring the UI behaves as expected as the user toggles them in sequence.
*   **(complete) Zeitgeist Map - Word Collisions:** (bug) Replaced the random placement algorithm in `zeitgeist_map.dart` with a robust Archimedean Spiral layout to prevent word overlaps and clipping.
*   **(complete) Zeitgeist Engine - Data Integrity and Diversity:** The _zeitgeistEngineLogicguarantees a payload of 5 themes, sourced from a minimum of 3 unique constellations. Includes fallback logic for sparse data.
*   **(complete) Automated Testing - Zeitgeist Map:** Updated `zeitgeist_map_test.dart` for the refactoring of `zeitgeist_map.dart` to use a robust Archimedean Spiral layout to prevent word overlaps and clipping.
*   **(complete) Notifications Screen - Button Feedback:** (bug) Add a spinning indicator to the "Accept" button so the app doesn't appear to be non-reponsive when approving comments. Also updated automated testing.
*   **(complete) Home Screen - Button Size:** (bug) Stacked the spinning indicator on top of the "Generate Haiku" button in so the button does not change width on press. Also updated automated testing.
*   **(complete) Explore Tab - Stale Data:** (bug) The Discover, Known, Zeitgeist, and Launch toolkits have stale data (again!) and do not reflect the semantic neighbors currently in Firestore.
*   **(complete) Explore Tab - No Persistence:** (bug) The Known and Zeitgeist toolkits maintain user setting persistence while on Explore Tab, but lose persistence when navigating to other screens.
*   **(complete) Explore Tab - Excessive Mini Graph Rebuilds:** (bug) Fixed an issue where excessive Gemini API calls to get travel advice were causing way too many rebuilds, always making the mini-graph flash on rebuild.
*   **(complete) Automated Testing - Explore Tab:** Updated automated testing for the refactoring of `explore_tab.dart` and `mini_constellation_graph_view.dart`. This completes the community features spring! d(^_^d)
*   **(complete) Zietgeist Word Cloud Excessive Jitter:** (bug) Fixed an excessive rebuild loop by removing the retry logic out of the build method and into the calculate visual method itself."
*   **(complete) Zeitgeist Toolkit - Extra Rebuild:** (bug) The Zeitgeist map had an extra unnessary rebuild whenever a word was clicked. Removed the Gemini travel advice call from inside the Zeitgeist toolkit.
*   **(complete) Knowledge Graph Self-Pruning:** Added capability for any user to remove dead links automatically when visiting a new constellation. This only prunes dead links.
*   **(complete) Knowledge Graph Self-Healing:** Added weekly schduled Cloud Run function knowledgegraphhealer() to autmatically update semantic neighbors of constellations that have been pruned to zero!
*   **(complete) Fable Vector Search Clean-up:** Modified Cloud Run function deletePublishedStar() to remove a constellation that has been deleted from Firestore from the Fable vector search index as well.
*   **(complete) Vector Search Index Maintenace:** Added Cloud Run functions cleanupfableindex() and cleanupstarindex() to remove orphans from the indexes by reconciling them with Firestore database ground-truth.
*   **(complete) Settings Tab - Policies:** Add links to EULA and Privacy Statement. Used large, static string constants in `legal_text.dart` and markdown formatting using the markdown_widget package import.
*   **(complete) Settings Tab - User Feedback:** Add a user feedback system to collect user suggestions, bug reports, and questions, enabling continuous improvement based on user input.
*   **(complete) Settings - Constrained Textboxes:** Change the data entry Textboxes to constrained textboxes so they do not expand more than 512 pixels on the Chrome platform.
*   **(complete) Automated Testing - Explore Tab:** Updated `explore_tab_test.dart` and `mini_constellation_graph_view_test.dart` for the refactoring for several mini-graph performance upgrades.
*   **(complete) Automated Testing - Feedback Screen:** Add widget and unit testing for `feedback_screen.dart`, `feedback_category.dart`, and `feedback_item.dart`.
*   **(complete) Automated Testing - Settings Tab:** Add widget and unit testing for `eula_screen_test.dart`, `privacy_screen_test.dart`, and `settings_tab.dart`.
*   **(complete) Launch Toolkit - Mini-graph Navigation:** (bug) When in the launch toolkit only, tapping a neighbor node now updates the travel advice. This is a more intutive UX for the Launch toolkit.
*   **(complete) Explore Tab - Refectored for Robustness:** Methods _fetchAndSetTravelAdvice() and _engineerTravelAdvicePrompt() now receive parameters. They no longer rely on a time-sensitive external state variables.
*   **(complete) Automated Testing - Explore Tab:** Updated explore_tab_test.dart for the refactoring of Explore Tab so tapping a neighbor node now updates the travel advice. Also fixed a test-only initialization bug.
*   **(complete) Customization Tab - Avatar Studio:** Manage the user's profile picture and the frame selector using polymorphism for a common interface that every frame widget must implement.
*   **(complete) Customization Tab - Animation Frame:** Define an abstract class that all border frames must adhere to. It will require specific parameters like `child` and `flairData`.
*   **(complete) Customization Tab - Frame Seelctor:** Add a simple CarouselSlider that displays previews of the available frames.
*   **(complete) Customization Tab - Comet Frame:** Add a plugable module that contains its own internal logic to drive comet animation, trail, and burst effects. it will use a separate customFramePainter.
*   **(complete) Customization Tab - Pride Frame:** Add a plugable module that contains its own internal logic to drive Pride rainbow animation, trail, and burst effects. it will use a separate customFramePainter.
*   **(complete) Customization Tab - Flair Slots:**  Make the animated frames "smart" by connecting them to the user's earned achievements displayed in an achievement gallery below the Avatar Studio.
*   **(complete) Customization Tab - Flair Interactions:** Implement collision logic for activating the "burst" effect for commet collision(s). The CometFramePainter will receive the list of active flair targets. 
*   **(complete) Customization Tab - Rainy Pond Frame:** Add a plugable module that contains its own internal logic to drive Rainy Pond ripples that occasionally have interference patterns.
*   **(complete) Customization Tab - Mosaic Frame:** Add a plugable module that contains its own internal logic to drive Mosaic tile shimmer and wave of change.
*   **(complete) Customization Tab - Thorn Frame:** Add a plugable module that contains its own internal logic to drive vine growth and decay, thorn spawns, and less frequent flower spawns.
*   **(complete) Customization Tab - None Frame:** Add a plugable module that contains its own internal logic to turn off the frame border and show a default view.
*   **(complete) Customization Tab - Moonlit Motes Frame:** Add a plugable module that contains its own internal logic to drive moon mote spawn, merge, split, and decay lifecycles. Implemented a super-mote collider!
*   **(complete) Customization Tab - Lava Lamp Frame:** Add a plugable module that contains its own internal logic to drive a hypnotic, lava-lamp feel with a bright pink ploppy blob. This was a Halone-level stress trial.
*   **(complete) Customization Tab - Spirograph Frame:** Add a plugable module that contains its own internal logic to drive a three phase laser-etched thin line spirograph using three vibrant colors.
*   **(complete) Customization Tab - Ecosystem Frame:** Add a plugable module that contains its own internal logic to drive a simulation of flocking, avoidance, and hunter simulation. Pure masochism for a Monday!
*   **(complete) Customization Tab - Mycelium Frame:** Add a plugable module that contains its own internal logic to drive a simulation of a mycelium network. Threads pass a flash message upon fuzing to form a node.
*   **(complete) Customization Tab - Circle of Life Frame:** Add a plugable module that contains its own internal logic to drive a simulation of a circle of life that flashes the colors a a hummingbird!
*   **(complete) Customization Tab - Kintsugi Frame:** Add a plugable module that contains its own internal logic to drive a simulation of a repairing cracks in a ring of square elements. Gold fades to a scar and heals.
*   **(complete) Customization Tab - Contrast Backdrops:** Added white circle contrast to flair slots for Circle of Life, Kintsugi, and Mosaic frames. Modified avatar_studio.dart to pass the activeFlair parameter.
*   **(complete) Acheivments Gallery - Scroll Arrows:** Modified `achievment_gallery.dart` to add arrow on the right margin to indicate scrollable content. Modified grid layout to enlarge left and right margins.
*   **(complete) Customization Tab - Persistence:** Store user choices for image frame customization in Firestore public_profile and persist the choices each time the user navigates back to the customization tab.
*   **(complete) Star Popup - Poet Tab:** Refactor `star_detail_popup.dart` to integrate customized border frames for user profile images. Enlarged profile image and moved Poet data to its own row. d(^_^d) 
*   **(complete) Atomicity for Audio File Updates:** (bug) If a star is published, audio URL needs to update both the /users/favorites collection and the /published_stars collection with atomicity.
*   **(complete) Followers Tab - Followers:** A complete unabridged list of all the poet's greatest fans! Also add the capability to reciprocate the follow! =^.^=
*   **(complete) Automated Testing - Star Popup:** Modify `star_detail_popup_test.dart` and `public_profile_test.dart` for the refactoring of the Poet Tab to use customized border profile frames. 
*   **(complete) Automated Testing - Customization Tab:** Add widget and unit testing for `followers_tab.dart` and `follower.dart`.
*   **(complete) Automated Testing - Customization Tab:** Add widget and unit testing for `customization_tab.dart`, `avatar_studio.dart`, and `acheivement_gallery.dart`.
*   **(complete) Automated Testing - Customization Tab:** Add widget and unit testing for `frame_selector.dart`, `flair_target.dart`. The class `animated_frame.dart` does not require a dedicated regression test.
*   **(complete) Images Missing - Chrome:** (bug) Images not displayed on the Chrome plaform. Gemini tells me there is a support group's worth of developers who have hit the "Firebase Storage CORS Flutter Web" wall ('@_@)
*   **(complete) Authentication Screen - Constrained Textboxes:** Change the data entry Textboxes to constrained textboxes so they do not expand more than 512 pixels on the Chrome platform.
*   **(complete) Firebase App Check - Android:** Add Firebase App check for Android and Android Emulator to verify requests are coming from your authentic app, adding another layer of security.
*   **(complete) Firebase App Check - Web:** Add Firebase App check for Web using reCAPTCHA Enterprise to verify requests are coming from your authentic app, adding another layer of security.
*   **(complete) Profile Images Not Updating in Community Views:** (bug) Stale denormalized URLs in constellations not updated after a profile image change, causing 403 errors. Fixed with syncProfileImageOnUpdate().
*   **(complete) Additional Sign-In Methods:** Add support for Android and Web use of Google Sign-In provider as an alternative to email/password Firebase Authentication. 
*   **(complete) Google Sign-in for Web:** (bug) Google sign-in now works for the Web application. Development environment requires LocalHost:5000 and the user must be signed into Chrome. Three days of hell.
*   **(complete) Offical Google Sign-in Button for Web App:** Replace the custom Google sign-in button for Web with the official Google sign-in button for Web.
*   **(complete) Authentication Screen - EULA:** Add a one-time screen for new users to check the box for `EULA`.  Moved user sign-in navigation logic from main.dart to auth_screen.dart. 
*   **(complete) Google Sign-in State:** (bug) Fixed stale callback of subsequent calls of Google Sign-in after integration of `eula_acceptance_screen.dart` and moving sign-in navigation to `auth_screen.dart`.
*   **(complete) EULA Abandonment:** (bug) Fixed a loop-hole that allowed users to reach Home Screen by resarting the App from the EULA screen without accepting the EULA. More refactoring of main.dart, but succesful.
*   **(complete) Restore Email Verification:** (bug) Restored email verification step for email/password users only; this step occurs immediately before the EULA Acceptance screen and updates Firebase authentication.
*   **(complete) Haikuverse Logo:** Added an animated Haikuverse logo simulation to the authentication screen. Used a spirograph pattern with fixed frequencies, a color-changing text, and theme complimenting colors.
*   **(complete) Automated Testing - User Verification:** Update `auth_screen.dart`, `user_profile.dart`, and `settings_tab.dart` for the the refactored sign-in and EULA processes.
*   **(complete) Automated Testing - New User:** Add widget and unit testing for `verification_screen.dart` and `eula_acceptance_screen.dart`.
*   **(complete) Automated Testing - Google Sign-in Connectors** Add widget and unit testing for `google_sign_in_button_connector.,dart` and `gsi_connector.dart`.
*   **(complete) Automated Testing - Haikuverse Logo** Add widget and unit testing for `haikuverse_logo.dart`.
*   **(complete) Nickname Updates to Public Records:** (bug) When the user updates thier nickname, the change should trigger an onUpdate cloud function that updates all public records, like profile image updates. 
*   **(complete) PreferencesScreen - Constrained Layout:** Change the EULA, Privacy Statment, and Submit Feeback buttons to constrained layouts so they do not expand more than 512 pixels on the Chrome platform.
*   **(complete) Customization Tab - Constrained Grid:** Change the achievements matrix so is does not expand more than 512 pixels wide on the Chrome platform.
*   **(complete) Star Popup - Constrained Layout:** Change the Star Popup layout so is does not expand more than 900 pixels wide on the Chrome platform. Used a min() function for 90% media query or 900 pixels.
*   **(complete) Star Popup - Action Listener:** Wrapped GestureDector and PointerScrollEvent in a Listener so the the mouse button scrolls on Web and the pinch motion works on Android.
*   **(complete) Image Slots Screen - Constrained Layout:** Changed the Image Slots Screen layout so is does not expand more than 512 pixels wide on the Chrome platform.
*   **(complete) Cloud Run Functions - Synch Profile Image:** (bug) Updated Cloud Run function syncprofileimageonupdate() to ensure it correctly updates documents where a user can be both the creator and the owner.
*   **(complete) Cloud Run Functions - Synch Nickname:** (bug) Updated Cloud Run function syncnicknameonupdate() to ensure it correctly updates documents where a user can be both the creator and the owner.
*   **(complete) Upgrade from ACL to Uniform:** Refactored `main.dart` and `cors.json` to support the Google sign-in pop-up window for the Web app. This was insanely painful security upgrade, using `gsutil cors set`.
*   **(complete) Refactor Audio Player for Firebase Streaming:** Refactored `audio_player_provider.dart` to use signed URLs and StreamAudioSoruce() for greater efficiency and use less device memory. Also painful.
*   **(complete) Service Layer - Audio Generation:** Moved the `generateHaikuAudio()` Cloud Function call from the UI state (`_MyAppState`) into the service layer (`FirestoreService`).
*   **(complete) Configuration Service:** Manage endpoint URLs in `lib/constants/app_constants.dart` rather than hardcoding them directly in the service classes.
*   **(complete) Audio Toolkit Modal Responsiveness:** (bug) Refactored `audio_toolkit_modal.dart` to manage its own local state, fixing a race condition between UI updates and the Firestore stream.
*   **(complete) Automated Testing - Just Audio Player:** Update widget and unit testing for the refactoring of `audio_player_provider.dart` and `audio_toolkit_modal.dart` to use the service layer and signed URLs.
*   **(complete) Automated Testing - Constellation Info Display:** Update widget and unit testing for the refactoring of `constellation_info_display.dart` to use dependency and injection and Uniform access to storage.
*   **(complete) Automated Testing - Constellation Detail Modal:** Update widget and unit testing for the refactoring of `constellation_detail_modal.dart` to use dependency and injection and Uniform access to storage.
*   **(complete) Haikuverse - Star Levels:** Implement star size and color level-up schema for 5, 10, and 25 likes. Add corona effects that follow the theme_provider.
*   **(complete) Haikuverse - Fable Popup:** Add a top app bar icon button to popup the constellation fable text/image screen that is similar to the star popup, but without the tabs.
*   **(complete) Haikuverse - Navigation Tooltip:** Add a navigation tooltip to the bottom of the navigation screen that fades upon the user's first interaction with the navigation area of the screen.
*   **(complete) Automated Testing - Haikuverse:** Update `haikuverse_navigation_screen_test.dart` and `constellation_view_painter_test.dart` for star level effects, dissappearing tooltip, and icon for fable pop-up.
*   **(complete) Automated Testing - Fable Popup:** Add regression detection coverage for `fable_detail_popup.dart`.
*   **(complete) App Launcher Icon:** Created `assets/icon.png` and used `flutter pub run flutter_launcher_icons` to create icons. Created web/favicon.png separately due to 16-pixel size. 
*   **(complete) FavoritesScreen - Star Names:** Add starname to the top of each haiku tile in the favorites list. Also updated automated test `favorites_screen_test.dart`.
*   **(complete) Reorder Images Screen - Reorder Logic** Add `reorder_slots_screen.dart` with logic to reorder which save slots images are saved into using image drag-and-drop to target save slot. 
*   **(complete) Reorder Images - Dependencies:** Add a link from `stars_screen.dart` to `reorder_images_screen.dart` and update `main.dart` with the `_saveReorderedImages()` callback.
*   **(complete) Unpublish Update to Private Collection:** (bug) Update Cloud Run function deletepublishedstar() to remove `constellationId` from the /user/favorites colelction to match the /published_star collection.
*   **(complete) Automated Testing - Reorder Images Screen:** Add unit and widget level testing for `reorder_images_screen.dart` and update `stars_screen_test.dart` for the reorder images feature.
*   **(complete) Constellation Customization Screen - Constrained Layout:** Change the Constellation Customization Screen layout so is does not expand more than 750 pixels wide on the Chrome platform.
*   **(complete) Audio Files Not Playing on Chrome:** (bug) Changed policy in firebase.json from 'require-corp' to 'credentialless', to give the browser permission to fetch the public audio from storage.googleapis.com.
*   **(complete) Images/Audio Not Playing on Chrome:** (bug) Changed Firebase Hosting policy (COEP) and corrected service account IAM roles to resolve critical asset loading failures (403 Forbidden) on signed URLs.
*   **(complete) Settings tab - Green Snackbar:** (bug) Replaced the green-colored snackbar and `Profile saved` comment with a theme_provider snackbar and `Nickname saved` comment. Updated `settings_tab_test.dart`.
*   **(complete) StarsScreen - Loading Indicator Too Large:** (bug) The loading indicator for the Generate Image button is too large. Reduced it's size and added badding so the haiku text below it does not bounce.
*   **(complete) StarsScreen - Remove Author Field:** Removed `Author` field from `stars_screen.dart` to free up screen space. Also updated 6 automated tests in `stars_screen_test.dart` for the refactor.
*   **(complete) HomeScreen - Star Progress Tracker:** Add a star progress tracker with Text, Audio, Visual, and Publish milestones. It will fit into the space vacated by the naming toolkit. 
*   **(complete) Publication Tab - Vector Search:** (bug) Publish_tab.dart unable to handle when Vector Search API returns a distance value that is a whole number rather than decimal values. Also updated testing.
*   **(complete) Automated Testing - Star Progress Tracker:** Add regression detection for `star_progress_tracker.dart` and update `home_screen_test.dart` for the modifications that were made.
*   **(complete) Pride Frame - Animated Heart Control:** Make the mobile heart in the animate frame turn-on and turn-off depending on whether the "Likes" flair is displayed or not displayed.
*   **(complete) Interrupted Image Generation:** (bug) Fixed a data corruption issue due to a race condition if a user navigates away from an image in mid-generation by adding a callback to the correct haiku object.
*   **(complete) Web Media Access Failures:** (bug) Resolved 401 Unauthorized errors for images on the web client. Updating Firebase Storage rules to allow public reads (relying on secure download tokens for access). 
*   **(complete) Storage Bucket Access Failures:** (bug) Configured the Cloud Storage bucket's CORS policy to resolve browser errors from the Same-Origin Policy, allowing the web app to request and display media assets.
*   **(complete) Web Caching - Themes:** Implemented Web App caching so themes are separated by user using the browser's shared preferences.
*   **(complete) Web App - Theme Persistence:** (bug) Fixed a state management bug observed on the web where one user would inherit another's theme. The fix was applied to the shared multi-platform code.
*   **(complete) Web App - Sequential Sign-in:** (bug) Fixed a state management bug in the Google Sign-in library that prevented a user from signing in twice in a row.
*   **(complete) Android Stability - Web-only Crash:** (bug) Fixed a runtime crash on Android by adding a platform check (`kIsWeb`) in `main.dart` to a web-only Firebase function call.
*   **(complete) Android Stability - Compile Error:** (bug) Fixed a compile error on Android by isolating web-only code in `settings_tab.dart` using a new conditional import structure (`web_reloader` files).
*   **(complete) UI - Consistent Snackbar Colors:** Refactored multiple `SnackBar` widgets in `settings_tab.dart` to use theme-aware error colors instead of hardcoded values.
*   **(complete) Automated Testing - SignOutProvider:** Update the unit test for `signout_provider.dart` to verify it calls the sign-out methods for both Google and Firebase when on the web.
*   **(complete) Automated Testing - SettingsTab:** Update the widget test for `settings_tab.dart` to verify the sign-out button correctly triggers a page reload on the web and navigates correctly on mobile.
