<script>
    import { onMount } from 'svelte';
    import axios from 'axios';
    import { navigate } from 'svelte-routing';

    // Variables
    let cliente = null;
    let pedidos = [];
    let ultimoPedido = null;
    const apiUrl = 'https://backendmuebles-production.up.railway.app';

    // Función para cargar los pedidos del cliente
    async function fetchPedidosCliente() {
        try {
            const response = await axios.get(`${apiUrl}/pedido/cliente/${cliente.idCliente}`);
            pedidos = response.data;

            // Establecer el último pedido del cliente
            if (pedidos.length > 0) {
                ultimoPedido = pedidos[pedidos.length - 1];
            }
        } catch (error) {
            console.error('Error al obtener los pedidos del cliente:', error);
        }
    }

    // Función para cerrar sesión
    function cerrarSesion() {
        localStorage.removeItem('cliente');
        navigate('/loginCliente');
    }

    // Cargar cliente desde el almacenamiento local y obtener pedidos al montar el componente
    onMount(() => {
        const storedCliente = localStorage.getItem('cliente');
        if (storedCliente) {
            cliente = JSON.parse(storedCliente);
            fetchPedidosCliente();
        } else {
            navigate('/login-cliente');
        }
    });
</script>

<div class="pedidos-cliente">
    <!-- Barra superior -->
    <nav class="barra-superior">
        <button on:click={() => navigate('/cliente')}>InicioCliente</button>
        <button on:click={() => navigate('/pedidosCliente')}>PedidosCliente</button>
        <button on:click={cerrarSesion}>Cerrar Sesión</button>
    </nav>

    <!-- Último pedido -->
    {#if ultimoPedido}
        <div class="ultimo-pedido">
            <h3>Último Pedido</h3>
            <p><strong>ID de Orden:</strong> {ultimoPedido.idPedido}</p>
            <p><strong>Fecha:</strong> {new Date(ultimoPedido.fechaPedido).toLocaleDateString()}</p>
            <p><strong>Estado:</strong> {ultimoPedido.estado}</p>
            <p><strong>Costo Total:</strong> ${ultimoPedido.precioTotal}</p>
        </div>
    {/if}

    <!-- Tabla de pedidos -->
    <section class="tabla-pedidos">
        <h2>Órdenes Realizadas</h2>
        <table>
            <thead>
                <tr>
                    <th>ID de Orden</th>
                    <th>Mueble</th>
                    <th>Material</th>
                    <th>Cantidad</th>
                    <th>Tiempo Estimado</th>
                    <th>Costo</th>
                    <th>Estado</th>
                    <th>Tiempo Estimado Total</th>
                    <th>Costo Total</th>
                </tr>
            </thead>
            <tbody>
                {#each pedidos as pedido}
                    <!-- Calcular cuántas filas tiene cada pedido -->
                    <tr>
                        <td rowspan={pedido.detallesPedido.length}>{pedido.idPedido}</td>
                        <td>{pedido.detallesPedido[0].mueble.nombreMueble}</td>
                        <td>{pedido.detallesPedido[0].material.nombreMaterial}</td>
                        <td>{pedido.detallesPedido[0].cantidad}</td>
                        <td>{pedido.detallesPedido[0].tiempoEstimado} horas</td>
                        <td>${pedido.detallesPedido[0].precioSubtotal}</td>
                        <td rowspan={pedido.detallesPedido.length}>{pedido.estado}</td>
                        <td rowspan={pedido.detallesPedido.length}>{pedido.tiempoTotalEstimado} horas</td>
                        <td rowspan={pedido.detallesPedido.length}>${pedido.precioTotal}</td>
                    </tr>
                    {#each pedido.detallesPedido.slice(1) as detalle, index}
                        <tr>
                            <td>{detalle.mueble.nombreMueble}</td>
                            <td>{detalle.material.nombreMaterial}</td>
                            <td>{detalle.cantidad}</td>
                            <td>{detalle.tiempoEstimado} horas</td>
                            <td>${detalle.precioSubtotal}</td>
                        </tr>
                    {/each}
                {/each}
            </tbody>
        </table>
        
        <button class="volver-inicio" on:click={() => navigate('/cliente')}>Volver al Inicio</button>
    </section>
</div>

<style>
    .pedidos-cliente {
        padding: 2rem;
    }

    .barra-superior {
        display: flex;
        justify-content: space-between;
        margin-bottom: 2rem;
        background-color: #f0f0f0;
        padding: 1rem;
        border-bottom: 2px solid #ddd;
    }

    .barra-superior button {
        background-color: #007bff;
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 5px;
        cursor: pointer;
    }

    .barra-superior button:hover {
        background-color: #0056b3;
    }

    .ultimo-pedido {
        background-color: #f9f9f9;
        padding: 1rem;
        border: 1px solid #ddd;
        border-radius: 5px;
        margin-bottom: 2rem;
    }

    .tabla-pedidos {
        margin-top: 2rem;
        padding: 1rem;
        border: 1px solid #ddd;
        background-color: #fff;
        border-radius: 5px;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 20px;
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

    tr:nth-child(even) {
        background-color: #f9f9f9;
    }


    .volver-inicio {
        margin-top: 1rem;
        background-color: #28a745;
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 5px;
        cursor: pointer;
    }

    .volver-inicio:hover {
        background-color: #218838;
    }
</style>
