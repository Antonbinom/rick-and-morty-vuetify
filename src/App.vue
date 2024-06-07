<template>
  <v-app>
    <v-toolbar
      dense
      floating
      class="px-6 py-3 ga-3 position-fixed top-0 left-0 right-0 header"
    >
      <v-text-field v-model="search" hide-details label="Name" input />
      <v-select
        label="Status"
        item-title="name"
        item-value="status"
        v-model="selectedStatus"
        :items="options"
        hide-details
        class="select"
      />
      <v-btn @click="applyFilters" color="#5865f2" variant="flat" class="button"
        >Применить</v-btn
      >
    </v-toolbar>
    <v-main>
      <v-row class="px-6 cards">
        <v-col
          v-for="character in characters"
          :key="character.id"
          cols="12"
          xs="12"
          sm="12"
          md="6"
          lg="4"
        >
          <v-sheet border rounded>
            <v-card class="d-sm-flex">
              <v-img
                height="220"
                width="230"
                max-width="230"
                lazy-src="https://rickandmortyapi.com/api/character/avatar/1.jpeg"
                :src="character.image"
                cover
              >
                <template v-slot:placeholder>
                  <div class="d-flex align-center justify-center fill-height">
                    <v-progress-circular color="grey-lighten-4" indeterminate />
                  </div>
                </template>
              </v-img>

              <div class="ml-5">
                <h2 class="my-1">{{ character.name }}</h2>
                <div class="mb-2 text-subtitle-2">
                  <v-icon
                    :color="statusColor(character.status)"
                    icon="mdi-checkbox-blank-circle"
                    size="small"
                  />
                  {{ character.status }} • {{ character.species }}
                </div>

                <div class="mb-3 text-subtitle-2">
                  <span class="text-grey text-caption"
                    >Last known location:</span
                  >
                  <div>{{ character.location.name }}</div>
                </div>

                <div class="text-subtitle-2">
                  <span class="text-grey text-caption">First seen in:</span>
                  <div>{{ character.firstEpisode }}</div>
                </div>
              </div>
            </v-card>
          </v-sheet>
        </v-col>
      </v-row>
    </v-main>
    <v-footer
      class="position-fixed bottom-0 left-0 right-0 d-flex justify-center"
    >
      <v-pagination
        v-model="page"
        :length="pagesCount"
        next-icon="mdi-menu-right"
        prev-icon="mdi-menu-left"
        :total-visible="2"
        @update:modelValue="fetchCharacters"
      />
    </v-footer>
  </v-app>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import axios from "axios";
import type { Character } from "./types";
//

const characters = ref<Character[]>([]);
const page = ref<number>(1);
const pagesCount = ref<number>(0);
const search = ref<string>("");
const selectedStatus = ref<string>("");
const options = ref<{ name: string; status: string }[]>([
  { name: "All", status: "" },
  { name: "Dead", status: "dead" },
  { name: "Alive", status: "alive" },
  { name: "Unknown", status: "unknown" },
]);

const filterParams = ref<{ name: string; status: string }>({
  name: "",
  status: "",
});

const statusColor = (status: string) => {
  if (status === "Alive") return "cyan";
  else if (status === "Dead") return "red";
  else return "grey";
};

const fetchCharacters = async () => {
  const lowerCasedSearch: string = filterParams.value?.name?.toLowerCase();
  const apiUrl: string = `https://rickandmortyapi.com/api/character/?page=${page.value}`;
  const query = new URLSearchParams();

  if (lowerCasedSearch) query.append("name", lowerCasedSearch);
  if (filterParams.value.status)
    query.append("status", filterParams.value.status);

  const url = `${apiUrl}&${query.toString()}`;

  try {
    const {
      data: {
        results,
        info: { pages },
      },
    } = await axios.get(url);

    characters.value = await Promise.all(
      results.map(async (item: Character) => {
        const {
          data: { name },
        } = await axios.get(item.episode[0]);
        return { ...item, firstEpisode: name };
      })
    );

    pagesCount.value = pages;
  } catch (error: any) {
    characters.value = [];
    console.error(error);
  }
};

const applyFilters = () => {
  page.value = 1;
  filterParams.value.name = search.value;
  filterParams.value.status = selectedStatus.value;
  fetchCharacters();
};

onMounted(fetchCharacters);
</script>

<style>
.header {
  z-index: 1000;
}
.v-input {
  margin-right: 10px;
}
.v-btn {
  margin-left: 10px;
}
.v-img {
  height: 220px;
  width: 230px;
  max-width: 230px;
}
.v-main {
  margin-block: 100px;
}
@media (max-width: 600px) {
  .v-main {
    margin-top: 220px;
  }
  .v-toolbar__content {
    display: block !important;
    height: auto !important;
  }
  .v-img {
    width: 100% !important;
    max-width: 100% !important;
    margin-bottom: 20px;
  }
  .v-input,
  .v-select,
  .v-btn {
    width: 100%;
    margin-bottom: 15px;
  }
  .v-btn {
    margin-left: 0;
    height: auto;
  }
}
</style>
