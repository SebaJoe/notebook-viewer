<template>
    <div class="container-fluid">

        <div class="row bg-light p-4">
            <div class="col">
                <div class="row">
                    <div class="col">
                        <h1>Notebook Query Viewer</h1>
                    </div>
                </div>
                <div class="row mt-2" >
                    <div class="col">
                        This is an interface for viewing and annotating benchmark queries for notebooks.
                        The guidelines for completing annotations can be found <a class="link-offset-1 link-underline link-underline-opacity-0 link-underline-opacity-100-hover" href="https://docs.google.com/presentation/d/1ip0RqfWrYtriep6HTK9h7TkhKxqnXHUo4AP6vM6g2sg/edit?usp=sharing">here</a>.
                    </div>
                </div>
            </div>
        </div>
    
    </div>

    <div class="container">

        <div class="row mt-4">
            <div class="col text-center">
                <h2>{{ parsed_file[findex]['title'] }}</h2>
            </div>  
        </div>

        <div class="row mt-2">
            <div class="col-3">
                <input class="form-control" type="file" ref="doc" @change="readfile()"/>
            </div>
            <div class="col">
                <button class="btn btn-dark float-right" style="float:right;" @click="dec_findex()" :disabled="disable_movement">
                    <i class="bi bi-chevron-left"></i>
                </button>
            </div>
            <div class="col d-flex justify-content-center">
                <h4><input style="width:45px;" :value="findex + 1" @input="move_to_target($event)"> / {{ parsed_file.length }}</h4>
            </div>
            <div class="col">
                <button class="btn btn-dark float-left" style="float:left;" @click="inc_findex()" :disabled="disable_movement">
                    <i class="bi bi-chevron-right"></i>
                </button>
            </div>
            <div class="col-3">
                <button class="btn btn-outline-secondary float-right" style="float:right;" @click="download_parsed_file()">
                    Download
                </button>
            </div>
        </div>

        <div class="row mt-2">
            <div class="col">
                <div class="row mt-1 mb-3 border-bottom border-dark">
                    <div class="col">
                        <h3>Original</h3>
                    </div>
                    <div class="col">
                        <h3>Generated Queries</h3>
                    </div>
                    <div class="col-1 my-auto">
                        <i>Options</i>
                    </div>
                </div>
                <div v-for="(alignment, index) in parsed_file[Math.max(0, findex)]['alignments']" :key="parsed_file[Math.max(0, findex)]['title'] + index">
                    <div v-if="alignment['cell_type'] == 'markdown'">
                        <MarkCell
                            :ground_truth="(alignment['matching'][0] == -1) ? null_val : parsed_file[findex]['source'][alignment['matching'][0]]['text']"
                            :eval_side="(alignment['matching'][1] == -1) ? null_val : parsed_file[findex]['target'][alignment['matching'][1]]['text']"
                            v-model:flagged="alignment['flagged']"
                            v-model:starred="alignment['starred']"
                            v-model:edited_text="alignment['edited_text']"
                            v-model:comments="alignment['comments']"
                        />
                    </div>
                    <div v-else>
                        <MarkCell
                            :ground_truth="(alignment['matching'][0] == -1) ? null_val : parsed_file[findex]['source'][alignment['matching'][0]]['text']"
                            :eval_side="(alignment['matching'][1] == -1) ? null_val : parsed_file[findex]['target'][alignment['matching'][1]]['text']"
                            v-model:flagged="alignment['flagged']"
                            v-model:starred="alignment['starred']"
                            v-model:edited_text="alignment['edited_text']"
                            v-model:comments="alignment['comments']"
                            is_code
                        />
                        <MarkCell
                            :ground_truth="(alignment['matching'][0] == -1) ? null_val : parsed_file[findex]['source'][alignment['matching'][0]]['output']"
                            :eval_side="(alignment['matching'][1] == -1) ? null_val : parsed_file[findex]['target'][alignment['matching'][1]]['output']"
                            v-model:flagged="alignment['flagged']"
                            v-model:starred="alignment['starred']"
                            v-model:edited_text="alignment['edited_text']"
                            v-model:comments="alignment['comments']"
                            is_output
                        />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import MarkCell from './MarkCell.vue'
    import download from "downloadjs";
    import axios from 'axios';

    export default {
        components: {
            MarkCell,
        },
        data () {
            return {
                findex: 0,
                file: null,
                parsed_file: [{
                    title: "Default",
                    source: [],
                    target: [],
                    alignments: [],
                }],
                null_val: "<|NULL|>",
            }
        },
        mounted() {
            let urlParams = new URLSearchParams(window.location.search);
            if (urlParams.has('load')) {
                this.load_file_from_sample(urlParams.get('load'));
            }
        },
        methods: {
            dec_findex() {
                if (this.findex > 0) {
                    this.findex--;
                }
            },
            inc_findex() {
                if (this.findex < this.parsed_file.length - 1) {
                    this.findex++;
                }
            },
            move_to_target(target_val) {
                let temp_val = target_val.target.value;
                //console.log("ENFKDF");
                //console.log(temp_val);
                //console.log(typeof temp_val);
                if (isNaN(parseInt(temp_val))) {
                    //console.log("The end times are nigh");
                    return;
                }

                if (temp_val - 1 >= this.parsed_file.length) {
                    this.findex = this.parsed_file.length - 1;
                } else if (temp_val - 1 < 0) {
                    this.findex = 0;
                } else {
                    this.findex = temp_val - 1;
                }
            },
            load_file_from_sample(fname) {
                console.log(fname);
                axios.get(`https://raw.githubusercontent.com/SebaJoe/notebook-viewer/master/samples/${fname}`)
                .then((response) => {
                    this.parsed_file = response.data;                    
                    //console.log(atob(response.data.content));
                    //this.parsed_file = JSON.parse(atob(response.data.content));
                    this.findex = 0;
                })
                .catch((response) => {console.log(response);});
            },
            readfile() {
                this.file = this.$refs.doc.files[0];
                const reader = new FileReader();
                if(this.file.name.includes('.json')) {
                    reader.onload = (res) => {
                        this.parsed_file = JSON.parse(res.target.result);
                        this.findex = 0;
                    }
                    reader.onerror = (err) => console.log(err);
                    reader.readAsText(this.file);
                }
            },
            download_parsed_file () {
                const current_date = new Date();
                let end_object = JSON.stringify(this.parsed_file)
                end_object = end_object.replace(/[\u007F-\uFFFF]/g, function(chr) {
                                return "\\u" + ("0000" + chr.charCodeAt(0).toString(16)).substr(-4);
                            });
                let new_filename = this.file.name.split(".json")[0];
                download(end_object, new_filename + "_annotated_" + current_date.toString().replace(" ", "_") + ".json", 'application/json');
            },
        },
    }
</script>

<style>

</style>