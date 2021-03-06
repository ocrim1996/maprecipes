<template>
  <div>
    <svg
      id="svg-el"
      xmlns="http://www.w3.org/2000/svg"
    >
      <symbol
        id="sbx-icon-search-13"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 40 40"
      >
        <path
          d="M26.804 29.01c-2.832 2.34-6.465 3.746-10.426 3.746C7.333 32.756 0 25.424 0 16.378 0 7.333 7.333 0 16.378 0c9.046 0 16.378 7.333 16.378 16.378 0 3.96-1.406 7.594-3.746 10.426l10.534 10.534c.607.607.61 1.59-.004 2.202-.61.61-1.597.61-2.202.004L26.804 29.01zm-10.426.627c7.323 0 13.26-5.936 13.26-13.26 0-7.32-5.937-13.257-13.26-13.257C9.056 3.12 3.12 9.056 3.12 16.378c0 7.323 5.936 13.26 13.258 13.26z"
          fill-rule="evenodd"
        />
      </symbol>
      <symbol
        id="sbx-icon-clear-2"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 20 20"
      >
        <path
          d="M8.96 10L.52 1.562 0 1.042 1.04 0l.522.52L10 8.96 18.438.52l.52-.52L20 1.04l-.52.522L11.04 10l8.44 8.438.52.52L18.96 20l-.522-.52L10 11.04l-8.438 8.44-.52.52L0 18.96l.52-.522L8.96 10z"
          fill-rule="evenodd"
        />
      </symbol>
    </svg>

    <vue-autosuggest
      v-model="query"
      :suggestions="suggestions"
      :get-suggestion-value="getSuggestionValue"
      :render-suggestion="getSuggestionValue"
      :section-configs="sectionConfigs"
      :input-props="{id:'autosuggest__input', placeholder:'Cerca una città, un piatto, una regione o un ingrediente...', spellcheck: false, type:'search', name:'search', class:'pt-sm-1 pt-2'}"
      @input="fetchSuggestions"
      @selected="onSelected"
      @keyup.enter="onEnter"
    />

    <button
      type="submit"
      title="Ricerca parola chiave."
      class="sbx-medium__submit"
    >
      <svg
        role="img"
        aria-label="Search"
      >
        <use xlink:href="#sbx-icon-search-13" />
      </svg>
    </button>
    <button
      type="reset"
      title="Cancella la ricerca."
      class="sbx-medium__reset"
      @click="resetInput"
    >
      <svg
        role="img"
        aria-label="Reset"
      >
        <use xlink:href="#sbx-icon-clear-2" />
      </svg>
    </button>
  </div>
</template>

<script>
import $ from 'jquery'
import { VueAutosuggest } from 'vue-autosuggest';
import recipes from '@/recipes.json'

export default {
  name: 'SearchBar',
  components: {
    VueAutosuggest
  },
  props: {
    mapMode: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      recipes: recipes,
      query: "",
      selected: "",
      suggestions: [],
      resultType: 'recipe',
      sectionConfigs: {
        default: {
          limit: 5,
          label: "Ricette",
          onSelected: selected => {
            if (selected != null) {
              this.resultType = 'recipe'
              this.selected = selected.item
            }
          }
        },
        regions: {
          limit: 3,
          label: "Regioni",
          onSelected: selected => {
            this.resultType = 'region'
            this.selected = selected.item;
          }
        },
        cities: {
          limit: 3,
          label: "Città",
          onSelected: selected => {
            this.resultType = 'city'
            this.selected = selected.item
          }
        },
        ingredients: {
          limit: 3,
          label: "Ingredienti",
          onSelected: selected => {
            this.resultType = 'ingredient'
            this.selected = selected.item
          }
        }
      }
    };
  },
  watch: {
    selected() {
      if (this.selected != null) {
        this.$emit('callback', { "selected": this.selected, "resultType": this.resultType })
      }
    },
    query() {
      if (this.query.trim() === '') {
        $('.sbx-medium__reset').hide();
      } else {
        $('.sbx-medium__reset').show();
      }
    }
  },
  mounted() {
    const reset = $('.sbx-medium__reset');
    const search = $(".sbx-medium__submit");

    reset.click(function() {
      search.focus();
      reset.hide();
    });

    if (this.mapMode) {
      search.css({'left':'5px'});
      reset.css({'right':'35px'});
    } else {
      search.css({'left':'20px'});
      reset.css({'right':'25px'});
    }

    function updatePlaceholder() {
      if ($(window).width() < 1199) {
        $('#autosuggest__input').attr('placeholder', 'Città, piatto, regione o ingrediente');
      } else {
        $('#autosuggest__input').attr('placeholder', 'Cerca una città, un piatto, una regione o un ingrediente...');
      }
    }

    updatePlaceholder();

    $(window).resize(function() {
      updatePlaceholder();
    });
  },
  methods: {
    onEnter() {
      if (this.query.trim() !== '') {
        this.resultType = 'none'
        this.selected = this.query
      }
    },
    onSelected(item) {
      this.selected = item.item
    },
    fetchSuggestions(query) {
      this.suggestions = [];
      this.selected = null;
      const trimmedQuery = query.trim()
      if (trimmedQuery.length > 0) {
        this.suggestions.push({ name: "default", data: this.filterRecipes(trimmedQuery) });
        this.suggestions.push({ name: "regions", data: this.filterRegions(trimmedQuery) });
        this.suggestions.push({ name: "cities", data: this.filterCities(trimmedQuery) });
        this.suggestions.push({ name: "ingredients", data: this.filterIngredients(trimmedQuery) });
      } else {
        this.callbackReset()
      }
    },
    filterRecipes(query) {
      const allRecipeNames = this.recipes.map(recipe => recipe.name)
      return this.filterAndSort(allRecipeNames, query)
    },
    filterRegions(query) {
      const allRegions = this.recipes.map(recipe => recipe.region)
      return this.filterAndSort(allRegions, query)
    },
    filterCities(query) {
      const allCities = this.recipes.map(recipe => recipe.city)
      return this.filterAndSort(allCities, query)
    },
    filterIngredients(query) {
      const allIngredients = this.recipes.flatMap(recipe => recipe.ingredients)
      const allIngredientNames = allIngredients.flatMap(ing => ing.name.replace(' q.b',''))
      return this.filterAndSort(allIngredientNames, query)
    },
    filterAndSort(array, query) {
      return [...new Set(array.filter(name => {
            return (this.normalized(name.toLowerCase()).indexOf(this.normalized(query.toLowerCase())) > -1)
          }).sort()
      )];
    },
    getSuggestionValue(suggestion) {
      return suggestion.item
    },
    resetInput() {
      this.query = ''
      this.callbackReset()
    },
    callbackReset() {
      this.$emit('callback', { "selected": '', "resultType": 'none' })
    },
    // https://stackoverflow.com/a/37511463
    normalized(string) {
      return string.normalize('NFD').replace(/[\u0300-\u036f]/g, "")
    },
  }
}
</script>

