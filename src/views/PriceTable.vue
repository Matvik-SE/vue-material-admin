<template>
  <v-container>
    <v-row>
      <v-col>
        <price-alert-form v-model="item" :type="type" @save="saveItem" @cancel="cancelItem" />
      </v-col>

      <v-col cols="7">
        <v-card tile>
          <v-card-text class="pa-0">
            <!--            {{ items }}-->
            <v-data-table
              :loading="loading"
              :headers="headers"
              :items="typeItems"
              :items-per-page="itemsPerPage"
              :items-per-page-options="[15, 30, 50]"
              :page.sync="currentPage"
              item-key="id"
            >
              <template #[`item.direction`]="{ item }">
                {{ item.direction ? 'Above' : 'Below' }}
              </template>
              <template #[`item.action`]="{ item }">
                <div>
                  <v-btn class="mx-2" x-small fab dark depressed color="primary" @click="editItem(item)">
                    <v-icon dark> mdi-pencil </v-icon>
                  </v-btn>
                  <v-btn class="mx-2" x-small fab dark depressed color="error" @click="deleteItem(item)">
                    <v-icon dark> mdi-close </v-icon>
                  </v-btn>
                </div>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import request from '@/util/request'
import TooltipMixin from '@/mixins/Tooltip'
import PriceAlertForm from '@/components/PriceAlertForm'

export default {
  components: { PriceAlertForm },
  mixins: [TooltipMixin],
  props: {
    type: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      loading: false,
      itemsPerPage: 15,
      currentPage: 1,
      headers: [
        {
          text: 'ID',
          value: 'id',
          align: 'center',
        },
        {
          text: 'Cryptocurrency',
          value: 'from_symbol',
          align: 'center',
        },
        {
          text: 'Above/Below',
          value: 'direction',
          align: 'center',
        },
        {
          text: 'Amount',
          value: 'threshold',
          align: 'center',
        },
        {
          text: 'Only once',
          value: 'target_coin',
          align: 'center',
        },
        {
          text: 'Action',
          value: 'action',
          align: 'center',
        },
      ],
      items: [],
      item: {},
    }
  },
  computed: {
    typeItems: function () {
      return this.items.filter((el) => el.alert_type.type === this.type)
    },
  },
  created() {
    this.fetchRecords()
  },
  methods: {
    async fetchRecords() {
      this.loading = true
      this.items = await request.get('/alerts')
      this.loading = false
    },

    editItem(item) {
      this.item = { ...item }
    },

    async saveItem(data) {
      let res

      // Custom message - user-alert.notes
      // Only once - user-alert.only_once
      if (data.id) {
        res = await request.put(`/alerts/${data.id}`, data)
      } else {
        res = await request.post('/alerts', { alert_type: this.type, ...data })
      }
      this.item = {}
      this.fetchRecords()
    },

    deleteItem({ id }) {
      console.log('deleteItem')
      this.fetchRecords()
    },

    cancelItem() {
      this.item = {}
    },
  },
}
</script>
