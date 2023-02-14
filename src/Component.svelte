<script>
  import { getContext, onDestroy } from "svelte";

  export let field;
  export let label;
  export let width;
  export let height;
  export let onBlur;
  export let validation;
  export let onEnterKey;
  export let placeholder;
  export let defaultValue = null;

  const formContext = getContext("form");
  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let fieldApi;
  let fieldState;

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: formStep = formStepContext ? $formStepContext || 1 : 1;

  $: formField = formApi?.registerField(
    field,
    "text",
    defaultValue,
    false,
    validation,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: value = fieldState?.value || defaultValue;

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  $: overrideWdith = width ? width + "px" : "auto";
  $: overrideHeight = height ? height + "px" : "auto";

  const handleInputEnterKey = (e) => {
    value = e.target.value;

    if (e.keyCode === 13 && value && onEnterKey) {
      onEnterKey({ value });
    }
  };

  const handleInputBlur = (e) => {
    value = e.target.value;

    fieldApi?.setValue(value);

    if (e.type === "blur" && value && onBlur) {
      onBlur({ value });
    }
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<div class="spectrum-Form--labelsAbove">
  <div class="spectrum-Form-item" use:styleable={$component.styles}>
    {#if !formContext}
      <div class="placeholder">
        Form components need to be wrapped in a form
      </div>
    {:else}
      <label
        class:hidden={!label}
        for={fieldState?.fieldId}
        class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
      >
        {label || " "}
      </label>

      <div class="spectrum-Form-itemField">
        <div
          class="spectrum-Textfield"
          style={`width: ${overrideWdith};height:${overrideHeight};`}
        >
          <input
            {value}
            type="text"
            {placeholder}
            inputmode="text"
            on:blur={handleInputBlur}
            on:keyup={handleInputEnterKey}
            class="spectrum-Textfield-input"
          />
        </div>

        {#if fieldState?.error}
          <div class="error">{fieldState?.error}</div>
        {/if}
      </div>
    {/if}
  </div>
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
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>
