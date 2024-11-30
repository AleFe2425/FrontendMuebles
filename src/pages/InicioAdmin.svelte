<script>
    import axios from 'axios';
    import { navigate } from 'svelte-routing';


    let activeSection = ''; // Por defecto, la sección activa es "pedidos"
    let pedidos = [];
    let newPedido = {
        cliente: null,
        fechaPedido: '',
        detallesPedido: []
    };
    let selectedPedido = null;
    let isEditingPedido = false;
    
    let muebles = [];
    let newMueble = { nombreMueble: '', precioMueble: 0, tiempoBaseProduccion: 0 };
    let selectedMueble = null;
    let isEditingMueble = false;

    let materiales = [];
    let newMaterial = { nombreMaterial: '', precioMaterial: 0, factorTiempo: 0 };
    let selectedMaterial = null;
    let isEditingMaterial = false;

    let clientes = [];
    let newCliente = { nombre: '', email: '', telefono: '', password: '' };
    let selectedCliente = null;
    let isEditingCliente = false;

    let empleados = [];
    let newEmpleado = { nombre: '', asistencia: false, pedidosAsignados: 0 };
    let selectedEmpleado = null;
    let isEditingEmpleado = false;

    const apiUrl = 'https://backendmuebles-production.up.railway.app'; // URL base del backend desplegado

    function setActiveSection(section) {
        activeSection = section;
        switch (section) {
            case 'pedidos':
                fetchPedidos();
                fetchClientes();
                fetchMateriales();
                fetchMuebles();
                break;
            case 'muebles':
                fetchMuebles();
                break;
            case 'materiales':
                fetchMateriales();
                break;
            case 'clientes':
                fetchClientes();
                break;
            case 'empleados':
                fetchEmpleados();
                break;
        }
    }

    //Funciones para Pedidos
    //Cargar todos los pedidos
    async function fetchPedidos() {
        try {
            const response = await axios.get(`${apiUrl}/pedido`);
            pedidos = response.data;
        } catch (error) {
            console.error('Error al obtener los pedidos:', error);
        }
    }

    //Agregar un nuevo pedido
    async function addPedido() {
        if (newPedido.detallesPedido.length === 0) {
            alert('Debe agregar al menos un detalle al pedido.');
            return;
        }

        try {
            const response = await axios.post(`${apiUrl}/pedido`, newPedido);
            pedidos = [...pedidos, response.data];
            newPedido = { cliente: null, fechaPedido: '', detallesPedido: [] }; // Resetear formulario
        } catch (error) {
            console.error('Error al agregar pedido:', error);
        }
    }
    //Contenedor de detalle del Pedido
    function handleViewPedido(pedido) {
        selectedPedido = pedido;
        isEditingPedido = false;
    }
    //Contenedor de edicion del Pedido
    function handleEditPedido(pedido) {
        selectedPedido = pedido;
        isEditingPedido = true; // Mostrar el formulario de edición
    }
    //Editar un Pedido
    async function editPedido() {
        try {
            const response = await axios.put(`${apiUrl}/pedido/${selectedPedido.idPedido}`, selectedPedido);
            pedidos = pedidos.map(p => (p.idPedido === response.data.idPedido ? response.data : p));
            selectedPedido = null; // Limpiar la selección después de editar
            isEditingPedido = false;
        } catch (error) {
            console.error('Error al editar pedido:', error);
        }
    }
    //Cancelar Edicion de un Pedido
    function cancelEditPedido() {
        isEditingPedido = false; // Ocultar el formulario de edición
        selectedPedido = null; // Limpiar la selección si es necesario
    }
    //Eliminar un Pedido
    async function handleDeletePedido(id) {
        try {
            await axios.delete(`${apiUrl}/pedido/${id}`);
            pedidos = pedidos.filter(p => p.idPedido !== id);
            selectedPedido = null;
        } catch (error) {
            console.error('Error al eliminar pedido:', error);
        }
    }
    //Agrega un nuevo detalle al pedido
    function addDetallePedido() {
        newPedido.detallesPedido = [
            ...newPedido.detallesPedido,
            {
                cantidad: 1,
                mueble: null,
                materiale: null
            }
        ];
    }
    //Elimina un detalle al pedido
    function removeDetallePedido(index) {
        newPedido.detallesPedido = newPedido.detallesPedido.filter((_,i) => i !==index);
    }


    //Funciones para Muebles
    //Cargar todos los muebles
    async function fetchMuebles() {
        try {
            const response = await axios.get(`${apiUrl}/mueble`);
            muebles = response.data;
        } catch (error) {
            console.error('Error al obtener los muebles:', error);
        }
    }
    //Agregar un nuevo mueble
    async function addMueble() {
        try {
            const response = await axios.post(`${apiUrl}/mueble`, newMueble);
            muebles = [...muebles, response.data];
            newMueble = { nombreMueble: '', precioMueble: 0, tiempoBaseProduccion: 0 }; // Resetear formulario
        } catch (error) {
            console.error('Error al agregar mueble:', error);
        }
    }
    //Contenedor de detalles del Mueble
    function handleViewMueble(mueble) {
        selectedMueble = mueble;
        isEditingMueble = false;
    }
    //Contenendor de edicion del Mueble
    function handleEditMueble(mueble) {
        selectedMueble = mueble;
        isEditingMueble = true; // Mostrar el formulario de edición
    }
    //Editar un Mueble
    async function editMueble() {
        try {
            const response = await axios.put(`${apiUrl}/mueble/${selectedMueble.idMueble}`, selectedMueble);
            muebles = muebles.map(m => (m.idMueble === response.data.idMueble ? response.data : m));
            selectedMueble = null; // Limpiar la selección después de editar
            isEditingMueble = false;
        } catch (error) {
            console.error('Error al editar mueble:', error);
        }
    }
    //Cancelar Edicion de un Mueble
    function cancelEditMueble() {
        isEditingMueble = false; // Ocultar el formulario de edición
        selectedMueble = null; // Limpiar la selección si es necesario
    }
    //Eliminar un Mueble
    async function handleDeleteMueble(id) {
        try {
            await axios.delete(`${apiUrl}/mueble/${id}`);
            muebles = muebles.filter(m => m.idMueble !== id);
            selectedMueble = null;
        } catch (error) {
            console.error('Error al eliminar mueble:', error);
        }
    }


    //Funciones para Materiales
    //Cargar todos los Materiales
    async function fetchMateriales() {
        try {
            const response = await axios.get(`${apiUrl}/material`);
            materiales = response.data;
        } catch (error) {
            console.error('Error al obtener los materiales:', error);
        }
    }
    //Agregar un nuevo Material
    async function addMaterial() {
        try {
            const response = await axios.post(`${apiUrl}/material`, newMaterial);
            materiales = [...materiales, response.data];
            newMaterial = { nombreMaterial: '', precioMaterial: 0, factorTiempo: 0 }; // Resetear formulario
        } catch (error) {
            console.error('Error al agregar material:', error);
        }
    }
    //Contenedor de detalles del Material
    function handleViewMaterial(material) {
        selectedMaterial = material;
        isEditingMaterial = false;
    }
    //Contenendor de edicion del Material
    function handleEditMaterial(material) {
        selectedMaterial = material;
        isEditingMaterial = true; // Mostrar el formulario de edición
    }
    //Editar un Material
    async function editMaterial() {
        try {
            const response = await axios.put(`${apiUrl}/material/${selectedMaterial.idMaterial}`, selectedMaterial);
            materiales = materiales.map(m => (m.idMaterial === response.data.idMaterial ? response.data : m));
            selectedMaterial = null; // Limpiar la selección después de editar
            isEditingMaterial = false;
        } catch (error) {
            console.error('Error al editar material:', error);
        }
    }
    //Cancelar Edicion de un Material
    function cancelEditMaterial() {
        isEditingMaterial = false; // Ocultar el formulario de edición
        selectedMaterial = null; // Limpiar la selección si es necesario
    }
    //Eliminar un Material
    async function handleDeleteMaterial(id) {
        try {
            await axios.delete(`${apiUrl}/material/${id}`);
            materiales = materiales.filter(m => m.idMaterial !== id);
            selectedMaterial = null;
        } catch (error) {
            console.error('Error al eliminar material:', error);
        }
    }

    //Funciones para Clientes
    //Cargar todos los Clientes
    async function fetchClientes() {
        try {
            const response = await axios.get(`${apiUrl}/cliente`);
            clientes = response.data;
        } catch (error) {
            console.error('Error al obtener los clientes:', error);
        }
    }
    //Agregar un nuevo cliente
    async function addCliente() {
        try {
            const response = await axios.post(`${apiUrl}/cliente/registrar`, newCliente);
            clientes = [...clientes, response.data];
            newCliente = { nombre: '', email: '', telefono: '', password: '' }; // Resetear formulario
            fetchClientes()
        } catch (error) {
            console.error('Error al agregar cliente:', error);
        }
    }
    //Contenendor de detalles del Cliente
    function handleViewCliente(cliente) {
        selectedCliente = cliente;
        isEditingCliente = false;
    }
    //Contenendor de edicion del Cliente
    function handleEditCliente(cliente) {
        selectedCliente = cliente;
        isEditingCliente = true; // Mostrar el formulario de edición
    }
    //Editar un Mueble
    async function editCliente() {
        try {
            const response = await axios.put(`${apiUrl}/cliente/${selectedCliente.idCliente}`, selectedCliente);
            clientes = clientes.map(m => (m.idCliente === response.data.idCliente ? response.data : m));
            selectedCliente = null; // Limpiar la selección después de editar
            isEditingCliente = false;
        } catch (error) {
            console.error('Error al editar ciente:', error);
        }
    }
    //Cancelar Edicion de un Ciente
    function cancelEditCiente() {
        isEditingMueble = false; // Ocultar el formulario de edición
        selectedMueble = null; // Limpiar la selección si es necesario
    }
    //Eliminar un Ciente
    async function handleDeleteCliente(email) {
        try {
            await axios.delete(`${apiUrl}/cliente/${email}`);
            clientes = clientes.filter(c => c.email !== email);
            selectedCliente = null;
        } catch (error) {
            console.error('Error al eliminar cliente:', error);
        }
    }

    //Funciones para Empleados
    //Cargar todos los Empleados
    async function fetchEmpleados() {
        try {
            const response = await axios.get(`${apiUrl}/empleado`);
            empleados = response.data;
        } catch (error) {
            console.error('Error al obtener los empleados:', error);
        }
    }
    //Agregar un nuevo Empleado
    async function addEmpleado() {
        try {
            const response = await axios.post(`${apiUrl}/empleado`, newEmpleado);
            empleados = [...empleados, response.data];
            newEmpleado = { nombre: '', asistencia: false, pedidosAsignados: 0 }; // Resetear formulario
        } catch (error) {
            console.error('Error al agregar empleado:', error);
        }
    }
    //Contenedor de detalles del Empleado
    function handleViewEmpleado(empleado) {
        selectedEmpleado = empleado;
        isEditingEmpleado = false;
    }
    //Contenendor de edicion del Mueble
    function handleEditEmpleado(empleado) {
        selectedEmpleado = empleado;
        isEditingEmpleado = true; // Mostrar el formulario de edición
    }
    //Editar un Mueble
    async function editEmpleado() {
        try {
            const response = await axios.put(`${apiUrl}/empleado/${selectedEmpleado.idEmpleado}`, selectedEmpleado);
            empleados = empleados.map(m => (m.idEmpleado === response.data.idEmpleado ? response.data : m));
            selectedEmpleado = null; // Limpiar la selección después de editar
            isEditingEmpleado = false;
        } catch (error) {
            console.error('Error al editar empleado:', error);
        }
    }
    //Cancelar Edicion de un Empleado
    function cancelEditEmpleado() {
        isEditingEmpleado = false; // Ocultar el formulario de edición
        selectedEmpleado = null; // Limpiar la selección si es necesario
    }
    //Eliminar un Empleado
    async function handleDeleteEmpleado(id) {
        try {
            await axios.delete(`${apiUrl}/empleado/${id}`);
            empleados = empleados.filter(e => e.idEmpleado !== id);
            selectedEmpleado = null;
        } catch (error) {
            console.error('Error al eliminar empleado:', error);
        }
    }

    function cerrarSesion() {
        navigate('/');
    }
