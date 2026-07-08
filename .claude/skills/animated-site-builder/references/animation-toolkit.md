# Animation Toolkit

Copy-paste, battle-tested. All of this lives inside `<style is:inline>` /
`<script is:inline>`. Uses CSS custom properties so one keyframe drives many elements.

## 1. Scroll reveal (the workhorse)

CSS:
```css
.anim { opacity:0; transform:translateY(32px); transition:opacity .7s var(--ease),transform .7s var(--ease); }
.anim.in { opacity:1; transform:none; }
.d1{transition-delay:.1s} .d2{transition-delay:.2s} .d3{transition-delay:.3s} .d4{transition-delay:.4s}
```
JS:
```js
const obs = new IntersectionObserver(entries => {
  entries.forEach(e => { if(e.isIntersecting){ e.target.classList.add('in'); obs.unobserve(e.target); } });
}, {threshold:0.12});
document.querySelectorAll('.anim').forEach(el => obs.observe(el));
```
Add `class="anim"` (plus `d1..d4` for stagger) to anything that should rise-and-fade in
on scroll. `--ease` is a shared cubic-bezier, e.g. `--ease:cubic-bezier(0.4,0,0.2,1)`.

## 2. Keyframe library

A menu of loops used across the HIO site. Drive them with per-element CSS vars.

```css
@keyframes fadeIn { from{opacity:0} to{opacity:1} }
@keyframes floatRoll { 0%,100%{transform:translateY(0) rotate(var(--ra))} 50%{transform:translateY(-18px) rotate(var(--rb))} }  /* floating decorations */
@keyframes dishFloat { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-5px)} }  /* gentle bob */
@keyframes ensoSpin { from{transform:rotate(0)} to{transform:rotate(360deg)} }  /* slow ring spin, 50s */
@keyframes dustFloat { 0%,100%{transform:translateY(0);opacity:0} 20%{opacity:.8} 50%{transform:translateY(-36px);opacity:.4} 80%{opacity:.7} }  /* rising particles */
@keyframes marqueeX { from{transform:translateX(0)} to{transform:translateX(-50%)} }  /* infinite ticker (duplicate the track) */
@keyframes waveMove { from{transform:translateX(0)} to{transform:translateX(-50%)} }  /* layered wave divider */
@keyframes shimmer { 0%{background-position:200% center} 100%{background-position:-200% center} }  /* gold text sweep on background-clip:text */
@keyframes letterIn { from{opacity:0;transform:translateY(50px) rotate(10deg) scale(.7)} to{opacity:1;transform:none} }  /* hero title letters, stagger via --i */
@keyframes scrollBounce { 0%,100%{transform:translateY(0);opacity:1} 50%{transform:translateY(8px);opacity:.4} }  /* scroll cue */
@keyframes ikuraPulse { 0%,100%{transform:scale(1)} 50%{transform:scale(1.22)} }  /* pulsing roe/dots */
@keyframes sesameFall { 0%{transform:translateY(-8px) rotate(0);opacity:0} 15%,90%{opacity:1} 100%{transform:translateY(95px) rotate(220deg);opacity:0} }  /* falling specks */
@keyframes flameFlick { 0%,100%{transform:scaleY(1) scaleX(1)} 30%{transform:scaleY(1.18) scaleX(.92)} 60%{transform:scaleY(.88) scaleX(1.08)} }  /* flame */
@keyframes steamRise { 0%{transform:translateY(0) scaleX(1);opacity:0} 25%{opacity:.55} 100%{transform:translateY(-26px) scaleX(1.4);opacity:0} }  /* steam wisps */
@keyframes lanternSwing { from{transform:rotate(-5deg)} to{transform:rotate(5deg)} }  /* pair with animation-direction:alternate */
@keyframes kenBurns { 0%,100%{transform:scale(1.02)} 50%{transform:scale(1.12) translate(-1.5%,1%)} }  /* slow photo zoom */
```
`floatRoll` pattern — each element sets its own rhythm:
```html
<div class="ffood" style="top:10%;right:5%;width:80px;--ra:-7deg;--rb:5deg;--fd:9s"> ... </div>
```
```css
.ffood { position:absolute; pointer-events:none; z-index:0; opacity:.8; animation:floatRoll var(--fd,8s) ease-in-out infinite; }
```

## 3. SVG path-draw (line that draws itself on scroll)

```css
.torii-path { stroke:var(--gold); fill:none; stroke-dasharray:var(--len); stroke-dashoffset:var(--len); transition:stroke-dashoffset 1.6s ease; }
.drawn .torii-path { stroke-dashoffset:0; }
```
JS: measure each path and set `--len` to its length, then add `.drawn` when the section
enters view (a second IntersectionObserver):
```js
document.querySelectorAll('.torii-path').forEach(p => {
  const len = p.getTotalLength ? p.getTotalLength() : 500;
  p.style.setProperty('--len', len);
});
new IntersectionObserver(([e]) => { if(e.isIntersecting) e.target.classList.add('drawn'); },
  {threshold:0.4}).observe(document.getElementById('yourSvgWrap'));
```

