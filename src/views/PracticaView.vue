<template>
    <div class="about">
        <h1 class="mb-5 mt-5"><i style="color: blueviolet;">Los empleado con vehículos</i></h1>

        <div>
            <div class="mb-3">
                <div class="input-group">
                    <span class="input-group-text" id="type-filter"><i class="bi bi-filter "></i> Filtrar (tipo auto)
                    </span>
                    <select class="form-select" @change="toFilter = $event.target.value">
                        <option value="">Todos</option>
                        <option v-for="tipo_auto in tipos_autos" :value="tipo_auto" :key="tipo_auto">{{ tipo_auto }}
                        </option>
                    </select>
                </div>
            </div>

            <div class="mb-3">
                <div class="input-group">
                    <span class="input-group-text" id="nombre-search-text"><i class="bi bi-search"></i> Buscar en
                        nombre </span>
                    <input type="search" class="form-control" id="nombre-search"
                        @search="this.toSearch = $event.target.value">
                </div>
            </div>
        </div>

        <div class="card">
            <div class="card-body">
                <form @submit.prevent="save()">
                    <table class="table table-striped">
                        <thead>
                            <tr>
                                <th colspan="2">

                                    <input type="text" class="form-control" id="nombre" v-model="newLink.nombre">

                                </th>
                                <th>

                                    <select class="form-select" aria-label="Default select example"
                                        v-model="newLink.area">
                                        <option v-for="area in areas" :value="area" :key="area">{{ area }}</option>
                                    </select>

                                </th>
                                <th>

                                    <select class="form-select" aria-label="Default select example"
                                        v-model="newLink.tipo_auto">
                                        <option v-for="tipo_auto in tipos_autos" :value="tipo_auto" :key="tipo_auto">{{
                                            tipo_auto }}</option>
                                    </select>

                                </th>
                                <th>
                                    <input type="text" class="form-control" id="placa" v-model="newLink.placa">

                                </th>
                                <th>
                                    <input type="text" class="form-control" id="color" v-model="newLink.color">
                                </th>
                                <th>
                                    <button type="submit" class="btn btn-primary">Agregar</button>
                                </th>
                            </tr>
                            <tr>
                                <th scope="col">N°</th>
                                <th scope="col">Nombre</th>
                                <th scope="col">Area</th>
                                <th scope="col">Tipo Vehiculo</th>
                                <th scope="col">Placa</th>
                                <th scope="col">Color</th>
                                <th></th>
                            </tr>
                        </thead>

                        <tbody>
                            <tr v-for="(link, index) in getList()" :key="index">
                                <th scope="row">{{ 1 + index }}</th>
                                <td>{{ link.nombre }}</td>
                                <td>{{ link.area }}</td>
                                <td>{{ link.tipo_auto }}</td>
                                <td>{{ link.placa }}</td>
                                <td>{{ link.color }}</td>

                                <td>
                                    <button type="button" class="btn btn-info btn-sm" @click="edit(index)"><i
                                            class="bi bi-pen"></i></button>

                                    <button type="button" class="btn btn-danger btn-sm" @click="remove(index)"><i
                                            class="bi bi-trash3"></i></button>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </form>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="editModal" tabindex="-1" aria-labelledby="editModalLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-5" id="editModalLabel">Editar</h1>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <form @submit.prevent="saveEdit()">
                        <div class="modal-body" v-if="itemSelected">
                            <div class="mb-3">
                                <label for="updateDescription" class="form-label">Nombre</label>
                                <input type="text" v-model="itemSelected.nombre" class="form-control" id="updatenombre">
                            </div>

                            <div class="mb-3">
                                <label for="updateArea" class="form-label">Area</label>
                                <select class="form-select" v-model="itemSelected.area" id="updatearea">
                                    <option v-for="area in areas" :value="area">{{ area }}</option>
                                </select>
                            </div>
                            <div class="mb-3">
                                <label for="updateTipoAuto" class="form-label">Tipo Auto</label>
                                <select class="form-select" v-model="itemSelected.tipo_auto" id="updatetipo_auto">
                                    <option v-for="tipo_auto in tipos_autos" :value="tipo_auto">{{ tipo_auto }}</option>
                                </select>
                            </div>
                            <div class="mb-3">
                                <label for="updatePlaca" class="form-label">Placa</label>
                                <input type="text" v-model="itemSelected.placa" class="form-control" id="updatePlaca">
                            </div>
                            <div class="mb-3">
                                <label for="updateColor" class="form-label">Color</label>
                                <input type="text" v-model="itemSelected.color" class="form-control" id="updateColor">
                            </div>

                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancelar</button>
                            <button type="submit" class="btn btn-primary">Guardar cambios</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
