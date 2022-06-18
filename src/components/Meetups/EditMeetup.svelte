<script>
  import meetups from '../../meetups-store';
  import TextInput from '../UI/TextInput.svelte';
  import Button from '../UI/Button.svelte';
  import Modal from '../UI/Modal.svelte';
  import { createEventDispatcher } from 'svelte';
  import { isEmpty, isValidEmail } from '../../helpers/validation';

  export let id = null;

  let title = '';
  let subtitle = '';
  let address = '';
  let contactEmail = '';
  let description = '';
  let imageUrl = '';

  if (id) {
    const unsubscribe = meetups.subscribe((items) => {
      const selectedMeetup = items.find((i) => i.id === id);
      title = selectedMeetup.title;
      subtitle = selectedMeetup.subtitle;
      address = selectedMeetup.address;
      contactEmail = selectedMeetup.contactEmail;
      description = selectedMeetup.description;
      imageUrl = selectedMeetup.imageUrl;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: subtitleValid = !isEmpty(subtitle);
  $: descriptionValid = !isEmpty(description);
  $: addressValid = !isEmpty(address);
  $: imageUrlValid = !isEmpty(imageUrl);
  $: emailValid = isValidEmail(contactEmail);
  $: formIsValid =
    titleValid &&
    subtitleValid &&
    descriptionValid &&
    addressValid &&
    imageUrlValid &&
    emailValid;

  function submitForm() {
    const meetupData = {
      title,
      subtitle,
      address,
      contactEmail,
      description,
      imageUrl,
    };

    // meetups.push(newMeetup); // DOES NOT WORK!!
    // lodadedMeetups = [newMeetup, ...lodadedMeetups];
    if (id) {
      fetch(
        `https://svelte-course-ca5cc-default-rtdb.europe-west1.firebasedatabase.app//meetups/${id}.json`,
        {
          method: 'PATCH',
          body: JSON.stringify(meetupData),
          headers: { 'Content-type': 'application/json' },
        }
      )
        .then((res) => {
          if (!res.ok) {
            throw new Error('an error occured');
          }
          meetups.updateMeetup(id, meetupData);
        })
        .catch((err) => console.log(err));
    } else {
      fetch(
        'https://svelte-course-ca5cc-default-rtdb.europe-west1.firebasedatabase.app/meetups.json',
        {
          method: 'POST',
          body: JSON.stringify({ ...meetupData, isFavorite: false }),
          headers: { 'Content-type': 'application/json' },
        }
      )
        .then((res) => {
          if (!res.ok) {
            throw new Error('an error occured');
          }
          return res.json();
        })
        .then((data) =>
          meetups.addMeetup({ ...meetupData, isFavorite: false, id: data.name })
        )
        .catch((err) => console.log(err));
    }
    dispatch('save');
  }

  function cancel() {
    dispatch('cancel');
  }

  function deleteMeetup() {
    fetch(
      `https://svelte-course-ca5cc-default-rtdb.europe-west1.firebasedatabase.app//meetups/${id}.json`,
      {
        method: 'DELETE',
      }
    )
      .then((res) => {
        if (!res.ok) {
          throw new Error('an error occured');
        }
        meetups.removeMeetup(id);
        dispatch('save');
      })
      .catch((err) => console.log(err));
  }
</script>

<Modal title="Edit Meetup Data" on:cancel>
  <form on:submit|preventDefault={submitForm}>
    <TextInput
      id="title"
      label="Title"
      valid={titleValid}
      validityMessage="Please enter a valid title."
      value={title}
      on:input={(event) => (title = event.target.value)}
    />
    <TextInput
      id="subtitle"
      label="Subtitle"
      valid={subtitleValid}
      validityMessage="Please enter a valid subtitle."
      value={subtitle}
      on:input={(event) => (subtitle = event.target.value)}
    />
    <TextInput
      id="address"
      label="Address"
      valid={addressValid}
      validityMessage="Please enter a valid address."
      value={address}
      on:input={(event) => (address = event.target.value)}
    />
    <TextInput
      id="imageUrl"
      label="Image URL"
      valid={imageUrlValid}
      validityMessage="Please enter a valid image url."
      value={imageUrl}
      on:input={(event) => (imageUrl = event.target.value)}
    />
    <TextInput
      id="email"
      label="E-Mail"
      valid={emailValid}
      validityMessage="Please enter a valid email."
      type="e-mail"
      value={contactEmail}
      on:input={(event) => (contactEmail = event.target.value)}
    />
    <TextInput
      id="description"
      label="Description"
      valid={descriptionValid}
      validityMessage="Please enter a valid description."
      controlType="textarea"
      rows={3}
      bind:value={description}
    />
  </form>
  <div slot="footer">
    <Button
      type="button"
      mode="outline"
      on:click={submitForm}
      disabled={!formIsValid}>Save</Button
    >
    <Button type="button" on:click={cancel}>Cancel</Button>
    {#if id}
      <Button type="button" on:click={deleteMeetup}>Delete</Button>
    {/if}
  </div>
</Modal>

<style>
  form {
    width: 100%;
  }
</style>
