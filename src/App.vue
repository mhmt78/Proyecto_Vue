<template>
  <div id="app">
    <div id='login' v-if='!logon' >
      <loginForm v-bind:firebase="firebase"
                 v-on:ingresoCorrecto='ingresoCorrecto($event)'></loginForm>
    </div>
    <div id='inicio' v-else>
      <div>Usuario: {{userData.usuario}} </div>
      <div>Tipo de gasto: {{userData.gastos.tipo}} </div>
      <div>Monto del Gastos: {{userData.gastos.monto}} </div>
    </div>
  </div> 
</template>

<script>

import firebase from 'firebase'
import 'firebase/firestore'
import loginForm from './components/loginForm.vue';

export default {
  name: 'app',
  data: function (){
    return {
      logon:false,
      coleccionRaiz:'usuarios',
      idUsuario:'null',
      coleccionGastos:'gastos',
      userData:{},
      firebase:'',
      db:''
    }
  },
  components: {
    loginForm
  },
  beforeMount: function () {
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
    const settings = {timestampsInSnapshots: true};
    this.db.settings(settings);
    this.firebase=firebase
  },
  methods: {
    ingresoCorrecto: function(usuario) {
      console.log('User: '+usuario)
      this.idUsuario=usuario
      this.logon=true
      const doc = this.db.collection(this.coleccionRaiz).doc(this.idUsuario)
      this.userData={usuario:doc.id,gastos:{}}
      const gastos = doc.collection(this.coleccionGastos)
      const alquilerCasa = gastos.doc('alquiler_casa')
      this.userData.gastos={tipo:alquilerCasa.id,monto:0}
      alquilerCasa.get().then((doc)=>{
      this.userData.gastos.monto=doc.data().monto
    })
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
