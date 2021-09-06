<template>
<div class="container">
    <div class="row justify-content-center mt-5">
        <div class="col-md-10">
            <h1> <img src="./assets/logo.png" width="50" height="50" /> Datos de una persona</h1>
            <Persona />
        </div>
    </div>
</div>
</template>

<script>
import Persona from './components/Persona.vue';
export default {
    components: {
        Persona
    }
}
</script>