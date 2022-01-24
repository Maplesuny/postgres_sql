<!--onRequest寫法 -->
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
import { ref, onMounted } from 'vue'
import option from "../javascript/Setoption";
import { api } from 'boot/axios'
import { useQuasar } from 'quasar'

const columns = [
    {
        name: 'desc',
        required: true,
        label: 'Dessert (100g serving)',
        align: 'left',
        field: row => row.name,
        format: val => `${val}`,
        sortable: true
    },
    { name: 'calories', align: 'center', label: 'Calories', field: 'calories', sortable: true },
    { name: 'fat', label: 'Fat (g)', field: 'fat', sortable: true },
    { name: 'carbs', label: 'Carbs (g)', field: 'carbs', sortable: true },
    { name: 'protein', label: 'Protein (g)', field: 'protein', sortable: true },
    { name: 'sodium', label: 'Sodium (mg)', field: 'sodium', sortable: true },
    { name: 'calcium', label: 'Calcium (%)', field: 'calcium', sortable: true, sort: (a, b) => parseInt(a, 10) - parseInt(b, 10) },
    { name: 'iron', label: 'Iron (%)', field: 'iron', sortable: true, sort: (a, b) => parseInt(a, 10) - parseInt(b, 10) }
]

const originalRows = [
    { id: 1, name: 'Frozen Yogurt', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
    { id: 2, name: 'Ice cream sandwich', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
    { id: 3, name: 'Eclair', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
    { id: 4, name: 'Cupcake', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
    { id: 5, name: 'Gingerbread', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
    { id: 6, name: 'Jelly bean', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
    { id: 7, name: 'Lollipop', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
    { id: 8, name: 'Honeycomb', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
    { id: 9, name: 'Donut', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
    { id: 10, name: 'KitKat', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' },
    { id: 11, name: 'Frozen Yogurt-1', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
    { id: 12, name: 'Ice cream sandwich-1', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
    { id: 13, name: 'Eclair-1', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
    { id: 14, name: 'Cupcake-1', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
    { id: 15, name: 'Gingerbread-1', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
    { id: 16, name: 'Jelly bean-1', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
    { id: 17, name: 'Lollipop-1', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
    { id: 18, name: 'Honeycomb-1', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
    { id: 19, name: 'Donut-1', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
    { id: 20, name: 'KitKat-1', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' },
    { id: 21, name: 'Frozen Yogurt-2', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
    { id: 22, name: 'Ice cream sandwich-2', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
    { id: 23, name: 'Eclair-2', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
    { id: 24, name: 'Cupcake-2', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
    { id: 25, name: 'Gingerbread-2', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
    { id: 26, name: 'Jelly bean-2', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
    { id: 27, name: 'Lollipop-2', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
    { id: 28, name: 'Honeycomb-2', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
    { id: 29, name: 'Donut-2', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
    { id: 30, name: 'KitKat-2', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' },
    { id: 31, name: 'Frozen Yogurt-3', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
    { id: 32, name: 'Ice cream sandwich-3', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
    { id: 33, name: 'Eclair-3', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
    { id: 34, name: 'Cupcake-3', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
    { id: 35, name: 'Gingerbread-3', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
    { id: 36, name: 'Jelly bean-3', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
    { id: 37, name: 'Lollipop-3', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
    { id: 38, name: 'Honeycomb-3', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
    { id: 39, name: 'Donut-3', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
    { id: 40, name: 'KitKat-3', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' }
]

export default {
    setup () {
        //-----for Loading timer------
        const $q = useQuasar()
        let host = "127.0.0.1";
        let port = "8000";
        let timer
        const model = ref('')
        const table_name = ref([])
        let dd = ref('BedRecordDim')
        const table_name_url = 'http://127.0.0.1:8000/api/v1/gettable_column?table=PhysicalFactMt'
        const tables_url =
            "http://" + host + ":" + port + "/api/v1/gettablenames?database=table";

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

        // 資料
        const db_data = []
        const data_rul = 'http://127.0.0.1:8000/api/v1/gettable_data/PhysicalFactMt'

        function get_data (data_len) {
            api.get(data_rul).then((res) => {
                data_len = res.data.length
                console.log('dfd123', data_len)
                for (let i = 0; i < res.data.length; i++) {
                    db_data.push(res.data[i])
                }
            })
        }

        console.log('DB_data', db_data)
        // get_data()

        const rows = ref([])
        const filter = ref('')
        const loading = ref(false)
        const pagination = ref({
            sortBy: 'desc',
            descending: false,
            page: 1,
            rowsPerPage: 100,
            rowsNumber: 100
        })


        // emulate ajax call
        // SELECT * FROM ... WHERE...LIMIT...
        function fetchFromServer (startRow, count, filter, sortBy, descending) {
            const data = filter
                ? db_data.filter(row => row.PatientNo.includes(filter))
                : db_data.slice()

            // handle sortBy
            if (sortBy) {
                const sortFn = sortBy === 'desc'
                    ? (descending
                        ? (a, b) => (a.name > b.name ? -1 : a.name < b.name ? 1 : 0)
                        : (a, b) => (a.name > b.name ? 1 : a.name < b.name ? -1 : 0)
                    )
                    : (descending
                        ? (a, b) => (parseFloat(b[sortBy]) - parseFloat(a[sortBy]))
                        : (a, b) => (parseFloat(a[sortBy]) - parseFloat(b[sortBy]))
                    )
                data.sort(sortFn)
            }

            return data.slice(startRow, startRow + count)
        }

        // emulate 'SELECT count(*) FROM ...WHERE...'
        function getRowsNumberCount (filter) {
            if (!filter) {
                // 要擺總count
                return db_data.length
            }
            let count = 0
            // debugger
            db_data.forEach(treat => {
                console.log('sdfdf', treat)
                console.log('111', treat.PatientNo)
                if (treat.PatientNo.includes(filter)) {
                    ++count
                }
                console.log('count+1', count)
            })

            return count
        }

        function onRequest (props) {
            const { page, rowsPerPage, sortBy, descending } = props.pagination
            const filter = props.filter
            console.log('filter', filter)

            loading.value = true

            // emulate server

            // update rowsCount with appropriate value
            pagination.value.rowsNumber = getRowsNumberCount(filter)

            // get all rows if "All" (0) is selected
            const fetchCount = rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage

            // calculate starting row of data
            const startRow = (page - 1) * rowsPerPage

            // fetch data from "server"
            const returnedData = fetchFromServer(startRow, fetchCount, filter, sortBy, descending)

            // clear out existing data and add new
            rows.value.splice(0, rows.value.length, ...returnedData)

            // don't forget to update local pagination object
            pagination.value.page = page
            pagination.value.rowsPerPage = rowsPerPage
            pagination.value.sortBy = sortBy
            pagination.value.descending = descending

            // ...and turn of loading indicator
            loading.value = false

        }

        // Loading 
        function showLoading () {
            $q.loading.show({
                message: 'Get Data information. Please wait...',
                boxClass: 'bg-grey-2 text-grey-9',
                spinnerColor: 'primary'
            })
            // hiding in 3s
            timer = setTimeout(() => {
                $q.loading.hide()
                timer = void 0
            }, 5000)
        }

        showLoading()

        onMounted(() => {
            let data_len
            get_table_names()
            get_data(data_len)
            console.log('資料長度', data_len)
            console.log('ooo', dd.value)
            // get initial data from server (1st page)
            setTimeout(() => {
                onRequest({
                    pagination: pagination.value,
                    filter: undefined
                })
            }, 5000)

        })

        return {
            model,


            filter,
            loading,
            pagination,
            columns, table_name,
            rows,
            onRequest
        }
    }
}
</script>