# Responsive contrast & reincorporation handoff

**Context:** A responsive design was built in a now-deleted scratch file (`branded-responsive.html`).
Most of it was re-applied into the production `index.html` during the 2026-06-17 cleanup, but in a
lighter form — two pieces drifted from the intended ("final") design. This doc is a section-by-section
contrast plus the exact code to close the gaps. **Nothing here has been applied to `index.html`.**

The production mobile layer lives in one late `<style>` block (`index.html` ≈ L2140–2219) plus a
one-line curve-JS change (≈ L1158). Desktop (≥901) is untouched and should stay that way.

Legend: ✅ matches final design · ⚠️ drifted · 🟥 structural gap

---

## 1. Global / breakpoints — ✅ matches
- Breakpoint contract `sm 600 / md 900 / lg 1200` present.
- `.slide-brand { display:none }` ≤900 present.
No action.

---

## 2. Hero — ✅ mostly matches, ⚠️ two subtle items
Production correctly: `aspect-ratio:auto; height:100svh; min-height:560px`, suppresses
`.mid/.subject/.anchor` + injected weave `divs`, keeps the `.bg` carousel translateX, bottom
scrim, stacked copy, full-width CTA, centered dots.

- ✅ **Surfer flip** — production *keeps* `scaleX(-1)` on the h2121 cover (L2162). This is a
  genuine fix the cleanup made; the scratch file's blanket `transform:none` had wrongly un-mirrored
  him. **Keep production's version.**
