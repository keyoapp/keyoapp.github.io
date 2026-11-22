# Keyo roadmap

Status as of 2026-11-16: the design system, skill path, lesson flow, result and level-up screens are built in
SwiftUI (iPhone + iPad) with the brand fonts, Dynamic Type (including the accessibility sizes), motion, sound
settings and snapshot tests. Five units: note reading, ear training, melody shapes, black keys, and rhythm with
two hands, each ending in a song you play along to. All data is on the device.

Phases run in order; items inside a phase can run in parallel. Each item says what "done" means.

---

## Phase 1 · Make the UI exact (polish) — mostly done

Done:
- **Real fonts.** Fraunces and Manrope are bundled (SIL OFL, `Keyo/Resources/Fonts`), with weights and
  Fraunces' optical size set through the variation axes (`KeyoFonts`). SF Symbols keep the system font.
- **Snapshot tests.** `KeyoTests` has 13 screen snapshots (iPhone, SE, iPad portrait/landscape, XXXL text,
  lesson states, result, level up, profile) plus contrast and lesson-rule tests. Run with ⌘U on the
  iPhone 17 simulator; re-record with `KEYO_RECORD_SNAPSHOTS=1`.
- **Motion.** Key press ripple, combo "3 in a row" float, streak/XP chip pop, lesson → result scale
  transition, level-up screen. Each has a Reduce Motion variant.
- **Sound.** Richer piano model (decaying harmonics, inharmonicity, hammer), 5 ms IO buffer, engine warmed
  at launch, Sound and Haptics toggles in Profile. Drop a `Piano.sf2` into the app target and it is played
  through `AVAudioUnitSampler` automatically.
- **Accessibility.** VoiceOver hints on path and piano keys, spoken right/wrong announcements, hint bubble
  read as one item, WCAG contrast test on every token pair (one design color lightened: text on violet).

Also done since:
- **Dark mode.** Adaptive color tokens (`KeyoColor`), with fixed tokens for things that must not change
  (piano keys, Keyo, dark text on bright fills). Follows the system, with a System/Light/Dark setting in
  Profile. The celebration screens keep their light cards on violet. Contrast is tested in both modes.
- **Pinned headers with the native blur.** Learn's chips and each tab's title stay put; on iOS 26 the
  system draws the gradient blur (a soft top scroll-edge effect), with no custom blur view.
- **Flame icon** for streaks (replacing the metronome).
- **Hearts.** See below.
- **Accessibility text sizes.** The XXXL cap is gone; text scales through AX5. On iPhone at AX sizes a lesson
  scrolls, the piano turns a quarter turn (low notes at the top, black keys on the left edge, each key a
  full-width bar), answer options go to one column, the hint waits behind a "Show hint" button and Check stays
  pinned at the bottom. The result screen scrolls with stacked tiles, and lesson card headers and feedback
  messages wrap. Snapshots: `lesson-ax3`, `lesson-wrong-ax5`, `result-ax3`, `learn-ax3`.

Left:
- **Sampled piano file (done 2026-11-16).** `Keyo/Resources/Piano.sf2` (9 MB, CC0 1.0; license in
  `Keyo/Resources/CC0-Piano.txt`). The `Keyo` folder is synced with the app target, so it is bundled
  automatically; `NotePlayer` plays its first melodic program through `AVAudioUnitSampler` instead of the
  built-in piano model.
