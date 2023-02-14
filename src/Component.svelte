<script>
  import { getContext, onDestroy } from "svelte";

  export let field;
  export let width;
  export let height;
  export let onBlur;
  export let onEnterKey;
  export let placeholder;
  export let defaultValue;

  const formContext = getContext("form");
  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formStepContext = getContext("form-step");

  let fieldApi;
  let fieldState;
  let value = defaultValue;

  const formApi = formContext?.formApi;
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "text",
    "",
    false,
    null,
    formStep
  );
  $: unsubscribe = formField?.subscribe((value) => {
    fieldApi = value?.fieldApi;
    fieldState = value?.fieldState;
  });

  if (width) $component.styles.normal.width = width + "px";

  if (height) $component.styles.normal.height = height + "px";

  const handleInputEnterKey = (e) => {
    if (e.keyCode === 13 && value && onEnterKey) {
      onEnterKey({ value });
    }
  };

  const handleInputBlur = (e) => {
    if (e.type === "blur" && value && onBlur) {
      fieldApi?.setValue(value);

      onBlur({ value });
    }
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

{#if !formContext}
  <div class="placeholder">Form components need to be wrapped in a form</div>
{:else}
  <div class="spectrum-Form-itemField" use:styleable={$component.styles}>
    <div class="spectrum-Textfield" style="width:{width};height:{height};">
      <input
        bind:value
        type="text"
        {placeholder}
        inputmode="text"
        on:blur={handleInputBlur}
        on:keyup={handleInputEnterKey}
        class="spectrum-Textfield-input"
      />
    </div>

    {#if fieldState.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  </div>
{/if}

<style>
  .spectrum-Textfield {
    width: 100%;
  }

  .placeholder {
    color: var(--spectrum-global-color-gray-600);
  }

  .spectrum-Form-itemField {
    position: relative;
  }

  .spectrum-Textfield-input {
    padding: 10px;
    border-radius: 5px;
    text-align: left;
  }

  .spectrum-Textfield-input:focus {
    border-color: var(
      --spectrum-textfield-m-border-color-down,
      var(--spectrum-alias-border-color-mouse-focus)
    );
  }

  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
</style>