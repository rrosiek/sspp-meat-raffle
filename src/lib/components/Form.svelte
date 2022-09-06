<script lang="ts">
  import { createEventDispatcher } from "svelte";

  export let schema = null;

  let errors = [];
  let inError = false;

  const dispatch = createEventDispatcher();
  const extraElements = {};

  $: hasError = (name) => {
    return errors.includes(name);
  };

  const handleExtra = (event) => {
    extraElements[event.detail.name] = event.detail.value;

    dispatch("extra", extraElements);
  };

  const handleSubmit = async (event) => {
    errors = [];
    inError = false;

    const formData = [].filter
      .call(event.target.elements, (el) => el.name !== "")
      .reduce((data, el) => {
        data[el.name] = el.type === "file" ? el.files : el.value;

        return data;
      }, {});

    let mergedFormData = Object.assign(formData, extraElements);

    try {
      if (schema) {
        await schema.validate(mergedFormData, { abortEarly: false });
      }

      dispatch("validated", mergedFormData);
    } catch (error) {
      console.log(error);
      inError = true;

      if (error.inner) {
        errors = error.inner.map((e) => e.path);
      }
    }
  };
</script>

<form on:submit|preventDefault={handleSubmit}>
  <slot extra={handleExtra} {hasError} />
</form>