- **Design match within 2 pt (done 2026-11-16)** against the Claude Design canvas "Keyo Design System"
  (iPhone path, lesson and result; iPad lesson), measured on renders at the artboards' size and safe areas
  (390 × 844 with 47/34, 1194 × 834 with 24/20). Shared fixes: bordered cards and the speech bubble now add their
  2 pt border outside the padding, as CSS does; display type is set at size + 6 with an exact line height on
  iOS 26 (Fraunces' own is about 1.33 × size). Lesson, result and Learn spacing now follows the artboards.
  Known exceptions, all on purpose or in the artboard itself:
  - iPad lesson: the artboard's keys start 7 pt above the bottom of Keyo's hint row; the app can't overlap them,
    so the keys sit 17 pt lower and Check stays where it is drawn.
  - Layouts changed on purpose after the artboards: the iPhone Learn header (wordmark and small chips), the
    native tab bar and iPad sidebar, the stacked lesson card on iPad, sentence-case buttons, hearts in the
    lesson bar. Their spacing was matched where the element exists in both.
  - Below iOS 26 display type keeps the font's natural line height (a few points taller per line).
  - The Phase 2 screens on the canvas were drawn from the app, so they match by construction.

### Hearts (mistake lives)

- Up to 5 hearts. The first wrong answer on a question costs one (retrying that question is free); the
  lesson ends after the feedback if the last heart goes.
- One heart comes back every 30 minutes, and the clock keeps running while the app is closed.
- Lessons never give hearts back (changed 2026-11-22; a perfect lesson used to earn 3). A lesson with no
  mistakes while the hearts are full earns a free **streak freeze** (you can hold up to 2).
- A streak freeze covers one missed day. Missing more days than you have freezes ends the streak and keeps
  the freezes. Shown on the streak cards in Learn and Profile.
- Hearts can be refilled with an in-app purchase (consumable `app.keyo.hearts.refill`, $1.99, bought through
  RevenueCat like Platinum: `Store/HeartsStore.swift`). The shop opens from the hearts chip on Learn, the out-of-hearts screen and the lesson card.
- Onboarding runs and the quick check never use hearts or hand out rewards.

Before this ships:
- Create the consumable in App Store Connect with that product ID at $1.99 (Tier 2), and add the same product
  ID in the RevenueCat dashboard (and to the Test Store while the SDK runs on a `test_…` key; until then the shop
  shows no price). It needs no entitlement or offering: it is fetched by product ID. `KeyoTests/Keyo.storekit`
  holds the same product at $1.99 for local StoreKit testing once the SDK is on the App Store key.
- RevenueCat finishes the transaction. A purchase approved later (Ask to Buy) is credited from the customer info
  stream: `RefillLedger` remembers that a refill is pending and which transactions were credited, so a refill
  is never counted twice and a reinstall doesn't credit the old purchase history again. The purchase rules and
  the ledger are unit-tested; the old `StoreKitHeartsTests` were removed with the direct StoreKit code. Still to
  try: a sandbox purchase on a real device, once the product exists in App Store Connect and RevenueCat.
- The backend should own `hearts`, `nextHeartAt` and `streakFreezes`, verify receipts, and apply the
  refill, so none of it can be changed on the device. Price and refill amount should be server-side config.
- Open design questions: earning hearts back by practicing, and whether a restore or subscription ("unlimited
  hearts") belongs next to the one-off refill.

## Phase 2 · Make the pages contentful — in progress

Done:
- **Curriculum.** Five units of eight lessons (40 lessons) in `Model/Curriculum.swift`, each ending in a
  song. Progress runs across units; a unit unlocks when the one before it is finished.
- **Black keys (Unit 4, "Sharps and flats").** `Pitch` has an `Accidental` (sharp/flat, black keys only).
  C♯ and D♭ are the same key (`isSameKey`) but spelled differently. Every on-screen keyboard's black keys are
  real answer keys with selected/correct/wrong/hint states, the staff draws ♯ and ♭, and naming questions
  mix black and white neighbors in the lesson's spelling. Ear lessons work on black keys too.
- **Rhythm tapping (Unit 5).** Hear a pattern (quarters, halves, eighths), then tap it back on a big pad that
  registers the moment a finger lands. Taps are lined up with the pattern on average, so starting a little
  early or late is fine. Each tap must land within 0.2 s, or less for quick eighths. The notation turns
  mint/coral per tap after checking.
- **Two hands (Unit 5).** Press two keys together (within 0.45 s). On iPad the keyboard spans C3–B4 and the
  card shows each hand's finger number (left pinky 5 on C3, right thumb 1 on middle C). On iPhone the same
  pairs are played with two thumbs in one octave, since each pair uses two different letters.
- **Song play-along.** Unit songs (Ode to Joy, Twinkle, Hot Cross Buns, Jingle Bells in F, Happy Birthday in
  3/4) scroll right to left past a playhead after a one-bar count-in. "Wait for me" (default) rests on each note
  until you play it and labels the keys. "Keep tempo" keeps going with a soft metronome and judges each note
  within 0.3 s. Accuracy and XP feed the usual result screen. `Features/Songs/PlayAlong*`.
- **Streak calendar with Rest Days.** Two Rest Days a month cover missed days first, then streak freezes. If
  they can't cover every missed day the streak ends and nothing is spent. A day log (saved with progress)
  drives a month calendar opened from Profile's streak card ("See calendar"), and covered days show sky blue
  in the week strip.
- **Sleepy Keyo.** Lessons less than 10 minutes apart count as one sitting. After 20 minutes, sleepy Keyo
  suggests a break once the result screen is done ("Take a break" / "One more lesson").
- **Learn.** All units stack on one scrolling path with a "2 of 8" progress pill each, a lock caption on
  units not yet reached, and it opens on the unit you are working in. Streak week and daily goal sit
  inline on iPhone and iPad portrait, with an "Up next" list on iPad portrait; landscape keeps the rail.
- **New lesson types.** Ear training ("tap the note you heard", no staff) and higher/lower (hear two notes).
  Notes play on their own when the question appears, and retrying an ear question replays it instead of
  highlighting the answer.
- **Locked units.** Pressing a key in a locked unit opens a full page: a strip of big piano keys you
  scroll through, with the lesson card below changing to match the key in the middle.
- **Songs tab.** Library of six songs (three unit songs, three coming soon) with a staff preview, difficulty,
  a "next song" progress card and a detail sheet. Unit songs start their song lesson.
- **Leagues tab.** Sample weekly league with promotion/demotion zones and your row ranked by this week's XP.
- **Profile.** Level progress, a 14-day XP chart against the daily goal, badges driven by real progress
  (tap one for what it is for, how to earn it and how close you are), settings.
- **Celebrations.** Level up, combo ("3 in a row"), streak chip pop (from Phase 1); result screen.
- **Grand staff for two hands.** Two-hand questions draw treble and bass staves joined by a brace, with the
  right hand's note on the treble staff and the left hand's on the bass staff in one column. On iPad each note
  has its finger number; beside the staff a row per hand lights up while its key is down, and a held note turns
  mint. At accessibility text sizes the larger hand cards stay. `GrandStaffView`, `TwoHandsCard`.
- **Play-along backing and speed.** Each bar gets a I, IV or V chord picked from the melody (the last bar is
  always I), played as a bass note on the downbeat and soft chords on the other beats. On by default, with a
  "Backing chords" toggle that is remembered. Keep tempo has a speed slider from 50% to 100% of the song's
  tempo; the 0.3 s timing window stays the same. `Features/Songs/Accompaniment.swift`.

- **Units 10 to 15 (2026-11-22).** Six more units with written plans in `LessonPlans.swift` (48 lessons, 120 in
  all), keeping the right hand on C4–B4 in lessons:
  10 Five-finger keys (D and E positions, major vs. minor by finger 3; song Scarborough Fair),
  11 Key signatures (G with F♯, F with B♭, D with two sharps; Minuet in G),
  12 Melody and bass (left-hand chord roots C, F, G under the tune; Oh! Susanna),
  13 Minor keys (A minor, the raised G♯, the E chord; Greensleeves),
  14 Quick notes (ties, syncopation, running eighths; Yankee Doodle),
  15 Recital pieces (learning in chunks, broken chords, steady tempo; Für Elise, opening section).
  Minor songs set `Song.isMinor`, so the backing plays i, iv and a major V.
- **More bonus songs (2026-11-22).** Row, Row, Row Your Boat; Brahms' Lullaby; Morning Mood; In the Hall of the
  Mountain King; Auld Lang Syne. All melodies are public domain (traditional, or composers who died over a
  century ago); the arrangements are Keyo's own. Amazing Grace is still "coming soon".

Left:
- **Empty, loading and error states.** Nothing reads from a network yet, so these arrive with the backend.
- **Real league content** needs accounts.

## Phase 3 · Onboarding — built

Try first, sign up later. `Features/Onboarding/` holds the flow; answers and progress are saved on the
device (`AppModel.load()`), so a relaunch picks up where you were.

Done:
1. **Welcome** with Keyo and "No account needed".
2. **Why are you learning?** (fun, a song, my child, music theory, something else.) Stored for later copy.
3. **Have you played before?** Beginner and "I know a little" go on; "I can read music" runs a six-question
   quick check. Passing (80% or better) starts you at Unit 2, otherwise Unit 1.
4. **Daily goal:** 10 / 30 / 50 XP, 30 selected.
5. **First win:** three questions from "Find middle C", then the result screen. Awards XP and starts the
   streak, and is not counted twice if onboarding is restarted.
6. **Reminder:** a time wheel, then the system notification prompt (only when you tap "Turn on reminders").
   Schedules one repeating local notification. "Not now" skips it.
7. **All set** summary, then the app. Back works on the question steps.
8. A new day: the 14-day history shifts, today's numbers reset, and a missed day ends the streak.

Left:
- **Sign in with Apple** and saving to an account. Needs the backend; progress is device-only until then.
- ~~**MIDI keyboard** step for iPad.~~ Dropped: no MIDI support. Real piano mode listens through the mic instead
  (Phase 5, P5), and its "Let Keyo listen" step takes this place. The iPad sidebar card is still a placeholder.
- **Goal and reminder settings (done).** Profile's Settings card shows the daily goal and reminder time; either
  row (or "Change goal" on the goal card) opens a sheet to pick 10/30/50 XP and turn the reminder on or off and
  set its time. Turning it on asks for permission if needed; if notifications are off for Keyo, it says so and
  links to the Settings app.
- **Rest Days** are local for now; the backend should own the day log and apply them at local midnight.
- **Reminder time follows the time zone** only when the app is opened. Server-driven reminders come later.

## Subscriptions (RevenueCat) — wired up, not yet configured for release

Keyo Platinum runs through RevenueCat (`Store/SubscriptionStore.swift`, `Features/Profile/PlatinumCard.swift`).
`SubscriptionStore` configures the SDK, keeps the latest `CustomerInfo` (via `customerInfoStream`) and exposes
`isPlatinum` for the `keyo_platinum` entitlement. Profile shows a card: **See Platinum** opens Keyo's paywall; once subscribed it shows the renewal date and **Manage subscription** opens Customer Center.

Before this ships:
- The SDK key in `SubscriptionStore.apiKey` is a **Test Store** key (`test_…`): purchases are simulated. Replace it
  with the App Store key (`appl_…`) and create `monthly` and `yearly` in App Store Connect, with a 3-day free trial
  (introductory offer) on `yearly` only.
- In the RevenueCat dashboard: products `monthly` and `yearly`, entitlement `keyo_platinum` with both attached, a
  current offering with `$rc_monthly` and `$rc_annual` packages (the code no longer reads `$rc_weekly`). The paywall is Keyo's own SwiftUI view
  (`Features/Platinum/PlatinumPaywallView.swift`) reading that offering, so no dashboard paywall is needed. Its perk
  list names only shipped features (updated 2026-11-21). Still missing: the Privacy Policy link next to Terms
  (there's no policy URL yet).
- What Platinum unlocks is decided: see [PLATINUM.md](PLATINUM.md) and Phase 5. Units 3 to 15 are gated on it
  (`AppModel.needsPlatinum`), along with the Phase 5 features.
- Plans: the app now shows monthly and yearly (weekly is gone). See PLATINUM.md, "Plans and pricing".
- When accounts arrive, call `Purchases.shared.logIn(userID)` so purchases follow the account.
- The hearts refill now goes through RevenueCat too (2026-11-21), so only RevenueCat finishes transactions.

## Phase 4 · Backend

**Stack: Supabase** (decided 2026-11-20): Postgres, Auth with Sign in with Apple, row-level security, Edge
Functions, Realtime and `pg_cron`. There's no separate server. The backend code is SQL migrations, RLS policies
and a few Postgres or Edge Functions, all in the repo.

### Decided (meeting, 2026-11-20)

- **The server owns anything that earns or unlocks.** XP, hearts, streaks, league standings and Platinum content
  are written only by server functions (for example `complete_lesson(lesson_id, answers, duration)`, which checks
  the run is plausible and awards the XP). The app never writes these columns directly; RLS blocks it.
  Cosmetic, on-device perks (piano sounds, outfits, keyboard skins) stay gated on the client. A cracked app on a
  jailbroken device can unlock those, and we accept that.
- **Anti-cracking.** Platinum is trusted only from the server: a RevenueCat webhook (an Edge Function) writes
  `is_platinum` and `platinum_expires_at` to the user's row, and RLS checks that column. Server-delivered content
  (bonus and monthly songs, future packs) is served only when that column is true. App Attest on
  `complete_lesson` once leagues go live.
- **Profiles are private.** There's no public profile page. Members of the same league can see each other's
  **league card**: username, avatar, level, streak, this week's XP and badges. No lesson history or personal
  details. One RLS rule: "read the league card of anyone in your current league."
- **Usernames and avatars.** Learners pick a username and choose from a small set of built-in person avatars.
  No photo uploads. The username field has a basic profanity check in the UI. The same check runs again in the
  database (a check constraint or trigger), because a UI check alone can be bypassed. App Review Guideline 1.2
  (user-generated content) also needs **Report** and **Block** on the league card, and a way to rename an
  offending username.
- **Leagues ship in v1, with practice rivals.** Until there are enough learners, the server fills each league of
  about 30 with simulated "practice rivals": generated usernames and avatars, and realistic weekly XP curves
  (daily regulars, midweek drop-offs, one strong rival), run by the `pg_cron` league job. Real learners go in
  first; rivals only top a league up, so they fade out as the app grows, with no app update.
  - They're labeled: a small "practice rival" tag on their card and a line in the league info sheet ("Keyo fills
    leagues with practice rivals until there are enough learners near you").
  - Nothing is sold against them (no "buy hearts to beat …"). Promotion and demotion against rivals are fine.
  - The App Review notes say: "Leagues include clearly labeled simulated participants until the user base grows."
  - Later: rivals replay real past weeks as anonymized "ghost" curves instead of made-up ones.

**Built in the app (2026-11-20), on the device until the backend exists:**
- `Model/Identity.swift`: the username rules (3–20 characters; letters, numbers, spaces, `_` and `.`), a basic
  profanity check (look-alike characters, spacing and repeated letters are seen through; short words are
  blocked only as whole words, so "Essex" and "Hancock" pass), names reserved for the team ("Keyo", "Admin"…),
  and 8 built-in person avatars (`PersonAvatarView`, drawn in code). Username, avatar, blocks and reports are
  saved with progress. Tests: `IdentityTests`.
- Profile: a name and avatar card at the top opens the editor, which checks the name as you type.
- Leagues: avatars in every row, a "Practice rival" tag, and a "Your league card" prompt until a name is picked.
  Tapping a row opens their league card (place, weekly XP, level, streak, badges). Learners can be **reported**
  (offensive username, cheating, something else) or **blocked** (name and avatar hidden; they keep their
  place). Practice rivals can't be reported and explain what they are. Your own card links to the editor.
- Debug: `-KeyoUsername`, `-KeyoAvatar`, `-KeyoIdentity 1`, `-KeyoLeagueMember <place>`.

### Wired to Supabase (2026-11-20)

Project `vwcvcsjyjnollpelbxsf`. The app talks to it through `supabase-swift` (2.x); the publishable key is in
`Store/Backend.swift` (`BackendConfig`), which is fine because RLS protects the data. Never ship the `service_role` key.

- **Accounts: Sign in with Apple only** (decided 2026-11-20; email codes and links were dropped). Native flow:
  `AppleSignInButton` asks Apple for an identity token with a hashed nonce, and `Backend.signInWithApple` hands
  it to Supabase (`signInWithIdToken`), which makes the account. No anonymous sessions. Onboarding stays
  try-first: lessons work without an account and progress stays on the device. The button is on Profile →
  Account and on the Leagues prompt. Signed in, Profile shows "Signed in with Apple", Sign out and **Delete
  account** (App Store guideline 5.1.1(v)). Entitlement: `Keyo.entitlements` (`com.apple.developer.applesignin`).
- **Database:** `supabase/migrations/20261120120000_accounts_and_leagues.sql`: `profiles` (made by a trigger when
  an account is created), `lesson_completions`, `xp_events`, `leagues`, `league_members`, `blocks`, `reports`, all
  with RLS. Functions: `username_problem` (the same rules and word lists as `Identity.swift`; keep them in sync),
  `set_identity`, `complete_lesson` (the server decides the XP: 10, songs 20, +5 perfect, replays 3; at most 30
  lessons in 10 minutes; a resend with the same client id counts once), `report_member`, `league_cards`
  (only league-card fields of your league this week), `delete_account`.
- **Sync (`Backend`):** at sign-in and whenever the app comes to the front: the username and avatar, queued
  lessons (`AppModel.pendingCompletions`, saved, sent oldest first), and this week's league (shown instead of the
  sample once you're in one). Reports and blocks of real members go to the server as they happen.
- Tests: `IdentityTests` (queueing, relaunch, real vs sample members, server league).

Not done yet:
- **League assignment and practice rivals:** nothing puts learners into `leagues` yet, so everyone sees the sample
  league. Next migration: a weekly `pg_cron` job that groups learners by tier, tops leagues up with rivals and
  moves rival XP along their curves, plus promotion and demotion.
- **XP from before the account** isn't sent (it can't be checked). The server's XP starts at sign-up; the app
  still shows the device's total.
- Streaks, badges, daily goal and reminders aren't on the server yet (`league_cards` shows 0 for streak and badges).
- RevenueCat webhook for `is_platinum`, App Attest, and `Purchases.logIn(userID)` at sign-in.
- Renaming an offending username after a report (an admin action; for now in the dashboard).
- **Revoke the Apple token when an account is deleted** (Apple requires it for Sign in with Apple). Needs an
  Edge Function holding the Sign in with Apple key (.p8) to call Apple's revoke endpoint; `delete_account` only
  deletes the Supabase user today.
- Supabase: turn off the Email provider (Authentication → Providers), since the app no longer uses it.

1. **Data model**
   - `profiles` (username, avatar id, goal, daily goal, level, timezone, reminder time, onboarding answers,
     `is_platinum`, `platinum_expires_at`)
   - `blocks`, `reports` (who blocked or reported whom, and why)
   - `units`, `lessons`, `exercises`: curriculum as versioned JSON, so content ships without app updates
   - `lesson_completions` (accuracy, duration, XP)
   - `xp_events`: append-only; XP only goes up
   - `streaks`: computed server-side in the user's timezone, with Rest Days (port `AppModel.rollOver` and `dayLog`)
   - `badges`, `user_badges`
   - `leagues`, `league_members`: weekly cohorts of about 30, promotion and demotion; members are learners or
     practice rivals (`is_rival`, with a generated XP curve)
2. **Server logic (Postgres functions, Edge Functions, `pg_cron`):** validate lesson completions and award XP
   (anti-cheat), roll streaks over at local midnight, run the weekly league reset and fill leagues with practice
   rivals, award badges, take the RevenueCat webhook, and check usernames for profanity.
3. **Client data layer**
   - A `Repository` protocol, so views stay unchanged.
   - Offline-first: SwiftData cache plus a sync queue for completions made offline.
   - Replace the in-memory `AppModel` sample data.
4. **Auth:** Sign in with Apple only (built). No anonymous sessions: progress stays on the device until an
   account is made.
5. **Notifications:** local reminders first (sleepy Keyo, one a day); APNs later for league events.
6. **Monetization:** Keyo Platinum through RevenueCat (decided, see PLATINUM.md). Every lesson is free (since 2026-11-20); the
   paywall shows after the onboarding "First win". The backend should own hearts and streak repair for
   Platinum learners.
7. **Ops:** staging and production projects, migrations in the repo, analytics events for the funnel
   (onboarding steps, lesson start/finish, day-1/day-7 retention).

## Phase 5 · Platinum features — first pass built

The full spec, with the free/Platinum split, is in [PLATINUM.md](PLATINUM.md). Everything runs on the device,
alongside Phase 4. Each feature is gated on `SubscriptionStore.isPlatinum` and added to
`PlatinumPaywallView.heroes` or `.extras` only when it ships.

Built (2026-11-18):
- **Gating basics.** `AppModel.isPlatinum` mirrors `SubscriptionStore.isPlatinum` (set in `KeyoApp`; debug override
  `-KeyoPlatinum 1`). (Units 3+ used to need Platinum; since 2026-11-20 every lesson is free.) On Platinum, hearts aren't used (lessons run with `hearts: nil`
  but still earn rewards, the chip shows ∞), and perfect lessons earn streak freezes up to 5. Freezes held when
  Platinum ends are kept. Rule tests: `PlatinumTests`. Still to add: snapshot tests of the Platinum states.
- **P1 · Piano sounds.** `PianoSound` (piano, electric piano, soft synth, 8-bit; music box was dropped) in `Theme/Feedback.swift`.
  The piano uses `Piano.sf2`; the other three are drawn by the built-in synth (a recipe of six partials each), so
  no extra sound files or licenses are needed. Picked in Profile → Settings → Piano sound
  (`PianoSoundSettingsView`): tapping plays a sample, and a Platinum sound without Platinum opens the paywall.
  Keys and ear-training prompts use the chosen sound; chimes, fanfares, the metronome and song backing stay on
  the piano. Without Platinum it goes back to the piano. Still to check: how each sound feels on a real device.

Built (2026-11-18), first pass; everything below runs on the device, gated on `AppModel.isPlatinum`, with rule
tests in `KeyoTests/PlatinumFeatureTests.swift`:
- **P2 · Song practice tools.** In the play-along's start panel (`PlayAlongView.practicePanel`): Right or Left
  hand (left plays the bass line, one note a bar on a bass staff, while Keyo plays the melody), loop any range of
  bars with a one-beat lead-in, and in Keep tempo "speed up 5% after each clean pass". Practice runs loop until
  Stop and earn no XP. Every run is recorded (`PlayAlongSession.recording`) and can be played back.
  Not done: sharing a recording as a video or audio file.
- **P3 · Keyo Coach.** `Model/Coach.swift`. Every lesson's first tries are logged (`AppModel.answerLog`, 90 days,
  saved, logged for free learners too). Profile → Weak spots: a keyboard colored by miss rate, with All /
  Reading / Ears filters, tap a key for "right 4 of 10". Learn → Keyo Coach: Practice Mix (10 questions, due keys
  first, Leitner boxes 0–4 waiting 0/1/2/4/8 days) and sight-reading (12 fresh questions, 4 levels from recent
  reading accuracy; black keys from Unit 4). Coach runs use lesson ids 10 000+, so they never move the path.
