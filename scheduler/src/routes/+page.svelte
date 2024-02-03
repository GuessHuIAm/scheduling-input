<script>
    import Calendar from "@event-calendar/core";
    import TimeGrid from "@event-calendar/time-grid";
    import Interaction from "@event-calendar/interaction";
    import "@event-calendar/core/index.css";

    let calendar;
    let plugins = [TimeGrid, Interaction];
    let options = {
        view: "timeGridWeek",
        initialView: "timeGridWeek",
        selectable: true,
        editable: true,
        allDaySlot: false,
        select: (info) => handleEvent(info, true),
        eventDrop: (info) => handleEvent(info, false),
        eventResize: (info) => handleEvent(info, false),
    };

    function handleEvent(info, isNewEvent) {
        let event, selectionStart, selectionEnd;
        const events = calendar.getEvents();

        if (isNewEvent) {
            calendar.unselect();
            event = makeEvent("event", info.startStr, info.endStr);
            calendar.addEvent(event);
        } else {
            event = info.event;
        }
        selectionStart = new Date(event.start);
        selectionEnd = new Date(event.end);
        console.log(selectionStart, selectionEnd);

        events.forEach((e) => {
            if (e.id === event.id || e.extendedProps.type != event.extendedProps.type) {
                return;
            }

            const eStart = new Date(e.start);
            const eEnd = new Date(e.end);

            console.log(eStart, eEnd);

            if (isTimeOverlap(eStart, eEnd, selectionStart, selectionEnd)) {
                console.log("Overlap between " + e.id + " and " + event.id);
                const newStart = eStart < selectionStart ? e.start : event.start;
                const newEnd = eEnd > selectionEnd ? e.end : event.end;

                calendar.removeEventById(e.id);
                event.start = newStart;
                event.end = newEnd;
            }
        });

        calendar.updateEvent(event);
    }

    function isTimeOverlap(start1, end1, start2, end2) {
        console.log(start1, end1, start2, end2);
        return start1 <= end2 && end1 >= start2;
    }

    function createEventId(type) {
        return String(Date.now());
    }

    function makeEvent(type, start, end) {
        return {
            id: createEventId(type),
            start: start,
            end: end,
            extendedProps: {
                type: type,
            },
        };
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
