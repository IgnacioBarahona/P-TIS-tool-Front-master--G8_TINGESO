<template>
  <div>

    <br>
    <div class="columns">
      <div class="column is-8 is-offset-2">
        <p class="title is-5 has-text-centered">Avance de {{ nombreCompleto(estudiante.usuario) }} - ({{ estudiante.iniciales }})</p>
      </div>
      <div class="column is-2"></div>
    </div>

    <div class="columns is-centered">
      <div class="column is-10">
        <p class="title is-6 has-text-left">Impedimentos:</p>
        <div class="content has-text-left">
          <ol type="1">
            <li v-for="(impedimento, index) in impedimentos" :key="index">
              <p>{{ impedimento.descripcion }}</p>
              <div v-if="!openComment">
                <a @click="abrirComentario(index, impedimento.id, 'impedimento')">comentar</a>
                <p>es {{sectionSelect.apartado}}</p>
              </div>
              <div v-else class="columns">
                <div class="column is-12">
                  <div class="content has-text-left">
                    <div class="field is-grouped">
                      <p class="control is-expanded has-icons-right">
                        <input v-model="listaComentarios[index].comentario" class="input is-normal" type="text" @input="validarComentario">
                        <span class="icon is-right">
                          <button class="delete" @click="removerComentario(comentario)"></button>
                        </span>
                      </p>
                    </div>
                    <p class="is-danger help" v-if="entradas.conclusiones">No se ha ingresado el comentario</p>
                  </div>
                </div>
            </div>
            </li>
          </ol>
        </div>
      </div>
    </div>

    <div class="columns is-centered">
      <div class="column is-10">
        <p class="title is-6 has-text-left">Logros:</p>
        <div class="content has-text-left">
          <ol type="1">
            <li v-for="(logro, index) in logros" :key="index">
              <p>{{ logro.descripcion }}</p>
              <div v-if="!mostrarComentarLogros[index]">
                <a @click="abrirComentario(index, logro.id,'logro')">comentar</a>
              </div>
              <div v-else class="columns">
                <div class="column is-12">
                  <div class="content has-text-left">
                    <div class="field is-grouped">
                      <p class="control is-expanded has-icons-right">
                        <input v-model="listaComentarios[index].comentario" class="input is-normal" type="text" @input="validarComentario">
                        <span class="icon is-right">
                          <button class="delete" @click="removerComentario(comentario)"></button>
                        </span>
                      </p>
                    </div>
                    <p class="is-danger help" v-if="entradas.conclusiones">No se ha ingresado el comentario</p>
                  </div>
                </div>
            </div>
            </li>
          </ol>
        </div>
      </div>
    </div>

    <div class="columns is-centered">
      <div class="column is-10">
        <p class="title is-6 has-text-left">Metas:</p>
        <div class="content has-text-left">
          <ol type="1">
            <li v-for="(meta, index) in metas" :key="index">
              <p>{{ meta.descripcion }}</p>
              <div v-if="!mostrarComentarMetas[index]">
                <a @click="abrirComentario(index, meta.id,'meta')">comentar</a>
              </div>
              <div v-else class="columns">
              <div class="column is-12">
                <div class="content has-text-left">
                  <div class="field is-grouped">
                    <p class="control is-expanded has-icons-right">
                      <input v-model="listaComentarios[index].comentario" class="input is-normal" type="text" @input="validarComentario">
                      <span class="icon is-right">
                        <button class="delete" @click="removerComentario(comentario)"></button>
                      </span>
                    </p>
                  </div>
                  <p class="is-danger help" v-if="entradas.conclusiones">No se ha ingresado el comentario</p>
                </div>
              </div>
            </div>
            </li>
          </ol>
        </div>
      </div>
    </div>
    <br>

  </div>
</template>

<script>
import Funciones from '@/services/funciones.js'

export default {
  name: 'VisorEstudiante',
  props: ['est', 'logros', 'metas', 'impedimentos', 'listaCom'],
  data () {
    return {
      estudiante: this.est,
      listaLogros: this.logros,
      listaMetas: this.metas,
      listaImpedimentos: this.impedimentos,
      listaComentarios: [],
      listaGenerales: [],
      listaAbiertos: [],
      mostrarComentarImpedimentos: [],
      mostrarComentarLogros: [],
      mostrarComentarMetas: [],
      comentario: {
        comentario: '',
        es_item: true,
        id_item: 0,
        apartado: '',
        id_est: 0
      },
      sectionSelect: {
        index: null,
        apartado: ''
      },
      entrada: {
        error: false,
        mensaje: ''
      },
      listaEntradas: [],
      entradas: {
        comentarios: false
      },
      comentariosMinuta: this.listaCom
    }
  },
  methods: {
    nombreCompleto: function (usuario) {
      return Funciones.nombreCompleto(usuario)
    },
    abrirComentario: function (index, id, apartado) {
      this.sectionSelect.index = index
      this.sectionSelect.apartado = apartado
      if (apartado === 'impedimento') {
        this.mostrarComentarImpedimentos[index] = true
      } else if (apartado === 'logro') {
        this.mostrarComentarLogros[index] = true
      } else if (apartado === 'meta') {
        this.mostrarComentarMetas[index] = true
      }
      this.listaComentarios[index].id_item = id
      this.listaComentarios[index].apartado = apartado
    },
    cerrarComentario: function (index) {
      this.mostrarComentarImpedimentos[index] = false
      this.listaComentarios[index].comentario = ''
      this.listaEntradas[index].error = false
      this.listaComentarios[index].apartado = ''
    },
    agregaComentario: function () {
      var comentario = Object.assign({}, this.comentario)
      comentario.es_item = false
      this.listaGenerales.push(comentario)
    },
    removerComentario: function (comentario) {
      return Funciones.removeFromArray(this.listaGenerales, comentario)
    },
    crearListas: function () {
      this.limpiarCampos()
      var objA
      for (var i = 0; i < this.logros.length; i++) {
        this.mostrarComentarLogros.push(false)
        objA = Object.assign({}, this.comentario)
        this.listaComentarios.push(objA)
      }
      for (var j = 0; j < this.metas.length; j++) {
        this.mostrarComentarMetas.push(false)
        objA = Object.assign({}, this.comentario)
        this.listaComentarios.push(objA)
      }
      for (var k = 0; k < this.impedimentos.length; k++) {
        this.mostrarComentarImpedimentos.push(false)
        objA = Object.assign({}, this.comentario)
        this.listaComentarios.push(objA)
      }
    },
    limpiarCampos: function () {
      this.mostrarComentarImpedimentos = []
      this.mostrarComentarLogros = []
      this.mostrarComentarMetas = []
      this.listaComentarios = []
      this.listaAbiertos = []
      this.listaEntradas = []
    },
    isOpenComentario: function (index, apartado) {
      if (apartado === 'impedimento' && this.mostrarComentarImpedimentos[index]) {
        return true
      } else if (apartado === 'logro' && this.mostrarComentarLogros[index]) {
        return true
      } else if (apartado === 'meta' && this.mostrarComentarMetas[index]) {
        return true
      }
      return false
    }
  },
  computed: {
    sinComentarios: function () {
      return Funciones.sinComentarios(this.listaComentarios.concat(this.listaGenerales))
    },
    openComment: function () {
      return this.isOpenComentario(this.sectionSelect.index, this.sectionSelect.apartado)
    }
  },
  beforeUpdate () {
    if (localStorage.user_tk) {
      this.crearListas()
    }
  }
}
</script>
