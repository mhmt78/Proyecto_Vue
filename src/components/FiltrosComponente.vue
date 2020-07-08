<template>
  <div class="row lead my-1">
      <div class="col-3">{{gasto.nombre}}</div>
      <div class="col-3">{{new Intl.NumberFormat("de-DE").format(gasto.monto)}}</div>
      <div class="col-3">{{gasto.tipo}}</div>
      <div class="col-3">
        <div id="editar" class="btn btn-primary fa fa-pencil-square m-1"
          v-on:click="editar($event,{
            indice:indice, 
            id:id, 
            nombre: gasto.nombre,
            monto: gasto.monto,
            tipo: gasto.tipo,
            mostrar: true})">
        </div>
        <div
          id="eliminar"
          class="btn btn-primary fa fa-trash m-1"
          v-on:click="manejarClick($event,{indice:indice, id:id, monto: gasto.monto})">
        </div>
      </div>
    </div>
</template>

<script>
export default {
  name: "filtros",
  props: ["gasto", "id", "indice"],
  data: function() {
    return {
      test: "test",
      monstrar: false
    };
  },
  methods: {
    manejarClick: function(evento, gastoID) {
      if (evento.target.id === "eliminar") {
        this.$emit("eliminarGasto", gastoID);
      }
    },
    editar: function(event, cambios) {
      this.$emit("editarGasto", cambios);
      //   this.monstrar = true;
    },
    agregarTarea: function(event) {
      console.log(event.mostrar);
      this.$emit("editarGasto", event);
      if (!event.mostrar) this.monstrar = false;
    }
  }
};
</script>

<style>
</style>
