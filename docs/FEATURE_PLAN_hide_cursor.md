# Feature Implementation Plan: Hide Cursor Icon with '0' Key

## Overview
Add functionality to hide/show the tool cursor icon (CuteCursor component) by pressing the '0' key, allowing users to take clean screenshots without the tool indicator visible.

---

## Current State Analysis

### Key Components

1. **CuteCursor Component** ([src/renderer/app_page/components/components/CuteCursor.js](src/renderer/app_page/components/components/CuteCursor.js))
   - Displays the current tool icon near the mouse pointer
   - Receives props: `mouseCoordinates`, `activeColorIndex`, `activeWidthIndex`, `activeTool`, `Icons`
   - Currently ALWAYS rendered (lines 1009-1015 in Application.js)

2. **Application Component** ([src/renderer/app_page/components/Application.js](src/renderer/app_page/components/Application.js))
   - Main component managing all app state
   - Contains `handleKeyDown` function (line 118) for keyboard shortcuts
   - Keys 1-8 already mapped to tools/colors/widths (lines 290-327)
   - Key '0' is currently unused ✅

3. **Keyboard Shortcuts**
   - **1**: Pen tool
   - **2**: Cycle shapes (arrow/rectangle/oval/line)
   - **3**: Text tool
   - **4**: Highlighter
   - **5**: Laser
   - **6**: Eraser
   - **7**: Cycle colors
   - **8**: Cycle widths
   - **0**: 🆕 **Available for cursor icon toggle**

---

## Implementation Steps

### Step 1: Add State Variable for Cursor Visibility

**File:** `src/renderer/app_page/components/Application.js`

**Location:** Around line 107 (after existing state declarations)

**Action:** Add new state variable to track cursor icon visibility

```javascript
const [isShiftPressed, setIsShiftPressed] = useState(false);
const [showCursorIcon, setShowCursorIcon] = useState(true); // 🆕 Add this line
```

**Reasoning:** 
- Follows existing pattern of boolean state variables
- Default `true` maintains current behavior (icon visible by default)
- Named clearly to indicate purpose

---

### Step 2: Add Keyboard Handler for '0' Key

**File:** `src/renderer/app_page/components/Application.js`

**Location:** Inside `handleKeyDown` function, after case '8' (around line 327)

**Action:** Add case handler for '0' key to toggle cursor visibility

```javascript
case '8':
  handleChangeWidth((activeWidthIndex + 1) % widthList.length);
  break;
case '0': // 🆕 Add this case
  setShowCursorIcon(prev => !prev);
  break;
```

**Reasoning:**
- Consistent with existing switch-case pattern
- Uses functional state update `prev => !prev` for reliable toggling
- Simple toggle behavior: press '0' to hide, press again to show
- No conflicts with other key bindings

---

### Step 3: Conditionally Render CuteCursor Component

**File:** `src/renderer/app_page/components/Application.js`

**Location:** Lines 1009-1015 (CuteCursor rendering section)

**Action:** Wrap CuteCursor in conditional rendering

**Current Code:**
```javascript
<CuteCursor
  mouseCoordinates={mouseCoordinates}
  activeColorIndex={activeColorIndex}
  activeWidthIndex={activeWidthIndex}
  activeTool={activeTool}
  Icons={Icons}
/>
```

**Updated Code:**
```javascript
{
  showCursorIcon && (
    <CuteCursor
      mouseCoordinates={mouseCoordinates}
      activeColorIndex={activeColorIndex}
      activeWidthIndex={activeWidthIndex}
      activeTool={activeTool}
      Icons={Icons}
    />
  )
}
```

**Reasoning:**
- Standard React conditional rendering pattern
- Uses `&&` operator for clean syntax
- Component fully unmounts when hidden (no memory overhead)
- Remounts with correct state when shown again

---

### Step 4: (Optional) Add Visual Feedback

**File:** `src/renderer/app_page/components/Application.js`

**Location:** After toggling `showCursorIcon` in case '0'

**Action:** Optionally add a Toast notification to confirm action

```javascript
case '0':
  setShowCursorIcon(prev => {
    const newValue = !prev;
    // Optional: Show toast notification
    // setRippleEffects(prev => [...prev, { 
    //   id: Date.now(), 
    //   text: newValue ? 'Cursor icon shown' : 'Cursor icon hidden' 
    // }]);
    return newValue;
  });
  break;
```

**Reasoning:**
- Provides user feedback about state change
- Follows existing pattern (rippleEffects used for notifications)
- Optional but enhances UX
- Can be omitted for minimal implementation

---

### Step 5: (Optional) Persist Setting

**File:** `src/main/index.js`

**Location:** electron-store schema definition

**Action:** Add `show_cursor_icon` to settings schema and save state

**In main process (index.js):**
```javascript
schema: {
  // ... existing settings
  show_cursor_icon: {
    type: 'boolean',
    default: true
  }
}
```

