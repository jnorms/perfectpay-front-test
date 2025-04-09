<script setup lang="ts">
const form = reactive({
  name: 'Nelson Vicente Mendes',
  document: '28695290077',
  email: 'nelson_vicente_mendes@madhause.com.br',
  mobilePhone: '83988995231',
  publicPlace: 'Rua José Ulisses de Lucena',
  number: '1234',
  complement: null,
  neighborhood: 'Mirante',
  postcode: '58407688',
  paymentType: '',
  holderName: 'marcelo h almeida',
  cardNumber: '5162306219378829',
  expiryMonth: '05',
  expiryYear: '2025',
  ccv: '318',
})

const order = reactive({
  hasError: false,
  message: '',
  orderSuccess: false,
  data: {}
})
async function sendOrder() {

  let body = {
    "client": {
      "name": form.name,
          "document": form.document,
          "email": form.email,
          "mobile_phone": form.mobilePhone,
          "address": {
        "public_place": form.publicPlace,
            "number": form.number,
            "neighborhood": form.neighborhood,
            "postcode": form.postcode,
            "complement": form.complement
      }
    },
    "billing_type": form.paymentType
  }

  if (form.paymentType === 'credit_card') {
    body = {
      ...body,
      ...{
        "credit_card_infos": {
          "holder_name": form.holderName,
          "number": form.cardNumber,
          "expiry_month": form.expiryMonth,
          "expiry_year": form.expiryYear,
          "ccv": form.ccv,
        }
      }
    }
  }

  const { data, error } = await useFetch('http://localhost:80/api/order', {
    method: 'POST',
    body: JSON.stringify(body)
  })

  if (error.value !== null) {
    order.hasError = true
    order.orderSuccess = false
    order.message = error.value.data.message
  }

  if (error.value === null && data.value !== null) {
    order.hasError = false
    order.orderSuccess = true
    order.data = data.value || {}
  }
}
</script>

<template>
  <div class="flex items-center w-full h-screen px-24">
    <div class="card bg-base-300 rounded-box grid grow p-10 place-items-center">
      <div role="alert" class="alert alert-error w-full mb-5" v-if="order.hasError">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 shrink-0 stroke-current" fill="none" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <span>{{ order.message }}</span>
      </div>
      <div class="flex gap-x-4" v-if="!order.orderSuccess">
        <fieldset class="fieldset w-xs bg-base-200 border border-base-300 p-4 rounded-box">
          <legend class="fieldset-legend">Dados Pessoais</legend>

          <label class="fieldset-label">Nome</label>
          <input v-model="form.name" type="text" class="input"/>

          <label class="fieldset-label">Documento</label>
          <input v-model="form.document" type="text" class="input" placeholder="CPF ou CNPJ"/>

          <label class="fieldset-label">Email</label>
          <input v-model="form.email" type="email" class="input" placeholder="Email"/>

          <label class="fieldset-label">Celular</label>
          <input v-model="form.mobilePhone" type="tel" class="input" placeholder="Celular"/>
        </fieldset>
        <fieldset class="fieldset w-xs bg-base-200 border border-base-300 p-4 rounded-box">
          <legend class="fieldset-legend">Endereço</legend>

          <label class="fieldset-label">Logradouro</label>
          <input v-model="form.publicPlace" type="text" class="input"/>

          <label class="fieldset-label">Número</label>
          <input v-model="form.number" type="text" class="input"/>

          <label class="fieldset-label">Complemento</label>
          <input v-model="form.complement" type="text" class="input"/>

          <label class="fieldset-label">Bairro</label>
          <input v-model="form.neighborhood" type="text" class="input"/>

          <label class="fieldset-label">CEP</label>
          <input v-model="form.postcode" type="text" class="input" size="8"/>
        </fieldset>
        <fieldset class="fieldset w-xs bg-base-200 h-30 border border-base-300 p-4 rounded-box">
          <legend class="fieldset-legend">Formas de Pagamento</legend>
          <select class="select" v-model="form.paymentType">
            <option disabled selected>Escolha a forma de pagamento</option>
            <option value="pix">PIX</option>
            <option value="boleto">Boleto</option>
            <option value="credit_card">Cartão de Crédito</option>
          </select>
          <label v-if="form.paymentType === 'credit_card'" class="fieldset-label">Nome do Titular (Impresso no cartão)</label>
          <input v-if="form.paymentType === 'credit_card'" v-model="form.holderName" type="text" class="input"/>
          <label v-if="form.paymentType === 'credit_card'" class="fieldset-label">Número do Cartão</label>
          <input v-if="form.paymentType === 'credit_card'" v-model="form.cardNumber" type="text" class="input"/>
          <label v-if="form.paymentType === 'credit_card'" class="fieldset-label">Mês de Expiração</label>
          <input v-if="form.paymentType === 'credit_card'" v-model="form.expiryMonth" type="text" class="input" size="2"/>
          <label v-if="form.paymentType === 'credit_card'" class="fieldset-label">Ano de Expiração</label>
          <input v-if="form.paymentType === 'credit_card'" v-model="form.expiryYear" type="text" class="input" size="4"/>
          <label v-if="form.paymentType === 'credit_card'" class="fieldset-label">CVV</label>
          <input v-if="form.paymentType === 'credit_card'" v-model="form.ccv" type="text" class="input" minlength="3" maxlength="4"/>
        </fieldset>
        <button class="btn btn-accent self-end" @click="() => sendOrder()">Pagar</button>
      </div>
      <div class="flex w-full" v-if="order.orderSuccess">
        <div>
          <h2 class="text-2xl font-bold mb-10">Pedido Gerado com Sucesso - {{order.data.uuid}}</h2>

          <div v-if="!!order.data.payment_infos.boleto">
            <h3 class="card-title">Código de Barras</h3>
            <p>{{ order.data.payment_infos.boleto.bar_code }}</p>
          </div>

          <div v-if="!!order.data.payment_infos.pix">
            <h3 class="card-title">PIX copia e cola</h3>
            <p>{{ order.data.payment_infos.pix.copy_and_paste }}</p>
            <h3 class="card-title mt-5">QR Code</h3>
            <img alt="QR Code" class="mx-auto" :src="'data:image/jpeg;base64,'+order.data.payment_infos.pix.qr_code">
          </div>
        </div>
      </div>
    </div>
    <div class="divider divider-horizontal" v-if="!order.orderSuccess"></div>
    <div class="card bg-base-300 rounded-box grid grow py-10 place-items-center" v-if="!order.orderSuccess">
      <div class="card w-96 card-xs">
        <div class="card-body">
          <h2  class="font-bold mb-5 text-3xl">Pedido</h2>
          <h2 class="card-title">BMW Miniatura 1:9</h2>
          <p>R$ 235,00</p>
          <h2 class="card-title">Ducati Miniatura 1:12</h2>
          <p>R$ 100,00</p>
          <p class="font-bold mt-10 text-lg">Total: R$ 335,00</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>