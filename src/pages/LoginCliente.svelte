<script>
    import { onMount } from 'svelte';
    import axios from 'axios';
    import { navigate } from 'svelte-routing';

    let email = '';
    let password = '';
    let errorMessage = '';
    let isRegistering = false;

    // Campos para el registro
    let nombre = '';
    let telefono = '';
    let confirmPassword = '';
    let registerError = '';
    let registerSuccess = '';

    async function handleLogin() {
        try {
            const response = await axios.post('http://localhost:8080/cliente/login', {
                email,
                password,
            });

            if (response.status === 200) {
                //Guardar la inforacion del cliente
                localStorage.setItem('cliente', JSON.stringify(response.data));

                // Redirigir a la página de inicio del cliente si el login es exitoso
                navigate('/cliente');
            }
        } catch (error) {
            errorMessage = 'Credenciales inválidas. Inténtalo de nuevo.';
            console.error('Error en el inicio de sesión:', error);
        }
    }

    async function handleRegister() {
        if (password !== confirmPassword) {
            registerError = 'Las contraseñas no coinciden.';
            return;
        }

        try {
            const response = await axios.post('http://localhost:8080/cliente/registrar', {
                nombre,
                email,
                telefono,
                password,
            });

            if (response.status === 200) {
                registerSuccess = 'Registro exitoso. Ahora puedes iniciar sesión.';
                // Limpiar el formulario
                nombre = '';
                email = '';
                telefono = '';
                password = '';
                confirmPassword = '';
                registerError = '';
            }
        } catch (error) {
            registerError = 'Error al registrar el cliente. Verifica los datos.';
            console.error('Error en el registro:', error);
        }
    }
</script>

<div class="login-container">
    {#if !isRegistering}
        <h2>Login Cliente</h2>
        <div class="form-group">
            <label for="email">Correo Electrónico:</label>
            <input type="email" id="email" bind:value={email} required />
        </div>
        <div class="form-group">
            <label for="password">Contraseña:</label>
            <input type="password" id="password" bind:value={password} required />
        </div>
        {#if errorMessage}
            <p class="error-message">{errorMessage}</p>
        {/if}
        <button on:click={handleLogin}>Iniciar Sesión</button>
        <p>
            ¿No tienes una cuenta?  
            <button class="link-button" on:click={() => (isRegistering = true)}>Regístrate aquí</button>
        </p>
        
    {/if}

    {#if isRegistering}
        <h2>Registro Cliente</h2>
        <div class="form-group">
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" bind:value={nombre} required />
        </div>
        <div class="form-group">
            <label for="email">Correo Electrónico:</label>
            <input type="email" id="email" bind:value={email} required />
        </div>
        <div class="form-group">
            <label for="telefono">Teléfono:</label>
            <input type="text" id="telefono" bind:value={telefono} required />
        </div>
        <div class="form-group">
            <label for="password">Contraseña:</label>
            <input type="password" id="password" bind:value={password} required />
        </div>
        <div class="form-group">
            <label for="confirmPassword">Confirmar Contraseña:</label>
            <input type="password" id="confirmPassword" bind:value={confirmPassword} required />
        </div>
        {#if registerError}
            <p class="error-message">{registerError}</p>
        {/if}
        {#if registerSuccess}
            <p class="success-message">{registerSuccess}</p>
        {/if}
        <button on:click={handleRegister}>Registrarse</button>
        <p>
            ¿Ya tienes una cuenta?  
            <button class="link-button" on:click={() => (isRegistering = false)}>Inicia sesión</button>
        </p>
        
    {/if}
</div>

<style>
    .login-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
    }

    .form-group {
        margin-bottom: 1rem;
    }

    input {
        display: block;
        width: 100%;
        padding: 0.5rem;
        margin-bottom: 1rem;
        border: 1px solid #ccc;
        border-radius: 4px;
    }

    button {
        padding: 0.5rem 1rem;
        background-color: #28a745;
        color: #fff;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    button:hover {
        background-color: #218838;
    }

    .error-message {
        color: red;
    }

    .success-message {
        color: green;
    }

    .link-button{
        color: #007bff;
        text-decoration: none;
    }

    .link-button:hover {
        text-decoration: underline;
    }
</style>
