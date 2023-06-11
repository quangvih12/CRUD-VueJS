
<template>
    <div>
        <div class="card">
            <Toolbar class="mb-4">
                <template #start>
                    <Button label="New" icon="pi pi-plus" severity="success" class="mr-2" @click="openNew" />
                </template>

                <template #end>
                    <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" chooseLabel="Import" class="mr-2 inline-block" />
                    <Button label="Export" icon="pi pi-upload" severity="help" @click="exportCSV($event)"  />
                </template>
            </Toolbar>

            <DataTable :value="catergory" tableStyle="min-width: 50rem">
                <template #header>
                    <div class="flex flex-wrap gap-2 align-items-center justify-content-between">
                        <h4 class="m-0">Manage Products</h4>
						<span class="p-input-icon-left">
                            <i class="pi pi-search" />
                            <InputText  placeholder="Search..." />
                        </span>
					</div>
                </template>

                <Column selectionMode="multiple" style="width: 3rem" :exportable="false"></Column>
                <Column field="id" header="id" sortable style="min-width:12rem"></Column>
                <Column field="name" header="Name" sortable style="min-width:16rem"></Column>
                <Column header="Image">
                    <template #body="slotProps">
                        <img :src="`https://primefaces.org/cdn/primevue/images/product/${slotProps.data.image}`" :alt="slotProps.data.image" class="shadow-2 border-round" style="width: 64px" />
                    </template>
                </Column>
               <Column field="price" header="Price" sortable style="min-width:8rem">
                     <template #body="slotProps">
                        {{formatCurrency(slotProps.data.price)}}
                    </template> 
                </Column>
              
                <Column :exportable="false" style="min-width:8rem">
                    <template #body="slotProps">
                        <Button icon="pi pi-pencil" outlined rounded class="mr-2" @click="editProduct(slotProps.data)" />
                        <Button icon="pi pi-trash" outlined rounded severity="danger" @click="confirmDeleteProduct(slotProps.data)" />
                    </template>
                </Column>
            </DataTable>
        </div>

        <Dialog v-model:visible="productDialog" :style="{width: '450px'}" header="catergory Details" :modal="true" class="p-fluid">
            <!-- <img v-if="product.image" :src="`https://primefaces.org/cdn/primevue/images/product/${catergory.image}`" :alt="catergory.image" class="block m-auto pb-3" /> -->
            <div class="field">
                <label for="name">Name</label>
                <InputText id="name" v-model="neItems.name" required="true" autofocus />
                <small class="p-error"  v-if="submitted && !neItems.name">Name is required.</small>
            </div>
            <div class="field">
                <label for="description">Description</label>
                <Textarea id="description" v-model="neItems.description"  required="true" rows="3" cols="20" />
            </div>

      
            <div class="formgrid grid">
                <div class="field col">
                    <label for="price">Price</label>
                    <InputNumber id="price" v-model="neItems.price" mode="currency" currency="USD" locale="en-US" />
                </div>
            </div>
            <template #footer>
                <Button label="Cancel" icon="pi pi-times" text @click="hideDialog"/>
                <Button label="Save" icon="pi pi-check" text @click="addCategory" />
            </template>
        </Dialog>

          <Dialog v-model:visible="productDialogUpdate" :style="{width: '450px'}" header="catergory Details" :modal="true" class="p-fluid">
            <!-- <img v-if="product.image" :src="`https://primefaces.org/cdn/primevue/images/product/${catergory.image}`" :alt="catergory.image" class="block m-auto pb-3" /> -->
            <div class="field">
                <label for="name">Name</label>
                <InputText id="name" v-model.trim="detail.name" required="true" autofocus />
                <!-- <small class="p-error" v-if="submitted && !catergory.name">Name is required.</small> -->
            </div>
            <div class="field">
                <label for="description">Description</label>
                <Textarea id="description" v-model="detail.description" required="true" rows="3" cols="20" />
            </div>

      
            <div class="formgrid grid">
                <div class="field col">
                    <label for="price">Price</label>
                    <InputNumber id="price" v-model="detail.price" mode="currency" currency="USD" locale="en-US" />
                </div>
            </div>
            <template #footer>
                <Button label="Cancel" icon="pi pi-times" text @click="hideDialog"/>
                <Button label="Save" icon="pi pi-check" text @click="updateItem(detail)" />
            </template>
        </Dialog>

        <Dialog v-model:visible="deleteProductDialog" :style="{width: '450px'}" header="Confirm" :modal="true">
            <div class="confirmation-content">
                <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                <span v-if="catergory">Are you sure you want to delete <b>{{deletes.name}}</b>?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="deleteProductDialog = false"/>
                <Button label="Yes" icon="pi pi-check" text @click="deleteItem(deletes.id)" />
            </template>
        </Dialog>
	</div>
</template>

<script setup>
import { defineComponent, ref, onMounted, reactive } from "vue";

onMounted(() => {
    axios
        .get("http://localhost:3000/catergory")
        .then((response) => {
          catergory.value = response.data;
          console.log(response);
        })
        .catch((error) => {
          console.log(error);
        });
});

    const catergory = ref([]);
    const detail = ref([]);
     const deletes = ref([]);
    const productDialog = ref(false);
    const productDialogUpdate = ref(false);
    const deleteProductDialog = ref(false);

  
    const neItems = reactive({
      name: "",
      description: "",
    });


    const addCategory = () => {
      axios
        .post("http://localhost:3000/catergory", neItems)
        .then(function (response) {
          catergory.value.push(response.data);
          if (response.status === 201) {
            alert("add thanh cong");
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    };
    const updateItem = (item) => {
      axios
        .put("http://localhost:3000/catergory/"+ item.id , item)
        .then(function (response) {
          if (response.status === 200) {
            alert("updae thanh cong");
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    };

    const deleteItem = (itemId) => {
      axios
        .delete("http://localhost:3000/catergory/" + itemId)
        .then(function (response) {
       window.open("/", '_self');
        });
    };
    const formatCurrency = (value) => {
    if(value)
        return value.toLocaleString('en-US', {style: 'currency', currency: 'USD'});
    return;
};
    const openNew = () => {
    productDialog.value = true;
};

    const hideDialog = () => {
    productDialog.value = false;
};
    const editProduct = (prod) => {
    detail.value = {...prod};
    productDialogUpdate.value = true;
  
};
    const confirmDeleteProduct = (prod) => {
        deletes.value = prod;
        deleteProductDialog.value = true;
    
};

  

</script>

<style>
.table {
  color: white;
}
</style>