</script>



<main>
    <h1>Gestión del Sistema</h1>
    <button on:click={cerrarSesion}>Cerrar Sesión</button>
    <nav class="navbar">
        <button on:click={() => setActiveSection('pedidos')}>Gestión de Pedidos</button>
        <button on:click={() => setActiveSection('muebles')}>Gestión de Muebles</button>
        <button on:click={() => setActiveSection('materiales')}>Gestión de Materiales</button>
        <button on:click={() => setActiveSection('clientes')}>Gestión de Clientes</button>
        <button on:click={() => setActiveSection('empleados')}>Gestión de Empleados</button>
        <button on:click={() => navigate('/admin/asignar-empleados')}>Asingacion de Empleados</button>

    </nav>

    {#if activeSection === 'pedidos'}
        <section>
            <h2>Gestión de Pedidos</h2>
            <table>
                <thead>
                    <tr>
                        <th>Id Pedido</th>
                        <th>Cliente</th>
                        <th>Fecha</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {#each pedidos as pedido}
                        <tr>
                            <td>{pedido.idPedido}</td>
                            <td>{pedido.cliente.nombre}</td>
                            <td>{new Date (pedido.fechaPedido).toLocaleDateString()}</td>
                            <td>{pedido.estado}</td>
                            <td>
                                <button on:click={() => handleViewPedido(pedido)}>Ver Detalles</button>
                                <button on:click={() => handleEditPedido(pedido)}>Editar</button>
                                <button on:click={() => handleDeletePedido(pedido.idPedido)}>Eliminar</button>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>

            {#if selectedPedido && !isEditingPedido}
                <div class="object-details">
                    <h3>Detalles del Pedido</h3>
                    <pre>{JSON.stringify(selectedPedido, null, 2)}</pre>
                    <button on:click={() => (selectedPedido = null)}>Cerrar Detalles</button>
                </div>
            {/if}

            {#if selectedPedido && isEditingPedido}
                <div class="form-container">
                    <h3>Editar Pedido</h3>
                    <form on:submit|preventDefault={editPedido}>
                        <label for="editTiempoTotalEstimado">Tiempo Estimado de Produccion:</label>
                        <input type="number" bind:value={selectedPedido.tiempoTotalEstimado} id="editTiempoTotalEstimado" required>
                        
                        <label for="editEstado">Estado:</label>
                        <select bind:value={selectedPedido.estado} id="editEstado" required>
                            <option value="Proceso">Proceso</option>
                            <option value="Finalizado">Finalizado</option>
                            <option value="Demorado">Demorado</option>
                        </select>

                        <button type="submit">Guardar Cambios</button>
                        <button type="button" on:click={() => cancelEditPedido()}>Cancelar</button>
                    </form>
                </div>
            {/if}

            <div class="form-container">
                <h3>Agregar Pedido</h3>
                <form on:submit|preventDefault={addPedido}>
                    <label for="clienteSelect">Cliente:</label>
                    <select bind:value={newPedido.cliente} id="clienteSelect" required>
                        <option value="" disabled selected>Seleccione un cliente</option>
                        {#each clientes as cliente}
                            <option value={cliente}>{cliente.nombre}</option>
                        {/each}
                    </select>

                    <label for="fechaPedido">Fecha del Pedido:</label>
                    <input type="date" bind:value={newPedido.fechaPedido} id="fechaPedido" required>

                    <h4>Detalles del Pedido</h4>
                    {#each newPedido.detallesPedido as detalle, index}
                        <div class="detalle">
                            <label for={`muebleSelect-${index}`}>Mueble:</label>
                            <select bind:value={detalle.mueble} id={`muebleSelect-${index}`} required>
                                <option value="" disabled selected>Seleccione un mueble</option>
                                {#each muebles as mueble}
                                    <option value={mueble}>{mueble.nombreMueble}</option>
                                {/each}
                            </select>

                            <label for={`materialSelect-${index}`}>Material:</label>
                            <select bind:value={detalle.material} id={`materialSelect-${index}`} required>
                                <option value="" disabled selected>Seleccione un material</option>
                                {#each materiales as material}
                                    <option value={material}>{material.nombreMaterial}</option>
                                {/each}
                            </select>

                            <label for={`cantidad-${index}`}>Cantidad:</label>
                            <input type="number" min="1" bind:value={detalle.cantidad} id={`cantidad-${index}`} required>

                            <button type="button" on:click={() => removeDetallePedido(index)}>Eliminar Detalle</button>
                        </div>
                    {/each}

                    <button type="button" on:click={addDetallePedido}>Agregar Detalle</button>
                    <button type="submit">Agregar Pedido</button>
                </form>
            </div>
        </section>
    {/if}

    {#if activeSection === 'muebles'}
        <section>

            <!--Contenedor de Muebles -->
            <h2>Gestión de Muebles</h2>
            <table>
                <thead>
                    <tr>
                        <th>Id Mueble</th>
                        <th>Nombre</th>
                        <th>Precio</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {#each muebles as mueble}
                        <tr>
                            <td>{mueble.idMueble}</td>
                            <td>{mueble.nombreMueble}</td>
                            <td>{mueble.precioMueble}</td>
                            <td>
                                <button on:click={() => handleViewMueble(mueble)}>Ver Detalles</button>
                                <button on:click={() => handleEditMueble(mueble)}>Editar</button>
                                <button on:click={() => handleDeleteMueble(mueble.idMueble)}>Eliminar</button>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>

            {#if selectedMueble && !isEditingMueble }
            <div class="object-details">
                <h3>Detalles del Mueble</h3>
                <pre>{JSON.stringify(selectedMueble, null, 2)}</pre>
                <button on:click={() => (selectedMueble = null)}>Cerrar Detalles</button>
            </div>
            {/if}

            {#if selectedMueble && isEditingMueble}
                <div class="form-container">
                    <h3>Editar Mueble</h3>
                    <form on:submit|preventDefault={editMueble}>
                        <label for="editNombreMueble">Nombre:</label>
                        <input type="text" bind:value={selectedMueble.nombreMueble} id="editNombreMueble" required>
                       
                        <label for="editPrecioMueble">Precio:</label>
                        <input type="number" step="0.01" bind:value={selectedMueble.precioMueble} id="editPrecioMueble" required>
                       
                        <label for="editTiempoBaseProduccion">Tiempo de Produccion:</label>
                        <input type="number" step="0.01" bind:value={selectedMueble.tiempoBaseProduccion} id="editTiempoBaseProduccion" required>
                        
                        <button type="submit">Guardar Cambios</button>
                        <button type="button" on:click={() => cancelEditMueble()}>Cancelar</button>
                    </form>
                </div>
            {/if}

            <div class="form-container">
                <h3>Agregar Mueble</h3>
                <form on:submit|preventDefault={addMueble}>
                    <label for="nombre">Nombre:</label>
                    <input type="text" bind:value={newMueble.nombreMueble} id="nombre" required>
                   
                    <label for="precio">Precio:</label>
                    <input type="number" step="0.01" bind:value={newMueble.precioMueble} id="precio" required>
                   
                    <label for="tiempoBaseProduccion">Tiempo de Produccion:</label>
                    <input type="number" step="0.01" bind:value={newMueble.tiempoBaseProduccion} id="tiempoBaseProduccion" required>
                   
                    <button type="submit">Agregar Mueble</button>
                </form>
            </div>
        </section>
    {/if}

    {#if activeSection === 'materiales'}
        <section>
            <h2>Gestión de Materiales</h2>
            <table>
                <thead>
                    <tr>
                        <th>Id Material</th>
                        <th>Nombre</th>
                        <th>Precio</th>
                        <th>Factor de Tiempo</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {#each materiales as material}
                        <tr>
                            <td>{material.idMaterial}</td>
                            <td>{material.nombreMaterial}</td>
                            <td>{material.precioMaterial}</td>
                            <td>{material.factorTiempo}</td>
                            <td>
                                <button on:click={() => handleViewMaterial(material)}>Ver Detalles</button>
                                <button on:click={() => handleEditMaterial(material)}>Editar</button>
                                <button on:click={() => handleDeleteMaterial(material.idMaterial)}>Eliminar</button>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>

            {#if selectedMaterial && !isEditingMaterial }
                <div class="object-details">
                    <h3>Detalles del Material</h3>
                    <pre>{JSON.stringify(selectedMaterial, null, 2)}</pre>
                    <button on:click={() => (selectedMaterial = null)}>Cerrar Detalles</button>
                </div>
            {/if}
            
            {#if selectedMaterial && isEditingMaterial}
                <div class="form-container">
                    <h3>Editar Material</h3>
                    <form on:submit|preventDefault={editMaterial}>
                        <label for="editNombreMaterial">Nombre:</label>
                        <input type="text" bind:value={selectedMaterial.nombreMaterial} id="editNombreMaterial" required>
                       
                        <label for="editPrecioMaterial">Precio:</label>
                        <input type="number" step="0.01" bind:value={selectedMaterial.precioMaterial} id="editPrecioMaterial" required>
                        
                        <label for="editFactorTiempo">Factor de Tiempo de Produccion:</label>
                        <input type="number" step="0.01" bind:value={selectedMaterial.factorTiempo} id="editFactorTiempo" required>
                        
                        <button type="submit">Guardar Cambios</button>
                        <button type="button" on:click={() => cancelEditMaterial()}>Cancelar</button>
                    </form>
                </div>
            {/if}

            <div class="form-container">
                <h3>Agregar Material</h3>
                <form on:submit|preventDefault={addMaterial}>
                    <label for="nombreMaterial">Nombre:</label>
                    <input type="text" bind:value={newMaterial.nombreMaterial} id="nombreMaterial" required>
                    
                    <label for="precioMaterial">Precio:</label>
                    <input type="number" step="0.01" bind:value={newMaterial.precioMaterial} id="precioMaterial" required>
                    
                    <label for="factorTiempo">Factor de Tiempo:</label>
                    <input type="number" step="0.01" bind:value={newMaterial.factorTiempo} id="factorTiempo" required>
                    
                    <button type="submit">Agregar Material</button>
                </form>
            </div>
        </section>
    {/if}

    {#if activeSection === 'clientes'}
        <section>
            <h2>Gestión de Clientes</h2>
            <table>
                <thead>
                    <tr>
                        <th>Id Cliente</th>
                        <th>Nombre</th>
                        <th>Email</th>
                        <th>Teléfono</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {#each clientes as cliente}
                        <tr>
                            <td>{cliente.idCliente}</td>
                            <td>{cliente.nombre}</td>
                            <td>{cliente.email}</td>
                            <td>{cliente.telefono}</td>
                            <td>
                                <button on:click={() => handleViewCliente(cliente)}>Ver Detalles</button>
                                <button on:click={() => handleEditCliente(cliente)}>Editar</button>
                                <button on:click={() => handleDeleteCliente(cliente.email)}>Eliminar</button>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>

            {#if selectedCliente && !isEditingCliente}
                <div class="object-details">
                    <h3>Detalles del Cliente</h3>
                    <pre>{JSON.stringify(selectedCliente, null, 2)}</pre>
                    <button on:click={() => (selectedCliente = null)}>Cerrar Detalles</button>
                </div>
            {/if}

            {#if selectedCliente && isEditingCliente}
                <div class="form-container">
                    <h3>Editar Cliente</h3>
                    <form on:submit|preventDefault={editCliente}>
                        <label for="editNombreCliente">Nombre:</label>
                        <input type="text" bind:value={selectedCliente.nombre} id="editNombreCliente" required>
                        
                        <label for="editEmailCliente">Email:</label>
                        <input type="text" bind:value={selectedCliente.email} id="editEmailCliente" required>
                        
                        <label for="editTelefonoCliente">Telefono:</label>
                        <input type="text" bind:value={selectedCliente.telefono} id="editTelefonoCliente" required>
                        
                        <button type="submit">Guardar Cambios</button>
                        <button type="button" on:click={() => cancelEditMueble()}>Cancelar</button>
                    </form>
                </div>
            {/if}

            <div class="form-container">
                <h3>Agregar Cliente</h3>
                <form on:submit|preventDefault={addCliente}>
                    <label for="nombreCliente">Nombre:</label>
                    <input type="text" bind:value={newCliente.nombre} id="nombreCliente" required>
                    
                    <label for="emailCliente">Email:</label>
                    <input type="email" bind:value={newCliente.email} id="emailCliente" required>
                    
                    <label for="telefonoCliente">Teléfono:</label>
                    <input type="text" bind:value={newCliente.telefono} id="telefonoCliente" required>
                    
                    <label for="passwordCliente">Password:</label>
                    <input type="password" bind:value={newCliente.password} id="passwordCliente" required>
                    
                    <button type="submit">Agregar Cliente</button>
                </form>
            </div>
        </section>
    {/if}

    {#if activeSection === 'empleados'}
        <section>
            <h2>Gestión de Empleados</h2>
            <table>
                <thead>
                    <tr>
                        <th>Id Empleado</th>
                        <th>Nombre</th>
                        <th>Asistencia</th>
                        <th>Pedidos Asignados</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {#each empleados as empleado}
                        <tr>
                            <td>{empleado.idEmpleado}</td>
                            <td>{empleado.nombre}</td>
                            <td>{empleado.asistencia ? 'Sí' : 'No'}</td>
                            <td>{empleado.pedidosAsignados}</td>
                            <td>
                                <button on:click={() => handleViewEmpleado(empleado)}>Ver Detalles</button>
                                <button on:click={() => handleEditEmpleado(empleado)}>Editar</button>
                                <button on:click={() => handleDeleteEmpleado(empleado.idEmpleado)}>Eliminar</button>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>

            {#if selectedEmpleado && !isEditingEmpleado}
                <div class="object-details">
                    <h3>Detalles del Empleado</h3>
                    <pre>{JSON.stringify(selectedEmpleado, null, 2)}</pre>               
                    <button on:click={() => (selectedEmpleado = null)}>Cerrar Detalles</button>
                </div>
            {/if}

            {#if selectedMueble && isEditingMueble}
                <div class="form-container">
                    <h3>Editar Mueble</h3>
                    <form on:submit|preventDefault={editMueble}>
                        <label for="editNombreMueble">Nombre:</label>
                        <input type="text" bind:value={selectedMueble.nombreMueble} id="editNombreMueble" required>
                        
                        <label for="editPrecioMueble">Precio:</label>
                        <input type="number" step="0.01" bind:value={selectedMueble.precioMueble} id="editPrecioMueble" required>
                        
                        <label for="editTiempoBaseProduccion">Tiempo de Produccion:</label>
                        <input type="number" step="0.01" bind:value={selectedMueble.tiempoBaseProduccion} id="editTiempoBaseProduccion" required>
                        
                        <button type="submit">Guardar Cambios</button>
                        <button type="button" on:click={() => cancelEditMueble()}>Cancelar</button>
                    </form>
                </div>
            {/if}

            {#if selectedEmpleado && isEditingEmpleado}
                <div class="form-container">
                    <h3>Editar Empleado</h3>
                    <form on:submit|preventDefault={editEmpleado}>
                        <label for="editNombreEmpleado">Nombre:</label>
                        <input type="text" bind:value={selectedEmpleado.nombre} id="editNombreEmpleado" required>

                        <label for="editAsistenciaEmpleado">Asistencia:</label>
                        <input type="checkbox" bind:checked={selectedEmpleado.asistencia} id="editAsistenciaEmpleado">

                        <label for="editPedidosAsignadosEmpleado">Pedidos Asignados:</label>
                        <input type="number" step="1" bind:value={selectedEmpleado.pedidosAsignados} id="editPedidosAsignadosEmpleado" required>

                        <button type="submit">Guardar Cambios</button>
                        <button type="button" on:click={() => cancelEditEmpleado()}>Cancelar</button>
                    </form>
                </div>
            {/if}
            <div class="form-container">
                <h3>Agregar Empleado</h3>
                <form on:submit|preventDefault={addEmpleado}>
                    <label for="nombreEmpleado">Nombre:</label>
                    <input type="text" bind:value={newEmpleado.nombre} id="nombreEmpleado" required>
                    
                    <label for="asistenciaEmpleado">Asistencia:</label>
                    <input type="checkbox" bind:checked={newEmpleado.asistencia} id="asistenciaEmpleado">
                    
                    <label for="pedidosAsignados">Pedidos Asignados:</label>
                    <input type="number" bind:value={newEmpleado.pedidosAsignados} id="pedidosAsignados" required>
                    <button type="submit">Agregar Empleado</button>
                </form>
            </div>
        </section>
    {/if}

</main>

<style>
    main {
        padding: 2rem;
    }

    .navbar {
        display: flex;
        justify-content: space-around;
        margin-bottom: 2rem;
        background-color: #f0f0f0;
        padding: 1rem;
        border-bottom: 2px solid #ddd;
    }

    .navbar button {
        background-color: #007bff;
        color: #fff;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 5px;
        cursor: pointer;
    }

    .navbar button:hover {
        background-color: #0056b3;
    }

    section {
        margin-top: 2rem;
        border: 1px solid #ddd;
        padding: 1rem;
        background-color: #fff;
    }

    h1 {
        text-align: center;
        margin-bottom: 1rem;
    }
</style>
