<template lang="pug">
.example-field(v-if="enums.length > 0", :class="{ showing: showAll }")
  strong Examples:{{ ' ' }}
  code(v-if="!showAll") {{ '"' + enums.slice(0, 5).join('", "') + '"' }}
    .show-btn(v-if="enums.length > 5", @click="toggleShowAll") Show&nbsp;More
  code(v-if="showAll && enums.length > 5") {{ '"' + enums.join('", "') + '"' }}
    .show-btn(@click="toggleShowAll") Show&nbsp;Less
</template>

<script>
export default {
  name: "ExampleField",
  props: {
    type: {
      required: true,
      type: String,
    },
  },
  data() {
    return {
      allEnums: null,
      show: false,
    };
  },
  computed: {
    showAll() {
      return this.show;
    },
    enums() {
      let enums;

      if (this.allEnums) {
        enums = this.allEnums[this.type];

        if (enums) {
          return enums.sort();
        }
      }

      return [];
    },
  },
  async beforeCreate() {
    await this.$helpers.setStoreState.apply(this, ["EnumValues"]);

    this.allEnums =
      this.$store.getters.EnumValues[this.$page.frontmatter.schema];
  },
  methods: {
    toggleShowAll() {
      this.show = !this.show;
    },
  },
};
</script>

<style lang="scss" scoped>
.example-field {
  position: relative;
  display: flex;
  align-items: center;

  .show-btn {
    display: inline;
    margin-left: 0.5rem;
    color: var(--accent-color);
    text-decoration: underline;
    cursor: pointer;

    &:hover {
      text-decoration: none;
    }
  }
}
</style>
