<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-6">
        <div class="card">
          <h5 class="card-header">Recursos</h5>
          <div class="card-body">
            <h5 class="card-title">Documentación</h5>
            <p class="card-text">
              Consulte la documentación funcional de nuestra API
            </p>
            <a
              href="https://emsnet.sistemasesco.com.ar/VisualFondos/docs/api/"
              class="btn btn-primary"
              >Visitar</a
            >
          </div>
        </div>
      </div>
      <div class="col-sm-6">
        <div class="card">
          <h5 class="card-header">Recursos</h5>
          <div class="card-body">
            <h5 class="card-title">Sandbox</h5>
            <p class="card-text">
              Visite el sandbox de nuestra API para poder comprender toda su
              funcionalidad
            </p>
            <a
              href="https://api.sistemasesco.com/swagger/index.html?urls.primaryName=VisualFondos%20v%202"
              target="_blank"
              class="btn btn-primary"
              >Visitar</a
            >
          </div>
        </div>
      </div>
    </div>
  </div>
  <br />
  <br />
  <br />
  <div class="container">
    <div class="row">
      <div class="col-sm-6">
        <div class="card">
          <h5 class="card-header">Ejemplos</h5>
          <div class="card-body">
            <h5 class="card-title">Consultivos</h5>
            <p class="card-text">Ejemplo de métodos Consultivos</p>

            <router-link to="./solicitudes" class="card-link"
              >Solicitudes</router-link
            >
            <router-link to="./liquidaciones" class="card-link"
              >Liquidaciones</router-link
            >
            <router-link to="./posicion" class="card-link"
              >Posición Cuotapartista</router-link
            >
            <router-link to="./valorescp" class="card-link"
              >Valores de Cuotaparte</router-link
            >
          </div>
        </div>
      </div>

      <div class="col-sm-6">
        <div class="card">
          <h5 class="card-header">Ejemplos</h5>
          <div class="card-body">
            <h5 class="card-title">Transaccionales</h5>
            <p class="card-text">
              Debajo encontrará ejemplos de los métodos transaccionales de la
              API de Fondos
            </p>
            <router-link to="./suscripcion" class="card-link"
              >Solicitudes de Suscripcion</router-link
            >

            <router-link to="./rescate" class="card-link"
              >Solicitudes de Rescate</router-link
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script></script>
