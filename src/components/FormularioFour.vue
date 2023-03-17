<template>
  <div class="container">
    <h1>Prueba Tecnica</h1>
    <div v-if="!timeUp">
      <div v-if="showForm">
        <div class="form-row">
          <div class="form-group col-md-6">
            <label>Nombre Completo</label>
            <input type="text" class="form-control" v-model="name" />
            <div
              class="input-errors"
              v-for="error of v$.name.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
          <div class="form-group col-md-6">
            <label for="birthDate">Fecha Nacimiento</label>
            <input
              type="date"
              class="form-control"
              v-model="birthDate"
              placeholder="Fecha Nacimiento"
            />
            <div
              class="input-errors"
              v-for="error of v$.birthDate.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
        </div>
        <div class="form-row">
          <div class="form-group col-md-4">
            <label for="countries">Pais</label>
            <b-form-select
              v-model="country"
              :options="countries"
              value-field="country_name"
              text-field="country_name"
              @change="getStates($event)"
            ></b-form-select>
            <div
              class="input-errors"
              v-for="error of v$.country.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
          <div class="form-group col-md-4">
            <label for="estado">Estado</label>
            <b-form-select
              :options="states"
              v-model="state"
              :disabled="!states.length"
              value-field="state_name"
              text-field="state_name"
              @change="getCities($event)"
            ></b-form-select>
            <div
              class="input-errors"
              v-for="error of v$.state.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
          <div class="form-group col-md-4">
            <label for="Ciudad">Ciudad</label>
            <b-form-select
              :options="cities"
              v-model="city"
              :disabled="!cities.length"
              value-field="city_name"
              text-field="city_name"
            ></b-form-select>
            <div
              class="input-errors"
              v-for="error of v$.city.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
        </div>
        <div class="form-row">
          <div class="form-group col-md-6">
            <label for="email">Email</label>
            <input type="email" class="form-control" v-model="email" />
            <div
              class="input-errors"
              v-for="error of v$.email.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
          <div class="form-group col-md-6">
            <label for="temperatura">Temperatura</label>
            <input
              type="text"
              class="form-control"
              v-bind:value="temperature + ' Grados'"
              disabled
            />
            <div
              class="input-errors"
              v-for="error of v$.temperatura.$errors"
              :key="error.$uid"
            >
              <div class="error-msg">{{ error.$message }}</div>
            </div>
          </div>
        </div>

        <button class="btn btn-primary" @click="submit">Enviar</button>
        <p v-if="countdown > 0">Tiempo restante: {{ minutes }}:{{ seconds }}</p>
        <p v-else>¡Se acabó el tiempo!</p>
      </div>
      <div v-if="showUserData">
        <div>
          <table class="my-table">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Fecha Nacimiento</th>
                <th>Pais</th>
                <th>Estado</th>
                <th>ciudad</th>
                <th>Correo</th>
                <th>Temperatura</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>{{ name }}</td>
                <td>{{ birthDate }}</td>
                <td>{{ country }}</td>
                <td>{{ state }}</td>
                <td>{{ city }}</td>
                <td>{{ email }}</td>
                <td>{{ temperature }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div v-else>
      <p>Tiempo agotado.</p>
    </div>
    <b-modal v-model="showModal" title="Modal de tiempo agotado">
      <p>El tiempo ha terminado. Debes salir del formulario.</p>
    </b-modal>
  </div>
</template>


<script>
import { useVuelidate } from "@vuelidate/core";
import { required, email, helpers } from "@vuelidate/validators";
import axios from "axios";
import "./FormularioFour.css";

export default {
  name: "FormularioFour",
  data: () => ({
    name: null,
    countries: [],
    cities: [],
    states: [],
    temperature: null,
    birthDate: null,
    email: null,
    country: null,
    state: null,
    city: null,
    userLocation: null,
    countdown: 120000,
    timeUp: false,
    showModal: false,
    showForm: true,
    showUserData: false,
    isValidForm: false,
    startInterval: ''
    
  }),
  setup: () => ({ v$: useVuelidate() }),
  validations() {
    const today = new Date();
    const maxDate = new Date(
      today.getFullYear() - 18,
      today.getMonth(),
      today.getDate()
    );
    const minDate = new Date(
      today.getFullYear() - 63,
      today.getMonth(),
      today.getDate()
    );

    return {
      name: {
        required: helpers.withMessage("Este Campo es Requerido", required),
      },
      birthDate: {
        required: helpers.withMessage("Este Campo es Requerido", required),
        minBirthDate: helpers.withMessage(
          "Debes tener menos de 63 años",
          (value) => new Date(value) > minDate
        ),
        maxBirthDate: helpers.withMessage(
          "Debes ser mayor de edad (+18)",
          (value) => new Date(value) < maxDate
        ),
      },
      country: {
        required: helpers.withMessage("Este Campo es Requerido", required),
      },
      state: {
        required: helpers.withMessage("Este Campo es Requerido", required),
      },
      city: {
        required: helpers.withMessage("Este Campo es Requerido", required),
      },
      email: {
        required: helpers.withMessage("Este Campo es Requerido", required),
        email: helpers.withMessage(
          "No es una direccion de correo valida",
          email
        ),
      },
      temperatura: {},
    };
  },
  computed: {
    minutes() {
      return Math.floor(this.countdown / 60000);
    },
    seconds() {
      return Math.floor((this.countdown % 60000) / 1000)
        .toString()
        .padStart(2, "0");
    },
  },
  methods: {
    async submit() {
      this.isValidForm = await this.v$.$validate();

      if (this.isValidForm) {
        this.showForm = false;
        this.showUserData = true;
        this.showUserData = true;
     clearInterval(    this.startInterval);
      }
    },
    async getAccessToken() {
      const data = await axios.get(
        "https://www.universal-tutorial.com/api/getaccesstoken",
        {
          headers: {
            Accept: "application/json",
            "api-token":
              "odVJUpz-eeb-oW7oAgiszFU82GIfKWa4FgDzMBCZgwDe5YwzEiGrWNghq6vECS-2vi0",
            "user-email": "hdgomez0@gmail.com",
          },
        }
      );
      const { auth_token } = data.data;
      return auth_token;
    },
    async getCountries() {
      const token = await this.getAccessToken();
      const data = await axios.get(
        "https://www.universal-tutorial.com/api/countries/",
        {
          headers: {
            Authorization: `Bearer ${token}`,
            Accept: "application/json",
          },
        }
      );
      this.countries = data.data;
    },
    async getStates() {
      const token = await this.getAccessToken();
      const data = await axios.get(
        "https://www.universal-tutorial.com/api/states/" + this.country,
        {
          headers: {
            Authorization: `Bearer ${token}`,
            Accept: "application/json",
          },
        }
      );
      this.states = data.data;
    },
    async getCities() {
      const token = await this.getAccessToken();
      const data = await axios.get(
        "https://www.universal-tutorial.com/api/cities/" + this.state,
        {
          headers: {
            Authorization: `Bearer ${token}`,
            Accept: "application/json",
          },
        }
      );
      this.cities = data.data;
    },
    getUserLocation() {},
    async getTemperature() {
      navigator.geolocation.getCurrentPosition(async (position) => {
        this.userLocation = position.coords;
        console.log(this.userLocation);

        const data = await axios.get(
          `http://api.weatherunlocked.com/api/current/${this.userLocation.latitude},${this.userLocation.longitude}?app_id=bc2eb085&app_key=8641f2df4f70968320f29a138ebf59be`,
          {
            headers: {
              Accept: "application/json",
            },
          }
        );

        this.temperature = data.data?.temp_c;
      });
    },
    startCountdown() {
      const endTime = new Date().getTime() + this.countdown;
       this.startInterval = setInterval(() => {
        const remainingTime = endTime - new Date().getTime();

        if (remainingTime <= 0) {
          clearInterval(    this.startInterval);
          this.countdown = 0;
          this.formSubmitted = true;
          this.timeUp = true;
          this.showModal = true;
          return;
        }

        this.countdown = remainingTime;
      }, 1000);
    },
  },
  async mounted() {
    this.startCountdown();
    this.getUserLocation();
    this.getCountries();
    this.getTemperature();
  },
};
</script>