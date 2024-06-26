<template>
    <div>
        <!-- -------------------------=-Menu------------------------- -->
        <div class="flex flex-col justify-between mx-5 my-3">
            <!-- Search and filter -->
            <div class="flex flex-grow justify-center md:justify-start mx-5 my-3">
                <!-- Add product button -->
                <a-tooltip placement="top" class="mr-2">
                    <template #title>
                        <span>Form Add new product</span>
                    </template>
                    <a-button @click="openAddProduct = true"
                        class="mx-3 text-white font-semibold text-medium bg-blue-400">New product
                        <PlusSquareOutlined />
                    </a-button>
                </a-tooltip>
                <!-- Search bar -->
                <a-input-search v-model:value="searchValue" placeholder="Input product name..."
                    style="width: 34rem; padding-left: 2rem;" @search="onSearch" />
            </div>
            
            <!-- Checkbox group -->
            <div class="flex flex-grow">
                <a-checkbox-group class="pl-7 text-medium font-semibold" v-model:value="checkboxValue"
                    style="width: 100%" :options="checkboxOptions">
                </a-checkbox-group>
                <a-button class="w-20 font-semibold" @click="handleCheckboxGroup">Filter</a-button>
            </div>




        </div>

        <!-- ----------------------------Body: Product lists-------------------- -->
        <a-table v-if="!isLoading" :columns="columns" :data-source="productList" @change="handleChange">
            <template #bodyCell="{ column, record }">
                <template v-if="column.key === 'imageURL'">
                    <NuxtImg class="ml-6 mt-3" height="80" width="100" alt="noImage"
                        :src="record.imageURL ? record.imageURL : 'noImage.png'" />
                </template>
                <template v-if="column.key === 'price'">
                    <p>{{ record.unitPrice.toLocaleString('vi', { style: 'currency', currency: 'VND' }) }}</p>
                </template>
                <template v-if="column.key === 'stock'">
                    <p>{{ record.quantityInStock }}</p>
                </template>
                <template v-if="column.key === 'action'">
                    <!-- Edit product -->
                    <a-tooltip placement="top">
                        <template #title>
                            <span>Edit</span>
                        </template>
                        <a-button info class="bg-amber-400 mr-2" @click="handleOpenEditProduct(record)">
                            <EditOutlined />
                        </a-button>
                    </a-tooltip>
                    <!-- Delete product -->
                    <a-tooltip placement="top">
                        <template #title>
                            <span>Delete</span>
                        </template>
                        <a-button danger @click="handleDeleteProduct(record)">
                            <DeleteOutlined />
                        </a-button>
                    </a-tooltip>
                </template>
            </template>
        </a-table>

        <!-- -----------------------------Modals-------------------------------- -->
        <!-- Add product -->
        <a-modal v-model:open="openAddProduct" title="Add new product" :footer="null">
            <a-form :model="newProduct" @finish="handleAddProduct">
                <a-form-item label="Category" name="category">
                    <a-select ref="select" v-model:value="newProduct.category" :options="categoryOptions" @change="">
                    </a-select>
                </a-form-item>
                <a-form-item label="Product Name" name="productName">
                    <a-input placeholder="Product name" v-model:value="newProduct.productName"></a-input>
                </a-form-item>

                <a-form-item label="Unit Price" name="unitPrice">
                    <a-input placeholder="VND" v-model:value="newProduct.unitPrice"></a-input>
                </a-form-item>

                <a-form-item label="Image">
                    <a-upload v-model:file-list="fileList" list-type="picture" :max-count="1">
                        <a-button>
                            <upload-outlined></upload-outlined>
                            Upload
                        </a-button>
                    </a-upload>
                </a-form-item>
                <a-form-item class="flex justify-end mr-6" :wrapper-col="{ offset: 8, span: 16 }">
                    <a-button type="primary" html-type="submit">Submit</a-button>
                </a-form-item>
            </a-form>

        </a-modal>
        <!-- Edit product -->
        <a-modal v-model:open="openEditProduct" title="Edit product">
            <a-form :model="selectProduct">
                <a-form-item label="Product name">
                    <a-input v-model:value="selectProduct.productName" />
                </a-form-item>

                <a-form-item label="Category">
                    <a-select ref="select" v-model:value="selectProduct.category" style="width: 120px"
                        :options="categoryOptions" @change="">
                    </a-select>
                </a-form-item>

                <a-form-item label="Price">
                    <a-input-number class="w-60" v-model:value="selectProduct.unitPrice" :controls="false" :min="0" />
                </a-form-item>


                <a-form-item label="Image">
                    <a-upload v-model:file-list="fileList" list-type="picture" :max-count="1">
                        <a-button>
                            <upload-outlined></upload-outlined>
                            Upload
                        </a-button>
                    </a-upload>
                </a-form-item>

            </a-form>
            <template #footer>
                <a-button key="back" @click="openEditProduct = false">Close</a-button>
                <a-button key="submit" type="primary" @click="handleEditProduct">Save</a-button>
            </template>
        </a-modal>
    </div>

</template>
<script lang="ts">
import type { UploadProps } from 'ant-design-vue';
import axios from 'axios';

interface Product {
    productName: string;
    category: string;
    stock: number;
    unitPrice: number;
    imageURL: string;
}

