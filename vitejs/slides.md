---
theme: light-icons
layout: center
titleTemplate: '%s - Sandeep Ramgolam'
title: ViteJS - Modern Front-end Tooling
---

<div class="space-y-10 text-center">
    <div class="text-5xl font-bold text-purple-500">ViteJS</div>
    <div class="text-5xl text-yellow-500">Modern Front-end Tooling</div>
    <div class="text-3xl">Meetup May 2022</div>
</div>

---
layout: image-left
image: ./sandeep.png
left: false
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
        Sandeep Ramgolam
  </div>  
  <div >
      - Senior Front-end Developer Ringier South Africa
  </div>
  <div >
      - Speaker
  </div>
  <div >
     - Topic: ViteJS
  </div>
</div>

---
layout: image-left
image: ./evan.jpeg
---

# What is Vite?

Vite is created by Evan You, who is the creator of Vue.js, but it's not a Vue-only tool. It can be used for React, Preact, Svelte, Vue, Vanilla JS, and LitElements.

<https://twitter.com/youyuxi/status/1252736540691910657>

---
layout: default
---

# At its core, Vite does 2 things

- Serve your code locally during development.
- Bundle your Javascript, CSS and other assets together for production.

---
layout: default
---

# Why Vite?

To understand, let's take a step back...

- How does JS end up in the browser?

---
layout: default
---

# How do we make a lot of JS run in the browser?

---
layout: default
---

# Bundlers !  - Webpack / Rollup / Parcel

---
layout: image
image: ./bundlers.png
---

---
layout: default
---

# The problem with bundlers

- Slow Server Start
- Slow Updates
- Reconstructing the bundle can be expensive
- Reloading the page blows away the current state of the application (Without HMR)
- The slow feedback loop can greatly affect developers' productivity and <span class="text-bold text-red-500">happiness.</span>

---
layout: image
image: ./esm.png
---

---
layout: image
image: ./esm-is-here.png
---

---
layout: center
---
<https://2021.stateofjs.com/en-US/libraries/#tier_list>

---
layout: default
---

# But what about HMR ?

---
layout: default
---

# Vite Goodies

- HMR is performed over native ESM
- Vite also leverages HTTP headers to speed up full page reloads (again, let the browser do more work for us)
- Source code module requests are made conditional via 304 Not Modified,
- Dependency module requests are strongly cached via Cache-Control: max-age=31536000,immutable so they don't hit the server again once cached.
- Vite pre-bundles dependencies using esbuild, written in Go 10-100x faster than JavaScript-based bundlers.
