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
          v-on:change="getMoneda($event)"
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
          v-on:change="setParcial($event)"
        />
        <label class="btn btn-outline-primary" for="cParcial">Parcial</label>

        <input
          type="radio"
          class="btn-check"
          name="group1"
          id="cTotal"
          autocomplete="off"
          v-on:change="setTotal($event)"
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
</template>

<script>
import axios from "axios";

const headersv2 = {
  accept: "application/json;odata.metadata=minimal;odata.streaming=true",
  "Content-Type": "application/json;",
  "api-version": "2",
};

const headers = {
  accept: "application/json;odata.metadata=minimal;odata.streaming=true",
  "Content-Type": "application/json;",
  "api-version": "3",
};

const apiURL = "http://192.168.22.14:6003/api/fondos/v3";
const apiURLv2 = "https://api.sistemasesco.com/api/fondos/v2";

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
      moneda: "",
    };
  },
  methods: {
    async getCuentas() {
      const response = await fetch(
        apiURLv2 +
          "/reportes/posicionCuotapartista?fecha=2021-10-15&pageSize=150",
        { headers: headersv2 }
      );
      const data = await response.json();

      data.data.forEach((cuenta) => {
        this.cuentas.push(cuenta.cuotapartista);
      });
      this.$refs.sCuotapartista.selected = 0;
    },
    async getCuentaBancaria(cuotapartista) {
      this.cuentasBancarias = [];
      const response = await fetch(
        apiURL +
          "/get-cuotapartistas?numCuotapartista=" +
          cuotapartista.target.value,
        { headers }
      );
      const data = await response.json();
      console.log(data);
      // this.cuentasBancarias = data.data[0].cuentasBancarias;
      data.data[0].cuentasBancarias.forEach((cuenta) => {
        if (cuenta.moneda.codISO === "ARS") {
          this.cuentasBancarias.push([
            cuenta.idCptCuentaBancaria,
            cuenta.numeroCuenta,
            cuenta.moneda.codISO,
          ]);
        }
      });
    },
    async getPosicion(event) {
      const response = await fetch(
        apiURLv2 +
          "/reportes/posicionCuotapartista?fecha=2021-10-15&pageSize=50&numCuotapartista=" +
          event.target.value,
        { headers: headersv2 }
      );
      const data = await response.json();
      this.cuentaSeleccionada = data.data;
    },
    async getMoneda(event) {
      const fondoClase = event.target.value.split("_");

      this.moneda = fondoClase[2];
      this.rescate.numFondo = fondoClase[0];
      this.rescate.clase = fondoClase[1];
      this.rescate.idSolicitud = String(Math.random()).replace("0.", "TEST");
      this.rescate.fechaConcertacion = "2021-10-15";
      this.cuotapartesTotal = fondoClase[3];
    },
    async setTotal(event) {
      this.rescate.cantidadCuotapartes = this.cuotapartesTotal;
      this.rescate.importe = 0;
      this.rescate.esTotal = true;
    },
    async setParcial(event) {
      this.rescate.cantidadCuotapartes = 0;
      this.rescate.importe = 0;
      this.rescate.esTotal = false;
    },
    insertRescate(event) {
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
        })
        .catch((error) => {
          console.log(error.response);
          alert(error.response.data.error.Msj);
        })
        .finally(() => {
          //Perform action in always
        });
    },
  },
  created() {
    this.getCuentas();
  },
};
</script>
