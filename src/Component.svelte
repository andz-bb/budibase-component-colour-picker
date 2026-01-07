<script>
  import { getContext, onDestroy, onMount } from "svelte";
  import { ColorPicker, Color } from "svelte-colorpick";

  export let field;
  export let label;

  export let customPreviewSize;
  export let previewSize;

  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let color;
  let fieldApi;
  let fieldState;
  let background = "#ffffff";

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: previewDim = previewSize === "custom" ? customPreviewSize : previewSize;

  $: {
    const themeRoot = document.getElementById("theme-root");
    background = themeRoot
      ? getComputedStyle(themeRoot).getPropertyValue("--navBackground")
      : "#ffffff";
  }

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "text",
    "#000000",
    false,
    null,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  onMount(() => {
    color = Color.hex(fieldState?.value);
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Form-itemField">
      {#if !field}
        <div class="error">Please select a field</div>
      {:else}
        <ColorPicker
          bind:color
          on:pickerCollapse={() => fieldApi?.setValue(color.toHex())}
          collapse={true}
          {background}
          handleWidth={previewDim}
          handleHeight={previewDim}
          tabbed={false}
          selectedTab="hsl"
          selectedDimension="hsl.h"
          showHex={true}
          showLabels={false}
          showMatrix={true}
          showNumeric={false}
          showSliders={{
            "hsl.h": true,
          }}
          selectDimensions={false}
          matrixWidth={150}
          matrixHeight={150}
          scrollbarHeight={10}
        />
        {#if fieldState?.error}
          <div class="error">{fieldState.error}</div>
        {/if}
      {/if}
    </div>
  {/if}
</div>

<style>
  .placeholder {
    color: var(--spectrum-global-color-gray-600);
  }
  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-300);
  }
</style>
