<template>
<div style="overflow:hidden"> 

    <b-card bg-variant="light" v-if= "!isMounted && !isDataReady">
        <b-overlay :show= "true"> 
            <b-card  style="min-height: 100px;"/>                   
            <template v-slot:overlay>               
            <div> 
                    <loading-spinner/> 
                    <p id="loading-label">Loading ...</p>
            </div>                
            </template> 
        </b-overlay> 
    </b-card>

    <b-card bg-variant="light" v-if= "isMounted && !isDataReady">
        <b-card  style="min-height: 100px;">
            <span v-if="errorCode==404">This <b>File-Number '{{this.criminalFileInformation.fileNumber}}'</b> doesn't exist in the <b>criminal</b> records. </span>
            <span v-else-if="errorCode==200 || errorCode==204"> Bad Data in <b>File-Number '{{this.criminalFileInformation.fileNumber}}'</b> ! </span>
            <span v-else> Server is not responding. <b>({{errorText}})</b> </span>
        </b-card>
        <b-card>         
            <b-button variant="info" @click="navigateToLandingPage">Back to the Landing Page</b-button>
        </b-card>
    </b-card>

    <b-row cols="2" >            
        <b-col md="2" cols="2" style="overflow: auto;">
            <criminal-side-panel v-if="isDataReady"/> 
        </b-col>
        <b-col col md="10" cols="10" style="overflow: auto;">

            <criminal-header-top v-if="isDataReady"/> 
            <criminal-header v-if="isDataReady"/> 

            <h2 style= "white-space: pre" v-if="isDataReady">
                {{selectedSideBar}}
            </h2>

            <criminal-participants v-if="showCaseDetails"/>            
            <criminal-adjudicator-restrictions v-if="showCaseDetails"/>
            <criminal-crown-information v-if="showCaseDetails"/>
            <criminal-crown-notes v-if="showCaseDetails"/>
            <criminal-past-appearances v-if="showPastAppearances" />
            <criminal-future-appearances v-if="showFutureAppearances" />
            <criminal-documents-view v-if="showDocuments"/>
            <criminal-witnesses v-if="showWitnesses" />
            <criminal-sentence v-if="showSentenceOrder"/>
            <b-card><br></b-card>  
        </b-col>
    </b-row>

    <b-modal v-if= "isMounted" v-model="banExists" id="bv-modal-ban" hide-header hide-footer>        
        <b-card> 
            A Ban has been ordered on at least one participant in this case. 
            Please check Ban Details before giving out sensitive details.
        </b-card>                     
        <b-button class="mt-3 bg-primary" @click="$bvModal.hide('bv-modal-ban')">Continue</b-button>
    </b-modal>
</div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { namespace } from 'vuex-class';
import CriminalDocumentsView from '@components/criminal/CriminalDocumentsView.vue';
import CriminalHeaderTop from '@components/criminal/CriminalHeaderTop.vue';
import CriminalHeader from '@components/criminal/CriminalHeader.vue';
import CriminalSidePanel from '@components/criminal/CriminalSidePanel.vue';
import CriminalParticipants from '@components/criminal/CriminalParticipants.vue';
import CriminalAdjudicatorRestrictions from '@components/criminal/CriminalAdjudicatorRestrictions.vue'
import CriminalCrownInformation from '@components/criminal/CriminalCrownInformation.vue';
import CriminalPastAppearances from '@components/criminal/CriminalPastAppearances.vue'
import CriminalFutureAppearances from '@components/criminal/CriminalFutureAppearances.vue'
import CriminalCrownNotes from '@components/criminal/CriminalCrownNotes.vue';
import CriminalWitnesses from '@components/criminal/CriminalWitnesses.vue';
import CriminalSentence from '@components/criminal/CriminalSentence.vue';
import '@store/modules/CriminalFileInformation';
import "@store/modules/CommonInformation";
const criminalState = namespace('CriminalFileInformation');
const commonState = namespace("CommonInformation");

enum DecodeCourtLevel {'P'= 0, 'S' = 1, 'A' = 2 }
enum DecodeCourtClass {
    'A' = 0, 'Y' = 1, 'T' = 2, 'F' = 3, 'C' = 4, 'M' = 5,        
    'L' = 6, 'R' = 7, 'B' = 8, 'D' = 9, 'E' = 10, 'G' = 11,        
    'H' = 12, 'N' = 13, 'O' = 14, 'P' = 15, 'S' = 16, 'V' = 17,
}

