# Gesture Recognizer Overhaul Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace FingerPose + custom heuristics with MediaPipe Gesture Recognizer for production-grade gesture recognition, keeping `isVulcan()` as a secondary check.

**Architecture:** Replace the entire MediaPipe Hands + FingerPose pipeline with `@mediapipe/tasks-vision` GestureRecognizer. Change `GESTURE_ICONS`/`GESTURE_NAMES` from arrays to objects keyed by gesture index (0,1,2,5,9). Remove all unused heuristic functions except `isVulcan()` and its dependencies.

**Tech Stack:** `@mediapipe/tasks-vision` (CDN), vanilla JavaScript, single-file `index.html`

---

### File Structure

- **Modify:** `index.html` — all changes in this single file

---

### Task 1: Change GESTURE_ICONS and GESTURE_NAMES from arrays to objects

Currently `GESTURE_ICONS` is an array indexed 0-6. The new gesture set uses sparse indices (0,1,2,5,9), so it must become an object map. Every reference to `GESTURE_ICONS[n]` throughout the file must continue to work.

**Files:**
- Modify: `index.html:1366-1368` (config section)
- Modify: `index.html:1477` (GESTURE_NAMES_ZH)
- Modify: `index.html:1550` (playSlotReel allIcons)
- Modify: `index.html:1677` (revealWrongGesture bounds check)

- [ ] **Step 1: Change GESTURE_ICONS, GESTURE_NAMES, and THEME_LABELS_ZH to objects**

Change lines 1366-1368 from:

```javascript
    const GESTURE_ICONS = ['\u270A', '\u261D\uFE0F', '\u270C\uFE0F', '\uD83D\uDD90\uFE0F', '\uD83E\uDD19', '\uD83D\uDD96', '\uD83E\uDD18'];
    const GESTURE_NAMES = ['Fist (0)', 'One (1)', 'Two (2)', 'Five (3)', 'Six (4)', 'Vulcan (5)', 'Rock (6)'];
    const THEME_LABELS_ZH = ['勢在必行', '自省', '覺醒', '不是這個', '六六大順', '生生不息', '搖滾'];
```

to:

```javascript
    const GESTURE_ICONS = {0: '\u270A', 1: '\u261D\uFE0F', 2: '\u270C\uFE0F', 5: '\uD83D\uDD90\uFE0F', 9: '\uD83D\uDD96'};
    const GESTURE_NAMES = {0: 'Fist (0)', 1: 'One (1)', 2: 'Two (2)', 5: 'Five (5)', 9: 'Vulcan (9)'};
    const THEME_LABELS_ZH = {0: '勢在必行', 1: '自省', 2: '覺醒', 5: '不是這個', 9: '生生不息'};
```

- [ ] **Step 2: Fix playSlotReel to use Object.values**

Change line 1550 from:

```javascript
        const allIcons = GESTURE_ICONS.slice();
```

to:

```javascript
        const allIcons = Object.values(GESTURE_ICONS);
```

- [ ] **Step 3: Fix revealWrongGesture bounds check**

Change line 1677 from:

```javascript
        if (wrongG < 0 || wrongG >= GESTURE_ICONS.length) {
```

to:

```javascript
        if (wrongG < 0 || !GESTURE_ICONS[wrongG]) {
```

- [ ] **Step 4: Fix GESTURE_NAMES_ZH**

Change line 1477 from:

```javascript
    const GESTURE_NAMES_ZH = ['拳頭', '一根手指', '兩根手指', '三根手指', '四根手指', '手掌'];
```

to:

```javascript
    const GESTURE_NAMES_ZH = {0: '拳頭', 1: '一根手指', 2: '兩根手指', 5: '手掌', 9: '瓦肯'};
```

- [ ] **Step 5: Verify all GESTURE_ICONS references still work**

