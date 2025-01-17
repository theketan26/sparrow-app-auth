<script lang="ts">
	import vector1 from '$lib/assets/Vector1.svg';
	import vector2 from '$lib/assets/Vector2.svg';
	import vector3 from '$lib/assets/Vector3.svg';
	import { handleRegister, handleRegisterValidation } from './register-page';
	import starIcon from '$lib/assets/starIcon.svg';
	import eyeHide from '$lib/assets/eye-hide.svg';
	import eyeShow from '$lib/assets/eye-show.svg';
	import { navigate } from 'svelte-navigator';
	import sparrowicon from '$lib/assets/sparrow-icon-bg.svg';
	import Redirect from '../redirect/Redirect.svelte';
	import SupportHelp from '$lib/components/help/SupportHelp.svelte';
	import Oauth from '$lib/components/o-auth/Oauth.svelte';
	import { notifications } from '$lib/components/toast-notification/ToastNotification';
	import constants from '$lib/utils/constants';
	export let id;
	let isRegistered = false;
	let redirectRules = {
		title: 'Welcome to Sparrow!',
		description: 'Bridging Frontend and Backend Development.',
		message: `Easily document and manage APIs for seamless collaboration between frontend and backend teams. Get started now to simplify your development workflows.`,
		isSpinner: true,
		buttonText: 'Open Desktop App',
		buttonClick: () => {},
		loadingMessage: 'Please wait while we sign you in....'
	};
	let userData = {
		email: id || '',
		firstName: '',
		lastName: '',
		password: '',
		tnsCheckbox: false
	};
	let isDuplicateEmail = false;
	let validationErrors: any = {};

	let isPasswordValid1 = false;
	let isPasswordValid2 = false;
	let isPasswordValid3 = false;

	let isCheckboxTouched = false;
	let isEmailTouched = false;
	let isFirstNameTouched = false;
	let isLastNameTouched = false;
	let isPasswordTouched = false;

	const validatePassword = () => {
		const password = userData.password;
		isPasswordValid1 = isValidPassword1(password);
		isPasswordValid2 = isValidPassword2(password);
		isPasswordValid3 = isValidPassword3(password);
	};

	const isValidPassword1 = (password: string) => {
		if (password.length >= 8) {
			return true;
		}
		return false;
	};

	const isValidPassword2 = (password: string) => {
		if (/(?=.*[0-9])/.test(password)) {
			return true;
		}
		return false;
	};

	const isValidPassword3 = (password: string) => {
		if (/(?=.*[!@#$%^&*])/.test(password)) {
			return true;
		}
		return false;
	};

	let isPasswordVisible = false;

	const togglePasswordVisibility = () => {
		isPasswordVisible = !isPasswordVisible;
		const passwordInput = document.getElementById('expamplePassword') as HTMLInputElement | null;
		if (passwordInput) {
			passwordInput.type = isPasswordVisible ? 'text' : 'password';
		}
	};
</script>

{#if isRegistered}
	<Redirect
		title={redirectRules.title}
		description={redirectRules.description}
		message={redirectRules.message}
		isSpinner={redirectRules.isSpinner}
		buttonText={redirectRules.buttonText}
		buttonClick={redirectRules.buttonClick}
		loadingMessage={redirectRules.loadingMessage}
	/>
{:else}
	<div class="parent d-flex align-items-center justify-content-center text-white rounded">
		<div
			class="entry-point rounded container d-flex flex-column align-items-center justify-content-center w-100"
		>
			<div class="text-white d-flex justify-content-center align-items-center">
				<img src={sparrowicon} width="60px" alt="" class="" />
			</div>
			<p
				class="container-header pt-4 pb-5 sparrow-fs-28 text-whiteColor text-center ms-2 me-2 fw-bold"
			>
				Welcome to Sparrow!
			</p>

			<form
				class="register-form w-100 text-whiteColor ps-1 pe-1 gap-16"
				novalidate
				on:submit|preventDefault={async () => {
					isCheckboxTouched = true;
					isEmailTouched = true;
					isFirstNameTouched = true;
					isLastNameTouched = true;
					isPasswordTouched = true;
					validationErrors = await handleRegisterValidation(userData);
					if (
						!validationErrors?.firstName &&
						!validationErrors?.lastName &&
						!validationErrors?.email &&
						!validationErrors?.password &&
						userData?.tnsCheckbox
					) {
						const response = await handleRegister(userData);
						if (response.isSuccessful) {
							isRegistered = true;
							const accessToken = response?.data.accessToken?.token;
							const refreshToken = response?.data.refreshToken?.token;
							const sparrowRedirect = `sparrow://?accessToken=${accessToken}&refreshToken=${refreshToken}&response=${JSON.stringify(response.data)}`;
							setTimeout(() => {
								let data = JSON.parse(window.atob(accessToken?.split('.')[1]));
								redirectRules.title = `Welcome ${data.name.split(" ")[0]}`;
								redirectRules.description = `Redirecting you to desktop app...`;
								redirectRules.message = `If the application does not open automatically,
								please click below.`;
								redirectRules.loadingMessage = '';
								redirectRules.isSpinner = false;
								navigate(sparrowRedirect);
								redirectRules.buttonClick = () => {
									navigate(sparrowRedirect);
								};
							}, 5000);
						} else {
							if (
								response.message ===
								'The account with the provided email currently exists. Please choose another one.'
							) {
								isDuplicateEmail = true;
							} else {
								notifications.error(response.message);
							}
						}
					}
				}}
			>
				<p class="card-subtitle sparrow-fs-20 mb-3">Create Account</p>
				<div class="form-group gap-0 mb-3">
					<div>
						<label for="email" class="form-label sparrow-fs-14">Email</label>
						<img src={starIcon} alt="" class="mb-3" style="width: 7px;" />
					</div>
					<input
						class="form-control sparrow-fs-16 mt-1 border:{(validationErrors?.email &&
							isEmailTouched) ||
						isDuplicateEmail
							? '3px'
							: '1px'} solid {(validationErrors?.email && isEmailTouched) || isDuplicateEmail
							? 'border-error'
							: 'border-default'}"
						type="email"
						name="email"
						id="email"
						placeholder="Please enter your email id"
						autocorrect="off"
						autocapitalize="none"
						autocomplete="off"
						bind:value={userData.email}
						on:blur={async () => {
							isEmailTouched = true;
							validationErrors = await handleRegisterValidation(userData);
						}}
						on:input={async () => {
							validationErrors = await handleRegisterValidation(userData);
							isDuplicateEmail = false;
						}}
					/>

					{#if validationErrors?.email && isEmailTouched}
						<small class="text-dangerColor form-text">{validationErrors?.email}</small>
					{:else if isDuplicateEmail}
						<small class="text-dangerColor form-text">Email ID already exists.</small>
					{/if}
				</div>
				<div class="form-group mb-3">
					<div>
						<label for="name" class="sparrow-fs-14">First Name</label>
						<img src={starIcon} alt="" class="mb-3" style="width: 7px;" />
					</div>

					<input
						class="form-control sparrow-fs-16 mt-1 border:{validationErrors?.firstName &&
						isFirstNameTouched
							? '3px'
							: '1px'} solid {validationErrors?.firstName && isFirstNameTouched
							? 'border-error'
							: 'border-default'}"
						type="text"
						name="name"
						placeholder="Please enter your first name"
						autocorrect="off"
						autocapitalize="none"
						autocomplete="off"
						id="name"
						bind:value={userData.firstName}
						on:blur={async () => {
							isFirstNameTouched = true;
							validationErrors = await handleRegisterValidation(userData);
						}}
						on:input={async () => {
							validationErrors = await handleRegisterValidation(userData);
						}}
					/>

					{#if validationErrors?.firstName && isFirstNameTouched}
						<small class="text-dangerColor form-text">{validationErrors?.firstName}</small>
					{/if}
				</div>

				<div class="form-group mb-3">
					<div>
						<label for="name" class="sparrow-fs-14">Last Name</label>
					</div>

					<input
						class="form-control sparrow-fs-16 mt-1 border:{false ? '3px' : '1px'} solid {false
							? 'border-success'
							: false
								? 'border-error'
								: false
									? 'border-error'
									: 'border-default'}"
						type="text"
						name="lastname"
						placeholder="Please enter your last name"
						autocorrect="off"
						autocapitalize="none"
						autocomplete="off"
						id="lastname"
						bind:value={userData.lastName}
						on:blur={async () => {
							isLastNameTouched = true;
							validationErrors = await handleRegisterValidation(userData);
						}}
						on:input={async () => {
							validationErrors = await handleRegisterValidation(userData);
						}}
					/>
					{#if validationErrors?.lastName && isLastNameTouched}
						<small class="text-dangerColor form-text">{validationErrors?.lastName}</small>
					{/if}
				</div>

				<div class="form-group">
					<div>
						<label for="password" id="password" class="sparrow-fs-14">Password</label>
						<img src={starIcon} alt="" class="mb-3" style="width: 7px;" />
					</div>
					<div class="d-flex position-relative">
						<input
							class="form-control sparrow-fs-16 mt-1 pe-5 border:{validationErrors?.password &&
							isPasswordTouched
								? '3px'
								: '1px'} solid {validationErrors?.password && isPasswordTouched
								? 'border-error'
								: 'border-default'}"
							type="password"
							name="password"
							id="expamplePassword"
							placeholder="Please enter your password"
							autocorrect="off"
							autocapitalize="none"
							autocomplete="off"
							bind:value={userData.password}
							on:blur={async () => {
								isPasswordTouched = true;
								validationErrors = await handleRegisterValidation(userData);
							}}
							on:input={async () => {
								validationErrors = await handleRegisterValidation(userData);
								validatePassword();
							}}
						/>
						<button
							type="button"
							on:click={togglePasswordVisibility}
							class=" border-0 position-absolute eye-icon d-flex align-items-center"
						>
							{#if isPasswordVisible}
								<img src={eyeShow} alt="eye-show" />
							{:else}
								<img src={eyeHide} alt="eye-hie" />
							{/if}
						</button>
					</div>
				</div>

				<div class="row">
					<div class="col-12 col-md-12 col-lg-12">
						<div class="d-flex flex-column align-items-start mt-1 text-sm" style="font-size: 13px;">
							<div class="d-flex align-items-center mb-0 gap-2">
								<img
									src={isPasswordValid1 ? vector2 : isPasswordTouched ? vector3 : vector1}
									alt=""
									class="mr-2"
								/>
								<p
									class="mb-0 {isPasswordValid1
										? 'text-successColor'
										: isPasswordTouched
											? 'text-dangerColor'
											: 'text-textColor'}"
								>
									Min 8 characters
								</p>
							</div>
							<div class="d-flex align-items-center mb-0 gap-2">
								<img
									src={isPasswordValid2 ? vector2 : isPasswordTouched ? vector3 : vector1}
									alt=""
									class="mr-2"
								/>
								<p
									class="mb-0 {isPasswordValid2
										? 'text-successColor'
										: isPasswordTouched
											? 'text-dangerColor'
											: 'text-textColor'}"
								>
									Has at least one number
								</p>
							</div>
							<div class="d-flex align-items-center gap-2">
								<img
									src={isPasswordValid3 ? vector2 : isPasswordTouched ? vector3 : vector1}
									alt=""
									class="mr-2"
								/>
								<p
									class="mb-0 {isPasswordValid3
										? 'text-successColor'
										: isPasswordTouched
											? 'text-dangerColor'
											: 'text-textColor'}"
								>
									Has at least one special character
								</p>
							</div>
						</div>
					</div>
				</div>

				<div class="form-group mt-2" data-tauri-drag-region>
					<input
						type="checkbox"
						class="form-check-input"
						id="tnsCheckbox"
						bind:checked={userData.tnsCheckbox}
						on:input={async () => {
							isCheckboxTouched = true;
						}}
					/>
					<label data-tauri-drag-region class="form-check-label ms-2" for="tnsCheckbox"
						>I agree to the <a href={constants.SPARROW_TERMS_OF_SERVICE} target="_blank" class="text-decoration-none text-primaryColor"
							>Terms of Service</a
						></label
					>
				</div>
				{#if !userData.tnsCheckbox && isCheckboxTouched}
					<small class="text-dangerColor form-text"
						>You will need to agree to the terms of service to create a Sparrow account.</small
					>
				{/if}

				<div class="mb-3 mt-4">
					<button class="btn btn-primary w-100 text-whiteColor border-0">Sign Up</button>
				</div>
			</form>
			<Oauth />
			<SupportHelp />
		</div>
	</div>
{/if}

<style>
	.btn-primary {
		background: linear-gradient(270deg, #6147ff -1.72%, #1193f0 100%);
	}
	.eye-icon {
		top: 10px;
		right: 5px;
		background-color: transparent;
	}
	.parent {
		min-height: 100vh;
		overflow: auto;
	}
	.entry-point {
		margin: 30px !important;
		background: linear-gradient(to bottom, rgba(51, 51, 51, 0.16), rgba(42, 42, 51, 1));
		max-width: 504px;
		padding: 48px 48px 64px 48px !important;
	}
	input {
		background-color: transparent;
	}
</style>
