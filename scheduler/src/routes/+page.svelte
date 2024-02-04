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

        if (selectMode) {
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
                if (
                    e.id === event.id ||
                    e.extendedProps.type != event.extendedProps.type
                ) {
                    return;
                }

                const eStart = new Date(e.start);
                const eEnd = new Date(e.end);

                console.log(eStart, eEnd);

                if (isTimeOverlap(eStart, eEnd, selectionStart, selectionEnd)) {
                    console.log("Overlap between " + e.id + " and " + event.id);
                    const newStart =
                        eStart < selectionStart ? e.start : event.start;
                    const newEnd = eEnd > selectionEnd ? e.end : event.end;

                    calendar.removeEventById(e.id);
                    event.start = newStart;
                    event.end = newEnd;
                }
            });

            calendar.updateEvent(event);
        } else if (!selectMode) {
            if (isNewEvent) {
                calendar.unselect();
                event = makeEvent("comment", info.startStr, info.endStr);
                calendar.addEvent(event);
            } else {
                event = info.event;
            }

            calendar.updateEvent(event);
        }
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
            editable: true,
            display: "auto",
            backgroundColor:
                type === "event"
                    ? "var(--color-event)"
                    : "var(--color-comment)",
        };
    }

    let selectMode = true;
    $: selectionText = selectMode
        ? "Drag to select the times you are available."
        : "Drag to annotate your availability.";
    function switchPhase() {
        selectMode = !selectMode;
        const events = calendar.getEvents();
        if (selectMode)
            events.forEach((e) => {
                if (e.extendedProps.type === "event") {
                    e.display = "auto";
                    e.editable = true;
                } else if (e.extendedProps.type === "comment") {
                    e.display = "background";
                    e.editable = false;
                    e.startEditable = false;
                    e.durationEditable = false;
                }
                calendar.updateEvent(e);
            });
        else
            events.forEach((e) => {
                if (e.extendedProps.type === "event") {
                    e.display = "background";
                    e.editable = false;
                    e.startEditable = false;
                    e.durationEditable = false;
                } else if (e.extendedProps.type === "comment") {
                    e.display = "auto";
                    e.editable = true;
                }
                calendar.updateEvent(e);
            });
    }
    $: colorPreview = selectMode
        ? "var(--color-event)"
        : "var(--color-comment)";
</script>

<div style="--color-preview: {colorPreview}">
    <div class="sticky-header mode-button-container">
        <button
            class="mode-button {selectMode ? 'selected' : ''}"
            on:click={switchPhase}>Selection Mode</button
        >
        <button
            class="mode-button comment-mode {!selectMode ? 'selected' : ''}"
            on:click={switchPhase}>Comment Mode</button
        >
    </div>
    <div class="center-text">
        {selectionText}
    </div>
    <Calendar bind:this={calendar} {plugins} {options} />
</div>
