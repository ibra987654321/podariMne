<template>
  <v-dialog
    transition="dialog-bottom-transition"
    max-width="900"
    v-model="dialog"
  >
    <template v-slot:activator="{ on, attrs }">
      <v-btn
        color="primary"
        v-bind="attrs"
        v-on="on"
        :small="$vuetify.breakpoint.mobile"
      >{{$props.textBtn}}</v-btn>
    </template>
    <template v-slot:default="dialog">
      <v-card>
        <div class="d-flex">
          <v-card-title class="mb-8" >Добавление </v-card-title>
          <v-card-title class="mb-8" >
            <v-dialog
              transition="dialog-bottom-transition"
              max-width="600"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  v-bind="attrs"
                  v-on="on"
                  small
                >Сколько осталось</v-btn>
              </template>
              <template v-slot:default="dialog">
                <v-card>
                  <v-toolbar
                    color="primary"
                    dark
                  >Количество товаров в этом филиале</v-toolbar>
                  <v-card-text>
                    <div class="pa-4">
                      <v-chip-group
                        active-class="primary--text"
                        column
                      >
                        <v-chip
                          v-for="(tag, idx) in goods"
                          :key="idx"
                        >
                          {{ tag.name }} | {{tag.amount}}
                        </v-chip>
                      </v-chip-group>
                    </div>
                  </v-card-text>
                  <v-card-actions class="justify-end">
                    <v-btn
                      text
                      @click="dialog.value = false"
                    >Закрыть</v-btn>
                  </v-card-actions>
                </v-card>
              </template>
            </v-dialog>
          </v-card-title>
        </div>

        <v-card-text class="d-flex justify-space-around">
          <v-row>
            <v-col cols="12" sm="6">
              <v-select
                v-model="item.categoryId"
                :items="category"
                item-value="id"
                item-text="name"
                outlined
                :dense="$vuetify.breakpoint.mobile"
                hide-details
                label="Тип"
              ></v-select>
            </v-col>
            <v-col cols="12" sm="6">
              <v-select
                v-model="selected"
                :items="productName"
                item-value="id"
                item-text="name"
                outlined
                :loading="categoryIdLoading"
                :dense="$vuetify.breakpoint.mobile"
                hide-details
                label="Название"
              ></v-select>
            </v-col>
            <v-col cols="12" sm="4">
              <v-select
                v-model="item.status"
                outlined
                :dense="$vuetify.breakpoint.mobile"
                hide-details
                :items="['Свободный', 'Заказ']"
                label="Статус"
              ></v-select>
            </v-col>
            <v-col cols="8" sm="4" v-if="item.status === 'Заказ'">
              <v-text-field
                v-model="item.orderedBy"
                outlined
                :dense="$vuetify.breakpoint.mobile"
                hide-details
                label="Кто"
              ></v-text-field>
            </v-col>
            <v-col cols="8" sm="4">
              <v-text-field
                v-model="item.pieces"
                outlined
                :dense="$vuetify.breakpoint.mobile"
                type="number"
                hide-details
                label="Количество"
              ></v-text-field>
            </v-col>
            <v-col cols="12">
              <v-divider></v-divider>
              <div class="text-sm-h5 mt-4 text-center">Продукты</div>
            </v-col>
            <v-col cols="12">
              <v-row v-for="(ingredient, idx) in item.ingredients">
                <v-col cols="12" sm="3">
                  <v-select
                    v-model="ingredient.categoryM"
                    label="Тип"
                    :items="category"
                    item-value="id"
                    item-text="name"
                    hide-details
                    outlined
                    :dense="$vuetify.breakpoint.mobile"
                    class="mr-2"
                    @change="handleIngredientChange($event, 'ingredients', 'getProduct')"
                  ></v-select>
                </v-col>
                <v-col cols="12" sm="5">
                  <v-select
                    v-model="ingredient.productId"
                    label="Товар"
                    :items="ingredient.productItems || []"
                    item-value="id"
                    item-text="name"
                    hide-details
                    outlined
                    :dense="$vuetify.breakpoint.mobile"
                    :loading="ingredient.loading"
                    class="mr-2"
                  ></v-select>
                </v-col>
                <v-col cols="8" sm="2">
                  <v-text-field
                    v-model="ingredient.amount"
                    outlined
                    :dense="$vuetify.breakpoint.mobile"
                    type="number"
                    hide-details
                    label="Количество"
                  ></v-text-field>
                </v-col>
                <v-col cols="4" sm="2" class="d-flex align-center">
                  <v-btn :fab="$vuetify.breakpoint.mobile" color="primary" :small="$vuetify.breakpoint.mobile" @click="item.ingredients.splice(idx, 1)">    <v-icon>{{icons.mdiMinus}}</v-icon></v-btn>
                </v-col>
              </v-row>
              <v-col cols="12">
                <v-btn color="primary" :class="$vuetify.breakpoint.mobile && 'mt-4'" :style="$vuetify.breakpoint.mobile ? 'width: 100%' : ''" @click="addIngredient">
                  <v-icon>{{icons.mdiPlus}}</v-icon>
                </v-btn>
              </v-col>
            </v-col>
            <v-divider></v-divider>
            <div class="text-sm-h5 mx-auto">Расходные материалы</div>
            <v-col cols="12">
              <v-row>
                <v-col cols="3"  v-for="check in materialComponent">
                  <v-checkbox :key="check.id"
                              v-model="check.selected"
                              :label="check.name"
                              @change="getSelectedItems(check)"
                  ></v-checkbox>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="6">
