# svelte-countdown

A CSS-agnostic countdown component for Svelte 5. It exposes the remaining time
to your markup, so you control the presentation.

## Installation 🔧

Install it in a Svelte 5 project:

```sh
npm install svelte-countdown
```

## Demo

Run the included demo locally:

```sh
npm install
npm run demo
```

The GitHub Pages deployment rebuilds the demo on each push to `master`. Its
target is set to eight months after the deployment build time.

## Features ❤
* Only 1 dependency: [Luxon](https://moment.github.io/luxon/)
* Support for timezones & DST
* _done_ key to simplify what to show once the countdown is finished

## How to use 🚀

1. First import the component on your svelte page's script section.

```js
import Countdown from 'svelte-countdown'
```

2. Call the component where you want it to be placed e.g.:

```html
<Countdown from="2026-12-31 23:59:59" zone="Europe/Athens">
    {#snippet children(remaining)}
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
    {/snippet}
</Countdown>
```

Pass any name you prefer to the `children` snippet parameter; it contains the
remaining-time keys shown above.

### Configuration Options

| Name | Default | Required | Description |
| ---- | ------- | -------- | ----------- |
| from | none | Yes | Target date/time string |
| dateFormat | yyyy-MM-dd HH:mm:ss | No | [Luxon format tokens](https://moment.github.io/luxon/#/formatting?id=table-of-tokens) |
| zone | none | No | Use canonical timezones from https://en.wikipedia.org/wiki/List_of_tz_database_time_zones |

## The Cons ❌
* No callback, I suppose the _done_ key is enough. Will update upon request.
* No milliseconds counting. Will update upon request.
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
