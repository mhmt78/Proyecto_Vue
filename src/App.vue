<template>
  <div v-if="logon" id="app" class="container">
    <!-- Contenido de las listas de gasto -->
    <div class="row text-primary">
      <div class="col-10">
        <h1>Lista de Gastos</h1>
      </div>
      <div class="col-2">
        <span
          v-bind:id="'agregar'"
          v-bind:class="['mr-1', 'btn', 'btn-primary', boton]"
          style="font-size: 10px; margin-top: 15px"
          v-on:click="butonPlus($event)"
        ></span>
      </div>
    </div>
    <div
      v-if="despliegue"
      key="despliegue"
      class="container border rounded text-primary lead p-2 mt-3"
    >
      <div class="row justify-content-md-center">
        <div class="col-3"></div>
        <div class="col-3">Nombre del gasto</div>
        <div class="col-3">
          <input v-model="nombreGasto" class="text-primary" type="text" />
        </div>
        <div class="col-3"></div>
      </div>
      <div class="row justify-content-md-center my-2">
        <div class="col-3"></div>
        <div class="col-3">Monto del gasto</div>
        <div class="col-3">
          <input v-model="montoGasto" class="text-primary" type="text" />
        </div>
        <div class="col-3"></div>
      </div>
      <div class="row justify-content-md-center my-2">
        <div class="col-3"></div>
        <div class="col-3">Tipo de gasto</div>
        <div class="col-3">
          <input v-model="tipoGasto" class="text-primary" type="text" />
        </div>
        <div class="col-3"></div>
      </div>
      <div class="row justify-content-md-center my-2">
        <div v-if="actualizar" class="col-12">
          <div id="actualizar" class="btn btn-primary" v-on:click="manejarClick($event)">Actualizar</div>
        </div>
        <div v-else class="col-12">
          <div id="agregar" class="btn btn-primary" v-on:click="manejarClick($event)">Agregar</div>
        </div>
      </div>
    </div>
    <!-- filtros -->
    <div class="row bd-example">
      <button
        id="hogar"
        type="button"
        v-bind:class="['btn', 'btn btn-outline-primary', mostrarHogar]"
        v-on:click="filtro($event)"
      >Hogar</button>
      <button
        id="trabajo"
        type="button"
        v-bind:class="['btn', 'btn btn-outline-primary', mostrarTrabajo]"
        v-on:click="filtro($event)"
      >Trabajo</button>
      <button
        id="carros"
        type="button"
        v-bind:class="['btn', 'btn btn-outline-primary', mostrarCarros]"
        v-on:click="filtro($event)"
      >Carros</button>
    </div>
    <!-- contenedor de la lista de gastos -->
    <div class="container border rounded text-primary mt-4">
      <div class="row lead border rounded font-weight-bold">
        <div class="col-3">Nombre del gasto</div>
        <div class="col-3">Monto del gasto</div>
        <div class="col-3">Tipo de gasto</div>
        <div class="col-3">Accion</div>
      </div>
      <!-- lista de los gastos -->
      <gastosComponente
        v-for="(gasto,index) in gastos"
        v-show="filter==='filtro'"
        v-bind:gasto="gasto"
        v-bind:id="gasto.id"
        v-bind:indice="index"
        v-bind:key="index"
        v-on:eliminarGasto="eliminar($event)"
        v-on:editarGasto="editarGasto($event)"
      ></gastosComponente>
      <!-- lista de filtros -->
      <filtrosComponente
        v-for="(gasto,index) in filtrados"
        v-show="gasto.tipo===filter"
        v-bind:gasto="gasto"
        v-bind:id="gasto.id"
        v-bind:indice="index"
        v-bind:key="filter+index"
        v-on:eliminarGasto="eliminar($event)"
        v-on:editarGasto="editarGasto($event)"
      ></filtrosComponente>
      <div class="row lead border rounded font-weight-bold">
        <div class="col-6">Total</div>
        <div
          v-if="filter==='filtro'"
          class="col-4"
        >{{new Intl.NumberFormat("de-DE").format(totalGasto)}}</div>
        <div v-else class="col-4">{{new Intl.NumberFormat("de-DE").format(cantidad)}}</div>
        <div class="col-2"></div>
      </div>
    </div>
  </div>
  <div v-else>
    <!-- Contenido de login -->
    <loginForm v-bind:firebase="firebase" v-on:ingresoCorrecto="ingresoCorrecto($event)"></loginForm>
  </div>
</template>

<script>
import firebase from "firebase";
import "firebase/firestore";
import loginForm from "./components/loginForm.vue";
import gastosComponente from "./components/GastosComponente.vue";
import filtrosComponente from "./components/FiltrosComponente.vue";

