<template>
  <div v-if="$apollo.loading" class="flex flex-col h-full w-full justify-center items-center font-bold text-secondaryText">
    <p>Picking the best<br>dishes for you ...</p>
    <img src="../assets/images/loading.svg"/>
  </div>

  <div class="w-full" v-else>
    <div style="overflow: scroll; margin-bottom: 100px;">

      <RecipeComponent
          v-for="recipe in searchForRecipes"
          v-bind:key="recipe.id"
          v-bind:recipe="recipe"
      ></RecipeComponent>

    </div>
  </div>
</template>

<script>
import RecipeComponent from '../components/listing/RecipeComponent.vue'
import gql from 'graphql-tag';

const GET_RECIPES = gql`query searchForRecipes ($ingredients: [String!]!, $tags: [String!]!) {
        searchForRecipes(take: 5, searchOptions: {ingredients: $ingredients, tags: $tags}) {
            ID,
            name,
            imagePath,
            duration,
            missingIngredients {ID, name}
        }
      }`;

export default {
  name: "Favourites",
  components: {
    RecipeComponent
  },
  data() {
    return {
      searchForRecipes: []
    }
  },
  apollo: {
    // Query with parameters
    searchForRecipes: {
      // gql query
      query: GET_RECIPES,
      context() { return { headers: { 'Authorization': localStorage.getItem("tender-user-token") } } },

      variables() {
        return {
          ingredients: this.selectedIngredients,
          tags: this.likedTags().map(tag => tag.id)
        }
      }
    },
  },
  methods: {
    goToRecipe(recipeId) {
      console.log(recipeId);
    },
    likedTags() {
        return this.$store.getters['tagsStorage/likedTags'];
    }
  },
  computed: {
    selectedIngredients() {
      const selectedIngredients = [];
      this.$store.getters['selectedIngredients'].forEach(selectedIngredient => {
        selectedIngredients.push(selectedIngredient.ID);
      })
     return selectedIngredients;
    }
  }
}
</script>

<style scoped>

</style>