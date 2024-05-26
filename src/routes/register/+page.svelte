<script>
	import { Stretch } from 'svelte-loading-spinners';
	import Icon from '@iconify/svelte';
    import { onMount } from 'svelte';
	import { getUserAuthenticationCSRFToken } from '$lib/services/csrfTokenFetcher/getUserAuthenticationCSRFToken';
	import { registerUserAPI } from '$lib/services/authentication/registerUserAPI';

    // Store for CSRF token
    let csrfToken = ""
	onMount(() => {
		getUserAuthenticationCSRFToken()
		.then((token) => {
			csrfToken = token
		}).catch((error) => {
			console.error(`Failed to fetch CRSF Token :: ${error}`)
			// TODO: Display error to the user...
		})
	});

	import UsernameInput from './usernameInput.svelte';
	let username = '';
	let usernameError = '';

	import EmailInput from './emailInput.svelte';
	let email = '';
	let emailError = '';
	let emailShowLoginInstead = false;

	import PasswordInput from './passwordInput.svelte';
	let password = '';
	let passwordError = '';

	let genericError = '';
	$: alertVisible = false;

	$: loadingAPI = false;

	async function register() {
		genericError = '';
		if (
			usernameError.length !== 0 ||
			emailError.length !== 0 ||
			passwordError.length !== 0 ||
			username.length === 0 ||
			email.length === 0 ||
			password.length === 0
		) {
			console.warn('There is still errors or blank fields, yet user invoked regisration function!');
			return;
		}

		try {
			loadingAPI = true;

			console.debug(`Using token :: ${csrfToken}`)
			const result = await registerUserAPI(username, email, password, csrfToken);
			console.debug(result);

			if (result.status === 'SUCCESS') {
				alertVisible = true;
				// TODO: Maybe redirect after to some other page after few seconds?
			} else if (result.status === 'USERNAME_TAKEN') {
				usernameError = 'The username has been taken!';
			} else if (result.status === 'EMAIL_TAKEN') {
				emailError = 'The email has been used! Forgot password?';
			} else {
				genericError = 'Unknown Error! Refresh page and try again!\nContact admin if issue persists!';
			}
		} catch (error) {
			// @ts-ignore
			console.error(error.message);
			genericError = 'Unknown Error! Refresh page and try again!\nContact admin if issue persists!';
		} finally {
			loadingAPI = false;
		}
	}

	$: registrationDisabled =
		usernameError.length !== 0 ||
		emailError.length !== 0 ||
		passwordError.length !== 0 ||
		username.length === 0 ||
		email.length === 0 ||
		password.length === 0;
</script>

{#if alertVisible}
	<aside class="m-9 alert variant-filled-success">
		<!-- Icon -->
		<div>
			<Icon icon="icon-park-outline:success" height="auto" />
		</div>
		<!-- TODO: Ask for email auth -->
		<div class="alert-message">
			<h3 class="h3">Account Request Success!</h3>
			<p>An account has been successfully created!</p>
			<p>Login again!</p>
		</div>

		<div class="alert-actions">
			<button type="button" class="btn-icon !bg-transparent">
				<Icon icon="carbon:close-filled" height="auto" />
			</button>
		</div>
	</aside>
{/if}

<div class="container h-full mx-auto justify-center items-center">

		<h2 class="h2 m-4">Register for a new account!</h2>

		<UsernameInput bind:disabled={loadingAPI} bind:error={usernameError} bind:value={username} />

		<EmailInput
			bind:disabled={loadingAPI}
			bind:showLoginInstead={emailShowLoginInstead}
			bind:error={emailError}
			bind:value={email}
		/>

		<PasswordInput bind:disabled={loadingAPI} bind:error={passwordError} bind:value={password} />

		{#if loadingAPI}
			<div>
				<Stretch size="60" color="#FF3E00" unit="px" duration="1s" />
			</div>
		{/if}

		<button
			type="button"
			disabled={registrationDisabled || loadingAPI}
			class="mt-3 btn variant-filled"
			on:click={register}
		>
			Register
		</button>

		<div class="mt-2">
			<i>
				Already have an account? <a class="anchor" href="/login">Login</a> instead.
			</i>
		</div>


		{#if genericError.length !== 0}
			<p class="text-red-500">{genericError}</p>
		{/if}
</div>