export default {
  name: "app",
  data: function() {
    return {
      boton: "fa fa-plus",
      gastos: [],
      filtrados: [],
      nombreGasto: "",
      tipoGasto: "",
      montoGasto: "",
      coleccion: {},
      logon: false,
      firebase: "",
      idUsuario: "",
      db: "",
      totalGasto: 0.0,
      actualizar: false,
      despliegue: false,
      idEditar: "",
      montoAnt: 0.0,
      indice: 0,
      filter: "filtro",
      mostrarHogar: "",
      mostrarTrabajo: "",
      mostrarCarros: "",
      cantidad: 0.0
    };
  },
  methods: {
    butonPlus: function(event) {
      if (event.target.id === "agregar") {
        this.boton = "fa fa-minus";
        document.getElementById("agregar").setAttribute("id", "actualizar");
        this.despliegue = true;
      } else if (event.target.id === "actualizar") {
        this.boton = "fa fa-plus";
        document.getElementById("actualizar").setAttribute("id", "agregar");
        this.despliegue = false;
        this.actualizar = false;
      }
      this.montoGasto = "";
      this.nombreGasto = "";
      this.tipoGasto = "";
      this.filter = "filtro";
      this.mostrarHogar = "";
      this.mostrarTrabajo = "";
      this.mostrarCarros = "";
    },
    manejarClick: function(evento) {
      if (evento.target.id === "agregar") {
        // Se agregara los nuevos gastos
        const gastoData = {
          MontoGasto: this.montoGasto,
          NombreGasto: this.nombreGasto,
          TipoGasto: this.tipoGasto
        };
        this.totalGasto += parseFloat(this.montoGasto);
        this.coleccion
          .add(gastoData)
          .then(docReference => {
            this.gastos.unshift({
              id: docReference.id,
              monto: gastoData.MontoGasto,
              nombre: gastoData.NombreGasto,
              tipo: gastoData.TipoGasto
            });
          })
          .catch(Errro => {
            alert(
              "No se pudo agregar el libro al sistema. Error: " + Errro.message
            );
          });
        this.montoGasto = "";
        this.nombreGasto = "";
        this.tipoGasto = "";
        this.despliegue = false;
        this.boton = "fa fa-plus";
        document.getElementById("actualizar").setAttribute("id", "agregar");
      }
      if (evento.target.id === "actualizar") {
        // Actualizando los gastos en firebase
        var washingtonRef = this.db
          .collection("/Usuarios/" + this.idUsuario + "/Gastos")
          .doc(this.idEditar);
        washingtonRef.update({
          MontoGasto: this.montoGasto,
          NombreGasto: this.nombreGasto,
          TipoGasto: this.tipoGasto
        });

        // Actualizando los valores de la lista de filtro
        console.log(this.montoAnt);
        this.cantidad -= this.montoAnt;
        if (this.filter !== "filtro") {
          this.filtrados[this.indice].nombre = this.nombreGasto;
          this.filtrados[this.indice].tipo = this.tipoGasto;
          this.filtrados[this.indice].monto = this.montoGasto;
          this.cantidad += parseFloat(this.montoGasto);
        }

        // actualizando los valores de la lista
        this.totalGasto -= this.montoAnt;
        this.gastos[
          this.gastos.indexOf(
            this.gastos.find(element => element.id === this.idEditar)
          )
        ].nombre = this.nombreGasto;
        this.gastos[
          this.gastos.indexOf(
            this.gastos.find(element => element.id === this.idEditar)
          )
        ].tipo = this.tipoGasto;
        this.gastos[
          this.gastos.indexOf(
            this.gastos.find(element => element.id === this.idEditar)
          )
        ].monto = this.montoGasto;
        this.totalGasto += parseFloat(this.montoGasto);

        this.montoGasto = "";
        this.nombreGasto = "";
        this.tipoGasto = "";
        this.despliegue = false;
        this.actualizar = false;
        this.boton = "fa fa-plus";
        document.getElementById("actualizar").setAttribute("id", "agregar");
      }
    },
    filtro: function(filt) {
      // filtros segun el tipo
      if (filt.target.id === "hogar") {
        if (this.mostrarHogar === "active") {
          this.mostrarHogar = "";
          this.filter = "filtro";
        } else {
          this.cantidad = 0.0;
          this.filtrados = [];
          this.gastos.forEach(gasto => {
            if (gasto.tipo === "Hogar") {
              this.filtrados.push({
                id: gasto.id,
                monto: gasto.monto,
                nombre: gasto.nombre,
                tipo: gasto.tipo
              });
              this.cantidad += parseFloat(gasto.monto);
            }
          });
          // console.log(this.filtrados + " " + this.cantidad);
          this.mostrarHogar = "active";
          this.mostrarTrabajo = "";
          this.mostrarCarros = "";
          this.filter = "Hogar";
        }
      } else if (filt.target.id === "trabajo") {
        if (this.mostrarTrabajo === "active") {
          this.mostrarTrabajo = "";
          this.filter = "filtro";
        } else {
          this.cantidad = 0.0;
          this.filtrados = [];
          this.gastos.forEach(gasto => {
            if (gasto.tipo === "Trabajo") {
              this.filtrados.push({
                id: gasto.id,
                monto: gasto.monto,
                nombre: gasto.nombre,
                tipo: gasto.tipo
              });
              this.cantidad += parseFloat(gasto.monto);
            }
          });
          // console.log(this.filtrados + " " + this.cantidad);
          this.mostrarHogar = "";
          this.mostrarTrabajo = "active";
          this.mostrarCarros = "";
          this.filter = "Trabajo";
        }
      } else {
        if (this.mostrarCarros === "active") {
          this.mostrarCarros = "";
          this.filter = "filtro";
        } else {
          this.cantidad = 0.0;
          this.filtrados = [];
          this.gastos.forEach(gasto => {
            if (gasto.tipo === "Carros") {
              this.filtrados.push({
                id: gasto.id,
                monto: gasto.monto,
                nombre: gasto.nombre,
                tipo: gasto.tipo
              });
              this.cantidad += parseFloat(gasto.monto);
            }
          });
          // console.log(this.filtrados + " " + this.cantidad);
          this.mostrarHogar = "";
          this.mostrarTrabajo = "";
          this.mostrarCarros = "active";
          this.filter = "Carros";
        }
      }
    },
    editarGasto: function(cambio) {
      // Modificar gasto
      this.actualizar = true;
      this.despliegue = true;
      this.boton = "fa fa-minus";

      if (!document.getElementById("actualizar")) {
        document.getElementById("agregar").setAttribute("id", "actualizar");
      }
      this.montoGasto = cambio.monto;
      this.nombreGasto = cambio.nombre;
      this.tipoGasto = cambio.tipo;
      this.idEditar = cambio.id;
      this.montoAnt = parseFloat(cambio.monto);
      this.indice = parseInt(cambio.indice);
    },
    eliminar: function(gastoID) {
      // Eliminar gastos

      // Eliminar gastos en la lista de filtros y la lista genera que es gastos
      if (this.filter !== "filtro") {
        this.cantidad -= parseFloat(gastoID.monto);
        this.totalGasto -= parseFloat(gastoID.monto);
        this.filtrados.splice(gastoID.indice, 1);
        this.coleccion.doc(gastoID.id).delete();
        this.gastos.splice(
          this.gastos.indexOf(
            this.gastos.find(element => element.id === gastoID.id)
          ),
          1
        );
      } else {
        this.totalGasto -= parseFloat(gastoID.monto);
        this.coleccion.doc(gastoID.id).delete();
        this.gastos.splice(gastoID.indice, 1);
      }
    },
    ingresoCorrecto: function(usuario) {
      console.log("User: " + usuario);
      this.idUsuario = usuario;
      this.logon = true;
      this.coleccion = this.db.collection("/Usuarios/" + usuario + "/Gastos");
      this.coleccion.get().then(gastos => {
        gastos.forEach(gasto => {
          this.gastos.push({
            id: gasto.id,
            monto: gasto.data().MontoGasto,
            nombre: gasto.data().NombreGasto,
            tipo: gasto.data().TipoGasto
          });
          this.totalGasto += parseFloat(gasto.data().MontoGasto);
        });
      });
    }
  },
  components: {
    loginForm,
    gastosComponente,
    filtrosComponente
  },
  beforeMount: function() {
    var config = {
      apiKey: "AIzaSyCrUxVyosflbJgmi1ZB9Jf3pru3ZAB2AuQ",
      authDomain: "vueproyecto-8768f.firebaseapp.com",
      databaseURL: "https://vueproyecto-8768f.firebaseio.com",
      projectId: "vueproyecto-8768f",
      storageBucket: "vueproyecto-8768f.appspot.com",
      messagingSenderId: "379224037943",
      appId: "1:379224037943:web:4e543933007a4aed627db2",
      measurementId: "G-0LK6CLF9NX"
    };
    firebase.initializeApp(config);
    this.db = firebase.firestore();
    const settings = { timestampsInSnapshots: true };
    this.db.settings(settings);
    this.firebase = firebase;
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
