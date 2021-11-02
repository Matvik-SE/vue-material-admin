<template>
  <v-sheet color="white" class="pa-5 elevation-2">
    <!--    {{ model }}-->
    <v-form ref="form" v-model="valid">
      <v-select
        v-model="model.from_symbol"
        :rules="[(v) => !!v || 'Is required']"
        :items="coins"
        :loading="loading"
        label="Cryptocurrency"
        item-value="symbol"
        item-text="symbol"
        dense
      />

      <v-row>
        <v-col cols="7">
          <span class="caption">Goes by</span>
          <v-radio-group
            v-model="model.direction"
            :rules="[(v) => !!v || 'Is required']"
            row
            dense
            style="margin-top: -5px"
          >
            <v-radio label="Above" :value="true"></v-radio>
            <v-radio label="Below" :value="false"></v-radio>
          </v-radio-group>
        </v-col>
        <v-col>
          <v-text-field
            v-model="model.threshold"
            :rules="[(v) => !!v || 'Is required']"
            dense
            label="Amount"
            hint="USD"
            class="mt-3"
          />
        </v-col>
      </v-row>

      <v-text-field v-if="type === 'Price'" v-model="model.notes" dense label="Custom message" />
      <v-select
        v-else
        v-model="model.Time_scope"
        :items="timeValues"
        :label="type === 'Percent' ? 'Time scope' : 'Frequency'"
        :rules="[(v) => !!v || 'Is required']"
        dense
      />

      <v-checkbox v-model="model.only_once" dense label="Disable this alert once it triggers" />

      <div class="d-flex justify-space-around">
        <v-btn color="primary" @click="handleSubmit">{{ model.id ? 'Save' : 'Create' }} Alert</v-btn>
        <v-btn color="secondary" @click="handleCancel">Cancel</v-btn>
      </div>
    </v-form>
  </v-sheet>
</template>

<script>
import request from '@/util/request'

export default {
  props: {
    value: {
      type: Object,
      default: () => {},
    },
    type: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      timeValues: [
        '5 min',
        '10 min',
        '15 min',
        '30 min',
        '1 hour',
        '3 hours',
        '6 hours',
        '12 hours',
        '1 day',
        '7 days',
      ],
      loading: false,
      valid: true,
      coins: [],
    }
  },
  computed: {
    model: {
      get: function () {
        return this.value
      },
      set: function (value) {
        this.$emit('input', value)
      },
    },
  },
  async created() {
    this.loading = true
    this.coins = await request.get('/coins')
    this.loading = false
  },
  methods: {
    handleSubmit() {
      const form = this.$refs.form

      if (form.validate()) {
        console.log('valid')
        this.$emit('save', this.model)
      }
    },

    handleCancel() {
      this.$refs.form.resetValidation()
      this.$emit('cancel')
    },
  },
}
</script>
