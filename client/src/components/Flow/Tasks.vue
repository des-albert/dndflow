<template>
  <v-container>
    <v-row justify='center'>
      <v-col v-for="(stage, index) in stages" :key='index'>

        <draggable :list="buckets[stage]" group="openTasks" :move='handleMove'>
          <span slot="header" :tag="stage" class="card-text-bold">{{ stage }}</span>
          <v-card color="commentCard" class="list-group mt-3" v-for="task in buckets[stage]" :key="task._id">
            <v-card-text>

              <span class="card-text-bold">{{ task.title}}</span>
            </v-card-text>

            <v-card-text>
              <span class="card-text"> {{ task.description}} </span>
            </v-card-text>

            <v-card-text>
              {{ task.assignee}}<v-icon right @click="goToTask(task._id)">edit</v-icon>
            </v-card-text>

          </v-card>
        </draggable>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { mapGetters } from 'vuex';
import draggable from 'vuedraggable';
import moment from 'moment';

export default {
  name: 'Tasks',

  components: {
    draggable
  },

  data() {
    return {
      stages: ['created', 'assigned', 'in progress', 'on hold', 'complete', 'archive']
    };
  },

  created() {
    this.handleGetTasks();
  },

  computed: {
    ...mapGetters(['user', 'loading', 'tasks', 'buckets'])
  },

  methods: {
    getTimeFromNow(time) {
      return moment(new Date(time)).fromNow();
    },
    goToTask(taskId) {
      this.$router.push(`/task/${taskId}`);
    },

    handleGetTasks() {
      const userRole = localStorage.getItem('role');
      const fullname = localStorage.getItem('fullname');

      switch (userRole) {
        case 'Manager': {
          this.$store.dispatch('getAllTasks');
          break;
        }

        case 'Requester': {
          this.$store.dispatch('getRequestTasks', {
            fullname: fullname
          });
          break;
        }

        case 'Assignee': {
          this.$store.dispatch('getAssignTasks', {
            fullname: fullname
          });

          break;
        }
      }
    },

    handleMove(evt) {
      const movedId = evt.draggedContext.element._id;
      var stageTag = evt.relatedContext.component.rootContainer.firstElementChild.innerText;
      if (this.stages.includes(stageTag)) {
        this.$store.dispatch('changeStatus', {
          taskId: movedId,
          status: stageTag
        });
      }
    }
  }
};
</script>
<style>
.card-text-bold {
  color: #82a3a1;
  font-size: 20px;
  font-weight: bold;
}

.card-text {
  color: #8f6663;
  font-size: 20px;
}
</style>

