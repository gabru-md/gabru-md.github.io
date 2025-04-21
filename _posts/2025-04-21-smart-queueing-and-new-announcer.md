---
layout: post
title: "Evolving the Announcer System — Smarter Queueing + Voice Switching"
date: 2025-04-21
tags: [typescript, web-audio, code]
---

## Evolving the Announcer System — Smarter Queueing + Voice Switching

Not long ago, I added a fun little **announcer voice** feature to my tournament app—things like “Get ready to fight!” or “You have won the game!” would play during key moments, just to give the whole thing a bit more energy.

It worked great! But like all things in dev... there was room for improvement.

---

## What Was Bugging Me

The first version had one major issue:  
- If you triggered multiple announcer lines back-to-back (which happens a lot in a tournament), they’d **overlap**. Two or more clips could end up playing at the same time, making it messy and hard to hear.

---

## Queued Playback to the Rescue

To fix that, I added a **promise-based audio queue**. Now, every sound clip waits its turn.

```ts
private static audioQueue: Promise<void> = Promise.resolve();
```

Then, in the `play()` method, each sound just chains itself onto that queue:

```ts
play(): void {
    if (!isAppSettingEnabled("enable-my-announcer")) return;

    Sounds.audioQueue = Sounds.audioQueue
        .then(() => this.playSound())
        .catch(err => console.error('Error playing sound:', err));
}
```

Simple, predictable, and no more overlapping chaos. 🎧

---

## Bonus Side Quest: Voice Variants

While fixing that, I got a little distracted (as one does) and thought:  
**"What if there were different announcer voices?"**

So I added that too. Right now, it toggles between *male* and *female* announcers using a feature flag:

```ts
private readonly announcer: string = isFeatureEnabled('FemaleAnnouncer') ? 'female' : 'male';
```

Then when playing a sound:

```ts
const url = `${Sounds.ANNOUNCER_PATH}${this.announcer}/${this.fileName}`;
```

Now I can switch announcers just by flipping a flag. I can now drop in other languages or styles without changing anything else. 

---

## Example Use

```ts
Sounds.TournamentHasBegun.play();  // Queued, voice-controlled, pitch-perfect
```

---

## Recap: What Got Better

- Sounds now play **one at a time**, no overlap
- Added support for **voice switching** (currently male/female)
- Still uses Web Audio API and pitch tweaks for cool factor

---

I’m pretty happy with how it turned out. The announcer system still feels lightweight, but it adds a ton of polish—and now it's way more flexible.  
Also: side-quests like adding voice variants? Totally worth it.

