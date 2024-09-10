<template>
  <main>
    <CardList
      v-for="(item, index) in CardListOptions"
      :key="index"
      :options="item"
      @addCard="addCardToOption"
      @deleteCard="deleteCardToOption"
    />
  </main>
</template>

<script setup>
import { ref, provide, toRaw } from "vue";
import axios from "axios";
import CardList from "./components/CardList.vue";

const firstList = ref([]);
const secondList = ref([]);
const lastList = ref([]);

const CardListOptions = ref([
  {
    color: "#0aa6e362",
    title: "Необработанные",
    cards: toRaw(firstList),
    id: 1,
  },
  {
    color: "#e86405bd",
    title: "В работе",
    cards: toRaw(secondList),
    id: 2,
  },
  {
    color: "#9400d364",
    title: "Завершенные",
    cards: toRaw(lastList),
    id: 3,
  },
]);

function addCardToOption(optionId, newCard) {
  const targetOption = CardListOptions.value.find(
    (option) => option.id === optionId
  );

  if (!targetOption) return;
  targetOption.cards.unshift(newCard);
}

function deleteCardToOption(optionId, cardId) {
  const targetOption = CardListOptions.value.find(
    (option) => option.id === optionId
  );

  if (!targetOption) return;
  targetOption.cards = targetOption.cards.filter((card) => card.id != cardId);
}

const fetchCards = async () => {
  try {
    await axios
      .get("https://fakestoreapi.com/products")
      .then((res) => (firstList.value = res.data));
    console.log("fetchCards - успешно");
  } catch (error) {
    console.log(error);
    alert("Упс! Ошибка получения данных ;(");
  }
};

fetchCards();

provide("firstList", firstList);
provide("secondList", secondList);
provide("lastList", lastList);
</script>

<style scoped>
main {
  margin: 5% 0;
  display: flex;
  justify-content: center;
  gap: 5%;
  width: 100%;
}
</style>
