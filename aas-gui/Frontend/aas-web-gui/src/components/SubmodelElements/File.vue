<template>
    <v-container fluid class="pa-0">
        <v-list-item class="px-1 pb-1 pt-0">
            <v-list-item-title class="text-subtitle-2 mt-2">{{ 'Path: ' }}</v-list-item-title>
        </v-list-item>
        <v-card color="elevatedCard" v-if="fileObject">
            <!-- Path (Value) of the File Element -->
            <v-list nav class="bg-elevatedCard pt-0">
                <v-list-item class="pb-0">
                    <!-- mimeType -->
                    <v-list-item-title>
                        <span class="text-caption">{{ 'Mime Type: ' }}</span>
                        <v-chip label size="x-small" border color="primary">{{ fileObject.mimeType ? fileObject.mimeType : 'no-mime' }}</v-chip>
                    </v-list-item-title>
                    <!-- Donwload File Button -->
                    <template v-slot:append>
                        <v-btn v-if="fileObject.value" size="small" color="primary" class="text-buttonText" :href="localPathValue" target="_blank">Download File</v-btn>
                    </template>
                </v-list-item>
                <!-- Path in Inputfield -->
                <v-list-item class="pt-0">
                    <v-list-item-title>
                        <v-text-field variant="outlined" density="compact" hide-details clearable @keydown.native.enter="updatePath()" v-model="newPathValue" @click:clear="clearPath()" @update:focused="setFocus">
                            <!-- Update Path Button -->
                            <template v-slot:append-inner="{ isFocused }">
                                <v-btn v-if="isFocused" size="small" variant="elevated" color="primary" class="text-buttonText" style="margin-top: -2.5px; right: -6px" @click.stop="updatePath()">
                                    <v-icon>mdi-upload</v-icon>
                                </v-btn>
                            </template>
                        </v-text-field>
                    </v-list-item-title>
                </v-list-item>
                <v-divider v-if="!fileObject.value"></v-divider>
                <!-- Alerts when File was not found/empty -->
                <v-list-item v-if="!fileObject.value">
                    <v-list-item-title class="pt-2">
                        <v-alert text="SubmodelElement doesn't contain a File!" density="compact" type="warning" variant="outlined"></v-alert>
                    </v-list-item-title>
                </v-list-item>
            </v-list>
            <v-divider></v-divider>
            <!-- Action Button to upload a File -->
            <v-list nav class="bg-elevatedCard pa-0">
                <v-list-item>
                    <template v-slot:title>
                        <!-- Upload-Button -->
                        <v-file-input variant="outlined" density="compact" :multiple="false" v-model="newFile" clearable hide-details class="my-1">
                            <template v-slot:append-inner>
                                <v-btn size="small" variant="elevated" color="primary" class="text-buttonText" style="margin-top: -2.5px; right: -6px" @click.stop="uploadFile()">Upload File</v-btn>
                            </template>
                        </v-file-input>
                    </template>
                </v-list-item>
            </v-list>
        </v-card>
    </v-container>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue';
import { useStore } from 'vuex';
import RequestHandling from '../../mixins/RequestHandling';

export default defineComponent({
    name: 'File',
    components: {
        RequestHandling, // Mixin to handle the requests to the AAS
    },
    mixins: [RequestHandling],
    props: ['fileObject'],

    setup() {
        const store = useStore()

        return {
            store, // Store Object
        }
    },

    data() {
        return {
            newPathValue: '',
            newFile: [] as any, // File Object to Upload
            localPathValue: '', // Path to the File when it is embedded to the AAS
            loading: false,
            isFocused: false, // boolean to check if the input field is focused
        }
    },

    mounted() {
        this.newPathValue = this.fileObject.value;
        this.localPathValue = this.getLocalPath(this.fileObject.value);
    },

    watch: {
        // Watch for changes in the selected Node and reset input
        SelectedNode: {
            deep: true,
            handler() {
                this.newPathValue = '';
                this.localPathValue = '';
            }
        },

        // watch for changes in the fileObject and set the newPathValue
        fileObject: {
            deep: true,
            handler() {
                if (!this.isFocused) {
                    this.newPathValue = this.fileObject.value;
                    this.localPathValue = this.getLocalPath(this.fileObject.value);
                }
            }
        }
    },

    computed: {
        // get Registry Server URL from Store
        registryServerURL() {
            return this.store.getters.getRegistryURL;
        },

        // get selected AAS from Store
        SelectedAAS() {
            return this.store.getters.getSelectedAAS;
        },

        // Get the selected Treeview Node (SubmodelElement) from the store
        SelectedNode() {
            return this.store.getters.getSelectedNode;
        },
    },

    methods: {
        // Function to update the Path of the File Element
        updatePath() {
            // console.log("Update Path: " + this.newPathValue);
            let path = this.SelectedAAS.endpoints[0].address + '/' + this.SelectedNode.path + '/value';
            let content = "'" + this.newPathValue + "'";
            let headers = { 'Content-Type': 'application/json' };
            let context = 'updating ' + this.fileObject.modelType.name + ' "' + this.fileObject.idShort + '"';
            let disableMessage = false;
            // Send Request to update the path of the file element
            this.putRequest(path, content, headers, context, disableMessage).then((response: any) => {
                if (response.success) {
                    this.$emit('updatePath'); // emit event to update the path in the parent component
                }
            });
        },

        // Function to clear the Path of the File Element
        clearPath() {
            this.newPathValue = '';
        },

        // Function to upload a File
        uploadFile() {
            // console.log("Upload File: ", this.newFile);
            // check if a file is selected
            if (this.newFile.length == 0) return;
            
            let context = 'uploading ' + this.fileObject.modelType.name + '-SubmodelElement' + ' "' + this.fileObject.idShort + '"';
            let disableMessage = false;
            let path = this.SelectedAAS.endpoints[0].address + '/' + this.SelectedNode.path + '/upload';
            var headers = new Headers();
            var formData = new FormData();
            formData.append("file", this.newFile[0]);
            this.loading = true;
            // Send Request to upload the file
            this.postRequest(path, formData, headers, context, disableMessage).then((response: any) => {
                this.loading = false;
                // TODO: Add success response handling including updating the File SubmodelElement -> mimeType + value
                if (response.success) {
                    console.log(response);
                }
            });
        },

        // Function to set the focus on the input field
        setFocus(e: boolean) {
            this.isFocused = e;
            if (!e) this.newPathValue = this.fileObject.value; // set input to current value in the AAS if the input field is not focused
        },

        // Function to prepare the Image Link for the Image Preview
        getLocalPath(path: string): string {
            if(!path) return '';
            // check if Link starts with '/'
            if (path.startsWith('/')) {
                path = this.SelectedAAS.endpoints[0].address.replace('/aas', '') + '/files' + path;
            }
            return path;
        }
    },
});
</script>
