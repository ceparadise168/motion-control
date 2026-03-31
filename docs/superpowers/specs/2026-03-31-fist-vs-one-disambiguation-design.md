# Fist vs One Gesture Disambiguation Design

## Problem

Gesture 0 (fist) and gesture 1 (index finger up) are frequently misdetected as each other. Both gestures share 4 curled fingers, and varying camera angles make it difficult to distinguish whether the index finger is extended or curled.

## Constraints

- Pure software fix; no change to user gesture habits
- Keep existing 10-frame stabilization window and response speed unchanged
- Gestures 0, 1, and 2 are all equally important for the game
- Minimal code changes; do not alter `isFist()`, FingerPose classifier definitions, or stabilization logic

## Approach: Fingertip Outlier Ratio

Core insight: when showing "1", the index fingertip is an outlier — significantly farther from the palm than the other 3 fingertips. When showing "0", all fingertips are clustered near the palm at similar distances.

### New Function: `disambiguateFistVsOne(landmarks)`

**Input:** 21 hand landmarks

**Output:** `"fist"` | `"one"` | `"ambiguous"`

**Algorithm:**

1. Compute palm center (reuse existing `palmCenter` calculation)
2. Compute distance from each of the 4 fingertips to palm center:
   - `indexDist` = distance(landmark[8], palmCenter)
   - `middleDist` = distance(landmark[12], palmCenter)
   - `ringDist` = distance(landmark[16], palmCenter)
   - `pinkyDist` = distance(landmark[20], palmCenter)
3. Compute average distance of the other 3 fingers:
   - `otherAvg` = (middleDist + ringDist + pinkyDist) / 3
4. Compute outlier ratio:
   - `outlierRatio` = indexDist / otherAvg
5. Decision:
   - `outlierRatio > 1.8` → return `"one"` (index clearly extended)
   - `outlierRatio < 1.3` → return `"fist"` (all fingertips similarly clustered)
   - Between 1.3 and 1.8 → return `"ambiguous"` (defer to original detection)

Thresholds 1.8 and 1.3 are initial values subject to tuning.

### Integration into `recognizeGesture()`

The disambiguation function is called at two points:

1. **After `isFist()` returns true:** Run `disambiguateFistVsOne()`. If result is `"one"`, override to gesture 1. If `"ambiguous"` or `"fist"`, keep gesture 0.

2. **After fallback logic determines gesture 1:** Run `disambiguateFistVsOne()`. If result is `"fist"`, override to gesture 0. If `"ambiguous"` or `"one"`, keep gesture 1.

### What is NOT changed

- `isFist()` function internals
- FingerPose classifier gesture definitions
- Stabilization window (10 frames, 7-frame threshold)
- Detection of other gestures (2, 3, 4, 5, 6)
- MediaPipe hand detection configuration

## Files to Modify

- `index.html`: Add `disambiguateFistVsOne()` function near `isFist()`, modify `recognizeGesture()` at the two integration points described above.