- **P4 · Improvise → Free play (2026-11-19).** Now a plain piano for free practice, free for everyone: no backing band,
  scale or chords. A studio-style control bar (record, play back, stop; a metronome with beat lights, 40–200 bpm
  and 4/4 or 3/4), a navigator strip for the whole range C2–C8 to drag the two-octave keyboard along, and a
  "Your melody" staff on iPad (takes recorded with the metronome are written at its tempo). On iPhone it always opens in
  landscape and turns back to portrait on close (the staff shows on iPad). Was: Songs → Improvise (`Features/Improvise`). Key (C, G, F major, A minor), style (calm, pop,
  waltz) and tempo; a I–IV–V–I loop. Keys outside the scale are faded (`PianoKeyState.dimmed`) and play quietly,
  the chord's notes glow (`.glow`). Record a take, play it back over the chords, and see it on a staff ("Your
  melody"). Made free for everyone on 2026-11-19 (no longer Platinum). Turns to landscape on iPhone (the only
  screen that does: `OrientationLock`), with two octaves and the controls in one row.
- **P5 · Real piano mode.** `Features/RealPiano`. Profile → Settings → "Eavesdropping" (asks for the
  microphone; `NSMicrophoneUsageDescription` is set). While on, lessons and play-along listen: YIN pitch detection
  (`PitchDetector`, C2–C7) and an onset tracker turn notes into key presses (`LessonSession.hear`,
  `PlayAlongSession.hear`). Heard keys check themselves (name-the-note questions pick the matching name);
  rhythm questions take note starts as taps; higher-or-lower is still answered on screen. A right answer moves on
  after a countdown ("Auto-continue": off, 2, 3 (default) or 5 seconds, under the switch). Notes Keyo
  itself played in the last second (and their octaves) are ignored; headphones are recommended. A "Listening"
  chip with a level meter shows in lessons and songs. No MIDI.
  Known limits: one note at a time, so two-hand questions need the two keys played within 0.45 s rather than
  truly together; not yet tried on a real acoustic or digital piano; detection runs on the main thread.
