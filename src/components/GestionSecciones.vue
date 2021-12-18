<template>
<div>
    <br>
    <!-- Botones -->
    <div class="columns">
        <div class="column is-8"></div>
        <div class="column is-4">
            <div class="field is-grouped is-grouped-right">
                <p class="control">
                    <a class="button is-light-usach">Ayuda</a>
                </p>
                <p class="control" v-if="verFormulario">
                    <a class="button is-info-usach">Asignar estudiante</a>
                </p>
            </div>
        </div>
    </div>
    <br>
    <!-- Mostrar Formulario -->
    <div v-if="verFormulario">
      <form class="form" method="post">
        <div class="columns has-text-left">
          <div class="column is-6">
            <div class="field">
              <label class="label">Jornada:</label>
              <div class="control">
                <input v-model="seccion.jornada.nombre" class="input" type="text" disabled>
              </div>
            </div>
          </div>
          <div class="column is-6">
            <div class="field">
              <label class="label">Código:</label>
              <div class="control">
                <input v-model="seccion.codigo" class="input" type="text" disabled>
              </div>
            </div>
          </div>
        </div>
        <div class="columns has-text-left">
          <div class="column is-6">
            <div class="field">
              <label class="label">Semestre:</label>
              <div class="control">
                <input v-model="seccion.semestre.numero" class="input" type="text" disabled>
              </div>
            </div>
          </div>
          <div class="column is-6">
            <div class="field">
              <label class="label">Curso:</label>
              <div class="control">
                <input v-model="seccion.curso.nombre" class="input" type="text" disabled>
              </div>
            </div>
          </div>
        </div>
        <div class="columns has-text-left">
          <div class="column is-6">
            <div class="field">
              <label class="label">Año:</label>
              <div class="control">
                <input v-model="seccion.semestre.agno" class="input" type="text" disabled>
              </div>
            </div>
          </div>
        </div>
        <div class="columns">
          <div class="column is-12">
            <div class="field is-grouped is-grouped-centered">
              <div class="control">
                <a class="button is-primary-usach">Actualizar</a>
              </div>
              <div class="control">
                <a class="button is-light-usach">Cancelar</a>
              </div>
            </div>
          </div>
        </div>
        <div v-if="mostrarListaEstudiantesSeccion">
          <div class="has-text-left">
            <label class="label">Estudiantes de la sección:</label>
          </div>
          <table class="table is-bordered is-narrow is-fullwidth" summary="Estudiantes">
            <thead>
              <tr class="has-background-light">
                <th scope="col" class="has-text-centered">N°</th>
                <th scope="col" class="has-text-centered">R.U.N</th>
                <th scope="col" class="has-text-centered">Nombre estudiante</th>
                <th scope="col" class="has-text-centered">Correo electrónico</th>
                <th scope="col" class="has-text-centered"><input type="checkbox"></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(estudiante, index) in listaEstudiantesSeccion" :key="estudiante.id" >
                <th scope="row" class="has-text-centered">{{index + 1}}</th>
                <td class="has-text-centered">{{visualizarRun(estudiante.run_est)}}</td>
                <td class="has-text-centered">{{nombreCompleto(estudiante)}}</td>
                <td class="has-text-centered">{{estudiante.correo}}</td>
                <td class="has-text-centered"><input type="checkbox"></td>
              </tr>
            </tbody>
          </table>
        </div>
      </form>
    </div>
    <!-- Mostrar Secciones -->
    <hr>
    <div v-if="mostrarLista">
        <table class="table is-bordered is-narrow is-fullwidth" summary="Secciones">
            <thead>
                <tr class="has-background-light">
                    <th scope="col" class="has-text-centered">N°</th>
                    <th scope="col" class="has-text-centered">Jornada</th>
                    <th scope="col" class="has-text-centered">Código</th>
                    <th scope="col" class="has-text-centered">Semestre</th>
                    <th scope="col" class="has-text-centered">Curso</th>
                    <th scope="col" class="has-text-centered"></th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(seccion, index) in listaSecciones" :key="seccion.id">
                    <th scope="row" class="is-vcentered has-text-centered">{{ index + 1 }}</th>
                    <td class="is-vcentered has-text-centered">{{seccion.jornada.nombre}}</td>
                    <td class="is-vcentered has-text-centered">{{seccion.codigo}}</td>
                    <td class="is-vcentered has-text-centered">{{seccion.semestre.numero}}/{{seccion.semestre.agno}}</td>
                    <td class="is-vcentered has-text-centered">{{seccion.curso.nombre}}</td>
                    <td class="has-text-centered"><a class="button is-primary-usach" @click="cargarSeccion(seccion); obtenerEstudiantesSeccion(seccion)">Editar</a></td>

                </tr>
            </tbody>

        </table>
    </div>
</div>
</template>

<script>
import Auth from '@/services/auth.js'
import Funciones from '@/services/funciones.js'

import { mapState } from 'vuex'
import axios from 'axios'

export default {
  name: 'GestionSecciones',
  data () {
    return {
      help: false,
      verFormulario: false,
      seccion: {
        id: 0,
        codigo: '',
        jornada: {
          nombre: ''
        },
        semestre: {
          numero: null,
          agno: null
        },
        curso: {
          nombre: ''
        }
      },
      listaSecciones: [],
      listaEstudiantesSeccion: [],
      mostrarLista: false,
      mostrarListaEstudiantesSeccion: false,
      faqs_open: []
    }
  },
  computed: {
    ...mapState(['apiUrl', 'secciones', 'faqsProfesor'])
  },
  methods: {
    async obtenerSecciones () {
      try {
        const secciones = await axios.get(this.apiUrl + 'profesor/secciones/mostrar_secciones', { headers: Auth.authHeader() })
        this.listaSecciones = secciones.data
        if (Object.keys(this.listaSecciones).length > 0) {
          this.mostrarLista = true
        } else {
          this.mostrarLista = false
        }
      } catch (error) {
        console.log(error)
      }
    },
    async obtenerAyuda () {
      try {
        const response = await axios.get(this.apiUrl + '/faqs/profesor/estudiante', { headers: Auth.authHeader() })
        this.$store.commit('setFaqsProfesor', response.data)
      } catch {
        console.log('No fue posible obtener las faqs')
      }
    },
    async cargarSeccion (seccion) {
      this.seccion.id = seccion.id
      this.seccion.codigo = seccion.codigo
      this.seccion.semestre.numero = seccion.semestre.numero
      this.seccion.semestre.agno = seccion.semestre.agno
      this.seccion.jornada.nombre = seccion.jornada.nombre
      this.seccion.curso.nombre = seccion.curso.nombre
      this.verFormulario = true
      this.mostrarListaEstudiantesSeccion = true
    },
    async obtenerEstudiantesSeccion (seccion) {
      try {
        const estudiantes = await axios.get(this.apiUrl + 'profesor/secciones/mostrar_secciones/' + seccion.id, { headers: Auth.authHeader() })
        this.listaEstudiantesSeccion = estudiantes.data
        if (Object.keys(this.listaEstudiantesSeccion).length > 0) {
          this.mostrarListaEstudiantesSeccion = true
        } else {
          this.mostrarListaEstudiantesSeccion = false
        }
      } catch (error) {
        console.log(error)
      }
    },
    nombreCompleto: function (estudiante) {
      return estudiante.nombre_est + ' ' + estudiante.apellido1 + ' ' + estudiante.apellido2
    },
    visualizarRun: function (run) {
      return Funciones.visualizarRun(run)
    }
  },
  created () {
    if (localStorage.user_tk) {
      this.obtenerSecciones()
      this.obtenerAyuda()
    }
  }
}
</script>
