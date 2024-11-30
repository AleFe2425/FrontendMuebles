<script>
    import { onMount } from 'svelte';
    import axios from 'axios';
    import { navigate } from 'svelte-routing';
    import { loop_guard } from 'svelte/internal';

    let cliente = null;
    let muebles = [];
    let materiales = [];
    let detallesPedido = [];
    let fechaPedido = new Date().toISOString().split("T")[0]; // Fecha actual
    let errorMessage = '';

    const apiUrl = 'https://backendmuebles-production.up.railway.app';

    onMount(() => {
        const storedCliente = localStorage.getItem('cliente');
        if (storedCliente) {
            cliente = JSON.parse(storedCliente);
            fetchMuebles();
            fetchMateriales();
        } else {
            navigate('/login-cliente');
        }
    });

    async function fetchMuebles() {
        try {
            const response = await axios.get(`${apiUrl}/mueble`);
            muebles = response.data;
        } catch (error) {
            console.error('Error al obtener muebles:', error);
        }
    }

    async function fetchMateriales() {
        try {
            const response = await axios.get(`${apiUrl}/material`);
            materiales = response.data;
        } catch (error) {
            console.error('Error al obtener materiales:', error);
        }
    }

    function addDetallePedido() {
        detallesPedido = [
            ...detallesPedido,
            { cantidad: 1, mueble: '', material: '' },
        ];
    }

    function removeDetallePedido(index) {
        detallesPedido = detallesPedido.filter((_, i) => i !== index);
    }

    async function realizarPedido() {
        if (detallesPedido.some(d => !d.mueble || !d.material || d.cantidad <= 0)) {
            errorMessage = 'Por favor, completa todos los campos de cada detalle.';
            return;
        }

        const pedido = {
            cliente,
            fechaPedido,
            detallesPedido: detallesPedido.map(detalle => ({
                cantidad: detalle.cantidad,
                mueble: muebles.find(m => m.idMueble == detalle.mueble),
                material: materiales.find(m => m.idMaterial == detalle.material),
            })),
        };

        try {
            await axios.post(`${apiUrl}/pedido`, pedido);
            alert('Pedido realizado con éxito.');
            detallesPedido = [];
            navigate('/cliente/pedidos');
        } catch (error) {
            console.error('Error al realizar el pedido:', error);
            errorMessage = 'Hubo un error al realizar el pedido. Inténtalo de nuevo.';
        }
    }

    function cerrarSesion() {
        localStorage.removeItem('cliente');
        navigate('/login-cliente');
    }
</script>

<div class="inicio-cliente">
    <!-- Barra superior -->
    <nav class="barra-superior">
        <button on:click={() => navigate('/cliente')}>InicioCliente</button>
        <button on:click={() => navigate('/cliente/pedidos')}>PedidosCliente</button>
        <button on:click={cerrarSesion}>Cerrar Sesión</button>
    </nav>

    <!-- Formulario de pedido -->
    <section class="pedido-section">
        <h2>Crear Pedido</h2>
        <div class="detalles-pedido">
            {#each detallesPedido as detalle, index}
                <div class="detalle">
                    <img src="../static/logo.png" alt="Logo de la empresa de muebles" />
                    <div class="inputs">
                        <select bind:value={detalle.mueble} required>
                            <option value="" disabled selected>Seleccionar Mueble</option>
                            {#each muebles as mueble}
                                <option value={mueble.idMueble}>{mueble.nombreMueble}</option>
                            {/each}
                        </select>
                        <select bind:value={detalle.material} required>
                            <option value="" disabled selected>Seleccionar Material</option>
                            {#each materiales as material}
                                <option value={material.idMaterial}>{material.nombreMaterial}</option>
                            {/each}
                        </select>
                        <input
                            type="number"
                            min="1"
                            bind:value={detalle.cantidad}
                            placeholder="Cantidad"
                            required
                        />
                    </div>
                    <button on:click={() => removeDetallePedido(index)}>Eliminar</button>
                </div>
            {/each}
            <button on:click={addDetallePedido}>Agregar Producto</button>
        </div>

        {#if errorMessage}
            <p class="error-message">{errorMessage}</p>
        {/if}

        <button class="realizar-pedido" on:click={realizarPedido}>Realizar Pedido</button>
    </section>
</div>

<style>
    .inicio-cliente {
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

    .pedido-section {
        margin-top: 2rem;
        padding: 1rem;
        border: 1px solid #ddd;
        background-color: #fff;
    }

    .detalles-pedido {
        display: flex;
        flex-wrap: wrap;
        gap: 1rem;
    }

    .detalles-pedido button {
        width: 150px; /* Ancho fijo */
        align-self: center; /* Centrar en la columna */
        background-color: #007bff;
        color: white;
        border: none;
        padding: 0.5rem;
        border-radius: 5px;
        cursor: pointer;
    }

    .detalles-pedido button:hover {
        background-color: #0056b3;
    }


    .detalle {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
        border: 1px solid #ccc;
        border-radius: 5px;
        background-color: #f9f9f9;
        width: 30%;
        box-sizing: border-box;
    }

    .detalle img {
        width: 100px;
        height: 100px;
        background-color: #ccc;
        margin-bottom: 1rem;
    }

    .detalle .inputs {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
    }

    .realizar-pedido {
        margin-top: 2rem;
        background-color: #28a745;
        color: white;
        border: none;
        padding: 1rem 2rem;
        border-radius: 5px;
        cursor: pointer;
    }

    .realizar-pedido:hover {
        background-color: #218838;
    }

    .error-message {
        color: red;
    }
</style>
