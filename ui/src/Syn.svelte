<script>
  import { connection, scribeStr, content, folks } from './stores.js'
  import { createEventDispatcher } from 'svelte'
  import { Connection} from './syn.js'
  import { bufferToBase64 } from './utils.js'

  let session

  // this properties are the app-defined functions to apply and undo changes
  export let applyDeltaFn
  export let undoFn

  // this is the list of sessions returned by the DNA
  let sessions

  export function requestChange(deltas) {
    $session.requestChange(deltas)
  }

  // -----------------------------------------------------------------------

  const dispatch = createEventDispatcher()

  let adminPort=1234
  let appPort=8888
  let appId='syn'
  async function toggle() {
    if (!$connection) {
      $connection = new Connection(appPort, appId)
      await $connection.open({title:'', body:''}, applyDeltaFn)

      session = $connection.syn.session

      console.log('joining session...')
      await $connection.joinSession()
      sessions = $connection.sessions
    }
    else {
      $connection.syn.clearState()
      sessions = undefined
      console.log('disconnected')
    }
  }

  async function commitChange() {
    $session.commitChange()
  }

  $: noscribe = $scribeStr === ''
</script>
<style>
  :global(.noscribe) {
  pointer-events: none;
  position: relative;
  }

  :global(.noscribe:after) {
  content: ' ';
  z-index: 20;
  display: block;
  position: absolute;
  height: 100%;
  top: 0;
  left: 0;
  right: 0;
  background: rgba(255, 255, 255, 0.7);
  }
  input {
    width: 4em;
    //border: 2px solid red;
    border-radius: 4px;
  }
  .session {
    border-radius: 4px;
    background-color: pink
  }
  button {
    cursor: pointer;
  }
</style>
<button class:noscribe on:click={commitChange}>Commit</button>

<div>
  <h4>Holochain Connection:</h4>
  App Port: <input bind:value={appPort}>
  AppId: <input bind:value={appId}>
  <button on:click={toggle}>
    {#if $connection}
      Disconnect
    {:else}
      Connect
    {/if}
  </button>
</div>

<div class='sessions'>
  Sessions:
  {#if sessions}
  {#each sessions as session}
    <span class='session'>
      Id: {bufferToBase64(session).slice(-4)}
    </span>
  {/each}
  {/if}
</div>
