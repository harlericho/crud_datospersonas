<template>
<div id="app">
    <!-- Button trigger modal -->
    <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#modelId" ref="modalAbrir" v-on:click="reset()">
        Añadir
    </button>

    <!-- Modal -->
    <div class="modal fade" id="modelId" tabindex="-1" role="dialog" aria-labelledby="modelTitleId" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Modal registro</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <form method="post" v-on:submit.prevent="guardar()">
                    <div class="modal-body">
                        <input type="hidden" id="id" v-model="persona.id" />
                        <div class="mb-3">
                            <label for="dni" class="form-label">Dni</label>
                            <input type="text" maxlength="10" class="form-control" ref="dni" placeholder="0999999999" v-model="persona.dni" required />
                        </div>
                        <div class="mb-3">
                            <label for="nombres" class="form-label">Nombres</label>
                            <input type="text" class="form-control" ref="nombres" placeholder="nombres" v-model="persona.nombres" required />
                        </div>
                        <div class="mb-3">
                            <label for="fecha" class="form-label">Fecha Nacimiento</label>
                            <input type="date" class="form-control" ref="fecha" min="1980-01-01" max="2021-12-31" v-model="persona.fecha" required />
                        </div>
                        <div class="mb-3">
                            <label for="correo" class="form-label">Correo</label>
                            <input type="email" placeholder="xyz@email.com" class="form-control" ref="correo" v-model="persona.correo" required />
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary btn-sm" data-bs-dismiss="modal" ref="modalCerrar">
                            Cerrar
                        </button>
                        <button type="submit" class="btn btn-primary btn-sm">
                            Guardar
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
    <table class="table table-bordered table-responsive mt-3">
        <thead>
            <tr>
                <th>#</th>
                <th>Dni</th>
                <th>Nombres</th>
                <th>Fechas Nacimientos</th>
                <th>Edades</th>
                <th colspan="2">Acciones</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(listado, i) in listados" :key="listado.id">
                <td scope="row">{{ i + 1 }}</td>
                <td>{{ listado.dni }}</td>
                <td>{{ listado.nombres }}</td>
                <td id="fecha">{{ listado.fecha }}</td>
                <td id="correo">{{ listado.correo }}</td>
                <td>
                    <button type="button" class="btn btn-primary btn-sm" v-on:click="obtenerPersona(listado.id)">
                        Editar
                    </button>
                </td>
                <td>
                    <button type="button" class="btn btn-secondary btn-sm" v-on:click="eliminarPersona(listado.id)">
                        Eliminar
                    </button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
</template>

<script>
const url = "http://localhost:5000/";
import axios from "axios";
import moment from "moment";
export default {
    name: "app",
    data() {
        return {
            persona: {
                id: '',
                dni: '',
                nombres: '',
                fecha: '',
                correo: ''
            },
            listados: [],
        };
    },
    mounted() {
        this.listadoPersona();
    },
    methods: {
        soloNumeros(dni) {
            var c = 0
            for (let index = 0; index < dni.length; index++) {
                if (dni[index] >= 0 && dni[index] <= 9) {
                    c = c + 1
                } else {
                    c = c - 1
                }
            }
            if (c === dni.length) {
                return true
            } else {
                return false
            }
        },
        listadoPersona() {
            axios
                .get(url)
                .then((res) => {
                    this.listados = res.data.datos;
                })
                .catch((err) => {
                    console.error(err);
                });
        },
        guardar() {
            const params = {
                dni: this.persona.dni,
                nombres: this.persona.nombres,
                fechaNacimiento: moment(this.persona.fecha).format(
                    "YYYY-MM-DD"
                ),
                correo: this.persona.correo
            }
            if (this.soloNumeros(this.persona.dni)) {
                if (this.persona.id == '') {
                    this.guardarPersona(params);
                } else {
                    this.actualizarPersona(params, this.persona.id);
                }
            } else {
                alert('Dni solo debe tener numeros del 0 - 9')
                this.$refs.dni.focus();
            }
        },
        guardarPersona(params) {
            axios
                .post(url, params)
                .then((res) => {
                    if (res.data.dni) {
                        alert(res.data.dni);
                        this.$refs.dni.focus();
                    } else if (res.data.correo) {
                        alert(res.data.correo);
                        this.$refs.correo.focus();
                    } else {
                        alert(res.data.datos);
                        this.$refs.modalCerrar.click();
                        this.listadoPersona();
                    }

                })
                .catch((err) => {
                    console.error(err);
                });
        },
        actualizarPersona(params, id) {
            axios
                .put(url + id, params)
                .then((res) => {
                    console.log(res);
                    if (res.data.dni) {
                        alert(res.data.dni);
                        this.$refs.dni.focus();
                    } else if (res.data.correo) {
                        alert(res.data.correo);
                        this.$refs.correo.focus();
                    } else {
                        alert(res.data.datos);
                        this.$refs.modalCerrar.click();
                        this.listadoPersona();
                    }

                })
                .catch((err) => {
                    console.error(err);
                });
        },
        reset() {
            this.persona.id = ''
            this.persona.dni = ''
            this.persona.nombres = ''
            this.persona.fecha = ''
            this.persona.correo = ''
        },
        obtenerPersona(id) {
            this.$refs.modalAbrir.click();
            axios
                .get(url + id)
                .then((res) => {
                    this.persona.id = res.data.datos.id;
                    this.persona.dni = res.data.datos.dni;
                    this.persona.nombres = res.data.datos.nombres;
                    //document.querySelector("input[type=date]").value = res.data.datos.fecha
                    this.persona.fecha = res.data.datos.fecha;
                    this.persona.correo = res.data.datos.correo;
                })
                .catch((err) => {
                    console.error(err);
                });
        },
        eliminarPersona(id) {
            if (confirm("¿Desea eliminar el registro?")) {
                axios.delete(url + id)
                    .then(res => {
                        alert(res.data.datos)
                        this.listadoPersona()
                    })
                    .catch(err => {
                        console.error(err);
                    })
            }
        }
    },
};
</script>

<style>
.table {
    text-align: center;
}
#fecha{
color:blueviolet
}
#correo{
    color:darkseagreen;
    font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;

}
</style>
