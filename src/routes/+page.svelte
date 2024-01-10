<script>
	// @ts-nocheck

	import { Avatar } from '@skeletonlabs/skeleton';
	import { GoogleGenerativeAI } from '@google/generative-ai';
	import { onMount } from 'svelte';
	import { getApp, getApps, initializeApp } from 'firebase/app';
	import { getFirestore, collection, onSnapshot, addDoc, deleteDoc, doc } from 'firebase/firestore';
	import { firebaseConfig } from '$lib/firebaseConfig';
	import { browser } from '$app/environment';

	const firebaseApp =
		browser && (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp());
	const db = browser && getFirestore(firebaseApp);

	let message = '';
	let genAI;
	let model;
	let history = [];

	onMount(() => {
		genAI = new GoogleGenerativeAI('AIzaSyDYjHGKpTd2MHkbMFEErol9WaBN_j25FOU');
		model = genAI.getGenerativeModel({ model: 'gemini-pro' });
	});

	const colRef = browser && collection(db, "history");
	const unsubscribe = browser && onSnapshot(colRef, (querySnapshot) => {
		let fbHistory = [];
		querySnapshot.forEach((doc) => {
			fbHistory = [{...doc.data(), id: doc.id}, ...fbHistory];
		});
		
		fbHistory.sort((a, b) => {
			const timestampA = new Date(a.createdAt).getTime();
			const timestampB = new Date(b.createdAt).getTime();
			return timestampA - timestampB;
		});

		history = fbHistory;
	});

	async function submitPrompt() {
		if (message !== '') {
			const docRef = await addDoc(collection(db, "history"), {
				role: 'user',
				text: message,
				createdAt: new Date().toLocaleString()
			});
		}
		predict();
		message = '';
	}

	async function predict() {
		if (!genAI || !model) {
			return;
		}

		const prompt = message;
		const result = await model.generateContent(prompt);
		const response = await result.response;
		const text = response.text();

		const docRef = await addDoc(collection(db, "history"), {
			role: 'model',
			text: text,
			createdAt: new Date().toLocaleString()
		});
	}

	async function deleteHistory(id) {
		await deleteDoc(doc(db, "history", id));
	}
</script>

<div class="container h-full mx-auto flex justify-center items-center pb-12">
	<div class="space-y-5">
		{#each history as { id, role, text, createdAt }}
			{#if role === 'model'}
				<div class="grid grid-cols-[auto_1fr] gap-2">
					<div class="flex flex-col justify-between items-end">
						<Avatar initials="JD" width="w-12" />
						<button on:click={deleteHistory(id)}>
							<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32" id="delete" class="w-4 h-4" fill="white"><path d="M24.2,12.193,23.8,24.3a3.988,3.988,0,0,1-4,3.857H12.2a3.988,3.988,0,0,1-4-3.853L7.8,12.193a1,1,0,0,1,2-.066l.4,12.11a2,2,0,0,0,2,1.923h7.6a2,2,0,0,0,2-1.927l.4-12.106a1,1,0,0,1,2,.066Zm1.323-4.029a1,1,0,0,1-1,1H7.478a1,1,0,0,1,0-2h3.1a1.276,1.276,0,0,0,1.273-1.148,2.991,2.991,0,0,1,2.984-2.694h2.33a2.991,2.991,0,0,1,2.984,2.694,1.276,1.276,0,0,0,1.273,1.148h3.1A1,1,0,0,1,25.522,8.164Zm-11.936-1h4.828a3.3,3.3,0,0,1-.255-.944,1,1,0,0,0-.994-.9h-2.33a1,1,0,0,0-.994.9A3.3,3.3,0,0,1,13.586,7.164Zm1.007,15.151V13.8a1,1,0,0,0-2,0v8.519a1,1,0,0,0,2,0Zm4.814,0V13.8a1,1,0,0,0-2,0v8.519a1,1,0,0,0,2,0Z"></path></svg>
						</button>
					</div>
					<div class="card p-4 variant-soft rounded-tl-none space-y-2">
						<header class="flex justify-between items-center">
							<p class="font-bold">Model</p>
							<small class="opacity-50">{createdAt}</small>
						</header>
						<p>{text}</p>
					</div>
				</div>
			{/if}
			{#if role === 'user'}
				<div class="grid grid-cols-[1fr_auto] gap-2">
					<div class="card p-4 rounded-tr-none space-y-2">
						<header class="flex justify-between items-center">
							<p class="font-bold">User</p>
							<small class="opacity-50">{createdAt}</small>
						</header>
						<p>{text}</p>
					</div>
					<div class="flex flex-col justify-between items-start">
						<Avatar initials="US" background="bg-tertiary-500" width="w-12" />
						<button on:click={deleteHistory(id)}>
							<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32" id="delete" class="w-4 h-4" fill="white"><path d="M24.2,12.193,23.8,24.3a3.988,3.988,0,0,1-4,3.857H12.2a3.988,3.988,0,0,1-4-3.853L7.8,12.193a1,1,0,0,1,2-.066l.4,12.11a2,2,0,0,0,2,1.923h7.6a2,2,0,0,0,2-1.927l.4-12.106a1,1,0,0,1,2,.066Zm1.323-4.029a1,1,0,0,1-1,1H7.478a1,1,0,0,1,0-2h3.1a1.276,1.276,0,0,0,1.273-1.148,2.991,2.991,0,0,1,2.984-2.694h2.33a2.991,2.991,0,0,1,2.984,2.694,1.276,1.276,0,0,0,1.273,1.148h3.1A1,1,0,0,1,25.522,8.164Zm-11.936-1h4.828a3.3,3.3,0,0,1-.255-.944,1,1,0,0,0-.994-.9h-2.33a1,1,0,0,0-.994.9A3.3,3.3,0,0,1,13.586,7.164Zm1.007,15.151V13.8a1,1,0,0,0-2,0v8.519a1,1,0,0,0,2,0Zm4.814,0V13.8a1,1,0,0,0-2,0v8.519a1,1,0,0,0,2,0Z"></path></svg>
						</button>
					</div>
				</div>
			{/if}
		{/each}
	</div>
</div>

<div class="container fixed bottom-0 flex justify-center items-center pb-4">
	<div class="input-group input-group-divider grid-cols-[auto_1fr_auto] rounded-container-token w-1/2">
		<button class="input-group-shim">+</button>
		<textarea
			bind:value={message}
			class="bg-transparent border-0 ring-0"
			name="prompt"
			id="prompt"
			placeholder="Write a message..."
			rows="1"
		/>
		<button class="variant-filled-primary" on:click={submitPrompt}>Send</button>
	</div>
</div>
