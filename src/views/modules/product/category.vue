<template>
    <div>
        <el-switch v-model="draggable"
                   active-text="不可拖拽"
                   inactive-text="拖拽修改">
        </el-switch>
        <el-button style="margin: 0 0 0 20px"
                   type="success"
                   round
                   v-if="draggable"
                   v-on:click="updateCateGorySort">
            批量修改
        </el-button>
        <el-button style="margin: 0 0 0 20px"
                   type="danger"
                   round
                   v-if="!draggable"
                   v-on:click="removeCheckedCateGory">
            批量删除
        </el-button>
        <el-tree class="margin"
                 :data="menus"
                 :props="defaultProps"
                 :expand-on-click-node="false"
                 show-checkbox
                 node-key="catId"
                 :default-expanded-keys="expandedKey"
                 :draggable="draggable"
                 :allow-drop="allowDrop"
                 @node-drop="handleDrop"
                 ref="categoryTree">
            <span class="custom-tree-node"
                  slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                  <el-button
                      type="text"
                      size="mini"
                      v-if="data.catLevel <=2"
                      @click="() => append(data)">
                    Append
                  </el-button>
                    <el-button
                        type="text"
                        size="mini"
                        @click="() => edit(data)">
                    Edit
                  </el-button>
                  <el-button
                      type="text"
                      size="mini"
                      v-if="data.children.length ==0"
                      @click="() => remove(node, data)">
                    Delete
                  </el-button>
                </span>
            </span>
        </el-tree>

        <el-dialog
            title="提示"
            :visible.sync="dialogVisible"
            width="30%">
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name"
                              autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="分类图标">
                    <el-input v-model="category.icon"
                              autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="分类单位">
                    <el-input v-model="category.productUnit"
                              autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer"
                  class="dialog-footer">
                <el-button @click="dialogVisible = false">
                    取 消
                </el-button>
                <el-button type="primary"
                           @click="submitCateGory">
                    确 定
                </el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
let id = 1000;
const defaultCategory = {
    catId: null,
    name: "",
    parentCid: 0,
    catLevel: 0,
    productUnit: "",
    icon: "",
    showStatus: 1,
    sort: 0
};