<!--              <input type="file" @change="handleFileChange" accept="image/*"  />-->
              <v-file-input outlined label="Выбрать фото" @change="handleFileChange"  ref="fileInput" type="file" ></v-file-input>
            </v-col>
          </v-row>
        </v-card-text>
        <v-card-actions class="justify-space-between">
          <v-btn
            text
            @click="dialog.value = false"
          >Закрыть</v-btn>
          <v-btn
            color="primary"
            @click="save"
            :loading="loading"
            :disabled="!valid"
          >Создать</v-btn>
        </v-card-actions>
      </v-card>
    </template>
  </v-dialog>
</template>

<script>
import {mdiMinus, mdiPlus} from '@mdi/js'
import { getToken } from '@/helpers/helpers'
export default {
  name: 'createDialog',
  props: {
    textBtn: String,
    propItem: Object,
    propItem1: Object,
  },
  data:() => ({
    dialog: false,
    loading: false,
    categoryIdLoading: false,
    productName: [],
    goods: [],
    materialComponent: [],
    categoryM: null,
    unitOfMeasurement: null,
    icons: {mdiMinus, mdiPlus},
    selected: '',
    item: {
      "name": "",
      "categoryId": 1,
      "status": "",
      "pieces": "",
      "orderedBy": "",
      "branchId": 0,
      "consumables": [],
      "ingredients": [],
    },
    cameraNotSupported: false,
    newMessage: {
      file: null
    },
    items: [],
    category: [],
    capturedPhoto: null,
    photoFile: null,
    materials: null,
    isFetchingProduct: false,
  }),
  mounted() {

  },
  computed: {
    valid() {
      return !!(this.item.name && this.item.status && this.item.consumables.length && this.item.ingredients.length);
    },
  },
  watch: {
    'dialog'(v) {
      if (v) {
        this.$store.dispatch('getCategory')
          .then(r => {
            this.category = r.data
            this.categoryM = 1
          })
          .catch(e => this.$store.commit('setSnackbars', e.message))
        this.$store.dispatch('getMaterial', this.item.categoryId)
          .then(r => {
            this.materials = r.data
          })
          .catch(e => this.$store.commit('setSnackbars', e.message))
        this.$store.dispatch('getProductName', this.item.categoryId)
          .then(r => {
            this.productName = r.data
          })
          .catch(e => this.$store.commit('setSnackbars', e.message))
        this.initEventSource()
      }
      if (this.propItem) {
        this.item.name = this.propItem.name
        this.item.productId = this.propItem.goods.product.id
        this.item.amount = this.propItem.amount
      }
    },
    'item.categoryId'(v) {
      this.categoryIdLoading = true
      this.$store.dispatch('getProductName', v)
        .then(r => {
          this.productName = r.data
          this.categoryIdLoading = false
        })
        .catch(e => {
          this.$store.commit('setSnackbars', e.message)
          this.categoryIdLoading = false
        })
    },
    selected(v) {
      v && this.$store.dispatch('getMaterialComponent', v)
        .then(r => {
          this.materialComponent = r.data.map((i) => {
            return {
              ...i,
              selected: false
            };
          })
        })
        .catch(e => {
          this.$store.commit('setSnackbars', e.message)
        })
      this.item.name = this.productName.find(i => i.id === v).name
    },
    'newMessage.file'() {
      // this.compressImage(this.newMessage.file, (1024 * 1024))
    }
  },
  methods: {
   save () {
      this.loading = true
      this.item.branchId = this.$route.params.id
      if (this.item.status !== 'Заказ') {
        this.item.orderedBy = ''
      }
      if (this.propItem) {
        if (this.newMessage.file !== null) {
          this.$store.dispatch('uploadImg',{
            file: this.newMessage.file,
            id: this.propItem.id
          })
            .then(() => {
              this.$emit('success', '')
              this.item = {
                "name": "",
                "categoryId": 1,
                "status": "",
                "branchId": 0,
                "pieces": "",
                "orderedBy": "",
                "consumables": [],
                "ingredients": [],
              }
              this.dialog = false
              this.loading = false
            })
          return
        }
        this.dialog = false
        this.loading = false
      } else {
        this.item.ingredients.map(i => {
          delete i.categoryM
          delete i.productItems
        })
        this.$store.dispatch('createSet', this.item)
          .then(async r => {
            if (this.newMessage.file !== null) {
              this.$store.dispatch('uploadImg',{
                file: await this.compressImage(this.newMessage.file, 1024 * 1024 * 3),
                id: r.data.id
              })
                .then(() => {
                  this.$emit('success', r.data)
                  this.item = {
                    "name": "",
                    "categoryId": 1,
                    "status": "",
                    "branchId": 0,
                    "pieces": "",
                    "orderedBy": "",
                    "consumables": [],
                    "ingredients": [],
                  }
                  this.$refs.fileInput.value = null;
                  this.selected = ''
                  this.dialog = false
                  this.loading = false
                  this.$store.commit('setSnackbars', 'Успешно создано');
                })
                .catch(() => {
                  this.dialog = false
                  this.loading = false
                  this.$store.commit('setSnackbars', 'Ошибка при загрузке картинки');
                })
              return
            }
            this.dialog = false
            this.loading = false
            this.$store.commit('setSnackbars', 'Успешно создано');
          })
          .catch(error => {
            const errorMessage = error.response ? error.response.data : 'Unknown error';
            this.$store.commit('setSnackbars', errorMessage);
            this.loading = false
          });
      }

    },
    handleFileChange(event) {
      const file = event;
      if (file) {
        this.newMessage.file = file;
        const reader = new FileReader();
        reader.onload = () => {
          this.capturedPhoto = reader.result;
        };
        reader.readAsDataURL(file);
      }
    },
    compressImage(file, targetSizeInBytes) {
      const maxQuality = 1.0;
      let currentQuality = 0.8; // Начальный уровень качества, можно настроить в соответствии с вашими требованиями

      return new Promise((resolve, reject) => {
        if (file.size <= targetSizeInBytes) {
          // Если размер файла уже меньше 1 МБ, просто возвращаем его без изменений
          resolve(file);
          return;
        }

        const attemptCompression = () => {
          const reader = new FileReader();

          reader.onload = (event) => {
            const image = new Image();

            image.onload = () => {
              const canvas = document.createElement('canvas');
              const ctx = canvas.getContext('2d');

              const maxWidth = 800;
              const maxHeight = 600;

              let width = image.width;
              let height = image.height;

              if (width > maxWidth) {
                height *= maxWidth / width;
                width = maxWidth;
              }

              if (height > maxHeight) {
                width *= maxHeight / height;
                height = maxHeight;
              }

              canvas.width = width;
              canvas.height = height;

              ctx.drawImage(image, 0, 0, width, height);

              canvas.toBlob((blob) => {
                if (blob.size <= targetSizeInBytes || currentQuality >= maxQuality) {
                  // Если размер файла удовлетворяет требованиям или достигнут максимальный уровень качества
                  resolve(new File([blob], file.name, { type: file.type, lastModified: file.lastModified }));
                } else {
                  // Если размер файла все еще слишком большой, уменьшаем уровень качества и повторяем попытку
                  currentQuality += 0.1; // Можете настроить этот шаг
                  attemptCompression();
                }
              }, 'image/jpeg', currentQuality);
            };

            image.src = event.target.result;
          };

          reader.readAsDataURL(file);
        };

        attemptCompression();
      });
    },
    addIngredient() {
      const newIngredient = { categoryM: 0, productId: 0, amount: null, productItems: [] };
      this.item.ingredients.push(newIngredient);
    },
    async handleIngredientChange(ingredient, arr, dispatch) {
      if (ingredient !== 0) {
        try {
          this.item[arr].forEach(item => {
            if (item.categoryM === ingredient) {
              item.loading = true
              setTimeout(() => {
                this.$store.dispatch(dispatch, ingredient)
                  .then(response => {
                    this.$set(item, 'productItems', response.data);
                    item.loading = false
                  })
              }, 1000)

            }
          })
        } catch (error) {
          this.$store.commit('setSnackbars', error.message);
        }
      }
    },
    getSelectedItems(checkbox) {
      if (checkbox.selected) {
        this.item.consumables.push({ materialId: checkbox.id, amount: 1 });
      } else {
        this.item.consumables =  this.item.consumables.filter(
          (item) => item.materialId !== checkbox.id
        );
      }
    },
    initEventSource() {
      this.$store.dispatch('currentAmountByGoods', this.$props.propItem1.stock.id)
        .then(r => {
          this.goods = r.data
        })
    },
  },

}
</script>

<style scoped>

</style>
