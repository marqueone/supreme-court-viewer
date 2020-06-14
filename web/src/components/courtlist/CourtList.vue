<template>
<body>
    <b-card class="mt-5 mx-3">
        <b-table
            :items="scheduleList"
            :fields="fields"            
            bordered
            head-variant="dark"
            small
            responsive="sm"
            >  

            <template v-slot:cell()="data">                    
                    <board :id="data.field.key">                        
                    </board>
            </template>

             <template v-slot:cell(Date)="data">
                    <b> {{data.value}} </b>
            </template>

        </b-table>
    </b-card>

    <b-card class="mt-5 mx-3">
        <b-row cols="2">
            <b-col md="8">
                <b-card border-variant="dark">
                    <h3> Unassinged Schedules</h3>
                    <b-button @click="addButton"> Add </b-button>
                    <board id="unassinged-cards"></board>
                </b-card>
            </b-col>
            <b-col md="4">
                <b-card border-variant="danger">
                    <h3> Delete <b-icon icon="trash"></b-icon></h3>
                    
                    <board id="del"></board>
                </b-card>
            </b-col>
        </b-row>
    </b-card>
   

            <b-modal v-model="showOrderMade"  hide-footer>
                <template v-slot:modal-title>
                    <h2 class="mb-0"> Edit Schedule-{{cardIds}} </h2>
                </template>

                <label >Date* (YYYY-MM-DD)</label>   
                <b-input-group class="mb-3">                
                    <b-form-input
                        
                        v-model="selectedDate"
                        type="text"
                        no-caret
                        placeholder="YYYY-MM-DD"
                        autocomplete="off"
                    ></b-form-input>
                    <b-input-group-append>
                        <b-form-datepicker
                        v-model="selectedDate"
                        button-only
                        right
                        locale="en-US"                   
                        ></b-form-datepicker>
                    </b-input-group-append>
                </b-input-group>           
            
                <b-form-group            
                    class = "mr-3"> 
                    <label for="roomSelect">Room*</label>
                    <b-form-select
                        v-model="selectedCourtRoom"
                        id="roomSelect"
                        :options="['101', 'JCM', '80', '005']"
                        style="height:39px">
                    </b-form-select>
                </b-form-group>
            
                <b-button class="m-3" @click="showOrderMade=false">Cancel</b-button>
                <b-button class="m-3" @click="addClick();showOrderMade=false">Save</b-button>
            </b-modal>
    

</body>
</template>



<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import Board from "@components/courtlist/Board.vue";
import ScheduleCard from "@components/courtlist/ScheduleCard.vue";

@Component({
    components: {
        Board,
        ScheduleCard
    }
})
export default class CourtList extends Vue {

    cardIds=1
 
    fields =  
    [
        {key:'Date', tdClass: '', headerStyle:'', cellStyle:'', variant: 'info', thStyle:'width:100px'},
        {key:'Bill Brown',  tdClass: '', headerStyle:'', cellStyle:''},
        {key:'Sam Thompson',  tdClass: '', headerStyle:'', cellStyle:''},
        {key:'Tom Jones',  tdClass: '', headerStyle:'', cellStyle:''},
        {key:'Cathy Butler',  tdClass: '', headerStyle:'', cellStyle:''},
        {key:'Jason Terry',  tdClass: '', headerStyle:'', cellStyle:''},

    ]

    // mounted () {                 
    //     this.getCourtListDetails();
    // }
    scheduleList = [
        {Date:'10/02/2018'},
        {Date:'10/05/2019'},
        {Date:'11/23/2019'}
    ]

    public addClick()
    {
        const unBoard = document.getElementById("unassinged-cards")
        
        const cardId = 'card-'+this.cardIds
        this.cardIds +=1;
        const ComponentClass = Vue.extend(ScheduleCard)
         const card = new ComponentClass({
            propsData: { id: cardId, draggable: 'true'}, 
            template: '<div>A custom component!</div>'           
        })
        console.log(card)
        card.$mount()
         if(unBoard) unBoard.appendChild(card.$el)

    }

    showOrderMade=false
    selectedDate = (new Date).toISOString().substring(0,10);
    selectedCourtRoom =''
    public addButton()
    {
        this.showOrderMade=true
    }

   

}

</script>