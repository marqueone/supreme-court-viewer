<template>
<body>
<criminal-documents-view></criminal-documents-view>


</body>
    
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import CriminalDocumentsView from './CriminalDocumentsView.vue';

@Component({
    components: {
        CriminalDocumentsView
    }
})
export default class CriminalCaseDetails extends Vue {
    
}
</script>