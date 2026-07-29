<script>
    import { DateTime } from 'luxon'
    import Countdown from '../../src/index.js'

    const zone = 'Europe/Athens'
    const from = import.meta.env.VITE_TARGET_DATE ?? DateTime.now()
        .setZone(zone)
        .plus({ months: 8 })
        .toFormat('yyyy-MM-dd HH:mm:ss')
</script>

<svelte:head>
    <meta name="description" content="A Svelte 5 countdown component demo" />
</svelte:head>

<main>
    <p class="eyebrow">Svelte 5 component demo</p>
    <h1>Until the next good thing.</h1>
    <p class="intro">A countdown rendered entirely by your own markup.</p>

    <Countdown {from} {zone}>
        {#snippet children(remaining)}
            {#if remaining.done}
                <section class="complete" aria-live="polite">
                    <span aria-hidden="true">✦</span>
                    <h2>The time has come!</h2>
                </section>
            {:else}
                <section class="countdown" aria-label="Countdown" aria-live="polite">
                    <div class="unit">
                        <strong>{remaining.days}</strong>
                        <span>days</span>
                    </div>
                    <div class="unit">
                        <strong>{remaining.hours}</strong>
                        <span>hours</span>
                    </div>
                    <div class="unit">
                        <strong>{remaining.minutes}</strong>
                        <span>minutes</span>
                    </div>
                    <div class="unit">
                        <strong>{String(remaining.seconds).padStart(2, '0')}</strong>
                        <span>seconds</span>
                    </div>
                </section>
            {/if}
        {/snippet}
    </Countdown>

    <p class="details">Target time: {from} ({zone})</p>
</main>

<style>
    :global(*) {
        box-sizing: border-box;
    }

    :global(body) {
        min-width: 320px;
        margin: 0;
        color: #f6f3ff;
        background: radial-gradient(circle at top, #3d2d70, #171722 55%);
        font-family: Inter, ui-sans-serif, system-ui, sans-serif;
    }

    main {
        display: grid;
        min-height: 100vh;
        padding: 2rem;
        place-content: center;
        text-align: center;
    }

    .eyebrow {
        margin: 0 0 1rem;
        color: #b7a9fa;
        font-size: 0.8rem;
        font-weight: 700;
        letter-spacing: 0.12em;
        text-transform: uppercase;
    }

    h1 {
        max-width: 12ch;
        margin: 0 auto;
        font-size: clamp(2.5rem, 8vw, 5.5rem);
        line-height: 0.95;
    }

    .intro, .details {
        color: #c9c2dc;
    }

    .intro {
        margin: 1.5rem 0 3rem;
    }

    .countdown {
        display: grid;
        grid-template-columns: repeat(4, minmax(4.5rem, 1fr));
        gap: 0.75rem;
        width: min(100%, 42rem);
        margin: auto;
    }

    .unit, .complete {
        padding: 1.25rem 0.75rem;
        border: 1px solid rgb(255 255 255 / 15%);
        border-radius: 1rem;
        background: rgb(255 255 255 / 8%);
        box-shadow: 0 1rem 3rem rgb(0 0 0 / 20%);
        backdrop-filter: blur(8px);
    }

    .unit strong {
        display: block;
        font-size: clamp(2rem, 7vw, 4rem);
        line-height: 1;
    }

    .unit span {
        display: block;
        margin-top: 0.55rem;
        color: #c9c2dc;
        font-size: 0.75rem;
        font-weight: 700;
        letter-spacing: 0.08em;
        text-transform: uppercase;
    }

    .complete span {
        color: #edcf7d;
        font-size: 2rem;
    }

    .complete h2 {
        margin: 0.5rem 0 0;
    }

    .details {
        margin: 2rem 0 0;
        font-size: 0.875rem;
    }

    @media (max-width: 480px) {
        main {
            padding: 1.25rem;
        }

        .countdown {
            grid-template-columns: repeat(2, 1fr);
        }
    }
</style>
