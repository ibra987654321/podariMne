<template>
  <v-data-table
    :items="itemsWithIndex"
    :headers="header"
    :loading="$store.state.loading"
    :footer-props="pagination"
  >
    <slot></slot>
    <template v-slot:top>
      <div class="padding">
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              v-if="$props.postDispatch"
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              {{ formTitle }}
            </v-btn>
          </template>
          <use-dialog-content-inner
            :title="formTitle"
            :edit-block="$props.showToEdit"
            :edited-item="editedItem"
            @close="close()"
            @save="save($event)"
          ></use-dialog-content-inner>
        </v-dialog>
        <v-dialog
          v-model="dialogDelete"
          max-width="500px"
        >
          <v-card>
            <v-card-title class="text-h5">
              Вы действительно хотите удалить?
            </v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="closeDelete"
              >
                Отмена
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="deleteItemConfirm"
              >
                OK
              </v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </div>
    </template>
    <template
      v-if="statusFilter"
      v-slot:[`item.${$props.filters.status}`]="{item}"
    >
      {{ item[$props.filters.status] | status }}
    </template>
    <template
      v-for="(Date, i) in $props.filters.date"
      v-if="dateFilter"
      v-slot:[`item.${Date}`]="{item}"
    >
      <div v-if="item[$props.filters.date[i]] !== null">
        {{ item[$props.filters.date[i]] | date}}
      </div>
      <div v-else></div>
    </template>
    <template
      v-for="(min, idx) in $props.filters.minute"
      v-if="minuteFilter"
      v-slot:[`item.${min}`]="{item}"
    >
      <div v-if="item[$props.filters.minute[idx]] !== null">
        {{ item[$props.filters.minute[idx]] | time}}
      </div>
      <div v-else></div>
    </template>
    <template
      v-for="(check, idx) in $props.filters.checkbox"
      v-if="checkFilter"
      v-slot:[`item.${check}`]="{item}"
    >
      <v-checkbox
        v-model="item[$props.filters.checkbox[idx]]"
        disabled
      ></v-checkbox>
    </template>
    <template v-slot:item.detail="{item}">
      <v-btn
        color="primary"
        small
        outlined
        rounded
        @click="$router.push($props.detailPage + item.id)"
      >
        Подробнее
      </v-btn>
    </template>
<!--    <template v-slot:item.more="{item}">-->
<!--      <full-screen-modal-->
<!--        :title="formTitle"-->
<!--        :edited-item="item"-->
<!--        :show-to-edit="$props.showToEdit"-->
<!--        @save="save($event)"-->
<!--      ></full-screen-modal>-->
<!--    </template>-->
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        {{ icons.edit }}
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        {{ icons.deleted }}
      </v-icon>
    </template>
  </v-data-table>
</template>

<script>
const useDialogContentInner = () => import('@/components/useDashboard/useDialogContentInner')
import {
  mdiCheckOutline,
  mdiClipboardOutline,
  mdiClipboardPlayOutline, mdiCloseOutline,
  mdiDelete,
  mdiPen,
} from '@mdi/js'

export default {
  name: 'useDataTable',
  components: {
    useDialogContentInner,
  },
  props: {
    headers: Array,
    showToEdit: Array,
    filters: Object,
    editedItems: Object,
    putDispatch: String,
    deleteDispatch: String,
    postDispatch: String,
    data: Array,
    detailPage: String,
    itemsPerPageOptions: {
      type: Array,
      default(){
        return [15,20,50]
      }
    },
  },
  data: () => ({
    menu1: false,
    dialog: false,
    dialogDelete: false,
    editedIndex: -1,
    editedItem: {},
    icons: {
      edit: mdiPen,
      deleted: mdiDelete,
      mdiCheckOutline,
      mdiClipboardOutline,
      mdiClipboardPlayOutline,
      mdiCloseOutline,
    },
    pagination: {}
  }),
  computed: {
    header() {
      this.pagination.itemsPerPageOptions = this.$props.itemsPerPageOptions
      return this.$props.headers
    },
    formTitle() {
      return this.editedIndex === -1 ? 'Создать' : 'Изменить'
    },
    dateFilter() {
      if (this.$props.filters) {
        if (this.$props.filters.date) {
          return this.$props.filters.date
        }
      }
    },
    minuteFilter() {
      if (this.$props.filters) {
        if (this.$props.filters.minute) {
          return this.$props.filters.minute
        }
      }
    },
    statusFilter() {
      if (this.$props.filters) {
        if (this.$props.filters.status) {
          return this.$props.filters.status
        }
      }
    },
    checkFilter() {
      if (this.$props.filters) {
        if (this.$props.filters.checkbox) {
          return this.$props.filters.checkbox
        }
      }
    },
    itemsWithIndex() {
      return this.$props.data.map(
        (items, index) => ({
          ...items,
          index: index + 1,
        }),
      )
    },
  },
  watch: {
    dialog(val) {
      if(val && this.formTitle === 'Создать') {
        this.$emit('newItem')
      }
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },
  mounted() {
    this.editedItem = this.$props.editedItems
  },
  methods: {
    editItem(item) {
      this.editedIndex = this.itemsWithIndex.indexOf(item)
      this.editedItem = { ...item }
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.itemsWithIndex.indexOf(item)
      this.editedItem = { ...item }
      this.dialogDelete = true
    },
    deleteItemGroup(item) {
      this.editedIndex = this.itemsWithIndex.indexOf(item)
      this.editedItem = { ...item }
      this.deleteItemConfirm()
    },

    deleteItemConfirm() {
      const item = this.itemsWithIndex.splice(this.editedIndex, 1)
      this.$store.dispatch(this.$props.deleteDispatch, ...item)
      this.data.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = { ...this.defaultItem }
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = { ...this.defaultItem }
        this.editedIndex = -1
      })
    },

    async save(event) {
      if (this.editedIndex > -1) {
        await this.$store.dispatch(this.putDispatch, this.editedItem)
        Object.assign(this.itemsWithIndex[this.editedIndex], this.editedItem)
        this.close()
      } else {
        if (this.$props.filters) {
          if (this.$props.filters.date) {
            event[this.$props.filters.date] = (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)
          }
        }
        await this.$store.dispatch(this.postDispatch, event)
          .then(r => {
            if (r !== undefined) {
              this.data.unshift(r.data)
              this.close()
            }
          })
      }
    },
  },
}
</script>

<style scoped>

</style>
