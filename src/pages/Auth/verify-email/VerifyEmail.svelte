<script lang="ts">
	import lineIcon from '$lib/assets/line.svg';
	import starIcon from '$lib/assets/starIcon.svg';
	import { errorMessageText, isLoading, username } from '$lib/store/auth.store';
	import { handleVerifyUserEmail, isSuccessfulResponse } from './verify-email';
	import sparrowicon from '$lib/assets/logoSparrowSquare.svg';
	import { writable } from 'svelte/store';
	import { onDestroy, onMount } from 'svelte';
	import { Link, navigate } from 'svelte-navigator';
	import { notifications } from '$lib/components/toast-notification/ToastNotification';
	import { forgotPassword, sendUserEmailVerification } from '$lib/services/auth.service';
	import Spinner from '$lib/components/transition/Spinner.svelte';
	import Button from '$lib/components/button/Button.svelte';
	import AngleUp from '$lib/assets/angle-up.svelte';
	import BgContainer from '$lib/components/bgContainer/BgContainer.svelte';
	import Redirect from '../redirect/Redirect.svelte';
	import constants from '$lib/utils/constants';
	export let id: string;

	let seconds = 60;
	const verifyString = writable('');
	let verifyLength: string = '';
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

	let timer: number;
	const calculateRemainingTime = () => {
		const currentTime = new Date().getTime();
		const storedTime = parseInt(localStorage.getItem(`timer-verify-${id}`));
		if (storedTime) {
			const elapsedTime = storedTime ? Math.floor((currentTime - storedTime) / 1000) : 0;
			const remainingTime = Math.max(60 - elapsedTime, 0);
			return remainingTime;
		} else {
			return 0;
		}
	};
	const startTimer = () => {
		clearInterval(timer);
		timer = setInterval(() => {
			seconds = calculateRemainingTime();
			if (seconds === 0) clearInterval(timer);
		}, 1000);
	};

	onMount(() => {
		seconds = calculateRemainingTime();
		startTimer();
	});
	onDestroy(() => clearInterval(timer));

	let verifyCodeCredential = {
		email: id || '',
		verificationCode: ''
	};

	let verifyCode: string = '';

	let validationErrors;

	let verificationCode1: string = '';
	let verificationCode2: string = '';
	let verificationCode3: string = '';
	let verificationCode4: string = '';
	let verificationCode5: string = '';
	let verificationCode6: string = '';

	let emailText: string = id || '';


	let userFromDesktop = localStorage.getItem('isUserFromDesktop');


	const handleVerificationCode = () => {
		verifyCode =
			verificationCode1 +
			verificationCode2 +
			verificationCode3 +
			verificationCode4 +
			verificationCode5 +
			verificationCode6;
		verifyCodeCredential.verificationCode = verifyCode;

		if (verifyCode.length === 6) {
			verifyString.set(verifyCode);
		}
	};

	verifyString.subscribe((value) => {
		if (value) {
			verifyLength = value;
		}
	});

	let isLoadingPage: boolean;
	isLoading.subscribe((value) => {
		isLoadingPage = value;
	});

	let verificationCodeError: boolean;
	isSuccessfulResponse.subscribe((value) => {
		verificationCodeError = value;
	});

	let errorMessage: string = '';
	errorMessageText.subscribe((value) => {
		if (value) {
			errorMessage = value;
		}
	});
	let resentCodeLoader = false;
	const handleResend = async () => {
		resentCodeLoader = true;
		const response = await sendUserEmailVerification({ email: id });
		if (response.isSuccessful) {
			notifications.success('Verification code sent successfully');
			localStorage.setItem(`timer-verify-${id}`, new Date().getTime());
			startTimer();
			verificationCode1 = '';
			verificationCode2 = '';
			verificationCode3 = '';
			verificationCode4 = '';
			verificationCode5 = '';
			verificationCode6 = '';
			onCodeInput();
		} else {
			notifications.error(response.message);
		}
		resentCodeLoader = false;
	};
	const onCodeInput = () => {
		errorMessageText.set('');
		isSuccessfulResponse.set(false);
	};
	let verifyCodeLoader = false;
	onDestroy(() => {
		onCodeInput();
	});

	function handlePaste(event: ClipboardEvent) {
		event.preventDefault();
		const clipboardData = event?.clipboardData || window?.clipboardData;
		const pastedText = clipboardData.getData('text').trim().slice(0, 6); // Considering you have 6 input fields
		verificationCode1 = '';
		verificationCode2 = '';
		verificationCode3 = '';
		verificationCode4 = '';
		verificationCode5 = '';
		verificationCode6 = '';
		// Update the values in the array
		const otpInputs = pastedText.split('');

		// Update the input fields
		if (otpInputs[0]) {
			verificationCode1 = otpInputs[0];
		} else {
			document.getElementById('verificationCode1')?.focus();
			onCodeInput();
			handleVerificationCode();
			return;
		}
		if (otpInputs[1]) {
			verificationCode2 = otpInputs[1];
		} else {
			document.getElementById('verificationCode2')?.focus();
			onCodeInput();
			handleVerificationCode();
			return;
		}
		if (otpInputs[2]) {
			verificationCode3 = otpInputs[2];
		} else {
			document.getElementById('verificationCode3')?.focus();
			onCodeInput();
			handleVerificationCode();
			return;
		}
		if (otpInputs[3]) {
			verificationCode4 = otpInputs[3];
		} else {
			document.getElementById('verificationCode4')?.focus();
			onCodeInput();
			handleVerificationCode();
			return;
		}
		if (otpInputs[4]) {
			verificationCode5 = otpInputs[4];
		} else {
			document.getElementById('verificationCode5')?.focus();
			onCodeInput();
			handleVerificationCode();
			return;
		}
		if (otpInputs[5]) {
			verificationCode6 = otpInputs[5];
		} else {
			document.getElementById('verificationCode6')?.focus();
			onCodeInput();
			handleVerificationCode();
			return;
		}
		document.getElementById('verificationCode6')?.focus();
		onCodeInput();
		handleVerificationCode();
	}
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
	<BgContainer>
		<p
			class="container-header pb-3 sparrow-fs-36 text-whiteColor ms-2 me-2 sparrow-fw-500 w-100 text-left"
			style="letter-spacing: 0.05em;"
		>
			Welcome to Sparrow!
		</p>
		<div class="login-form text-lightGray ps-1 pe-1 gap-16">
			<div class="d-flex flex-column align-items-left mb-2">
				<div class="d-flex align-items-center justify-content-start mb-3 gap-2">
					<a
						class="border-0 bg-transparent font-monospace"
						style="transform: rotate(-90deg);"
						href="/init"
					>
						<AngleUp color="var(--sparrow-text-color)" height={20} width={20} />
					</a>
					<p class="text-whiteColor sparrow-fs-14 sparrow-fw-500 mb-0">Go back</p>
				</div>

				<div class="sparrow-fs-14 sparrow-fs-300">
					<p>
						Check your inbox for the verification code sent to <span
							class="sparrow-fw-700 text-whiteColor cursor-pointer">{emailText}</span
						> and enter it below to proceed.
					</p>
					{#if seconds > 0}
						<div class="d-flex flex-column">
							<div class="d-flex align-items-center mb-2">
								<p class="mb-1 sparrow-fs-14">Verification Code</p>
								<img src={starIcon} alt="" class="mb-2 ms-1" style="width: 7px;" />
							</div>
							<div
								class="d-flex mb-2 align-items-center justify-content-start"
								style="padding: 6px, 12px, 6px, 12px;border-radius: 4px;border: 1px;gap:8px"
							>
								<input
									type="text"
									autocorrect="off"
									autocapitalize="none"
									autocomplete="off"
									id="verificationCode1"
									class="form-control text-center rounded fs-5 border:{verificationCodeError ===
									true
										? '3px'
										: '1px'} solid {verificationCodeError === true
										? 'border-error'
										: 'border-default'}"
									style="width:48px;height:36px;border-none"
									bind:value={verificationCode1}
									on:click={(e) => {
										e.target.select();
									}}
									on:input={(e) => {
										if (verificationCode1.length === 1) {
											document.getElementById('verificationCode2')?.focus();
											document.getElementById('verificationCode2')?.select();
										} else if (e.inputType === 'insertText' && verificationCode1.length > 1) {
											verificationCode1 = verificationCode1.charAt(1);
											document.getElementById('verificationCode2')?.focus();
											document.getElementById('verificationCode2')?.select();
										}
										onCodeInput();
									}}
									on:input={handleVerificationCode}
									on:paste={handlePaste}
								/>
								<img src={lineIcon} alt="" />
								<input
									type="text"
									autocorrect="off"
									autocapitalize="none"
									autocomplete="off"
									id="verificationCode2"
									style="width:48px;height:36px;"
									class="form-control text-center rounded fs-5 border:{verificationCodeError ===
									true
										? '3px'
										: '1px'} solid {verificationCodeError === true
										? 'border-error'
										: 'border-default'}"
									bind:value={verificationCode2}
									on:click={(e) => {
										e.target.select();
									}}
									on:input={(e) => {
										if (verificationCode2.length === 1) {
											document.getElementById('verificationCode3')?.focus();
											document.getElementById('verificationCode3')?.select();
										} else if (e.inputType === 'insertText' && verificationCode2.length > 1) {
											verificationCode2 = verificationCode2.charAt(1);
											document.getElementById('verificationCode3')?.focus();
											document.getElementById('verificationCode3')?.select();
										}
										onCodeInput();
									}}
									on:keydown={(e) => {
										if (e.key === 'Backspace' && verificationCode2.length === 0) {
											document.getElementById('verificationCode1')?.focus();
										}
									}}
									on:input={handleVerificationCode}
									on:paste={handlePaste}
								/>
								<img src={lineIcon} alt="" />
								<input
									type="text"
									autocorrect="off"
									autocapitalize="none"
									autocomplete="off"
									id="verificationCode3"
									style="width:48px;height:36px;"
									class="form-control text-center rounded fs-5 border:{verificationCodeError ===
									true
										? '3px'
										: '1px'} solid {verificationCodeError === true
										? 'border-error'
										: 'border-default'}"
									bind:value={verificationCode3}
									on:click={(e) => {
										e.target.select();
									}}
									on:input={(e) => {
										if (verificationCode3.length === 1) {
											document.getElementById('verificationCode4')?.focus();
											document.getElementById('verificationCode4')?.select();
										} else if (e.inputType === 'insertText' && verificationCode3.length > 1) {
											verificationCode3 = verificationCode3.charAt(1);

											document.getElementById('verificationCode4')?.focus();
											document.getElementById('verificationCode4')?.select();
										}
										onCodeInput();
									}}
									on:keydown={(e) => {
										if (e.key === 'Backspace' && verificationCode3.length === 0) {
											document.getElementById('verificationCode2')?.focus();
										}
									}}
									on:input={handleVerificationCode}
									on:paste={handlePaste}
								/>
								<img src={lineIcon} alt="" />
								<input
									type="text"
									autocorrect="off"
									autocapitalize="none"
									autocomplete="off"
									id="verificationCode4"
									style="width:48px;height:36px;"
									class="form-control text-center rounded fs-5 border:{verificationCodeError ===
									true
										? '3px'
										: '1px'} solid {verificationCodeError === true
										? 'border-error'
										: 'border-default'}"
									bind:value={verificationCode4}
									on:click={(e) => {
										e.target.select();
									}}
									on:input={(e) => {
										if (verificationCode4.length === 1) {
											document.getElementById('verificationCode5')?.focus();
											document.getElementById('verificationCode5')?.select();
										} else if (e.inputType === 'insertText' && verificationCode4.length > 1) {
											verificationCode4 = verificationCode4.charAt(1);

											document.getElementById('verificationCode5')?.focus();
											document.getElementById('verificationCode5')?.select();
										}
										onCodeInput();
									}}
									on:keydown={(e) => {
										if (e.key === 'Backspace' && verificationCode4.length === 0) {
											document.getElementById('verificationCode3')?.focus();
										}
									}}
									on:input={handleVerificationCode}
									on:paste={handlePaste}
								/>
								<img src={lineIcon} alt="" />
								<input
									type="text"
									autocorrect="off"
									autocapitalize="none"
									autocomplete="off"
									id="verificationCode5"
									style="width:48px;height:36px; "
									class="form-control text-center rounded fs-5 border:{verificationCodeError ===
									true
										? '3px'
										: '1px'} solid {verificationCodeError === true
										? 'border-error'
										: 'border-default'}"
									bind:value={verificationCode5}
									on:click={(e) => {
										e.target.select();
									}}
									on:input={(e) => {
										if (verificationCode5.length === 1) {
											document.getElementById('verificationCode6')?.focus();
											document.getElementById('verificationCode6')?.select();
										} else if (e.inputType === 'insertText' && verificationCode5.length > 1) {
											verificationCode5 = verificationCode5.charAt(1);
											document.getElementById('verificationCode6')?.focus();
											document.getElementById('verificationCode6')?.select();
										}
										onCodeInput();
									}}
									on:keydown={(e) => {
										if (e.key === 'Backspace' && verificationCode5.length === 0) {
											document.getElementById('verificationCode4')?.focus();
										}
									}}
									on:input={handleVerificationCode}
									on:paste={handlePaste}
								/>
								<img src={lineIcon} alt="" />
								<input
									type="text"
									autocorrect="off"
									autocapitalize="none"
									autocomplete="off"
									id="verificationCode6"
									style="width:48px;height:36px;"
									class="form-control text-center rounded fs-5 border:{verificationCodeError ===
									true
										? '3px'
										: '1px'} solid {verificationCodeError === true
										? 'border-error'
										: 'border-default'}"
									bind:value={verificationCode6}
									on:click={(e) => {
										e.target.select();
									}}
									on:input={(e) => {
										if (verificationCode6.length > 1) {
											verificationCode6 = verificationCode6.charAt(1);
										}
										onCodeInput();
									}}
									on:keydown={(e) => {
										if (e.key === 'Backspace' && verificationCode6.length === 0) {
											document.getElementById('verificationCode5')?.focus();
										}
									}}
									on:input={handleVerificationCode}
									on:paste={handlePaste}
								/>
							</div>
							{#if verificationCodeError === true}
								<small class="form-text text-dangerColor">
									{errorMessage}
								</small>
							{/if}
						</div>
					{/if}

					{#if seconds > 0}
						<p class="my-4">
							The code will expire in <span class="text-dangerColor">{seconds} seconds</span>
						</p>
					{:else}
						<p class="my-4 text-dangerColor">Verification code expired</p>
					{/if}
					{#if !(seconds > 0)}
						<p>Please enter the verification code within the specified timeframe.</p>
					{/if}
				</div>
				{#if seconds > 0}
					<Button
						disable={verifyCodeLoader}
						title={'Verify'}
						buttonClassProp={'w-100 py-2 align-items-center d-flex justify-content-center sparrow-fs-16'}
						type={'primary'}
						loader={verifyCodeLoader}
						onClick={async () => {
							verifyCodeLoader = true;
							let response = await handleVerifyUserEmail(verifyCodeCredential);
							if (response?.isSuccessful) {
								isRegistered = true;
								const accessToken = response?.data.accessToken?.token;
								const refreshToken = response?.data.refreshToken?.token;
								const sparrowRedirect = `sparrow://?accessToken=${accessToken}&refreshToken=${refreshToken}&response=${JSON.stringify(response.data)}&event=register&method=email`;
								const sparrowWebRedirect = constants.SPARROW_WEB_URL +`?accessToken=${accessToken}&refreshToken=${refreshToken}&response=${JSON.stringify(response)}&event=register&method=email`;

								if(userFromDesktop === "true"){
								setTimeout(() => {
									let data = JSON.parse(window.atob(accessToken?.split('.')[1]));
									redirectRules.title = `Welcome ${data.name.split(' ')[0]}`;
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
							}

								else{
									navigate(sparrowWebRedirect);
								}
							}

							verifyCodeLoader = false;
						}}
					/>
				{:else}
					<Button
						onClick={() => {
							handleResend();
						}}
						title={'Try again'}
						buttonClassProp={'w-100 py-2 align-items-center d-flex justify-content-center sparrow-fs-16'}
						type={'primary'}
					/>
				{/if}
			</div>

			{#if seconds > 0}
				<div class="d-flex gap-3 align-items-center mt-3">
					<p style="font-size: 13px;" class="mb-0">Didn’t received the code?</p>
					{#if !resentCodeLoader}
						<span
							on:click={handleResend}
							style="font-size: 13px;"
							class="cursor-pointer text-decoration-none text-primaryColor fw-bold"
							>Resend
						</span>
					{:else}
						<Spinner size={'12px'} />
					{/if}
				</div>
			{/if}
		</div>
	</BgContainer>
{/if}

<style>
	input {
		background-color: var(--blackColor);
	}
	.cursor-pointer {
		cursor: pointer;
	}
</style>
