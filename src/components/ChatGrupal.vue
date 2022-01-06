<template>
<div>
  <div class="column is-8 is-offset-2">
    <div class="card" >
      <header class="card-header">
        <div class="column is-6">
          <span class="icon-text">
            <span class="icon" style=" vertical-align: middle; font-size:22px; color:#757575; margin-right: 8px">
              <i class="fas fa-user-circle fa-lg"></i>
            </span>
            <span style="color:#F9952D; font-weight: bold; font-size:110%">Cliente</span>
          </span>
        </div>
      </header>
      <div class="card-content" style="background-color:#F9F9F9">
        <div class="column is-6">
          <div class="box" style="padding:10px">
                <p style="color:#F9952D; font-weight: bold">Cliente</p>
                <p>Buenas Tardes</p>
                <div class="content has-text-right">
                  <p style="font-size:75%">4:20 pm</p>
                </div>
          </div>
        </div>
      </div>
      <footer class="card-footer">
        <div class="column is-11">
          <div class="control">
            <input class="input" type="text" style="background-color:#F9F9F9" placeholder="Escribe un mensaje aquí...">
          </div>
        </div>
        <div class="column is-1">
          <span class="icon" style="color:#757575;margin-top:5px">
              <i class="fas fa-paper-plane fa-lg"></i>
          </span>
        </div>
      </footer>
    </div>
  </div>
</div>
</template>
<script>
export default {
  name: 'ChatGrupal',
  data () {
    return {
      cliente: {
        nombre: '',
        apellido_paterno: '',
        apellido_materno: '',
        email: ''
      },
      estudiantes: [],
      grupo: {
        nombre: '',
        proyecto: ''
      }
    }
  },
  methods: {
  }
}
</script>
