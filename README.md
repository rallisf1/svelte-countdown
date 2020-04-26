# svelte-countdown

A simple css-agnostic countdown component for Svelte 3 with just 1 dependency!

## Installation 🔧

First you need a [Svelte](https://svelte.dev) 3 project. Its starter template lives at https://github.com/sveltejs/template.

Then install the component by running the following command in your project's directory:

```sh
npm install svelte-countdown
```

## Features ❤
* Only 1 dependency! [dayjs](https://day.js.org/)
* Support for timezones & DST
* _done_ key to simplify what to show once the countdown is finished

### Why not momentjs ?
dayjs is a modern alternative to momentjs with compatible Api. The main difference is that dayjs is tree shakeable which leaves a really tiny footprint to your bundle (aka loads faster). If you're still using momentjs in your project I'd suggest you take a look at [dayjs](https://day.js.org/). In most cases if you just switch packages and `import {dayjs as moment}` you won't have to change any of your code.

## How to use 🚀

1. First import the component on your svelte page's script section.

```js
import Countdown from 'svelte-countdown';
```

2. Call the component where you want it to be placed e.g.:

```html
<Countdown from="2020-11-09 09:30:00" format="YYYY-MM-DD H:m:s" zone="Europe/Athens" let:remaining>
    <div class="whatever">
        {#if remaining.done === false}
        <span>{remaining.years} years</span>
        <span>{remaining.months} months</span>
        <span>{remaining.weeks} weeks</span>
        <span>{remaining.days} days</span>
        <span>{remaining.hours} hours</span>
        <span>{remaining.minutes} minutes</span>
        <span>{remaining.seconds} seconds</span>
        {:else}
        <h2>The time has come!</h2>
        {/if}
    </div>
</Countdown>
```

In the slot space between `<Countdown></Countdown>` you can write your template however you like. Use the keys as per the example above.

_I recommend not to change the "remaining" object name. If you need to you can do it like so:_
```
let:remaining={yourvariable}
```

### Configuration Options

| Name | Default | Required | Description |
| ---- | ------- | -------- | ----------- |
| from | none | Yes | Effect duration in milliseconds |
| format | YYYY-MM-DD H:m:s | No | See https://day.js.org/docs/en/parse/string-format |
| zone | none | No | Use canonical timezones from https://en.wikipedia.org/wiki/List_of_tz_database_time_zones |

## The Cons ❌
* Timezones & DST computation is hardcoded (valid @ April 2020) - Will accept PRs for fixes
* No callback, I suppose the _done_ key is enough. Will update upon request.
* No milliseconds counting. Will update upon request.
* Tests - It's pretty straight forward but feel free to contribute.
* No Demo Page - Will do.

## Contribution 🖇️

Feel free to fork. If you find a bug or got something great to add make a pull request!

## Authors ✒️

* ** John Rallis ** - * Initial Work * - [rallisf1](https://github.com/rallisf1)

You can also look at the list of all the [contributors](https://github.com/rallisf1/svelte-countdown/contributors) who have participated in this project. 

## License 📄

This project is free to use, edit & distribute under the MIT License.

## Expressions of Gratitude 🎁

* Tell others about this project 📢 
* Buy me a beer 🍺 or coffee ☕ | ₿ [Crypto](https://freewallet.org/id/rallisf1/) |💰 [Cash](https://www.paypal.me/rallisf1) 
* Publicly thanks 🤓

---
⌨️ with ❤️ by  [rallisf1](https://github.com/rallisf1) 😊