<template>
    <TabBlock :Tab="Tab">
        <div class="poc-container">
            <div class="poc-search-container">
                <select class="poc-page-size-input" v-model="poc">
                    <option value="" selected disabled>CHOOSE POC</option>
                    <option v-for="(value, key) in pocs" :key="key" :value="key">{{key}}</option>
                </select>
                <input type="text" class="poc-search-input" placeholder="TARGET" v-model="target">
                <div class="poc-search" @click="doPoc">
                    ATTACK
                </div>
            </div>
            <div class="result-container">
                <div class="poc-info-container">
                    <div class="poc-title">POC INFO</div>
                    <div class="poc-info-item" v-if="poc != ''">
                        <div>Date</div>
                        <div>{{pocs[poc].date == "" || pocs[poc].date == undefined ? "Unknown" : pocs[poc].date}}</div>
                    </div>
                    <div class="poc-info-item" v-if="poc != ''">
                        <div>ID</div>
                        <div>{{pocs[poc].id == "" || pocs[poc].id == undefined ? "Unknown" : pocs[poc].id}}</div>
                    </div>
                    <div class="poc-info-item" v-if="poc != ''">
                        <div>Platform</div>
                        <div>{{pocs[poc].platform == "" || pocs[poc].platform == undefined ? "Unknown" : pocs[poc].platform}}</div>
                    </div>
                    <div class="poc-info-item" v-if="poc != ''">
                        <div>Reference</div>
                        <div>{{pocs[poc].reference == "" || pocs[poc].reference == undefined ? "Unknown" : pocs[poc].reference}}</div>
                    </div>
                    <div class="poc-info-item" v-if="poc != ''">
                        <div>Text</div>
                        <div>{{pocs[poc].text == "" || pocs[poc].text == undefined ? "Unknown" : pocs[poc].text}}</div>
                    </div>
                    <div class="poc-info-item" v-if="poc != ''">
                        <div>Type</div>
                        <div>{{pocs[poc].type == "" || pocs[poc].type == undefined ? "Unknown" : pocs[poc].type}}</div>
                    </div>
                </div>
                <div class="poc-result-container">
                    <div class="poc-result-top-container">
                        <div class="poc-result-id">#</div>
                        <div class="poc-result-url">URL</div>
                        <div class="poc-result-status">STATUS</div>
                    </div>
                    <div class="poc-result-main-container">
                        <div class="poc-result-main-item" v-for="(item, index) in result" :key="index">
                            <div class="poc-result-id">{{index+1}}</div>
                            <div class="poc-result-url">{{item.host}}</div>
                            <div class="poc-result-status">{{item.exploitable}}</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </TabBlock>
</template>

<script>
import Vue from 'vue'
import TabBlock from '../components/TabBlock'
export default {
    name: 'Poc',
    components: {
        TabBlock,
    },
    data() {
        return {
            show: false,
            Tab: {
                title: 'Assassinate',
                subtitle: 'poc'
            },
            pocs: {},
            target: "",
            poc: "",
            urls: [],
            result: [],
        }
    },
    methods: {
        getPoc () {
            const url = "/api/poc";
            const msg = {
            };
            this.pocs = {};
            this.ajax_get(url, msg).then(data => {
                if(data.flag == 1) {
                    for(let i in data.data.poc_list) {
                        Vue.set(this.pocs, i, data.data.poc_list[i]);
                        // this.pocs[i] = data.data.poc_list[i];
                    }
                }
            });
        },
        doPoc () {
            const url1 = "/api/target";
            const url2 = "/api/poc/"+this.poc;
            const url3 = "/poc/"+this.poc;
            let target = this.target.split(",");
            this.result = [];
            if(target.length > 1) {
                const data = {
                    targets: this.target
                };
                this.ajax_post(url1, data).then(data => {
                });
                const msg = {
                    concurrency: 10,
                };
                this.ws(url3, msg, this.showResult);
            }
            else if(target.length ==  1){
                const data = {
                    target: this.target
                };
                this.ajax_post(url1, data).then(data => {
                });
                const msg = {

                };
                this.ajax_get(url2, msg).then(data => {
                    if(data.flag == 1) {
                        this.result.push(data.data);
                    }

                })
            }
        },
        showResult (data) {
            this.result.push(data);
        }
    },
    watch: {
        show: function () {
            if(this.show == true) {
                this.getPoc();
            }
        }
    }
}
</script>

