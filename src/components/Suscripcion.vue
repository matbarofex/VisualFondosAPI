<template>
  <h1>Alta Suscripción</h1>
  <div style="width:550px">
    <form
      class="g-3 needs-validation"
      v-on:submit.prevent="insertSuscripcion"
      method="post"
    >
      <div class="input-group mb-3">
        <span class="input-group-text">Cuotapartista</span>

        <select v-model="cuentaSeleccionada" class="form-select" required>
          <option
            v-for="cuenta in cuentas"
            :key="cuenta.numero"
            :value="cuenta.numero"
            >Cuenta {{ cuenta.numero }}</option
          >
        </select>
      </div>
      <div class="input-group mb-3">
        <span class="input-group-text">Fondo</span>
        <select
          v-model="fondoSeleccionado"
          class="form-select"
          required
          v-on:change="getMoneda($event)"
        >
          <option
            v-for="fondo in fondos"
            :key="
              fondo[0].numFondo +
                '_' +
                fondo[1].abreviatura +
                '_' +
                fondo[2].cafci
            "
            v-bind:value="
              fondo[0].numFondo +
                '_' +
                fondo[1].abreviatura +
                '_' +
                fondo[2].cafci
            "
            >Fondo {{ fondo[0].numFondo }} - {{ fondo[1].descripcion }}</option
          >
        </select>
      </div>

      <div class="input-group mb-3">
        <span class="input-group-text">{{ moneda }}</span>

        <input
          type="number"
          class="form-control"
          id="validationCustom05"
          min="1"
          required
        />
      </div>

      <div class="col-12">
        <button class="btn btn-primary" type="submit">Ingresar</button>
      </div>
    </form>
  </div>
</template>

<script>
import axios from "axios";

const headers = {
  accept: "application/json;odata.metadata=minimal;odata.streaming=true",
  "Content-Type": "application/json;",
  "api-version": "2",
};

const apiURL = "http://192.168.22.14:6003/api/fondos/v3";
const apiURLv2 = "https://api.sistemasesco.com/api/fondos/v2";

const login = { userName: "sa", password: "sasa" };

export default {
  name: "Suscripcion",
  data: function() {
    return {
      suscripcion: {
        
      },
      cuentas: [],
      fondos: [],
      cuentaSeleccionada: 0,
      fondoSeleccionado: 0,
      moneda: "",
    };
  },
  methods: {
    async getCuentas() {
      const response = await fetch(
        apiURLv2 +
          "/reportes/posicionCuotapartista?fecha=2021-10-15&pageSize=150",
        { headers }
      );
      const data = await response.json();

      data.data.forEach((cuenta) => {
        this.cuentas.push(cuenta.cuotapartista);
      });

      // console.log(this.cuentas);
    },
    async getFondos() {
      const response = await fetch(
        apiURLv2 + "/reportes/valorCuotapartes?fecha=2015-10-10&pageSize=50",
        { headers }
      );
      const data = await response.json();

      data.data.forEach((fondo) => {
        const fondoClase = [];
        fondoClase;
        this.fondos.push([
          fondo.fondo,
          fondo.tipoValorCuotaparte,
          fondo.moneda,
        ]);
      });

      // console.log(this.fondos);
    },
    async getMoneda(event) {
      const fondoClase = event.target.value.split("_");

      console.log(fondoClase);
      this.moneda = fondoClase[2];
    },
    insertSuscripcion() {
      axios
        .post(apiURL + "/insert-solicitud-suscripcion", this.form)
        .then((res) => {
          //Perform Success Action
        })
        .catch((error) => {
          // error.response.status Check status code
        })
        .finally(() => {
          //Perform action in always
        });
    },
  },
  created() {
    this.getFondos();
    this.getCuentas();
  },
};
</script>