## 4. Realistic SVG illustrations

Flat fills read as clip-art. For food/product art that looks real:
- **Layered radial gradients** for every rounded element (rice, fish, sphere):
  ```svg
  <radialGradient id="ri" cx="42%" cy="38%" r="62%">
    <stop offset="0%" stop-color="#FDFAF2"/><stop offset="100%" stop-color="#DDD4BC"/>
  </radialGradient>
  ```
- A **drop shadow ellipse** under the subject: `<ellipse cx cy rx ry fill="rgba(0,0,0,.38)"/>`
- **Specular highlight**: a low-opacity white ellipse near the top-left light source.
- **Texture strokes**: faint curved `<path stroke=... fill=none opacity=.4>` for grain/marbling.
- Build sushi-roll cross-sections as concentric circles: rice ring → sesame dots
  (rotated ellipses) → nori ring → inner rice → pie-slice `<path>` fillings.
- **But**: a real self-hosted photo almost always beats SVG. Reach for SVG only for
  decorative floating elements, logos, and icons.

## 5. JS-driven page transition (the reliable pattern)

Do NOT chain CSS `animation` with `forwards` for multi-phase transitions — a filled
end-state blocks the next animation and you get a stuck/black screen. Instead drive
phases with `setTimeout` toggling classes, and animate with CSS `transition`.

Example: two panels close like a curtain, chopsticks descend and grab a nigiri, lift it,
curtain opens on the new section. The critical lesson: **time the phases so the moving
element actually completes its travel before the next phase**. (Our first version had a
1.4s descent but started the lift at 1.3s, so the "grab" never visibly happened.)

CSS (essentials):
```css
#ptOverlay { position:fixed; inset:0; z-index:300; visibility:hidden; pointer-events:none; }
#ptOverlay.pt-active { visibility:visible; pointer-events:all; }
.pt-panel { position:absolute; left:0; width:100%; height:50.5%; transform:scaleY(0);
  transition:transform .42s cubic-bezier(0.4,0,0.2,1); background:linear-gradient(180deg,#0B0A08,#0E0C08); }
.pt-top { top:0; transform-origin:top; } .pt-bot { bottom:0; transform-origin:bottom; }
#ptOverlay.pt-closing .pt-panel, #ptOverlay.pt-open .pt-panel { transform:scaleY(1); }
#ptOverlay.pt-opening .pt-panel { transform:scaleY(0); }
.pt-grab { transform:translateY(-235px); transition:transform .6s cubic-bezier(0.33,0,0.2,1); }
#ptOverlay.pt-open .pt-grab { transform:translateY(0); }             /* descend */
#ptOverlay.pt-grabbed .pt-stick-l { transform:rotate(0); }           /* pinch closed */
.pt-lift .pt-grab, .pt-lift .pt-nigiri { transform:translateY(-285px) rotate(-3deg) !important;
  transition:transform .55s cubic-bezier(0.4,0,1,1) !important; }     /* lift together */
```
JS (phase timeline — the timings are the whole trick):
```js
function runTransition(target){
  if(busy) return; busy = true; const cl = ov.classList;
  cl.add('pt-active','pt-closing');                                   // 0ms curtain closes
  setTimeout(()=>{ cl.remove('pt-closing'); cl.add('pt-open'); },430);// chopsticks descend (.6s)
  setTimeout(()=> jumpTo(target), 720);                              // scroll behind closed curtain
  setTimeout(()=> cl.add('pt-grabbed'), 1050);                       // arrive → pinch
  setTimeout(()=> cl.add('pt-lift'), 1300);                          // lift grabbed item
  setTimeout(()=>{ cl.remove('pt-open','pt-grabbed','pt-lift'); cl.add('pt-opening'); },1950); // open
  setTimeout(()=>{ cl.remove('pt-active','pt-opening'); busy=false; },2420);                    // reset
}
// intercept in-page anchor links; respect reduced motion:
document.querySelectorAll('a[href^="#"]').forEach(a => a.addEventListener('click', e => {
  const t = a.getAttribute('href')==='#' ? document.body : document.querySelector(a.getAttribute('href'));
  if(!t) return; e.preventDefault();
  if(matchMedia('(prefers-reduced-motion:reduce)').matches){ jumpTo(t); return; }
  runTransition(t);
}));
function jumpTo(t){ const h=document.documentElement, p=h.style.scrollBehavior;
  h.style.scrollBehavior='auto'; t===document.body?scrollTo(0,0):t.scrollIntoView(); h.style.scrollBehavior=p; }
```

## 6. Always-visible CTA (sticky FAB)

```css
.fab-prenota { position:fixed; bottom:2rem; right:2rem; z-index:150; animation:fabPulse 3s ease-in-out infinite; }
@keyframes fabPulse { 0%,100%{box-shadow:0 0 0 0 rgba(201,168,76,.4)} 50%{box-shadow:0 0 0 12px rgba(201,168,76,0)} }
```

## 7. Reduced motion

Gate expensive/looping motion behind `@media (prefers-reduced-motion:no-preference)`
where practical, and always give the page transition the reduced-motion bypass shown above.
