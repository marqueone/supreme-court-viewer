<template>
<div>
   <b-card  v-if= "isMounted" no-body>
        <div>         
            <h3 class="mx-4 font-weight-normal"> Documents ({{NumberOfDocuments}}) </h3>
            <hr class="mx-3 bg-light" style="height: 5px;"/>                   
        </div>
       
        <b-tabs nav-wrapper-class = "bg-light text-dark"
                active-nav-item-class="text-uppercase font-weight-bold text-white bg-primary"                     
                pills
                no-body
                class="mx-3"
                >
            <b-tab 
            v-for="(tabMapping, index) in categories" 
            :key="index"                 
            :title="tabMapping"                 
            v-on:click="activetab = tabMapping" 
            v-bind:class="[ activetab === tabMapping ? 'active mb-3' : 'mb-3' ]"
            ></b-tab>
        </b-tabs> 
        
        <b-overlay :show="loadingPdf" rounded="sm">  
            <b-card bg-variant="light" style="max-height: 500px; overflow-y: auto;" no-body class="mx-3 mb-5">           
                <b-table
                :items="FilteredDocuments"
                :fields="fields[fieldsTab]"
                :sort-by="sortBy"
                :sort-desc.sync="sortDesc"
                :no-sort-reset="true"
                sort-icon-left
                small
                striped
                responsive="sm"
                >   
                    <template v-for="(field,index) in fields[fieldsTab]" v-slot:[`head(${field.key})`]="data">
                        <b v-bind:key="index" :class="field.headerStyle" > {{ data.label }}</b>
                    </template>

                    <template v-slot:[`cell(${fields[fieldsTab][datePlace[fieldsTab]].key})`]="data" >
                        <span v-if="data.item.Sealed" :style="data.field.cellStyle" class="text-muted">
                            {{ data.value | beautify-date}}
                        </span>
                        <span v-else :style="data.field.cellStyle">
                            {{ data.value | beautify-date}}
                        </span>
                    </template> 

                    <template v-slot:[`cell(${fields[fieldsTab][documentPlace[fieldsTab]].key})`]="data" >
                        <b-button 
                            v-if="data.item.PdfAvail && !data.item.Sealed" 
                            variant="outline-primary text-info" 
                            :style="data.field.cellStyle"
                            @click="cellClick(data)"
                            size="sm">
                                {{data.value}}
                        </b-button>
                        <span class="ml-2" v-else-if="!data.item.PdfAvail && !data.item.Sealed">
                             {{data.value}}
                        </span>
                        <span class="ml-2 text-muted" v-else-if="data.item.Sealed">
                             {{data.value}}
                        </span>
                    </template>

                    <template v-slot:cell(Act)="data" >
                        <b-badge 
                            variant="secondary"
                            :style="data.field.cellStyle"                     
                            v-for="(act, actIndex) in data.value"  
                            v-bind:key="actIndex"                               
                            v-b-tooltip.hover.left 
                            :title="act.Description"> 
                                {{act.Code}}<br> 
                        </b-badge>
                    </template>

                    <template v-slot:cell(Issues)="data" >                               
                        <li 
                            v-for="(issue, issueIndex) in data.value"  
                            v-bind:key="issueIndex"
                            :style="data.field.cellStyle">
                            <span v-if="data.item.Sealed" class="text-muted">{{ issue }}</span>
                            <span v-else>{{ issue }}</span>
                        </li>
                    </template>
                    
                    <template v-slot:cell(Seq.)="data">
                        <span v-if="data.item.Sealed" class="ml-2 text-muted" :style="data.field.cellStyle"> 
                            {{data.value}}
                        </span>
                        <span v-else class="ml-2" :style="data.field.cellStyle"> 
                            {{data.value}}
                        </span>
                    </template>

                </b-table>
            </b-card>
            <template v-slot:overlay>               
               <div style="text-align: center"> 
                    <loading-spinner/> 
                    <p id="Downloading-label">Downloading PDF file ...</p>
               </div>                
            </template> 
        </b-overlay>
   </b-card> 
</div>
</template>

<script lang="ts">
import { Component, Vue, Watch} from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import '@store/modules/CivilFileInformation';
const civilState = namespace('CivilFileInformation');

enum fieldTab {Categories=0, Summary}

@Component
export default class CivilDocumentsView extends Vue {

    /* eslint-disable */
    @civilState.State
    public civilFileInformation!: any

    @civilState.Action
    public UpdateCivilFile!: (newCivilFileInformation: any) => void

    documents: any[] = [];
    summaryDocuments: any[] = [];
    /* eslint-enable */