- ⚠️ **Per-slide framing.** Production uses `object-position:center` for every slide (L2158).
  The final design framed each scene so the subject stays in the tall crop. With the couple slide
  now removed, only two remain — recommended tuning:
  ```css
  .opening-hero-slide[data-weave="h2121"] .opening-hero-layer.bg .hero-cover { object-position: 50% 42% !important; }
  .opening-hero-slide[data-weave="h2106"] .opening-hero-layer.bg .hero-cover { object-position: 60% 30% !important; }
  ```
  (Keep the `scaleX(-1) !important` on h2121 — append, don't replace.)

---

## 3. Curve / "the Graf" — ⚠️ THE flagged regression
**Symptom:** ATHLETE / INDEPENDENT / DEPENDENT are crammed into the left gutter (`x=204`), tiny and
overlapping the plot edge — the exact cramped-gutter problem the final design moved away from.

**Final design:** drop the gutter labels; show **two zone labels centered in the open space** above
and below the curve — `Athlete` in the upper zone, `Dependent on others` in the lower zone — and hide
`INDEPENDENT` + the rotated `CAPABILITY`/`AGE` titles on mobile. The curve runs top-left→bottom-right,
so the plot's horizontal centre (x=864) is clear in both zones. Also crops the viewBox to reduce the
letterbox. Curve constants in this file: `PL=216 PR=1512 PT=160 PB=736`, viewBox `1728×810`, centre x=864.

### Fix 3a — CSS (add inside the existing `@media (max-width:900px)` block)
```css
/* zone labels replace the cramped left-gutter axis labels */
.curve-instrument #cp-independent,
.curve-instrument #cp-ycap,
.curve-instrument #cp-xage { display: none !important; }
.curve-instrument .grafzone { font-family: var(--audit); fill: #5A6378; letter-spacing: .04em; font-size: 62px; }
```
The existing `.endlab { font-size:24px }` (L2188) and the `#cp-dependent tspan:nth-child(2)` rule
(L2189) become irrelevant once the labels are repositioned + reclassed by the script below — you can
leave or delete them.

### Fix 3b — script (add immediately AFTER the curve IIFE's `</script>`)
Runs phone-only, after the curve script (which is in FIXED mode on mobile → no scroll re-render, so
these positions stay put):
```html
<script>
/* MOBILE CURVE ZONE LABELS — re-present the y-axis as two labels centered in the
   open space above/below the curve; drop INDEPENDENT + rotated axis titles. */
(function(){
  if(!matchMedia('(max-width:900px)').matches) return;
  var svg=document.getElementById('cp-plot'); if(!svg) return;
  svg.setAttribute('viewBox','176 112 1376 700');   // trim dead margin → less letterbox
  var cx=864;                                        // plot centre (PL+PR)/2
  var ath=document.getElementById('cp-athlete'), dep=document.getElementById('cp-dependent');
  if(ath){ ath.textContent='Athlete'; ath.setAttribute('x',cx); ath.setAttribute('y',324); ath.setAttribute('text-anchor','middle'); ath.setAttribute('class','grafzone'); ath.style.opacity=1; }
  if(dep){ dep.textContent='Dependent on others'; dep.setAttribute('x',cx); dep.setAttribute('y',660); dep.setAttribute('text-anchor','middle'); dep.setAttribute('class','grafzone'); dep.style.opacity=1; }
})();
</script>
```
Tune `y:324` (Athlete) / `y:660` (Dependent) if either crowds the line at slider extremes — they sit
~28% down and ~14% up from the plot edges and stay clear of the curve at the default cap=40/risk=50.

### Other curve notes (judgment calls, not bugs)
- `FIXED=0.86` (L1158) vs the final design's `1` — both render the finished frame; negligible.
- Production hides `#cp-body` (the intro paragraph) on mobile (L2186); the final design kept it.
  Optional — restore by deleting that rule if you want the lede back.
- The red `58%` event marker is a newer curve feature, not a regression.

---

## 4. Experience section — 🟥 structural gap
**Production:** the designer's horizontal `.exp-rail` filmstrip force-stacked vertically (L2202–2214).
It works, but the panels read as loose stacked blocks, there's no numbered spine, and a now-dead tab
bar (`.exp-tabs`) sits at the bottom.

**Final design:** a numbered **`.xp` timeline** — one gestalt card per step (photo bleeds into the
card's rounded top, copy, then a tinted "detail" footer divided by a hairline), a connecting spine
with numbered nodes, an overview-pill header, and a gold-accented step 04. Same layout at all
breakpoints (max-width 780, centered).

**Assets:** all referenced photos (`assets/photos/exp-0[1-4]-*.jpg`) and logos
(`assets/logos/*.svg`) exist and are already used by the current filmstrip — paths below are correct
as-is (logos are local, not CDN).

### Integration steps
1. Delete the production filmstrip experience **markup** — `<section class="exp-section" id="experience">…</section>` (includes its inline `expRail`/tabs `<script>`).
2. Delete the filmstrip's experience **CSS** (the `.exp-*` rules — base block + the desktop sticky/325vh + the ≤900/≤600 stacking rules at L2202–2214). They become dead once the markup is gone.
3. Insert the `.xp` `<style>` + `<section>` below in its place.
4. Verify at 390 / 768 / 1440 and on a real phone.

### `.xp` CSS
```css
<style>
  .xp { background:#FAF7F2; color:#1D1B18; padding:clamp(64px,9vh,120px) clamp(24px,7vw,80px); }
  .xp-inner { max-width:780px; margin:0 auto; }
  .xp-head { margin:0 0 clamp(40px,6vh,68px); }
  .xp-eyebrow { display:block; font-family:var(--audit); font-size:12px; letter-spacing:.16em; text-transform:uppercase; color:#8A6A2F; }
  .xp-title { font-family:var(--voice); font-weight:400; font-size:clamp(30px,4.4vw,52px); line-height:1.04; letter-spacing:-.02em; margin:14px 0 0; }
  .xp-lede { font-family:var(--voice); font-size:clamp(16px,1.4vw,19px); line-height:1.55; color:#5A6378; margin:16px 0 0; max-width:56ch; }
  .xp-map { list-style:none; display:flex; flex-wrap:wrap; gap:8px; margin:28px 0 0; padding:0; }
  .xp-map li { display:inline-flex; align-items:center; gap:8px; font-family:var(--voice); font-size:14px; color:#1D1B18; padding:7px 14px; border:1px solid rgba(36,36,38,.16); border-radius:999px; white-space:nowrap; }
  .xp-map-num { font-family:var(--audit); font-size:11px; color:#8A6A2F; }
  .xp-steps { list-style:none; margin:0; padding:0; }
  .xp-step { display:grid; grid-template-columns:40px minmax(0,1fr); gap:clamp(16px,3vw,24px); padding-bottom:clamp(40px,6vh,56px); }
  .xp-step:last-child { padding-bottom:0; }
  .xp-rail { position:relative; display:flex; justify-content:center; }
  .xp-num { display:flex; align-items:center; justify-content:center; width:40px; height:40px; border-radius:50%; border:1px solid rgba(36,36,38,.22); background:#FAF7F2; font-family:var(--audit); font-size:13px; color:#1D1B18; position:relative; z-index:1; }
  .xp-step:not(:last-child) .xp-rail::after { content:""; position:absolute; top:40px; bottom:calc(-1 * clamp(40px,6vh,56px)); left:50%; width:1px; transform:translateX(-50%); background:rgba(36,36,38,.16); }
  .xp-step--result .xp-num { background:#8A6A2F; border-color:#8A6A2F; color:#FAF7F2; }
  /* gestalt: each step is ONE card (common region) */
  .xp-content { min-width:0; --xp-pad:clamp(18px,2.4vw,26px); background:#FFFDFA; border:1px solid rgba(36,36,38,.12); border-radius:16px; overflow:hidden; padding:var(--xp-pad); box-shadow:0 1px 2px rgba(36,36,38,.04), 0 18px 34px -26px rgba(36,36,38,.30); }
  .xp-media { margin:calc(-1*var(--xp-pad)) calc(-1*var(--xp-pad)) clamp(16px,2vw,20px); }
  .xp-media > img { width:100%; height:clamp(180px,27vw,240px); object-fit:cover; display:block; border-radius:0; }
  .xp-media.split { display:grid; grid-template-columns:1fr 1fr; gap:2px; }
  .xp-half { position:relative; min-width:0; }
  .xp-half img { width:100%; height:clamp(160px,25vw,220px); object-fit:cover; border-radius:0; display:block; }
  .xp-mtag { position:absolute; left:10px; bottom:10px; font-family:var(--audit); font-size:10px; letter-spacing:.1em; text-transform:uppercase; color:#fff; background:rgba(20,22,28,.55); padding:4px 8px; border-radius:4px; }
  .xp-tag { font-family:var(--audit); font-size:11px; letter-spacing:.12em; text-transform:uppercase; color:#8A6A2F; margin:0; }
  .xp-h { font-family:var(--voice); font-weight:400; font-size:clamp(22px,2.8vw,30px); line-height:1.08; letter-spacing:-.015em; margin:8px 0 0; }
  .xp-p { font-family:var(--voice); font-size:clamp(15px,1.2vw,17px); line-height:1.6; color:#5A6378; margin:12px 0 0; }
  .xp-detail { margin:clamp(16px,2vw,20px) calc(-1*var(--xp-pad)) calc(-1*var(--xp-pad)); padding:clamp(15px,1.8vw,18px) var(--xp-pad) var(--xp-pad); border:0; border-top:1px solid rgba(36,36,38,.10); border-radius:0; background:rgba(36,36,38,.025); }
  .xp-step--result .xp-content { border-color:rgba(138,106,47,.38); }
  .xp-step--result .xp-detail { background:rgba(138,106,47,.05); border-top-color:rgba(138,106,47,.18); }
  .xp-dl { display:block; font-family:var(--audit); font-size:11px; letter-spacing:.1em; text-transform:uppercase; color:#5A6378; }
  .xp-dl + * { margin-top:12px; }
  * + .xp-dl { margin-top:16px; }
  .xp-logos { display:grid; grid-template-columns:repeat(4,1fr); gap:8px; }
  .xp-logo { display:flex; align-items:center; justify-content:center; height:46px; border:1px solid rgba(36,36,38,.12); border-radius:10px; background:#fff; }
  .xp-logo img { max-width:60%; max-height:44%; object-fit:contain; }
  .xp-provider img { height:26px; width:auto; display:block; }
  .xp-stats { display:flex; gap:32px; flex-wrap:wrap; }
  .xp-stat-num { font-family:var(--voice); font-weight:500; font-size:26px; color:#1D1B18; line-height:1; }
  .xp-stat-num small { font-size:14px; font-weight:400; color:#5A6378; }
  .xp-stat-lbl { font-family:var(--audit); font-size:11px; letter-spacing:.04em; color:#5A6378; margin-top:6px; }
  .xp-markers { display:grid; gap:10px; }
  .xp-mk { font-family:var(--voice); font-size:14px; line-height:1.45; }
  .xp-mk b { font-weight:500; color:#1D1B18; }
  .xp-mk span { color:#5A6378; }
  @media (max-width:560px){
    .xp-step { grid-template-columns:32px minmax(0,1fr); gap:14px; }
    .xp-num { width:32px; height:32px; font-size:12px; }
    .xp-step:not(:last-child) .xp-rail::after { top:32px; }
    .xp-stats { gap:24px; }
  }
</style>
```

### `.xp` markup
```html
<section class="xp" id="experience" data-logo-tone="dark" data-logo-placement="hidden" aria-label="The Aleron experience">
  <div class="xp-inner">
    <header class="xp-head">
      <span class="xp-eyebrow">The Aleron experience</span>
      <h2 class="xp-title">Four steps to your plan.</h2>
      <p class="xp-lede">Onboarding, genetics, and bloodwork build your complete picture, each through a partner you already trust. Then your physician turns it into one plan for what matters next.</p>
      <ol class="xp-map">
        <li><span class="xp-map-num">01</span> Onboarding</li>
        <li><span class="xp-map-num">02</span> Genetics</li>
        <li><span class="xp-map-num">03</span> Bloodwork</li>
        <li><span class="xp-map-num">04</span> Your plan</li>
      </ol>
    </header>
    <ol class="xp-steps">

      <li class="xp-step">
        <div class="xp-rail"><span class="xp-num">01</span></div>
        <div class="xp-content">
          <figure class="xp-media"><img src="assets/photos/exp-01-onboarding.jpg" alt="Onboarding with phone and Apple Watch"></figure>
          <div class="xp-tag">Onboarding &middot; 15 minutes</div>
          <h3 class="xp-h">Onboarding takes fifteen minutes.</h3>
          <p class="xp-p">Demographics, family history, lifestyle, screening history, collected the way every other app on your phone does it. Then your wearable connects in two taps and starts streaming the signals your physician will actually use.</p>
          <div class="xp-detail">
            <span class="xp-dl">Connects with</span>
            <div class="xp-logos">
              <span class="xp-logo"><img src="assets/logos/apple.svg" alt="Apple Watch"></span>
              <span class="xp-logo"><img src="assets/logos/polar.svg" alt="Polar"></span>
              <span class="xp-logo"><img src="assets/logos/whoop.svg" alt="Whoop"></span>
              <span class="xp-logo"><img src="assets/logos/garmin.svg" alt="Garmin"></span>
              <span class="xp-logo"><img src="assets/logos/fitbit.svg" alt="Fitbit"></span>
              <span class="xp-logo"><img src="assets/logos/eightsleep.svg" alt="Eight Sleep"></span>
              <span class="xp-logo"><img src="assets/logos/dexcom.svg" alt="Dexcom"></span>
              <span class="xp-logo"><img src="assets/logos/withings.svg" alt="Withings"></span>
            </div>
          </div>
        </div>
      </li>

      <li class="xp-step">
        <div class="xp-rail"><span class="xp-num">02</span></div>
        <div class="xp-content">
          <figure class="xp-media"><img src="assets/photos/exp-02-genetics.jpg" alt="Genetics kit on a kitchen counter"></figure>
          <div class="xp-tag">Genetics</div>
          <h3 class="xp-h">A small kit arrives at your door.</h3>
          <p class="xp-p">Spit in a tube. Drop it in the prepaid mailer. The 163-gene Invitae panel screens for the variants that actually change a care plan: moderate-penetrance cancer risks, cardiovascular variants, pharmacogenomic markers.</p>
          <div class="xp-detail">
            <span class="xp-dl">Genetic testing by</span>
            <div class="xp-provider"><img src="assets/logos/invitae.svg?v=provider-logos-20260601" alt="Invitae"></div>
            <div class="xp-stats">
              <div><div class="xp-stat-num">163<small> genes</small></div><div class="xp-stat-lbl">Invitae panel</div></div>
              <div><div class="xp-stat-num">~3<small> wks</small></div><div class="xp-stat-lbl">Results return</div></div>
            </div>
          </div>
        </div>
      </li>

      <li class="xp-step">
        <div class="xp-rail"><span class="xp-num">03</span></div>
        <div class="xp-content">
          <figure class="xp-media split">
            <div class="xp-half"><img src="assets/photos/exp-03-bloodwork-location.jpg?v=split-h2-20260608" alt="Member leaving a Quest patient service center"><span class="xp-mtag">In person</span></div>
            <div class="xp-half"><img src="assets/photos/exp-03-bloodwork-athome.jpg?v=split-h2-20260608" alt="Member using an at-home capillary collection device"><span class="xp-mtag">At home</span></div>
          </figure>
          <div class="xp-tag">Bloodwork</div>
          <h3 class="xp-h">Bloodwork on your terms.</h3>
          <p class="xp-p">Draw the same Elite panel two ways: walk into your nearest Quest patient service center, or skip the trip with an at-home capillary kit that ships to your door. Either way you get a comprehensive metabolic, cardiovascular, and inflammatory workup, including the ApoB and Lp(a) markers most physicals miss.</p>
          <div class="xp-detail">
            <span class="xp-dl">Labs through</span>
            <div class="xp-provider"><img src="assets/logos/quest.svg?v=quest-logo-larger-20260601" alt="Quest"></div>
            <span class="xp-dl">The Elite panel includes</span>
            <div class="xp-markers">
              <div class="xp-mk"><b>Core panel</b> <span>CBC with differential, CMP, liver markers, kidney markers</span></div>
              <div class="xp-mk"><b>Cardiometabolic</b> <span>Lipid panel, ApoB, Lp(a), HbA1c, fasting insulin, uric acid</span></div>
              <div class="xp-mk"><b>Inflammation</b> <span>hsCRP, homocysteine, ferritin, iron/TIBC</span></div>
              <div class="xp-mk"><b>Thyroid + nutrients</b> <span>TSH, free T4, vitamin D, B12, folate</span></div>
              <div class="xp-mk"><b>Depth</b> <span>UACR and 80+ additional markers across metabolic, cardiovascular, inflammatory, renal, hepatic, and micronutrient domains</span></div>
            </div>
          </div>
        </div>
      </li>

      <li class="xp-step xp-step--result">
        <div class="xp-rail"><span class="xp-num">04</span></div>
        <div class="xp-content">
          <figure class="xp-media split">
            <div class="xp-half"><img src="assets/photos/exp-04-sync.jpg?v=risk-split-20260608" alt="Member on a live video visit with an Aleron physician"><span class="xp-mtag">Live visit</span></div>
            <div class="xp-half"><img src="assets/photos/exp-04-async.jpg?v=risk-split-20260608" alt="Member reviewing their game plan from home"><span class="xp-mtag">Async</span></div>
          </figure>
          <div class="xp-tag">Your plan</div>
          <h3 class="xp-h">A physician in your corner, on your schedule.</h3>
          <p class="xp-p">Aleron combines labs, genetics, wearable trends, records, and physician judgment into a domain-by-domain risk picture. Talk it through live on a video visit, or handle it asynchronously, message your care team and review your game plan whenever it suits you. The output is not a report. It is a plan for what matters next.</p>
          <div class="xp-detail">
            <div class="xp-stats">
              <div><div class="xp-stat-num">5<small> domains</small></div><div class="xp-stat-lbl">Risk models run</div></div>
              <div><div class="xp-stat-num">1</div><div class="xp-stat-lbl">Physician-led game plan</div></div>
            </div>
          </div>
        </div>
      </li>

    </ol>
  </div>
</section>
```
> The header copy ("The Aleron experience" / "Four steps to your plan." / the lede) was derived, not
> from a designer source — Jason may want to redline it. All step copy matches the current filmstrip.

---

## 5. Other sections — ✅ (designer's own responsive, not part of this work)
`howwedoit`, `founders-note`, `continuum`, `paths`, `site-footer` stack correctly on mobile via the
designer's existing queries; founder photo is wired. No contrast found.

---

## Priority
1. **Curve zone labels** (§3) — the visible regression Jason flagged. Small, self-contained.
2. **Experience `.xp` timeline** (§4) — larger, structural; the main thing missing.
3. **Hero per-slide framing** (§2) — optional polish.
