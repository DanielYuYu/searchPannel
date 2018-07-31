<template>

    <div>
        <div class="search-panel flex-container flex-center-horizontal">

            <div>
                <div class="flex-container flex-center-vertical">
                    <div class="search-label">查詢區間(一年以內)</div>
                    <div>自</div>
                    <div>
                    <!--畫面上產生datepicker-->
                        <datepicker
                            :value.sync="dateSearch.startDate"
                            :disabled-days-of-Week="disabled"
                            :format="'yyyy-MM-dd'"
                            :show-reset-button="reset">
                        </datepicker>
                    </div>
                    <div>&nbsp;&nbsp;~迄&nbsp;&nbsp;</div>
                    <div>
                        <datepicker
                            :value.sync="dateSearch.endDate"
                            :disabled-days-of-Week="disabled"
                            :format="'yyyy-MM-dd'"
                            :show-reset-button="reset">
                        </datepicker>
                    </div>
                </div>
                <br>
                <sending>
                    <div class="flex-container">
                        <div class="search-label" style="width: 120px">查詢區域</div>
                        <div>
                            <div class="flex-container flex-center-vertical">
                                <div class="region-select-container flex-container">
                                    <div class="region-select-label">區域</div>
                                    <select class="form-control" v-model="selectedRegionId">
                                        <option :value="null">全部</option>
                                    <!--v-for是取得不重覆的名稱，selectedRegionId是名稱的連動，value可以顯示名字，但因為還要利用它作後台select條件-->
                                    <option v-for="a in getRegionIdName":value="a.regionId">{{a.regionName}}</option>
               
                                    </select>
                                </div>
                                <div class="region-select-container flex-container">
                                    <div class="region-select-label">科別</div>
                                    <select class="form-control" v-model="selectedTeamId">
                                        <option :value="null">全部</option>
                                        <option v-for="b in getCenterIdName" :value="b.centerId">{{b.centerName}}</option>
                                        
                                    </select>
                                </div>
                            </div>
                        </div>
                    </div>
                    <br/>
                    <div class="flex-container">
                        <div class="search-label" style="width: 120px"></div>
                        <div>
                            <div class="flex-container flex-center-vertical">
                                <div class="region-select-container flex-container">
                                    <div class="region-select-label">專家</div>
                                    <select class="form-control" v-model="selectedExpert">
                                        <option :value="null">全部</option>
                                         <option v-for="c in getIdName" :value="c.account">{{c.name}}</option>
                                    </select>
                                </div>
                            </div>
                        </div>
                    </div>

                    <br>
                    <div class="flex-container flex-center-horizontal">
                        <button class="btn btn-action" @click="handleSearchClicked">查詢</button>
                        &nbsp;
                        &nbsp;
                        &nbsp;
                        <button class="btn btn-action" @click="handleDownloadClicked">報表下載</button>
                    </div>
                </sending>

            </div>

        </div>

    </div>

</template>

<style scoped lang="scss" rel="stylesheet/scss">
    .search-panel {
        font-size: 22px;
    }

    .search-label {
        color: #87a4b8;
    }

    .region-select-label {
        width: 120px;
        margin-left: 8px;
        text-align: right;
        padding-right: 4px;
    }

    .region-select-container {
        width: 300px;
    }

    .datepicker {
        font-size: 15px;
    }
</style>

