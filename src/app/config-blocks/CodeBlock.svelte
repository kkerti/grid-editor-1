<script context="module">
  // config descriptor parameters
  export const information = {
    short: 'cb',
    name: 'CodeBlock',
    rendering: 'standard',
    category: 'code',
    desc: 'Code Block',
    color: '#887880',
    defaultLua: 'print("hello")',
    icon: `
    <svg width="100%" height="100%" viewBox="0 0 333 265" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M329.594 123.925L252.587 42.2591C247.854 37.2344 239.954 37.0052 234.934 41.7467C229.922 46.4843 229.689 54.3964 234.426 59.4172L303.345 132.5L234.426 205.591C229.689 210.612 229.922 218.52 234.934 223.262C237.349 225.541 240.433 226.67 243.505 226.67C246.823 226.67 250.136 225.354 252.588 222.757L329.595 141.087C334.135 136.267 334.135 128.742 329.594 123.925Z" fill="black"/>
      <path d="M98.5775 205.588L29.6629 132.5L98.5775 59.4133C103.31 54.3925 103.082 46.4798 98.0657 41.7428C93.0537 37.0052 85.1449 37.2344 80.4126 42.2552L3.4058 123.921C-1.13527 128.738 -1.13527 136.267 3.4058 141.084L80.4165 222.754C82.8724 225.358 86.1816 226.671 89.4993 226.671C92.5711 226.671 95.656 225.537 98.0657 223.258C103.086 218.52 103.31 210.608 98.5775 205.588Z" fill="black"/>
      <path d="M186.703 0.142824C179.889 -0.890373 173.512 3.79254 172.471 10.6135L135.841 250.612C134.8 257.437 139.483 263.816 146.301 264.858C146.942 264.954 147.574 265 148.203 265C154.268 265 159.588 260.571 160.533 254.387L197.163 14.3888C198.204 7.56336 193.521 1.18448 186.703 0.142824Z" fill="black"/>
    </svg>
    `
  }

</script>

<script>


  import * as luamin from "../main/user-interface/code-editor/luamin.js";
  import stringManipulation from '../main/user-interface/_string-operations';

  import {createEventDispatcher, onMount} from 'svelte';


  import SendFeedback from "../main/user-interface/SendFeedback.svelte"

  import {appSettings} from "../runtime/app-helper.store"
  import {monaco_elementtype} from "../runtime/monaco-helper"

  import * as monaco from '../../../node_modules/monaco-editor/esm/vs/editor/editor.api'

  const dispatch = createEventDispatcher();

  export let config;
  export let index;
  export let advanced;
  export let advancedClickAddon;
  export let access_tree

  let committedCode = '';

  let codePreview;



  const creation_timestamp = Date.now();

  onMount(()=>{


    committedCode = config.script


    let human = stringManipulation.humanize(committedCode)
    let beautified = luamin.Beautify(human, {RenameVariables: false,RenameGlobals: false, SolveMath: false});
  
    if( beautified.charAt( 0 ) === '\n' )
        beautified = beautified.slice( 1 );

    codePreview.innerHTML  = beautified
    monaco.editor.colorizeElement(codePreview, {theme: "my-theme", tabSize: 2});

  })

  $: if(committedCode != $appSettings.monaco_code_committed && $appSettings.monaco_code_committed !== undefined){

    if ($appSettings.monaco_timestamp == creation_timestamp){


      committedCode = $appSettings.monaco_code_committed
      dispatch('output', {short: 'cb', script: committedCode});

      let human = stringManipulation.humanize(committedCode)
      let beautified = luamin.Beautify(human, {RenameVariables: false,RenameGlobals: false, SolveMath: false});

      if( beautified.charAt( 0 ) === '\n' )
        beautified = beautified.slice( 1 );

      codePreview.innerHTML = beautified
      monaco.editor.colorizeElement(codePreview, {theme: "my-theme", tabSize: 2});



    }
  }

  function open_monaco(){

    $appSettings.monaco_element = "encoder";  

    $appSettings.monaco_code_committed = committedCode
    $appSettings.monaco_timestamp = creation_timestamp

    $monaco_elementtype = access_tree.elementtype
    $appSettings.modal = "code";

  }


</script>


<code-block   
  class="w-full flex flex-col p-4">

    <div class="w-full flex flex-col">

      <div class="text-gray-500 text-sm font-bold">Code preview:</div>
      
      <div class="grid w-full">
        <pre on:dblclick={open_monaco} class="bg-secondary opacity-80 my-4 p-2 w-full overflow-auto" bind:this={codePreview}  data-lang="intech_lua" ></pre>
      </div>

      <button on:click={open_monaco} class="bg-commit hover:bg-commit-saturate-20 text-white rounded px-2 py-0.5 text-sm focus:outline-none">Edit Code</button>
        
    </div>

    <SendFeedback feedback_context="CodeBlock"/>

</code-block>

