<template>
  <main>
    <div>
      <h1>Les catégories de produits</h1>
    </div>
    <div>
      <div>
        <select id="selectionCategorie" size="1" v-model="selectedCategory" @change="updateProducts">
          <option value="">Toutes les catégories</option>
          <option v-for="categorie in data.listeCategories" :value="categorie.code">
            {{ categorie.libelle }}
          </option>
        </select>
      </div>
    </div>
    <div>
      <table>
        <caption>Les produits</caption>
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
      </table>
    </div>
  </main>
</template>

<script setup>
import {reactive, onMounted, ref, watch} from "vue";
import {doAjaxRequest} from "@/api";


let data = reactive({
  // La liste des produits affichée sous forme de table
  productsList: [],
  // La liste des categories de la dropdown liste sous forme de table
  listeCategories: []
});

// Categorie sélectionée
let selectedCategory = ref("");

function showError(error) {
  console.log("Erreur : status %d", error.status)
  console.log(error.body);
  alert(error.message);
}

function productsLoading(category) {
  // Appel à l'API pour avoir la liste des produits
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  // Si une catégorie a été selectionnée dans la dropdown liste
  if(category){
    doAjaxRequest("/api/categories/"+category+"/produits?sort=nom,asc")
        .then((json) => {
          data.productsList = json._embedded.produits;
        })
        .catch(showError);
  }else{
    // Si on ne filtre sur aucune catégorie
    doAjaxRequest("/api/produits?sort=nom,asc")
        .then((json) => {
          data.productsList = json._embedded.produits;
        })
        .catch(showError);
  }
}

function chargeCategories() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest("/api/categories?sort=code,desc")
      .then((json) => {
        data.listeCategories = json._embedded.categories;
        console.log(data.listeCategories)
      })
      .catch(showError);
}

/**
 * Fonction utilisées lors de la sélection d'une catégorie dans la dropdown list
 */
function updateProducts() {
    productsLoading(selectedCategory.value);
}
// A l'affichage du composant, on affiche la liste
onMounted(() => {
  productsLoading();
  chargeCategories();
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
