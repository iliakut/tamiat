<template>
  <div class="contentType">
    <div class="container is-widescreen">

      <!-- View for edit/add new field -->
      <router-view :edit-field="editField" :add-field="addField"></router-view>

      <!-- notification -->
      <transition mode="out-in" name="fade">
        <div v-if="notification.message" :class="'notification is-' + notification.type">
          <button class="delete" @click="hideNotifications"></button>{{notification.message}}
        </div>
      </transition>
      <div class="columns">
        <div class="column is-two-thirds">
          <h2>Create content type</h2>
          <div class="box">
            <div class="columns">
              <div class="column">
                <div class="field">
                  <label class="label">Content type name</label>
                  <div class="control">
                    <input v-if="!selectedContent" class="input" type="text" placeholder="e.g. Movies" v-model="name">
                    <input v-else class="input" type="text" placeholder="e.g. Movies" v-model="selectedContent.name">
                  </div>
                </div>
                <!-- area to add new field (variables) to the Content -->
                <div class="field">
                  <button type="submit" class="button is-info is-small"  @click="showModal = true">Add new field</button>
                  <!-- Modal -->
                  <modal class="modal" @close="showModal = false"  @addContentField='addNewContentField($event)' v-if="showModal" :kind="'addContentField'" :header="'Add content field'">
                    <!-- Modal Slot - made for adding content type fields -->
                    <option v-for="field in fieldTypes" :key="field.id">{{ field.label }}</option>
                  </modal>
                </div>
                <!-- fields area -->
                <span class="label">Available content fields</span>
                <span class="label has-text-danger is-size-7" v-if="Object.keys(availableContentFields).length">Select the content fields you want to include in your content type</span>
                <div class="field">
                  <div class="field" v-for="(fieldsType, typeIndex) in Object.keys(availableContentFields)" :key="fieldsType + typeIndex">
                    <span>
                      {{ fieldsType }}
                    </span>
                    <ul class="nav-preview">
                      <li v-for="(field, fieldIndex) in availableContentFields[fieldsType]" :key="field + fieldIndex">
                        <span>
                          {{ field }}
                          <span class="link-actions">
                            <span class="has-text-danger fa fa-trash" @click="deleteAvailableContentField(fieldsType, fieldIndex)"></span>
                            <span class="has-text-info fa fa-edit" @click="callEditModal(fieldsType, field)"></span>
                            <span class="has-text-success fa fa-plus-square" @click="addContentFieldToContent(fieldsType, field)"></span>
                          </span>
                        </span>
                      </li>
                    </ul>
                  </div>
                  <!-- editing -->
                  <!-- Modal -->
                  <modal class="modal"
                         @close="showEditModal = false"
                         @addContentField='editContentField($event)'
                         v-if="showEditModal"
                         :kind="'addContentField'"
                         :header="'Edit content field'"
                         :contentFieldName="editingField.name"
                         :contentFieldType="editingField.type">
                    <!-- Modal Slot - made for adding content type fields -->
                    <option v-for="field in fieldTypes" :key="field.id">{{ field.label }}</option>
                  </modal>
                </div>

                <!-- Content fields of current Content -->
                <span class="label">Content fields</span>
                <div class="field">
                  <ul class="nav-preview">
                      <li v-for="(contentField, index) in contentFields" :key="contentField + index">
                        <span>
                          {{ contentField.name }}
                          <span class="link-actions">
                            <span class="has-text-danger fa fa-trash" @click="deleteContentFieldFromContent(index)"></span>
                          </span>
                        </span>
                      </li>
                    </ul>
                </div>

                <!-- Custom Fields -->
                <label class="label">Fields</label>
                <label class="has-text-danger is-size-7" v-if="fields.length">Select the fields you want to be shown in content lists</label>
                <div class="field is-grouped is-grouped-multiline">
                  <ul class="nav-preview">
                    <li v-for="(field, fieldKey) in checkedFields" :key="fieldKey" v-if="field.checked">
                      {{ field.name }}
                      <span>
                        <span v-if="fieldKey !== 0" class="has-text-success fa fa-arrow-up" @click="moveFieldUp(field, checkedFields[fieldKey - 1])"></span>
                        <span v-if="fieldKey !== checkedFields.length - 1" class="fa fa-arrow-down" @click="moveFieldDown(field, checkedFields[fieldKey + 1])"></span>
                        <!-- <span @mouseover="showDesc = !showDesc">
                          <checkbox v-if="field.type === 'textbox'" v-model="field.sortable" /> </span>
                        <span v-if="showDesc && field.type === 'textbox'" class="has-text-danger is-size-7">Check if you want this field to be shown in the table</span> -->
                        <span><checkbox v-if="field.type === 'textbox'" v-model="field.sortable" /></span>
                      </span>
                    </li>
                  </ul>
                </div>
                <br /><br />

                <div>
                  <label class="label">Slug - <strong v-text="slug"></strong></label>

                  <div class="select">
                    <select v-model="slug" placeholder="Select Column For Slug">
                      <option value="" selected>Select Column For Slug</option>
                      <option v-for="(field, fieldKey) in checkedFields" :key="fieldKey" v-if="field.checked && field.type === 'textbox'">
                        {{ field.name }}
                      </option>
                    </select>
                  </div>

                  <br /><br />
                  <p>This will be used with :key to identify record.</p>
                </div>
              </div>
              <div class="column">
                <label class="label">Available fields</label>
                <label class="label has-text-danger is-size-7" v-if="fields.length">Select the fields you want to include in your content type</label>
                <div class="field is-grouped is-grouped-multiline">
                  <ul class="nav-preview">
                    <li class="control" v-for="(field, fieldKey) in fields" :key="fieldKey">
                      <span class="field">
                        <checkbox :label="field.name" v-model="field.checked" />
                        <span class="link-actions">
                          <span class="has-text-danger fa fa-trash" @click="removeField(field)"></span>
                          <router-link :to="'/admin/content/fieldEdit/' + field['.key']"><span class="has-text-info fa fa-edit"></span></router-link>
                        </span>
                      </span>
                    </li>
                  </ul>
                </div>
                <br>
                <router-link to="/admin/content/fieldNew" class="button is-info is-small">Add new field</router-link>
              </div>
            </div>
            <div class="buttons">
              <button v-if="selectedContent" type="submit" class="button is-success" :disabled="!selectedContent.name || !checkedFields.length" @click="createMenuItem(true)">Edit</button>
              <button v-else type="submit" class="button is-success" :disabled="!name || !checkedFields.length" @click="createMenuItem(false)">Create new</button>
            </div>
          </div>
        </div>
        <div class="column">
          <h2>Edit content type</h2>
          <div class="box" v-if="contentsLoaded">
            <div class="dropdown" :class="{'is-active': dropdownActive}" @click="toggleDropdown">
              <div class="dropdown-trigger">
                <button class="button" aria-haspopup="true" aria-controls="dropdown-menu" style="min-width: 200px !important;">
                  <span>{{selectedContentType.label}}</span>
                  <span class="icon is-small">
                    <i class="fa fa-angle-down" aria-hidden="true"></i>
                  </span>
                </button>
              </div>
              <div class="dropdown-menu" id="dropdown-menu" role="menu">
                <div class="dropdown-content">
                  <a class="dropdown-item" v-if="option" v-for="(option, optionKey) in createdContentTypes" :key="optionKey" @click="selectContentType(option)">
                    {{option.label ? option.label : null }}
                  </a>
                </div>
              </div>
            </div>
            <button v-if="selectedContent" type="submit" class="button is-danger" @click="deleteMenuItem()">Delete</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import checkbox from '@/admin/components/shared/Checkbox'
