<template>
  <div>
    <div class="modal" :class="help ? 'is-active': ''" >
      <div class="modal-background"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title title is-4">Revisión de avances</p>
          <button class="delete" aria-label="close" @click="modificarModal"></button>
        </header>
        <section class="modal-card-body">
         <div class="content" v-for="faq in faqsProfesor" :key="faq.id">
            <h2 class="title is-5">{{faq.pregunta}}</h2>
            <p>{{faq.respuesta}}</p>
          </div>
        </section>
      </div>
    </div>

    <br>

    <div v-if="!revisarMinuta">
      <div class="columns">
        <div class="column is-8"></div>
        <div class="column is-4" >
          <div class="control">
            <div class="field is-grouped is-grouped-right">
              <p class="control">
                <a class="button is-light-usach" @click="modificarModal">Ayuda</a>
              </p>
            </div>
          </div>
        </div>
      </div>

      <br>
      <SelectorJornada/>

      <SelectorGrupo @eleccion="seleccionarGrupo"/>

      <br>
      <br>
      <div v-if="grupoElegido">
        <div class="columns">
          <div class="column is-10 is-offset-1">
            <div v-if="mostrarMinutas">
              <div class="field">
                <div class="control">
                  <label id="avances" class="label">Minutas de avance semanal</label>
                </div>
              </div>
              <div>
                <table class="table is-bordered is-fullwidth is-narrow" aria-describedby="avances">
                  <thead>
                    <tr class="has-background-light">
                      <th scope="col" class="has-text-centered">N°</th>
                      <th scope="col" class="has-text-centered">Código minuta</th>
                      <th scope="col" class="has-text-centered">Sprint</th>
                      <th scope="col" class="has-text-centered">Creada el</th>
                      <th scope="col" class="has-text-centered">Emitida el</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(bitacora, index) in listaFiltrada" :key="bitacora.id">
                      <th class="has-text-centered" scope="row">{{ index + 1 }}</th>
                      <td><a @click="revisarAvance(bitacora)">{{ bitacora.minuta.codigo }}</a></td>
                      <td class="has-text-centered">{{ bitacora.minuta.numero_sprint }}</td>
                      <td class="has-text-centered">{{ convertirFecha(bitacora.minuta.creada_el) }}</td>
                      <td class="has-text-centered">{{ convertirFecha(bitacora.fecha_emision) }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
            <div v-else>
              <br>
              <p class="subtitle is-5 has-text-centered">No hay minutas de avance semanal para revisar en este grupo</p>
            </div>
          </div>
        </div>
      </div>

    </div>

    <div v-else>

      <br>
      <RevisionSemanal :grupo="grupoSeleccionado" :minuta="bitacora"/>

      <div class="columns is-centered">
        <div class="column is-5">
          <div class="field">
            <div class="control">
              <button class="button is-primary-usach is-fullwidth" @click="cerrarRevision">Volver</button>
            </div>
          </div>
        </div>
      </div>

    </div>

  </div>
</template>

<script>
import Auth from '@/services/auth.js'
import Funciones from '@/services/funciones.js'
import axios from 'axios'
import { mapState } from 'vuex'

import SelectorJornada from '@/components/SelectorJornada.vue'
import SelectorGrupo from '@/components/SelectorGrupo.vue'
import RevisionSemanal from '@/components/semanal/RevisionSemanal.vue'

export default {
  name: 'RevisionAvances',
  components: {
    SelectorJornada,
    SelectorGrupo,
    RevisionSemanal
  },
  data () {
    return {
      help: false,
      grupoSeleccionado: {},
      listaAvances: [],
      revisarMinuta: false,
      bitacora: {}
    }
  },
  computed: {
    ...mapState(['apiUrl', 'jornadaActual', 'faqsProfesor']),

    grupoElegido: function () {
      return Object.keys(this.grupoSeleccionado).length > 0
    },
    mostrarMinutas: function () {
      return this.listaFiltrada.length > 0
    },
    listaFiltrada: function () {
      var lista = []
      for (var i = 0; i < this.listaAvances.length; i++) {
        if (this.listaAvances[i].minuta.bitacora_estado.tipo_estado.abreviacion === 'CER') {
          lista.push(this.listaAvances[i])
        }
      }
      return lista
    }
  },
  methods: {
    convertirFecha: function (timestamp) {
      return Funciones.obtenerFecha(timestamp)
    },
    seleccionarGrupo: function (grupo) {
      this.grupoSeleccionado = grupo
      this.obtenerAvances()
    },
    async obtenerAvances () {
      try {
        const response = await axios.get(this.apiUrl + '/minutas/avances/semanales/grupo/' + this.grupoSeleccionado.id, { headers: Auth.authHeader() })
        this.listaAvances = response.data
      } catch (e) {
        console.log('No se han obtenido las minutas de avance del grupo seleccionado')
        console.log(e)
      }
    },
    async obtenerAyuda () {
      try {
        const response = await axios.get(this.apiUrl + '/faqs/profesor/avances', { headers: Auth.authHeader() })
        this.$store.commit('setFaqsProfesor', response.data)
      } catch {
        console.log('No fue posible obtener las faqs')
      }
    },
    revisarAvance: function (bitacora) {
      this.bitacora = bitacora
      this.revisarMinuta = true
    },
    cerrarRevision: function () {
      this.revisarMinuta = false
      this.grupoSeleccionado = {}
      this.listaAvances = []
      this.bitacora = {}
    },
    modificarModal: function () {
      if (!this.help) {
        this.help = true
      } else {
        this.help = false
      }
    }
  },
  created () {
    if (localStorage.user_tk) {
      this.obtenerAyuda()
    }
  },
  watch: {
    jornadaActual: function () {
      this.grupoSeleccionado = {}
    }
  }
}
</script>
