<script>
    import Calendar from '@event-calendar/core';
    import TimeGrid from '@event-calendar/time-grid';
    import Interaction from '@event-calendar/interaction';
    import '@event-calendar/core/index.css';
    import PhaseToggle from './phaseToggle.svelte';

    let calendar;
    let plugins = [TimeGrid, Interaction];
    let options = {
        view: 'timeGridWeek',
        initialView: 'timeGridWeek',
        selectable: true,
        select: handleDateSelect,
    };

    function handleDateSelect(info) {
        calendar.unselect();
        
        calendar.addEvent({
            id: createEventId(),
            start: info.startStr,
            end: info.endStr,
            allDay: info.allDay
        });
    }

    function createEventId() {
        return String(Date.now());
    }

/* Play Area Containment */

    let selectPhase = true;
    let testText;

    function switchPhase() {
        selectPhase = !selectPhase;
        console.log('I have switched!');
    }
    console.log(selectPhase);

    if (selectPhase) {
        testText = 'in selection phase!'
    } else {
        testText = 'in commenting phase!'
    }

/* Play Area [end] */

</script>


<button on:click={switchPhase}> <!-- how to get button in line with toolbar?--> 
    We are {testText}
</button>
<Calendar bind:this={calendar} {plugins} {options} />
