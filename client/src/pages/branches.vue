<template>
  <section class= "section">
        <div class= "main-header" >{{componentName}}</div>
    <div class = "container flex col center-hor">
        <div class=" card " >
            <div  class= "flex col1 list-item outline1" v-for="(item,idx) in list">
                <div class=" width100-sm flex row col1 width-30-pr ">
                    <div class="flex font-size-2em-sm row space-around ">
                        <div class="item " > {{item.name}} </div>
                    </div>
                </div>
                <div class="width100-sm   flex row1 " >
                    <div class="item width10" >markers color: </div>
                        <div class='color-box' :style="{ 'background-color':item.color}"> 
                    </div>
                <!--</div>-->
                    <b-button size="sm" variant='warning' class="margin-sides1 margin-left-sm " @click="editRecord(item)">
                            <b-tooltip :delay="tooltipDelay" content="edit page">
                                    <i class="material-icons">mode_edit</i>
                            </b-tooltip>
                    </b-button>
                </div>
            </div>
        </div>
    </div>
    </div>
    <div  class="buttons-pnl margin-top-1 flex row center-ver">
                <b-button variant='primary' @click="addNewLocalRecord(listName)">
                    <b-tooltip :delay="tooltipDelay" content="add new record">
                        <i class="material-icons">add_box</i>
                    </b-tooltip>
                </b-button>
            <b-tooltip class="margin-sides1" :delay="tooltipDelay" content="save all">
                <b-button  size="lg" variant='primary' @click="saveAll(list,listName)">save all</b-button>
            </b-tooltip>
            
            <b-button variant='primary'  @click="getList(listName,{},$event)">
                <b-tooltip :delay="tooltipDelay" content="refresh list">
                        <i class="material-icons">refresh</i>
                </b-tooltip>
            </b-button>
    </div>
           <!--====================MODAL========================-->
      <transition name="fade">
            <div  v-on:click="closeModal"  v-if="pageEdit" class="modal" >
                <div v-on:click.stop="pageEdit=true" class="inner-modal flex col center-hor ">
                    <a href="javascript:void(0)" class="closebtn" @click.stop="closeModal">&times;</a>
                    <div v-if="item._id" class= "main-header" >Edit {{item.name}}</div>
                    <div v-else          class= "main-header" >New {{itemName}}</div>
                    <div   class="field-line flex col form">
                        <div class="field  col ">
                                <div class="field header">Name:</div>
                                <b-form-input size="lg" @keydown.native="modalIsDirty" class="input" v-model="item.name"></b-form-input>
                        </div>
                        <div class="field  col ">
                                <div class="field header ">color:</div>
                                <input type="color" @keydown.native="modalIsDirty" class="input" v-model="item.color"></input>
                        </div>
                    <!--===========BUTTONS PANNEL========-->
                    <div  class="buttons-pnl">
                        <b-button :class="{ btnDisabled: !modalDirty }" :disabled= "!modalDirty" variant='primary'  
                            @click.stop="saveRecord(listName,item,$event)">
                            <b-tooltip  class="margin-sides1" :delay="tooltipDelay" content="backup">
                                    <i  class="material-icons">backup</i>
                            </b-tooltip>
                        </b-button>
                        
                        <b-button variant='danger'  
                                    @click.stop="deleteFromList(listName,item._id,$event)">
                                <b-tooltip class="margin-sides1" :delay="tooltipDelay" content="delete">
                                    <i class="material-icons">delete_forever</i>
                                </b-tooltip>
                        </b-button>
                        
                        <b-button variant='warning' @click.stop="closeModal">
                            <b-tooltip class="margin-sides1" :delay="tooltipDelay" content="undo">
                                <i class="material-icons">undo</i>
                            </b-tooltip>
                        </b-button>
                </div>
                </div>
            </div>
                <div class="modal-background "></div>
        </div>
        </transition>

                      <!--====================MODAL2========================-->
      <transition name="fade">
            <div  v-on:click="modal2='false'"  v-if="modal2" class="modal" >
                <div v-on:click.stop="modal2=true" class="inner-modal flex col center-hor ">
                   <!--============HEADER=============-->
                    <a href="javascript:void(0)" class="closebtn" @click.stop="closeModal2">&times;</a>
                    <div class= "main-header" >are you sure you want to delete?</div>
                    <div   class="field-line flex col form">

                    <!--===========BUTTONS PANNEL========-->
                    <div  class="buttons-pnl">
                        <b-button variant='danger'  
                                    @click.stop="deleteFromList2(listName,item,$event)">
                                <b-tooltip class="margin-sides1" :delay="tooltipDelay" content="delete">
                                    <i class="material-icons">delete_forever</i>
                                </b-tooltip>
                        </b-button>
                        <b-button variant='warning' @click.stop="closeModal2">
                            <b-tooltip class="margin-sides1" :delay="tooltipDelay" content="undo">
                                <i class="material-icons">undo</i>
                            </b-tooltip>
                        </b-button>
                    </div>
                </div>
            </div>
            <div class="modal-background "></div>
        </div>
        </transition>

  </section>
