# AR Hand Gesture Collection Game - Design Doc

## Overview
An interactive AR game for all-hands meetings. Users scan a QR code to open a web page, which activates the camera and presents 3 randomly assigned hand gestures to collect. Each gesture maps to a theme: Self-reflection, Awakening, Action. Collecting all 3 triggers a success screen with auto-screenshot.

## Tech Stack
- Single HTML file (easy deployment)
- MediaPipe Hands - hand tracking + finger counting (0-5)
- Three.js - particle burst effects on success
- Canvas API - screenshot compositing
- Responsive design - mobile portrait + desktop landscape

## Game Flow
1. Start screen with button
2. Camera permission request
3. Display task panel: 3 random gestures (free order)
   - Example: Fist(0) = Self-reflection, V(2) = Awakening, Open(5) = Action
4. User performs gesture -> detected -> charge progress bar (2.5s)
   - Gesture changes mid-charge -> progress resets
   - Charge complete -> particle burst + mark as collected
5. All 3 collected -> success screen
6. Auto-download screenshot + screen stays for manual screenshot

## Gesture Recognition
- MediaPipe Hands 21 landmarks per hand
- Each finger: tip y < base y = extended (thumb uses x-axis)
- Count extended fingers = gesture number (0-5)
- 6 possible gestures, game randomly picks 3

## UI Layout

### During Game
- Full-screen mirrored camera background
- Top: task panel (3 gesture cards - gray/glowing/lit)
- Center: detected gesture number + circular progress bar
- Bottom: hint text

### Success Screen
- Camera background with dark overlay
- Center: large "Mission Complete!" title
- Three gesture icons with labels (Self-reflection / Awakening / Action)
- Reserved logo area
- Continuous particle effects

## Visual Style (Hybrid)
- Game UI: clean modern - dark semi-transparent panels, rounded cards, smooth transitions
- Charging: circular progress bar + soft glow around hand
- Single collect: particle burst (v1 style) + screen flash + card light-up animation
- All complete: full-screen particle celebration + speed lines + shake effect

## Gesture-Theme Mapping
Randomly assigned each session from pool of 6 gestures (0-5):
1. Self-reflection (zi-xing)
2. Awakening (jue-xing)
3. Action imperative (shi-zai-bi-xing)

## Device Support
- Mobile (portrait) and Desktop (landscape) equally supported
- May be projected on large screen at venue
