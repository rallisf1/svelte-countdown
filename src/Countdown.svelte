<script>
import { DateTime } from 'luxon'

let {
    from,
    dateFormat = 'yyyy-MM-dd HH:mm:ss',
    zone,
    children
} = $props()

let remaining = $state({
    years: 0,
    months: 0,
    weeks: 0,
    days: 0,
    hours: 0,
    minutes: 0,
    seconds: 0,
    done: true
})

const emptyRemaining = () => ({
    years: 0,
    months: 0,
    weeks: 0,
    days: 0,
    hours: 0,
    minutes: 0,
    seconds: 0,
    done: true
})

const units = ['years', 'months', 'weeks', 'days', 'hours', 'minutes', 'seconds']

$effect(() => {
    if (typeof from !== 'string' || typeof dateFormat !== 'string') {
        console.warn('[svelte-countdown] "from" and "dateFormat" must be strings.')
        remaining = emptyRemaining()
        return
    }

    let target = DateTime.fromFormat(from, dateFormat)

    if (zone) {
        target = target.setZone(zone, { keepLocalTime: true })
    }

    if (!target.isValid) {
        console.warn(`[svelte-countdown] ${target.invalidReason}`)
        remaining = emptyRemaining()
        return
    }

    const update = () => {
        const duration = target.diffNow(units)

        if (duration.toMillis() <= 0) {
            remaining = emptyRemaining()
            return false
        }

        remaining = {
            years: duration.years,
            months: duration.months,
            weeks: duration.weeks,
            days: duration.days,
            hours: duration.hours,
            minutes: duration.minutes,
            seconds: Math.trunc(duration.seconds),
            done: false
        }
        return true
    }

    if (!update()) return

    const timer = setInterval(() => {
        if (!update()) clearInterval(timer)
    }, 1000)

    return () => clearInterval(timer)
})
</script>

{@render children?.(remaining)}
