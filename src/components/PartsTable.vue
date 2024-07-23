<template>
    <div>
        <table>
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
                    <tr :class="[{ 'row-hover': hoveredRow === part.id }, { 'row-active': visibleSubrows[part.id] }]"
                        @click="visibleSubrows[part.id] = !visibleSubrows[part.id]"
                        @mouseover="setHoveredRow(part.id)"
                        @mouseleave="clearHoveredRow">
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

const TOKEN = 'eyJhbGciOiJSUzI1NiIsImtpZCI6IjcxMjlBNUVFNEQxQUIwOTc3QjE4Q0Q0RUZFOTY0RjQ4MDgzQzA1NURSUzI1NiIsInR5cCI6ImF0K2p3dCIsIng1dCI6ImNTbWw3azBhc0pkN0dNMU9fcFpQU0FnOEJWMCJ9.eyJuYmYiOjE3MjE3NTk4MDIsImV4cCI6MTcyMTc2MzQwMiwiaXNzIjoiT0F1dGhTZXJ2ZXIiLCJhdWQiOlsiSW5ub3ZhdG9yIiwiT0F1dGhTZXJ2ZXIvcmVzb3VyY2VzIl0sImNsaWVudF9pZCI6IklPTUFwcCIsInN1YiI6InJvb3QiLCJhdXRoX3RpbWUiOjE3MjE3NTk4MDIsImlkcCI6ImxvY2FsIiwidXNlcm5hbWUiOiJyb290IiwiZGF0YWJhc2UiOiJTb2Z0d2VyayIsImp0aSI6IjNBNDNGRkUwMkJBMEI0NjAzODk1RTNFMzU3RjlBRkM3IiwiaWF0IjoxNzIxNzU5ODAyLCJzY29wZSI6WyJJbm5vdmF0b3IiXSwiYW1yIjpbInBhc3N3b3JkIl19.wQrzThfw_cbgGG0ZiuntK8YFkj2GwCzQkVRGTIhKWeuW9bC2WTYfa8-AlPXAMVyHtmSXL6urlTNKyLpL9cwqdtW8cV3CivdHQAlOoMkngHgUpO766XNAVMbdPOeQKN2YetFftvFVFCaups9rl1Qq1Ixywq4RwPQSVpQznvxEUfSH5WkepLzYanceEdn4zaaGw5VpvlFOZ9Us-Zlx1DFDbFrvSwFP750QqB4nF31gpN5yrUxSq4Hqm0fXU_eTnJdzhQtdZfIxAUfvH0IXrbCSlFj26yGSTfswzCz0wC6CDEQ0-KEMU0PCh_K16LnisvueGgML8royoS1ZlxjRgedByw';

export default {
    data() {
        return {
            visibleSubrows: {},
            items: []
        }
    },
    created() {
        this.items.forEach(part => {
        this.visibleSubrows[part.id] = false;
        });
    },
    mounted() {
        this.fetchData();
    },
    methods: {
        async fetchData() {
            try {
                const response = await axios.get('https://plm.softwerk.digital/Softwerk/Server/odata/Part?$expand=Part BOM($expand=related_id)', {
                    headers: {
                        'Authorization': 'Bearer ' + TOKEN
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
        setHoveredRow(partId) {
            this.hoveredRow = partId;
        },
        clearHoveredRow() {
            this.hoveredRow = null;
        }
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