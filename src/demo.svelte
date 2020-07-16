<script>
    import {flip} from "svelte/animate";
    
    export let data = [];

    let ghost;
    let grabbed;

    let lastTarget;

    let mouseY = 0; // pointer y coordinate within client
    let offsetY = 0; // y distance from top of grabbed element to pointer
    let layerY = 0; // distance from top of list to top of client

    function grab(clientY, element) {
        // modify grabbed element
        grabbed = element;
        grabbed.dataset.grabY = clientY;

        // modify ghost element (which is actually dragged)
        ghost.innerHTML = grabbed.innerHTML;

        // record offset from cursor to top of element
        // (used for positioning ghost)
        offsetY = grabbed.getBoundingClientRect().x - clientY;
        drag(clientY);
    }

    // drag handler updates cursor position
    function drag(clientY) {
        if (grabbed) {
            mouseY = clientY;
            layerY = ghost.parentNode.getBoundingClientRect().x;
        }
    }

    // touchEnter handler emulates the mouseenter event for touch input
    // (more or less)
    function touchEnter(ev) {       
        drag(ev.clientY);
        // trigger dragEnter the first time the cursor moves over a list item
        let target = document.elementFromPoint(ev.clientX, ev.clientY).closest(".item");
        if (target && target != lastTarget) {
            lastTarget = target;
            dragEnter(ev, target);
        }
    }

    function dragEnter(ev, target) {
        // swap items in data
        if (grabbed && target != grabbed && target.classList.contains("item")) {
            moveDatum(parseInt(grabbed.dataset.index), parseInt(target.dataset.index));
        }
    }

    // does the actual moving of items in data
    function moveDatum(from, to) {
        let temp = data[from];
        data = [...data.slice(0, from), ...data.slice(from + 1)];
        data = [...data.slice(0, to), temp, ...data.slice(to)];
    }

    function release(ev) {
        grabbed = null;
    }
</script>

<style>
    main {
        position: relative;
    }

    .list {
        cursor: grab;
        z-index: 5;
        display: flex;
				flex-wrap:wrap;
				width:100vw;
    }

    .item {
        box-sizing: border-box;
        padding-right: 32px;
        width: 200px;
        min-height: 3em;
        margin-bottom: 0.5em;
        background-color: white;
        border: 1px solid rgb(190, 190, 190);
        border-radius: 2px;
        user-select: none;
    }

    .item:last-child {
        margin-bottom: 0;
    }

    .item:not(#grabbed):not(#ghost) {
        z-index: 10;
    }

textarea {
    padding: 10px;
    line-height: 1.5;
    border-radius: 5px;
    border: 1px solid #ccc;
    box-shadow: 1px 1px 1px #999;
		width:200px;
}
    .buttons {
        width: 32px;
        min-width: 32px;
        margin: auto 0;
        display: flex;
    }

    .buttons button {
        cursor: pointer;
        width: 18px;
        height: 18px;
        margin: 0 auto;
        padding: 0;
        border: 1px solid rgba(0, 0, 0, 0);
        background-color: inherit;
    }

    .buttons button:focus {
        border: 1px solid black;
    }

    #grabbed {
        opacity: 0.0;
    }

    #ghost {
        pointer-events: none;
        z-index: -5;
        position: absolute;
        top: 0;
        left: 0;
        opacity: 0.0;
    }

    #ghost * {
        pointer-events: none;
    }

    #ghost.haunting {
        z-index: 20;
        opacity: 1.0;
    }
</style>

<!-- All the documentation has to go up here, sorry.
     (otherwise it conflicts with the HTML or svelte/animate) 
     The .list has handlers for pointer movement and pointer up/release/end.
     Each .item has a handler for pointer down/click/start, which assigns that
     element as the item currently being "grabbed".  They also have a handler
     for pointer enter (the touchmove handler has extra logic to behave like the
     no longer extant 'touchenter'), which swaps the entered element with the
     grabbed element when triggered.
     You'll also find reactive styling below, which keeps it from being directly
     part of the imperative javascript handlers. -->

<button on:click={(()=>{data=[data[0]+"a",...data]})}> add another note</button>

<main class="dragdroplist">
    <div 
        bind:this={ghost}
        id="ghost"
        class={grabbed ? "item haunting" : "item"}
        style={"left: " + (mouseY + offsetY - layerY) + "px"}><p></p></div>
    <div 
        class="list"
        on:mousemove={function(ev) {ev.stopPropagation(); drag(ev.clientX);}}
        on:touchmove={function(ev) {ev.stopPropagation(); drag(ev.touches[0].clientY);}}
        on:mouseup={function(ev) {ev.stopPropagation(); release(ev);}}
        on:touchend={function(ev) {ev.stopPropagation(); release(ev.touches[0]);}}>
        {#each data as datum, i (datum.id ? datum.id : JSON.stringify(datum))}
            <div 
                id={(grabbed && (datum.id ? datum.id : JSON.stringify(datum)) == grabbed.dataset.id) ? "grabbed" : ""}
                class="item"
                data-index={i}
                data-id={(datum.id ? datum.id : JSON.stringify(datum))}
                data-grabY="0"
                on:mousedown={function(ev) {grab(ev.clientX, this);}}
                on:touchstart={function(ev) {grab(ev.touches[0].clientY, this);}}
                on:mouseenter={function(ev) {ev.stopPropagation(); dragEnter(ev, ev.target);}}
                on:touchmove={function(ev) {ev.stopPropagation(); ev.preventDefault(); touchEnter(ev.touches[0]);}}
                animate:flip|local={{duration: 200}}>
								<label for="story">Drag me</label>

<textarea id="story" name="story"
          rows="5" cols="33" on:mousedown={(e)=>{e.stopPropagation();}}>Write notes here
</textarea>

            </div>
        {/each}
    </div>
</main>
