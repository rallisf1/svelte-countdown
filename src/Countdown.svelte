<script>
import { onMount } from 'svelte';
import dayjs from 'dayjs';
import duration from 'dayjs/plugin/duration';
import database from './database.js';

dayjs.extend(duration);

export let from, dateFormat, zone;

let remaining = {
    years: 0,
    months: 0,
    weeks: 0,
    days: 0,
    hours: 0,
    minutes: 0,
    seconds: 0,
    done: false
}
let r = 0;
var timer;

onMount(() => {
    if(!dateFormat){
        dateFormat = "YYYY-MM-DD H:m:s";
    }
    let target = dayjs(from, dateFormat);
    let local = dayjs();
    if(zone){
        let remoteTZ = database.find( ({ timezone }) => timezone === zone );
        let localTZ = database.find( ({ timezone }) => timezone === Intl.DateTimeFormat().resolvedOptions().timeZone );
        if(remoteTZ && localTZ){
            // calc UTC + DST diff
            let remoteDiff = UTCdiff(target, remoteTZ);
            let localDiff = UTCdiff(local, localTZ);

            target = (remoteDiff > 0) ? target.add(remoteDiff, 'minutes') : target.subtrack(Math.abs(remoteDiff), 'minutes');
            local = (localDiff > 0) ? local.add(localDiff, 'minutes') : local.subtrack(Math.abs(localDiff), 'minutes');
        } else {
            if(!remoteTZ) console.warn('[svelte-countdown] Timezone not found in database. Please use a canonical timezone from https://en.wikipedia.org/wiki/List_of_tz_database_time_zones');
            if(!localTZ) console.warn('[svelte-countdown] Intl API not supported by browser, cannot calculate timezone difference!');
        }
    } else {
        console.warn('[svelte-countdown] Countdown might not be precice as a timezone was not defined.');
    }

    let diff = target.valueOf() - local.valueOf();

    timer = setInterval(function(){
        if(diff > 0){
            let r = dayjs.duration(diff);
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
            diff-=1000;
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
            clearInterval(timer);
        }

    }, 1000);
});

