<template>
  <div class="d-flex justify-content-end mx-10">
    <button
      class="btn btn-primary btn-sm rounded-pill text-white my-3 mx-3 float-end"
      @click="openCreatePopup()"
    >
      Create New Application
    </button>
  </div>
  <div class="popup" v-if="showCreatePopup">
    <div class="card">
      <div class="card-header">
        <h5 class="card-title">Create New Application</h5>
      </div>
      <div class="card-body">
        <div class="form-group row my-3">
          <label class="col-sm-2 col-form-label" for="field1">Name</label>
          <div class="col-sm-10">
            <input
              type="text"
              class="form-control"
              id="field1"
              v-model="field1Value"
            />
          </div>
        </div>
        <div class="form-group row my-3">
          <label class="col-sm-2 col-form-label" for="field1">Email</label>
          <div class="col-sm-10">
            <input
              type="text"
              class="form-control"
              id="field2"
              v-model="field2Value"
            />
          </div>
        </div>
        <div class="form-group row my-3">
          <label class="col-sm-2 col-form-label" for="field1">Status</label>
          <div class="col-sm-10">
            <select class="form-control" id="field3" v-model="field3Value">
              <option value="" disabled>Select an option</option>
              <option
                v-for="(option, index) in dropdownOptions"
                :key="index"
                :value="option.value"
              >
                {{ option.label }}
              </option>
            </select>
          </div>
        </div>
      </div>
      <div class="card-footer">
        <button type="button" class="btn btn-secondary mx-3" @click="cancel">
          Cancel
        </button>
        <button
          type="button"
          class="btn btn-primary"
          @click="createApplication"
        >
          Create
        </button>
      </div>
    </div>
  </div>

  <table>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Status</th>
      <th>Actions</th>
    </tr>
    <tr v-for="user in list" :key="user.id">
      <td>
        {{ user.fullName }}
      </td>
      <td>
        {{ user.email }}
      </td>
      <td>
        {{ user.status }}
      </td>
      <td>
        <button class="btn btn-danger mx-2" @click="deleteData(user.id)">
          Delete
        </button>
        <button class="btn btn-primary" @click="openEditPopup(user)">
          Edit
        </button>
        <!-- <EditPopup id={{ user.id }} /> -->
        <div class="popup" v-if="showPopup">
          <div class="card">
            <div class="card-header">
              <h5 class="card-title">Edit Application</h5>
            </div>
            <div class="card-body">
              <div class="form-group row my-3">
                <label class="col-sm-2 col-form-label" for="field1">Name</label>
                <div class="col-sm-10">
                  <input
                    type="text"
                    class="form-control"
                    id="field1"
                    v-model="field1Value"
                  />
                </div>
              </div>
              <div class="form-group row my-3">
                <label class="col-sm-2 col-form-label" for="field2"
                  >Email</label
                >
                <div class="col-sm-10">
                  <input
                    type="text"
                    class="form-control"
                    id="field2"
                    v-model="field2Value"
                  />
                </div>
              </div>
              <div class="form-group row my-3">
                <label class="col-sm-2 col-form-label" for="field3"
                  >Status</label
                >
                <div class="col-sm-10">
                  <select
                    class="form-control"
                    id="field3"
                    v-model="field3Value"
                  >
                    <option value="" disabled>Select an option</option>
                    <option
                      v-for="(option, index) in dropdownOptions"
                      :key="index"
                      :value="option.value"
                    >
                      {{ option.label }}
                    </option>
                  </select>
                </div>
              </div>
            </div>
            <div class="card-footer">
              <button
                type="button"
                class="btn btn-secondary mx-3"
                @click="cancel"
              >
                Cancel
              </button>
              <button type="button" class="btn btn-primary" @click="save">
                Save
              </button>
            </div>
          </div>
        </div>
      </td>
    </tr>
  </table>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      list: [],
      showPopup: false,
      showCreatePopup: false,
      appId: "",
      field1Value: "",
      field2Value: "",
      field3Value: "",
      dropdownOptions: [
        { label: "APPLIED", value: "APPLIED" },
        { label: "INTERVIEW", value: "INTERVIEW" },
        { label: "REJECTED", value: "REJECTED" },
        { label: "OFFER SENT", value: "OFFER_SENT" },
        { label: "HIRED", value: "HIRED" },
      ],
    };
  },
  async mounted() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      axios
        .get("http://localhost:9090/api/candidates/")
        .then((response) => {
          this.list = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    deleteData(dataId) {
      axios
        .delete(`http://localhost:9090/api/candidates/${dataId}`)
        .then(() => {
          this.fetchData(); // call fetchData method to refresh data
        })
        .catch((error) => {
          console.log(error);
        });
    },
    createApplication() {
      axios
        .post(`http://localhost:9090/api/candidates`, this.getRequestBody())
        .then(() => {
          this.fetchData();
          this.cancel();
        })
        .catch((error) => {
          // Handle error
          console.error("Error making POST request:", error);
        });
    },
    updateApplication() {
      axios
        .put(
          `http://localhost:9090/api/candidates/${this.appId}`,
          this.getRequestBody()
        )
        .then(() => {
          this.fetchData();
        })
        .catch((error) => {
          console.error("Error making PUT request:", error);
        });
    },
    emptyValues() {
      this.field1Value = "";
      this.field2Value = "";
      this.field3Value = "";
    },

    //handling save button events
    save() {
      this.updateApplication();
      this.cancel();
    },
    //handling cancel button events
    cancel() {
      this.showCreatePopup = false;
      this.showPopup = false;
      this.emptyValues();
    },
    togglePopup() {
      this.showPopup = !this.showPopup;
    },
    toggleCreatePopup() {
      this.showCreatePopup = !this.showCreatePopup;
    },
    openCreatePopup() {
      this.toggleCreatePopup();
    },
    getRequestBody() {
      return {
        fullName: this.field1Value,
        email: this.field2Value,
        status: this.field3Value,
      };
    },
    openEditPopup(data) {
      console.log(data);
      console.log(this.field1Value);
      this.appId = data.id;
      this.field1Value = data.fullName;
      this.field2Value = data.email;
      this.field3Value = data.status;
      this.togglePopup();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.popup {
  width: 75vw;
  height: 30vh;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
}
table {
  width: 100vw;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
