<template>
    <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">

        <v-btn slot="activator" color="primary" fab small dark>
            <v-icon>add</v-icon>
        </v-btn>

        <v-card>

            <v-toolbar dark color="primary">
                <v-btn icon dark @click="dialog = false">
                    <v-icon>close</v-icon>
                </v-btn>
                <v-toolbar-title>Add Template</v-toolbar-title>
            </v-toolbar>

            <v-card-text>
                <v-container grid-list-md pa-0>
                    <v-layout justify-center row wrap>
                        <v-flex xs12 sm8 md6>
                            <v-form ref="adderForm">
                                <v-layout justify-center row wrap>
                                    <v-flex xs12>
                                        <v-subheader>Add a new Template to be used as calorie</v-subheader>
                                    </v-flex>
                                    <v-flex xs12>
                                        <v-text-field v-model="formdata.title" :rules="rules.text" label="Title" outline></v-text-field>
                                    </v-flex>
                                    <v-flex xs12 sm6>
                                        <v-text-field v-model="formdata.calories" :rules="rules.number" label="Calories per 100 g/ml" outline></v-text-field>
                                    </v-flex>
                                    <v-flex xs12 sm6>
                                        <v-text-field v-model="formdata.amount" :rules="rules.number" label="Default amount" outline></v-text-field>
                                    </v-flex>
                                    <v-flex xs12 v-if="!formdata.image">
                                        <div class="dropbox" id="imageUpload">
                                            <input type="file" name="img" @change="filesChange($event.target.name, $event.target.files)" accept="image/*" class="input-file">
                                            <p>Click here or drop image to upload</p>
                                        </div>
                                    </v-flex>
                                    <v-flex v-if="formdata.image" xs12>
                                        <div class="text-xs-center">
                                            <h4>Image</h4>
                                            <v-img :src="formdata.image" max-height="500" contain></v-img>
                                            <v-btn @click="reset()" color="primary">Retry</v-btn>
                                        </div>
                                    </v-flex>
                                    <v-flex xs12>
                                        <v-btn :disabled="disabled" depressed block large color="primary" @click="addTemplate()">Add</v-btn>
                                    </v-flex>
                                </v-layout>
                            </v-form>
                        </v-flex>
                    </v-layout>
                </v-container>
            </v-card-text>

        </v-card>
    </v-dialog>
</template>

<script>
    export default {

        name: 'Adder',
        components: {

        },

        methods: {

            reset() {
                this.uploadedFiles = [];
                this.formdata.image = false;
            },

            filesChange(fieldName, fileList) {

                var vm = this;
                const formData = new FormData();
                if (!fileList.length) {return};
                formData.append(fieldName, fileList[0], fileList[0].name);
                formData.append('jwt', this.$store.state.auth.token);

                vm.axiosPost({
                    url:'template/create/upload/',
                    data: formData,
                }).then(function (response) {
                    var path = response.config.baseURL + 'template/read/thumbnails/' + response.data.content;
                    vm.$data.formdata.image = path;
                }).catch(function (error) {
                    vm.$notify({
                        group: 'default',
                        type: 'error',
                        title: vm.$t('alerts.error.title'),
                        text: vm.$t('alerts.error.text')
                    });
                });

            },

            addTemplate(){

                var vm = this;

                if(vm.$refs.adderForm.validate()){
                    var postData = vm.$data.formdata;
                    postData.jwt = this.$store.state.auth.token;
                    vm.$data.disabled=true;

                    vm.axiosPost({
                        url:'template/create/',
                        data: postData,
                    }).then(function (response) {

                        vm.$notify({
                            group: 'default',
                            type: 'success',
                            title: vm.$t('alerts.success.title'),
                            text: vm.$t('alerts.success.text')
                        });

                        vm.$store.state.content.templates.unshift({
                            id: response.data.content,
                            title: vm.$data.formdata.title,
                            calories: vm.$data.formdata.calories,
                            amount:  vm.$data.formdata.amount,
                            image:  vm.$data.formdata.image,
                        });

                        vm.$data.dialog = false;
                        vm.$refs.adderForm.reset();
                        vm.reset();
                        vm.$data.disabled=false;

                    }).catch(function (error) {
                        vm.$notify({
                            group: 'default',
                            type: 'error',
                            title: vm.$t('alerts.error.title'),
                            text: vm.$t('alerts.error.text')
                        });
                        vm.$data.disabled=false;
                    });
                }
            }

        },

        data () {
            return {
                uploadedFiles: [],
                dialog: false,
                disabled: false,
                formdata: {
                    title: null,
                    calories: null,
                    amount: null,
                    image: null
                },
                rules: {
                    valid: false,
                    text: [
                    (v) => !!v || this.$t('errors.required'),
                    (v) => v && v.length <= 100 || this.$t('errors.valid')
                    ],
                    number: [
                    (v) => !!v || this.$t('errors.required'),
                    (v) => v && v <= 9999 && v >= 0 || this.$t('errors.valid')
                    ],
                }
            }
        }

    }
</script>

<style scoped>

    .dropbox {
        outline: 2px dashed grey; /* the dash box */
        outline-offset: -10px;
        color: dimgray;
        padding: 10px 10px;
        min-height: 200px; /* minimum height */
        position: relative;
        cursor: pointer;
    }

    .input-file {
        opacity: 0; /* invisible but it's there! */
        width: 100%;
        height: 200px;
        position: absolute;
        cursor: pointer;
    }

    .dropbox:hover {
        background: lightgrey; /* when mouse over to the drop zone, change color */
    }

    .dropbox p {
        font-size: 1.2em;
        text-align: center;
        padding: 50px 0;
    }

</style>