export default {
    name: "category",
    data() {
        return {
            updateNodes: [],
            updateParentID: [],
            maxLevel: 1,
            menus: [],
            defaultProps: {
                children: "children",
                label: "name"
            },
            isEdit: false,
            expandedKey: [],
            dialogVisible: false,
            draggable: false,
            category: Object.assign({}, defaultCategory)
        };
    },
    mounted() {
        this.getProduct();
    },
    methods: {
        getProduct() {
            this.$http({
                url: this.$http.adornUrl("/product/category/list/tree"),
                method: "get"
            }).then(({ data }) => {
                this.menus = data.page;
            });
        },
        append(data) {
            this.category = Object.assign({}, defaultCategory);
            this.isEdit = false;
            console.log(data);
            this.category.parentCid = data.catId;
            this.category.catLevel = data.catLevel * 1 + 1;
            this.dialogVisible = true;
        },
        edit(data) {
            console.log(data);
            this.category = Object.assign({}, defaultCategory);
            this.isEdit = true;
            this.category.name = data.name;
            this.category.icon = data.icon;
            this.category.productUnit = data.productUnit;
            this.category.catId = data.catId;
            this.dialogVisible = true;
        },
        submitCateGory() {
            if (this.isEdit) {
                this.editCateGory();
            } else {
                this.addCateGory();
            }
        },
        addCateGory() {
            console.log(this.category);
            this.$http({
                url: this.$http.adornUrl("/product/category/save"),
                method: "post",
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                this.$message({
                    message: "菜单添加成功",
                    type: "success"
                });
                this.getProduct();
                this.expandedKey = [this.category.parentCid];
                this.dialogVisible = false;
            });
        },
        editCateGory() {
            console.log(this.category);
            let { catId, name, icon, productUnit } = this.category;
            this.$http({
                url: this.$http.adornUrl("/product/category/update"),
                method: "post",
                data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
                this.$message({
                    message: "菜单修改成功",
                    type: "success"
                });
                this.getProduct();
                this.expandedKey = [this.category.parentCid];
                this.dialogVisible = false;
            });
        },
        updateCateGorySort() {
            this.$http({
                url: this.$http.adornUrl("/product/category/update/sort"),
                method: "post",
                data: this.$http.adornData(this.updateNodes, false)
            }).then(({ data }) => {
                this.$message({
                    message: "菜单修改成功",
                    type: "success"
                });
                this.getProduct();
                this.expandedKey = [this.category.parentCid];
                this.updateNodes = [];
                this.expandedKey = this.updateParentID;
                this.updateParentID = [];
            });
        },

        remove(node, data) {
            this.$confirm(`是否删除【${data.name}】`, "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            }).then(() => {
                let ids = [data.catId];
                this.$http({
                    url: this.$http.adornUrl("/product/category/delete"),
                    method: "post",
                    data: this.$http.adornData(ids, false)
                }).then(({ data }) => {
                    this.$message({
                        message: "菜单删除成功",
                        type: "success"
                    });
                    this.getProduct();
                    this.expandedKey = [node.parent.data.catId];
                });
            }).catch(() => {
                this.$message({
                    type: "info",
                    message: "已取消删除"
                });
            });
        },
        removeCheckedCateGory(){
            let checkedKeys = this.$refs.categoryTree.getCheckedKeys();
            console.log(JSON.stringify(checkedKeys))
            this.$confirm(`是否批量删除【${checkedKeys}】菜单`, "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl("/product/category/delete"),
                    method: "post",
                    data: this.$http.adornData(checkedKeys, false)
                }).then(({ data }) => {
                    this.$message({
                        message: "菜单批量删除成功",
                        type: "success"
                    });
                    this.getProduct();
                });
            }).catch(() => {
                this.$message({
                    type: "info",
                    message: "已取消删除"
                });
            });

        },

        allowDrop(draggingNode, dropNode, type) {
            this.countMaxLevel(draggingNode);
            let deep = this.maxLevel - draggingNode.level + 1;
            if (type == "inner") {
                return deep + dropNode.level <= 3;
            } else {
                return deep + dropNode.parent.level <= 3;
            }
        },

        countMaxLevel(node) {
            this.maxLevel = node.level;
            if (node.childNodes != null && node.childNodes.length > 0) {
                for (let i = 0; i < node.childNodes.length; i++) {
                    this.countMaxLevel(node.childNodes[i]);
                }
            }
        },
        handleDrop(draggingNode, dropNode, dropType, ev) {
            console.log(draggingNode, dropNode, dropType, ev);
            let parentNode;
            if (dropType == "inner") {
                parentNode = dropNode;
            } else {
                parentNode = dropNode.parent;
            }
            this.updateParentID.push(parentNode.data.catId);
            let newChildNodes = parentNode.childNodes;
            // 根据变更的节点，循环它的父节点下所有的子节点
            for (let i in newChildNodes) {
                let node = newChildNodes[i];
                let category = node.data;
                // 当循环进行到移动的节点时 需要更新改节点的层级以及父节点信息
                if (category.catId == draggingNode.data.catId) {
                    category.parentCid = parentNode.data.catId;
                    let level = parentNode.level + 1;
                    if (category.catLevel != level) {
                        category.catLevel = level;
                        // 需要更新子节点的层级
                        let childCateGoryNodes = node.childNodes;
                        if (childCateGoryNodes != null && childCateGoryNodes.length > 0) {
                            for (let j in childCateGoryNodes) {
                                let childCategory = childCateGoryNodes[j].data;
                                childCategory.catLevel = level + 1;
                                let { catId, catLevel } = childCategory;
                                this.updateNodes.push({ catId, catLevel });
                            }
                        }
                    }
                }
                // 其他节点 只需要修改排序
                category.sort = i;
                let { catId, catLevel, sort, parentCid } = category;
                this.updateNodes.push({ catId, catLevel, sort, parentCid });
            }
            console.log(JSON.stringify(this.updateNodes));
        }
    }
};
</script>

<style>
.margin {
    margin: 20px 0 0 0;
}
</style>
