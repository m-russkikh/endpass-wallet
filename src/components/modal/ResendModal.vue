<template lang="html">
  <div
    class="new-account-modal"
    data-test="resend-modal"
  >
    <v-modal @close="close">
      <v-form
        id="sendEther"
        @submit="confirmResend"
        :isFormValid="isFormValid"
        >
        <v-input
          id="gasPrice"
          v-model="newTransaction.gasPrice"
          label="Gas price"
          name="gasPrice"
          data-vv-name="gasPrice"
          type="number"
          aria-describedby="gasPrice"
          placeholder="Gas price"
          data-test="gas-price-input"
          v-validate="`required|numeric|integer|between:${minimumGasPrice},100`"
          :error="errors.first('gasPrice')"
          autofocus
          required
        >
          <div
            slot="addon"
            class="control"
          >
            <a class="button is-static">Gwei</a>
          </div>
        </v-input>
        <v-input
          id="gasLimit"
          v-model="newTransaction.gasLimit"
          v-validate="`required|numeric|integer|between:${transaction.gasLimit},1000000`"
          :error="errors.first('gasLimit')"
          label="Gas limit"
          name="gasLimit"
          data-vv-name="gasLimit"
          type="number"
          aria-describedby="gasLimit"
          placeholder="Gas limit"
          data-test="gas-limit-input"
          required
        />

      </v-form>
      <div
        slot="footer"
        class="buttons"
      >
        <v-button
          form="sendEther"
          class-name="is-primary is-medium"
          data-test="submit-button"
          :disabled="!isFormValid"
        >
          Send
        </v-button>
      </div>
    </v-modal>
  </div>
</template>

<script>
import formMixin from '@/mixins/form';
import { Transaction } from '@/class';

export default {
  name: 'ResendModal',

  props: {
    transaction: {
      type: Object,
      required: true,
    },
  },

  data() {
    return {
      newTransaction: null,
    };
  },
  computed: {
    minimumGasPrice() {
      return Math.floor(Number(this.transaction.gasPrice) + 1);
    },
  },
  methods: {
    confirmResend() {
      this.$emit('confirm', this.newTransaction);
    },
    close() {
      this.$emit('close');
    },
  },
  created() {
    this.newTransaction = {
      ...Transaction.clone(this.transaction),
      gasPrice: this.minimumGasPrice,
    };
  },
  mixins: [formMixin],
};
</script>

<style lang="css">
</style>
