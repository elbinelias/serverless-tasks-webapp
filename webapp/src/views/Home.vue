<template>
  <main role="main">
    <div class="container bg-light p-3 mt-4 mb-1">
      <div v-if="error != ''" class="alert alert-danger" role="alert">
        <span class="mr-2">
          <!-- Source: https://icons.getbootstrap.com/ License: https://github.com/twbs/icons/blob/main/LICENSE.md -->
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            fill="currentColor"
            class="bi bi-x-circle"
            viewBox="0 0 16 16"
          >
            <path
              d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z"
            />
            <path
              d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708z"
            /></svg></span
        >{{ error }}
      </div>
      <form @submit.prevent>
        <div class="form-group mb-3">
          <label for="taskTitleInput" class="form-label">Enter your payment reference</label>
          <input
            v-model="title"
            type="text"
            class="form-control"
            id="taskTitleInput"
            placeholder="Payment Reference"
            required
          />
        </div>
        <div class="form-group mb-3" style="display: none;">
          <label for="taskBodyInput" class="form-label">Body</label>
          <textarea
            v-model="body"
            class="form-control"
            id="taskBodyInput"
            placeholder="Body"
            rows="3"
            required
          />
        </div>
        <div class="form-group mb-3" style="display: none;">
          <label for="taskBodyInput" class="form-label">Due date</label>
          <div class="input-group">
            <input
              id="datepickerduedate"
              name="datepickerduedate"
              type="date"
              v-model="dueDate"
            />
          </div>
        </div>
        <div class="form-group">
          <button v-on:click="createTask" class="btn btn-primary">
            Search
          </button>
        </div>
        <div class="form-group">
          <label>Or upload a file:&nbsp;&nbsp;</label>
          <br>
          <button class="btn btn-outline-secondary float-left">
            Upload file
          </button>
        </div>
      </form>
    </div>

    <div class="container bg-light p-3">
      <h2>Payments List</h2>
      <div v-if="loading">
        <p>Loading...</p>
      </div>
      <div v-else>
        <div v-for="item in tasks" :key="item.id">
          <div class="card w-100">
            <div class="card-body">
              <button
                @click="deleteTask(item.id)"
                class="btn btn-outline-secondary float-right"
              >
                Delete
              </button>
              <h5 class="card-title">{{ item.title }}</h5>
              <label class="form-label"><b>End-to-End ID:</b>&nbsp;{{ item.endtoendID }}</label> &nbsp;&nbsp; <label class="form-label"><b>System Reference 1:</b>&nbsp;{{ item.gcpID}}</label> &nbsp;&nbsp; <label class="form-label" id="showStatus"><b>Status:</b>&nbsp;{{ item.statuschange}}</label>
              <br>
              <label class="form-label"><b>Additional Info:</b>&nbsp;{{ item.remittanceInfo}}</label>
              <br>
              <label><b>Client Reference:</b>&nbsp;{{ item.clientref}}</label> &nbsp;&nbsp; <label><b>Amount:</b>&nbsp;{{ item.amt}}</label>
              <br>
              <label><b>Transaction ID:</b>&nbsp;{{ item.transactionid}}</label> &nbsp;&nbsp; <label><b>System Reference 2:</b>&nbsp;{{ item.bifastid}}</label>
              <br>
              <small class="text-secondary"
                >Searched on: {{ item.createdAt }}</small
              >
              <small class="text-secondary" v-if="item.dueDate" style="display: none;"
                >&nbsp;/&nbsp;Due: {{ item.dueDate }}</small
              >
              <p class="card-text" style="display: none;">
                {{ item.body }}
              </p>
              <div class="progress" v-if="item.progress">
                <div
                  class="
                    progress-bar progress-bar-striped progress-bar-animated
                  "
                  role="progressbar"
                  style="width: 100%"
                >
                  {{ item.progress }}
                </div>
              </div>
              <div class="text-secondary mb-2">
                <button @click="getTaskById(item.id)" class="btn btn-outline-secondary float-left">
                  Refresh
                </button>
               <div class="text-secondary mb-2" v-if="item.upload" href="#" style="display: none;">
                1 attachment
              </div>
              <div v-else class="mb-2" style="display: none;">
                Attachment:
                <input type="file" @change="fileChanged" ref="file" />
                <button
                  class="btn btn-outline-secondary btn-sm"
                  @click="submitFile(item)"
                >
                  Upload
                </button>
              </div>
              
              <small v-if="item.labels" class="text-secondary"
                >Detected labels:</small
              >
              <div v-for="label in item.labels" :key="label.id">
                <span class="badge badge-info float-left mr-1 mb-1">{{
                  label
                }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
<script>
import Vue from "vue";
import axios from "axios";
import { getAuthToken } from "../utils/auth";
import { sleep } from "../utils/core";

export default {
  name: "home",
  data() {
    return {
      error: "",
      loading: true,
      title: null,
      body: null,
      dueDate: null,
      tasks: [],
      attachment: null,
      uploadUrl: null,
    };
  },
  mounted() {
    this.getTasks();
  },
  methods: {
    async createTask() {
      this.error = "";
      let token = getAuthToken();
      console.log(`Creating task`);
      await axios
        .post(
          "/tasks",
          {
            title: this.title,
            body: this.body,
            dueDate: this.dueDate,
          },
          { headers: { Authorization: `Bearer ${token}` } }
        )
        .then((res) => {
          console.log(JSON.stringify(res.data, null, 2));
        })
        .catch((err) => {
          console.log(err);
          this.error = `Error creating task: ${err.message}`;
        });
      await this.getTasks();
    },
    async deleteTask(id) {
      id = id.split("#")[1];
      console.log(`Deleting task ${id}`);
      this.error = "";
      let token = getAuthToken();
      await axios
        .delete(`/tasks/${id}`, {
          headers: { Authorization: `Bearer ${token}` },
        })
        .then(() => {
          console.log(`Deleted task ${id}`);
        })
        .catch((err) => {
          console.log(err);
          this.error = `Error deleting task: ${err.message}`;
        });
      await this.getTasks();
    },
    async getTasks() {
      console.log("Getting tasks");
      this.error = "";
      let token = getAuthToken();
      this.loading = true;
      await axios
        .get("/tasks", { headers: { Authorization: `Bearer ${token}` } })
        .then((res) => {
          console.log(JSON.stringify(res.data, null, 2));
          this.tasks = res.data;
          // sort descending by date
          this.tasks = this.tasks.sort((a, b) =>
            a.createdAt < b.createdAt ? 1 : -1
          );
        })
        .catch((err) => {
          console.log(err);
          this.error = `Error getting tasks: ${err.message}`;
        });
      this.loading = false;
    },
    async getTaskById(id) {
      id = id.split("#")[1];
      console.log(`Getting task ${id}`);
      this.error = "";
      let token = getAuthToken();
      await axios
        .get(`/tasks/${id}`, {headers: { Authorization: `Bearer ${token}` },
        })
        .then(() => {
          console.log(`Got task ${id}`);
        })
        .catch((err) => {
          console.log(err);
          this.error = `Error getting task: ${err.message}`;
        });
        await this.getTasks();
    },
    fileChanged(e) {
      var files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;
      console.log(files[0]);
      this.attachment = files[0];
    },
    async getSignedUrl(taskId) {
      console.log("Getting signed URL");

      let token = getAuthToken();
      const config = {
        headers: { Authorization: `Bearer ${token}` },
        params: {
          filename: this.attachment.name,
          filetype: this.attachment.type,
          taskId: taskId,
        },
      };

      const response = await axios.get("/signedUrl", config);
      return new URL(response.data.uploadURL);
    },
    async submitFile(taskItem) {
      if (!this.attachment) {
        alert("Please select a file to upload.");
      }
      const taskId = taskItem.id.split("#")[1];
      const uploadUrl = await this.getSignedUrl(taskId);

      const config = {
        headers: { "Content-Type": this.attachment.type },
      };

      console.log(`config: ${JSON.stringify(config)}`);

      console.log(`Uploading to S3: ${uploadUrl}`);

      var instance = axios.create();
      delete instance.defaults.headers.common['Authorization'];

      const res = await instance.put(uploadUrl, this.attachment, config);

      console.log(res.status); // HTTP status
      Vue.set(taskItem, "progress", "Detecting labels...");
      await sleep(5000); // wait for Lambda to trigger and Rekognition to process
      await this.getTasks();
    },
  },
};
</script>
