<template>
  <span v-if="field.visible" :class="ajaxClasses">
    <button
      ref="novabutton"
      type="button"
      class="nova-button"
      v-html="buttonText"
      :disabled="disabled"
      @click="handleClick"
      :class="buttonClasses"
      :style="{ 'min-width': buttonWidth }"
      :title="field.title"
      :dusk="field.attribute"
    ></button>
  </span>
</template>

<style>
.nova-button {
  white-space: nowrap;
}

.nova-button-loading,
.nova-button-success,
.nova-button-error {
  pointer-events: none;
}
</style>

<script>
import { queue } from '../queue.js';

export default {
  props: ['resource', 'resourceName', 'resourceId', 'field', 'ajaxClasses', 'disabled'],
  data: () => ({
    buttonWidth: null,
    loading: false,
    success: false,
    error: false,
  }),
  mounted() {
    this.$nextTick(() => {
      this.buttonWidth = this.$refs.novabutton.clientWidth + 2 + 'px';
    });
  },
  methods: {
    async handleClick() {
      if (this.field.disabled) {
        return;
      }

      queue.add(this.resourceId);

      this.$emit('clicked');

      try {
        await this.post();

        this.success = true;
        this.loading = false;

        queue.hasSuccess = true;
        queue.remove(this.resourceId);

        this.$emit('success');
        this.$emit('finished');

        this.navigate();
      } catch (error) {
        this.error = true;
        this.loading = false;

        queue.hasError = true;

        queue.remove(this.resourceId);

        this.$emit('error');
        this.$emit('finished');
      }
    },
    post() {
      this.$emit('loading');

      if (_.isEmpty(this.resourceName) || _.isEmpty(this.resourceId) || _.isEmpty(this.field.key)) {
        return;
      }

      window.setTimeout(() => {
        this.loading = true;
      }, 200);

      return Nova.request().post(
        `/nova-vendor/dnwjn/nova-button/${this.resourceName}/${this.resourceId}/${this.field.key}`,
        { event: this.field.event }
      );
    },
    navigate() {
      if (this.field.type === 'route') {
        this.$router.push(this.field.route);
      }

      if (this.field.type === 'link') {
        window.open(this.field.link.href, this.field.link.target);
      }
    },
  },
  computed: {
    buttonText() {
      if (this.field.link && this.field.link.target === '_blank') {
        return this.field.text;
      }

      if (this.error && this.field.errorText) {
        return this.field.errorText;
      }

      if (this.success && this.field.successText) {
        return this.field.successText;
      }

      if (this.loading && this.field.loadingText) {
        return this.field.loadingText;
      }

      return this.field.text;
    },
    buttonClasses() {
      if (this.field.link && this.field.link.target === '_blank') {
        return this.field.classes;
      }

      if (this.error && this.field.errorClasses.length) {
        return this.field.errorClasses + ' text-center nova-button-error';
      }

      if (this.success && this.field.successClasses.length) {
        return this.field.successClasses + ' text-center nova-button-success';
      }

      if (this.loading && this.field.loadingClasses) {
        return this.field.loadingClasses + ' text-center nova-button-loading';
      }

      return this.field.classes;
    },
  },
};
</script>
