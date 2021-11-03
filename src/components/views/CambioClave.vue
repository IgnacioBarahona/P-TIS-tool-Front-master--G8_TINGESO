<template>
  <div>

    <div class="columns">

      <div class="column is-6 is-offset-3">

        <div class="field has-text-left">
          <label class="label">Contraseña actual:</label>
        </div>
        <div class="field">
          <div class="control has-icons-left">
            <input class="input" v-model="actual" :class="{ 'is-danger' : entradas.actual.error }" @input="limpiarActual" type="password" placeholder="Contraseña actual">
            <span class="icon is-small is-left">
              <i class="fas fa-id-card" aria-hidden="true"></i>
            </span>
            <p class="help is-danger" v-if="entradas.actual.error">{{ entradas.actual.mensaje }}</p>
          </div>
        </div>

        <div class="field has-text-left">
          <label class="label">Nueva contraseña:</label>
        </div>
        <div class="field">
          <div class="control has-icons-left">
            <input class="input" v-model="nueva" :class="{ 'is-danger' : entradas.nueva.error }" @input="limpiarNueva" type="password" placeholder="Nueva contraseña">
            <span class="icon is-small is-left">
              <i class="fas fa-key" aria-hidden="true"></i>
            </span>
            <p class="help is-danger" v-if="entradas.nueva.error">{{ entradas.nueva.mensaje }}</p>
          </div>
        </div>

        <div class="field has-text-left">
          <label class="label">Repita nueva contraseña:</label>
        </div>
        <div class="field">
          <div class="control has-icons-left">
            <input class="input" v-model="repetirNueva" :class="{ 'is-danger' : entradas.repetir.error }" @input="limpiarRepetir" type="password" placeholder="Repita nueva contraseña">
            <span class="icon is-small is-left">
              <i class="fas fa-lock" aria-hidden="true"></i>
            </span>
            <p class="help is-danger" v-if="entradas.repetir.error">{{ entradas.repetir.mensaje }}</p>
          </div>
        </div>

        <div class="field mt-6 is-grouped is-grouped-centered">
          <div class="control">
            <a class="button is-primary-usach" @click="cambiarClave">Cambiar contraseña</a>
          </div>
          <div class="control">
            <a class="button is-light-usach" @click="cancelarCambio">Cancelar</a>
          </div>
        </div>

      </div>
    </div>

    <div class="column is-3"></div>

  </div>
</template>

<script>
import axios from 'axios'
import Auth from '@/services/auth.js'
import { mapState } from 'vuex'

export default {
  name: 'CambioClave',
  data () {
    return {
      actual: '',
      nueva: '',
      repetirNueva: '',
      entradas: {
        actual: {
          error: false,
          mensaje: ''
        },
        nueva: {
          error: false,
          mensaje: ''
        },
        repetir: {
          error: false,
          mensaje: ''
        }
      }
    }
  },
  computed: {
    ...mapState(['apiUrl', 'usuario'])
  },
  methods: {
    validarActual: function () {
      if (this.actual !== undefined) {
        if (this.actual !== '') {
          this.entradas.actual.error = false
          this.entradas.actual.mensaje = ''
          return true
        } else {
          this.entradas.actual.error = true
          this.entradas.actual.mensaje = 'No se ha ingresado la clave actual'
          return false
        }
      } else {
        this.entradas.actual.error = true
        this.entradas.actual.mensaje = 'Este campo no puede estar vacío'
        return false
      }
    },
    validarNueva: function () {
      if (this.nueva !== undefined) {
        if (this.nueva !== '') {
          this.entradas.nueva.error = false
          this.entradas.nueva.mensaje = ''
          return true
        } else {
          this.entradas.nueva.error = true
          this.entradas.nueva.mensaje = 'No se ha ingresado la nueva clave'
          return false
        }
      } else {
        this.entradas.nueva.error = true
        this.entradas.nueva.mensaje = 'Este campo no puede estar vacío'
        return false
      }
    },
    validarPass: function () {
      if (this.nueva !== undefined) {
        if (this.repetirNueva !== undefined) {
          if (this.nueva !== '') {
            if (this.repetirNueva !== '') {
              if (this.nueva === this.repetirNueva) {
                this.entradas.nueva.error = false
                this.entradas.repetir.error = false
                return true
              } else {
                this.entradas.nueva.error = true
                this.entradas.nueva.mensaje = ''
                this.entradas.repetir.error = true
                this.entradas.repetir.mensaje = 'Las contraseñas no coinciden'
                return false
              }
            } else {
              this.entradas.repetir.error = true
              this.entradas.repetir.mensaje = 'No se ha reingresado la nueva contraseña'
              return false
            }
          } else {
            this.entradas.nueva.error = true
            this.entradas.nueva.mensaje = 'No se ha ingresado la nueva contraseña'
            return false
          }
        } else {
          this.entradas.repetir.error = true
          this.entradas.repetir.mensaje = 'Falta reingresar la nueva contraseña'
          return false
        }
      } else {
        this.entradas.nueva.error = true
        this.entradas.nueva.mensaje = 'Falta ingresar la nueva contraseña'
        return false
      }
    },
    validarFormulario: function () {
      var validar = true
      validar = validar && this.validarActual()
      validar = validar && this.validarNueva()
      validar = validar && this.validarPass()
      return validar
    },
    async autenticar () {
      try {
        await Auth.login(this.usuario.email, this.actual)
        return true
      } catch {
        this.entradas.actual.error = true
        this.entradas.actual.mensaje = 'La clave ingresada no es válida'
        return false
      }
    },
    async cambiarClave () {
      if (this.validarFormulario()) {
        if (await this.autenticar()) {
          const usuario = {
            password: this.actual,
            usuario: {
              password: this.nueva,
              password_confirmation: this.repetirNueva
            }
          }
          try {
            await axios.patch(this.apiUrl + '/usuarios/' + this.usuario.id, usuario, { headers: Auth.postHeader() })
            await Auth.login(this.usuario.email, this.nueva)
            this.redirigirUsuario()
          } catch (e) {
            console.log('No se ha podido cambiar la clave de acceso')
            console.log(e)
          }
        }
      }
    },
    limpiarActual: function () {
      this.entradas.actual.error = false
      this.entradas.actual.mensaje = ''
    },
    limpiarNueva: function () {
      this.entradas.nueva.error = false
      this.entradas.nueva.mensaje = ''
    },
    limpiarRepetir: function () {
      this.entradas.repetir.error = false
      this.entradas.repetir.mensaje = ''
    },
    redirigirUsuario () {
      if (this.usuario.rol.rango === 1) {
        this.$router.push({ path: '/coordinador' })
      } else if (this.usuario.rol.rango === 2) {
        this.$router.push({ path: '/profesor' })
      } else if (this.usuario.rol.rango === 3) {
        this.$router.push({ path: '/estudiante' })
      } else if (this.usuario.rol.rango === 4) {
        this.$router.push({ path: '/cliente' })
      } else {
        this.$router.push({ path: '/' })
      }
    },
    cancelarCambio: function () {
      this.redirigirUsuario()
      this.actual = ''
      this.nueva = ''
      this.repetirNueva = ''
      this.limpiarActual()
      this.limpiarNueva()
      this.limpiarRepetir()
    }
  }
}
</script>
