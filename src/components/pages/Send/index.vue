<template>
  <div class="app-page send-page">
    <div class="section is-narrow">
      <div class="container is-narrow">
        <div class="card app-card main-app-card">
          <div class="card-header">
            <h1 class="card-header-title">Send ETH</h1>
          </div>
          <div class="card-content">
            <transaction-form
              :transaction="transaction"
              @submit="handleTransactionSend"
            />
            <div
              v-if="transactionHash"
              class="transaction-status message is-success"
              data-test="transaction-status"
            >
              <div class="message-header">
                <p>Transaction Sent!</p>
              </div>
              <div class="message-body">
                <p>
                  Your transaction has been broadcast to the network. It may
                  take a few minutes before the transaction is confirmed.
                </p>
                <p class="label">Transaction Id</p>
                <p class="code">{{ transactionHash }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <transaction-modal
      v-if="isWaitingConfirm"
      :transaction="transaction"
      @confirm="confirmTransaction"
      @close="cancelTransaction"
    />
    <password-modal
      v-if="isTransactionConfirmed"
      @confirm="sendConfirmedTransaction"
      @close="cancelTransaction"
    />
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex';
import { TransactionFactory } from '@/class';
import AccountChooser from '@/components/AccountChooser';
import TransactionModal from '@/components/modal/TransactionModal';
import PasswordModal from '@/components/modal/PasswordModal';
import privatePage from '@/mixins/privatePage';
import { getShortStringWithEllipsis } from '@endpass/utils/strings';
import TransactionForm from './TransactionForm.vue';

const defaultTx = {
  token: null,
  gasPrice: '40',
  gasLimit: '22000',
  value: '0',
  nonce: 0,
  to: '',
  from: '',
  data: '0x',
};

export default {
  data: () => ({
    transaction: { ...defaultTx },
    transactionHash: null,
    isSending: false,
    isWaitingConfirm: false,
    isTransactionConfirmed: false,
  }),

  computed: {
    ...mapState({
      address: state => state.accounts.address,
      activeNet: state => state.web3.activeNet,
    }),
  },

  watch: {
    async address(newValue, prevValue) {
      if (newValue === prevValue) return;

      this.transaction.from = newValue;

      await this.updateNonceWithClearHash();
    },

    async activeNet(newValue, prevValue) {
      if (newValue.id === prevValue.id) return;

      await this.updateNonceWithClearHash();
    },
  },

  methods: {
    ...mapActions('transactions', ['sendTransaction', 'getNextNonce']),
    ...mapActions('gasPrice', ['getGasPrice']),

    handleTransactionSend() {
      this.isWaitingConfirm = true;
    },

    cancelTransaction() {
      this.isWaitingConfirm = false;
      this.isTransactionConfirmed = false;
    },

    async confirmTransaction() {
      this.isWaitingConfirm = false;
      this.isTransactionConfirmed = true;
    },

    async sendConfirmedTransaction(password) {
      this.isSending = true;

      Object.assign(this.transaction, {
        from: this.address,
        networkId: this.activeNet.id,
      });

      this.resetForm();
      this.isTransactionConfirmed = false;
      this.isWaitingConfirm = false;

      try {
        const trx = TransactionFactory.fromSendForm(this.transaction);
        const hash = await this.sendTransaction({
          transaction: trx,
          password,
        });
        const shortHash = getShortStringWithEllipsis(hash);

        this.transactionHash = hash;

        this.$notify({
          title: 'Transaction Sent',
          text: `Transaction ${shortHash} sent`,
          type: 'is-info',
        });
      } catch (err) {
        this.transactionHash = null;
      } finally {
        this.isSending = false;
      }
    },

    async handleTransactionFormSubmit() {
      this.toggleTransactionModal();
    },

    async resetForm() {
      const nonce = await this.getNextNonce();

      this.transaction = {
        ...defaultTx,
        from: this.address,
        nonce,
      };
    },

    async updateNonceWithClearHash() {
      if (this.transactionHash) {
        this.transactionHash = null;
      }

      this.transaction.nonce = await this.getNextNonce();
    },
  },

  async created() {
    this.transaction.nonce = await this.getNextNonce();
    this.transaction.from = this.address;
  },

  mixins: [privatePage],

  components: {
    AccountChooser,
    TransactionModal,
    PasswordModal,
    TransactionForm,
  },
};
</script>

<style lang="scss">
.send-page {
  .send-amount {
    margin-top: 2em;
    margin-bottom: 2em;
  }

  .field-label {
    margin-bottom: 0;
  }
}
</style>
