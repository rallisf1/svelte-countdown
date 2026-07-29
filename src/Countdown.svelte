<script>
import { DateTime } from 'luxon'

let {from, dateFormat, zone} = $props()


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

let diff = 0
let r, target, local, timer

$effect(()=>{
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