<script type="text/ecmascript-6">
    import { modal, datepicker, alert } from 'vue-strap';
    import moment from 'moment';
    import Sending from '../ui/Sending.vue';
    import _ from 'underscore';
    import $ from 'jquery';

    export default {
        props: ['handleSearch', 'handleDownload', 'yearLimit',　'clearSearch'],
        data() {
            return {
                dateSearch: {
                    startDate: moment().startOf('M').format('YYYY-MM-DD'),
                    endDate: moment().endOf('d').format('YYYY-MM-DD'),
                },

                experts: [],
                selectedRegionId: null,
                selectedTeamId: null,
                selectedExpert: null,
                selectedRegionName: '全部',
                selectedTeamIdName: '全部',
                selectedExpertName: '全部',
                
            };
        },
        computed: {
          //讓下拉選單顯示的名稱不重覆
          getRegionIdName(){
                   let result = {};
                   let finalResult=[];
                   for(let i=0;i<this.experts.length;i++){
                       result[this.experts[i].regionName]=this.experts[i];
                   }
      
                   for(var item in result){
                       finalResult.push(result[item]);
                   }
                  return finalResult;
             },
           getCenterIdName(){
                   let result = {};
                   let finalResult=[];
                   for(let i=0;i<this.experts.length;i++){
                       result[this.experts[i].centerName]=this.experts[i];
                   }
      
                   for(var item in result){
                       finalResult.push(result[item]);
                   }
                  return finalResult.filter(it => it.regionId === this.selectedRegionId);  
             },
             
           getIdName(){
                   let result = {};
                   let finalResult=[];
                   for(let i=0;i<this.experts.length;i++){
                       result[this.experts[i].name]=this.experts[i];
                   }
      
                   for(var item in result){
                       finalResult.push(result[item]);
                   }
                 
                  return finalResult.filter(it => it.centerId === this.selectedTeamId);
             }
        },
        components: {
            Sending,
            datepicker,
        },
        watch: {
            dateSearch: {
                handler(newVal) {
                    this.loadRegionAndLocate();
                },
                deep: true,
            },
            selectedRegionId: {
            //這裡是下拉選單的連動
                handler (newVal){
                if (newVal) {
                         this.selectedRegionName = _(this.experts).find(it => it.regionId === this.selectedRegionId).regionName;
                  } else {
                          this.selectedRegionName = '全部';
                  }
                   this.selectedTeamId = null;
                   this.selectedExpert = null;
               },
             deep: true,
            },
            
            selectedTeamId: {
                handler (newVal){
                  if (newVal) {
                          this.selectedTeamIdName = _(this.experts).find(it => it.centerId === this.selectedTeamId).centerName;
                  } else {
                          this.selectedTeamIdName = '全部';
                  }
                  this.selectedExpert = null;
                },
             deep: true,
            },
            selectedExpert: { 
                handler (newVal){
                  if (newVal) {
                          this.selectedExpertName = _(this.experts).find(it => it.account === this.selectedExpert).name;
                  }	else {
                          this.selectedExpertName = '全部';
                  }
                },
                deep: true,
            },
        },
        created() {
           //網頁打開的時候會先呼叫method
            this.loadRegionAndLocate();
        },
        ready() {
            const el = this.$el;
            _(el.querySelectorAll("input.datepicker-input")).each((it) => {
                $(it).on("keypress keydown keyup", function (e) {
                    e.preventDefault();
                    e.stopPropagation();
                });
            })
        },
        methods: {
            loadRegionAndLocate() {
            //利用開始及結束時間去POST後台API，將回傳的Json存到experts陣列裡
                const req = {
                    startDate: moment(this.dateSearch.startDate, 'YYYY-MM-DD'),
                    endDate: moment(this.dateSearch.endDate, 'YYYY-MM-DD'),
                };
                this.$http.post('/mvc/api/v1/admin/reports/ExpertTrainingEventReport', req)
                    .then((res) => {
                        this.experts = res.data.expertTrainingEventHistories;
                });
            },
            handleSearchClicked() {
                if (this.handleSearch) {
                    const startDate = moment(this.dateSearch.startDate, 'YYYY-MM-DD').toDate();
                    const endDate = moment(this.dateSearch.endDate, 'YYYY-MM-DD').toDate();
                    let yearLimit = this.yearLimit || 1;
                    if (moment(startDate).isBefore(moment().subtract(yearLimit, "y").toDate()) || moment(endDate).isBefore(moment().subtract(yearLimit, "y").toDate())) {
                        window.alert(`日期區域不得超過${yearLimit}年以內。`);
                        return;
                    }
                    if (moment(endDate).isBefore(moment(startDate))) {
                        window.alert("時間區間不正確。");
                        return;
                    }

                    let diff = Math.abs(moment(startDate).diff(moment(endDate)));
                    let duration = moment.duration(diff);
                    console.log(duration.years());
                    console.log(duration.months());
                    console.log(duration.days());
                    if (duration.months() > 3 || (duration.months() === 3 && duration.days() > 0)) {
                        window.alert("日期區間不可超過三個月。");
                        return;
                    }

                    this.handleSearch({
                        startDate,
                        endDate,
                        regionId: this.selectedRegionId,
                        teamId: this.selectedTeamId,
                        expertId: this.selectedExpert,
                        regionName: this.selectedRegionName,
                        teamIdName: this.selectedTeamIdName,
                        expertIdName: this.selectedExpertName,          
                    });
                }   
            },
            handleDownloadClicked() {
                this.handleSearchClicked();
                this.$nextTick(() => {
                    if (this.handleDownload) {
                        this.handleDownload();
                    }
                });
            }
        },
    };

</script>
