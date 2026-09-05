# MobilePhotonExplore — Version 1 Plan

## 1. Version 1 Goal

Build a small reusable Android library named `MobilePhotonExplore` that can be added to each MobilePhoton app.

The library will:

- Fetch inspiration content from a public GitHub-hosted URL.
- Display a page containing a list of scenario cards.
- Open a full detail page when a card is tapped.
- Display an image, animated GIF, or video on the detail page.
- Show the MobilePhoton apps used for each scenario.
- Support tappable links on the detail page.
- Cache downloaded content for offline use.
- Show an empty state when no cached or remote content is available.

Version 1 is intentionally limited to one list design and one detail design. Content can change remotely, but changes to the screen layout or behavior will require a new library and app release.

## 2. User Experience

### Entry Point

Each MobilePhoton app adds a **Get Inspired** entry in its existing UI.

Tapping it opens the inspiration list supplied by `MobilePhotonExplore`.

### Inspiration List

The first page displays a vertically scrolling list of cards.

Each card contains:

- An optional static background or poster image.
- A title.
- A short description.
- A row of icons representing the apps used for the scenario.

Example:

```text
┌────────────────────────────────────┐
│                                    │
│        Background photograph       │
│                                    │
│  Star Trails                       │
│  Capture and combine a sequence    │
│  of photos into smooth trails.     │
│                                    │
│  [Motion ProCam] [LightTrails]     │
└────────────────────────────────────┘
```

Tapping anywhere on the card opens its detail page.

### Scenario Detail

The detail page contains:

- One large image, animated GIF, or video.
- The scenario title.
- The full description.
- Optional sections or paragraphs.
- The apps used for the scenario.
- Optional tappable links.

Links may open:

- A related web page.
- A MobilePhoton app through a deep link.
- The app's Google Play page.

Videos display a poster first and play only after the user taps them. They include basic play, pause, seek, and mute controls. GIFs animate only while their detail page is visible. Cards always use static preview images so scrolling remains lightweight.

Version 1 does not need rich text or remote HTML. Detail content is displayed as plain paragraphs with optional link buttons. Each scenario supports one primary detail media item; media galleries can be added later.

## 3. Version 1 Content Format

Use a single `library.json` file containing the app definitions and scenarios.

```json
{
  "version": 1,
  "title": "Explore What’s Possible",
  "subtitle": "Creative photography ideas using MobilePhoton apps.",
  "apps": [
    {
      "id": "motion-procam",
      "name": "Motion ProCam",
      "icon": "images/apps/motion-procam.png",
      "deepLink": "",
      "playStoreUrl": "https://play.google.com/store/apps/details?id=com.mobilephoton.motionprocam"
    },
    {
      "id": "lighttrails",
      "name": "LightTrails",
      "icon": "images/apps/lighttrails.png",
      "deepLink": "",
      "playStoreUrl": "https://play.google.com/store/apps/details?id=com.mobilephoton.lighttrails"
    }
  ],
  "scenarios": [
    {
      "id": "star-trails-01",
      "title": "Star Trails",
      "shortDescription": "Capture and combine a sequence of photos into smooth star trails.",
      "cardImage": "images/star-trails-01-card.webp",
      "detailMedia": {
        "type": "video",
        "url": "videos/star-trails-01.mp4",
        "poster": "images/star-trails-01.webp"
      },
      "appIds": ["motion-procam", "lighttrails"],
      "content": [
        {
          "heading": "Capture the frames",
          "body": "Use Motion ProCam to capture a sequence of photos over time."
        },
        {
          "heading": "Create the trail",
          "body": "Open the sequence in LightTrails and combine the frames."
        }
      ],
      "links": [
        {
          "title": "Open Motion ProCam",
          "url": "https://mobilephoton.com/apps/motion-procam"
        },
        {
          "title": "Open LightTrails",
          "url": "https://mobilephoton.com/apps/lighttrails"
        }
      ]
    }
  ]
}
```

`detailMedia.type` supports `image`, `gif`, or `video`:

```json
{ "type": "image", "url": "images/star-trails-01.webp", "poster": "" }
```

```json
{ "type": "gif", "url": "gifs/star-trails-01.gif", "poster": "images/star-trails-01.webp" }
```

```json
{ "type": "video", "url": "videos/star-trails-01.mp4", "poster": "images/star-trails-01.webp" }
```

All media paths are relative to the location of `library.json`. App icons use PNG so they can preserve transparency. Photography images can use WebP, animations use GIF, and version 1 videos use MP4. A GIF or video should include a static WebP poster for loading and offline display.

The `deepLink` field is left empty in version 1. It is reserved for direct app-to-app navigation in a later version.

Keep scenario and app IDs stable after publishing them.

## 4. Public Content Repository

Use this simple published structure:

```text
mobilephoton-library/
├── library.json
├── images/
│   ├── apps/
│   │   ├── motion-procam.png
│   │   └── lighttrails.png
│   ├── star-trails-01-card.webp
│   └── star-trails-01.webp
├── gifs/
│   └── star-trails-01.gif
└── videos/
    └── star-trails-01.mp4
```

GitHub Actions publishes these files as static content through GitHub Pages.

The app reads the content from one stable HTTPS address, for example:

```text
https://mobilephoton.github.io/mobilephoton-library/library.json
```

If a MobilePhoton custom domain is available, prefer:

```text
https://content.mobilephoton.com/library.json
```

The custom domain makes it easier to move the content to another host later without changing every app.

## 5. Android Library Structure

Create one self-contained Android library module. It follows the same Java and Android Views source-module approach as the existing `imagepicker` library so current MobilePhoton apps can import it without adding Kotlin or Compose build configuration.

