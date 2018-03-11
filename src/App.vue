<template>
    <div id="app">
        <header>
            <div class="container-fluid">
                <div class="row">
                    <div class="col">
                        <h1>
                            Todo
                            <span class="badge badge-info">APP</span>
                        </h1>
                    </div>
                    <div class="col text-right">
                        <div class="btn-group margin-top-5px" role="group">
                            <button type="button" class="btn btn-lg btn-outline-info" :class="{ active:isActive('all') }" v-on:click="filter = 'all'">All</button>
                            <button type="button" class="btn btn-lg btn-outline-info" :class="{ active:isActive('todo') }"v-on:click="filter = 'todo'">To do</button>
                            <button type="button" class="btn btn-lg btn-outline-info" :class="{ active:isActive('complete') }"v-on:click="filter = 'complete'">Complete</button>
                        </div>
                    </div>
                    <div class="col">
                        <button class="btn btn-info btn-lg float-right margin-top-5px" v-on:click="showCreate" role="button">
                            New Todo
                        </button>
                    </div>
                </div>
                <hr>
            </div>
        </header>
        <div class="container-fluid">
            <div class="row">
                <div v-for="todo in todos">
                    <div class="card" v-if="filter === 'all' || filter === todo.progress">
                        <div class="card-header" :class="todo.progress">
                            <h5 class="card-title">{{ todo.title }}</h5>
                        </div>
                        <div class="card-body">
                            <p class="card-text">{{ todo.description }}</p>
                            <button
                                    type="submit"
                                    class="btn btn-outline-info"
                                    v-on:click="markComplete(todo.id)"
                                    v-if="todo.progress === 'todo'"
                            >
                                complete
                            </button>
                            <button
                                    type="submit"
                                    class="btn btn-outline-secondary"
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
        <footer>
            <div class="container-fluid">
                <hr>
                <div class="row">
                    <div class="col">
                        <button class="btn btn-outline-success btn-sm float-left" v-on:click="markAllComplete" role="button">All complete</button>
                        <button class="btn btn-outline-danger btn-sm float-right" v-on:click="deleteAll" role="button">Delete All</button>
                    </div>
                </div>
            </div>
        </footer>

        <transition name="modal">
            <div class="modal-mask" v-show="displayModal">
                <div class="modal-wrapper">
                    <div class="modal-container">

                        <div class="modal-header">
                            <slot name="header">
                                New Todo
                            </slot>
                        </div>

                        <div class="modal-body">
                            <slot name="body">
                                <div class="form-group">
                                    <input
                                           class="form-control"
                                           v-bind:class="{ 'is-invalid':invalid.title }"
                                           id="title"
                                           placeholder="Title"
                                           maxlength="30"
                                           min="1"
                                           v-model="todo.title"
                                    >
                                </div>
                                <div class="form-group">
                                    <textarea
                                            class="form-control"
                                            v-bind:class="{ 'is-invalid':invalid.description }"
                                            id="description"
                                            placeholder="Description"
                                            maxlength="150"
                                            rows="3"
                                            v-model="todo.description"
                                    >
                                    </textarea>
                                </div>
                            </slot>
                        </div>

                        <div class="modal-footer">
                            <slot name="footer">
                                <button
                                        class="btn btn-info btn-sm"
                                        v-on:click="addTodo"
                                >
                                    Save
                                </button>
                                <button
                                        class="btn btn-danger btn-sm"
                                        v-on:click="showCreate"
                                >
                                    Cancel
                                </button>
                            </slot>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
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

            isValid() {
                let valid = true;
                if(!this.todo.title) {
                    valid = false;
                    this.invalid.title = true;
                }

                if(!this.todo.description) {
                    valid = false;
                    this.invalid.description = true;
                }

                return valid;
            },

            resetValidation() {
                this.invalid.title = false;
                this.invalid.description = false;
            },

            resetTodo() {
                this.todo.title = '';
                this.todo.description = '';
                this.todo.progress = 'todo';
            },

            showCreate() {
                this.displayModal = !this.displayModal;
                this.resetValidation();
            },

            addTodo() {
                if(!this.isValid()) {
                    return;
                }

                const todo = {
                    id : this.todos.length,
                    title : this.todo.title,
                    description : this.todo.description,
                    progress : 'todo',
                };

                this.storeTodo(todo);

                this.displayModal = false;
                this.resetValidation();
            },

            storeTodo(todo) {
                this.todos.push(todo);

                this.resetTodo();

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
                displayModal: false,
                invalid: {
                    title: false,
                    description: false,
                },
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

    #app {
        margin-bottom:100px;
    }

    .container-fluid {
        margin-right:auto;
        margin-left:auto;
        max-width:900px;
    }

    header {
        padding:15px;
    }

    footer {
        background-color:#FFFFFF;
        position:fixed;
        padding:15px;
        height:90px;
        width:100%;
        bottom:0;
    }

    .margin-top-5px {
        margin-top:5px;
    }

    .card {
        margin:0 15px 15px 0;
        min-height:300px;
        min-width:285px;
    }

    .todo {
        background-color:#17a2b8;
    }

    .modal-mask {
        background-color:rgba(0, 0, 0, .5);
        transition:opacity .3s ease;
        position:fixed;
        display:table;
        z-index:9998;
        height:100%;
        width:100%;
        left:0;
        top:0;
    }

    .modal-wrapper {
        vertical-align:middle;
        display:table-cell;
    }

    .modal-container {
        box-shadow:0 2px 8px rgba(0, 0, 0, .33);
        transition:all .3s ease;
        background-color:#fff;
        border-radius:2px;
        padding:20px 30px;
        margin:0 auto;
        width:500px;
    }

    .modal-header h3 {
        color:#42b983;
        margin-top:0;
    }

    .modal-body {
        margin:20px 0;
    }

    .modal-default-button {
        float:right;
    }

    .modal-enter {
        opacity:0;
    }

    .modal-leave-active {
        opacity:0;
    }

    .modal-enter .modal-container,
    .modal-leave-active .modal-container {
        -webkit-transform:scale(1.1);
        transform:scale(1.1);
    }
</style>
