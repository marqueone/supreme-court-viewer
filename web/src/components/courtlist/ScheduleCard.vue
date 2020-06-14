<template>
        <b-button
            :id="id"
            class="text-danger bg-info m-1"
            
            style="font-size:16px;height:40px;width:80px"
            :draggable="draggable"
            @dragstart="dragStart"
            @dragover.stop
            @click = "openModal(id)"
        >
            <b> {{id}} </b>

            <b-modal v-model="showOrderMade"  hide-footer>
                <template v-slot:modal-title>
                    <h2 class="mb-0"> Schedule {{id}} </h2>
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
                <b-button class="m-3" @click="showOrderMade=false">Save</b-button>
            </b-modal>
        </b-button>

</template>



<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';

@Component
export default class ScheduleCard extends Vue {

    @Prop()'id'
    @Prop()'draggable'   

    selectedDate = (new Date).toISOString().substring(0,10);
    selectedCourtRoom =''

    public dragStart(e) {
        const target = e.target;
        e.dataTransfer.setData('card_id', target.id);

        // setTimeout(() => {
        //     target.style.display = "none";
        // },  0);
    }

    public modalID(id)
    {
        return 'bv-modal-'+id
    }

    showOrderMade=false
    public openModal(id)
    {
        this.showOrderMade =true
    }

}

</script>