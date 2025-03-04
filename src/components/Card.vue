<template>
  <div>
    <div class="card draggable" draggable="true" v-for="task in tasks" :key="task.id">
      <div class="card">
        <div class="card-body p-2 border-left border-success">
          <div class="card-title">
            <a href class="lead font-weight-light">{{task.title}}</a>
          </div>
          <p>{{task.description}}</p>
          <button @click="editTask(task)" class="btn btn-secondary">Edit</button>
          <button @click="deleteTask(task.id)" class="btn btn-secondary">Delete</button>
          <div class="card-title">
            <i
              v-if="!firstPosition"
              @click="changeCategory(prevCategoryId, task.id)"
              class="fa fa-angle-left button-arrow"
            ></i>
            <i
              v-if="!lastPosition"
              @click="changeCategory(nextCategoryId, task.id)"
              class="fa fa-angle-right float-right button-arrow"
            ></i>
          </div>
        </div>
      </div>
    </div>
    <loading :active.sync="isLoading" :can-cancel="false" :is-full-page="fullPage"></loading>
  </div>
</template>

<script>
import axios from "axios";
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";
export default {
  name: "Card",
  data: function() {
    return {
      tasks: [],
      isLoading: false,
      fullPage: true
    };
  },
  props: [
    "projectId",
    "categoryId",
    "firstPosition",
    "lastPosition",
    "prevCategoryId",
    "nextCategoryId"
  ],
  components: {
    Loading
  },
  created: function() {
    this.loadTasks();
  },
  methods: {
    loadTasks() {
      this.isLoading = true;
      axios
        .get(
          `https://still-basin-93678.herokuapp.com/tasks?ProjectId=${this.projectId}&CategoryId=${this.categoryId}`,
          {
            headers: {
              access_token: localStorage.getItem("access_token")
            }
          }
        )

        .then(({ data }) => {
          this.isLoading = false;
          this.tasks = data.tasks;
        })
        .catch(err => {
          this.isLoading = false;
          console.log(err);
        });
    },
    editTask(task) {
      this.$emit("editTask", task);
    },
    changeCategory(categoryId, taskId) {
      this.isLoading = true;
      axios
        .patch(
          `https://still-basin-93678.herokuapp.com/tasks/${taskId}`,
          { CategoryId: categoryId },
          {
            headers: {
              access_token: localStorage.getItem("access_token")
            }
          }
        )
        .then(response => {
          this.isLoading = false;
          let status = {
            title: "Category changed!",
            body: "Category successfully changed.",
            type: "success",
            canTimeout: true,
            duration: 2000
          };
          this.$vToastify.success(status);
          this.$emit("forceRerender");
        })
        .catch(err => {
          this.isLoading = false;
          let status = {
            title: "Failed!",
            body: err.response.data.errors,
            type: "error",
            canTimeout: true,
            duration: 2000
          };
          this.$vToastify.error(status);
        });
    },
    deleteTask(id) {
      this.isLoading = true;
      axios
        .delete(`https://still-basin-93678.herokuapp.com/tasks/${id}`, {
          headers: {
            access_token: localStorage.getItem("access_token")
          }
        })
        .then(response => {
          this.isLoading = false;
          let status = {
            title: "Task Deleted!",
            body: "Task successfully deleted.",
            type: "success",
            canTimeout: true,
            duration: 2000
          };
          this.$vToastify.success(status);
          this.$emit("forceRerender");
        })
        .catch(err => {
          this.isLoading = true;
          let status = {
            title: "Failed!",
            body: err.response,
            type: "error",
            canTimeout: true,
            duration: 2000
          };
          this.$vToastify.error(status);
        });
    }
  }
};
</script>

<style>
.button-arrow {
  cursor: pointer;
}
</style>