<style scoped>
.poc-container {
    width: 100%;
    height: 607px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
}
.poc-search-container {
    height: 100px;
    width: 100%;
    margin: 20px 0 0 0;
    display: flex;
    justify-content: center;
    align-items: center;
}
.poc-search-container > select {
    cursor: pointer;
}
.poc-select-container {
    height: 50px;
    width: 80px;
    box-sizing: border-box;
    padding-left: 20px;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    border: 1px solid gray;
    border-right: none;
    border-top-left-radius: 5px;
    border-bottom-left-radius: 5px;
}
.angle {
    font-size: 12px;
    margin-right: 10px;
}
.poc-select {
    position: relative;
    height: 100%;
    width: 25px;;
}
.poc-select:hover .poc-options-container {
    display: block;
}
.poc-select i {
    margin: 17px 0;
    text-decoration: none;
}
.poc-search-input,
.poc-page-size-input {
    height: 50px;
    width: 500px;
    box-sizing: border-box;
    padding: 0 20px;
    border: 1px solid gray;
    font-size: 16px;
}
.poc-page-size-input {
    width: 200px;
    border-radius: 0px;
}
.poc-search-input:focus {
    outline: none;
    border: 1px solid gray;
}
.poc-page-size-input:focus {
    outline: none;
    border: 1px solid gray;
}
.poc-options-container {
    display: none;
}
.poc-options {
    cursor: pointer;
}
.poc-search {
    cursor: pointer;
    height: 50px;
    width: 80px;
    line-height: 50px;
    text-align: center;
    box-sizing: border-box;
    border: 1px solid gray;
    border-left: none;
}
.poc-checkbox {
    margin: 0 15px;
}
.poc-help {
    height: 50px;
    width: 120px;
    line-height: 50px;
    text-align: center;
    text-decoration: underline;
}
.poc-result-container {
    height: 100%;
    width: 876px;
    box-sizing: border-box;
    padding: 20px;
    padding-top: 0;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
}
.poc-result-top-container,
.poc-result-main-item {
    height: 50px;
    min-height: 50px;
    width: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    
}
.poc-result-top-container {
    margin-bottom: 10px;
    border-bottom: 2px solid gray;
    color: gray;
    font-weight: bold;
}
.poc-result-main-item {
    box-shadow: none;
}
.poc-result-main-container {
    width: 100%;
    height: 480px;
    box-sizing: border-box;
    overflow: auto;
}
.poc-result-id,
.poc-result-status {
    height: 100%;
    width: 80px;
    line-height: 50px;
    text-align: center;
    /* border: 1px solid gray; */
}
.poc-result-url {
    height: 100%;
    flex-grow: 1;
    max-width: 1100px;
    line-height: 50px;
    text-align: left;
    word-break: break-all;
    overflow: hidden;
}
.result-container {
    height: 540px;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}
.poc-info-container {
    height: 100%;
    width: 270px;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    flex-direction: column;
}
.poc-title {
    height: 50px;
    line-height: 50px;
    width: 100%;
    border-bottom: 2px solid gray;
    color: gray;
    font-weight: bold;
    margin-bottom: 10px;
}
.poc-info-item {
    min-height: 50px;
    height: auto;
    width: 100%;
    box-sizing: border-box;
    padding: 0 10px;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin: 5px;
}
.poc-info-item > div:last-of-type {
    margin-left: 10px;
    color: #e56845;
}
</style>