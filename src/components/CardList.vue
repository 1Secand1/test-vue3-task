<template>
  <div>
    <div class="sorting-menu" :style="`background: ${options.color}`">
      <v-speed-dial location="right center" transition="slide-x-transition">
        <template v-slot:activator="{ props: activatorProps }">
          <v-fab v-bind="activatorProps" icon="mdi-filter" />
        </template>

        <v-btn key="1" icon="mdi-filter" />
        <v-btn
          key="2"
          icon="mdi-sort-descending"
          @click="currentSortingStatus = sortingStatuses.descending"
        />
        <v-btn
          key="4"
          icon="mdi-sort-ascending"
          @click="currentSortingStatus = sortingStatuses.ascending"
        />
        <v-btn
          key="3"
          icon="mdi-sort-variant-remove"
          @click="currentSortingStatus = sortingStatuses.unsorted"
        />
      </v-speed-dial>
    </div>

    <section
      :style="`background: ${options.color}`"
      @drop="onDrop($event, options.id)"
      @dragover.prevent
      @dragenter.prevent
    >
      <div class="title">
        <h2>
          {{ options.title }}
        </h2>
        <div class="counter">
          <span>{{ options.cards.length }}</span>
        </div>
      </div>

      <v-btn
        icon="mdi-plus"
        variant="tonal"
        class="mt-5"
        color="white"
        @click="isNewCardDialogOpen = true"
      />

      <CardItem
        v-for="(card, index) in sortedCards"
        draggable="true"
        :key="index"
        :card="card"
        :options="props.options"
        @delete-card="deleteCard(card.id)"
        @dragstart="onDragStart($event, card)"
        @sorting="currentSortingStatus = sortingStatuses.descending"
      />

      <CardForm
        title="Добавление новой карточки"
        v-model="isNewCardDialogOpen"
        :form="form"
        @save-card="addCard"
        @close-form="isNewCardDialogOpen = false"
      />
    </section>
  </div>
</template>

<script setup>
import CardItem from "./CardItem.vue";
import CardForm from "./CardForm.vue";

import { ref, inject } from "vue";
import { computed } from "vue";
import { sortingStatuses } from "../constants/sortingStatuses";

const firstList = inject("firstList");
const secondList = inject("secondList");
const lastList = inject("lastList");

const props = defineProps({
  options: {
    type: Object,
    required: true,
    default() {
      return {
        color: "",
        title: "",
        cards: [],
        id: 0,
      };
    },
  },
});

const emits = defineEmits(["addCard", "deleteCard"]);

const isNewCardDialogOpen = ref(false);
const currentSortingStatus = ref(sortingStatuses.unsorted);

const form = ref({
  image: "",
  id: null,
  title: "",
  description: "",
  category: "",
  price: null,
  rating: {},
});

let cards = ref([]);

const sortedCards = computed(() => {
  switch (currentSortingStatus.value) {
    case sortingStatuses.ascending:
      return [...props.options.cards].sort(
        (a, b) => a.rating.rate - b.rating.rate
      );

    case sortingStatuses.descending:
      return [...props.options.cards].sort(
        (a, b) => b.rating.rate - a.rating.rate
      );

    default:
      return props.options.cards;
  }
});

function getLocalCards() {
  switch (props.options.id) {
    case 1:
      cards = firstList;
      break;
    case 2:
      cards = secondList;
      break;
    case 3:
      cards = lastList;
      break;
  }
}

getLocalCards();

function addCard() {
  emits("addCard", props.options.id, form.value);
  isNewCardDialogOpen.value = false;
}

function deleteCard(cardId) {
  emits("deleteCard", props.options.id, cardId);
}

function onDragStart(event, item) {
  event.dataTransfer.dropEffect = "move";
  event.dataTransfer.effectAllowed = "move";
  event.dataTransfer.setData("itemId", item.id.toString());
}

function onDrop(event, optionsId) {
  const itemId = parseInt(event.dataTransfer.getData("itemId"));
  let otherLists = [];

  switch (optionsId) {
    case 1:
      cards.value = firstList.value;
      otherLists = secondList.value.concat(lastList.value);
      break;
    case 2:
      cards.value = secondList.value;
      otherLists = firstList.value.concat(lastList.value);
      break;
    case 3:
      cards.value = lastList.value;
      otherLists = secondList.value.concat(firstList.value);
      break;
  }

  const newCard = otherLists.find((card) => card.id === itemId);

  firstList.value = firstList.value.filter((card) => card.id !== newCard.id);
  secondList.value = secondList.value.filter((card) => card.id !== newCard.id);
  lastList.value = lastList.value.filter((card) => card.id !== newCard.id);
  cards.value.unshift(newCard);

  switch (optionsId) {
    case 1:
      firstList.value = cards.value;
      break;
    case 2:
      secondList.value = cards.value;
      break;
    case 3:
      lastList.value = cards.value;
      break;
  }
}
</script>

<style lang="scss" scoped>
section {
  padding: 10px;
  width: 400px;
  min-height: 500px;
  height: fit-content;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;

  .title {
    padding-bottom: 10px;
    width: 90%;
    display: flex;
    align-items: center;
    justify-content: center;
    border-bottom: 2px solid white;

    .counter {
      margin-left: 10px;
      background: white;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
  }
}

.sorting-menu {
  align-items: center;

  min-height: 65px;
  padding: 20px 10px;

  color: white;
  background: white;
  border-radius: 10px;
}
</style>
