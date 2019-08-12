<script>
	import persist from './store';
	import axios from 'axios';

	let loading = false;
	let errored = false;

	const url = 'https://api-dev.okpock.com';

	let isLoggedIn = persist('isLoggedIn', false);
	let lastSignInAt = persist('lastSignInAt', '');
	let user = persist('user', null);

	let newusername = '';
	let username = '';
	let password = '';

	function getUrl(path) {
		return url + path;
	}

	function handleLogin() {
		loading = true
      	axios.post(getUrl('/login'), {
        	username: username,
        	password: password
      	}).then(response => {
			lastSignInAt = response.data.lastSignInAt;
			isLoggedIn = true;
      	}).catch(error => {
        	console.log(error)
        	errored = true
      	})
      	.finally(() => loading = false)
	}

	function handleLogout() {
		loading = true
      	axios.delete(getUrl('/logout')).then(response => {
        	isLoggedIn = false
			lastSignInAt = ''
			user = null
      	}).catch(error => {
        	console.log(error)
        	errored = true
      	})
		.finally(() => loading = false)
	}

	function loadAccount() {
		loading = true
      	axios.get(getUrl('/account'), {
        	withCredentials: true
      	}).then(response => {
			user = response.data
      	}).catch(error => {
        	console.log(error)
        	errored = true
      	})
      	.finally(() => loading = false)
	}

	function handleUsernameChange() {
		loading = true
      	axios.put(getUrl('/account/username'), {
        	username: newusername
      	}, {
        	withCredentials: true
      	}).then(response => {
			user.username = response.data.username;
      	}).catch(error => {
			console.log(error)
        	errored = true
      	})
      	.finally(() => loading = false)
	}
</script>

<style>
	#app {
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
	}
</style>

<div id="app">
	<h1>This is an login page</h1>

	{#if errored}
		<p>We're sorry, we're not able to retrieve this information at the moment, please try back later</p>
	{:else}
		{#if loading}
			<div>Loading...</div>
		{:else}
			{#if isLoggedIn === true}
				{#if lastSignInAt}
					<p>{lastSignInAt}</p>
				{/if}
				<button type="button" on:click={loadAccount}>Load Account</button>

				{#if user !== null}
					<p>{user.email}</p>
            		<p>{user.username}</p>
					
					<form on:submit|preventDefault={handleUsernameChange}>
              			<label for="newusername">New Username: </label>
              			<input type="text" name="newusername" id="newusername" bind:value={newusername}>
              			<button type="submit">Change</button>
            		</form>

            		<button v-on:click="handleLogout">Logout</button>
				{/if}
			{:else}
				<form on:submit|preventDefault={handleLogin}>
					<label for="username">Username: </label>
					<input type="text" name="username" id="username" bind:value={username}>
					<br>
					<label for="password">Password: </label>
					<input type="password" name="password" id="password" bind:value={password}>
					<button type="submit">Login</button>
				</form>
			{/if}
		{/if}
	{/if}


</div>
