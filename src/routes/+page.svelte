<script lang="ts">
	interface Task {
		id: number;
		text: string;
		completed: boolean;
		el: HTMLDivElement | null;
		movedArray: number;
	}

	let currentToDo: string = '';
	let dropped_in: boolean = false;
	let activeEvent = '';
	let originalX = '';
	let originalY = '';
	let drop_zone: any;

	let tasks: Task[] = [
		{ id: 0, text: 'Hola', completed: false, el: null, movedArray: -1 },
		{ id: 1, text: 'Adios', completed: false, el: null, movedArray: -1 }
	];

	let importantDroppedTask: Task[] = [];
	let eliminateDroppedTask: Task[] = [];
	let delegateDroppedTask: Task[] = [];
	let scheduleDroppedTask: Task[] = [];

	function addTask(event: Event | undefined) {
		if (event !== undefined) {
			let target = event.target as HTMLInputElement;
			let text = target.value;
			tasks = [
				...tasks,
				{ id: tasks.length + 1, text, completed: false, el: null, movedArray: -1 }
			];
		}
	}

	function removeTask(id: number, arrayToRemove: number) {
		switch (arrayToRemove) {
			case -1:
				tasks = tasks.filter((task) => task.id !== id);
				break;
			case 0:
				importantDroppedTask = importantDroppedTask.filter((task) => task.id !== id);
				break;
			case 1:
				eliminateDroppedTask = eliminateDroppedTask.filter((task) => task.id !== id);
				break;
			case 2:
				delegateDroppedTask = delegateDroppedTask.filter((task) => task.id !== id);
				break;
			case 3:
				scheduleDroppedTask = scheduleDroppedTask.filter((task) => task.id !== id);
		}
	}

	function handleDragOver(event: DragEvent) {
		event.preventDefault();
		console.log('Dragging over the element');
	}

	function handleDragDrop(e: any, listToAdd: number) {
		e.preventDefault();
		var element_id = e.dataTransfer.getData('text');
		let dropped_element: Task = tasks.find((task) => task.id == element_id)!;
		switch (listToAdd) {
			case 0:
				dropped_element.movedArray = 0;
				importantDroppedTask = [...importantDroppedTask, dropped_element];
				break;
			case 1:
				dropped_element.movedArray = 1;
				eliminateDroppedTask = [...eliminateDroppedTask, dropped_element];
				break;
			case 2:
				dropped_element.movedArray = 2;
				delegateDroppedTask = [...delegateDroppedTask, dropped_element];
				break;
			case 3:
				dropped_element.movedArray = 3;
				scheduleDroppedTask = [...scheduleDroppedTask, dropped_element];
		}
		tasks = tasks.filter((task) => task.id !== dropped_element.id);
		dropped_in = true;
	}

	function handleDragStart(e: any) {
		e.dataTransfer.dropEffect = 'move';
		e.dataTransfer.setData('text', e.target.getAttribute('id'));
	}

	function handleDragEnd(e: any) {
		dropped_in = false;
	}

	function handleTouchStart(e: any) {
		originalX = e.target.offsetLeft - 10 + 'px';
		originalY = e.target.offsetTop - 10 + 'px';
		activeEvent = 'start';
	}

	function handleTouchMove(e: any) {
		let touchLocation = e.targetTouches[0];
		let pageX = Math.floor(touchLocation.pageX - 50) + 'px';
		let pageY = Math.floor(touchLocation.pageY - 50) + 'px';
		e.target.style.position = 'absolute';
		e.target.style.left = pageX;
		e.target.style.top = pageY;
		activeEvent = 'move';
	}

	function handleTouchEnd(e: any) {
		e.preventDefault();
		if (activeEvent === 'move') {
			let pageX = parseInt(e.target.style.left) - 50;
			let pageY = parseInt(e.target.style.top) - 50;

			if (
				detectTouchEnd(
					drop_zone.offsetLeft,
					drop_zone.offsetTop,
					pageX,
					pageY,
					drop_zone.offsetWidth,
					drop_zone.offsetHeight
				)
			) {
				let dropped_element: Task = tasks.find((task) => task.id == e.target.id)!;
				importantDroppedTask = [...importantDroppedTask, dropped_element];
				dropped_in = true;
				e.target.style.position = 'initial';
				dropped_in = true;
			} else {
				e.target.style.left = originalX;
				e.target.style.top = originalY;
			}
		}
	}

	function detectTouchEnd(x1: number, y1: number, x2: number, y2: number, w: number, h: any) {
		//Very simple detection here
		if (x2 - x1 > w) return false;
		if (y2 - y1 > h) return false;
		return true;
	}
