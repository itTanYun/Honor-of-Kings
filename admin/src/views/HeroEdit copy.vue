<template>
  <div class="about">
    <h1>{{ id ? '编辑' : '新建' }}英雄</h1>
    <el-form label-width="120px" @submit.native.prevent="save">
      <el-form-item label="名称">
        <el-input v-model="model.name"></el-input>
      </el-form-item>
      <el-form-item label="称号">
        <el-input v-model="model.title"></el-input>
      </el-form-item>
      <el-form-item label="类型">
        <el-select v-model="model.categories" multiple>
          <el-option
            v-for="item of categories"
            :key="item._id"
            :label="item.name"
            :value="item._id"
          ></el-option
        ></el-select>
      </el-form-item>
      <el-form-item label="难度">
        <el-rate
          style="margin-top: 0.6rem"
          :max="10"
          show-score
          v-model="model.scores.difficult"
        ></el-rate>
      </el-form-item>
      <el-form-item label="技能">
        <el-rate
          style="margin-top: 0.6rem"
          :max="10"
          show-score
          v-model="model.scores.skills"
        ></el-rate>
      </el-form-item>
      <el-form-item label="攻击">
        <el-rate
          style="margin-top: 0.6rem"
          :max="10"
          show-score
          v-model="model.scores.attack"
        ></el-rate>
      </el-form-item>
      <el-form-item label="生存">
        <el-rate
          style="margin-top: 0.6rem"
          :max="10"
          show-score
          v-model="model.scores.survive"
        ></el-rate>
        <!-- <el-input v-model="model.scores.difficult"></el-input> -->
      </el-form-item>
      <el-form-item label="顺风出装">
        <el-select v-model="model.items1" multiple>
          <el-option
            v-for="item of items"
            :key="item._id"
            :label="item.name"
            :value="item._id"
          ></el-option
        ></el-select>
      </el-form-item>
      <el-form-item label="逆风出装">
        <el-select v-model="model.items2" multiple>
          <el-option
            v-for="item of items"
            :key="item._id"
            :label="item.name"
            :value="item._id"
          ></el-option
        ></el-select>
      </el-form-item>
      <el-form-item label="使用技巧">
        <el-input type="textarea" v-model="model.usageTips"></el-input>
      </el-form-item>
      <el-form-item label="对抗技巧">
        <el-input type="textarea" v-model="model.battleTips"></el-input>
      </el-form-item>
      <el-form-item label="团战思路">
        <el-input type="textarea" v-model="model.teamTips"></el-input>
      </el-form-item>
      <el-form-item label="头像">
        <el-input v-model="model.avatar"></el-input>
      </el-form-item>
      <el-form-item label="上传头像">
        <el-upload
          class="avatar-uploader"
          :action="$http.defaults.baseURL + '/upload'"
          :show-file-list="false"
          :on-success="afterUpload"
          :before-upload="beforeAvatarUpload"
        >
          <img v-if="model.avatar" :src="model.avatar" class="avatar" />
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" native-type="submit">保存</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  export default {
    props: {
      id: {},
    },
    data() {
      return {
        categories: [],
        items: [],
        model: {
          scores: {
            difficult: 0,
          },
        },
        parents: [],
        imageUrl: '',
      };
    },
    created() {
      this.fetchParents();
      this.fetchCategories();
      this.fetchItems();
      this.id && this.fetch(this.id);
    },
    methods: {
      afterUpload(res, file) {
        console.log(res, file);
        this.$set(this.model, 'avatar', res.url);
        // this.model.icon = res.url; // 可能会无法响应赋值,也可以先在data上定义好，就 不用set赋值了。
        // this.imageUrl = URL.createObjectURL(res.raw);
      },
      beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg';
        const isLt2M = file.size / 1024 / 1024 < 2;

        if (!isJPG) {
          this.$message.error('上传头像图片只能是 JPG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
      },
      async fetch(id) {
        const res = await this.$http.get(`/rest/hero/${id}`);
        console.log(res);
        // 通过依次浅拷贝，确保model中有多层属性
        this.model = Object.assign({}, this.model, res.data);
        // this.model = res.data;
      },
      async fetchParents() {
        const res = await this.$http.get(`/rest/hero/`);
        console.log(res);
        this.parents = res.data;
      },
      async fetchCategories() {
        const res = await this.$http.get(`/rest/categories/`);
        console.log(res);
        this.categories = res.data;
      },
      async fetchItems() {
        const res = await this.$http.get(`/rest/item/`);
        // console.log(res);
        this.items = res.data;
      },
      async save() {
        if (this.id) {
          const res = await this.$http.put(`/rest/hero/${this.id}`, this.model);
          console.log(res);
          this.$message({
            type: 'success',
            message: '编辑成功',
          });
          this.$router.push('/hero/list');
          return;
        }
        // async await 与  this.$http.post().then 相似
        const res = await this.$http.post('/rest/hero/', this.model);
        console.log(res);
        this.$message({
          type: 'success',
          message: '创建成功',
        });
        this.$router.push('/hero/list');
      },
    },
  };
</script>

<style>
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409eff;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>