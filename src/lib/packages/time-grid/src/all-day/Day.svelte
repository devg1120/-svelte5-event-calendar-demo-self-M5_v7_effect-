<script lang="ts">
    import { getContext } from "svelte";
    import { datesEqual, outsideRange, runReposition, setPayload } from "@event-calendar/core";
    import Event from "./Event.svelte";

    let { date, chunks, bgChunks, longChunks, iChunks = [], resource = undefined } = $props();

    let { highlightedDates, theme, validRange, _interaction, _today } = getContext("state");

    let el = $state();
    let dayChunks = $derived(chunks.filter((chunk) => datesEqual(chunk.date, date))),
        dayBgChunks = $derived(bgChunks.filter((bgChunk) => datesEqual(bgChunk.date, date)));
    let isToday = $derived(datesEqual(date, $_today)),
        highlight = $derived($highlightedDates.some((d) => datesEqual(d, date))),
        disabled = $state();
    let refs = $state([]);

    $effect(() => {
        disabled = outsideRange(date, $validRange);
    });

    // dateFromPoint
    $effect(() => {
        if (el) {
            setPayload(el, () => ({
                allDay: true,
                date,
                resource,
                dayEl: el,
                disabled,
            }));
        }
    });

    export function reposition() {
        if (!disabled) {
            runReposition(refs, dayChunks);
        }
    }
</script>

<div
    bind:this={el}
    class="{$theme.day} {$theme.weekdays?.[date.getUTCDay()]}{isToday ? ' ' + $theme.today : ''}{highlight
        ? ' ' + $theme.highlight
        : ''}{disabled ? ' ' + $theme.disabled : ''}"
    role="cell"
    onpointerdown={!disabled ? $_interaction.action?.select : undefined}
>
    <div class={$theme.bgEvents}>
        {#if !disabled}
            {#each dayBgChunks as chunk (chunk.event)}
                <Event {chunk} />
            {/each}
        {/if}
    </div>
    <!-- Drag, Resize & Select -->
    {#if iChunks[0] && datesEqual(iChunks[0].date, date) && !disabled}
        <div class="{$theme.events} {$theme.preview}">
            <Event chunk={iChunks[0]} />
        </div>
    {/if}
    <div class={$theme.events}>
        {#if !disabled}
            {#each dayChunks as chunk, i (chunk.event)}
                <Event {chunk} {longChunks} bind:this={refs[i]} />
            {/each}
        {/if}
    </div>
</div>
