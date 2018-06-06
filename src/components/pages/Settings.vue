<template>
  <div class="app-page settings-page">
    <div class="section">
      <div class="container is-narrow">
        <div class="card app-card main-app-card">
          <div class="card-header">
            <h1 class="card-header-title">Settings</h1>
          </div>
          <div class="card-content">
            <v-form id="save-settings">

              <v-select v-model="newSettings.fiatCurrency"
                        label="Fiat Currency"
                        :options="availableCurrencies" />

              <v-button id="save-button"
                        :disabled="!isFormValid"
                        @click.prevent="updateSettings(newSettings)">Save</v-button>

            </v-form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex';
import VForm from '@/components/ui/form/VForm.vue';
import VSelect from '@/components/ui/form/VSelect.vue';
import VButton from '@/components/ui/form/VButton.vue';

export default {
  name: 'settings-page',
  data: () => ({
    newSettings: {},
  }),
  computed: {
    ...mapState('accounts', ['settings', 'availableCurrencies']),
    isFormValid() {
      return JSON.stringify(this.newSettings) !== JSON.stringify(this.settings);
    },
  },
  methods: {
    ...mapActions('accounts', {
      updateSettingsInStore: 'updateSettings',
    }),
    updateSettings(settings) {
      this.updateSettingsInStore(settings)
        .then(() => {
          this.$notify({
            title: 'Successful',
            text: 'Settings was saved',
            type: 'is-info',
          });
        })
        .catch(e => {
          this.$notify({
            title: e.title,
            text: e.text,
            type: 'is-warning',
          });
        });
    },
  },
  components: {
    VForm,
    VSelect,
    VButton,
  },
  mounted() {
    // this.newSettings = this.settings;
    try {
      this.newSettings = JSON.parse(JSON.stringify(this.settings));
    } catch (e) {
      console.error(e);
    }
  },
};
</script>

<style lang="scss">
</style>