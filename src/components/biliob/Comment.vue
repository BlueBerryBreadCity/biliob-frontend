<template>
  <VContainer>
    <VRow dense>
      <VCol cols="12">
        <VExpansionPanels tile>
          <VExpansionPanel>
            <VExpansionPanelHeader class="primary--text"
              ><h3>
                <VIcon color="primary" left>
                  mdi-comment-multiple-outline </VIcon
                >评论系统 / COMMENTS SYS
              </h3></VExpansionPanelHeader
            >
            <VExpansionPanelContent>
              <h4>评论系统规则</h4>
              <div class="body-2">
                <li>
                  只有登陆观测者能够发表评论。
                </li>
                <li>
                  请发表你觉得能够发表的言论。
                </li>
                <li>发表评论将 消耗1积分</li>
                <li>点赞评论将 消耗0.1积分</li>
                <li>评论被点赞将 获得0.1积分</li>
              </div>
            </VExpansionPanelContent>
          </VExpansionPanel>
        </VExpansionPanels>
      </VCol>
      <VCol cols="12">
        <VTabs class="elevation-3">
          <VTab @click="sort = 0">喜欢序 / LIKE</VTab>
          <VTab @click="sort = 1">时间序 / Latest</VTab>
        </VTabs>
      </VCol>
      <VFadeTransition style="width: 100%" group>
        <VCol
          v-for="comment in comments"
          :key="comment.commentId"
          style="padding: 4px"
          cols="12"
        >
          <VCard tile>
            <VCardTitle>
              <h5>{{ comment.user.title }} {{ comment.user.nickName }}:</h5>
              <VSpacer></VSpacer>
              <VChip
                v-if="comment.liked == false"
                pill
                label
                class="ma-2"
                color="red"
                outlined
                @click.stop="postLike(comment.commentId)"
              >
                <VIcon left>mdi-heart-multiple-outline</VIcon>
                {{ comment.like }}
              </VChip>
              <VChip v-else pill outlined label class="ma-2" color="red">
                <VIcon color="red" left>mdi-heart-multiple</VIcon>
                {{ comment.like }}
              </VChip>
            </VCardTitle>
            <VCardText
              style="white-space: pre;"
              class="body-2"
              v-text="getEmoji(comment.content)"
            >
            </VCardText>
            <VCardText class="caption">
              发布时间: {{ $timeFormat(comment.date, "YYYY-MM-DD HH:mm:SS") }}
            </VCardText>
          </VCard>
        </VCol>
      </VFadeTransition>
      <VCol
        v-if="comments.length % pageSize == 0 && comments.length != 0"
        cols="12"
      >
        <VBtn
          color="primary"
          block
          outlined
          @click="getData($route.path, true)"
        >
          下一页 / NEXT</VBtn
        >
      </VCol>
      <VCol cols="12">
        <VCard v-if="$store.getters.getLoginState">
          <VCardTitle>
            <h5>{{ $store.state.nickName }}：</h5>
          </VCardTitle>
          <VCardText>
            <CommentTextArea
              ref="CommentTextArea"
              @getText="updateCommentContent"
            ></CommentTextArea>
            <VBtn block color="primary" @click.stop="postComment"
              ><VIcon>mdi-send</VIcon></VBtn
            >
          </VCardText>
        </VCard>
        <VCard v-else tile>
          <VCardText> 登陆后，且经验值大于100才能发表评论!</VCardText>
          <VCardActions>
            <VSpacer></VSpacer>
            <VBtn color="primary" to="/login" outlined>
              <VIcon left>mdi-login</VIcon> 前往登陆页面</VBtn
            >
          </VCardActions>
        </VCard>
      </VCol>
    </VRow>
  </VContainer>
</template>
<script>
export default {
  components: {
    CommentTextArea: () => import("../biliob/Textarea.vue")
  },
  data() {
    return {
      page: 0,
      pageSize: 20,
      comments: [],
      commentContent: "",
      sort: 0
    };
  },
  watch: {
    $route(to) {
      this.getData(to.path);
    },
    sort() {
      this.comments = [];
      this.getData(this.$route.path);
    }
  },
  mounted() {
    console.log(this.$refs);

    this.getData(this.$route.path);
  },
  methods: {
    getData(path, append) {
      if (!append) {
        this.page = 0;
      } else {
        this.page += 1;
      }
      if (path == "") {
        path = "-";
      }
      this.axios
        .get(
          `/comment?path=${path}&p=${this.page}&ps=${this.pageSize}&s=${this.sort}`
        )
        .then(res => {
          this.comments = res.data;
        });
    },
    postLike(commentId) {
      this.axios.put(`/user/comment/${commentId}/like`).then(() => {
        this.comments.forEach(comment => {
          if (comment.commentId == commentId) {
            comment.like += 1;
            comment.liked = true;
          }
        });
      });
    },
    getEmoji(val) {
      var patt = /&#\d+;/g;
      var H, L, code;
      var arr = val.match(patt) || [];
      for (var i = 0; i < arr.length; i++) {
        code = arr[i];
        code = code.replace("&#", "").replace(";", "");
        H = Math.floor((code - 0x10000) / 0x400) + 0xd800;
        L = ((code - 0x10000) % 0x400) + 0xdc00;
        code = "&#" + code + ";";
        var s = String.fromCharCode(H, L);
        val = val.replace(code, s);
      }
      return val.trim();
    },
    updateCommentContent(text) {
      this.commentContent = text;
    },
    postComment() {
      this.axios
        .post(`/user/comment`, {
          path: this.$route.path,
          content: this.commentContent
        })
        .then(() => {
          this.getData(this.$route.path);
          this.$refs.CommentTextArea.clean();
        });
    }
  }
};
</script>