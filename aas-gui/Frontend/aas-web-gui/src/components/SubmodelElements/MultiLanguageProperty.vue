<template>
    <v-container fluid class="pa-0">
        <v-card color="elevatedCard" v-if="multiLanguagePropertyObject" class="mt-4">
            <!-- Value(s) of the MultiLanguageProperty -->
            <v-list nav class="bg-elevatedCard pt-0" v-if="multiLanguagePropertyObject.value && multiLanguagePropertyObject.value.length > 0">
                <v-list-item v-for="(value, i) in mlpValue" :key="i">
                    <v-list-item-title class="pt-2">
                        <!-- Input Field containing the Variable Value -->
                        <v-text-field v-model="value.text" variant="outlined" density="compact" hide-details clearable append-icon="mdi-delete" @click:append="removeEntry(i)" @update:focused="setFocus($event, value)" @keydown.native.enter="updateValue()">
                            <template v-slot:prepend-inner>
                                <!-- language -->
                                <v-chip label size="x-small" border style="margin-top: 2px">
                                    <span>{{ value.language ? value.language : 'no-lang' }}</span>
                                    <v-icon site="x-small" style="margin-right: -3px">mdi-chevron-down</v-icon>
                                    <!-- Menu to select the Language -->
                                    <v-menu activator="parent">
                                        <v-list density="compact" class="pa-0">
                                            <v-list-item v-for="language in languages" :key="language.id" @click="selectLanguage(language,  value)">
                                                <v-list-item-title class="py-0">{{ language.short }}</v-list-item-title>
                                            </v-list-item>
                                        </v-list>
                                    </v-menu>
                                </v-chip>
                            </template>
                            <!-- Update Value Button -->
                            <template v-slot:append-inner>
                                <v-btn v-if="value.isFocused" size="small" variant="elevated" color="primary" class="text-buttonText" style="margin-top: -2.5px; right: -6px" @click.stop="updateValue()">
                                    <v-icon>mdi-upload</v-icon>
                                </v-btn>
                            </template>
                        </v-text-field>
                    </v-list-item-title>
                </v-list-item>
            </v-list>
            <!-- Warning when MultiLanguageProperty has no value(s) -->
            <v-list nav class="bg-elevatedCard pt-0" v-else>
                <v-list-item>
                    <v-list-item-title class="pt-2">
                        <v-alert text="MultiLanguageProperty doesn't contain a Value!" density="compact" type="warning" variant="outlined"></v-alert>
                    </v-list-item-title>
                </v-list-item>
            </v-list>
            <v-divider></v-divider>
            <!-- Edit the MultiLanguageProperty -->
            <v-list nav class="bg-elevatedCard py-0">
                <v-list-item>
                    <template v-slot:append>
                        <v-btn color="primary" size="small" variant="elevated" class="text-buttonText" @click="addEntry()">
                            <div>Add new Entry</div>
                            <v-icon class="ml-1">mdi-plus</v-icon>
                        </v-btn>
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
    name: 'MultiLanguageProperty',
    components: {
        RequestHandling, // Mixin to handle the requests to the AAS
    },
    mixins: [RequestHandling],
    props: ['multiLanguagePropertyObject'],

    setup() {
        const store = useStore()

        return {
            store, // Store Object
        }
    },

    data() {
        return {
            mlpValue: {} as any,
            languages: [
                { id: 1, text: 'Deutsch', short: 'de' },
                { id: 1, text: 'English', short: 'en' },
                { id: 1, text: 'Français', short: 'fr' },
                { id: 1, text: 'Español', short: 'es' },
                { id: 1, text: 'Italiano', short: 'it' },
                { id: 1, text: 'Kanton Zürich', short: 'zh' },
                { id: 1, text: '한국인', short: 'kr' },
            ] as any,
        }
    },

    mounted() {
        this.mlpValue = this.multiLanguagePropertyObject.value;
    },

    watch: {
        // Watch for changes in the selected Node and reset input
        SelectedNode: {
            deep: true,
            handler() {
                this.mlpValue = {};
            }
        },

        // Watch for changes in the propertyObject and update the newPropertyValue if the input field is not focused
        multiLanguagePropertyObject: {
            deep: true,
            handler() {
                this.mlpValue = this.multiLanguagePropertyObject.value;
            }
        },
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
        // Function to remove an Entry from the MultiLanguageProperty
        removeEntry(position: number) {
            // console.log('removeEntry: ', value);
            this.mlpValue.splice(position, 1);
            this.multiLanguagePropertyObject.value = this.mlpValue;
            this.updateMLP();
        },

        // Function to add an Entry to the MultiLanguageProperty
        addEntry() {
            this.mlpValue.push({
                language: '',
                text: '',
            });
            this.multiLanguagePropertyObject.value = this.mlpValue;
            // console.log('addEntry: ', this.multiLanguagePropertyObject)
            this.updateMLP();
        },

        // Function to select the Language of the Entry
        selectLanguage(language: any, value: any) {
            // console.log('selectLanguage: ', language);
            value.language = language.short;
            this.updateMLP();
        },

        // Function to update the Value of the MultiLanguageProperty
        updateValue() {
            // console.log('updateValue: ', this.mlpValue);
            if (document.activeElement) (document.activeElement as HTMLElement).blur(); // remove focus from input field
            this.multiLanguagePropertyObject.value = this.mlpValue;
            this.updateMLP();
        },

        // Function to update the value of the property
        updateMLP() {
            // console.log("Update Value: ", this.multiLanguagePropertyObject);
            let path = this.SelectedAAS.endpoints[0].address + '/' + this.multiLanguagePropertyObject.path;
            let content = JSON.stringify(this.multiLanguagePropertyObject);
            let headers = { 'Content-Type': 'application/json' };
                let context = 'updating ' + this.multiLanguagePropertyObject.modelType.name + ' "' + this.multiLanguagePropertyObject.idShort + '"';
            let disableMessage = false;
            // Send Request to update the value of the property
            this.putRequest(path, content, headers, context, disableMessage).then((response: any) => {
                if (response.success) {
                    // // this.newPropertyValue = ''; // reset input
                    // let updatedPropertyObject = { ...this.propertyObject }; // copy the propertyObject
                    // updatedPropertyObject.value = content.toString().replace(/'/g, ''); // update the value of the propertyObject
                    // this.$emit('updateValue', updatedPropertyObject); // emit event to update the value in the parent component
                }
            });
    },

        // Function to set the focus on the input field
        setFocus(e: boolean, value: any) {
            value.isFocused = e;
        },
    },
});
</script>
