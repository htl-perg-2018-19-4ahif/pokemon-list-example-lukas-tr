<template>
  <div id="app">
    <b-navbar toggleable="md" type="dark" variant="info">
      <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
      <b-navbar-brand href="/">Pokemon List</b-navbar-brand>
    </b-navbar>
    <pokemon-list/>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from "vue-property-decorator";
import PokemonList from "./components/PokemonList.vue";

@Component({
  components: {
    PokemonList
  }
})
export default class AppComponent extends Vue {}
</script>
