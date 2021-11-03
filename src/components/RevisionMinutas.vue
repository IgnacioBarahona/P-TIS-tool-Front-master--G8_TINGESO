<template >
  <div>
    <br>

    <div v-if="!mostrarFormulario">

      <SelectorJornada/>

      <SelectorGrupo @eleccion="seleccionarGrupo"/>

      <div v-if="mostrarMinutas">
        <br>
        <div class="columns">
          <div class="column is-10 is-offset-1">
            <div v-if="listaMinutas.length > 0">
              <div class="field">
                <div class="control">
                  <label id="minutas" class="label">Minutas</label>
                </div>
              </div>
              <div >
                <table class="table is-bordered is-fullwidth is-narrow" aria-describedby="minutas">
                  <thead>
                    <tr class="has-background-light">
                      <th scope="col" class="has-text-centered">N°</th>
                      <th scope="col" class="has-text-centered">Código minuta</th>
                      <th scope="col" class="has-text-centered">Tipo de minuta</th>
                      <th scope="col" class="has-text-centered">Fase de revisión</th>
                      <th scope="col" class="has-text-centered">Creada por</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(bitacora, index) in listaMinutas" :key="bitacora.id">
                      <th scope="row" class="has-text-centered">{{ index + 1 }}</th>
                      <td><a @click="traerMinuta(bitacora.id)">{{ bitacora.minuta.codigo }}_{{ bitacora.revision}}</a></td>
                      <td class="has-text-centered">{{ actualizarTipo(bitacora.minuta.tipo) }}</td>
                      <td class="has-text-centered">{{ convertirRevision(bitacora.identificador) }}</td>
                      <td class="has-text-centered">{{ bitacora.minuta.creada_por }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
            <div v-else>
              <br>
              <p class="subtitle is-5">No hay minutas emitidas para revisar</p>
            </div>
          </div>
        </div>
      </div>

    </div>

    <div v-else>

      <Informacion :proyecto="grupoSeleccionado" :minuta="bitacora"/>
      <Objetivos :lista="bitacora.minuta.objetivos"/>
      <Conclusiones :lista="bitacora.minuta.conclusiones"/>
      <Items :lista="bitacora.minuta.items" :asistentes="bitacora.minuta.asistencia" :comentar="false" :responder="false" :lista-com="[]" :ver-respuestas="false"/>

      <br>
      <div class="columns">
        <div class="column is-4 is-offset-2">
          <div class="field">
            <div class="control">
              <a class="button is-primary-usach is-fullwidth" @click="cerrarFormulario">Volver</a>
            </div>
          </div>
        </div>
        <div class="column is-4">
          <div class="field">
            <div class="control">
              <a class="button is-dark is-fullwidth" @click="verRegistros">Ver registro</a>
            </div>
          </div>
        </div>
      </div>

      <br>
      <div v-if="mostrarRegistros">
        <Registros :id-bitacora="bitacora.id" @cerrar="cerrarRegistros"/>
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
import Informacion from '@/components/minutas/Informacion.vue'
import Objetivos from '@/components/minutas/Objetivos.vue'
import Conclusiones from '@/components/minutas/Conclusiones.vue'
import Items from '@/components/minutas/Items.vue'
import Registros from '@/components/RegistroMinuta.vue'

export default {
  name: 'RevisionMinutas',
  components: {
    SelectorJornada,
    SelectorGrupo,
    Informacion,
    Objetivos,
    Conclusiones,
    Items,
    Registros
  },
  data () {
    return {
      mostrarFormulario: false,
      mostrarMinutas: false,
      mostrarRegistros: false,
      grupoSeleccionado: {},
      listaMinutas: [],
      bitacora: {}
    }
  },
  computed: {
    ...mapState(['apiUrl', 'jornadaActual'])
  },
  methods: {
    seleccionarGrupo: function (grupo) {
      this.grupoSeleccionado = grupo
      this.obtenerMinutas(grupo.id)
      this.mostrarMinutas = true
    },
    async obtenerMinutas (grupoId) {
      try {
        const response = await axios.get(this.apiUrl + '/minutas/grupo/' + grupoId, { headers: Auth.authHeader() })
        this.listaMinutas = response.data
      } catch {
        console.log('No fue posible obtener las minutas')
      }
    },
    async traerMinuta (bitacoraId) {
      try {
        const response = await axios.get(this.apiUrl + '/minutas/' + bitacoraId, { headers: Auth.authHeader() })
        this.bitacora = response.data
        this.mostrarFormulario = true
      } catch {
        console.log('No fue posible obtener la información de la minuta seleccionada')
      }
    },
    cerrarFormulario: function () {
      this.mostrarFormulario = false
      this.mostrarRegistros = false
    },
    verRegistros: function () {
      this.mostrarRegistros = true
    },
    cerrarRegistros: function () {
      this.mostrarRegistros = false
    },
    convertirRevision: function (identificador) {
      return Funciones.convertirRevisionAEstado(identificador)
    },
    actualizarTipo: function (tipo) {
      return Funciones.actualizarTipo(tipo)
    }
  },
  watch: {
    jornadaActual: function () {
      this.mostrarMinutas = false
    }
  }
}

</script>
