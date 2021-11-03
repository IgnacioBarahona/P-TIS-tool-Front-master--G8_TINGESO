<template>
  <div class="has-text-left">

    <div v-if="crearMinuta">

      <Minuta :tipo-minuta="tipo" :id-bitacora="idBitacora" :id-motivo="idMotivo" :re-emitir="esNuevaEmision" :letra-revision="nuevaRevision" :estado="revisionEstado" v-if="verFormulario" @cerrar="cerrarFormulario"/>

      <div v-else-if="verSemanal">
        <Semanal :avance="bitacoraAvance" :tipo-minuta="tipo"  @cerrar="cerrarSemanal"/>
      </div>

      <div v-else>

        <div class="columns">
          <div class="column is-10"></div>
          <div class="column is-2">
            <button class="button is-info-usach" @click="nuevaMinuta">Nueva Minuta</button>
          </div>
        </div>
        <div v-if="seleccionarMinuta">
          <div class="columns">
            <div class="column is-8 is-offset-1">
              <div class="field is-horizontal">
                <div class="field-label-2c is-normal">
                  <label class="label">Elegir tipo minuta:</label>
                </div>
                <div class="field-body">
                  <div class="field is-grouped">
                    <div class="control is-expanded">
                      <div class="select is-fullwidth">
                        <select v-model="tipo">
                          <option v-for="item in tipoMinutas" :key="item.id" :value="item.id">{{ item.tipo }}</option>
                        </select>
                      </div>
                    </div>
                    <div class="control">
                      <a class="button is-info-usach" @click="elegirTipo">Elegir</a>
                    </div>
                    <div class="control">
                      <a class="button is-light-usach" @click="cancelarMinuta">Cancelar</a>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <br>
        </div>

        <Tablero :seleccionado="valorActual" :contador="tableroEst" @cambiar="cambiarTab" @bitacora="establecerBitacora" @revision="establecerRevision" @comentarios="revisarComentarios" @respuestas="revisarRespuestas" @emitir="nuevaVersion" @avance="editarAvance" @revisar-avance="revisarAvance" @ver-minuta="mostrarMinuta"/>

      </div>

    </div>

    <div v-else-if="verRevision">
      <Comentar :id-bitacora="idRevision" @cerrar="mostrarTablero"/>
    </div>

    <div v-else-if="verComentarios">
      <Responder :id-bitacora="idComentarios" @cerrar="mostrarTablero"/>
    </div>

    <div v-else-if="verRespuestas">
      <Respuestas :id-bitacora="idRespuestas" @cerrar="mostrarTablero"/>
    </div>

    <div v-if="verEmision">
      <Emision :id-bitacora="idEmision" @cerrar="nuevaEmision" @revisar="revisarAprobacion" @cancelar="mostrarTablero"/>
    </div>

    <div v-else-if="revisarSemanal">
      <RevisionSemanal :grupo="grupo" :minuta="bitacoraAvance"/>

      <br>
      <div class="columns is-centered">
        <div class="column is-5">
          <div class="field">
            <div class="control">
              <button class="button is-primary-usach is-fullwidth" @click="cerrarAvance">Volver</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="verMinuta">
      <RevisarMinuta :id-bitacora="idVerMinuta" @cerrar="cerrarMinuta"/>
    </div>

  </div>
</template>

<script>
import Minuta from '@/components/Minuta.vue'
import Tablero from '@/components/TableroEst.vue'
import Comentar from '@/components/comentarios/ComentarMinuta.vue'
import Responder from '@/components/comentarios/ResponderMinuta.vue'
import Respuestas from '@/components/comentarios/RespuestasMinuta.vue'
import Emision from '@/components/comentarios/NuevaMinuta.vue'
import Semanal from '@/components/semanal/Semanal.vue'
import RevisionSemanal from '@/components/semanal/RevisionSemanal.vue'
import RevisarMinuta from '@/components/comentarios/RevisarMinuta.vue'

import axios from 'axios'
import Auth from '@/services/auth.js'
import Funciones from '@/services/funciones.js'
import { mapState } from 'vuex'

