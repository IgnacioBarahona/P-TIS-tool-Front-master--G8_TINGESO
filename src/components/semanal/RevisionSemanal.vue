<template>
  <div>

    <br>
    <InfoAvance :grupo="grupoSeleccionado" :minuta="bitacora"/>
    <br>

    <div v-for="estudiante in grupoSeleccionado.estudiantes" :key="estudiante.id" >

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
            <li v-for="(impedimento, index) in impedimentosPorEstudiante(estudiante.id)" :key="index">
              <p>{{ impedimento.descripcion }}</p>
              <div v-if="!mostrarComentar[index]">
                <a @click="abrirComentario(index, impedimento.id,'impedimento')">comentar</a>
                <!-- <p>es {{sectionSelect.apartado}}</p> -->
              </div>
              <div v-else class="columns">
                <div class="column is-12">
                  <div class="content has-text-left">
                    <div class="field is-grouped">
                      <p class="control is-expanded has-icons-right">
                        <input v-model="listaComentarios[index].comentario" class="input is-normal" type="text" @input="validarComentario">
                        <span class="icon is-right">
                          <button class="delete" @click="cerrarComentario(index)"></button>
                        </span>
                      </p>
                    </div>
                    <p class="is-danger help" v-if="entrada.comentarios">No se ha ingresado el comentario</p>
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
            <li v-for="(logro, index) in logrosPorEstudiante(estudiante.id)" :key="index">
              <p>{{ logro.descripcion }}</p>
              <div v-if="!mostrarComentar[index]">
                <a @click="abrirComentario(index, logro.id,'logro')">comentar</a>
              </div>
              <div v-else class="columns">
                <div class="column is-12">
                  <div class="content has-text-left">
                    <div class="field is-grouped">
                      <p class="control is-expanded has-icons-right">
                        <input v-model="listaComentarios[index].comentario" class="input is-normal" type="text" @input="validarComentario">
                        <span class="icon is-right">
                          <button class="delete" @click="cerrarComentario(index)"></button>
                        </span>
                      </p>
                    </div>
                    <p class="is-danger help" v-if="entrada.comentarios">No se ha ingresado el comentario</p>
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
            <li v-for="(meta, index) in metasPorEstudiante(estudiante.id)" :key="index">
              <p>{{ meta.descripcion }}</p>
              <div v-if="!mostrarComentar[index]">
                <a @click="abrirComentario(index, meta.id,'meta')">comentar</a>
              </div>
              <div v-else class="columns">
              <div class="column is-12">
                <div class="content has-text-left">
                  <div class="field is-grouped">
                    <p class="control is-expanded has-icons-right">
                      <input v-model="listaComentarios[index].comentario" class="input is-normal" type="text" @input="validarComentario">
                      <span class="icon is-right">
                        <button class="delete" @click="cerrarComentario(index)"></button>
                      </span>
                    </p>
                  </div>
                  <p class="is-danger help" v-if="entrada.comentarios">No se ha ingresado el comentario</p>
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

    <!-- <div v-for="estudiante in grupoSeleccionado.estudiantes" :key="estudiante.id">
      <VisorEstudiante :est="estudiante" :logros="logrosPorEstudiante(estudiante.id)" :metas="metasPorEstudiante(estudiante.id)" :impedimentos="impedimentosPorEstudiante(estudiante.id)" :listaCom ="comentariosPorEstudiante(estudiante.id)"/>
      <br>
    </div> -->

  </div>
</template>

<script>
import InfoAvance from '@/components/semanal/InfoAvance.vue'
// import VisorEstudiante from '@/components/semanal/VisorEstudiante.vue'

import Funciones from '@/services/funciones.js'

