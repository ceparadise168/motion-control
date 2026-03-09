# AR Gesture Collection Game - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single-page AR game where users collect 3 randomly assigned hand gestures (0-5 fingers) mapped to themes, with charge-up mechanics and celebratory effects.

**Architecture:** Single HTML file with inline CSS/JS. MediaPipe Hands for finger counting, Three.js for particle effects on success moments, Canvas API for screenshot compositing. Game state machine: START -> PLAYING -> SUCCESS.

**Tech Stack:** MediaPipe Hands 0.4, Three.js r128, vanilla JS, Canvas API

---

## Task 1: HTML Structure + CSS Foundation

Create `index.html` with all UI layers: start screen, camera video, Three.js container, game HUD (task panel, progress ring, hint text), success screen, speed lines overlay, and screen flash element. All CSS for responsive layout (mobile + desktop), card states (default/active/collected), progress ring, success animations.

## Task 2: Game State Machine + Gesture Assignment

Implement game config (gesture icons, theme labels, charge duration), game state object, random gesture picker (3 unique from 0-5), task panel rendering, charge logic (start/update/reset based on detected fingers), progress ring update, gesture collection with flash effect, success screen with auto-screenshot via Canvas API (mirrored video frame + overlay + title + gesture cards), and start button handler.

## Task 3: MediaPipe Hands + Finger Counting

Implement camera init with MediaPipe Hands (single hand, high confidence), hand results handler that calls updateCharge, and finger counting algorithm (thumb via x-axis comparison, other fingers via tip.y < PIP.y).

## Task 4: Three.js Particle Effects

Implement Three.js scene setup, particle burst system (create/update/cleanup), glow sprite generation, burst triggers on gesture collection (green + gold), and continuous celebration bursts on success screen.

## Task 5: Polish and Final Integration

Add smooth game loop for progress bar updates, replay button on success screen, responsive layout verification, and full end-to-end testing.
