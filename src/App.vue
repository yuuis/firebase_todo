<template>
  <div id="app">
    <h1>tasks</h1>
    <div class="form-inline">
      <div class="new-task-input form-group">
        <input type="text" class="new-task-name form-control" v-model="newTaskName" placeholder="task name">
        <button class="btn btn-success" type="submit" v-on:click="createTask()">create task</button>
      </div>
    </div>
    <div class="filter-buttons">
      <button class="btn btn-primary" type="submit" v-on:click="showTaskType = 'all'">ALL</button>
      <button class="btn btn-primary" type="submit" v-on:click="showTaskType = 'active'">TODO</button>
      <button class="btn btn-primary" type="submit" v-on:click="showTaskType = 'complete'">DONE</button>
    </div>
    <div class="tasks">
      <ul v-for="(todo, key) in filteredTasks" class="list-unstyled list-group task">
        <li class="list-group-item"><input class="toggle" type="checkbox" v-model="todo.isDone" v-on:click="updateTask(todo, key)">{{todo.name}}
        <button class="btn btn-danger" type="submit" v-on:click="removeTask(key)">remove</button></li>
      </ul>
    </div>
  </div>
</template>

<script>
import Firebase from 'firebase'

export default {
  name: 'App',
  created: function () {
    this.database = Firebase.database()
    this.tasksRef = this.database.ref('tasks')

    var _this = this
    this.tasksRef.on('value', function (snapshot) {
      _this.tasks = snapshot.val()
    })
  },
  computed: {
    filteredTasks: function () {
      if (this.showTaskType === 'all') {
        return this.tasks
      } else {
        var showIsDone = false
        if (this.showTaskType === 'complete') { showIsDone = true }
        var filteredTasks = {}
        for (var key in this.tasks) {
          var task = this.tasks[key]
          if (task.isDone === showIsDone) { filteredTasks[key] = task }
        }
        return filteredTasks
      }
    }
  },
  methods: {
    createTask: function () {
      if (this.newTaskName === '') { return }
      this.tasksRef.push({
        name: this.newTaskName,
        isDone: false
      })
      this.newTaskName = ''
    },
    updateTask: function (task, key) {
      task.isDone = !task.isDone
      var updates = {}
      updates['/tasks/' + key] = task
      this.database.ref().update(updates)
    },
    removeTask: function (key) {
      this.database.ref('tasks').child(key).remove()
    }
  },

  data () {
    return {
      database: null,
      tasksRef: null,
      newTaskName: '',
      showTaskType: 'all',
      tasks: []
    }
  }
}
</script>

<style>
#app {
  text-align: center;
  margin-top: 5%;
}
.form-inline {
  display: flex;
  justify-content: center;
  margin-bottom: 2%;
}
.new-task-name {
  width: 200px;
  margin: auto;
}
.filter-buttons {
  width: 200px;
  margin: auto;
  margin-bottom: 1%;
  display: flex;
  justify-content: space-around;
}
.btn-danger {
  float: right;
}
.btn-primary {
  /* width: 25%; */
}
.list-group-item {
  width: 300px;
  text-align: center;
  margin: auto;
}

</style>
