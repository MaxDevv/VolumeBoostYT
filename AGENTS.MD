# GranularVolumeYT - Technical Documentation for Agents

This project is an iOS jailbreak tweak (originally VolumeBoostYT) modified to provide granular volume control for the YouTube app. It allows users to adjust the application's internal volume level from 0% to 200% with high precision.

## Architecture Overview

The tweak operates by intercepting low-level audio rendering calls and high-level UI events. It consists of four primary components:
1.  **Audio Hooking Layer:** Intercepts `AVPlayer`, `AVAudioPlayer`, and related classes to apply a custom volume multiplier.
2.  **Gesture Recognition Layer:** Hooks `UIWindow`'s `sendEvent:` to detect a custom pan gesture from the right edge of the screen.
3.  **UI/HUD Layer:** A custom overlay (`YTVolumeHUD`) that provides visual feedback to the user during volume adjustment.
4.  **Settings Integration:** Hooks YouTube's internal settings classes to provide a toggle for the tweak under "Granular Volume".

## File Breakdown

### `Tweak.x`
The core logic of the tweak, written in Logos.
-   **Audio Hooks:** Hooks `AVPlayer`, `AVAudioPlayerNode`, `AVAudioPlayer`, and `AVSampleBufferAudioRenderer`. It intercepts `setVolume:` and multiplies the input volume by a custom scalar.
-   **Linear Scaling:** Uses `GetLogarithmicAudioMultiplier()` (now returning a linear value) to map the UI multiplier (0.0 to 2.0) directly to the audio output for predictable granular control.
-   **Gesture Tracking:** Hooks `UIWindow sendEvent:` to implement a custom touch-tracking state machine. It looks for touches within 25 points of the right edge and transitions into a "Volume Gesture" if the user swipes left.
-   **Granular Sensitivity:** The vertical swipe sensitivity is set to `300.0f` points per 1.0 multiplier, allowing for very fine adjustments across the 600-point total range (0% to 200%).
-   **YouTube Settings:** Injects a "Granular Volume" section into the native YouTube settings menu.

### `YTVolumeHUD.h` & `YTVolumeHUD.m`
A custom `UIView` subclass that serves as the volume indicator.
-   **Singleton Pattern:** Uses `sharedHUD` for consistent access.
-   **Visuals:** Shows "App Vol: XX%" where XX ranges from 0 to 200.
-   **Animation:** Fades in when updated and fades out after 1.5 seconds.

### `Makefile`
The build configuration for Theos.
-   **Targets:** `iphoneos-arm64` and `iphoneos-arm64e`.
-   **Frameworks:** Links against `UIKit`, `AVFoundation`, and `AudioToolbox`.

### `VolumeBoostYT.plist`
Filter file targeted at `com.google.ios.youtube`.

## Technical Details

### Volume Calculation
The tweak uses a simple linear multiplier:
-   **Range:** 0.0 (0%) to 2.0 (200%).
-   **Default:** 1.0 (100%).

### Touch Handling
-   **Edge Trigger:** 25 points from the right screen edge.
-   **Activation:** Horizontal swipe left (>15pts) locks the gesture.
-   **Adjustment:** Vertical swipe adjusts volume. Divisor `300.0f` provides high granularity.

## Future Development Notes
-   **Persistence:** `ENABLE_VOLUME_PERSISTENCE` can be set to `1` in `Tweak.x` to remember the custom volume level across app restarts.
-   **Renaming:** While the file names still reference `VolumeBoostYT`, the UI and core logic have been pivoted to "Granular Volume".
