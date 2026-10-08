# Sports Venue Discovery App — Product & Technical Plan

> Working title: **Courtside** (placeholder — rename anytime)
> Status: Planning / pre-MVP. Constraints still being defined (see [Open questions](#10-open-questions-constraints-to-decide)).

---

## 1. The problem

You show up to a court, field, or gym and either:

- **Nobody is there** — you drove over to play pickup and end up shooting alone, or
- **It's packed** — you brought a full squad and now you're waiting three games for a run.

Moving to a new city or school makes it worse: you don't know where people play, when they play, or who plays there.

## 2. The solution (one sentence)

A map of every place to play sports, with a **live busyness meter** and **typical-busyness-by-hour chart**, plus a social layer — check-ins, highlights, events, and points — that both rewards users and *generates the busyness data itself*.

That last point matters: **the social features are not extras, they are the data engine.** Every check-in is a data point for the heat map.

## 3. Who it's for

| Persona | Need | Key feature |
|---|---|---|
| **New in town** (moved, transferred schools) | "Where do people play around here?" | Map + filters by sport |
| **Pickup player** | "Is anyone at the court *right now*?" | Live busyness meter |
| **Group organizer** | "When is the field empty enough for our 7v7?" | Hourly busyness forecast |
| **Competitive player** | "Show off, get recognized, find tournaments" | Highlights, leaderboard, events |
| **Organizer / league / gym owner** (later) | "Fill my tournament / promote my facility" | Event posting, verified venue pages |

## 4. Core features

### 4.1 Map & discovery
- Map view + list view of venues: parks, outdoor courts, indoor gyms, fields, rinks, tracks.
- Filter by sport (basketball, soccer, tennis, pickleball, volleyball, football, etc.), indoor/outdoor, free/paid, lights at night, surface type.
- Venue page: photos, amenities (lights, restrooms, water, parking), hours, number of courts/hoops/fields, rating.

### 4.2 Busyness meter + heat map (the hero feature)
- **Live meter**: Empty · Light · Moderate · Busy · Packed, with "last updated X min ago".
- **Hourly chart**: "Usually busy at 6 PM on Tuesdays" — like Google's Popular Times, per venue.
- **Heat map layer** on the map: color venues by current activity.

**Where the data comes from** (the hardest technical/product problem — plan for it on day one):

| Source | How | Pros | Cons |
|---|---|---|---|
| **Manual check-ins** | User taps "I'm here" (verified by GPS within ~100 m) | Simple, accurate, earns points | Depends on user count |
| **Crowd reports** | "How busy is it?" one-tap poll after check-in | Captures non-app users present | Subjective |
| **Auto check-in (opt-in)** | Geofence detects you've been at a venue 10+ min | Passive, lots of data | Battery, privacy, OS permission friction |
| **Third-party foot traffic** | e.g. BestTime.app API or similar | Data on day one, no users needed | Costs money, mostly covers businesses (gyms), weak for public parks |
| **Historical model** | Average past check-ins by day-of-week × hour | Fills the hourly chart | Needs a few weeks of data |

**Recommendation:** MVP = manual check-ins + "how busy?" one-tap report, with **check-ins expiring after ~90 min** (or on manual check-out). Busyness = active check-ins + recent crowd reports, weighted by recency. Add a third-party API for indoor gyms only if cold-start becomes a blocker.

> **Note on Google Popular Times:** it is *not* available through Google's official Places API. Don't plan around scraping it — it violates Terms of Service and breaks often.

### 4.3 Check-ins
- GPS-verified "I'm here" with optional sport + skill level + "looking for players" flag.
- Shows "4 people checked in · 2 looking for a game".
- Privacy: show counts by default; names only to friends / if user opts in.

### 4.4 Highlights
- Upload short clips (cap at ~30–60 s) and photos tagged to a venue.
- Venue page gets a highlight reel; user profile gets their own reel.
- Likes, comments, share-out to other socials (good for growth).

### 4.5 Events & tournaments
- Anyone can post a pickup run ("5v5 at Smith Park, Sat 10 AM, need 3"); verified organizers can post tournaments.
- RSVP, reminders, event chat (chat can wait until v2).

### 4.6 Points & progression
| Action | Points (draft) | Anti-abuse rule |
|---|---|---|
| Check in (GPS-verified) | +10 | Max 1 per venue per 3 hrs; must be within geofence |
| Report busyness | +5 | Only while checked in |
| Rate / review a venue | +15 | Once per venue (editable) |
| Upload a highlight | +20 | Daily cap; removed if reported/taken down |
| Add a missing venue (approved) | +50 | Only after moderator/community approval |
| Attend an event (checked in at event) | +25 | Must be at the venue during event time |
| Streak bonus | +5/day | Consecutive days with a check-in |

Points unlock **levels/badges** ("Court Regular", "Explorer — 10 venues", "Local Legend") and appear on **venue leaderboards** ("Top players at Smith Park this month"). Later: redeem for partner perks (gym day passes, gear discounts).

> Points directly create busyness data, so **cheating = bad data**. Location spoofing checks, rate limits, and caps are mandatory, not optional.

### 4.7 Social basics
Profile (sports, skill level, home courts), follow/friends, "notify me when my friends check in" or "when Smith Park gets active."

## 5. MVP scope (what to build first)

The goal of the MVP is to prove **one thing**: people will check in, and that check-in data makes the busyness meter useful.

**In the MVP**
1. Sign up / log in (email + Apple/Google sign-in)
2. Map + list of venues for **one launch city**, filtered by sport
3. Venue detail page
4. GPS-verified check-in / check-out + "how busy?" report
5. Live busyness meter + simple hourly chart
6. Points + 3–5 badges
7. Ratings
8. Basic profile

**Deliberately later**
- Highlights/video (storage, moderation, and cost make this a v2 feature)
- Events & tournaments
- Friends, follows, notifications, chat
- Auto check-in via geofencing
- Venue owner accounts, monetization

## 6. Recommended tech stack

Chosen for: one codebase for iOS + Android, beginner-friendly, cheap at small scale, and strong geo support.

| Layer | Choice | Why |
|---|---|---|
| Mobile app | **React Native + Expo** (TypeScript) | One codebase → iOS + Android; Expo handles builds, location, camera, push |
| Maps | **react-native-maps** (Google/Apple Maps) or **Mapbox** | Mapbox has better heat-map styling; Google is easier to start |
| Backend / DB | **Supabase** (Postgres + **PostGIS**, Auth, Storage, Realtime) | "Venues within 5 miles" is a one-line PostGIS query; built-in auth; realtime updates for live meter |
| Venue seed data | **OpenStreetMap** (Overpass API: `leisure=pitch`, `sport=*`, `leisure=sports_centre`, `leisure=park`) + Google Places for gyms | OSM is free and has great court/field data; Google Places fills in businesses |
| Video (v2) | Supabase Storage or **Mux / Cloudflare Stream** | Dedicated video services handle transcoding + streaming |
| Push notifications | Expo Notifications | Free, simple |
| Analytics | PostHog | Funnels: install → first check-in |

Alternative if you prefer Google's ecosystem: **Flutter + Firebase** (Firestore + GeoFire). Either is fine; pick one and commit.

## 7. Data model (first draft)

```
users        (id, username, avatar_url, sports[], skill_level, home_city, points, level, created_at)
venues       (id, name, type, sports[], location GEOGRAPHY(Point), address, indoor, free,
              has_lights, amenities jsonb, hours jsonb, source, source_id, verified, created_by)
check_ins    (id, user_id, venue_id, sport, looking_for_game bool,
              checked_in_at, checked_out_at, expires_at, lat, lng, accuracy_m)
busy_reports (id, user_id, venue_id, level 1-5, created_at)
venue_hourly_stats (venue_id, day_of_week 0-6, hour 0-23, avg_count, samples)   -- rebuilt nightly
ratings      (id, user_id, venue_id, stars 1-5, review, created_at)  UNIQUE(user_id, venue_id)
point_events (id, user_id, action, points, ref_id, created_at)        -- ledger; never just increment
badges       (id, name, rule) / user_badges (user_id, badge_id, earned_at)
-- v2
highlights   (id, user_id, venue_id, media_url, thumb_url, caption, status, created_at)
events       (id, organizer_id, venue_id, sport, title, starts_at, ends_at, capacity, type)
event_rsvps  (event_id, user_id, status)
follows      (follower_id, following_id)
reports      (id, reporter_id, target_type, target_id, reason, status)
```

**Live busyness (computed):**
```
score = active_check_ins
      + Σ busy_reports in last 45 min, weighted by recency
→ mapped to Empty / Light / Moderate / Busy / Packed relative to venue capacity
```

## 8. Roadmap

| Phase | Length (solo, part-time est.) | Deliverable |
|---|---|---|
| 0. Foundations | 2–3 wks | Learn Expo + Supabase basics, wireframes in Figma, pick launch city |
| 1. Map | 3–4 wks | Venues seeded from OSM, map + list + filters + venue page |
| 2. Check-ins & meter | 3–4 wks | Auth, GPS check-in, busy reports, live meter, hourly chart |
| 3. Points & profile | 2–3 wks | Point ledger, badges, ratings, profile |
| 4. Beta | 4+ wks | TestFlight / Play internal testing with 30–100 local players; measure check-ins/week |
| 5. v2 social | 6–8 wks | Highlights, events/tournaments, follows, notifications |
| 6. Growth & revenue | ongoing | Venue owner pages, sponsored tournaments, premium, partner rewards |

## 9. Risks & how to handle them

| Risk | Mitigation |
|---|---|
| **Cold start** — empty meter = useless app | Launch in **one** city/campus; recruit local leagues, school teams, and run clubs; seed with OSM venues; show "be the first to check in" prompts; consider third-party data for gyms |
| **Fake check-ins / point farming** | GPS radius + accuracy checks, rate limits, daily caps, mock-location detection, ledger-based points |
| **Safety & privacy** (real-time location of people, possibly minors) | Never show exact live locations of individuals; counts by default; friends-only visibility; minimum age 13 (COPPA), extra protections for under-18; block/report |
| **Video moderation & cost** | Delay to v2; length caps; automated moderation API + report queue; transcode/compress |
| **Venue data quality** | User-suggested edits + "add a venue" with approval; flag closed courts |
| **App store review** | Clear location-permission explanations; privacy policy; account deletion in-app (required by Apple) |

## 10. Open questions (constraints to decide)

Answering these will lock in scope, stack, and timeline:

1. **Budget** — $0 bootstrapped, or some monthly spend for APIs/hosting? (Apple dev account is $99/yr; Google Play is $25 once.)
2. **Timeline** — target date for a beta?
3. **Team** — solo, or friends who can design/code/market?
4. **Platforms** — iOS + Android from day one, or one first?
5. **Launch market** — which city or campus first?
6. **Sports at launch** — all, or start with 1–3 (e.g. basketball, soccer, pickleball)?
7. **Age range** — open to under-18s? (Big impact on privacy/safety requirements.)
8. **Monetization** — free with ads, premium tier, venue/organizer fees, sponsorships?
9. **Private gyms** — include paid/membership gyms, or public/free venues only at launch?
10. **Skill matching** — should players be able to filter by skill level ("casual" vs "competitive" runs)?

## 11. Immediate next steps

1. Answer the open questions above.
2. Sketch 5 screens (Map, Venue page, Check-in sheet, Profile, Leaderboard) — paper or Figma.
3. Pull OSM venue data for the launch city to see coverage.
4. Scaffold the Expo app + Supabase project and get venues showing on a map.