</template>
<script>
import { SENDMSG } from '../store/store'
import Helpers from '../services/helpers.service.js';
import { sortlistBy2Keys, cloneDeep,getObjById} from '../helpers/functions'
import moment from 'moment'
export default {
  name: 'Branches',
  data() {
    return {
      Helpers:Helpers,
      componentName:'Branches',
      state:this.$store.getters.fetchGetState,
      facilities:this.$store.getters.fetchGetFacilities,//
      pageMode:'readOnly',
      editMode:false,
      pageEdit:false,
      tooltipDelay:700,
      fetchList:this.$store.getters.fetchGetBranches,//need to be changed!!
      list:[],
      listName:'branches',//need to be changed!!
      item: '',
      itemName: 'branch',
      modalDirty:false,
      modal2:false
    }
  },
  mounted () {
        this.getList(this.listName,{});
        var list = this.$store.getters.fetchGetBranches;
            list = Helpers.sortlistBy2Keys(list,'fac','idx');
            this.list = list  
        },
    watch:{
      list1: function(newList){
            var list = this.$store.getters.fetchGetBranches;
            list = Helpers.sortlistBy2Keys(list,'fac','idx');
            this.list = list
      },
    },
    computed: {
        list1() {
            var list = this.$store.getters.fetchGetBranches;
            list = Helpers.sortlistBy2Keys(list,'fac','idx');
            return list;
        },
    },
  methods: {//
      isDeletedItem(item){
           if(item.mode=== 'deleted'){
                return 'deleted-item'
           }else{
                return '';
           }
        //    return (item.mode=== 'deleted')? 'deleted-item': '';
      },
    modalIsDirty(){
        this.modalDirty = true;
    },
  closeModal(){
            this.pageEdit =false;
            this.modalDirty = false;
  },
      editRecord(item){
            this.pageEdit =!this.pageEdit;
            this.item = Helpers.cloneDeep(item);
      },
        toggleField(e,bul){
            var parent = e.path[5];
            var inputNodes = parent.getElementsByTagName('input');
            this.toggleDisable(inputNodes,bul)
        },
      saveAll(list,listName){
        for (var i = 0; i < list.length; i++) {
            list[i].mode = 'saved';
            const acts =[
                    { actType: 'updateInList', newObj: list[i], list:listName,askFrom:'server' },
                        ]
            this.sendMsg( {acts});
        }                   
      },
      saveRecord(list,newObj,e){
          
            newObj.mode = 'saved';
            const acts =[]
            if(newObj._id){
                acts.push({ actType: 'updateInList', newObj, list,askFrom:'server' })
            }else{
                    acts.push({ actType: 'addToList', newObj, list,askFrom:'server' })
            }
                acts.push({ actType: 'getList', list, criteria:{},askFrom:'server' })
                this.sendMsg( {acts});
                this.closeModal()
      },
      addNewLocalRecord(arr){
        this.item = {mode :'edit'};
        this.pageEdit =true;
      },
      getList(listName,criteria){
            console.log('graders.getList', listName)
            const acts =[{ actType: 'getList', list:listName, criteria:{} }];
            this.sendMsg({acts});
      },
      deleteFromList2(list,obj,e){
            const acts =[
                            { actType: 'deleteFromList', objId:obj._id, list},
                            { actType: 'getList', list:list, criteria:{} }
                        ]
            this.sendMsg({acts});
            obj.mode = 'deleted'
            this.closeModal();
            this.modal2 =false;
      },
      closeModal2(){
        this.modal2 =false;
      },
      deleteFromList(list,id,e){
        this.modal2 =true;
      },
      toggleDisable(inputNodes,bul){
            for (var i = 0; i < inputNodes.length; i++) {
                inputNodes[i].disabled = bul;
            }
      },
    sendMsg(msg){
          this.$store.dispatch({ type: SENDMSG, msg })
    },
    sendLocalMsg(msg){//msgs to the store
          this.$store.dispatch({ type: SENDLOCALMSG, msg })
    },
  },
}
</script>
<style scoped>
.row{
    margin-right:0;
    margin-left:0;
}
width-lg{
    width:15em;
}
.list-item{
    padding:0.5em;
    padding-left:1em;
}
select{
    width:100%;
}
.color-box{
    width:3em;
    min-height:3em;
    height:100%;
}
.deleted-item{
    background-color: red;
}
*{
    /*outline:1px solid black;*/
}
.margin-left-sm{
    margin-left:2em;
}
.item{
    font-size: 1.5em;
}
.width-30-pr{
   width:30%; 
}
.width-50-pr{
   width:50%; 
}
.card{
    width:90%;
}
@media screen and (max-width: 700px) {
    .width100-sm{
         width:100%;
     }
     .font-size-2em-sm{
         font-size:1.5em;
     }
}
</style>