<template>
  <div>
    <component :is="adminLayout" v-if="userId || $options.gatewayMode === 'modal'">
      <v-alert
          slot="page-header"
          :value="showEmailUnverified"
          color="error"
      >
        Please verify your e-mail address. This is required before you can do stuff in this system.
      </v-alert>

      <slot />
      <GatewayModal />
    </component>
    <component :is="publicLayout" v-if="publicLayout && !userDetailsLoaded && $options.gatewayMode === 'redirect'">
      <slot />
    </component>
  </div>
</template>

<script>
  import authConfig from '../config';
  import AuthMixin from '../mixins/auth';

  /**
   * Depending on the 'gatewayMode' setting.
   */
  export default {
    mixins: [AuthMixin],

    props: {
      adminLayout: Function,
      publicLayout: { type: Function, default: null },
    },

    components: {
      GatewayModal: () => import('./GatewayModal'),
    },

    data: () => ({
      requiresAuth: true,
    }),

    mounted() {
      this.$options.gatewayMode = authConfig.gatewayMode;
      if(!this.publicLayout && authConfig.gatewayMode === 'redirect') {
        throw new Error('The "publicLayout" property is required on the-gateway component if gatewayMode is "redirect"')
      }

      if (this.userDetailsLoaded) {
        this.$store.dispatch('notify', { text: `Welcome back ${this.$store.state.user.profile.displayName}` });
      }
    },

    computed: {
      showEmailUnverified() {
        const isLoading = !this.$store.state.user.userDetailsLoaded;

        return !isLoading && !this.$store.state.user.isEmailVerified;
      },
      userDetailsLoaded() {
        return this.$store.state.user.userDetailsLoaded;
      },
      userId() {
        return this.$store.state.user.userId;
      },
    },
    watch: {
      userDetailsLoaded(userDetailsLoaded) {
        if (userDetailsLoaded) {
          this.$store.dispatch('notify', { text: `Welcome back ${this.$store.state.user.profile.displayName}` });
        }
      },
    },
  };
</script>

<style>
  input:-webkit-autofill,
  input:-webkit-autofill:hover,
  input:-webkit-autofill:focus,
  textarea:-webkit-autofill,
  textarea:-webkit-autofill:hover,
  textarea:-webkit-autofill:focus,
  select:-webkit-autofill,
  select:-webkit-autofill:hover,
  select:-webkit-autofill:focus {
    -webkit-box-shadow: 0 0 0 1000px #ffffff inset;
    transition: background-color 5000s ease-in-out 0s;
  }
</style>
