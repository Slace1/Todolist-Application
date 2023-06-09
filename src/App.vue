<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const name = ref('')
const isAuthenticated = ref(false) // Utilisateur authentifié
const username = ref('')

const input_content = ref('')
const input_category = ref(null)
const input_expired_date = ref('')

const todos_asc = computed(() => todos.value.sort((a,b) =>{
	return a.expired_date - b.expired_date
}))

const todos_expired_sorted = computed (()=> {
    const now= new Date().getTime();
    return todos_asc.value.filter(todo => todo.expired_date < now && !todo.done);
})

const todos_not_expired_sorted = computed (()=> {
    const now= new Date().getTime();
    return todos_asc.value.filter(todo => todo.expired_date > now && !todo.done);
})

const todos_done = computed (()=> {
    return todos_asc.value.filter(todo => todo.done);
})

watch(name, (newVal) => {
	localStorage.setItem('name', newVal)
})

watch(todos, (newVal) => {
	localStorage.setItem('todos', JSON.stringify(newVal))
}, {
	deep: true
})

const addTodo = () => {
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
	}

	todos.value.push({
		content: input_content.value,
		category: input_category.value,
		done: false,
		editable: false,
		createdAt: new Date().getTime(),
		expired_date:new Date(input_expired_date.value).getTime()
	})
}

const removeTodo = (todo) => {
	todos.value = todos.value.filter((t) => t !== todo)
}

const login = () => {
	if (username.value.trim() === '') {
		return
	}

	name.value = username.value
	localStorage.setItem('name', name.value)

	isAuthenticated.value = true // Authentifier l'utilisateur
}

onMounted(() => {
	name.value = localStorage.getItem('name') || ''

	if (name.value !== '') {
		isAuthenticated.value = true // Authentifier l'utilisateur s'il est déjà connecté
	}

	todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>


<template>
	<main class="app" v-if="isAuthenticated">
		<section class="greeting">
			<h2 class="title">
				What's up, <input type="text" id="name" placeholder="Name here" v-model="name">
			</h2>
		</section>

		<section class="create-todo">
			<h3>CREATE A TODO</h3>

			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>What's on your todo list?</h4>
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="e.g. make a video"
					v-model="input_content" />
				
				<h4>Pick a category</h4>
				<div class="options">

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category1" 
							value="business"
							v-model="input_category" />
						<span class="bubble business"></span>
						<div>Business</div>
					</label>

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category2" 
							value="personal"
							v-model="input_category" />
						<span class="bubble personal"></span>
						<div>Personal</div>
					</label>

					<h4>When is it due?</h4>
					<input type="date" name="due-date" id="due-date" v-model="input_expired_date" />


				</div>

				<input type="submit" value="Add todo" />
			</form>
		</section>

		<section class="todo-list">
			<h3>TODO LIST</h3>
			<div class="list" id="todo-list">

				<div v-for="todo in todos_not_expired_sorted" :class="`todo-item ${todo.done && 'done'}`">
					<label>
						<input type="checkbox" v-model="todo.done" />
						<span :class="`bubble ${
							todo.category == 'business' 
								? 'business' 
								: 'personal'
						}`"></span>
					</label>

					<div class="todo-content">
						<input type="text" v-model="todo.content" />
						<p>Deadline: {{ new Date(todo.expired_date).toLocaleDateString() }}</p> <!-- Ajout de la date limite -->
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>


		<section class="todo-list">
			<h3>EXPIRED TODO LIST</h3>
			<div class="list" id="todo-list">

				<div v-for="todo in todos_expired_sorted" :class="`todo-item ${todo.done && 'done'}`">
					<label>
						<input type="checkbox" v-model="todo.done" />
						<span :class="`bubble ${
							todo.category == 'business' 
								? 'business' 
								: 'personal'
						}`"></span>
					</label>

					<div class="todo-content">
						<input type="text" v-model="todo.content" />
						<p>Deadline: {{ new Date(todo.expired_date).toLocaleDateString() }}</p> <!-- Ajout de la date limite -->
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>

		<section class="todo-list">
			<h3>DONE TODO LIST</h3>
			<div class="list" id="todo-list">

				<div v-for="todo in todos_done" :class="`todo-item ${todo.done && 'done'}`">
					<label>
						<input type="checkbox" v-model="todo.done" />
						<span :class="`bubble ${
							todo.category == 'business' 
								? 'business' 
								: 'personal'
						}`"></span>
					</label>

					<div class="todo-content">
						<input type="text" v-model="todo.content" />
						<p>Deadline: {{ new Date(todo.expired_date).toLocaleDateString() }}</p> <!-- Ajout de la date limite -->
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>

		<!-- Reste du code actuel ici -->

	</main>
	<main v-else>
		<section class="login">
			<h2 class="title">Enter your name to login:</h2>
			<form @submit.prevent="login">
				<label>
					Name:
					<input type="text" v-model="username" required />
				</label>
				<input type="submit" value="Login" />
			</form>
		</section>
	</main>
</template>
