<script lang="ts">
  import Hanzi from 'hanzi-writer';

  export let chars = '';

  let container: HTMLDivElement | null = null;

  $: {
    const els = [...chars].map((char) => {
      const el = document.createElement('div');

      const w = Math.min(238 / chars.length, 75);
      let hz = new Hanzi(el, {
        strokeColor: '#fff',
        outlineColor: '#222',
        strokeAnimationSpeed: 1,
        delayBetweenStrokes: 250,
        delayBetweenLoops: 1500,
        width: w,
        padding: 0,
        height: w,
        charDataLoader: () =>
          fetch(
            `https://cdn.jsdelivr.net/npm/hanzi-writer-data/${char}.json`,
          )
            .then((res) => res.json())
            .then((data) => {
              return data;
            })
            .catch(() => {
              el.innerHTML = char;

              return {};
            }),
      });

      hz.setCharacter(char);
      hz.loopCharacterAnimation();

      return el;
    });

    if (container) container.replaceChildren(...els);
  }
</script>

<div
  bind:this={container}
  class="w-full flex justify-center items-center -mt-2 mb-2 font-bold text-xl"
>
  {chars}
</div>
