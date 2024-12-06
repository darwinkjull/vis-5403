<script>
  export let selectedCondition1 = "Acute Coronary Syndrome";
  export let selectedCondition2 = "Angina";
  export let selectedScale = "Frequency";
  export let selectedSide = "same";
  export let enableCondition2 = true; // Toggle for Condition Selector 2

  function updateCondition1(event) {
    selectedCondition1 = event.target.value; // Update condition based on selection
  }
  function updateCondition2(event) {
    selectedCondition2 = event.target.value; // Update condition based on selection
  }
  function updateScale(event) {
    selectedScale = event.target.value; // Update scale based on selection
  }
  function updateSides(event) {
    selectedSide = event.target.value; // Update sides based on selection
  }
  function toggleCondition2() {
    enableCondition2 = !enableCondition2; // Toggle Condition Selector 2 visibility
  }
</script>

<div class="plot-controls">
  <h3>Histogram Controls</h3>
  <div class="selectors">
    <!-- Left Column: Grouped Smaller Selector Boxes -->
    <div class="left-group">
      <div class="horizontal-box">
        <h4>
          Y Axis Scale: 
          <span class="tooltip-icon" title="Choose between Frequency or Relative Frequency for the Y-axis">
            ⓘ
          </span>
        </h4>
        <label><input type="radio" bind:group={selectedScale} value="Frequency" on:change={updateScale}> Frequency </label>
        <label><input type="radio" bind:group={selectedScale} value="Relative" on:change={updateScale}> Relative Frequency </label>
      </div>
      <div class="horizontal-box">
        <h4>
          X Axis Sides: 
          <span class="tooltip-icon" title="Choose whether to display bars on the same or opposite sides of the X-axis">
            ⓘ
          </span>
        </h4>
        <label><input type="radio" bind:group={selectedSide} value="same" on:change={updateSides}> Same </label>
        <label><input type="radio" bind:group={selectedSide} value="opposite" on:change={updateSides}> Opposite </label>
      </div>
    </div>

    <!-- Middle Column: Condition Selector 1 -->
    <div class="vertical-box">
      <h4>
        Condition Selector 1
        <span class="tooltip-icon" title="Select the first condition to visualize">
          ⓘ
        </span>
      </h4>
      <label><input type="radio" bind:group={selectedCondition1} value="Acute Coronary Syndrome" on:change={updateCondition1}> Acute Coronary Syndrome</label>
      <label><input type="radio" bind:group={selectedCondition1} value="Angina" on:change={updateCondition1}> Angina</label>
      <label><input type="radio" bind:group={selectedCondition1} value="Arrhythmias" on:change={updateCondition1}> Arrhythmias</label>
      <label><input type="radio" bind:group={selectedCondition1} value="Atrial Fibrillation" on:change={updateCondition1}> Atrial Fibrillation</label>
      <label><input type="radio" bind:group={selectedCondition1} value="Cardiac Amyloidosis" on:change={updateCondition1}> Cardiac Amyloidosis</label>
      <label><input type="radio" bind:group={selectedCondition1} value="Cardiomyopathy" on:change={updateCondition1}> Cardiomyopathy</label>
    </div>

    <!-- Right Column: Condition Selector 2 -->
    <div class="vertical-box">
      <div class="title-row">
        <h4>
          Condition Selector 2
          <span class="tooltip-icon" title="Select the second condition to visualize or toggle it on/off">
            ⓘ
          </span>
        </h4>
        <button on:click={toggleCondition2}>{enableCondition2 ? "Disable" : "Enable"}</button>
      </div>
      {#if enableCondition2}
        <label><input type="radio" bind:group={selectedCondition2} value="Acute Coronary Syndrome" on:change={updateCondition2}> Acute Coronary Syndrome</label>
        <label><input type="radio" bind:group={selectedCondition2} value="Angina" on:change={updateCondition2}> Angina</label>
        <label><input type="radio" bind:group={selectedCondition2} value="Arrhythmias" on:change={updateCondition2}> Arrhythmias</label>
        <label><input type="radio" bind:group={selectedCondition2} value="Atrial Fibrillation" on:change={updateCondition2}> Atrial Fibrillation</label>
        <label><input type="radio" bind:group={selectedCondition2} value="Cardiac Amyloidosis" on:change={updateCondition2}> Cardiac Amyloidosis</label>
        <label><input type="radio" bind:group={selectedCondition2} value="Cardiomyopathy" on:change={updateCondition2}> Cardiomyopathy</label>
      {/if}
    </div>
  </div>
</div>

<style>
  .plot-controls {
    margin-top: 20px;
    padding: 20px;
    background-color: #f1f1f1;
    border: 1px solid #ccc;
    border-radius: 8px;
    max-width: 800px; /* Same width as visualization */
    margin-left: auto;
    margin-right: auto;
  }

  .plot-controls h3 {
    text-align: center;
    margin-bottom: 20px;
  }

  .selectors {
    display: flex;
    justify-content: space-between;
    gap: 20px;
    align-items: flex-start;
  }

  /* Left Column: Grouped Smaller Selector Boxes */
  .left-group {
    display: flex;
    flex-direction: column;
    gap: 20px;
    flex: 1;
  }

  /* Middle and Right Columns */
  .vertical-box {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #ffffff;
    flex: 1;
  }

  .horizontal-box {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #ffffff;
  }

  h4 {
    margin-bottom: 10px;
    font-size: 14px;
    font-weight: bold;
  }

  .tooltip-icon {
    font-size: 12px;
    color: #007BFF;
    cursor: pointer;
    margin-left: 5px;
  }

  .title-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  label {
    display: block;
    margin-bottom: 5px;
  }
</style>
