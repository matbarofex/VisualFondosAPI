<template>
  <h1>Alta Suscripción</h1>
  <div style="width:550px">
    <form
      class="g-3 needs-validation"
      v-on:submit.prevent="insertSuscripcion"
      method="post"
      ref="formSol"
    >
      <div class="input-group mb-3">
        <span class="input-group-text">Fondo</span>
        <select
          class="form-select"
          required
          v-on:change="getMoneda($event)"
          name="sFondo"
          ref="sFondo"
        >
          <option>Seleccione un fondo</option>
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
        <span class="input-group-text">Cuotapartista</span>

        <select
          v-model="suscripcion.numCuotapartista"
          class="form-select"
          v-on:change="getCuentaBancaria($event)"
          required
          name="sCuotapartista"
          ref="sCuotapartista"
        >
          <option value="0" selected="selected"
            >Seleccione una cuenta cuotapartista</option
          >
          <option
            v-for="cuenta in cuentas"
            :key="cuenta.numero"
            :value="cuenta.numero"
            >Cuenta {{ cuenta.numero }}</option
          >
        </select>
      </div>
      <div class="input-group mb-3">
        <span class="input-group-text">Cuenta Bancaria</span>

        <select
          v-model="suscripcion.idCuentaBancariaCpt"
          class="form-select"
          required
          name="sCuentaBancaria"
          ref="sCuentaBancaria"
        >
          <option value="0" selected="selected"
            >Seleccione una cuenta bancaria</option
          >
          <option
            v-for="cuenta in cuentasBancarias"
            :key="cuenta[0]"
            :value="cuenta[0]"
            >{{ cuenta[2] }} {{ cuenta[1] }}</option
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
          v-model="suscripcion.importe"
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
import moment from "moment";

const login = async () => {
  const loginData = { userName: "sa", password: "sasa" };
  const headerPost = {
    "api-version": "3",
    "Content-Type":
      "application/json;odata.metadata=minimal;odata.streaming=true",
    accept: "*/*",
  };

  const response = await axios.post(
    "https://api.sistemasesco.com/api/fondos/v3/login",
    loginData,
    {
      headers: headerPost,
    }
  );

  return response.data.access_token;
};

const apiURL = "https://api.sistemasesco.com/api/fondos/v3";

export default {
  name: "Suscripcion",
  data: function() {
    return {
      suscripcion: {
        numFondo: 0,
        numCuotapartista: 0,
        clase: "",
        fechaConcertacion: "",
        importe: 0,
        idSolicitud: "",
        idCuentaBancariaCpt: "",
      },
      cuentas: [],
      cuentasBancarias: [],
      fondos: [],
      moneda: "",
    };
  },
  methods: {
    async getCuentas() {
      const _token = await login();
      const headers = {
        "api-version": "3",
        "Content-Type":
          "application/json;odata.metadata=minimal;odata.streaming=true",
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
        Authorization: "Bearer " + _token,
      };

      const response = await fetch(
        apiURL +
          "/reportes/posicionCuotapartista?fecha=2021-10-15&pageSize=150",
        { headers }
      );
      const data = await response.json();

      data.data.forEach((cuenta) => {
        this.cuentas.push(cuenta.cuotapartista);
      });
    },
    async getCuentaBancaria(cuotapartista) {
      const _token = await login();
      const headers = {
        "api-version": "3",
        "Content-Type":
          "application/json;odata.metadata=minimal;odata.streaming=true",
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
        Authorization: "Bearer " + _token,
      };

      this.cuentasBancarias = [];
      const response = await fetch(
        apiURL +
          "/get-cuotapartistas?numCuotapartista=" +
          cuotapartista.target.value,
        { headers }
      );
      const data = await response.json();
      // this.cuentasBancarias = data.data[0].cuentasBancarias;
      data.data[0].cuentasBancarias.forEach((cuenta) => {
        this.cuentasBancarias.push([
          cuenta.idCptCuentaBancaria,
          cuenta.numeroCuenta,
          cuenta.moneda.simbolo,
        ]);
      });
    },
    async getFondos() {
      const _token = await login();
      const headers = {
        "api-version": "3",
        "Content-Type":
          "application/json;odata.metadata=minimal;odata.streaming=true",
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
        Authorization: "Bearer " + _token,
      };

      const response = await fetch(
        apiURL + "/reportes/valorCuotapartes?fecha=2015-10-10&pageSize=50",
        { headers }
      );
      const data = await response.json();

      data.data.forEach((fondo) => {
        this.fondos.push([
          fondo.fondo,
          fondo.tipoValorCuotaparte,
          fondo.moneda,
        ]);
      });
      // console.log(this.$refs.sCuentaBancaria);
      this.$refs.sFondo.selected = 0;
      this.$refs.sCuotapartista.selected = 0;
      this.$refs.sCuentaBancaria.selected = 0;
    },
    async getMoneda(event) {
      const fondoClase = event.target.value.split("_");

      this.moneda = fondoClase[2];
      this.suscripcion.numFondo = fondoClase[0];
      this.suscripcion.clase = fondoClase[1];
      this.suscripcion.idSolicitud = String(Math.random()).replace(
        "0.",
        "TEST"
      );
      this.suscripcion.fechaConcertacion = this.getDate();
    },
    insertSuscripcion(event) {
      const headerPost = {
        "api-version": "3",
        "Content-Type":
          "application/json;odata.metadata=minimal;odata.streaming=true",
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
      };
      // console.log(this.suscripcion);

      axios
        .post(apiURL + "/insert-solicitud-suscripcion", this.suscripcion, {
          headers: headerPost,
        })
        .then((res) => {
          alert(
            "La solicitud " + res.data.codigo + " se ingresó satisfactoriamente"
          );
          this.$refs.formSol.reset();
          event.target.reset();
          this.sFondo.selected = undefined;
          this.sCuotapartista.selected = undefined;
          this.sCuentaBancaria.selected = undefined;
        })
        .catch((error) => {
          alert(error.response.data.error.Msj);
          // console.log(error.response.data.error);
        });
    },
    getDate() {
      return moment().format("YYYY-MM-DD");
    },
  },
  created() {
    this.getCuentas();
    this.getFondos();
  },
};
</script>
