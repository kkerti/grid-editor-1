<script>

  import { onMount } from 'svelte';

  import { appSettings } from '../../../../runtime/app-helper.store.js';

  import { select } from '../event-handlers/select.js';

  import Encoder from '../elements/Encoder.svelte';
  import Fader from '../elements/Fader.svelte';
  import Led from '../elements/Led.svelte';

  import { elementPositionStore } from '../../../../runtime/runtime.store';
  import { ledColorStore } from '../../../../runtime/runtime.store';

  export let moduleWidth;
  export let selectedElement = {id: '', brc: {}, event: {}};
  export let id = 'EF44';
  export let rotation = 0;

  let dx, dy;

  let elementposition_array = [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0];
  let ledcolor_array = [[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0]];

  elementPositionStore.subscribe(value => {

    try {
      let eps = value[dx][dy]

      for (const key in eps) {
        elementposition_array[key] = eps[key];
      }

    } catch (error) {
      return;
    }
  });

  ledColorStore.subscribe(value => {
    try {
      let lcs = value[dx][dy]

      for (const key in lcs) {
        ledcolor_array[key] = lcs[key];

      }

    } catch (error) {
      return;
    }
  });


  onMount(()=>{

    if(id !== undefined && (id.length > 4)){
      dx = +id.split(';')[0].split(':').pop();
      dy = +id.split(';')[1].split(':').pop();
    }

  });

</script>

<div id={id} draggable={$appSettings.layoutMode} style="transform: rotate({rotation*-90+'deg'})">

  <slot></slot>

  <div 
    use:select={[id]}
    class:disable-pointer-events={$appSettings.layoutMode}
    class="module-dimensions border-2 {(dx == selectedElement.brc.dx && dy == selectedElement.brc.dy) ? ' border-gray-500' : 'border-transparent'} " 
    style="--module-size: {moduleWidth+'px'}"
    >

    {#each  [0,1] as row }
      <div class="control-row" style="--control-row-mt: {$appSettings.size * 3.235 +'px'}; --control-row-mx: {$appSettings.size * 6.835 + 'px'}; --control-row-mb: {$appSettings.size * 6.835 + 'px'}" >
        {#each  [0+4*row,1+4*row,2+4*row,3+4*row] as elementNumber}

          {#if row == 0}
          <div class:active-element={dx == selectedElement.brc.dx && dy == selectedElement.brc.dy && selectedElement.event.elementnumber == elementNumber} data-element-number={elementNumber} class="knob-and-led">
            <Led
              color={ledcolor_array[elementNumber]} 
              size={$appSettings.size}/>
            <Encoder {elementNumber} 
              position={elementposition_array[elementNumber]} 
              size={$appSettings.size}/>
          </div>
          {:else}
          <div class:active-element={dx == selectedElement.brc.dx && dy == selectedElement.brc.dy && selectedElement.event.elementnumber == elementNumber} data-element-number={elementNumber} class="knob-and-led">
            <Led
              color={ledcolor_array[elementNumber]} 
              size={$appSettings.size}/>
            <Fader {elementNumber} 
              position={elementposition_array[elementNumber]} 
              size={$appSettings.size}
              rotation={rotation*-90}
              faderHeight={68}
              />
          </div>
          {/if}


        {/each}
      </div>
    {/each}

  </div>
</div>