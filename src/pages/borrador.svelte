<script>
    import { onMount } from 'svelte';
    import { navigate } from 'svelte-routing';
    import axios from 'axios';

    const apiUrl = 'http://localhost:8080';

    let empleados = [];
    let pedidosPendientes = [];
    let pedidosEnProceso = [];
    let pedidosDemorados = [];
    let pedidosCompletados = [];
    let empleadosDisponibles = 0;

    onMount(async() => {
        const pedidosEnProceso = await fetchPedidos();
        await fetchEmpleados(pedidosEnProceso);
    });

    async function fetchPedidos() {
        try {
            const response = await axios.get(`${apiUrl}/pedido`);
            console.log('Pedidos obtenidos:', response.data);
            const pedidos = response.data;

            pedidosPendientes = pedidos.filter(p => p.estado === 'Pendiente');
            pedidosEnProceso = pedidos.filter(p => p.estado === 'Proceso');
            pedidosDemorados = pedidos.filter(p => p.estado === 'Demorado');
            pedidosCompletados = pedidos.filter(p => p.estado === 'Completado');

            return pedidosEnProceso;
        } catch (error) {
            console.error('Error al obtener pedidos:', error);
        }
    }

    async function fetchEmpleados(pedidosEnProceso) {
        try {
            const response = await axios.get(`${apiUrl}/empleado`);
            empleados = response.data;

            // Marcar empleados que están asignados como "deshabilitados"
            empleados.forEach(empleado => {
                empleado.deshabilitado = pedidosEnProceso.some(pedido =>
                    pedido.asignaciones?.some(asignacion => asignacion.empleado.idEmpleado === empleado.idEmpleado)
                );
            });

            empleadosDisponibles = empleados.filter(e => e.asistencia && !e.deshabilitado).length;
        } catch (error) {
            console.error('Error al obtener empleados:', error);
        }
    }

    async function actualizarAsistencia() {
        try {
            for (const empleado of empleados) {
                await axios.put(`${apiUrl}/empleado/${empleado.idEmpleado}`, empleado);
            }
            empleadosDisponibles = empleados.filter(e => e.asistencia).length;
            alert('Asistencia actualizada correctamente.');
            window.location.reload();
        } catch (error) {
            console.error('Error al actualizar asistencia:', error);
        }
    }

    async function finalizarPedido(idPedido) {
        try {
            const pedido = pedidosEnProceso.find(p => p.idPedido === idPedido);
            pedido.estado = 'Completado';
            console.log('Finalizando pedido:', pedido);
            await axios.put(`${apiUrl}/pedido/${idPedido}`, pedido);
            fetchPedidos();
            alert('Pedido finalizado.');
            window.location.reload();
        } catch (error) {
            console.error('Error al finalizar pedido:', error);
        }
    }

    async function reasignarPedidosDemorados() {
        try {
            await axios.put(`${apiUrl}/asignacionEmpleado/reasignar`);
            fetchPedidos();
            alert('Reasignación de empleados completada.');
        } catch (error) {
            console.error('Error al reasignar pedidos demorados:', error);
        }
    }
</script>