**In renderer (Application.js):**
```javascript
// Load initial value from settings
const initialShowCursorIcon = show_cursor_icon ?? true;

// In state declaration:
const [showCursorIcon, setShowCursorIcon] = useState(initialShowCursorIcon);

// Add useEffect to persist changes:
useEffect(() => {
  window.electronAPI.saveSetting('show_cursor_icon', showCursorIcon);
}, [showCursorIcon]);
```

**Reasoning:**
- Preserves user preference across app restarts
- Follows existing pattern for persistent settings
- Optional: Can be added later if user requests persistence
- Requires IPC implementation in preload.js

---

## Testing Plan

### Manual Testing Steps

1. **Basic Toggle Functionality**
   - Launch DrawPen application
   - Verify cursor icon is visible by default
   - Press '0' key
   - Verify cursor icon disappears
   - Press '0' again
   - Verify cursor icon reappears

2. **Interaction with Other Tools**
   - Hide cursor icon (press '0')
   - Switch tools using keys 1-6
   - Verify icon remains hidden
   - Show cursor icon (press '0')
   - Switch tools again
   - Verify icon updates correctly for each tool

3. **Drawing Behavior**
   - Hide cursor icon
   - Draw with pen tool (key '1')
   - Verify drawing still works normally
   - Verify no icon appears during drawing

4. **Screenshot Scenario**
   - Draw some annotations on screen
   - Hide cursor icon with '0'
   - Take screenshot (external tool)
   - Verify no tool icon in screenshot
   - Show cursor icon with '0'
   - Continue drawing

5. **Edge Cases**
   - Press '0' multiple times rapidly (verify no issues)
   - Hide icon, restart app, verify state (if persistence added)
   - Test with text editor active (should ignore '0' key)

### Code Review Checklist

- [ ] State variable added correctly
- [ ] Keyboard handler follows existing pattern
- [ ] Conditional rendering implemented properly
- [ ] No performance regressions (icon unmounts cleanly)
- [ ] Compatible with existing keyboard shortcuts
- [ ] Text editor doesn't interfere with toggle
- [ ] Code style consistent with project conventions

---

## Files to Modify

| File | Changes | Lines to Edit |
|------|---------|---------------|
| [src/renderer/app_page/components/Application.js](src/renderer/app_page/components/Application.js) | Add state variable | ~107 |
| [src/renderer/app_page/components/Application.js](src/renderer/app_page/components/Application.js) | Add case '0' handler | ~327 |
| [src/renderer/app_page/components/Application.js](src/renderer/app_page/components/Application.js) | Wrap CuteCursor conditionally | ~1009-1015 |
| [src/main/index.js](src/main/index.js) | (Optional) Add setting to schema | Schema section |

---

## Implementation Notes

### Code Style Guidelines
- Follow functional component patterns (hooks)
- Use `useState` for component state
- Use camelCase for state variables
- Follow existing keyboard handler structure
- Maintain consistent indentation (2 spaces)

### Compatibility Considerations
- Key '0' is not used by any other feature ✅
- CuteCursor has no dependencies on always being rendered ✅
- Toggle works independently of other states ✅
- No IPC changes required for basic implementation ✅

### Future Enhancements
- Add global keyboard shortcut (in main process)
- Add toolbar button to toggle cursor visibility
- Add setting in settings page UI
- Add keyboard shortcut to README documentation
- Add tooltip/hint on first use

---

## Minimal Implementation (Fastest Path)

For quickest implementation, only complete **Steps 1, 2, and 3**:
1. Add `showCursorIcon` state variable
2. Add `case '0':` keyboard handler
3. Wrap `<CuteCursor>` in conditional rendering

**Estimated time:** 5 minutes  
**Lines of code:** ~5 new lines

This provides full functionality without persistence or visual feedback.

---

## Success Criteria

✅ Pressing '0' hides the cursor icon  
✅ Pressing '0' again shows the cursor icon  
✅ Icon remains hidden/shown across tool switches  
✅ Drawing functionality unaffected  
✅ No console errors  
✅ Clean screenshots possible with hidden icon  

---

## Related Documentation

- [Application.js](src/renderer/app_page/components/Application.js) - Main component with keyboard handlers
- [CuteCursor.js](src/renderer/app_page/components/components/CuteCursor.js) - Cursor icon component
- [.github/copilot-instructions.md](.github/copilot-instructions.md) - Project coding guidelines
- [README.md](README.md) - User-facing keybindings documentation

---

## Implementation Checklist

- [ ] Add `showCursorIcon` state variable (Step 1)
- [ ] Add keyboard handler for '0' key (Step 2)
- [ ] Add conditional rendering of CuteCursor (Step 3)
- [ ] Test basic toggle functionality
- [ ] Test with different tools (1-6)
- [ ] Test screenshot scenario
- [ ] (Optional) Add visual feedback toast
- [ ] (Optional) Implement persistence
- [ ] (Optional) Update README with new keybinding
- [ ] Commit changes to `hide-cursor` branch
- [ ] Test compiled binary with `npm run package_no_sign`

---

**Branch:** `hide-cursor` (already checked out from tag v0.0.33)  
**Ready for implementation by next LLM agent** ✅