```text
MobilePhotonExplore/
├── model/
│   ├── LibraryContent.java
│   ├── InspirationScenario.java
│   ├── ScenarioMedia.java
│   ├── MobilePhotonApp.java
│   ├── ContentSection.java
│   └── ContentLink.java
├── data/
│   ├── LibraryService.java
│   ├── LibraryCache.java
│   └── LibraryRepository.java
├── ui/
│   ├── InspirationAdapter.java
│   ├── AppIconViews.java
│   └── ScenarioMediaView.java
├── navigation/
│   └── LibraryLinkOpener.java
├── MobilePhotonExplore.java
├── MobilePhotonExploreActivity.java
├── MobilePhotonExploreConfig.java
└── res/
    ├── layout/
    └── drawable/
```

### `LibraryService`

- Downloads `library.json`.
- Converts it into Java content models.
- Resolves relative image paths against the content URL.

### `LibraryCache`

- Stores the last downloaded JSON file.
- Stores downloaded images and GIFs through the selected media loader's disk cache.
- Streams videos when the user chooses to play them instead of downloading every video in advance.

### `LibraryRepository`

- Gives the UI one place to request library content.
- Returns cached content immediately when available.
- Refreshes remote content when the inspiration page opens.
- Keeps using the existing content if a refresh fails.

### Android Views UI

- `MobilePhotonExploreActivity` displays the scrolling list and complete detail page.
- `InspirationAdapter` displays the image, title, short description, and app icons on each card.
- `AppIconViews` resolves app IDs and displays their icons.
- `ScenarioMediaView` displays the detail image or GIF, or provides video playback controls.

### `LibraryLinkOpener`

- Opens HTTPS links using an Android intent.
- Opens Google Play URLs for the related MobilePhoton apps.
- Keeps deep-link support reserved for a later version.

## 6. Library Configuration

Each host app provides a small configuration object:

```java
MobilePhotonExploreConfig config = new MobilePhotonExploreConfig.Builder(
        "motion-procam")
        .build();
```

The default builder fetches `library.json` directly from the GitHub repository over HTTPS. A host
app can pass a different content URL as the second argument when it needs another endpoint.

For version 1, `currentAppId` is available to the library but does not need to change the card layout. It can be used on the detail page to avoid recommending the app that the user is already using.

The library supplies its own day/night theme. Host apps can override its `mpe_` color resources when they want the screens to match app-specific branding.

## 7. Loading Flow

When the user opens **Get Inspired**:

1. Load the last cached `library.json`.
2. Fetch the latest remote `library.json` in the background.
3. Display cached content immediately when available.
4. Save and display the remote content when it is available.
5. Load card images and detail media only when the UI needs them.
6. Stream video only after the user taps Play.

Do not block the page while waiting for GitHub if cached content is available.

## 8. Basic UI States

Version 1 needs only these states:

- **Content:** show the scenario cards.
- **Initial loading:** show a small loading indicator while remote content is loading.
- **Empty:** show a short message when there are no scenarios.
- **Offline:** silently continue showing cached content.

An offline warning is not necessary for version 1 because the content remains usable.

## 9. First Content Set

Start with three scenarios:

1. Star Trails
2. Light Trails
3. Motion Blur

Each scenario needs:

- One card image.
- One detail image, GIF, or video.
- One static poster when the detail media is a GIF or video.
- A title and short description.
- Two or three short detail sections.
- The relevant app IDs.
- At least one useful app or website link.

This is enough content to establish the visual design and content update flow.

## 10. Implementation Steps

### Phase 1 — Remote Content

1. Create the content repository.
2. Add `library.json`.
3. Add app icons and the first scenario images.
4. Publish the folder through GitHub Pages.
5. Confirm the JSON and images have stable public HTTPS URLs.

### Phase 2 — Library Data Layer

1. Create the Android library module.
2. Add the Java content models, including the media type.
3. Implement JSON downloading.
4. Implement the local JSON cache.
5. Add image and GIF loading with memory and disk caching.
6. Add simple streamed MP4 video playback.

### Phase 3 — Library UI

1. Build the inspiration card.
2. Build the scrolling inspiration list.
3. Build the scenario detail page.
4. Display app icons from the app registry.
5. Display image, GIF, or video detail media.
6. Add link buttons to the detail page.

### Phase 4 — First App Integration

1. Add the library dependency to one MobilePhoton app.
2. Add the **Get Inspired** entry.
3. Pass the app ID and content URL to the library.
4. Connect the list and detail navigation.
5. Connect external link opening.

### Phase 5 — Remaining Apps

1. Add the same library dependency to each remaining app.
2. Pass the correct `currentAppId` from each app.
3. Add the **Get Inspired** entry to each app.

## 11. Content Update Flow

After version 1 is released, content updates follow this process:

```text
Add or edit a scenario
        ↓
Update its media and library.json
        ↓
Push the changes to GitHub
        ↓
GitHub Pages publishes the files
        ↓
Each app receives the update the next time it refreshes
```

No Android app release is needed for changes to images, GIFs, videos, titles, descriptions, sections, app references, or links that use the version 1 content format.

## 12. Not Included in Version 1

Keep these out of the first implementation:

- Multiple card or detail layouts.
- Remote HTML or server-driven UI.
- User accounts, favorites, and syncing.
- Search, filtering, and category screens.
- Multiple-media galleries or playlists.
- Autoplaying video in the scenario list.
- Content pagination.
- Push notifications for new content.
- Analytics specific to the inspiration library.
- Localization of remote content.
- A content-management dashboard.
- Automatic scheduled background synchronization.
- Manifest signing or advanced security infrastructure.
- A dedicated CDN or backend API.

These can be added later after the basic list, detail, caching, and remote update flow is working in all apps.
