---
theme: light-icons
layout: center
titleTemplate: '%s - Sandeep Ramgolam'
canvasWidth: 1920
title: Open Data in Mauritius
---

<div class="space-y-10 text-center">
    <div class="text-6xl">Open Data in Mauritius</div>
    <div class="text-6xl font-bold">Meetup April 2022</div>
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
      - Senior Front-end Developer @ Ringier South Africa
  </div>
  <div >
      - Speaker
  </div>
  <div >
     - Topic: Open Data and Open Source Projects
  </div>
</div>


---
layout: image-right
left: false
---

# 

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
       What is Open Data ?
  </div>  
  <div >
      - What are some sources of Open Data?
  </div>
  <div >
      - Is there Open Data available in Mauritius ?
  </div>
</div>


---
layout: center
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
       Why should you care (as a developer or student) ?
  </div>  
  <div>
      - Build fun projects based on real data
  </div>
  <div>
      - Elevate your profile as a job seeker
  </div>
  <div>
      - Create useful applications and make money ?!
  </div>
    <div>
      - Can all of this be free of charge? (hosting + compute + domain)
  </div>
</div>

---
layout: center
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
       Ok but how much do these cost?
  </div>  
  <div>
      - Hosting  FREE - Netlify / Vercel /
  </div>
  <div>
      - Compute  FREE - Github Actions
  </div>
  <div>
      - (subdomain)Domain FREE - Netlify
  </div>
    <div>
      - CI/CD FREE - GH + Netlify
  </div>
</div>

---
layout: intro
---

# Project : CEB Power Outages

<iframe class="border p-4 rounded-md w-full h-screen" src="https://ceb.mu/customer-corner/power-outage-information"></iframe>

---
layout: center
---

# What if we want to use this data in our web page? 

---
---

# Scraping Part 1

```ts {4|5|12|18-23|all}
import request from 'request'
import deepmerge from 'deepmerge';

const URL = 'https://ceb.mu/customer-corner/power-outage-information'
const path = './data/power-outages.json'

request(URL, function (error, response, body) {

    let result = /var arDistrictLocations = ({".+"});/gm.exec(body); // THANKS @JulesMike  !!

    // The newly fetched data from CEB Website
    let newData = extractFromSource(JSON.parse(result[1]));

    //file exists
    if (fs.existsSync(path)) {
        console.log('Found file. Opening...')

        let rawdata = fs.readFileSync(path); // open file
        let oldData = JSON.parse(rawdata.toString());
        let mergedData = deepmerge(oldData, newData) // Merge old and new data
        let uniq = makeUniq(mergedData); // remove duplicate data & empty values
        fs.writeFileSync(path, JSON.stringify(uniq));

    } else {
        // Create the file
        console.log('creating file')
        fs.writeFileSync(path, JSON.stringify(newData));
    }
});
```

---

# Scraping Part 2

```ts {9-16|18-26|all} 
export const extractFromSource = (data) => {
    const districts = Object.keys(data);
    const dataset = {}

    for (const district in districts) {
        if (Object.prototype.hasOwnProperty.call(districts, district)) {
            const element = districts[district];
            let currentDistrict = convert(data[element],
                group('[id^=table-mauritius] tbody tr',
                    {
                        date: text('td:nth-child(1)'),
                        locality: text('td:nth-child(2)'),
                        streets: text('td:nth-child(3)'),
                        district: element,
                    }
                )
            )

            dataset[element] = currentDistrict.map(item => {
                return {
                    ...item,
                    "from": parseDate(item.date, 'from'),
                    "to": parseDate(item.date, 'to'),
                    "id": md5(JSON.stringify(item))
                }
            })
        }
    }


    return dataset
} 
```

---

# Automating it using Github Actions

```yml {7|20|27,28|all}
name: Scrape latest data

on:
  push:
  workflow_dispatch:
  schedule:
    - cron: "0 6-18 * * *" # “At minute 0 past every hour from 6 through 18.”

jobs:
  scheduled:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repo
        uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: "16"
          cache: "npm"
      - name: Fetch latest data
        run: npm ci && npm run fetch
      - name: Commit and push if it changed
        run: |-
          git config user.name "Automated"
          git config user.email "actions@users.noreply.github.com"
          git add -A
          timestamp=$(date -u)
          git commit -m "Latest data: ${timestamp}" || exit 0
          git push
```

---
layout: center
---

# Github Actions in.... action

<https://github.com/MrSunshyne/mauritius-dataset-electricity/actions>

---
---

# Let's make a front-end for it? Part 1: fetch data

```ts
const API_ENDPOINT = 'https://raw.githubusercontent.com/MrSunshyne/mauritius-dataset-electricity/main/data/power-outages.json'

export async function fetchJson(url = API_ENDPOINT) {
  try {
    const response = await fetch(url)
    return response.json()
  } catch (error) {
    console.log(error)
    throw error
  }
}
```

---
---

# Part 2: Loop on the data and display them in the Cell Component

```html
<script>
import data = fetchJson();
</script>

<div v-for="(outage) in data">
      <Cell :data="outage" />
</div>
```

---
---

# Part 3: Style the Cell Component


```html
<div>
    <div>
        <div >{{ locality }}</div>
        <div>{{ timeUntil }}</div>
        <div>{{ streets }}</div>
    <div>
        <div>
            <div>Power {{ state === 'ongoing' ? 'will resume in' : state === 'upcoming' ? 'will cut in' : 'has resumed since' }}</div>
            <vue-countdown
                :time="timeDifference" 
                v-slot="{ days, hours, minutes, seconds }"
            >{{ days ? days + 'd,' : '' }} {{ hours ? hours + 'h' : '' }} {{ minutes }}m {{ seconds }}s</vue-countdown>
            </div>
            <div>
            <RomanticCandle v-if="state === 'ongoing'"  />
            <div v-else class="w-2 h-2 relative left-[4px] top-[4px] shine rounded-full bg-green-500">&nbsp;</div>
            </div>
      </div> 
    </div>
  </div>
```

```ts
const state = computed(() => {
  let on = 'upcoming'
  const from = new Date(data.from)
  const to = new Date(data.to)
  const now = new Date()
  if (now.getTime() > from.getTime()) {
    on = 'ongoing'
  }
  if (now.getTime() > to.getTime()) {
    on = 'past'
  }
  return on
})

```

---

<https://power-outages-mauritius.netlify.app/>

<https://github.com/MrSunshyne/mauritius-power-outages/blob/master/src/components/Cell.vue>

---
layout: image-left
image: ./okouran.png
left: false
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
        Okouran
  </div>  
        https://www.okouran.mu/
</div>

---
layout: image-left
image: ./wasiim.png
left: false
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
        Twitter
  </div>  
</div>

---
layout: image-left
image: ./kritesh.png
left: false
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
        LinkedIn
  </div>  
</div>

---
layout: image-left
image: ./outagesapp.png
left: false
---

<div class="space-y-10 text-3xl text-primary dark:text-primary">
  <div class="text-black dark:text-white text-opacity-60 dark:text-opacity-60 pt-2 text-6xl font-bold">
        Paperboat
  </div>  
  https://outages.app/
</div>

---
layout: center
---

# Other Projects

<https://github.com/MrSunshyne/mauritius-dataset-meteo>

<https://mauritius-internet-prices.netlify.app>

<https://mauritius-fuel-prices.netlify.app>

<https://mrsunshyne.github.io/mauritius-sea-cable>



---
layout: center
---

# You can do it as well ! 

---
