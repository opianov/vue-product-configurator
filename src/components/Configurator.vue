<template>
  <div class="hello">
    <h1>{{ title }}</h1>
    <p>
      Please select product options
    </p>
    <div v-for="(option, optionKey) in options" :key="optionKey">
      <h3>{{option.title}}</h3>
      <div v-for="(value, valueKey) in option.values" :key="valueKey" class="column">
        <a
          v-if="valueEnabled(value.option_type_id)"
          v-on:click="toggleOptionValue(option, value.option_type_id)"
          :class="{selected: valueSelected(value.option_type_id) }"
        >{{value.title}}</a>
        <span v-else>{{value.title}}</span>
      </div>
    </div>
    <button v-if="allOptionsSelected">Buy</button>
    <span v-else>Please select all options</span>
  </div>
</template>

<script>
  import productConfig from "../product-config.json";
  export default {
    name: 'Configurator',
    data() {
      return {
        options: productConfig.options,
        exclusions: productConfig.exclusions,
        selectedValueIds: new Set(),
      };
    },
    methods: {
      toggleOptionValue(option, valueId) {
        if (this.selectedValueIds.has(valueId)) {
          this.selectedValueIds.delete(valueId);
        } else {
          option.values.forEach(value => this.selectedValueIds.delete(value.option_type_id));
          this.selectedValueIds.add(valueId);
        }
        // to trigger reactivity, as Vue cant handle Set/Map modifications at the moment
        this.selectedValueIds = new Set(this.selectedValueIds);
      },
      valueSelected(id) {
        return this.selectedValueIds.has(id);
      },
      valueEnabled(id) {
        return !this.disabledValueIds.has(id);
      },
    },
    props: {
      title: String
    },
    computed: {
      disabledValueIds() {
        const disabledIds = this.exclusions.map(exclusion => {
          const exclusionRest = Object.keys(exclusion).filter(id => !this.selectedValueIds.has(id));
          // only one item from exclusion rule remains, this means we cant select it
          if (exclusionRest.length === 1) return exclusionRest[0];
          return null;
        }).filter(id => id !== null);
        return new Set([...disabledIds]);
      },
      allOptionsSelected() {
        return this.options.every(option => {
          return option.values.some(value => this.selectedValueIds.has(value.option_type_id));
        });
      }
    },
  }
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

a.selected {
  font-weight: bold;
}
</style>
