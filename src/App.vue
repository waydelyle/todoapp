<template>
    <div id="app">
        <div class="container">
            <hr>
            <div class="row">
                <div class="col">
                    <div class="jumbotron">
                        <h1 class="display-4">TodoApp</h1>
                        <p class="lead">Your personal todo list.</p>
                        <hr class="my-4">
                        <p class="lead">
                            <button class="btn btn-primary btn-lg" v-on:click="showCreate" role="button">
                                Add Todo
                            </button>
                        </p>
                    </div>
                </div>
                <transition name="fade">
                    <div class="col" v-show="displayCreate">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">
                                    New Todo
                                </h5>
                                <div class="form-group">
                                    <label for="title">Title</label>
                                    <input type="email" class="form-control" id="title" placeholder="Title" maxlength="30" min="1" v-model="todo.title">
                                </div>
                                <div class="form-group">
                                    <label for="description">Description</label>
                                    <textarea class="form-control" id="description" placeholder="Description" maxlength="150" rows="3" v-model="todo.description"></textarea>
                                </div>
                                <button type="submit" class="btn btn-primary btn-lg" v-on:click="addTodo">Save</button>
                                <button class="btn btn-danger btn-lg" v-on:click="showCreate" role="button">Cancel</button>
                            </div>
                        </div>
                    </div>
                </transition>
            </div>
            <div class="row">
                <div class="col text-left">
                    <button class="btn btn-outline-success btn-sm" v-on:click="markAllComplete" role="button">All complete</button>
                </div>
                <div class="col text-right">
                    <button class="btn btn-outline-danger btn-sm" v-on:click="deleteAll" role="button">Delete All</button>
                </div>
            </div>
            <hr>
            <div class="row">
                <div class="col text-center">
                    <div class="btn-group" role="group" aria-label="Basic example">
                        <button type="button" class="btn btn-outline-info" :class="{ active:isActive('all') }" v-on:click="filter = 'all'">All</button>
                        <button type="button" class="btn btn-outline-info" :class="{ active:isActive('todo') }"v-on:click="filter = 'todo'">To do</button>
                        <button type="button" class="btn btn-outline-info" :class="{ active:isActive('complete') }"v-on:click="filter = 'complete'">Complete</button>
                    </div>
                </div>
            </div>
            <hr>
            <div class="row">
                <div v-for="todo in todos">
                    <div class="card" v-if="filter === 'all' || filter === todo.progress">
                        <div class="card-header">
                            <h5 class="card-title">{{ todo.title }}</h5>
                        </div>
                        <div class="card-body">
                            <p class="card-text">{{ todo.description }}</p>
                            <button
                                    type="submit"
                                    class="btn btn-outline-success"
                                    v-on:click="markComplete(todo.id)"
                                    v-if="todo.progress === 'todo'"
                            >
                                complete
                            </button>
                            <button
                                    type="submit"
                                    class="btn btn-outline-info"
                                    v-on:click="markTodo(todo.id)"
                                    v-else=""
                            >
                                todo
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import LocalStorage from 'local-storage';

    export default {
        name: 'app',

        created() {
            const todos = LocalStorage.get('todos');

            if (todos !== null) {
                this.todos = todos;
            } else {
                LocalStorage.set('todos', []);
            }
        },

        methods: {

            isActive: function (item) {
                return this.filter === item;
            },

            showCreate() {
                this.displayCreate = !this.displayCreate;
            },

            addTodo() {
                if(!this.todo.title || !this.todo.description) {
                    return;
                }

                const todo = {
                    id : this.todos.length,
                    title : this.todo.title,
                    description : this.todo.description,
                    progress : 'todo',
                };

                this.todos.push(todo);

                this.todo.title = '';
                this.todo.description = '';
                this.todo.progress = 'todo';

                let todos = LocalStorage.get('todos');

                todos.push(todo);

                LocalStorage.set('todos', todos);
            },

            markComplete(id) {
                this.todos[id]['progress'] = 'complete';

                LocalStorage.set('todos', this.todos);
            },

            markTodo(id) {
                this.todos[id]['progress'] = 'todo';

                LocalStorage.set('todos', this.todos);
            },

            markAllComplete() {
                let i;

                for(i = 0; i < this.todos.length; i++){
                    this.todos[i]['progress'] = 'complete';
                }

                LocalStorage.set('todos', this.todos);
            },

            deleteAll() {
                LocalStorage.clear();
                LocalStorage.set('todos', []);
                this.todos = [];
            },

        },

        data() {
            return {
                filter: 'all',
                displayCreate: false,
                todo: {
                    title: '',
                    description: '',
                    progress: '',
                },
                todos: [],
            }
        }

    }
</script>
<style lang="scss">
    @import '../node_modules/bootstrap/scss/bootstrap.scss';

    .card {
        max-width:300px;
        margin:0 15px 15px 0;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .3s;
    }

    .fade-enter, .fade-leave-to {
        opacity: 0;
    }
</style>
