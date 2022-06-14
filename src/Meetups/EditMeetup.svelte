<script>
  import meetups from '../Meetups/meetups-store';
  import TextInput from '../UI/TextInput.svelte';
  import Button from '../UI/Button.svelte';
  import Modal from '../UI/Modal.svelte';
  import { createEventDispatcher } from 'svelte';
  import { isEmpty, isValidEmail } from '../helpers/validation';

  export let id = null;

  let title = '';
  let subtitle = '';
  let address = '';
  let email = '';
  let description = '';
  let imageUrl = '';

  if (id) {
    const unsubscribe = meetups.subscribe((items) => {
      const selectedMeetup = items.find((i) => i.id === id);
      title = selectedMeetup.title;
      subtitle = selectedMeetup.subtitle;
      address = selectedMeetup.address;
      email = selectedMeetup.contactEmail;
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
  $: emailValid = isValidEmail(email);
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
      email,
      description,
      imageUrl,
    };

    // meetups.push(newMeetup); // DOES NOT WORK!!
    // lodadedMeetups = [newMeetup, ...lodadedMeetups];
    if (id) {
      meetups.updateMeetup(id, meetupData);
    } else {
      meetups.addMeetup(meetupData);
    }
    dispatch('save');
  }

  function cancel() {
    dispatch('cancel');
  }

  function deleteMeetup() {
    meetups.removeMeetup(id);
    dispatch('save');
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
      value={email}
      on:input={(event) => (email = event.target.value)}
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
