<script>
    import Calendar from "@event-calendar/core";
    import TimeGrid from "@event-calendar/time-grid";
    import Interaction from "@event-calendar/interaction";
    import "@event-calendar/core/index.css";
    import CalendarCheck from "svelte-icons/fa/FaCalendarCheck.svelte";

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

        /* TOP

        eventContent: (info) => { 
        return {html: '<div class="cal-container"><div class="cal-header"><div>' 
            + info.timeText + '</div><div>X</div></div><div class="cal-body">BODY</div></div>'}; 
        }, 
        
        eventClick: (info) => { 
        if ((info.jsEvent.layerX > info.el.offsetWidth - 20) && (info.jsEvent.layerY < 20)) { 
            calendar.removeEventById(info.event.id); 
            setTimeout(() => { calendar.saveCalendarEvents(); }, 1000); }},
        
         */

        eventContent: (info) => {
            return info.event.display === "auto"
                ? {
                      html:
                          "<div>" +
                          info.timeText +
                          "</div>" +
                          '<button id="delete-button"> Delete </button>' +
                          // '<div class="icon"> <CalendarCheck /> </div>' + //doesn't show up here, try something else?
                          "<div>" +
                          info.event.title +
                          "</div>",
                  }
                : "";
        },

        eventClick: (info) => {
            if (info.event.display === "auto") {
                let btn = info.el.querySelector("button");
                if (info.jsEvent.target === btn) {
                    calendar.removeEventById(info.event.id);
                }
                if (info.event.extendedProps.type === "comment") {
                    //editComment(info.event);
                }
            }
        },
    };

    let defaultCommentOptions = [
        //make calendar icon the deafault for normal events?
        "Preferable", // thumbs up?
        "Not Preferable", // thumbs down? or something more nuetral?
        "Something Immediately Before", // using << icon?
        "Something Immediately After", // using >> icon?
        "Other", // unsure
    ];

    let commentStart, commentEnd, previewEventId;
    let commentText = defaultCommentOptions[0];
    let popupVisible = false;
    let editPopupVisible = false;
    function handleEvent(info, isNewEvent) {
        let event, selectionStart, selectionEnd;
        const events = calendar.getEvents();

        if (selectMode) {
            // Selection mode
            if (isNewEvent) {
                calendar.unselect();
                event = makeEvent("event", info.startStr, info.endStr);
                calendar.addEvent(event);
            } else {
                event = info.event;
            }
        } else if (!selectMode) {
            // Comment mode
            if (isNewEvent) {
                calendar.unselect();
                popupVisible = true;
                commentStart = info.startStr;
                commentEnd = info.endStr;
                event = makeEvent("preview", info.startStr, info.endStr);
                previewEventId = event.id;
                calendar.addEvent(event);
            } else {
                event = info.event;
            }
        }
        event = checkEventOverlap(event);
        calendar.updateEvent(event);
    }

    function checkEventOverlap(event) {
        const events = calendar.getEvents();
        const start = new Date(event.start);
        const end = new Date(event.end);
        events.forEach((e) => {
            if (
                e.id === event.id ||
                e.extendedProps.type != event.extendedProps.type ||
                e.title != event.title
            ) {
                return event;
            }
            const eStart = new Date(e.start);
            const eEnd = new Date(e.end);
            if (isTimeOverlap(eStart, eEnd, start, end)) {
                console.log("Overlap between " + e.id + " and " + event.id);
                const newStart = eStart < start ? e.start : event.start;
                const newEnd = eEnd > end ? e.end : event.end;

                calendar.removeEventById(e.id);
                event.start = newStart;
                event.end = newEnd;
            }
        });
        return event;
    }

    function hidePopup() {
        popupVisible = false;
        calendar.removeEventById(previewEventId);
    }

    /*
    function hideEditPopup() {
        editPopupVisible = false;
    }
    */

    function addNewComment() {
        const input = document.querySelector("#comment-other");
        let newCommentText =
            commentText == "Other" && input.value != ""
                ? input.value
                : commentText;
        let event = makeEvent(
            "comment",
            commentStart,
            commentEnd,
            newCommentText,
        );
        calendar.addEvent(event);
        event = checkEventOverlap(event);
        calendar.updateEvent(event);
        hidePopup();
    }
    /*
    function editComment(editEvent) {
        console.log('the editcomment function actvivated');
        editPopupVisible = true;
        console.log('Is the popup visible?: ' + editPopupVisible);
        const input = document.querySelector("#comment-other");
        let newCommentText = (commentText == "Other" && input.value != "") ? input.value : commentText;
        editEvent.title = newCommentText;
        calendar.updateEvent(editEvent);
        //hideEditPopup();
    }
*/
    function changeCommentOptions() {
        const options = document.querySelector("#comment-options");
        commentText = options.value;
    }

    function isTimeOverlap(start1, end1, start2, end2) {
        console.log(start1, end1, start2, end2);
        return start1 <= end2 && end1 >= start2;
    }

    function createEventId(type) {
        return String(Date.now());
    }

    function makeEvent(type, start, end, title = "") {
        if (type === "comment" && title === "") {
            throw new Error("Comment events must have a title.");
        }

        return {
            id: createEventId(type),
            start: start,
            end: end,
            extendedProps: {
                type: type,
            },
            editable: true,
            display: type === "preview" ? "preview" : "auto",
            title: title,
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

    let otherVisibility = false;
    function toggleOtherVisibility() {
        otherVisibility = !otherVisibility;
    }
</script>

<div style="--color-preview: {colorPreview}">
    <div class="sticky-header">
        <div class="mode-buttons-container">
            <button
                class="mode-button {selectMode ? 'selected' : ''}"
                on:click={switchPhase}>Selection Mode</button
            >
            <button
                class="mode-button comment-mode {!selectMode ? 'selected' : ''}"
                on:click={switchPhase}>Comment Mode</button
            >
        </div>
        <button
            class="availability-button {otherVisibility ? 'selected' : ''}"
            on:click={toggleOtherVisibility}>See Other's Availability</button
        >
    </div>
    <div class="center-text">
        {selectionText}
    </div>
    <div class="popup" style={popupVisible ? "" : "display: none;"}>
        <div class="popup-content">
            Add new comment for this time slot:
            <select id="comment-options" on:change={changeCommentOptions}>
                {#each defaultCommentOptions as option}
                    <option value={option}>{option}</option>
                {/each}
            </select>
            <input
                type="text"
                id="comment-other"
                style={commentText === "Other" ? "" : "display: none;"}
            />
            <div class="popup-buttons-container">
                <button class="popup-button" on:click={hidePopup}>Cancel</button
                >
                <button class="popup-button" on:click={addNewComment}
                    >Add</button
                >
            </div>
        </div>
    </div>
    <!--<div class="popup" style={editPopupVisible ? "" : "display: none;"}>
        <div class="popup-content">
            Edit your comment for this time slot:
            <select id="comment-options" on:change={changeCommentOptions}>
                {#each defaultCommentOptions as option}
                    <option value={option}>{option}</option>
                {/each}
            </select>
            <input type="text" id="comment-other" style={commentText === "Other" ? "" : "display: none;"} />
            <div class="popup-buttons-container">
                <button class="popup-button" on:click={hideEditPopup}>Cancel</button>
                <button class="popup-button" on:click={hideEditPopup}>Edit</button>
            </div>
        </div>
    </div> -->
    <div
        class="popup-background"
        style={popupVisible ? "" : "display: none;"}
    ></div>
    <Calendar bind:this={calendar} {plugins} {options} />
</div>
