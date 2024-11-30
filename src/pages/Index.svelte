<script>
    import { onMount } from 'svelte';
    import axios from 'axios';
    import { navigate } from 'svelte-routing';

    let username = '';
    let password = '';
    let errorMessage = '';

    async function handleLogin() {
        try {
            const response = await axios.post('https://backendmuebles-production.up.railway.app/administrador/login', {
                username,
                password
            });

            if (response.status === 200) {
                // Redirigir a la página /admin si el login es exitoso
                navigate('/admin');
            }
        } catch (error) {
            errorMessage = 'Credenciales inválidas. Inténtalo de nuevo.';
            console.error('Error en el inicio de sesión:', error);
        }
    }
</script>

<div class="login-container">
    <h2>Login</h2>
    <div class="form-group">
        <label for="username">Usuario:</label>
        <input type="text" id="username" bind:value={username} required>
    </div>
    <div class="form-group">
        <label for="password">Contraseña:</label>
        <input type="password" id="password" bind:value={password} required>
    </div>
    {#if errorMessage}
        <p class="error-message">{errorMessage}</p>
    {/if}
    <button on:click={handleLogin}>Iniciar Sesión</button>
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
</style>


