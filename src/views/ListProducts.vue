<template>
  <main>
    <div>
      <h1>La liste des produits</h1>
    </div>
    <div>
      <table>
        <caption>Les produits - page {{ num }} / {{ data.total - 1 }}</caption>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
          <th>Stock</th>
          <th>Commande</th>
        </tr>
        <!-- Si le tableau des catégories est vide -->
        <tr v-if="data.productsList.length === 0">
          <td colspan="4">Veuillez patienter, chargement des produits...</td>
        </tr>
        <!-- Si le tableau des catégories n'est pas vide -->
        <tr v-for="product in data.productsList" :key="product.reference">
          <td>{{ product.nom }}</td>
          <td>{{ product.prixUnitaire }}</td>
          <td>{{ product.unitesEnStock }}</td>
          <td>{{ product.unitesCommandees }}</td>
        </tr>
        <tr>
          <th>
            <button @click="num = 0" :disabled="num ===0"> ⇇</button>
          </th>
          <th>
            <button @click="num--" :disabled="num ===0"> ⭠</button>
          </th>
          <th>
            <button @click="num++" :disabled="num === data.total-1"> ⭢</button>
          </th>
          <th>
            <button @click="num = data.total-1" :disabled="num === data.total-1"> ⇉</button>
          </th>
        </tr>
      </table>
    </div>
  </main>
</template>

<script setup>
import {reactive, onMounted, ref, watch} from "vue";
import {doAjaxRequest} from "@/api";

let num = ref(0);
let data = reactive({
  // La liste des produits affichée sous forme de table
  productsList: [],
  total: 0
});

watch(num, (newnum) => {
  if (newnum >= 0 || newnum <= 15) {
    productsLoading()
  }
});

function showError(error) {
  console.log("Erreur : status %d", error.status)
  console.log(error.body);
  alert(error.message);
}

function productsLoading() {
  // Appel à l'API pour avoir la liste des produits
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest("/api/produits?page=" + num.value + "&size=5&sort=nom,asc")
      .then((json) => {
        data.productsList = json._embedded.produits;
        data.total = json.page.totalPages;
      })
      .catch(showError);
}

// A l'affichage du composant, on affiche la liste
onMounted(() => {
  productsLoading();
});

</script>


<style scoped>
td,
th {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #232623;
  color: rgb(255, 255, 255);
}
</style>
