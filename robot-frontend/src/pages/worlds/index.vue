<template>
    <v-container max-width="1100" class="pa-8">

        <div class="d-flex justify-space-between ma-8">
            <h1 class="font-weight-bold">월드 목록</h1>
        </div>

        <div class="d-flex justify-space-between ma-4">
            <!--
                hide-details       : 에러/힌트 메시지 공간 제거
                density="compact"  : 높이 작게, 여백 최소
            -->
            <v-text-field v-model="name" placeholder="이름으로 검색" density="compact" prepend-icon="mdi-account"
                max-width="300px" variant="outlined" hide-details>
            </v-text-field>
            <v-btn color="primary" prepend-icon="mdi-plus" @click="openDialog">월드 생성</v-btn>
        </div>

        <v-card variant="outlined">

            <!-- 
                items : 테이블에 표시할 데이터 배열
                loading : true면 로딩 스피너 표시
                headers : 컬럼 정의 배열(번호, 이름, 이메일 등등)
                @click:row : 행 클릭 시 함수 호출
                style : 스타일
                hover : 행에 마우스 올리면 배경색 변경
                items-length : 서버의 전체 데이터 개수(페이지 네이션 개산에 사용)
                v-model:page="page" : 페이지 변경 시 page 변수에 양방향 바인딩
                v-model:items-per-page="itemsPerPage" : 페이지당 항목 수 변경 시 양방향 바인딩
                @update:options : 페이지/정렬/검색 변경 시 fetchWorlds 호출
                must-sort : 정렬 없음 단계 제거(asc <=> desc 2단계만 순환)
            
            -->

            <v-data-table-server :items="worlds" :loading="loading" :headers="headers" @click:row="goToWorldInfo"
                style="cursor: pointer;" hover :items-length="totalItems" :page="page" :items-per-page="itemsPerPage"
                v-model:page="page" v-model:items-per-page="itemsPerPage" @update:options="fetchWorlds" must-sort>

                <!-- 커스텀 렌더링 -->
                <template v-slot:item.actions="{ item }">
                    <v-btn color="primary" size="small" variant="tonal" @click.stop="openEditDialog(item)"
                        class="mr-1">수정</v-btn>
                    <v-btn color="error" size="small" variant="tonal" @click.stop="openDeleteDialog(item.id)"
                        class="mr-1">삭제</v-btn>
                </template>

                <!-- 테이블 하단 영역 커스텀-->
                <template v-slot:bottom>
                    <div class="d-flex align-center justify-center pa-4" style="gap: 16px;">
                        <!-- page가 1보다 작으면 비활성화 / 1페이지로 이동-->
                        <v-btn icon density="comfortable" variant="text" :disabled="page <= 1" @click="page = 1">
                            <v-icon>mdi-page-first</v-icon>
                        </v-btn>
                        <!-- page가 1보다 작으면 비활성화 / 1페이지 전으로 이동-->
                        <v-btn icon density="comfortable" variant="text" :disabled="page <= 1" @click="page--">
                            <v-icon>mdi-chevron-left</v-icon>
                        </v-btn>

                        <!-- 현재 페이지 -->
                        <span class="text-body-1">{{ page }} / {{ Math.ceil(totalItems / itemsPerPage) }}</span>

                        <!-- 현재 페이지가 마지막 페이지 일 경우 비활성화 / 1페이지 뒤로 이동 -->
                        <!-- Math.ceil : 소수점 올림 함수-->
                        <v-btn icon density="comfortable" variant="text"
                            :disabled="page >= Math.ceil(totalItems / itemsPerPage)" @click="page++">
                            <v-icon>mdi-chevron-right</v-icon>
                        </v-btn>
                        <!-- 현재 페이지가 마지막 페이지 일 경우 비활성화 / 맨 뒤 페이지로 이동 -->
                        <v-btn icon density="comfortable" variant="text"
                            :disabled="page >= Math.ceil(totalItems / itemsPerPage)"
                            @click="page = Math.ceil(totalItems / itemsPerPage)">
                            <v-icon>mdi-page-last</v-icon>
                        </v-btn>

                        <!-- items: 드롭다운에 표시할 데이터 목록 -->
                        <span class="text-body-2 text-grey">페이지당:</span>
                        <v-select v-model="itemsPerPage" :items="[
                            { title: '5', value: 5 },
                            { title: '10', value: 10 },
                            { title: '15', value: 15 },
                            { title: '20', value: 20 },
                        ]" density="compact" variant="outlined" hide-details style="max-width: 100px;" />
                    </div>
                </template>
            </v-data-table-server>
        </v-card>

    </v-container>

    <!-- 생성 / 수정 다이얼로그 -->
    <v-dialog v-model="dialog" max-width="600px">
        <v-form @submit.prevent="handleSubmit">
            <v-card prepend-icon="mdi-account" :title="isEdit ? '월드 수정' : '월드 생성'">
                <v-card-text>
                    <v-text-field label="이름" v-model="form.name" placeholder="이름을 입력하세요."
                        density="comfortable"></v-text-field>
                    <v-text-field label="전화번호" v-model="form.phoneNumber" placeholder="전화번호를 입력하세요."
                        density="comfortable"></v-text-field>
                    <v-text-field label="이메일" v-model="form.email" placeholder="이메일을 입력하세요."
                        density="comfortable"></v-text-field>
                    <v-text-field label="주소" v-model="form.address" placeholder="주소를 입력하세요."
                        density="comfortable"></v-text-field>
                </v-card-text>
                <v-card-actions>
                    <v-btn color="primary" type="submit">
                        {{ isEdit ? '월드 수정' : '월드 생성' }}
                    </v-btn>
                    <v-btn color="error" @click="dialog = false">
                        취소
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-form>
    </v-dialog>

    <!-- 삭제 다이얼로그 -->
    <v-dialog v-model="deleteDialog" max-width="600">
        <v-card>
            <v-card-title>
                월드 삭제
            </v-card-title>
            <v-card-text>
                정말로 삭제하시겠습니까?
            </v-card-text>
            <v-card-actions>
                <v-btn color="error" type="submit" @click="deleteConfirm">
                    월드 삭제
                </v-btn>
                <v-btn color="primary" @click="deleteDialog = false">
                    취소
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>

