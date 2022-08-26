<script lang="ts">
	import '../app.css';
	import { onMount } from 'svelte';

	let iframe: HTMLIFrameElement;
	let options = ['untitled'];
	let current = 'untitled';
	let loaded = false;
	let hue = 10;
	let dirty = 0;
	let invert = false;

	$: if (loaded) {
		const data = localStorage.getItem(current + '-page');
		localStorage.setItem('current', current);
		if (data) {
			setHTML(data);
		} else clearHTML();
	}

	$: loaded && localStorage.setItem('options', JSON.stringify(options));
	$: loaded &&
		(document.body.style.filter = `hue-rotate(${hue * 18}deg) ${invert ? 'invert()' : ''}`);

	function kd(e: KeyboardEvent) {
		if (e.key.toLowerCase() === 's' && e.ctrlKey) {
			e.preventDefault();
			save();
		} else {
			dirty++;
		}
	}

	function onLoad() {
		loaded = true;
		const cw = iframe.contentWindow!;
		cw.addEventListener('keydown', kd);
		// const oldec = cw.document.execCommand.bind(cw.document);
		// cw.document.execCommand = (...args) => {
		// 	if (args[0] == 'insertHTML') {
		// 		oldec(...args);
		// 	}
		// };
	}

	onMount(() => {
		const warning = (e: BeforeUnloadEvent) => {
			if (dirty < 10) return;
			e = e || window.event;
			e && (e.returnValue = `Are you sure?`);
			return `Are you sure?`;
		};
		window.addEventListener(`beforeunload`, warning);
		if (localStorage.getItem('options'))
			options = JSON.parse(localStorage.getItem('options') ?? 'null');
		if (localStorage.getItem('current')) current = localStorage.getItem('current')!;
		if (iframe.contentDocument?.readyState) onLoad();
		else iframe.onload = onLoad;
	});

	function getHTML() {
		const ans = iframe.contentDocument!.getElementById('answer1');
		return ans!.innerHTML;
	}

	function setHTML(html: string | null) {
		if (!html) return;
		const ans = iframe.contentDocument!.getElementById('answer1');
		ans!.innerHTML = html;
	}

	function clearHTML() {
		const ans = iframe.contentDocument!.getElementById('answer1');
		ans!.innerHTML = '';
	}

	function createNew() {
		const name = prompt('name?');
		if (!name) {
			alert('enter name');
		} else if (options.includes(name)) {
			alert('name in use');
		} else {
			save();
			options = [...options, name];
			current = name;
		}
	}

	function save() {
		localStorage.setItem(current + '-page', getHTML());
		blink(document.querySelector('.indicator')!);
		dirty = 0;
	}

	function onChange() {
		save();
	}

	function blink(e: HTMLElement) {
		e.style.filter = 'saturate(100)';
		setTimeout(function () {
			e.style.filter = '';
		}, 300);
	}

	function rename() {
		const oldName = current;
		const newName = prompt('new name?');
		if (!newName) {
			alert('enter name');
		} else if (options.includes(newName)) {
			alert('name in use');
		} else {
			current = newName;
			save();
			options.splice(options.indexOf(oldName), 1);
			options = [...options, newName];
			localStorage.removeItem(oldName + '-page');
		}
	}

	function del() {
		if (!confirm(`delete ${current}?`)) return;
		options.splice(options.indexOf(current), 1);
		localStorage.removeItem(current + '-page');
		if (options.length === 0) {
			options = [...options, 'untitled'];
			current = 'untitled';
			clearHTML();
		} else {
			current = options[0];
			options = options;
		}
	}
</script>

<svelte:window on:keydown={kd} />

<svelte:head>
	<title>m-edit</title>
	<meta name="description" content="abitti-editor but it saves your work" />
</svelte:head>

<section class="flex flex-col h-screen">
	<div class="flex overflow-x-auto overflow-y-hidden">
		<select class="border-l-[0px]" on:change={onChange} bind:value={current}>
			{#each options as o}
				<option value={o}>{o}</option>
			{/each}
		</select>
		<button on:click={createNew}>New </button>
		<button on:click={save}>Save</button>
		<button on:click={rename}>Rename</button>
		<button on:click={del}>Delete</button>
		<input class="border-r-[3px]" bind:value={hue} type="number" />
		<input type="checkbox" bind:checked={invert} />
		<div
			class="border-slate-600 bg-slate-100 border-[3px] indicator min-h-[36.5px] w-full min-w-[37px] mr-[1.5px] flex justify-center items-center p-1 whitespace-nowrap"
		>
			Ctrl-S to save
		</div>
	</div>
	<iframe class="h-full" bind:this={iframe} src="math.html">
		Your browser doesn't support iframes
	</iframe>
</section>

<style>
	:global(body) {
		transition: filter 200ms linear;
		box-sizing: border-box;
		filter: hue-rotate(180deg);
	}
	.indicator {
		transition: filter 200ms linear;
	}
	button:hover,
	select:hover {
		filter: brightness(93%);
	}
	button,
	select,
	input {
		padding: 5px 20px;
		border-width: 3px 1.5px;
		@apply bg-slate-100;
		@apply border-slate-600;
	}
	input[type='number'] {
		max-width: 100px;
	}
	input[type='checkbox'] {
		height: 38px;
		min-width: 40px;
		@apply bg-slate-100;
	}
</style>
