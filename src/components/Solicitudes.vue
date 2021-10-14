<template>
  <h1>Solicitudes</h1>
  <table class="table">
    <thead>
      <tr>
        <th scope="col">#</th>
        <th scope="col">Solicitud</th>
        <th scope="col">Fondo</th>
        <th scope="col">Fecha Concertacion</th>
        <th scope="col">Fecha Liquidación</th>
        <th scope="col">Moneda</th>
        <th scope="col">Importe/Cuotapartes</th>
        <th scope="col">Cuotapartista</th>
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
        <td>{{ format_date(sol.fechaLiquidacion) }}</td>
        <td>{{ sol.moneda.simbolo }}</td>
        <td>{{ sol.importe.toLocaleString() }}</td>
        <td>Cuenta {{ sol.cuotapartista.numero }}</td>
      </tr>
    </tbody>
  </table>

  <nav role="navegation" aria-label="pagination">
    <ul class="pagination">
      <li class="page-item">
        <a class="page-link" v-on:click="changePage(1)">Primera</a>
      </li>
      <li class="page-item">
        <a class="page-link" v-on:click="changePage(previousPageNumber)"
          >Anterior</a
        >
      </li>
      <li class="page-item">
        <a class="page-link">{{ page }}</a>
      </li>
      <li class="page-item">
        <a class="page-link" v-on:click="changePage(nextPageNumber)"
          >Siguiente</a
        >
      </li>
      <li class="page-item">
        <a class="page-link" v-on:click="changePage(totalPages - 1)">Ultima</a>
      </li>
    </ul>
  </nav>
</template>

<script>
import moment from "moment";
import axios from "axios";

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

export default {
  name: "Solicitudes",
  data: function() {
    return {
      solicitudes: [{}],
      page: 1,
      hasPreviousPage: false,
      hasNextPage: true,
      nextPageNumber: 2,
      pageSize: 10,
      previousPageNumber: 1,
      totalPages: 98,
      totalItems: 975,
      token: "",
    };
  },
  methods: {
    async getSolicitudes() {
      const _token = await login();
      const headers = {
        "api-version": "3",
        "Content-Type":
          "application/json;odata.metadata=minimal;odata.streaming=true",
        accept: "application/json;odata.metadata=minimal;odata.streaming=true",
        Authorization: "Bearer " + _token,
      };

      axios
        .get(
          "https://api.sistemasesco.com/api/fondos/v3/reportes/solicitudes?fechaDesde=2020-04-27&fechaHasta=" +
            this.getDate() +
            "&pageSize=15&pageNumber=" +
            this.page,
          { headers }
        )
        .then((response) => {
          (this.page = response.data.pageNumber),
            (this.hasPreviousPage = response.data.hasPreviousPage);
          this.hasNextPage = response.data.hasNextPage;
          this.nextPageNumber = response.data.nextPageNumber;
          this.pageSize = response.data.pageSize;
          this.previousPageNumber = response.data.previousPageNumber;
          this.totalPages = response.data.totalPages;
          this.totalItems = response.data.totalItems;
          this.solicitudes = response.data.data;
        })
        .catch((e) => {
          console.log(e);
        });
    },
    changePage(page) {
      this.page = page <= 0 || page > this.totalPages ? this.page : page;
      this.getSolicitudes();
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
    // this.token = this.login();
    this.getSolicitudes();
  },
};
</script>
