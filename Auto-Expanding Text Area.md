# Auto-Expanding Text Area Implementation Guide

## Overview

An auto-expanding text area is a user interface component that automatically adjusts its height based on the content being typed. This provides a better user experience by eliminating the need for manual resizing while maintaining visual consistency and preventing excessive space usage.

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Basic Implementation](#basic-implementation)
3. [Advanced Features](#advanced-features)
4. [CSS Styling](#css-styling)
5. [JavaScript Logic](#javascript-logic)
6. [Best Practices](#best-practices)
7. [Accessibility Considerations](#accessibility-considerations)
8. [Browser Compatibility](#browser-compatibility)
9. [Common Issues & Solutions](#common-issues--solutions)
10. [Integration Examples](#integration-examples)

## Core Concepts

### How It Works

The auto-expanding textarea works by:

1. **Dynamic Height Calculation**: Measuring the scroll height of the content
2. **Height Constraints**: Setting minimum and maximum height boundaries
3. **Overflow Management**: Controlling when scrollbars appear
4. **Event-Driven Updates**: Responding to user input in real-time

### Key Properties

- `scrollHeight`: The total height of the content including hidden overflow
- `clientHeight`: The visible height of the element
- `offsetHeight`: The total height including borders and padding

## Basic Implementation

### HTML Structure

```html
<div class="textarea-wrapper">
    <textarea 
        id="autoExpandingTextarea"
        class="auto-expanding-textarea"
        placeholder="Type your message..."
        rows="1"
        aria-label="Message input"
    ></textarea>
</div>
```

### Essential CSS

```css
.auto-expanding-textarea {
    width: 100%;
    min-height: 44px;        /* Minimum height */
    max-height: 320px;       /* Maximum height (prevents infinite growth) */
    padding: 12px 16px;
    border: 1px solid #d0d0d0;
    border-radius: 8px;
    font-family: inherit;
    font-size: 14px;
    resize: none;            /* Disable manual resize */
    overflow-y: hidden;      /* Hide scrollbar initially */
    transition: height 0.15s ease-out; /* Smooth height transitions */
    box-sizing: border-box;
    word-wrap: break-word;
    white-space: pre-wrap;   /* Preserve whitespace and line breaks */
}

.auto-expanding-textarea:focus {
    outline: none;
    border-color: #4a90e2;
    box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.1);
}
```

### Core JavaScript Function

```javascript
function adjustTextAreaHeight(textarea) {
    const minHeight = 44;  // Minimum height in pixels
    const maxHeight = 320; // Maximum height in pixels
    
    // Store current scroll position to prevent jumping
    const scrollPos = textarea.scrollTop;
    
    // Reset height completely for accurate measurement
    textarea.style.height = 'auto';
    textarea.style.height = '1px';
    
    // Get the actual content height needed
    const scrollHeight = textarea.scrollHeight;
    
    // Calculate new height based on content, with proper bounds
    let newHeight = Math.max(scrollHeight, minHeight);
    newHeight = Math.min(newHeight, maxHeight);
    
    // Apply the calculated height
    textarea.style.height = newHeight + 'px';
    
    // Restore scroll position if needed (only when at max height)
    if (scrollPos > 0 && newHeight === maxHeight) {
        textarea.scrollTop = scrollPos;
    }
    
    // Show scrollbar only when content exceeds max height
    textarea.style.overflowY = scrollHeight > maxHeight ? 'auto' : 'hidden';
}

// Enhanced resize handler with empty content detection
function handleTextAreaResize(textarea) {
    const minHeight = 44;
    
    // Check if textarea is empty and force reset to minimum height
    if (textarea.value.trim().length === 0) {
        // Same reset logic as when sending a message
        textarea.style.height = minHeight + 'px';
        textarea.style.overflowY = 'hidden';
    } else {
        // Use requestAnimationFrame to ensure DOM updates are complete
        requestAnimationFrame(() => {
            adjustTextAreaHeight(textarea);
        });
    }
}
```

## Advanced Features

### 1. Enhanced Height Calculation

```javascript
function adjustTextAreaHeight(textarea, options = {}) {
    const {
        minHeight = 44,
        maxHeight = 320,
        lineHeight = null,
        smoothTransition = true
    } = options;
    
    // Calculate line height if not provided
    const computedLineHeight = lineHeight || 
        parseInt(window.getComputedStyle(textarea).lineHeight) || 20;
    
    // Store current state
    const currentScrollTop = textarea.scrollTop;
    const currentHeight = textarea.offsetHeight;
    
    // Reset height for accurate measurement
    textarea.style.height = 'auto';
    
    // Get the natural height needed for content
    const scrollHeight = textarea.scrollHeight;
    
    // Calculate new height with constraints
    let newHeight = Math.max(minHeight, scrollHeight);
    newHeight = Math.min(newHeight, maxHeight);
    
    // Snap to line boundaries for cleaner appearance
    if (lineHeight) {
        const lines = Math.round((newHeight - 24) / computedLineHeight); // 24px for padding
        newHeight = (lines * computedLineHeight) + 24;
        newHeight = Math.max(minHeight, Math.min(newHeight, maxHeight));
    }
    
    // Apply transition only when height actually changes
    if (smoothTransition && Math.abs(newHeight - currentHeight) > 1) {
        textarea.style.transition = 'height 0.15s ease-out';
    } else {
        textarea.style.transition = 'none';
    }
    
    // Set the new height
    textarea.style.height = newHeight + 'px';
    
    // Manage overflow
    textarea.style.overflowY = scrollHeight > maxHeight ? 'auto' : 'hidden';
    
    // Restore scroll position
    textarea.scrollTop = currentScrollTop;
    
    return {
        oldHeight: currentHeight,
        newHeight: newHeight,
        hasScrollbar: scrollHeight > maxHeight
    };
}
```

### 2. Debounced Input Handling

```javascript
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

// Usage
const debouncedAdjustHeight = debounce((textarea) => {
    adjustTextAreaHeight(textarea);
}, 10);

textarea.addEventListener('input', () => {
    debouncedAdjustHeight(textarea);
});
```

### 3. Reset Functionality

```javascript
function resetTextAreaHeight(textarea) {
    const minHeight = parseInt(textarea.dataset.minHeight) || 44;
    
    // Clear content
    textarea.value = '';
    
    // Reset to minimum height
    textarea.style.height = minHeight + 'px';
    textarea.style.overflowY = 'hidden';
    
    // Remove any transition for immediate reset
    textarea.style.transition = 'none';
    
    // Re-enable transitions after a brief delay
    setTimeout(() => {
        textarea.style.transition = 'height 0.15s ease-out';
    }, 10);
}

// Enhanced event handling with comprehensive input detection
function setupEventListeners(textarea) {
    // Main resize handler with empty content detection
    function handleResize() {
        if (textarea.value.trim().length === 0) {
            // Force reset to minimum height when empty
            textarea.style.height = '44px';
            textarea.style.overflowY = 'hidden';
        } else {
            // Use requestAnimationFrame for smooth updates
            requestAnimationFrame(() => {
                adjustTextAreaHeight(textarea);
            });
        }
    }
    
    // Input events for typing
    textarea.addEventListener('input', handleResize);
    
    // Handle cut and paste operations
    textarea.addEventListener('cut', () => {
        setTimeout(handleResize, 5);
    });
    
    textarea.addEventListener('paste', () => {
        setTimeout(handleResize, 5);
    });
    
    // Handle delete operations specifically
    textarea.addEventListener('keyup', (event) => {
        if (event.key === 'Backspace' || event.key === 'Delete') {
            handleResize();
        }
    });
    
    // Handle Enter key for new lines
    textarea.addEventListener('keydown', (event) => {
        if (event.key === 'Enter') {
            setTimeout(handleResize, 5);
        }
    });
}
```

## CSS Styling

### Responsive Design

```css
/* Base styles */
.auto-expanding-textarea {
    width: 100%;
    min-height: var(--textarea-min-height, 44px);
    max-height: var(--textarea-max-height, 320px);
    padding: 12px 16px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    font-family: inherit;
    font-size: 14px;
    line-height: 1.4;
    resize: none;
    overflow-y: hidden;
    transition: height 0.15s ease-out, border-color 0.2s ease;
    box-sizing: border-box;
    word-wrap: break-word;
    white-space: pre-wrap;
}

/* Mobile optimizations */
@media (max-width: 768px) {
    .auto-expanding-textarea {
        font-size: 16px; /* Prevents zoom on iOS */
        --textarea-max-height: 200px; /* Smaller max height on mobile */
    }
}

/* Tablet optimizations */
@media (min-width: 769px) and (max-width: 1024px) {
    .auto-expanding-textarea {
        --textarea-max-height: 280px;
    }
}

/* Custom scrollbar styling */
.auto-expanding-textarea::-webkit-scrollbar {
    width: 6px;
}

.auto-expanding-textarea::-webkit-scrollbar-track {
    background: transparent;
}

.auto-expanding-textarea::-webkit-scrollbar-thumb {
    background-color: rgba(0, 0, 0, 0.2);
    border-radius: 3px;
}

.auto-expanding-textarea::-webkit-scrollbar-thumb:hover {
    background-color: rgba(0, 0, 0, 0.4);
}
```

### Dark Mode Support

```css
@media (prefers-color-scheme: dark) {
    .auto-expanding-textarea {
        background-color: #2a2a2a;
        color: #ffffff;
        border-color: #404040;
    }
    
    .auto-expanding-textarea::placeholder {
        color: #999999;
    }
    
    .auto-expanding-textarea:focus {
        border-color: #6ab7ff;
        box-shadow: 0 0 0 2px rgba(106, 183, 255, 0.2);
    }
}
```

## JavaScript Logic

### Complete Implementation

```javascript
class AutoExpandingTextArea {
    constructor(element, options = {}) {
        this.textarea = element;
        this.options = {
            minHeight: 44,
            maxHeight: 320,
            smoothTransition: true,
            debounceDelay: 10,
            ...options
        };
        
        this.init();
    }
    
    init() {
        // Set initial properties
        this.textarea.style.resize = 'none';
        this.textarea.style.overflow = 'hidden';
        this.textarea.style.height = this.options.minHeight + 'px';
        
        // Bind event listeners
        this.bindEvents();
        
        // Initial adjustment
        this.adjust();
    }
    
    bindEvents() {
        // Create debounced adjust function
        this.debouncedAdjust = this.debounce(
            () => this.adjust(),
            this.options.debounceDelay
        );
        
        // Input events for all text changes
        this.textarea.addEventListener('input', this.debouncedAdjust);
        
        // Handle paste operations
        this.textarea.addEventListener('paste', () => {
            // Slight delay to let paste content render
            setTimeout(() => this.adjust(), 5);
        });
        
        // Handle cut operations
        this.textarea.addEventListener('cut', () => {
            // Slight delay to let cut operation complete
            setTimeout(() => this.adjust(), 5);
        });
        
        // Handle delete/backspace operations
        this.textarea.addEventListener('keyup', (event) => {
            if (event.key === 'Backspace' || event.key === 'Delete') {
                this.adjust();
            }
        });
        
        // Keyboard events
        this.textarea.addEventListener('keydown', (event) => {
            // Handle Enter key
            if (event.key === 'Enter') {
                // Delay adjustment to account for new line
                setTimeout(() => this.adjust(), 5);
            }
        });
        
        // Focus events for mobile optimization
        this.textarea.addEventListener('focus', () => {
            // Ensure proper sizing on mobile focus
            if (window.innerWidth <= 768) {
                setTimeout(() => this.adjust(), 100);
            }
        });
    }
    
    adjust() {
        const { minHeight, maxHeight, smoothTransition } = this.options;
        
        // Check if textarea is empty and force reset
        if (this.textarea.value.trim().length === 0) {
            this.textarea.style.height = minHeight + 'px';
            this.textarea.style.overflowY = 'hidden';
            this.textarea.style.transition = 'none';
            
            // Re-enable transitions after brief delay
            setTimeout(() => {
                this.textarea.style.transition = 'height 0.15s ease-out';
            }, 10);
            
            return;
        }
        
        // Store current state
        const scrollPos = this.textarea.scrollTop;
        const currentHeight = this.textarea.offsetHeight;
        
        // Reset height for measurement
        this.textarea.style.height = 'auto';
        this.textarea.style.height = '1px';
        
        // Calculate required height
        const scrollHeight = this.textarea.scrollHeight;
        let newHeight = Math.max(minHeight, scrollHeight);
        newHeight = Math.min(newHeight, maxHeight);
        
        // Apply height with optional transition
        if (smoothTransition && Math.abs(newHeight - currentHeight) > 2) {
            this.textarea.style.transition = 'height 0.15s ease-out';
        } else {
            this.textarea.style.transition = 'none';
        }
        
        this.textarea.style.height = newHeight + 'px';
        
        // Manage overflow
        this.textarea.style.overflowY = scrollHeight > maxHeight ? 'auto' : 'hidden';
        
        // Restore scroll position (only when at max height)
        if (scrollPos > 0 && newHeight === maxHeight) {
            this.textarea.scrollTop = scrollPos;
        }
        
        // Emit custom event
        this.textarea.dispatchEvent(new CustomEvent('heightChanged', {
            detail: { oldHeight: currentHeight, newHeight: newHeight }
        }));
    }
    
    reset() {
        this.textarea.value = '';
        this.textarea.style.height = this.options.minHeight + 'px';
        this.textarea.style.overflowY = 'hidden';
        this.textarea.style.transition = 'none';
        
        // Re-enable transitions
        setTimeout(() => {
            this.textarea.style.transition = 'height 0.15s ease-out';
        }, 10);
    }
    
    debounce(func, wait) {
        let timeout;
        return function executedFunction(...args) {
            const later = () => {
                clearTimeout(timeout);
                func.apply(this, args);
            };
            clearTimeout(timeout);
            timeout = setTimeout(later, wait);
        };
    }
    
    destroy() {
        this.textarea.removeEventListener('input', this.debouncedAdjust);
        this.textarea.style.height = '';
        this.textarea.style.resize = '';
        this.textarea.style.overflow = '';
        this.textarea.style.transition = '';
    }
}

// Usage
const textarea = document.getElementById('myTextarea');
const autoExpander = new AutoExpandingTextArea(textarea, {
    minHeight: 44,
    maxHeight: 320,
    smoothTransition: true
});

// Listen for height changes
textarea.addEventListener('heightChanged', (event) => {
    console.log('Height changed:', event.detail);
});
```

## Best Practices

### 1. Performance Optimization

- **Debounce Input Events**: Prevent excessive height calculations during rapid typing
- **Minimal DOM Manipulations**: Cache values when possible
- **Efficient Measurements**: Use `scrollHeight` over complex calculations

### 2. User Experience

- **Smooth Transitions**: Use CSS transitions for height changes
- **Appropriate Constraints**: Set reasonable min/max heights
- **Consistent Behavior**: Ensure predictable resizing across different content types

### 3. Code Organization

```javascript
// Configuration object
const TEXTAREA_CONFIG = {
    DEFAULT_MIN_HEIGHT: 44,
    DEFAULT_MAX_HEIGHT: 320,
    MOBILE_MAX_HEIGHT: 200,
    DEBOUNCE_DELAY: 10,
    TRANSITION_DURATION: 150
};

// Utility functions
const TextAreaUtils = {
    getComputedLineHeight(element) {
        const computed = window.getComputedStyle(element);
        return parseInt(computed.lineHeight) || 20;
    },
    
    isMobileDevice() {
        return window.innerWidth <= 768;
    },
    
    preventIOSZoom(textarea) {
        if (this.isMobileDevice()) {
            textarea.style.fontSize = '16px';
        }
    }
};
```

## Accessibility Considerations

### ARIA Labels and Roles

```html
<div class="textarea-container" role="textbox" aria-multiline="true">
    <label for="message-input" class="sr-only">Type your message</label>
    <textarea
        id="message-input"
        class="auto-expanding-textarea"
        aria-label="Message input"
        aria-describedby="char-count helper-text"
        aria-expanded="false"
        placeholder="Type your message..."
    ></textarea>
    <div id="char-count" aria-live="polite" class="sr-only">
        0 characters
    </div>
    <div id="helper-text" class="sr-only">
        Press Enter to send, Shift+Enter for new line
    </div>
</div>
```

### Screen Reader Support

```javascript
function updateAriaAttributes(textarea) {
    const lineCount = textarea.value.split('\n').length;
    const charCount = textarea.value.length;
    const isExpanded = textarea.offsetHeight > parseInt(textarea.dataset.minHeight || 44);
    
    // Update aria-expanded
    textarea.setAttribute('aria-expanded', isExpanded.toString());
    
    // Update character count for screen readers
    const charCountElement = document.getElementById('char-count');
    if (charCountElement) {
        charCountElement.textContent = `${charCount} characters, ${lineCount} lines`;
    }
}
```

### Keyboard Navigation

```javascript
function handleKeyboardNavigation(event) {
    const textarea = event.target;
    
    switch (event.key) {
        case 'Enter':
            if (!event.shiftKey) {
                // Handle send action
                event.preventDefault();
                handleSendMessage();
            }
            break;
            
        case 'Escape':
            // Clear textarea
            textarea.value = '';
            adjustTextAreaHeight(textarea);
            break;
            
        case 'Tab':
            // Allow natural tab navigation
            break;
    }
}
```

## Browser Compatibility

### Modern Browser Support

The auto-expanding textarea works in all modern browsers:

- Chrome 60+
- Firefox 60+
- Safari 12+
- Edge 79+

### Fallback for Older Browsers

```javascript
function createAutoExpandingTextArea(element, options) {
    // Feature detection
    const supportsScrollHeight = 'scrollHeight' in element;
    const supportsGetComputedStyle = typeof window.getComputedStyle === 'function';
    
    if (!supportsScrollHeight || !supportsGetComputedStyle) {
        // Fallback: Use fixed height with scrollbar
        element.style.height = options.maxHeight + 'px';
        element.style.overflowY = 'auto';
        return;
    }
    
    // Full implementation for modern browsers
    return new AutoExpandingTextArea(element, options);
}
```

### CSS Fallbacks

```css
.auto-expanding-textarea {
    height: 44px; /* Fallback height */
    min-height: 44px;
    max-height: 320px;
    overflow-y: auto; /* Fallback overflow */
}

/* Enhanced for modern browsers */
@supports (height: max-content) {
    .auto-expanding-textarea {
        overflow-y: hidden;
        resize: none;
    }
}
```

## Common Issues & Solutions

### Issue 1: Height Calculation Flickering

**Problem**: Textarea flickers during height adjustments

**Solution**: Use proper height reset sequence

```javascript
function adjustHeightWithoutFlicker(textarea) {
    // Store current scroll position
    const scrollPos = textarea.scrollTop;
    
    // Use requestAnimationFrame for smooth updates
    requestAnimationFrame(() => {
        textarea.style.height = '1px';
        
        requestAnimationFrame(() => {
            const newHeight = Math.min(textarea.scrollHeight, maxHeight);
            textarea.style.height = newHeight + 'px';
            textarea.scrollTop = scrollPos;
        });
    });
}
```

### Issue 2: iOS Safari Zoom on Focus

**Problem**: Mobile Safari zooms in when textarea is focused

**Solution**: Ensure font-size is at least 16px

```css
@media screen and (max-width: 767px) {
    .auto-expanding-textarea {
        font-size: 16px; /* Prevents iOS zoom */
    }
}
```

### Issue 3: Content Overflow with Long Words

**Problem**: Long words cause horizontal overflow

**Solution**: Proper word-wrapping CSS

```css
.auto-expanding-textarea {
    word-wrap: break-word;
    word-break: break-word;
    overflow-wrap: break-word;
    white-space: pre-wrap;
}
```

### Issue 5: Textarea Not Shrinking When Content is Deleted

**Problem**: Textarea stays expanded even when all content is removed

**Solution**: Add empty content detection with forced reset

```javascript
function handleTextAreaResize(textarea) {
    const minHeight = 44;
    
    // Special handling for empty content
    if (textarea.value.trim().length === 0) {
        // Force immediate reset to minimum height
        textarea.style.height = minHeight + 'px';
        textarea.style.overflowY = 'hidden';
        return;
    }
    
    // Normal height adjustment for non-empty content
    requestAnimationFrame(() => {
        textarea.style.height = 'auto';
        textarea.style.height = '1px';
        
        const scrollHeight = textarea.scrollHeight;
        const newHeight = Math.min(Math.max(scrollHeight, minHeight), maxHeight);
        
        textarea.style.height = newHeight + 'px';
        textarea.style.overflowY = scrollHeight > maxHeight ? 'auto' : 'hidden';
    });
}

// Comprehensive event handling
textarea.addEventListener('input', () => handleTextAreaResize(textarea));
textarea.addEventListener('keyup', (event) => {
    if (event.key === 'Backspace' || event.key === 'Delete') {
        handleTextAreaResize(textarea);
    }
});
textarea.addEventListener('cut', () => {
    setTimeout(() => handleTextAreaResize(textarea), 5);
});
```

### Issue 4: Incorrect Height with Different Fonts

**Problem**: Height calculation varies across different fonts

**Solution**: Font-aware height calculation

```javascript
function getFontAwareHeight(textarea) {
    const style = window.getComputedStyle(textarea);
    const lineHeight = parseInt(style.lineHeight) || 1.2 * parseInt(style.fontSize);
    const paddingTop = parseInt(style.paddingTop);
    const paddingBottom = parseInt(style.paddingBottom);
    
    return {
        lineHeight,
        verticalPadding: paddingTop + paddingBottom
    };
}
```

## Integration Examples

### React Implementation

```jsx
import { useEffect, useRef, useCallback } from 'react';

const AutoExpandingTextArea = ({
    value,
    onChange,
    placeholder,
    minHeight = 44,
    maxHeight = 320,
    className = '',
    ...props
}) => {
    const textareaRef = useRef(null);
    
    const adjustHeight = useCallback(() => {
        const textarea = textareaRef.current;
        if (!textarea) return;
        
        // Handle empty content
        if (textarea.value.trim().length === 0) {
            textarea.style.height = minHeight + 'px';
            textarea.style.overflowY = 'hidden';
            return;
        }
        
        textarea.style.height = 'auto';
        textarea.style.height = '1px';
        const newHeight = Math.min(
            Math.max(textarea.scrollHeight, minHeight),
            maxHeight
        );
        textarea.style.height = newHeight + 'px';
        textarea.style.overflowY = textarea.scrollHeight > maxHeight ? 'auto' : 'hidden';
    }, [minHeight, maxHeight]);
    
    useEffect(() => {
        adjustHeight();
    }, [value, adjustHeight]);
    
    const handleChange = (event) => {
        onChange?.(event);
        adjustHeight();
    };
    
    return (
        <textarea
            ref={textareaRef}
            value={value}
            onChange={handleChange}
            placeholder={placeholder}
            className={`auto-expanding-textarea ${className}`}
            style={{
                minHeight: `${minHeight}px`,
                maxHeight: `${maxHeight}px`,
                resize: 'none',
                overflowY: 'hidden',
                transition: 'height 0.15s ease-out'
            }}
            {...props}
        />
    );
};
```

### Vue Implementation

```vue
<template>
    <textarea
        ref="textarea"
        :value="modelValue"
        @input="handleInput"
        :placeholder="placeholder"
        class="auto-expanding-textarea"
        :style="textareaStyles"
    ></textarea>
</template>

<script setup>
import { ref, computed, nextTick, watch } from 'vue';

const props = defineProps({
    modelValue: String,
    placeholder: String,
    minHeight: { type: Number, default: 44 },
    maxHeight: { type: Number, default: 320 }
});

const emit = defineEmits(['update:modelValue']);

const textarea = ref(null);

const textareaStyles = computed(() => ({
    minHeight: `${props.minHeight}px`,
    maxHeight: `${props.maxHeight}px`,
    resize: 'none',
    overflowY: 'hidden',
    transition: 'height 0.15s ease-out'
}));

const adjustHeight = async () => {
    await nextTick();
    const el = textarea.value;
    if (!el) return;
    
    // Handle empty content
    if (el.value.trim().length === 0) {
        el.style.height = props.minHeight + 'px';
        el.style.overflowY = 'hidden';
        return;
    }
    
    el.style.height = 'auto';
    el.style.height = '1px';
    const newHeight = Math.min(
        Math.max(el.scrollHeight, props.minHeight),
        props.maxHeight
    );
    el.style.height = newHeight + 'px';
    el.style.overflowY = el.scrollHeight > props.maxHeight ? 'auto' : 'hidden';
};

const handleInput = (event) => {
    emit('update:modelValue', event.target.value);
    adjustHeight();
};

watch(() => props.modelValue, adjustHeight);
</script>
```

### Angular Implementation

```typescript
// auto-expanding-textarea.component.ts
import { Component, ElementRef, Input, forwardRef, AfterViewInit } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
    selector: 'app-auto-expanding-textarea',
    template: `
        <textarea
            #textarea
            [placeholder]="placeholder"
            [value]="value"
            (input)="onInput($event)"
            class="auto-expanding-textarea"
            [style.min-height.px]="minHeight"
            [style.max-height.px]="maxHeight"
        ></textarea>
    `,
    providers: [
        {
            provide: NG_VALUE_ACCESSOR,
            useExisting: forwardRef(() => AutoExpandingTextareaComponent),
            multi: true
        }
    ]
})
export class AutoExpandingTextareaComponent implements ControlValueAccessor, AfterViewInit {
    @Input() placeholder = '';
    @Input() minHeight = 44;
    @Input() maxHeight = 320;
    
    value = '';
    private onChange = (value: string) => {};
    private onTouched = () => {};
    
    constructor(private elementRef: ElementRef) {}
    
    ngAfterViewInit() {
        this.adjustHeight();
    }
    
    onInput(event: Event) {
        const target = event.target as HTMLTextAreaElement;
        this.value = target.value;
        this.onChange(this.value);
        this.adjustHeight();
    }
    
    private adjustHeight() {
        const textarea = this.elementRef.nativeElement.querySelector('textarea');
        if (!textarea) return;
        
        // Handle empty content
        if (textarea.value.trim().length === 0) {
            textarea.style.height = this.minHeight + 'px';
            textarea.style.overflowY = 'hidden';
            return;
        }
        
        textarea.style.height = 'auto';
        textarea.style.height = '1px';
        const newHeight = Math.min(
            Math.max(textarea.scrollHeight, this.minHeight),
            this.maxHeight
        );
        textarea.style.height = newHeight + 'px';
        textarea.style.overflowY = textarea.scrollHeight > this.maxHeight ? 'auto' : 'hidden';
    }
    
    writeValue(value: string) {
        this.value = value;
        setTimeout(() => this.adjustHeight(), 0);
    }
    
    registerOnChange(fn: (value: string) => void) {
        this.onChange = fn;
    }
    
    registerOnTouched(fn: () => void) {
        this.onTouched = fn;
    }
}
```

## Conclusion

The auto-expanding textarea is a powerful UI component that significantly improves user experience in messaging applications, comment forms, and any interface where users input variable amounts of text. 

Key takeaways:

1. **Performance**: Always debounce input events and minimize DOM manipulations
2. **Accessibility**: Include proper ARIA attributes and keyboard navigation
3. **Responsive Design**: Adapt behavior for different screen sizes and devices
4. **Browser Compatibility**: Provide fallbacks for older browsers
5. **Framework Integration**: The core logic can be adapted to work with any framework
6. **Empty Content Handling**: Always check for empty content and force reset to minimum height
7. **Comprehensive Event Handling**: Listen for all content-changing events (input, cut, paste, delete)

By following this implementation guide, you can create smooth, accessible, and performant auto-expanding textareas that enhance your application's user experience. The key insight is that empty content requires special handling - when the textarea is empty, bypass complex calculations and immediately reset to the minimum height for optimal user experience.
