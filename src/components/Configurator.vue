<template>
  <div class="container">
    <h1>{{ title }}</h1>
    <p>
      Please select product options
    </p>
    <div class="row">
      <div v-for="(option, optionKey) in options" :key="optionKey" class="col">
        <h4>{{option.title}}</h4>
        <div class="btn-group-vertical" role="group">
          <button
            v-for="(value, valueKey) in option.values" :key="valueKey"
            type="button"
            v-on:click="toggleOptionValue(option, value.option_type_id)"
            class="btn btn-outline-primary btn-block"
            :disabled="valueDisabled(value.option_type_id)"
            :class="{active: valueSelected(value.option_type_id) }"
          >{{value.title}}</button>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <button type="button" :disabled="!allOptionsSelected" class="btn btn-primary">Add to Cart</button>
      </div>
    </div>
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
      valueDisabled(id) {
        return this.disabledValueIds.has(id);
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
        // Every option has at least one value selected
        return this.options.every(option => {
          return option.values.some(value => this.selectedValueIds.has(value.option_type_id));
        });
      }
    },
  }
</script>

<style scoped>
h4 {
  margin: 20px 0;
}
.btn.btn-outline-primary:disabled{
  border-color: #6c757d;
  color: #6c757d;
}

</style>
