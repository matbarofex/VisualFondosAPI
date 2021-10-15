<template>
  <h1>Alta Rescate</h1>
  <div style="width:650px">
    <form
      class="g-3 needs-validation"
      v-on:submit.prevent="insertRescate"
      method="post"
      ref="formSol"
    >
      <div class="input-group mb-3">
        <span class="input-group-text">Cuotapartista</span>

        <select
          v-model="rescate.numCuotapartista"
          class="form-select"
          v-on:change="
            getPosicion($event);
            getCuentaBancaria($event);
          "
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
      <div class="mb-8" v-if="cuentaSeleccionada.length > 0">
        <table class="table">
          <thead>
            <tr>
              <th scope="col">Fondo</th>
              <th scope="col">Cuotapartes Totales</th>
              <th scope="col">Último VCP</th>
              <th scope="col">Posición</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="pos in this.cuentaSeleccionada"
              :key="pos.fondo.numFondo"
            >
              <td>
                Fondo {{ pos.fondo.numFondo }} -
                {{ pos.tipoValorCuotaparte.descripcion }}
              </td>
              <td>{{ pos.cuotapartesTotal.toLocaleString("es-AR") }}</td>
              <td>
                {{ pos.ultimoValorCuotaparte.toLocaleString("es-AR") }}
              </td>
              <td>
                {{ pos.moneda.idMoneda }}
                {{ pos.cuotapartesValuadas.toLocaleString("es-AR") }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="input-group mb-3">
        <span class="input-group-text">Fondo</span>
        <select
          class="form-select"
          required
          name="sFondo"
          ref="sFondo"
          v-on:change="setDataFondo($event)"
        >
          <option>Seleccione un fondo</option>
          <option
            v-for="fondo in cuentaSeleccionada"
            :key="
              fondo.fondo.numFondo +
                '_' +
                fondo.tipoValorCuotaparte.abreviatura +
                '_' +
                fondo.moneda.idMoneda +
                '_' +
                fondo.cuotapartesTotal
            "
            v-bind:value="
              fondo.fondo.numFondo +
                '_' +
                fondo.tipoValorCuotaparte.abreviatura +
                '_' +
                fondo.moneda.idMoneda +
                '_' +
                fondo.cuotapartesTotal
            "
            >Fondo {{ fondo.fondo.numFondo }} -
            {{ fondo.tipoValorCuotaparte.descripcion }}</option
          >
        </select>
      </div>

      <div class="input-group mb-3">
        <span class="input-group-text">Cuenta Bancaria</span>

        <select
          v-model="rescate.idCuentaBancariaCpt"
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
        <input
          type="radio"
          class="btn-check"
          name="group1"
          id="cParcial"
          autocomplete="off"
          checked
          v-on:change="setParcial()"
          ref="cParcial"
        />
        <label class="btn btn-outline-primary" for="cParcial">Parcial</label>

        <input
          type="radio"
          class="btn-check"
          name="group1"
          id="cTotal"
          autocomplete="off"
          v-on:change="setTotal()"
        />
        <label class="btn btn-outline-primary" for="cTotal">Total</label>
      </div>

      <div class="input-group mb-3" v-if="this.rescate.esTotal">
        <span class="input-group-text">CC</span>

        <input
          disabled
          class="form-control"
          id="validationCustom05"
          min="1"
          required
          v-if="this.rescate.esTotal"
          v-model="rescate.cantidadCuotapartes"
        />
      </div>

      <div class="input-group mb-3" v-if="this.rescate.esTotal == false">
        <span class="input-group-text">{{ moneda }}</span>

        <input
          type="number"
          class="form-control"
          id="validationCustom05"
          min="1"
          required
          v-model="rescate.importe"
        />
      </div>

      <div class="col-12">
        <button class="btn btn-primary" type="submit">Ingresar</button>
      </div>
    </form>
  </div>

  <!-- <div class="mb-8" v-if="cuentaSeleccionada.length > 0" style="width:850px">
    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Solicitud</th>
          <th scope="col">Fondo</th>
          <th scope="col">Fecha Concertacion</th>
          <th scope="col">Moneda</th>
          <th scope="col">Importe</th>
          <th scope="col">Cuotapartes</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="sol in this.solicitudes" :key="sol.idSolicitud">
          <th scope="row">{{ sol.numero }}</th>
          <td>{{ sol.tipo.descripcion }}</td>
          <td>
            Fondo {{ sol.fondo.numFondo }} -
            {{ sol.tipoValorCuotaparte.descripcion }}
          </td>
          <td>{{ format_date(sol.fechaConcertacion) }}</td>
          <td>{{ sol.moneda.simbolo }}</td>
          <td>{{ sol.importe.toLocaleString() }}</td>
          <td>
            {{ sol.cantidadCuotapartes }}
          </td>
        </tr>
      </tbody>
    </table>
  </div> -->
</template>

<script>
import axios from "axios";
import moment from "moment";
const apiURL = "https://api.sistemasesco.com/api/fondos/v3";

const login = async () => {
  const loginData = { userName: "sa", password: "sasa" };
  const headerPost = {
    "api-version": "3",
    "Content-Type":
      "application/json;odata.metadata=minimal;odata.streaming=true",
    accept: "*/*",
  };

  const response = await axios.post(apiURL + "/login", loginData, {
    headers: headerPost,
  });

  return response.data.access_token;
};

export default {
  name: "Rescate",
  data: function() {
    return {
      rescate: {
        numFondo: 0,
        numCuotapartista: 0,
        clase: "",
        fechaConcertacion: "",
        importe: 0,
        idSolicitud: "",
        idCuentaBancariaCpt: "",
        esRescateImporte: true,
        esTotal: false,
        cantidadCuotapartes: 0,
      },
      cuotapartesTotal: 0,
      cuentas: [],
      cuentasBancarias: [],
      cuentaSeleccionada: [],
      fondos: [],
      solicitudes: [{}],
      moneda: "",
    };
  },
  methods: {
    async getCuentas() {
      // Obtiene la lista de cuotapartistas disponibles en la base de datos
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
          "/reportes/posicionCuotapartista?fecha=" +
          this.getDate() +
          "&pageSize=150",
        { headers }
      );
      const data = await response.json();

      data.data.forEach((cuenta) => {
        this.cuentas.push(cuenta.cuotapartista);
      });
    },
    async getCuentaBancaria(cuotapartista) {
      // Consulta las cuentas bancarias del cuotapartista seleccionado
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

      console.log(data);

      data.data[0].cuentasBancarias.forEach((cuenta) => {
        this.cuentasBancarias.push([
          cuenta.idCptCuentaBancaria,
          cuenta.numeroCuenta,
          cuenta.moneda.simbolo,
        ]);
        console.log(this.cuentasBancarias);
      });
    },
    async getPosicion(event) {
      // Obtiene la posición del cuotapartista seleccionado

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
          "/reportes/posicionCuotapartista?fecha=" +
          this.getDate() +
          "&pageSize=50&numCuotapartista=" +
          event.target.value,
        { headers }
      );
      const data = await response.json();
      this.cuentaSeleccionada = data.data;
    },
    // async getSolicitudes(event) {
    //   const _token = await login();
    //   const headers = {
    //     "api-version": "3",
    //     "Content-Type":
    //       "application/json;odata.metadata=minimal;odata.streaming=true",
    //     accept: "application/json;odata.metadata=minimal;odata.streaming=true",
    //     Authorization: "Bearer " + _token,
    //   };

    //   axios
    //     .get(
    //       "https://api.sistemasesco.com/api/fondos/v3/reportes/solicitudes?fechaDesde=" +
    //         this.getDate() +
    //         "&fechaHasta=" +
    //         this.getDate() +
    //         "&pageSize=15&pageNumber=1&NumCuotapartista=" +
    //         event.target.value,
    //       { headers }
    //     )
    //     .then((response) => {
    //       this.solicitudes = response.data.data;
    //     })
    //     .catch((e) => {
    //       console.log(e);
    //     });
    // },
    async setDataFondo(event) {
      // Setea las variables de la operación al momento de seleccion del fondo
      const fondoClase = event.target.value.split("_");

      this.moneda = fondoClase[2];
      this.rescate.numFondo = fondoClase[0];
      this.rescate.clase = fondoClase[1];
      this.rescate.idSolicitud = String(Math.random()).replace("0.", "TEST");
      this.rescate.fechaConcertacion = this.getDate();
      this.cuotapartesTotal = fondoClase[3];
    },
    async setTotal() {
      // Setea si la solicitud es total
      this.rescate.cantidadCuotapartes = this.cuotapartesTotal;
      this.rescate.importe = 0;
      this.rescate.esTotal = true;
      this.rescate.esRescateImporte = false;
    },
    async setParcial() {
      // Setea si la solicitud es parcial
      this.rescate.cantidadCuotapartes = 0;
      this.rescate.importe = 0;
      this.rescate.esTotal = false;
      this.rescate.esRescateImporte = true;
    },
    insertRescate(event) {
      // Inserta la solicitud de rescate en la base de datos
      const headerPost = {
        "api-version": "3",
        "Content-Type":
          "application/json;odata.metadata=minimal;odata.streaming=true",
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
      };
      console.log(this.rescate);

      axios
        .post(apiURL + "/insert-solicitud-rescate", this.rescate, {
          headers: headerPost,
        })
        .then((res) => {
          console.log(res);
          alert(
            "La solicitud " + res.data.codigo + " se ingresó satisfactoriamente"
          );
          this.$refs.formSol.reset();
          event.target.reset();
          this.sFondo.selected = undefined;
          this.sCuotapartista.selected = undefined;
          this.sCuentaBancaria.selected = undefined;
          this.setParcial();
          this.cuentaSeleccionada = [];
          this.$refs.cParcial.checked = true;
          this.rescate = [];
        })
        .catch((error) => {
          alert(error.response.data.error.Msj);
        });
    },
    format_date(value) {
      if (value) {
        return moment(String(value)).format("DD/MM/YYYY");
      }
    },
    getDate() {
      return moment().format("YYYY-MM-DD");
    },
  },
  created() {
    this.getCuentas();
  },
};
</script>
