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
        console.log(info);
        const events = calendar.getEvents();
        console.log(events);

        if (isNewEvent) {
            calendar.unselect();
            event = {
                id: createEventId(),
                start: info.startStr,
                end: info.endStr,
            };
            calendar.addEvent(event);
        } else {
            event = info.event;
        }
        selectionStart = new Date(event.start);
        selectionEnd = new Date(event.end);
        console.log(selectionStart, selectionEnd);

        events.forEach((e) => {
            if (!isNewEvent && e.id === event.id) return;

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

    function createEventId() {
        return String(Date.now());
    }
</script>

<Calendar bind:this={calendar} {plugins} {options} />
