<template>
    <div class="container">
        <h1>PLAYLIST</h1>

        <b-alert 
        :show="dismissCountDown" 
        dismissible 
        :variant="mensaje.color" 
        @dismissed="dismissCountDown=0" 
        @dismiss-count-down="countDownChanged" 
        > 
          {{mensaje.texto}} 
        </b-alert>

        <form @submit.prevent="AgregarNota()" v-if="!editar">
          <h3>Agregar un song nuevo</h3>

          <input type="text" class= "form-control my-2" placeholder="nombre" v-model="nota.nombre">
          <input type="text" class= "form-control my-2" placeholder="descripcion" v-model="nota.descripcion">
          <b-button class="btn-success my-2" type="submit">Agregar</b-button>


        </form>
        
        <form @submit.prevent="EditarNota(notaEditar)" v-if="editar">
          <h3>Editar song</h3>

          <input type="text" class= "form-control my-2" placeholder="nombre" v-model="notaEditar.nombre">
          <input type="text" class= "form-control my-2" placeholder="descripcion" v-model="notaEditar.descripcion">
          <b-button class="btn-success my-2 mx-2" type="submit">editar</b-button>
          <b-button class= "my-2" type="submit" @click="editar= false">Cancelar</b-button>

        </form>



        <table class="table">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">Nombre</th>
      <th scope="col">Descripcion</th>
      <th scope="col">Item</th>
      <th scope="col">Fecha</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(item, index) in notas" :key="index">
      <th scope="row">{{item._id}}</th>
      <td>{{item.nombre}}</td>
      <td>{{item.descripcion}}</td>
      <td>{{item.usuarioId}}</td>
      <td>{{item.date}}</td>
      <td>
        <b-button class="btn-warning mx-2" @click="EliminarNota(item._id)">Eliminar</b-button>
        <b-button class="btn-blue mx-2" @click="ActivarEdicion(item._id)">Editar</b-button>
      </td>
    </tr>
  </tbody>
</table>
</div> 
</template>

<script>

export default {
  data(){
    return {
      notas: [],
      mensaje: {color: 'success', texto: ''}, 
      dismissSecs: 5, 
      dismissCountDown: 0,
      nota:{nombre:"", descripcion:""},
      editar: false,
      notaEditar:{}

    };
  },
  created(){
    this.listarNotas();

  },
  methods: {
    listarNotas(){
      this.axios.get('/nota')
      .then(res=> {
        console.log(res.data);
          this.notas=res.data;
      })
      .catch(e=>{
        console.log(e.response);
      })
    },
    AgregarNota(){
      this.axios.post('/nueva-nota', this.nota)
      .then(res=>{
        this.notas.push(res.data)
        this.nota.nombre="";
        this.nota.descripcion="";
        this.mensaje.color="success";
        this.mensaje.texto="Mensaje Agregado";
        this.showAlert();

      })
      .catch(e =>{
        console.log(e.response);
      })


  },
  EliminarNota(id){
    this.axios.delete(`/nota/${id}`)
    .then(res=>{

      const index = this.notas.findIndex(item=> item._id===res.data._id);
      this.notas.splice(index, 1);
      this.mensaje.color="success";
      this.mensaje.texto="Nota Eliminada";
      this.showAlert();

    })
    .catch(e=>{
      console.log(e.response);
    })

  },

  ActivarEdicion(id){
    this.editar=true;
    this.axios.get(`/nota/${id}`)
    .then(res=>{
      this.notaEditar=res.data;
    })
    .catch(e=>{
      console.log(e.response);
    })
  },

  EditarNota(item){
    this.axios.put(`/nota/${item._id}` , item)
    .then(res=>{
      const index= this.notas.findIndex(n=> n._id===res.data._id);
      this.notas[index].nombre=res.data.nombre;
      this.notas[index].descripcion=res.data.descripcion;
      this.mensaje.color="success";
      this.mensaje.texto="Nota Editada";
      this.showAlert();
      this.editar= false;
    })
    .catch(e=>{
      console.log(e.response);
    })

  },
  countDownChanged(dismissCountDown) { 
    this.dismissCountDown = dismissCountDown 
  }, 
  showAlert() { 
    this.dismissCountDown = this.dismissSecs 
  }
  }
  
};
</script>