import dropdown from '@/admin/components/shared/Dropdown'
import { contentsRef, fieldsRef, contentFieldsRef } from '@/admin/firebase_config'
import notifier from '@/admin/mixins/notifier'
import modal from '@/admin/components/shared/Modal'

export default {
  name: 'content-type',
  mixins: [notifier],
  firebase: {
    contents: {
      source: contentsRef,
      readyCallback: function () {
        // Load content types in select after content data loaded from firebase
        this.loadContentTypes()
      }
    },
    fields: fieldsRef,
    contentFieldsTemplate: contentFieldsRef
  },
  data () {
    return {
      test: false,
      name: '',
      fieldTypes: [
        {
          id: 'textarea',
          label: 'Textarea'
        },
        {
          id: 'textbox',
          label: 'Textbox'
        },
        {
          id: 'integer',
          label: 'Integer'
        },
        {
          id: 'boolean',
          label: 'Boolean'
        },
        {
          id: 'url',
          label: 'Url'
        },
        {
          id: 'richtextbox',
          label: 'Richtextbox'
        },
        {
          id: 'tags',
          label: 'Tags'
        },
        {
          id: 'select',
          label: 'Select'
        },
        {
          id: 'image',
          label: 'Image'
        }
      ],
      availableContentFields: {
      /* this obj will contain all new fields for Content
        * every array contains its area objects (with names and etc.) */
      },
      contentFields: [],
      slug: '',
      showDesc: false,
      createdContentTypes: null,
      selectedContentType: {
        id: '',
        label: 'Select content type'
      },
      dropdownActive: false,
      selectedContent: null,
      contentsLoaded: false,
      selectedContntFields: null,
      showModal: false,
      showEditModal: false,
      editingField: {
        type: '',
        name: '',
        index: null
      }
    }
  },
  created () {
    this.clearChecked()
    this.loadContentTypes()
  },
  computed: {
    checkedFields () {
      return this.fields.filter(field => {
        return field.checked
      })
    }
  },
  methods: {
    testFirebase (item) {
      this.$firebaseRefs.contentFields.push(item)
      this.$firebaseRefs.contents.child(this.selectedContent['.key']).set(item)
    },
    addNewContentField (contentFieldArrParams) {
      // contentFieldArrParams is arr that contains two elements 0 - name of Field 1 - type of Field
      const fieldName = contentFieldArrParams[0]
      const fieldType = contentFieldArrParams[1]
      if (fieldName === '' || fieldType === '' || fieldName === undefined || fieldType === undefined) return
      // create arr for Field if it was not created
      if (!this.availableContentFields[fieldType]) this.availableContentFields[fieldType] = []
      // add new field to field Type array
      this.availableContentFields[fieldType].push(fieldName)
      // close modal
      this.showModal = false
    },
    addContentFieldToContent (type, name) {
      this.contentFields.push({ type: type, name: name })
    },
    deleteContentFieldFromContent (index) {
      this.contentFields.splice(index, 1)
    },
    editContentField (contentFieldArrParams) {
      // contentFieldArrParams is arr that contains two elements 0 - name of Field 1 - type of Field
      const fieldName = contentFieldArrParams[0]
      const fieldType = contentFieldArrParams[1]
      if (fieldName === '' || fieldType === '' || fieldName === undefined || fieldType === undefined) return
      // create arr for Field if it was not created
      if (!this.availableContentFields[fieldType]) this.availableContentFields[fieldType] = []
      // add new field to field Type array
      this.availableContentFields[fieldType].push(fieldName)
      // delete previous Field
      this.deleteAvailableContentField(this.editingField.type, this.editingField.index)
      // close modal
      this.showEditModal = false
    },
    deleteAvailableContentField (fieldType, index) {
      let currentContentFields = this.availableContentFields
      currentContentFields[fieldType].splice(index, 1)
      // if Field array is empty - delete it
      if (currentContentFields[fieldType].length === 0) delete currentContentFields[fieldType]
      // this made to help Vue see changes inside of Object
      this.availableContentFields = Object.assign({}, currentContentFields)
    },
    resetContentFields () {
      this.editingField = {
        type: '',
        name: '',
        index: null
      }
      this.contentFields = []
    },
    callEditModal (fieldType, name) {
      this.editingField.name = name
      this.editingField.type = fieldType
      this.editingField.index = this.availableContentFields[fieldType].indexOf(name)
      this.showEditModal = true
    },
    loadContentTypes () {
      this.contentsLoaded = false
      this.selectedContentType = {
        id: '',
        label: 'Select content type'
      }
      this.createdContentTypes = this.contents.map(content => {
        return {
          id: content['.key'],
          label: content.name
        }
      })
      this.createdContentTypes.push({ id: '', label: 'Select content type' })
      this.contentsLoaded = true
    },
    createMenuItem (edit) {
      let path
      if (edit) {
        path = this.selectedContent.name.toLowerCase()
      } else {
        path = this.name.toLowerCase()
      }
      path = path.replace(/^[, ]+|[, ]+$|[, ]+/g, '').trim()

      let selectedFields = this.checkedFields.map(field => {
        return Object.assign({
          name: field.name,
          type: field.type,
          multiValue: field.multiValue
        }, field.sortable ? { sortable: true } : null)
      })

      let item = {
        name: this.name,
        // load contentFields to item to send it to Firebase
        contentFields: this.contentFields,
        slug: this.slug,
        path: `/admin/content/${path}`,
        icon: 'fa-file-text',
        fields: selectedFields
      }
      if (edit) {
        this.selectedContent.path = `/admin/content/${path}`
        this.selectedContent.fields = selectedFields
        this.selectedContent.contentFields = this.contentFields
        this.selectedContent.slug = this.slug
        let item = { ...this.selectedContent }
        delete item['.key']
        this.$firebaseRefs.contents.child(this.selectedContent['.key']).set(item).then(() => {
          this.resetForm()
          this.loadContentTypes()
          this.showNotification('success', 'Content edited successfully')
        })
      } else {
        this.$firebaseRefs.contents.push(item).then(() => {
          this.loadContentTypes()
          this.showNotification('success', 'Content type added successfully')
          this.resetForm()
        })
      }
    },
    deleteMenuItem () {
      this.$firebaseRefs.contents.child(this.selectedContent['.key']).remove()
        .then(() => {
          this.loadContentTypes()
          this.showNotification('success', 'Content type removed successfully')
        })
    },
    resetForm () {
      this.name = ''
      this.resetContentFields()
      this.slug = ''
      this.selectedContent = null
      for (var fieldKey in this.fields) {
        this.fields[fieldKey].checked = false
      }
      for (var key in this.supports) {
        this.supports[key] = false
      }
    },
    addField (field) {
      this.$firebaseRefs.fields.push(field).then(() => {
        this.showNotification('success', 'Field added successfully')
      })
    },
    editField (field, f) {
      this.$firebaseRefs.fields.child(field['.key']).set(f).then(() => {
        this.showNotification('success', 'Field edited successfully')
      })
    },
    removeField (field) {
      this.$firebaseRefs.fields.child(field['.key']).remove()
        .then(() => {
          this.showNotification('success', 'Field removed successfully')
        })
    },
    moveFieldUp (field, previousField) {
      let itemCopy = Object.assign({}, field)
      let previousItemCopy = Object.assign({}, previousField)

      delete itemCopy['.key']
      delete previousItemCopy['.key']

      this.$firebaseRefs.fields.child(field['.key']).set(previousItemCopy)
      this.$firebaseRefs.fields.child(previousField['.key']).set(itemCopy)
    },
    moveFieldDown (field, nextField) {
      let itemCopy = Object.assign({}, field)
      let nextItemCopy = Object.assign({}, nextField)

      delete itemCopy['.key']
      delete nextItemCopy['.key']

      this.$firebaseRefs.fields.child(field['.key']).set(nextItemCopy)
      this.$firebaseRefs.fields.child(nextField['.key']).set(itemCopy)
    },
    clearChecked () {
      for (var key in this.fields) {
        delete this.fields[key]['checked']
      }
    },
    selectContentType (option) {
      if (option.id === '') return

      this.selectedContentType = option
      this.selectedContent = this.contents.filter(content => {
        if (content.name === option.label) {
          return content
        }
      })[0]
      this.slug = this.selectedContent.slug
      // load selected fields from selectedField to current content Field
      if (this.selectedContent.contentFields !== undefined) this.contentFields = this.selectedContent.contentFields
      else (this.contentFields = [])
      this.clearChecked()
      if (option.id) {
        this.mapFields()
      }
    },
    toggleDropdown () {
      this.dropdownActive = !this.dropdownActive
    },
    mapFields () {
      for (var j = 0; j < this.fields.length; j++) {
        for (var i = 0; i < this.selectedContent.fields.length; i++) {
          if (this.selectedContent.fields[i].name === this.fields[j].name) {
            this.selectedContent.fields[i].checked = true
            this.$set(this.fields[j], 'checked', true)
          }
        }
      }
    },
    setAsSlug (column) {
      this.slug = this.slug === column ? null : column
    }
  },
  components: {
    checkbox,
    dropdown,
    modal
  }
}

</script>

<style lang="scss" scoped>
.contentType {
  .nav-preview {
    padding-left: 15px;
  }

  .link-actions {
    display: none;
    span {
      cursor: pointer;
    }
  }
  .cursor-pointer {
    cursor: pointer;
  }

  .nav-preview > li:hover .link-actions {
    display: inline;
  }

  .modal {
    z-index: 1024;
  }
}
</style>
