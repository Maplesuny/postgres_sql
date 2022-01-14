<template>
    <div class="q-pa-md" style="max-width: 300px">
        <div class="q-gutter-md">
            <q-select
                filled
                v-model="model"
                use-input
                input-debounce="0"
                label="Simple filter"
                :options="options"
                @filter="filterFn"
                style="width: 250px"
                behavior="menu"
            >
                <template v-slot:no-option>
                    <q-item>
                        <q-item-section class="text-grey">No results</q-item-section>
                    </q-item>
                </template>
            </q-select>
        </div>
    </div>
    <div class="q-pa-md">
        <q-table
            title="Treats"
            :rows="rows"
            :columns="table_name"
            row-key="id"
            v-model:pagination="pagination"
            :loading="loading"
            :filter="filter"
            @request="onRequest"
            binary-state-sort
        >
            <template v-slot:top-right>
                <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
                    <template v-slot:append>
                        <q-icon name="search" />
                    </template>
                </q-input>
            </template>
        </q-table>
    </div>
</template>

<script>
import axios from 'axios'
import { api } from 'boot/axios'
import option from "../javascript/Setoption";
import { exportFile, useQuasar } from 'quasar'
import { ref, onMounted, watch, onBeforeUnmount, onBeforeMount, reactive } from "vue";

const table_name = ref([])
const table_name_url = 'http://127.0.0.1:8000/api/v1/gettable_column?table=AllergyRecordDim'
// Table名稱
function get_table_names () {
    api.get(table_name_url).then((res) => {
        for (let i = 0; i < res.data.length; i++) {
            table_name.value.push({
                name: res.data[i],
                required: true,
                label: res.data[i],
                align: 'center',
                field: res.data[i],
                sortable: true
            })
        }
    }).catch(() => {
        alert('dfdf')
    })
}


const db_data = []
const data_rul = 'http://10.65.51.164:1801/api/v1/gettable_data/AllergyRecordDim?database=table'
function get_data () {
    api.get(data_rul).then((res) => {
        console.log(res.data.length)
        for (let i = 0; i < res.data.length; i++) {
            db_data.push(res.data[i])
        }
    })
}


export default {
    setup () {
        const model = ref('')
        const rows = ref([])
        const filter = ref('')
        const loading = ref(false)
        const stringOptions = []
        const options = ref(stringOptions)
        const pagination = ref({
            sortBy: 'desc',
            descending: false,
            page: 1,
            rowsPerPage: 5,
            rowsNumber: 10
        })
        const new_stringOptions = []
        const all_tables_url = 'http://127.0.0.1:8000/api/v1/gettablenames'


        const { axioss } = option()

        // emulate ajax call
        // SELECT * FROM ... WHERE...LIMIT...
        function fetchFromServer (startRow, count, filter, sortBy, descending) {
            console.log('fetchFromServer內的fliter', filter)
            console.log('fetchFromServer內的sortBy', sortBy)
            const data = filter
                ? db_data.filter(row => row.name.includes(filter))
                : db_data.slice()

            // handle sortBy   a.PatientNo  b.PatientNo
            if (sortBy) {
                const sortFn = sortBy === 'desc'
                    ? (descending
                        ? (a, b) => (a.PatientNo > b.PatientNo ? -1 : a.PatientNo < b.PatientNo ? 1 : 0)
                        : (a, b) => (a.PatientNo > b.PatientNo ? 1 : a.PatientNo < b.PatientNo ? -1 : 0)
                    )
                    : (descending
                        ? (a, b) => (parseFloat(b[sortBy]) - parseFloat(a[sortBy]))
                        : (a, b) => (parseFloat(a[sortBy]) - parseFloat(b[sortBy]))
                    )
                data.sort(sortFn)
            }

            console.log('結果', data.slice(startRow, startRow + count))

            return data.slice(startRow, startRow + count)
        }

        // emulate 'SELECT count(*) FROM ...WHERE...'
        function getRowsNumberCount (filter) {
            console.log('filter', filter)
            if (!filter) {
                return db_data.length
            }
            let count = 2
            return count
        }


        async function filterFn (val, update) {
            let new_stringOptions = await axioss(all_tables_url)
            console.log('new_stringOptions', new_stringOptions)
            console.log('val', val, update)
            if (val === '') {
                update(() => {
                    options.value = new_stringOptions
                })
                return
            }
            debugger;
            update(() => {
                const needle = val.toLowerCase()
                console.log('needle', needle)
                options.value = new_stringOptions.filter(v => v.toLowerCase().indexOf(needle) > -1)
            })
        }

        function onRequest (props) {
            const { page, rowsPerPage, sortBy, descending } = props.pagination
            const filter = props.filter

            loading.value = true

            // emulate server
            setTimeout(() => {
                // update rowsCount with appropriate value
                pagination.value.rowsNumber = getRowsNumberCount(filter)
                console.log(' pagination.value.rowsNumber ', pagination.value.rowsNumber)

                // get all rows if "All" (0) is selected
                const fetchCount = rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage
                console.log('fetchCount', fetchCount)

                // calculate starting row of data
                const startRow = (page - 1) * rowsPerPage

                // fetch data from "server"
                const returnedData = fetchFromServer(startRow, fetchCount, filter, sortBy, descending)
                console.log('returnedData', returnedData)

                // clear out existing data and add new
                rows.value.splice(0, rows.value.length, ...returnedData)

                // don't forget to update local pagination object
                pagination.value.page = page
                pagination.value.rowsPerPage = rowsPerPage
                pagination.value.sortBy = sortBy
                pagination.value.descending = descending

                // ...and turn of loading indicator
                loading.value = false
            }, 1500)
        }

        onMounted(() => {
            get_table_names()
            get_data()
            onRequest({
                pagination: pagination.value,
            })
        })

        return {
            filter,
            loading,
            pagination,
            model,
            table_name, rows,
            options,
            onRequest, filterFn

        }
    }
}
</script>