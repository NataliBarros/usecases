<template>
   <div class="single-cell-insilico-experiments">
    <div class="title">Please select a model</div>
    <div class="content">
      <md-whiteframe md-elevation="1">
        <div class="search-container">
          <i class="material-icons">search</i>
          <input class="searchbox" type="text" v-model="filter" placeholder="Search e.g 'Hippocampus' or click on the item's title to filter">
        </div>
      </md-whiteframe>
      <models-list :models="models" v-on:selected="touched" v-on:tagfilter="addSearch"></models-list>
    </div>
   </div>
</template>

<script>
   import BspNmcModelList from 'components/singlecellinsilicoexperiments/models-list.vue'
   const VIEWER_URL = 'https://bbp.epfl.ch/public/morph-view/'
   import collabAuthentication from 'mixins/collabAuthentication.js'
   import createCollab from 'mixins/createCollab.js'
   import ModelsConfig from 'assets/config_files/models.json'
   import ModelsBSP from 'assets/config_files/singlecellmodeling_structure.json'
   import ModelsNMC from 'assets/config_files/nmcportalmodels_structure.json'
   import modelsMixins from 'mixins/models.js'
   
   export default {
      name: 'modelContainer',
      components: {
         'models-list': BspNmcModelList
      },
      data () {
        return {
          models: [],
          originalModels: [],
          filter: ''
        }
      },
      mixins: [collabAuthentication, createCollab, modelsMixins],
      props: ['list_usecases', 'uc_name'],
      methods: {
        touched (modelItem) {
          let viewUrl = VIEWER_URL + modelItem.folderName + '.html'
          window.open(viewUrl, '_blank');
          var category = this.$route.path.split('/')[1]
          this.sendStatistics(null, this.uc_name, category, modelItem.folderName, null)
        },
        getNMCMetadata (modelsJson, path) {
          for (let i = 0; i < modelsJson.length; i++) {
            let elem = modelsJson[i]
            let fileName = Object.keys(elem)[0]
            let modelInfo = elem[fileName].meta
            let morphPath = path + elem[fileName].morph
            modelInfo.morphImg = morphPath
            let responsePath = path + elem[fileName].responses
            modelInfo.reponsesImg = responsePath
            modelInfo.folderName = fileName
            modelInfo.modelTitle = this.getModelTitle(modelInfo)
            this.models.push(modelInfo)
            this.originalModels = this.models // save all the models
          }
        },
        addSearch (obj) {
          this.filter += ' ' + obj.text
        },
        performNMC () {
          let modelConfigNMC = ModelsConfig['nmcportal']
          this.getNMCMetadata(ModelsNMC, modelConfigNMC.raw)
          this.searchedModels = this.models
        }
      },
      created () {
        let that = this
        let modelConfigBSP = ModelsConfig['ca1models']
        document.querySelector('title').innerHTML = 'Models'
        this.models = this.getBSPMetadata(ModelsBSP, modelConfigBSP.raw)
        if (window.requestIdleCallback) {
          window.requestIdleCallback(that.performNMC)
        } else {
          that.performNMC()
        }
      },
      watch: {
        'filter': async function (newVal) {
          this.models = await this.search(newVal, this.originalModels)
        }
      }
   }
</script>

<style scoped>
.search-container {
  display: flex;
  align-items: center;
  margin: 20px 0;
  padding: 10px;
  background-color: #cfe2e8;
}
.searchbox {
  font-size: 22px;
  padding-left: 30px;
  border: 1px solid #ddd;
  margin-bottom: 0;
  margin-left: 12px;
  width: 100%;
}
.title {
  box-shadow: 0 2px 5px rgba(0,0,0,.26);
  position: fixed;
  text-align: left;
  color: #fff;
  background-color: rgba(172, 96, 103, 0.95);
  padding: 20px;
  font-size: 20px;
  font-weight: 600;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 2;
}
.content {
  padding: 10px;
  margin-top: 50px;
}
</style>
