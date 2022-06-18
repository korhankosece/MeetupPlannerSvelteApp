<script>
  import { createEventDispatcher } from 'svelte';
  import meetups from '../../meetups-store';
  import Button from './../UI/Button.svelte';
  import Badge from './../UI/Badge.svelte';
  import LoadingSpinner from '../UI/LoadingSpinner.svelte';

  export let id;
  export let title;
  export let subTitle;
  export let imageUrl;
  export let description;
  export let address;
  export let contactEmail;
  export let isFav;

  let isLoading = false;
  const dispatch = createEventDispatcher();

  function toggleFavorite() {
    isLoading = true;
    fetch(
      `https://svelte-course-ca5cc-default-rtdb.europe-west1.firebasedatabase.app//meetups/${id}.json`,
      {
        method: 'PATCH',
        body: JSON.stringify({ isFavorite: !isFav }),
        headers: { 'Content-type': 'application/json' },
      }
    )
      .then((res) => {
        if (!res.ok) {
          throw new Error('an error occured');
        }
        isLoading = false;
        meetups.toggleFavorite(id);
      })
      .catch((err) => {
        isLoading = false;
        console.log(err);
      });
  }
</script>

<article>
  <header>
    <h1>
      {title}
      {#if isFav}
        <Badge>FAVORITE</Badge>
      {/if}
    </h1>
    <h2>{subTitle}</h2>
    <p>{address}</p>
  </header>
  <div class="image">
    <img src={imageUrl} alt={title} />
  </div>
  <div class="content">
    <p>{description}</p>
  </div>
  <footer>
    <Button mode="outline" type="button" on:click={() => dispatch('edit', id)}
      >Edit</Button
    >
    {#if isLoading}
      <!-- <LoadingSpinner /> -->
      <span>Changing...</span>
    {:else}
      <Button
        type="button"
        mode="outline"
        color={isFav ? null : 'success'}
        on:click={toggleFavorite}>{isFav ? 'Unfavorite' : 'Favorite'}</Button
      >
    {/if}

    <Button href={`/${id}`}>Show Details</Button>
  </footer>
</article>

<style>
  article {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
    border-radius: 5px;
    background: white;
    margin: 1rem;
  }

  header,
  .content,
  footer {
    padding: 1rem;
  }

  .image {
    width: 100%;
    height: 14rem;
  }

  .image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  h1 {
    font-size: 1.25rem;
    margin: 0.5rem 0;
    font-family: 'Roboto Slab', sans-serif;
  }

  h1.is-favorite {
    background: #01a129;
    color: white;
    padding: 0 0.5rem;
    border-radius: 5px;
  }

  h2 {
    font-size: 1rem;
    color: #808080;
    margin: 0.5rem 0;
  }

  p {
    font-size: 1.25rem;
    margin: 0;
  }

  div {
    text-align: right;
  }

  .content {
    height: 4rem;
  }
</style>
