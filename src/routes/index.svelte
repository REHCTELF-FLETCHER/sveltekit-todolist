<script>
	import { initializeApp, getApps, getApp } from "firebase/app";
	import {
	  getFirestore,
	  collection,
	  onSnapshot,
	  doc,
	  addDoc,
	  updateDoc,
	  deleteDoc,
	} from "firebase/firestore";
	import { firebaseConfig } from "$lib/firebaseConfig";
	import { browser } from "$app/env";
  
	const firebaseApp =
	  browser &&
	  (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp());
  
	const db = browser && getFirestore();
  
	const colRef = browser && collection(db, "todos");
  
	let todos = [];
  
	const unsubscribe =
	  browser &&
	  onSnapshot(colRef, (querySnapshot) => {
		let fbTodos = [];
		querySnapshot.forEach((doc) => {
		  let todo = { ...doc.data(), id: doc.id };
		  fbTodos = [todo, ...fbTodos];
		});
		todos = fbTodos;
	  });
  
  
	let task = "";
	let error = "";
  
	const addTodo = async () => {
	  if (task !== "") {
		const docRef = await addDoc(collection(db, "todos"), {
		  task: task,
		  isComplete: false,
		  createdAt: new Date(),
		});
		error = "";
	  } else {
		error = "Task is empty";
	  }
	  task = "";
	};
  
	const markTodoAsComplete = async (item) => {
	  await updateDoc(doc(db, "todos", item.id), {
		isComplete: !item.isComplete,
	  });
	};
  
	const deleteTodo = async (id) => {
	  await deleteDoc(doc(db, "todos", id));
	};
  
	const keyIsPressed = (event) => {
	  if (event.key === "Enter") {
		addTodo();
	  }
	};
  </script>
  
  <input type="text" placeholder="Add a task" bind:value={task} class="textbox" />
  <button on:click={addTodo} class=add >Add </button>
  
  <ol class=todoitem>
	{#each todos as item}
	  <li class:complete={item.isComplete}>
		<span>
		  {item.task}
		</span>
		<span>
		  <button on:click={() => markTodoAsComplete(item)}>✔</button>
		  <button on:click={() => deleteTodo(item.id)}>✘</button>
		</span>
	  </li>
	{:else}
	  <p class=text >No todos found</p>
	{/each}
	<p class="error">{error}</p>
  </ol>
  
  <svelte:window on:keydown={keyIsPressed} />
  
  <style>
	.complete {
	  text-decoration: line-through;
	}
	.error {
	  color: red;
	}

	:global(body) {
      background-color:#3b948f9c;
}

.textbox {
	border-radius:5px;
	text-align: center;
	position: absolute;
  left: 45%;
  top: 10px;
}

.add {
	border-radius:5px;
	text-align: center;
	position: absolute;
  left: 55%;
  top: 10px;
}

.text {
	position: absolute; 
  left: 50%; 
  top: 25px;
}

.todoitem {
	position: absolute;
  left: 45%;
  top: 25px;
}

@font-face {
  font-family: 'SFPRODISPLAYBOLD';
  font-style: normal;
  font-weight: 400;
  src: url(fonts/SFPRODISPLAYBOLD.OTF);
}@font-face {
  font-family: 'SFPRODISPLAYMEDIUM';
  font-style: normal;
  font-weight: 400;
  src: url(fonts/SFPRODISPLAYMEDIUM.OTF);
}
</style>