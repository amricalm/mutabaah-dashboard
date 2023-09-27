<template>
        <div class="container">
            <div class="card mb-4">
                <div class="card-header">
                    <h6 class="mb-0">Input Nilai</h6>
                </div>
                <div class="card-body">
                    <form @submit.prevent="store">
                        <div class="form-group">
                            <label>Tanggal</label>
                            <input type="date" class="form-control is-valid" v-model="task.Tanggal">
                            <!-- <p class="form-text valid-feedback">Please enter correct password</p> -->
                        </div>
                        <div class="form-group">
                            <label>Kelompok</label>
                            <select v-model="task.Kelas" @change="kelasSelected" class="form-control is-valid">
                                <option v-for="(kelas, index) in listKelas" :key="index" v-text="kelas.kelas"></option>
                            </select>
                        </div>
                        <div class="form-group">
                            <label>Siswa</label>
                            <select v-model="task.Siswa" @change="siswaSelected" class="form-control is-valid">
                                <option v-for="(siswa, index) in listSiswa" :key="index" :value="siswa.nis" v-text="siswa.nama_lengkap"></option>
                            </select>
                        </div>
                        <div class="form-group">
                            <label>Halaman (<i>Halaman Terakhir {{ capaianSiswa.halaman ?? 0 }}</i>)</label>
                            <input type="text" class="form-control is-valid" v-model="task.Halaman" v-text="capaianSiswa.halaman">
                        </div>
                        <div class="form-group">
                            <label>Baris (<i>Baris Terakhir {{ capaianSiswa.baris ?? 0 }}</i>)</label>
                            <input type="text" class="form-control is-valid" v-model="task.Baris" v-text="capaianSiswa.baris">
                        </div>
                        <div class="row">
                            <div class="col">
                                <!-- <div class="alert alert-success"><b>Congratulations!</b> Your address has been saved to your adresss list successfully.</div> -->
                            </div>
                            <div class="col-auto">
                                <button type="submit" class="btn btn-primary align-right">SIMPAN</button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
</template>

<script>
import { reactive } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import { onMounted, ref } from 'vue'

axios.defaults.baseURL = 'http://mutabaah-api.test/api'

export default {
    methods: {
        kelasSelected() {
            let listSiswa = ref([])
            axios.get('/task',{ //get API from Laravel Backend
                params : {
                    Kelas : this.task.Kelas
                }
            })
            .then(response => {
                this.listSiswa = response.data.data.siswa //response from backend
                console.log(response.data.data.siswa)
            }).catch(error => {
                console.log(error.response.listSiswa)
            })
            return {
                listSiswa
            }
        },
        siswaSelected() {
            let capaianSiswa = ref([])
            axios.get('/task',{ //get API from Laravel Backend
                params : {
                    Kelas : this.task.Kelas,
                    Siswa : this.task.Siswa
                }
            })
            .then(response => {
                this.capaianSiswa = response.data.data.capaian //response from backend
                console.log(response.data.data.capaian)
            }).catch(error => {
                console.log(error.response.data.data.capaian)
            })
            return {
                capaianSiswa
            }
        },
    },
    setup() {
        const router = useRouter()
        const task = reactive({
            Tanggal: '',
            Kelas: '',
            Siswa: '',
            Halaman: '',
            Baris: ''
        })
        //reactive state
        let listKelas = ref([])
        let listSiswa = ref([])
        let capaianSiswa = ref([])

        //mounted
        onMounted(() => {
            axios.get('/kelas') //get API dari Laravel Backend
            .then(response => {
              listKelas.value = response.data.data //Validasi dari data respons
            }).catch(error => {
                console.log(error.response.data)
            })
        })

        //method store
        function store() {
            let Kelas   = task.Kelas
            let Siswa   = task.Siswa
            let Tanggal = task.Tanggal
            let Halaman = task.Halaman
            let Baris = task.Baris

            axios.post('/task', {
                Tanggal: Tanggal,
                Kelas: Kelas,
                Siswa: Siswa,
                Halaman: Halaman,
                Baris: Baris
            }).then(() => {
                router.push({
                    name: 'task.index'
                })
            }).catch(error => {
                listKelas.value = error.response.data
            })
        }

        //return
        return {
            task,
            listKelas,
            listSiswa,
            capaianSiswa,
            router,
            store
        }
    }
}
</script>