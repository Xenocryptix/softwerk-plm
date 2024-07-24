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
                    <transition-group>
                        <template v-if="visibleSubrows[part.id] && part.bomParts && part.bomParts.length > 0">
                            <tr v-for="bomPart in part.bomParts" :key="bomPart.id" class="table-row-subset">
                                <td class="table-cell">{{ bomPart.item_number }}</td>
                                <td class="table-cell">{{ bomPart.major_rev }}</td>
                                <td class="table-cell">{{ bomPart.name }}</td>
                                <td class="table-cell">{{ bomPart.classification }}</td>
                                <td class="table-cell">{{ bomPart.state }}</td>
                            </tr>
                        </template>
                    </transition-group>
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
            token: 'eyJhbGciOiJSUzI1NiIsImtpZCI6IjcxMjlBNUVFNEQxQUIwOTc3QjE4Q0Q0RUZFOTY0RjQ4MDgzQzA1NURSUzI1NiIsInR5cCI6ImF0K2p3dCIsIng1dCI6ImNTbWw3azBhc0pkN0dNMU9fcFpQU0FnOEJWMCJ9.eyJuYmYiOjE3MjE3ODQwMTIsImV4cCI6MTcyMTc4NzYxMiwiaXNzIjoiT0F1dGhTZXJ2ZXIiLCJhdWQiOlsiSW5ub3ZhdG9yIiwiT0F1dGhTZXJ2ZXIvcmVzb3VyY2VzIl0sImNsaWVudF9pZCI6IklPTUFwcCIsInN1YiI6InJvb3QiLCJhdXRoX3RpbWUiOjE3MjE3ODQwMTIsImlkcCI6ImxvY2FsIiwidXNlcm5hbWUiOiJyb290IiwiZGF0YWJhc2UiOiJTb2Z0d2VyayIsImp0aSI6IkEzQ0VEQjYxQjI3OUQxRDY4MkNDNzgzMTk5QUE2RUQyIiwiaWF0IjoxNzIxNzg0MDEyLCJzY29wZSI6WyJJbm5vdmF0b3IiXSwiYW1yIjpbInBhc3N3b3JkIl19.fMkqONgJfuJOb5UsgFH3uV6PeV2Q-NQYGpN9QhZ7mVzp1Z93roanG6uszvHnZPrft0_SNxaxyEqs80VNTzXVXtbUGcXKIUXeSOi4WubCHNg7cdCDYVqT4ZmyJ8zMlSIYzPVD2jjyJbb3bOgG64Jr7NItI2dTRPdGd8v2WPfZDZFtklcwAMW0qaNxKOXa96fuI1jzB6fH_pDZA-pfkxWNVY9vjxzqkwiS-oaO4d96KNk3DKHJDlvY98GuINiVrM0R6Uouo7oRdPgAlxQpCM46VGpbMJEC73ce6ZuixmQb3M18SnuKwvWdMKBOqu--ePM06YNPQr7d9uEW9dUMrgCBng'
            //manually insert token from Postman if the auth() function is not working
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