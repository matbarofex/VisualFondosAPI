<template>
  <h1>Valores de Cuotaparte</h1>
  <table class="table table-striped">
    <thead>
      <tr>
        <th scope="col">Fecha</th>
        <th scope="col">Fondo</th>
        <th scope="col">Valor Cuotaparte</th>
        <th scope="col">Patrimonio Neto</th>
        <th scope="col">Variación</th>
        <th scope="col">Cuotapartes en Circulación</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="vcp in this.vcps" :key="vcp.fondo.numFondo">
        <td>{{ format_date(vcp.fecha) }}</td>
        <td>
          Fondo {{ vcp.fondo.numFondo }} -
          {{ vcp.tipoValorCuotaparte.descripcion }}
        </td>
        <td>
          {{ vcp.moneda.simbolo }}
          {{ vcp.valorCuotaparte.toLocaleString("es-AR") }}
        </td>
        <td>{{ vcp.patrimonioNeto.toLocaleString("es-AR") }}</td>

        <td>
          {{ vcp.variacion.toLocaleString("es-AR") }} %
          <i
            v-if="vcp.variacion > 0"
            class="bi-arrow-up-short"
            style="color:green"
          ></i>
          <i
            v-if="vcp.variacion < 0"
            class="bi-arrow-down-short"
            style="color:red"
          ></i>
        </td>
        <td>
          {{ vcp.moneda.simbolo }}
          {{ vcp.cuotapartesCirculacion.toLocaleString("es-AR") }}
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import moment from "moment";

export default {
  name: "ValoresCP",
  data: function() {
    return {
      vcps: [{}],
    };
  },
  methods: {
    async getVCP() {
      const headers = {
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
        "Content-Type": "application/json;",
        "api-version": "2",
      };

      const response = await fetch(
        "https://api.sistemasesco.com/api/fondos/v2/reportes/valorCuotapartes?fecha=2021-10-15&pageSize=50",
        { headers }
      );
      const data = await response.json();
      this.vcps = data.data;
    },
    format_date(value) {
      if (value) {
        return moment(String(value)).format("DD/MM/YYYY");
      }
    },
  },
  created() {
    this.getVCP();
  },
};
</script>
