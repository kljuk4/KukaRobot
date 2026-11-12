# Pencil drawing physics for a robot with a spring-loaded (compliant) holder: force, darkness, breakage

Scope: numbers for a simulator model of contact force vs spring compression, line darkness vs force, and the force/angle at which a pencil lead or tip breaks. Every sourced fact has a link. Engineering derivations are under "Inferences" and are marked as estimates.

Source note: most primary sources (JIS S 6005 full text, Blau & Gardner 1996, Sause 2011, ISO 9177-3 span table) are paywalled or scanned. The ISO 9177-3:1994 preview was OCR'd and gives the minimum-strength table and the formula. Several important numbers (pencil-on-paper friction coefficient, darkness vs force curve, wooden-pencil core strength) have **no reliable source found**. They are flagged in Gaps.

---

## 1. Spring-loaded pen and pencil holders on robots and plotters (spring rate, preload, stroke, drawing force)

### Takeaway
Commercial plotters draw with very low pen force: about 0.1–0.45 N (gravity or a light spring). No published spring rate (N/mm) was found for AxiDraw or any robot drawing end effector. For a mechanical pencil, the hard upper limit of the holder is set by the lead clutch slip force, not by the spring.

### Cited Findings
- **Standard AxiDraw has no spring.** The pen is pressed down only by the weight of the floating Z stage plus the pen. The Z servo applies no downforce. Pressure is the same whatever the pen-down height setting (e.g. 55% vs 25%) — [Evil Mad Scientist forum](https://evilmadscientist.com/?p=20671); [EMSL forum, "Axidraw Spring?"](https://www.evilmadscientist.com/?p=30680)
- **AxiDraw maximum recommended pen weight: 1.6 oz (45 g).** That puts gravity-only pen force at about 0.44 N or less. A modest added weight is usually fine — [EMSL forum, "Servo weight limit?"](https://www.evilmadscientist.com/?p=28109)
- **AxiDraw brushless servo upgrade includes a spring kit (light and heavy spring) for ballpoints.** Its documented trade-off is reduced vertical travel. No spring rates or forces were published — [EMSL wiki, Brushless Servo](https://wiki.evilmadscientist.com/Brushless_Servo); [Bantam Tools product page](https://bantamtools.com/products/brushless-upgrade-axidraw)
- **HP 7475A pen plotter service manual: pen-down force 19 ± 10 g**, i.e. about 0.09–0.28 N — [ManualsLib, HP 7475A manual p.52](https://www.manualslib.com/manual/3361463/Hp-7475a.html?page=52)
- **AutoGraph graphite pencil plotter (hackaday) design rules:**
  - The force between graphite and paper controls marking darkness.
  - Spring force at full compression must be near, but no greater than, the friction force with which the mechanical-pencil clutch holds the lead. Above that, the lead slips back into the pencil.
  - Cheap mechanical pencils slip at low forces, which limits the darkest achievable mark.
  - A practical upper limit is the darkest mark achievable without deforming the paper.
  - The spring is modelled with Hooke's law (F = k·x).
  - Source: [hackaday.io AutoGraph project](https://hackaday.io/project/180956-autograph); the logs page now returns HTTP 410, so the specific numbers could not be retrieved.
- **Compliant pen mounts keep the line consistent despite Z errors and uneven surface height** — [Berkeley EDG 2023 Fall team 19 report](https://edg.berkeley.edu/wp-content/uploads/2023/12/2023Fall-team19.pdf); [robot-art end effector page](https://sites.google.com/view/robot-art/implementation/end-effector)
- **Mechanical pencil sliding-sleeve force (Zebra DelGuard / Kuru Toga test): up to about 1 N** — [bleistift.blog, Zebra DelGuard](https://bleistift.blog/2015/09/19)

### Inferences
- Model a spring holder as `F_n = F_preload + k·x` for 0 ≤ x ≤ stroke. Add `F_n = F_preload + k·stroke + k_hard·(x − stroke)` past bottom-out, with k_hard large (rigid robot). This is linear Hooke behaviour, per the AutoGraph note.
- **Plausible design range (ESTIMATE, not sourced):**
  - k ≈ 0.1–0.5 N/mm
  - preload ≈ 0.2–0.5 N
  - stroke ≈ 5–15 mm
  - working force ≈ 0.5–2 N
  - Basis: this range covers plotter forces (0.1–0.45 N) through handwriting forces (about 1 N; see section 2). A soft k makes the force tolerant to robot Z error: with k = 0.2 N/mm, a 2 mm Z error changes force by only 0.4 N.
- Hidden failure modes for a mechanical pencil in the holder, by force:
  1. Lead clutch slip, around 1 N for a sliding sleeve, otherwise unknown.
  2. Bottom-out of the spring, after which force rises steeply.
  3. Lead fracture (section 3).

  In a simulator, "lead pushed back" should be its own event, separate from "lead breaks".

### Gaps
- No published spring rate (N/mm), preload or stroke for AxiDraw springs, KUKA/UR drawing end effectors, or commercial compliant tool holders was found.
- No measured clutch slip force for 0.5/0.7 mm mechanical pencils, apart from the roughly 1 N sleeve force above.

---

## 2. Normal handwriting and drawing force, and line darkness/width vs force (HB, 2B…)

### Takeaway
Human writing force is about 0.6–1.2 N on average (axial), with peaks a few times higher. Darkness rises with force and with softness (B grades), but no quantitative darkness-vs-force curve was found in the literature.

### Cited Findings
- **Adults, ballpoint, cursive and simple patterns: mean axial pen force (APF) 83.6–115.3 g (about 0.82–1.13 N) across 6 conditions.**
  - Scribbles 83.6 g; wrist movement 87.5 g; circles 108.9 / 115.3 g; cursive 109.7 g; finger movement 113.5 g.
  - Cursive shows the largest variance (806 g², SD ≈ 28 g) and a rising time trend (+8 g/s).
  - The transducer range was 0–10 N.
  - Source: [Schomaker & Plamondon 1990, Biol. Cybern. 63:277–289 (preprint), Table 2](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf)
- **Average pen angle to the paper is about 50°; variation is only about ±2.5° (seldom more than 10°).** Axial and normal force correlated at r ≥ 0.96 — [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf)
- **Normal and axial force relation: F_N = F_A·sin(φ)**, where φ is the angle between the pen axis and the writing plane — [Wikipedia, Axial pen force](https://en.wikipedia.org/wiki/Axial_pen_force); [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf)
- **Ballpoint friction is about 4% of the normal force in the 0–2 N range** (Dooijes 1984, cited in [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf))
- **Grade 4 children (n = 74):**
  - Mean axial force 0.65–1.23 N (dynamic thumb 0.65 / 0.88 N; lateral thumb 0.96 / 1.23 N on two tasks)
  - Grip force 4.2–5.6 N
  - Source: [Schwellnus et al. 2013, AJOT 67(2):218–227](https://pmc.ncbi.nlm.nih.gov/articles/PMC3722657)
- **Axial pen force increases with processing demands** (Van Den Heuvel et al. 1998, Acta Psychologica) — [Wikipedia, Axial pen force](https://en.wikipedia.org/wiki/Axial_pen_force)
- **Darkness depends on force:** low force gives a light mark, heavy pressure gives a dark mark — [hackaday AutoGraph](https://hackaday.io/project/180956-autograph); [fiveable HB pencil](https://fiveable.me/drawing-foundations/key-terms/hb-pencil) (qualitative)
- **Softer grades (B) transfer more graphite and give darker lines than HB/H at the same force. A 2B gives a dark mark with less pressure than an HB.** These are retail/educational sources, so treat them as qualitative — [fiveable](https://fiveable.me/drawing-foundations/key-terms/hb-pencil); [Lazada stationery guide](https://stationery.lazada.com.ph/pencil-grades-decoded-what-hb-2b-and-4b-mean-and-which-to-choose-20260806/)
- **There is no industry standard for darkness per grade.** A Camel 2B was noticeably darker than a Staedtler Noris 2B — [blog.pencils.com](https://blog.pencils.com/?p=115296); [Gentleman Stationer](https://www.gentlemanstationer.com/blog/tag/2B+Pencils)
- **Wear-particle transfer to paper correlated with core hardness (2B/HB/2H) on 2 of 3 paper types** — [Blau & Gardner 1996, Wear 197:233–241 (abstract)](https://www.infona.pl/resource/bwmeta1.element.elsevier-ed3cf2f1-a89e-3cf0-9ba8-9861bf3817c2)
- **Graphite shows unusually large wear coefficients at moderate loads, and debris size is governed by paper roughness.** This is an unreviewed preprint — [Research Square rs-10448967](https://www.researchsquare.com/article/rs-10448967/v1)

### Inferences
- **Target normal force for robot pencil drawing (ESTIMATE):**
  - 0.3–0.5 N: light line
  - about 1 N: normal (matches human means)
  - 2–3 N: dark / heavy
  - Above about 3–5 N: likely paper embossing and frequent breaks of thin leads
- **Darkness model (modelling assumption, NOT sourced):** use a saturating curve, `D(F) = D_max(grade)·(1 − exp(−(F − F_0)/F_s))` for F > F_0, with
  - F_0 ≈ 0.1–0.2 N (threshold)
  - F_s ≈ 1 N
  - D_max increasing from H to B (e.g. relative D_max: 2H 0.5, HB 0.7, 2B 0.85, 4B 1.0; illustrative only)

  Saturation is justified qualitatively: paper valleys fill and darkness stops increasing.
- **Line width:** grows with the contact flat on the tip. A worn conical tip widens over time, roughly with deposited volume. Mechanical lead width is roughly the lead diameter.

### Gaps
- No peer-reviewed curve of optical density (or reflectance) vs normal force for HB/2B was found. Calibrate on the real setup: draw patches at 0.25, 0.5, 1, 2, 3 N and scan them.
- No adult pencil-specific (rather than ballpoint) force values were found, and no peak-force distributions.

---

## 3. Breaking force of graphite leads and wooden pencil tips

### Takeaway
ISO 9177-3 gives minimum 3-point bending strengths for HB mechanical-pencil leads:

| Lead | Minimum bending strength |
|---|---|
| 0.3/0.35 mm | 240 MPa |
| 0.5 mm | 190 MPa |
| 0.7 mm | 160 MPa |
| 0.9/1 mm | 95 MPa |
| 2 mm (ceramic) | 70 MPa |

Modern Japanese polymer leads are graded "A" at ≥ 430 MPa in patents, so real leads are often about 2× the ISO minimum. With 1–3 mm exposed, 0.5 mm leads snap from lateral loads on the order of 0.5–5 N.

### Cited Findings
- **ISO 9177-3:1994 Table 1, minimum bending strength σ of HB leads:**
  - Polymer (P): 0.35 mm 240 MPa; 0.5 mm 190 MPa; 0.7 mm 160 MPa; 1 mm 95 MPa
  - Ceramic (C): 2 mm 70 MPa
  - Note in the standard: 0.35 and 1 correspond to 0.3 and 0.9
  - Source: [ISO 9177-3:1994 preview (iTeh)](https://cdn.standards.iteh.ai/samples/20426/06434724f5dc482b8b49b8d5a29b1a2a/ISO-9177-3-1994.pdf)
- **ISO 9177-3 test method:**
  - Lead on two anvils (radius 0.2 mm), with the load applied midway between them
  - Loading rate 0.1 N/s for 0.35/0.5/0.7/1 mm and 0.5 N/s for 2 mm
  - σ is calculated from the breaking load F, the span l (Table 2, values not legible in the preview) and the actual diameter d. The OCR shows a d³ denominator, i.e. the 3-point formula σ = 8·F·l/(π·d³).
  - Source: [ISO 9177-3:1994 preview](https://cdn.standards.iteh.ai/samples/20426/06434724f5dc482b8b49b8d5a29b1a2a/ISO-9177-3-1994.pdf)
- **JIS S 6005:2019 (mechanical pencil leads) has a bending-strength clause (6.1).** It replaces the 2013 edition. The value table is not in the free preview — [JSA preview JIS S 6005:2019](https://webdesk.jsa.or.jp/preview/pre_jis_s_06005_000_000_2019_e_ed10_ch.pdf)
- **Japanese maker patents measure bending strength per JIS S 6005:2007 (mean of 100 samples).** Internal grading: graphite leads ≥ 430 MPa = grade "A"; BN (white) leads ≥ 250 MPa = "A". These are internal criteria, not standard minimums — [USPTO patent 9896591](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9896591); [USPTO patent 11179962](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/11179962)
- **Bending strength tends to increase with hardness (H > HB > B), but it varies notably between brands at the same hardness** — [Matsuo & Yashiro 2017, J. JSEM 17(1):21–26](https://www.jstage.jst.go.jp/article/jjsem/17/1/17_21/_article/-char/en)
- **Pencil lead elastic modulus E ≈ 10.5 GPa and density 1.78 Mg/m³** (forum post citing Sause; secondary source, use with caution) — [Physics Forums thread](https://www.physicsforums.com/threads/how-to-determine-the-breaking-force-of-pencil-lead.922486/)
- **Hobby test, 0.5 mm lead at 45°:**
  - Even a very long extension needed > 2 N to break.
  - A DelGuard is "virtually impossible to break" below about 1.5–1.6 mm extension.
  - Kuru Toga leads break more easily as extension grows.
  - The author admits the measurements are rough.
  - Source: [bleistift.blog, Zebra DelGuard](https://bleistift.blog/2015/09/19)
- **Pencil-lead break (Hsu-Nielsen) is a standard reproducible acoustic-emission source (ASTM E976).** Sause measured rupture forces for different diameters, free lengths and angles against an aluminium block. The force tables could not be retrieved — [Sause, J. Acoustic Emission 29 (2011), via typeset/scispace](https://typeset.io/pdf/investigation-of-pencil-lead-breaks-as-acoustic-emission-55szcvv9vz.pdf); [ASTM E976](https://softbank.iust.ac.ir/Railway/Codes/ASTM-standard/ASTM_2004_Standards/ASTM%202004%20PART%20IA/ASTM%202004%20Volume%2003.03%20Nondestructive%20Testing/E%20976%20-%2000%20%20_RTK3NG__.pdf)
- **Pencil-casing sharpening tests use a 20 N feed force, and casing bending strength is measured by 3-point bending.** This is not a tip-break value — [USPTO patent 8188175](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/8188175)

### Inferences (engineering estimates)
**Cantilever bending of the exposed lead.** The lead is clamped at the sleeve or wood edge and a lateral (perpendicular-to-axis) force F_⊥ acts at the tip over the exposed length L:

```
σ_max = 32·F_⊥·L / (π·d³)     →     F_⊥,break = σ_f·π·d³ / (32·L)
```

Computed F_⊥,break (N) at the ISO minimum σ_f, and in brackets at about 2× (≈ modern leads, ESTIMATE):

| Lead | σ_f (ISO min) | L = 1 mm | L = 2 mm | L = 3 mm | L = 5 mm |
|---|---|---|---|---|---|
| 0.3 mm | 240 MPa | 0.64 [1.3] | 0.32 | 0.21 | 0.13 |
| 0.5 mm | 190 MPa | 2.33 [4.7] | 1.17 | 0.78 | 0.47 |
| 0.7 mm | 160 MPa | 5.4 [10.8] | 2.7 | 1.8 | 1.08 |
| 0.9 mm | 95 MPa | 6.8 | 3.4 | 2.3 | 1.36 |
| 2 mm | 70 MPa | 55 | 27 | 18 | 11 |

- Cross-check: the hobby result (0.5 mm at 45° needed > 2 N even when long) implies σ_f is well above 190 MPa. That is consistent with the ≥ 430 MPa patent grades. Use σ_f ≈ 300–450 MPa for good-brand 0.5 mm HB, and apply about ±20–30% Weibull-like scatter.
- **Wooden pencil, sharpened cone (2 mm HB core typical; core strength ESTIMATE 50–100 MPa, by analogy to the 2 mm ceramic minimum of 70 MPa).**
  - For a cone of half-angle α, the diameter at distance x from the tip is d = 2x·tanα. The stress there is σ(x) = 4F_⊥/(π·x²·tan³α), which rises sharply toward the tip.
  - So a fresh sharp point always crumbles first and self-blunts to a small flat. This is wear, not a dramatic break, and should be modelled as a quick tip-radius increase.
  - A "snap" happens at the wood edge, where d ≈ 2 mm and L ≈ the exposed graphite length (typically about 4–8 mm, ESTIMATE). That gives F_⊥,break ≈ 70·π·8/(32·6) ≈ 9 N (range about 5–15 N).
  - A real sharpened point also has an internal crack or defect risk, so treat about 2–5 N lateral as "risky" for a long, needle-sharp 2B. B grades are weaker than H.
- **Axial buckling (Euler, fixed-free), 0.5 mm, E = 10.5 GPa:** P_cr = π²EI/(4L²) with I = πd⁴/64 = 3.07×10⁻³ mm⁴. That gives about 80 N at L = 1 mm and about 3.2 N at L = 5 mm. Pure axial load is rarely critical. Clutch slip (about 1 N) or bending governs first.
- In the simulator, combine both checks: break if `σ_bend + σ_axial ≥ σ_f`, where σ_axial = 4F_∥/(πd²) (small).

### Gaps
- No JIS S 6005 table values (paywalled).
- No ISO 9177-3 span (Table 2 illegible).
- No measured wooden-pencil core flexural strength per grade.
- No Sause force tables.
- No compressive (crush) strength of graphite-clay leads.

---

## 4. Pen tilt angle and friction: converting normal force into bending load

### Takeaway
At tilt θ from vertical, the lateral (bending) load on the tip is about F_n·(sinθ + μ·cosθ) when dragging away from the lean, and F_n·|sinθ − μ·cosθ| when pushing toward it. With μ ≈ 0.2–0.4 (ESTIMATE), even a vertical pencil carries 20–40% of F_n as lateral load from friction alone.

### Cited Findings
- **F_N = F_A·sinφ (φ = pen-to-plane angle)** — [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf); [Wikipedia](https://en.wikipedia.org/wiki/Axial_pen_force)
- **Humans write at about 50° pen-to-paper (i.e. about 40° from vertical)** — [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf)
- **Writing requires X/Y force components linearly related to the normal force (Coulomb friction)** (Deinet & Linke 1987, cited in [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf))
- **Blau & Gardner 1996:**
  - Static friction coefficients of 2B/HB/2H mechanical pencil cores on copier paper, manila and Tyvek were independent of grade and paper type (except Tyvek).
  - Kinetic friction showed no clear correlation with grade or paper.
  - Numerical values are not in the abstract.
  - Source: [Blau & Gardner, Wear 197 (1996)](https://www.infona.pl/resource/bwmeta1.element.elsevier-ed3cf2f1-a89e-3cf0-9ba8-9861bf3817c2); [CAS catalogue record](https://katalog.lib.cas.cz/KNAV/EdsRecord/edsair,edsair.doi...........52e339369f547a4bad1e5480dc17f325)
- **Graphite friction in general: about 0.1 for thin layers, rising to about 0.5 with layer thickness (ploughing)** (secondary review) — [KIT publication](https://publikationen.bibliothek.kit.edu/1000149412/149344332)
- **Graphite on graphite: static μ ≈ 0.1** (CRC handbook via secondary student page) — [hypertextbook](https://hypertextbook.com/facts/2004/NinaChen.shtml)
- **Hobby break test used a 45° angle** — [bleistift.blog](https://bleistift.blog/2015/09/19)

### Inferences
- **Geometry.** θ is the tilt from vertical; F_n is the normal force (from the spring); F_t = μ·F_n is friction, opposite to the drag velocity. Decompose relative to the pencil axis:
  - Lateral (bending): `F_⊥ = F_n·sinθ ± μ·F_n·cosθ` (+ when the pencil is "pulled", tip trailing; − when "pushed", tip leading, where friction and normal components partly cancel)
  - Axial: `F_∥ = F_n·cosθ ∓ μ·F_n·sinθ`
  - Spring note: if the spring acts along the pencil axis, the axial spring force is F_s = F_n/cos θ (roughly). For the same spring force, tilt lowers F_n and raises bending.
- **Worked example (ESTIMATE):** 0.5 mm lead, L = 2 mm, F_break,⊥ ≈ 1.2 N (ISO min) to 2.8 N (σ = 450 MPa). F_⊥ at F_n = 1 N, μ = 0.3:

  | θ (tilt from vertical) | F_⊥ (worst direction) |
  |---|---|
  | 0° | 0.30 N |
  | 20° | 0.62 N |
  | 40° | 0.87 N |
  | 60° | 1.02 N |

  At F_n = 2 N with θ = 40°, F_⊥ = 1.74 N, which already breaks ISO-minimum leads at L = 2 mm.
- **Risky angles:** for thin mechanical leads (≤ 0.5 mm), keep θ ≤ 15–20° from vertical (near-perpendicular) and exposed length ≤ 1–1.5 mm. Tilt above about 45° roughly triples bending load compared with vertical. Wooden pencils with long cones can be tilted 30–60° (as for shading) because the critical section is thick (2 mm), but the point crumbles faster.
- **Direction dependence:** pulling the pencil (tip trailing, like normal handwriting) adds friction to the sinθ bending. Pushing it (tip leading) causes stick-slip/chatter and digging into paper, which can produce impact loads. In the simulator, prefer trailing-tip motion and add an impulse factor for pushed strokes.

### Gaps
- No primary numeric μ for pencil graphite on paper was retrieved (Blau & Gardner tables are paywalled). Use μ_k ≈ 0.2–0.4 as an ESTIMATE, and measure it with a force sensor or an inclined-plane test on your paper.

---

## 5. Effect of drag speed and lateral friction on breakage

### Takeaway
No direct study links drawing speed to lead breakage. Friction (and therefore lateral load) is roughly proportional to normal force and only weakly dependent on speed. Speed matters mainly through dynamic force spikes: Z-compliance resonance, paper texture and stick-slip.

### Cited Findings
- **The pencil-drawing pattern depends on paper texture, pencil hardness, applied load and stroke speed** (preprint) — [Research Square rs-10448967](https://www.researchsquare.com/article/rs-10448967/v1); [INPHYNI / LOMA seminar summary](https://www.loma.cnrs.fr/bertin-24-march-2026/)
- **Pen-force signals contain extra power above 8 Hz, attributed to friction hysteresis and paper surface irregularities** — [Schomaker & Plamondon 1990](https://www.ai.rug.nl/~lambert/papers/pen-pressure.pdf)
- **Kinetic friction of pencil cores was not clearly correlated with grade or paper** — [Blau & Gardner 1996](https://www.infona.pl/resource/bwmeta1.element.elsevier-ed3cf2f1-a89e-3cf0-9ba8-9861bf3817c2)
- **ISO lead strength is measured at a slow 0.1 N/s (thin leads).** Breaking under fast, impact-type loading is not covered by the standard — [ISO 9177-3](https://cdn.standards.iteh.ai/samples/20426/06434724f5dc482b8b49b8d5a29b1a2a/ISO-9177-3-1994.pdf)

### Inferences
- **Simulator:**
  - Use quasi-static Coulomb friction F_t = μ_k·F_n, with μ_k constant (no speed term) as a first model.
  - Add a dynamic factor on F_n: `F_n,peak = F_n·(1 + c·v)` or a simple mass-spring-damper Z model. Spikes come from robot Z error at speed, approach velocity at touchdown, and holder resonance.
  - The touchdown impact is the most likely real breaker. Limit approach speed to about 5–20 mm/s for the last few mm (ESTIMATE).
- Sharp direction reversals in "push" direction (tip leading) are the highest-risk events (see section 4).
- Graphite brittle failure is a statistical event. Model a per-event break probability, e.g. Weibull with modulus m ≈ 5–10 (ESTIMATE, typical for brittle ceramics/composites; not sourced for pencil leads), rather than a hard threshold.

### Gaps
- No measured relation of speed to friction or break rate for pencils on paper.
- No Weibull modulus for pencil leads.

---

## 6. Simple engineering formulas for tip breaking force

### Takeaway
Use cantilever bending, `F_⊥,break = σ_f·π·d³/(32·L)`, with σ_f from ISO 9177-3 minima (190 MPa at 0.5 mm, 160 MPa at 0.7 mm, 70 MPa at 2 mm) or about 2× those for good modern leads. Combine it with the tilt/friction decomposition, `F_⊥ = F_n·(sinθ + μ·cosθ)`.

### Cited Findings
- **Standard 3-point lead bending test** (span l, central load F, diameter d), from which σ = 8Fl/(πd³) follows by beam theory — [ISO 9177-3:1994](https://cdn.standards.iteh.ai/samples/20426/06434724f5dc482b8b49b8d5a29b1a2a/ISO-9177-3-1994.pdf)
- **Strength values** (190/160/95/70 MPa minimums; ≥ 430 MPa graded "A" in patents) — [ISO 9177-3](https://cdn.standards.iteh.ai/samples/20426/06434724f5dc482b8b49b8d5a29b1a2a/ISO-9177-3-1994.pdf); [USPTO 9896591](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9896591)
- **E ≈ 10.5 GPa** (secondary source) — [Physics Forums](https://www.physicsforums.com/threads/how-to-determine-the-breaking-force-of-pencil-lead.922486/)
- **Forum posters note there is no consensus formula, because support inside the sleeve or clutch makes the effective clamp length uncertain** — [Physics Forums](https://www.physicsforums.com/threads/how-to-determine-the-breaking-force-of-pencil-lead.922486/)

### Inferences — compact simulator model (all parameters calibratable)

```
Inputs: spring compression x, preload F0, rate k, stroke s, tilt θ (from vertical),
        μ (0.2–0.4), lead d, exposed length L, σ_f, grade, drag direction sign ±1

F_n   = F0 + k·min(x, s) + k_hard·max(0, x − s)         # contact normal force
F_t   = μ·F_n                                            # friction
F_perp = |F_n·sinθ ± F_t·cosθ|                           # bending component
F_ax   = F_n·cosθ ∓ F_t·sinθ                             # axial component
σ     = 32·F_perp·L/(π·d³) + 4·F_ax/(π·d²)               # cantilever + axial
break if σ ≥ σ_f·W   (W ~ Weibull scatter, mean 1, m≈5–10 — ESTIMATE)
slip  if F_ax ≥ F_clutch (≈1 N for some pencils — measure)
darkness D = D_max(grade)·(1 − exp(−(F_n − F_th)/F_s))   (ASSUMPTION; calibrate)
```

- **Suggested defaults (ESTIMATES):**

  | Parameter | Value |
  |---|---|
  | σ_f, 0.5 mm HB | 300 MPa |
  | σ_f, 0.7 mm HB | 250 MPa |
  | σ_f, wooden 2B core 2 mm | 60 MPa |
  | σ_f, wooden HB core 2 mm | 75 MPa |
  | L, mechanical pencil | 1–2 mm |
  | L, wooden | 5 mm, effective d = 2 mm at the wood edge |
  | μ | 0.3 |
  | F_th | 0.15 N |
  | F_s | 1 N |
  | Working F_n | 0.5–1.5 N |

  Scale σ_f for grade (B weaker than H, per Matsuo & Yashiro).
- **Sanity numbers:**
  - 0.5 mm, L = 1.5 mm, σ_f = 300 MPa: F_⊥,break ≈ 2.45 N. At θ = 30°, μ = 0.3: F_⊥/F_n = 0.76, so F_n,break ≈ 3.2 N.
  - At L = 3 mm: F_n,break ≈ 1.6 N.

### Gaps
- The formula's effective L is uncertain (clamp compliance in sleeve/wood); calibrate by breaking real leads at known L and θ with a scale or load cell.
- No dedicated published model for sharpened wooden pencil tips was found.
