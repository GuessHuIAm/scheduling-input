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
            "thumbs-up": document.getElementById("thumbs-up").innerHTML,
            "thumbs-down": document.getElementById("thumbs-down").innerHTML,
            "angle-double-left":
                document.getElementById("angle-double-left").innerHTML,
            "angle-double-right":
                document.getElementById("angle-double-right").innerHTML,
            comment: document.getElementById("comment").innerHTML,
        };
    });

    let options = {
        view: "timeGridWeek",
        initialView: "timeGridWeek",
        selectable: true,
        editable: true,
        allDaySlot: false,
        displayEventTime: true,
        slotMinTime: "08:00:00",
        slotMaxTime: "18:00:00",
        duration: { days: 6 },
        select: (info) => handleEvent(info, true),
        eventDrop: (info) => handleEvent(info, false),
        eventResize: (info) => handleEvent(info, false),
        eventContent: (info) => {
            let content = {
                html: "",
            };
            let eventType = info.event.extendedProps.type;
            let iconType = info.event.extendedProps.icon_type;
            if (info.event.display === "auto") {
                if (eventType === "comment") {
                    content = {
                        html: `
                    <div class="comment-body" data-id="${info.event.id}">
                        <div class="title">${info.event.title}</div>
                    </div>
                    `,
                    };
                } else if (eventType === "event") {
                    content = {
                        html: `
                    <div class="event-body" data-id="${info.event.id}">
                        <div class="title">${info.timeText}</div>
                    </div>
                    `,
                    };
                }
            } else if (info.event.display === "background") {
                if (eventType === "icon") {
                    content = {
                        html: `
                        <div class="icon" data-id="${info.event.id}">
                            ${icons[iconType] ? icons[iconType] : ""}
                        </div>
                        `,
                    };
                } else if (
                    eventType === "comment-other"
                ) {
                    content = {
                        html: `
                        <div class="${eventType}-body" data-id="${info.event.id}">
                            <div class='title'>${info.event.title}</div>
                            <div class='icon'>${icons[iconType] ? icons[iconType] : ""}</div>
                        </div>
                        `,
                    };
                } else if (
                    eventType === "event-other" ||
                    eventType === "event-other" ||
                    eventType === "comment"
                ) {
                    content = {
                        html: `
                        <div class="${eventType}-body" data-id="${info.event.id}">
                        </div>
                        `,
                    };
                }
            }
            return content;
        },

        eventDidMount: (info) => {
            let events = document.querySelectorAll(
                `[data-id="${info.event.id}"]`,
            );
            for (let event of events) {
                let eventType = info.event.extendedProps.type;

                if (eventType === "icon") {
                    let parent = event.closest(".ec-bg-event");
                    parent.classList.add("icon-event");
                } else if (eventType === "comment-other") {
                    let parent = event.closest(".ec-bg-event");
                    parent.classList.add("comment-event-other");
                } else if (eventType === "comment") {
                    let parent = event.closest(".ec-bg-event");
                    if (parent)  parent.classList.add("comment-event");
                }
            }
        },

        eventClick: (info) => {
            if (info.event.display === "auto") {
                populatePopupVariables(info.event);
            }
        },

        views: {
            timeGridWeek: {
                titleFormat: { year: "numeric", month: "long", day: "numeric" },
            },
            currentStart: "2024-02-07",
            currentEnd: "2024-02-12",
        },

        headerToolbar: {
            start: "",
            center: "title",
            end: "",
        },
    };

    let defaultCommentOptions = [
        "Preferable", // thumbs up
        "Not Preferable", // thumbs down
        "Something Immediately Before", // << icon
        "Something Immediately After", // >> icon
        "Other", // comment bubble
    ];

    let otherEvents = [
        makeEvent("event-other", "2024-02-10T08:00:00", "2024-02-10T13:00:00"),
        makeEvent("event-other", "2024-02-10T10:00:00", "2024-02-10T11:00:00"),
        makeEvent(
            "comment-other",
            "2024-02-09T9:30:00",
            "2024-02-09T11:00:00",
            "Preferable",
        ),
        makeEvent(
            "comment-other",
            "2024-02-11T17:00:00",
            "2024-02-11T18:00:00",
            "Preferable",
        ),
        makeEvent("event-other", "2024-02-12T13:00:00", "2024-02-12T15:00:00"),
        makeEvent("event-other", "2024-02-12T13:00:00", "2024-02-12T15:00:00"),
        makeEvent(
            "comment-other",
            "2024-02-09T14:00:00",
            "2024-02-09T15:00:00",
            "Something Immediately Before",
        ),
        makeEvent(
            "comment-other",
            "2024-02-10T13:00:00",
            "2024-02-10T14:00:00",
            "Something Immediately After",
        )
    ];

    let popupEventStart, popupEventEnd, popupEventId;
    let popupCommentOption = defaultCommentOptions[0],
        popupComment = defaultCommentOptions[0];
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
                popupCommentOption = defaultCommentOptions[0],
                popupComment = defaultCommentOptions[0];
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
                addEvent(event);
            } else {
                event = info.event;
            }
        } else if (!selectMode) {
            // Comment mode
            if (isNewEvent) {
                calendar.unselect();
                event = makeEvent("preview", info.startStr, info.endStr);
                populatePopupVariables(event, true);
                addEvent(event);
            } else {
                event = info.event;
            }
        }
        event = checkEventOverlap(event);
        updateEvent(event);
    }

    let commentIdToIconId = {};
    function addToCommentStarts(event) {
        console.log(event);
        let type = event.extendedProps.type;
        if (type === "comment" || type === "comment-other") {
            let icon = makeEvent(
                "icon",
                event.start,
                event.end,
                event.title,
            );
            console.log(icon);
            addEvent(icon);
            commentIdToIconId[event.id] = icon.id;
        }
    }

    function updateCommentStarts(event) {
        let type = event.extendedProps.type;
        if (type === "comment" || type === "comment-other") {
            // If the titles are different, remove the old icon and add a new one
            let icon = calendar.getEventById(commentIdToIconId[event.id]);
            if (icon.title !== event.title) {
                removeEvent(icon.id);
                icon = makeEvent(
                    "icon",
                    event.start,
                    event.end,
                    event.title,
                );
                addEvent(icon);
                commentIdToIconId[event.id] = icon.id;
            } else {
                icon.start = event.start;
                icon.end = event.end;
                updateEvent(icon);
            }
        }
    }

    function removeFromCommentStarts(event) {
        let type = event.extendedProps.type;
        if (type === "comment") {
            let icon = calendar.getEventById(commentIdToIconId[event.id]);
            removeEvent(icon.id);
        }
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
                removeEvent(e.id);
                event.start = newStart;
                event.end = newEnd;
            }
        });
        return event;
    }

    function hidePopup() {
        popupVisible = false;
    }

    function removeEvent(id) {
        console.log("Removing event " + id);
        removeFromCommentStarts(calendar.getEventById(id));
        calendar.removeEventById(id);
    }

    function addEvent(event) {
        console.log("Adding event " + event.id);
        addToCommentStarts(event);
        calendar.addEvent(event);
    }

    function updateEvent(event) {
        console.log("Updating event " + event.id);
        updateCommentStarts(event);
        calendar.updateEvent(event);
    }

    function addNewComment() {
        const input = document.querySelector("#comment-other");
        let newpopupComment =
            popupCommentOption === "Other" && input.value != ""
                ? input.value
                : popupCommentOption;
        let event = makeEvent(
            "comment",
            popupEventStart,
            popupEventEnd,
            newpopupComment,
        );
        addEvent(event);
        event = checkEventOverlap(event);
        updateEvent(event);
        hidePopup();
        removeEvent(popupEventId);
    }

    function editComment() {
        const input = document.querySelector("#comment-other");
        removeEvent(popupEventId);
        let newpopupComment =
            popupCommentOption === "Other" && input.value != ""
                ? input.value
                : popupCommentOption;
        let event = makeEvent(
            "comment",
            popupEventStart,
            popupEventEnd,
            newpopupComment,
        );
        addEvent(event);
        event = checkEventOverlap(event);
        updateEvent(event);
        hidePopup();
    }

    function changeCommentOptions() {
        const options = document.querySelector("#comment-options");
        popupCommentOption = options.value;
        if (popupCommentOption === "Other") {
            popupComment = "";
        } else {
            popupComment = popupCommentOption;
        }
    }

    function isTimeOverlap(start1, end1, start2, end2) {
        console.log(start1, end1, start2, end2);
        return start1 <= end2 && end1 >= start2;
    }

    function createEventId(type) {
        var d = new Date().getTime(); //Timestamp
        var d2 =
            (typeof performance !== "undefined" &&
                performance.now &&
                performance.now() * 1000) ||
            0; //Time in microseconds since page-load or 0 if unsupported
        return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(
            /[xy]/g,
            function (c) {
                var r = Math.random() * 16; //random number between 0 and 16
                if (d > 0) {
                    //Use timestamp until depleted
                    r = (d + r) % 16 | 0;
                    d = Math.floor(d / 16);
                } else {
                    //Use microseconds since page-load if supported
                    r = (d2 + r) % 16 | 0;
                    d2 = Math.floor(d2 / 16);
                }
                return (c === "x" ? r : (r & 0x3) | 0x8).toString(16);
            },
        );
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
                icon_type:
                    title === defaultCommentOptions[0]
                        ? "thumbs-up" //Preferable
                        : title === defaultCommentOptions[1]
                          ? "thumbs-down" // Not Preferable
                          : title === defaultCommentOptions[2]
                            ? "angle-double-left" // Something Immediately Before
                            : title === defaultCommentOptions[3]
                              ? "angle-double-right" // Something Immediately After
                              : title !== ""
                                ? "comment"
                                : "", //Other
            },
            editable: type === "icon" ? false : true,
            display:
                type === "preview"
                    ? "preview"
                    : type === "event-other" ||
                        type === "comment-other" ||
                        type === "icon"
                      ? "background"
                      : "auto",
            title: title,
            backgroundColor:
                type === "event"
                    ? "var(--color-event)"
                    : type === "comment"
                      ? "transparent"
                      : type === "event-other"
                        ? "var(--color-event-others)"
                        : type === "comment-other"
                          ? "transparent"
                          : type === "icon"
                            ? "var(--color-icon-event)"
                            : "var(--color-preview)",
        };
    }

    let selectMode = true;
    $: selectionText = selectMode
        ? "Drag to select the times you are available. Click to delete."
        : "Drag to annotate your availability. Click to edit or delete your comments.";
    function switchPhase() {
        selectMode = !selectMode;
        const events = calendar.getEvents();
        if (selectMode) {
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
        } else {
            events.forEach((e) => {
                if (e.extendedProps.type === "event") {
                    e.display = "background";
                    e.editable = false;
                    e.startEditable = false;
                } else if (e.extendedProps.type === "comment") {
                    e.display = "auto";
                    e.editable = true;
                }
                calendar.updateEvent(e);
            });
        }
    }

    $: colorPreview = selectMode
        ? "var(--color-event)"
        : "var(--color-comment)";

    let otherVisibility = false;
    let submit = false;

    function toggleOtherVisibility() {
        otherVisibility = !otherVisibility;
        let eventList = calendar.getEvents();
        let newEventsReal;
        if (otherVisibility) {
            newEventsReal = eventList.concat(otherEvents);
            calendar.setOption("events", newEventsReal);
        } else if (!otherVisibility) {
            eventList.forEach((e) => {
                if (
                    e.extendedProps.type === "event-other" ||
                    e.extendedProps.type === "comment-other"
                ) {
                    removeEvent(e.id);
                }
            });
        }
    }

    function submitAvailability() {
        submit = !submit;
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
        <div class="mode-buttons-container">
            <button
                class="availability-button {otherVisibility ? 'selected' : ''}"
                on:click={toggleOtherVisibility}
                >See Other's Availability</button
            >
            <button
                class="submit-button {submit ? 'selected' : ''}"
                on:click={submitAvailability}
                >{submit ? "Submitted" : "Submit"}</button
            >
        </div>
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
                            removeEvent(popupEventId);
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
                            removeEvent(popupEventId);
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
                    <button
                        class="popup-button"
                        on:click={() => {
                            hidePopup();
                            removeEvent(popupEventId);
                        }}>Cancel</button
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
