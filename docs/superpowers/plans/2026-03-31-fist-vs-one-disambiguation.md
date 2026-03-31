# Fist vs One Gesture Disambiguation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reduce misdetection between gesture 0 (fist) and gesture 1 (index finger) by adding a fingertip outlier ratio check.

**Architecture:** A new `disambiguateFistVsOne(landmarks)` function computes how far the index fingertip is from the palm relative to the other three fingertips. This ratio is used to override ambiguous 0/1 decisions at three points in `recognizeGesture()`.

**Tech Stack:** Vanilla JavaScript in `index.html`, using existing MediaPipe hand landmark data.

---

### File Structure

- **Modify:** `index.html` — add `disambiguateFistVsOne()` function and integrate it into `recognizeGesture()`

---

### Task 1: Add `disambiguateFistVsOne()` function

**Files:**
- Modify: `index.html:2163` (after `isFist()`, before `isSix()`)

- [ ] **Step 1: Add the disambiguation function after `isFist()` (line 2163)**

Insert the following function between `isFist()` and `isSix()`:

```javascript
    function disambiguateFistVsOne(landmarks) {
        var palmCenter = getPalmCenter2D(landmarks);
        var indexDist = dist2D(landmarks[8], palmCenter);
        var middleDist = dist2D(landmarks[12], palmCenter);
        var ringDist = dist2D(landmarks[16], palmCenter);
        var pinkyDist = dist2D(landmarks[20], palmCenter);

        var otherAvg = Math.max((middleDist + ringDist + pinkyDist) / 3, 0.001);
        var outlierRatio = indexDist / otherAvg;

        console.log('[disambiguate] outlierRatio:', outlierRatio.toFixed(2));

        if (outlierRatio > 1.8) return 'one';
        if (outlierRatio < 1.3) return 'fist';
        return 'ambiguous';
    }
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Add disambiguateFistVsOne() function with outlier ratio detection"
```

---

### Task 2: Integrate disambiguation after `isFist()` check

**Files:**
- Modify: `index.html:2214` (inside `recognizeGesture()`)

- [ ] **Step 1: Replace the direct fist return with disambiguation check**

Change line 2214 from:

```javascript
        if (fist) return 0;
```

to:

```javascript
        if (fist) {
            var fistCheck = disambiguateFistVsOne(landmarks);
            if (fistCheck === 'one') return 1;
            return 0;
        }
```

This means: if `isFist()` says fist but the index finger is clearly an outlier, override to gesture 1. Otherwise (fist or ambiguous), keep gesture 0.

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Add disambiguation check after isFist() detection"
```

---

### Task 3: Integrate disambiguation after FingerPose fist result

**Files:**
- Modify: `index.html:2235-2237` (inside `recognizeGesture()`, FingerPose fist branch)

- [ ] **Step 1: Add disambiguation to the FingerPose fist branch**

Change lines 2235-2237 from:

```javascript
            if (gestureName === 'fist') {
                return 0; // fist (isFist already handles thumb check)
            }
```

to:

```javascript
            if (gestureName === 'fist') {
                var fpFistCheck = disambiguateFistVsOne(landmarks);
                if (fpFistCheck === 'one') return 1;
                return 0;
            }
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Add disambiguation check after FingerPose fist classification"
```

---

### Task 4: Integrate disambiguation after fallback gesture 0 and 1 results

**Files:**
- Modify: `index.html:2245-2249` (inside `recognizeGesture()`, fallback block)

- [ ] **Step 1: Add disambiguation to the fallback 0/1 branches**

Change lines 2245-2249 from:

```javascript
        if (!thumbOut) {
            if (extendedFingers === 0) return 0; // fist
            if (extendedFingers === 1) return 1; // one
            if (extendedFingers === 2) return 2; // two
            if (extendedFingers >= 3) return 3;  // treat 3-4 fingers as five
        }
```

to:

```javascript
        if (!thumbOut) {
            if (extendedFingers === 0) {
                var fbFistCheck = disambiguateFistVsOne(landmarks);
                if (fbFistCheck === 'one') return 1;
                return 0;
            }
            if (extendedFingers === 1) {
                var fbOneCheck = disambiguateFistVsOne(landmarks);
                if (fbOneCheck === 'fist') return 0;
                return 1;
            }
            if (extendedFingers === 2) return 2; // two
            if (extendedFingers >= 3) return 3;  // treat 3-4 fingers as five
        }
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Add disambiguation check in fallback gesture 0/1 detection"
```

---

### Task 5: Integration after FingerPose open_palm with 1 extended finger

**Files:**
- Modify: `index.html:2226-2228` (inside `recognizeGesture()`, open_palm branch)

- [ ] **Step 1: Add disambiguation when FingerPose says open_palm with 1 finger**

Change lines 2226-2230 from:

```javascript
            if (gestureName === 'open_palm' && !thumbOut) {
                // Without thumb: map by finger count (only 1 and 2 have indices)
                if (extendedFingers === 1) return 1;
                if (extendedFingers === 2) return 2;
                return -1;
            }
```

to:

```javascript
            if (gestureName === 'open_palm' && !thumbOut) {
                if (extendedFingers === 1) {
                    var fpOneCheck = disambiguateFistVsOne(landmarks);
                    if (fpOneCheck === 'fist') return 0;
                    return 1;
                }
                if (extendedFingers === 2) return 2;
                return -1;
            }
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "Add disambiguation check in FingerPose open_palm single-finger path"
```

---

### Task 6: Manual testing and threshold tuning

- [ ] **Step 1: Open the app in a browser and open the developer console**

Run: Open `index.html` in the browser (or the dev server), open DevTools Console.

- [ ] **Step 2: Test gesture 0 (fist) at various angles**

Show a fist to the camera at different angles (straight on, tilted, rotated). Check:
- Console logs should show `outlierRatio` values below 1.3
- Gesture should be recognized as 0

- [ ] **Step 3: Test gesture 1 (index finger) at various angles**

Show index finger to the camera at different angles. Check:
- Console logs should show `outlierRatio` values above 1.8
- Gesture should be recognized as 1

- [ ] **Step 4: Adjust thresholds if needed**

If outlier ratios overlap between gestures 0 and 1, adjust the thresholds in `disambiguateFistVsOne()`:
- Lower the `1.8` threshold if gesture 1 is not being recognized
- Raise the `1.3` threshold if gesture 0 is being overridden incorrectly
- Keep a gap between the two thresholds for the `ambiguous` zone

- [ ] **Step 5: Remove or reduce console.log after tuning**

Once thresholds are validated, consider reducing log frequency (e.g., log every 30 frames) or removing the log entirely.

- [ ] **Step 6: Commit final tuned thresholds**

```bash
git add index.html
git commit -m "Tune fist vs one disambiguation thresholds after testing"
```
