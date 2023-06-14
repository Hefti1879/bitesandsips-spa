<template>
  <div>
    <label>API Token</label>
    <input type="text" v-model="token" @focusout="setToken">
  </div>
</template>

<script>
export default {
  name: "APITokenSetter",
  created() {},
  data() {
    return {
      token: ''
    };
  },
  props: {
    apiToken: String
  },
  watch: {
    apiToken: {
      immediate: true,
      handler(newVal) {
        this.token = newVal;
      }
    }
  },
  methods: {
    setToken(){
      this.$emit('setToken', this.token);
    }
  },
};
</script>

<style scoped></style>