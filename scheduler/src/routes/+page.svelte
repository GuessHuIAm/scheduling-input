<script>
    import Calendar from "@event-calendar/core";
    import TimeGrid from "@event-calendar/time-grid";
    import Interaction from "@event-calendar/interaction";
    import "@event-calendar/core/index.css";
    import Comment from "svelte-icons/fa/FaComment.svelte";
    import AngleDoubleLeft from "svelte-icons/fa/FaAngleDoubleLeft.svelte";
    import AngleDoubleRight from "svelte-icons/fa/FaAngleDoubleRight.svelte";
    import ThumbsUp from "svelte-icons/fa/FaThumbsUp.svelte";
    import ThumbsDown from "svelte-icons/fa/FaThumbsDown.svelte";
    import { onMount } from "svelte";

    let calendar;
    let plugins = [TimeGrid, Interaction];
    let icons;
    onMount(() => {
        icons = {
            "thumbs-up":
                document.getElementById("thumbs-up").innerHTML,
            "thumbs-down":
                document.getElementById("thumbs-down").innerHTML,
            "angle-double-left":
                document.getElementById("angle-double-left").innerHTML,
            "angle-double-right":
                document.getElementById("angle-double-right").innerHTML,
            "comment":
                document.getElementById("comment").innerHTML,
        };
    });
    let options = {
        view: "timeGridWeek",
        initialView: "timeGridWeek",
        selectable: true,
        editable: true,
        allDaySlot: false,
        select: (info) => handleEvent(info, true),
        eventDrop: (info) => handleEvent(info, false),
        eventResize: (info) => handleEvent(info, false),
        eventContent: (info) => {
            return info.event.display === "auto"
                ? {
                      html: `
                    <div>${info.timeText}</div>
                    <div>${info.event.title}</div>
                    <div class="icon" id="${info.event.id}">
                        ${
                            icons[info.event.extendedProps.icon_type]
                                ? icons[info.event.extendedProps.icon_type]
                                : ""
                        }
                    `,
                  }
                : "";
        },

        eventDidMount: (info) => {
            let icon = document.getElementById(info.event.id);
            if (icon) {
                let parent = icon.closest(".ec-event");
                parent.appendChild(icon);
            }
        },

        eventClick: (info) => {
            if (info.event.display === "auto") {
                populatePopupVariables(info.event);
            }
        },
    };

    let defaultCommentOptions = [
        "Preferable", // thumbs up?
        "Not Preferable", // thumbs down? or something more nuetral?
        "Something Immediately Before", // using << icon?
        "Something Immediately After", // using >> icon?
        "Other", // unsure
    ];

    let popupEventStart, popupEventEnd, popupEventId;
    let popupCommentOption = defaultCommentOptions[0],
        popupComment = defaultCommentOptions[0];
    // PopupCommentOption would be 'Other' if the user has a comment that is not in the defaultCommentOptions
    let popupVisible = false,
        editingComment = false,
        editingEvent = false;
    function populatePopupVariables(event, newEvent = false) {
        popupVisible = true;
        popupEventId = event.id;
        popupEventStart = event.start;
        popupEventEnd = event.end;
        if (!newEvent) {
            if (event.extendedProps.type === "comment") {
                editingComment = true;
                editingEvent = false;
                popupComment = event.title;
                popupCommentOption = defaultCommentOptions.includes(
                    popupComment,
                )
                    ? popupComment
                    : "Other";
            } else {
                editingComment = false;
                editingEvent = true;
            }
        } else {
            editingComment = false;
            editingEvent = false;
        }
        changeCommentOptions();
    }

    function handleEvent(info, isNewEvent) {
        let event;
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
                event = makeEvent("preview", info.startStr, info.endStr);
                populatePopupVariables(event, true);
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
        calendar.removeEventById(popupEventId);
    }

    function addNewComment() {
        const input = document.querySelector("#comment-other");
        let newpopupComment =
            popupComment == "Other" && input.value != ""
                ? input.value
                : popupComment;
        let event = makeEvent(
            "comment",
            popupEventStart,
            popupEventEnd,
            newpopupComment,
        );
        calendar.addEvent(event);
        event = checkEventOverlap(event);
        calendar.updateEvent(event);
        hidePopup();
    }

    function editComment() {
        const input = document.querySelector("#comment-other");
        calendar.removeEventById(popupEventId);
        let newpopupComment =
            popupComment == "Other" && input.value != ""
                ? input.value
                : popupComment;
        let event = makeEvent(
            "comment",
            popupEventStart,
            popupEventEnd,
            newpopupComment,
        );
        calendar.addEvent(event);
        event = checkEventOverlap(event);
        calendar.updateEvent(event);
        hidePopup();
    }

    function changeCommentOptions() {
        const options = document.querySelector("#comment-options");
        popupComment = options.value;
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
                icon_type: title === defaultCommentOptions[0] ? "thumbs-up" : //Preferable
                           title === defaultCommentOptions[1] ? "thumbs-down": //Not Preferable
                           title === defaultCommentOptions[2] ? "angle-double-left": //Something Immediately Before
                           title === defaultCommentOptions[3] ? "angle-double-right": //Something Immediately After
                           title !== "" ? "comment": "", //Other
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
    let submit = false;
    function toggleOtherVisibility() {
        otherVisibility = !otherVisibility;
    }
    function submitAvailability() { //figure treating submitting as a one type switch would be fine?
                                    //feel free to change to a toggle if unsubmitting feels better
        submit = true;
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
        <button 
            class="submit-button {submit ? 'selected' : ''}" 
            on:click={submitAvailability}>Submit</button
        >
    </div>
    <div class="center-text">
        {selectionText}
    </div>
    <div class="popup" style={popupVisible ? "" : "display: none;"}>
        <div class="popup-content">
            {#if editingComment}
                Edit comment for this time slot:
                <select id="comment-options" on:change={changeCommentOptions}>
                    {#each defaultCommentOptions as option}
                        <option
                            value={option}
                            selected={option === popupCommentOption}
                            >{option}</option
                        >
                    {/each}
                </select>
                <input
                    type="text"
                    id="comment-other"
                    style={popupCommentOption === "Other"
                        ? ""
                        : "display: none;"}
                />
                <div class="popup-buttons-container">
                    <button class="popup-button" on:click={hidePopup}
                        >Cancel</button
                    >
                    <button
                        class="popup-button delete-button"
                        on:click={() => {
                            calendar.removeEventById(popupEventId);
                            hidePopup();
                        }}>Remove Comment</button
                    >
                    <button class="popup-button" on:click={editComment}
                        >Save</button
                    >
                </div>
            {:else if editingEvent}
                Remove this event?
                <div class="popup-buttons-container">
                    <button class="popup-button" on:click={hidePopup}
                        >Cancel</button
                    >
                    <button
                        class="popup-button delete-button"
                        on:click={() => {
                            calendar.removeEventById(popupEventId);
                            hidePopup();
                        }}>Remove</button
                    >
                </div>
            {:else}
                Add new comment for this time slot:
                <select id="comment-options" on:change={changeCommentOptions}>
                    {#each defaultCommentOptions as option}
                        <option
                            value={option}
                            selected={option === popupCommentOption}
                            >{option}</option
                        >
                    {/each}
                </select>
                <input
                    type="text"
                    id="comment-other"
                    style={popupCommentOption === "Other"
                        ? ""
                        : "display: none;"}
                />
                <div class="popup-buttons-container">
                    <button class="popup-button" on:click={hidePopup}
                        >Cancel</button
                    >
                    <button class="popup-button" on:click={addNewComment}
                        >Add</button
                    >
                </div>
            {/if}
        </div>
    </div>
    <div
        class="popup-background"
        style={popupVisible ? "" : "display: none;"}
    ></div>
    <Calendar bind:this={calendar} {plugins} {options} />
    <div id="thumbs-up" hidden><ThumbsUp /></div>
    <div id="thumbs-down" hidden><ThumbsDown /></div>
    <div id="angle-double-left" hidden><AngleDoubleLeft /></div>
    <div id="angle-double-right" hidden><AngleDoubleRight /></div>
    <div id="comment" hidden><Comment /></div>
    
</div>