export default {
  name: 'Estudiante',
  components: {
    Minuta,
    Tablero,
    Comentar,
    Responder,
    Respuestas,
    Emision,
    Semanal,
    RevisionSemanal,
    RevisarMinuta
  },
  data () {
    return {
      verFormulario: false,
      tipo: 0,
      seleccionarMinuta: false,
      idBitacora: 0,
      idRevision: 0,
      idComentarios: 0,
      idRespuestas: 0,
      idEmision: 0,
      idVerMinuta: 0,
      crearMinuta: true,
      verRevision: false,
      verComentarios: false,
      verRespuestas: false,
      verEmision: false,
      verSemanal: false,
      verMinuta: false,
      revisarSemanal: false,
      idMotivo: 0,
      nuevaRevision: '',
      esNuevaEmision: false,
      valorActual: 0,
      tableroEst: 0,
      bitacoraAvance: {},
      revisionEstado: ''
    }
  },
  computed: {
    ...mapState(['apiUrl', 'tipoMinutas', 'usuario', 'estudiante', 'grupo', 'motivos']),

    minutasFiltradas: function () {
      var lista = []
      for (var i = 0; i < this.tipoMinutas.length; i++) {
        if (this.tipoMinutas[i].tipo !== 'Semanal') {
          lista.push(this.tipoMinutas[i])
        }
      }
      return lista
    }
  },
  methods: {
    nuevaMinuta: function () {
      this.seleccionarMinuta = true
    },
    cancelarMinuta: function () {
      this.seleccionarMinuta = false
      this.tipo = 0
    },
    elegirTipo: function () {
      if (this.tipo === this.buscarIdTipoMinuta('Semanal')) {
        this.verSemanal = true
        this.seleccionarMinuta = false
      } else {
        this.verFormulario = true
        this.seleccionarMinuta = false
        this.idMotivo = this.buscarIdMotivo('ECI')
        this.nuevaRevision = 'A'
        this.revisionEstado = 'Coordinación de Grupo'
      }
    },
    cerrarFormulario: function () {
      this.verFormulario = false
      this.tipo = 0
      this.idBitacora = 0
      this.esNuevaEmision = false
      this.tableroEst++
    },
    async obtenerTipoMinutas () {
      try {
        const response = await axios.get(this.apiUrl + '/tipo_minutas', { headers: Auth.authHeader() })
        this.$store.commit('setTipoMinutas', response.data)
      } catch {
        console.log('No se han obtenido los tipos de minutas')
      }
    },
    async obtenerEstudiante () {
      try {
        const response = await axios.get(this.apiUrl + '/estudiantes/' + this.usuario.id, { headers: Auth.authHeader() })
        this.$store.commit('setEstudiante', response.data)
        this.obtenerGrupo()
      } catch {
        console.log('No se ha obtenido la infomración del estudiante')
      }
    },
    async obtenerGrupo () {
      try {
        const response = await axios.get(this.apiUrl + '/grupos/' + this.estudiante.grupo_id, { headers: Auth.authHeader() })
        this.$store.commit('setGrupo', response.data)
      } catch {
        console.log('No se ha obtenido la información del grupo')
      }
    },
    async obtenerAprobaciones () {
      try {
        const response = await axios.get(this.apiUrl + '/tipo_aprobaciones', { headers: Auth.authHeader() })
        this.$store.commit('setTipoAprobaciones', response.data)
      } catch (e) {
        console.log(e)
      }
    },
    async obtenerMotivos () {
      try {
        const response = await axios.get(this.apiUrl + '/motivos', { headers: Auth.authHeader() })
        this.$store.commit('setMotivos', response.data)
      } catch {
        console.log('No fue posible obtener los motivos de emisión')
      }
    },
    cambiarTab: function () {
      this.verEmision = false
    },
    establecerBitacora: function (id) {
      this.idBitacora = id
      this.verFormulario = true
      this.verEmision = false
    },
    establecerRevision: function (id) {
      this.idRevision = id
      this.verRevision = true
      this.crearMinuta = false
      this.verEmision = false
    },
    mostrarTablero: function () {
      this.verRevision = false
      this.verComentarios = false
      this.verRespuestas = false
      this.crearMinuta = true
      this.idRevision = 0
      this.idEmision = 0
      this.verEmision = false
      this.valorActual = 0
      this.tableroEst++
    },
    revisarComentarios: function (id) {
      this.idComentarios = id
      this.verComentarios = true
      this.crearMinuta = false
    },
    revisarRespuestas: function (id) {
      this.idRespuestas = id
      this.verRespuestas = true
      this.crearMinuta = false
      this.verEmision = false
    },
    nuevaVersion: function (id) {
      this.idEmision = id
      this.verEmision = true
    },
    revisarAprobacion: function () {
      this.crearMinuta = false
      this.valorActual = 0
    },
    buscarIdMotivo: function (valor) {
      return Funciones.obtenerIdDeLista(this.motivos, 'identificador', valor)
    },
    buscarIdTipoMinuta: function (valor) {
      return Funciones.obtenerIdDeLista(this.tipoMinutas, 'tipo', valor)
    },
    nuevaEmision: function (identificador, revision) {
      this.verRevision = false
      this.verComentarios = false
      this.verEmision = false
      this.crearMinuta = true
      this.verFormulario = true
      this.idRevision = 0
      this.idMotivo = this.buscarIdMotivo(identificador)
      this.revisionesPorEstados(identificador)
      this.nuevaRevision = revision
      this.idBitacora = this.idEmision
      this.esNuevaEmision = true
      this.valorActual = 0
      this.tableroEst++
    },
    cerrarSemanal: function () {
      this.verSemanal = false
      this.mostrarTablero()
    },
    editarAvance: function (bitacora) {
      this.bitacoraAvance = bitacora
      this.verSemanal = true
      this.seleccionarMinuta = false
    },
    revisarAvance: function (bitacora) {
      this.bitacoraAvance = bitacora
      this.revisarSemanal = true
      this.crearMinuta = false
    },
    cerrarAvance: function () {
      this.revisarSemanal = false
      this.mostrarTablero()
    },
    mostrarMinuta: function (id) {
      this.idVerMinuta = id
      this.verMinuta = true
      this.crearMinuta = false
    },
    cerrarMinuta: function () {
      this.verMinuta = false
      this.idVerMinuta = 0
      this.mostrarTablero()
    },
    revisionesPorEstados: function (identificador) {
      this.revisionEstado = Funciones.convertirRevisionAEstado(identificador)
    }
  },
  mounted () {
    if (localStorage.user_tk) {
      this.obtenerTipoMinutas()
      this.obtenerEstudiante()
      this.obtenerAprobaciones()
      this.obtenerMotivos()
    }
  }
}
</script>

<style lang="css" scoped>
.new-section {
  padding: 1rem 1.5rem;
}
</style>