All existing references use `GESTURE_ICONS[someIndex]` which works identically with object property access. No other changes needed for: lines 1457, 1498, 1561, 1653, 1687, 1689, 1704, 1719, 1751, 1820.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Change gesture icon/name arrays to objects for sparse index support"
```

---

### Task 2: Update help screen to remove six and rock gestures

**Files:**
- Modify: `index.html:1093-1099` (help modal gesture list)

- [ ] **Step 1: Remove six and rock from help gesture list**

Change lines 1093-1099 from:

```html
                <div class="help-gesture-item"><span class="emoji">✊</span><span class="label">拳頭</span></div>
                <div class="help-gesture-item"><span class="emoji">☝️</span><span class="label">一根手指</span></div>
                <div class="help-gesture-item"><span class="emoji">✌️</span><span class="label">兩根手指</span></div>
                <div class="help-gesture-item"><span class="emoji">🖐️</span><span class="label">五根手指</span></div>
                <div class="help-gesture-item"><span class="emoji">🤙</span><span class="label">六</span></div>
                <div class="help-gesture-item"><span class="emoji">🖖</span><span class="label">瓦肯</span></div>
                <div class="help-gesture-item"><span class="emoji">🤘</span><span class="label">搖滾</span></div>
```

to:

```html
                <div class="help-gesture-item"><span class="emoji">✊</span><span class="label">拳頭</span></div>
                <div class="help-gesture-item"><span class="emoji">☝️</span><span class="label">一根手指</span></div>
                <div class="help-gesture-item"><span class="emoji">✌️</span><span class="label">兩根手指</span></div>
                <div class="help-gesture-item"><span class="emoji">🖐️</span><span class="label">五根手指</span></div>
                <div class="help-gesture-item"><span class="emoji">🖖</span><span class="label">瓦肯</span></div>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Remove six and rock gestures from help screen"
```

---

### Task 3: Replace MediaPipe Hands + FingerPose with GestureRecognizer

This is the core task. Replace script tags, initialization, per-frame callback, and gesture mapping.

**Files:**
- Modify: `index.html:1136-1139` (script tags)
- Modify: `index.html:1870-1924` (initCamera + onHandResults)
- Modify: `index.html:1927-1997` (FingerPose definitions, replaced with mapGesture)
- Modify: `index.html:2000-2030` (stabilizer — keep but simplify)

- [ ] **Step 1: Replace script tags**

Change lines 1136-1139 from:

```html
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands@0.4.1646424915/hands.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/fingerpose@0.1.0/dist/fingerpose.min.js"></script>
```

to:

```html
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js"></script>
```

- [ ] **Step 2: Replace initCamera with GestureRecognizer initialization**

Replace the entire `initCamera()` function (lines 1870-1902) with:

```javascript
    let gestureRecognizer = null;

    async function initCamera() {
        if (cameraStarted) return;
        cameraStarted = true;

        const videoElement = document.querySelector('.camera-video');
        const mobile = isMobile();

        // Load GestureRecognizer
        const { GestureRecognizer, FilesetResolver } = await import(
            'https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/vision_bundle.mjs'
        );

        const vision = await FilesetResolver.forVisionTasks(
            'https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm'
        );

        gestureRecognizer = await GestureRecognizer.createFromOptions(vision, {
            baseOptions: {
                modelAssetPath: 'https://storage.googleapis.com/mediapipe-models/gesture_recognizer/gesture_recognizer/float16/1/gesture_recognizer.task',
                delegate: mobile ? 'CPU' : 'GPU'
            },
            runningMode: 'VIDEO',
            numHands: 1,
            minHandDetectionConfidence: 0.7,
            minTrackingConfidence: 0.6
        });

        const camWidth = mobile ? 640 : 1280;
        const camHeight = mobile ? 480 : 720;

        const cam = new Camera(videoElement, {
            onFrame: async function() {
                if (!gestureRecognizer) return;
                const result = gestureRecognizer.recognizeForVideo(videoElement, performance.now());
                onGestureResult(result);
            },
            width: camWidth,
            height: camHeight,
            facingMode: 'user',
        });
        cam.start();
    }
```

- [ ] **Step 3: Replace onHandResults with onGestureResult**

Replace the entire `onHandResults()` function (lines 1904-1924) with:

```javascript
    function onGestureResult(result) {
        if (!result.landmarks || result.landmarks.length === 0) {
            if (gameState.phase === 'PLAYING') {
                detectedGestureEl.textContent = '?';
                detectedGestureEl.classList.remove('matching');
                resetCharge();
                gameState.tasks.forEach((t, i) => {
                    document.getElementById('task-card-' + i).classList.remove('active');
                });
                hintText.textContent = '比一個手勢來猜猜看';
            }
            return;
        }

        const rawGesture = mapGesture(result);
        const gesture = stabilize(rawGesture);

        updateCharge(gesture, true);
    }
