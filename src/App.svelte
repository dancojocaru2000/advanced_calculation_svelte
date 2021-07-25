<script>
	import { onMount } from 'svelte';
  import ParameterCard from './ParameterCard.svelte';

	onMount(() => {
		fetch('/api/ops')
			.then(res => res.json())
			.then(data => {
				if (data.status === 'ok') {
					operations = data.ops;
				}
			})
	})

	let operations = null;
	let operation = null;
  let params = [];
  let result = null;

	function onOperationSelected(op) {
		operation = op;
    params = Array(operation.args).fill(null);
    setTimeout(() => {
      // document.getElementById('calculation-pane').scrollIntoView()
    }, 0)
  }

  function debounce(fn, time) {
    let handle = null;
    return function() {
      const args = arguments
      if (handle) {
        clearTimeout(handle)
      }
      handle = setTimeout(() => {
        fn.apply(null, args)
      }, time)
    }
  }

  const fetchResult = debounce(() => {
    const url = new URL(`/api/calculate/${operation.name}`, globalThis.location);
    params.forEach((value, idx) => {
      url.searchParams.set(`param${idx}`, value.toString())
    })
    fetch(url.toString())
      .then(data => data.json())
      .then(data => {
        result = {
          result: data.result,
          opStr: data.opStr,
        }
      })
  }, 200);

  function paramChanged(event) {
    console.log(`paramChanged: ${event}`)
    const {paramNo, value} = event.detail;
    params[paramNo] = parseInt(value);

    if (!params.some(x => !x && x !== 0)) {
      fetchResult();
    }
    else {
      result = null;
    }
  }


	$: calculationMessage = operation ? 'Input parameters' : '<- Select Operation First';
  $: paramLoopHelper = operation ? Array(operation.args).fill(null).map((_, idx) => idx) : null;
</script>

<main>
	<h1>Advanced Calculation</h1>

	<div id="splitview">
		<div id="operation-selector">
			<h2>Select Operation</h2>

			{#if operations}
				<ul id="operations-list">
          {#each operations as op (op.name)}
            <li on:click={onOperationSelected(op)}>{op.name}</li>
          {/each}
				</ul>
			{:else}
				<p>Loading...</p>
			{/if}
		</div>
    {#if operation}
      <div id="right-side">
        <div id="params-pane">
          <h2>Input Parameters</h2>

          {#each paramLoopHelper as idx (idx)}
            <ParameterCard paramNo={idx} on:change={paramChanged}/>
          {/each}
        </div>
        {#if result}
          <div id="result-pane">
            <h2>Result</h2>
            <p>{result.opStr} = {result.result}</p>
          </div>
        {/if}
      </div>
    {/if}
	</div>
</main>

<style>
	#splitview {
		display: flex;
		align-items: stretch;
		flex-direction: row;
	}

  #splitview div {
    padding: 8px;
  }

  #right-side {
    display: flex;
    flex-direction: column;
    align-items: stretch;
  }
  
  #params-pane {
    flex-grow: 1;
  }

  #operations-list > li {
    text-align: start;
    list-style: none;
    margin: 8px;
    padding: 8px;
    cursor: pointer;
  }

  #operations-list > li:hover {
    background-color: lightblue;
  }

	#splitview > * {
		flex-grow: 1;
	}

	#splitview h2 {
		font-size: 1.2em;
	}
	
	@media (max-width: 700px) {
		#splitview {
			flex-direction: column;
		}
	}

	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>