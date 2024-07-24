<template>
    <div>
        <table class="table-auto">
            <thead>
                <tr>
                    <th class="table-header">Part Number</th>
                    <th class="table-header">Revision</th>
                    <th class="table-header">Name</th>
                    <th class="table-header">Type</th>
                    <th class="table-header">State</th>
                </tr>
            </thead>
            <tbody>
                <template v-for="part in items" :key="part.id">
                    <tr class="bg-[#FFFFFF] hover:bg-[#F5F5F5] active:bg-[#DCDCDC]"
                        @click="toggleSubrows(part.id)">
                        <td class="table-cell table-cell-main">{{ part.item_number }}</td>
                        <td class="table-cell">{{ part.major_rev }}</td>
                        <td class="table-cell">{{ part.name }}</td>
                        <td class="table-cell">{{ part.classification }}</td>
                        <td class="table-cell">{{ part.state }}</td>
                    </tr>
                        <template v-if="visibleSubrows[part.id] && part.bomParts && part.bomParts.length > 0">
                            <tr v-for="bomPart in part.bomParts" :key="bomPart.id" class="table-row-subset">
                                <td class="table-cell">{{ bomPart.item_number }}</td>
                                <td class="table-cell">{{ bomPart.major_rev }}</td>
                                <td class="table-cell">{{ bomPart.name }}</td>
                                <td class="table-cell">{{ bomPart.classification }}</td>
                                <td class="table-cell">{{ bomPart.state }}</td>
                            </tr>
                        </template>
                </template>
            </tbody>
        </table>
    </div>
</template>

<script>
import axios from 'axios';
import CryptoJS from 'crypto-js';

export default {
    data() {
        return {
            visibleSubrows: {},
            items: [],
            hoveredRow: null,
            token: 'eyJhbGciOiJSUzI1NiIsImtpZCI6IjcxMjlBNUVFNEQxQUIwOTc3QjE4Q0Q0RUZFOTY0RjQ4MDgzQzA1NURSUzI1NiIsInR5cCI6ImF0K2p3dCIsIng1dCI6ImNTbWw3azBhc0pkN0dNMU9fcFpQU0FnOEJWMCJ9.eyJuYmYiOjE3MjE3Nzk4OTEsImV4cCI6MTcyMTc4MzQ5MSwiaXNzIjoiT0F1dGhTZXJ2ZXIiLCJhdWQiOlsiSW5ub3ZhdG9yIiwiT0F1dGhTZXJ2ZXIvcmVzb3VyY2VzIl0sImNsaWVudF9pZCI6IklPTUFwcCIsInN1YiI6InJvb3QiLCJhdXRoX3RpbWUiOjE3MjE3Nzk4OTEsImlkcCI6ImxvY2FsIiwidXNlcm5hbWUiOiJyb290IiwiZGF0YWJhc2UiOiJTb2Z0d2VyayIsImp0aSI6IjFFOEFGMTlBODVFNDBDQTVFMkZBMkI1MzhFRDQ1OEYzIiwiaWF0IjoxNzIxNzc5ODkxLCJzY29wZSI6WyJJbm5vdmF0b3IiXSwiYW1yIjpbInBhc3N3b3JkIl19.aNwu3Kd2_k5uNV05rEwlUMLFL2y01VGA8nUqtCaExgN2QH9nasHyPlPmg4MHt6uCK2eFaH1zryoPW25CDgO9eJ15FjJXGIKPmAsJVWTacmeAniNmvMcpvaGka7V_MSzJjwlXnApWMCqpHNkTTqPSXSHNF_2ECutJrc1uVzQn2PWCUsJZ8BbXBM1gKjYWP8utFTkQJZRSKGx8JBJlDKGizmKGiOkHmW99-UpCYQaatkBGcTF0OT1pgBzLZcAex3-a3SoSexJKiAv0gDWASTIL9RTrrvuAx5pQZmWQigyWuPO81FrxIHawgOSGV3y69Y2g1tFI85z6k5C8t5dCZsMORA'
        }
    },
    created() {
        this.items.forEach(part => {
            this.visibleSubrows[part.id] = false;
        });
        this.auth();
    },
    mounted() {
        this.fetchData();
    },
    methods: {
        async auth() {
            try {
                const response = await axios.post('https://plm.softwerk.digital/Softwerk/oauthserver/connect/token', {
                    grant_type: 'password',
                    scope: 'Innovator',
                    client_id: 'IOMApp',
                    username: 'root',
                    password: CryptoJS.MD5('innovator').toString(),
                    database: 'Softwerk'
                })
                this.token = response.data.access_token;
            } catch (error) {
                console.error(error);
            }
        },

        async fetchData() {
            try {
                const response = await axios.get('https://plm.softwerk.digital/Softwerk/Server/odata/Part?$expand=Part BOM($expand=related_id)', {
                    headers: {
                        'Authorization': 'Bearer ' + this.token
                    }
                });
                const items = response.data.value.map(item => {
                    const bomParts = item['Part BOM'] && item['Part BOM'].length > 0
                        ? item['Part BOM'].map(bomPart => bomPart.related_id)
                        : [];
                    console.log(bomParts)
                    return {
                        ...item,
                        bomParts
                    }
                })
                this.items = items;
                console.log(items)
            } catch (error) {
                console.error(error);
            }
        },
        toggleSubrows(partId) {
            this.visibleSubrows[partId] = !this.visibleSubrows[partId];
        },
    }
}

</script>

<style lang="css">
table {
    width: 100%;
    border-collapse: collapse;
}
th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}
</style>