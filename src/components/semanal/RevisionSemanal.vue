<template>
  <div>

    <br>
    <InfoAvance :grupo="grupoSeleccionado" :minuta="bitacora"/>
    <br>

    <div v-for="estudiante in grupoSeleccionado.estudiantes" :key="estudiante.id">
      <VisorEstudiante :est="estudiante" :logros="logrosPorEstudiante(estudiante.id)" :metas="metasPorEstudiante(estudiante.id)" :impedimentos="impedimentosPorEstudiante(estudiante.id)"/>
      <br>
    </div>

  </div>
</template>

<script>
import InfoAvance from '@/components/semanal/InfoAvance.vue'
import VisorEstudiante from '@/components/semanal/VisorEstudiante.vue'

import Funciones from '@/services/funciones.js'

export default {
  name: 'RevisionSemanal',
  components: {
    InfoAvance,
    VisorEstudiante
  },
  props: ['grupo', 'minuta'],
  data () {
    return {
      grupoSeleccionado: this.grupo,
      bitacora: this.minuta,
      itemsLogros: [],
      itemsMetas: [],
      itemsImpedimentos: []
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
    }
  },
  mounted () {
    if (this.mostrarBitacora) {
      this.separarItems(this.bitacora.minuta.items)
    }
  }
}
</script>
