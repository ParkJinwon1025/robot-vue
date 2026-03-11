<template>
    <v-container max-width="1100" class="pa-8">

        <div class="d-flex justify-space-between ma-8">
            <h1 class="font-weight-bold">월드 목록</h1>
        </div>

        <div class="d-flex justify-space-between ma-4">
            <v-text-field v-model="name" placeholder="이름으로 검색" density="compact" prepend-icon="mdi-account"
                max-width="300px" variant="outlined" hide-details>
            </v-text-field>
            <v-btn color="primary" prepend-icon="mdi-plus" @click="openDialog">월드 생성</v-btn>
        </div>

        <v-card variant="outlined">
            <v-data-table-server :items="worlds" :search="search" :loading="loading" :headers="headers"
                @click:row="goToWorldInfo" style="cursor: pointer;" hover :items-length="totalItems"
                v-model:items-per-page="itemsPerPage" @update:options="fetchWorlds">

                <template v-slot:item.actions="{ item }">
                    <v-btn color="primary" size="small" variant="tonal" @click.stop="openEditDialog(item)"
                        class="mr-1">수정</v-btn>
                    <v-btn color="error" size="small" variant="tonal" @click.stop="openDeleteDialog(item.id)"
                        class="mr-1">삭제</v-btn>
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
    <v-dialog v-model="deleteDialog" max-width="600px">
        <v-card>
            <v-card-title>월드 삭제</v-card-title>
            <v-card-text>정말로 삭제하시겠습니까?</v-card-text>
            <v-card-actions>
                <v-btn color="error" @click="deleteConfirm">삭제</v-btn>
                <v-btn color="primary" @click="deleteDialog = false">취소</v-btn>
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
            },
            name: '',
            loading: false,
            search: '',
            itemsPerPage: 10,
            totalItems: 0,
            worlds: []
        }
    },
    watch: {
        name() {
            this.search = String(Date.now());
        }
    },
    async mounted() {
        this.fetchWorlds();
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
                const response = await axios.post("http://localhost:8080/api/worlds/" + world.id);
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
                this.fetchWorlds();
            } catch (error) {
                console.error("월드를 삭제할 수 없습니다.");
            }
        },
        async fetchWorlds() {
            try {
                const response = await axios.post("http://localhost:8080/api/worlds/search", ({
                    keyword: `%${this.name}%`
                }))
                this.worlds = response.data._embedded.worlds;
                this.totalItems = response.data.page.totalElements;
            } catch (error) {
                console.error("월드 정보를 불러올 수 없습니다.");
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
                this.fetchWorlds();
                this.dialog = false;
            }
        }
    }
}
</script>

<style src="@/styles/main.css"></style>