<div class="asignacion-empleados">
    <h1>Asignación de Empleados</h1>
    <button on:click={() => navigate('/admin')}>Regresar</button>

    <!-- Gestión de Asistencia de Empleados -->
    <section>
        <h2>Gestión de Asistencia</h2>
        <table>
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Nombre</th>
                    <th>Asistencia</th>
                </tr>
            </thead>
            <tbody>
                {#each empleados as empleado}
                    <tr>
                        <td>{empleado.idEmpleado}</td>
                        <td>{empleado.nombre}</td>
                        <td>
                            <input 
                            type="checkbox" 
                            bind:checked={empleado.asistencia} 
                            disabled={empleado.deshabilitado} 
                            />
                        </td>
                    </tr>
                {/each}
            </tbody>
        </table>
        <button on:click={actualizarAsistencia}>Actualizar Asistencia</button>
    </section>

    <!-- Gestión de Pedidos -->
    <section>
        <h2>Gestión de Pedidos</h2>

        <!-- Pedidos Pendientes -->
        <div>
            <h3>Pedidos Pendientes</h3>
            <ul>
                {#each pedidosPendientes as pedido}
                    <li>Pedido #{pedido.idPedido}: {pedido.detallesPedido.length} muebles pendientes</li>
                {/each}
            </ul>
        </div>

        <!-- Pedidos En Proceso -->
        <div>
            <h3>Pedidos En Proceso</h3>
            {#each pedidosEnProceso as pedido}
                <details>
                    <summary>Pedido #{pedido.idPedido}</summary>
                    <ul>
                        <li><strong>Cliente:</strong> {pedido.cliente.nombre}</li>
                        <li><strong>Fecha Pedido:</strong> {new Date(pedido.fechaPedido).toLocaleDateString()}</li>
                        <li><strong>Estado:</strong> {pedido.estado}</li>
                        <li><strong>Detalles:</strong></li>
                        <ul>
                            {#each pedido.detallesPedido as detalle}
                                <li>
                                    {detalle.cantidad} x {detalle.mueble.nombreMueble} ({detalle.material.nombreMaterial}) - Tiempo: {detalle.tiempoEstimado} horas, Subtotal: ${detalle.precioSubtotal}
                                </li>
                            {/each}
                        </ul>
                        <li><strong>Empleados Asignados:</strong></li>
                        <ul>
                            {#each pedido.asignaciones as asignacion}
                                <li>{asignacion.empleado.nombre} (Asignado el: {new Date(asignacion.fechaAsignacion).toLocaleDateString()})</li>
                            {/each}
                        </ul>
                    </ul>
                    <button on:click={() => finalizarPedido(pedido.idPedido)}>Finalizar Pedido</button>
                </details>
            {/each}
        </div>

        <!-- Pedidos Demorados -->
        <div>
            <h3>Pedidos Demorados</h3>
            <table>
                <thead>
                    <tr>
                        <th>ID Pedido</th>
                        <th>Cliente</th>
                        <th>Detalles</th>
                    </tr>
                </thead>
                <tbody>
                    {#each pedidosDemorados as pedido}
                        <tr>
                            <td>{pedido.idPedido}</td>
                            <td>{pedido.cliente.nombre}</td>
                            <td>
                                {pedido.detallesPedido.map(detalle => 
                                    `${detalle.cantidad} x ${detalle.mueble.nombreMueble}`
                                ).join(', ')}
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>
            <button on:click={reasignarPedidosDemorados}>Reasignar Empleados</button>
        </div>
    </section>

    <!-- Historial y Disponibilidad de Empleados -->
    <section>
        <h2>Historial y Disponibilidad</h2>
        <div>
            <h3>Disponibilidad de Empleados</h3>
            <p>Empleados sin pedidos asignados: {empleadosDisponibles}</p>
        </div>

        <div>
            <h2>Carga de Pedidos para los Empleados</h2>
            <table>
                <thead>
                    <tr>
                        <th>ID Pedido</th>
                        <th>ID Empleado</th>
                        <th>Nombre</th>
                        <th>Pedidos Asignados</th>
                    </tr>
                </thead>
                <tbody>
                    {#each pedidosEnProceso as pedidos}
                        <tr>
                            <td rowspan={pedidos.asignaciones.length}>{ pedidos.idPedido}</td>
                            <td>{ pedidos.asignaciones[0].empleado.idEmpleado}</td>
                            <td>{ pedidos.asignaciones[0].empleado.nombre}</td>
                            <td>{ pedidos.asignaciones[0].empleado.pedidosAsignados}</td>
                        </tr>
                        {#each pedidos.asignaciones.slice(1) as asignacion, index }
                            <tr>
                                <td>{ asignacion.empleado.idEmpleado}</td>
                                <td>{ asignacion.empleado.nombre}</td>
                                <td>{ asignacion.empleado.pedidosAsignados}</td>
                            </tr>
                        {/each}
                    {/each}
                </tbody>
            </table>
        </div>
        <div>
            <h3>Historial de Pedidos Completados</h3>
            <ul>
                {#each pedidosCompletados as pedido}
                    <li>
                        Pedido #{pedido.idPedido}: {pedido.detallesPedido.length} muebles, completado
                    </li>
                {/each}
            </ul>
        </div>
    </section>
</div>

<style>
    .asignacion-empleados {
        padding: 2rem;
    }

    section {
        margin-bottom: 2rem;
        border: 1px solid #ddd;
        padding: 1rem;
        border-radius: 5px;
        background-color: #f9f9f9;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-bottom: 1rem;
    }

    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }

    th {
        background-color: #007bff;
        color: white;
    }

    ul {
        list-style: none;
        padding: 0;
    }

    button {
        background-color: #007bff;
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 5px;
        cursor: pointer;
    }

    button:hover {
        background-color: #0056b3;
    }
</style>
