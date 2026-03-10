<template>
    <v-container max-width="1100" class="pa-8">

        <!-- 헤더 -->
        <div class="d-flex align-center mb-6">
            <h1 class="font-weight-bold">유저 목록</h1>
        </div>

        <!-- 검색 -->
        <div class="d-flex align-center justify-space-between mb-4">
            <v-text-field variant="outlined" density="compact" placeholder="이름 검색" prepend-inner-icon="mdi-magnify"
                hide-details style="max-width: 300px;" v-model="search" />
            <v-btn color="primary" prepend-icon="mdi-plus" @click="openDialog()">유저 추가</v-btn>
        </div>

        <v-card variant="outlined">
            <!-- 
                :headers="headers" : 테이블 컬럼 정의
                :items="worlds" : 테이블 데이터
                :loading="loading": 로딩 스피너
                :search="search"  : 검색 필터링 (Vue가 알아서 처리)
                @click:row="goToUserInfo" : 행 클릭 시 실행
            -->
            <v-data-table :headers="headers" :items="worlds" :loading="loading" :search="search"
                @click:row="goToUserInfo" style="cursor: pointer;">
                <template v-slot:item.actions="{ item }">
                    <v-btn size="small" variant="tonal" color="primary" class="mr-1"
                        @click.stop="openEditDialog(item)">수정</v-btn>
                    <v-btn size="small" variant="tonal" color="error" @click.stop="deleteWorld(item.id)">삭제</v-btn>
                </template>
            </v-data-table>
        </v-card>

    </v-container>

    <!-- 생성/수정 Dialog -->
    <v-dialog v-model="dialog" max-width="600">
        <v-form @submit.prevent="handleSubmit">
            <v-card prepend-icon="mdi-account" :title="isEdit ? '유저 수정' : '유저 생성'">
                <v-card-text>
                    <v-text-field label="이름" v-model="form.name" placeholder="이름을 입력하세요." density="comfortable" />
                    <v-text-field label="전화번호" v-model="form.phoneNumber" placeholder="전화번호를 입력하세요."
                        density="comfortable" />
                    <v-text-field label="이메일" v-model="form.email" placeholder="이메일을 입력하세요." density="comfortable" />
                    <v-text-field label="주소" v-model="form.address" placeholder="주소를 입력하세요." density="comfortable" />
                </v-card-text>
                <v-card-actions class="justify-end">
                    <v-btn color="primary" type="submit">{{ isEdit ? '유저 수정' : '유저 생성' }}</v-btn>
                    <v-btn color="error" @click="closeDialog">취소</v-btn>
                </v-card-actions>
            </v-card>
        </v-form>
    </v-dialog>

    <!-- 삭제 확인 Dialog -->
    <v-dialog v-model="deleteDialog" max-width="400">
        <v-card>
            <v-card-title>삭제 확인</v-card-title>
            <v-card-text>정말로 삭제하시겠습니까?</v-card-text>
            <v-card-actions class="justify-end">
                <v-btn color="error" @click="confirmDelete">삭제</v-btn>
                <v-btn color="primary" @click="deleteDialog = false">취소</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import axios from 'axios';
export default {
    name: 'list',
    data() {
        return {
            worlds: [],
            loading: true,
            search: '',
            dialog: false,
            isEdit: false,
            deleteDialog: false,
            deleteTargetId: null,
            headers: [
                { title: '번호', key: 'id' },
                { title: '이름', key: 'name' },
                { title: '전화번호', key: 'phoneNumber' },
                { title: '관리', key: 'actions', sortable: false },
            ],
            form: {
                id: null,
                name: '',
                phoneNumber: '',
                email: '',
                address: '',
            }
        }
    },

    // 컴포넌트 마운트 시 데이터 불러오기
    mounted() {
        this.loadItems()
    },

    methods: {
        // User 정보 화면으로 이동
        goToUserInfo(event, { item }) {
            this.$router.push(`/worldInfo/${item.id}`)
        },

        // World 생성 Dialog 열기
        openDialog() {
            this.isEdit = false
            this.form = { id: null, name: '', phoneNumber: '', email: '', address: '' }
            this.dialog = true
        },

        // World 수정 Dialog 열기 - API로 최신 데이터 가져오기
        async openEditDialog(world) {
            this.isEdit = true
            try {
                const response = await axios.post(`http://localhost:8080/api/worlds/${world.id}`)
                this.form = {
                    id: response.data.id,
                    name: response.data.name,
                    phoneNumber: response.data.phoneNumber,
                    email: response.data.email,
                    address: response.data.address,
                }
            } catch (error) {
                console.error("유저 정보를 가져올 수 없습니다.", error)
            }
            this.dialog = true
        },

        // Dialog 닫기
        closeDialog() {
            this.dialog = false
        },

        // World 목록 가져오기 - 전체 데이터 한 번에 가져오기
        async loadItems() {
            this.loading = true
            try {
                const response = await axios.post("http://localhost:8080/api/worlds/search", {
                    keyword: `%%`,
                })
                this.worlds = response.data._embedded.worlds
            } catch (error) {
                console.error("유저 정보를 가져올 수 없습니다.", error)
            } finally {
                this.loading = false
            }
        },

        // World 핸들링 함수
        async handleSubmit() {
            try {
                if (this.isEdit) {
                    await axios.put(`http://localhost:8080/api/worlds/${this.form.id}`, {
                        name: this.form.name,
                        phoneNumber: this.form.phoneNumber,
                        email: this.form.email,
                        address: this.form.address,
                    })
                } else {
                    await axios.post("http://localhost:8080/api/worlds", {
                        name: this.form.name,
                        phoneNumber: this.form.phoneNumber,
                        email: this.form.email,
                        address: this.form.address,
                    })
                }
                this.loadItems()
                this.closeDialog()
            } catch (error) {
                console.error("저장 실패", error)
            }
        },

        // 삭제 Dialog 열기
        deleteWorld(id) {
            this.deleteTargetId = id
            this.deleteDialog = true
        },

        // 삭제 확인
        async confirmDelete() {
            try {
                await axios.delete(`http://localhost:8080/api/worlds/${this.deleteTargetId}`)
                this.loadItems()
                this.deleteDialog = false
            } catch (error) {
                console.error("삭제 실패", error)
            }
        }
    }
}
</script>

<style src="@/styles/main.css"></style>