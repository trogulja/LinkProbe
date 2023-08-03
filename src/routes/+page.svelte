<script lang="ts">
  import { JSONEditor } from 'svelte-jsoneditor';
  import type { Content, OnChangeStatus, JSONContent } from 'svelte-jsoneditor';

  let inputString = 'http://app.productive.io.localhost/1-development-d-o-o/insights/draft?filter=eyJyZXBvcnRMYXlvdXRJZCI6IjIiLCJjaGFydFR5cGVJZCI6IjEiLCJidWRnZXRfc3RhdHVzIjp7ImVxIjoiMSJ9LCJkZWFsX3R5cGUiOnsiZXEiOiIyIn19&templateData=eyJmaWx0ZXJhYmxlQ29sbGVjdGlvbiI6InNlcnZpY2VzIiwibmFtZSI6Ik5ldyBTZXJ2aWNlcyBpbnNpZ2h0IiwicGFyYW1zIjp7ImJ1ZGdldFN0YXR1cyI6IjEifX0%3D&trigger=new-insight';
  let copyCopy = 'Copy';
  let url: URL | undefined = undefined;
  let content: Content = { json: {} };

  const base64test = /^([0-9a-zA-Z+/]{4})*(([0-9a-zA-Z+/]{2}==)|([0-9a-zA-Z+/]{3}=))?$/;

  const isStringURLEncoded = (str: string) => {
    return encodeURIComponent(decodeURIComponent(str)) === str;
  };

  const isStringBase64 = (str: string) => {
    return base64test.test(str);
  };

  const isStringBase64Encoded = (str: string) => {
    try {
      return atob(btoa(str)) === str;
    } catch (error) {
      return false;
    }
  };

  const isJSONContent = (obj: Content): obj is JSONContent => {
    return 'json' in obj;
  };

  const decodeBase64 = (encodedValue: string) => {
    try {
      return isStringBase64Encoded(encodedValue) && isStringBase64(encodedValue) ? atob(encodedValue) : encodedValue;
    } catch (error) {
      return encodedValue;
    }
  };

  const getSearchParams = () => {
    try {
      url = new URL(inputString);
      const params = Array.from(url.searchParams.entries()) ?? [];

      const decodedParams = params.map(([key, value]) => {
        const decodedValue = isStringURLEncoded(value) ? decodeBase64(decodeURIComponent(value)) : decodeBase64(value);

        try {
          return [key, JSON.parse(decodedValue)];
        } catch (error) {
          return [key, decodedValue];
        }
      });

      return decodedParams;
    } catch (error) {
      url = undefined;
      return [];
    }
  };

  const setContent = () => {
    try {
      content = { json: Object.fromEntries(getSearchParams()) };
    } catch (error) {
      content = { json: {} };
    }
  };

  const updateSearchParams = (content: Content) => {
    try {
      const searchParams = url?.searchParams;
      if (!searchParams) {
        return;
      }

      let changes = isJSONContent(content) ? content.json : JSON.parse(content.text);

      for (const [key, value] of searchParams) {
        const newValue = changes[key];

        if (newValue === undefined) {
          searchParams.delete(key);
          continue;
        }

        if (typeof newValue === 'object') {
          searchParams.set(key, encodeURIComponent(btoa(JSON.stringify(newValue))));
          continue;
        }

        searchParams.set(key, encodeURIComponent(newValue));
      }

      inputString = url ? url.href : '';
    } catch (error) {
      return '';
    }
  }

  function copyToClipboard(this: HTMLButtonElement) {
    copyCopy = 'Copied!';
    this.blur();
    setTimeout(() => {
      copyCopy = 'Copy';
    }, 1000);
    navigator?.clipboard.writeText(inputString);
  }

  function handleInput(this: HTMLInputElement) {
    inputString = this.value;
    setContent();
  }

  function handleEdit(updatedContent: Content, previousContent: Content, { contentErrors, patchResult }: OnChangeStatus) {
    content = updatedContent;
    updateSearchParams(updatedContent);
  }

  setContent();
</script>

<nav class="container-fluid">
  <ul>
    <li><span class="contrast">LinkProbe: R2 Edition</span></li>
  </ul>
</nav>

<main class="container">
  <article class="grid">
    <div>
      <hgroup>
        <h1>URL input</h1>
        <h2>Paste your URL here and our R2 unit will try it's best to decode it</h2>
      </hgroup>

      <div class="grid inline-input">
        <input type="text" value={inputString} on:change={handleInput}>
        <button on:click={copyToClipboard}>{copyCopy}</button>
      </div>
    </div>
  </article>

  <article class="json-editor jse-theme-dark">
    <JSONEditor {content} onChange="{handleEdit}" />
  </article>
</main>

<footer class="container-fluid">
  <small>
    Built with ❤️
  </small>
</footer>

<style lang="scss">
  @import 'svelte-jsoneditor/themes/jse-theme-dark.css';

  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    min-height: calc(100vh - 7rem);
    padding: 1rem 0;
  }

  .inline-input {
    grid-template-columns: repeat(5, 1fr);
  }

  .inline-input > input { grid-area: 1 / 1 / 2 / 5; }
  .inline-input > button { grid-area: 1 / 5 / 2 / 6; }

  .json-editor {
    --jse-background-color: var(--background-color);
    --line-height: normal;
    font-size: var(--jse-font-size-mono) !important;
  }
</style>