export default {
    data() {
        return {
            areas: ['pruduccion', 'finanzas', 'contabilidad'],
            tipos_autos: ['automovil', 'moto'],
            newLink: {
                nombre: '',
                area: '',
                tipo_auto: '',
                placa: '',
                color: ''
            },
            linksArray: [],
            itemSelected: null,
            indexSelected: null,
            toFilter: '',
            toSearch: ''
        };
    },
    mounted() {
        this.loadData();
    },
    methods: {
        // 🔹 Cargar datos desde JSON Server
        async loadData() {
            try {
                const response = await fetch("http://localhost:3000/empleados");
                this.linksArray = await response.json();
            } catch (error) {
                console.error("Error cargando los datos:", error);
            }
        },

        // 🔹 Agregar un nuevo empleado
        async save() {
            if (this.newLink.nombre && this.newLink.area && this.newLink.tipo_auto && this.newLink.placa && this.newLink.color) {
                try {
                    const response = await fetch("http://localhost:3000/empleados", {
                        method: "POST",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify(this.newLink)
                    });
                    const newEmpleado = await response.json();
                    this.linksArray.push(newEmpleado); // Agregar a la lista sin recargar
                    this.newLink = { nombre: '', area: '', tipo_auto: '', placa: '', color: '' };
                } catch (error) {
                    console.error("Error guardando el empleado:", error);
                }
            } else {
                alert("Por favor, completa todos los campos.");
            }
        },

        // 🔹 Eliminar un empleado
        async remove(index) {
            if (confirm("¿Está seguro de eliminar este ítem?")) {
                try {
                    const id = this.linksArray[index].id;
                    await fetch(`http://localhost:3000/empleados/${id}`, { method: "DELETE" });
                    this.linksArray.splice(index, 1);
                } catch (error) {
                    console.error("Error eliminando el empleado:", error);
                }
            }
        },
        edit(index) {
            this.itemSelected = { ...this.linksArray[index] };
            this.indexSelected = index;
            const editModal = new bootstrap.Modal(document.getElementById('editModal'));
            editModal.show();
        },

        // 🔹 Editar un empleado
        async saveEdit() {
            try {
                const id = this.itemSelected.id;
                await fetch(`http://localhost:3000/empleados/${id}`, {
                    method: "PUT",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(this.itemSelected)
                });

                // Actualizar la lista en el frontend sin recargar
                const index = this.linksArray.findIndex(emp => emp.id === id);
                this.linksArray[index] = { ...this.itemSelected };

                // Cerrar el modal
                const modalElement = document.getElementById('editModal');
                const modalInstance = bootstrap.Modal.getInstance(modalElement) || new bootstrap.Modal(modalElement);
                modalInstance.hide();

                this.itemSelected = null;
                this.indexSelected = null;
            } catch (error) {
                console.error("Error editando el empleado:", error);
            }
        },
        getList() {
            return this.linksArray.filter((item) => {
                return (!this.toSearch || item.nombre.includes(this.toSearch)) &&
                    (!this.toFilter || item.tipo_auto === this.toFilter);
            });
        }

    }
};

</script>

<style>
.btn {
    margin-right: 3px;
}

.card {
    overflow-x: auto;
}
</style>