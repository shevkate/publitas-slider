# Described Technical Decisions

## Architecture Choices

### Fixed-size Canvas vs Responsive
**Decision:** Fixed-size canvas (600×380)

**Reasoning:**
- Reference implementation uses fixed size
- Simpler scroll calculations
- No resize edge cases
- Matches requirements exactly

**Trade-off:** Less flexible on mobile, but seems acceptable for this challenge.

---

### Free-scrolling vs Snapping
**Decision:** Free-scrolling model (no 'snap-to-image')

**Reasoning:**
- Matches reference behavior exactly
- More direct manipulation feel
- Simpler implementation

**Alternative considered:** Momentum + snapping, but reference does not have it.

---

### DPR Handling
**Decision:** Scale canvas buffer by devicePixelRatio

**Reasoning:**
- Crisp rendering on Retina displays
- Best practice
- Minimal performance cost

### Image Loading Strategy
**Decision:** Progressive loading with counter

**Reasoning:**
- Better UX since user sees progress
- No blocking
-  Degradation goes gracefully if one image fails

**Considered:** Promise.all() was rejected because no progress feedback (poor UX on slow connections)

---

### Performance Optimizations

**'Dirty flag' pattern:**
- Only render when state changes
- ~0 FPS when idle vs constant 60 FPS

**Visibility culling:**
- Only draw visible images
- Saves draw calls when scrolled

**requestAnimationFrame:**
- Browser-optimized timing
- Pauses when tab hidden

---

## Some Trade-offs & Future Improvements

### Not Implemented (Scope Decision)
- **Keyboard navigation** - Could add arrow keys
- **Touch gestures** - Pointer Events handle it, but could add swipe velocity
- **Accessibility** - No ARIA labels, would add for production
- **Tests** - Time-boxed, would add unit + integration tests

### Production Considerations
If this were production code, I would also add:
1. Error boundary component for graceful failures
2. Performance monitoring (Web Vitals)
3. Analytics events (drag start/end, image views)
4. Lazy loading for large image sets
5. Service Worker for offline support
6. A11y improvements (keyboard nav, screen reader support)
---

## Testing Strategy (Not Implemented)
**Unit tests:**
- Aspect ratio calculations
- Scroll boundary clamping
- DPR scaling logic

**Integration tests:**
- Full drag interaction flow
- Image loading states
- Error handling

**E2E tests:**
- Visual regression tests
- Cross-browser compatibility