<template>
<body>
<criminal-documents-view v-if="showDocuments"></criminal-documents-view>


</body>
    
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import CriminalDocumentsView from '@components/criminal/CriminalDocumentsView.vue';
import '@store/modules/CriminalFileInformation';
const criminalState = namespace('CriminalFileInformation');

@Component({
    components: {
        CriminalDocumentsView
    }
})
export default class CriminalCaseDetails extends Vue {

    @criminalState.State
    public showCaseDetails

    @criminalState.State
    public showFutureAppearances

    @criminalState.State
    public showPastAppearances

    @criminalState.State
    public showWitnesses
    
    @criminalState.State
    public showDocuments

    @criminalState.State
    public showSentenceOrderDetails  
    
}
</script>