export default {
  name: 'RevisionSemanal',
  components: {
    InfoAvance
    // VisorEstudiante
  },
  props: ['grupo', 'minuta'],
  data () {
    return {
      grupoSeleccionado: this.grupo,
      bitacora: this.minuta,
      itemsLogros: [],
      itemsMetas: [],
      itemsImpedimentos: [],
      listaGenerales: [],
      listaEntradas: [],
      listaComentarios: [],
      comentario: {
        comentario: '',
        es_item: true,
        id_item: 0
      },
      entrada: {
        error: false,
        mensaje: ''
      },
      mostrarComentar: []
    }
  },
  computed: {
    mostrarBitacora: function () {
      return Object.keys(this.bitacora).length > 0
    }
  },
  methods: {
    separarItems: function (listaItems) {
      for (var i = 0; i < listaItems.length; i++) {
        if (listaItems[i].tipo_item.tipo === 'Logro') {
          this.itemsLogros.push(listaItems[i])
        } else if (listaItems[i].tipo_item.tipo === 'Meta') {
          this.itemsMetas.push(listaItems[i])
        } else if (listaItems[i].tipo_item.tipo === 'Impedimento') {
          this.itemsImpedimentos.push(listaItems[i])
        }
      }
    },
    buscarIdAsistencia: function (idEstudiante) {
      if (this.mostrarBitacora) {
        return Funciones.buscarIdAsistencia(this.bitacora, idEstudiante)
      }
    },
    logrosPorEstudiante: function (idEstudiante) {
      return Funciones.separarPorEstudiante(this.itemsLogros, this.buscarIdAsistencia(idEstudiante))
    },
    metasPorEstudiante: function (idEstudiante) {
      return Funciones.separarPorEstudiante(this.itemsMetas, this.buscarIdAsistencia(idEstudiante))
    },
    impedimentosPorEstudiante: function (idEstudiante) {
      return Funciones.separarPorEstudiante(this.itemsImpedimentos, this.buscarIdAsistencia(idEstudiante))
    },
    nombreCompleto: function (usuario) {
      return Funciones.nombreCompleto(usuario)
    },
    abrirComentario: function (index, id) {
      this.mostrarComentar[index] = true
      this.listaComentarios[index].id_item = id
    },
    limpiarCampos: function () {
      this.mostrarComentar = []
      this.listaComentarios = []
      this.listaGenerales = []
    },
    limpiarErrorItem: function (index) {
      this.listaEntradas[index].error = false
      this.listaEntradas[index].mensaje = ''
    },
    cerrarComentario: function (index) {
      this.mostrarComentar[index] = false
      this.listaComentarios[index].comentario = ''
      this.listaEntradas[index].error = false
    },
    crearListas: function () {
      this.limpiarCampos()
      for (var i = 0; i < this.bitacora.minuta.items.length; i++) {
        this.mostrarComentar.push(false)
        this.listaComentarios.push(Object.assign({}, this.comentario))
        this.listaEntradas.push(Object.assign({}, this.entrada))
      }
    },
    enviarComentarios: function () {
      if (this.validarComentarios()) {
        var comentarios = this.listaComentarios.concat(this.listaGenerales)
        this.$emit('comentar', comentarios)
      }
    },
    cancelarEnvio: function () {
      this.$emit('cerrar')
      this.limpiarCampos()
      this.limpiarErrorGeneral()
      for (var i = 0; i < this.listaEntradas.length; i++) {
        this.listaEntradas[i].error = false
        this.listaEntradas[i].mensaje = ''
      }
    },
    validarComentarioItem: function (index) {
      if (this.mostrarComentar[index]) {
        if (this.listaComentarios[index].comentario === '' || this.listaComentarios[index].comentario === undefined) {
          this.listaEntradas[index].error = true
          this.listaEntradas[index].mensaje = 'Falta ingresar el comentario'
          return false
        } else {
          return true
        }
      } else {
        return true
      }
    }
  },
  created () {
    if (localStorage.user_tk) {
      this.crearListas()
    }
  },
  mounted () {
    if (this.mostrarBitacora) {
      this.separarItems(this.bitacora.minuta.items)
    }
  }
}
</script>
