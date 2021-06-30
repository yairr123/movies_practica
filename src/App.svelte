<script>
  import MoviItem from "./Movie/Item.svelte";
  import { fly } from "svelte/transition";
  import { dataset_dev } from "svelte/internal";

  const APIKEY = "680ee273b817fc5b1516448d8ad88a21";
  const BASEURL = `https://api.themoviedb.org/3`;
  const APISETTINGS = `?api_key=${APIKEY}&language=en-US`;

  const movies = (async () => {
    const URL = `${BASEURL}/discover/movie/${APISETTINGS}&sort_by=popularity.desc`;
    const response = await fetch(URL);
    return await response.json();
  })();

  let likedMovies = [];

  function togglelike(event) {
    const movie = event.detail;

    let index = likedMovies.findIndex((m) => m.id === movie.id);
    if (index >= 0) {
      likedMovies.splice(index, 1);
      console.log(likedMovies);
    } else {
      likedMovies.push(movie);
    }
    likedMovies = likedMovies;
  }

  $: like = (id) => {
    let index = likedMovies.findIndex((m) => m.id === id);
    return index >= 0;
  };

  let time = 0;
	let duration;
	let paused = true;

	let showControls = true;
	let showControlsTimeout;
  		//Ver los controles tiempo
	function handleMousemove(e) {
		// Make the controls visible, but fade out after
		// 2.5 seconds of inactivity
		clearTimeout(showControlsTimeout);
		showControlsTimeout = setTimeout(() => showControls = false, 2500);
		showControls = true;

		if (!(e.buttons & 1)) return; // mouse not down
		if (!duration) return; // video not loaded yet

		const { left, right } = this.getBoundingClientRect();
		time = duration * (e.clientX - left) / (right - left);
	}
		//escuchar click
	function handleMousedown(e) {
		// we can't rely on the built-in click event, because it fires
		// after a drag — we have to listen for clicks ourselves

		function handleMouseup() {
			if (paused) e.target.play();
			else e.target.pause();
			cancel();
		}

		function cancel() {
			e.target.removeEventListener('mouseup', handleMouseup);
		}

		e.target.addEventListener('mouseup', handleMouseup);

		setTimeout(cancel, 200);
	}
		//
	function format(seconds) {
		if (isNaN(seconds)) return '...';

		const minutes = Math.floor(seconds / 60);
		seconds = Math.floor(seconds % 60);
		if (seconds < 10) seconds = '0' + seconds;

		return `${minutes}:${seconds}`;
	}
 	
</script>

<svelte:head>
  <title>Movies Svelte</title>
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css"
    integrity="undefined"
    crossorigin="anonymous"
  />
</svelte:head>


<center >
	<h1 > 
		El cortometraje de la semana
	</h1>
	<a href="http://www.caminandes.com/">Haz click para descubrir más</a>
	<div>
		<video
		poster="https://sveltejs.github.io/assets/caminandes-llamigos.jpg"
		src="https://sveltejs.github.io/assets/caminandes-llamigos.mp4"
			on:mousemove={handleMousemove}
			on:mousedown={handleMousedown}
			bind:currentTime={time}
			bind:duration
			bind:paused>
				<track kind="captions"/>
		</video>
	
		<div class="controls" style="opacity: {duration && showControls ? 1 : 0}">
			<progress value="{(time / duration) || 0}"/>
	
			<div class="info">
				<span class="time">{format(time)}</span>
				<span>haga clic en cualquier lugar para{paused ? 'play' : 'pause'} /arrastrar para buscar</span>
				<span class="time">{format(duration)}</span>
			</div>
		</div>
	</div>
	
</center>


<main class="container">
  <div class="row">
    <div class="col-12 col-md-6 col-lg-8 border panel">
      <h2>Peliculas populares</h2>
      <div class="row">
        {#await movies}
          <!-- promise is pending -->
          <div class="col-12">
            <p>Cargando Datos . . . .</p>
          </div>
        {:then data}
          {@debug data}
          <!-- promise was fulfilled -->
          {#each data.results as movie}
            <div class="col-12 col-md-6 col-lg-4 p-2">
              <MoviItem
                like={like(movie.id)}
                id={movie.id}
                title={movie.title}
                overview={movie.overview}
                cover={movie.poster_path}
                on:onToggleLike={togglelike}
              />
            </div>
          {/each}
        {/await}
      </div>
    </div>
    <div class="col-12 col-md-6 col-lg-4 border panel ">
      <h2>Peliculas favoritas</h2>
      <div class="row">
        {#if likedMovies.length}
          {#each likedMovies as movie, i (movie.id)}
            <div
              in:fly={{ duration: 2000, y: 20 }}
              out:fly={{ duration: 1000, y: -20 }}
              class="col-12 col-md-6 col-lg-4 p-2"
            >
              <MoviItem
                like={like(movie.id)}
                id={movie.id}
                title={movie.title}
                overview=""
                cover={movie.cover}
                on:onToggleLike={togglelike}
              />
            </div>
          {/each}
        {:else}
          <div class="col-12">
            <p>No Tienes peliculas favoritas...</p>
          </div>
        {/if}
      </div>
    </div>
  </div>
</main>

<style>
	center{
		background-color: dimgray;
	}
  .panel {
    height: 100vh;
    overflow: auto;
  }

  main {
    text-align: center;
    padding: 0;
    max-width: 240px;
    margin: 0 auto;
    background-color: dimgray;
  }
  

  top{
		font-family: sans-serif;
        text-align: center;		
		
  }
  

  .controls {
		align-items: center;
		top: 100%;
		width: 80%;
		transition: opacity 1s;
	}

	.info {
		display: flex;
		width: 100%;
		justify-content: space-between;
	}

	span {
		padding: 0.2em 0.5em;
		color: white;
		text-shadow: 0 0 8px black;
		font-size: 1.4em;
		opacity: 0.7;
	}

	.time {
		width: 3em;
	}

	.time:last-child { text-align: right }

	progress {
		display: block;
		width: 100%;
		height: 10px;
		-webkit-appearance: none;
		appearance: none;
	}

	progress::-webkit-progress-bar {
		background-color: rgba(0,0,0,0.2);
	}

	progress::-webkit-progress-value {
		background-color: rgba(255,255,255,0.6);
	}

	video {
		width: 80%;
	}

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
	
  }
</style>