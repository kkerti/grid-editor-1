<script>
  import { configNodeBinding } from '../../runtime/app-helper.store.js';

  import { clickOutside } from '../../_actions/click-outside.action.js';
  import { menuBoundaries } from '../../_actions/boundaries.action.js';

  import { onMount, createEventDispatcher } from 'svelte';

  import grid from '../../../protocol/grid-protocol.js';
  
  import { logger, user_input } from '../../../runtime/runtime.store.js';

  let advancedClickAddon;
  
  export let index = undefined;
  export let config = undefined;

  let topOffset = 0;



  import { writable, get, derived } from 'svelte/store';




  function createActionPrefStore(){

    const default_values = {
      advanced: {
        index: undefined, 
        visible: false,
      }
    }

    const store = writable(default_values);

    return {
      ...store,
      showAdvanced: (index, bool) => {
          store.update(s => {
              s.advanced = {
                index: index, 
                visible: s.advanced.visible = ! s.advanced.visible,
              }

            return s
          });
      },
      reset: () => {
        store.update(s => {s = default_values; return s;});
      }
    }
  }


  actionPrefStore = createActionPrefStore()


  function filterDuplicateTypes() {
    let arr = grid.properties.LUA;
    // v, i, a = value, index, array
    return arr.filter((v,i,a)=>a.findIndex(t=>(t.type === v.type))===i).map(e => e.type);
  }

  function addThisManually(elem){
    advancedClickAddon = elem;
  }

  function colorByDesc(desc){
    let color = '';
    switch (desc.split('_')[0]) {
      case 'led':
        color = 'text-yellow-400'
        break;
      case 'midi':
        color = 'text-indigo-400'
        break;
      case 'keyboard':
        color = 'text-green-400'
        break;
      case 'page':
        color = 'text-pink-400'
        break;
      default:
        color = 'text-white'
        break;
    }
    return color;
  }

  let sg = "all"; // show suggestion

  // managing lifecycle as child components dont receive data otherwise
  let ready = false;
  onMount(()=>{
    ready = true;
  });

</script>