@Component({
    components: {
        CriminalDocumentsView,
        CriminalSidePanel,
        CriminalHeaderTop,
        CriminalHeader,
        CriminalParticipants,
        CriminalAdjudicatorRestrictions,
        CriminalCrownInformation,
        CriminalPastAppearances,
        CriminalFutureAppearances,
        CriminalCrownNotes,
        CriminalWitnesses,
        CriminalSentence
    }
})
export default class CriminalCaseDetails extends Vue {

    @criminalState.State
    public showSections
    
    @commonState.State
    public displayName!: string;

    /* eslint-disable */
    @criminalState.State
    public criminalFileInformation!: any

    @criminalState.Action
    public UpdateCriminalFile!: (newCriminalFileInformation: any) => void
    
    @commonState.Action
    public UpdateDisplayName!: (newInputNames: any) => void
   
    participantList: any[] = [];
    adjudicatorRestrictionsInfo: any[] = [];
    bans: any[] = [];
    /* eslint-enable */

    isDataReady = false
    isMounted = false
    banExists = false
    errorCode =0;
    errorText ='';
    
    participantJson;
    adjudicatorRestrictionsJson;
    sidePanelTitles = [ 
       'Case Details', 'Future Appearances', 'Past Appearances', 'Witnesses', 'Documents', 'Sentence/Order Details'    
    ];

    topTitles = [ 
       'Case Details', 'Future Appearances', 'Past Appearances', 'Witnesses', 'Criminal Documents', 'Criminal Sentences'    
    ];

    statusFields = 
    [
        {key:'Warrant Issued',      abbr:'W',   code:'warrantYN'},
        {key:'In Custody',          abbr:'IC',  code:'inCustodyYN'},
        {key:'Detention Order',     abbr:'DO',  code:'detainedYN'} , 
        {key:'Interpreter Required',abbr:'INT', code:'interpreterYN'}
    ];

    mounted () { 
        this.criminalFileInformation.fileNumber = this.$route.params.fileNumber
        this.UpdateCriminalFile(this.criminalFileInformation);        
        this.getFileDetails();
    }

    public getFileDetails(): void {
        this.errorCode=0;
        this.$http.get('/api/files/criminal/'+ this.criminalFileInformation.fileNumber)
            .then(Response => Response.json(), err => {this.errorCode= err.status;this.errorText= err.statusText;console.log(err);}        
            ).then(data => {
                if(data){
                    this.criminalFileInformation.detailsData = data; 
                    this.participantJson = data.participant
                    this.adjudicatorRestrictionsJson = data.hearingRestriction;
                    const courtLevel = DecodeCourtLevel[data.courtLevelCd];
                    const courtClass = DecodeCourtClass[data.courtClassCd];
                    this.ExtractFileInfo()
                    if(this.participantList.length)
                    {
                        this.criminalFileInformation.participantList = this.participantList;
                        this.criminalFileInformation.adjudicatorRestrictionsInfo = this.adjudicatorRestrictionsInfo;
                        if (this.bans.length > 0) {
                            this.banExists = true;
                        }
                        this.criminalFileInformation.bans = this.bans;
                        this.criminalFileInformation.courtLevel = courtLevel;
                        this.criminalFileInformation.courtClass = courtClass;
                        this.UpdateCriminalFile(this.criminalFileInformation);
                        this.isDataReady = true;
                    }
                    else
                        this.errorCode=200;                    
                }
                else
                    if(this.errorCode==0) this.errorCode=200;
                    
                
                this.isMounted = true;
                       
            });
    }
    
    get selectedSideBar()
    {
        for(const titleInx in this.sidePanelTitles)
        {
          if (this.showSections[this.sidePanelTitles[titleInx]] == true ) return '  '+ this.topTitles[titleInx]
        }
        return ''
    }

    get showCaseDetails()
    {        
        return (this.showSections['Case Details'] && this.isDataReady)
    }
    
    get showDocuments()
    {        
        return (this.showSections['Documents'] && this.isDataReady)
    }

    get showFutureAppearances()
    {        
        return ((this.showSections['Case Details'] || this.showSections['Future Appearances'] ) && this.isDataReady)
    }

