<script lang="ts">
	import "$lib/styles/dashboard.css";
	import { InputValidation } from "$lib/scripts/input-validation";
	import { get, writable } from "svelte/store";
	import { onMount } from "svelte";

	import type { VatsimUserData, WatchedCallsign } from "$lib/types/vatsim";
	import Feedback from "$lib/components/Feedback.svelte";
	import { urlBase64ToUint8Array } from "$lib/scripts/base64-convert";

	let enteredCallsign = "";
	let enteredDiscordNotification = "";
	let enteredTopdown = false;

	export let data: {
		watchedCallsigns: WatchedCallsign[];
		user: VatsimUserData;
		discordNotifications: string[];
		isIgnored: boolean;
	};

	const watchedCallsignsStore = writable(data.watchedCallsigns);
	const discordNotificationsStore = writable(data.discordNotifications);
	const isIgnoredStore = writable(data.isIgnored);
	const pushNotificationsStore = writable(false);

	onMount(() => {
		if (Notification.permission === "granted" && "serviceWorker" in navigator) {
			navigator.serviceWorker.getRegistration("/service-worker.js").then((registration) => {
				if (registration) {
					registration.pushManager.getSubscription().then((subscription) => {
						if (subscription) {
							pushNotificationsStore.set(true);
						}
					});
				}
			});
		}
	});

	async function addCallsignHandler(): Promise<string | void> {
		if (!enteredCallsign) {
			return alert("Please enter a callsign");
		}

		if (!InputValidation.isCallsign(enteredCallsign)) {
			return alert("Please enter a valid callsign");
		}

		document.body.style.cursor = "wait";
		const res = await fetch(`/api/callsign?callsign=${enteredCallsign}&topdown=${enteredTopdown}`, {
			method: "POST",
		});
		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${(await res.json()).message}`);
		}

		watchedCallsignsStore.update((watchedCallsigns) => {
			watchedCallsigns.push({ string: enteredCallsign, topdown: enteredTopdown });
			return watchedCallsigns;
		});

		enteredCallsign = "";
	}

	async function deleteCallsignHandler(callsign: string): Promise<string | void> {
		if (!confirm("Are you sure you want to delete this callsign?")) {
			return;
		}

		document.body.style.cursor = "wait";
		const res = await fetch(`/api/callsign?callsign=${callsign}`, {
			method: "DELETE",
		});
		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${(await res.json()).message}`);
		}

		watchedCallsignsStore.update((watchedCallsigns) => {
			return watchedCallsigns.filter((c) => c.string !== callsign);
		});
	}

	export async function addDiscordNotificationHandler(): Promise<string | void> {
		if (!enteredDiscordNotification) {
			return alert("Please enter a notification");
		}

		if (!InputValidation.isDiscordWebhookUrl(enteredDiscordNotification)) {
			return alert("Please enter a valid notification");
		}

		document.body.style.cursor = "wait";
		const res = await fetch(`/api/discord-notification?webhookUrl=${enteredDiscordNotification}`, {
			method: "POST",
		});
		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${(await res.json()).message}`);
		}

		discordNotificationsStore.update((discordNotifications) => {
			discordNotifications.push(enteredDiscordNotification);
			return discordNotifications;
		});

		enteredDiscordNotification = "";
	}

	export async function deleteDiscordNotificationHandler(webookUrl: string): Promise<string | void> {
		if (!confirm("Are you sure you want to delete this notification?")) {
			return;
		}

		document.body.style.cursor = "wait";
		const res = await fetch(`/api/discord-notification?webhookUrl=${webookUrl}`, {
			method: "DELETE",
		});
		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${(await res.json()).message}`);
		}

		discordNotificationsStore.update((discordNotifications) => {
			return discordNotifications.filter((n) => n !== webookUrl);
		});
	}

	export async function testDiscordNotificationHandler(webookUrl: string): Promise<string | void> {
		document.body.style.cursor = "wait";
		const res = await fetch(webookUrl, {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
			},
			body: JSON.stringify({
				content: "VatNotif test notification",
			}),
		});
		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${await res.text()}`);
		}

		alert("Notification sent");
	}

	export async function ignoreHandler(): Promise<string | void> {
		isIgnoredStore.update((isIgnored) => !isIgnored);

		document.body.style.cursor = "wait";

		const res = await fetch("/api/ignore", {
			method: "POST",
			body: JSON.stringify({
				state: $isIgnoredStore,
			}),
		});

		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${(await res.json()).message}`);
		}
	}

	export async function pushHandler(): Promise<string | void> {
		const permsResult = await Notification.requestPermission();

		if (permsResult !== "granted") {
			return alert("Please allow push notifications");
		}

		if (!("serviceWorker" in navigator)) {
			return alert("Service workers are not supported");
		}
		if (!get(pushNotificationsStore)) {
			// Disable push notifications
			const registration = await navigator.serviceWorker.getRegistration("/service-worker.js");
			const subscription = await registration?.pushManager.getSubscription();

			if (registration) {
				await registration.unregister();
			}

			const res = await fetch("/api/push-notification", {
				method: "DELETE",
				headers: {
					"Content-type": "application/json",
				},
				body: JSON.stringify({
					subscription: subscription,
				}),
			});

			if (!res.ok) {
				return alert(`Something went wrong\nError: ${(await res.json()).message}`);
			}
			return;
		} else {
			// Enable push notifications
			document.body.style.cursor = "wait";

			const registration = await navigator.serviceWorker.register("/service-worker.js");
			let subscription = await registration.pushManager.getSubscription();

			if (!subscription) {
				const vapidPublicKey = urlBase64ToUint8Array(await (await fetch("https://api.vatnotif.kristn.co.uk/push/public-key")).text());
				subscription = await registration.pushManager.subscribe({
					userVisibleOnly: true,
					applicationServerKey: vapidPublicKey,
				});
			}
			const res = await fetch("/api/push-notification", {
				method: "POST",
				headers: {
					"Content-type": "application/json",
				},
				body: JSON.stringify({
					subscription: subscription,
				}),
			});

			if (!res.ok) {
				return alert(`Something went wrong\nError: ${(await res.json()).message}`);
			}

			document.body.style.cursor = "default";
		}
	}

	export async function topdownToggle(callsign: WatchedCallsign) {
		document.body.style.cursor = "wait";
		const res = await fetch(`/api/callsign?callsign=${callsign.string}&topdown=${!callsign.topdown}`, {
			method: "PUT",
		});
		document.body.style.cursor = "default";

		if (!res.ok) {
			return alert(`Something went wrong\nError: ${(await res.json()).message}`);
		}
	}
