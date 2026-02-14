# GSAP Mastery: Zero to Production-Ready Animations

**Target:** 30 minutes/day hands-on learning  
**Philosophy:** Understand deeply, don't memorize  
**Goal:** Build animation intuition for production interfaces

---

## Table of Contents

1. [Mental Model: How Animation Works](#mental-model-how-animation-works)
2. [GSAP Fundamentals](#gsap-fundamentals)
3. [Week 1: Core Methods](#week-1-core-methods)
4. [Week 2: Timelines & Sequencing](#week-2-timelines--sequencing)
5. [Week 3: ScrollTrigger Plugin](#week-3-scrolltrigger-plugin)
6. [Week 4: Advanced Techniques](#week-4-advanced-techniques)
7. [Week 5-6: Real Projects](#week-5-6-real-projects)
8. [Production Patterns](#production-patterns)
9. [Debugging & Performance](#debugging--performance)
10. [Resources](#resources)

---

## Mental Model: How Animation Works

### **What is animation in the browser?**

Animation = changing CSS properties over time.

```
Frame 1: opacity: 0, x: -100px
Frame 2: opacity: 0.1, x: -90px
Frame 3: opacity: 0.2, x: -80px
...
Frame 60: opacity: 1, x: 0px
```

**Browser refresh rate:** ~60fps (60 frames per second) = 1 frame every 16.67ms

**Why GSAP exists:**
- CSS animations: Limited control, hard to sequence, poor performance on complex chains
- `requestAnimationFrame()`: Low-level, manual frame calculations
- GSAP: High-level API, automatic performance optimization, timeline control

**GSAP's job:** Calculate intermediate values between start/end states at 60fps, update the DOM efficiently.

---

### **Key Concepts (Build These Mental Models)**

#### **1. Tweens**
A "tween" = a single animation from point A to point B.

```javascript
// This is ONE tween
gsap.to('.box', { x: 100, duration: 1 });
```

**Mental model:** Think of it like a slider. You tell GSAP the start/end positions, it smoothly moves the slider from 0 to 100.

---

#### **2. Easing**
Easing = how the animation accelerates/decelerates.

**Linear easing:** Constant speed (robotic, unnatural)
```
0 ------- 50 ------- 100  (same speed throughout)
```

**Ease-out:** Fast start, slow end (natural, how physics works)
```
0 ----------- 80 -- 95 - 100  (deceleration)
```

**Ease-in:** Slow start, fast end (less common, feels urgent)
```
0 - 15 -- 40 ----------- 100  (acceleration)
```

**Why this matters:** Real-world objects don't move at constant speed. A ball rolling to a stop decelerates. Easing makes animations feel physically realistic.

**GSAP default:** `power1.out` (gentle deceleration)

**Rule of thumb:**
- UI entrances: `power2.out` (smooth, professional)
- UI exits: `power2.in` (quick departure)
- Playful/bouncy: `back.out` (slight overshoot)
- Urgent/attention: `power4.in` (fast acceleration)

---

#### **3. Stagger**
Stagger = delay between animating multiple elements.

**Without stagger:**
```
Element 1: starts at 0s
Element 2: starts at 0s
Element 3: starts at 0s
(All animate together - chaotic)
```

**With stagger (0.1s):**
```
Element 1: starts at 0s
Element 2: starts at 0.1s
Element 3: starts at 0.2s
(Cascading effect - organized)
```

**Mental model:** Like dominoes falling. Each triggers slightly after the previous one.

---

#### **4. Timelines**
Timeline = container for multiple tweens that play in sequence or overlap.

**Without timeline (messy):**
```javascript
gsap.to('.box1', { x: 100, duration: 1 });
setTimeout(() => {
  gsap.to('.box2', { y: 100, duration: 1 });
}, 1000);
setTimeout(() => {
  gsap.to('.box3', { scale: 2, duration: 1 });
}, 2000);
```

**With timeline (clean):**
```javascript
const tl = gsap.timeline();
tl.to('.box1', { x: 100, duration: 1 })
  .to('.box2', { y: 100, duration: 1 })
  .to('.box3', { scale: 2, duration: 1 });
```

**Mental model:** Think of it like a video editor timeline. You arrange clips (tweens) in order, control playback speed, pause/resume.

---

## GSAP Fundamentals

### **Installation (React + Vite)**

```bash
npm install gsap
```

**Why gsap, not @gsap/core?**  
`gsap` is the main package. Plugins (ScrollTrigger, SplitText) are imported separately.

---

### **Basic Setup in React**

```javascript
import { useGSAP } from '@gsap/react';
import gsap from 'gsap';

function MyComponent() {
  // useGSAP hook = useEffect but optimized for GSAP
  // It cleans up animations when component unmounts
  useGSAP(() => {
    gsap.to('.box', { x: 100, duration: 1 });
  });

  return <div className="box w-20 h-20 bg-blue-500"></div>;
}
```

**Why `useGSAP` instead of `useEffect`?**
- Automatically kills animations on cleanup (prevents memory leaks)
- Scopes animations to component (avoids global pollution)
- Better performance (batches updates)

**Install the hook:**
```bash
npm install @gsap/react
```

---

## Week 1: Core Methods

**Goal:** Master the 3 fundamental methods: `to`, `from`, `fromTo`  
**Time:** 30 min/day × 7 days = 3.5 hours

---

### **Day 1: `gsap.to()` - The Foundation**

**What it does:** Animates FROM current state TO a new state.

```javascript
// Current state: x = 0, opacity = 1
gsap.to('.box', { 
  x: 100,        // Move 100px to the right
  opacity: 0.5,  // Fade to 50%
  duration: 1    // Over 1 second
});
// End state: x = 100, opacity = 0.5
```

**Mental model:** "Take this element and move it THERE."

---

#### **Hands-On Exercise 1.1: Basic Movement**

**Setup:**
```jsx
function Day1() {
  const boxRef = useRef(null);

  useGSAP(() => {
    gsap.to(boxRef.current, { x: 200, duration: 2 });
  });

  return <div ref={boxRef} className="w-20 h-20 bg-blue-500" />;
}
```

**What's happening:**
1. `boxRef.current` = the DOM element
2. GSAP calculates: "Element is at x=0, needs to be at x=200"
3. Over 2 seconds, GSAP updates `transform: translateX()` 60 times/second
4. Browser renders smooth movement

**Verify it worked:**
- Open DevTools → Inspect element
- Watch the `transform` style change in real-time
- See `translateX()` go from 0px → 200px

**Debug exercise:** Change `duration: 2` to `duration: 0.1`. What happens? (Should snap almost instantly)

---

#### **Hands-On Exercise 1.2: Multiple Properties**

```javascript
gsap.to('.box', {
  x: 100,
  y: 50,
  rotation: 360,    // Rotate 360 degrees
  scale: 1.5,       // Grow 150%
  opacity: 0.5,
  duration: 2,
  ease: 'power2.out'
});
```

**What each property does:**
- `x`: Horizontal movement (uses `translateX`)
- `y`: Vertical movement (uses `translateY`)
- `rotation`: Spin (uses `rotate`)
- `scale`: Size (uses `scale`)
- `opacity`: Transparency

**Why these properties are fast:**  
GSAP uses CSS `transform` and `opacity` which are GPU-accelerated. Animating `left`, `top`, `width`, `height` is slow (causes layout recalculation).

**Rule:** Always animate `transform` properties (x, y, scale, rotation), not layout properties.

---

#### **Hands-On Exercise 1.3: Easing Comparison**

Create 3 boxes with different easing:

```javascript
useGSAP(() => {
  gsap.to('.box1', { x: 300, duration: 2, ease: 'none' });       // Linear
  gsap.to('.box2', { x: 300, duration: 2, ease: 'power2.out' }); // Ease-out
  gsap.to('.box3', { x: 300, duration: 2, ease: 'back.out' });   // Bounce
});
```

**Watch them move.** Feel the difference.

- `none`: Constant speed (feels mechanical)
- `power2.out`: Natural deceleration (professional standard)
- `back.out`: Overshoots then settles (playful, attention-grabbing)

**Debugging tip:** Open https://gsap.com/docs/v3/Eases/ and play with the visualizer.

---

### **Day 2: `gsap.from()` - Reverse Thinking**

**What it does:** Animates FROM a new state TO the current state.

```javascript
// Current state: x = 0, opacity = 1
gsap.from('.box', { 
  x: -100,       // Starts at x = -100
  opacity: 0,    // Starts invisible
  duration: 1 
});
// Animation: x goes -100 → 0, opacity goes 0 → 1
```

**Mental model:** "This element should enter FROM over there."

**Use case:** Element entrances. The element exists in the DOM at its final position, but you animate it FROM off-screen or faded out.

---

#### **Hands-On Exercise 2.1: Fade-In Effect**

```javascript
useGSAP(() => {
  gsap.from('.hero-title', {
    opacity: 0,
    y: -50,        // Starts 50px above
    duration: 1,
    ease: 'power2.out'
  });
});
```

**What happens:**
1. DOM renders with `opacity: 1, y: 0` (final state)
2. GSAP overrides it to `opacity: 0, y: -50` instantly
3. Animates back to `opacity: 1, y: 0` over 1 second

**Common mistake:** Forgetting `useGSAP` runs after render. If you see a flash of content before animation, that's why.

**Fix:** Add `opacity: 0` to initial CSS, then animate opacity to 1.

---

#### **Hands-On Exercise 2.2: Slide-In Cards**

```javascript
useGSAP(() => {
  gsap.from('.card', {
    x: -100,
    opacity: 0,
    duration: 0.8,
    stagger: 0.2,    // Each card delayed by 0.2s
    ease: 'power2.out'
  });
});
```

**With 3 cards:**
- Card 1: starts at 0s
- Card 2: starts at 0.2s
- Card 3: starts at 0.4s

**Stagger rule of thumb:**
- 0.05-0.1s: Subtle, elegant
- 0.2-0.3s: Noticeable, rhythmic
- 0.5s+: Too slow, user waits

---

### **Day 3: `gsap.fromTo()` - Full Control**

**What it does:** Defines BOTH start and end states explicitly.

```javascript
gsap.fromTo('.box', 
  { x: -100, opacity: 0 },     // FROM state
  { x: 100, opacity: 1, duration: 1 }  // TO state
);
```

**Mental model:** "Animate from HERE to THERE, ignoring current state."

**Use case:** When you need precise control over both states (complex transitions, reusable animations).

---

#### **Hands-On Exercise 3.1: Controlled Transition**

```javascript
gsap.fromTo('.modal',
  { 
    scale: 0.8, 
    opacity: 0,
    y: 50 
  },
  { 
    scale: 1, 
    opacity: 1,
    y: 0,
    duration: 0.5,
    ease: 'back.out'
  }
);
```

**Why `fromTo` here?**  
We want the modal to ALWAYS start small/faded, regardless of its CSS. This ensures consistency even if the component re-renders.

---

#### **Hands-On Exercise 3.2: Hover Effect with GSAP**

```javascript
const cardRef = useRef(null);

const handleMouseEnter = () => {
  gsap.to(cardRef.current, {
    scale: 1.05,
    boxShadow: '0 10px 30px rgba(0,0,0,0.3)',
    duration: 0.3,
    ease: 'power2.out'
  });
};

const handleMouseLeave = () => {
  gsap.to(cardRef.current, {
    scale: 1,
    boxShadow: '0 2px 10px rgba(0,0,0,0.1)',
    duration: 0.3,
    ease: 'power2.out'
  });
};

return (
  <div 
    ref={cardRef} 
    onMouseEnter={handleMouseEnter}
    onMouseLeave={handleMouseLeave}
    className="p-6 bg-white rounded-lg"
  >
    Hover me
  </div>
);
```

**Why GSAP for hover?**  
CSS `:hover` is instant or uses CSS transitions. GSAP gives you easing control, interruption handling (hover multiple times quickly).

---

### **Day 4-7: Practice Projects**

**Project 1: Animated Header**
- Logo fades in from top
- Nav items stagger in from right
- CTA button scales in with bounce

**Project 2: Card Grid**
- 6 cards fade in with stagger
- Hover: card lifts, shadow grows
- Click: card flips (rotate Y axis)

**Project 3: Modal Animation**
- Background fades in (opacity 0 → 0.8)
- Modal scales in (scale 0.8 → 1)
- Content fades in after modal settles

**Debugging checklist:**
- [ ] Animation plays on mount?
- [ ] Animation cleans up on unmount? (Check with React DevTools)
- [ ] Smooth 60fps? (DevTools → Performance → Record)
- [ ] Easing feels natural?

---

## Week 2: Timelines & Sequencing

**Goal:** Master coordinating multiple animations  
**Why it matters:** Real UIs have choreographed sequences (loader → content → CTA)

---

### **Day 8: Timeline Basics**

**What is a timeline?**  
A container that plays tweens in sequence or overlaps them with precise control.

```javascript
const tl = gsap.timeline();

tl.to('.box1', { x: 100, duration: 1 })      // Starts at 0s
  .to('.box2', { y: 100, duration: 1 })      // Starts at 1s (after box1)
  .to('.box3', { rotation: 360, duration: 1 }); // Starts at 2s
```

**Mental model:** Like a playlist. Each song (tween) plays after the previous one ends.

---

#### **Hands-On Exercise 8.1: Sequential Animations**

```javascript
useGSAP(() => {
  const tl = gsap.timeline();
  
  tl.from('.logo', { opacity: 0, y: -20, duration: 0.5 })
    .from('.nav-item', { opacity: 0, x: 20, stagger: 0.1 })
    .from('.cta-button', { scale: 0, duration: 0.3, ease: 'back.out' });
});
```

**What happens:**
1. Logo fades in (0.5s)
2. Nav items stagger in (0.5s total if 5 items @ 0.1s each)
3. CTA button pops in (0.3s)

**Total duration:** 0.5 + 0.5 + 0.3 = 1.3s

---

### **Day 9: Position Parameter (Advanced Timing)**

**The position parameter controls WHEN a tween starts within a timeline.**

```javascript
tl.to('.box1', { x: 100, duration: 1 })
  .to('.box2', { y: 100, duration: 1 }, '-=0.5')  // Start 0.5s BEFORE box1 ends
  .to('.box3', { scale: 2, duration: 1 }, '+=0.2'); // Start 0.2s AFTER box2 ends
```

**Position values:**
- `'-=0.5'`: Start 0.5s before previous tween ends (overlap)
- `'+=0.2'`: Start 0.2s after previous tween ends (gap)
- `'<'`: Start at same time as previous tween (simultaneous)
- `'>'`: Start when previous tween ends (default)
- `2`: Start at 2 seconds on the timeline (absolute)

---

#### **Hands-On Exercise 9.1: Overlapping Animations**

```javascript
const tl = gsap.timeline();

tl.to('.background', { opacity: 0.8, duration: 0.5 })
  .to('.modal', { scale: 1, duration: 0.5 }, '-=0.3')  // Starts 0.2s into background fade
  .to('.modal-content', { opacity: 1, y: 0, duration: 0.4 }, '-=0.2'); // Starts before modal finishes
```

**Why overlap?**  
Feels more fluid. Real-world motion doesn't wait for things to finish—multiple things happen at once.

**Rule:** Overlap by 0.2-0.3s for smooth transitions. Full overlap (`<`) can feel chaotic.

---

### **Day 10: Timeline Control (Play/Pause/Reverse)**

**Timelines are objects you can control.**

```javascript
const tl = gsap.timeline({ paused: true }); // Don't play automatically

tl.to('.box1', { x: 100, duration: 1 })
  .to('.box2', { y: 100, duration: 1 });

// Control methods
tl.play();       // Start/resume
tl.pause();      // Pause
tl.reverse();    // Play backwards
tl.restart();    // Reset to start and play
tl.seek(1.5);    // Jump to 1.5 seconds
tl.progress(0.5); // Jump to 50% completion
```

---

#### **Hands-On Exercise 10.1: Interactive Timeline**

```javascript
function AnimatedMenu() {
  const tlRef = useRef(null);

  useGSAP(() => {
    tlRef.current = gsap.timeline({ paused: true });
    
    tlRef.current
      .to('.menu-bg', { height: '100vh', duration: 0.5 })
      .to('.menu-item', { opacity: 1, y: 0, stagger: 0.1 }, '-=0.3');
  });

  const openMenu = () => tlRef.current.play();
  const closeMenu = () => tlRef.current.reverse();

  return (
    <>
      <button onClick={openMenu}>Open Menu</button>
      <button onClick={closeMenu}>Close Menu</button>
      <div className="menu-bg h-0 bg-black" />
      <div className="menu-item opacity-0 translate-y-4">Item 1</div>
    </>
  );
}
```

**Why `paused: true`?**  
We want the timeline to exist but not play until the button is clicked.

**Why `reverse()`?**  
It plays the timeline backwards smoothly. Much better than creating a separate "close" animation.

---

### **Day 11-14: Practice Projects**

**Project 4: Loading Sequence**
- Spinner fades in
- Spinner rotates continuously
- On data load: spinner fades out, content fades in

**Project 5: Multi-Step Form**
- Step 1 slides in
- User clicks "Next"
- Step 1 slides out left, Step 2 slides in right
- Reverse on "Back"

**Project 6: Hero Section Sequence**
- Background image fades in
- Overlay darkens
- Headline types in (character by character, requires SplitText)
- Subtext fades in
- CTA button pops in

---

## Week 3: ScrollTrigger Plugin

**Goal:** Trigger animations based on scroll position  
**Why it matters:** Most modern sites use scroll-based animations (parallax, reveals)

---

### **Day 15: ScrollTrigger Setup**

**What is ScrollTrigger?**  
A plugin that detects when elements enter/leave the viewport and triggers animations.

```bash
npm install gsap
```

**Import:**
```javascript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);
```

**Why `registerPlugin`?**  
Plugins are separate to keep the core library small. You only load what you need.

---

#### **Basic ScrollTrigger Syntax**

```javascript
gsap.to('.box', {
  x: 100,
  scrollTrigger: {
    trigger: '.box',       // Element that triggers the animation
    start: 'top 80%',      // When top of .box hits 80% of viewport
    end: 'bottom 20%',     // When bottom of .box hits 20% of viewport
    scrub: true,           // Link animation progress to scroll position
    markers: true          // Show debug markers (remove in production)
  }
});
```

**Mental model:**  
"When `.box` enters the viewport (at 80% down the screen), start animating. Progress the animation as the user scrolls."

---

### **Day 16: Understanding Start/End Positions**

**Start/End syntax:** `'[trigger point] [viewport point]'`

```
start: 'top 80%'
       ↑     ↑
     trigger  viewport
```

**Translation:** "When the TOP of the trigger element reaches 80% down the viewport."

**Common configurations:**

```javascript
// Animation starts when element is halfway visible
start: 'top 50%'

// Animation starts when element just enters screen
start: 'top 100%'

// Animation starts when element reaches top of screen
start: 'top 0%'

// Alternative keywords
start: 'top center'  // Same as 'top 50%'
start: 'top bottom'  // Same as 'top 100%'
start: 'top top'     // Same as 'top 0%'
```

---

#### **Hands-On Exercise 16.1: Fade In on Scroll**

```javascript
useGSAP(() => {
  gsap.from('.section', {
    opacity: 0,
    y: 50,
    duration: 1,
    scrollTrigger: {
      trigger: '.section',
      start: 'top 80%',   // Start when section is 80% down viewport
      end: 'top 50%',     // End when section is centered
      markers: true,      // DEBUG: See the trigger points
      toggleActions: 'play none none reverse'
    }
  });
});
```

**toggleActions explained:**
```
'play none none reverse'
  ↓    ↓    ↓     ↓
onEnter onLeave onEnterBack onLeaveBack
```

- `onEnter`: Play animation when scrolling down into trigger
- `onLeave`: Do nothing when scrolling past trigger
- `onEnterBack`: Do nothing when scrolling back up into trigger
- `onLeaveBack`: Reverse animation when scrolling back up past trigger

**Common patterns:**
- `'play none none none'`: One-time trigger (animation stays)
- `'play reverse play reverse'`: Toggle on/off (up and down)
- `'play none none reset'`: Reset when scrolling back up

---

### **Day 17: Scrub (Scroll-Linked Animations)**

**What is `scrub`?**  
Links animation progress directly to scroll position. Scrolling = animation playing.

```javascript
gsap.to('.box', {
  x: 100,
  scrollTrigger: {
    trigger: '.box',
    start: 'top 80%',
    end: 'top 20%',
    scrub: true  // Animation progresses with scroll
  }
});
```

**Scrub values:**
- `scrub: true`: Instant (animation follows scroll exactly)
- `scrub: 1`: 1-second lag (smooths out jerky scrolling)
- `scrub: 2`: 2-second lag (very smooth, dreamy effect)

**Use cases:**
- Parallax effects (background moves slower than foreground)
- Progress bars (fill as user scrolls)
- Number counters (increment with scroll)

---

#### **Hands-On Exercise 17.1: Parallax Background**

```javascript
useGSAP(() => {
  gsap.to('.background', {
    y: 200,  // Move down 200px
    scrollTrigger: {
      trigger: '.hero-section',
      start: 'top top',
      end: 'bottom top',
      scrub: 1.5,  // Smooth lag
    }
  });
});
```

**What happens:**  
As user scrolls through `.hero-section`, the background moves slower than the content (parallax effect).

**Why `end: 'bottom top'`?**  
Animation completes when the bottom of `.hero-section` reaches the top of the viewport (user has scrolled past entire section).

---

### **Day 18: Pin (Sticky Elements)**

**What is `pin`?**  
Locks an element in place while other content scrolls. Used for sticky headers, reading progress, "pinned" sections.

```javascript
gsap.to('.box', {
  rotation: 360,
  scrollTrigger: {
    trigger: '.section',
    start: 'top top',
    end: 'bottom bottom',
    pin: '.box',  // Pin .box while .section scrolls
    scrub: true
  }
});
```

**Mental model:**  
"Stick `.box` to the viewport while `.section` scrolls past. Animate it during that time."

---

#### **Hands-On Exercise 18.1: Pinned Text Reveal**

```javascript
useGSAP(() => {
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: '.pin-section',
      start: 'top top',
      end: '+=200%',  // Pin for 2x the viewport height
      pin: true,
      scrub: 1
    }
  });

  tl.from('.text1', { opacity: 0, y: 50 })
    .from('.text2', { opacity: 0, y: 50 })
    .from('.text3', { opacity: 0, y: 50 });
});
```

**What happens:**
1. Section pins to top of viewport
2. User scrolls 200% viewport height
3. During that scroll, text elements fade in sequentially
4. Section unpins, normal scroll resumes

**Use case:** Storytelling sections (Apple product pages, journalism sites).

---

### **Day 19-21: Practice Projects**

**Project 7: Parallax Landing Page**
- Hero background moves slower than foreground
- Feature cards fade in on scroll (staggered)
- Stats counter animates from 0 to final value as user scrolls

**Project 8: Scroll Progress Bar**
- Fixed bar at top of page
- Width increases 0% → 100% as user scrolls entire page

**Project 9: Pinned Product Showcase**
- Product image stays centered while features scroll past
- Each feature lights up when it reaches center

---

## Week 4: Advanced Techniques

**Goal:** Text animations, custom easings, performance optimization

---

### **Day 22-23: SplitText Plugin (Text Animations)**

**What is SplitText?**  
Breaks text into characters, words, or lines so you can animate them individually.

**WARNING:** SplitText is a **premium plugin** (requires GSAP membership, $99/year).  
**Free alternative:** Use CSS `clip-path` or manually split text in React.

**If you have SplitText:**

```javascript
import { SplitText } from 'gsap/SplitText';
gsap.registerPlugin(SplitText);

useGSAP(() => {
  const split = new SplitText('.headline', { type: 'chars' });
  
  gsap.from(split.chars, {
    opacity: 0,
    y: 20,
    stagger: 0.05,
    duration: 0.8,
    ease: 'power2.out'
  });
});
```

**Free alternative (React):**

```javascript
function AnimatedText({ text }) {
  const chars = text.split('');

  useGSAP(() => {
    gsap.from('.char', {
      opacity: 0,
      y: 20,
      stagger: 0.05,
      duration: 0.8,
      ease: 'power2.out'
    });
  }, []);

  return (
    <h1>
      {chars.map((char, i) => (
        <span key={i} className="char inline-block">
          {char === ' ' ? '\u00A0' : char}
        </span>
      ))}
    </h1>
  );
}
```

**Use case:** Hero headlines, attention-grabbing titles, typewriter effects.

---

### **Day 24: Custom Easings**

**GSAP easing types:**

```javascript
// Power easings (most common)
'power1.out'  // Subtle
'power2.out'  // Standard (default for most UIs)
'power3.out'  // Strong
'power4.out'  // Very strong

// Back (overshoot)
'back.out'    // Slight bounce at end
'back.inOut'  // Overshoot both directions

// Elastic (springy)
'elastic.out' // Bouncy spring effect

// Bounce (literal bounce)
'bounce.out'  // Ball bouncing to rest

// Custom cubic-bezier
'cubic-bezier(0.4, 0, 0.2, 1)'
```

**Easing visualizer:** https://gsap.com/docs/v3/Eases/

**When to use each:**
- **Power2.out:** 90% of UI animations (safe default)
- **Back.out:** Buttons, CTAs, attention-grabbing elements
- **Elastic:** Playful UIs (games, kids' apps)
- **Bounce:** Rarely (feels dated unless intentional)

---

### **Day 25: Performance Optimization**

**GSAP is fast, but you can make it faster.**

#### **Rule 1: Animate transform properties only**

```javascript
// ❌ SLOW (causes layout recalculation)
gsap.to('.box', { left: 100, top: 50, width: 200 });

// ✅ FAST (GPU-accelerated)
gsap.to('.box', { x: 100, y: 50, scale: 2 });
```

**Why?**  
`transform` and `opacity` are composited on the GPU. `left`, `top`, `width`, `height` trigger layout recalculation on the CPU.

---

#### **Rule 2: Use `will-change` for complex animations**

```css
.animated-box {
  will-change: transform, opacity;
}
```

**What it does:** Tells the browser "this element will animate, prepare a GPU layer."

**Warning:** Don't overuse. Too many `will-change` elements slow down the page.

**Rule of thumb:** Only add to elements that animate frequently (carousels, parallax backgrounds).

---

#### **Rule 3: Batch animations with timelines**

```javascript
// ❌ SLOWER (multiple timelines)
gsap.to('.box1', { x: 100 });
gsap.to('.box2', { x: 100 });
gsap.to('.box3', { x: 100 });

// ✅ FASTER (single timeline)
const tl = gsap.timeline();
tl.to('.box1', { x: 100 })
  .to('.box2', { x: 100 }, '<')  // Simultaneous
  .to('.box3', { x: 100 }, '<');
```

**Why?** GSAP batches DOM updates when using timelines.

---

#### **Rule 4: Kill animations on cleanup**

```javascript
useGSAP(() => {
  const ctx = gsap.context(() => {
    gsap.to('.box', { x: 100, duration: 10 });
  });

  return () => ctx.revert(); // Cleanup
});
```

**Why?** Prevents memory leaks when components unmount.

**useGSAP does this automatically**, but if you're using vanilla `useEffect`, you need manual cleanup.

---

### **Day 26-28: Practice Projects**

**Project 10: Animated Hero Text**
- Headline splits into characters
- Characters fade in with stagger
- Cursor blinks (CSS animation)

**Project 11: Smooth Page Transitions**
- Page content fades out on route change
- New page content fades in
- Use React Router + GSAP

**Project 12: Cursor Follower**
- Custom cursor that follows mouse
- Grows on hover over links
- Shrinks on click

---

## Week 5-6: Real Projects

**Build these from scratch. No tutorials. Apply everything you've learned.**

---

### **Project 13: SaaS Landing Page (Week 5)**

**Features:**
1. **Hero Section**
   - Background gradient animates on scroll (scrub)
   - Headline text splits and staggers in
   - CTA button scales in with back.out easing

2. **Feature Cards**
   - Grid of 6 cards
   - Fade in with stagger on scroll
   - Hover: lift effect (scale + shadow)

3. **Stats Counter**
   - Numbers count from 0 to final value
   - Triggered by scroll (when section enters viewport)

4. **Footer**
   - Links fade in on scroll
   - Social icons bounce in (back.out)

**Constraints:**
- No AI-generated layouts (design it in Figma first)
- Mobile-responsive (test on 375px width)
- 60fps performance (check DevTools)

---

### **Project 14: Interactive Portfolio (Week 6)**

**Features:**
1. **Custom Cursor**
   - Circle that follows mouse with lag (lerp)
   - Grows on hover over project thumbnails

2. **Project Grid**
   - Images fade in with parallax on scroll
   - Click opens modal with project details

3. **Modal Animation**
   - Background fades in
   - Modal slides up with back.out
   - Content staggers in

4. **Skills Section**
   - Skill bars fill on scroll (scrub)
   - Percentage counters animate

5. **Contact Form**
   - Input fields highlight on focus (GSAP tween)
   - Submit button has loading animation (rotation)

**Constraints:**
- Design system: 8px spacing scale, defined color palette
- Accessibility: ARIA labels, keyboard navigation
- Performance: Lazy load images, optimize animations

---

## Production Patterns

**Reusable animation hooks for React + GSAP**

---

### **Pattern 1: Fade-In on Mount**

```javascript
// hooks/useFadeIn.js
import { useGSAP } from '@gsap/react';
import gsap from 'gsap';

export function useFadeIn(ref, options = {}) {
  const { duration = 0.8, delay = 0, y = 20 } = options;

  useGSAP(() => {
    gsap.from(ref.current, {
      opacity: 0,
      y,
      duration,
      delay,
      ease: 'power2.out'
    });
  }, [ref]);
}

// Usage
function MyComponent() {
  const ref = useRef(null);
  useFadeIn(ref, { duration: 1, y: 30 });

  return <div ref={ref}>Content</div>;
}
```

---

### **Pattern 2: Scroll Reveal (Reusable)**

```javascript
// hooks/useScrollReveal.js
import { useGSAP } from '@gsap/react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

export function useScrollReveal(ref, options = {}) {
  const { start = 'top 80%', end = 'top 50%', scrub = false } = options;

  useGSAP(() => {
    gsap.from(ref.current, {
      opacity: 0,
      y: 50,
      duration: 1,
      scrollTrigger: {
        trigger: ref.current,
        start,
        end,
        scrub,
        toggleActions: 'play none none reverse'
      }
    });
  }, [ref]);
}
```

---

### **Pattern 3: Stagger Grid**

```javascript
export function useStaggerGrid(containerRef, options = {}) {
  const { childSelector = '.grid-item', stagger = 0.1 } = options;

  useGSAP(() => {
    gsap.from(`${containerRef.current} ${childSelector}`, {
      opacity: 0,
      y: 30,
      duration: 0.8,
      stagger,
      ease: 'power2.out'
    });
  }, [containerRef]);
}

// Usage
function Grid() {
  const gridRef = useRef(null);
  useStaggerGrid(gridRef, { childSelector: '.card', stagger: 0.15 });

  return (
    <div ref={gridRef} className="grid grid-cols-3 gap-4">
      <div className="card">Card 1</div>
      <div className="card">Card 2</div>
      <div className="card">Card 3</div>
    </div>
  );
}
```

---

## Debugging & Performance

### **Common Issues and Fixes**

#### **Issue 1: Animation doesn't play**

**Checklist:**
- [ ] Is the element in the DOM? (Check with DevTools)
- [ ] Is `useGSAP` inside the component? (Not outside)
- [ ] Is the selector correct? (Try `console.log(ref.current)`)
- [ ] Is duration set? (Default is 0.5s, might be too fast to see)

**Debug:**
```javascript
useGSAP(() => {
  console.log('Element:', ref.current); // Should not be null
  gsap.to(ref.current, { x: 100, duration: 2 }); // Slow it down
});
```

---

#### **Issue 2: Animation is janky (not 60fps)**

**Check:**
1. Open DevTools → Performance → Record → Scroll/interact → Stop
2. Look for "Scripting" (yellow) and "Rendering" (purple) bars
3. If lots of purple: You're animating layout properties (bad)

**Fix:**
```javascript
// ❌ Animates width (layout property)
gsap.to('.box', { width: 200 });

// ✅ Animate scaleX (transform)
gsap.to('.box', { scaleX: 2 });
```

---

#### **Issue 3: ScrollTrigger fires multiple times**

**Cause:** Component re-renders, creates duplicate ScrollTriggers.

**Fix:**
```javascript
useGSAP(() => {
  ScrollTrigger.getAll().forEach(st => st.kill()); // Kill old triggers

  gsap.to('.box', {
    x: 100,
    scrollTrigger: { trigger: '.box', start: 'top 80%' }
  });
}, [dependency]); // Only re-run when dependency changes
```

---

#### **Issue 4: Text animation breaks layout**

**Cause:** SplitText wraps characters in `<span>`, changing inline layout.

**Fix:**
```css
.char {
  display: inline-block; /* Allows transform on inline elements */
}
```

---

### **Performance Monitoring**

```javascript
// Log GSAP performance stats
gsap.ticker.add(() => {
  console.log('FPS:', gsap.ticker.fps);
});

// Kill after 5 seconds
setTimeout(() => gsap.ticker.remove(), 5000);
```

**Target:** 60fps. Anything below 50fps feels choppy.

---

## Resources

### **Official Docs**
- GSAP Docs: https://gsap.com/docs/v3/
- Easing Visualizer: https://gsap.com/docs/v3/Eases/
- ScrollTrigger Demos: https://gsap.com/docs/v3/Plugins/ScrollTrigger/

### **Courses (Paid)**
- **GSAP Official Course:** https://gsap.com/learn/ (Worth it for SplitText, advanced techniques)
- **JavaScript Mastery GSAP Course:** (The video you watched)

### **Free Tutorials**
- GSAP YouTube Channel: https://www.youtube.com/@GreenSockLearning
- Fireship GSAP Quickstart: https://www.youtube.com/watch?v=YqOhQWbouuE

### **Inspiration (Study These)**
- Apple Product Pages (iPhone, MacBook)
- Stripe Homepage: https://stripe.com
- Linear: https://linear.app
- Awwwards: https://www.awwwards.com (winner sites use GSAP heavily)

### **Tools**
- GSAP Ease Visualizer: https://gsap.com/docs/v3/Eases/
- React DevTools (check component renders)
- Chrome DevTools Performance Tab (check 60fps)

---

## Final Checklist: Are You Production-Ready?

**You should be able to:**
- [ ] Animate any element with `to`, `from`, `fromTo` without docs
- [ ] Create multi-step sequences with timelines
- [ ] Control timeline playback (play/pause/reverse/seek)
- [ ] Trigger animations on scroll (ScrollTrigger)
- [ ] Create parallax effects with `scrub`
- [ ] Pin elements while scrolling
- [ ] Optimize animations for 60fps
- [ ] Debug janky animations (DevTools Performance)
- [ ] Build reusable animation hooks in React
- [ ] Explain easing to a designer (and justify your choices)

**Test yourself:**
- Build a landing page with 0 tutorials (Figma → Code)
- Animate a complex modal (fade background, scale modal, stagger content)
- Create a scroll-triggered counter (0 → 1000 as user scrolls)
- Optimize a slow animation (use Performance tab to find issue)

**If you can do these, you're production-ready.**

---

## Next Steps After Mastery

1. **Framer Motion (Alternative to GSAP for React)**
   - Declarative API (easier for React devs)
   - Similar concepts (timelines = variants, scrub = scroll)
   - Worth learning for comparison

2. **Three.js (3D Animations)**
   - GSAP works with Three.js for 3D object animations
   - Next level: Animate camera, meshes, materials

3. **Lottie (Designer-Friendly Animations)**
   - Import After Effects animations to web
   - Controlled by GSAP (seek, play, reverse)

4. **Canvas Animations**
   - GSAP can animate canvas properties
   - Games, data visualizations, creative coding

---

**Good luck. Build something beautiful.**