    get showPastAppearances()
    {        
        return ((this.showSections['Case Details'] || this.showSections['Past Appearances'] ) && this.isDataReady)
    }

    get showWitnesses()
    {        
        return (this.showSections['Witnesses'] && this.isDataReady)
    }

    get showSentenceOrder()
    {        
        return (this.showSections['Sentence/Order Details'] && this.isDataReady)
    }

    public ExtractFileInfo(): void {

        for (const partIndex in this.participantJson) {
            const participantInfo = {};
            const jParticipant = this.participantJson[partIndex];
            participantInfo["Index"] = partIndex;
            participantInfo["First Name"] = jParticipant.givenNm.trim().length>0 ? jParticipant.givenNm : "";
            participantInfo["Last Name"] = jParticipant.lastNm ? jParticipant.lastNm : jParticipant.orgNm;
            this.UpdateDisplayName({'lastName': participantInfo["Last Name"], 'givenName': participantInfo["First Name"]});
            participantInfo["Name"] = this.displayName;

            participantInfo["D.O.B."] = jParticipant.birthDt? (new Date(jParticipant.birthDt.split(' ')[0])).toUTCString().substr(4,12) : '';
            participantInfo["Part ID"] = jParticipant.partId;
            participantInfo["Prof Seq No"] = jParticipant.profSeqNo;
            participantInfo["Charges"] = [];         
            const charges: any[] = [];         
            for(const charge of jParticipant.charge)
            {              
                const chargeInfo = {};                   
                chargeInfo["Description"]= charge.sectionDscTxt
                chargeInfo["Code"]= charge.sectionTxt
                charges.push(chargeInfo);
            }
            participantInfo["Charges"] = charges;

            participantInfo["Status"] = [];
            for (const status of this.statusFields)
            {
                if(jParticipant[status.code] =='Y')
                    participantInfo["Status"].push(status);
            }
            
            for(const ban of jParticipant.ban)
            {              
                const banInfo = {};
                banInfo["Ban Participant"] = participantInfo["Name"];                   
                banInfo["Ban Type"] = ban.banTypeDescription;
                banInfo["Order Date"] = ban.banOrderedDate;                   
                banInfo["Act"] = ban.banTypeAct;
                banInfo["Sect."] = ban.banTypeSection;                   
                banInfo["Sub"] = ban.banTypeSubSection;
                banInfo["Description"] = ban.banStatuteId;                   
                banInfo["Comment"] = ban.banCommentText;                  
                this.bans.push(banInfo);
            }                      

            participantInfo['DocumentsJson'] = jParticipant.document;
            participantInfo['CountsJson'] = jParticipant.count;

            this.UpdateDisplayName({'lastName': jParticipant.counselLastNm? jParticipant.counselLastNm: '', 'givenName': jParticipant.counselGivenNm? jParticipant.counselGivenNm: ''});
            participantInfo['Counsel'] = this.displayName.trim.length? 'JUSTIN: ' + this.displayName: '';
            participantInfo['Counsel Designation Filed'] = jParticipant.designatedCounselYN
            this.participantList.push(participantInfo);
        }

        for (const jRestriction of this.adjudicatorRestrictionsJson) {
            const restrictionInfo = {};     
            restrictionInfo["Adj Restriction"] = jRestriction.adjInitialsTxt?jRestriction.hearingRestrictionTypeDsc+ ": " + jRestriction.adjInitialsTxt:jRestriction.hearingRestrictionTypeDsc;
            restrictionInfo["Full Name"] = jRestriction.adjFullNm;                 
            restrictionInfo["Adjudicator"] =   jRestriction.adjInitialsTxt?jRestriction.adjInitialsTxt +" - " + jRestriction.adjFullNm: jRestriction.adjFullNm;
            restrictionInfo["Status"] = jRestriction.hearingRestrictionTypeDsc + ' ';
            restrictionInfo["Applies to"] = jRestriction.partNm ? jRestriction.partNm: 'All participants on file'      
            this.adjudicatorRestrictionsInfo.push(restrictionInfo);      
        }
    }

    public navigateToLandingPage() {
        this.$router.push({name:'Home'})
    }    
    
}
</script>

<style scoped>
    .card {
        border: white;
    }

   body {
        overflow-x: hidden;
    }
</style>