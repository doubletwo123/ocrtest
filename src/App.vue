<template>
  <div class="container">
    <h1 class="title">医学文献小助手</h1>
    <!-- 侧边栏容器 -->
    <div class="sidebar" :class="{ 'sidebar-open': previewImage }">
      <!-- 预览图片 -->
      <img v-if="previewImage" :src="previewImage" alt="预览图片" class="preview-image">
    </div>
    <div class="search-bar">
      <input type="text"  v-model="searchText" placeholder="请输入搜索关键字" class="search-input">
      <button @click="search" class="search-btn">搜索</button>
    </div>
    <!-- 历史搜索信息 -->
    <div class="history-search">
      <div v-for="item in history" :key="item" class="history-item" @click="selectHistory(item)">{{ item }}</div>
    </div>
    <ul class="results-list">
      <li v-for="result in filteredResults" :key="result.id" class="result-item">
        <!-- 坐标提示 -->
        <el-tooltip class="item" effect="dark" :content="`location（${result.left},${result.top}）`" placement="top-start">
          <span v-html="highlightText(result.text)"></span>
        </el-tooltip>
      </li>
      <li v-if="showEllipsis" class="result-item ellipsis">...</li>
    </ul>

    <el-upload
      class="upload-demo"
      action="https://jsonplaceholder.typicode.com/posts/"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      :file-list="fileList"
      list-type="picture"
      :auto-upload="false"
      :before-upload="beforeUpload"
    >
    <el-button size="small" type="primary">点击上传</el-button>
      <div class="el-upload__tip">{{ tip }}</div>
    </el-upload>
  </div>

</template>

<script>
import axios from 'axios';
import { Tooltip } from 'element-ui';
export default {
  components: {
    'el-tooltip': Tooltip,
  }, 
  data() {
    return {
      results: [],      // 存储从链接获取的数据
      searchText: '',   // 存储搜索关键字
      history: [],      // 历史搜索信息
      fileList: [],     
      tip: "只能上传jpg/png文件，且不超过500kb",
      previewImage: '', // 预览的图片URL
    };
  },
  created() {
    this.fetchData();
  },
  computed: {
    filteredResults() {
      if (this.searchText.trim() === '') {
        return this.results.slice(0, 4);
      } else {
        const keyword = this.searchText.trim().toLowerCase();
        const filtered = this.results.filter(result =>
          result.text.toLowerCase().includes(keyword)
        );
        return filtered;
      }
    },
    showEllipsis() {
      return this.searchText.trim() === '' && this.filteredResults.length < this.results.length;
    },
  },
  methods: {
    fetchData() {
      const url = 'https://api.jsonbin.io/v3/b/6458c9b98e4aa6225e98c3af';

      axios.get(url)
        .then(response => {
          this.results = response.data.record.results;
        })
        .catch(error => {
          console.error('请求数据时出现错误：', error);
        });
    },
    highlightText(text) {
      if (this.searchText.trim() === '') {
        return text;
      } else {
        const keyword = this.searchText.trim();
        const regex = new RegExp(keyword, 'gi');
        return text.replace(regex, "<span class='highlight'>$&</span>");
      }
    },
    search() {
      if (this.searchText.trim() !== '') {
        this.history.push(this.searchText.trim());
      }
    },
    selectHistory(item) {
      this.searchText = item;
    },
    handlePreview(file) {
      console.log(file);
      // 在这里实现预览图片的逻辑
      this.previewImage = file.url || file.thumbUrl;
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
      // 在这里实现移除图片的逻辑
    },
    beforeUpload(file) {
      const isJPG = file.type === "image/jpeg" || file.type === "image/png";
      const isLt500KB = file.size / 1024 < 500;

      if (!isJPG) {
        this.$message.error("只能上传 JPG/PNG 格式的图片");
      }
      if (!isLt500KB) {
        this.$message.error("图片大小不能超过 500KB");
      }

      return isJPG && isLt500KB;
    },
  },
};
</script>
<style>
.title {
  display: flex;
  justify-content: center;
  font-size: 24px;
  margin-bottom: 10px;
}

.container {
  display: flex;
  flex-direction: column;
}

.container > * {
  width: 100%;
  max-width: 50vw; /* 设置最大宽度为屏幕宽度的一半 */ 
}
.search-bar {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.search-input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.search-btn {
  padding: 10px 15px;
  font-size: 16px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.search-btn:hover {
  background-color: #0056b3;
}

.history-search {
  margin-bottom: 10px;
}

.history-item {
  padding: 5px 10px;
  font-size: 14px;
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.history-item:hover {
  background-color: #e0e0e0;
}

.results-list {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.result-item {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 5px;
}

.highlight {
  background-color: yellow;
}

.ellipsis {
  text-align: center;
  font-weight: bold;
}

.upload-demo {
  margin-top: 20px;
}

.el-upload__tip {
  margin-top: 10px;
  font-size: 14px;
  color: #888;
}

/* 侧边栏样式 */
.sidebar {
  position: fixed;
  top: 0;
  right: -50vw; /* 初始化时隐藏侧边栏 */
  bottom: 0;
  width: 45%; /* 最多占据屏幕宽度的一半 */
  background-color: #f0f0f0;
  transition: right 0.3s ease;
}

.sidebar-open {
  right: 0; /* 显示侧边栏 */
}

/* 预览图片样式 */
.preview-image {
  max-width: 100%;
  max-height: 100%;
}
</style>