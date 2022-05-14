<template>
    <v-row>
    <v-col class="text-center">
        <blockquote class="blockquote">
        <header>
            <small>
            <em>word graph</em>
            </small>
        </header>
        <body>
            <p>ここにWordGraphを表示する．</p>
        </body>
        <footer>
            <small>
            <em>&mdash;word graph</em>
            </small>
        </footer>
        </blockquote>
    </v-col>
    </v-row>
</template>

<script>
import NuxtLogo from '../components/NuxtLogo.vue'
import Tutorial from '../components/Tutorial.vue'
export default {
    components: { NuxtLogo, Tutorial },
    name: 'SamplePages'
}
</script>