</template>

<script>
import axios from 'axios';
export default {
    data() {
        return {
            isEdit: false,
            dialog: false,
            deleteDialog: false,
            deleteTargetId: null,
            headers: [
                { align: 'center', key: 'id', sortable: true, title: '번호' },
                { align: 'center', key: 'name', sortable: true, title: '이름' },
                { align: 'center', key: 'phoneNumber', sortable: true, title: '전화번호' },
                { align: 'center', key: 'actions', title: '수정 / 삭제', sortable: false },
            ],
            form: {
                id: null,
                name: '',
                phoneNumber: '',
                email: '',
                address: '',
            },
            name: '',
            loading: false,
            page: 1,
            itemsPerPage: 5,
            totalItems: 0,
            worlds: []
        }
    },
    watch: {
        name() { // 검색어가 바뀌면 1페이지부터 다시 가져오기
            if (this.page !== 1) { // 1페이지 아닌 경우 page=1로 바꾸고 page watch가 fetchWorlds 호출
                this.page = 1
            } else { // 이미 1페이지면 직접 호출
                this.fetchWorlds({ page: 1, itemsPerPage: this.itemsPerPage })
            }
        },
        page() { // 페이지가 바뀌면 fetchWorlds 직접 호출
            this.fetchWorlds({ page: this.page, itemsPerPage: this.itemsPerPage });
        },
        itemsPerPage() { // 페이지당 항목 수가 바뀌면 1페이지부터 다시 가져오기
            if (this.page !== 1) { // 1페이지 아닌 경우 page=1로 바꾸고 page watch가 fetchWorlds 호출
                this.page = 1
            } else { // 이미 1페이지면 직접 호출
                this.fetchWorlds({ page: 1, itemsPerPage: this.itemsPerPage })
            }
        }
    },
    methods: {
        goToWorldInfo(event, { item }) {
            this.$router.push("/worlds/" + item.id);
        },
        openDialog() {
            this.isEdit = false;
            this.form = { id: null, name: '', phoneNumber: '', email: '', address: '' };
            this.dialog = true;
        },
        async openEditDialog(world) {
            this.isEdit = true;
            try {
                const response = await axios.get("http://localhost:8080/api/worlds/" + world.id);
                this.form = {
                    id: response.data.id,
                    name: response.data.name,
                    phoneNumber: response.data.phoneNumber,
                    email: response.data.email,
                    address: response.data.address,
                }
            } catch (error) {
                console.error("월드 정보를 가져올 수 없습니다.");
            } finally {
                this.dialog = true;
            }
        },
        openDeleteDialog(id) {
            this.deleteTargetId = id;
            this.deleteDialog = true;
        },
        async deleteConfirm() {
            try {
                await axios.delete("http://localhost:8080/api/worlds/" + this.deleteTargetId);
                this.deleteDialog = false;
                this.fetchWorlds({ page: this.page, itemsPerPage: this.itemsPerPage });
            } catch (error) {
                console.error("월드를 삭제할 수 없습니다.");
            }
        },

        // this.fetchWorlds({ page: this.page, itemsPerPage: this.itemsPerPage }); // options :  {} 안에 있는 값
        async fetchWorlds(options = {}) {
            this.loading = true;
            try {
                const page = options?.page || this.page;
                const itemsPerPage = options?.itemsPerPage || this.itemsPerPage;
                const sortBy = options?.sortBy?.[0]
                    ? options.sortBy[0].key + ',' + options.sortBy[0].order
                    : 'id,asc';

                const response = await axios.post("http://localhost:8080/api/worlds/search", ({
                    keyword: `%${this.name}%`
                }), {
                    params: {
                        page: page - 1,
                        size: itemsPerPage,
                        sort: sortBy,
                    }
                })
                this.worlds = response.data._embedded.worlds;
                this.totalItems = response.data.page.totalElements;
            } catch (error) {
                console.error("월드 정보를 불러올 수 없습니다.", error);
            } finally {
                this.loading = false;
            }
        },
        async handleSubmit() {
            try {
                if (this.isEdit) {
                    await axios.put("http://localhost:8080/api/worlds/" + this.form.id, ({
                        name: this.form.name,
                        phoneNumber: this.form.phoneNumber,
                        email: this.form.email,
                        address: this.form.address,
                    }))
                } else {
                    await axios.post("http://localhost:8080/api/worlds", ({
                        name: this.form.name,
                        phoneNumber: this.form.phoneNumber,
                        email: this.form.email,
                        address: this.form.address,
                    }))
                }
            } catch (error) {
                console.error("월드 정보를 저장할 수 없습니다.");
            } finally {
                this.fetchWorlds({ page: this.page, itemsPerPage: this.itemsPerPage });
                this.dialog = false;
            }
        }
    }
}
</script>

<style src="@/styles/main.css"></style>