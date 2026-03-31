# Gesture Recognizer Overhaul Design

## Problem

The current gesture recognition pipeline (MediaPipe Hands + FingerPose + custom heuristics) is fragile. Gesture 0 (fist) and 1 (index finger) are frequently confused, and adding workarounds creates cascading issues (e.g., thumbs_up misdetection). The rule-based classification layer (FingerPose + heuristics) is fundamentally brittle across different camera angles.

## Solution

Replace the classification layer with MediaPipe Gesture Recognizer (`@mediapipe/tasks-vision`), a production-grade ML-based solution from Google. Keep only `isVulcan()` as a secondary check for distinguishing vulcan from open palm.

## Final Gesture Set

| Index | Emoji | Name | Recognition Method |
|-------|-------|------|--------------------|
| 0 | ✊ | fist | MediaPipe: Closed_Fist |
| 1 | ☝️ | one | MediaPipe: Pointing_Up |
| 2 | ✌️ | two | MediaPipe: Victory |
| 5 | 🖐️ | five | MediaPipe: Open_Palm (and not vulcan) |
| 9 | 🖖 | vulcan | MediaPipe: Open_Palm + `isVulcan()` heuristic |

**Removed gestures:** three (3), six/call me (4), rock/horns (6)

## Architecture

### Recognition Pipeline

```
GestureRecognizer.recognizeForVideo(videoFrame, timestamp)
    ↓
Result: gesture name + confidence + landmarks
    ↓
mapGesture(result):
    Open_Palm → isVulcan(landmarks) ? 9 : 5
    Closed_Fist → 0
    Pointing_Up → 1
    Victory → 2
    other/None → -1
    ↓
updateCharge() — game logic (only accepts 0, 1, 2)
```

### What Gets Replaced

- **MediaPipe Hands** (`@mediapipe/hands`) → replaced by GestureRecognizer (includes hand detection)
- **FingerPose library** → removed entirely
- **Custom heuristics** → removed: `isFist()`, `isSix()`, `isRock()`, `disambiguateFistVsOne()`, `countExtendedFingers()`, `isThumbExtended()`
- **Stabilization window** (10-frame sliding window) → removed; GestureRecognizer has built-in `min_tracking_confidence`. Re-add simplified stabilization only if jitter is observed after migration.
- **Gesture templates** (FingerPose gesture descriptions) → removed

### What Gets Kept

- `isVulcan()` and its dependencies: `dist2D()`, `isFingerExtended()`, `getPalmCenter2D()`, `getPalmWidth()`, `angleDeg2D()`, `FINGER_ANGLE_THRESH`, `FINGER_LANDMARKS`
- All game logic (charge system, task cards, effects, UI)
- Three.js effects pipeline
- Camera setup logic (though initialization changes)

## Technical Integration

### Loading

Use CDN to load `@mediapipe/tasks-vision`, consistent with existing CDN-based loading of MediaPipe:

```html
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision/vision_bundle.mjs" type="module"></script>
```

Or use the recommended import pattern:

```javascript
import { GestureRecognizer, FilesetResolver } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision/vision_bundle.mjs";
```

### Initialization

```javascript
const vision = await FilesetResolver.forVisionTasks(
    "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision/wasm"
);
const gestureRecognizer = await GestureRecognizer.createFromOptions(vision, {
    baseOptions: {
        modelAssetPath: "https://storage.googleapis.com/mediapipe-models/gesture_recognizer/gesture_recognizer/float16/1/gesture_recognizer.task",
        delegate: "GPU"
    },
    runningMode: "VIDEO",
    numHands: 1,
    minHandDetectionConfidence: 0.7,
    minTrackingConfidence: 0.6
});
```

### Per-Frame Recognition

Replace the existing `hands.onResults()` callback with a requestAnimationFrame loop that calls:

```javascript
const result = gestureRecognizer.recognizeForVideo(video, performance.now());
```

Result structure:
- `result.gestures[0][0].categoryName` — gesture name string
- `result.gestures[0][0].score` — confidence (0-1)
- `result.landmarks[0]` — 21 hand landmarks (normalized x, y, z)

### Gesture Mapping Function

Replaces the entire `recognizeGesture()` function:

```javascript
function mapGesture(result) {
    if (!result.gestures || result.gestures.length === 0) return -1;

    var gesture = result.gestures[0][0];
    var name = gesture.categoryName;
    var landmarks = result.landmarks[0];

    if (name === 'Open_Palm') {
        return isVulcan(landmarks) ? 9 : 5;
    }

    var map = {
        'Closed_Fist': 0,
        'Pointing_Up': 1,
        'Victory': 2
    };

    return map[name] !== undefined ? map[name] : -1;
}
```

### Landmark Format Adaptation

MediaPipe Gesture Recognizer returns landmarks as `{x, y, z}` normalized objects, same format as the old MediaPipe Hands. The existing `isVulcan()` and its helper functions should work without modification.

## Game Logic Changes

### Task Cards

- Task card pool: only gestures 0, 1, 2
- Remove any references to gestures 3 (three), 4 (six), 6 (rock) from task generation

### Charge System

- Only 0, 1, 2 trigger charge/task completion
- Gestures 5 (five) and 9 (vulcan) are recognized and shown visually but do not participate in game mechanics

### UI

- Remove six (🤙) and rock (🤘) icons from any gesture displays
- Keep five (🖐️) and vulcan (🖖) visual feedback (emoji display) as easter eggs
- Update gesture icon mapping to reflect new index set

## Constraints

- No training step required — uses only MediaPipe's pre-trained model
- Pure CDN loading, no npm/bundler changes
- Maintain existing camera resolution settings (1280x720 desktop, 640x480 mobile)
- If stabilization jitter is observed after migration, re-add a simplified sliding window (but start without it)

## Files to Modify

- `index.html` — all changes in this single file:
  - Replace MediaPipe Hands + FingerPose script tags with `@mediapipe/tasks-vision`
  - Replace initialization code
  - Replace per-frame recognition callback
  - Replace `recognizeGesture()` with `mapGesture()`
  - Remove unused heuristic functions and FingerPose definitions
  - Update game logic (task cards, gesture mapping, UI)
  - Keep `isVulcan()` and its dependencies
