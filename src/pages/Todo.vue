<template>
  <q-page class="bg-grey-3 column">
    <div class="row q-pa-sm bg-primary">
      <q-input
        @keyup.enter="addTask"
        class="col"
        filled
        square
        v-model="newTask"
        bg-color="white"
        placeholder="Add task"
        dense>
        <template v-slot:append>
          <q-btn 
            @click="addTask"
            round
            dense
            flat
            icon="add" />
        </template>
      </q-input>
    </div>
    <q-list
      class="bg-white"
      separator
      bordered>
      <q-item
        v-for="task in tasks"
        :key="task.title"
        @click="task.done = !task.done; setDone(task)"
        :class="{ 'done bg-pink-2': task.done}"
        clickable
        v-ripple>
        <q-item-section avatar>
          <q-checkbox
            v-model="task.done"
            class="no-pointer-events"
            color="primary" />
        </q-item-section>
        <q-item-section>
          <q-item-label>{{task.title}}</q-item-label>
        </q-item-section>
        <q-item-section
          v-if="task.done"
          side>
          <q-btn
            @click.stop="confirmModal = true; confirmModalTask = task"
            flat
            round
            dense
            color="primary"
            icon="delete"
          />
        </q-item-section>
      </q-item>
    </q-list>
    <div
      v-if="!tasks.length"
      class="no-tasks absolute-center">
      <q-icon
        name="check"
        size="100px"
        color="primary"
      />
      <div class="text-h5 text-primary text-center">
        No tasks
      </div>
    </div>
    <q-dialog v-model="confirmModal" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <span class="q-ml-sm">Are you sure to delete this task?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="primary" v-close-popup />
          <q-btn flat label="Confirm" color="primary" v-close-popup @click="deleteTask" />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-page>
</template>

<script lang="ts">
import { LocalStorage } from 'quasar';
import { defineComponent, ref } from 'vue';
interface TaskInterface {
  title: string
  done: boolean
}
export default defineComponent({
  name: 'TodoPage',

  mounted() {
    for (const [title, done] of Object.entries(LocalStorage.getAll() as boolean)) {
      this.tasks.push({ title, done })
    }
  },

  methods: {
    addTask() {
      this.tasks.push({
        title: this.newTask,
        done: false
      })
      LocalStorage.set(this.newTask, false)
      this.newTask = ''
    },
    
    setDone(task: TaskInterface) {
      LocalStorage.set(task.title, task.done)
    },

    deleteTask() {
      this.tasks = this.tasks.filter(storeTask => storeTask.title !== this.confirmModalTask.title)
      this.$q.notify('Task deleted')
      LocalStorage.remove(this.confirmModalTask.title)
    }
  },

  setup() {
    return {
      newTask: ref(''),
      confirmModalTask: ref<TaskInterface | Record<string, never>>({}),
      confirmModal: ref(false),
      tasks: ref<TaskInterface[]>([])
    }
  }
});
</script>

<style lang="scss">
  .done {
    .q-item__label {
      text-decoration: line-through;
      color: #808080;
    }
  }
  .no-tasks {
    opacity: 0.5;
  }
</style>