</script>

<h1 class="text-7xl m-2">To-Do List</h1>
<div class="flex">
	<div class="mr-auto">
		<input
			type="text"
			class="p-4 m-2"
			placeholder="Add a new task..."
			bind:value={currentToDo}
			on:keydown={(e) => {
				if (e.key === 'Enter') {
					addTask(e);
					currentToDo = '';
				}
			}}
		/>
		<ul>
			{#each tasks as task (task.id)}
				<div
					role="listitem"
					class="todo m-2"
					id={task.id.toString()}
					draggable="true"
					bind:this={task.el}
					on:dragstart={handleDragStart}
					on:dragend={handleDragEnd}
					on:touchstart={handleTouchStart}
					on:touchmove={handleTouchMove}
					on:touchend={handleTouchEnd}
				>
					<input type="checkbox" class="checkbox" bind:checked={task.completed} />
					<label for="taskname">
						{task.text}
					</label>
					<button class="btn btn-neutral" on:click={() => removeTask(task.id, task.movedArray)}
						>Delete</button
					>
				</div>
			{/each}
		</ul>

		<div class="container m-2">
			<div class="do">
				<h1 class="bold text-lg">Do</h1>

				<div
					role="listitem"
					on:drop={(e) => handleDragDrop(e, 0)}
					bind:this={drop_zone}
					id="drop_zone"
					on:dragover={handleDragOver}
				>
					{#each importantDroppedTask as drop}
						<div>
							<input type="checkbox" bind:checked={drop.completed} />
							<label for="taskname">
								{drop.text}
							</label>
							<button class="btn btn-neutral" on:click={() => removeTask(drop.id, drop.movedArray)}
								>Delete</button
							>
						</div>
					{/each}
				</div>
			</div>
			<div class="eliminate">
				<h1 class="bold text-lg">Eliminate</h1>

				<div
					role="listitem"
					on:drop={(e) => handleDragDrop(e, 1)}
					bind:this={drop_zone}
					id="drop_zone"
					on:dragover={handleDragOver}
				>
					{#each eliminateDroppedTask as drop}
						<div>
							<input type="checkbox" bind:checked={drop.completed} />
							<label for="taskname">
								{drop.text}
							</label>
							<button class="btn btn-neutral" on:click={() => removeTask(drop.id, drop.movedArray)}
								>Delete</button
							>
						</div>
					{/each}
				</div>
			</div>
			<div class="delegate">
				<h1 class="bold text-lg">Delegate</h1>

				<div
					role="listitem"
					on:drop={(e) => handleDragDrop(e, 2)}
					bind:this={drop_zone}
					class="bg-primary-content"
					id="drop_zone"
					on:dragover={handleDragOver}
				>
					{#each delegateDroppedTask as drop}
						<div>
							<input type="checkbox" bind:checked={drop.completed} />
							<label for="taskname">
								{drop.text}
							</label>
							<button class="btn btn-neutral" on:click={() => removeTask(drop.id, drop.movedArray)}
								>Delete</button
							>
						</div>
					{/each}
				</div>
			</div>
			<div class="schedule">
				<h1 class="bold text-lg">Schedule</h1>

				<div
					role="listitem"
					on:drop={(e) => handleDragDrop(e, 3)}
					bind:this={drop_zone}
					id="drop_zone"
					on:dragover={handleDragOver}
				>
					{#each scheduleDroppedTask as drop}
						<div>
							<input type="checkbox" bind:checked={drop.completed} />
							<label for="taskname">
								{drop.text}
							</label>
							<button class="btn btn-neutral" on:click={() => removeTask(drop.id, drop.movedArray)}
								>Delete</button
							>
						</div>
					{/each}
				</div>
			</div>
		</div>
	</div>
</div>

<style>
	:global(html),
	:global(body) {
		margin: 8;
		height: 100%;
		overflow: auto;
		user-select: none;
		-webkit-user-select: none;
	}

	#drop_zone {
		background-color: #050516;
		border: #999 1px solid;
		width: 280px;
		height: 200px;
		padding: 8px;
		font-size: 19px;
	}

	.container {
		display: grid;
		grid-template-columns: '1fr 1fr';
		grid-template-rows: '1fr 1fr';
	}

	.do {
		grid-column: 1/1;
		grid-row: 1/1;
	}

	.eliminate  {
		grid-column: 2/2;
		grid-row: 1/1;
	}
	.delegate {
		grid-column: 1/1;
		grid-row: 2/2;
	}
	.schedule {
		grid-column: 2/2;
		grid-row: 2/2;
	}
</style>