function UTCdiff(date, tz){
    let isMinus = (tz['Offset'].charAt(0) == '-');
    let st = tz['Offset'].split(':').map(function(item){
        return parseInt(item);
    });
    if(tz['Offset'] == tz['DST Offset']){
        // no DST
        if(st[0] === 0 && st[1] === 0) return 0; // no diff
        return parseInt(((isMinus)?'-':'')+(st[0]*60)+st[1]); // return UTC diff in minutes
    } else {
        // calc DST
        let dst = tz['DST Offset'].split(':').map(function(item){
            return parseInt(item);
        });
        let isDST = false;
        let tmpDate;
        if(tz['Timezone'].indexOf('Europe') === 0){
            switch(true){
                // March/Last Sunday to October/Last Sunday @ 1am UTC
                case (date.month() > 2 && date.month() < 9):
                    isDST = true;
                    break;
                case (date.month() == 2):
                    tmpDate = date.endOf('month');
                    while(tmpDate.day() != 0){
                        tmpDate = tmpDate.subtrack(1, 'day');
                    }
                    if(date.date() == tmpDate.date()){
                        if(date.hour() > st[0] + 1) isDST = true;
                    } else {
                        if(date.date() > tmpDate.date()) isDST = true;
                    }
                    break;
                case (date.month() == 9):
                    tmpDate = date.endOf('month');
                    while(tmpDate.day() != 0){
                        tmpDate = tmpDate.subtrack(1, 'day');
                    }
                    if(date.date() == tmpDate.date()){
                        if(date.hour() < st[0] + 1) isDST = true;
                    } else {
                        if(date.date() < tmpDate.date()) isDST = true;
                    }
                    break;
                default:
                    break;
            }
        } else {
            switch(tz['Timezone']){
                case 'America/Havana':
                    switch(true){
                        // March/3rd Sunday to October/Last Sunday
                        case (date.month() > 2 && date.month() < 9):
                            isDST = true;
                            break;
                        case (date.month() == 2):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            tmpDate.add(14, 'day');
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 9):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'America/Mexico_City':
                    switch(true){
                        // April/1st Sunday to October/Last Sunday
                        case (date.month() > 3 && date.month() < 9):
                            isDST = true;
                            break;
                        case (date.month() == 3):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 9):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'America/Campo_Grande':
                case 'America/Cuiaba':
                    switch(true){
                        // October/3rd Sunday to February/3rd Sunday
                        case (date.month() > 9 || date.month() < 1):
                            isDST = true;
                            break;
                        case (date.month() == 9):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            tmpDate.add(14, 'day');
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 1):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            tmpDate.add(14, 'day');
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'America/Santiago':
                case 'Pacific/Easter':
                    switch(true){
                        // October/11 to March/29
                        case (date.month() > 9 || date.month() < 2):
                            isDST = true;
                            break;
                        case (date.month() == 9):
                            if(date.date() == 11){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > 11) isDST = true;
                            }
                            break;
                        case (date.month() == 2):
                            if(date.date() == 29){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < 29) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'America/Asuncion':
                    switch(true){
                        // October/3rd Sunday to March/2nd Sunday
                        case (date.month() > 9 || date.month() < 2):
                            isDST = true;
                            break;
                        case (date.month() == 9):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            tmpDate.add(14, 'day');
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 2):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            tmpDate.add(7, 'day');
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Africa/Cairo':
                    switch(true){
                        // April/Last Friday to September/Last Thursday
                        case (date.month() > 3 && date.month() < 8):
                            isDST = true;
                            break;
                        case (date.month() == 3):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 5){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 8):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 4){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Asia/Jerusalem':
                    switch(true){
                        // March/Friday before Last Sunday to October/Last Sunday
                        case (date.month() > 2 && date.month() < 9):
                            isDST = true;
                            break;
                        case (date.month() == 2):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            tmpDate.subtrack(2, 'day');
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 9):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Asia/Amman':
                    switch(true){
                        // March/Last Sunday to September/Last Friday
                        case (date.month() > 2 && date.month() < 8):
                            isDST = true;
                            break;
                        case (date.month() == 2):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 8):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 5){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Asia/Beirut':
                    switch(true){
                        // March/Last Sunday to October/Last Sunday
                        case (date.month() > 2 && date.month() < 9):
                            isDST = true;
                            break;
                        case (date.month() == 2):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 9):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Asia/Damascus':
                    switch(true){
                        // March/30 to September/21
                        case (date.month() > 2 && date.month() < 8):
                            isDST = true;
                            break;
                        case (date.month() == 2):
                            if(date.date() == 30){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > 30) isDST = true;
                            }
                            break;
                        case (date.month() == 8):
                            if(date.date() == 21){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < 21) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Australia/Tasmania':
                    switch(true){
                        // October/1st Sunday to March/Last Sunday
                        case (date.month() > 9 || date.month() < 2):
                            isDST = true;
                            break;
                        case (date.month() == 9):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 2):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Pacific/Auckland':
                case 'Pacific/Chatham':
                    switch(true){
                        // September/Last Sunday to April/1st Sunday
                        case (date.month() > 8 || date.month() < 3):
                            isDST = true;
                            break;
                        case (date.month() == 8):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 3):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                case 'Pacific/Tongatapu':
                    switch(true){
                        // November/1st Sunday to January/Last Sunday
                        case (date.month() > 10):
                            isDST = true;
                            break;
                        case (date.month() == 10):
                            tmpDate = date.startOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.add(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() > 2) isDST = true;
                            } else {
                                if(date.date() > tmpDate.date()) isDST = true;
                            }
                            break;
                        case (date.month() == 0):
                            tmpDate = date.endOf('month');
                            while(tmpDate.day() != 0){
                                tmpDate = tmpDate.subtrack(1, 'day');
                            }
                            if(date.date() == tmpDate.date()){
                                if(date.hour() < 2) isDST = true;
                            } else {
                                if(date.date() < tmpDate.date()) isDST = true;
                            }
                            break;
                        default:
                            break;
                    }
                    break;
                default:
                    if(tz['Timezone'].indexOf('Australia') === 0){
                        // Had to keep apart from Tasmania
                        switch(true){
                            // October/1st Sunday to April/1st Sunday
                            case (date.month() > 9 || date.month() < 3):
                                isDST = true;
                                break;
                            case (date.month() == 9):
                                tmpDate = date.startOf('month');
                                while(tmpDate.day() != 0){
                                    tmpDate = tmpDate.add(1, 'day');
                                }
                                if(date.date() == tmpDate.date()){
                                    if(date.hour() > 2) isDST = true;
                                } else {
                                    if(date.date() > tmpDate.date()) isDST = true;
                                }
                                break;
                            case (date.month() == 3):
                                tmpDate = date.startOf('month');
                                while(tmpDate.day() != 0){
                                    tmpDate = tmpDate.add(1, 'day');
                                }
                                if(date.date() == tmpDate.date()){
                                    if(date.hour() < 2) isDST = true;
                                } else {
                                    if(date.date() < tmpDate.date()) isDST = true;
                                }
                                break;
                            default:
                                break;
                        }
                    } else {
                        // USA / Canada & rest of the world... (sorry)
                        switch(true){
                            // March/2nd Sunday to November/1st Sunday
                            case (date.month() > 2 && date.month() < 10):
                                isDST = true;
                                break;
                            case (date.month() == 2):
                                tmpDate = date.startOf('month');
                                while(tmpDate.day() != 0){
                                    tmpDate = tmpDate.add(1, 'day');
                                }
                                tmpDate.add(7, 'day');
                                if(date.date() == tmpDate.date()){
                                    if(date.hour() > 2) isDST = true;
                                } else {
                                    if(date.date() > tmpDate.date()) isDST = true;
                                }
                                break;
                            case (date.month() == 10):
                                tmpDate = date.startOf('month');
                                while(tmpDate.day() != 0){
                                    tmpDate = tmpDate.add(1, 'day');
                                }
                                if(date.date() == tmpDate.date()){
                                    if(date.hour() < 2) isDST = true;
                                } else {
                                    if(date.date() < tmpDate.date()) isDST = true;
                                }
                                break;
                            default:
                                break;
                        }
                    }
                    break;
            }
        }
        return (isDST) ? parseInt(((isMinus)?'-':'')+(dst[0]*60)+dst[1]) : parseInt(((isMinus)?'-':'')+(st[0]*60)+st[1]);;
    }
}

</script>

<slot {remaining}></slot>