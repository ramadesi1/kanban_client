<template>
  <div>
    <form @submit.prevent="formEditTask">
      <div class="form-group">
        <label>Title</label>
        <input
          type="text"
          class="form-control"
          placeholder="Enter title"
          v-model="title"
        />
      </div>
      <div class="form-group">
        <label>Description</label>
        <input
          type="text"
          class="form-control"
          placeholder="Enter description"
          v-model="description"
        />
      </div>
      <button type="submit" class="btn btn-primary">Edit Task</button>
    </form>
    <loading :active.sync="isLoading" :can-cancel="false" :is-full-page="fullPage"></loading>
  </div>
</template>

<script>
import axios from "axios";
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";
export default {
  name: "AddTask",
  props: ["task"],
  data: function() {
    return {
      title: "",
      description: "",
      isLoading: false,
      fullPage: true
    };
  },
  created: function() {
    this.title = this.task.title;
    this.description = this.task.description;
  },
  components: {
    Loading
  },
  methods: {
    formEditTask: function() {
      this.isLoading = true;
      let data = {
        title: this.title,
        description: this.description,
        ProjectId: this.task.ProjectId,
        CategoryId: this.task.CategoryId
      };
      axios
        .put(`https://still-basin-93678.herokuapp.com/tasks/${this.task.id}`, data, {
          headers: {
            access_token: localStorage.getItem("access_token")
          }
        })
        .then(result => {
          this.isLoading = false;
          let status = {
            title: "Task Updated!",
            body: "Task successfully updated.",
            type: "success",
            canTimeout: true,
            duration: 2000
          };
          this.$vToastify.success(status);
          this.$emit("changePage", "category");
        })
        .catch(err => {
          this.isLoading = false;
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
</style>