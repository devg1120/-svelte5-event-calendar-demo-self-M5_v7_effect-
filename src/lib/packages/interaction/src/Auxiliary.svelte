<script>
    import { getContext } from "svelte";
    import { bgEvent, helperEvent, listen } from "@event-calendar/core";
    import Action from "./Action.svelte";
    import Pointer from "./Pointer.svelte";
    import Resizer from "./Resizer.svelte";

    let { theme, editable, eventStartEditable, pointer, _bodyEl, _interaction, _iClasses, _draggable } =
        getContext("state");

    $_interaction.resizer = Resizer;

    function bodyScrollHandler() {
        for (let component of Object.values($_interaction)) {
            component?.handleScroll?.();
        }
    }
    $effect(() => {
        $_draggable = (event) => (event.startEditable ?? $eventStartEditable) || (event.editable ?? $editable);
    });
    $effect(() => {
        $_iClasses = (className, event) => {
            let { display } = event;
            return helperEvent(display)
                ? [$theme[display]]
                : !bgEvent(display) && $_draggable(event)
                  ? [$theme.draggable]
                  : [];
        };
    });
    $effect(() => {
        if ($_bodyEl) {
            listen($_bodyEl, "scroll", bodyScrollHandler);
        }
    });
</script>

<Action bind:this={$_interaction.action} />
{#if $pointer}
    <Pointer bind:this={$_interaction.pointer} />
{/if}
