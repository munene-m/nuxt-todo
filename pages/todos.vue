<template>
  <div class="relative">
    <button
      @click="openModal"
      class="px-4 py-2 text-white bg-cyan-800 rounded-md"
    >
      Create Todo
    </button>
    <div v-for="todo in todos" :key="todo.id" class="flex items-center">
      <p class="bg-zinc-200 w-2/3 px-2 py-1 rounded">{{ todo.task }}</p><span @click="toggleComplete(todo)" :class="['bg-zinc-200', 'py-1', 'px-2', 'w-fit', 'cursor-pointer', 'text-center', 'ml-auto', 'm-3', 'rounded', { 'text-red-500': !todo.is_complete, 'text-green-500': todo.is_complete }]">{{ todo.is_complete ? 'Completed ✅' : 'Pending ⌛' }}</span>
      <button @click="deleteTodo(todo)"><IconsDeleteIcon class="cursor-pointer text-red-500 bg-red-800 bg-opacity-40 p-2  rounded w-8 h-8"/></button>
    </div>
  </div>
  <Teleport to="body">
    <div
      v-if="modalOpen"
      class="font-raleway absolute z-50 top-0 left-0 bg-black bg-opacity-60 backdrop-filter w-full h-full flex justify-center items-center"
    >
      <form
        action=""
        @submit.prevent="handleTodo"
        class="w-1/2 m-auto p-10 border border-slate-700 rounded-md flex flex-col bg-white py-9 px-4 rounded-3xl mx-3 transition-all duration-300 ease-linear"
      >
        <IconsCloseIcon
          @click="modalOpen = false"
          class="cursor-pointer bg-slate-200 rounded-full p-1 ml-auto"
        />
        <label class="block text-sm md:text-base md:pb-2" for="task"
          >Enter task here</label
        >
        <input
          class="bg-slate-300 md:bg-slate-200 w-full p-3 mt-1 rounded-lg outline-transparent"
          type="text"
          v-model="task"
          placeholder="Enter todo task here"
        />
        <span class="text-xs ml-1 mt-1 text-red-600">{{ taskError }}</span>
        
        <button type="submit" class="px-4 py-2 text-white bg-cyan-800 rounded-md mt-7 m-auto">
          Submit
        </button>
      </form>
    </div>
  </Teleport>
</template>
<script setup>
const supabase = useSupabaseClient();
definePageMeta({
  middleware: ["auth"],
});
const user = useSupabaseUser();
// console.log(user.value);

const modalOpen = ref(false);

function openModal() {
  modalOpen.value = true;
}

const task = ref("");
const taskError = ref("");

const todos = ref([])

watch(
  () => task.value,
  (newValue, oldValue) => {
    if (newValue !== "") {
      taskError.value = "";
    }
  }
);

async function handleTodo() {
  task.value === "" ? (taskError.value = "This field is required") : (taskError.value = "");
  try {
      // Insert the todo data into the "todotable" table
      const { data, error } = await supabase
        .from('todotable')
        .insert([{ user_id: user.value.id, task: task.value, is_complete: false }]);

      if (error) {
        // Handle the error case
        console.error(error);
      } else {
        // Data was successfully inserted into the table
        console.log('Todo data saved:', data);
        getTodoData()
      }
    } catch (error) {
      // Handle any unexpected errors
      console.error(error);
    }
    task.value = ""
}

async function getTodoData() {
  try {
    const { data, error } = await supabase.from('todotable').select('*');
    
    if (error) {
      console.error('Error retrieving todo data:', error);
      return;
    }
    
    console.log('Todo data:', data);
    todos.value = data
    // Process the retrieved data as needed
    
  } catch (error) {
    console.error('Error retrieving todo data:', error.message);
  }
}

async function toggleComplete(todo) {
    todo.is_complete = !todo.is_complete;

    try {
      const { data, error } = await supabase
        .from('todotable')
        .update({ is_complete: todo.is_complete })
        .eq('id', todo.id)
        .single();

      if (error) {
        throw error;
      }

      console.log('Todo updated:', data);
    } catch (error) {
      console.error('Error updating todo:', error);
    }
  }

  async function deleteTodo(todo) {
    try {
    const { data, error } = await supabase
      .from('todotable')
      .delete()
      .eq('id', todo.id);

    if (error) {
      console.error('Error deleting todo:', error);
    } else {
      console.log('Todo deleted:', data);

      // Update the todo data in the UI
      getTodoData()
    }
  } catch (error) {
    console.error('Error deleting todo:', error);
  }
  }

onMounted(() => {
  getTodoData()
})
</script>
