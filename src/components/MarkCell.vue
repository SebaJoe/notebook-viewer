<template>
    <div class="row bg-dark rounded" @click="toggle_button('output_toggle')" v-if="is_output && !output_toggle">
        <div class="col text-center">
            <i class="bi bi-chevron-down text-white h1"></i>
        </div>
    </div>
    <div class="row" 
        :style="{backgroundColor: cell_indicator}"
        :class="{border: is_code || is_output, 'border-3': is_code || is_output, 'rounded': is_code || is_output, 'border-success': is_code, 'border-dark': is_output}" v-else>
        <div class="col border border-right overflow-auto" v-if="!gt_null">
            <vue-markdown :source="ground_truth" :options="md_options"/>
        </div>
        <div class="col border border-right bg-light" v-else>
        </div>
        <div class="col border border-left overflow-auto" :class="{ 'border-3': edited && !original_view , 'border-primary': edited && !original_view }" v-if="!es_null && !commenting">
            <div v-if="!edit_val">
                <vue-markdown :source="eval_side" :options="md_options" v-if="!edited || original_view"/>
                <vue-markdown :source="edited_text" :options="md_options" v-else/>
            </div>
            <textarea class="form-control" style="height: 100%;" v-model="temp_edits" v-else></textarea>
        </div>
        <div class="col border border-right bg-light" v-if="es_null && !commenting">
        </div>
        <div class="col border border-left border-3 border-info" v-if="commenting">
            <textarea class="form-control" style="height: 100%;" @change="change_comment($event)" v-model="temp_comments" placeholder="Make your comments here..."></textarea>
        </div>
        <div class="col-1 my-auto" v-if="!is_output">
            <button class="btn btn-light" @click="toggle_button('expand')" v-if="!expand">
                <i class="bi bi-chevron-right h4"></i>
            </button>
            <div class="btn-group-vertical mt-2 mb-2" v-else>
                    <button class="btn btn-success" title="Star" @click="toggle_button('star')" v-if="starred">
                        <i class="bi bi-star-fill h4"></i>
                    </button>
                    <button class="btn btn-success" title="Star" @click="toggle_button('star')" v-else>
                        <i class="bi bi-star h4"></i>
                    </button>

                    <button class="btn btn-aqua" title="Evaluable" @click="toggle_button('eval')" v-if="evaluable">
                        <i class="bi bi-patch-check-fill h4"></i>
                    </button>
                    <button class="btn btn-aqua" title="Evaluable" @click="toggle_button('eval')" v-else>
                        <i class="bi bi-patch-check h4"></i>
                    </button>

                    <button class="btn btn-danger" title="Flag" @click="toggle_button('flag')" v-if="flagged">
                        <i class="bi bi-flag-fill h4"></i>
                    </button>

                    <button class="btn btn-danger" title="Flag" @click="toggle_button('flag')" v-else>
                        <i class="bi bi-flag h4"></i>
                    </button>

                    <button class="btn btn-primary" title="Done" @click="toggle_button('edit')" v-if="edit_val" :disabled='es_null || commenting'>
                        <i class="bi bi-check-lg h4"></i>
                    </button>
                    <button class="btn btn-primary" title="Edit" @click="toggle_button('edit')" v-else :disabled='es_null || commenting'>
                        <i class="bi bi-pencil h4"></i>
                    </button>

                    <button class="btn btn-info" title="Close Comments" @click="toggle_button('comments')" v-if="commenting" :disabled="edit_val">
                        <i class="bi bi-chat-dots-fill h4"></i>
                    </button>
                    <button class="btn btn-info" title="Open Comments" @click="toggle_button('comments')" v-else :disabled="edit_val">
                        <i class="bi bi-chat-dots h4"></i>
                    </button>

                    <button class="btn btn-dark" title="Look at Edit" @click="toggle_button('oview')" :disabled="!edited" v-if="original_view">
                        <i class="bi bi-back h4"></i>
                    </button>
                    <button class="btn btn-secondary" title="Look at Original" @click="toggle_button('oview')" :disabled="!edited" v-else>
                        <i class="bi bi-back h4"></i>
                    </button>

                    <button type="button" class="btn btn-warning" @click="cancel_edits()" v-if="edited">
                        <i class="bi bi-x h4"></i>
                    </button>

                    <button class="btn btn-light" @click="toggle_button('expand')" v-if="expand">
                        <i class="bi bi-chevron-left h4"></i>
                    </button>
                    
            </div>
        </div>
        <div class="col-1 bg-dark text-center" @click="toggle_button('output_toggle')" v-if="is_output && output_toggle">
            <div class="row h-100">
                <div class="col my-auto">
                    <i class="bi bi-chevron-up text-white h1"></i>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    import VueMarkdown from 'vue-markdown-render'

    import hljs from 'highlight.js'

    export default {
        components: {
            VueMarkdown
        },
        props: {
            ground_truth: { required: true, type: String},
            eval_side: { required: true, type: String},
            flagged: { required: true, default: false, type: Boolean},
            starred: { required: true, default: false, type: Boolean},
            evaluable: {required: true, default: false, type: Boolean},
            edited_text: { required:true, type: String},
            comments: {required: true, default: "", type: String},
            is_code: {default: false, type: Boolean},
            is_output: {default: false, type: Boolean},
        },
        data () {
            var defaults = {
                    html: true,
                    typographer: true,
                    _highlight: true, // <= THIS IS WHAT YOU NEED
                };

                // and then do this:

                defaults.highlight = function (str, lang) {
                    if (lang && hljs.getLanguage(lang)) {
                        try {
                            return hljs.highlight(str, { language: lang }).value;
                        } catch (__) {}
                    }

                    return ''; // use external default escaping
                }

                console.log(this.edited_text);

                return {
                    md_options: defaults,
                    star_val: this.starred,
                    edit_val: false,
                    gt_null: this.ground_truth === "<|NULL|>",
                    es_null: this.eval_side === "<|NULL|>",
                    edited: this.edited_text !== "<DEFAULT>",
                    temp_edits: "",
                    original_view: false,
                    output_toggle: false,
                    temp_comments: this.comments,
                    cell_indicator: this.update_colors_return(),
                    expand: false,
                    commenting: false,
                }
        },
        methods: {
            toggle_button(type) {
                if (type === 'star') {

                    this.$emit('update:starred', !this.starred);
                    this.$nextTick(() => {
                        this.update_colors();
                    });
                } else if (type === 'flag') {

                    this.$emit('update:flagged', !this.flagged);
                    this.$nextTick(() => {
                        this.update_colors();
                    });
                } else if (type === 'eval') {

                    this.$emit('update:evaluable', !this.evaluable);
                    this.$nextTick(() => {
                        this.update_colors();
                    });
                } else if (type === 'edit') {
                    if (!this.edit_val) this.start_editing();
                    else this.end_editing();
                } else if (type === 'oview') {
                    this.original_view = !this.original_view;
                } else if (type === 'expand') {
                    this.expand = !this.expand;
                } else if (type === 'output_toggle') {
                    this.output_toggle = !this.output_toggle;
                } else if (type === 'comments') {
                    this.commenting = !this.commenting;
                }
            },
            start_editing() {
                if (this.edited_text === "<DEFAULT>") {
                    this.temp_edits = this.eval_side;
                }
                this.edit_val = true;
            },
            end_editing() {
                console.log("here");
                this.edited = true;
                this.$emit('update:edited_text', this.temp_edits);
                this.$nextTick(() => {
                    console.log(this.edited_text);
                    this.edit_val = false;
                });
            },
            cancel_edits() {
                this.temp_edits = this.eval_side;
                this.$emit('update:edited_text', "<DEFAULT>");
                this.$nextTick(() => {
                    this.edited = false;
                });
            },
            update_colors() {
                this.cell_indicator = this.update_colors_return();
            },
            update_colors_return() {
                if ((this.starred || this.evaluable) && this.flagged) {
                    return "#ECD2FC";
                }
                if (this.flagged) {
                    return "#F9C0CB";
                }
                if (this.starred) {
                    return "#CBF9C0";
                }
                if (this.evaluable) {
                    return "#bfffe9";
                }
                return (this.is_code || this.is_output) ? "#EEEEEE": "#FFFFFF";
            },
            change_comment(event) {
                this.$emit('update:comments', this.temp_comments);
                this.$nextTick(() => {
                    console.log(this.comments);
                });
            }
        },
    }
</script>

<style>
.overlay-button {
    position: absolute;   /* Position button absolutely within the container */
    top: 10px;             /* 10px from the top */
    right: 10px;           /* 10px from the right */
    z-index: 10;
}

.btn-aqua {
    background-color: #7FFFD4;
    border: #00F4A2;
}

.btn-aqua:hover {
    background-color: #00F4A2;
    border: #00C684;
}

</style>