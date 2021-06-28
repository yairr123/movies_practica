<script>
	import MoviItem from "./Movie/Item.svelte";
	import { fly } from 'svelte/transition';

import { dataset_dev } from "svelte/internal";
  
	const APIKEY = "680ee273b817fc5b1516448d8ad88a21";  
	const BASEURL = `https://api.themoviedb.org/3`;
	const APISETTINGS = `?api_key=${APIKEY}&language=es-MX`;

	
	//function fetchMovies() {
	  //const URL = `${BASEURL}/discover/movie/${APISETTINGS}&sort_by=popularity.desc`;
	  //fetch(URL)
		//.then((res) => res.json())
		//.then(({ results }) => {
		  //movies = results;
		  //console.log(movies);
		//});
	//}

	const movies = (async () =>{
		const URL = `${BASEURL}/discover/movie/${APISETTINGS}&sort_by=popularity.desc`;
		const response = await fetch(URL)
		return await response.json()
	})()
  
	let likedMovies=[]

	function togglelike(event){
		const movie = event.detail;

		let index = likedMovies.findIndex(m => m.id === movie.id);
		if(index >= 0){
			likedMovies.splice(index, 1);
			console.log(likedMovies);
		}else{
			likedMovies.push(movie);
		}
			likedMovies=likedMovies
	}

	$: like = (id) =>{
		let index = likedMovies.findIndex(m => m.id === id);
		return index >= 0;
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
  <main class="container">
		<div class="row">
			<div class="col-12 col-md-6 col-lg-8 border panel">
				<h2>Peliculas populasres</h2>	
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
			<div class="col-12 col-md-6 col-lg-4 border panel " >
				<h2>Peliculas favoritas</h2>
				<div class="row"> 
					{#if likedMovies.length}
					{#each likedMovies as movie, i (movie.id)}
							<div in:fly="{{duration:2000, y: 20}}" out:fly="{{duration:1000, y: -20}}" class="col-12 col-md-6 col-lg-4 p-2">
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
								<p>
									No Tienes peliculas favoritas...
								</p>
							</div>
						{/if}
				</div>
			</div>
		</div>
  </main>
  
  <style>
	  .panel{
		  height: 100vh;
		  overflow: auto;
	  }

	main {
	  text-align: center;
	  padding: 0;
	  max-width: 240px;
	  margin: 0 auto;
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