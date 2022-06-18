<script context="module">
  export function preload(page) {
    return this.fetch(
      'https://svelte-course-ca5cc-default-rtdb.europe-west1.firebasedatabase.app/meetups.json'
    )
      .then((res) => {
        if (!res.ok) {
          throw new Error('Fetching meetups failed, please try again later!');
        }
        return res.json();
      })
      .then((data) => {
        const loadedMeetups = [];
        for (const key in data) {
          loadedMeetups.push({ ...data[key], id: key });
        }
        return { fetchedMeetups: loadedMeetups.reverse() };
        // setTimeout(() => {
        //   isLoading = false;
        //   meetups.setMeetups(loadedMeetups.reverse());
        // }, 1000);
      })
      .catch((err) => {
        isLoading = false;
        error = err;
        console.log(err);
        this.error(500, 'Could not fetched meetups');
      });
  }
</script>

<script>
  import { createEventDispatcher, onMount } from 'svelte';
  import { flip } from 'svelte/animate';
  import { scale } from 'svelte/transition';
  import meetups from '../meetups-store';
  import MeetupItem from '../components/Meetups/MeetupItem.svelte';
  import MeetupFilter from '../components/Meetups/MeetupFilter.svelte';
  import Button from '../components/UI/Button.svelte';
  import EditMeetup from '../components/Meetups/EditMeetup.svelte';
  import LoadingSpinner from '../components/UI/LoadingSpinner.svelte';
  import { onDestroy } from 'svelte/internal';

  export let fetchedMeetups;

  let loadedMeetups = [];
  let editMode;
  let editedId;
  let isLoading;
  let unsubscribe;

  const dispatch = createEventDispatcher();
  let favsOnly = false;

  $: filteredMeetups = favsOnly
    ? loadedMeetups.filter((m) => m.isFavorite)
    : loadedMeetups;

  onMount(() => {
    unsubscribe = meetups.subscribe((items) => (loadedMeetups = items));
    meetups.setMeetups(fetchedMeetups);
  });

  onDestroy(() => {
    if (unsubscribe) {
      unsubscribe();
    }
  });

  function setFilter(event) {
    favsOnly = event.detail === 1;
  }

  function savedMeetup() {
    editMode = null;
    editedId = null;
  }

  function cancelEdit() {
    editMode = null;
    editedId = null;
  }

  function startEdit(event) {
    editMode = 'edit';
    editedId = event.detail;
  }

  function startAdd() {
    editMode = 'edit';
  }
</script>

<svelte:head>
  <title>All Meetups</title>
</svelte:head>

{#if editMode === 'edit'}
  <EditMeetup id={editedId} on:save={savedMeetup} on:cancel={cancelEdit} />
{/if}
{#if isLoading}
  <LoadingSpinner />
{:else}
  <section id="meetup-controls">
    <MeetupFilter on:select={setFilter} />
    <Button on:click={startAdd}>New Meetup</Button>
  </section>
  {#if filteredMeetups.length === 0}
    <p id="no-meetups">No meetups found, you can start adding some</p>
  {/if}
  <section id="meetups">
    {#each filteredMeetups as meetup (meetup.id)}
      <div transition:scale animate:flip={{ duration: 300 }}>
        <MeetupItem
          id={meetup.id}
          title={meetup.title}
          subTitle={meetup.subtitle}
          description={meetup.description}
          address={meetup.address}
          imageUrl={meetup.imageUrl}
          contactEmail={meetup.contactEmail}
          isFav={meetup.isFavorite}
          on:showdetails
          on:edit={startEdit}
        />
      </div>
    {/each}
  </section>
{/if}

<style>
  #meetups {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }

  #meetup-controls {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }

  #no-meetups {
    margin: 1rem;
  }

  @media (min-width: 768px) {
    #meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>