    documentsDetailsJson;
    loadingPdf = false;
    isMounted = false;
    activetab = 'ALL';
    sortDesc = false;
    categories: string[] = []; 
    fieldsTab = fieldTab.Categories;
    documentPlace = [1,0]
    datePlace = [3,1]

    fields = [ 
        [
            {key:'Seq.',           sortable:true,  headerStyle:'text-primary',  cellStyle:'font-size: 16px;'},
            {key:'Document Type',  sortable:true,  headerStyle:'text-primary',  cellStyle:'border:0px; font-size: 16px;'},
            {key:'Act',            sortable:false, headerStyle:'text',          cellStyle:'display: block; margin-top: 1px; font-size: 14px; max-width : 50px;'},
            {key:'Date Filed',     sortable:true,  headerStyle:'text-danger',   cellStyle:'font-size: 16px;'},
            {key:'Issues',         sortable:false, headerStyle:'text',          cellStyle:'white-space: pre-line; font-size: 16px; margin-left: 20px;'}
        ],
        [
            {key:'Document Type',    sortable:false, headerStyle:'text-primary',    cellStyle:'border:0px; font-size: 16px;'},
            {key:'Appearance Date',  sortable:true,  headerStyle:'text-danger',     cellStyle:'font-size: 16px;'},
        ]  
        
    ];

    public getDocuments(): void {
        
        this.documents = this.civilFileInformation.documentsInfo;
        this.summaryDocuments = this.civilFileInformation.summaryDocumentsInfo;
        this.categories = this.civilFileInformation.categories;
        this.categories.sort()
        if(this.summaryDocuments.length > 0) this.categories.push("COURT SUMMARY")
        this.categories.unshift("ALL")        
        this.isMounted = true;
    }

    @Watch('$route', { immediate: false, deep: true })
    onUrlChange() {
        this.civilFileInformation.fileNumber = this.$route.params.fileNumber;
        this.UpdateCivilFile(this.civilFileInformation);
        location.reload();
    }

    mounted () { 
        this.civilFileInformation.fileNumber = this.$route.params.fileNumber
        this.UpdateCivilFile(this.civilFileInformation);        
        this.getDocuments();        
    }

    public cellClick(data)
    {          
        if(data.value !='CourtSummary')        
            this.openDocumentsPdf(data.item['Document ID']);        
        else        
            this.openCourtSummaryPdf(data.item['Appearance ID'])              
    }

    public navigateToLandingPage() {
        this.$router.push({name:'Home'})
    } 

    public ExtractIssues(issues) {
        let issueString =''; 
        for (const issue of issues)
        {
            issueString += issue.issueDsc + '\n';
        }
        return issueString
    }

    get FilteredDocuments() {

        if(this.activetab == 'COURT SUMMARY')
        {
            this.fieldsTab = fieldTab.Summary;
            return this.summaryDocuments;
        }
        else {       
            return this.documents.filter(doc => {                
                this.fieldsTab = fieldTab.Categories;
                if(this.activetab == 'CONCLUDED') {
                    if(doc["Concluded"] === "Y") return true; else return false;
                
                } else if(this.activetab == 'SCHEDULED') {
                    if(doc["Appearance Date"]){        
                        if(new Date(doc["Appearance Date"]) > new Date()) return true; else return false;
                    
                    } else {
                        return false
                    }  

                } else if ( this.activetab != 'ALL' )
                {
                    if (doc["Category"].toUpperCase() == this.activetab.toUpperCase()) return true;                                   
                                  
                    return false;                     
                }
                else
                {
                    return true;
                }
            }); 
        }       
    }
    
    get sortBy()
    {
        if(this.activetab == 'COURT SUMMARY')
        {
            this.sortDesc = true;
            return 'Appearance Date';
        }
        else
        {
           this.sortDesc = false;
           return 'Seq.'; 
        }
    }

    public openDocumentsPdf(documentId): void {
        this.loadingPdf = true;
        const filename = 'doc'+documentId+'.pdf';
        window.open(`/api/files/document/${documentId}/${filename}?isCriminal=false`)
        this.loadingPdf = false;
    }
    
    public openCourtSummaryPdf(appearanceId): void {

        this.loadingPdf = true;        
        const filename = 'court summary_'+appearanceId+'.pdf';
        window.open(`/api/files/civil/court-summary-report/${appearanceId}/${filename}`)
        this.loadingPdf = false;
    }
    
    get NumberOfDocuments() {       
        if(this.activetab == 'COURT SUMMARY')
        {           
            return(this.summaryDocuments.length)
        }
        else{  
            return(this.documents.length)            
        }    
    }
}
</script>

<style scoped>
 .card {
        border: white;
    }

</style>