<style>

#svg-el {
  display: none;
}

#autosuggest__input {
  -webkit-transition: box-shadow .4s ease, background .4s ease;
  transition: box-shadow .4s ease, background .4s ease;
  outline: none;
  position: relative;
  display: block;
  border: var(--main-border);
  padding-left: 40px;
  width: 100%;
  box-sizing: border-box;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  border-radius: var(--main-radius);
  font-size: 25px;
  height: 50px;
  background-color: var(--searchbar-fill);
  box-shadow: var(--soft-shadow);
  vertical-align: middle;
  white-space: normal;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

#autosuggest__input::placeholder {
  color: var(--text-color);
  opacity: 1;
}

.autosuggest__results-container {
  position: relative;
  width: 98%;
  display: table;
  margin: 0 auto;
  top: 0;
}

.autosuggest__results {
  border: var(--main-border);
  box-shadow: var(--soft-shadow);
  position: absolute;
  width: 100%;
  z-index: 10000001;
  border-radius: 11px;
  background: var(--main-bg);
  padding: 0;
  max-height: 300px;
  overflow-y: scroll;
}

.autosuggest__results ul {
  list-style: none;
  padding-left: 0;
  margin: 0;
}

.autosuggest__results ul .autosuggest__results-before:only-child {
  display: none;
}

#autosuggest ul:nth-child(1) > .autosuggest__results-before {
  border-top: none;
}

.autosuggest__results .autosuggest__results-before {
  color: red;
  font-size: 25px;
  text-align: left;
  margin-left: 0;
  padding: 10px 15px 5px;
  border-top: 1px solid var(--ingredients-sidebar-bg);
}

.autosuggest__results .autosuggest__results-item {
  cursor: pointer;
  height: 40px;
  padding-left: 15px;
  padding-top: 8px;
  font-size: 20px;
  text-align: left;
}

.autosuggest__results .autosuggest__results-item:active,
.autosuggest__results .autosuggest__results-item:hover,
.autosuggest__results .autosuggest__results-item:focus,
.autosuggest__results .autosuggest__results-item.autosuggest__results-item--highlighted {
  background-color: var(--ingredients-sidebar-bg);
}

.sbx-medium__submit {
  position: absolute;
  top: 0;
  right: inherit;
  margin: 0;
  border: 0;
  background-color: rgba(255, 255, 255, 0);
  padding: 0;
  width: 37px;
  height: 100%;
  vertical-align: middle;
  text-align: center;
  font-size: inherit;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  cursor: auto !important;
}

.sbx-medium__submit::before {
  display: inline-block;
  margin-right: -4px;
  height: 100%;
  vertical-align: middle;
  content: '';
}
.sbx-medium__submit:focus {
  outline: 0;
}
.sbx-medium__submit svg {
  width: 17px;
  height: 17px;
  vertical-align: middle;
  fill: var(--text-color);
}
.sbx-medium__reset {
  display: none;
  position: absolute;
  top: 30%;
  right: 25px;
  margin: 0;
  border: 0;
  background: none;
  cursor: pointer;
  padding: 0;
  font-size: inherit;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.sbx-medium__reset svg {
  display: block;
  margin: 4px;
  width: 13px;
  height: 13px;
}

.sbx-medium__reset:focus {
  outline: 0;
}

#autosuggest__input::-webkit-search-decoration,
#autosuggest__input::-webkit-search-cancel-button,
#autosuggest__input::-webkit-search-results-button,
#autosuggest__input::-webkit-search-results-decoration {
  display: none;
}

</style>