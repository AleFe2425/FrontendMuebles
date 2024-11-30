<script>
    import { Router, Route } from "svelte-routing";
    import Index from "./pages/Index.svelte";
    import InicioAdmin from "./pages/InicioAdmin.svelte";
    import LoginCliente from "./pages/LoginCliente.svelte";
    import InicioCliente from "./pages/InicioCliente.svelte";
    import PedidosCliente from "./pages/PedidosCliente.svelte";
    import AsignacionEmpleados from "./pages/AsignacionEmpleados.svelte";
</script>

<Router>
    <Route path="/" component={Index} />
    <Route path="/admin" component={InicioAdmin} />
    <Route path="/login-cliente" component={LoginCliente} />
    <Route path="/cliente" component={InicioCliente} />
    <Route path="/cliente/pedidos" component={PedidosCliente} />
    <Route path="/admin/asignar-empleados" component={AsignacionEmpleados}/>
</Router>

<style>
    /* Puedes agregar estilos globales aquí si es necesario */
</style>
