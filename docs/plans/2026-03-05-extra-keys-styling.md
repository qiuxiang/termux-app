# Extra Keys Styling Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Make the Termux extra keys (keyboard toolbar) background and text colors adapt to the terminal's styling (colors.properties).

**Architecture:** 
1. Modify `TermuxTerminalSessionActivityClient` to pass styling colors to `ExtraKeysView` when refreshing fonts/colors.
2. Update `ExtraKeysView` to handle dynamic color updates and ensure these colors are applied to all buttons during the `reload()` process.

**Tech Stack:** Android (Java), Termux Shared Library.

---

### Task 1: Update TermuxTerminalSessionActivityClient to propagate colors

**Files:**
- Modify: `/Users/7c00/Downloads/termux-app/app/src/main/java/com/termux/app/terminal/TermuxTerminalSessionActivityClient.java`

**Step 1: Modify updateBackgroundColor to apply colors to ExtraKeysView**

Update the `updateBackgroundColor()` method to extract foreground and background colors and pass them to the activity's extra keys view.

```java
    public void updateBackgroundColor() {
        if (!mActivity.isVisible()) return;
        TerminalSession session = mActivity.getCurrentSession();
        if (session != null && session.getEmulator() != null) {
            int bgColor = session.getEmulator().mColors.mCurrentColors[TextStyle.COLOR_INDEX_BACKGROUND];
            int fgColor = session.getEmulator().mColors.mCurrentColors[TextStyle.COLOR_INDEX_FOREGROUND];
            mActivity.getWindow().getDecorView().setBackgroundColor(bgColor);

            ExtraKeysView extraKeysView = mActivity.getExtraKeysView();
            if (extraKeysView != null) {
                // Use a slightly different color for active/pressed state
                // For now, we use the foreground color as the active text color 
                // and a standard gray for active background as a safe default
                int activeBgColor = 0xFF7F7F7F; 
                extraKeysView.setButtonColors(fgColor, fgColor, bgColor, activeBgColor);
            }
        }
    }
```

**Step 2: Ensure reload is called in checkForFontAndColors**

Verify `checkForFontAndColors` calls `updateBackgroundColor()`.

**Step 3: Commit**

```bash
git add app/src/main/java/com/termux/app/terminal/TermuxTerminalSessionActivityClient.java
git commit -m "feat: propagate styling colors to extra keys view"
```

---

### Task 2: Update TermuxActivity to refresh ExtraKeysView colors

**Files:**
- Modify: `/Users/7c00/Downloads/termux-app/app/src/main/java/com/termux/app/TermuxActivity.java`

**Step 1: Update the reload logic in the broadcast receiver**

Ensure that when styling is reloaded via broadcast, the extra keys view is properly updated with the current properties if it exists.

**Step 2: Commit**

```bash
git add app/src/main/java/com/termux/app/TermuxActivity.java
git commit -m "feat: refresh extra keys on style reload broadcast"
```

---

### Task 3: Ensure ExtraKeysView applies colors to buttons

**Files:**
- Modify: `/Users/7c00/Downloads/termux-app/termux-shared/src/main/java/com/termux/shared/termux/extrakeys/ExtraKeysView.java`

**Step 1: Verify reload() uses mButtonBackgroundColor and mButtonTextColor**

Ensure that the button inflation logic in `ExtraKeysView.java` (likely in `reload` or a helper method that creates `MaterialButton` instances) uses the member variables `mButtonTextColor` and `mButtonBackgroundColor`.

**Step 2: Commit**

```bash
git add termux-shared/src/main/java/com/termux/shared/termux/extrakeys/ExtraKeysView.java
git commit -m "fix: ensure ExtraKeysView applies colors during button creation"
```
