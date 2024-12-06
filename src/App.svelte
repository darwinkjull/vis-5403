<script>
	import Header from './components/Header.svelte';
	import Visualization from './components/Visualization.svelte';
	import Selectors from './components/Selectors.svelte';

	let selectedCondition1 = "Acute Coronary Syndrome"; // Default condition
	let selectedCondition2 = "Acute Coronary Syndrome";
	let selectedScale = "Frequency"; // Default y axis scale (count vs relative)
	let selectedSide = "same";
	let enableCondition2 = false;
  </script>
  
  <main>
	<Header />
	<div class="content">
	  <div class="visualization-container">
  		<Visualization {selectedCondition1} {selectedCondition2} {selectedScale} {selectedSide} {enableCondition2} />
	  </div>
	  <div class="selectors-container">
		<Selectors bind:selectedCondition1 bind:selectedCondition2 bind:selectedScale bind:selectedSide bind:enableCondition2/>
	  </div>
	</div>
  </main>
  
  <style>
	main {
	  font-family: Arial, sans-serif;
	  padding: 0;
	  margin: 0;
	}
  
	.content {
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	  margin: 20px;
	}
  
	.visualization-container {
	  width: 80%;
	  align-items: center;
	}
  
	.selectors-container {
	  justify-content: space-between;
	  width: 80%;
	  align-items: center;
	}
  </style>
  