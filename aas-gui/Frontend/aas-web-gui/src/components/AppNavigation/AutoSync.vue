<template>
    <v-btn class="mr-6" variant="outlined">
        <span class="mr-1">{{ 'Auto Sync:' }}</span>
        <span class="text-primary">{{ (autoSyncStatus ? 'On' : 'Off') }}</span>
        <v-icon :style="{ 'margin-left': autoSyncStatus ? '12.5px' : '6px' }">mdi-chevron-down</v-icon>
        <v-menu activator="parent" open-on-hover :close-on-content-click="false" width="300px">
            <v-list nav class="py-0 bg-elevatedCard">
                <!-- Switch to activate/deactive auto-sync -->
                <v-list-item class="py-0">
                    <v-switch label="Auto Sync" color="primary" class="mx-3" hide-details v-model="autoSyncStatus" @change="updateAutoSync()">
                        <template v-slot:append>
                            <v-icon>mdi-autorenew</v-icon>
                        </template>
                    </v-switch>
                </v-list-item>
                <!-- Hint -->
                <v-list-item class="py-0" style="margin-top: -10px">
                    <v-list-item-subtitle class="ml-1">{{ 'Only SubmodelElements can be auto-synced.' }}</v-list-item-subtitle>
                    <v-list-item-subtitle class="ml-1">{{ 'Submodels are synced on selection.' }}</v-list-item-subtitle>
                </v-list-item>
                <!-- Input Field to set the sync-interval -->
                <v-list-item class="py-0">
                    <v-text-field density="compact" variant="outlined" type="number" v-model="intervalTime" hide-details @update:focused="checkMin">
                        <template v-slot:append-inner>
                            <span>ms</span>
                        </template>
                    </v-text-field>
                </v-list-item>
                <!-- Hint -->
                <v-list-item class="py-0" style="margin-top: -10px">
                    <v-list-item-subtitle class="ml-1">{{ 'Be careful decreasing the time!' }}</v-list-item-subtitle>
                    <v-list-item-subtitle class="ml-1">{{ 'A smaller time means more requests.' }}</v-list-item-subtitle>
                </v-list-item>
            </v-list>
        </v-menu>
    </v-btn>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue';
import { useStore } from 'vuex';

export default defineComponent({
    name: 'AutoSync',
    props: ['submodelObject'],

    setup() {
        const store = useStore()

        return {
            store, // Store Object
        }
    },

    data() {
        return {
            autoSyncStatus: false, // Status of the auto-sync (true = active, false = inactive)
            intervalTime: 1000, // Interval time for the auto-sync
        }
    },

    mounted() {
    },

    watch: {
    },

    computed: {
    },

    methods: {
        // Checks if the input is smaller than 100ms and sets it to 100ms if it is
        checkMin(e: boolean) {
            if (this.intervalTime < 100 && !e) this.intervalTime = 100;
            if (!e) this.updateAutoSync();
        },

        // Updates the auto-sync (+ interval) in the store
        updateAutoSync() {
            this.store.dispatch('dispatchUpdateAutoSync', { state: this.autoSyncStatus, interval: this.intervalTime });
        },
    },
});
</script>