- **P6 · Smaller perks.**
  - Streak repair: a streak lost in the last 48 hours can be repaired once a month (`AppModel.repairStreak`;
    the gap days count as frozen). The card shows to everyone; free learners get the paywall.
  - Up to 5 streak freezes (from the gating basics).
  - Keyo outfits (bow tie, headphones, crown, glasses) and keyboard colors (mint, sky, sun, lilac), in
    Profile → Settings → Outfit and keys (`LookSettingsView`), passed down as `\.keyoOutfit` and `\.keyboardSkin`.
    Skins pass the contrast test.
  - Legendary: a finished unit offers "Go Legendary", two questions from each lesson with no hints; a perfect run
    wins a gold crown (`AppModel.legendaryUnits`). Not done: the timer from the spec.
  - Keyo Wrapped: a Profile card with the month in numbers (days played, questions, accuracy, most improved key,
    streak, crowns), shown for last month during the first week; Share renders a PNG card.
  - Bonus songs: Mary Had a Little Lamb, Frère Jacques and London Bridge, Platinum only, played from Songs.
    "New songs every month" needs someone to keep adding them (or the backend's content pipeline).

Also (2026-11-19): playing a lesson you already finished earns 3 XP (`AppModel.replayXP`), shown as "+3 XP" on its
card.

Still to do for Phase 5: snapshot tests of the Platinum screens, VoiceOver and Reduce Motion passes on the new
screens, and trying real piano mode on real pianos.

Alongside: change the plans to monthly and yearly, set up the paywall moments (after "First win", out of
hearts, locked content, day 7 of a streak), and add analytics events for the paywall and feature use.

## Decisions needed before Phase 4

- ~~Backend stack.~~ Decided 2026-11-20: Supabase (see Phase 4, "Decided").
- Who writes the curriculum, and in what format (JSON in the repo vs. an admin tool).
- **Lessons that teach (2026-11-20).** Path lessons follow written plans in `Model/LessonPlans.swift`: teach
  (Keyo explains, demonstrates, then lights the keys for "your turn"), practice, play short phrases, review, and
  preview the next unit song. New exercise types: `.teach` (ungraded), `.findKey` and `.phrase` (staff, letters
  or echo by ear; a wrong note stops the phrase and names it). Generated drills remain for the quick check, coach
  and the practice part of Units 4–5.
- ~~Free vs. paid content split.~~ Decided 2026-11-20: all lessons are free; Platinum sells practice tools (see PLATINUM.md).
- ~~Whether leagues ship in v1 or later.~~ Decided 2026-11-20: v1, with labeled practice rivals until there are
  enough learners (see Phase 4, "Decided").
- Which avatars to draw for the avatar set, and how many (a small set of person avatars).
- ~~Where extra piano sounds come from.~~ Decided: drawn by the built-in synth, so no extra files.
