<script>
import { onMount } from 'svelte'
import { DateTime } from 'luxon'

export let from, dateFormat, zone

let remaining = {
    years: 0,
    months: 0,
    weeks: 0,
    days: 0,
    hours: 0,
    minutes: 0,
    seconds: 0,
    done: true
}

let diff = 0
let r, target, timer

onMount(() => {
    if(!dateFormat){
        dateFormat = "yyyy-MM-dd HH:mm:ss"
    }

	target = zone ? DateTime.fromFormat(from, dateFormat).setZone(zone, { keepLocalTime: true }) : DateTime.fromFormat(from, dateFormat)

	if(!target.isValid) {
		console.warn(`[svelte-countdown] ${target.invalidReason}`)
	}

	if(DateTime.isDateTime(target)){
		r = target.diffNow(['years', 'months', 'weeks', 'days', 'hours', 'minutes', 'seconds'])
		diff = r.toMillis()
	}

    timer = setInterval(function(){
        if(diff > 0){
			r = target.diffNow(['years', 'months', 'weeks', 'days', 'hours', 'minutes', 'seconds'])
			diff = r.toMillis()
            remaining = {
                years: r.years,
                months: r.months,
                weeks: r.weeks,
                days: r.days,
                hours: r.hours,
                minutes: r.minutes,
                seconds: Math.trunc(r.seconds),
                done: false
            }
        } else {
            remaining = {
                years: 0,
                months: 0,
                weeks: 0,
                days: 0,
                hours: 0,
                minutes: 0,
                seconds: 0,
                done: true
            }
            clearInterval(timer)
        }

    }, 1000)
})
</script>

<slot {remaining}></slot>