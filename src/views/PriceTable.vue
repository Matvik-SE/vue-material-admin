<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-card tile>
          <v-card-text class="pa-0">
            {{ items }}
            <v-data-table
              :loading="loadingItems"
              :headers="headers"
              :items="items"
              :items-per-page="itemsPerPage"
              :items-per-page-options="[15, 30, 50]"
              :page.sync="filter['page']"
              item-key="id"
            >
              <template #[`item.action`]="{ item }">
                <v-menu>
                  <template #activator="{ on: menu }">
                    <v-tooltip bottom>
                      <template #activator="{ on: tooltip }">
                        <v-btn icon v-on="onTooltip({ ...tooltip, ...menu })">
                          <v-icon>mdi-dots-vertical</v-icon>
                        </v-btn>
                      </template>
                      <span>Action</span>
                    </v-tooltip>
                  </template>
                  <v-list class="pa-0" dense>
                    <v-list-item v-for="action in actions" :key="action.text" @click="action.click(item)">
                      <v-list-item-icon class="mr-2">
                        <v-icon small>{{ action.icon }}</v-icon>
                      </v-list-item-icon>
                      <v-list-item-title>{{ action.text }}</v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-dialog v-model="showDialog" scrollable width="840">
      <price-alert-form :item="selectedItem" @form:success="handleFormSuccess" />
    </v-dialog>
  </v-container>
</template>

<script>
import request from '@/util/request'
import TooltipMixin from '@/mixins/Tooltip'
import PriceAlertForm from '@/components/PriceAlertForm'

export default {
  components: { PriceAlertForm },
  mixins: [TooltipMixin],
  data() {
    return {
      showDialog: false,
      search: '',
      loadingItems: false,
      selectedItem: null,
      serverItemsLength: 0,
      itemsPerPage: 15,
      showFilter: true,
      filter: {
        page: 1,
        'filter[name]': null,
        'filter[project_id]': null,
        'filter[status]': null,
      },
      headers: [
        {
          text: 'ID',
          value: 'id',
        },
        {
          text: 'Cryptocurrency',
          value: 'coin',
        },
        {
          text: 'Above/Below',
          value: 'direction',
        },
        {
          text: 'Amount',
          value: 'threshold',
        },
        {
          text: 'Custom message',
          value: 'user_alerts',
        },
        {
          text: 'Only once',
          value: 'alert_type',
        },
        {
          text: 'Action',
          value: 'action',
        },
      ],
      items: [],
      actions: [
        {
          text: 'Edit Item',
          icon: 'mdi-pencil',
          click: this.handleEditItem,
        },
        {
          text: 'Delete Item',
          icon: 'mdi-close',
          click: this.handleDeleteItem,
        },
      ],
    }
  },
  created() {
    this.fetchRecords()
  },
  methods: {
    async fetchRecords() {
      this.loadingItems = true
      this.items = []
      this.items = await request.get('/alerts')
      this.loadingItems = false
    },

    handleFormSuccess(event) {
      console.log('handleFormSuccess', event)
    },

    //
    //
    //
    updateFilterQuery(query) {
      const filter = Object.assign(this.filter, this.transformQuery(query))
      return filter
    },
    transformQuery(query) {
      const numbers = ['filter[project_id]', 'filter[status]', 'page']
      for (let key in query) {
        if (numbers.includes(key)) {
          const val = query[key] ? parseInt(query[key]) : query[key]
          query[key] = val
        }
      }
      return query
    },

    //action
    handleCreateItem() {
      this.selectedItem = null
      this.showDialog = true
    },
    handleEditItem(item) {
      this.selectedItem = item
      this.showDialog = true
    },
    handleDeleteItem({ id }) {
      if (window.confirm('Are you sure to delete this')) {
        this.$store.dispatch('deleteTask', id).then(() => {
          this.items = this.items.filter((item) => item.id != id)
        })
      }
    },
    handleRefreshItem() {
      this.fetchRecords(this.filter)
    },
  },
}
</script>
