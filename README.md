<h1><a href=//bacionejs.github.io/cat/index.html style=text-decoration: none; color: inherit;>Play</a></h1>

<a href=//bacionejs.github.io/cat/index.html target=_blank>
    <img src=README.jpg width=33% />
</a>
<br>
<br>

---
**Story**  
Catwoman is having a snowy run across Gotham's rooftops.  
She can hear the jazz clubs playing in the streets below.  

---
**Controls**  
There are 7 invisible jazz notes evenly spaced across the screen.  
The middle note causes her to jump.  

---
**Challenge**  
💡 Try to create jazz music while jumping.  
There is no penalty for missing a rooftop.  
There is no score, level, mission or bad guys.  

---
**Optional**  
Tap the top to mute and then play background jazz - [spotify](https://open.spotify.com/playlist/6gqJPa4A4gXTwTSGWcpC1d)  

---
**Tutorial**  
How to play jazz music - [youtube](https://www.youtube.com/watch?v=E5WLNmErkiY)  

---
## Credits  
- Music Player: [pl_synth](https://github.com/phoboslab/pl_synth)

---


## Post-mortem

This game is unorthodox, so it requires a little extra attention from the player to make it challenging and fun.
There is no score to tell you that you are a success, only your own eyes and ears...

- First, watch the [jazz music tutorial](https://www.youtube.com/watch?v=E5WLNmErkiY) video on how to create jazz music improv.

- Then, forget about Catwoman, close your eyes and play the seven notes across the screen. Work on your music improv skills based on the tutorial you watched.

- Then, when you're confident, try to combine that with hitting the middle note at the exact time to make Catwoman jump to the next rooftop.  

It's impossible to score your music improv, but you know if you sound good. Also, Catwoman does fall from the rooftop, just not very far. It's more of a gentle visual cue than a punishment. And like the instruction says, there's no penalty for missing. So she just keeps going. Anything more than that would negate the Zen vibe. Enjoy and happy music making 🎶



### 📝 Backstory  

It was the end of the competition and even with my prodding in the forum, nobody planned to make a Catwoman and it would be a shame if she wasn't represented. 
I really wanted to see a **sleek and smooth Catwoman**. 
But with no experience in drawing or animation, I decided not to worry too much about form and instead focus on capturing a **natural, fluid stride**.  

I also imagined the game with **jazz background music** — something cool and relaxing to match the zen vibe of a snowy nighttime Gotham rooftop run.  

The problem: I’ve never composed music before. I tried several tracker editors but struggled with them. Eventually, I even [built my own tracker](https://github.com/bacionejs/battito). Still, I couldn’t create a jazz piece that truly fit the mood I wanted.  

After watching countless jazz tutorials, I had an epiphany: instead of composing the perfect track myself, why not let the [player play simple jazz](https://www.youtube.com/watch?v=E5WLNmErkiY) — and make one of the notes trigger the cat’s jump? Naturally, this turned into a **multitasking nightmare** 😅.  

In the end, you have two choices:  

- For the **Zen experience**, tap at the top to mute the in-game sound and play your own background jazz.  
- Or, if you’re feeling daring, try to **jump and play jazz at the same time** — and see how long you last!

## 🎹 Jump trigger

- **Instrument:** improvise over the jazz soundtrack with piano notes.  
Among the notes, one key is special — when you hit the middle note, it doesn’t just play sound: it also triggers a **jump**



👉 Tests your ability to multitask music and jumping.

## 🐾 Body Physics
 
The most interesting part of this animation is how the **shin (lower leg)** and the **body bobbing** work together to make the body’s stride look alive.
  
### 1. Phase delay in the shin
 
The shin doesn’t move in perfect sync with the thigh. Its motion is shifted slightly in time (`lowerDelay`), so the knee bends **after** the thigh starts swinging. This gives the motion a sense of follow-through, like muscle and tendon reaction rather than mechanical linkage.  

`let t = (sin(p - l.lowerDelay) + 1) / 2;`

### 2. Quadratic ease-in/out
 
The raw sine motion is reshaped with a **quadratic ease function**:  

`let e = t < .5 ? 2*t*t : -1 + (4 - 2*t)*t;`

This curve makes the shin:
- **Ease in** slowly from rest  
- **Accelerate** in mid-swing  
- **Ease out** to slow down at the end  

So instead of robotic motion, the shin **extends and retracts with momentum**, like a real limb.  

### 3. Range-limited extension
 
The eased value `e` then sweeps the shin angle between a **minimum bend** and a **maximum extension**, in a range that looks natural:  

`let angle = u + l.lowerMin + (l.lowerMax - l.lowerMin) * e;`

### 4. Subtle body bobbing
 
To tie it all together, the whole body is given a slight **up-and-down bob**:  

`let b = sin(p0*2 + PI/4) * 6 * size;`  
`let h0 = { x: h[0].x, y: h[0].y + b };`

- The bobbing is tied to **double the leg cycle frequency** (`sin(p*2)`), matching the leg rhythm.  
- The amplitude is small (just a few pixels), but it makes the body look like it’s **absorbing impact and pushing off the ground**.  

Without this bobbing, the legs would move correctly, but the run would feel flat and mechanical. With it, the stride feels weighty and natural.
  
### 5. Physics analogy
 
- **Thigh (upper leg):** like a driven pendulum.  
- **Shin (lower leg):** a secondary pendulum with delay and easing, following the thigh.  
- **Torso bobbing:** the “mass-on-springs” effect of body weight shifting with each step.  

Together, these three effects — **delay, easing, and bobbing** — produce a run cycle that feels organic and convincing with very little code.  

👉 The lower leg physics makes the stride believable, and the body bobbing seals the illusion of a real running catwoman.

## 🏃‍♀️ Animation Performance: Pre-rendering the Run Cycle

To ensure a silky-smooth animation without performance hits, Catwoman's entire run cycle isn't drawn in real-time during the game. Instead, it's **pre-rendered** into a series of static images when the game first loads.

### How It Works

1.  **One-Time Generation:** At startup, the code procedurally generates each of the 30 frames of the run cycle. For every frame, it calculates the precise position and angle of the torso, head, arms, and legs using the physics logic described above.

2.  **Drawing to an Off-screen Canvas:** Each calculated frame is drawn onto a hidden canvas that the player never sees.

3.  **Storing as Images:** This canvas is immediately converted into an image data URL (`.toDataURL()`) and stored in an array. This process is repeated for all 30 frames.

4.  **Playback:** During the game, the animation loop simply picks the correct pre-made image from the array and draws it to the screen.

### The Advantage

This technique is a classic **space-time tradeoff**. It uses a bit more memory upfront to store the image frames, but it drastically reduces the CPU's workload during gameplay. Instead of performing dozens of complex physics calculations every single frame, the game only needs to do one simple thing: draw an image.

👉 This pre-rendering process is key to making the animation fluid and responsive, even on less powerful devices.