```

- [ ] **Step 4: Replace FingerPose definitions with mapGesture function**

Remove the entire FingerPose section (lines 1927-1998: all gesture descriptions, gestureEstimator, GESTURE_NAME_TO_INDEX, GESTURE_MIN_SCORE) and replace with:

```javascript
    // ============================================================
    // GESTURE MAPPING
    // ============================================================
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

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Replace MediaPipe Hands + FingerPose with GestureRecognizer"
```

---

### Task 4: Remove unused heuristic functions

Remove all functions and code that are no longer needed. Keep `isVulcan()` and its dependencies.

**Files:**
- Modify: `index.html` — remove dead code

- [ ] **Step 1: Remove unused functions**

Remove the following functions entirely:
- `isFist()` (lines ~2138-2163)
- `isSix()` (lines ~2165-2172)
- `isRock()` (lines ~2174-2181)
- `disambiguateFistVsOne()` (the function we added earlier)
- `isThumbExtended()` (lines ~2120-2127)
- `countExtendedFingers()` (lines ~2129-2136)
- `recognizeGesture()` (the entire old function, lines ~2213-2286)

**Keep** these (needed by `isVulcan()`):
- `dist2D()`
- `angleDeg2D()`
- `getPalmCenter2D()`
- `getPalmWidth()`
- `isFingerExtended()`
- `FINGER_ANGLE_THRESH`
- `FINGER_LANDMARKS`
- `isVulcan()`

Also remove `angleDeg()` (3D version, lines ~2064-2072) and the comment "Keep 3D version for fingerpose compatibility" — no longer needed.

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Remove unused heuristic functions (isFist, isSix, isRock, etc.)"
```

---

### Task 5: Adapt isVulcan for GestureRecognizer landmark format

MediaPipe Gesture Recognizer returns landmarks as `NormalizedLandmark` objects with `{x, y, z}` properties, same as the old MediaPipe Hands. However, we need to verify `isVulcan()` works correctly with the new landmark format.

**Files:**
- Modify: `index.html` — adapt `isVulcan()` if needed

- [ ] **Step 1: Verify landmark format compatibility**

The GestureRecognizer returns `result.landmarks[0]` as an array of `{x, y, z}` normalized objects. This is the same format that `isVulcan()` currently expects (it uses `landmarks[N].x` and `landmarks[N].y` via `dist2D` and `angleDeg2D`).

No code change needed if formats match. If they differ, adapt `isVulcan()` to use the new format.

- [ ] **Step 2: Commit (only if changes were needed)**

```bash
git add index.html
git commit -m "Adapt isVulcan for GestureRecognizer landmark format"
```

---

### Task 6: Manual testing

- [ ] **Step 1: Start local server and open browser**

```bash
python3 -m http.server 8080
```

Open `http://localhost:8080` in browser with DevTools Console open.

- [ ] **Step 2: Test each gesture**

| Gesture | Expected | Check |
|---------|----------|-------|
| ✊ fist | Recognized as 0 | Multiple angles |
| ☝️ one finger | Recognized as 1 | Straight, tilted, **side view** |
| ✌️ two fingers | Recognized as 2 | Multiple angles |
| 🖐️ open palm | Recognized as 5 | Fingers together |
| 🖖 vulcan | Recognized as 9 | Fingers spread with middle-ring gap |

- [ ] **Step 3: Test game flow**

- Start game → 3 task cards show (gestures 0, 1, 2)
- Complete all 3 → success screen
- Replay works

- [ ] **Step 4: Test edge cases**

- No hand visible → shows "?"
- Thumbs up → should return -1 (no charge)
- Transition between 0 and 1 → should be clean, no jitter
- five (5) does not trigger task completion (not in task pool)

- [ ] **Step 5: Commit any fixes from testing**

```bash
git add index.html
git commit -m "Fix issues found during manual testing"
```
