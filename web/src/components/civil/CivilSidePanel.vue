<template>
    <div class="m-2">
        <br/>
        <div class="mx-auto bg-white">
            <b-nav vertical>
                <b-nav-item style="font-size: 10px; font-weight: bold;" disabled>ON THIS FILE</b-nav-item>
                    <b-nav-item  
                        v-for="(panelItem, index) in panelItems" 
                        :key="index" 
                        style="line-height: 1.25;"
                        v-on:click="SelectPanelItem(panelItem)" >
                            {{panelItem}}
                    </b-nav-item>    
            </b-nav>
        </div>
    </div>    
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import '@store/modules/CivilFileInformation';
const civilState = namespace('CivilFileInformation');

@Component
export default class CivilSidePanel extends Vue {

    @civilState.State
    public showSections 
    
    /* eslint-disable */
    @civilState.Action
    public UpdateShowSections!: (newShowSections: any) => void
    /* eslint-enable */
    
    panelItems = [ 
       'Case Details', 'Future Appearances', 'Past Appearances'    
    ];

    public SelectPanelItem(panelItem) {

        const sections = this.showSections
    
        for(const item of this.panelItems)
        {
            if(item == panelItem )
                sections[item] = true;
            else
                sections[item] = false;
        }
        
        this.UpdateShowSections(sections);
    }

}
</script>