<template>
  <b-container fluid>
    <b-row>
      <b-col md="6" class="my-1">
        <b-form-group horizontal label="Filter" class="mb-0">
          <b-input-group>
            <b-form-input v-model="filter" placeholder="Type to Search"/>
            <b-input-group-append>
              <b-btn :disabled="!filter" @click="filter = ''">Clear</b-btn>
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>
      <b-col md="6" class="my-1">
        <b-form-group horizontal label="Per page" class="mb-0">
          <b-form-select :options="pageOptions" v-model="perPage"/>
        </b-form-group>
      </b-col>
    </b-row>

    <b-table
      show-empty
      stacked="md"
      :items="items"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filter"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      @filtered="onFiltered"
    >
      <template slot="name" slot-scope="row">{{row.value}}</template>
      <template slot="actions" slot-scope="row">
        <b-button size="sm" @click.stop="info(row.item, row.index, $event.target)" class="mr-1">Info</b-button>
      </template>
    </b-table>

    <b-row>
      <b-col md="6" class="my-1">
        <b-pagination
          :total-rows="totalRows"
          :per-page="perPage"
          v-model="currentPage"
          class="my-0"
        />
      </b-col>
    </b-row>

    <b-modal id="modalInfo" @hidden="resetModal" :title="modalInfo.name" ok-only>
      <b-img :src="modalInfo.sprite" fluid-grow :alt="modalInfo.name"/>
      <b-list-group class="mb-4">
        <b-list-group-item>Name: {{modalInfo.name}}</b-list-group-item>
        <b-list-group-item>Weight: {{modalInfo.weight}}</b-list-group-item>
      </b-list-group>

      <b-list-group v-for="(ability, id) in modalInfo.abilities" :key="id" class="mb-4">
        <b-list-group-item>Ability: {{ability.name}}</b-list-group-item>
        <b-list-group-item>{{ability.details}}</b-list-group-item>
      </b-list-group>
    </b-modal>
  </b-container>
</template>

<script lang="ts">
import axios from "axios";
import { Vue, Component, Prop } from "vue-property-decorator";

interface IAbilityResponse {
  effect_changes: any[];
  effect_entries: ({ effect: string; language: any })[];
  flavor_text_entries: any[];
  generation: any;
  id: number;
  is_main_series: boolean;
  name: string;
  names: any[];
  pokemon: any[];
}

interface IAllPokemonReponse {
  count: number;
  next: any;
  previous: any;
  results: ({ name: string; url: string })[];
}

interface IPokemonItem {
  name: string;
  url: string;
}

interface IPokemonDetails {
  name: string;
  sprite: string;
  weight: number;
  abilities: ({
    visible: boolean;
    name: string;
    url: string;
    details: string;
  })[];
}

interface IDetailedPokemonResponse {
  abilities: ({
    is_hidden: boolean;
    ability: {
      name: string;
      url: string;
    };
  })[];
  base_experience: number;
  forms: any[];
  game_indices: any[];
  height: number;
  held_items: any[];
  id: number;
  is_default: boolean;
  location_area_encounters: string;
  moves: any[];
  name: string;
  order: number;
  species: { name: string; url: string };
  sprites: {
    back_default: string;
    back_female: string;
    back_shiny: string;
    back_shiny_female: string;
    front_default: string;
  };
  stats: any[];
  types: any[];
  weight: number;
}

@Component
export default class PokemonList extends Vue {
  items: IPokemonItem[] = [];
  fields = [
    {
      key: "name",
      label: "Pokemon name",
      sortable: true,
      sortDirection: "desc"
    },
    { key: "actions", label: "Actions" }
  ];
  currentPage = 1;
  perPage = 5;
  totalRows = 0;
  pageOptions = [5, 10, 15];
  sortBy = null;
  sortDesc = false;
  sortDirection = "asc";
  filter = null;
  modalInfo: IPokemonDetails = {
    name: "",
    sprite: "",
    weight: 0,
    abilities: []
  };

  async mounted() {
    const response = await axios.get<IAllPokemonReponse>(
      "https://pokeapi.co/api/v2/pokemon/"
    );
    this.items = response.data.results;
    this.totalRows = response.data.results.length;
  }

  async info(item: IPokemonItem, index: number, button: any) {
    const { data } = await axios.get<IDetailedPokemonResponse>(item.url);

    this.modalInfo.name = data.name;
    this.modalInfo.sprite = data.sprites.front_default;
    this.modalInfo.weight = data.weight;
    this.modalInfo.abilities = await Promise.all(
      data.abilities.map(async ability => ({
        name: ability.ability.name,
        url: ability.ability.url,
        visible: false,
        details: (await this.fetchAbilityInfo(
          ability.ability.url
        )).effect_entries
          .map(entry => entry.effect)
          .join(", ")
      }))
    );

    this.$root.$emit("bv::show::modal", "modalInfo", button);
  }

  async fetchAbilityInfo(url: string) {
    const { data } = await axios.get<IAbilityResponse>(url);
    console.log(data);
    return data;
  }

  resetModal() {
    this.modalInfo.name = "";
    this.modalInfo.sprite = "";
    this.modalInfo.weight = 0;
    this.modalInfo.abilities = [];
  }

  onFiltered(filteredItems: IPokemonItem[]) {
    this.totalRows = filteredItems.length;
    this.currentPage = 1;
  }
}
</script>

<style scoped>
.lds-ring {
  display: inline-block;
  position: relative;
  width: calc(25px + 2 * 6px);
  height: calc(25px + 2 * 6px);
}
.lds-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 25px;
  height: 25px;
  margin: 6px;
  border: 6px solid var(--primary);
  border-radius: 50%;
  animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: var(--primary) transparent transparent transparent;
}
.lds-ring div:nth-child(1) {
  animation-delay: -0.45s;
}
.lds-ring div:nth-child(2) {
  animation-delay: -0.3s;
}
.lds-ring div:nth-child(3) {
  animation-delay: -0.15s;
}
@keyframes lds-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
