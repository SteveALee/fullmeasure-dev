<script>
// @sts-nocheck  
import { readable } from 'svelte/store'
import { onMount } from 'svelte';

function isClient()
{
  return (typeof window !== 'undefined')
}

const gamepads = readable([], set => {
  let animationFrame
  const next = () => {
    // TODO optimise so doesn't render each animation frame
    set(window.navigator.getGamepads())
    animationFrame = requestAnimationFrame(next)
  }
  if (isClient() && window.requestAnimationFrame) {
    next()
    return () => cancelAnimationFrame(animationFrame)
  }
})

function addgamepad({gamepad}) {
  console.log('Gamepad added:', gamepad)
}
function removegamepad({gamepad}) {
  console.log('Gamepad removed:', gamepad)
}
</script>

<svelte:window on:gamepadconnected={addgamepad} 
               on:gamepaddisconnected={removegamepad} />

<p>Connect a Gamepad and press a button or waggle a joystick to get started. See the browser console for add/remove events.</p>

{#each $gamepads as gamepad, i (i)}
  {#if gamepad !== null}
    <h2>{gamepad.index} - {gamepad.id}</h2>

    {#if gamepad.buttons.length }
    <h3>{gamepad.buttons.length} Buttons</h3>
    <div class="buttons">
      {#each gamepad.buttons as button, i}
        <span class="button" 
              class:pressed={button.pressed}
              class:touched={button.touched}
              style="background-size: {Math.round(button.value * 100)}%"
              >{i}</span> 
      {/each}   
    </div>
    {:else}
    <h3>No buttons</h3>
    {/if}   

    {#if gamepad.axes.length }
    <h3>{gamepad.axes.length} Axes</h3>
    <div class="axes">
      {#each gamepad.axes as axes, i}
      {i}: {axes.toFixed(3)} <input type="range" class="axis" disabled min="-1.0" max="1.0" value="{axes}"> 
      {/each}   
    </div>
    {:else}
    <h3>No axes</h3>
    {/if}   
  {:else}
    <h3>{i}: Empty</h3>
  {/if}   

{:else}
  <h2>No gamepads found</h2>
{/each}

<style>
.axes {
padding: 1em;
}

.buttons {
margin-left: 1em;
}

/*meter*/.axis {
min-width: 200px;
margin: 1em;
}

.button {
display: inline-block;
width: 1em;
text-align: center;
padding: 1em;
border-radius: 20px;
border: 1px solid black;
background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAIAAACQd1PeAAAAAXNSR0IArs4c6QAAAAxJREFUCNdjYPjPAAACAgEAqiqeJwAAAABJRU5ErkJggg==);
background-size: 0% 0%;
background-position: 50% 50%;
background-repeat: no-repeat;
}

.pressed {
border: 1px solid red;
}

.touched::after {
content: "touch";
display: block;
position: absolute;
margin-top: -0.2em;
margin-left: -0.5em;
font-size: 0.8em;
opacity: 0.7;
}
</style>