</script>

<svelte:head>
	<title>VatNotif - Dashboard</title>
</svelte:head>

<Feedback />

<div class="main-container">
	<h1>Dashboard</h1>

	<div class="section center">
		<h2>User info</h2>
		<p>CID: {data.user.cid}</p>
		<p>Name: {data.user.personal.name_full}</p>
	</div>

	<div class="section center">
		<h2>Privacy settings</h2>
		<p>
			Your CID currently <b>{$isIgnoredStore ? "can't" : "can"}</b> be tracked by other VatNotif users, they
			<b>{$isIgnoredStore ? "won't" : "will"}</b> receive a notification when you come online on a tracked position
		</p>
		<label class="switch">
			<input type="checkbox" bind:checked={$isIgnoredStore} on:click={ignoreHandler} />
			<span class="slider" />
		</label>
	</div>

	<div class="section">
		<h2>Watched callsigns</h2>
		<p>
			Enter your watched callsigns here, e.g. EGNX_GND.
			<br />
			Wildcard (%) is supported, e.g. EGNX_%. This will watch all EGNX positions. The callsign cannot contain only a wildcard and cannot start with
			a wildcard.
		</p>
		<div class="watched-callsigns-div section">
			{#if $watchedCallsignsStore.length === 0}
				<p>You have no watched callsigns</p>
			{:else}
				{#each $watchedCallsignsStore as callsign}
					<div class="watched-callsign">
						<p>{callsign.string}</p>
						<div style="width: 50%" />
						<p>top-down</p>
						<label class="switch">
							<input type="checkbox" bind:checked={callsign.topdown} on:click={() => topdownToggle(callsign)} />
							<span class="slider" />
						</label>
						<button on:click={() => deleteCallsignHandler(callsign.string)}>Delete</button>
					</div>
				{/each}
			{/if}
			<div class="watched-callsign">
				<input placeholder="Enter callsign" bind:value={enteredCallsign} />
				<div style="width: 20%" />
				<p>top-down</p>
				<label class="switch">
					<input type="checkbox" bind:checked={enteredTopdown} />
					<span class="slider" />
				</label>
				<button on:click={addCallsignHandler}>Add</button>
			</div>
		</div>
	</div>

	<div class="section">
		<h2>Discord Notifications</h2>
		<p>
			Enter your discord webhooks here, they should have a url like this:
			<br />
			https://discord.com/api/webhooks/123456789/abcdefghijklmnopqrstuvwxyz123456789
		</p>
		<div class="notifications-div section">
			{#if $discordNotificationsStore.length === 0}
				<p>You have no notifications</p>
			{:else}
				{#each $discordNotificationsStore as notification}
					<div class="notification">
						<p>{notification}</p>
						<button on:click={() => testDiscordNotificationHandler(notification)}>Test</button>
						<button on:click={() => deleteDiscordNotificationHandler(notification)}>Delete</button>
					</div>
				{/each}
			{/if}
			<div class="notification">
				<input placeholder="Enter webhook url" bind:value={enteredDiscordNotification} style="word-break: break-all" />
				<button on:click={addDiscordNotificationHandler}>Add</button>
			</div>
		</div>
	</div>

	<div class="section">
		<h2>Push Notifications</h2>

		<div class="notifications-div section">
			<p>Enable push notifications</p>

			<label class="switch">
				<input type="checkbox" bind:checked={$pushNotificationsStore} on:click={pushHandler} />
				<span class="slider" />
			</label>
		</div>
	</div>
</div>
