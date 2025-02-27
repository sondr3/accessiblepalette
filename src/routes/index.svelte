<script lang="ts">
  import Palette from '../components/Palette.svelte';
  import Description from '../components/Description.svelte';
  import { onDestroy, onMount } from 'svelte';
  import {
    baseColors,
    lightnessShades,
    storeAsURL,
    defaultColors,
    defaultLightness,
    defaultStateAsURL
  } from '../stores';
  import type { BaseColor, LightnessInterface } from '../stores';
  import { nanoid } from 'nanoid';
  import chroma from 'chroma-js';
  import type { Unsubscriber } from 'svelte/store';
  import Rainbow from '../components/Rainbow.svelte';
  import Toolbar from '../components/Toolbar.svelte';

  function saveStateToURL(params: string): void {
    window.history.pushState([$lightnessShades, $baseColors], '', '?' + params);
  }

  function decodeStoreFromParams(params: string): [LightnessInterface, BaseColor[]] {
    const paramsObj = new URLSearchParams(params);
    let lightness: LightnessInterface = Object.assign({}, $lightnessShades);
    let colors: BaseColor[] = [];
    let failed = false;

    paramsObj.forEach((value, key) => {
      const values = value.split(',');

      if (key === 'lightness') {
        // Parse lightness
        if (values.length === Object.keys(lightness).length) {
          Object.keys(lightness).forEach((key: string, index: number) => {
            lightness[key] = parseFloat(values[index]);
          });
        } else {
          failed = true;
          console.log('Unexpected number of lightness shades in the URL.');
        }
      } else {
        // Parse color
        if (chroma.valid(key)) {
          colors.push({
            name: nanoid(8),
            color: '#' + key,
            isLab: values[0] == '1',
            hueCorrection: parseInt(values[1], 10) || 0
          });
        } else {
          failed = true;
          console.log(`${key} is not a valid color`);
        }
      }
    });

    if (failed) {
      return undefined;
    } else {
      return [lightness, colors];
    }
  }

  function updateState(newLightness: LightnessInterface, newBaseColors: BaseColor[]) {
    lightnessShades.set(newLightness);
    baseColors.set(newBaseColors);
  }

  function recoverState(event: PopStateEvent) {
    const { state } = event;

    if (state && Object.keys(state).length === 0) {
      // console.log('🏁 Blank state. Nothing to restore.');
      updateState(defaultLightness, defaultColors);
    } else {
      // console.log('↔️ We have state to recover!');
      updateState(state[0], state[1]);
    }
  }

  let unsubscribeFromStoreAsURL: Unsubscriber;

  onMount(() => {
    // Whenever state is changed, save it to the URL
    unsubscribeFromStoreAsURL = storeAsURL.subscribe((state: string): void => {
      if (state === defaultStateAsURL) {
        // console.log('State set to the default.');
      } else if (state === window.location.search.substring(1)) {
        // console.log('State changed. No need to update browser history as URL params match state.');
      } else {
        // console.log('State changed. Update browser history.');
        saveStateToURL(state);
      }
    });

    // Get state from URL parameters
    if (window.location.search.substring(1) !== '' && window.location.search.substring(1) !== $storeAsURL) {
      const decodedState = decodeStoreFromParams(window.location.search.substring(1));
      if (decodedState) {
        // console.log('URL params were successfully decoded.')
        updateState(decodedState[0], decodedState[1]);
      }
    }

    window.addEventListener('popstate', recoverState, false);
  });

  onDestroy(() => {
    if (typeof window !== 'undefined') {
      unsubscribeFromStoreAsURL();
      window.removeEventListener('popstate', recoverState, false);
    }
  });
</script>

<Toolbar />

<Palette />

<Rainbow />

<Description />

<Rainbow />

<style>
  /* https://accessiblepalette.com/?lightness=98,93,88,80,70,60,50,38,22,14&ff6f50=0,0&34d275=1,0&647899=1,0 */

  :global(:root) {
    --c-bg: #FBFBFC;

    --c-slate-50: #f9f9fb;
    --c-slate-100: #e9ebf0;
    --c-slate-200: #d9dde6;
    --c-slate-300: #c0c7d5;
    --c-slate-400: #a1acc1;
    --c-slate-500: #8291ad;
    --c-slate-600: #647899;
    --c-slate-700: #4c5a72;
    --c-slate-800: #2e3542;
    --c-slate-900: #20242b;

    --c-red-100: #ffe5e0;
    --c-red-600: #c3553d;

    --c-green-100: #d0f4d8;
    --c-green-600: #2c874e;

    /* https://accessiblepalette.com/?lightness=100,100,100,100,60,50,40,0,0,0&ea6856=0,0&b18b32=0,-10&43a447=0,0&21a19c=0,0&2c9acc=1,15&9085de=0,0 */
    --rainbow-40: to right, #994438, #795822, #2C6B2E, #166966, #346285, #5E5791;
    --rainbow-50: to right, #C15647, #b18b32, #37873A, #1B8480, #3B7DAA, #776EB7;
    --rainbow-60: to right, #EA6856, #b18b32, #43a447, #21a19c, #2c9acc, #9085de;

    /* Padding */
    --pad: 1.25rem; /* 20px */
    --pad2x: 2.5rem; /* 40px */
  }
  /*
  414px: iPhone XR @2x, XS Max @3x, 11 @2x, 11 Pro Max @3x
  428px: iPhone 12 Pro Max @3x
  */
  @media only screen and (min-width: 414px) {
    :global(:root) {
      --pad: 1.875rem; /* 30px */
      --pad2x: 3.75rem; /* 60px */
    }
  }
  /*
  1194px: Landscape iPad 11" (Pro 1-2nd)
  1280px: Full-screen MB 13", half-screen iMac 27"
  */
  @media only screen and (min-width: 1194px) {
    :global(:root) {
      --pad: 2.5rem; /* 40px */
      --pad2x: 5rem; /* 80px */
    }
  }



  /* Global */

  :global(body) {
    min-height: 100vh;
    margin: 0;
    padding: 0;
    font: 1rem Inter, sans-serif;
  }
  :global(input[type='number'], input[type='text']) {
    margin: 0;
    border: 1px solid var(--c-slate-200);
    border-radius: 2px;
    padding: 0.2em 0.4em;
    font-family: Inter, sans-serif;
  }
  :global(input[type='number']) {
    padding-right: 0;
  }
  :global(input[type='number']:focus, input[type='text']:focus) {
    outline: none;
    box-shadow: 0 0 0 2px #dceff8;
  }
</style>
