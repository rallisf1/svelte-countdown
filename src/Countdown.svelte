<script>
import { onMount } from 'svelte'
import dayjs from 'dayjs'
import duration from 'dayjs/plugin/duration.js'
import utc from 'dayjs/plugin/utc.js'
import tz from 'dayjs/plugin/timezone.js'
import customParseFormat from 'dayjs/plugin/customParseFormat.js'

dayjs.extend(duration)
dayjs.extend(utc)
dayjs.extend(tz)
dayjs.extend(customParseFormat)

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
let r, target, local, timer

onMount(() => {
    if(!dateFormat){
        dateFormat = "YYYY-MM-DD H:m:s"
    }
	try {
		target = zone ? dayjs(from, dateFormat, zone) : dayjs(from, dateFormat)
	} catch(e) {
		if(e.message.indexOf('Invalid time zone') > -1) {
			target = dayjs(from, dateFormat)
			console.warn('[svelte-countdown] Countdown might not be precice as a proper timezone was not defined.')
		} else {
			console.warn('[svelte-countdown] Could not calculate date, make sure your "from" and "dateFormat" inputs are correct.')
		}
	}

	if(dayjs.isDayjs(target)){
		local = dayjs()	
		diff = target.valueOf() - local.valueOf()
	}

    timer = setInterval(function(){
        if(diff > 0){
            r = dayjs.duration(diff)
            remaining = {
                years: r.years(),
                months: r.months(),
                weeks: r.weeks(),
                days: r.days(),
                hours: r.hours(),
                minutes: r.minutes(),
                seconds: r.seconds(),
                done: false
            }
            diff-=1000
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