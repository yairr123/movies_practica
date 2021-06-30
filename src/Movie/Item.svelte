<script>
  import { createEventDispatcher } from "svelte";
  export let id;
  export let title;
  export let overview;
  export let cover;
  export let like;

  $: coverURL = `https://image.tmdb.org/t/p/w185_and_h278_bestv2${cover}`;

  const dispatch = createEventDispatcher();

  function togglelike() {
    dispatch("onToggleLike", { id, title, overview, cover });
  }
</script>


<div
  class="card text-white bg-dark mb-3 card border-dark mb-3 "
  style="max-width: 18rem;"
>
  <img src={coverURL} class="card-img-top" alt="..." />
  <div class="card-body">
    <h5 class="card-title">{title}</h5>
    <p class="card-text">{overview}</p>
    <div class="text-center">
      <button
        class={like ? "btn btn-secondary" : "btn btn-primary"}
        on:click={togglelike}>{like ? "Ya no me gusta" : "Me gusta"}</button
      >
    </div>
  </div>
</div>

