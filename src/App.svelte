<script lang="ts">
  import {
    BehaviorSubject,
    fromEventPattern,
    map,
    throttleTime,
  } from 'rxjs';
  import List from './lib/List.svelte';
  import type { ItemObject } from './vite-env';
  import { numberToMark } from 'pinyin-utils';

  import { selected } from './lib/store';
  import Hanzi from './lib/Hanzi.svelte';

  import DictWorker from './worker?worker';
  import { onMount } from 'svelte';

  const worker = new DictWorker();
  const input = new BehaviorSubject('');
  let simplified = true;

  const data = fromEventPattern<MessageEvent<ItemObject[]>>(
    (handler) => worker.addEventListener('message', handler),
    (handler) => worker.removeEventListener('message', handler),
  ).pipe(map((e) => e.data));

  onMount(() => {
    input
      .pipe(
        throttleTime(300, undefined, {
          leading: true,
          trailing: true,
        }),
      )
      .subscribe((value) => {
        worker.postMessage(value);
      });
  });

  $: items = $data || [];
  $: pinyin = $selected?.pinyin as string;

  $: related =
    items?.filter((d) => d.pinyin.includes(pinyin)) || [];

  $: hanzi = $selected
    ? $selected.hanzi.split(' ')[simplified ? 1 : 0]
    : '';
</script>

<main class="mx-auto max-w-[320px] p-4 pt-6 text-slate-50">
  {#if $selected}
    <section
      class="mx-auto p-6 rounded flex flex-col gap-y-8 text-center mb-10 border border-neutral-700"
    >
      <div>
        <Hanzi chars={hanzi} />
        汉字
        <span class="text-neutral-600 absolute ml-3 font-thin"
          >Hàn Zi</span
        >
      </div>

      <div>
        <span class="text-xl mb-4">
          {numberToMark($selected.pinyin)}
        </span>
        <br />
        拼音
        <span class="text-neutral-600 absolute ml-3 font-thin"
          >Pīn Yin</span
        >
      </div>

      <div>
        <span class="text-xl mb-4">
          {$selected.def.replaceAll('/', ' — ')}
        </span>
        <br />
        定義
        <span class="text-neutral-600 absolute ml-3 font-thin"
          >Definition</span
        >
      </div>
    </section>
  {/if}

  <label
    for="combobox"
    class="block text-2xl font-bold text-center mb-2"
  >
    Type your 拼音
    <span class="text-neutral-600 absolute ml-3 font-thin"
      >Pīn Yin</span
    >
  </label>
  <div class="relative mt-1">
    <input
      on:input={(e) => input.next(e.currentTarget.value)}
      id="combobox"
      type="text"
      autofocus
      class="w-full rounded-md border border-gray-300 bg-neutral-900 py-2 pl-3 pr-12 shadow-sm focus:border-indigo-600 focus:outline-none focus:ring-1 focus:ring-indigo-600 sm:text-sm"
      role="combobox"
      aria-controls="options"
      aria-expanded={!!items.length}
    />

    <fieldset class="absolute">
      <input
        bind:checked={simplified}
        type="checkbox"
        name="simplified"
        id="simplified"
        class="h-3 w-3 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
      />
      <label
        for="simplified"
        class="mt-1 text-neutral-600 text-sm absolute ml-2 font-thin"
        >Simplified?</label
      >
    </fieldset>

    <List items={items?.slice(0, 100)} {simplified} />
  </div>

  <ul class="my-16">
    {#each related as item}
      <li class="mb-4">
        {@html item.hanzi
          .split(' ')
          [simplified ? 1 : 0].replaceAll(
            hanzi,
            `<span class="text-indigo-600">${hanzi}</span>`,
          )}

        <span class="text-neutral-400 ml-1 font-extralight">
          {numberToMark(item.pinyin)}
        </span>
        <p class="font-light">
          {item.def}
        </p>
      </li>
    {/each}
  </ul>

  <section
    class="font-thin mt-16 text-neutral-600 hover:text-neutral-400"
  >
    <h3 class="tracking-wide leading-8">Instructions</h3>

    <p class="text-sm">
      In the input box, you can type Pīn Yin, Hàn Zi, or English.
      It will give you a list of options that matches best your
      input. When using Pīn Yin, you can use the tone marks (ā,
      á, ǎ, à, a) or the numbers (1, 2, 3, 4, 5). E.g. "zhong3"
      or "zhǒng". Once selected, a list of related words will
      appear below the input box.
    </p>
  </section>
</main>