{#if ready}

  {#if index == $actionPrefStore.advanced.index && $actionPrefStore.advanced.visible}

  <advanced-config class="w-full flex absolute">
    <container 
      id="config-menu"
      use:menuBoundaries
      on:offset-top={(e)=>{topOffset = e.detail;}} 
      style="right: calc(100% + 2rem);top:{-150 + topOffset}px;width:600px;height:600px;position:absolute;" 
      class="absolute shadow-md rounded-md bg-primary p-2 z-50">
      
      <wrapper 
        use:clickOutside={{useCapture: false}}
        on:click-outside={()=>{actionPrefStore.showAdvanced(index)}} 
        class="flex flex-col h-full font-mono pointer-events-auto">

        <div class="flex items-center justify-between">
          <div class="font-bold w-auto py-2 mx-2 border-b border-gray-700 text-white">Advanced</div>
          <div on:click={()=>{actionPrefStore.showAdvanced(index)}} id="close-btn" class="p-1 mx-1 cursor-pointer not-draggable rounded hover:bg-secondary">
            <svg class="w-5 h-5 p-1 fill-current text-gray-300" viewBox="0 0 29 29" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M2.37506 0.142151L28.4264 26.1935L26.1934 28.4264L0.142091 2.37512L2.37506 0.142151Z" />
              <path d="M28.4264 2.37512L2.37506 28.4264L0.14209 26.1935L26.1934 0.142151L28.4264 2.37512Z" />
            </svg>
          </div>
        </div>

        <div class="flex flex-grow overflow-y-scroll">

          <advanced-menu class="w-3/12 flex flex-col ">
            
              <select-config-suggestions class="w-full text-white flex flex-col text-sm">
                <div class=" flex flex-row py-2 flex-wrap">
                  {#each ['all',...filterDuplicateTypes()] as key}
                    <div on:click={()=>{sg = key}} class="{sg == key ? 'bg-select' : 'hover:bg-select-desaturate-10'} m-1 px-2 py-1 rounded-lg cursor-pointer">{key}</div>
                  {/each}
                </div>
              </select-config-suggestions>

              <config-pool class="w-full flex flex-col overflow-y-scroll">
                {#if sg == "all" || sg == "arithmetic_operator"}
                  <basic-functions class="w-full flex flex-col p-2">      
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Arithmetic Operators</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'arithmetic_operator' && m.allowed.find(a => a == $user_input.event.eventtype)) as syntax}
                        <div on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black text-white">{syntax.human}</div>
                      {/each}
                    </div>
                  </basic-functions>
                {/if}

                {#if sg == "all" || sg == "relational_operator"}
                  <oparators class="w-full flex flex-col p-2">
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Relational Operators</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'relational_operator' && m.allowed.find(a => a == $user_input.event.eventtype)) as syntax}
                        <div on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black text-white">{syntax.human}</div>
                      {/each}
                    </div>
                  </oparators>
                {/if}

                {#if sg == "all" || sg == "logical_operator"}
                  <oparators class="w-full flex flex-col p-2">
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Logical Operators</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'logical_operator' && m.allowed.find(a => a == $user_input.event.eventtype)) as syntax}
                        <div on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black text-white">{syntax.human}</div>
                      {/each}
                    </div>
                  </oparators>
                {/if}

                {#if sg == "all" || sg == "global"}
                  <config-functions class="w-full flex flex-col p-2">
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Grid Functions</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'global') as syntax}
                        <div  on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black {colorByDesc(syntax.human)}">{syntax.human}</div>
                      {/each}
                    </div>
                  </config-functions>
                {/if}

                {#if sg == "all" || sg == "encoder"}
                  <variables class="w-full flex flex-col p-2">
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Encoder</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'encoder') as syntax}
                        <div on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black text-white">{syntax.human}</div>
                      {/each}
                    </div>
                  </variables>
                {/if}

                {#if sg == "all" || sg == "button"}
                  <variables class="w-full flex flex-col p-2">
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Button</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'button') as syntax}
                        <div on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black text-white">{syntax.human}</div>
                      {/each}
                    </div>
                  </variables>
                {/if}

                {#if sg == "all" || sg == "potmeter"}
                  <variables class="w-full flex flex-col p-2">
                    {#if sg == "all"}<span class="text-gray-500 text-sm">Potmeter</span>{/if}
                    <div class="flex -ml-1 items-start flex-wrap">
                      {#each grid.properties.LUA.filter(m => m.type === 'potmeter') as syntax}
                        <div on:click={()=>{addThisManually(syntax)}} class="rounded-lg text-sm px-3 py-1 cursor-pointer hover:shadow-md border border-pick-saturate-20 hover:border-pick m-1 bg-gray-900 hover:bg-black text-white">{syntax.human}</div>
                      {/each}
                    </div>
                  </variables>
                {/if}
              </config-pool>
          </advanced-menu>

          <advanced-code class="w-9/12 px-4 overflow-y-scroll">
            <svelte:component slot="config" this={config.component} {config} advanced={true} {advancedClickAddon} {index} on:output/>  
          </advanced-code>

        </div>

      </wrapper>

    </container>
      
  </advanced-config>

  {/if}
{/if}

<style>

  segment:first-child{
    margin-top: 0px;
  }

  segment:last-child{
    margin-bottom: 0px;
  }

  ::-webkit-scrollbar {
      height: 6px;
      width: 6px;
      background: #1e2628;
  }
  
  ::-webkit-scrollbar-thumb {
      background: #286787;
      box-shadow: 0px 1px 2px rgba(0, 0, 0, 0.75);
  }
  
  ::-webkit-scrollbar-corner {
      background: #1e2628
  }

</style>