<script lang="ts">
  import { JSONEditor } from 'svelte-jsoneditor';
  let inputString = 'http://app.productive.io.localhost/1-development-d-o-o/insights/draft?filter=eyJyZXBvcnRMYXlvdXRJZCI6IjIiLCJjaGFydFR5cGVJZCI6IjEiLCJidWRnZXRfc3RhdHVzIjp7ImVxIjoiMSJ9LCJkZWFsX3R5cGUiOnsiZXEiOiIyIn19&templateData=eyJmaWx0ZXJhYmxlQ29sbGVjdGlvbiI6InNlcnZpY2VzIiwibmFtZSI6Ik5ldyBTZXJ2aWNlcyBpbnNpZ2h0IiwicGFyYW1zIjp7ImJ1ZGdldFN0YXR1cyI6IjEifX0%3D&trigger=new-insight';
  let copyCopy = 'Copy result to clipboard';

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

  const handleDecodedString = (dValue: string) => {
    console.log(dValue);
    try {
      return isStringBase64Encoded(dValue) && isStringBase64(dValue) ? JSON.parse(atob(dValue)) : dValue;
    } catch (error) {
      return dValue;
    }
  };

  const getSearchParams = (inputString: string) => {
    try {
      const url = new URL(inputString);
      const params = Array.from(url.searchParams.entries()) ?? [];

      const decodedParams = params.map(([key, value]) => {
        return isStringURLEncoded(value) ? [key, handleDecodedString(decodeURIComponent(value))] : [key, handleDecodedString(value)];
      });

      return decodedParams;
    } catch (error) {
      return [];
    }
  };

  const getConstructedUrl = () => {
    try {
      const url = new URL(inputString);
      searchParams.forEach(([key, value]) => {
        const oldValue = url.searchParams.get(key);
        const needUrlEncode = oldValue && isStringURLEncoded(oldValue);
        if (needUrlEncode) {
          const oldValueDecoded = decodeURIComponent(oldValue);
          const needbase64Encode = isStringBase64Encoded(oldValueDecoded) && isStringBase64(oldValueDecoded);
          if (needbase64Encode) {
            url.searchParams.set(key, encodeURIComponent(btoa(value)));
          } else {
            url.searchParams.set(key, encodeURIComponent(value));
          }
        } else {
          const needbase64Encode = oldValue && isStringBase64Encoded(oldValue) && isStringBase64(oldValue);
          if (needbase64Encode) {
            url.searchParams.set(key, btoa(value));
          } else {
            url.searchParams.set(key, value);
          }
        }
      });

      return url.href;
    } catch (error) {
      return '';
    }
  }

  function copyToClipboard(this: HTMLButtonElement) {
    copyCopy = 'Copied!';
    this.blur();
    setTimeout(() => {
      copyCopy = 'Copy result to clipboard';
    }, 1000);
    navigator?.clipboard.writeText(getConstructedUrl());
  }

  $: searchParams = inputString ? getSearchParams(inputString) : [];
  $: constructedUrl = searchParams.length ? getConstructedUrl() : '';
  $: content = searchParams.length ? { json: Object.fromEntries(searchParams) } : { json: {} };
</script>

<nav class="container-fluid">
  <ul>
    <li><span class="contrast">LinkProbe: R2 Edition</span></li>
  </ul>
</nav>

<main class="container">
  <article class="json-editor jse-theme-dark">
    <JSONEditor bind:content />
  </article>

  <article class="grid">
    <div>
      <hgroup>
        <h1>URL input</h1>
        <h2>Paste your URL here and our R2 unit will try it's best to decode it</h2>
      </hgroup>

      <div class="grid inline-input">
        <input type="text" bind:value={inputString}>
        <button>Probe</button>
      </div>
    </div>
  </article>

  <article>
    <hgroup>
      <h1>searchParams</h1>
      <h2>Here's what we got</h2>
    </hgroup>

    {#each searchParams as [key, value], i}
      <label for={`qp${i + 1}`}>
        {key}
        <input type="text" id={`qp${i + 1}`} bind:value={value}>
      </label>
    {/each}
  </article>

  <article>
    <h1>Results</h1>

    <h6>input</h6>
    <pre class="url-block">{inputString}</pre>

    <h6>result</h6>
    <pre class="url-block">{constructedUrl}</pre>

    <button on:click={copyToClipboard}>{copyCopy}</button>
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

  .url-block {
    white-space: pre-line;
  }

  .json-editor {
    --jse-background-color: var(--background-color);
    --line-height: normal;
    font-size: var(--jse-font-size-mono) !important;
  }
</style>
