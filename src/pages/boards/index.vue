<script setup lang="ts">
import { useAlerts } from "@/stores/alerts";
import boardsQuery from "@/graphql/queries/boards.query.gql";
import createBoardMutation from "@/graphql/mutations/createBoard.mutation.gql";
import { useMutation, useQuery } from "@vue/apollo-composable";
import { computed } from "vue";
import router from "@/router";

const { result, loading, onError } = useQuery(boardsQuery);
const boards = computed(() => result.value?.boardsList?.items || []);

const alerts = useAlerts();

onError(() => alerts.error("Error loading boards"));

const { mutate: createBoard } = useMutation(createBoardMutation, () => ({
  update(cache, { data: { boardCreate } }) {
    cache.updateQuery({ query: boardsQuery }, (res) => ({
      boardsList: {
        items: [...res.boardsList.items, boardCreate],
      },
    }));
  },
}));

async function handleBoardCreate() {
  const newBoardPayload = {
    data: {
      title: "My New Board",
    },
  };
  await createBoard(newBoardPayload);
  alerts.success("New Board created!");
}

const getCoolGradient = (index: number) => {
  let finalGradientString = "";
  switch (index) {
    case 1:
      finalGradientString = "from-orange-400 to-pink-500";
      break;
    case 2:
      finalGradientString = "from-green-400 to-blue-400";
      break;
    case 3:
      finalGradientString = "from-purple-400 to-blue-400";
      break;
    default:
      finalGradientString = "from-orange-400 to-yellow-500";
  }
  return finalGradientString;
};
</script>

<template>
  <h1 class="mb-5 text-3xl">Boards</h1>
  <div class="flex flex-wrap gap-2">
    <div
      class="border rounded-md bg-gradient-to-tr"
      v-for="(board, index) in boards"
      :key="board.id"
      :class="getCoolGradient(index)"
    >
      <BoardCard
        :board="board"
        class="transition duration-100 ease-in border rounded-md hover:-rotate-3"
      />
    </div>
    <button class="text-gray-500" @click="handleBoardCreate">
      <span>New Board +</span>
    </button>
  </div>
  <AppLoader v-if="loading" :overlay="true" />
</template>
