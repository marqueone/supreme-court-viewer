<template>
    <b-card
        :id="id"
        class = "board"
        style="width-min:40px"        
        @dragover.prevent
        @drop.prevent="drop"
    >
        <slot />        
    </b-card>
</template>s


<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';

@Component
export default class Board extends Vue {

    @Prop()'id'  

    public drop(e) {
        const cardid = e.dataTransfer.getData('card_id');
        const card = document.getElementById(cardid);
        if(card) card.style.display = "block";
        e.target.appendChild(card);
        console.log(this.id)
        console.log(cardid)
        if(this.id=="del")
            e.target.removeChild(card);

        
    }

}

</script>
<style scoped>
 .card {
        border: black;
    }

</style>