export default {
    setup() {
        const fileList = ref<UploadProps['fileList']>([]);   
        return {
            fileList,
        }
    },
    data() {
        return {
            searchValue: '',
            openAddProduct: false,
            openEditProduct: false,
            checkboxValue: [],
            checkboxOptions: [
                { label: 'Laptop', value: 'Laptop' },
                { label: 'Watch', value: 'Watch' },
                { label: 'TV', value: 'TV' },
                { label: 'Accesories', value: 'Accesories' },
                { label: 'Headphone', value: 'Headphone' },

            ],
            selectProduct: {
                id: null,
                productName: 'Example product',
                category: 'Laptop',
                stock: 24,
                unitPrice: 9999,
                imageURL: ''
            },
            newProduct: reactive<Product>({
                productName: '',
                category: '',
                stock: 0,
                unitPrice: 0,
                imageURL: ''
            }),
            categoryOptions: [
                {
                    value: 'laptop',
                    label: 'Laptop'
                },
                {
                    value: 'watch',
                    label: 'Watch'
                }
            ],
            columns: [
                {
                    key: 'imageURL'
                },
                {
                    title: 'Product',
                    dataIndex: 'productName',
                    key: 'productName',
                    filterSearch: true,
                },
                {
                    title: 'Category',
                    dataIndex: 'category',
                    key: 'category',
                    filters: [
                        { text: 'Laptop', value: 'Laptop' },
                        { text: 'Watch', value: 'Watch' },
                    ],
                },
                {
                    title: 'Price',
                    key: 'price',
                    dataIndex: 'price',
                    sorter: (a: any, b: any) => a.unitPrice - b.unitPrice,
                },
                {
                    title: 'Stock',
                    key: 'stock',
                    dataIndex: 'stock',
                },
                {
                    title: 'Action',
                    key: 'action',
                },
            ],
            data: [
                {
                    key: '1',
                    imageURL: 'https://cdn.tgdd.vn/Products/Images/44/321059/asus-tuf-gaming-a15-fa507nu-r5-lp105w-thumb-600x600.jpg',
                    productName: 'ASUS TUF GAMING',
                    price: 18800000,
                    category: 'Laptop',
                    stock: 90,
                },
                {
                    key: '2',
                    imageURL: 'data:image/jpeg;base64,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',
                    productName: 'ASUS VIVOBOOK X509',
                    price: 9000000,
                    category: 'Laptop',
                    stock: 105,
                },
                {
                    key: '3',
                    imageURL: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT6AKHNBOR4GSKT38o-SPiewFDP2QVPVSHVdrqUNK2GumuWPlZz',
                    productName: 'Apple Watch Series 8',
                    price: 13990000,
                    category: 'Watch',
                    stock: 105,
                },
                {
                    key: '4',
                    imageURL: 'https://lzd-img-global.slatic.net/g/p/c2aae10768d28efd815ebc02fcc90aff.jpg_400x400q75.jpg_.webp',
                    productName: 'Apple Watch Series 8',
                    price: 11990000,
                    category: 'Watch',
                    stock: 105,
                },
            ],
            productList: [],
            isLoading: true,
        }
    },

    methods: {
        onSearch(value: string) {
            console.log(value)
        },
        async handleCheckboxGroup() {
            let bearerToken = localStorage.getItem("user_token");
            await axios.post('http://localhost:8000/api/products/getProductByCategory',
                { category: this.checkboxValue },
                {
                    withCredentials: true,
                    headers: {
                        "Content-Type": "application/json",
                        "Authorization": `Bearer ${bearerToken}`
                    },
                })
                .then(response => {
                    // change products list 
                    this.productList = response.data.products;
                })
                .catch(error => {
                    console.error('Error fetching data:', error);

                })
            this.isLoading = false;

        },

        async handleAddProduct(values: any) {
            try {
                await axios.post('http://localhost:8000/api/products', {
                    productName: values.productName,
                    category: values.category,
                    unitPrice: values.unitPrice,
                    // imageURL: '',
                })
                this.getProductList();
                this.openAddProduct = false;
            } catch (error) {
                console.log(error)
            }
        },
        async handleDeleteProduct(product: any) {
            if (confirm("Do you Want to delete these items?") == true) {
                try {
                    let bearerToken = localStorage.getItem("user_token");
                    await axios.delete(`http://localhost:8000/api/products/${product.id}`,
                        {
                            withCredentials: true,
                            headers: {
                                "Content-Type": "application/json",
                                "Authorization": `Bearer ${bearerToken}`
                            },
                        }
                    )
                    // get new list
                    this.getProductList();
                } catch (error) {
                    console.log(error)
                }
            } else {
                console.log("do nothing")
            }

        },

        handleOpenEditProduct(product: any) {
            if (product != null) {
                this.selectProduct = product
                this.openEditProduct = true;
            }
        },
        async handleEditProduct() {
            try {
                let bearerToken = localStorage.getItem("user_token");
                await axios.put(`http://localhost:8000/api/products/${this.selectProduct.id}`,
                    {
                        productName: this.selectProduct.productName,
                        category: this.selectProduct.category,
                        unitPrice: this.selectProduct.unitPrice,
                    },
                    {
                        withCredentials: true,
                        headers: {
                            "Content-Type": "application/json",
                            "Authorization": `Bearer ${bearerToken}`
                        },
                    }
                )
                // get new list
                this.openEditProduct = false;
                this.getProductList();
            } catch (error) {
                console.log(error)
            }

        },
        handleChange(pagination: any, filters: any, sorter: any) {
            console.log({
                pagination: pagination,
                filters: filters,
                sorter: sorter
            })
        },
        async getProductList() {
            let bearerToken = localStorage.getItem("user_token");
            await axios.get('http://localhost:8000/api/products',
                {
                    withCredentials: true,
                    headers: {
                        "Content-Type": "application/json",
                        "Authorization": `Bearer ${bearerToken}`
                    },
                }
            )
                .then(response => {
                    this.productList = response.data.data;
                })
                .catch(error => {
                    console.error('Error fetching data:', error);

                })
            this.isLoading = false
        }
    },
    created() {
        this.getProductList();
    },
}



</script>
