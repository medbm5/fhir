<template>
  <!-- <h1>{{ msg }}</h1> -->
  <div class="container">
    <div style="text-align: center">
      <h1>Smart on FHIR - Patient Info</h1>
      <p><strong>Username:</strong> fhircamila</p>
      <p><strong>Password:</strong> epicepic1</p>
      <a
        class="btn btn-info"
        v-if="code == undefined"
        style="text-decoration: none"
        target="_blank"
        :href="authorizeLink"
      >
        Sign in
      </a>
      <hr />
    </div>
    
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      code: "",
      accesstoken: "",
      patient: "",
      patientdata: {},
      clientId: "", // Replace with your client id
      redirect: "http://localhost:3000",
    };
  },
  computed: {
    authorizeLink() {
      return `https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.clientId}&state=1234&scope=patient.read,patient.search,list.read`;
    },
  },
  async mounted() {
    console.log(this.$route.query.code);
    this.code = this.$route.query.code;
    if (this.code != undefined) {
      const params = new URLSearchParams();
      params.append("grant_type", "authorization_code");
      params.append("redirect_uri", this.redirect);
      params.append("code", this.code);
      params.append("client_id", this.clientId);
      params.append("state", "1234");
      const config = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      };
      await axios
        .post(
          "https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token",
          params,
          config
        )
        .then((response) => {
          console.log(response.data);
          this.accesstoken = response.data.access_token;
          this.patient = response.data.patient;
        })
        .catch(function (error) {
          console.log(error);
        });
    }

    if (this.accesstoken != "") {
      console.log("here")
      await axios
        .get(
          `https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/Patient/erXuFYUfucBZaryVksYEcMg3`,
          { headers: { Authorization: `Bearer ${this.accesstoken}` } }
        )
        .then((response) => {
          this.patientdata = response.data;
          console.log("allo",response);
        })
        .catch((error) => {
          console.log("allo",error);
        });
    }
  },
};
// const state = reactive({ count: 0 })
</script>

<style scoped>
/* a {
  color: #42b983;
} */
</style>
