<template>
  <router-view/>
  <div class="container">
    <h2>To-Do List</h2>
    <input
        class="form-control"
        type="text"
        v-model='searchText'
        placeholder="Search"
        @keyup.enter="searchTodo"
    >
    <hr/>
    <TodoSimpleForm @add-todo="addTodo"/>
    <div style="color: red">{{error}}</div>

    <div v-if="!todos.length">
      There is nothing to display
    </div>
    <TodoList
        :todos="todos"
        @toggle-todo="toggleTodo"
        @delete-todo="deleteTodo"
    />
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li v-if="currentPage !== 1" class="page-item">
          <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage-1)">Previous</a>
        </li>

        <li
            v-for="page in numberOfPages"
            :key="page"
            class="page-item"
            :class="currentPage === page ? 'active' : ''"
        >
          <a style="cursor: pointer" class="page-link" @click="getTodos(page)">{{page}}</a>
        </li>

        <li v-if="numberOfPages !== currentPage" class="page-item">
          <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage+1)">Next</a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import {ref , computed, watchEffect, watch} from "vue";
import TodoSimpleForm from "@/components/TodoSimpleForm.vue";
import TodoList from "@/components/TodoList.vue";
import axios from "axios";
export default {
  components: {
    TodoSimpleForm,
    TodoList
  },
  setup() {
    const todos = ref([]);
    const error = ref('');
    const numberOfTodos = ref(0);
    const limit = 5;
    const currentPage = ref(1);
    const searchText = ref('');

    // watch([currentPage,numberOfTodos],(v,prev)=>{
    //   console.log(v,prev);
    // })

    watchEffect(() => {
      // console.log(currentPage.value);
      // console.log(numberOfTodos.value);
    });

    const numberOfPages = computed(() =>{
      return Math.ceil(numberOfTodos.value/limit);
    })

    const getTodos = async (page = currentPage.value) =>{
      currentPage.value = page;
      error.value = '';
      try{
        const res = await axios.get(`http://localhost:3001/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${limit}`);
        numberOfTodos.value = res.headers['x-total-count'];
        todos.value = res.data;

      } catch (err){
        console.log(err);
        error.value = 'Something went wrong.';
      }
    };
    getTodos();

    const addTodo = async (todo) => {
      // 데이터베이스 투두를 저장
      error.value = ''; // 다시 요청할때 이전 error value 초기화 시키기 위해
      try {
        await axios.post('http://localhost:3001/todos', {
          subject: todo.subject,
          completed: todo.completed,
        });
        getTodos(1);
      } catch (err){
        error.value = 'Something went wrong';
      }
      //     .then(res => {
      //   console.log(res);
      //   todos.value.push(res.data);
      // }).catch(err => {
      //   error.value = 'Something went wrong';
      //   console.log(err);
      // });
    };

    const toggleTodo = async  (index) => {
      error.value = '';
      const id = todos.value[index].id;
      try {
        await axios.patch('http://localhost:3001/todos/'+id,{
          completed: !todos.value[index].completed
        });
        todos.value[index].completed = !todos.value[index].completed;
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong';
      }
    };


    const deleteTodo = async (index)=>{
      error.value = '';
      const id = todos.value[index].id;
      try {
        await axios.delete('http://localhost:3001/todos/' + id);
        getTodos(1);
      } catch (err){
        console.log(err);
        error.value = 'Something went wrong.';
      }
    };

    let timeout = null;
    const searchTodo = () =>{
      clearTimeout(timeout);
      getTodos(1);

    }
    watch(searchText,()=>{
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        getTodos(1);
      }, 2000);
    });
    // const filteredTodos = computed(() =>
    // {
    //   if (searchText.value) {
    //     return todos.value.filter(todo => {
    //       return todo.subject.includes(searchText.value);
    //     })
    //   }
    //   return todos.value;
    // });



    return {
      searchTodo,
      todos,
      deleteTodo,
      addTodo,
      toggleTodo,
      searchText,
      // filteredTodos,
      error,
      numberOfPages,
      currentPage,
      getTodos,

    };
  }
}
</script>

<style>
.todo {
  color: rgb(128, 128, 128);
  text-decoration: line-through;
}
</style>