<template>
    <v-container max-width="1100" class="pa-8">

        <!-- 헤더 -->
        <div class="d-flex align-center justify-space-between mb-6">
            <h1 class="font-weight-bold">유저 정보</h1>
        </div>

        <!-- 목록 버튼 -->
        <div class="d-flex align-center justify-space-between mb-4">
            <v-btn color="primary" prepend-icon="mdi-arrow-left" @click="$router.back()">목록으로</v-btn>
        </div>

        <!-- 정보 카드 -->
        <v-card variant="outlined">
            <v-table>
                <tbody>
                    <tr>
                        <td class="text-grey" style="width: 150px;">번호</td>
                        <td>{{ world.id }}</td>
                    </tr>
                    <tr>
                        <td class="text-grey">이름</td>
                        <td>{{ world.name }}</td>
                    </tr>
                    <tr>
                        <td class="text-grey">전화번호</td>
                        <td>{{ world.phoneNumber }}</td>
                    </tr>
                    <tr>
                        <td class="text-grey">이메일</td>
                        <td>{{ world.email }}</td>
                    </tr>
                    <tr>
                        <td class="text-grey">주소</td>
                        <td>{{ world.address }}</td>
                    </tr>
                </tbody>
            </v-table>
        </v-card>

    </v-container>
</template>

<script>
import axios from 'axios';
export default {
    name: 'worldInfo',
    data() {
        return {
            world: {}
        }
    },
    async mounted() {
        await this.fetchWorld();
    },
    methods: {
        async fetchWorld() {
            try {
                const response = await axios.post("http://localhost:8080/api/worlds/" + this.$route.params.id)
                this.world = response.data;
            } catch (error) {
                console.error("유저 정보를 가져올 수 없습니다.", error)
            }
        },
    }
}
</script>