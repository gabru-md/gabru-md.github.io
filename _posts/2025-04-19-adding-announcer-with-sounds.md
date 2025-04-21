---
layout: post
title: "Creating an Announcer System"
date: 2025-04-19
tags: [typescript, web-audio, code]
---

## Creating an Announcer System

While working on my latest side project—a tournament app—I had this idea: *"What if there was an announcer hyping things up?"* You know, moments like “Get ready to fight!” or “You have won the game!” just feel so much more exciting when someone *says* them.

So I put together a little TypeScript setup to handle announcer-style voice clips, and it ended up being super handy. Here's a quick breakdown of how it works and how you can do something similar.

---

## What I Wanted

The goal was pretty simple:

- A clean way to reference announcer sound clips (without repeating file names everywhere).
- A way to turn the whole feature on or off with a single flag.
- Slight pitch control to make things sound a little more dynamic.

---

## Setup

I created a class called `Sounds` that holds all the announcer lines as static members. Each one is basically just tied to a specific audio file.

```ts
import {isAppSettingEnabled} from "./settings/AppSettings.ts";

export default class Sounds {
    private constructor(public fileName: string) {}

    static readonly GetReadyToFight = new Sounds("get_ready_to_fight.mp3");
    static readonly SadToSeeYouGo = new Sounds("sad_to_see_you_go.mp3");
    static readonly TournamentHasBegun = new Sounds("tournament_has_begun.mp3");
    static readonly TournamentIsOver = new Sounds("tournament_is_over.mp3");
    static readonly WaitingForNextRound = new Sounds("waiting_for_next_round.mp3");
    static readonly WaitingForTheNextMatch = new Sounds("waiting_for_the_next_match.mp3");
    static readonly YouHaveWonTheGame = new Sounds("you_have_won_the_game.mp3");
    static readonly YourOpponentHasWonTheGame = new Sounds("your_opponent_has_won_the_game.mp3");
}
```

This gives me a bunch of reusable constants I can call from anywhere. No magic strings, no duplication.

---

## Playing a Sound

Want to play a sound? Just call the `play()` method like this:

```ts
Sounds.TournamentHasBegun.play();
```

That's it. Dead simple.

---

## Feature Flag

I didn’t want the announcer blasting during every test or user session. So I added a little feature flag check inside the `play()` method:

```ts
play() {
    if (!isAppSettingEnabled("enable-my-announcer")) {
        console.log('Announcer is disabled');
        return;
    }
    const url = `./assets/sounds/announcer/${this.fileName}`;
    this.playPitchAudio(url).catch(err => console.error(err));
}
```

This way, I can toggle the whole announcer feature from my app settings—super useful for different environments or user preferences.

---

## Tiny bit of Audio Magic

To actually play the sound, I used the Web Audio API and added a slight pitch tweak (just to make things sound a little cooler):

```ts
private async playPitchAudio(url: string, pitch: number = 0.95): Promise<void> {
    const AudioContext = window.AudioContext || (window as any).webkitAudioContext;
    const audioContext = new AudioContext();

    const response = await fetch(url);
    const arrayBuffer = await response.arrayBuffer();
    const audioBuffer = await audioContext.decodeAudioData(arrayBuffer);

    const source = audioContext.createBufferSource();
    source.buffer = audioBuffer;
    source.playbackRate.value = pitch;

    source.connect(audioContext.destination);
    source.start(0);
}
```

Right now I’m using a pitch of `0.95`—just enough to give the voice a deeper, slightly more epic tone. You could totally tweak this or randomize it if you want more variation.

---

A lightweight setup that added some extra polish to the app using static class patterns and the Web Audio API.
